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
    'version': '1.0',
    'depends': [
        'mail',
        'portal',
        'resource',
    ],
    'data': [
        'security/ir.model.access.csv',
        'data/digest_template_data.xml',
        'data/digest_data.xml',
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

from odoo.http import Controller, request, route


class DigestController(Controller):

    @route('/digest/<int:digest_id>/unsubscribe', type='http', website=True, auth='user')
    def digest_unsubscribe(self, digest_id, **post):
        digest = request.env['digest.digest'].sudo().browse(digest_id)
        digest.action_unsubcribe()
        return request.render('digest.portal_digest_unsubscribed', {
            'digest': digest,
        })

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest_digest_default" model="digest.digest">
            <field name="name">Weekly Stats in Odoo</field>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="next_run_date" eval="(DateTime.now() + timedelta(days=7)).strftime('%Y-%m-%d')"/>
            <field name="kpi_res_users_connected">True</field>
            <field name="kpi_mail_message_total">True</field>
        </record>
    </data>

    <data>
        <record id="digest_tip_mail_0" model="digest.tip">
            <field name="sequence">1</field>
            <field name="tip_description" type="html">
<div>
    % set users = object.env['res.users'].search([('share', '=' , False)], limit=10, order='id desc')
    % set channel_id = object.env.ref('mail.channel_all_employees').id
    <strong style="font-size: 16px;">Did you know...?</strong>
    <div style="font-size: 14px;">You can ping colleagues by tagging them in your messages using "@". They will be instantly notified.<br/>
        <div>
            <center>
                <img src="/digest/static/src/img/notification.png" width="70%" height="100%"/><br/>
            </center>
            ${', '.join(users.mapped('name'))} signed up. Say hello in the <a href="/web#action=mail.action_discuss&amp;active_id=${channel_id}" style="color: #006d6b;">company's discussion channel.</a>
        </div>
    </div>
</div>
            </field>
        </record>
        <record id="digest_tip_mail_1" model="digest.tip">
            <field name="sequence">7</field>
            <field name="tip_description" type="html">
<div>
    <strong style="font-size: 16px;">Get things done with activities</strong>
    <div style="font-size: 14px;">You don't have any activity scheduled. Use activities on any business document to schedule meetings, calls and todos.</div>
    <center>
        <img src="/digest/static/src/img/activity.png" width="70%" height="100%"/><br/>
    </center>
</div>
            </field>
        </record>

        <record id="digest_tip_mail_2" model="digest.tip">
            <field name="group_id" ref='base.group_system'/>
            <field name="sequence">0</field>
            <field name="tip_description" type="html">
<div style="font-size: 14px;">
    To reach your full potential with Odoo, have a look at our <a href="https://www.odoo.com/page/tour">apps videos</a> and discover new tools to work better and faster.
</div>
            </field>
        </record>

    </data>
</odoo>

```

## File: data\digest_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="digest_mail_template" model="mail.template">
        <field name="name">Digest: Default main template</field>
        <field name="model_id" ref="digest.model_digest_digest"/>
        <field name="subject">${'%s: %s' % (ctx.get('user', user).company_id.name, object.name)}</field>
        <field name="email_from">${('%s' % (object.company_id.partner_id.email_formatted or user.email_formatted))|safe}</field>
        <field name="body_html" type="html">
<table style="width: 100%; border-spacing: 0; font-family: Helvetica,Arial,Verdana,sans-serif;">
    <tr>
        <td align="center" valign="top" style="border-collapse: collapse; padding: 0">
            % set user = ctx.get('user', user)
            % set company = user.company_id
            % set data = object.compute_kpis(company, user)
            % set tips = object.compute_tips(company, user)
            % set kpi_actions = object.compute_kpis_actions(company, user)
            % set kpis = data.yesterday.keys()
            <table style="width: 100%; max-width: 600px; border-spacing: 0; border: 1px solid #e7e7e7; border-bottom: none; color: #6e7172; line-height: 23px; text-align: left;">
                <tr>
                    <td style="border-collapse: collapse; padding: 10px 20px; text-align: left;">
                        <strong style="color: #000000; font-size: 22px; line-height: 32px;">${object.name}</strong>
                        <div style="color: #000000; font-size: 15px;">${datetime.date.today().strftime('%B %d, %Y')}</div>
                    </td>
                    <td style="text-align: right; padding: 10px 20px">
                        <img style="padding: 0px; margin: 0px; height: auto; width: 80px;" src="/logo.png?company=${company.id}"/>
                        <div style="padding-top: 5px;">
                            <a href="/web/login" style="background-color: #875A7B; margin-left: 5px; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">Connect</a>
                        </div>
                    </td>
                </tr>
                <tr><td colspan="2" style="text-align: center;">
                    <hr width="95%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 16px 0px 16px 14px;"/>
                </td></tr>
            </table>
            % for kpi in kpis:
                <table style="border-spacing: 0; width: 100%; max-width: 600px;">
                    <tr>
                        <td style="border-collapse: collapse; background-color: #ffffff; border-left: 1px solid #e7e7e7; border-right: 1px solid #e7e7e7; line-height: 21px; padding: 0 20px 10px 20px; text-align: left;"><br/>
                            <span style="color: #3d466e; font-size: 18px; font-weight: 500; line-height: 23px;">${object.fields_get()[kpi]['string']}</span>
                            %if kpi in kpi_actions:
                                <span style="float: right;">
                                    <a href="/web#action=${kpi_actions[kpi]}">View more</a>
                                </span>
                            %endif
                        </td>
                    </tr>
                    <tr>
                        <td style="border-collapse: collapse; margin: 0; padding:0;">
                            <table style="width: 100%; border-spacing: 0; background-color: #f9f9f9; border: 1px solid #e7e7e7; border-top: none;">
                                <tr>
                                    <td style="border-collapse: collapse; margin: 0; padding: 0; display: block; border-top: 2px solid #56b3b5;">
                                        <table style="width: 100%; max-width: 199px; border-spacing: 0;">
                                            <tr>
                                                <td style="border-collapse: collapse; padding: 20px; text-align: center;">
                                                    <span style="color: #56b3b5; font-size: 35px; font-weight: bold; text-decoration: none; line-height: 36px;">${data['yesterday'][kpi][kpi]}</span><br/>
                                                    <span style="color: #888888; display: inline-block; font-size: 12px; line-height: 18px; text-transform: uppercase;">Yesterday</span>
                                                    % if data['yesterday'][kpi]['margin'] != 0.0:
                                                        <span style="color: #888888; display: block; font-size: 12px; line-height: 18px; text-transform: uppercase;">
                                                            % if data['yesterday'][kpi]['margin'] is greaterthan(0.0):
                                                                <span style="color: #0bbc22;">▲</span>${"%.2f" % data['yesterday'][kpi]['margin']} %
                                                            % endif
                                                            % if data['yesterday'][kpi]['margin'] is lessthan(0.0):
                                                                <span style="color: #ff0000;">▼</span>${"%.2f" % data['yesterday'][kpi]['margin']} %
                                                            % endif
                                                        </span>
                                                    % endif
                                                </td>
                                            </tr>
                                        </table>
                                    </td>
                                    <td style="border-collapse: collapse; margin: 0; padding: 0; border-top: 2px solid #9a5b82;">
                                        <table style="width: 100%; max-width: 199px; border-spacing: 0; margin: 0; padding: 0;">
                                            <tr>
                                                <td style="border-collapse: collapse; padding: 20px; text-align: center;">
                                                    <span style="color: #9a5b82; font-size: 35px; font-weight: bold; text-decoration: none; line-height: 36px;">${data['lastweek'][kpi][kpi]}</span><br/>
                                                    <span style="color: #888888; display: inline-block; font-size: 12px; line-height: 18px; text-transform: uppercase;">Last 7 Days</span>
                                                    % if data['lastweek'][kpi]['margin'] != 0.0:
                                                        <span style="color: #888888; display: block; font-size: 12px; line-height: 18px; text-transform: uppercase;">
                                                            % if data['lastweek'][kpi]['margin'] is greaterthan(0.0):
                                                                <span style="color: #0bbc22;">▲</span>${"%.2f" % data['lastweek'][kpi]['margin']} %
                                                            % endif
                                                            % if data['lastweek'][kpi]['margin'] is lessthan(0.0):
                                                                <span style="color: #ff0000;">▼</span>${"%.2f" % data['lastweek'][kpi]['margin']} %
                                                            %endif
                                                        </span>
                                                    %endif
                                                </td>
                                            </tr>
                                        </table>
                                    </td>
                                    <td style="border-collapse: collapse; margin: 0; padding: 15px; border-top: 2px solid #56b3b5;">
                                        <table style="width: 100%; max-width: 199px; border-spacing: 0; margin: 0; padding: 0;">
                                            <tr>
                                                <td style="border-collapse: collapse; margin: 0; text-align: center;">
                                                    <span style="color: #56b3b5; font-size: 35px; font-weight: bold; text-decoration: none; line-height: 36px">${data['lastmonth'][kpi][kpi]}</span><br/>
                                                    <span style="color: #888888; display: inline-block; font-size: 12px; line-height: 18px; text-transform: uppercase;">Last 30 Days</span>
                                                    % if data['lastmonth'][kpi]['margin'] != 0.0:
                                                        <span style="color: #888888; display: block; font-size: 12px; line-height: 18px; text-transform: uppercase;">
                                                             % if data['lastmonth'][kpi]['margin'] is greaterthan(0.0):
                                                                <span style="color: #0bbc22;">▲</span>${"%.2f" % data['lastmonth'][kpi]['margin']} %
                                                            % endif
                                                            % if data['lastmonth'][kpi]['margin'] is lessthan(0.0):
                                                                <span style="color: #ff0000;">▼</span>${"%.2f" % data['lastmonth'][kpi]['margin']} %
                                                            %endif
                                                        </span>
                                                    %endif
                                                </td>
                                            </tr>
                                        </table>
                                    </td>
                                </tr>
                            </table>
                        </td>
                    </tr>
                </table>
            % endfor
            % if tips:
                <table style="width: 100%; max-width: 600px; margin-top: 5px; border: 1px solid #e7e7e7;">
                    <tr>
                        <td style="border-collapse: collapse; background-color: #ffffff; line-height: 21px; padding: 0px 20px 20px;"><br/>
                            <div style="color: #3d466e; line-height: 23px;">${tips | safe}</div>
                        </td>
                    </tr>
                </table>
            % endif
            <table style="width: 100%; max-width: 600px; margin-top: 5px; border: 1px solid #e7e7e7;">
                <tr>
                    <td style="border-collapse: collapse; background-color: #ffffff; line-height: 21px; padding: 0 20px 10px 20px; text-align: center;"><br/>
                        <div style="color: #3d466e; font-size: 16px; font-weight: 600; line-height: 23px;">Run your business from anywhere with Odoo Mobile.</div>
                    </td>
                </tr>
                <tr>
                    <td style="padding-bottom:20px;">
                        <div style="text-align: center;"><a href="https://play.google.com/store/apps/details?id=com.odoo.mobile" target="_blank"><img src="/digest/static/src/img/google_play.png" style="display: inline-block; height: 30px; margin-left: auto; margin-right: 12px;"/></a><a href="https://itunes.apple.com/us/app/odoo/id1272543640" target="_blank"><img src="/digest/static/src/img/app_store.png" style="display: inline-block; height: 30px; margin-left: 12px; margin-right: auto;"/></a>
                        </div>
                    </td>
                </tr>
            </table>
            <table style="margin-top: 5px; border: 1px solid #e7e7e7; font-size: 15px; width: 100%; max-width: 600px;">
                <tr>
                    <td style="border-collapse: collapse; margin: 0; padding: 10px 20px;">
                        % if user.has_group('base.group_system'):
                            <div style="margin-top: 20px;"> 
                                Want to customize this email?
                                <a href="/web#view_type=form&amp;model=digest.digest&amp;id=${object.id}" target="_blank" style="color: #875A7B;">Choose the metrics you care about</a>
                            </div>
                            <br />
                        % endif
                        <p style="font-size: 11px; margin-top: 10px;">
                            <strong>
                                Sent by
                                <a href="https://www.odoo.com" style="text-decoration: none; color: #875A7B;">Odoo</a> - <a href="/web#view_type=form&amp;model=digest.digest&amp;id=${object.id}" target="_blank" style="color: #888888;">Unsubscribe</a>
                            </strong>
                        </p>
                    </td>
                </tr>
            </table>
        </td>
    </tr>
</table></field>
        <field name="lang">${user.lang}</field>
        <field name="auto_delete" eval="True"/>
        <field name="user_signature" eval="False"/>
    </record>
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
import math
import pytz

from datetime import datetime, date
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, tools
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
    periodicity = fields.Selection([('weekly', 'Weekly'),
                                    ('monthly', 'Monthly'),
                                    ('quarterly', 'Quarterly')],
                                   string='Periodicity', default='weekly', required=True)
    next_run_date = fields.Date(string='Next Send Date')
    template_id = fields.Many2one('mail.template', string='Email Template',
                                  domain="[('model','=','digest.digest')]",
                                  default=lambda self: self.env.ref('digest.digest_mail_template'),
                                  required=True)
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
        return fields.Date.to_string(self._context.get('start_date')), fields.Date.to_string(self._context.get('end_date')), self._context.get('company')

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
        vals['next_run_date'] = date.today() + relativedelta(days=3)
        return super(Digest, self).create(vals)

    def action_subscribe(self):
        if self.env.user not in self.user_ids:
            self.sudo().user_ids |= self.env.user

    def action_unsubcribe(self):
        if self.env.user in self.user_ids:
            self.sudo().user_ids -= self.env.user

    def action_activate(self):
        self.state = 'activated'

    def action_deactivate(self):
        self.state = 'deactivated'

    def action_send(self):
        for digest in self:
            for user in digest.user_ids:
                subject = '%s: %s' % (user.company_id.name, digest.name)
                digest.template_id.with_context(user=user).send_mail(digest.id, force_send=True, raise_exception=True, email_values={'email_to': user.email, 'subject': subject})
            digest.next_run_date = digest._get_next_run_date()

    def compute_kpis(self, company, user):
        self.ensure_one()
        res = {}
        for tf_name, tf in self._compute_timeframes(company).items():
            digest = self.with_context(start_date=tf[0][0], end_date=tf[0][1], company=company).with_user(user)
            previous_digest = self.with_context(start_date=tf[1][0], end_date=tf[1][1], company=company).with_user(user)
            kpis = {}
            for field_name, field in self._fields.items():
                if field.type == 'boolean' and field_name.startswith(('kpi_', 'x_kpi_', 'x_studio_kpi_')) and self[field_name]:

                    try:
                        compute_value = digest[field_name + '_value']
                        # Context start and end date is different each time so invalidate to recompute.
                        digest.invalidate_cache([field_name + '_value'])
                        previous_value = previous_digest[field_name + '_value']
                        # Context start and end date is different each time so invalidate to recompute.
                        previous_digest.invalidate_cache([field_name + '_value'])
                    except AccessError:  # no access rights -> just skip that digest details from that user's digest email
                        continue
                    margin = self._get_margin_value(compute_value, previous_value)
                    if self._fields[field_name+'_value'].type == 'monetary':
                        converted_amount = self._format_human_readable_amount(compute_value)
                        kpis.update({field_name: {field_name: self._format_currency_amount(converted_amount, company.currency_id), 'margin': margin}})
                    else:
                        kpis.update({field_name: {field_name: compute_value, 'margin': margin}})

                res.update({tf_name: kpis})
        return res

    def compute_tips(self, company, user):
        tip = self.env['digest.tip'].search([('user_ids', '!=', user.id), '|', ('group_id', 'in', user.groups_id.ids), ('group_id', '=', False)], limit=1)
        if not tip:
            return False
        tip.user_ids += user
        body = tools.html_sanitize(tip.tip_description)
        tip_description = self.env['mail.template']._render_template(body, 'digest.tip', self.id)
        return tip_description

    def compute_kpis_actions(self, company, user):
        """ Give an optional action to display in digest email linked to some KPIs.

        :return dict: key: kpi name (field name), value: an action that will be
          concatenated with /web#action={action}
        """
        return {}

    def _get_next_run_date(self):
        self.ensure_one()
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
        return {
            'yesterday': (
                (start_date + relativedelta(days=-1), start_date),
                (start_date + relativedelta(days=-2), start_date + relativedelta(days=-1))),
            'lastweek': (
                (start_date + relativedelta(weeks=-1), start_date),
                (start_date + relativedelta(weeks=-2), start_date + relativedelta(weeks=-1))),
            'lastmonth': (
                (start_date + relativedelta(months=-1), start_date),
                (start_date + relativedelta(months=-2), start_date + relativedelta(months=-1))),
        }

    def _get_margin_value(self, value, previous_value=0.0):
        margin = 0.0
        if (value != previous_value) and (value != 0.0 and previous_value != 0.0):
            margin = float_round((float(value-previous_value) / previous_value or 1) * 100, precision_digits=2)
        return margin

    def _format_currency_amount(self, amount, currency_id):
        pre = currency_id.position == 'before'
        symbol = u'{symbol}'.format(symbol=currency_id.symbol or '')
        return u'{pre}{0}{post}'.format(amount, pre=symbol if pre else '', post=symbol if not pre else '')

    def _format_human_readable_amount(self, amount, suffix=''):
        for unit in ['', 'K', 'M', 'G']:
            if abs(amount) < 1000.0:
                return "%3.2f%s%s" % (amount, unit, suffix)
            amount /= 1000.0
        return "%.2f%s%s" % (amount, 'T', suffix)

    @api.model
    def _cron_send_digest_email(self):
        digests = self.search([('next_run_date', '=', fields.Date.today()), ('state', '=', 'activated')])
        for digest in digests:
            try:
                digest.action_send()
            except MailDeliveryException as e:
                _logger.warning('MailDeliveryException while sending digest %d. Digest is now scheduled for next cron update.')

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

    @api.model
    def create(self, vals):
        """ Automatically subscribe employee users to default digest if activated """
        user = super(ResUsers, self).create(vals)
        default_digest_emails = self.env['ir.config_parameter'].sudo().get_param('digest.default_digest_emails')
        default_digest_id = self.env['ir.config_parameter'].sudo().get_param('digest.default_digest_id')
        if user.has_group('base.group_user') and default_digest_emails and default_digest_id:
            digest = self.env['digest.digest'].sudo().browse(int(default_digest_id)).exists()
            if digest:
                digest.user_ids |= user
        return user

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
access_digest_tip_user,digest.tip.user,model_digest_tip,base.group_user,0,0,0,0

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
                            <field name="template_id" groups="base.group_no_one"/>
                            <field name="next_run_date" groups="base.group_system"/>
                            <field name="company_id" options="{'no_create': True}" invisible="1"/>
                            <!-- <field name="digest_history_ids" invisible="1"/> -->
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
        sequence="93"/>
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
            <xpath expr="//div[@id='statistics_div']" position="inside">
                <div class="col-12 col-lg-6 o_setting_box" title="New users are automatically added as recipient of the following digest email.">
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
            </xpath>
        </field>
    </record>
</odoo>

```

