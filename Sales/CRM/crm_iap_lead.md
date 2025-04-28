# Odoo Module: crm_iap_lead

Category: Sales/CRM

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Lead Generation',
    'summary': 'Generate Leads/Opportunities based on country, industries, size, etc.',
    'version': '1.1',
    'category': 'Sales/CRM',
    'version': '1.1',
    'depends': [
        'iap_crm',
        'iap_mail',
    ],
    'data': [
        'data/crm.iap.lead.industry.csv',
        'data/crm.iap.lead.role.csv',
        'data/crm.iap.lead.seniority.csv',
        'data/crm_iap_lead_data.xml',
        'data/ir_sequence_data.xml',
        'security/ir.model.access.csv',
        'views/assets.xml',
        'views/crm_lead_view.xml',
        'views/crm_iap_lead_views.xml',
        'views/res_config_settings_views.xml',
        'views/mail_templates.xml',
    ],
    'qweb': [
        'static/src/xml/leads_tree_generate_leads_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\crm.iap.lead.industry.csv

```csv
"id",name,reveal_id
"crm_iap_lead_industry_30","Consumer Discretionary","30"
"crm_iap_lead_industry_33","Consumer Staples","33"
"crm_iap_lead_industry_69","Insurance","69"
"crm_iap_lead_industry_86","Media","86"
"crm_iap_lead_industry_114","Real Estate","114"
"crm_iap_lead_industry_136","Transportation","136"
"crm_iap_lead_industry_138","Utilities","138"
"crm_iap_lead_industry_146","Industrials","146"
"crm_iap_lead_industry_148","Materials","148"
"crm_iap_lead_industry_149","Telecommunication Services","149"
"crm_iap_lead_industry_150","Consumer Services","150"
"crm_iap_lead_industry_151","Diversified Consumer Services","151"
"crm_iap_lead_industry_152","Retailing","152"
"crm_iap_lead_industry_153","Food & Staples Retailing","153"
"crm_iap_lead_industry_154","Food, Beverage & Tobacco","154"
"crm_iap_lead_industry_155","Household & Personal Products","155"
"crm_iap_lead_industry_156","Energy Equipment & Services","156"
"crm_iap_lead_industry_157","Banks","157"
"crm_iap_lead_industry_158","Diversified Financial Services","158"
"crm_iap_lead_industry_159","Diversified Financials","159"
"crm_iap_lead_industry_160","Health Care Equipment & Services","160"
"crm_iap_lead_industry_161","Pharmaceuticals, Biotechnology & Life Sciences","161"
"crm_iap_lead_industry_162","Capital Goods","162"
"crm_iap_lead_industry_163","Commercial & Professional Services","163"
"crm_iap_lead_industry_164","Semiconductors & Semiconductor Equipment","164"
"crm_iap_lead_industry_165","Software & Services","165"
"crm_iap_lead_industry_166","Technology Hardware & Equipment","166"
"crm_iap_lead_industry_167","Construction Materials","167"
"crm_iap_lead_industry_168","Independent Power and Renewable Electricity Producers","168"
"crm_iap_lead_industry_238","Automobiles & Components","238"
"crm_iap_lead_industry_239","Consumer Durables & Apparel","239"

```

## File: data\crm.iap.lead.role.csv

```csv
"id",name,reveal_id
"crm_iap_lead_role_1","ceo","ceo"
"crm_iap_lead_role_2","communications","communications"
"crm_iap_lead_role_3","consulting","consulting"
"crm_iap_lead_role_4","customer_service","customer_service"
"crm_iap_lead_role_5","education","education"
"crm_iap_lead_role_6","engineering","engineering"
"crm_iap_lead_role_7","finance","finance"
"crm_iap_lead_role_8","founder","founder"
"crm_iap_lead_role_9","health_professional","health_professional"
"crm_iap_lead_role_10","human_resources","human_resources"
"crm_iap_lead_role_11","information_technology","information_technology"
"crm_iap_lead_role_12","legal","legal"
"crm_iap_lead_role_13","marketing","marketing"
"crm_iap_lead_role_14","operations","operations"
"crm_iap_lead_role_15","owner","owner"
"crm_iap_lead_role_16","president","president"
"crm_iap_lead_role_17","product","product"
"crm_iap_lead_role_18","public_relations","public_relations"
"crm_iap_lead_role_19","real_estate","real_estate"
"crm_iap_lead_role_20","recruiting","recruiting"
"crm_iap_lead_role_21","research","research"
"crm_iap_lead_role_22","sale","sale"

```

## File: data\crm.iap.lead.seniority.csv

```csv
"id",name,reveal_id
"crm_iap_lead_seniority_1","director","director"
"crm_iap_lead_seniority_2","executive","executive"
"crm_iap_lead_seniority_3","manager","manager"

```

## File: data\crm_iap_lead_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="lead_generation_no_credits" model="mail.template">
            <field name="name">IAP Lead Generation Notification</field>
            <field name="email_from">iap@odoo.com</field>
            <field name="email_to">iap@odoo.com</field>
            <field name="subject">IAP Lead Generation Notification</field>
            <field name="model_id" ref="iap.model_iap_account"/>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p>Dear,</p>
    <p>There are no more credits on your IAP Lead Generation account.<br/>
    You can charge your IAP Lead Generation account in the settings of the CRM app.<br/></p>
    <p>Best regards,</p>
    <p>Odoo S.A.</p>
</div></field>
        </record>
    </data>
</odoo>

```

## File: data\ir_sequence_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Sequences for lead mining requests -->
        <record id="seq_crm_iap_lead_mining_request" model="ir.sequence">
            <field name="name">Lead Mining Request</field>
            <field name="code">crm.iap.lead.mining.request</field>
            <field name="prefix">LMR</field>
            <field name="padding">3</field>
            <field name="company_id" eval="False"/>
        </record>

    </data>
</odoo>

```

## File: models\crm_iap_lead.py

```python
from odoo import api, fields, models


class IndustryTag(models.Model):
    """ Industry Tags of Acquisition Rules """
    _name = 'crm.iap.lead.industry'
    _description = 'Industry Tag'

    name = fields.Char(string='Tag Name', required=True, translate=True)
    reveal_id = fields.Char(required=True)
    color = fields.Integer(string='Color Index')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'Tag name already exists!'),
    ]


class PeopleRole(models.Model):
    """ CRM Reveal People Roles for People """
    _name = 'crm.iap.lead.role'
    _description = 'People Role'

    name = fields.Char(string='Role Name', required=True, translate=True)
    reveal_id = fields.Char(required=True)
    color = fields.Integer(string='Color Index')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'Role name already exists!'),
    ]

    @api.depends('name')
    def name_get(self):
        return [(role.id, role.name.replace('_', ' ').title()) for role in self]


class PeopleSeniority(models.Model):
    """ Seniority for People Rules """
    _name = 'crm.iap.lead.seniority'
    _description = 'People Seniority'

    name = fields.Char(string='Name', required=True, translate=True)
    reveal_id = fields.Char(required=True)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'Name already exists!'),
    ]

    @api.depends('name')
    def name_get(self):
        return [(seniority.id, seniority.name.replace('_', ' ').title()) for seniority in self]

```

## File: models\crm_iap_lead_helpers.py

```python
from math import floor, log10
from odoo import api, models


class CRMHelpers(models.Model):
    _name = 'crm.iap.lead.helpers'
    _description = 'Helper methods for crm_iap_lead modules'

    @api.model
    def notify_no_more_credit(self, service_name, model_name, notification_parameter):
        """
        Notify about the number of credit.
        In order to avoid to spam people each hour, an ir.config_parameter is set
        """
        already_notified = self.env['ir.config_parameter'].sudo().get_param(notification_parameter, False)
        if already_notified:
            return
        mail_template = self.env.ref('crm_iap_lead.lead_generation_no_credits')
        iap_account = self.env['iap.account'].search([('service_name', '=', service_name)], limit=1)
        # Get the email address of the creators of the records
        res = self.env[model_name].search_read([], ['create_uid'])
        uids = set(r['create_uid'][0] for r in res if r.get('create_uid'))
        res = self.env['res.users'].search_read([('id', 'in', list(uids))], ['email'])
        emails = set(r['email'] for r in res if r.get('email'))

        email_values = {
            'email_to': ','.join(emails)
        }
        mail_template.send_mail(iap_account.id, force_send=True, email_values=email_values)
        self.env['ir.config_parameter'].sudo().set_param(notification_parameter, True)

    @api.model
    def lead_vals_from_response(self, lead_type, team_id, tag_ids, user_id, company_data, people_data):
        country_id = self.env['res.country'].search([('code', '=', company_data['country_code'])]).id
        website_url = 'https://www.%s' % company_data['domain'] if company_data['domain'] else False
        lead_vals = {
            # Lead vals from record itself
            'type': lead_type,
            'team_id': team_id,
            'tag_ids': [(6, 0, tag_ids)],
            'user_id': user_id,
            'reveal_id': company_data['clearbit_id'],
            # Lead vals from data
            'name': company_data['name'] or company_data['domain'],
            'partner_name': company_data['legal_name'] or company_data['name'],
            'email_from': next(iter(company_data.get('email', [])), ''),
            'phone': company_data['phone'] or (company_data['phone_numbers'] and company_data['phone_numbers'][0]) or '',
            'website': website_url,
            'street': company_data['location'],
            'city': company_data['city'],
            'zip': company_data['postal_code'],
            'country_id': country_id,
            'state_id': self._find_state_id(company_data['state_code'], country_id),
        }

        # If type is people then add first contact in lead data
        if people_data:
            lead_vals.update({
                'contact_name': people_data[0]['full_name'],
                'email_from': people_data[0]['email'],
                'function': people_data[0]['title'],
            })
        return lead_vals

    @api.model
    def _find_state_id(self, state_code, country_id):
        state_id = self.env['res.country.state'].search([('code', '=', state_code), ('country_id', '=', country_id)])
        if state_id:
            return state_id.id
        return False

```

## File: models\crm_iap_lead_mining_request.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, _
from odoo.addons.iap.tools import iap_tools

_logger = logging.getLogger(__name__)

DEFAULT_ENDPOINT = 'https://iap-services.odoo.com'

MAX_LEAD = 200

MAX_CONTACT = 5

CREDIT_PER_COMPANY = 1
CREDIT_PER_CONTACT = 1


class CRMLeadMiningRequest(models.Model):
    _name = 'crm.iap.lead.mining.request'
    _description = 'CRM Lead Mining Request'

    def _default_lead_type(self):
        if self.env.user.has_group('crm.group_use_lead'):
            return 'lead'
        else:
            return 'opportunity'

    def _default_country_ids(self):
        return self.env.user.company_id.country_id

    name = fields.Char(string='Request Number', required=True, readonly=True, default=lambda self: _('New'), copy=False)
    state = fields.Selection([('draft', 'Draft'), ('done', 'Done'), ('error', 'Error')], string='Status', required=True, default='draft')

    # Request Data
    lead_number = fields.Integer(string='Number of Leads', required=True, default=3)
    search_type = fields.Selection([('companies', 'Companies'), ('people', 'Companies and their Contacts')], string='Target', required=True, default='companies')
    error = fields.Text(string='Error', readonly=True, copy=False)

    # Lead / Opportunity Data

    lead_type = fields.Selection([('lead', 'Leads'), ('opportunity', 'Opportunities')], string='Type', required=True, default=_default_lead_type)
    display_lead_label = fields.Char(compute='_compute_display_lead_label')
    team_id = fields.Many2one(
        'crm.team', string='Sales Team',
        domain="[('use_opportunities', '=', True)]", readonly=False, compute='_compute_team_id', store=True)
    user_id = fields.Many2one('res.users', string='Salesperson', default=lambda self: self.env.user)
    tag_ids = fields.Many2many('crm.tag', string='Tags')
    lead_ids = fields.One2many('crm.lead', 'lead_mining_request_id', string='Generated Lead / Opportunity')
    lead_count = fields.Integer(compute='_compute_lead_count', string='Number of Generated Leads')

    # Company Criteria Filter
    filter_on_size = fields.Boolean(string='Filter on Size', default=False)
    company_size_min = fields.Integer(string='Size', default=1)
    company_size_max = fields.Integer(default=1000)
    country_ids = fields.Many2many('res.country', string='Countries', default=_default_country_ids)
    state_ids = fields.Many2many('res.country.state', string='States')
    industry_ids = fields.Many2many('crm.iap.lead.industry', string='Industries')

    # Contact Generation Filter
    contact_number = fields.Integer(string='Number of Contacts', default=10)
    contact_filter_type = fields.Selection([('role', 'Role'), ('seniority', 'Seniority')], string='Filter on', default='role')
    preferred_role_id = fields.Many2one('crm.iap.lead.role', string='Preferred Role')
    role_ids = fields.Many2many('crm.iap.lead.role', string='Other Roles')
    seniority_id = fields.Many2one('crm.iap.lead.seniority', string='Seniority')

    # Fields for the blue tooltip
    lead_credits = fields.Char(compute='_compute_tooltip', readonly=True)
    lead_contacts_credits = fields.Char(compute='_compute_tooltip', readonly=True)
    lead_total_credits = fields.Char(compute='_compute_tooltip', readonly=True)

    @api.depends('lead_type', 'lead_number')
    def _compute_display_lead_label(self):
        selection_description_values = {
            e[0]: e[1] for e in self._fields['lead_type']._description_selection(self.env)}
        for request in self:
            lead_type = selection_description_values[request.lead_type]
            request.display_lead_label = '%s %s' % (request.lead_number, lead_type)


    @api.onchange('lead_number', 'contact_number')
    def _compute_tooltip(self):
        for record in self:
            company_credits = CREDIT_PER_COMPANY * record.lead_number
            contact_credits = CREDIT_PER_CONTACT * record.contact_number
            total_contact_credits = contact_credits * record.lead_number
            record.lead_contacts_credits = _("Up to %d additional credits will be consumed to identify %d contacts per company.") % (contact_credits*company_credits, record.contact_number)
            record.lead_credits = _('%d credits will be consumed to find %d companies.') % (company_credits, record.lead_number)
            record.lead_total_credits = _("This makes a total of %d credits for this request.") % (total_contact_credits + company_credits)

    @api.depends('lead_ids.lead_mining_request_id')
    def _compute_lead_count(self):
        if self.ids:
            leads_data = self.env['crm.lead'].read_group(
                [('lead_mining_request_id', 'in', self.ids)],
                ['lead_mining_request_id'], ['lead_mining_request_id'])
        else:
            leads_data = []
        mapped_data = dict(
            (m['lead_mining_request_id'][0], m['lead_mining_request_id_count'])
            for m in leads_data)
        for request in self:
            request.lead_count = mapped_data.get(request.id, 0)

    @api.depends('user_id')
    def _compute_team_id(self):
        for record in self:
            record.team_id = record.user_id.sale_team_id

    @api.onchange('lead_number')
    def _onchange_lead_number(self):
        if self.lead_number <= 0:
            self.lead_number = 1
        elif self.lead_number > MAX_LEAD:
            self.lead_number = MAX_LEAD

    @api.onchange('contact_number')
    def _onchange_contact_number(self):
        if self.contact_number <= 0:
            self.contact_number = 1
        elif self.contact_number > MAX_CONTACT:
            self.contact_number = MAX_CONTACT

    @api.onchange('country_ids')
    def _onchange_country_ids(self):
        self.state_ids = []

    @api.onchange('company_size_min')
    def _onchange_company_size_min(self):
        if self.company_size_min <= 0:
            self.company_size_min = 1
        elif self.company_size_min > self.company_size_max:
            self.company_size_min = self.company_size_max

    @api.onchange('company_size_max')
    def _onchange_company_size_max(self):
        if self.company_size_max < self.company_size_min:
            self.company_size_max = self.company_size_min
    
    def _prepare_iap_payload(self):
        """
        This will prepare the data to send to the server
        """
        self.ensure_one()
        payload = {'lead_number': self.lead_number,
                   'search_type': self.search_type,
                   'countries': self.country_ids.mapped('code')}
        if self.state_ids:
            payload['states'] = self.state_ids.mapped('code')
        if self.filter_on_size:
            payload.update({'company_size_min': self.company_size_min,
                            'company_size_max': self.company_size_max})
        if self.industry_ids:
            payload['industry_ids'] = self.industry_ids.mapped('reveal_id')
        if self.search_type == 'people':
            payload.update({'contact_number': self.contact_number,
                            'contact_filter_type': self.contact_filter_type})
            if self.contact_filter_type == 'role':
                payload.update({'preferred_role': self.preferred_role_id.reveal_id,
                                'other_roles': self.role_ids.mapped('reveal_id')})
            elif self.contact_filter_type == 'seniority':
                payload['seniority'] = self.seniority_id.reveal_id
        return payload

    def _perform_request(self):
        """
        This will perform the request and create the corresponding leads.
        The user will be notified if he hasn't enough credits.
        """
        server_payload = self._prepare_iap_payload()
        reveal_account = self.env['iap.account'].get('reveal')
        dbuuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        endpoint = self.env['ir.config_parameter'].sudo().get_param('reveal.endpoint', DEFAULT_ENDPOINT) + '/iap/clearbit/1/lead_mining_request'
        params = {
            'account_token': reveal_account.account_token,
            'dbuuid': dbuuid,
            'data': server_payload
        }
        try:
            response = iap_tools.iap_jsonrpc(endpoint, params=params, timeout=300)
            return response['data']
        except iap_tools.InsufficientCreditError as e:
            self.error = 'Insufficient credits. Recharge your account and retry.'
            self.state = 'error'
            self._cr.commit()
            raise e

    def _create_leads_from_response(self, result):
        """ This method will get the response from the service and create the leads accordingly """
        self.ensure_one()
        lead_vals_list = []
        messages_to_post = {}
        for data in result:
            lead_vals_list.append(self._lead_vals_from_response(data))

            template_values = data['company_data']
            template_values.update({
                'flavor_text': _("Opportunity created by Odoo Lead Generation"),
                'people_data': data.get('people_data'),
            })
            messages_to_post[data['company_data']['clearbit_id']] = template_values
        leads = self.env['crm.lead'].create(lead_vals_list)
        for lead in leads:
            if messages_to_post.get(lead.reveal_id):
                lead.message_post_with_view('iap_mail.enrich_company', values=messages_to_post[lead.reveal_id], subtype_id=self.env.ref('mail.mt_note').id)

    # Methods responsible for format response data into valid odoo lead data
    @api.model
    def _lead_vals_from_response(self, data):
        self.ensure_one()
        company_data = data.get('company_data')
        people_data = data.get('people_data')
        lead_vals = self.env['crm.iap.lead.helpers'].lead_vals_from_response(self.lead_type, self.team_id.id, self.tag_ids.ids, self.user_id.id, company_data, people_data)
        lead_vals['lead_mining_request_id'] = self.id
        return lead_vals

    @api.model
    def get_empty_list_help(self, help):
        help_title = _('Create a Lead Mining Request')
        sub_title = _('Generate new leads based on their country, industry, size, etc.')
        return '<p class="o_view_nocontent_smiling_face">%s</p><p class="oe_view_nocontent_alias">%s</p>' % (help_title, sub_title)

    def action_draft(self):
        self.ensure_one()
        self.name = _('New')
        self.state = 'draft'

    def action_submit(self):
        self.ensure_one()
        if self.name == _('New'):
            self.name = self.env['ir.sequence'].next_by_code('crm.iap.lead.mining.request') or _('New')
        results = self._perform_request()
        if results:
            self._create_leads_from_response(results)
            self.state = 'done'
        if self.lead_type == 'lead':
            return self.action_get_lead_action()
        elif self.lead_type == 'opportunity':
            return self.action_get_opportunity_action()

    def action_get_lead_action(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_all_leads")
        action['domain'] = [('id', 'in', self.lead_ids.ids), ('type', '=', 'lead')]
        action['help'] = _("""<p class="o_view_nocontent_empty_folder">
            No leads found
        </p><p>
            No leads could be generated according to your search criteria
        </p>""")
        return action

    def action_get_opportunity_action(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_opportunities")
        action['domain'] = [('id', 'in', self.lead_ids.ids), ('type', '=', 'opportunity')]
        action['help'] = _("""<p class="o_view_nocontent_empty_folder">
            No opportunities found
        </p><p>
            No opportunities could be generated according to your search criteria
        </p>""")
        return action

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Lead(models.Model):
    _inherit = 'crm.lead'

    lead_mining_request_id = fields.Many2one('crm.iap.lead.mining.request', string='Lead Mining Request', index=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import crm_lead
from . import crm_iap_lead
from . import crm_iap_lead_helpers
from . import crm_iap_lead_mining_request

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_crm_iap_lead_industry,access_crm_iap_lead_industry,model_crm_iap_lead_industry,sales_team.group_sale_manager,1,1,1,1
access_crm_iap_lead_role,access_crm_iap_lead_role,model_crm_iap_lead_role,sales_team.group_sale_manager,1,1,1,1
access_crm_iap_lead_seniority,access_crm_iap_lead_seniority,model_crm_iap_lead_seniority,sales_team.group_sale_manager,1,1,1,1
access_crm_iap_lead_mining_request,access_crm_iap_lead_mining_request,model_crm_iap_lead_mining_request,sales_team.group_sale_manager,1,1,1,1
access_crm_iap_lead_helpers,access_crm_iap_lead_helpers,model_crm_iap_lead_helpers,,0,0,0,0

```

## File: static\src\js\leads_tree_generate_leads.js

```javascript
odoo.define('crm.leads.tree', function (require) {
"use strict";
    var ListController = require('web.ListController');
    var ListView = require('web.ListView');

    var KanbanController = require('web.KanbanController');
    var KanbanView = require('crm.crm_kanban').CrmKanbanView;

    var viewRegistry = require('web.view_registry');

    function renderGenerateLeadsButton() {
        if (this.$buttons) {
            var self = this;
            var lead_type = self.initialState.getContext()['default_type'];
            this.$buttons.on('click', '.o_button_generate_leads', function () {
                self.do_action({
                    name: 'Generate Leads',
                    type: 'ir.actions.act_window',
                    res_model: 'crm.iap.lead.mining.request',
                    target: 'new',
                    views: [[false, 'form']],
                    context: {'is_modal': true, 'default_lead_type': lead_type},
                });
            });
        }
    }

    var LeadMiningRequestListController = ListController.extend({
        willStart: function() {
            var self = this;
            var ready = this.getSession().user_has_group('sales_team.group_sale_manager')
                .then(function (is_sale_manager) {
                    if (is_sale_manager) {
                        self.buttons_template = 'LeadMiningRequestListView.buttons';
                    }
                });
            return Promise.all([this._super.apply(this, arguments), ready]);
        },
        renderButtons: function () {
            this._super.apply(this, arguments);
            renderGenerateLeadsButton.apply(this, arguments);
        }
    });

    var LeadMiningRequestListView = ListView.extend({
        config: _.extend({}, ListView.prototype.config, {
            Controller: LeadMiningRequestListController,
        }),
    });

    var LeadMiningRequestKanbanController = KanbanController.extend({
        willStart: function() {
            var self = this;
            var ready = this.getSession().user_has_group('sales_team.group_sale_manager')
                .then(function (is_sale_manager) {
                    if (is_sale_manager) {
                        self.buttons_template = 'LeadMiningRequestKanbanView.buttons';
                    }
                });
            return Promise.all([this._super.apply(this, arguments), ready]);
        },
        renderButtons: function () {
            this._super.apply(this, arguments);
            renderGenerateLeadsButton.apply(this, arguments);
        }
    });

    var LeadMiningRequestKanbanView = KanbanView.extend({
        config: _.extend({}, KanbanView.prototype.config, {
            Controller: LeadMiningRequestKanbanController,
        }),
    });

    viewRegistry.add('crm_iap_lead_mining_request_tree', LeadMiningRequestListView);
    viewRegistry.add('crm_iap_lead_mining_request_kanban', LeadMiningRequestKanbanView);
});

```

## File: static\src\js\tours\crm_iap_lead.js

```javascript
odoo.define('crm_iap_lead.generate_leads_steps', function (require) {
"use strict";

var tour = require('web_tour.tour');
var core = require('web.core');

require('crm.tour');
var _t = core._t;

var DragOppToWonStepIndex = _.findIndex(tour.tours.crm_tour.steps, function (step) {
    return (step.id === 'drag_opportunity_to_won_step');
});

tour.tours.crm_tour.steps.splice(DragOppToWonStepIndex + 1, 0, {
    /**
     * Add some steps between "Drag your opportunity to <b>Won</b> when you get
     * the deal. Congrats !" and "Let’s have a look at an Opportunity." to
     * include the steps related to the lead generation (crm_iap_lead).
     * This eases the on boarding for the Lead Generation process.
     *
     */
    trigger: ".o_button_generate_leads",
    content: _t("Looking for more opportunities ?<br>Try the <b>Lead Generation</b> tool."),
    position: "bottom",
    run: function (actions) {
        actions.auto('.o_button_generate_leads');
    },
}, {
    trigger: '.modal-body .o_industry',
    content: _t("Which Industry do you want to target?"),
    position: "right",
}, {
    trigger: '.modal-footer button[name=action_submit]',
    content: _t("Now, just let the magic happen!"),
    position: "bottom",
    run: function (actions) {
        actions.auto('.modal-footer button[special=cancel]');
}
});

});
```

## File: static\src\xml\leads_tree_generate_leads_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="LeadMiningRequest.generate_leads_button">
        <button type="button" class="btn btn-secondary o_button_generate_leads">
            Generate Leads
        </button>
    </t>

    <t t-extend="ListView.buttons" t-name="LeadMiningRequestListView.buttons">
        <t t-jquery="button.o_list_button_add" t-operation="after">
            <t t-call="LeadMiningRequest.generate_leads_button"/>
        </t>
    </t>

    <t t-extend="KanbanView.buttons" t-name="LeadMiningRequestKanbanView.buttons">
        <t t-jquery="button" t-operation="after">
            <t t-call="LeadMiningRequest.generate_leads_button"/>
        </t>
    </t>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="crm_iap_lead assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/crm_iap_lead/static/src/js/leads_tree_generate_leads.js"/>
            <script type="text/javascript" src="/crm_iap_lead/static/src/js/tours/crm_iap_lead.js"></script>
        </xpath>
    </template>
</odoo>
```

## File: views\crm_iap_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_iap_lead_mining_request_form" model="ir.ui.view">
        <field name="name">crm.iap.lead.mining.request.form</field>
        <field name="model">crm.iap.lead.mining.request</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <button name="action_submit" type="object" string="Submit" states="draft" class="oe_highlight" invisible="context.get('is_modal')"/>
                    <button name="action_submit" type="object" string="Retry" states="error" class="oe_highlight"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,done" invisible="context.get('is_modal')"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_get_opportunity_action" class="oe_stat_button" type="object" icon="fa-handshake-o" attrs="{'invisible': ['|', ('lead_type', '!=', 'opportunity'), ('state', '!=' , 'done')]}">
                            <div class="o_stat_info">
                                <field name="lead_count"/>
                                <span class="o_stat_text">Opportunities</span>
                            </div>
                        </button>
                        <button name="action_get_lead_action" class="oe_stat_button" type="object" icon="fa-handshake-o" groups="crm.group_use_lead" attrs="{'invisible': ['|', ('lead_type', '!=', 'lead'), ('state', '!=' , 'done')]}">
                            <div class="o_stat_info">
                                <field name="lead_count"/>
                                <span class="o_stat_text">Leads</span>
                            </div>
                        </button>
                    </div>
                    <span invisible="context.get('is_modal')">
                        <div class="row">
                            <div class="col-md-6">
                                <div class="oe_title">
                                    <h1>
                                        <field name="name"/>
                                    </h1>
                                </div>
                            </div>
                        </div>
                    </span>

                    <h2>What do you need ?</h2>

                    <group name="requests">
                        <group>
                            <div  class="o_row">
                                <field name="lead_number" attrs="{'readonly': [('state', '!=', 'draft')]}" nolabel="1" class="oe_inline col-md-2 pl-0"/>
                                <field name="search_type" widget="selection" attrs="{'readonly': [('state', '!=', 'draft')]}" nolabel="1" class="oe_inline col-md-6"/>
                                <field name="error" attrs="{'invisible': [('state', '!=', 'error')]}"/>
                            </div>
                        </group>
                    </group>

                    <group>
                        <group name="companies">
                            <field name="country_ids" widget="many2many_tags" attrs="{'readonly': [('state', '!=', 'draft')]}" required="True" options="{'no_create': True, 'no_open': True}"/>
                            <field name="state_ids" widget="many2many_tags" attrs="{'invisible': [('country_ids', '=', [])], 'readonly': [('state', '!=', 'draft')]}" domain="[('country_id', 'in', country_ids)]" options="{'no_create': True, 'no_open': True}"/>
                            <field name="industry_ids" widget="many2many_tags" attrs="{'readonly': [('state', '!=', 'draft')]}" options="{'no_create': True, 'no_open': True}" class="o_industry"/>
                            <field name="filter_on_size" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                            <label for="company_size_min" attrs="{'invisible': [('filter_on_size', '=', False)]}"/>
                            <div attrs="{'invisible': [('filter_on_size', '=', False)]}">
                                From
                                <field name="company_size_min" class="oe_inline col-sm-3" attrs="{'required': [('filter_on_size', '=', True)], 'readonly': [('state', '!=', 'draft')]}"/>
                                to
                                <field name="company_size_max" class="oe_inline col-sm-3" attrs="{'required': [('filter_on_size', '=', True)], 'readonly': [('state', '!=', 'draft')]}"/>
                                employees
                            </div>

                        </group>
                        <group name="lead_info">
                            <field name="lead_type" groups="crm.group_use_lead" invisible="context.get('is_modal')" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                            <field name="team_id" no_create="1" no_open="1" attrs="{'readonly': [('state', '!=', 'draft')]}" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
                            <field name="user_id" no_create="1" no_open="1" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                            <field name="tag_ids" string="Default Tags" widget="many2many_tags" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                        </group>
                    </group>

                    <group name="contacts" attrs="{'invisible': [('search_type', '!=', 'people')]}">
                        <div>
                            <field name="contact_number" attrs="{'readonly': [('state', '!=', 'draft')], 'required': [('search_type', '=', 'people')]}" nolabel="1" class="col-md-1"/>
                             <span class="col-md-6">Extra contacts per Company</span>
                        </div>
                    </group>
                    <group attrs="{'invisible': [('search_type', '!=', 'people')]}">
                        <group>
                            <field name="contact_filter_type" widget="radio" attrs="{'readonly': [('state', '!=', 'draft')]}" options="{'horizontal': true}"/>
                            <field name="preferred_role_id" options="{'no_create_edit': True, 'no_quick_create': True}" attrs="{'invisible': [('contact_filter_type','!=', 'role')], 'required': [('search_type', '=', 'people'), ('contact_filter_type', '=', 'role')], 'readonly': [('state', '!=', 'draft')]}"/>
                            <field name="role_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True, 'no_quick_create': True}" attrs="{'invisible': ['|', ('preferred_role_id','=', False), ('contact_filter_type','!=', 'role')], 'readonly': [('state', '!=', 'draft')]}"/>
                            <field name="seniority_id" options="{'no_create_edit': True, 'no_quick_create': True}" attrs="{'invisible': [('contact_filter_type', '!=', 'seniority')], 'required': [('search_type', '=', 'people'), ('contact_filter_type', '=', 'seniority')], 'readonly': [('state', '!=', 'draft')]}"/>
                        </group>
                    </group>
                    <footer>
                        <button string="Generate Leads" name="action_submit" type="object" default_focus="1" class="btn-primary" invisible="not context.get('is_modal')"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" invisible="not context.get('is_modal')"/>
                    </footer>
                </sheet>
            </form>
        </field>
    </record>

    <record id="crm_iap_lead_mining_request_tree" model="ir.ui.view">
        <field name="name">crm.iap.lead.mining.request.tree</field>
        <field name="model">crm.iap.lead.mining.request</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name" decoration-bf="1"/>
                <field name="display_lead_label" string="Number of Leads"/>
                <field name="search_type"/>
                <field name="country_ids" widget="many2many_tags"/>
                <field name="team_id"/>
                <field name="user_id"/>
                <field name="tag_ids" widget="many2many_tags"/>
                <field name="state" readonly="1" decoration-info="state == 'draft'" decoration-success="state == 'done'" decoration-danger="state == 'error'" widget="badge"/>
            </tree>
        </field>
    </record>

    <record id="crm_iap_lead_mining_request_search" model="ir.ui.view">
        <field name="name">crm.iap.lead.mining.request.search</field>
        <field name="model">crm.iap.lead.mining.request</field>
        <field name="arch" type="xml">
            <search string="Lead Mining Request">
                <field name="name"/>
                <field name="team_id"/>
                <field name="user_id"/>
                <field name="tag_ids"/>
                <filter name="state_is_draft" string="Draft" domain="[('state', '=', 'draft')]"/>
                <filter name="state_is_done" string="Done" domain="[('state', '=', 'done')]"/>
                <filter name="state_is_error" string="Error" domain="[('state', '=', 'error')]"/>
                <separator/>
                <filter name="type_is_lead" string="Leads" domain="[('lead_type', '=', 'lead')]"/>
                <filter name="type_is_opportunity" string="Opportunities" domain="[('lead_type', '=', 'opportunity')]"/>
                <group expand="0" string="Group By">
                    <filter string="Type" name="groupby_lead_type" domain="[]" context="{'group_by':'lead_type'}"/>
                    <filter string="Sales Team" name="groupby_team_id" domain="[]" context="{'group_by':'team_id'}"/>
                    <filter string="Salesperson" name="groupby_user_id" domain="[]" context="{'group_by':'user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="crm_iap_lead_mining_request_action" model="ir.actions.act_window">
        <field name="name">Lead Mining Requests</field>
        <field name="res_model">crm.iap.lead.mining.request</field>
        <field name="view_mode">tree,form</field>
    </record>

    <!-- This menu is display in CRM app when crm_iap_lead is installed-->
    <menuitem
        id="crm_menu_lead_generation"
        name="Lead Generation"
        parent="crm.crm_menu_config"
        sequence="20"/>

    <menuitem
        id="crm_iap_lead_mining_request_menu_action"
        action="crm_iap_lead_mining_request_action"
        parent="crm_menu_lead_generation"
        sequence="0"/>

</odoo>

```

## File: views\crm_lead_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <record id="crm_iap_opportunity_tree" model="ir.ui.view">
        <field name="name">crm.opportunity.inherited.tree</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_case_tree_view_oppor" />
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="js_class">crm_iap_lead_mining_request_tree</attribute>
            </xpath>
        </field>
    </record>

    <record id="crm_iap_opportunity_kanban" model="ir.ui.view">
        <field name="name">crm.opportunity.inherited.kanban</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_case_kanban_view_leads" />
        <field name="arch" type="xml">
            <xpath expr="//kanban" position="attributes">
                <attribute name="js_class">crm_iap_lead_mining_request_kanban</attribute>
            </xpath>
        </field>
    </record>

    <record id="crm_iap_lead_tree" model="ir.ui.view">
        <field name="name">crm.lead.inherited.tree</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_case_tree_view_leads" />
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="js_class">crm_iap_lead_mining_request_tree</attribute>
            </xpath>
        </field>
    </record>

    <record id="crm_iap_lead_kanban" model="ir.ui.view">
        <field name="name">crm.lead.inherited.kanban</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.view_crm_lead_kanban" />
        <field name="arch" type="xml">
            <xpath expr="//kanban" position="attributes">
                <attribute name="js_class">crm_iap_lead_mining_request_kanban</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <template id="enrich_company" inherit_id="iap_mail.enrich_company">
        <xpath expr="//div[hasclass('o_partner_autocomplete_enrich_info')]" position="inside">
            <t t-if="people_data">
                <div style="font-size:16px; margin: 9px 0;">
                    <b>Contacts</b>
                </div>
                <table style="width:100%;
                    border-top-style: solid;border-top-color: #eeeeee;border-top-width: 1px;
                    border-bottom-style: solid;border-bottom-color: #eeeeee;border-bottom-width: 1px;
                    border-left-style: solid;border-left-color: #eeeeee;border-left-width: 1px;
                    border-right-style: solid;border-right-color: #eeeeee;border-right-width: 1px;" t-if="people_data">
                    <thead>
                        <tr style="background-color: #eeeeee">
                            <th style="padding: 5px; width: 20%;">
                                Name
                            </th>
                            <th style="padding: 5px; width: 20%;">
                                Title
                            </th>
                            <th style="padding: 5px; width: 30%;">
                                <img style="vertical-align: text-top;" src="web_editor/font_to_img/61664/rgb(102,102,102)/13"/>
                                Email
                            </th>
                            <th style="padding: 5px; width: 30%;">
                                <img style="vertical-align: text-top;" src="web_editor/font_to_img/61589/rgb(102,102,102)/13"/>
                                Phone
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="people_data" t-as="people">
                            <tr t-att-style="people_odd and 'background-color:#eeeeee' or None">
                                <td style="padding: 5px">
                                    <t t-esc="people['full_name'] or ''"/>
                                </td>
                                <td style="padding: 5px">
                                    <t t-esc="people['title'] or ''"/>
                                </td>
                                <td style="padding: 5px">
                                    <a t-if="people['email']" t-attf-href="mailto:{{people['email']}}" target="_top">
                                        <t t-esc="people['email']"/>
                                    </a>
                                </td>
                                <td style="padding: 5px">
                                    <a t-if="people['phone']" t-attf-href="tel:{{people['phone']}}">
                                        <t t-esc="people['phone']"/>
                                    </a>
                                </td>
                            </tr>
                        </t>
                    </tbody>
                </table>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.crm.iap.lead</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="crm.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="crm_iap_lead_settings" position="inside">
                <widget name="iap_buy_more_credits" service_name="reveal"/>
            </div>
        </field>
    </record>
</odoo>

```

