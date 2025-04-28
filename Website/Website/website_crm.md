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
Add capability to your website forms to generate leads or opportunities in the CRM app.
Forms has to be customized inside the *Website Builder* in order to generate leads.

This module includes contact phone and mobile numbers validation.""",
    'depends': ['website', 'crm'],
    'data': [
        'security/ir.model.access.csv',
        'data/crm_lead_merge_template.xml',
        'data/ir_actions_data.xml',
        'data/ir_model_data.xml',
        'views/crm_lead_views.xml',
        'views/website_visitor_views.xml',
        'views/website_templates_contactus.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'website.assets_wysiwyg': [
            'website_crm/static/src/js/website_crm_editor.js',
        ],
        'web.assets_tests': [
            'website_crm/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\website_form.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import tools
from odoo.addons.phone_validation.tools import phone_validation
from odoo.addons.website.controllers import form
from odoo.http import request


class WebsiteForm(form.WebsiteForm):

    def _get_country(self):
        visitor_partner = request.env['website.visitor']._get_visitor_from_request().partner_id
        if visitor_partner:
            # match same behaviour as in partner._phone_format()
            country = visitor_partner.country_id or request.env.company.country_id
            if country:
                return country
        country_code = request.geoip.country_code
        if country_code:
            return request.env['res.country'].sudo().search([('code', '=', country_code)], limit=1)
        return request.env['res.country']

    # Check and insert values from the form on the model <model> + validation phone fields
    def _handle_website_form(self, model_name, **kwargs):
        model_record = request.env['ir.model'].sudo().search([('model', '=', model_name), ('website_form_access', '=', True)])
        if model_record:
            try:
                data = self.extract_data(model_record, request.params)
            except:
                # no specific management, super will do it
                pass
            else:
                record = data.get('record', {})
                phone_fields = request.env[model_name]._phone_get_number_fields()
                country = request.env['res.country'].browse(record.get('country_id'))
                contact_country = country if country.exists() else self._get_country()
                for phone_field in phone_fields:
                    if not record.get(phone_field):
                        continue
                    number = record[phone_field]
                    fmt_number = phone_validation.phone_format(
                        number, contact_country.code if contact_country else None,
                        contact_country.phone_code if contact_country else None,
                        force_format='INTERNATIONAL',
                        raise_exception=False
                    )
                    request.params.update({phone_field: fmt_number})

        if model_name == 'crm.lead' and not request.params.get('state_id'):
            geoip_country_code = request.geoip.country_code
            geoip_state_code = request.geoip.subdivisions[0].iso_code if request.geoip.subdivisions else None
            if geoip_country_code and geoip_state_code:
                state = request.env['res.country.state'].search([('code', '=', geoip_state_code), ('country_id.code', '=', geoip_country_code)])
                if state:
                    request.params['state_id'] = state.id
        return super(WebsiteForm, self)._handle_website_form(model_name, **kwargs)

    def insert_record(self, request, model, values, custom, meta=None):
        is_lead_model = model.sudo().model == 'crm.lead'
        if is_lead_model:
            values_email_normalized = tools.email_normalize(values.get('email_from'))
            visitor_sudo = request.env['website.visitor']._get_visitor_from_request(force_create=True)
            visitor_partner = visitor_sudo.partner_id
            if values_email_normalized and visitor_partner and visitor_partner.email_normalized == values_email_normalized:
                # Here, 'phone' in values has already been formatted, see _handle_website_form.
                values_phone = values.get('phone')
                # We write partner id on crm only if no phone exists on partner or in input,
                # or if both numbers (after formating) are the same. This way we get additional phone
                # if possible, without modifying an existing one. (see inverse function on model crm.lead)
                if values_phone and visitor_partner.phone:
                    if values_phone == visitor_partner.phone:
                        values['partner_id'] = visitor_partner.id
                    elif (visitor_partner._phone_format('phone') or visitor_partner.phone) == values_phone:
                        values['partner_id'] = visitor_partner.id
                else:
                    values['partner_id'] = visitor_partner.id
            if 'company_id' not in values:
                values['company_id'] = request.website.company_id.id
            lang = request.context.get('lang', False)
            values['lang_id'] = values.get('lang_id') or request.env['res.lang']._get_data(code=lang).id

        result = super(WebsiteForm, self).insert_record(request, model, values, custom, meta=meta)

        if is_lead_model and visitor_sudo and result:
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

from . import website_form

```

## File: data\crm_lead_merge_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="crm_lead_merge_summary_inherit_website" inherit_id="crm.crm_lead_merge_summary">
    <xpath expr="//div[@name='marketing']" position="attributes">
        <attribute name="t-if">lead.campaign_id or lead.medium_id or lead.source_id or lead.visitor_ids</attribute>
    </xpath>

    <xpath expr="//div[@name='marketing']" position="inside">
        <div t-if="lead.visitor_ids">
            Web Visitors: <t t-out="', '.join(lead.visitor_ids.mapped('display_name'))"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: data\ir_actions_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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

## File: data\ir_model_data.xml

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
                'lead_properties',
            ]"/>
        </function>
    </data>
</odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Lead(models.Model):
    _inherit = 'crm.lead'

    visitor_ids = fields.Many2many('website.visitor', string="Web Visitors")
    visitor_page_count = fields.Integer('# Page Views', compute="_compute_visitor_page_count")

    @api.depends('visitor_ids.page_ids')
    def _compute_visitor_page_count(self):
        mapped_data = {}
        if self.ids:
            self.flush_model(['visitor_ids'])
            self.env['website.track'].flush_model(['visitor_id'])
            sql = """ SELECT l.id as lead_id, count(*) as page_view_count
                        FROM crm_lead l
                        JOIN crm_lead_website_visitor_rel lv ON l.id = lv.crm_lead_id
                        JOIN website_visitor v ON v.id = lv.website_visitor_id
                        JOIN website_track p ON p.visitor_id = v.id
                        WHERE l.id in %s
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

    def _merge_get_fields_specific(self):
        fields_info = super(Lead, self)._merge_get_fields_specific()
        # add all the visitors from all lead to merge
        fields_info['visitor_ids'] = lambda fname, leads: [(6, 0, leads.visitor_ids.ids)]
        return fields_info

    def website_form_input_filter(self, request, values):
        values['medium_id'] = values.get('medium_id') or \
                              self.sudo().default_get(['medium_id']).get('medium_id') or \
                              self.env['utm.medium']._fetch_or_create_utm_medium('website').id
        values['team_id'] = values.get('team_id') or \
                            request.website.crm_default_team_id.id
        values['user_id'] = values.get('user_id') or \
                            request.website.crm_default_user_id.id
        if values.get('team_id'):
            values['type'] = 'lead' if self.env['crm.team'].sudo().browse(values['team_id']).use_leads else 'opportunity'
        else:
            values['type'] = 'lead' if self.env.user.has_group('crm.group_use_lead') else 'opportunity'

        return values

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


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

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.osv import expression


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
                main_lead._handle_partner_assignment(create_missing=True)
                self.partner_id = main_lead.partner_id.id
            return True
        return check

    def _inactive_visitors_domain(self):
        """ Visitors tied to leads are considered always active and should not be deleted. """
        domain = super()._inactive_visitors_domain()
        return expression.AND([domain, [('lead_ids', '=', False)]])

    def _merge_visitor(self, target):
        """ Link the leads to the main visitor to avoid them being lost. """
        if self.lead_ids:
            target.write({
                'lead_ids': [(4, lead.id) for lead in self.lead_ids]
            })

        return super()._merge_visitor(target)

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
from . import website
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M39 46a8 8 0 0 0 8-8V8H13a4 4 0 0 0-4 4v34h30Z" fill="#FC868B"/><path fill-rule="evenodd" clip-rule="evenodd" d="M39 42a4 4 0 0 0 4-4V4H13a8 8 0 0 0-8 8v30h34Z" fill="#2EBCFA"/><path fill-rule="evenodd" clip-rule="evenodd" d="M43 38a4 4 0 0 1-4 4H9V12a4 4 0 0 1 4-4h30v30Z" fill="#2D6388"/><path d="M25 28a9 9 0 0 0-9 9h10a9 9 0 0 0 9-9H25Z" fill="#fff"/><circle cx="25" cy="19" r="6" fill="#fff"/></svg>

```

## File: static\src\js\website_crm_editor.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import FormEditorRegistry from "@website/js/form_editor_registry";

FormEditorRegistry.add('create_lead', {
    formFields: [{
        type: 'char',
        required: true,
        name: 'contact_name',
        fillWith: 'name',
        string: _t('Your Name'),
    }, {
        type: 'tel',
        name: 'phone',
        fillWith: 'phone',
        string: _t('Phone Number'),
    }, {
        type: 'email',
        required: true,
        fillWith: 'email',
        name: 'email_from',
        string: _t('Your Email'),
    }, {
        type: 'char',
        required: true,
        fillWith: 'commercial_company_name',
        name: 'partner_name',
        string: _t('Your Company'),
    }, {
        type: 'char',
        modelRequired: true,
        name: 'name',
        string: _t('Subject'),
    }, {
        type: 'text',
        required: true,
        name: 'description',
        string: _t('Your Question'),
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

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="crm_lead_view_form" model="ir.ui.view">
        <field name="name">crm.lead.view.form.inherit.website.crm</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_schedule_meeting']" position="after">
                <button name="action_redirect_to_page_views" type="object" class="oe_stat_button" icon="fa-tags"
                        invisible="visitor_page_count == 0">
                    <field name="visitor_page_count" widget="statinfo" string="Page views"/>
                </button>
            </xpath>
        </field>
    </record>

    <!--Website visitor actions-->
    <record id="crm_lead_action_from_visitor" model="ir.actions.act_window">
        <field name="name">Leads</field>
        <field name="res_model">crm.lead</field>
        <field name="view_mode">list,form</field>
        <field name="domain">[('visitor_ids', 'in', [active_id])]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
              No lead linked for this visitor
            </p>
        </field>
    </record>
</data></odoo>

```

## File: views\website_templates_contactus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="contactus_form" name="Contact Form (Opportunity)" inherit_id="website.contactus">
            <xpath expr="//t[@t-set='contactus_form_values']" position="after">
                <t t-set="contactus_form_values" t-value="dict(contactus_form_values, **{
                    'contact_name': request.params.get('contact_name', ''),
                    'partner_name': request.params.get('partner_name', ''),
                    'description': request.params.get('description', ''),
                })"/>
            </xpath>
		</template>
    </data>
</odoo>

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="website_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form.inherit.website.crm</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@id='w_visitor_visit_counter']" position="before">
                <button name="%(website_crm.crm_lead_action_from_visitor)d" type="action" class="oe_stat_button" icon="fa-star"
                        groups="sales_team.group_sale_salesman"
                        invisible="lead_count == 0">
                    <field name="lead_count" widget="statinfo" string="Leads"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.list.inherit.website.crm</field>
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
            <xpath expr="//filter[@name='filter_type_visitor']" position="attributes">
                <attribute name="domain">[('partner_id', '=', False), ('lead_ids', '=', False)]</attribute>
            </xpath>
            <xpath expr="//filter[@name='filter_type_visitor']" position="after">
                <filter string="Leads" name="filter_type_lead" domain="[('partner_id', '=', False), ('lead_ids', '!=', False)]"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban.inherit.website.crm</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='o_page_count']" position="after">
                <div t-if="record.lead_count.raw_value" groups="sales_team.group_sale_salesman">
                    Leads / Opportunities
                    <field name="lead_count" class="float-end fw-bold"/>
                </div>
            </xpath>
            <xpath expr="//div[@id='wvisitor_visited_page']" position="after">
                <div class="col-lg col-sm-4 col-6 py-0 my-2" groups="sales_team.group_sale_salesman">
                    <span t-att-class="record.lead_count.raw_value ? 'fw-bold' : 'text-muted'">
                        <field name="lead_count"/>
                    </span>
                    <div t-att-class="record.lead_count.raw_value ? '' : 'text-muted'">Leads / Opportunities</div>
                </div>
            </xpath>
        </field>
    </record>
</data></odoo>

```

