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
    'sequence': 245,
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
    'assets': {
        'web.assets_backend': [
            'mass_mailing_sms/static/src/js/fields_sms_widget.js',
        ],
    },
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
            return request.redirect('/web')
        return request.render('mass_mailing_sms.blacklist_main', {
            'mailing_id': mailing_id,
            'trace_code': trace_code,
        })

    @http.route(['/sms/<int:mailing_id>/unsubscribe/<string:trace_code>'], type='http', website=True, auth='public')
    def blacklist_number(self, mailing_id, trace_code, **post):
        check_res = self._check_trace(mailing_id, trace_code)
        if not check_res.get('trace'):
            return request.redirect('/web')
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
            unsubscribe_error = _('Number %s not found', tocheck_number)
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
        return request.redirect(request.env['link.tracker'].get_url_from_code(code), code=301, local=False)

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
        <field name="trace_status">sent</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="write_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_1" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000001</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_13"/>
        <field name="trace_status">sent</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="write_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_2" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000002</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_14"/>
        <field name="trace_status">sent</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="write_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_3" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000003</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_24"/>
        <field name="trace_status">sent</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="write_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_4" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32465000004</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_33"/>
        <field name="trace_status">error</field>
        <field name="failure_type">sms_server</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="write_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_5" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">123123</field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_33"/>
        <field name="trace_status">bounce</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="write_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_0_trace_6" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_0"/>
        <field name="trace_type">sms</field>
        <field name="sms_number"></field>
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_34"/>
        <field name="trace_status">error</field>
        <field name="failure_type">sms_number_missing</field>
        <field name="sent_datetime" eval="False"/>
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
        <field name="trace_status">sent</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_1_trace_1" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32456001111</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_1"/>
        <field name="trace_status">sent</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_1_trace_2" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32456001122</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_2"/>
        <field name="trace_status">sent</field>
        <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
    <record id="mailing_sms_1_trace_3" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">+32456001133</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_3"/>
        <field name="trace_status">error</field>
        <field name="failure_type">sms_credit</field>
    </record>
    <record id="mailing_sms_1_trace_4" model="mailing.trace">
        <field name="mass_mailing_id" ref="mailing_sms_1"/>
        <field name="trace_type">sms</field>
        <field name="sms_number">dummy</field>
        <field name="model">mailing.contact</field>
        <field name="res_id" ref="mass_mailing_sms.mailing_contact_0_4"/>
        <field name="trace_status">error</field>
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
        <field name="email">hubert.farnsworth@example.com</field>
        <field name="list_ids" eval="[(4, ref('mass_mailing_sms.mailing_list_sms_0'))]"/>
    </record>
    <record id="mailing_contact_0_1" model="mailing.contact">
        <field name="name">Philip Fry</field>
        <field name="mobile">+32456001111</field>
        <field name="email">philip.fry@example.com</field>
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

    def _phone_get_number_fields(self):
        return ['mobile']

```

## File: models\mailing_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailingList(models.Model):
    _inherit = 'mailing.list'

    contact_count_sms = fields.Integer(compute="_compute_mailing_list_statistics", string="SMS Contacts")

    def action_view_mailings(self):
        if self.env.context.get('mailing_sms'):
            action = self.env["ir.actions.actions"]._for_xml_id('mass_mailing_sms.mailing_mailing_action_sms')
            action['domain'] = [('id', 'in', self.mailing_ids.ids)]
            action['context'] = {
                'default_mailing_type': 'sms',
                'default_contact_list_ids': self.ids,
                'mailing_sms': True
            }
            return action
        else:
            return super(MailingList, self).action_view_mailings()

    def action_view_contacts_sms(self):
        action = self.action_view_contacts()
        action['context'] = dict(action.get('context', {}), search_default_filter_valid_sms_recipient=1)
        return action

    def _get_contact_statistics_fields(self):
        """ See super method docstring for more info.
        Adds:
        - contact_count_sms:           all valid sms
        - contact_count_blacklisted:   override the dict entry to add SMS blacklist condition """

        values = super(MailingList, self)._get_contact_statistics_fields()
        values.update({
            'contact_count_sms': '''
                SUM(CASE WHEN
                    (c.phone_sanitized IS NOT NULL
                    AND COALESCE(r.opt_out,FALSE) = FALSE
                    AND bl_sms.id IS NULL)
                THEN 1 ELSE 0 END) AS contact_count_sms''',
            'contact_count_blacklisted': f'''
                SUM(CASE WHEN (bl.id IS NOT NULL OR bl_sms.id IS NOT NULL)
                THEN 1 ELSE 0 END) AS contact_count_blacklisted'''
        })
        return values

    def _get_contact_statistics_joins(self):
        return super(MailingList, self)._get_contact_statistics_joins() + '''
            LEFT JOIN phone_blacklist bl_sms ON c.phone_sanitized = bl_sms.number and bl_sms.active
        '''

    def _mailing_get_opt_out_list_sms(self, mailing):
        """ Check subscription on all involved mailing lists. If user is opt_out
        on one list but not on another, one opted in and the other one opted out,
        send mailing anyway.

        :return list: opt-outed record IDs
        """
        subscriptions = self.subscription_ids if self else mailing.contact_list_ids.subscription_ids
        opt_out_contacts = subscriptions.filtered(lambda sub: sub.opt_out).mapped('contact_id')
        opt_in_contacts = subscriptions.filtered(lambda sub: not sub.opt_out).mapped('contact_id')
        return list(set(c.id for c in opt_out_contacts if c not in opt_in_contacts))

```

## File: models\mailing_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from urllib.parse import urljoin

from odoo import api, fields, models, _
from odoo.addons.link_tracker.models.link_tracker import LINK_TRACKER_MIN_CODE_LENGTH
from odoo.exceptions import UserError
from odoo.osv import expression

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
    mailing_type = fields.Selection(selection_add=[
        ('sms', 'SMS')
    ], ondelete={'sms': 'set default'})

    # 'sms_subject' added to override 'subject' field (string attribute should be labelled "Title" when mailing_type == 'sms').
    # 'sms_subject' should have the same helper as 'subject' field when 'mass_mailing_sms' installed.
    # otherwise 'sms_subject' will get the old helper from 'mass_mailing' module.
    # overriding 'subject' field helper in this model is not working, since the helper will keep the new value
    # even when 'mass_mailing_sms' removed (see 'mailing_mailing_view_form_sms' for more details).
    sms_subject = fields.Char('Title', help='For an email, the subject your recipients will see in their inbox.\n'
                              'For an SMS, the internal title of the message.',
                              related='subject', translate=False, readonly=False)
    # sms options
    body_plaintext = fields.Text('SMS Body', compute='_compute_body_plaintext', store=True, readonly=False)
    sms_template_id = fields.Many2one('sms.template', string='SMS Template', ondelete='set null')
    sms_has_insufficient_credit = fields.Boolean(
        'Insufficient IAP credits', compute='_compute_sms_has_iap_failure',
        help='UX Field to propose to buy IAP credits')
    sms_has_unregistered_account = fields.Boolean(
        'Unregistered IAP account', compute='_compute_sms_has_iap_failure',
        help='UX Field to propose to Register the SMS IAP account')
    sms_force_send = fields.Boolean(
        'Send Directly', help='Immediately send the SMS Mailing instead of queuing up. Use at your own risk.')
    # opt_out_link
    sms_allow_unsubscribe = fields.Boolean('Include opt-out link', default=False)
    # A/B Testing
    ab_testing_sms_winner_selection = fields.Selection(related="campaign_id.ab_testing_sms_winner_selection", default="clicks_ratio", readonly=False, copy=True)

    @api.depends('mailing_type')
    def _compute_medium_id(self):
        super(Mailing, self)._compute_medium_id()
        for mailing in self:
            if mailing.mailing_type == 'sms' and (not mailing.medium_id or mailing.medium_id == self.env.ref('utm.utm_medium_email')):
                mailing.medium_id = self.env.ref('mass_mailing_sms.utm_medium_sms').id
            elif mailing.mailing_type == 'mail' and (not mailing.medium_id or mailing.medium_id == self.env.ref('mass_mailing_sms.utm_medium_sms')):
                mailing.medium_id = self.env.ref('utm.utm_medium_email').id

    @api.depends('sms_template_id', 'mailing_type')
    def _compute_body_plaintext(self):
        for mailing in self:
            if mailing.mailing_type == 'sms' and mailing.sms_template_id:
                mailing.body_plaintext = mailing.sms_template_id.body

    @api.depends('mailing_trace_ids.failure_type')
    def _compute_sms_has_iap_failure(self):
        failures = ['sms_acc', 'sms_credit']
        if not self.ids:
            self.sms_has_insufficient_credit = self.sms_has_unregistered_account = False
        else:
            traces = self.env['mailing.trace'].sudo().read_group([
                        ('mass_mailing_id', 'in', self.ids),
                        ('trace_type', '=', 'sms'),
                        ('failure_type', 'in', failures)
            ], ['mass_mailing_id', 'failure_type'], ['mass_mailing_id', 'failure_type'], lazy=False)

            trace_dict = dict.fromkeys(self.ids, {key: False for key in failures})
            for t in traces:
                trace_dict[t['mass_mailing_id'][0]][t['failure_type']] =  t['__count'] and True or False

            for mail in self:
                mail.sms_has_insufficient_credit = trace_dict[mail.id]['sms_credit']
                mail.sms_has_unregistered_account = trace_dict[mail.id]['sms_acc']

    # --------------------------------------------------
    # ORM OVERRIDES
    # --------------------------------------------------

    @api.model
    def create(self, values):
        # Get subject from "sms_subject" field when SMS installed (used to build the name of record in the super 'create' method)
        if values.get('mailing_type') == 'sms' and values.get('sms_subject'):
            values['subject'] = values['sms_subject']
        return super(Mailing, self).create(values)

    # --------------------------------------------------
    # BUSINESS / VIEWS ACTIONS
    # --------------------------------------------------

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
        self.action_put_in_queue()

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
        """ Give list of opt-outed records, depending on specific model-based
        computation if available.

        :return list: opt-outed record IDs
        """
        self.ensure_one()
        opt_out = []
        target = self.env[self.mailing_model_real]
        if hasattr(self.env[self.mailing_model_name], '_mailing_get_opt_out_list_sms'):
            opt_out = self.env[self.mailing_model_name]._mailing_get_opt_out_list_sms(self)
            _logger.info("Mass SMS %s targets %s: optout: %s contacts", self, target._name, len(opt_out))
        else:
            _logger.info("Mass SMS %s targets %s: no opt out list available", self, target._name)
        return opt_out

    def _get_seen_list_sms(self):
        """Returns a set of emails already targeted by current mailing/campaign (no duplicates)"""
        self.ensure_one()
        target = self.env[self.mailing_model_real]

        partner_fields = []
        if isinstance(target, self.pool['mail.thread.phone']):
            phone_fields = ['phone_sanitized']
        elif isinstance(target, self.pool['mail.thread']):
            phone_fields = [
                fname for fname in target._sms_get_number_fields()
                if fname in target._fields and target._fields[fname].store
            ]
            partner_fields = target._sms_get_partner_fields()
        else:
            phone_fields = []
            if 'mobile' in target._fields and target._fields['mobile'].store:
                phone_fields.append('mobile')
            if 'phone' in target._fields and target._fields['phone'].store:
                phone_fields.append('phone')
        partner_field = next(
            (fname for fname in partner_fields if target._fields[fname].store and target._fields[fname].type == 'many2one'),
            False
        )
        if not phone_fields and not partner_field:
            raise UserError(_("Unsupported %s for mass SMS", self.mailing_model_id.name))

        query = """
            SELECT %(select_query)s
              FROM mailing_trace trace
              JOIN %(target_table)s target ON (trace.res_id = target.id)
              %(join_add_query)s
             WHERE (%(where_query)s)
               AND trace.mass_mailing_id = %%(mailing_id)s
               AND trace.model = %%(target_model)s
        """
        if phone_fields:
            # phone fields are checked on target mailed model
            select_query = 'target.id, ' + ', '.join('target.%s' % fname for fname in phone_fields)
            where_query = ' OR '.join('target.%s IS NOT NULL' % fname for fname in phone_fields)
            join_add_query = ''
        else:
            # phone fields are checked on res.partner model
            partner_phone_fields = ['mobile', 'phone']
            select_query = 'target.id, ' + ', '.join('partner.%s' % fname for fname in partner_phone_fields)
            where_query = ' OR '.join('partner.%s IS NOT NULL' % fname for fname in partner_phone_fields)
            join_add_query = 'JOIN res_partner partner ON (target.%s = partner.id)' % partner_field

        query = query % {
            'select_query': select_query,
            'where_query': where_query,
            'target_table': target._table,
            'join_add_query': join_add_query,
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
            if res_ids:
                composer = self.env['sms.composer'].with_context(active_id=False).create(mailing._send_sms_get_composer_values(res_ids))
                composer._action_send_sms()

            mailing.write({
                'state': 'done',
                'sent_date': fields.Datetime.now(),
                'kpi_mail_required': not mailing.sent_date,
                })
        return True

    # ------------------------------------------------------
    # STATISTICS
    # ------------------------------------------------------

    def _prepare_statistics_email_values(self):
        """Return some statistics that will be displayed in the mailing statistics email.

        Each item in the returned list will be displayed as a table, with a title and
        1, 2 or 3 columns.
        """
        values = super(Mailing, self)._prepare_statistics_email_values()
        if self.mailing_type == 'sms':
            mailing_type = self._get_pretty_mailing_type()
            values['title'] = _('24H Stats of %(mailing_type)s "%(mailing_name)s"',
                                mailing_type=mailing_type,
                                mailing_name=self.subject
                               )
            values['kpi_data'][0] = {
                'kpi_fullname': _('Report for %(expected)i %(mailing_type)s Sent',
                                  expected=self.expected,
                                  mailing_type=mailing_type
                                 ),
                'kpi_col1': {
                    'value': f'{self.received_ratio}%',
                    'col_subtitle': _('RECEIVED (%i)', self.delivered),
                },
                'kpi_col2': {
                    'value': f'{self.clicks_ratio}%',
                    'col_subtitle': _('CLICKED (%i)', self.clicked),
                },
                'kpi_col3': {
                    'value': f'{self.bounced_ratio}%',
                    'col_subtitle': _('BOUNCED (%i)', self.bounced),
                },
                'kpi_action': None,
                'kpi_name': self.mailing_type,
            }
        return values

    def _get_pretty_mailing_type(self):
        if self.mailing_type == 'sms':
            return _('SMS Text Message')
        return super(Mailing, self)._get_pretty_mailing_type()

    # --------------------------------------------------
    # TOOLS
    # --------------------------------------------------

    def _get_default_mailing_domain(self):
        mailing_domain = super(Mailing, self)._get_default_mailing_domain()
        if self.mailing_type == 'sms' and 'phone_sanitized_blacklisted' in self.env[self.mailing_model_name]._fields:
            mailing_domain = expression.AND([mailing_domain, [('phone_sanitized_blacklisted', '=', False)]])

        return mailing_domain

    def convert_links(self):
        sms_mailings = self.filtered(lambda m: m.mailing_type == 'sms')
        res = {}
        for mailing in sms_mailings:
            tracker_values = mailing._get_link_tracker_values()
            body = mailing._shorten_links_text(mailing.body_plaintext, tracker_values)
            res[mailing.id] = body
        res.update(super(Mailing, self - sms_mailings).convert_links())
        return res

    def get_sms_link_replacements_placeholders(self):
        """Get placeholders for replaced links in sms widget for accurate computation of sms counts.

        Reminders and assumptions:
          * Links wille be transformed to the format "[base_url]/r/[link_tracker_code]/s/[sms_id]".
          * unsubscribe is formatted as: "\nSTOP SMS : [base_url]/sms/[mailing_id]/[trace_code]".

        :return: Character counts used for links, formatted as `{link: str, unsubscribe: str}`.
        """
        if self:
            self.ensure_one()

        self.check_access_rights('write')

        max_sms = self.env['sms.sms'].sudo().search_read([], ['id'], order='id desc', limit=1)
        sms_id_length = max(len(str(max_sms[0]['id'])), 5) if max_sms else 5  # Assumes a mailing won't be more than 10⁵ sms at once
        max_code = self.env['link.tracker.code'].sudo().search_read([], ['code'], order='id DESC', limit=1)
        code_length = len(max_code[0]['code']) + 1 if max_code else LINK_TRACKER_MIN_CODE_LENGTH

        if self.id:
            mailing_id_placeholder_length = len(str(self.id))
        else:
            max_mailing = self.env['mailing.mailing'].sudo().search_read([], ['id'], order='id DESC', limit=1)
            mailing_id_placeholder_length = len(str(max_mailing[0]['id'] + 1)) if max_mailing else 1
        mailing_id_placeholder = 'x' * mailing_id_placeholder_length

        base_url = self.get_base_url()
        opt_out_url = urljoin(base_url, f"sms/{mailing_id_placeholder}/{'x' * self.env['mailing.trace'].CODE_SIZE}")
        return {
            'link': urljoin(base_url, f"r/{'x' * code_length}/s/{'x' * sms_id_length}"),
            'unsubscribe': f"\n{self.env['sms.composer']._get_unsubscribe_info(opt_out_url)}"
        }

    # ------------------------------------------------------
    # A/B Test Override
    # ------------------------------------------------------

    def _get_ab_testing_description_modifying_fields(self):
        fields_list = super()._get_ab_testing_description_modifying_fields()
        return fields_list + ['ab_testing_sms_winner_selection']

    def _get_ab_testing_description_values(self):
        values = super()._get_ab_testing_description_values()
        if self.mailing_type == 'sms':
            values.update({
                'ab_testing_winner_selection': self.ab_testing_sms_winner_selection,
            })
        return values

    def _get_ab_testing_winner_selection(self):
        result = super()._get_ab_testing_winner_selection()
        if self.mailing_type == 'sms':
            ab_testing_winner_selection_description = dict(
                self._fields.get('ab_testing_sms_winner_selection').related_field.selection
            ).get(self.ab_testing_sms_winner_selection)
            result.update({
                'value': self.campaign_id.ab_testing_sms_winner_selection,
                'description': ab_testing_winner_selection_description
            })
        return result

    def _get_ab_testing_siblings_mailings(self):
        mailings = super()._get_ab_testing_siblings_mailings()
        if self.mailing_type == 'sms':
            mailings = self.campaign_id.mailing_sms_ids.filtered('ab_testing_enabled')
        return mailings

    def _get_default_ab_testing_campaign_values(self, values=None):
        campaign_values = super()._get_default_ab_testing_campaign_values(values)
        values = values or dict()
        if self.mailing_type == 'sms':
            sms_subject = values.get('sms_subject') or self.sms_subject
            if sms_subject:
                campaign_values['name'] = _("A/B Test: %s", sms_subject)
            campaign_values['ab_testing_sms_winner_selection'] = self.ab_testing_sms_winner_selection
        return campaign_values

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

    trace_type = fields.Selection(selection_add=[
        ('sms', 'SMS')
    ], ondelete={'sms': 'set default'})
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
        ('sms_acc', 'Unregistered Account'),
        # mass mode specific codes
        ('sms_blacklist', 'Blacklisted'),
        ('sms_duplicate', 'Duplicate'),
        ('sms_optout', 'Opted Out'),
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

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import api, fields, models, modules, _


class Users(models.Model):
    _name = 'res.users'
    _inherit = ['res.users']

    @api.model
    def systray_get_activities(self):
        """ Split mass_mailing and mass_mailing_sms activities in systray by 
            removing the single mailing.mailing activity represented and
            doing a new query to split them by mailing_type.
        """
        activities = super(Users, self).systray_get_activities()
        for activity in activities:
            if activity.get('model') == 'mailing.mailing':
                activities.remove(activity)
                query = """SELECT m.mailing_type, count(*), act.res_model as model, act.res_id,
                            CASE
                                WHEN %(today)s::date - act.date_deadline::date = 0 Then 'today'
                                WHEN %(today)s::date - act.date_deadline::date > 0 Then 'overdue'
                                WHEN %(today)s::date - act.date_deadline::date < 0 Then 'planned'
                            END AS states
                        FROM mail_activity AS act
                        JOIN mailing_mailing AS m ON act.res_id = m.id
                        WHERE act.res_model = 'mailing.mailing' AND act.user_id = %(user_id)s  
                        GROUP BY m.mailing_type, states, act.res_model, act.res_id;
                        """
                self.env.cr.execute(query, {
                    'today': fields.Date.context_today(self),
                    'user_id': self.env.uid,
                })
                activity_data = self.env.cr.dictfetchall()
                
                user_activities = {}
                for act in activity_data:
                    if not user_activities.get(act['mailing_type']):
                        if act['mailing_type'] == 'sms':
                            module = 'mass_mailing_sms'
                            name = _('SMS Marketing')
                        else:
                            module = 'mass_mailing'
                            name = _('Email Marketing')
                        icon = module and modules.module.get_module_icon(module)
                        res_ids = set()
                        user_activities[act['mailing_type']] = {
                            'name': name,
                            'model': 'mailing.mailing',
                            'type': 'activity',
                            'icon': icon,
                            'total_count': 0, 'today_count': 0, 'overdue_count': 0, 'planned_count': 0,
                            'res_ids': res_ids,
                        }
                    user_activities[act['mailing_type']]['res_ids'].add(act['res_id'])
                    user_activities[act['mailing_type']]['%s_count' % act['states']] += act['count']
                    if act['states'] in ('today', 'overdue'):
                        user_activities[act['mailing_type']]['total_count'] += act['count']

                for mailing_type in user_activities.keys():
                    user_activities[mailing_type].update({
                        'actions': [{'icon': 'fa-clock-o', 'name': 'Summary',}],
                        'domain': json.dumps([['activity_ids.res_id', 'in', list(user_activities[mailing_type]['res_ids'])]])
                    })
                activities.extend(list(user_activities.values()))
                break

        return activities

```

## File: models\sms_sms.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import fields, models, tools


class SmsSms(models.Model):
    _inherit = ['sms.sms']

    mailing_id = fields.Many2one('mailing.mailing', string='Mass Mailing')
    mailing_trace_ids = fields.One2many('mailing.trace', 'sms_sms_id', string='Statistics')

    def _update_body_short_links(self):
        """ Override to tweak shortened URLs by adding statistics ids, allowing to
        find customer back once clicked. """
        res = dict.fromkeys(self.ids, False)
        for sms in self:
            if not sms.mailing_id or not sms.body:
                res[sms.id] = sms.body
                continue

            body = sms.body
            for url in set(re.findall(tools.TEXT_URL_REGEX, body)):
                if url.startswith(sms.get_base_url() + '/r/'):
                    body = re.sub(re.escape(url) + r'(?![\w@:%.+&~#=/-])', url + f'/s/{sms.id}', body)
            res[sms.id] = body
        return res

    def _postprocess_iap_sent_sms(self, iap_results, failure_reason=None, unlink_failed=False, unlink_sent=True):
        all_sms_ids = [item['res_id'] for item in iap_results]
        if any(sms.mailing_id for sms in self.env['sms.sms'].sudo().browse(all_sms_ids)):
            for state in self.IAP_TO_SMS_STATE.keys():
                sms_ids = [item['res_id'] for item in iap_results if item['state'] == state]
                traces = self.env['mailing.trace'].sudo().search([
                    ('sms_sms_id_int', 'in', sms_ids)
                ])
                if traces and state == 'success':
                    traces.set_sent()
                elif traces:
                    traces.set_failed(failure_type=self.IAP_TO_SMS_STATE[state])
        return super(SmsSms, self)._postprocess_iap_sent_sms(
            iap_results, failure_reason=failure_reason,
            unlink_failed=unlink_failed, unlink_sent=unlink_sent
        )

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

    # A/B Testing
    ab_testing_sms_winner_selection = fields.Selection([
        ('manual', 'Manual'),
        ('clicks_ratio', 'Highest Click Rate')], string="SMS Winner Selection", default="clicks_ratio")

    @api.depends('mailing_mail_ids', 'mailing_sms_ids')
    def _compute_ab_testing_total_pc(self):
        super()._compute_ab_testing_total_pc()
        for campaign in self:
            campaign.ab_testing_total_pc += sum([
                mailing.ab_testing_pc for mailing in campaign.mailing_sms_ids.filtered('ab_testing_enabled')
            ])

    @api.depends('mailing_sms_ids')
    def _compute_mailing_sms_count(self):
        for campaign in self:
            campaign.mailing_sms_count = len(campaign.mailing_sms_ids)

    def action_create_mass_sms(self):
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.action_create_mass_mailings_from_campaign")
        action['context'] = {
            'default_campaign_id': self.id,
            'default_mailing_type': 'sms',
            'search_default_assigned_to_me': 1,
            'search_default_campaign_id': self.id,
            'default_user_id': self.env.user.id,
        }
        return action

    def action_redirect_to_mailing_sms(self):
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing_sms.mailing_mailing_action_sms")
        action['context'] = {
            'default_campaign_id': self.id,
            'default_mailing_type': 'sms',
            'search_default_assigned_to_me': 1,
            'search_default_campaign_id': self.id,
            'default_user_id': self.env.user.id,
        }
        action['domain'] = [('mailing_type', '=', 'sms')]
        return action

    @api.model
    def _cron_process_mass_mailing_ab_testing(self):
        ab_testing_campaign = super()._cron_process_mass_mailing_ab_testing()
        for campaign in ab_testing_campaign:
            ab_testing_mailings = campaign.mailing_sms_ids.filtered(lambda m: m.ab_testing_enabled)
            if not ab_testing_mailings.filtered(lambda m: m.state == 'done'):
                continue
            ab_testing_mailings.action_send_winner_mailing()
        return ab_testing_campaign

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_contact
from . import mailing_list
from . import mailing_mailing
from . import mailing_trace
from . import res_users
from . import sms_sms
from . import utm

```

## File: report\mailing_trace_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_trace_report_sms_view_tree" model="ir.ui.view">
        <field name="name">mailing.sms.trace.report.view.tree</field>
        <field name="model">mailing.trace.report</field>
        <field name="inherit_id" ref="mass_mailing.mailing_trace_report_view_tree"/>
        <field name="priority" eval="50"/>
        <field name="mode">primary</field>
        <field name="type">tree</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='opened']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='replied']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="mailing_trace_report_sms_view_pivot" model="ir.ui.view">
        <field name="name">mailing.sms.trace.report.view.pivot</field>
        <field name="model">mailing.trace.report</field>
        <field name="inherit_id" ref="mass_mailing.mailing_trace_report_view_pivot"/>
        <field name="priority" eval="50"/>
        <field name="mode">primary</field>
        <field name="type">pivot</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='opened']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='replied']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="mailing_trace_report_sms_view_graph" model="ir.ui.view">
        <field name="name">mailing.sms.trace.report.view.graph</field>
        <field name="model">mailing.trace.report</field>
        <field name="inherit_id" ref="mass_mailing.mailing_trace_report_view_graph"/>
        <field name="priority" eval="50"/>
        <field name="mode">primary</field>
        <field name="type">graph</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='replied']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <!-- Actions and Menuitems -->
    <record id="mailing_trace_report_action_sms" model="ir.actions.act_window">
        <field name="name">SMS Marketing Analysis</field>
        <field name="res_model">mailing.trace.report</field>
        <field name="view_mode">graph,pivot,tree</field>
        <field name="domain">[('mailing_type', '=', 'sms')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p><p>
                Come back once some SMS Mailings are sent to check out aggregated results.
            </p>
        </field>
    </record>

    <record id="mailing_trace_report_action_sms_view_graph" model="ir.actions.act_window.view">
        <field name="sequence">0</field>
        <field name="view_mode">graph</field>
        <field name="act_window_id" ref="mailing_trace_report_action_sms"/>
        <field name="view_id" ref="mailing_trace_report_sms_view_graph"/>
    </record>

    <record id="mailing_trace_report_action_sms_view_pivot" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">pivot</field>
        <field name="act_window_id" ref="mailing_trace_report_action_sms"/>
        <field name="view_id" ref="mailing_trace_report_sms_view_pivot"/>
    </record>

    <record id="mailing_trace_report_action_sms_view_tree" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="mailing_trace_report_action_sms"/>
        <field name="view_id" ref="mailing_trace_report_sms_view_tree"/>
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
access_mailing_sms_test,access.mailing.sms.test,model_mailing_sms_test,mass_mailing.group_mass_mailing_user,1,1,1,0
access_phone_blacklist_remove_mass_mailing_user,acesss.phone.blacklist.remove.mass_mailing_user,phone_validation.model_phone_blacklist_remove,mass_mailing.group_mass_mailing_user,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><g><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/></g><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" fill-opacity=".151" d="M3 69.6c-2 0-4-1-4-4.1V29.7l18.6-18.9L44.3 12v16.727l4.585-3.484 7.039.82-3.47 4.437 3.062 5L44.3 46.516v6.83L31.2 69.6H3z"/><path fill="#000" fill-opacity=".3" fill-rule="nonzero" d="M44.3 14c0-1.1-.9-2-2-2H19c-1.1 0-2 .9-2 2v40.6c0 1.1.9 2 2 2h23.3c1.1 0 2-.9 2-2v-10c-.7.5-1.5 1-2.3 1.4v5.3c0 .2-.1.3-.3.3H19.6c-.2 0-.3-.1-.3-.3v-33c0-.2.1-.3.3-.3h22.1c.2 0 .3.1.3.3v2.8c.5 0 1.1-.1 1.6-.1h.7v-7zM30.7 52.4c.8 0 1.5.7 1.5 1.5s-.7 1.5-1.5 1.5-1.5-.7-1.5-1.5.7-1.5 1.5-1.5zm3.1-35.9h-6.1c-.3 0-.5-.2-.5-.5s.2-.5.5-.5h6.1c.3 0 .5.2.5.5s-.3.5-.5.5z"/><path fill="#FFF" fill-rule="nonzero" d="M43.6 42.4c-.5 0-1.1 0-1.6-.1v7.1c0 .2-.1.3-.3.3H19.6c-.2 0-.3-.1-.3-.3v-33c0-.2.1-.3.3-.3h22.1c.2 0 .3.1.3.3v2.9c.5 0 1.1-.1 1.6-.1h.7V12c0-1.1-.9-2-2-2H19c-1.1 0-2 .9-2 2v40.7c0 1.1.9 2 2 2h23.3c1.1 0 2-.9 2-2V42.4h-.7zm-16-28.7h6.1c.3 0 .5.2.5.5s-.2.5-.4.5h-6.1c-.3 0-.5-.2-.5-.5s.1-.5.4-.5zm3.1 39.9c-.8 0-1.5-.7-1.5-1.5s.7-1.5 1.5-1.5 1.5.7 1.5 1.5c0 .9-.7 1.5-1.5 1.5z"/><text fill="#000" fill-opacity=".3" font-family="IBMPlexMono-Bold, IBM Plex Mono" font-size="18" font-weight="bold" letter-spacing="-.215" transform="translate(17 10)"><tspan x="8.144" y="29">SMS</tspan></text><text fill="#FFF" font-family="IBMPlexMono-Bold, IBM Plex Mono" font-size="18" font-weight="bold" letter-spacing="-.215" transform="translate(17 10)"><tspan x="8.144" y="27">SMS</tspan></text></g></g></svg>
```

## File: static\src\js\fields_sms_widget.js

```javascript
/** @odoo-module **/

import { _t } from "web.core";
import SmsWidget from '@sms/js/fields_sms_widget';

const TEXT_URL_REGEX = /https?:\/\/[\w@:%.+&~#=/-]+(?:\?\S+)?/g;  // from tools.mail.TEXT_URL_REGEX

/**
 * Override to provide extra characters count information to
 * consider links converted with link_tracker and opt-out
 * link if the option is selected.
 */
SmsWidget.include({
    events: _.extend({}, SmsWidget.prototype.events || {}, {
        'focusin': '_onClick',
    }),
    /**
     *@override
     */
    willStart: async function () {
        await this._super.apply(this, arguments);
        if (this.model === "mailing.mailing" && this.mode === 'edit') {
            const { unsubscribe, link } = await this._rpc({
                model: 'mailing.mailing',
                method: 'get_sms_link_replacements_placeholders',
                args: [this.res_id]
            });
            this.linkReplacementsPlaceholders = { unsubscribe, link };
            this.noticeLinksReplaced = false;
        }
    },
    /**
     * Add the characters for opt-out link if enabled.
     *
     * @override
     */
    _compute: function() {
        this._super.apply(this, arguments);
        if (!this.linkReplacementsPlaceholders) {
            return;
        }
        // this is the simplest way to read from another field in a legacy fix.
        const unsubscribe_el = document.querySelector('div[name="sms_allow_unsubscribe"] input');
        if (unsubscribe_el && unsubscribe_el.checked) {
            this.optOutEnabled = true;
            this.nbrChar += this.linkReplacementsPlaceholders.unsubscribe.length;
        }
    },
    /**
     * Add the applicable extra characters notice.
     *
     * @override
     */
    _getNbrCharExplanationTemplate: function () {
        if (this.optOutEnabled) {
            return this.noticeLinksReplaced
                ? _t("%s characters (including link trackers and opt-out link), fits in %s SMS (%s) ")
                : _t("%s characters (including opt-out link), fits in %s SMS (%s) ");
        }
        return this.noticeLinksReplaced
            ? _t("%s characters (including link trackers), fits in %s SMS (%s) ")
            : this._super.apply(this, arguments); // Also default when no linkReplacementsPlaceholders
    },
    /**
     * Convert links to link trackers-length replacements.
     * Works under the assumption that these trackers urls only
     * contain GSM7 characters.
     *
     * @override
     */
    _getValueForSmsCounts: function () {
        const value = this._super.apply(this, arguments);
        if (this.linkReplacementsPlaceholders) {
            const replaced = value.replaceAll(TEXT_URL_REGEX, this.linkReplacementsPlaceholders.link);
            this.noticeLinksReplaced = replaced !== value;
            return replaced;
        }
        return value;
    },
    _onClick: function() {
        this._super.apply(this, arguments);
        if (this.linkReplacementsPlaceholders) {  // covers mailing and edit mode
            this._updateSMSInfo();
        }
    }
});

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
        <field name="inherit_id" ref="mass_mailing.mailing_contact_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="mobile"/>
                <field name="phone_sanitized" invisible="1"/>
            </xpath>
            <xpath expr="//filter[@name='filter_blacklisted']" position="attributes">
                <attribute name="domain">['|', ('is_blacklisted','=',True), ('phone_sanitized_blacklisted','=',True)]</attribute>
            </xpath>
            <xpath expr="//filter[@name='filter_valid_email_recipient']" position="after">
                <separator/>
                <filter string="Valid SMS Recipients"
                    name="filter_valid_sms_recipient"
                    domain="[('opt_out', '=', False), ('phone_sanitized_blacklisted', '=', False), ('phone_sanitized', '!=', False)]"
                    invisible="not context.get('default_list_ids')"/>
            </xpath>
            <xpath expr="//filter[@name='filter_not_email_bl']" position="after">
                <separator/>
                <filter string="Exclude Blacklisted Phone"
                    name="filter_not_phone_bl"
                    domain="[('phone_sanitized_blacklisted', '=', False)]"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_contact_view_tree" model="ir.ui.view">
        <field name="name">mailing.contact.view.tree.inherit.sms</field>
        <field name="model">mailing.contact</field>
        <field name="inherit_id" ref="mass_mailing.mailing_contact_view_tree"/>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='is_blacklisted']" position="after">
                <field name="mobile" class="o_force_ltr"/>
                <field name="phone_sanitized" invisible="1"/>
                <field name="phone_sanitized_blacklisted"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_contact_view_form" model="ir.ui.view">
        <field name="name">mailing.contact.view.form.inherit.sms</field>
        <field name="model">mailing.contact</field>
        <field name="inherit_id" ref="mass_mailing.mailing_contact_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='email_details']" position="after">
                <label for="mobile" class="oe_inline"/>
                <div class="o_row o_row_readonly" name="phone_details">
                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                        type="object" context="{'default_phone': mobile}" groups="base.group_user"
                        attrs="{'invisible': [('mobile_blacklisted', '=', False)]}"/>
                    <field name="mobile" widget="phone" options="{'enable_sms': True}"/>
                    <field name="phone_sanitized" invisible="1"/>
                    <field name="mobile_blacklisted" invisible="1"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="mailing_contact_view_kanban" model="ir.ui.view">
        <field name="name">mailing.contact.view.kanban.inherit.sms</field>
        <field name="model">mailing.contact</field>
        <field name="inherit_id" ref="mass_mailing.mailing_contact_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="after">
                <field name="mobile" widget="phone"/>
                <field name="phone_sanitized" invisible="1"/>
            </xpath>
            <xpath expr="//t[@t-esc='record.email.value']" position="after">
                <t t-esc="record.mobile.value"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_contact_action_sms" model="ir.actions.act_window">
        <field name="name">Mailing List Contacts</field>
        <field name="res_model">mailing.contact</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'mailing_sms': True, 'search_default_filter_not_phone_bl': 1, }</field>
        <field name="view_id" ref="mailing_contact_view_tree"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a mailing contact
            </p><p>
                Mailing contacts allow you to separate your marketing audience from your contact directory.
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
    <record id="mailing_list_view_kanban" model="ir.ui.view">
        <field name="name">mailing.list.view.kanban.inherit.mass.mailing.sms</field>
        <field name="model">mailing.list</field>
        <field name="inherit_id" ref="mass_mailing.mailing_list_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='contact_count_email']" position="after">
                <field name="contact_count_sms"/>
            </xpath>
            <xpath expr="//div[hasclass('o_mass_mailing_kanban_contact_links')]" position="inside">
              <a name="action_view_contacts_sms" type="object">
                  <span >Valid SMS Recipients</span>
                  <span t-esc="record.contact_count_sms.value" class="ml-3"/>
              </a>
            </xpath>
        </field>
    </record>

    <record id="mailing_list_action_sms" model="ir.actions.act_window">
        <field name="name">Mailing Lists</field>
        <field name="res_model">mailing.list</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="context">{'mailing_sms': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a Mailing List
          </p><p>
            No need to import mailing lists, you can send SMS Text Messages to contacts saved in other Odoo apps.
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
            <xpath expr="//button[@name='action_launch']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', ('mailing_type', '!=', 'mail'), ('state', 'in', ('in_queue', 'sending', 'done'))]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_schedule']" position="before">
                <field name="sms_force_send" invisible="1"/>
                <field name="schedule_type" invisible="1"/>
                <button name="action_put_in_queue" type="object"
                    string="Send" class="oe_highlight" data-hotkey="v"
                    attrs="{'invisible': ['|', '|', ('mailing_type', '=', 'mail'), ('state', 'in', ('in_queue', 'sending', 'done')), ('sms_force_send', '=', True)]}"
                    confirm="This will send SMS to all recipients. Do you still want to proceed ?"/>
                <button name="action_send_mail" type="object"
                    string="Send Now" class="oe_highlight" data-hotkey="g"
                    attrs="{'invisible': ['|', '|', '|', ('mailing_type', '=', 'mail'), ('state', 'in', ('in_queue', 'done')), ('schedule_type', '=', 'scheduled'), ('sms_force_send', '!=', True)]}"
                    confirm="This will send SMS to all recipients now. Do you still want to proceed ?"/>
            </xpath>
            <!-- Headers / Warnings -->
            <xpath expr="//header" position="after">
                <field name="sms_has_insufficient_credit" invisible="1"/>
                <div class="alert alert-warning text-center o-hidden-ios" attrs="{'invisible': [('sms_has_insufficient_credit', '=', False)]}" role="alert">
                    <button class="btn-link py-0"
                            name="action_buy_sms_credits"
                            type="object">
                        <strong>
                            It appears you don't have enough IAP credits. Click here to buy credits.
                        </strong>
                    </button>
                </div>
                <field name="sms_has_unregistered_account" invisible="1"/>
                <div class="alert alert-warning text-center o-hidden-ios" attrs="{'invisible': [('sms_has_unregistered_account', '=', False)]}" role="alert">
                    <button class="btn-link py-0"
                            name="action_buy_sms_credits"
                            type="object">
                        <strong>
                            It appears your SMS account is not registered. Click here to set up your account.
                        </strong>
                    </button>
                </div>
            </xpath>
            <xpath expr="//span[@name='canceled_text']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='canceled_text']" position="after">
                <span name="canceled_text_sms" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">SMS Text Message have been canceled and will not be sent.</span>
            </xpath>
            <xpath expr="//span[@name='scheduled_text']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='scheduled_text']" position="after">
                <span name="scheduled_text_sms" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">SMS Text Message are in queue and will be sent soon.</span>
            </xpath>
            <xpath expr="//span[@name='sent']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='sent']" position="after">
                <span name="sent_sms" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">SMS Text Message have been sent.</span>
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
            <xpath expr="//button[@name='action_view_opened']" position="attributes">
                <attribute name="attrs">{'invisible': ['|',('mailing_type', '!=', 'mail'),('state', 'in', ('draft','test'))]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_view_replied']" position="attributes">
                <attribute name="attrs">{'invisible': ['|',('mailing_type', '!=', 'mail'),('state', 'in', ('draft','test'))]}</attribute>
            </xpath>
            <!-- Form -->
            <xpath expr="//field[@name='subject']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')], 'readonly': [('state', 'in', ('sending', 'done'))], 'required': [('mailing_type', '=', 'mail')]}</attribute>
                <!-- overrided in xml view to prevent remaining helper changes (on mass_mailing module) when mass_mailing_sms uninstalled-->
                <attribute name="help">For an Email, Subject your Recipients will see in their inbox.
                    For an SMS Text Message, internal Title of the Message.</attribute>
            </xpath>
            <xpath expr="//field[@name='subject']" position="after">
                <field class="text-break" name="sms_subject" string="Title" placeholder="e.g. Black Friday SMS coupon" attrs="{'invisible': [('mailing_type', '!=', 'sms')], 'readonly': [('state', 'in', ('sending', 'done'))], 'required': [('mailing_type', '=', 'sms')]}"/>
            </xpath>
            <xpath expr="//field[@name='preview']" position="attributes">
                <attribute name="attrs">{'readonly': [('state', 'in', ('sending', 'done'))], 'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//page[@name='mail_body']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//page[@name='mail_body']" position="after">
                <page string="SMS Content" name="sms_body" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">
                    <field name="body_plaintext" widget="sms_widget"
                        attrs="{'required': [('mailing_type', '=', 'sms')], 'readonly': [('state', 'in', ('sending', 'done'))]}"
                        options='{"enable_emojis": True}'/>
                </page>
            </xpath>
            <xpath expr="//page[@name='settings']/group/group[1]" position="after">
                <group string="Options" attrs="{'invisible': [('mailing_type', '!=', 'sms')]}">
                    <field name="sms_allow_unsubscribe" attrs="{'invisible': [('mailing_type', '!=', 'sms')], 'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                </group>
            </xpath>

            <xpath expr="//field[@name='contact_list_ids']" position="attributes">
                <attribute name="context">
                    {'mailing_sms' : context.get('mailing_sms'),
                    'form_view_ref': 'mass_mailing.mailing_list_view_form_simplified'}
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
                <attribute name="attrs">{'invisible': ['|', ('mailing_type', '!=', 'mail'),
                    ('mail_server_available', '=', False)], 'readonly': [('state', 'in', ('sending', 'done'))]}</attribute>
            </xpath>
            <!-- A/B Testing -->
            <xpath expr="//field[@name='ab_testing_winner_selection']" position="after">
                <label for="ab_testing_sms_winner_selection" string="Winner Selection"
                    attrs="{'invisible': ['|', ('ab_testing_enabled', '=', False), ('mailing_type', '!=', 'sms')]}"/>
                <field name="ab_testing_sms_winner_selection" nolabel="1"
                    attrs="{'required': [('ab_testing_enabled', '=', True), ('mailing_type', '=', 'sms')], 'invisible': ['|', ('ab_testing_enabled', '=', False), ('mailing_type', '!=', 'sms')], 'readonly': [('state', '!=', 'draft')]}"/>
            </xpath>
            <xpath expr="//field[@name='ab_testing_schedule_datetime']" position="replace">
                 <field name="ab_testing_schedule_datetime"
                        attrs="{'required': [('ab_testing_enabled', '=', True), ('ab_testing_winner_selection', '!=', 'manual'), ('ab_testing_sms_winner_selection', '!=', 'manual')], 'readonly': ['|', ('ab_testing_enabled', '=', False), ('state', '!=', 'draft')], 'invisible': ['|', '|', ('ab_testing_enabled', '=', False), ('ab_testing_winner_selection', '=', 'manual'), ('ab_testing_sms_winner_selection', '=', 'manual')]}"/>
            </xpath>
            <xpath expr="//span[@name='ab_test_manual']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', ('ab_testing_winner_selection', '!=', 'manual'),
                    ('ab_testing_sms_winner_selection', '!=', 'manual')]}</attribute>
            </xpath>
            <xpath expr="//span[@name='ab_test_auto']" position="attributes">
                <attribute name="attrs">{'invisible': [('ab_testing_winner_selection', '=', 'manual'),
                    ('ab_testing_sms_winner_selection', '=', 'manual')]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_select_as_winner']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', ('ab_testing_completed', '!=', False), '|',
                    ('ab_testing_winner_selection', '=', 'manual'),
                    ('ab_testing_sms_winner_selection', '=', 'manual')]}</attribute>
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
                <field name="sms_has_unregistered_account"/>
            </xpath>
            <xpath expr="//div[@name='stat_opened']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//div[@name='stat_replied']" position="attributes">
                <attribute name="attrs">{'invisible': [('mailing_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//div[@name='div_responsible_avatar']" position="after">
                <div class="alert alert-warning mb-0 mt-3 o-hidden-ios" role="alert" attrs="{'invisible': [('sms_has_insufficient_credit', '=', False)]}">
                    <a name="action_buy_sms_credits" type="object">Insufficient credits</a>
                </div>
                <div class="alert alert-warning mb-0 mt-3" role="alert" attrs="{'invisible': [('sms_has_unregistered_account', '=', False)]}">
                    <a name="action_buy_sms_credits" type="object">Unregistered account</a>
                </div>
            </xpath>
        </field>
    </record>

    <record id="mailing_mailing_view_tree_sms" model="ir.ui.view">
        <field name="name">mailing.mailing.view.tree.sms</field>
        <field name="model">mailing.mailing</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="SMS Marketing" sample="1" decoration-info="state == 'draft'">
                <field name="subject"/>
                <field name="mailing_type" invisible="1"/>
                <field name="mailing_model_id" string="Recipients"/>
                <field name="user_id" widget="many2one_avatar_user"/>
                <field name="schedule_date" string="Scheduled" widget="remaining_days"/>
                <field name="sent_date" widget="date"/>
                <field name="state" decoration-info="state == 'draft' or state == 'in_queue'" decoration-success="state == 'sending' or state == 'done'" widget="badge"/>
                <field name="campaign_id" string="Campaign" groups="mass_mailing.group_mass_mailing_campaign"/>
                <field name="sent"/>
                <field name="clicked"/>
                <field name="bounced"/>
            </tree>
        </field>
    </record>

    <record id="mailing_mailing_action_sms" model="ir.actions.act_window">
        <field name="name">SMS Marketing</field>
        <field name="res_model">mailing.mailing</field>
        <field name="view_mode">kanban,tree,form,calendar,graph</field>
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
            Create a SMS Marketing Mailing
          </p><p>
            Write an appealing SMS Text Message, define recipients and track its results.
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
        sequence="120"
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
        name="Mailing Lists"
        parent="mass_mailing_sms_menu_root"
        sequence="2"
        groups="mass_mailing.group_mass_mailing_user"/>
    <!-- SMS Marketing / Contacts Lists / Contacts Lists -->
    <menuitem id="mailing_list_menu_sms"
        name="Mailing Lists"
        parent="mass_mailing_sms_menu_contacts"
        sequence="1"
        groups="mass_mailing.group_mass_mailing_user"/>
        <!-- SMS Marketing / Contacts Lists / Contacts -->
    <menuitem id="mailing_contact_menu_sms"
        name="Mailing List Contacts"
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
        name="Blacklisted Phone Numbers"
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
                <field name="mass_mailing_id"/>
                <field name="sms_number"/>
                <field name="sent_datetime"/>
                <field name="links_click_datetime"/>
                <field name="trace_status" widget="badge"/>
                <field name="failure_type" optional="show"/>
                <field name="open_datetime" optional="hide"/>
                <field name="reply_datetime" optional="hide"/>
                <button name="action_view_contact" type="object"
                        string="Open Recipient" icon="fa-user"/>
            </tree>
        </field>
    </record>

    <record id="mailing_trace_view_form" model="ir.ui.view">
        <field name="name">mailing.trace.view.form.inherit.sms</field>
        <field name="model">mailing.trace</field>
        <field name="inherit_id" ref="mass_mailing.mailing_trace_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='mail_mail_id_int']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='message_id']" position="attributes">
                <attribute name="attrs">{'invisible': [('trace_type', '!=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='email']" position="after">
                <field name="sms_number" attrs="{'invisible': [('trace_type', '!=', 'sms')]}"/>
            </xpath>
            <xpath expr="//field[@name='message_id']" position="after">
                <field name="sms_sms_id_int" string="SMS ID"
                    attrs="{'invisible': [('trace_type', '!=', 'sms')]}"
                    groups="base.group_no_one"/>
                <field name="sms_code" attrs="{'invisible': [('trace_type', '!=', 'sms')]}"
                    groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>

    <record id="mailing_trace_view_form_sms" model="ir.ui.view">
        <field name="name">mailing.trace.view.form.sms</field>
        <field name="model">mailing.trace</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <form string="SMS Trace" create="0" edit="0">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_contact"
                                type="object" icon="fa-user" class="oe_stat_button">
                            <span widget="statinfo">Open Recipient</span>
                        </button>
                    </div>
                    <div class="alert alert-info text-center" attrs="{'invisible': [('trace_status', '!=', 'error')]}" role="alert">
                        <strong>This SMS could not be sent.</strong>
                    </div>
                    <div class="alert alert-info text-center" attrs="{'invisible': [('trace_status', '!=', 'bounce')]}" role="alert">
                        <strong>This number appears to be invalid.</strong>
                    </div>
                    <group>
                        <group string="Status">
                            <field name="trace_status"/>
                            <field name="failure_type" attrs="{'invisible' : [('failure_type', '=', False)]}"/>
                            <field name="sent_datetime" attrs="{'invisible' : [('sent_datetime', '=', False)]}"/>
                            <field name="links_click_datetime" attrs="{'invisible' : [('links_click_datetime', '=', False)]}"/>
                            <field name="open_datetime" attrs="{'invisible' : [('open_datetime', '=', False)]}"/>
                            <field name="reply_datetime" attrs="{'invisible' : [('reply_datetime', '=', False)]}"/>
                        </group>
                        <group string="Mailing">
                            <field name="trace_type" invisible="1"/>
                            <field name="sms_number"/>
                            <field name="mass_mailing_id"/>
                            <field name="sms_sms_id_int" string="SMS ID" groups="base.group_no_one"/>
                            <field name="sms_code" groups="base.group_no_one"/>
                        </group>
                        <group string="Marketing">
                            <field name="campaign_id" groups="mass_mailing.group_mass_mailing_campaign"/>
                            <field name="medium_id"/>
                            <field name="source_id"/>
                            <field name="sms_sms_id_int" groups="base.group_no_one"/>
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
                <page string="SMS" name="sms" attrs="{'invisible': [('mailing_sms_count', '=', 0)]}">
                    <group>
                        <field name="mailing_sms_ids" readonly="1" nolabel="1">
                            <tree>
                                <field name="subject"/>
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
                    <t t-out="record.mailing_sms_count.raw_value"/> SMS
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

from odoo import fields, models, _
from odoo.addons.phone_validation.tools import phone_validation


class MassSMSTest(models.TransientModel):
    _name = 'mailing.sms.test'
    _description = 'Test SMS Mailing'

    def _default_numbers(self):
        return self.env.user.partner_id.phone_sanitized or ""

    numbers = fields.Text(string='Number(s)', required=True,
                          default=_default_numbers, help='Carriage-return-separated list of phone numbers')
    mailing_id = fields.Many2one('mailing.mailing', string='Mailing', required=True, ondelete='cascade')

    def action_send_sms(self):
        self.ensure_one()

        numbers = [number.strip() for number in self.numbers.splitlines()]
        sanitize_res = phone_validation.phone_sanitize_numbers_w_record(numbers, self.env.user)
        sanitized_numbers = [info['sanitized'] for info in sanitize_res.values() if info['sanitized']]
        invalid_numbers = [number for number, info in sanitize_res.items() if info['code']]

        record = self.env[self.mailing_id.mailing_model_real].search([], limit=1)
        body = self.mailing_id.body_plaintext
        if record:
            # Returns a proper error if there is a syntax error with qweb
            body = self.env['mail.render.mixin']._render_template(body, self.mailing_id.mailing_model_real, record.ids)[record.id]

        # res_id is used to map the result to the number to log notifications as IAP does not return numbers...
        # TODO: clean IAP to make it return a clean dict with numbers / use custom keys / rename res_id to external_id
        sent_sms_list = self.env['sms.api']._send_sms_batch([{
            'res_id': number,
            'number': number,
            'content': body,
        } for number in sanitized_numbers])

        error_messages = {}
        if any(sent_sms.get('state') != 'success' for sent_sms in sent_sms_list):
            error_messages = self.env['sms.api']._get_sms_api_error_messages()

        notification_messages = []
        if invalid_numbers:
            notification_messages.append(_('The following numbers are not correctly encoded: %s',
                ', '.join(invalid_numbers)))

        for sent_sms in sent_sms_list:
            if sent_sms.get('state') == 'success':
                notification_messages.append(
                    _('Test SMS successfully sent to %s', sent_sms.get('res_id')))
            elif sent_sms.get('state'):
                notification_messages.append(
                    _('Test SMS could not be sent to %s:<br>%s',
                    sent_sms.get('res_id'),
                    error_messages.get(sent_sms['state'], _("An error occurred.")))
                )

        if notification_messages:
            self.mailing_id._message_log(body='<ul>%s</ul>' % ''.join(
                ['<li>%s</li>' % notification_message for notification_message in notification_messages]
            ))

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
                    Send a sample SMS for testing purpose to the numbers below (carriage-return-separated list).
                </p>
                <group>
                    <field name="numbers" placeholder="+32 495 85 85 77&#10;+33 545 55 55 55"/>
                    <field name="mailing_id" invisible="1"/>
                </group>
                <footer>
                    <button string="Send" name="action_send_sms" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn btn-secondary" special="cancel" data-hotkey="z"/>
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

from odoo import api, fields, models, _


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
        return werkzeug.urls.url_join(
            self.get_base_url(),
            '/sms/%s/%s' % (self.mailing_id.id, trace_code)
        )

    @api.model
    def _get_unsubscribe_info(self, url):
        return _('STOP SMS : %(unsubscribe_url)s', unsubscribe_url=url)

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
            trace_values['failure_type'] = sms_values['failure_type']
            trace_values['trace_status'] = 'error'
        elif sms_values['state'] == 'canceled':
            trace_values['failure_type'] = sms_values['failure_type']
            trace_values['trace_status'] = 'cancel'
        else:
            if self.mass_sms_allow_unsubscribe:
                stop_sms = self._get_unsubscribe_info(self._get_unsubscribe_url(record.id, trace_code, sms_values['number']))
                sms_values['body'] = '%s\n%s' % (sms_values['body'] or '', stop_sms)
        return trace_values

    def _get_optout_record_ids(self, records, recipients_info):
        """ Fetch opt-out records based on mailing. """
        res = super(SMSComposer, self)._get_optout_record_ids(records, recipients_info)
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
                body = self.env['mail.render.mixin'].sudo()._shorten_links_text(body, tracker_values)
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

