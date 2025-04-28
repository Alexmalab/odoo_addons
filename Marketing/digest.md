# Odoo Module: digest

Category: Marketing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'KPI Digests',
    'category': 'Marketing',
    'description': """
Send KPI Digests periodically
=============================
""",
    'version': '1.1',
    'depends': [
        'mail',
        'portal',
        'resource',
    ],
    'data': [
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'data/digest_tips_data.xml',
        'data/ir_cron_data.xml',
        'data/res_config_settings_data.xml',
        'views/digest_views.xml',
        'views/digest_templates.xml',
        'views/res_config_settings_views.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import Forbidden
from werkzeug.urls import url_encode

from odoo import _
from odoo.http import Controller, request, route


class DigestController(Controller):

    @route('/digest/<int:digest_id>/unsubscribe', type='http', website=True, auth='user')
    def digest_unsubscribe(self, digest_id):
        digest = request.env['digest.digest'].browse(digest_id).exists()
        digest.action_unsubcribe()
        return request.render('digest.portal_digest_unsubscribed', {
            'digest': digest,
        })

    @route('/digest/<int:digest_id>/set_periodicity', type='http', website=True, auth='user')
    def digest_set_periodicity(self, digest_id, periodicity='weekly'):
        if not request.env.user.has_group('base.group_erp_manager'):
            raise Forbidden()
        if periodicity not in ('daily', 'weekly', 'monthly', 'quarterly'):
            raise ValueError(_('Invalid periodicity set on digest'))

        digest = request.env['digest.digest'].browse(digest_id).exists()
        digest.action_set_periodicity(periodicity)

        url_params = {
            'model': digest._name,
            'id': digest.id,
            'active_id': digest.id,
        }
        return request.redirect('/web?#%s' % url_encode(url_params))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\digest_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!-- Email layout: encapsulation when sending (not used in backend display) -->
    <template id="digest_mail_layout">
&lt;!DOCTYPE html&gt;
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <meta name="format-detection" content="telephone=no"/>
        <meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1.0; user-scalable=no;"/>
        <meta http-equiv="X-UA-Compatible" content="IE=9; IE=8; IE=7; IE=EDGE" />

        <style type="text/css">
            body {
                margin: 0;
                padding: 0;
                font-family: Arial, Helvetica, Verdana, sans-serif;
            }
            #header_background {
                background-color: #875a7b;
            }
            .global_layout {
                max-width: 588px;
                margin: 0 auto;
                border: 1px solid #eeeeee;
                border-top: 0;
                background-color: #ffffff;
            }
            .company_name {
                display: inline;
                vertical-align: middle;
                font-weight: bold;
                color: #8f8f8f;
            }
            .title_subtitle {
                font-weight: bold;
                color: #282f33;
                word-break: break-all;
            }
            .button {
                float: right;
                background-color: #875a7b;
                color: #ffffff;
                border-radius: 5px;
            }
            #button_connect {
                text-align: center;
            }
            .date {
                color: #8f8f8f;
            }
            .tip_title {
                margin-top: 0;
                font-weight: bold;
            }
            .tip_content {
                margin: 0 auto;
                color:#333333;
                text-align: justify;
                text-justify: inter-word;
            }
            .illustration_border {
                width: 100%;
                border: 1px solid grey;
            }
            .kpi_row_footer {
                padding-bottom: 20px;
            }
            .kpi_header {
                overflow: auto;
                margin-bottom: 10px;
                padding-bottom: 10px;
                border-bottom: 1px solid #eeeeee;
                font-size: 15px;
                font-weight: bold;
                color: #282f33;
            }
            #button_open_report {
                padding: 5px 10px;
                font-size: 12px;
                font-weight: normal;
            }
            .kpi_cell {
                float: left;
                width: 33%;
                text-align: center;
            }
            .kpi_value {
                color: #282f33;
                font-weight: bold;
                text-decoration: none;
            }
            .kpi_value_label {
                display: inline-block;
                margin-bottom: 10px;
                color: #888888;
                font-size: 12px;
                text-transform: uppercase;
            }
            .kpi_margin {
                width: 75px;
                padding: 5px 10px;
                text-decoration: none;
                border-radius: 5px;
            }
            .positive_kpi_margin {
                border: 1px solid #c4ecd7;
                background-color: #d5f1e2;
                color: #17613a;
            }
            .negative_kpi_margin {
                border: 1px solid #f4cfce;
                background-color: #f7dddc;
                color: #712b29;
            }
            .kpi_trick {
                clear: both;
            }
            .download_app {
                height: 40px;
                margin-bottom: 5px;
                display: inline-block;
            }
            .preference_div {
                clear: both;
                background-color: #eeeeee;
                text-align: center;
            }
            .preference {
                margin-bottom: 15px;
                color: #6b6d70;
            }
            .preference a {
                text-decoration: none;
            }
            .by_odoo {
                color: #8f8f8f;
            }
            .odoo_link {
                text-decoration: none;
            }
            .odoo_link_text {
                font-weight: bold;
                color: #875a7b;
            }
            .run_business {
                color: #2d2a26;
            }
            #footer {
                background-color: #eeeeee;
                color: #8f8f8f;
                text-align: center;
            }

            @media only screen and (max-width: 650px) {
                .d-block {
                    display: block !important;
                }
                #header_background {
                    padding-top: 0px;
                }
                #header {
                    padding: 15px 20px;
                    border: 1px solid #eeeeee;
                }
                .global_layout {
                    padding: 20px;
                }
                .company_name {
                    font-size: 15px;
                }
                .title_subtitle {
                    margin: 5px auto;
                }
                #button_connect {
                    padding: 4px 10px;
                    font-size: 12px;
                }
                .date {
                    margin-top: 10px;
                    font-size: 10px;
                }
                .tip_title {
                    font-size: 14px;
                }
                .tip_content {
                    margin: 10px auto 0 auto;
                    font-size: 12px;
                    line-height: 20px;
                }
                .illustration_border {
                    margin-top: 15px;
                }
                .kpi_value {
                    font-size: 20px;
                }
                .kpi_margin {
                    font-size: 10px;
                }
                .kpi_margin_margin {
                    margin-bottom: 5px;
                }
                .preference_div {
                    padding: 15px;
                }
                .preference {
                    font-size: 12px;
                }
                .by_odoo {
                    font-size: 10px;
                }
                .run_business {
                    margin: 20px auto;
                    font-size: 12px;
                }
                #footer {
                    font-size: 15px;
                }
                #powered {
                    margin-top: 15px;
                }
                .tip_twocol {
                    overflow: auto;
                    margin-top: 15px;
                }
                .tip_twocol_left {
                    text-align: left;
                }
                .tip_twocol_img {
                    width: 90%;
                }
            }

            @media only screen and (min-width: 651px) {
                #header_background {
                    padding-top: 70px;
                }
                #header {
                    padding: 20px 30px 25px 30px;
                    border-left: 1px solid #875a7b;
                    border-right: 1px solid #875a7b;
                }
                .global_layout {
                    padding: 25px 30px 30px 30px;
                }
                .company_name {
                    font-size: 25px;
                }
                .title_subtitle {
                    margin: 10px auto;
                }
                .button {
                    padding: 5px 10px;
                }
                #button_connect {
                    padding: 6px 10px;
                    font-size: 15px;
                }
                .date {
                    margin-top: 15px;
                    font-size: 12px;
                }
                .tip_title {
                    font-size: 20px;
                }
                .tip_content {
                    margin: 15px auto 0 auto;
                    font-size: 15px;
                    line-height: 25px;
                }
                .illustration_border {
                    margin-top: 20px;
                }
                .kpi_value {
                    font-size: 30px;
                }
                .kpi_margin {
                    font-size: 12px;
                }
                .kpi_margin_margin {
                    margin-bottom: 10px;
                }
                .preference_div {
                    padding: 20px;
                }
                .preference {
                    font-size: 15px;
                }
                .by_odoo {
                    font-size: 12px;
                }
                .run_business {
                    margin: 15px auto;
                    font-size: 18px;
                }
                #footer {
                    font-size: 20px;
                }
                #stock_tip {
                    overflow: auto;
                    margin-top: 20px;
                }
                #stock_div_img {
                    text-align: center;
                }
                #stock_img {
                    width: 70%;
                }
            }
         </style>
    </head>
    <body>
        <t t-raw="body"/>
    </body>
</html>
    </template>

    <!-- DIGEST MAIN TEMPLATE -->
    <template id="digest_mail_main">
<div id="header_background">
    <div class="global_layout" id="header">
        <div style="overflow: auto;">
            <p t-field="company.name" class="company_name" />
            <a t-att-href="top_button_url" target="_blank"><span t-esc="top_button_label" class="button" id="button_connect" /></a>
        </div>
        <div class="title_subtitle">
            <p t-esc="title" />
            <p t-if="sub_title" t-esc="sub_title" />
        </div>
        <div t-esc="formatted_date" class="date" />
    </div>
</div>

<div style="background-color: #eeeeee;">
<div t-foreach="tips" t-as="tip" t-raw="tip" class="global_layout" />

<div t-if="kpi_data" class="global_layout" style="padding-bottom: 0;">
    <div t-foreach="kpi_data" t-as="kpi_info" class="kpi_row_footer">
        <div t-if="kpi_info.get('kpi_col1') or kpi_info.get('kpi_col2') or kpi_info.get('kpi_col3')">
            <div class="kpi_header">
                <span t-esc="kpi_info['kpi_fullname']" style="vertical-align: middle;" />
                <a t-if="kpi_info['kpi_action']" t-att-href="'/web#action=%s' % kpi_info['kpi_action']"><span class="button" id="button_open_report">Open Report</span></a>
            </div>
            <div t-if="kpi_info.get('kpi_col1')" class="kpi_cell">
                <div t-call="digest.digest_tool_kpi" >
                    <t t-set="kpi_value" t-value="kpi_info['kpi_col1']['value']"/>
                    <t t-set="kpi_margin" t-value="kpi_info['kpi_col1'].get('margin')"/>
                    <t t-set="kpi_subtitle" t-value="kpi_info['kpi_col1']['col_subtitle']"/>
                </div>
            </div>
            <div t-if="kpi_info.get('kpi_col2')" class="kpi_cell">
                <div t-call="digest.digest_tool_kpi">
                    <div t-set="kpi_value" t-value="kpi_info['kpi_col2']['value']"/>
                    <div t-set="kpi_margin" t-value="kpi_info['kpi_col2'].get('margin')"/>
                    <div t-set="kpi_subtitle" t-value="kpi_info['kpi_col2']['col_subtitle']"/>
                </div>
            </div>
            <div t-if="kpi_info.get('kpi_col3')" class="kpi_cell">
                <div t-call="digest.digest_tool_kpi">
                    <div t-set="kpi_value" t-value="kpi_info['kpi_col3']['value']"/>
                    <div t-set="kpi_margin" t-value="kpi_info['kpi_col3'].get('margin')"/>
                    <div t-set="kpi_subtitle" t-value="kpi_info['kpi_col3']['col_subtitle']"/>
                </div>
            </div>
            <div class="kpi_trick" />
        </div>
    </div>
</div>

<div t-if="preferences">
    <div class="global_layout">
        <div class="preference_div">
            <div t-foreach="preferences" t-as="preference" class="preference">
                <t t-raw="preference"/>
            </div>
            <div class="by_odoo">
                Sent by <a href="https://www.odoo.com" target="_blank" class="odoo_link"><span class="odoo_link_text">Odoo</span></a> –
                <a t-att-href="'/web#view_type=form&amp;model=digest.digest&amp;id=%s' % object.id" target="_blank" style="text-decoration: none;"><span style="color: #8f8f8f;">Unsubscribe</span></a>
            </div>
        </div>
    </div>
</div>

<t t-if="body" t-raw="body"/>
<div t-if="display_mobile_banner" t-call="digest.digest_section_mobile" />

<div class="global_layout" id="footer">
    <p style="font-weight: bold;" t-esc="company.name"/>
    <p class="by_odoo" id="powered">
        Powered by <a href="https://www.odoo.com" target="_blank" class="odoo_link"><span class="odoo_link_text">Odoo</span></a>
    </p>
</div>
</div>
    </template>


    <!--                     DIGEST PARTS                    -->

    <!-- MOBILE BANNER -->
    <template id="digest_section_mobile">
<div class="global_layout" style="overflow: auto;">
    <div style="width: 50%; float: left; text-align: right;">
        <img src="https://www.odoo.com/web/image/24717933/odoo-mobile.png" alt="Odoo Mobile" />
    </div>
    <div style="width: 50%; float: left;">
        <p class="run_business">Run your business from anywhere with <b>Odoo Mobile</b>.</p>
        <div>
            <a href="https://play.google.com/store/apps/details?id=com.odoo.mobile" target="_blank"><img class="download_app" src="https://www.odoo.com/digest/static/src/img/google_play.png" /></a>
        </div>
        <div>
            <a href="https://itunes.apple.com/us/app/odoo/id1272543640" target="_blank"><img class="download_app" src="https://www.odoo.com/digest/static/src/img/app_store.png" /></a>
        </div>
    </div>
</div>
    </template>


    <!--                     DIGEST TOOLS                    -->

    <!-- KPI DISPLAY -->
    <template id="digest_tool_kpi">
<span t-esc="kpi_value" class="kpi_value" />
<br/>
<span t-esc="kpi_subtitle" class="kpi_value_label" />
<div t-if="kpi_margin" class="kpi_margin_margin">
    <span   t-if="kpi_margin &gt; 0.0" class="kpi_margin positive_kpi_margin">▲ <t t-esc="'%.2f' % kpi_margin"/> %</span>
    <span t-elif="kpi_margin &lt; 0.0" class="kpi_margin negative_kpi_margin">▼ <t t-esc="'%.2f' % kpi_margin"/> %</span>
</div>
    </template>

</data>
</odoo>

```

## File: data\digest_tips_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest_digest_default" model="digest.digest">
            <field name="name">Your Odoo Periodic Digest</field>
            <field name="periodicity">daily</field>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="next_run_date" eval="DateTime.now().strftime('%Y-%m-%d')"/>
            <field name="kpi_res_users_connected">True</field>
            <field name="kpi_mail_message_total">True</field>
        </record>

        <record id="digest_tip_digest_0" model="digest.tip">
            <field name="name">Tip: Speed up your workflow with shortcuts</field>
            <field name="sequence">800</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Speed up your workflow with shortcuts</p>
    <p class="tip_content">Press ALT in any screen to highlight shortcuts for every button in the screen. It is useful to process multiple documents in batch.</p>
    <img src="/digest/static/src/img/alt-shortcuts.gif" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_1" model="digest.tip">
            <field name="name">Tip: Click on an avatar to chat with a user</field>
            <field name="sequence">2100</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Click on an avatar to chat with a user</p>
    <p class="tip_content">Have a question about a document? Click on the responsible user's picture to start a conversation. If his avatar has a green dot, he is online.</p>
    <img src="/digest/static/src/img/avatar.gif" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_2" model="digest.tip">
            <field name="name">Tip: A calculator in Odoo</field>
            <field name="sequence">2300</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: A calculator in Odoo</p>
    <p class="tip_content">When editing a number, you can use formulae by typing the `=` character. This is useful when computing a margin or a discount on a quotation, sale order or invoice.</p>
    <img src="/digest/static/src/img/calculator.gif" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_3" model="digest.tip">
            <field name="name">Tip: How to ping users in internal notes?</field>
            <field name="sequence">2400</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: How to ping users in internal notes?</p>
    <p class="tip_content">Type "@" to notify someone in a message, or "#" to link to a channel. Try to notify @OdooBot to test the feature.</p>
    <img src="/digest/static/src/img/notifications.gif" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_4" model="digest.tip">
            <field name="name">Tip: Knowledge is power</field>
            <field name="sequence">2500</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Knowledge is power</p>
    <p class="tip_content">When following documents, use the pencil icon to fine-tune the information you want to receive.
Follow a project / sales team to keep track of this project's tasks / this team's opportunities.</p>
    <img src="/digest/static/src/img/following.png" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record forcecreate="True" id="ir_cron_digest_scheduler_action" model="ir.cron">
        <field name="name">Digest Emails</field>
        <field name="model_id" ref="model_digest_digest"/>
        <field name="state">code</field>
        <field name="code">model._cron_send_digest_email()</field>
        <field name="user_id" ref="base.user_root"/>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="nextcall" eval="(DateTime.now() + timedelta(hours=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
</odoo>

```

## File: data\res_config_settings_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo noupdate="1">
    <record id="default_emails_digest" model="ir.config_parameter" forcecreate="0">
        <field name="key">digest.default_digest_emails</field>
        <field name="value">True</field>
    </record>
    <record id="default_digest" model="ir.config_parameter" forcecreate="0">
        <field name="key">digest.default_digest_id</field>
        <field name="value" ref="digest.digest_digest_default"/>
    </record>
</odoo>

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pytz

from datetime import datetime, date
from dateutil.relativedelta import relativedelta
from werkzeug.urls import url_join

from odoo import api, fields, models, tools, _
from odoo.addons.base.models.ir_mail_server import MailDeliveryException
from odoo.exceptions import AccessError
from odoo.tools.float_utils import float_round

_logger = logging.getLogger(__name__)


class Digest(models.Model):
    _name = 'digest.digest'
    _description = 'Digest'

    # Digest description
    name = fields.Char(string='Name', required=True, translate=True)
    user_ids = fields.Many2many('res.users', string='Recipients', domain="[('share', '=', False)]")
    periodicity = fields.Selection([('daily', 'Daily'),
                                    ('weekly', 'Weekly'),
                                    ('monthly', 'Monthly'),
                                    ('quarterly', 'Quarterly')],
                                   string='Periodicity', default='daily', required=True)
    next_run_date = fields.Date(string='Next Send Date')
    currency_id = fields.Many2one(related="company_id.currency_id", string='Currency', readonly=False)
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company.id)
    available_fields = fields.Char(compute='_compute_available_fields')
    is_subscribed = fields.Boolean('Is user subscribed', compute='_compute_is_subscribed')
    state = fields.Selection([('activated', 'Activated'), ('deactivated', 'Deactivated')], string='Status', readonly=True, default='activated')
    # First base-related KPIs
    kpi_res_users_connected = fields.Boolean('Connected Users')
    kpi_res_users_connected_value = fields.Integer(compute='_compute_kpi_res_users_connected_value')
    kpi_mail_message_total = fields.Boolean('Messages')
    kpi_mail_message_total_value = fields.Integer(compute='_compute_kpi_mail_message_total_value')

    def _compute_is_subscribed(self):
        for digest in self:
            digest.is_subscribed = self.env.user in digest.user_ids

    def _compute_available_fields(self):
        for digest in self:
            kpis_values_fields = []
            for field_name, field in digest._fields.items():
                if field.type == 'boolean' and field_name.startswith(('kpi_', 'x_kpi_', 'x_studio_kpi_')) and digest[field_name]:
                    kpis_values_fields += [field_name + '_value']
            digest.available_fields = ', '.join(kpis_values_fields)

    def _get_kpi_compute_parameters(self):
        return fields.Date.to_string(self._context.get('start_date')), fields.Date.to_string(self._context.get('end_date')), self.env.company

    def _compute_kpi_res_users_connected_value(self):
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            user_connected = self.env['res.users'].search_count([('company_id', '=', company.id), ('login_date', '>=', start), ('login_date', '<', end)])
            record.kpi_res_users_connected_value = user_connected

    def _compute_kpi_mail_message_total_value(self):
        discussion_subtype_id = self.env.ref('mail.mt_comment').id
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            total_messages = self.env['mail.message'].search_count([('create_date', '>=', start), ('create_date', '<', end), ('subtype_id', '=', discussion_subtype_id), ('message_type', 'in', ['comment', 'email'])])
            record.kpi_mail_message_total_value = total_messages

    @api.onchange('periodicity')
    def _onchange_periodicity(self):
        self.next_run_date = self._get_next_run_date()

    @api.model
    def create(self, vals):
        digest = super(Digest, self).create(vals)
        if not digest.next_run_date:
             digest.next_run_date = digest._get_next_run_date()
        return digest

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    def action_subscribe(self):
        if self.env.user.has_group('base.group_user') and self.env.user not in self.user_ids:
            self.sudo().user_ids |= self.env.user

    def action_unsubcribe(self):
        if self.env.user.has_group('base.group_user') and self.env.user in self.user_ids:
            self.sudo().user_ids -= self.env.user

    def action_activate(self):
        self.state = 'activated'

    def action_deactivate(self):
        self.state = 'deactivated'

    def action_set_periodicity(self, periodicity):
        self.periodicity = periodicity

    def action_send(self):
        to_slowdown = self._check_daily_logs()
        for digest in self:
            for user in digest.user_ids:
                digest.with_context(
                    digest_slowdown=digest in to_slowdown,
                    lang=user.lang
                )._action_send_to_user(user, tips_count=1)
            if digest in to_slowdown:
                digest.write({'periodicity': 'weekly'})
            digest.next_run_date = digest._get_next_run_date()

    def _action_send_to_user(self, user, tips_count=1, consum_tips=True):
        web_base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')

        rendered_body = self.env['mail.render.mixin']._render_template(
            'digest.digest_mail_main',
            'digest.digest',
            self.ids,
            engine='qweb',
            add_context={
                'title': self.name,
                'top_button_label': _('Connect'),
                'top_button_url': url_join(web_base_url, '/web/login'),
                'company': user.company_id,
                'user': user,
                'tips_count': tips_count,
                'formatted_date': datetime.today().strftime('%B %d, %Y'),
                'display_mobile_banner': True,
                'kpi_data': self.compute_kpis(user.company_id, user),
                'tips': self.compute_tips(user.company_id, user, tips_count=tips_count, consumed=consum_tips),
                'preferences': self.compute_preferences(user.company_id, user),
            },
            post_process=True
        )[self.id]
        full_mail = self.env['mail.render.mixin']._render_encapsulate(
            'digest.digest_mail_layout',
            rendered_body,
            add_context={
                'company': user.company_id,
                'user': user,
            },
        )
        # create a mail_mail based on values, without attachments
        mail_values = {
            'auto_delete': True,
            'email_from': (
                self.company_id.partner_id.email_formatted
                or self.env.user.email_formatted
                or self.env.ref('base.user_root').email_formatted
            ),
            'email_to': user.email_formatted,
            'body_html': full_mail,
            'state': 'outgoing',
            'subject': '%s: %s' % (user.company_id.name, self.name),
        }
        mail = self.env['mail.mail'].sudo().create(mail_values)
        return True

    @api.model
    def _cron_send_digest_email(self):
        digests = self.search([('next_run_date', '<=', fields.Date.today()), ('state', '=', 'activated')])
        for digest in digests:
            try:
                digest.action_send()
            except MailDeliveryException as e:
                _logger.warning('MailDeliveryException while sending digest %d. Digest is now scheduled for next cron update.', digest.id)

    # ------------------------------------------------------------
    # KPIS
    # ------------------------------------------------------------

    def compute_kpis(self, company, user):
        """ Compute KPIs to display in the digest template. It is expected to be
        a list of KPIs, each containing values for 3 columns display.

        :return list: result [{
            'kpi_name': 'kpi_mail_message',
            'kpi_fullname': 'Messages',  # translated
            'kpi_action': 'crm.crm_lead_action_pipeline',  # xml id of an action to execute
            'kpi_col1': {
                'value': '12.0',
                'margin': 32.36,
                'col_subtitle': 'Yesterday',  # translated
            },
            'kpi_col2': { ... },
            'kpi_col3':  { ... },
        }, { ... }] """
        self.ensure_one()
        digest_fields = self._get_kpi_fields()
        invalid_fields = []
        kpis = [
            dict(kpi_name=field_name,
                 kpi_fullname=self.env['ir.model.fields']._get(self._name, field_name).field_description,
                 kpi_action=False,
                 kpi_col1=dict(),
                 kpi_col2=dict(),
                 kpi_col3=dict(),
                 )
            for field_name in digest_fields
        ]
        kpis_actions = self._compute_kpis_actions(company, user)

        for col_index, (tf_name, tf) in enumerate(self._compute_timeframes(company)):
            digest = self.with_context(start_date=tf[0][0], end_date=tf[0][1]).with_user(user).with_company(company)
            previous_digest = self.with_context(start_date=tf[1][0], end_date=tf[1][1]).with_user(user).with_company(company)
            for index, field_name in enumerate(digest_fields):
                kpi_values = kpis[index]
                kpi_values['kpi_action'] = kpis_actions.get(field_name)
                try:
                    compute_value = digest[field_name + '_value']
                    # Context start and end date is different each time so invalidate to recompute.
                    digest.invalidate_cache([field_name + '_value'])
                    previous_value = previous_digest[field_name + '_value']
                    # Context start and end date is different each time so invalidate to recompute.
                    previous_digest.invalidate_cache([field_name + '_value'])
                except AccessError:  # no access rights -> just skip that digest details from that user's digest email
                    invalid_fields.append(field_name)
                    continue
                margin = self._get_margin_value(compute_value, previous_value)
                if self._fields['%s_value' % field_name].type == 'monetary':
                    converted_amount = tools.format_decimalized_amount(compute_value)
                    compute_value = self._format_currency_amount(converted_amount, company.currency_id)
                kpi_values['kpi_col%s' % (col_index + 1)].update({
                    'value': compute_value,
                    'margin': margin,
                    'col_subtitle': tf_name,
                })

        # filter failed KPIs
        return [kpi for kpi in kpis if kpi['kpi_name'] not in invalid_fields]

    def compute_tips(self, company, user, tips_count=1, consumed=True):
        tips = self.env['digest.tip'].search([
            ('user_ids', '!=', user.id),
            '|', ('group_id', 'in', user.groups_id.ids), ('group_id', '=', False)
        ], limit=tips_count)
        tip_descriptions = [
            self.env['mail.render.mixin']._render_template(tools.html_sanitize(tip.tip_description), 'digest.tip', tip.ids, post_process=True)[tip.id]
            for tip in tips
        ]
        if consumed:
            tips.user_ids += user
        return tip_descriptions

    def _compute_kpis_actions(self, company, user):
        """ Give an optional action to display in digest email linked to some KPIs.

        :return dict: key: kpi name (field name), value: an action that will be
          concatenated with /web#action={action}
        """
        return {}

    def compute_preferences(self, company, user):
        """ Give an optional text for preferences, like a shortcut for configuration.

        :return string: html to put in template
        """
        preferences = []
        if self._context.get('digest_slowdown'):
            preferences.append(_("We have noticed you did not connect these last few days so we've automatically switched your preference to weekly Digests."))
        elif self.periodicity == 'daily' and user.has_group('base.group_erp_manager'):
            preferences.append('<p>%s<br /><a href="/digest/%s/set_periodicity?periodicity=weekly" target="_blank" style="color:#875A7B; font-weight: bold;">%s</a></p>' % (
                _('Prefer a broader overview ?'),
                self.id,
                _('Switch to weekly Digests')
            ))
        if user.has_group('base.group_erp_manager'):
            preferences.append('<p>%s<br /><a href="/web#view_type=form&amp;model=%s&amp;id=%s" target="_blank" style="color:#875A7B; font-weight: bold;">%s</a></p>' % (
                _('Want to customize this email?'),
                self._name,
                self.id,
                _('Choose the metrics you care about')
            ))

        return preferences

    def _get_next_run_date(self):
        self.ensure_one()
        if self.periodicity == 'daily':
            delta = relativedelta(days=1)
        if self.periodicity == 'weekly':
            delta = relativedelta(weeks=1)
        elif self.periodicity == 'monthly':
            delta = relativedelta(months=1)
        elif self.periodicity == 'quarterly':
            delta = relativedelta(months=3)
        return date.today() + delta

    def _compute_timeframes(self, company):
        now = datetime.utcnow()
        tz_name = company.resource_calendar_id.tz
        if tz_name:
            now = pytz.timezone(tz_name).localize(now)
        start_date = now.date()
        return [
            (_('Yesterday'), (
                (start_date + relativedelta(days=-1), start_date),
                (start_date + relativedelta(days=-2), start_date + relativedelta(days=-1)))
            ), (_('Last 7 Days'), (
                (start_date + relativedelta(weeks=-1), start_date),
                (start_date + relativedelta(weeks=-2), start_date + relativedelta(weeks=-1)))
            ), (_('Last 30 Days'), (
                (start_date + relativedelta(months=-1), start_date),
                (start_date + relativedelta(months=-2), start_date + relativedelta(months=-1)))
            )
        ]

    # ------------------------------------------------------------
    # FORMATTING / TOOLS
    # ------------------------------------------------------------

    def _get_kpi_fields(self):
        return [field_name for field_name, field in self._fields.items()
                if field.type == 'boolean' and field_name.startswith(('kpi_', 'x_kpi_', 'x_studio_kpi_')) and self[field_name]
               ]

    def _get_margin_value(self, value, previous_value=0.0):
        margin = 0.0
        if (value != previous_value) and (value != 0.0 and previous_value != 0.0):
            margin = float_round((float(value-previous_value) / previous_value or 1) * 100, precision_digits=2)
        return margin

    def _check_daily_logs(self):
        three_days_ago = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0) - relativedelta(days=3)
        to_slowdown = self.env['digest.digest']
        for digest in self.filtered(lambda digest: digest.periodicity == 'daily'):
            users_logs = self.env['res.users.log'].sudo().search_count([
                ('create_uid', 'in', digest.user_ids.ids),
                ('create_date', '>=', three_days_ago)
            ])
            if not users_logs:
                to_slowdown += digest
        return to_slowdown

    def _format_currency_amount(self, amount, currency_id):
        pre = currency_id.position == 'before'
        symbol = u'{symbol}'.format(symbol=currency_id.symbol or '')
        return u'{pre}{0}{post}'.format(amount, pre=symbol if pre else '', post=symbol if not pre else '')

```

## File: models\digest_tip.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models
from odoo.tools.translate import html_translate


class DigestTip(models.Model):
    _name = 'digest.tip'
    _description = 'Digest Tips'
    _order = 'sequence'

    sequence = fields.Integer(
        'Sequence', default=1,
        help='Used to display digest tip in email template base on order')
    name = fields.Char('Name', translate=True)
    user_ids = fields.Many2many(
        'res.users', string='Recipients',
        help='Users having already received this tip')
    tip_description = fields.Html('Tip description', translate=html_translate)
    group_id = fields.Many2one(
        'res.groups', string='Authorized Group',
        default=lambda self: self.env.ref('base.group_user'))

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    digest_emails = fields.Boolean(string="Digest Emails", config_parameter='digest.default_digest_emails')
    digest_id = fields.Many2one('digest.digest', string='Digest Email', config_parameter='digest.default_digest_id')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class ResUsers(models.Model):
    _inherit = "res.users"

    @api.model_create_multi
    def create(self, vals_list):
        """ Automatically subscribe employee users to default digest if activated """
        users = super(ResUsers, self).create(vals_list)
        default_digest_emails = self.env['ir.config_parameter'].sudo().get_param('digest.default_digest_emails')
        default_digest_id = self.env['ir.config_parameter'].sudo().get_param('digest.default_digest_id')
        if default_digest_emails and default_digest_id:
            digest = self.env['digest.digest'].sudo().browse(int(default_digest_id)).exists()
            digest.user_ids |= users.filtered_domain([('share', '=', False)])
        return users

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import digest
from . import digest_tip
from . import res_config_settings
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_digest_digest_system,digest.digest.administration,model_digest_digest,base.group_erp_manager,1,1,1,1
access_digest_digest_user,digest.digest.user,model_digest_digest,base.group_user,1,0,0,0
access_digest_tip_system,digest.tip.administration,model_digest_tip,base.group_erp_manager,1,1,1,1
access_digest_tip_user,digest.tip.user,model_digest_tip,base.group_user,1,0,0,0

```

## File: views\digest_templates.xml

```xml
<odoo>
    <template id="portal_digest_unsubscribed" name="Unsubscription">
        <t t-call="portal.portal_layout">
            <div class="container mt8">
                <div class="row">
                    <div class="col-lg-6 offset-lg-3">
                        <h3>Digest Subscriptions</h3>
                        <div class="alert alert-success text-center" role="status">
                            <p>You have been successfully unsubscribed from <strong t-field="digest.name"/></p>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>
</odoo>
```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="digest_digest_view_tree" model="ir.ui.view">
        <field name="name">digest.digest.view.tree</field>
        <field name="model">digest.digest</field>
        <field name="arch" type="xml">
            <tree string="KPI Digest">
                <field name="name"/>
                <field name="periodicity"/>
                <field name="next_run_date" groups="base.group_no_one"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form</field>
        <field name="model">digest.digest</field>
        <field name="arch" type="xml">
            <form string="KPI Digest">
                <field name="is_subscribed" invisible="1"/>
                <header>
                    <button type="object" name="action_subscribe" string="Subscribe"
                        class="oe_highlight"
                        attrs="{'invisible': ['|',('is_subscribed', '=', True), ('state','=','deactivated')]}"/>
                    <button type="object" name="action_unsubcribe" string="Unsubscribe me"
                        class="oe_highlight"
                        attrs="{'invisible': ['|',('is_subscribed', '=', False), ('state','=','deactivated')]}"/>
                    <button type="object" name="action_deactivate" string="Deactivate for everyone"
                        class="oe_highlight"
                        attrs="{'invisible': [('state','=','deactivated')]}" groups="base.group_system"/>
                    <button type="object" name="action_activate" string="Activate"
                        class="oe_highlight"
                        attrs="{'invisible': [('state','=','activated')]}" groups="base.group_system"/>
                    <button type="object" name="action_send" string="Send Now"
                        class="oe_highlight"
                        attrs="{'invisible': [('state','=','deactivated')]}" groups="base.group_system"/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="periodicity" widget="radio" options="{'horizontal': true}"/>
                            <field name="next_run_date" groups="base.group_system"/>
                            <field name="company_id" options="{'no_create': True}" invisible="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="kpis" string="KPIs">
                            <group name="kpis">
                                <group name="kpi_general" string="General" groups="base.group_system">
                                    <field name="kpi_res_users_connected"/>
                                    <field name="kpi_mail_message_total"/>
                                </group>
                                <group name="kpi_sales"/>
                            </group>
                        </page>
                        <page name="recipients" string="Recipients" groups="base.group_system">
                            <group>
                                <field name="user_ids" options="{'no_create': True}">
                                    <tree string="Recipients">
                                        <field name="name"/>
                                        <field name="email"/>
                                    </tree>
                                </field>
                            </group>
                        </page>
                        <page name="how_to" string="How to customize your digest?" groups="base.group_no_one">
                            <div class="alert alert-info" role="alert">
                                In order to build your customized digest, follow these steps:
                                <ol>
                                    <li>
                                        You may want to add new computed fields with Odoo Studio:
                                        <ul>
                                            <li>
                                                you must create 2 fields on the
                                                <code>digest</code>
                                                object:
                                            </li>
                                            <li>
                                                first create a boolean field called
                                                <code>kpi_myfield</code>
                                                and display it in the KPI's tab;
                                            </li>
                                            <li>
                                                then create a computed field called
                                                <code>kpi_myfield_value</code>
                                                that will compute your customized KPI.
                                            </li>
                                        </ul>
                                    </li>
                                    <li>Select your KPIs in the KPI's tab.</li>
                                    <li>
                                        Create or edit the mail template: you may get computed KPI's value using these fields:
                                        <code>
                                            <field name="available_fields" class="oe_inline" />
                                        </code>
                                    </li>
                                </ol>
                            </div>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record id="digest_digest_view_search" model="ir.ui.view">
        <field name="name">digest.digest.view.search</field>
        <field name="model">digest.digest</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="user_ids"/>
                <group expand="1" string="Group by">
                    <filter string="Periodicity" name="periodicity" context="{'group_by': 'periodicity'}"/>
                </group>
            </search>
        </field>
    </record>
    <record id="digest_digest_action" model="ir.actions.act_window">
        <field name="name">Digest Emails</field>
        <field name="res_model">digest.digest</field>
        <field name="search_view_id" ref="digest_digest_view_search"/>
    </record>

    <menuitem id="digest_menu"
        action="digest_digest_action"
        parent="base.menu_email"
        groups="base.group_erp_manager"
        sequence="80"/>

    <!-- DIGEST.TIP -->
    <record id="digest_tip_view_tree" model="ir.ui.view">
        <field name="name">digest.tip.view.tree</field>
        <field name="model">digest.tip</field>
        <field name="arch" type="xml">
            <tree string="KPI Digest Tips">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="group_id"/>
            </tree>
        </field>
    </record>
    <record id="digest_tip_view_form" model="ir.ui.view">
        <field name="name">digest.tip.view.form</field>
        <field name="model">digest.tip</field>
        <field name="arch" type="xml">
            <form string="KPI Digest Tip">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="tip_description"/>
                        <field name="group_id"/>
                        <field name="user_ids" widget="many2many_tags"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="digest_tip_view_search" model="ir.ui.view">
        <field name="name">digest.tip.view.search</field>
        <field name="model">digest.tip</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="tip_description"/>
                <field name="group_id"/>
            </search>
        </field>
    </record>
    <record id="digest_tip_action" model="ir.actions.act_window">
        <field name="name">Digest Tips</field>
        <field name="res_model">digest.tip</field>
        <field name="search_view_id" ref="digest_tip_view_search"/>
    </record>

    <menuitem id="digest_tip_menu"
        action="digest_tip_action"
        parent="base.menu_email"
        groups="base.group_erp_manager"
        sequence="81"/>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.digest</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='contacts_settings']" position="before">
                <div id="statistics" >
                    <h2>Statistics</h2>
                    <div class='row mt16 o_settings_container' id='statistics_div'>
                        <div class="col-12 col-lg-6 o_setting_box"
                            title="New users are automatically added as recipient of the following digest email."
                            name="digest_email_setting_container">
                            <div class="o_setting_left_pane">
                                <field name="digest_emails"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label string="Digest Email" for="digest_emails"/>
                                <div class="text-muted" id="msg_module_digest">
                                    Add new users as recipient of a periodic email with key metrics
                                </div>
                                <div class="content-group" attrs="{'invisible': [('digest_emails','=',False)]}">
                                    <div class="mt16">
                                        <label for="digest_id" class="o_light_label"/>
                                        <field name="digest_id" class="oe_inline"/>
                                    </div>
                                    <div class="mt8">
                                        <button type="action" name="%(digest.digest_digest_action)d" string="Configure Digest Emails" icon="fa-arrow-right" class="btn-link"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

