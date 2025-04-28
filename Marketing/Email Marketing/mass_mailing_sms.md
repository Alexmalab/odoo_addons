# Odoo Module: mass_mailing_sms

Category: Marketing/Email Marketing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'SMS Marketing',
    'summary': 'Design, send and track SMS',
    'description': '',
    'version': '1.0',
    'category': 'Marketing/Email Marketing',
    'depends': [
        'portal',
        'mass_mailing',
        'sms',
    ],
    'data': [
        'data/utm_data.xml',
        'security/ir.model.access.csv',
        'views/mailing_sms_menus.xml',
        'views/mailing_list_views.xml',
        'views/mailing_contact_views.xml',
        'views/mailing_trace_views.xml',
        'views/mailing_mailing_views.xml',
        'views/link_tracker_views.xml',
        'views/mass_mailing_sms_templates_portal.xml',
        'views/utm_campaign_views.xml',
        'report/mailing_trace_report_views.xml',
        'wizard/sms_composer_views.xml',
        'wizard/mailing_sms_test_views.xml',
    ],
    'demo': [
        'data/utm_demo.xml',
        'data/mailing_list_demo.xml',
        'data/mailing_demo.xml',
    ],
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug

from odoo import http, _
from odoo.addons.phone_validation.tools import phone_validation
from odoo.http import request


class MailingSMSController(http.Controller):

    def _check_trace(self, mailing_id, trace_code):
        try:
            mailing = request.env['mailing.mailing'].sudo().search([('id', '=', mailing_id)])
        except:
            mailing = False
        if not mailing:
            return {'error': 'mailing_error'}
        trace = request.env['mailing.trace'].sudo().search([
            ('trace_type', '=', 'sms'),
            ('sms_code', '=', trace_code),
            ('mass_mailing_id', '=', mailing.id)
        ])
        if not trace:
            return {'error': 'trace_error'}
        return {'trace': trace}

    @http.route(['/sms/<int:mailing_id>/<string:trace_code>'], type='http', website=True, auth='public')
    def blacklist_page(self, mailing_id, trace_code, **post):
        check_res = self._check_trace(mailing_id, trace_code)
        if not check_res.get('trace'):
            return werkzeug.utils.redirect('/web')
        return request.render('mass_mailing_sms.blacklist_main', {
            'mailing_id': mailing_id,
            'trace_code': trace_code,
        })

    @http.route(['/sms/<int:mailing_id>/unsubscribe/<string:trace_code>'], type='http', website=True, auth='public')
    def blacklist_number(self, mailing_id, trace_code, **post):
        check_res = self._check_trace(mailing_id, trace_code)
        if not check_res.get('trace'):
            return werkzeug.utils.redirect('/web')
        country_code = request.session.get('geoip', False) and request.session.geoip.get('country_code', False) if request.session.get('geoip') else None
        # parse and validate number
        sms_number = post.get('sms_number', '').strip(' ')
        sanitize_res = phone_validation.phone_sanitize_numbers([sms_number], country_code, None)[sms_number]
        tocheck_number = sanitize_res['sanitized'] or sms_number

        trace = check_res['trace'].filtered(lambda r: r.sms_number == tocheck_number)[:1]
        mailing_list_ids = trace.mass_mailing_id.contact_list_ids

        # compute opt-out / blacklist information
        lists_optout = request.env['mailing.list'].sudo()
        lists_optin = request.env['mailing.list'].sudo()
        unsubscribe_error = False
        if tocheck_number and trace:
            if mailing_list_ids:
                subscriptions = request.env['mailing.contact.subscription'].sudo().search([
                    ('list_id', 'in', mailing_list_ids.ids),
                    ('contact_id.phone_sanitized', '=', tocheck_number),
                ])
                subscriptions.write({'opt_out': True})
                lists_optout = subscriptions.mapped('list_id')
            else:
                blacklist_rec = request.env['phone.blacklist'].sudo().add(tocheck_number)
                blacklist_rec._message_log(
                    body=_('Blacklist through SMS Marketing unsubscribe (mailing ID: %s - model: %s)') %
                          (trace.mass_mailing_id.id, trace.mass_mailing_id.mailing_model_id.display_name))
            lists_optin = request.env['mailing.contact.subscription'].sudo().search([
                ('contact_id.phone_sanitized', '=', tocheck_number),
                ('list_id', 'not in', mailing_list_ids.ids),
                ('opt_out', '=', False),
            ]).mapped('list_id')
        elif tocheck_number:
            unsubscribe_error = _('Number %s not found' % tocheck_number)
        else:
            unsubscribe_error = sanitize_res['msg']

        return request.render('mass_mailing_sms.blacklist_number', {
            'mailing_id': mailing_id,
            'trace_code': trace_code,
            'sms_number': sms_number,
            'lists_optin': lists_optin,
            'lists_optout': lists_optout,
            'unsubscribe_error': unsubscribe_error,
        })

    @http.route('/r/<string:code>/s/<int:sms_sms_id>', type='http', auth="public")
    def sms_short_link_redirect(self, code, sms_sms_id, **post):
        # don't assume geoip is set, it is part of the website module
        # which mass_mailing doesn't depend on
        country_code = request.session.get('geoip', False) and request.session.geoip.get('country_code', False)
        if sms_sms_id:
            trace_id = request.env['mailing.trace'].sudo().search([('sms_sms_id_int', '=', int(sms_sms_id))]).id
        else:
            trace_id = False

        request.env['link.tracker.click'].sudo().add_click(
            code,
            ip=request.httprequest.remote_addr,
            country_code=country_code,
            mailing_trace_id=trace_id
        )
        return werkzeug.utils.redirect(request.env['link.tracker'].get_url_from_code(code), 301)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\mailing_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">
    <record id="mailing_sms_0" model="mailing.mailing">
        <field name="name">XMas Promo</field>
        <field name="subject">XMas Promo</field>
        <field name="mailing_type">sms</field>
        <field name="state">done</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="sent_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="campaign_id" ref="utm_campaign_0"/>
        <field name="mailing_model_id" ref="base.model_res_partner"/>
        <field name="mailing_domain" eval="[('parent_id', '=', ref('base.res_partner_4'))]"/>
        <field name="body_plaintext">This week-end, incredible promotion for XMas ! See http://sms.example.com !</field>
    </record>
    <!-- Generate link tracker information from it -->
    <function model="mailing.mailing" name="convert_links" eval="[ref('mass_mailing_sms.mailing_sms_0')]"/>
    <!-- Simulate traces -->
    <record id="mailing_sms_0_trace_0" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000000</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_7"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_1" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000001</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_13"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_2" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000002</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_14"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_3" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000003</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_24"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_4" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000004</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_33"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="exception" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_5" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">123123</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_33"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="bounced" eval="(DateTime.today() - relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_6" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number"></field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_34"/>
        <field name="sent" eval="False"/>
        <field name="exception" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>

    <!-- Generate some clicks -->
    <function model="link.tracker.click" name="add_click">
        <value model="link.tracker.code"
            search="[('link_id.url', '=', 'http://sms.example.com')]"
            use="code"/>
        <value name="ip">100.01.02.03</value>
        <value name="country_code">BE</value>
        <value name="mailing_trace_id" eval="ref('mailing_sms_0_trace_0')"/>
    </function>
    <function model="link.tracker.click" name="add_click">
        <value model="link.tracker.code"
            search="[('link_id.url', '=', 'http://sms.example.com')]"
            use="code"/>
        <value name="ip">100.01.02.04</value>
        <value name="country_code">BE</value>
        <value name="mailing_trace_id" eval="ref('mailing_sms_0_trace_0')"/>
    </function>
    <function model="link.tracker.click" name="add_click">
        <value model="link.tracker.code"
            search="[('link_id.url', '=', 'http://sms.example.com')]"
            use="code"/>
        <value name="ip">100.01.02.05</value>
        <value name="country_code">BE</value>
        <value name="mailing_trace_id" eval="ref('mailing_sms_0_trace_1')"/>
    </function>
    <function model="link.tracker.click" name="add_click">
        <value model="link.tracker.code"
            search="[('link_id.url', '=', 'http://sms.example')]"
            use="code"/>
        <value name="ip">100.01.02.06</value>
        <value name="country_code">BE</value>
        <value name="mailing_trace_id" eval="ref('mailing_sms_0_trace_2')"/>
    </function>

    <record id="mailing_sms_1" model="mailing.mailing">
        <field name="name">Extra Promo</field>
        <field name="subject">Extra Promo</field>
        <field name="mailing_type">sms</field>
        <field name="state">done</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="sent_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="campaign_id" ref="utm_campaign_0"/>
        <field name="mailing_model_id" ref="mass_mailing.model_mailing_list"/>
        <field name="contact_list_ids" eval="[(5, 0), (4, ref('mass_mailing_sms.mailing_list_sms_0'))]"/>
        <field name="body_plaintext">Extra promotion for you !</field>
    </record>
    <!-- Simulate traces -->
    <record id="mailing_sms_1_trace_0" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32456001100</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_0"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_1_trace_1" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32456001111</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_1"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_1_trace_2" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32456001122</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_2"/>
        <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_1_trace_3" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32456001133</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_3"/>
        <field name="exception" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="failure_type">sms_credit</field>
    </record>
    <record id="mailing_sms_1_trace_4" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">dummy</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_4"/>
        <field name="exception" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="failure_type">sms_number_format</field>
    </record>

</data></odoo>

```

## File: data\mailing_list_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">
    <record id="mailing_list_sms_0" model="mailing.list">
        <field name="name">Interested in Tree Promotions</field>
    </record>

    <record id="mailing_contact_0_0" model="mailing.contact">
        <field name="name">Hubert Farnsworth</field>
        <field name="mobile">+32456001100</field>
        <field name="list_ids" eval="[(4, ref('mass_mailing_sms.mailing_list_sms_0'))]"/>
    </record>
    <record id="mailing_contact_0_1" model="mailing.contact">
        <field name="name">Philip Fry</field>
        <field name="mobile">+32456001111</field>
        <field name="list_ids" eval="[(4, ref('mass_mailing_sms.mailing_list_sms_0'))]"/>
    </record>
    <record id="mailing_contact_0_2" model="mailing.contact">
        <field name="name">Turanga Leela</field>
        <field name="mobile">+32456001122</field>
        <field name="list_ids" eval="[(4, ref('mass_mailing_sms.mailing_list_sms_0'))]"/>
    </record>
    <record id="mailing_contact_0_3" model="mailing.contact">
        <field name="name">John Zoidberg</field>
        <field name="mobile">+32456001133</field>
        <field name="list_ids" eval="[(4, ref('mass_mailing_sms.mailing_list_sms_0'))]"/>
    </record>
    <record id="mailing_contact_0_4" model="mailing.contact">
        <field name="name">Zapp Brannigan</field>
        <field name="mobile">dummy</field>
        <field name="list_ids" eval="[(4, ref('mass_mailing_sms.mailing_list_sms_0'))]"/>
    </record>
</data></odoo>

```

## File: data\utm_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

	<record model="utm.medium" id="utm_medium_sms">
        <field name="name">SMS</field>
    </record>

</data></odoo>

```

## File: data\utm_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">
    <record id="mailing_tag_0" model="utm.tag">
        <field name="name">Bioutifoul SMS</field>
        <field name="color" eval="2"/>
    </record>
    <record id="utm_campaign_0" model="utm.campaign">
        <field name="name">XMas Promo</field>
        <field name="stage_id" ref="utm.campaign_stage_1"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="tag_ids" eval="[(4, ref('mailing_tag_0')), (4, ref('utm.utm_tag_1'))]"/>
    </record>
</data></odoo>

```

## File: models\mailing_contact.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailingContact(models.Model):
    _name = 'mailing.contact'
    _inherit = ['mailing.contact', 'mail.thread.phone']

    mobile = fields.Char(string='Mobile')

    def _sms_get_number_fields(self):
        return ['mobile']

```

## File: models\mailing_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailingList(models.Model):
    _inherit = 'mailing.list'

    def _compute_contact_nbr(self):
        if self.env.context.get('mailing_sms'):
            self.env.cr.execute('''
select list_id, count(*)
from mailing_contact_list_rel r
left join mailing_contact c on (r.contact_id=c.id)
left join phone_blacklist bl on c.phone_sanitized = bl.number and bl.active
where
    list_id in %s
    AND COALESCE(r.opt_out,FALSE) = FALSE
    AND c.phone_sanitized IS NOT NULL
    AND bl.id IS NULL
group by list_id''', (tuple(self.ids), ))
            data = dict(self.env.cr.fetchall())
            for mailing_list in self:
                mailing_list.contact_nbr = data.get(mailing_list.id, 0)
            return
        return super(MailingList, self)._compute_contact_nbr()

    def action_view_contacts(self):
        if self.env.context.get('mailing_sms'):
            action = self.env.ref('mass_mailing_sms.mailing_contact_action_sms').read()[0]
            action['domain'] = [('list_ids', 'in', self.ids)]
            context = dict(self.env.context, search_default_filter_valid_sms_recipient=1, default_list_ids=self.ids)
            action['context'] = context
            return action
        return super(MailingList, self).action_view_contacts()

```

## File: models\mailing_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class Mailing(models.Model):
    _inherit = 'mailing.mailing'

    @api.model
    def default_get(self, fields):
        res = super(Mailing, self).default_get(fields)
        if fields is not None and 'keep_archives' in fields and res.get('mailing_type') == 'sms':
            res['keep_archives'] = True
        return res

    # mailing options
    mailing_type = fields.Selection(selection_add=[('sms', 'SMS')])
    # sms options
    body_plaintext = fields.Text('SMS Body')
    sms_template_id = fields.Many2one('sms.template', string='SMS Template', ondelete='set null')
    sms_has_insufficient_credit = fields.Boolean(
        'Insufficient IAP credits', compute='_compute_sms_has_insufficient_credit',
        help='UX Field to propose to buy IAP credits')
    sms_force_send = fields.Boolean(
        'Send Directly', help='Use at your own risks.')
    # opt_out_link
    sms_allow_unsubscribe = fields.Boolean('Include opt-out link', default=False)

    @api.onchange('mailing_type')
    def _onchange_mailing_type(self):
        if self.mailing_type == 'sms' and (not self.medium_id or self.medium_id == self.env.ref('utm.utm_medium_email')):
            self.medium_id = self.env.ref('mass_mailing_sms.utm_medium_sms').id
        elif self.mailing_type == 'mail' and (not self.medium_id or self.medium_id == self.env.ref('mass_mailing_sms.utm_medium_sms')):
            self.medium_id = self.env.ref('utm.utm_medium_email').id

    @api.onchange('sms_template_id', 'mailing_type')
    def _onchange_sms_template_id(self):
        if self.mailing_type == 'sms' and self.sms_template_id:
            self.body_plaintext = self.sms_template_id.body

    @api.depends('mailing_trace_ids.failure_type')
    def _compute_sms_has_insufficient_credit(self):
        mailing_ids = self.env['mailing.trace'].sudo().search([
            ('mass_mailing_id', 'in', self.ids),
            ('trace_type', '=', 'sms'),
            ('failure_type', '=', 'sms_credit')
        ]).mapped('mass_mailing_id')
        for mailing in self:
            mailing.sms_has_insufficient_credit = mailing in mailing_ids

    # --------------------------------------------------
    # CRUD
    # --------------------------------------------------

    @api.model
    def create(self, values):
        if values.get('mailing_type') == 'sms':
            if not values.get('medium_id'):
                values['medium_id'] = self.env.ref('mass_mailing_sms.utm_medium_sms').id
            if values.get('sms_template_id') and not values.get('body_plaintext'):
                values['body_plaintext'] = self.env['sms.template'].browse(values['sms_template_id']).body
        return super(Mailing, self).create(values)

    # --------------------------------------------------
    # BUSINESS / VIEWS ACTIONS
    # --------------------------------------------------

    def action_put_in_queue_sms(self):
        res = self.action_put_in_queue()
        if self.sms_force_send:
            self.action_send_mail()
        return res

    def action_send_now_sms(self):
        if not self.sms_force_send:
            self.write({'sms_force_send': True})
        return self.action_send_mail()

    def action_retry_failed(self):
        mass_sms = self.filtered(lambda m: m.mailing_type == 'sms')
        if mass_sms:
            mass_sms.action_retry_failed_sms()
        return super(Mailing, self - mass_sms).action_retry_failed()

    def action_retry_failed_sms(self):
        failed_sms = self.env['sms.sms'].sudo().search([
            ('mailing_id', 'in', self.ids),
            ('state', '=', 'error')
        ])
        failed_sms.mapped('mailing_trace_ids').unlink()
        failed_sms.unlink()
        self.write({'state': 'in_queue'})

    def action_test(self):
        if self.mailing_type == 'sms':
            ctx = dict(self.env.context, default_mailing_id=self.id)
            return {
                'name': _('Test SMS marketing'),
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'mailing.sms.test',
                'target': 'new',
                'context': ctx,
            }
        return super(Mailing, self).action_test()

    def _action_view_traces_filtered(self, view_filter):
        action = super(Mailing, self)._action_view_traces_filtered(view_filter)
        if self.mailing_type == 'sms':
            action['views'] = [(self.env.ref('mass_mailing_sms.mailing_trace_view_tree_sms').id, 'tree'),
                               (self.env.ref('mass_mailing_sms.mailing_trace_view_form_sms').id, 'form')]
        return action

    def action_buy_sms_credits(self):
        url = self.env['iap.account'].get_credits_url(service_name='sms')
        return {
            'type': 'ir.actions.act_url',
            'url': url,
        }

    # --------------------------------------------------
    # SMS SEND
    # --------------------------------------------------

    def _get_opt_out_list_sms(self):
        """Returns a set of emails opted-out in target model"""
        self.ensure_one()
        opt_out = []
        target = self.env[self.mailing_model_real]
        if self.mailing_model_real == 'mailing.contact':
            # if user is opt_out on One list but not on another
            # or if two user with same email address, one opted in and the other one opted out, send the mail anyway
            # TODO DBE Fixme : Optimise the following to get real opt_out and opt_in
            subscriptions = self.env['mailing.contact.subscription'].sudo().search(
                [('list_id', 'in', self.contact_list_ids.ids)])
            opt_out_contacts = subscriptions.filtered(lambda sub: sub.opt_out).mapped('contact_id')
            opt_in_contacts = subscriptions.filtered(lambda sub: not sub.opt_out).mapped('contact_id')
            opt_out = list(set(c.id for c in opt_out_contacts if c not in opt_in_contacts))

            _logger.info("Mass SMS %s targets %s: optout: %s contacts", self, target._name, len(opt_out))
        else:
            _logger.info("Mass SMS %s targets %s: no opt out list available", self, target._name)
        return opt_out

    def _get_seen_list_sms(self):
        """Returns a set of emails already targeted by current mailing/campaign (no duplicates)"""
        self.ensure_one()
        target = self.env[self.mailing_model_real]

        if issubclass(type(target), self.pool['mail.thread.phone']):
            phone_fields = ['phone_sanitized']
        elif issubclass(type(target), self.pool['mail.thread']):
            phone_fields = [
                fname for fname in target._sms_get_number_fields()
                if fname in target._fields and target._fields[fname].store
            ]
        else:
            phone_fields = []
            if 'mobile' in target._fields and target._fields['mobile'].store:
                phone_fields.append('mobile')
            if 'phone' in target._fields and target._fields['phone'].store:
                phone_fields.append('phone')
        if not phone_fields:
            raise UserError(_("Unsupported %s for mass SMS") % self.mailing_model_id.name)

        query = """
            SELECT %(select_query)s
              FROM mailing_trace trace
              JOIN %(target_table)s target ON (trace.res_id = target.id)
             WHERE (%(where_query)s)
             AND trace.mass_mailing_id = %%(mailing_id)s
             AND trace.model = %%(target_model)s
        """
        query = query % {
            'select_query': 'target.id, ' + ', '.join('target.%s' % fname for fname in phone_fields),
            'where_query': ' OR '.join('target.%s IS NOT NULL' % fname for fname in phone_fields),
            'target_table': target._table
        }
        params = {'mailing_id': self.id, 'target_model': self.mailing_model_real}
        self._cr.execute(query, params)
        query_res = self._cr.fetchall()
        seen_list = set(number for item in query_res for number in item[1:] if number)
        seen_ids = set(item[0] for item in query_res)
        _logger.info("Mass SMS %s targets %s: already reached %s SMS", self, target._name, len(seen_list))
        return list(seen_ids), list(seen_list)

    def _send_sms_get_composer_values(self, res_ids):
        return {
            # content
            'body': self.body_plaintext,
            'template_id': self.sms_template_id.id,
            'res_model': self.mailing_model_real,
            'res_ids': repr(res_ids),
            # options
            'composition_mode': 'mass',
            'mailing_id': self.id,
            'mass_keep_log': self.keep_archives,
            'mass_force_send': self.sms_force_send,
            'mass_sms_allow_unsubscribe': self.sms_allow_unsubscribe,
        }

    def action_send_mail(self, res_ids=None):
        mass_sms = self.filtered(lambda m: m.mailing_type == 'sms')
        if mass_sms:
            mass_sms.action_send_sms(res_ids=res_ids)
        return super(Mailing, self - mass_sms).action_send_mail(res_ids=res_ids)

    def action_send_sms(self, res_ids=None):
        for mailing in self:
            if not res_ids:
                res_ids = mailing._get_remaining_recipients()
            if not res_ids:
                raise UserError(_('There are no recipients selected.'))

            composer = self.env['sms.composer'].with_context(active_id=False).create(mailing._send_sms_get_composer_values(res_ids))
            composer._action_send_sms()
            mailing.write({'state': 'done', 'sent_date': fields.Datetime.now()})
        return True

```

## File: models\mailing_trace.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import random
import string

from odoo import api, fields, models
from odoo.osv import expression


class MailingTrace(models.Model):
    """ Improve statistics model to add SMS support. Main attributes of
    statistics model are used, only some specific data is required. """
    _inherit = 'mailing.trace'
    CODE_SIZE = 3

    trace_type = fields.Selection(selection_add=[('sms', 'SMS')])
    sms_sms_id = fields.Many2one('sms.sms', string='SMS', index=True, ondelete='set null')
    sms_sms_id_int = fields.Integer(
        string='SMS ID (tech)',
        help='ID of the related sms.sms. This field is an integer field because '
             'the related sms.sms can be deleted separately from its statistics. '
             'However the ID is needed for several action and controllers.',
        index=True,
    )
    sms_number = fields.Char('Number')
    sms_code = fields.Char('Code')
    failure_type = fields.Selection(selection_add=[
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_server', 'Server Error'),
        # mass mode specific codes
        ('sms_blacklist', 'Blacklisted'),
        ('sms_duplicate', 'Duplicate'),
    ])

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if 'sms_sms_id' in values:
                values['sms_sms_id_int'] = values['sms_sms_id']
            if values.get('trace_type') == 'sms' and not values.get('sms_code'):
                values['sms_code'] = self._get_random_code()
        return super(MailingTrace, self).create(values_list)

    def _get_random_code(self):
        """ Generate a random code for trace. Uniqueness is not really necessary
        as it serves as obfuscation when unsubscribing. A valid trio
        code / mailing_id / number will be requested. """
        return ''.join(random.choice(string.ascii_letters + string.digits) for dummy in range(self.CODE_SIZE))

    def _get_records_from_sms(self, sms_sms_ids=None, additional_domain=None):
        if not self.ids and sms_sms_ids:
            domain = [('sms_sms_id_int', 'in', sms_sms_ids)]
        else:
            domain = [('id', 'in', self.ids)]
        if additional_domain:
            domain = expression.AND([domain, additional_domain])
        return self.search(domain)

    def set_failed(self, failure_type):
        for trace in self:
            trace.write({'exception': fields.Datetime.now(), 'failure_type': failure_type})

    def set_sms_sent(self, sms_sms_ids=None):
        statistics = self._get_records_from_sms(sms_sms_ids, [('sent', '=', False)])
        statistics.write({'sent': fields.Datetime.now()})
        return statistics

    def set_sms_clicked(self, sms_sms_ids=None):
        statistics = self._get_records_from_sms(sms_sms_ids, [('clicked', '=', False)])
        statistics.write({'clicked': fields.Datetime.now()})
        return statistics

    def set_sms_ignored(self, sms_sms_ids=None):
        statistics = self._get_records_from_sms(sms_sms_ids, [('ignored', '=', False)])
        statistics.write({'ignored': fields.Datetime.now()})
        return statistics

    def set_sms_exception(self, sms_sms_ids=None):
        statistics = self._get_records_from_sms(sms_sms_ids, [('exception', '=', False)])
        statistics.write({'exception': fields.Datetime.now()})
        return statistics

```

## File: models\sms_sms.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import fields, models

TEXT_URL_REGEX = r'https?://[a-zA-Z0-9@:%._+~#=/-]+'


class SmsSms(models.Model):
    _inherit = ['sms.sms']

    mailing_id = fields.Many2one('mailing.mailing', string='Mass Mailing')
    mailing_trace_ids = fields.One2many('mailing.trace', 'sms_sms_id', string='Statistics')

    def _update_body_short_links(self):
        """ Override to tweak shortened URLs by adding statistics ids, allowing to
        find customer back once clicked. """
        shortened_schema = self.env['ir.config_parameter'].sudo().get_param('web.base.url') + '/r/'
        res = dict.fromkeys(self.ids, False)
        for sms in self:
            if not sms.mailing_id or not sms.body:
                res[sms.id] = sms.body
                continue

            body = sms.body
            for url in re.findall(TEXT_URL_REGEX, body):
                if url.startswith(shortened_schema):
                    body = body.replace(url, url + '/s/%s' % sms.id)
            res[sms.id] = body
        return res

    def _postprocess_iap_sent_sms(self, iap_results, failure_reason=None, delete_all=False):
        all_sms_ids = [item['res_id'] for item in iap_results]
        if any(sms.mailing_id for sms in self.env['sms.sms'].sudo().browse(all_sms_ids)):
            for state in self.IAP_TO_SMS_STATE.keys():
                sms_ids = [item['res_id'] for item in iap_results if item['state'] == state]
                traces = self.env['mailing.trace'].sudo().search([
                    ('sms_sms_id_int', 'in', sms_ids)
                ])
                if traces and state == 'success':
                    traces.write({'sent': fields.Datetime.now(), 'exception': False})
                elif traces:
                    traces.set_failed(failure_type=self.IAP_TO_SMS_STATE[state])
        return super(SmsSms, self)._postprocess_iap_sent_sms(iap_results, failure_reason=failure_reason, delete_all=delete_all)

```

## File: models\utm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    mailing_sms_ids = fields.One2many(
        'mailing.mailing', 'campaign_id',
        domain=[('mailing_type', '=', 'sms')],
        string='Mass SMS')
    mailing_sms_count = fields.Integer('Number of Mass SMS', compute="_compute_mailing_sms_count")

    @api.depends('mailing_sms_ids')
    def _compute_mailing_sms_count(self):
        for campaign in self:
            campaign.mailing_sms_count = len(campaign.mailing_sms_ids)

    def action_create_mass_sms(self):
        action = self.env.ref('mass_mailing.action_create_mass_mailings_from_campaign').read()[0]
        action['context'] = {
            'default_campaign_id': self.id,
            'default_mailing_type': 'sms',
            'search_default_assigned_to_me': 1,
            'search_default_campaign_id': self.id,
            'default_user_id': self.env.user.id,
        }
        return action

    def action_redirect_to_mailing_sms(self):
        action = self.env.ref('mass_mailing.action_view_mass_mailings_from_campaign').read()[0]
        action['context'] = {
            'default_campaign_id': self.id,
            'default_mailing_type': 'sms',
            'search_default_assigned_to_me': 1,
            'search_default_campaign_id': self.id,
            'default_user_id': self.env.user.id,
        }
        action['domain'] = [('mailing_type', '=', 'sms')]
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_contact
from . import mailing_list
from . import mailing_mailing
from . import mailing_trace
from . import sms_sms
from . import utm

```

## File: report\mailing_trace_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_trace_report_view_pivot" model="ir.ui.view">
        <field name="name">mailing.trace.report.view.pivot</field>
        <field name="model">mailing.trace.report</field>
        <field name="arch" type="xml">
            <pivot string="Mass Mailing Statistics" disable_linking="True">
                <field name="campaign" type="row"/>
                <field name="sent" type="measure"/>
                <field name="delivered" type="measure"/>
                <field name="opened" type="measure"/>
                <field name="bounced" type="measure"/>
                <field name="replied" type="measure"/>
                <field name="clicked" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="mailing_trace_report_view_graph" model="ir.ui.view">
        <field name="name">mailing.trace.report.view.graph</field>
        <field name="model">mailing.trace.report</field>
        <field name="arch" type="xml">
            <graph string="Mass Mailing Statistics">
                <field name="campaign"/>
                <field name="sent" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="mailing_trace_report_view_search" model="ir.ui.view">
        <field name="name">mailing.trace.report.view.search</field>
        <field name="model">mailing.trace.report</field>
        <field name="arch" type="xml">
            <search string="Mass Mailing Statistics">
                <group expand="0" string="Extended Filters...">
                    <field name="scheduled_date"/>
                </group>
                <group expand="1" string="Group By...">
                    <filter string="Mass Mailing Campaign" name="mass_mailing_campaign"
                        domain="[]" context="{'group_by':'campaign'}"/>
                    <filter string="State" domain="[]" name="state"
                        context="{'group_by':'state'}"/>
                    <filter string="Sent By" domain="[]" name="sent_by"
                        context="{'group_by':'email_from'}"/>
                    <separator/>
                    <filter string="Scheduled Period" name="scheduled_date"
                        domain="[]" context="{'group_by':'scheduled_date'}"/>
                </group>
            </search>
        </field>
    </record>

    <!-- Actions and Menuitems -->
    <record id="mailing_trace_report_action_sms" model="ir.actions.act_window">
        <field name="name">SMS Marketing Analysis</field>
        <field name="res_model">mailing.trace.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="domain">[('mailing_type', '=', 'sms')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                SMS Marketing Statistics allows you to check different mailing related information like number of sent SMS or bounced SMS.
                You can sort out your analysis by different groups to get accurate grained analysis.
            </p>
        </field>
    </record>

    <!-- SMS Marketing / Reporting -->
    <record id="mass_mailing_sms_menu_reporting" model="ir.ui.menu">
        <field name="action" ref="mailing_trace_report_action_sms"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><g><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/></g><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" fill-opacity=".151" d="M3 69.6c-2 0-4-1-4-4.1V29.7l18.6-18.9L44.3 12v16.727l4.585-3.484 7.039.82-3.47 4.437 3.062 5L44.3 46.516v6.83L31.2 69.6H3z"/><path fill="#000" fill-opacity=".3" fill-rule="nonzero" d="M44.3 14c0-1.1-.9-2-2-2H19c-1.1 0-2 .9-2 2v40.6c0 1.1.9 2 2 2h23.3c1.1 0 2-.9 2-2v-10c-.7.5-1.5 1-2.3 1.4v5.3c0 .2-.1.3-.3.3H19.6c-.2 0-.3-.1-.3-.3v-33c0-.2.1-.3.3-.3h22.1c.2 0 .3.1.3.3v2.8c.5 0 1.1-.1 1.6-.1h.7v-7zM30.7 52.4c.8 0 1.5.7 1.5 1.5s-.7 1.5-1.5 1.5-1.5-.7-1.5-1.5.7-1.5 1.5-1.5zm3.1-35.9h-6.1c-.3 0-.5-.2-.5-.5s.2-.5.5-.5h6.1c.3 0 .5.2.5.5s-.3.5-.5.5z"/><path fill="#FFF" fill-rule="nonzero" d="M43.6 42.4c-.5 0-1.1 0-1.6-.1v7.1c0 .2-.1.3-.3.3H19.6c-.2 0-.3-.1-.3-.3v-33c0-.2.1-.3.3-.3h22.1c.2 0 .3.1.3.3v2.9c.5 0 1.1-.1 1.6-.1h.7V12c0-1.1-.9-2-2-2H19c-1.1 0-2 .9-2 2v40.7c0 1.1.9 2 2 2h23.3c1.1 0 2-.9 2-2V42.4h-.7zm-16-28.7h6.1c.3 0 .5.2.5.5s-.2.5-.4.5h-6.1c-.3 0-.5-.2-.5-.5s.1-.5.4-.5zm3.1 39.9c-.8 0-1.5-.7-1.5-1.5s.7-1.5 1.5-1.5 1.5.7 1.5 1.5c0 .9-.7 1.5-1.5 1.5z"/><text fill="#000" fill-opacity=".3" font-family="IBMPlexMono-Bold, IBM Plex Mono" font-size="18" font-weight="bold" letter-spacing="-.215" transform="translate(17 10)"><tspan x="8.144" y="29">SMS</tspan></text><text fill="#FFF" font-family="IBMPlexMono-Bold, IBM Plex Mono" font-size="18" font-weight="bold" letter-spacing="-.215" transform="translate(17 10)"><tspan x="8.144" y="27">SMS</tspan></text></g></g></svg>
```

## File: views\link_tracker_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="link_tracker_menu"
        name="Link Tracker"
        parent="mass_mailing_sms_menu_configuration"
        sequence="2"
        action="link_tracker.link_tracker_action"/>
</odoo>

```

## File: views\mailing_contact_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_contact_view_search" model="ir.ui.view">
        <field name="name">mailing.contact.view.search.inherit.sms</field>
        <field name="model">mailing.contact</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_contact_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="mobile"/>
                <field name="phone_sanitized" invisible="1"/>
            </xpath>
            <xpath expr="//filter[@name='filter_not_optout']" position="after">
                <separator/>
                <filter string="Valid SMS Recipients"
                    name="filter_valid_sms_recipient"
                    domain="[('opt_out', '=', False), ('phone_blacklisted', '=', False), ('phone_sanitized', '!=', False)]"
                    invisible="not context.get('default_list_ids')"/>
                <separator/>
                <filter string="Exclude Blacklisted Phone"
                    name="filter_not_phone_bl"
                    domain="[('phone_blacklisted', '=', False)]"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_contact_view_tree_sms" model="ir.ui.view">
        <field name="name">mailing.contact.view.tree.sms</field>
        <field name="model">mailing.contact</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="Contacts">
                <field name="create_date"/>
                <field name="name"/>
                <field name="company_name"/>
                <field name="mobile" class="o_force_ltr"/>
                <field name="phone_sanitized" invisible="1"/>
                <field name="phone_blacklisted"/>
                <field name="opt_out" invisible="'default_list_ids' not in context"/>
            </tree>
        </field>
    </record>

    <record id="mailing_contact_view_form" model="ir.ui.view">
        <field name="name">mailing.contact.view.form.inherit.sms</field>
        <field name="model">mailing.contact</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_contact_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='email_details']" position="after">
                <label for="mobile" class="oe_inline"/>
                <div class="o_row o_row_readonly" name="phone_details">
                    <i class="fa fa-ban" style="color: red;" role="img" title="This number is blacklisted for SMS Marketing"
                        aria-label="Phone Blacklisted" attrs="{'invisible': [('phone_blacklisted', '=', False)]}" groups="base.group_user"></i>
                    <field name="mobile" widget="phone"/>
                    <field name="phone_sanitized" invisible="1"/>
                    <field name="phone_blacklisted" invisible="1"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="mailing_contact_view_kanban" model="ir.ui.view">
        <field name="name">mailing.contact.view.kanban.inherit.sms</field>
        <field name="model">mailing.contact</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_contact_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="after">
                <field name="mobile" class="o_force_ltr"/>
                <field name="phone_sanitized" invisible="1"/>
            </xpath>
            <xpath expr="//t[@t-esc='record.email.value']" position="after">
                <t t-esc="record.mobile.value"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_contact_action_sms" model="ir.actions.act_window">
        <field name="name">Contacts</field>
        <field name="res_model">mailing.contact</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'mailing_sms': True, 'search_default_filter_not_phone_bl': 1, }</field>
        <field name="view_id" ref="mailing_contact_view_tree_sms"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new contact
            </p>
        </field>
    </record>

    <!-- SMS Marketing / Contacts Lists / Contacts Lists -->
    <record id="mass_mailing_sms.mailing_contact_menu_sms" model="ir.ui.menu">
        <field name="action" ref="mailing_contact_action_sms"/>
    </record>

</odoo>

```

## File: views\mailing_list_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_list_action_sms" model="ir.actions.act_window">
        <field name="name">Contact Lists</field>
        <field name="res_model">mailing.list</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="context">{'mailing_sms': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new contacts list
          </p><p>
            You don't need to import your contacts lists, you can easily
            send SMS to any contact saved in other Odoo apps.
          </p>
        </field>
    </record>

    <!-- SMS Marketing / Contacts Lists / Contacts Lists -->
    <record id="mass_mailing_sms.mailing_list_menu_sms" model="ir.ui.menu">
        <field name="action" ref="mailing_list_action_sms"/>
    </record>
</odoo>

```

## File: views\mailing_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record model="ir.ui.view" id="mailing_mailing_view_search_sms">
        <field name="name">mailing.mailing.search</field>
        <field name="model">mailing.mailing</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_search"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='assigned_to_me']" position="attributes">
                <attribute name="string">My SMS Marketing</attribute>
            </xpath>
        </field>
    </record>

    <record id="mailing_mailing_view_form_sms" model="ir.ui.view">
        <field name="name">mailing.mailing.view.form.inherit.sms</field>
        <field name="model">mailing.mailing</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_form"/>
        <field name="arch" type="xml">
            <!-- Buttons / Actions -->
            <xpath expr="//button[@name='action_put_in_queue']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', ('state', 'in', ('in_queue', 'done')), ('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_put_in_queue']" position="after">
                <button name="action_put_in_queue_sms" type="object"
                    class="oe_highlight" string="Put in Queue"
                    attrs="{'invisible': ['|', ('mailing_type', '=', 'mail'), ('state', 'in', ('in_queue', 'done'))]}"
                    confirm="This will schedule an SMS marketing to all recipients. Do you still want to proceed ?"/>
                <button name="action_send_now_sms" type="object"
                    string="Send Now"
                    attrs="{'invisible': ['|', ('mailing_type', '=', 'mail'), ('state', 'in', ('done'))]}"
                    confirm="This will send SMS to all recipients now. Do you still want to proceed ?"/>
            </xpath>
            <!-- Headers / Warnings -->
            <xpath expr="//header" position="after">
                <field name="sms_has_insufficient_credit" invisible="1"/>
                <div class="alert alert-warning text-center" attrs="{'invisible': [('sms_has_insufficient_credit', '=', False)]}" role="alert">
                    <button class="btn-link py-0"
                            name="action_buy_sms_credits"
                            type="object">
                        <strong>
                            It appears you don't have enough IAP credits. Click here to buy credits.
                        </strong>
                    </button>
                </div>
            </xpath>
            <xpath expr="//span[@name='ignored_text']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='ignored_text']" position="after">
                <span name="ignored_text_sms" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">SMS Text Message have been ignored and will not be sent.</span>
            </xpath>
            <xpath expr="//span[@name='scheduled_text']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='scheduled_text']" position="after">
                <span name="scheduled_text_sms" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">SMS Text Message are in queue and will be sent soon.</span>
            </xpath>
            <xpath expr="//span[@name='failed_text']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='failed_text']" position="after">
                <span name="failed_text_sms" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">SMS Text Message could not be sent.</span>
            </xpath>
            <xpath expr="//span[@name='next_departure_text']" position='attributes'>
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='next_departure_text']" position='after'>
                <span name="next_departure_text" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">This SMS marketing is scheduled for </span>
            </xpath>
            <!-- Stat Buttons -->
            <xpath expr="//button[@id='button_view_sent']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', '&amp;', ('sent', '=', 0), ('state', 'in', ('draft', 'test')), ('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//button[@id='button_view_sent']" position="after">
                <button name="action_view_sent"
                    type="object"
                    context="{'search_default_filter_sent': True}"
                    icon="fa-comment-o" class="oe_stat_button"
                    attrs="{'invisible': ['|', '&amp;', ('sent', '=', 0), ('state', 'in', ('draft', 'test')), ('mailing_type', '!=', 'sms')]}" >
                    <field name="sent" string="Sent" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//button[@name='action_view_opened']" position="after">
                <button name="action_view_clicked"
                    type="object"
                    context="{'search_default_filter_clicked': True}" 
                    class="oe_stat_button"
                    attrs="{'invisible': ['|',('mailing_type', '!=', 'sms'),('state', 'in', ('draft','test'))]}">
                    <field name="clicks_ratio" string="Clicked" widget="percentpie"/>
                </button>
            </xpath>
            <xpath expr="//button[@name='action_view_opened']" position="attributes">
                <attribute name="attrs">{'invisible': ['|',('mailing_type', '!=', 'mail'),('state', 'in', ('draft','test'))]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_view_replied']" position="attributes">
                <attribute name="attrs">{'invisible': ['|',('mailing_type', '!=', 'mail'),('state', 'in', ('draft','test'))]}</attribute>
            </xpath>
            <!-- Form -->
            <xpath expr="//page[@name='mail_body']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//page[@name='mail_body']" position="after">
                <page string="SMS Content" name="sms_body" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">
                    <field name="body_plaintext" widget="sms_widget" attrs="{'required': [('mailing_type', '=', 'sms')]}"/>
                    <group>
                        <field name="sms_force_send" invisible="1"/>
                    </group>
                </page>
            </xpath>
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="sms_allow_unsubscribe" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}"/>
            </xpath>

            <xpath expr="//field[@name='contact_list_ids']" position="attributes">
                <attribute name="context">
                    {'mailing_sms' : context.get('mailing_sms')}
                </attribute>
            </xpath>
            <!-- Option page tweaks -->
            <xpath expr="//field[@name='email_from']" position="attributes">
                <attribute name="attrs">{
                    'invisible': [('mailing_type', '!=', 'mail')], 
                    'readonly': [('state', 'in', ('sending', 'done'))]}
                </attribute>
            </xpath>
            <xpath expr="//label[@for='reply_to']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//div[@name='reply_to_details']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//label[@for='attachment_ids']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//div[@name='attachment_ids_details']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='mail_server_id']" position="attributes">
                <attribute name="attrs">{
                    'invisible': [('mailing_type', '!=', 'mail')], 
                    'readonly': [('state', 'in', ('sending', 'done'))]}
                </attribute>
            </xpath>
        </field>
    </record>

    <record id="mailing_mailing_view_form_mixed" model="ir.ui.view">
        <!-- View allowign to display the mailing type and therefore choosing
        the way of mailing: not prioritized one, used in some specific cases
        like "contacting people" without predefining mail or sms -->
        <field name="name">mailing.mailing.view.form.mixed</field>
        <field name="model">mailing.mailing</field>
        <field name="mode">primary</field>
        <field name="priority">30</field>
        <field name="inherit_id" ref="mass_mailing_sms.mailing_mailing_view_form_sms"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='mailing_type']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>

    <record id="mailing_mailing_view_kanban_sms" model="ir.ui.view">
        <field name="name">mailing.mailing.view.kanban.inherit.sms</field>
        <field name="model">mailing.mailing</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='active']" position="after">
                <field name="sms_has_insufficient_credit"/>
            </xpath>
            <xpath expr="//div[@name='stat_opened']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//div[@name='stat_replied']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//div[@name='div_responsible_avatar']" position="after">
                <div class="alert alert-warning mb-0 mt-3" role="alert" attrs="{'invisible': [('sms_has_insufficient_credit', '=', False)]}">
                    <a name="action_buy_sms_credits" type="object">Insufficient credits</a>
                </div>
            </xpath>
        </field>
    </record>

    <record id="mailing_mailing_view_tree_sms" model="ir.ui.view">
        <field name="name">mailing.mailing.view.tree.sms</field>
        <field name="model">mailing.mailing</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="SMS Marketing">
                <field name="subject"/>
                <field name="mailing_type" invisible="1"/>
                <field name="sent"/>
                <field name="clicked"/>
                <field name="bounced"/>
                <field name="campaign_id" groups="mass_mailing.group_mass_mailing_campaign"/>
            </tree>
        </field>
    </record>

    <record id="mailing_mailing_action_sms" model="ir.actions.act_window">
        <field name="name">SMS Marketing</field>
        <field name="res_model">mailing.mailing</field>
        <field name="view_mode">kanban,tree,form,graph</field>
        <field name="search_view_id" ref="mailing_mailing_view_search_sms"/>
        <field name="domain">[('mailing_type', '=', 'sms')]</field>
        <field name="context">{
                'search_default_assigned_to_me': 1,
                'default_user_id': uid,
                'default_mailing_type': 'sms',
                'mailing_sms': True
        }</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new SMS Marketing
          </p><p>
            You can easily send SMS to any contact saved in other Odoo apps.
          </p>
        </field>
    </record>
    <record id="mailing_mailing_action_sms_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="mailing_mailing_view_kanban_sms"/>
        <field name="act_window_id" ref="mailing_mailing_action_sms"/>
    </record>
    <record id="mailing_mailing_action_sms_view_tree" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="mailing_mailing_view_tree_sms"/>
        <field name="act_window_id" ref="mailing_mailing_action_sms"/>
    </record>

    <!-- SMS Marketing / SMS Marketing -->
    <record id="mass_mailing_sms_menu_mass_sms" model="ir.ui.menu">
        <field name="action" ref="mailing_mailing_action_sms"/>
    </record>

</data></odoo>

```

## File: views\mailing_sms_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- SMS Marketing -->
    <menuitem id="mass_mailing_sms_menu_root"
    	name="SMS Marketing"
    	sequence="60"
    	web_icon="mass_mailing_sms,static/description/icon.png"
    	groups="mass_mailing.group_mass_mailing_user"/>

    <!-- SMS Marketing / SMS Marketing -->
    <menuitem id="mass_mailing_sms_menu_mass_sms"
        name="SMS Marketing"
        parent="mass_mailing_sms_menu_root"
        sequence="1"
        groups="mass_mailing.group_mass_mailing_user"/>

    <!-- SMS Marketing / Contacts Lists -->
    <menuitem id="mass_mailing_sms_menu_contacts"
        name="Contacts Lists"
        parent="mass_mailing_sms_menu_root"
        sequence="2"
        groups="mass_mailing.group_mass_mailing_user"/>
    <!-- SMS Marketing / Contacts Lists / Contacts Lists -->
    <menuitem id="mailing_list_menu_sms"
        name="Contacts Lists"
        parent="mass_mailing_sms_menu_contacts"
        sequence="1"
        groups="mass_mailing.group_mass_mailing_user"/>
        <!-- SMS Marketing / Contacts Lists / Contacts -->
    <menuitem id="mailing_contact_menu_sms"
        name="Contacts"
        parent="mass_mailing_sms_menu_contacts"
        sequence="2"
        groups="mass_mailing.group_mass_mailing_user"/>

    <!-- SMS Marketing / Reporting -->
    <menuitem id="mass_mailing_sms_menu_reporting"
        name="Reporting"
        parent="mass_mailing_sms_menu_root"
        sequence="80"
        groups="mass_mailing.group_mass_mailing_user"/>

    <!-- SMS Marketing / Configuration -->
    <menuitem id="mass_mailing_sms_menu_configuration"
        name="Configuration"
        parent="mass_mailing_sms_menu_root"
        sequence="100"
        groups="mass_mailing.group_mass_mailing_user"/>
    <!-- SMS Marketing / Configuration / Blacklist -->
    <menuitem id="phone_blacklist_menu"
    	name="Phone Blacklist"
    	parent="mass_mailing_sms_menu_configuration"
    	sequence="1"
        action="phone_validation.phone_blacklist_action"
        groups="mass_mailing.group_mass_mailing_user"/>
    <!-- SMS Marketing / Configuration / Link Tracker -->
    <menuitem id="link_tracker_menu"
        name="Link Tracker Blacklist"
        parent="mass_mailing_sms_menu_configuration"
        sequence="2"
        action="link_tracker.link_tracker_action"
        groups="mass_mailing.group_mass_mailing_user"/>
</odoo>
```

## File: views\mailing_trace_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_trace_view_search" model="ir.ui.view">
        <field name="name">mailing.trace.view.search.inherit.sms</field>
        <field name="model">mailing.trace</field>
        <field name="inherit_id" ref="mass_mailing.mailing_trace_view_search"/>
        <field name="arch" type="xml">
           <xpath expr="//field[@name='email']" position="after">
                <field name="sms_sms_id_int"/>
                <field name="sms_sms_id"/>
                <field name="sms_number"/>
           </xpath>
        </field>
    </record>

    <record id="mailing_trace_view_tree" model="ir.ui.view">
        <field name="name">mailing.trace.view.tree.inherit.sms</field>
        <field name="model">mailing.trace</field>
        <field name="inherit_id" ref="mass_mailing.mailing_trace_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="before">
                <field name="trace_type"/>
            </xpath>
            <xpath expr="//field[@name='email']" position="after">
                <field name="sms_number"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_trace_view_tree_sms" model="ir.ui.view">
        <field name="name">mailing.trace.view.tree.sms</field>
        <field name="model">mailing.trace</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="SMS Traces" create="0">
                <field name="sms_number"/>
                <field name="state"/>
                <field name="scheduled"/>
                <field name="sent"/>
                <field name="clicked"/>
                <field name="bounced"/>
                <field name="exception"/>
                <field name="ignored"/>
                <field name="failure_type"/>
            </tree>
        </field>
    </record>

    <record id="mailing_trace_view_form" model="ir.ui.view">
        <field name="name">mailing.trace.view.form.inherit.sms</field>
        <field name="model">mailing.trace</field>
        <field name="inherit_id" ref="mass_mailing.mailing_trace_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//span[@name='trace_type_name_mail']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='trace_type_name_mail']" position="after">
                <span name="trace_type_name_sms" attrs="{'invisible': [('trace_type', '!=', 'sms')]}">This sms</span>
            </xpath>
            <xpath expr="//field[@name='email']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='mail_mail_id_int']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='message_id']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='opened']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '=', 'sms')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='replied']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '=', 'sms')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='email']" position="after">
                <field name="sms_number" attrs="{'invisible': [('trace_type', '!=', 'sms')]}"/>
                <field name="sms_sms_id_int" attrs="{'invisible': [('trace_type', '!=', 'sms')]}"/>
                <field name="sms_code" attrs="{'invisible': [('trace_type', '!=', 'sms')]}"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_trace_view_form_sms" model="ir.ui.view">
        <field name="name">mailing.trace.view.form.sms</field>
        <field name="model">mailing.trace</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <form string="SMS Trace" create="0">
                <header>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="alert alert-info text-center" attrs="{'invisible': [('exception', '=', False)]}" role="alert">
                        <strong>This SMS could not be sent.</strong>
                    </div>
                    <div class="alert alert-info text-center" attrs="{'invisible': [('bounced', '=', False)]}" role="alert">
                        <strong>This number appears to be invalid.</strong>
                    </div>
                    <group>
                        <group>
                            <field name="sms_number"/>
                            <field name="mass_mailing_id"/>
                            <field name="campaign_id" groups="mass_mailing.group_mass_mailing_campaign"/>
                            <field name="sms_sms_id_int" groups="base.group_no_one"/>
                            <field name="model" groups="base.group_no_one"/>
                            <field name="res_id" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="scheduled"/>
                            <field name="sent"/>
                            <field name="clicked"/>
                            <field name="bounced"/>
                            <field name="exception"/>
                            <field name="failure_type" attrs="{'invisible': [('exception', '=', False)]}"/>
                            <field name="ignored"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: views\mass_mailing_sms_templates_portal.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="blacklist_main" name="Unsubscribed">
        <t t-call="portal.frontend_layout">
            <div class="container mb64">
                <div class="row">
                    <div class="col-lg-6 offset-lg-3">
                        <h3>SMS Subscription</h3>

                        <form t-att-action="'/sms/%s/unsubscribe/%s' % (mailing_id, trace_code)" method="post">
                            <p>Please enter your phone number</p>
                            <div class="form-group row">
                                <label for="sms_number" class="col-sm-2 col-form-label">Number</label>
                                <div class="col-sm-10">
                                    <input type="text" class="form-control" name="sms_number" id="sms_number" t-att-required="true"/>
                                </div>
                            </div>
                            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                            <input type="hidden" name="trace_code" t-att-value="trace_code"/>
                            <input type="hidden" name="mailing_id" t-att-value="mailing_id"/>
                            <div class="form-group row">
                                <div class="col-sm-10 offset-sm-2">
                                    <button type="submit" class="btn btn-primary">Unsubscribe me</button>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="blacklist_number" name="Unsubscribed">
        <t t-call="portal.frontend_layout">
            <div class="container mb64">
                <div class="row">
                    <div class="col-lg-6 offset-lg-3">
                        <h3>SMS Subscription</h3>

                        <div t-if="unsubscribe_error" class="alert alert-danger text-center" role="alert">
                            <p>There was an error when trying to unsubscribe <strong t-esc="sms_number"/></p>
                            <p t-esc="unsubscribe_error"/>
                        </div>
                        <div t-else="" class="alert alert-success text-center" role="status">
                            <t t-if="lists_optout">
                                <p><strong t-esc="sms_number"/> has been successfully removed from</p>
                                <t t-foreach="lists_optout" t-as="list_id">
                                    <strong t-esc="list_id.name"/><br />
                                </t>
                            </t>
                            <p t-else="">
                                <strong t-esc="sms_number"/> has been successfully blacklisted
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>
</odoo>

```

## File: views\utm_campaign_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="utm_campaign_view_form">
        <field name="name">utm.campaign.view.form</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="action_create_mass_sms" type="object"  class="oe_highlight" string="Send SMS"/>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="action_redirect_to_mailing_sms"
                 type="object"
                 class="oe_stat_button order-11"
                 attrs="{'invisible': [('mailing_sms_count', '=', 0)]}"
                 icon="fa-mobile">
                    <field name="mailing_sms_count" widget="statinfo" string="SMS"/>
                </button>
            </xpath>
            <xpath expr="//notebook" position="inside">
                <page string="SMS" attrs="{'invisible': [('mailing_sms_count', '=', 0)]}">
                    <group>
                        <field name="mailing_sms_ids" readonly="1" nolabel="1">
                            <tree>
                                <field name="name"/>
                                <field name="sent_date"/>
                                <field name="state"/>
                                <field name="bounced"/>
                                <field name="delivered"/>
                                <button name="action_duplicate" type="object" string="Duplicate"/>
                            </tree>
                        </field>
                    </group>
                </page>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_campaign_view_kanban">
        <field name="name">utm.campaign.view.kanban</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="mailing_sms_count"/>
            </xpath>
            <xpath expr="//ul[@id='o_utm_actions']">
                <a name="action_redirect_to_mailing_sms" type="object"
                    t-attf-class="oe_mailings #{record.mailing_sms_count.raw_value === 0 ? 'text-muted' : ''}">
                    <t t-raw="record.mailing_sms_count.raw_value"/> SMS
                </a>
            </xpath>
        </field>
    </record>

    <menuitem name="Campaigns" id="menu_email_campaigns"
        parent="mass_mailing_sms_menu_root" sequence="5"
        action="mass_mailing.action_view_utm_campaigns"
        groups="mass_mailing.group_mass_mailing_campaign"/>
</odoo>

```

## File: wizard\mailing_sms_test.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, exceptions, fields, models, _
from odoo.addons.phone_validation.tools import phone_validation


class MassSMSTest(models.TransientModel):
    _name = 'mailing.sms.test'
    _description = 'Test SMS Mailing'

    def _default_numbers(self):
        return self.env.user.partner_id.phone_sanitized or ""

    numbers = fields.Char(string='Number(s)', required=True,
                          default=_default_numbers, help='Comma-separated list of phone numbers')
    mailing_id = fields.Many2one('mailing.mailing', string='Mailing', required=True, ondelete='cascade')

    def action_send_sms(self):
        self.ensure_one()
        numbers = [number.strip() for number in self.numbers.split(',')]
        sanitize_res = phone_validation.phone_sanitize_numbers_w_record(numbers, self.env.user)
        sanitized_numbers = [info['sanitized'] for info in sanitize_res.values() if info['sanitized']]
        invalid_numbers = [number for number, info in sanitize_res.items() if info['code']]
        if invalid_numbers:
            raise exceptions.UserError(_('Following numbers are not correctly encoded: %s, example : "+32 495 85 85 77, +33 545 55 55 55"') % repr(invalid_numbers))
        self.env['sms.api']._send_sms_batch([{
            'res_id': 0,
            'number': number,
            'content': self.mailing_id.body_plaintext,
        } for number in sanitized_numbers])
        return True

```

## File: wizard\mailing_sms_test_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mailing_sms_test_view_form" model="ir.ui.view">
        <field name="name">mailing.sms.test.view.form</field>
        <field name="model">mailing.sms.test</field>
        <field name="arch" type="xml">
            <form string="Send a Sample SMS">
                <p class="text-muted">
                    Send a sample SMS for testing purpose to the numbers below (comma-separated list). 
                </p>
                <group>
                    <field name="numbers" placeholder="+32 495 85 85 77, +33 545 55 55 55"/>
                    <field name="mailing_id" invisible="1"/>
                </group>
                <footer>
                    <button string="Send" name="action_send_sms" type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="mailing_sms_test_action" model="ir.actions.act_window">
        <field name="name">Test SMS Marketing</field>
        <field name="res_model">mailing.sms.test</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\sms_composer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug.urls

from odoo import fields, models, _


class SMSComposer(models.TransientModel):
    _inherit = 'sms.composer'

    # mass mode with mass sms
    mass_sms_allow_unsubscribe = fields.Boolean('Include opt-out link', default=True)
    mailing_id = fields.Many2one('mailing.mailing', string='Mailing')
    utm_campaign_id = fields.Many2one('utm.campaign', string='Campaign')

    # ------------------------------------------------------------
    # Mass mode specific
    # ------------------------------------------------------------

    def _get_unsubscribe_url(self, res_id, trace_code, number):
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        return werkzeug.urls.url_join(
            base_url,
            '/sms/%s/%s' % (self.mailing_id.id, trace_code)
        )

    def _prepare_mass_sms_trace_values(self, record, sms_values):
        trace_code = self.env['mailing.trace']._get_random_code()
        trace_values = {
            'model': self.res_model,
            'res_id': record.id,
            'trace_type': 'sms',
            'mass_mailing_id': self.mailing_id.id,
            'sms_number': sms_values['number'],
            'sms_code': trace_code,
        }
        if sms_values['state'] == 'error':
            if sms_values['error_code'] == 'sms_number_format':
                trace_values['sent'] = fields.Datetime.now()
                trace_values['bounced'] = fields.Datetime.now()
            else:
                trace_values['exception'] = fields.Datetime.now()
        elif sms_values['state'] == 'canceled':
            trace_values['ignored'] = fields.Datetime.now()
        else:
            if self.mass_sms_allow_unsubscribe:
                sms_values['body'] = '%s\n%s' % (sms_values['body'] or '', _('STOP SMS : %s') % self._get_unsubscribe_url(record.id, trace_code, sms_values['number']))
        return trace_values

    def _get_blacklist_record_ids(self, records, recipients_info):
        """ Consider opt-outed contact as being blacklisted for that specific
        mailing. """
        res = super(SMSComposer, self)._get_blacklist_record_ids(records, recipients_info)
        if self.mailing_id:
            optout_res_ids = self.mailing_id._get_opt_out_list_sms()
            res += optout_res_ids
        return res

    def _get_done_record_ids(self, records, recipients_info):
        """ A/B testing could lead to records having been already mailed. """
        res = super(SMSComposer, self)._get_done_record_ids(records, recipients_info)
        if self.mailing_id:
            seen_ids, seen_list = self.mailing_id._get_seen_list_sms()
            res += seen_ids
        return res

    def _prepare_body_values(self, records):
        all_bodies = super(SMSComposer, self)._prepare_body_values(records)
        if self.mailing_id:
            tracker_values = self.mailing_id._get_link_tracker_values()
            for sms_id, body in all_bodies.items():
                body = self.env['link.tracker'].sudo()._convert_links_text(body, tracker_values)
                all_bodies[sms_id] = body
        return all_bodies

    def _prepare_mass_sms_values(self, records):
        result = super(SMSComposer, self)._prepare_mass_sms_values(records)
        if self.composition_mode == 'mass' and self.mailing_id:
            for record in records:
                sms_values = result[record.id]

                trace_values = self._prepare_mass_sms_trace_values(record, sms_values)
                sms_values.update({
                    'mailing_id': self.mailing_id.id,
                    'mailing_trace_ids': [(0, 0, trace_values)],
                })
        return result

    def _prepare_mass_sms(self, records, sms_record_values):
        sms_all = super(SMSComposer, self)._prepare_mass_sms(records, sms_record_values)
        if self.mailing_id:
            updated_bodies = sms_all._update_body_short_links()
            for sms in sms_all:
                sms.body = updated_bodies[sms.id]
        return sms_all

```

## File: wizard\sms_composer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sms_composer_view_form" model="ir.ui.view">
        <field name="name">sms.composer.views.inherit.sms</field>
        <field name="model">sms.composer</field>
        <field name="inherit_id" ref="sms.sms_composer_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='res_model']" position="after">
                <field name="utm_campaign_id" groups="mass_mailing.group_mass_mailing_campaign"
                    invisible="1"/>
                <field name="mailing_id" invisible="1"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_sms_test
from . import sms_composer

```

