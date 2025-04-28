# Odoo Module: crm

Category: Sales/CRM

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard

from odoo import api, SUPERUSER_ID


```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'CRM',
    'version': '1.2',
    'category': 'Sales/CRM',
    'sequence': 15,
    'summary': 'Track leads and close opportunities',
    'description': "",
    'website': 'https://www.odoo.com/page/crm',
    'depends': [
        'base_setup',
        'sales_team',
        'mail',
        'calendar',
        'resource',
        'fetchmail',
        'utm',
        'web_tour',
        'contacts',
        'digest',
        'phone_validation',
    ],
    'data': [
        'security/crm_security.xml',
        'security/ir.model.access.csv',

        'data/crm_lead_prediction_data.xml',
        'data/crm_lost_reason_data.xml',
        'data/crm_stage_data.xml',
        'data/crm_team_data.xml',
        'data/digest_data.xml',
        'data/mail_data.xml',
        'data/crm_recurring_plan_data.xml',

        'wizard/crm_lead_lost_views.xml',
        'wizard/crm_lead_to_opportunity_views.xml',
        'wizard/crm_lead_to_opportunity_mass_views.xml',
        'wizard/crm_merge_opportunities_views.xml',

        'views/assets.xml',
        'views/calendar_views.xml',
        'views/crm_recurring_plan_views.xml',
        'views/crm_menu_views.xml',
        'views/crm_lost_reason_views.xml',
        'views/crm_stage_views.xml',
        'views/crm_lead_views.xml',
        'views/digest_views.xml',
        'views/mail_activity_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/utm_campaign_views.xml',
        'report/crm_activity_report_views.xml',
        'report/crm_opportunity_report_views.xml',
        'views/crm_team_views.xml',
    ],
    'demo': [
        'data/crm_team_demo.xml',
        'data/mail_activity_demo.xml',
        'data/crm_lead_demo.xml',
    ],
    'css': ['static/src/css/crm.css'],
    'installable': True,
    'application': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo.addons.mail.controllers.main import MailController
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
                record.convert_opportunity(record.partner_id.id)
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
            Can you send me the details ?</p>]]></field>
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
            <field name="team_id" ref="sales_team.crm_team_1"/>
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
            <field name="team_id" ref="sales_team.crm_team_1"/>
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
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_admin"/>
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
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_admin"/>
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
Could you send me your product catalogue please ?<br />
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
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_demo"/>
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
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_admin"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
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
        <!--Main attachment is usually registered by message_post, so here we must set it manually-->
        <record id="crm_case_15" model="crm.lead">
            <field name="message_main_attachment_id" ref="msg_case15_attach1"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
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
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_demo"/>
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
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
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
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_admin"/>
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
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
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
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor7')])]"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_admin"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=6)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=6)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
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
            <field name="team_id" ref="sales_team.team_sales_department"/>
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
            <field name="date_open" eval="(DateTime.today() - relativedelta(months=2)).strftime('%Y-%m-%d %H:%M')"/>
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
            <field name="date_open" eval="(DateTime.today() - relativedelta(months=2)).strftime('%Y-%m-%d %H:%M')"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
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
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
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
<field name="team_id" ref="sales_team.crm_team_1"/>
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
        <record id="crm_pls_fields_param" model="ir.config_parameter">
            <field name="key">crm.pls_fields</field>
            <field name="value" eval="'state_id,country_id,phone_state,email_state,source_id'"/>
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
        <field name="alias_user_id" ref="base.user_admin"/>
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
        <record id="sales_team.crm_team_1" model="crm.team">
            <field name="use_leads">True</field>
        </record>
    </data>
</odoo>

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
    % set record = object.env['crm.team'].search([('alias_name', '!=', 'False')], limit=1)
    <p class="tip_title">Tip: Convert incoming emails into opportunities</p>
    % if record and record.alias_domain
    <p class="tip_content">Did you know emails sent to ${record.alias_id.display_name} generate opportunities in your pipeline?<br/>
    <a href="mailto:${record.alias_id.display_name}" target="_blank">Try sending an email</a> to your CRM. This email address is configurable by sales team members.</p>
    % else
    <p class="tip_content">Did you know emails sent to a Sales Team alias generate opportunities in your pipeline?</p>
    % endif
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
    <img src="/crm/static/src/img/generate-leads.gif" class="illustration_border" />
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
    <img src="/crm/static/src/img/probability-rate.gif" class="illustration_border" />
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
    <img src="/crm/static/src/img/pipeline-progress.gif" class="illustration_border" />
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
    <img src="/crm/static/src/img/autofill.gif" class="illustration_border" />
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
    <img src="/crm/static/src/img/mapview-toggle.gif" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\mail_activity_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mail_activity_demo_followup_quote" model="mail.activity.type">
        <field name="name">Follow-up Quote</field>
        <field name="icon">fa-file-text-o</field>
        <field name="res_model_id" ref="crm.model_crm_lead"/>
        <field name="delay_count">30</field>
    </record>
    <record id="mail_activity_demo_make_quote" model="mail.activity.type">
        <field name="name">Make Quote</field>
        <field name="icon">fa-file-text-o</field>
        <field name="res_model_id" ref="crm.model_crm_lead"/>
        <field name="delay_count">15</field>
    </record>
    <record id="mail_activity_demo_call_demo" model="mail.activity.type">
        <field name="name">Call for Demo</field>
        <field name="icon">fa-phone</field>
        <field name="res_model_id" ref="crm.model_crm_lead"/>
        <field name="delay_count">10</field>
    </record>

    <record id="mail_template_demo_crm_lead" model="mail.template">
        <field name="name">Welcome Demo</field>
        <field name="model_id" ref="crm.model_crm_lead"/>
        <field name="partner_to">${object.partner_id != False and object.partner_id.id}</field>
        <field name="email_to">${(not object.partner_id and object.email_from)|safe}</field>
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
                    <span style="font-size: 20px; font-weight: bold;">
                        ${object.name}
                    </span>
                </td><td valign="middle" align="right">
                    <img src="/logo.png?company=${object.company_id.id}" style="padding: 0px; margin: 0px; height: 48px;" alt="${object.company_id.name}"/>
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
                            Hi ${object.partner_id and object.partner_id.name or ''},<br/><br/>
                            Welcome to ${object.company_id.name}.
                            It's great to meet you! Now that you're on board, you'll discover what ${object.company_id.name} has to offer. My name is ${object.user_id.name} and I'll help you get the most out of Odoo. Could we plan a quick demo soon?<br/>
                            Feel free to reach out at any time!<br/><br/>
                            Best,<br/>
                            % if object.user_id:
                                <b>${object.user_id.name}</b>
                                <br/>Email: ${object.user_id.email or ''}
                                <br/>Phone: ${object.user_id.phone or ''}
                            % else:
                                ${object.company_id.name}
                            % endif
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
            <b>${object.company_id.name}</b><br/>
            <div style="color: #999999;">
                ${object.company_id.phone}
                % if object.company_id.email
                    | <a href="'mailto:%s' % ${object.company_id.email}" style="text-decoration:none; color: #999999;">${object.company_id.email}</a>
                % endif
                % if object.company_id.website
                    | <a href="'%s' % ${object.company_id.website}" style="text-decoration:none; color: #999999;">${object.company_id.website}</a>
                % endif
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
        <field name="lang">${object.partner_id.lang}</field>
        <field name="auto_delete" eval="True"/>
    </record>

    <record id="mail_activity_type_demo_email_with_template" model="mail.activity.type">
        <field name="name">Email: Welcome Demo</field>
        <field name="icon">fa-envelope</field>
        <field name="res_model_id" ref="crm.model_crm_lead"/>
        <field name="mail_template_ids" eval="[(4, ref('crm.mail_template_demo_crm_lead'))]"/>
    </record>
</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <!--default alias for leads-->
        <record id="mail_alias_lead_info" model="mail.alias">
            <field name="alias_name"></field>
            <field name="alias_model_id" ref="model_crm_lead"/>
            <field name="alias_user_id" ref="base.user_admin"/>
            <field name="alias_parent_model_id" ref="model_crm_team"/>
        </record>

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

        <!--Definition of an email template with an empty body that will be used in opportunity mailing.
            Used to give a basis for email recipients, name and to ease the definition of a further
            elaborated template.  -->
        <record id="email_template_opportunity_mail" model="mail.template">
            <field name="name">Lead/Opportunity: Mass Mail</field>
            <field name="model_id" ref="crm.model_crm_lead"/>
            <field name="partner_to">${object.partner_id != False and object.partner_id.id}</field>
            <field name="email_to">${(not object.partner_id and object.email_from)|safe}</field>
            <field name="body_html"></field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
        </record>

    </data>
</odoo>

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
                event.opportunity_id.log_meeting(event.name, event.start, event.duration)
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
import re
import threading
from datetime import date, datetime, timedelta
from psycopg2 import sql

from odoo import api, fields, models, tools, SUPERUSER_ID
from odoo.osv import expression
from odoo.tools.translate import _
from odoo.tools import email_split
from odoo.exceptions import UserError, AccessError
from odoo.addons.phone_validation.tools import phone_validation
from collections import OrderedDict, defaultdict

from . import crm_stage

_logger = logging.getLogger(__name__)

CRM_LEAD_FIELDS_TO_MERGE = [
    'name',
    'partner_id',
    'campaign_id',
    'company_id',
    'country_id',
    'team_id',
    'state_id',
    'stage_id',
    'medium_id',
    'source_id',
    'user_id',
    'title',
    'city',
    'contact_name',
    'description',
    'mobile',
    'partner_name',
    'phone',
    'probability',
    'expected_revenue',
    'street',
    'street2',
    'zip',
    'create_date',
    'date_action_last',
    'email_from',
    'email_cc',
    'website']

# Subset of partner fields: sync any of those
PARTNER_FIELDS_TO_SYNC = [
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
                'phone.validation.mixin']
    _primary_email = 'email_from'

    # Description
    name = fields.Char(
        'Opportunity', index=True, required=True,
        compute='_compute_name', readonly=False, store=True)
    user_id = fields.Many2one('res.users', string='Salesperson', index=True, tracking=True, default=lambda self: self.env.user)
    user_email = fields.Char('User Email', related='user_id.email', readonly=True)
    user_login = fields.Char('User Login', related='user_id.login', readonly=True)
    company_id = fields.Many2one('res.company', string='Company', index=True, default=lambda self: self.env.company.id)
    referred = fields.Char('Referred By')
    description = fields.Text('Notes')
    active = fields.Boolean('Active', default=True, tracking=True)
    type = fields.Selection([
        ('lead', 'Lead'), ('opportunity', 'Opportunity')],
        index=True, required=True, tracking=15,
        default=lambda self: 'lead' if self.env['res.users'].has_group('crm.group_use_lead') else 'opportunity')
    priority = fields.Selection(
        crm_stage.AVAILABLE_PRIORITIES, string='Priority', index=True,
        default=crm_stage.AVAILABLE_PRIORITIES[0][0])
    team_id = fields.Many2one(
        'crm.team', string='Sales Team', index=True, tracking=True,
        compute='_compute_team_id', readonly=False, store=True)
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
    activity_date_deadline_my = fields.Date(
        'My Activities Deadline', compute='_compute_activity_date_deadline_my',
        search='_search_activity_date_deadline_my', compute_sudo=False,
        readonly=True, store=False, groups="base.group_user")
    tag_ids = fields.Many2many(
        'crm.tag', 'crm_tag_rel', 'lead_id', 'tag_id', string='Tags',
        help="Classify and analyze your lead/opportunity categories like: Training, Service")
    color = fields.Integer('Color Index', default=0)
    # Opportunity specific
    expected_revenue = fields.Monetary('Expected Revenue', currency_field='company_currency', tracking=True)
    prorated_revenue = fields.Monetary('Prorated Revenue', currency_field='company_currency', store=True, compute="_compute_prorated_revenue")
    recurring_revenue = fields.Monetary('Recurring Revenues', currency_field='company_currency', groups="crm.group_use_recurring_revenues")
    recurring_plan = fields.Many2one('crm.recurring.plan', string="Recurring Plan", groups="crm.group_use_recurring_revenues")
    recurring_revenue_monthly = fields.Monetary('Expected MRR', currency_field='company_currency', store=True,
                                               compute="_compute_recurring_revenue_monthly",
                                               groups="crm.group_use_recurring_revenues")
    recurring_revenue_monthly_prorated = fields.Monetary('Prorated MRR', currency_field='company_currency', store=True,
                                               compute="_compute_recurring_revenue_monthly_prorated",
                                               groups="crm.group_use_recurring_revenues")
    company_currency = fields.Many2one("res.currency", string='Currency', related='company_id.currency_id', readonly=True)
    # Dates
    date_closed = fields.Datetime('Closed Date', readonly=True, copy=False)
    date_action_last = fields.Datetime('Last Action', readonly=True)
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
        'res.partner', string='Customer', index=True, tracking=10,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="Linked partner (optional). Usually created when converting the lead. You can find a partner by its Name, TIN, Email or Internal Reference.")
    partner_is_blacklisted = fields.Boolean('Partner is blacklisted', related='partner_id.is_blacklisted', readonly=True)
    contact_name = fields.Char(
        'Contact Name', tracking=30,
        compute='_compute_contact_name', readonly=False, store=True)
    partner_name = fields.Char(
        'Company Name', tracking=20, index=True,
        compute='_compute_partner_name', readonly=False, store=True,
        help='The name of the future partner company that will be created while converting the lead into opportunity')
    function = fields.Char('Job Position', compute='_compute_function', readonly=False, store=True)
    title = fields.Many2one('res.partner.title', string='Title', compute='_compute_title', readonly=False, store=True)
    email_from = fields.Char(
        'Email', tracking=40, index=True,
        compute='_compute_email_from', inverse='_inverse_email_from', readonly=False, store=True)
    phone = fields.Char(
        'Phone', tracking=50,
        compute='_compute_phone', inverse='_inverse_phone', readonly=False, store=True)
    mobile = fields.Char('Mobile', compute='_compute_mobile', readonly=False, store=True)
    phone_mobile_search = fields.Char('Phone/Mobile', store=False, search='_search_phone_mobile_search')
    phone_state = fields.Selection([
        ('correct', 'Correct'),
        ('incorrect', 'Incorrect')], string='Phone Quality', compute="_compute_phone_state", store=True)
    email_state = fields.Selection([
        ('correct', 'Correct'),
        ('incorrect', 'Incorrect')], string='Email Quality', compute="_compute_email_state", store=True)
    website = fields.Char('Website', index=True, help="Website of the contact", compute="_compute_website", readonly=False, store=True)
    lang_id = fields.Many2one(
        'res.lang', string='Language',
        compute='_compute_lang_id', readonly=False, store=True)
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
    # External records
    meeting_count = fields.Integer('# Meetings', compute='_compute_meeting_count')
    lost_reason = fields.Many2one(
        'crm.lost.reason', string='Lost Reason',
        index=True, ondelete='restrict', tracking=True)
    ribbon_message = fields.Char('Ribbon message', compute='_compute_ribbon_message')

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

    @api.depends('activity_ids.date_deadline')
    @api.depends_context('uid')
    def _compute_activity_date_deadline_my(self):
        todo_activities = []
        if self.ids:
            todo_activities = self.env['mail.activity'].search([
                ('user_id', '=', self._uid),
                ('res_model', '=', self._name),
                ('res_id', 'in', self.ids)
            ], order='date_deadline ASC')

        for record in self:
            record.activity_date_deadline_my = next(
                (activity.date_deadline for activity in todo_activities if activity.res_id == record.id),
                False
            )

    def _search_activity_date_deadline_my(self, operator, operand):
        return ['&', ('activity_ids.user_id', '=', self._uid), ('activity_ids.date_deadline', operator, operand)]

    @api.depends('user_id', 'type')
    def _compute_team_id(self):
        """ When changing the user, also set a team_id or restrict team id
        to the ones user_id is member of. """
        for lead in self:
            # setting user as void should not trigger a new team computation
            if not lead.user_id:
                continue
            user = lead.user_id
            if lead.team_id and user in lead.team_id.member_ids | lead.team_id.user_id:
                continue
            team_domain = [('use_leads', '=', True)] if lead.type == 'lead' else [('use_opportunities', '=', True)]
            team = self.env['crm.team']._get_default_team_id(user_id=user.id, domain=team_domain)
            lead.team_id = team.id

    @api.depends('team_id', 'type')
    def _compute_stage_id(self):
        for lead in self:
            if not lead.stage_id:
                lead.stage_id = lead._stage_find(domain=[('fold', '=', False)]).id

    @api.depends('user_id')
    def _compute_date_open(self):
        for lead in self:
            lead.date_open = fields.Datetime.now() if lead.user_id else False

    @api.depends('stage_id')
    def _compute_date_last_stage_update(self):
        for lead in self:
            lead.date_last_stage_update = fields.Datetime.now()

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
        """ compute the lang based on partner when partner_id has changed """
        wo_lang = self.filtered(lambda lead: not lead.lang_id and lead.partner_id)
        if not wo_lang:
            return
        # prepare cache
        lang_codes = [code for code in wo_lang.mapped('partner_id.lang') if code]
        lang_id_by_code = dict(
            (code, self.env['res.lang']._lang_get_id(code))
            for code in lang_codes
        )
        for lead in wo_lang:
            lead.lang_id = lang_id_by_code.get(lead.partner_id.lang, False)

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
                    if tools.email_normalize(email):
                        email_state = 'correct'
                        break
            lead.email_state = email_state

    @api.depends('probability', 'automated_probability')
    def _compute_is_automated_probability(self):
        """ If probability and automated_probability are equal probability computation
        is considered as automatic, aka probability is sync with automated_probability """
        for lead in self:
            lead.is_automated_probability = tools.float_compare(lead.probability, lead.automated_probability, 2) == 0

    @api.depends(lambda self: ['tag_ids', 'stage_id', 'team_id'] + self._pls_get_safe_fields())
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

    def _compute_meeting_count(self):
        if self.ids:
            meeting_data = self.env['calendar.event'].sudo().read_group([
                ('opportunity_id', 'in', self.ids)
            ], ['opportunity_id'], ['opportunity_id'])
            mapped_data = {m['opportunity_id'][0]: m['opportunity_id_count'] for m in meeting_data}
        else:
            mapped_data = dict()
        for lead in self:
            lead.meeting_count = mapped_data.get(lead.id, 0)

    @api.depends('email_from', 'phone', 'partner_id')
    def _compute_ribbon_message(self):
        for lead in self:
            will_write_email = lead._get_partner_email_update()
            will_write_phone = lead._get_partner_phone_update()

            if will_write_email and will_write_phone:
                lead.ribbon_message = _('By saving this change, the customer email and phone number will also be updated.')
            elif will_write_email:
                lead.ribbon_message = _('By saving this change, the customer email will also be updated.')
            elif will_write_phone:
                lead.ribbon_message = _('By saving this change, the customer phone number will also be updated.')
            else:
                lead.ribbon_message = False

    def _search_phone_mobile_search(self, operator, value):
        value = re.sub(r'[^\d+]+', '', value)
        if len(value) <= 2:
            raise UserError(_('Please enter at least 3 digits when searching on phone / mobile.'))

        query = f"""
                SELECT model.id
                FROM {self._table} model
                WHERE REGEXP_REPLACE(model.phone, '[^\d+]+', '', 'g') SIMILAR TO CONCAT(%s, REGEXP_REPLACE(%s, '\D+', '', 'g'), '%%')
                  OR REGEXP_REPLACE(model.mobile, '[^\d+]+', '', 'g') SIMILAR TO CONCAT(%s, REGEXP_REPLACE(%s, '\D+', '', 'g'), '%%')
            """

        # searching on +32485112233 should also finds 00485112233 (00 / + prefix are both valid)
        # we therefore remove it from input value and search for both of them in db
        if value.startswith('+') or value.startswith('00'):
            if value.startswith('00'):
                value = value[2:]
            starts_with = '00|\+'
        else:
            starts_with = '%'

        self._cr.execute(query, (starts_with, value, starts_with, value))
        res = self._cr.fetchall()
        if not res:
            return [(0, '=', 1)]
        return [('id', 'in', [r[0] for r in res])]

    @api.onchange('phone', 'country_id', 'company_id')
    def _onchange_phone_validation(self):
        if self.phone:
            self.phone = self.phone_format(self.phone)

    @api.onchange('mobile', 'country_id', 'company_id')
    def _onchange_mobile_validation(self):
        if self.mobile:
            self.mobile = self.phone_format(self.mobile)

    def _prepare_values_from_partner(self, partner):
        """ Get a dictionary with values coming from partner information to
        copy on a lead. Non-address fields get the current lead
        values to avoid being reset if partner has no value for them. """

        # Sync all address fields from partner, or none, to avoid mixing them.
        values = self._prepare_address_values_from_partner(partner)

        # For other fields, get the info from the partner, but only if set
        values.update({f: partner[f] or self[f] for f in PARTNER_FIELDS_TO_SYNC})
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
        partner_name = partner.parent_id.name
        if not partner_name and partner.is_company:
            partner_name = partner.name
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
            lead_phone_formatted = self.phone_format(self.phone) if self.phone else False or self.phone or False
            partner_phone_formatted = self.phone_format(self.partner_id.phone) if self.partner_id.phone else False or self.partner_id.phone or False
            return lead_phone_formatted != partner_phone_formatted
        return False

    # ------------------------------------------------------------
    # ORM
    # ------------------------------------------------------------

    def _auto_init(self):
        res = super(Lead, self)._auto_init()
        tools.create_index(self._cr, 'crm_lead_user_id_team_id_type_index',
                           self._table, ['user_id', 'team_id', 'type'])
        tools.create_index(self._cr, 'crm_lead_create_date_team_id_idx',
                           self._table, ['create_date', 'team_id'])
        return res

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

        stage_updated, stage_is_won = vals.get('stage_id'), False
        # stage change: update date_last_stage_update
        if stage_updated:
            stage = self.env['crm.stage'].browse(vals['stage_id'])
            if stage.is_won:
                vals.update({'probability': 100, 'automated_probability': 100})
                stage_is_won = True

        # stage change with new stage: update probability and date_closed
        if vals.get('probability', 0) >= 100 or not vals.get('active', True):
            vals['date_closed'] = fields.Datetime.now()
        elif tools.float_compare(vals.get('probability', 0), 0, precision_digits=2) > 0:
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
    def search(self, args, offset=0, limit=None, order=None, count=False):
        """ Override to support ordering on activity_date_deadline_my.

        Ordering through web client calls search_read with an order parameter set.
        Search_read then calls search. In this override we therefore override search
        to intercept a search without count with an order on activity_date_deadline_my.
        In that case we do the search in two steps.

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
        if count or not order or 'activity_date_deadline_my' not in order:
            return super(Lead, self).search(args, offset=offset, limit=limit, order=order, count=count)
        order_items = [order_item.strip().lower() for order_item in (order or self._order).split(',')]

        # Perform a read_group on my activities to get a mapping lead_id / deadline
        # Remember date_deadline is required, we always have a value for it. Only
        # the earliest deadline per lead is kept.
        activity_asc = any('activity_date_deadline_my asc' in item for item in order_items)
        my_lead_activities = self.env['mail.activity'].read_group(
            [('res_model', '=', self._name), ('user_id', '=', self.env.uid)],
            ['res_id', 'date_deadline:min'],
            ['res_id'],
            orderby='date_deadline ASC'
        )
        my_lead_mapping = dict((item['res_id'], item['date_deadline']) for item in my_lead_activities)
        my_lead_ids = list(my_lead_mapping.keys())
        my_lead_domain = expression.AND([[('id', 'in', my_lead_ids)], args])
        my_lead_order = ', '.join(item for item in order_items if 'activity_date_deadline_my' not in item)

        # Search leads linked to those activities and order them. See docstring
        # of this method for more details.
        search_res = super(Lead, self).search(my_lead_domain, offset=0, limit=None, order=my_lead_order, count=count)
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
        # activity_date_deadline_my when calling super() .
        lead_limit = (limit - len(my_lead_ids_keep)) if limit else None
        if offset:
            lead_offset = max((offset - len(search_res), 0))
        else:
            lead_offset = 0
        lead_order = ', '.join(item for item in order_items if 'activity_date_deadline_my' not in item)

        other_lead_res = super(Lead, self).search(
            expression.AND([[('id', 'not in', my_lead_ids_skip)], args]),
            offset=lead_offset, limit=lead_limit, order=lead_order, count=count
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
                        leads_leave_lost |= lead
                    leads_reach_won |= lead
                elif lead.stage_id.id in won_stage_ids and lead.active:  # a lead can be lost at won_stage
                    leads_leave_won |= lead
            if 'active' in vals:
                if not vals['active'] and lead.active:  # archive lead
                    if lead.stage_id.id in won_stage_ids and lead not in leads_leave_won:
                        leads_leave_won |= lead
                    leads_reach_lost |= lead
                elif vals['active'] and not lead.active:  # restore lead
                    leads_leave_lost |= lead

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
        default['date_open'] = fields.Datetime.now() if self.type == 'opportunity' else False
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
    def _fields_view_get(self, view_id=None, view_type='form', toolbar=False, submenu=False):
        if self._context.get('opportunity_id'):
            opportunity = self.browse(self._context['opportunity_id'])
            action = opportunity.get_formview_action()
            if action.get('views') and any(view_id for view_id in action['views'] if view_id[1] == view_type):
                view_id = next(view_id[0] for view_id in action['views'] if view_id[1] == view_type)
        res = super(Lead, self)._fields_view_get(view_id=view_id, view_type=view_type, toolbar=toolbar, submenu=submenu)
        if view_type == 'form':
            res['arch'] = self._fields_view_get_address(res['arch'])
        return res

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

    def _stage_find(self, team_id=False, domain=None, order='sequence'):
        """ Determine the stage of the current lead with its teams, the given domain and the given team_id
            :param team_id
            :param domain : base search domain for stage
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
        return self.env['crm.stage'].search(search_domain, order=order, limit=1)

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
            activated.write({'lost_reason': False})
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
            stage_id = lead._stage_find(domain=[('is_won', '=', True)])
            if stage_id in leads_by_won_stage:
                leads_by_won_stage[stage_id] |= lead
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
                    'img_url': '/web/image/%s/%s/image_1024' % (self.team_id.user_id._name, self.team_id.user_id.id) if self.team_id.user_id.image_1024 else '/web/static/src/img/smile.svg',
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
        message = False
        if self.user_id and self.team_id and self.expected_revenue:
            self.flush()  # flush fields to make sure DB is up to date
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

    def action_schedule_meeting(self):
        """ Open meeting's calendar view to schedule meeting on current opportunity.
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
            'default_attendee_ids': [(0, 0, {'partner_id': pid}) for pid in partner_ids],
            'default_team_id': self.team_id.id,
            'default_name': self.name,
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
    # BUSINESS
    # ------------------------------------------------------------

    def log_meeting(self, meeting_subject, meeting_date, duration):
        if not duration:
            duration = _('unknown')
        else:
            duration = str(duration)
        meet_date = fields.Datetime.from_string(meeting_date)
        meeting_usertime = fields.Datetime.to_string(fields.Datetime.context_timestamp(self, meet_date))
        html_time = "<time datetime='%s+00:00'>%s</time>" % (meeting_date, meeting_usertime)
        message = _("Meeting scheduled at '%s'<br> Subject: %s <br> Duration: %s hours") % (html_time, meeting_subject, duration)
        return self.message_post(body=message)

    # ------------------------------------------------------------
    # MERGE LEADS / OPPS
    # ------------------------------------------------------------

    def _merge_get_result_type(self):
        """ Define the type of the result of the merge.  If at least one of the
        element to merge is an opp, the resulting new element will be an opp.
        Otherwise it will be a lead. """
        if any(record.type == 'opportunity' for record in self):
            return 'opportunity'
        return 'lead'

    def _merge_data(self, fields):
        """ Prepare lead/opp data into a dictionary for merging. Different types
            of fields are processed in different ways:
                - text: all the values are concatenated
                - m2m and o2m: those fields aren't processed
                - m2o: the first not null value prevails (the other are dropped)
                - any other type of field: same as m2o

            :param fields: list of fields to process
            :return dict data: contains the merged values of the new opportunity
        """
        # helpers
        def _get_first_not_null(attr, opportunities):
            for opp in opportunities:
                val = opp[attr]
                if val:
                    return val
            return False

        def _get_first_not_null_id(attr, opportunities):
            res = _get_first_not_null(attr, opportunities)
            return res.id if res else False

        # process the fields' values
        data = {}
        for field_name in fields:
            field = self._fields.get(field_name)
            if field is None:
                continue
            if field.type in ('many2many', 'one2many'):
                continue
            elif field.type == 'many2one':
                data[field_name] = _get_first_not_null_id(field_name, self)  # take the first not null
            elif field.type == 'text':
                data[field_name] = '\n\n'.join(it for it in self.mapped(field_name) if it)
            else:
                data[field_name] = _get_first_not_null(field_name, self)

        # define the resulting type ('lead' or 'opportunity')
        data['type'] = self._merge_get_result_type()
        return data

    def _merge_notify_get_merged_fields_message(self, fields):
        """ Generate the message body with the changed values

        :param fields : list of fields to track
        :returns a list of message bodies for the corresponding leads
        """
        bodies = []
        for lead in self:
            title = "%s : %s\n" % (_('Merged opportunity') if lead.type == 'opportunity' else _('Merged lead'), lead.name)
            body = [title]
            _fields = self.env['ir.model.fields'].search([
                ('name', 'in', fields or []),
                ('model_id.model', '=', lead._name),
            ])
            for field in _fields:
                value = getattr(lead, field.name, False)
                if field.ttype == 'selection':
                    selections = lead.fields_get()[field.name]['selection']
                    value = next((v[1] for v in selections if v[0] == value), value)
                elif field.ttype == 'many2one':
                    if value:
                        value = value.sudo().display_name
                elif field.ttype == 'many2many':
                    if value:
                        value = ','.join(
                            val.display_name
                            for val in value.sudo()
                        )
                body.append("%s: %s" % (field.field_description, value or ''))
            bodies.append("<br/>".join(body + ['<br/>']))
        return bodies

    def _merge_notify(self, opportunities):
        """ Post a message gathering merged leads/opps informations. It explains
        which fields has been merged and their new value. `self` is the resulting
        merge crm.lead record.

        :param opportunities: see ``merge_dependences``
        """
        # TODO JEM: mail template should be used instead of fix body, subject text
        self.ensure_one()
        # mail message's subject
        result_type = opportunities._merge_get_result_type()
        merge_message = _('Merged leads') if result_type == 'lead' else _('Merged opportunities')
        subject = merge_message + ": " + ", ".join(opportunities.mapped('name'))
        # message bodies
        message_bodies = opportunities._merge_notify_get_merged_fields_message(list(CRM_LEAD_FIELDS_TO_MERGE))
        message_body = "\n\n".join(message_bodies)
        return self.message_post(body=message_body, subject=subject)

    def _merge_opportunity_history(self, opportunities):
        """ Move mail.message from the given opportunities to the current one. `self` is the
            crm.lead record destination for message of `opportunities`.

        :param opportunities: see ``merge_dependences``
        """
        self.ensure_one()
        for opportunity in opportunities:
            for message in opportunity.message_ids:
                if message.subject:
                    subject = _("From %(source_name)s : %(source_subject)s", source_name=opportunity.name, source_subject=message.subject)
                else:
                    subject = _("From %(source_name)s", source_name=opportunity.name)
                message.write({
                    'res_id': self.id,
                    'subject': subject,
                })
        return True

    def _merge_opportunity_attachments(self, opportunities):
        """ Move attachments of given opportunities to the current one `self`, and rename
            the attachments having same name than native ones.

        :param opportunities: see ``merge_dependences``
        """
        self.ensure_one()

        # return attachments of opportunity
        def _get_attachments(opportunity_id):
            return self.env['ir.attachment'].search([('res_model', '=', self._name), ('res_id', '=', opportunity_id)])

        first_attachments = _get_attachments(self.id)
        # counter of all attachments to move. Used to make sure the name is different for all attachments
        count = 1
        for opportunity in opportunities:
            attachments = _get_attachments(opportunity.id)
            for attachment in attachments:
                values = {'res_id': self.id}
                for attachment_in_first in first_attachments:
                    if attachment.name == attachment_in_first.name:
                        values['name'] = "%s (%s)" % (attachment.name, count)
                count += 1
                attachment.write(values)
        return True

    def merge_dependences(self, opportunities):
        """ Merge dependences (messages, attachments, ...). These dependences will be
            transfered to `self`, the most important lead.

        :param opportunities : recordset of opportunities to transfer. Does not
          include `self` which is the target crm.lead being the result of the merge.
        """
        self.ensure_one()
        self._merge_notify(opportunities)
        self._merge_opportunity_history(opportunities)
        self._merge_opportunity_attachments(opportunities)

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
        if len(self.ids) <= 1:
            raise UserError(_('Please select more than one element (lead or opportunity) from the list view.'))

        if len(self.ids) > 5 and not self.env.is_superuser():
            raise UserError(_("To prevent data loss, Leads and Opportunities can only be merged by groups of 5."))

        opportunities = self._sort_by_confidence_level(reverse=True)

        # get SORTED recordset of head and tail, and complete list
        opportunities_head = opportunities[0]
        opportunities_tail = opportunities[1:]

        # merge all the sorted opportunity. This means the value of
        # the first (head opp) will be a priority.
        merged_data = opportunities._merge_data(list(CRM_LEAD_FIELDS_TO_MERGE))

        # force value for saleperson and Sales Team
        if user_id:
            merged_data['user_id'] = user_id
        if team_id:
            merged_data['team_id'] = team_id

        # merge other data (mail.message, attachments, ...) from tail into head
        opportunities_head.merge_dependences(opportunities_tail)

        # check if the stage is in the stages of the Sales Team. If not, assign the stage with the lowest sequence
        if merged_data.get('team_id'):
            team_stage_ids = self.env['crm.stage'].search(['|', ('team_id', '=', merged_data['team_id']), ('team_id', '=', False)], order='sequence')
            if merged_data.get('stage_id') not in team_stage_ids.ids:
                merged_data['stage_id'] = team_stage_ids[0].id if team_stage_ids else False

        # write merged data into first opportunity
        opportunities_head.write(merged_data)

        # delete tail opportunities
        # we use the SUPERUSER to avoid access rights issues because as the user had the rights to see the records it should be safe to do so
        if auto_unlink:
            opportunities_tail.sudo().unlink()

        return opportunities_head

    def _sort_by_confidence_level(self, reverse=False):
        """ Sorting the leads/opps according to the confidence level of its stage, which relates to the probability of winning it
        The confidence level increases with the stage sequence
        An Opportunity always has higher confidence level than a lead
        """
        def opps_key(opportunity):
            return opportunity.type == 'opportunity', opportunity.stage_id.sequence, -opportunity._origin.id

        return self.sorted(key=opps_key, reverse=reverse)

    def _convert_opportunity_data(self, customer, team_id=False):
        """ Extract the data from a lead to create the opportunity
            :param customer : res.partner record
            :param team_id : identifier of the Sales Team to determine the stage
        """
        new_team_id = team_id if team_id else self.team_id.id
        upd_values = {
            'type': 'opportunity',
            'date_open': fields.Datetime.now(),
            'date_conversion': fields.Datetime.now(),
        }
        if customer != self.partner_id:
            upd_values['partner_id'] = customer.id if customer else False
        if not self.stage_id:
            stage = self._stage_find(team_id=new_team_id)
            upd_values['stage_id'] = stage.id
        return upd_values

    def convert_opportunity(self, partner_id, user_ids=False, team_id=False):
        customer = False
        if partner_id:
            customer = self.env['res.partner'].browse(partner_id)
        for lead in self:
            if not lead.active or lead.probability == 100:
                continue
            vals = lead._convert_opportunity_data(customer, team_id)
            lead.write(vals)

        if user_ids or team_id:
            self.handle_salesmen_assignment(user_ids, team_id)

        return True

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
            domain += ['&', ('active', '=', True), '|', ('probability', '=', False), ('probability', '<', 100)]

        return self.with_context(active_test=False).search(domain)

    def _create_customer(self):
        """ Create a partner from lead data and link it to the lead.

        :return: newly-created partner browse record
        """
        Partner = self.env['res.partner']
        contact_name = self.contact_name
        if not contact_name:
            contact_name = Partner._parse_partner_name(self.email_from)[0] if self.email_from else False

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

    def _prepare_customer_values(self, partner_name, is_company=False, parent_id=False):
        """ Extract data from lead to create a partner.

        :param name : furtur name of the partner
        :param is_company : True if the partner is a company
        :param parent_id : id of the parent partner (False if no parent)

        :return: dictionary of values to give at res_partner.create()
        """
        email_split = tools.email_split(self.email_from)
        res = {
            'name': partner_name,
            'user_id': self.env.context.get('default_user_id') or self.user_id.id,
            'comment': self.description,
            'team_id': self.team_id.id,
            'parent_id': parent_id,
            'phone': self.phone,
            'mobile': self.mobile,
            'email': email_split[0] if email_split else False,
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
        if self.lang_id:
            res['lang'] = self.lang_id.code
        return res

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
                partner = self.env['res.partner'].search([('name', 'ilike', '%' + customer_potential_name + '%')], limit=1)
                if partner:
                    break

        return partner

    def handle_partner_assignment(self, force_partner_id=False, create_missing=True):
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

    def handle_salesmen_assignment(self, user_ids=None, team_id=False):
        """ Assign salesmen and salesteam to a batch of leads.  If there are more
        leads than salesmen, these salesmen will be assigned in round-robin. E.g.
        4 salesmen (S1, S2, S3, S4) for 6 leads (L1, L2, ... L6) will assigned as
        following: L1 - S1, L2 - S2, L3 - S3, L4 - S4, L5 - S1, L6 - S2.

        :param list user_ids: salesmen to assign
        :param int team_id: salesteam to assign
        """
        update_vals = {'team_id': team_id} if team_id else {}
        if not user_ids:
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
    # TOOLS
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
    def get_empty_list_help(self, help):
        help_title, sub_title = "", ""
        if self._context.get('default_type') == 'lead':
            help_title = _('Create a new lead')
        else:
            help_title = _('Create an opportunity to start playing with your pipeline.')
        alias_record = self.env['mail.alias'].search([
            ('alias_name', '!=', False),
            ('alias_name', '!=', ''),
            ('alias_model_id.model', '=', 'crm.lead'),
            ('alias_parent_model_id.model', '=', 'crm.team'),
            ('alias_force_thread_id', '=', False)
        ], limit=1)
        if alias_record and alias_record.alias_domain and alias_record.alias_name:
            email = '%s@%s' % (alias_record.alias_name, alias_record.alias_domain)
            email_link = "<b><a href='mailto:%s'>%s</a></b>" % (email, email)
            sub_title = _('Use the top left <i>Create</i> button, or send an email to %s to test the email gateway.') % (email_link)
        return '<p class="o_view_nocontent_smiling_face">%s</p><p class="oe_view_nocontent_alias">%s</p>' % (help_title, sub_title)

    # ------------------------------------------------------------
    # MAILING
    # ------------------------------------------------------------

    def _creation_subtype(self):
        return self.env.ref('crm.mt_lead_create')

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'stage_id' in init_values and self.probability == 100 and self.stage_id:
            return self.env.ref('crm.mt_lead_won')
        elif 'lost_reason' in init_values and self.lost_reason:
            return self.env.ref('crm.mt_lead_lost')
        elif 'stage_id' in init_values:
            return self.env.ref('crm.mt_lead_stage')
        elif 'active' in init_values and self.active:
            return self.env.ref('crm.mt_lead_restored')
        elif 'active' in init_values and not self.active:
            return self.env.ref('crm.mt_lead_lost')
        return super(Lead, self)._track_subtype(init_values)

    def _notify_get_groups(self, msg_vals=None):
        """ Handle salesman recipients that can convert leads into opportunities
        and set opportunities as won / lost. """
        groups = super(Lead, self)._notify_get_groups(msg_vals=msg_vals)
        local_msg_vals = dict(msg_vals or {})

        self.ensure_one()
        if self.type == 'lead':
            convert_action = self._notify_get_action_link('controller', controller='/lead/convert', **local_msg_vals)
            salesman_actions = [{'url': convert_action, 'title': _('Convert to opportunity')}]
        else:
            won_action = self._notify_get_action_link('controller', controller='/lead/case_mark_won', **local_msg_vals)
            lost_action = self._notify_get_action_link('controller', controller='/lead/case_mark_lost', **local_msg_vals)
            salesman_actions = [
                {'url': won_action, 'title': _('Won')},
                {'url': lost_action, 'title': _('Lost')}]

        if self.team_id:
            custom_params = dict(local_msg_vals, res_id=self.team_id.id, model=self.team_id._name)
            salesman_actions.append({
                'url': self._notify_get_action_link('view', **custom_params),
                'title': _('Sales Team Settings')
            })

        salesman_group_id = self.env.ref('sales_team.group_sale_salesman').id
        new_group = (
            'group_sale_salesman', lambda pdata: pdata['type'] == 'user' and salesman_group_id in pdata['groups'], {
                'actions': salesman_actions,
            })

        return [new_group] + groups

    def _notify_get_reply_to(self, default=None, records=None, company=None, doc_names=None):
        """ Override to set alias of lead and opportunities to their sales team if any. """
        aliases = self.mapped('team_id').sudo()._notify_get_reply_to(default=default, records=None, company=company, doc_names=None)
        res = {lead.id: aliases.get(lead.team_id.id) for lead in self}
        leftover = self.filtered(lambda rec: not rec.team_id)
        if leftover:
            res.update(super(Lead, leftover)._notify_get_reply_to(default=default, records=None, company=company, doc_names=doc_names))
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
                if lead.partner_id:
                    lead._message_add_suggested_recipient(recipients, partner=lead.partner_id, reason=_('Customer'))
                elif lead.email_from:
                    lead._message_add_suggested_recipient(recipients, email=lead.email_from, reason=_('Customer Email'))
        except AccessError:  # no read access rights -> just ignore suggested recipients because this imply modifying followers
            pass
        return recipients

    @api.model
    def message_new(self, msg_dict, custom_values=None):
        """ Overrides mail_thread message_new that is called by the mailgateway
            through message_process.
            This override updates the document according to the email.
        """

        # remove external users
        if self.env.user.has_group('base.group_portal'):
            self = self.with_context(default_user_id=False)

        # remove default author when going through the mail gateway. Indeed we
        # do not want to explicitly set user_id to False; however we do not
        # want the gateway user to be responsible if no other responsible is
        # found.
        if self._uid == self.env.ref('base.user_root').id:
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

        # assign right company
        if 'company_id' not in defaults and 'team_id' in defaults:
            defaults['company_id'] = self.env['crm.team'].browse(defaults['team_id']).company_id.id
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
        for email, partner_info in zip(emails, result):
            if partner_info.get('partner_id') or not email or not (self.partner_name or self.contact_name):
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

    def _phone_get_number_fields(self):
        """ Use mobile or phone fields to compute sanitized phone number """
        return ['mobile', 'phone']

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
    # The frequencies are split by team_id, so each team has his own frequencies environment. (Team A doesn't impact B)
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

        # get all variable related records from frequency table, no matter the team_id
        frequencies = self.env['crm.lead.scoring.frequency'].search([('variable', 'in', list(leads_fields))], order="team_id asc")

        # get all team_ids from frequencies
        frequency_teams = frequencies.mapped('team_id')
        frequency_team_ids = [0] + [team.id for team in frequency_teams]

        # 1. Compute each variable value count individually
        # regroup each variable to be able to compute their own probabilities
        # As all the variable does not enter into account (as we reject unset values in the process)
        # each value probability must be computed only with their own variable related total count
        # special case: for lead for which team_id is not in frequency table,
        # we consider all the records, independently from team_id (this is why we add a result[-1])
        result = dict((team_id, dict((field, dict(won_total=0, lost_total=0)) for field in leads_fields)) for team_id in frequency_team_ids)
        result[-1] = dict((field, dict(won_total=0, lost_total=0)) for field in leads_fields)
        for frequency in frequencies:
            team_result = result[frequency.team_id.id if frequency.team_id else 0]

            field = frequency['variable']
            value = frequency['value']

            # To avoid that a tag take to much importance if his subset is too small,
            # we ignore the tag frequencies if we have less than 50 won or lost for this tag.
            if field == 'tag_id' and (frequency['won_count'] + frequency['lost_count']) < 50:
                continue

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

            lead_team_id = lead_values['team_id'] if lead_values['team_id'] else 0  # team_id = None -> Convert to 0
            lead_team_id = lead_team_id if lead_team_id in result else -1  # team_id not in frequency Table -> convert to -1
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
        In this case, the lost count should be de-increment by 1 for each PLS parameter linked ot the lead.

        Live increment must be done before writing the new values because we need to know the state change (from and to).
        This would not be an issue for the reach won or reach lost as we just need to increment the frequencies with the
        final state of the lead.
        This issue is when the lead leaves a closed state because once the new values have been writen, we do not know
        what was the previous state that we need to decrement.
        This is why 'is_won' and 'decrement' parameters are used to describe the from / to change of his state.
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
        auto_commit = not getattr(threading.currentThread(), 'testing', False)
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
        leads_pls_fields = list(leads_pls_fields)

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
        first_stage_id = self.env['crm.stage'].search([('team_id', '=', False)], order='sequence', limit=1)
        if str(first_stage_id.id) not in team_results.get('stage_id', []):
            return 0, 0, 0
        stage_result = team_results['stage_id'][str(first_stage_id.id)]
        return stage_result['won'], stage_result['lost'], stage_result['won'] + stage_result['lost']

    # PLS: Rebuild Frequency Table Tools
    # ----------------------------------
    def _pls_prepare_frequencies(self, lead_values, leads_pls_fields, target_state=None):
        """new state is used when getting frequencies for leads that are changing to lost or won.
        Stays none if we are checking frequencies for leads already won or lost."""
        # Frequencies must include tag_id
        pls_fields = set(leads_pls_fields + ['tag_id'])
        frequencies = dict((field, {}) for field in pls_fields)

        stage_ids = self.env['crm.stage'].search_read([], ['sequence', 'name', 'id'], order='sequence')
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

        if domain:
            # active_test = False as domain should take active into 'active' field it self
            from_clause, where_clause, where_params = self.env['crm.lead'].with_context(active_test=False)._where_calc(domain).get_sql()
            str_fields = ", ".join(["{}"] * len(pls_fields))
            args = [sql.Identifier(field) for field in pls_fields]

            # Get leads values
            self.flush(['probability'])
            query = """SELECT id, probability, %s
                        FROM %s
                        WHERE %s order by team_id asc"""
            query = sql.SQL(query % (str_fields, from_clause, where_clause)).format(*args)
            self._cr.execute(query, where_params)
            lead_results = self._cr.dictfetchall()

            # Get tags values
            query = """SELECT crm_lead.id as lead_id, t.id as tag_id
                            FROM %s
                            LEFT JOIN crm_tag_rel rel ON crm_lead.id = rel.lead_id
                            LEFT JOIN crm_tag t ON rel.tag_id = t.id
                            WHERE %s order by crm_lead.team_id asc"""
            query = sql.SQL(query % (from_clause, where_clause)).format(*args)
            self._cr.execute(query, where_params)
            tag_results = self._cr.dictfetchall()

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
    team_id = fields.Many2one('crm.team', 'Sales Team')


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
        lead_data = self.env['crm.lead'].with_context(active_test=False).read_group([('lost_reason', 'in', self.ids)], ['lost_reason'], ['lost_reason'])
        mapped_data = dict((data['lost_reason'][0], data['lost_reason_count']) for data in lead_data)
        for reason in self:
            reason.leads_count = mapped_data.get(reason.id, 0)

    def action_lost_leads(self):
        return {
            'name': _('Leads'),
            'view_mode': 'tree,form',
            'domain': [('lost_reason', 'in', self.ids)],
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
        """ Hack :  when going from the pipeline, creating a stage with a sales team in
            context should not create a stage for the current Sales Team only
        """
        ctx = dict(self.env.context)
        if ctx.get('default_team_id') and not ctx.get('crm_team_mono'):
            ctx.pop('default_team_id')
        return super(Stage, self.with_context(ctx)).default_get(fields)

    name = fields.Char('Stage Name', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=1, help="Used to order stages. Lower is better.")
    is_won = fields.Boolean('Is Won Stage?')
    requirements = fields.Text('Requirements', help="Enter here the internal requirements for this stage (ex: Offer sent to customer). It will appear as a tooltip over the stage's name.")
    team_id = fields.Many2one('crm.team', string='Sales Team', ondelete='set null',
        help='Specific team that uses this stage. Other teams will not be able to see or use this stage.')
    fold = fields.Boolean('Folded in Pipeline',
        help='This stage is folded in the kanban view when there are no records in that stage to display.')

    # This field for interface only
    team_count = fields.Integer('team_count', compute='_compute_team_count')

    @api.depends('name')
    def _compute_team_count(self):
        team_count = self.env['crm.team'].search_count([])
        for stage in self:
            stage.team_count = team_count

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import datetime

from odoo import api, fields, models, _
from odoo.tools.safe_eval import safe_eval


class Team(models.Model):
    _name = 'crm.team'
    _inherit = ['mail.alias.mixin', 'crm.team']
    _description = 'Sales Team'

    use_leads = fields.Boolean('Leads', help="Check this box to filter and qualify incoming requests as leads before converting them into opportunities and assigning them to a salesperson.")
    use_opportunities = fields.Boolean('Pipeline', default=True, help="Check this box to manage a presales process with opportunities.")
    alias_id = fields.Many2one(
        'mail.alias', string='Alias', ondelete="restrict", required=True,
        help="The email address associated with this channel. New emails received will automatically create new leads assigned to the channel.")
    # statistics about leads / opportunities / both
    lead_unassigned_count = fields.Integer(
        string='# Unassigned Leads', compute='_compute_lead_unassigned_count')
    lead_all_assigned_month_count = fields.Integer(
        string='# Leads/Opps assigned this month', compute='_compute_lead_all_assigned_month_count',
        help="Number of leads and opportunities assigned this last month.")
    opportunities_count = fields.Integer(
        string='# Opportunities', compute='_compute_opportunities_data')
    opportunities_amount = fields.Monetary(
        string='Opportunities Revenues', compute='_compute_opportunities_data')
    opportunities_overdue_count = fields.Integer(
        string='# Overdue Opportunities', compute='_compute_opportunities_overdue_data')
    opportunities_overdue_amount = fields.Monetary(
        string='Overdue Opportunities Revenues', compute='_compute_opportunities_overdue_data',)
    # alias: improve fields coming from _inherits, use inherited to avoid replacing them
    alias_user_id = fields.Many2one(
        'res.users', related='alias_id.alias_user_id', inherited=True,
        domain=lambda self: [('groups_id', 'in', self.env.ref('sales_team.group_sale_salesman_all_leads').id)])

    def _compute_lead_unassigned_count(self):
        leads_data = self.env['crm.lead'].read_group([
            ('team_id', 'in', self.ids),
            ('type', '=', 'lead'),
            ('user_id', '=', False),
        ], ['team_id'], ['team_id'])
        counts = {datum['team_id'][0]: datum['team_id_count'] for datum in leads_data}
        for team in self:
            team.lead_unassigned_count = counts.get(team.id, 0)

    def _compute_lead_all_assigned_month_count(self):
        limit_date = datetime.datetime.now() - datetime.timedelta(days=30)
        leads_data = self.env['crm.lead'].read_group([
            ('team_id', 'in', self.ids),
            ('date_open', '>=', fields.Datetime.to_string(limit_date)),
            ('user_id', '!=', False),
        ], ['team_id'], ['team_id'])
        counts = {datum['team_id'][0]: datum['team_id_count'] for datum in leads_data}
        for team in self:
            team.lead_all_assigned_month_count = counts.get(team.id, 0)

    def _compute_opportunities_data(self):
        opportunity_data = self.env['crm.lead'].read_group([
            ('team_id', 'in', self.ids),
            ('probability', '<', 100),
            ('type', '=', 'opportunity'),
        ], ['expected_revenue:sum', 'team_id'], ['team_id'])
        counts = {datum['team_id'][0]: datum['team_id_count'] for datum in opportunity_data}
        amounts = {datum['team_id'][0]: datum['expected_revenue'] for datum in opportunity_data}
        for team in self:
            team.opportunities_count = counts.get(team.id, 0)
            team.opportunities_amount = amounts.get(team.id, 0)

    def _compute_opportunities_overdue_data(self):
        opportunity_data = self.env['crm.lead'].read_group([
            ('team_id', 'in', self.ids),
            ('probability', '<', 100),
            ('type', '=', 'opportunity'),
            ('date_deadline', '<', fields.Date.to_string(fields.Datetime.now()))
        ], ['expected_revenue', 'team_id'], ['team_id'])
        counts = {datum['team_id'][0]: datum['team_id_count'] for datum in opportunity_data}
        amounts = {datum['team_id'][0]: (datum['expected_revenue']) for datum in opportunity_data}
        for team in self:
            team.opportunities_overdue_count = counts.get(team.id, 0)
            team.opportunities_overdue_amount = amounts.get(team.id, 0)

    @api.onchange('use_leads', 'use_opportunities')
    def _onchange_use_leads_opportunities(self):
        if not self.use_leads and not self.use_opportunities:
            self.alias_name = False

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

    # ------------------------------------------------------------
    # MESSAGING
    # ------------------------------------------------------------

    def _alias_get_creation_values(self):
        values = super(Team, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('crm.lead').id
        if self.id:
            if not self.use_leads and not self.use_opportunities:
                values['alias_name'] = False
            values['alias_defaults'] = defaults = ast.literal_eval(self.alias_defaults or "{}")
            has_group_use_lead = self.env.user.has_group('crm.group_use_lead')
            defaults['type'] = 'lead' if has_group_use_lead and self.use_leads else 'opportunity'
            defaults['team_id'] = self.id
        return values

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    #TODO JEM : refactor this stuff with xml action, proper customization,
    @api.model
    def action_your_pipeline(self):
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_action_pipeline")
        user_team_id = self.env.user.sale_team_id.id
        if user_team_id:
            # To ensure that the team is readable in multi company
            user_team_id = self.search([('id', '=', user_team_id)], limit=1).id
        else:
            user_team_id = self.search([], limit=1).id
            action['help'] = _("""<p class='o_view_nocontent_smiling_face'>Add new opportunities</p><p>
    Looks like you are not a member of a Sales Team. You should add yourself
    as a member of one of the Sales Team.
</p>""")
            if user_team_id:
                action['help'] += _("<p>As you don't belong to any Sales Team, Odoo opens the first one by default.</p>")

        action_context = safe_eval(action['context'], {'uid': self.env.uid})
        if user_team_id:
            action_context['default_team_id'] = user_team_id

        action['context'] = action_context
        return action

    def _compute_dashboard_button_name(self):
        super(Team, self)._compute_dashboard_button_name()
        team_with_pipelines = self.filtered(lambda el: el.use_opportunities)
        team_with_pipelines.update({'dashboard_button_name': _("Pipeline")})

    def action_primary_channel_button(self):
        if self.use_opportunities:
            return self.env["ir.actions.actions"]._for_xml_id("crm.crm_case_form_view_salesteams_opportunity")
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
        return super(Team,self)._graph_title_and_key()

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import AccessError


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_crm_lead_created = fields.Boolean('New Leads/Opportunities')
    kpi_crm_lead_created_value = fields.Integer(compute='_compute_kpi_crm_lead_created_value')
    kpi_crm_opportunities_won = fields.Boolean('Opportunities Won')
    kpi_crm_opportunities_won_value = fields.Integer(compute='_compute_kpi_crm_opportunities_won_value')

    def _compute_kpi_crm_lead_created_value(self):
        if not self.env.user.has_group('sales_team.group_sale_salesman'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            record.kpi_crm_lead_created_value = self.env['crm.lead'].search_count([
                ('create_date', '>=', start),
                ('create_date', '<', end),
                ('company_id', '=', company.id)
            ])

    def _compute_kpi_crm_opportunities_won_value(self):
        if not self.env.user.has_group('sales_team.group_sale_salesman'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            record.kpi_crm_opportunities_won_value = self.env['crm.lead'].search_count([
                ('type', '=', 'opportunity'),
                ('probability', '=', '100'),
                ('date_closed', '>=', start),
                ('date_closed', '<', end),
                ('company_id', '=', company.id)
            ])

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


class IrConfigParameter(models.Model):
    _inherit = 'ir.config_parameter'

    def write(self, vals):
        result = super(IrConfigParameter, self).write(vals)
        if any(record.key == "crm.pls_fields" for record in self):
            self.flush()
            self.env.registry.setup_models(self.env.cr)
        return result

    @api.model_create_multi
    def create(self, vals_list):
        records = super(IrConfigParameter, self).create(vals_list)
        if any(record.key == "crm.pls_fields" for record in records):
            self.flush()
            self.env.registry.setup_models(self.env.cr)
        return records

    def unlink(self):
        pls_emptied = any(record.key == "crm.pls_fields" for record in self)
        result = super(IrConfigParameter, self).unlink()
        if pls_emptied:
            self.flush()
            self.env.registry.setup_models(self.env.cr)
        return pls_emptied

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    crm_alias_prefix = fields.Char(
        'Default Alias Name for Leads',
        compute="_compute_crm_alias_prefix" , readonly=False, store=True)
    generate_lead_from_alias = fields.Boolean(
        'Manual Assignment of Emails', config_parameter='crm.generate_lead_from_alias',
        compute="_compute_generate_lead_from_alias", readonly=False, store=True)
    group_use_lead = fields.Boolean(string="Leads", implied_group='crm.group_use_lead')
    group_use_recurring_revenues = fields.Boolean(string="Recurring Revenues", implied_group='crm.group_use_recurring_revenues')
    module_crm_iap_lead = fields.Boolean("Generate new leads based on their country, industries, size, etc.")
    module_crm_iap_lead_website = fields.Boolean("Create Leads/Opportunities from your website's traffic")
    module_crm_iap_lead_enrich = fields.Boolean("Enrich your leads automatically with company data based on their email address.")
    module_mail_client_extension = fields.Boolean("See and manage users, companies, and leads from our mail client extensions.")
    lead_enrich_auto = fields.Selection([
        ('manual', 'Enrich leads on demand only'),
        ('auto', 'Enrich all leads automatically'),
    ], string='Enrich lead automatically', default='manual', config_parameter='crm.iap.lead.enrich.setting')
    lead_mining_in_pipeline = fields.Boolean("Create a lead mining request directly from the opportunity pipeline.", config_parameter='crm.lead_mining_in_pipeline')
    predictive_lead_scoring_start_date = fields.Date(string='Lead Scoring Starting Date', compute="_compute_pls_start_date", inverse="_inverse_pls_start_date_str")
    predictive_lead_scoring_start_date_str = fields.Char(string='Lead Scoring Starting Date in String', config_parameter='crm.pls_start_date')
    predictive_lead_scoring_fields = fields.Many2many('crm.lead.scoring.frequency.field', string='Lead Scoring Frequency Fields', compute="_compute_pls_fields", inverse="_inverse_pls_fields_str")
    predictive_lead_scoring_fields_str = fields.Char(string='Lead Scoring Frequency Fields in String', config_parameter='crm.pls_fields')

    def _find_default_lead_alias_id(self):
        alias = self.env.ref('crm.mail_alias_lead_info', False)
        if not alias:
            alias = self.env['mail.alias'].search([
                ('alias_model_id.model', '=', 'crm.lead'),
                ('alias_force_thread_id', '=', False),
                ('alias_parent_model_id.model', '=', 'crm.team'),
                ('alias_parent_thread_id', '=', False),
                ('alias_defaults', '=', '{}')
            ], limit=1)
        return alias

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

    @api.depends('group_use_lead')
    def _compute_generate_lead_from_alias(self):
        """ Reset alias / leads configuration if leads are not used """
        for setting in self.filtered(lambda r: not r.group_use_lead):
            setting.generate_lead_from_alias = False

    @api.depends('generate_lead_from_alias')
    def _compute_crm_alias_prefix(self):
        for setting in self:
            setting.crm_alias_prefix = (setting.crm_alias_prefix or 'contact') if setting.generate_lead_from_alias else False

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        alias = self._find_default_lead_alias_id()
        res.update(
            crm_alias_prefix=alias.alias_name if alias else False,
        )
        return res

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        alias = self._find_default_lead_alias_id()
        if alias:
            alias.write({'alias_name': self.crm_alias_prefix})
        else:
            self.env['mail.alias'].create({
                'alias_name': self.crm_alias_prefix,
                'alias_model_id': self.env['ir.model']._get('crm.lead').id,
                'alias_parent_model_id': self.env['ir.model']._get('crm.team').id,
            })
        for team in self.env['crm.team'].search([]):
            team.alias_id.write(team._alias_get_creation_values())

    # ACTIONS
    def action_reset_lead_probabilities(self):
        if self.env.user._is_admin():
            self.env['crm.lead'].sudo()._cron_update_automated_probabilities()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Partner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    team_id = fields.Many2one('crm.team', string='Sales Team')
    opportunity_ids = fields.One2many('crm.lead', 'partner_id', string='Opportunities', domain=[('type', '=', 'opportunity')])
    meeting_ids = fields.Many2many('calendar.event', 'calendar_event_res_partner_rel', 'res_partner_id', 'calendar_event_id', string='Meetings', copy=False)
    opportunity_count = fields.Integer("Opportunity", compute='_compute_opportunity_count')
    meeting_count = fields.Integer("# Meetings", compute='_compute_meeting_count')

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

    def _compute_opportunity_count(self):
        # retrieve all children partners and prefetch 'parent_id' on them
        all_partners = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        all_partners.read(['parent_id'])

        opportunity_data = self.env['crm.lead'].read_group(
            domain=[('partner_id', 'in', all_partners.ids)],
            fields=['partner_id'], groupby=['partner_id']
        )

        self.opportunity_count = 0
        for group in opportunity_data:
            partner = self.browse(group['partner_id'][0])
            while partner:
                if partner in self:
                    partner.opportunity_count += group['partner_id_count']
                partner = partner.parent_id

    def _compute_meeting_count(self):
        result = self._compute_meeting()
        for p in self:
            p.meeting_count = len(result.get(p.id, []))

    def _compute_meeting(self):
        if self.ids:
            all_partners = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])

            event_id = self.env['calendar.event']._search([])  # ir.rules will be applied
            subquery_string, subquery_params = event_id.select()
            subquery = self.env.cr.mogrify(subquery_string, subquery_params).decode()

            self.env.cr.execute("""
                SELECT res_partner_id, calendar_event_id, count(1)
                  FROM calendar_event_res_partner_rel
                 WHERE res_partner_id IN %s AND calendar_event_id IN ({})
              GROUP BY res_partner_id, calendar_event_id
            """.format(subquery), [tuple(all_partners.ids)])

            meeting_data = self.env.cr.fetchall()

            # Create a dict {partner_id: event_ids} and fill with events linked to the partner
            meetings = {}
            for p_id, m_id, _ in meeting_data:
                meetings.setdefault(p_id, set()).add(m_id)

            # Add the events linked to the children of the partner
            for p in self.browse(meetings.keys()):
                partner = p
                while partner.parent_id:
                    partner = partner.parent_id
                    if partner in self:
                        meetings[partner.id] = meetings.get(partner.id, set()) | meetings[p.id]
            return {p_id: list(meetings[p_id]) if p_id in meetings else [] for p_id in self.ids}
        return {}


    def schedule_meeting(self):
        self.ensure_one()
        partner_ids = self.ids
        partner_ids.append(self.env.user.partner_id.id)
        action = self.env["ir.actions.actions"]._for_xml_id("calendar.action_calendar_event")
        action['context'] = {
            'default_partner_ids': partner_ids,
            'default_attendee_ids': [(0, 0, {'partner_id': pid}) for pid in partner_ids],
        }
        action['domain'] = ['|', ('id', 'in', self._compute_meeting()[self.id]), ('partner_ids', 'in', self.ids)]
        return action

    def action_view_opportunity(self):
        '''
        This function returns an action that displays the opportunities from partner.
        '''
        action = self.env['ir.actions.act_window']._for_xml_id('crm.crm_lead_opportunities')
        if self.is_company:
            action['domain'] = [('partner_id.commercial_partner_id.id', '=', self.id)]
        else:
            action['domain'] = [('partner_id.id', '=', self.id)]
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

from odoo import fields, models, api, SUPERUSER_ID

class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    use_leads = fields.Boolean('Use Leads', compute='_compute_use_leads')
    crm_lead_count = fields.Integer('Leads/Opportunities count', groups='sales_team.group_sale_salesman', compute="_compute_crm_lead_count")

    def _compute_use_leads(self):
        for campaign in self:
            campaign.use_leads = self.env.user.has_group('crm.group_use_lead')

    def _compute_crm_lead_count(self):
        lead_data = self.env['crm.lead'].with_context(active_test=False).read_group([
            ('campaign_id', 'in', self.ids)],
            ['campaign_id'], ['campaign_id'])
        mapped_data = {datum['campaign_id'][0]: datum['campaign_id_count'] for datum in lead_data}
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
from . import ir_config_parameter
from . import res_config_settings
from . import res_partner
from . import digest
from . import crm_lead_scoring_frequency
from . import utm
from . import crm_recurring_plan

```

## File: report\crm_activity_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools, api


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
        disccusion_subtype = self.env.ref('mail.mt_comment')
        return """
            WHERE
                m.model = 'crm.lead' AND (m.mail_activity_type_id IS NOT NULL OR m.subtype_id = %s)
        """ % (disccusion_subtype.id,)

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
                <graph string="Activities Analysis" stacked="True" sample="1" disable_linking="1">
                    <field name="mail_activity_type_id" type="col"/>
                    <field name="date" interval="month" type="row"/>
                </graph>
            </field>
        </record>

        <record id="crm_activity_report_view_pivot" model="ir.ui.view">
            <field name="name">crm.activity.report.pivot</field>
            <field name="model">crm.activity.report</field>
            <field name="arch" type="xml">
                <pivot string="Activities Analysis" disable_linking="True" sample="1">
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
                    <field name="author_id"/>
                    <field name="mail_activity_type_id"/>
                    <field name="body"/>
                    <field name="company_id" groups="base.group_multi_company"/>
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
           <field name="name">Pipeline Activities</field>
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

       <menuitem
            id="crm_activity_report_menu"
            name="Activities"
            groups="sales_team.group_sale_manager"
            parent="crm_menu_report"
            action="crm_activity_report_action"
            sequence="3"/>

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
                </pivot>
            </field>
        </record>

        <!-- Opportunities by user and team Graph View -->
        <record id="crm_opportunity_report_view_graph" model="ir.ui.view">
            <field name="name">crm.opportunity.report.graph</field>
            <field name="model">crm.lead</field>
            <field name="arch" type="xml">
                <graph string="Pipeline Analysis" stacked="True" sample="1">
                    <field name="stage_id" type="row"/>
                    <field name="date_deadline" type="row" interval="month"/>
                    <field name="prorated_revenue" type="measure"/>
                    <field name="color" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="crm_opportunity_report_view_graph_lead" model="ir.ui.view">
            <field name="name">crm.opportunity.report.graph.lead</field>
            <field name="model">crm.lead</field>
            <field name="priority">20</field>
            <field name="arch" type="xml">
                <graph string="Leads Analysis" stacked="True" sample="1">
                    <field name="create_date" interval="month" type="col"/>
                    <field name="team_id" type="col"/>
                    <field name="color" invisible="1"/>
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
                    <filter string="My Opportunities" name="my"
                            domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="lead" string="Lead" domain="[('type','=', 'lead')]" help="Show only lead"/>
                    <filter name="opportunity" string="Opportunity" domain="[('type','=','opportunity')]" help="Show only opportunity"/>
                    <separator/>
                    <filter string="Won" name="won"
                            domain="[('probability', '=', 100)]"/>
                    <filter string="Lost" name="lost"
                            domain="[('probability', '=', 0), ('active', '=', False)]"/>
                    <field name="team_id" context="{'invisible_team': False}"/>
                    <field name="user_id" string="Salesperson"/>
                    <separator/>
                    <filter string="Creation Date" name="filter_create_date" date="create_date" default_period="this_year"/>
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
                    <group expand="1" string="Group By">
                        <filter string="Salesperson" name="salesperson" context="{'group_by':'user_id'}" />
                        <filter string="Sales Team" name="saleschannel" context="{'group_by':'team_id'}"/>
                        <filter string="City" name="city" context="{'group_by':'city'}" />
                        <filter string="Country" name="country" context="{'group_by':'country_id'}" />
                        <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Stage" name="stage" context="{'group_by':'stage_id'}"/>
                        <separator orientation="vertical" />
                        <filter string="Creation Date" context="{'group_by':'create_date:month'}" name="month"/>
                        <filter string="Conversion Date" context="{'group_by':'date_conversion:month'}" name="conversion_date" help="Conversion Date from Lead to Opportunity"/>
                        <filter string="Expected Closing" context="{'group_by':'date_deadline:month'}" name="date_deadline"/>
                        <filter string="Closed Date" context="{'group_by':'date_closed'}" name="date_closed_groupby"/>
                        <filter string="Lost Reason" name="lostreason" context="{'group_by':'lost_reason'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="crm_opportunity_report_action" model="ir.actions.act_window">
            <field name="name">Pipeline Analysis</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">pivot,graph,tree,form</field>
            <field name="search_view_id" ref="crm.crm_opportunity_report_view_search"/>
            <field name="context">{'search_default_opportunity': True, 'search_default_current': True}</field>
            <field name="view_ids"
                   eval="[(5, 0, 0),
                          (0, 0, {'view_mode': 'graph', 'view_id': ref('crm_opportunity_report_view_graph')}),
                          (0, 0, {'view_mode': 'pivot', 'view_id': ref('crm_opportunity_report_view_pivot')}),
                          (0, 0, {'view_mode': 'tree', 'view_id': ref('crm_case_tree_view_oppor')})]"/>
             <field name="help">Pipeline Analysis gives you an instant access to
                your opportunities with information such as the expected revenue, planned cost,
                missed deadlines or the number of interactions per opportunity. This report is
                mainly used by the sales manager in order to do the periodic review with the
                teams of the sales pipeline.</field>
        </record>

        <record id="crm_opportunity_report_menu" model="ir.ui.menu">
            <field name="action" ref="crm.crm_opportunity_report_action"/>
        </record>

        <record id="crm_opportunity_report_action_lead" model="ir.actions.act_window">
            <field name="name">Leads Analysis</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">pivot,graph,tree,form</field>
            <field name="search_view_id" ref="crm.crm_opportunity_report_view_search"/>
            <field name="context">{'search_default_current': True, 'search_default_archived': True}</field>
            <field name="view_ids"
                   eval="[(5, 0, 0),
                          (0, 0, {'view_mode': 'graph', 'view_id': ref('crm_opportunity_report_view_graph_lead')}),
                          (0, 0, {'view_mode': 'pivot', 'view_id': ref('crm_opportunity_report_view_pivot_lead')})]"/>
            <field name="help">This report analyses the source of your leads.</field>
        </record>

        <record id="crm_opportunity_report_menu_lead" model="ir.ui.menu">
            <field name="action" ref="crm.crm_opportunity_report_action_lead"/>
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
<data>

    <record id="group_use_lead" model="res.groups">
        <field name="name">Show Lead Menu</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_use_recurring_revenues" model="res.groups">
        <field name="name">Show Recurring Revenues Menu</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record model="res.users" id="base.user_root">
        <field eval="[(4,ref('base.group_partner_manager'))]" name="groups_id"/>
    </record>

    <record model="res.users" id="base.user_admin">
        <field eval="[(4,ref('base.group_partner_manager'))]" name="groups_id"/>
    </record>

    <record id="contacts.res_partner_menu_config" model="ir.ui.menu">
        <field name="name">Configuration</field>
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
    </record>

</data>

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

</data>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_crm_lead_manager,crm.lead.manager,model_crm_lead,sales_team.group_sale_manager,1,1,1,1
access_crm_lead,crm.lead,model_crm_lead,sales_team.group_sale_salesman,1,1,1,0
access_crm_stage,crm.stage,model_crm_stage,,1,0,0,0
access_crm_stage_manager,crm.stage,model_crm_stage,sales_team.group_sale_manager,1,1,1,1
access_res_partner_manager,res.partner.crm.manager,base.model_res_partner,sales_team.group_sale_manager,1,0,0,0
access_res_partner_category_manager,res.partner.category.crm.manager,base.model_res_partner_category,sales_team.group_sale_manager,1,0,0,0
access_res_partner,res.partner.crm.user,base.model_res_partner,sales_team.group_sale_salesman,1,1,1,0
access_res_partner_category,res.partner.category.crm.user,base.model_res_partner_category,sales_team.group_sale_salesman,1,1,1,0
access_crm_lead_partner_manager,crm.lead.partner.manager,model_crm_lead,base.group_partner_manager,1,0,0,0
access_crm_lost_reason_manager,crm.lost.reason.manager,model_crm_lost_reason,sales_team.group_sale_manager,1,1,1,1
access_crm_lost_reason_salesman,crm.lost.reason.salesman,model_crm_lost_reason,sales_team.group_sale_salesman,1,0,0,0
access_crm_lost_reason_user,crm.lost.reason.user,model_crm_lost_reason,base.group_user,1,0,0,0
access_crm_activity_report_user,crm.activity.report.user,model_crm_activity_report,base.group_user,1,0,0,0
access_calendar_event_manager,calendar.event.manager,calendar.model_calendar_event,sales_team.group_sale_manager,1,1,1,1
access_calendar_event,calendar.event,calendar.model_calendar_event,sales_team.group_sale_salesman,1,1,1,0
access_calendar_event_type_sale_manager,calendar.event.type.manager,calendar.model_calendar_event_type,sales_team.group_sale_manager,1,1,1,0
access_calendar_event_type_sale_user,calendar.event.type.user,calendar.model_calendar_event_type,base.group_user,1,0,0,0
access_calendar_event_type_sale_salesman,calendar.event.type.salesman,calendar.model_calendar_event_type,sales_team.group_sale_salesman,1,0,0,0
access_mail_activity_type_sale_manager,mail.activity.type.sale.manager,mail.model_mail_activity_type,sales_team.group_sale_manager,1,1,1,1
access_crm_lead_scoring_frequency,access_crm_lead_scoring_frequency,model_crm_lead_scoring_frequency,sales_team.group_sale_salesman,1,0,0,0
access_crm_lead_scoring_frequency_system,access_crm_lead_scoring_frequency_system,model_crm_lead_scoring_frequency,base.group_system,1,0,0,0
access_crm_lead_scoring_frequency_field,access_crm_lead_scoring_frequency_field,model_crm_lead_scoring_frequency_field,sales_team.group_sale_salesman,1,0,0,0
access_crm_lead_lost,access.crm.lead.lost,model_crm_lead_lost,sales_team.group_sale_salesman,1,1,1,0
access_crm_lead2opportunity_partner,access.crm.lead2opportunity.partner,model_crm_lead2opportunity_partner,sales_team.group_sale_salesman,1,1,1,0
access_crm_lead2opportunity_partner_mass,access.crm.lead2opportunity.partner.mass,model_crm_lead2opportunity_partner_mass,sales_team.group_sale_salesman,1,1,1,0
access_crm_merge_opportunity,access.crm.merge.opportunity,model_crm_merge_opportunity,sales_team.group_sale_salesman,1,1,1,0
crm_recurring_plan_access_manager,crm.recurring.plan.access.manager,model_crm_recurring_plan,sales_team.group_sale_manager,1,1,1,1
crm_recurring_plan_access_salesman,crm.recurring.plan.access.salesman,model_crm_recurring_plan,sales_team.group_sale_salesman,1,0,0,0
access_crm_lead_scoring_frequency_field_system,access_crm_lead_scoring_frequency_field_system,model_crm_lead_scoring_frequency_field,base.group_system,1,0,0,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/>
        <linearGradient id="icon-c" x1="98.162%" x2="0%" y1="1.838%" y2="100%">
            <stop offset="0%" stop-color="#797DA5"/>
            <stop offset="50.799%" stop-color="#6D7194"/>
            <stop offset="100%" stop-color="#626584"/>
        </linearGradient>
        <path id="icon-d" d="M56.583 23.333h-3.5c-.518 0-.983.226-1.304.584h-1.715l-2.27-2.647-.012-.013a7.581 7.581 0 0 0-5.707-2.59h-3.394c-1.294 0-2.545.36-3.623 1.021a8.19 8.19 0 0 0-3.95-1.021h-2.342c-2.107 0-4.2.818-5.775 2.391l-2.857 2.859H18.22a1.745 1.745 0 0 0-1.304-.584h-3.5a1.75 1.75 0 0 0-1.75 1.75v17.5c0 .967.783 1.75 1.75 1.75h3.5a1.75 1.75 0 0 0 1.65-1.166h1.37l5.495 4.927c1.862 1.928 4.37 3.24 7.042 3.24 1.195 0 2.354-.281 3.362-.798 1.818.037 3.726-.756 5.036-2.29a7.109 7.109 0 0 0 3.698-2.523c1.53-.32 2.97-1.202 3.896-2.556h2.967a1.75 1.75 0 0 0 1.65 1.166h3.5a1.75 1.75 0 0 0 1.75-1.75v-17.5a1.75 1.75 0 0 0-1.75-1.75zM15.167 42a1.167 1.167 0 1 1 0-2.333 1.167 1.167 0 0 1 0 2.333zm30.08-.42c-1.12 1.042-2.69.826-2.914.583.103.976-1.331 2.993-3.579 2.835-.404 1.351-2.057 2.467-3.754 1.878-.648.648-1.638.957-2.526.957-1.82 0-3.483-1.06-4.604-2.254l-5.928-5.316a2.332 2.332 0 0 0-1.557-.596h-1.718v-12.25h1.95c.619 0 1.212-.246 1.65-.684l3.2-3.2a4.667 4.667 0 0 1 3.3-1.366h2.34c.424 0 .84.057 1.24.167l-3.155 3.682a5.376 5.376 0 0 0-.048 6.96c2.362 2.833 6.663 2.86 9.077.143l1.894-2.193 5.282 6.99c.98 1.065.799 2.781-.15 3.664zm6.086-1.913H49.55a5.985 5.985 0 0 0-1.441-3.967l-5.693-7.534a1.752 1.752 0 0 0-2.907-1.894l-3.911 4.53a2.49 2.49 0 0 1-3.765-.068 1.885 1.885 0 0 1 .016-2.44l4.224-4.928a3.434 3.434 0 0 1 2.608-1.2h3.394c1.175 0 2.293.507 3.068 1.389l3.312 3.862h2.878v12.25zm3.5 2.333a1.167 1.167 0 1 1 0-2.333 1.167 1.167 0 0 1 0 2.333z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <path fill="url(#icon-c)" d="M0 0H70V70H0z"/>
            <path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/>
            <path fill="#000" d="M33.423 69H4.006C2.003 69 0 68.854 0 64.911V38.065l12.348-13.998 8.686.293L29.375 21h14.023l6.01 4.089L58 24.36v19.558L33.423 69z" opacity=".165"/>
            <path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/>
            <path fill="#000" fill-rule="nonzero" d="M56.583 25.333h-3.5c-.518 0-.983.226-1.304.584h-1.715l-2.27-2.647-.012-.013a7.581 7.581 0 0 0-5.707-2.59h-3.394c-1.294 0-2.545.36-3.623 1.021a8.19 8.19 0 0 0-3.95-1.021h-2.342c-2.107 0-4.2.818-5.775 2.391l-2.857 2.859H18.22a1.745 1.745 0 0 0-1.304-.584h-3.5a1.75 1.75 0 0 0-1.75 1.75v17.5c0 .967.783 1.75 1.75 1.75h3.5a1.75 1.75 0 0 0 1.65-1.166h1.37l5.495 4.927c1.862 1.928 4.37 3.24 7.042 3.24 1.195 0 2.354-.281 3.362-.798 1.818.037 3.726-.756 5.036-2.29a7.109 7.109 0 0 0 3.698-2.523c1.53-.32 2.97-1.202 3.896-2.556h2.967a1.75 1.75 0 0 0 1.65 1.166h3.5a1.75 1.75 0 0 0 1.75-1.75v-17.5a1.75 1.75 0 0 0-1.75-1.75zM15.167 44a1.167 1.167 0 1 1 0-2.333 1.167 1.167 0 0 1 0 2.333zm30.08-.42c-1.12 1.042-2.69.826-2.914.583.103.976-1.331 2.993-3.579 2.835-.404 1.351-2.057 2.467-3.754 1.878-.648.648-1.638.957-2.526.957-1.82 0-3.483-1.06-4.604-2.254l-5.928-5.316a2.332 2.332 0 0 0-1.557-.596h-1.718v-12.25h1.95c.619 0 1.212-.246 1.65-.684l3.2-3.2a4.667 4.667 0 0 1 3.3-1.366h2.34c.424 0 .84.057 1.24.167l-3.155 3.682a5.376 5.376 0 0 0-.048 6.96c2.362 2.833 6.663 2.86 9.077.143l1.894-2.193 5.282 6.99c.98 1.065.799 2.781-.15 3.664zm6.086-1.913H49.55a5.985 5.985 0 0 0-1.441-3.967l-5.693-7.534a1.752 1.752 0 0 0-2.907-1.894l-3.911 4.53a2.49 2.49 0 0 1-3.765-.068 1.885 1.885 0 0 1 .016-2.44l4.224-4.928a3.434 3.434 0 0 1 2.608-1.2h3.394c1.175 0 2.293.507 3.068 1.389l3.312 3.862h2.878v12.25zm3.5 2.333a1.167 1.167 0 1 1 0-2.333 1.167 1.167 0 0 1 0 2.333z" opacity=".372"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-d"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\crm_form.js

```javascript
odoo.define("crm.crm_form", function (require) {
    "use strict";

    /**
     * This From Controller makes sure we display a rainbowman message
     * when the stage is won, even when we click on the statusbar.
     * When the stage of a lead is changed and data are saved, we check
     * if the lead is won and if a message should be displayed to the user
     * with a rainbowman like when the user click on the button "Mark Won".
     */

    var FormController = require('web.FormController');
    var FormView = require('web.FormView');
    var viewRegistry = require('web.view_registry');

    var CrmFormController = FormController.extend({
        /**
         * Main method used when saving the record hitting the "Save" button.
         * We check if the stage_id field was altered and if we need to display a rainbowman
         * message.
         *
         * This method will also simulate a real "force_save" on the email and phone
         * when needed. The "force_save" attribute only works on readonly field. For our
         * use case, we need to write the email and the phone even if the user didn't
         * change them, to synchronize those values with the partner (so the email / phone
         * inverse method can be called).
         *
         * We base this synchronization on the value of "ribbon_message", which is a
         * computed field that hold a value whenever we need to synch.
         *
         * @override
         */
        saveRecord: function (recordID, options) {
            recordID = recordID || this.handle;
            const localData = this.model.localData[recordID];
            const changes = localData._changes || {};

            const needsSynchronization = changes.ribbon_message === undefined
                ? localData.data.ribbon_message // original value
                : changes.ribbon_message; // new value

            if (needsSynchronization && changes.email_from === undefined && localData.data.email_from) {
                changes.email_from = localData.data.email_from;
            }
            if (needsSynchronization && changes.phone === undefined && localData.data.phone) {
                changes.phone = localData.data.phone;
            }
            if (!localData._changes && Object.keys(changes).length) {
                localData._changes = changes;
            }

            return this._super(...arguments).then((modifiedFields) => {
                if (modifiedFields.indexOf('stage_id') !== -1) {
                    this._checkRainbowmanMessage(this.renderer.state.res_id)
                }
            });
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Apply change may be called with 'event.data.force_save' set to True.
         * This typically happens when directly clicking in the statusbar widget on a new stage.
         * If it's the case, we check for a modified stage_id field and if we need to display a
         * rainbowman message.
         *
         * @param {string} dataPointID
         * @param {Object} changes
         * @param {OdooEvent} event
         * @override
         * @private
         */
        _applyChanges: function (dataPointID, changes, event) {
            return this._super(...arguments).then(() => {
                if (event.data.force_save && 'stage_id' in changes) {
                    this._checkRainbowmanMessage(parseInt(event.target.res_id));
                }
            });
        },

        /**
         * When updating a crm.lead, through direct use of the status bar or when saving the
         * record, we check for a rainbowman message to display.
         *
         * (see Widget docstring for more information).
         *
         * @param {integer} recordId
         */
        _checkRainbowmanMessage: async function(recordId) {
            const message = await this._rpc({
                model: 'crm.lead',
                method : 'get_rainbowman_message',
                args: [[recordId]],
            });
            if (message) {
                this.trigger_up('show_effect', {
                    message: message,
                    type: 'rainbow_man',
                });
            }
        }
    });

    var CrmFormView = FormView.extend({
        config: _.extend({}, FormView.prototype.config, {
            Controller: CrmFormController,
        }),
    });

    viewRegistry.add('crm_form', CrmFormView);

    return {
        CrmFormController: CrmFormController,
        CrmFormView: CrmFormView,
    };
});

```

## File: static\src\js\crm_kanban.js

```javascript
odoo.define('crm.crm_kanban', function (require) {
    "use strict";

    /**
     * This Kanban Model make sure we display a rainbowman
     * message when a lead is won after we moved it in the
     * correct column and when it's grouped by stage_id (default).
     */

    var KanbanModel = require('web.KanbanModel');
    var KanbanView = require('web.KanbanView');
    var viewRegistry = require('web.view_registry');

    var CrmKanbanModel = KanbanModel.extend({
        /**
         * Check if the kanban view is grouped by "stage_id" before checking if the lead is won
         * and displaying a possible rainbowman message.
         * @override
         */
        moveRecord: async function (recordID, groupID, parentID) {
            var result = await this._super(...arguments);
            if (this.localData[parentID].groupedBy[0] === this.defaultGroupedBy[0]) {
                const message = await this._rpc({
                    model: 'crm.lead',
                    method : 'get_rainbowman_message',
                    args: [[parseInt(this.localData[recordID].res_id)]],
                });
                if (message) {
                    this.trigger_up('show_effect', {
                        message: message,
                        type: 'rainbow_man',
                    });
                }
            }
            return result;
        },
    });

    var CrmKanbanView = KanbanView.extend({
        config: _.extend({}, KanbanView.prototype.config, {
            Model: CrmKanbanModel,
        }),
    });

    viewRegistry.add('crm_kanban', CrmKanbanView);

    return {
        CrmKanbanModel: CrmKanbanModel,
        CrmKanbanView: CrmKanbanView,
    };

});

```

## File: static\src\js\systray_activity_menu.js

```javascript
odoo.define('crm.systray.ActivityMenu', function (require) {
"use strict";

var ActivityMenu = require('mail.systray.ActivityMenu');

ActivityMenu.include({

    //--------------------------------------------------
    // Private
    //--------------------------------------------------

    /**
     * @override
     */
    _getViewsList(model) {
        if (model === "crm.lead") {
                return [[false, 'list'], [false, 'kanban'],
                        [false, 'form'], [false, 'calendar'],
                        [false, 'pivot'], [false, 'graph'],
                        [false, 'activity']
                    ];
        }
        return this._super(...arguments);
    },

    //-----------------------------------------
    // Handlers
    //-----------------------------------------

    /**
     * @private
     * @override
     */
    _onActivityFilterClick: function (event) {
        // fetch the data from the button otherwise fetch the ones from the parent (.o_mail_preview).
        var data = _.extend({}, $(event.currentTarget).data(), $(event.target).data());
        var context = {};
        if (data.res_model === "crm.lead") {
            if (data.filter === 'my') {
                context['search_default_activities_overdue'] = 1;
                context['search_default_activities_today'] = 1;
            } else {
                context['search_default_activities_' + data.filter] = 1;
            }
            // Necessary because activity_ids of mail.activity.mixin has auto_join
            // So, duplicates are faking the count and "Load more" doesn't show up
            context['force_search_count'] = 1;
            this.do_action('crm.crm_lead_action_my_activities', {
                additional_context: context,
                clear_breadcrumbs: true,
            });
        } else {
            this._super.apply(this, arguments);
        }
    },
});
});

```

## File: static\src\js\tours\crm.js

```javascript
odoo.define('crm.tour', function(require) {
"use strict";

var core = require('web.core');
var tour = require('web_tour.tour');

var _t = core._t;

tour.register('crm_tour', {
    url: "/web",
    rainbowManMessage: _t("Congrats, best of luck catching such big fish! :)"),
    sequence: 10,
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="crm.crm_menu_root"]',
    content: _t('Ready to boost your sales? Let\'s have a look at your <b>Pipeline</b>.'),
    position: 'bottom',
    edition: 'community',
}, {
    trigger: '.o_app[data-menu-xmlid="crm.crm_menu_root"]',
    content: _t('Ready to boost your sales? Let\'s have a look at your <b>Pipeline</b>.'),
    position: 'bottom',
    edition: 'enterprise',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_opportunity_kanban',
    content: _t("<b>Create your first opportunity.</b>"),
    position: 'bottom',
}, {
    trigger: ".o_kanban_quick_create .o_field_widget[name='partner_id']",
    content: _t('<b>Write a few letters</b> to look for a company, or create a new one.'),
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
    content: _t("Now, <b>add your Opportunity</b> to your Pipeline."),
    position: "bottom",
}, {
    trigger: ".o_opportunity_kanban .o_kanban_group:first-child .o_kanban_record:last-child .oe_kanban_content",
    extra_trigger: ".o_opportunity_kanban",
    content: _t("<b>Drag &amp; drop opportunities</b> between columns as you progress in your sales cycle."),
    position: "right",
    run: "drag_and_drop .o_opportunity_kanban .o_kanban_group:eq(2) ",
}, {
    trigger: ".o_kanban_record:not(.o_updating) .o_activity_color_default",
    extra_trigger: ".o_opportunity_kanban",
    content: _t("Looks like nothing is planned. :(<br><br><i>Tip : Schedule activities to keep track of everything you have to do!</i>"),
    position: "bottom",
}, {
    trigger: ".o_schedule_activity",
    extra_trigger: ".o_opportunity_kanban",
    content: _t("Let's <b>Schedule an Activity.</b>"),
    position: "bottom",
    width: 200,
}, {
    trigger: '.modal-footer button[name="action_close_dialog"]',
    content: _t("All set. Let’s <b>Schedule</b> it."),
    position: "top",  // dot NOT move to bottom, it would cause a resize flicker, see task-2476595
    run: function (actions) {
        actions.auto('.modal-footer button[special=cancel]');
    },
}, {
    id: "drag_opportunity_to_won_step",
    trigger: ".o_opportunity_kanban .o_kanban_record:last-child",
    content: _t("Drag your opportunity to <b>Won</b> when you get the deal. Congrats !"),
    position: "bottom",
    run: "drag_and_drop .o_opportunity_kanban .o_kanban_group:eq(3) ",
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
    content: _t("This bar also allows you to switch stage."),
    position: "bottom"
}, {
    trigger: ".breadcrumb-item:not(.active):first",
    content: _t("Click on the breadcrumb to go back to the Pipeline."),
    position: "bottom",
    run: function (actions) {
        actions.auto(".breadcrumb-item:not(.active):last");
    }
}]);

});

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
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="opportunity_id"/>
            </xpath>
        </field>
    </record>

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
                        <button name="action_set_won_rainbowman" string="Mark Won"
                            type="object" class="oe_highlight"
                            attrs="{'invisible': ['|','|', ('active','=',False), ('probability', '=', 100), ('type', '=', 'lead')]}"/>
                        <button name="%(crm.crm_lead_lost_action)d" string="Mark Lost"
                            type="action" class="oe_highlight" context="{'default_lead_id': active_id}" attrs="{'invisible': ['|', ('type', '=', 'lead'), '&amp;',('active', '=', False),('probability', '&lt;', 100)]}"/>
                        <button name="%(crm.action_crm_lead2opportunity_partner)d" string="Convert to Opportunity" type="action" help="Convert to Opportunity"
                            class="oe_highlight" attrs="{'invisible': ['|', ('type', '=', 'opportunity'), ('active', '=', False)]}"/>
                        <button name="toggle_active" string="Restore" type="object"
                            attrs="{'invisible': ['|', ('probability', '&gt;', 0), ('active', '=', True)]}"/>
                        <button name="action_set_lost" string="Mark as Lost" type="object"
                            attrs="{'invisible': ['|', ('type', '=', 'opportunity'), '&amp;', ('probability', '=', 0), ('active', '=', False)]}"/>
                        <field name="stage_id" widget="statusbar"
                            options="{'clickable': '1', 'fold_field': 'fold'}"
                            domain="['|', ('team_id', '=', team_id), ('team_id', '=', False)]"
                            attrs="{'invisible': ['|', ('active', '=', False), ('type', '=', 'lead')]}"/>
                    </header>
                    <div class="text-center alert alert-primary oe_edit_only" role="alert" attrs="{'invisible': ['|', ('ribbon_message', '=', False), ('ribbon_message', '=', '')]}">
                        <field name="ribbon_message"/>
                    </div>
                    <sheet>
                        <field name="active" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <button name="action_schedule_meeting" type="object"
                                class="oe_stat_button" icon="fa-calendar"
                                context="{'partner_id': partner_id}"
                                attrs="{'invisible': [('type', '=', 'lead')]}">
                                <div class="o_stat_info">
                                    <field name="meeting_count" class="o_stat_value"/>
                                    <span class="o_stat_text" attrs="{'invisible': [('meeting_count', '&lt;', 2)]}"> Meetings</span>
                                    <span class="o_stat_text" attrs="{'invisible': [('meeting_count', '&gt;', 1)]}"> Meeting</span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Lost" bg_color="bg-danger" attrs="{'invisible': ['|', ('probability', '&gt;', 0), ('active', '=', True)]}"/>
                        <widget name="web_ribbon" title="Won" attrs="{'invisible': [('probability', '&lt;', 100)]}" />
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only" string="Lead" attrs="{'invisible': [('type', '=', 'opportunity')]}"/>
                            <label for="name" class="oe_edit_only" attrs="{'invisible': [('type', '=', 'lead')]}"/>
                            <h1><field name="name" placeholder="e.g. Product Pricing"/></h1>
                            <h2 class="o_row no-gutters align-items-end">
                                <div class="col" attrs="{'invisible': [('type', '=', 'lead')]}">
                                    <label for="expected_revenue" class="oe_edit_only" />
                                    <div class="o_row">
                                        <field name="company_currency" invisible="1"/>
                                        <field name="expected_revenue" class="oe_inline" widget='monetary' options="{'currency_field': 'company_currency'}"/>
                                        <span class="oe_grey p-2" groups="crm.group_use_recurring_revenues"> + </span>
                                        <span class="oe_grey p-2" groups="!crm.group_use_recurring_revenues"> at </span>
                                    </div>
                                </div>
                                <div class="col" attrs="{'invisible': [('type', '=', 'lead')]}" groups="crm.group_use_recurring_revenues">
                                    <div class="o_row">
                                        <field name="recurring_revenue" class="pr-2 oe_inline" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                                    </div>
                                </div>
                                <div class="col" attrs="{'invisible': [('type', '=', 'lead')]}" groups="crm.group_use_recurring_revenues">
                                    <div class="o_row">
                                        <field name="recurring_plan" class="oe_inline" placeholder="E.g. Monthly"
                                               attrs="{'required': [('recurring_revenue', '!=', 0)]}" options="{'no_create': True, 'no_open': True}"/>
                                        <span class="oe_grey p-2"> at </span>
                                    </div>
                                </div>
                                <div class="col">
                                    <div class="oe_edit_only d-flex align-items-center">
                                        <label for="probability"/>
                                        <button class="btn btn-link" name="action_set_automated_probability" type="object"
                                                attrs="{'invisible': [('is_automated_probability', '=', True)]}">
                                            <i class="fa fa-gear" role="img" title="Switch to automatic probability" aria-label="Switch to automatic probability"></i>
                                        </button>
                                        <small class="oe_grey h6 mb0" attrs="{'invisible': [('is_automated_probability', '=', True)]}">
                                            <field class="mb0" name="automated_probability" force_save="1"/> %%
                                        </small>
                                    </div>
                                    <div id="probability" class="o_row d-flex">
                                        <field name="is_automated_probability" invisible="1"/>
                                        <field name="probability" widget="float" class="oe_inline"/>
                                        <span class="oe_grey"> %%</span>
                                    </div>
                                </div>
                            </h2>
                        </div>
                        <group>
                            <group name="lead_partner" attrs="{'invisible': [('type', '=', 'opportunity')]}">
                                <!-- Preload all the partner's information -->
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
                                        'show_vat': True
                                    }" groups="base.group_no_one"/>
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
                                <field name="lang_id"/>
                            </group>

                            <group name="opportunity_partner" attrs="{'invisible': [('type', '=', 'lead')]}">
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
                                        'show_vat': True,
                                    }"
                                />
                                <field name="is_blacklisted" invisible="1"/>
                                <field name="partner_is_blacklisted" invisible="1"/>
                                <field name="phone_blacklisted" invisible="1"/>
                                <field name="mobile_blacklisted" invisible="1"/>
                                <field name="email_state" invisible="1"/>
                                <field name="phone_state" invisible="1"/>
                                <label for="email_from" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="mail_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This email is blacklisted for mass mailings. Click to unblacklist."
                                        type="object" context="{'default_email': email_from}" groups="base.group_user"
                                        attrs="{'invisible': [('is_blacklisted', '=', False)]}"/>
                                    <field name="email_from" string="Email" widget="email"/>
                                </div>
                                <label for="phone" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                                        type="object" context="{'default_phone': phone}" groups="base.group_user"
                                        attrs="{'invisible': [('phone_blacklisted', '=', False)]}"/>
                                    <field name="phone" widget="phone"/>
                                </div>
                            </group>
                            <group name="lead_info" attrs="{'invisible': [('type', '=', 'opportunity')]}">
                                <label for="contact_name"/>
                                <div class="o_row">
                                    <field name="contact_name"/>
                                    <field name="title" placeholder="Title" domain="[]" options='{"no_open": True}'/>
                                </div>
                                <field name="is_blacklisted" invisible="1"/>
                                <field name="phone_blacklisted" invisible="1"/>
                                <field name="email_state" invisible="1"/>
                                <field name="phone_state" invisible="1"/>
                                <label for="email_from_group_lead_info" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="mail_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This email is blacklisted for mass mailings. Click to unblacklist."
                                        type="object" context="{'default_email': email_from}" groups="base.group_user"
                                        attrs="{'invisible': [('is_blacklisted', '=', False)]}"/>
                                    <field name="email_from" id="email_from_group_lead_info" string="Email" widget="email"/>
                                </div>
                                <field name="email_cc" groups="base.group_no_one"/>
                                <field name="function"/>
                                <label for="phone_group_lead_info" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                                        type="object" context="{'default_phone': phone}" groups="base.group_user"
                                        attrs="{'invisible': [('phone_blacklisted', '=', False)]}"/>
                                    <field name="phone" id="phone_group_lead_info" widget="phone"/>
                                </div>
                                <label for="mobile" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                                        type="object" context="{'default_phone': mobile}" groups="base.group_user"
                                        attrs="{'invisible': [('mobile_blacklisted', '=', False)]}"/>
                                    <field name="mobile" widget="phone" string="Mobile"/>
                                </div>
                            </group>
                            <group attrs="{'invisible': [('type', '=', 'lead')]}">
                                <field name="date_deadline"/>
                                <field name="priority" widget="priority"/>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            </group>
                            <group>
                                <field name="user_id" domain="[('share', '=', False)]"
                                    context="{'default_sales_team_id': team_id}" widget="many2one_avatar_user"/>
                                <field name="team_id" widget="selection"
                                    domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]"/>
                                <field name="type" invisible="1"/>
                            </group>
                            <group name="lead_priority" attrs="{'invisible': [('type', '=', 'opportunity')]}">
                                <field name="priority" widget="priority"/>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            </group>
                            <group name="opportunity_info" attrs="{'invisible': [('type', '=', 'lead')]}">
                                <field name="lost_reason" attrs="{'invisible': [('active', '=', True)]}"/>
                                <field name="date_conversion" invisible="1"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                            </group>
                        </group>

                        <notebook>
                            <page string="Internal Notes" name="internal_notes">
                                <field name="description" placeholder="Add a description..."/>
                            </page>
                            <page name="extra" string="Extra Info" attrs="{'invisible': [('type', '=', 'opportunity')]}">
                                <group>
                                    <group string="Email" groups="base.group_no_one">
                                        <field name="message_bounce" readonly="1"/>
                                    </group>
                                    <group string="Tracking" name="categorization">
                                        <field name="company_id"
                                            groups="base.group_multi_company"
                                            options="{'no_create': True}"/>
                                        <field name="campaign_id" />
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
                            <page name="lead" string="Extra Information" attrs="{'invisible': [('type', '=', 'lead')]}">
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
                                        <field name="lang_id" options="{'no_create': True}"/>
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
                                                attrs="{'invisible': [('mobile_blacklisted', '=', False)]}"/>
                                            <field name="mobile" id="mobile_page_lead" widget="phone"/>
                                        </div>
                                    </group>
                                    <group string="Marketing">
                                        <field name="campaign_id" />
                                        <field name="medium_id" />
                                        <field name="source_id" />
                                    </group>
                                    <group string="Misc" name="Misc">
                                        <field name="day_open" groups="base.group_no_one"/>
                                        <field name="day_close" groups="base.group_no_one"/>
                                        <field name="referred"/>
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
                    <field name="date_deadline" invisible="1"/>
                    <field name="create_date" optional="hide"/>
                    <field name="name" string="Lead" readonly="1"/>
                    <field name="contact_name" optional="hide"/>
                    <field name="partner_name" optional="hide"/>
                    <field name="email_from" optional="show"/>
                    <field name="phone" optional="show" class="o_force_ltr"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show"/>
                    <field name="city" optional="show"/>
                    <field name="state_id" optional="hide"/>
                    <field name="country_id" optional="show"/>
                    <field name="partner_id" invisible="1"/>
                    <field name="user_id" optional="show"  widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                    <field name="team_id" optional="show"/>
                    <field name="active" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="campaign_id" optional="hide"/>
                    <field name="referred" invisible="1"/>
                    <field name="medium_id" optional="hide"/>
                    <field name="source_id" optional="hide"/>
                    <field name="message_needaction" invisible="1"/>
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
                                  <field name="tag_ids"/>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <field name="priority" widget="priority"/>
                                        <div class="o_kanban_inline_block">
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="user_id" widget="many2one_avatar_user"/>
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
                    <field name="partner_id" avatar_field="image_128"/>
                    <field name="user_id" filters="1" invisible="1"/>
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
                            string='Organization / Contact'
                            context="{
                            'res_partner_search_mode': type == 'opportunity' and 'customer' or False,
                            'default_name': contact_name or partner_name,
                            'default_is_company': type == 'opportunity' and contact_name == False,
                            'default_company_name': type == 'opportunity' and partner_name,
                            'default_phone': phone,
                            'default_email': email_from,
                            'show_vat': True}"/>
                        <field name="name" placeholder="e.g. Product Pricing" />
                        <field name="email_from" string="Email" />
                        <field name="phone" string="Phone" />
                        <label for="expected_revenue"/>
                        <div class="o_row">
                            <field name="expected_revenue" class="oe_inline mr-5" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                            <field name="priority" class="oe_inline" nolabel="1" widget="priority"/>
                        </div>
                        <div class="o_row">
                            <field name="recurring_revenue" class="oe_inline pr-4" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                            <field name="recurring_plan" class="oe_inline" placeholder="E.g. Monthly"
                                   attrs="{'required': [('recurring_revenue', '!=', 0)]}" options="{'no_create': True, 'no_open': True}"/>
                        </div>
                        <field name="company_currency" invisible="1"/>
                        <field name="company_id" invisible="1"/>
                        <field name="user_id" invisible="1"/>
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
                            <img t-att-src="activity_image('res.users', 'image_128', record.user_id.raw_value)" t-att-title="record.user_id.value" t-att-alt="record.user_id.value"/>
                            <div>
                                <field name="name" display="full"/>
                                <field name="expected_revenue" widget="monetary" display="full" muted="1"/>
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
                    archivable="false" sample="1">
                    <field name="stage_id" options='{"group_by_tooltip": {"requirements": "Description"}}'/>
                    <field name="color"/>
                    <field name="priority"/>
                    <field name="expected_revenue"/>
                    <field name="kanban_state"/>
                    <field name="activity_date_deadline"/>
                    <field name="user_email"/>
                    <field name="user_id"/>
                    <field name="partner_id"/>
                    <field name="activity_summary"/>
                    <field name="active"/>
                    <field name="company_currency"/>
                    <field name="activity_state" />
                    <field name="activity_ids" />
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}' sum_field="expected_revenue" help="This bar allows to filter the opportunities based on scheduled activities."/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''} oe_kanban_global_click">
                                <div class="o_dropdown_kanban dropdown">

                                    <a class="dropdown-toggle o-no-caret btn" role="button" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                        <span class="fa fa-ellipsis-v"/>
                                    </a>
                                    <div class="dropdown-menu" role="menu">
                                        <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit</a></t>
                                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                                    </div>
                                </div>
                                <div class="oe_kanban_content">
                                    <div class="o_kanban_record_title">
                                        <strong><field name="name"/></strong>
                                    </div>
                                    <div class="o_kanban_record_subtitle">
                                        <t t-if="record.expected_revenue.raw_value">
                                            <field name="expected_revenue" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                                            <span t-if="record.recurring_revenue and record.recurring_revenue.raw_value"> + </span>
                                        </t>
                                        <t t-if="record.recurring_revenue and record.recurring_revenue.raw_value">
                                            <field name="recurring_revenue" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                                            <field name="recurring_plan"/>
                                        </t>
                                    </div>
                                    <div>
                                        <span t-if="record.partner_id.value" t-esc="record.partner_id.value"></span>
                                    </div>
                                    <div>
                                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                    </div>
                                    <div class="o_kanban_record_bottom">
                                        <div class="oe_kanban_bottom_left">
                                            <field name="priority" widget="priority" groups="base.group_user"/>
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                        <div class="oe_kanban_bottom_right">
                                            <field name="user_id" widget="many2one_avatar_user"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="oe_clear"/>
                            </div>
                        </t>
                    </templates>
                </kanban>
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
                            domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                            help="Show all opportunities for which the next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                            domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                            domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
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
                    </group>
                </search>
            </field>
        </record>

        <!--
            MASS MAILING
        -->
        <record id="crm_lead_act_window_compose" model="ir.actions.act_window">
            <field name="name">Send email</field>
            <field name="res_model">mail.compose.message</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="context" eval="{
    'default_composition_mode': 'comment',
    'default_use_template': True,
    'default_template_id': ref('crm.email_template_opportunity_mail'),
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
    'default_use_template': True,
    'default_template_id': ref('crm.email_template_opportunity_mail'),
                }"/>
            <field name="binding_model_id" ref="model_crm_lead"/>
            <field name="binding_view_types">list</field>
        </record>

        <!--Update of email_template defined in crm_lead_data, to add ref_ir_act_window
            allowing to have a well formed email template (context action considered as set). -->
        <record id="email_template_opportunity_mail" model="mail.template">
            <field name="ref_ir_act_window" ref="crm.action_lead_mass_mail"/>
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
                    <field name="date_deadline" invisible="1"/>
                    <field name="create_date" optional="hide"/>
                    <field name="name" string="Opportunity" readonly="1"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="contact_name" optional="show"/>
                    <field name="email_from"/>
                    <field name="phone" class="o_force_ltr"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show"/>
                    <field name="city" optional="hide"/>
                    <field name="state_id" optional="hide"/>
                    <field name="country_id" optional="hide"/>
                    <field name="user_id" widget="many2one_avatar_user" optional="show" domain="[('share', '=', False)]"/>
                    <field name="team_id" optional="show"/>
                    <field name="priority" optional="hide"/>
                    <field name="activity_ids" widget="list_activity"/>
                    <field name="activity_user_id" optional="hide" string="Activity by" widget="many2one_avatar_user"/>
                    <field name="activity_date_deadline_my" string="My Deadline" widget="remaining_days" options="{'allow_order': '1'}"/>
                    <field name="campaign_id" optional="hide"/>
                    <field name="medium_id" optional="hide"/>
                    <field name="source_id" optional="hide"/>
                    <field name="company_currency" invisible="1"/>
                    <field name="expected_revenue" sum="Expected Revenues" optional="show" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                    <field name="recurring_revenue_monthly" sum="Expected MRR" optional="show" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                    <field name="recurring_revenue" sum="Recurring Revenue" optional="hide" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                    <field name="recurring_plan" optional="hide"/>
                    <field name="stage_id" optional="show" decoration-bf="1"/>
                    <field name="active" invisible="1"/>
                    <field name="probability" optional="hide"/>
                    <field name="tag_ids" optional="hide" widget="many2many_tags" options="{'color_field': 'color'}"/>
                    <field name="referred" invisible="1"/>
                    <field name="message_needaction" invisible="1"/>
                </tree>
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
                    <attribute name="default_order">activity_date_deadline_my</attribute>
                </xpath>
                <xpath expr="//tree" position="inside">
                    <header>
                        <button name="%(crm.action_lead_mass_mail)d" type="action" string="Email" />
                    </header>
                </xpath>
                <field name="user_id" position="attributes">
                    <attribute name="optional">hide</attribute>
                </field>
                <field name="team_id" position="attributes">
                    <attribute name="optional">hide</attribute>
                </field>
                <field name="message_needaction" position="after">
                    <button name="action_snooze" class="text-warning" type="object" string="Snooze 7d" icon="fa-bell-slash" />
                    <button name="%(crm.crm_lead_act_window_compose)d" type="action" string="Email" icon="fa-envelope"/>
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
                    <field name="stage_id" type="col"/>
                    <field name="user_id" type="row"/>
                    <field name="color" invisible="1"/>
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
                    <field name="recurring_revenue_monthly" type="measure"/>
                    <field name="color" invisible="1"/>
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
                    <separator/>
                    <filter string="My Pipeline" name="assigned_to_me"
                        domain="[('user_id', '=', uid)]"
                        help="Opportunities that are assigned to me"/>
                    <filter string="Unassigned" name="unassigned"
                        domain="[('user_id', '=', False)]" help="No salesperson"/>
                    <filter string="Open Opportunities" name="open_opportunities"
                        domain="[('probability', '&lt;', 100), ('type', '=', 'opportunity')]"
                        help="Open Opportunities"/>
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
                    <separator/>
                    <filter string="Creation Date" name="creation_date" date="create_date"/>
                    <filter string="Closed Date" name="close_date" date="date_closed"/>
                    <separator/>
                    <filter string="Won" name="won" domain="['&amp;', ('active', '=', True), ('stage_id.is_won', '=', True)]"/>
                    <filter string="Lost" name="lost" domain="['&amp;', ('active', '=', False), ('probability', '=', 0)]"/>
                    <separator/>
                    <filter invisible="1" string="Overdue Opportunities" name="overdue_opp" domain="['&amp;', ('date_closed', '=', False), ('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all opportunities for which the next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By" colspan="16">
                        <filter string="Salesperson" name="salesperson" context="{'group_by':'user_id'}"/>
                        <filter string="Sales Team" name="saleschannel" context="{'group_by':'team_id'}"/>
                        <filter name="stage" string="Stage" context="{'group_by':'stage_id'}"/>
                        <filter name="city" string="City" context="{'group_by': 'city'}"/>
                        <filter string="Country" name="country" context="{'group_by':'country_id'}" />
                        <filter string="Lost Reason" name="lostreason" context="{'group_by':'lost_reason'}"/>
                        <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Campaign" name="compaign" domain="[]" context="{'group_by':'campaign_id'}"/>
                        <filter string="Medium" name="medium" domain="[]" context="{'group_by':'medium_id'}"/>
                        <filter string="Source" name="source" domain="[]" context="{'group_by':'source_id'}"/>
                        <separator orientation="vertical" />
                        <filter string="Creation Date" context="{'group_by':'create_date:month'}" name="month"/>
                        <filter string="Conversion Date" name="date_conversion" context="{'group_by': 'date_conversion'}" groups="crm.group_use_lead"/>
                        <filter string="Expected Closing" name="date_deadline" context="{'group_by':'date_deadline'}"/>
                        <filter string="Closed Date" name="date_closed" context="{'group_by':'date_closed'}"/>
                    </group>
                </search>
            </field>
        </record>

        <!--
            'Mark as Lost' in action dropdown
        -->
        <record id="action_mark_as_lost" model="ir.actions.server">
            <field name="name">Mark as lost</field>
            <field name="model_id" ref="model_crm_lead"/>
            <field name="binding_model_id" ref="crm.model_crm_lead"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
if not 'opportunity' in records.mapped('type'):
    records.action_set_lost()
elif records:
    action_values = env.ref('crm.crm_lead_lost_action').sudo().read()[0]
    action_values.update({'context': env.context})
    action = action_values
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

        <record id="crm_menu_leads" model="ir.ui.menu">
            <field name="action" ref="crm.crm_lead_all_leads"/>
        </record>

        <!-- My Activities Menu -->
        <record id="crm_lead_action_my_activities" model="ir.actions.act_window">
            <field name="name">My Activities</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">tree,kanban,graph,pivot,calendar,form,activity</field>
            <field name="view_id" ref="crm_lead_view_list_activities"/>
            <field name="domain">[('activity_ids','!=',False)]</field>
            <field name="search_view_id" ref="crm.view_crm_case_my_activities_filter"/>
            <field name="context">{'default_type': 'opportunity',
                    'search_default_assigned_to_me': 1}
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

        <record id="action_your_pipeline" model="ir.actions.server">
            <field name="name">Crm: My Pipeline</field>
            <field name="model_id" ref="crm.model_crm_team"/>
            <field name="state">code</field>
            <field name="groups_id"  eval="[(4, ref('base.group_user'))]"/>
            <field name="code">action = model.action_your_pipeline()</field>
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

        <record id="menu_crm_opportunities" model="ir.ui.menu">
            <field name="action" ref="crm.action_your_pipeline"/>
        </record>
        <record id="crm_menu_root" model="ir.ui.menu">
            <field name="action" ref="crm.action_your_pipeline"/>
        </record>
        <record id="crm_lead_menu_my_activities" model="ir.ui.menu">
            <field name="action" ref="crm.crm_lead_action_my_activities"/>
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
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <div class="oe_edit_only">
                            <label for="name"/>
                        </div>
                        <h1 class="mb32">
                            <field name="name" class="mb16"/>
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
            Define a new lost reason
          </p><p>
            Use lost reasons to explain why an opportunity is lost.
          </p><p>
            Some examples of lost reasons: "We don't have people/skill", "Price too high"
          </p>
        </field>
    </record>

    <record id="menu_crm_lost_reason" model="ir.ui.menu">
        <field name="action" ref="crm.crm_lost_reason_action"/>
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
        sequence="6"/>

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
        sequence="1"/>
    <menuitem
        id="crm_lead_menu_my_activities"
        name="My Activities"
        parent="crm_menu_sales"
        groups="sales_team.group_sale_manager"
        sequence="2"/>

    <menuitem
        id="sales_team_menu_team_pipeline"
        name="Teams"
        parent="crm_menu_sales"
        action="sales_team.crm_team_salesteams_pipelines_act"
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
        groups="crm.group_use_lead"
        sequence="5"/>

    <!-- REPORTING -->
    <menuitem
        id="crm_menu_report"
        name="Reporting"
        parent="crm_menu_root"
        sequence="20"
        groups="sales_team.group_sale_manager"/>
    <menuitem
        id="crm_opportunity_report_menu_lead"
        name="Leads"
        parent="crm_menu_report"
        groups="crm.group_use_lead"
        sequence="1"/>
    <menuitem
        id="crm_opportunity_report_menu" 
        name="Pipeline"
        parent="crm_menu_report" 
        sequence="2"/>

    <!-- CONFIGURATION -->
    <menuitem
        id="crm_menu_config"
        name="Configuration"
        parent="crm_menu_root"
        sequence="25" groups="sales_team.group_sale_manager"/>
    <menuitem
        id="crm_config_settings_menu"
        name="Settings"
        parent="crm_menu_config"
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
        action="sales_team.sales_team_config_action"
        sequence="5"/>
    <menuitem
        id="crm_team_menu_config_activity_types"
        name="Activity Types"
        parent="crm_menu_config"
        action="sales_team.mail_activity_type_action_config_sales"
        sequence="10"/>
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
                        <div class="oe_edit_only">
                            <label for="name"/>
                        </div>
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="is_won"/>
                            <field name="fold"/>
                            <field name="team_id" options='{"no_open": True, "no_create": True}' attrs="{'invisible': [('team_count', '&lt;=', 1)]}" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
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

    <record id="menu_crm_lead_stage_act" model="ir.ui.menu">
        <field name="action" ref="crm.crm_stage_action"/>
    </record>

</odoo>

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
           <field name="context">{'search_default_team_id': [active_id], 'search_default_filter_create_date': 1}}</field>
           <field name="domain">[]</field>
           <field name="view_mode">graph,pivot,tree,form</field>
           <field name="search_view_id" ref="crm.crm_opportunity_report_view_search"/>
           <field name="help">Leads Analysis allows you to check different CRM related information like the treatment delays or number of leads per state. You can sort out your leads analysis by different groups to get accurate grained analysis.</field>
       </record>

       <record id="action_report_crm_opportunity_salesteam" model="ir.actions.act_window">
            <field name="name">Pipeline Analysis</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">graph,pivot,tree,form</field>
            <field name="search_view_id" ref="crm.crm_opportunity_report_view_search"/>
            <field name="context">{
                'search_default_team_id': [active_id],
                'tree_view_ref': 'crm.crm_case_tree_view_oppor',
                'search_default_opportunity': True,
                'search_default_filter_create_date': 1}</field>
            <field name="domain">[]</field>
            <field name="help">Opportunities Analysis gives you an instant access to your opportunities with information such as the expected revenue, planned cost, missed deadlines or the number of interactions per opportunity. This report is mainly used by the sales manager in order to do the periodic review with the channels of the sales pipeline.</field>
        </record>

        <record id="sales_team_form_view_in_crm" model="ir.ui.view">
            <field name="name">crm.team.form.inherit</field>
            <field name="model">crm.team</field>
            <field name="inherit_id" ref="sales_team.crm_team_view_form"/>
            <field name="priority">12</field>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='options_active']" position="inside">
                    <div class="o_row">
                        <span name="opportunities">
                            <field name="use_opportunities"/>
                            <label for="use_opportunities"/>
                        </span>
                        <span class="o_row" groups="crm.group_use_lead">
                            <field name="use_leads"/>
                            <label for="use_leads" string="Leads"/>
                        </span>
                    </div>
                </xpath>
                <xpath expr="//field[@name='user_id']" position="after">
                    <label for="alias_name" string="Email Alias"
                        attrs="{'invisible': [('use_leads', '=', False),('use_opportunities', '=', False)]}"/>
                    <div class="oe_inline" name="alias_def"
                        attrs="{'invisible': [('use_leads', '=', False),('use_opportunities', '=', False)]}">
                        <field name="alias_id" class="oe_read_only oe_inline"
                            string="Email Alias" required="0"/>
                        <div class="oe_edit_only oe_inline" name="edit_alias" style="display: inline;" >
                            <div attrs="{'invisible': [('alias_domain', '=', False)]}">
                                <field name="alias_name" class="oe_inline"/>@<field name="alias_domain" class="oe_inline" readonly="1"/>
                            </div>
                            <button icon="fa-arrow-right" type="action" name="%(base_setup.action_general_configuration)d" string="Configure a custom domain" class="p-0 btn-link" attrs="{'invisible': [('alias_domain', '!=', False)]}"/>
                        </div>
                    </div>
                    <field name="alias_contact"
                        string="Accept Emails From"
                        attrs="{'invisible': [('use_leads', '=', False), ('use_opportunities', '=', False)]}"/>
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
            <field name="type">ir.actions.act_window</field>
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

        <record id="sales_team.crm_team_salesteams_pipelines_act" model="ir.actions.act_window">
            <field name="domain">[('use_opportunities', '=', True)]</field>
        </record>

        <record id="crm_team_salesteams_view_kanban" model="ir.ui.view">
            <field name="name">crm.team.kanban</field>
            <field name="model">crm.team</field>
            <field name="inherit_id" ref="sales_team.crm_team_salesteams_view_kanban"/>
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
                        <div t-if="record.use_leads.raw_value and record.alias_name.value and record.alias_domain.value">
                            <small t-translation="off"><i class="fa fa-envelope-o" aria-label="Leads" title="Leads" role="img"></i>&amp;nbsp; <field name="alias_id"/></small>
                        </div>
                    </xpath>

                    <xpath expr="//t[@name='first_options']" position="after">
                        <div class="row" t-if="record.lead_unassigned_count.raw_value">
                            <div class="col-8">
                                <a name="%(crm_case_form_view_salesteams_lead)d" type="action" context="{'search_default_unassigned_leads': 1}">
                                    <field name="lead_unassigned_count"/>
                                    <t t-if="record.lead_unassigned_count.raw_value == 1">Unassigned Lead</t>
                                    <t t-else="">Unassigned Leads</t>
                                </a>
                            </div>
                        </div>
                        <div class="row" t-if="record.opportunities_count.raw_value">
                            <div class="col-8">
                                <a name="%(crm_case_form_view_salesteams_opportunity)d" type="action" context="{'search_default_open_opportunities': True}"> <!-- context="{'search_default_probability': NOT or < 100}" -->
                                    <field name="opportunities_count"/>
                                    <t t-if="record.opportunities_count.raw_value == 1">Open Opportunity</t>
                                    <t t-else="">Open Opportunities</t>
                                </a>
                            </div>
                            <div class="col-4 text-right text-truncate">
                                <field name="opportunities_amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            </div>
                        </div>
                        <div class="row" t-if="record.opportunities_overdue_count.raw_value">
                            <div class="col-8">
                                <a name="%(crm_lead_action_team_overdue_opportunity)d" type="action">
                                    <field name="opportunities_overdue_count"/>
                                    <t t-if="record.opportunities_overdue_count.raw_value == 1">Overdue Opportunity</t>
                                    <t t-else="">Overdue Opportunities</t>
                                </a>
                            </div>
                             <div class="col-4 text-right text-truncate">
                                <field name="opportunities_overdue_amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            </div>
                        </div>
                    </xpath>

                    <xpath expr="//div[hasclass('o_kanban_manage_view')]" position="inside">
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

                    <xpath expr="//div[hasclass('o_kanban_manage_new')]" position="inside">
                        <div t-if="record.use_opportunities.raw_value">
                            <a  name="%(action_opportunity_form)d" type="action">
                                Opportunity
                            </a>
                        </div>
                    </xpath>

                    <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
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
        <field name="inherit_id" ref="digest.digest_digest_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpi_general']" position="after">
                <group name="kpi_crm" string="CRM" groups="sales_team.group_sale_salesman_all_leads">
                    <field name="kpi_crm_lead_created"/>
                    <field name="kpi_crm_opportunities_won"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sales_team.mail_activity_type_action_config_sales" model="ir.actions.act_window">
        <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', 'in', ['crm.lead', 'res.partner'])]</field>
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
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="CRM" string="CRM" data-key="crm" groups="sales_team.group_sale_manager">
                    <h2>CRM</h2>
                    <div class="row mt16 o_settings_container" name="qualification_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="crm_lead"
                            title="Use leads if you need a qualification step before creating an opportunity or a customer. It can be a business card you received, a contact form filled in your website, or a file of unqualified prospects you import, etc. Once qualified, the lead can be converted into a business opportunity and/or a new customer in your address book.">
                            <div class="o_setting_left_pane">
                                <field name="group_use_lead"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_use_lead"/>
                                <div class="text-muted">
                                    Add a qualification step before the creation of an opportunity
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="crm_lead"
                                attrs="{'invisible': [('group_use_lead','=',False)]}"
                                title="Emails received to that address generate new leads not assigned to any Sales Team yet. This can be made when converting them into opportunities. Incoming emails can be automatically assigned to specific Sales Teams. To do so, set an email alias on the Sales Team.">
                            <div class="o_setting_left_pane">
                                <field name="generate_lead_from_alias"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="generate_lead_from_alias" string="Incoming Emails"/>
                                <div class="text-muted">
                                    Create leads from incoming emails
                                </div>
                                <div class="content-group" attrs="{'invisible': [('generate_lead_from_alias','=',False)]}">
                                    <div class="mt16">
                                        <field name="crm_alias_prefix" class="oe_inline"
                                            attrs="{'required': [('generate_lead_from_alias', '=', True)]}"/>
                                        <label class="mr-0" for="alias_domain" string="@"/>
                                        <field name="alias_domain" readonly="1" force_save="1" class="oe_inline"/>
                                    </div>
                                    <div attrs="{'invisible': [('alias_domain', 'not in', ['localhost', '', False])]}">
                                        <button type="action"
                                            name="base_setup.action_general_configuration"
                                            string="Use an External Email Server" icon="fa-arrow-right" class="oe_link"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="group_use_recurring_revenues"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_use_recurring_revenues"/>
                                <div class="text-muted">
                                    Define recurring plans and revenues on Opportunities
                                </div>
                                <div attrs="{'invisible': [('group_use_recurring_revenues', '=', False)]}">
                                    <button type="action" name="crm.crm_recurring_plan_action"
                                            string="Manage Recurring Plans" icon="fa-arrow-right" class="oe_link"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box"
                            title="This can be used to compute statistical probability to close a lead"
                            name="predictive_lead_setting_container">
                            <div class="o_setting_left_pane"></div>
                            <div class="o_setting_right_pane">
                                <b>Predictive Lead Scoring</b>
                                <div class="text-muted">
                                    The success rate is computed based on the stage, but you can add more fields in the statistical analysis.
                                </div>
                                <div class="mt16">
                                    <field name="predictive_lead_scoring_fields" class="oe_inline" widget="many2many_tags" placeholder="Extra fields..."/>
                                    <field name="predictive_lead_scoring_fields_str" invisible="1"/>
                                </div>
                                <div class="mt16">
                                    Consider leads created as of the
                                    <field name="predictive_lead_scoring_start_date_str" invisible="1"/>
                                    <field name="predictive_lead_scoring_start_date" class="oe_inline" required="1"/>
                                </div>
                                <div class="mt16" groups="base.group_erp_manager">
                                    <div class="text-muted">
                                        Use this button to update the probabilities of all leads. This can take up to several minutes depending on how many there are.
                                    </div>
                                    <button name="action_reset_lead_probabilities" type="object" string="Update Probabilities" class="btn-primary"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Lead Generation</h2>
                    <div class="row mt16 o_settings_container" name="convert_visitor_setting_container">
                        <div class="col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="module_crm_iap_lead_website"/>
                            </div>

                            <div class="o_setting_right_pane" id="crm_iap_lead_website_settings">
                                <label string="Visits to Leads" for="module_crm_iap_lead_website"/>
                                <div class="text-muted">
                                    Convert visitors of your website into leads and perform data enrichment based on their IP address
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="module_crm_iap_lead_enrich"/>
                            </div>

                            <div class="o_setting_right_pane" id="crm_iap_lead_enrich">
                                <label string="Lead Enrichment" for="module_crm_iap_lead_enrich"/>
                                <div class="text-muted">
                                    Enrich your leads automatically with company data based on their email address
                                </div>
                                <div id="crm_iap_lead_enrich">
                                    <div class="mt8" attrs="{'invisible': [('module_crm_iap_lead_enrich','=',False)]}">
                                        <field name="lead_enrich_auto" class="o_light_label" widget="radio" required="True"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="row mt16 o_settings_container" name="generate_lead_setting_container">
                        <div class="col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="module_crm_iap_lead"/>
                            </div>

                            <div class="o_setting_right_pane" id="crm_iap_lead_settings">
                                <label string="Lead Mining" for="module_crm_iap_lead"/>
                                <a href="https://www.odoo.com/documentation/14.0/applications/sales/crm/acquire_leads/lead_mining.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Generate new leads based on their country, industry, size, etc.
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="module_mail_client_extension"/>
                            </div>

                            <div class="o_setting_right_pane" id="mail_client_extension">
                                <label string="Outlook CRM Extension" for="module_mail_client_extension"/>
                                <a href="https://www.odoo.com/documentation/14.0/applications/sales/crm/optimize/outlook_extension.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Turn emails received in your Outlook mailbox into leads and log their content as internal notes.
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="crm_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_id" ref="res_config_settings_view_form"/>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'crm', 'bin_size': False}</field>
    </record>

    <record id="crm_config_settings_menu" model="ir.ui.menu">
        <field name="action" ref="crm.crm_config_settings_action"/>
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
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
            <field name="arch" type="xml">
                <field name="mobile" position="after">
                    <field name="opportunity_count"/>
                    <field name="meeting_count"/>
                </field>
                <xpath expr="//span[hasclass('oe_kanban_partner_links')]" position="inside">
                    <span class="badge badge-pill" t-if="record.opportunity_count.value>0"><i class="fa fa-fw fa-star" aria-label="Favorites" role="img" title="Favorites"/><t t-esc="record.opportunity_count.value"/></span>
                    <span class="badge badge-pill" t-if="record.meeting_count.value>0"><i class="fa fa-fw fa-calendar" aria-label="Meetings" role="img" title="Meetings"/><t t-esc="record.meeting_count.value"/></span>
                </xpath>
            </field>
        </record>

        <!-- Add contextual button on partner form view -->
        <record id="view_partners_form_crm1" model="ir.ui.view">
            <field name="name">view.res.partner.form.crm.inherited1</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field eval="1" name="priority"/>
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
            <field name="arch" type="xml">
                <data>
                    <div name="button_box" position="inside">
                        <button class="oe_stat_button o_res_partner_tip_opp" type="object"
                            name="action_view_opportunity"
                            icon="fa-star"
                            groups="sales_team.group_sale_salesman"
                            context="{'default_partner_id': active_id}">
                            <field string="Opportunities" name="opportunity_count" widget="statinfo"/>
                        </button>
                        <button class="oe_stat_button" type="object"
                            name="schedule_meeting"
                            icon="fa-calendar"
                            groups="sales_team.group_sale_salesman"
                            context="{'partner_id': active_id, 'partner_name': name}">
                            <field string="Meetings" name="meeting_count" widget="statinfo"/>
                        </button>
                    </div>
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
            <xpath expr="//div[@id='utm_statistics']" position="inside">
                <div class="mr-3"
                    groups="sales_team.group_sale_salesman"
                    t-att-title="record.use_leads.raw_value ? 'Leads' : 'Opportunities'">
                    <i class="fa fa-star text-muted"></i>
                    <small class="font-weight-bold"><field name="crm_lead_count"/></small>
                </div>
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
                        <span class="o_stat_text" attrs="{'invisible': [('use_leads', '=', False)]}">Leads</span>
                        <span class="o_stat_text" attrs="{'invisible': [('use_leads', '=', True)]}">Opportunities</span>
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

from odoo import api, fields, models


class CrmLeadLost(models.TransientModel):
    _name = 'crm.lead.lost'
    _description = 'Get Lost Reason'

    lost_reason_id = fields.Many2one('crm.lost.reason', 'Lost Reason')

    def action_lost_reason_apply(self):
        leads = self.env['crm.lead'].browse(self.env.context.get('active_ids'))
        return leads.action_set_lost(lost_reason=self.lost_reason_id.id)

```

## File: wizard\crm_lead_lost_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="crm_lead_lost_view_form" model="ir.ui.view">
            <field name="name">crm.lead.lost.form</field>
            <field name="model">crm.lead.lost</field>
            <field name="arch" type="xml">
                <form string="Lost Reason">
                    <group class="oe_title">
                        <field name="lost_reason_id" options="{'no_create_edit': True}" />
                    </group>
                    <footer>
                        <button name="action_lost_reason_apply" string="Submit" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="crm_lead_lost_action" model="ir.actions.act_window">
            <field name="name">Lost Reason</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">crm.lead.lost</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="crm_lead_lost_view_form"/>
            <field name="target">new</field>
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

        if not result.get('lead_id') and self.env.context.get('active_id'):
            result['lead_id'] = self.env.context.get('active_id')
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
            team_domain = []
            team = self.env['crm.team']._get_default_team_id(user_id=user.id, domain=team_domain)
            convert.team_id = team.id

    @api.model
    def view_init(self, fields):
        # JEM TDE FIXME: clean that brol
        """ Check some preconditions before the wizard executes. """
        for lead in self.env['crm.lead'].browse(self._context.get('active_ids', [])):
            if lead.probability == 100:
                raise UserError(_("Closed/Dead leads cannot be converted into opportunities."))
        return False

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
        (to_merge - result_opportunity).unlink()
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

            lead.convert_opportunity(lead.partner_id.id, [], False)

        leads_to_allocate = leads
        if not self.force_assignment:
            leads_to_allocate = leads_to_allocate.filtered(lambda lead: not lead.user_id)

        if user_ids:
            leads_to_allocate.handle_salesmen_assignment(user_ids, team_id=team_id)

    def _convert_handle_partner(self, lead, action, partner_id):
        # used to propagate user_id (salesman) on created partners during conversion
        lead.with_context(default_user_id=self.user_id.id).handle_partner_assignment(
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
            team_domain = []
            team = self.env['crm.team']._get_default_team_id(user_id=user.id, domain=team_domain)
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
                    <field name="team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
                    <field name="user_ids" widget="many2many_tags" domain="[('share', '=', False)]"/>
                    <field name="force_assignment"/>
                </group>
                <label for="duplicated_lead_ids" string="Leads with existing duplicates (for information)" help="Leads that you selected that have duplicates. If the list is empty, it means that no duplicates were found" attrs="{'invisible': [('deduplicate', '=', False)]}"/>
                <group attrs="{'invisible': [('deduplicate', '=', False)]}">
                    <field name="duplicated_lead_ids" colspan="4" nolabel="1" readonly="1">
                        <tree create="false" delete="false">
                            <field name="create_date" widget="date"/>
                            <field name="name"/>
                            <field name="type"/>
                            <field name="contact_name"/>
                            <field name="country_id" invisible="context.get('invisible_country', True)"/>
                            <field name="email_from"/>
                            <field name="stage_id"/>
                            <field name="user_id"/>
                            <field name="team_id"/>
                        </tree>
                    </field>
                </group>
                <group attrs="{'invisible': [('name', '!=', 'convert')]}" string="Customers" col="1">
                    <field name="action" class="oe_inline" widget="radio"/>
                    <group col="2">
                        <field name="partner_id"
                            widget="res_partner_many2one"
                            attrs="{'required': [('action', '=', 'exist')], 'invisible':[('action','!=','exist')]}"
                            context="{'show_vat': True}"
                            class="oe_inline"/>
                    </group>
                </group>
                <footer>
                    <button string="Convert to Opportunities" name="action_mass_convert" type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
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
                    <field name="team_id" widget="selection"/>
                </group>
                <group string="Opportunities" attrs="{'invisible': [('name', '!=', 'merge')]}">
                    <field name="lead_id" invisible="1"/>
                    <field name="duplicated_lead_ids" colspan="2" nolabel="1">
                        <tree>
                            <field name="create_date" widget="date"/>
                            <field name="name"/>
                            <field name="type"/>
                            <field name="contact_name"/>
                            <field name="country_id" invisible="context.get('invisible_country', True)"/>
                            <field name="email_from"/>
                            <field name="stage_id"/>
                            <field name="user_id"/>
                            <field name="team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
                        </tree>
                    </field>
                </group>
                <group name="action" attrs="{'invisible': [('name', '!=', 'convert')]}" string="Customer" col="1">
                    <field name="action" nolabel="1" widget="radio"/>
                    <group col="2">
                        <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'customer', 'show_vat': True}" attrs="{'required': [('action', '=', 'exist')], 'invisible':[('action','!=','exist')]}"/>
                    </group>
                </group>
                <footer>
                    <button name="action_apply" string="Create Opportunity" type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_crm_lead2opportunity_partner" model="ir.actions.act_window">
        <field name="name">Convert to opportunity</field>
        <field name="type">ir.actions.act_window</field>
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
    user_id = fields.Many2one('res.users', 'Salesperson', index=True)
    team_id = fields.Many2one(
        'crm.team', 'Sales Team', index=True,
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
                        <field name="team_id" class="oe_inline" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
                    </group>
                    <group string="Select Leads/Opportunities">
                        <field name="opportunity_ids" nolabel="1">
                            <tree>
                                <field name="create_date"/>
                                <field name="name"/>
                                <field name="type"/>
                                <field name="contact_name"/>
                                <field name="email_from"/>
                                <field name="phone" class="o_force_ltr"/>
                                <field name="stage_id"/>
                                <field name="user_id"/>
                                <field name="team_id"/>
                            </tree>
                        </field>
                    </group>
                    <footer>
                        <button name="action_merge" type="object" string="Merge" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
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

```

