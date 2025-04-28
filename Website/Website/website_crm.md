# Odoo Module: website_crm

Category: Website/Website

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
    'name': 'Contact Form',
    'category': 'Website/Website',
    'sequence': 54,
    'summary': 'Generate leads from a contact form',
    'version': '2.1',
    'description': """
Generate leads or opportunities in the CRM app from a contact form published on the Contact us page of your website.
This form can be customized thanks to the *Form Builder* module (available in Odoo Enterprise).

This module includes contact phone and mobile numbers validation.""",
    'depends': ['website_form', 'crm'],
    'data': [
        'security/ir.model.access.csv',
        'data/website_crm_data.xml',
        'views/website_crm_lead_views.xml',
        'views/website_crm_templates.xml',
        'views/res_config_settings_views.xml',
        'views/website_visitor_views.xml',
    ],
    'qweb': ['static/src/xml/*.xml'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.website_form.controllers.main import WebsiteForm


class WebsiteForm(WebsiteForm):

    def _get_country(self):
        country_code = request.session.geoip and request.session.geoip.get('country_code') or False
        if country_code:
            return request.env['res.country'].sudo().search([('code', '=', country_code)], limit=1)
        return request.env['res.country']

    def _get_phone_fields_to_validate(self):
        return ['phone', 'mobile']

    # Check and insert values from the form on the model <model> + validation phone fields
    def _handle_website_form(self, model_name, **kwargs):
        model_record = request.env['ir.model'].sudo().search([('model', '=', model_name), ('website_form_access', '=', True)])
        if model_record and hasattr(request.env[model_name], 'phone_format'):
            try:
                data = self.extract_data(model_record, request.params)
            except:
                # no specific management, super will do it
                pass
            else:
                record = data.get('record', {})
                phone_fields = self._get_phone_fields_to_validate()
                country = request.env['res.country'].browse(record.get('country_id'))
                contact_country = country.exists() and country or self._get_country()
                for phone_field in phone_fields:
                    if not record.get(phone_field):
                        continue
                    number = record[phone_field]
                    fmt_number = request.env[model_name].phone_format(number, contact_country)
                    request.params.update({phone_field: fmt_number})

        if model_name == 'crm.lead' and not request.params.get('state_id'):
            geoip_country_code = request.session.get('geoip', {}).get('country_code')
            geoip_state_code = request.session.get('geoip', {}).get('region')
            if geoip_country_code and geoip_state_code:
                state = request.env['res.country.state'].search([('code', '=', geoip_state_code), ('country_id.code', '=', geoip_country_code)])
                if state:
                    request.params['state_id'] = state.id
        return super(WebsiteForm, self)._handle_website_form(model_name, **kwargs)

    def insert_record(self, request, model, values, custom, meta=None):
        is_lead_model = model.model == 'crm.lead'
        if is_lead_model:
            if 'company_id' not in values:
                values['company_id'] = request.website.company_id.id
            lang = request.context.get('lang', False)
            values['lang_id'] = values.get('lang_id') or request.env['res.lang']._lang_get_id(lang)

        result = super(WebsiteForm, self).insert_record(request, model, values, custom, meta=meta)

        if is_lead_model:
            visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
            if visitor_sudo and result:
                lead_sudo = request.env['crm.lead'].browse(result).sudo()
                if lead_sudo.exists():
                    vals = {'lead_ids': [(4, result)]}
                    if not visitor_sudo.lead_ids and not visitor_sudo.partner_id:
                        vals['name'] = lead_sudo.contact_name
                    visitor_sudo.write(vals)
        return result

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\website_crm_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="crm.model_crm_lead" model="ir.model">
            <field name="website_form_key">create_lead</field>
            <field name="website_form_default_field_id" ref="crm.field_crm_lead__description" />
            <field name="website_form_access">True</field>
            <field name="website_form_label">Create an Opportunity</field>
        </record>
        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>crm.lead</value>
            <value eval="[
                'contact_name',
                'description',
                'email_from',
                'name',
                'partner_name',
                'phone',
                'team_id',
                'user_id',
            ]"/>
        </function>
    </data>
    <data noupdate="1">
        <record id="action_open_website" model="ir.actions.act_url">
            <field name="name">Website Contact Form</field>
            <field name="target">self</field>
            <field name="url">/contactus</field>
        </record>

        <record id="base.open_menu" model="ir.actions.todo">
            <field name="action_id" ref="action_open_website"/>
            <field name="state">open</field>
        </record>
    </data>
</odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, SUPERUSER_ID


class Lead(models.Model):
    _inherit = 'crm.lead'

    visitor_ids = fields.Many2many('website.visitor', string="Web Visitors")
    visitor_page_count = fields.Integer('# Page Views', compute="_compute_visitor_page_count")

    @api.depends('visitor_ids.page_ids')
    def _compute_visitor_page_count(self):
        mapped_data = {}
        if self.ids:
            self.flush(['visitor_ids'])
            sql = """ SELECT l.id as lead_id, count(*) as page_view_count
                        FROM crm_lead l
                        JOIN crm_lead_website_visitor_rel lv ON l.id = lv.crm_lead_id
                        JOIN website_visitor v ON v.id = lv.website_visitor_id
                        JOIN website_track p ON p.visitor_id = v.id
                        WHERE l.id in %s AND v.active = TRUE
                        GROUP BY l.id"""
            self.env.cr.execute(sql, (tuple(self.ids),))
            page_data = self.env.cr.dictfetchall()
            mapped_data = {data['lead_id']: data['page_view_count'] for data in page_data}
        for lead in self:
            lead.visitor_page_count = mapped_data.get(lead.id, 0)

    def action_redirect_to_page_views(self):
        visitors = self.visitor_ids
        action = self.env["ir.actions.actions"]._for_xml_id("website.website_visitor_page_action")
        action['domain'] = [('visitor_id', 'in', visitors.ids)]
        # avoid grouping if only few records
        if len(visitors.website_track_ids) > 15 and len(visitors.website_track_ids.page_id) > 1:
            action['context'] = {'search_default_group_by_page': '1'}
        return action

    def _merge_data(self, fields):
        merged_data = super(Lead, self)._merge_data(fields)
        # add all the visitors from all lead to merge
        merged_data['visitor_ids'] = [(6, 0, self.visitor_ids.ids)]
        return merged_data

    def website_form_input_filter(self, request, values):
        values['medium_id'] = values.get('medium_id') or \
                              self.default_get(['medium_id']).get('medium_id') or \
                              self.sudo().env.ref('utm.utm_medium_website').id
        values['team_id'] = values.get('team_id') or \
                            request.website.crm_default_team_id.id
        values['user_id'] = values.get('user_id') or \
                            request.website.crm_default_user_id.id
        if values.get('team_id'):
            values['type'] = 'lead' if self.env['crm.team'].sudo().browse(values['team_id']).use_leads else 'opportunity'
        else:
            values['type'] = 'lead' if self.with_user(SUPERUSER_ID).env['res.users'].has_group('crm.group_use_lead') else 'opportunity'

        return values


class Website(models.Model):
    _inherit = 'website'

    def _get_crm_default_team_domain(self):
        if not self.env.user.has_group('crm.group_use_lead'):
            return [('use_opportunities', '=', True)]
        return [('use_leads', '=', True)]

    crm_default_team_id = fields.Many2one(
        'crm.team', string='Default Sales Teams',
        default=lambda self: self.env['crm.team'].search([], limit=1),
        domain=lambda self: self._get_crm_default_team_domain(),
        help='Default Sales Team for new leads created through the Contact Us form.')
    crm_default_user_id = fields.Many2one(
        'res.users', string='Default Salesperson', domain=[('share', '=', False)],
        help='Default salesperson for new leads created through the Contact Us form.')

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def _get_crm_default_team_domain(self):
        if not self.env.user.has_group('crm.group_use_lead'):
            return [('use_opportunities', '=', True)]
        return [('use_leads', '=', True)]

    crm_default_team_id = fields.Many2one(
        'crm.team', string='Default Sales Team', related='website_id.crm_default_team_id', readonly=False,
        domain=lambda self: self._get_crm_default_team_domain(),
        help='Default Sales Team for new leads created through the Contact Us form.')
    crm_default_user_id = fields.Many2one(
        'res.users', string='Default Salesperson', related='website_id.crm_default_user_id', domain=[('share', '=', False)], readonly=False,
        help='Default salesperson for new leads created through the Contact Us form.')

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    lead_ids = fields.Many2many('crm.lead', string='Leads', groups="sales_team.group_sale_salesman")
    lead_count = fields.Integer('# Leads', compute="_compute_lead_count", groups="sales_team.group_sale_salesman")

    @api.depends('lead_ids')
    def _compute_lead_count(self):
        for visitor in self:
            visitor.lead_count = len(visitor.lead_ids)

    @api.depends('partner_id.email_normalized', 'partner_id.mobile', 'lead_ids.email_normalized', 'lead_ids.mobile')
    def _compute_email_phone(self):
        super(WebsiteVisitor, self)._compute_email_phone()
        self.flush()

        left_visitors = self.filtered(lambda visitor: not visitor.email or not visitor.mobile)
        leads = left_visitors.mapped('lead_ids').sorted('create_date', reverse=True)
        visitor_to_lead_ids = dict((visitor.id, visitor.lead_ids.ids) for visitor in left_visitors)

        for visitor in left_visitors:
            visitor_leads = leads.filtered(lambda lead: lead.id in visitor_to_lead_ids[visitor.id])
            if not visitor.email:
                visitor.email = next((lead.email_normalized for lead in visitor_leads if lead.email_normalized), False)
            if not visitor.mobile:
                visitor.mobile = next((lead.mobile or lead.phone for lead in visitor_leads if lead.mobile or lead.phone), False)

    def _check_for_message_composer(self):
        check = super(WebsiteVisitor, self)._check_for_message_composer()
        if not check and self.lead_ids:
            sorted_leads = self.lead_ids._sort_by_confidence_level(reverse=True)
            partners = sorted_leads.mapped('partner_id')
            if not partners:
                main_lead = self.lead_ids[0]
                main_lead.handle_partner_assignment(create_missing=True)
                self.partner_id = main_lead.partner_id.id
            return True
        return check

    def _prepare_message_composer_context(self):
        if not self.partner_id and self.lead_ids:
            sorted_leads = self.lead_ids._sort_by_confidence_level(reverse=True)
            lead_partners = sorted_leads.mapped('partner_id')
            partner = lead_partners[0] if lead_partners else False
            if partner:
                return {
                    'default_model': 'crm.lead',
                    'default_res_id': sorted_leads[0].id,
                    'default_partner_ids': partner.ids,
                }
        return super(WebsiteVisitor, self)._prepare_message_composer_context()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import res_config_settings
from . import website_visitor

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_website_visitor_salesman,access_website_visitor_salesman,model_website_visitor,sales_team.group_sale_salesman,1,0,0,0
access_website_track_salesman,access_website_track_salesman,website.model_website_track,sales_team.group_sale_salesman,1,0,0,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V36.068L12.746 22l2.101-1h17.128L39 14h10v2.968l-4.079 4.3v6.773L52 21l7 2-.691 19.842L37.604 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M52 44h3a2 2 0 0 0 2-2V27a2 2 0 0 0-2-2h-3v-2h4a3 3 0 0 1 3 3v17a3 3 0 0 1-3 3h-4v-2zM36 25H16a2 2 0 0 0-2 2v15a2 2 0 0 0 2 2h20v2H15a3 3 0 0 1-3-3V26a3 3 0 0 1 3-3h21v2zm7 25V19h-4v-3h10v3h-4v31h4v3H39v-3h4zM30.468 33.075c.097-.07.24-.005.24.106v4.606c0 .597-.533 1.08-1.19 1.08H19.19c-.657 0-1.191-.483-1.191-1.08v-4.604c0-.113.141-.176.24-.106.557.392 1.294.89 3.826 2.559.524.347 1.407 1.076 2.288 1.072.887.007 1.788-.739 2.291-1.072 2.532-1.67 3.267-2.17 3.823-2.561zm-6.114 2.691c-.575.01-1.405-.658-1.822-.932-3.293-2.17-3.544-2.36-4.304-2.9A.526.526 0 0 1 18 31.51v-.428c0-.597.534-1.081 1.191-1.081h10.326c.658 0 1.192.484 1.192 1.081v.428a.523.523 0 0 1-.229.426c-.76.54-1.01.73-4.304 2.899-.417.274-1.246.941-1.822.932z" opacity=".3"/><path fill="#FFF" d="M52 42h3a2 2 0 0 0 2-2V25a2 2 0 0 0-2-2h-3v-2h4a3 3 0 0 1 3 3v17a3 3 0 0 1-3 3h-4v-2zM36 23H16a2 2 0 0 0-2 2v15a2 2 0 0 0 2 2h20v2H15a3 3 0 0 1-3-3V24a3 3 0 0 1 3-3h21v2zm7 25V17h-4v-3h10v3h-4v31h4v3H39v-3h4zM30.468 31.075c.097-.07.24-.005.24.106v4.606c0 .597-.533 1.08-1.19 1.08H19.19c-.657 0-1.191-.483-1.191-1.08v-4.604c0-.113.141-.176.24-.106.557.392 1.294.89 3.826 2.559.524.347 1.407 1.076 2.288 1.072.887.007 1.788-.739 2.291-1.072 2.532-1.67 3.267-2.17 3.823-2.561zm-6.114 2.691c-.575.01-1.405-.658-1.822-.932-3.293-2.17-3.544-2.36-4.304-2.9A.526.526 0 0 1 18 29.51v-.428c0-.597.534-1.081 1.191-1.081h10.326c.658 0 1.192.484 1.192 1.081v.428a.523.523 0 0 1-.229.426c-.76.54-1.01.73-4.304 2.899-.417.274-1.246.941-1.822.932z"/></g></g></svg>
```

## File: static\src\js\website_crm_editor.js

```javascript
odoo.define('website_crm.form', function (require) {
'use strict';

var core = require('web.core');
var FormEditorRegistry = require('website_form.form_editor_registry');

var _t = core._t;

FormEditorRegistry.add('create_lead', {
    formFields: [{
        type: 'char',
        required: true,
        name: 'contact_name',
        string: 'Your Name',
    }, {
        type: 'tel',
        name: 'phone',
        string: 'Phone Number',
    }, {
        type: 'email',
        required: true,
        name: 'email_from',
        string: 'Your Email',
    }, {
        type: 'char',
        required: true,
        name: 'partner_name',
        string: 'Your Company',
    }, {
        type: 'char',
        modelRequired: true,
        name: 'name',
        string: 'Subject',
    }, {
        type: 'text',
        required: true,
        name: 'description',
        string: 'Your Question',
    }],
    fields: [{
        name: 'team_id',
        type: 'many2one',
        relation: 'crm.team',
        domain: [['use_opportunities', '=', true]],
        string: _t('Sales Team'),
        title: _t('Assign leads/opportunities to a sales team.'),
    }, {
        name: 'user_id',
        type: 'many2one',
        relation: 'res.users',
        domain: [['share', '=', false]],
        string: _t('Salesperson'),
        title: _t('Assign leads/opportunities to a salesperson.'),
    }],
});

});

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.crm</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="webmaster_settings" position="after">
                <h2>Communication</h2>
                <div class="row mt16 o_settings_container" id="communication_settings">
                    <div class="col-12 col-lg-6 o_setting_box" id="contact_form_install_setting" title="New messages are managed as leads or opportunities in your CRM app.">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Contact Form</span>
                            <div class="text-muted" id="msg_contact_form_install_setting">
                                Add a contact form in the <a href="/contactus">Contact Us</a> page
                            </div>
                            <div class="row">
                                <label class="col-lg-3 o_light_label" string="Sales Team" for="crm_default_team_id"/>
                                <field name="crm_default_team_id" class="oe_inline"/>
                            </div>
                            <div class="content-group mb16" attrs="{'invisible': [('crm_default_team_id', '!=', False)]}">
                                <div class="text-muted" id="msg_sale_team_setting">
                                    Please set a Sales Team for the website. Otherwise you can't follow related events.
                                </div>
                            </div>

                            <div class="row">
                                <label class="col-lg-3 o_light_label" string="Salesperson" for="crm_default_user_id"/>
                                <field name="crm_default_user_id" class="oe_inline"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\website_crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="crm_lead_view_form" model="ir.ui.view">
        <field name="name">crm.lead.view.form.inherit.website.crm</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="action_redirect_to_page_views" type="object" class="oe_stat_button" icon="fa-tags"
                        attrs="{'invisible': [('visitor_page_count', '=', 0)]}">
                    <field name="visitor_page_count" widget="statinfo" string="Page views"/>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\website_crm_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="contactus_form" name="Contact Form (Opportunity)" inherit_id="website_form.contactus_form" customize_show="True">
            <xpath expr="//span[@data-for='contactus_form']" position="attributes">
                <attribute name="t-att-data-values">{
                    'contact_name': request.params.get('contact_name', ''),
                    'phone': request.params.get('phone', ''),
                    'email_from': request.params.get('email_from', ''),
                    'partner_name': request.params.get('partner_name', ''),
                    'name': request.params.get('name', ''),
                    'description': request.params.get('description', ''),
                }</attribute>
            </xpath>
            <xpath expr="//div[@id='contactus_section']" position="replace">
                <div id="contactus_section">
                    <section class="s_website_form" data-vcss="001" data-snippet="s_website_form">
                        <div class="container">
                            <form id="contactus_form" action="/website_form/" method="post" enctype="multipart/form-data" data-model_name="crm.lead" data-success-mode="redirect" data-success-page="/contactus-thank-you" class="o_mark_required" data-mark="*" data-editable-form="false">
                                <div class="s_website_form_rows row s_col_no_bgcolor">
                                    <div class="form-group col-12 s_website_form_field s_website_form_required" data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="opportunity1">
                                                <span class="s_website_form_label_content">Your Name</span>
                                                <span class="s_website_form_mark"> *</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="opportunity1" type="text" class="form-control s_website_form_input" name="contact_name" required=""/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="form-group col-12 s_website_form_field" data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="opportunity2">
                                                <span class="s_website_form_label_content">Phone Number</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="opportunity2" type="tel" class="form-control s_website_form_input" name="phone" />
                                            </div>
                                        </div>
                                    </div>
                                    <div class="form-group col-12 s_website_form_field s_website_form_required" data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="opportunity3">
                                                <span class="s_website_form_label_content">Email</span>
                                                <span class="s_website_form_mark"> *</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="opportunity3" type="text" class="form-control s_website_form_input" name="email_from" required="" />
                                            </div>
                                        </div>
                                    </div>
                                    <div class="form-group col-12 s_website_form_field" data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="opportunity4">
                                                <span class="s_website_form_label_content">Your Company</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="opportunity4" type="text" class="form-control s_website_form_input" name="partner_name" />
                                            </div>
                                        </div>
                                    </div>
                                    <div class="form-group col-12 s_website_form_field s_website_form_required s_website_form_model_required" data-type="char" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="opportunity5">
                                                <span class="s_website_form_label_content">Subject</span>
                                                <span class="s_website_form_mark"> *</span>
                                            </label>
                                            <div class="col-sm">
                                                <input id="opportunity5" type="text" class="form-control s_website_form_input" name="name" required="" />
                                            </div>
                                        </div>
                                    </div>
                                    <div class="form-group col-12 s_website_form_field s_website_form_required" data-type="text" data-name="Field">
                                        <div class="row s_col_no_resize s_col_no_bgcolor">
                                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="opportunity6">
                                                <span class="s_website_form_label_content">Your Question</span>
                                                <span class="s_website_form_mark"> *</span>
                                            </label>
                                            <div class="col-sm">
                                                <textarea id="opportunity6" class="form-control s_website_form_input" name="description" required=""></textarea>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="form-group col-12 s_website_form_submit" data-name="Submit Button">
                                        <div style="width: 200px;" class="s_website_form_label"/>
                                        <a href="#" role="button" class="btn btn-primary btn-lg s_website_form_send">Submit</a>
                                        <span id="s_website_form_result"></span>
                                    </div>
                                </div>
                            </form>
                        </div>
                    </section>
                </div>
			</xpath>
		</template>

        <template id="assets_editor" name="Website CRM Assets Editor" inherit_id="website.assets_editor">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_crm/static/src/js/website_crm_editor.js"></script>
            </xpath>
        </template>

        <template id="assets_tests" name="Website CRM Assets Tests" inherit_id="web.assets_tests">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_crm/static/tests/tours/website_crm.js"></script>
            </xpath>
        </template>

</odoo>

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!--Website visitor actions-->
    <record id="website_crm.website_visitor_crm_lead_action" model="ir.actions.act_window">
        <field name="name">Leads</field>
        <field name="res_model">crm.lead</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('visitor_ids', 'in', [active_id])]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
              No lead linked for this visitor
            </p>
        </field>
    </record>

    <!-- website visitor views -->
    <record id="website_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form.inherit.website.crm</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@id='w_visitor_visit_counter']" position="before">
                <button name="%(website_crm.website_visitor_crm_lead_action)d" type="action" class="oe_stat_button" icon="fa-star"
                        groups="sales_team.group_sale_salesman" attrs="{'invisible': [('lead_count', '=', 0)]}">
                    <field name="lead_count" widget="statinfo" string="Leads"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree.inherit.website.crm</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='page_ids']" position="after">
                <field name="lead_count"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_search" model="ir.ui.view">
        <field name="name">website.visitor.view.search.inherit.website.crm</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='type_visitor']" position="replace">
                <filter string="Visitors" name="type_visitor" domain="[('partner_id', '=', False), ('lead_ids', '=', False)]"/>
                <filter string="Leads" name="type_lead" domain="[('partner_id', '=', False), ('lead_ids', '!=', False)]"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban.inherit.website.crm</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_kanban"/>
        <field name="arch" type="xml">
            <field name="page_ids" position="after">
                <field name="lead_count"/>
            </field>
            <xpath expr="//div[@id='o_page_count']" position="after">
                <div groups="sales_team.group_sale_salesman">Leads/Opportunities<span class="float-right font-weight-bold"><field name="lead_count"/></span></div>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions_ungrouped')]" position="before">
                <div class="col" groups="sales_team.group_sale_salesman">
                    <b><field name="lead_count"/></b>
                    <div>Leads/Opportunities</div>
                </div>
            </xpath>
        </field>
    </record>
</data></odoo>

```

