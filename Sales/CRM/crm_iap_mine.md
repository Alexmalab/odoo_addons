# Odoo Module: crm_iap_mine

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
    'category': 'Sales/CRM',
    'version': '1.2',
    'depends': [
        'iap_crm',
        'iap_mail',
    ],
    'data': [
        'data/crm.iap.lead.industry.csv',
        'data/crm.iap.lead.role.csv',
        'data/crm.iap.lead.seniority.csv',
        'data/mail_template_data.xml',
        'data/ir_sequence_data.xml',
        'security/ir.model.access.csv',
        'views/crm_lead_views.xml',
        'views/crm_iap_lead_mining_request_views.xml',
        'views/res_config_settings_views.xml',
        'views/mail_templates.xml',
        'views/crm_menus.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'crm_iap_mine/static/src/js/**/*',
            'crm_iap_mine/static/src/views/*.js',
            'crm_iap_mine/static/src/views/*.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\crm.iap.lead.industry.csv

```csv
"id",name,reveal_ids,sequence
"crm_iap_mine_industry_30_155","Consumer Discretionary","30,155",1
"crm_iap_mine_industry_33","Consumer Staples","33",4
"crm_iap_mine_industry_69_157","Banks & Insurance","69,157",2
"crm_iap_mine_industry_86","Media","86",7
"crm_iap_mine_industry_114","Real Estate","114",21
"crm_iap_mine_industry_136","Transportation","136",16
"crm_iap_mine_industry_138_156","Energy & Utilities ","138",22
"crm_iap_mine_industry_148","Materials","148",17
"crm_iap_mine_industry_149","Telecommunication Services","149",20
"crm_iap_mine_industry_150_151","Consumer Services","150,151",9
"crm_iap_mine_industry_152","Retailing","152",3
"crm_iap_mine_industry_153_154","Food, Beverage & Tobacco","153,154",12
"crm_iap_mine_industry_158_159","Diversified Financials & Financial Services","158,159",15
"crm_iap_mine_industry_160","Health Care Equipment & Services","160",11
"crm_iap_mine_industry_161","Pharmaceuticals, Biotechnology & Life Sciences","161",13
"crm_iap_mine_industry_162","Capital Goods","162",6
"crm_iap_mine_industry_163","Commercial & Professional Services","163",5
"crm_iap_mine_industry_165","Software & Services","165",8
"crm_iap_mine_industry_166","Technology Hardware & Equipment","166",19
"crm_iap_mine_industry_167","Construction Materials","167",23
"crm_iap_mine_industry_168","Independent Power and Renewable Electricity Producers","168",14
"crm_iap_mine_industry_238","Automobiles & Components","238",18
"crm_iap_mine_industry_239","Consumer Durables & Apparel","239",10

```

## File: data\crm.iap.lead.role.csv

```csv
"id",name,reveal_id
"crm_iap_mine_role_1","CEO","CEO"
"crm_iap_mine_role_2","communications","communications"
"crm_iap_mine_role_3","consulting","consulting"
"crm_iap_mine_role_4","customer_service","customer_service"
"crm_iap_mine_role_5","education","education"
"crm_iap_mine_role_6","engineering","engineering"
"crm_iap_mine_role_7","finance","finance"
"crm_iap_mine_role_8","founder","founder"
"crm_iap_mine_role_9","health_professional","health_professional"
"crm_iap_mine_role_10","human_resources","human_resources"
"crm_iap_mine_role_11","information_technology","information_technology"
"crm_iap_mine_role_12","legal","legal"
"crm_iap_mine_role_13","marketing","marketing"
"crm_iap_mine_role_14","operations","operations"
"crm_iap_mine_role_15","owner","owner"
"crm_iap_mine_role_16","president","president"
"crm_iap_mine_role_17","product","product"
"crm_iap_mine_role_18","public_relations","public_relations"
"crm_iap_mine_role_19","real_estate","real_estate"
"crm_iap_mine_role_20","recruiting","recruiting"
"crm_iap_mine_role_21","research","research"
"crm_iap_mine_role_22","sale","sale"

```

## File: data\crm.iap.lead.seniority.csv

```csv
"id",name,reveal_id
"crm_iap_mine_seniority_1","director","director"
"crm_iap_mine_seniority_2","executive","executive"
"crm_iap_mine_seniority_3","manager","manager"

```

## File: data\ir_sequence_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Sequences for lead mining requests -->
        <record id="ir_sequence_crm_iap_mine" model="ir.sequence">
            <field name="name">Lead Mining Request</field>
            <field name="code">crm.iap.lead.mining.request</field>
            <field name="prefix">LMR</field>
            <field name="padding">3</field>
            <field name="company_id" eval="False"/>
        </record>

    </data>
</odoo>

```

## File: data\mail_template_data.xml

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

## File: models\crm_iap_lead_helpers.py

```python
from math import floor, log10
from odoo import api, models


class CRMHelpers(models.Model):
    _name = 'crm.iap.lead.helpers'
    _description = 'Helper methods for crm_iap_mine modules'

    @api.model
    def notify_no_more_credit(self, service_name, model_name, notification_parameter):
        """
        Notify about the number of credit.
        In order to avoid to spam people each hour, an ir.config_parameter is set
        """
        already_notified = self.env['ir.config_parameter'].sudo().get_param(notification_parameter, False)
        if already_notified:
            return
        mail_template = self.env.ref('crm_iap_mine.lead_generation_no_credits')
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

## File: models\crm_iap_lead_industry.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class CrmIapLeadIndustry(models.Model):
    """ Industry Tags of Acquisition Rules """
    _name = 'crm.iap.lead.industry'
    _description = 'CRM IAP Lead Industry'
    _order = 'sequence,id'

    name = fields.Char(string='Industry', required=True, translate=True)
    reveal_ids = fields.Char(required=True) # The list of reveal_ids for this industry, separated with ','
    color = fields.Integer(string='Color Index')
    sequence = fields.Integer('Sequence')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'Industry name already exists!'),
    ]

```

## File: models\crm_iap_lead_mining_request.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, _
from odoo.addons.iap.tools import iap_tools
from odoo.exceptions import UserError

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
    state = fields.Selection([('draft', 'Draft'), ('error', 'Error'), ('done', 'Done')], string='Status', required=True, default='draft')

    # Request Data
    lead_number = fields.Integer(string='Number of Leads', required=True, default=3)
    search_type = fields.Selection([('companies', 'Companies'), ('people', 'Companies and their Contacts')], string='Target', required=True, default='companies')
    error_type = fields.Selection([
        ('credits', 'Insufficient Credits'),
        ('no_result', 'No Result'),
    ], string='Error Type', copy=False, readonly=True)

    # Lead / Opportunity Data

    lead_type = fields.Selection([('lead', 'Leads'), ('opportunity', 'Opportunities')], string='Type', required=True, default=_default_lead_type)
    display_lead_label = fields.Char(compute='_compute_display_lead_label')
    team_id = fields.Many2one(
        'crm.team', string='Sales Team', ondelete="set null",
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
    available_state_ids = fields.One2many('res.country.state', compute='_compute_available_state_ids')
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
            leads_data = self.env['crm.lead']._read_group(
                [('lead_mining_request_id', 'in', self.ids)],
                ['lead_mining_request_id'], ['lead_mining_request_id'])
        else:
            leads_data = []
        mapped_data = dict(
            (m['lead_mining_request_id'][0], m['lead_mining_request_id_count'])
            for m in leads_data)
        for request in self:
            request.lead_count = mapped_data.get(request.id, 0)

    @api.depends('user_id', 'lead_type')
    def _compute_team_id(self):
        """ When changing the user, also set a team_id or restrict team id
        to the ones user_id is member of. """
        for mining in self:
            # setting user as void should not trigger a new team computation
            if not mining.user_id:
                continue
            user = mining.user_id
            if mining.team_id and user in mining.team_id.member_ids | mining.team_id.user_id:
                continue
            team_domain = [('use_leads', '=', True)] if mining.lead_type == 'lead' else [('use_opportunities', '=', True)]
            team = self.env['crm.team']._get_default_team_id(user_id=user.id, domain=team_domain)
            mining.team_id = team.id

    @api.depends('country_ids')
    def _compute_available_state_ids(self):
        """ States for some specific countries should not be offered as filtering options because
        they drastically reduce the amount of IAP reveal results.

        For example, in Belgium, only 11% of companies have a defined state within the
        reveal service while the rest of them have no state defined at all.

        Meaning specifying states for that country will yield a lot less results than what you could
        expect, which is not the desired behavior.
        Obviously all companies are active within a state, it's just a lack of data in the reveal
        service side.

        To help users create meaningful iap searches, we only keep the states filtering for several
        whitelisted countries (based on their country code).
        The complete list and reasons for this change can be found on task-2471703. """

        for lead_mining_request in self:
            countries = lead_mining_request.country_ids.filtered(lambda country:
                country.code in iap_tools._STATES_FILTER_COUNTRIES_WHITELIST)
            lead_mining_request.available_state_ids = self.env['res.country.state'].search([
                ('country_id', 'in', countries.ids)
            ])

    @api.onchange('available_state_ids')
    def _onchange_available_state_ids(self):
        self.state_ids -= self.state_ids.filtered(
            lambda state: (state._origin.id or state.id) not in self.available_state_ids.ids
        )

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

    @api.model
    def get_empty_list_help(self, help_string):
        help_title = _('Create a Lead Mining Request')
        sub_title = _('Generate new leads based on their country, industry, size, etc.')
        return '<p class="o_view_nocontent_smiling_face">%s</p><p class="oe_view_nocontent_alias">%s</p>' % (help_title, sub_title)

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
            # accumulate all reveal_ids (separated by ',') into one list
            # eg: 3 records with values: "175,176", "177" and "190,191"
            # will become ['175','176','177','190','191']
            all_industry_ids = [
                reveal_id.strip()
                for reveal_ids in self.mapped('industry_ids.reveal_ids')
                for reveal_id in reveal_ids.split(',')
            ]
            payload['industry_ids'] = all_industry_ids
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
        The user will be notified if they don't have enough credits.
        """
        self.error_type = False
        server_payload = self._prepare_iap_payload()
        reveal_account = self.env['iap.account'].get('reveal')
        dbuuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        params = {
            'account_token': reveal_account.account_token,
            'dbuuid': dbuuid,
            'data': server_payload
        }
        try:
            response = self._iap_contact_mining(params, timeout=300)
            if not response.get('data'):
                self.error_type = 'no_result'
                return False

            return response['data']
        except iap_tools.InsufficientCreditError as e:
            self.error_type = 'credits'
            self.state = 'error'
            return False
        except Exception as e:
            raise UserError(_("Your request could not be executed: %s", e))

    def _iap_contact_mining(self, params, timeout=300):
        endpoint = self.env['ir.config_parameter'].sudo().get_param('reveal.endpoint', DEFAULT_ENDPOINT) + '/iap/clearbit/1/lead_mining_request'
        return iap_tools.iap_jsonrpc(endpoint, params=params, timeout=timeout)

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
        elif self.env.context.get('is_modal'):
            # when we are inside a modal already, we re-open the same record
            # that way, the form view is updated and the correct error message appears
            # (sadly, there is no way to simply 'reload' a form view within a modal)
            return {
                'name': _('Generate Leads'),
                'res_model': 'crm.iap.lead.mining.request',
                'views': [[False, 'form']],
                'target': 'new',
                'type': 'ir.actions.act_window',
                'res_id': self.id,
                'context': dict(self.env.context, edit=True, form_view_initial_mode='edit')
            }
        else:
            # will reload the form view and show the error message on top
            return False

    def action_get_lead_action(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_all_leads")
        action['domain'] = [('id', 'in', self.lead_ids.ids), ('type', '=', 'lead')]
        return action

    def action_get_opportunity_action(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_opportunities")
        action['domain'] = [('id', 'in', self.lead_ids.ids), ('type', '=', 'opportunity')]
        return action

    def action_buy_credits(self):
        return {
            'type': 'ir.actions.act_url',
            'url': self.env['iap.account'].get_credits_url(service_name='reveal'),
        }

```

## File: models\crm_iap_lead_role.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


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

```

## File: models\crm_iap_lead_seniority.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


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

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Lead(models.Model):
    _inherit = 'crm.lead'

    lead_mining_request_id = fields.Many2one('crm.iap.lead.mining.request', string='Lead Mining Request', index='btree_not_null')

    def _merge_get_fields(self):
        return super(Lead, self)._merge_get_fields() + ['lead_mining_request_id']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import crm_iap_lead_helpers
from . import crm_iap_lead_industry
from . import crm_iap_lead_role
from . import crm_iap_lead_seniority
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

## File: static\src\js\tours\crm_iap_lead.js

```javascript
odoo.define('crm_iap_mine.generate_leads_steps', function (require) {
"use strict";

var tour = require('web_tour.tour');
const {Markup} = require('web.utils');
var core = require('web.core');

require('@crm/js/tours/crm');
var _t = core._t;

var DragOppToWonStepIndex = _.findIndex(tour.tours.crm_tour.steps, function (step) {
    return (step.id === 'drag_opportunity_to_won_step');
});

tour.tours.crm_tour.steps.splice(DragOppToWonStepIndex + 1, 0, {
    /**
     * Add some steps between "Drag your opportunity to <b>Won</b> when you get
     * the deal. Congrats !" and "Let’s have a look at an Opportunity." to
     * include the steps related to the lead generation (crm_iap_mine).
     * This eases the on boarding for the Lead Generation process.
     *
     */
    trigger: ".o_button_generate_leads",
    content: Markup(_t("Looking for more opportunities ?<br>Try the <b>Lead Generation</b> tool.")),
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

## File: static\src\views\generate_leads_hook.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";

const { onWillStart, useComponent } = owl;

export function useGenerateLeadsButton() {
    const component = useComponent();
    const user = useService("user");
    const action = useService("action");

    onWillStart(async () => {
        component.isSalesManager = await user.hasGroup("sales_team.group_sale_manager");
    });

    component.onClickGenerateLead = () => {
        const leadType = component.props.context.default_type;
        action.doAction({
            name: "Generate Leads",
            type: "ir.actions.act_window",
            res_model: "crm.iap.lead.mining.request",
            target: "new",
            views: [[false, "form"]],
            context: { is_modal: true, default_lead_type: leadType },
        });
    };
}

```

## File: static\src\views\generate_leads_views.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { patch } from "@web/core/utils/patch";
import { useGenerateLeadsButton } from "@crm_iap_mine/views/generate_leads_hook";
import { crmKanbanView } from "@crm/views/crm_kanban/crm_kanban_view";
import { ListController } from "@web/views/list/list_controller";
import { listView } from "@web/views/list/list_view";

export class LeadMiningRequestListController extends ListController {
    setup() {
        super.setup();
        useGenerateLeadsButton();
    }
}

registry.category("views").add("crm_iap_lead_mining_request_tree", {
    ...listView,
    Controller: LeadMiningRequestListController,
    buttonTemplate: "LeadMiningRequestListView.buttons",
});

patch(crmKanbanView.Controller.prototype, "crm_iap_lead_mining_request_kanban", {
    setup() {
        this._super(...arguments);
        useGenerateLeadsButton();
    },
});
crmKanbanView.buttonTemplate = "LeadMiningRequestKanbanView.buttons";

```

## File: static\src\views\generate_leads_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="LeadMiningRequest.generate_leads_button" owl="1">
        <button t-if="isSalesManager" type="button" class="btn btn-secondary o_button_generate_leads" t-on-click="onClickGenerateLead">
            Generate Leads
        </button>
    </t>

    <t t-name="LeadMiningRequestListView.buttons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary" owl="1">
        <!-- Before the export button -->
        <xpath expr="//t[contains(@t-if, 'isExportEnable')]" position="before">
            <t t-call="LeadMiningRequest.generate_leads_button"/>
        </xpath>
    </t>

    <t t-name="LeadMiningRequestKanbanView.buttons" t-inherit="web.KanbanView.Buttons" t-inherit-mode="primary" owl="1">
        <xpath expr="//div[@t-if='props.showButtons']" position="inside">
            <t t-call="LeadMiningRequest.generate_leads_button"/>
        </xpath>
    </t>
</templates>

```

## File: views\crm_iap_lead_mining_request_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_iap_lead_mining_request_view_form" model="ir.ui.view">
        <field name="name">crm.iap.lead.mining.request.view.form</field>
        <field name="model">crm.iap.lead.mining.request</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <button name="action_submit" type="object" string="Submit" states="draft" class="oe_highlight" invisible="context.get('is_modal')"/>
                    <button name="action_submit" type="object" string="Retry" states="error" class="oe_highlight" invisible="context.get('is_modal')"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,done" invisible="context.get('is_modal')"/>
                </header>
                <div class="alert alert-danger text-center my-0" role="alert" attrs="{'invisible': [('error_type', '=', False)]}">
                    <field name="error_type" invisible="1"/>
                    <field name="lead_type" invisible="1"/>
                    <span attrs="{'invisible': [('error_type', '!=', 'credits')]}">
                        <span>You do not have enough credits to submit this request.
                            <button name="action_buy_credits" type="object" class="oe_inline p-0 border-0 align-top text-primary">Buy credits.</button>
                        </span>
                    </span>
                    <span attrs="{'invisible': [('error_type', '!=', 'no_result')]}">Your request did not return any result (no credits were used). Try removing some filters.</span>
                </div>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_get_opportunity_action" class="oe_stat_button" type="object" icon="fa-star" attrs="{'invisible': ['|', ('lead_type', '!=', 'opportunity'), ('state', '!=' , 'done')]}">
                            <div class="o_stat_info">
                                <field name="lead_count"/>
                                <span class="o_stat_text">Opportunities</span>
                            </div>
                        </button>
                        <button name="action_get_lead_action" class="oe_stat_button" type="object" icon="fa-star" groups="crm.group_use_lead" attrs="{'invisible': ['|', ('lead_type', '!=', 'lead'), ('state', '!=' , 'done')]}">
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
                            <div  class="o_row py-3">
                                <field name="lead_number" attrs="{'readonly': [('state', '=', 'done')]}" nolabel="1" class="oe_inline pe-2 o_input_3ch"/>
                                <field name="search_type" widget="selection" attrs="{'readonly': [('state', '=', 'done')]}" nolabel="1" class="oe_inline"/>
                            </div>
                        </group>
                    </group>

                    <group>
                        <group name="companies">
                            <field name="country_ids" widget="many2many_tags" attrs="{'readonly': [('state', '=', 'done')]}" required="True" options="{'no_create': True, 'no_open': True}"/>
                            <field name="available_state_ids" invisible="1"/>
                            <field name="state_ids" widget="many2many_tags"
                                attrs="{'invisible': ['|', ('country_ids', '=', []), ('available_state_ids', '=', [])], 'readonly': [('state', '=', 'done')]}"
                                domain="[('id', 'in', available_state_ids)]" options="{'no_create': True, 'no_open': True}"/>
                            <field name="industry_ids" widget="many2many_tags" attrs="{'readonly': [('state', '=', 'done')]}" options="{'no_create': True, 'no_open': True}" class="o_industry" required="1"/>
                            <field name="filter_on_size" attrs="{'readonly': [('state', '=', 'done')]}"/>
                            <label for="company_size_min" attrs="{'invisible': [('filter_on_size', '=', False)]}"/>
                            <div attrs="{'invisible': [('filter_on_size', '=', False)]}">
                                From
                                <field name="company_size_min" class="oe_inline px-2 o_input_5ch" attrs="{'required': [('filter_on_size', '=', True)], 'readonly': [('state', '=', 'done')]}"/>
                                to
                                <field name="company_size_max" class="oe_inline px-2 o_input_7ch" attrs="{'required': [('filter_on_size', '=', True)], 'readonly': [('state', '=', 'done')]}"/>
                                employees
                            </div>

                        </group>
                        <group name="lead_info">
                            <field name="lead_type" groups="crm.group_use_lead" invisible="context.get('is_modal')" attrs="{'readonly': [('state', '=', 'done')]}"/>
                            <field name="team_id" no_create="1" no_open="1" attrs="{'readonly': [('state', '=', 'done')]}" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
                            <field name="user_id" no_create="1" no_open="1" attrs="{'readonly': [('state', '=', 'done')]}"/>
                            <field name="tag_ids" string="Default Tags" widget="many2many_tags" attrs="{'readonly': [('state', '=', 'done')]}"/>
                        </group>
                    </group>

                    <group name="contacts" attrs="{'invisible': [('search_type', '!=', 'people')]}">
                        <div colspan="2" class="py-3">
                             <field name="contact_number" attrs="{'readonly': [('state', '=', 'done')], 'required': [('search_type', '=', 'people')]}" nolabel="1" class="oe_inline pe-2 col-md-1 o_input_2ch"/>
                             <span class="col-md-6">Extra contacts per Company</span>
                        </div>
                    </group>
                    <group attrs="{'invisible': [('search_type', '!=', 'people')]}">
                        <group>
                            <field name="contact_filter_type" widget="radio" attrs="{'readonly': [('state', '=', 'done')]}" options="{'horizontal': true}"/>
                            <field name="preferred_role_id" options="{'no_create_edit': True, 'no_quick_create': True, 'no_open': True}" attrs="{'invisible': [('contact_filter_type','!=', 'role')], 'required': [('search_type', '=', 'people'), ('contact_filter_type', '=', 'role')], 'readonly': [('state', '=', 'done')]}"/>
                            <field name="role_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True, 'no_quick_create': True}" attrs="{'invisible': ['|', ('preferred_role_id','=', False), ('contact_filter_type','!=', 'role')], 'readonly': [('state', '=', 'done')]}"/>
                            <field name="seniority_id" options="{'no_create_edit': True, 'no_quick_create': True, 'no_open': True}" attrs="{'invisible': [('contact_filter_type', '!=', 'seniority')], 'required': [('search_type', '=', 'people'), ('contact_filter_type', '=', 'seniority')], 'readonly': [('state', '=', 'done')]}"/>
                        </group>
                    </group>
                    <footer>
                        <button string="Generate Leads" name="action_submit" type="object" default_focus="1" class="btn-primary" invisible="not context.get('is_modal')" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" invisible="not context.get('is_modal')"/>
                    </footer>
                </sheet>
            </form>
        </field>
    </record>

    <record id="crm_iap_lead_mining_request_view_tree" model="ir.ui.view">
        <field name="name">crm.iap.lead.mining.request.view.tree</field>
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

    <record id="crm_iap_lead_mining_request_view_search" model="ir.ui.view">
        <field name="name">crm.iap.lead.mining.request.view.search</field>
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
</odoo>

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <record id="crm_lead_view_tree_opportunity" model="ir.ui.view">
        <field name="name">crm.lead.view.tree.opportunity.inherit.iap.mine</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_case_tree_view_oppor" />
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="js_class">crm_iap_lead_mining_request_tree</attribute>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_view_tree_lead" model="ir.ui.view">
        <field name="name">crm.lead.view.tree.lead.inherit.iap.mine</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_case_tree_view_leads" />
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="js_class">crm_iap_lead_mining_request_tree</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\crm_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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

## File: views\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <template id="enrich_company" inherit_id="iap_mail.enrich_company">
        <xpath expr="//div[hasclass('o_partner_autocomplete_enrich_info')]" position="inside">
            <t t-if="people_data">
                <t t-set="hasPhoneNumbers" t-value="any([people['phone'] for people in people_data])"/>
                <div style="font-size:16px; margin: 9px 0;">
                    <b>Contacts</b>
                </div>
                <table style="width:100%;
                    border-top-style: solid;border-top-color: #eeeeee;border-top-width: 1px;
                    border-bottom-style: solid;border-bottom-color: #eeeeee;border-bottom-width: 1px;
                    border-start-style: solid;border-start-color: #eeeeee;border-start-width: 1px;
                    border-end-style: solid;border-end-color: #eeeeee;border-end-width: 1px;" t-if="people_data">
                    <thead>
                        <tr style="background-color: #eeeeee">
                            <th t-attf-style="padding: 5px; width: {{hasPhoneNumbers and '30%;' or '50%;'}}">
                                <img style="vertical-align: text-top;" src="web_editor/font_to_img/61447/rgb(102,102,102)/13"/>
                                Name
                            </th>
                            <th t-attf-style="padding: 5px; width: {{hasPhoneNumbers and '40%;' or '50%;'}}">
                                <img style="vertical-align: text-top;" src="web_editor/font_to_img/61664/rgb(102,102,102)/13"/>
                                Email
                            </th>
                            <th style="padding: 5px; width: 30%;" t-if="hasPhoneNumbers">
                                <img style="vertical-align: text-top;" src="web_editor/font_to_img/61589/rgb(102,102,102)/13"/>
                                Phone
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="people_data" t-as="people">
                            <tr t-att-style="people_odd and 'background-color:#eeeeee' or None">
                                <td style="padding: 5px">
                                    <t t-set="fullName" t-value="people['full_name'] or ''"/>
                                    <t t-set="title" t-value="people['title'] or ''"/>
                                    <t t-if="fullName">
                                        <h5 style="margin: 0;"><t t-esc="fullName"/><t t-if="title">,</t></h5>
                                    </t>
                                    <small t-esc="title" t-if="title"/>
                                </td>
                                <td style="padding: 5px">
                                    <a t-if="people['email']" t-attf-href="mailto:{{people['email']}}" target="_top">
                                        <t t-esc="people['email']"/>
                                    </a>
                                </td>
                                <td style="padding: 5px" t-if="hasPhoneNumbers">
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
            <div id="crm_iap_mine_settings" position="inside">
                <widget name="iap_buy_more_credits" service_name="reveal"/>
            </div>
        </field>
    </record>
</odoo>

```

