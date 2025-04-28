# Odoo Module: website_crm_iap_reveal

Category: Sales/CRM

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
    'name': 'Lead Generation From Website Visits',
    'summary': 'Generate Leads/Opportunities from your website\'s traffic',
    'version': '1.1',
    'category': 'Sales/CRM',
    'depends': [
        'iap_crm',
        'iap_mail',
        'crm_iap_mine',
        'website_crm',
    ],
    'data': [
        'data/ir_cron_data.xml',
        'data/ir_model_data.xml',
        'security/ir_rules.xml',
        'security/ir.model.access.csv',
        'views/crm_lead_views.xml',
        'views/crm_reveal_views.xml',
        'views/res_config_settings_views.xml',
        'views/crm_menus.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\website_form.py

```python
# -*- coding: utf-8 -*-
import json

from odoo import http
from odoo.addons.website.controllers.form import WebsiteForm
from odoo.http import request


class ContactController(WebsiteForm):

    def _handle_website_form(self, model_name, **kwargs):
        if model_name == 'crm.lead':
            # Add the ip_address to the request in order to add this to the lead
            # that will be created. With this, we avoid to create a lead from
            # reveal if a lead is already created from the contact form.
            request.params['reveal_ip'] = request.httprequest.remote_addr

        return super(ContactController, self)._handle_website_form(model_name, **kwargs)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website_form

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Scheduler for Lead Generation -->
        <record id="ir_cron_crm_reveal_lead" model="ir.cron">
            <field name="name">Lead Generation: Leads/Opportunities Generation</field>
            <field name="model_id" ref="model_crm_reveal_rule"/>
            <field name="state">code</field>
            <field name="code">model._process_lead_generation()</field>
            <field name="user_id" ref="base.user_root"/>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
        </record>
    </data>
</odoo>

```

## File: data\ir_model_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <function model="ir.model.fields" name="formbuilder_whitelist">
        <value>crm.lead</value>
        <value eval="['reveal_ip']"/>
    </function>
</odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Lead(models.Model):
    _inherit = 'crm.lead'

    reveal_ip = fields.Char(string='IP Address')
    reveal_iap_credits = fields.Integer(string='IAP Credits')
    reveal_rule_id = fields.Many2one('crm.reveal.rule', string='Lead Generation Rule', index='btree_not_null')

    def _merge_get_fields(self):
        return super(Lead, self)._merge_get_fields() + ['reveal_ip', 'reveal_iap_credits', 'reveal_rule_id']

```

## File: models\crm_reveal_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import itertools
import logging
import re
from dateutil.relativedelta import relativedelta

import odoo
from odoo import api, fields, models, tools, _
from odoo.addons.iap.tools import iap_tools
from odoo.addons.crm.models import crm_stage
from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)

DEFAULT_ENDPOINT = 'https://iap-services.odoo.com'
DEFAULT_REVEAL_BATCH_LIMIT = 25
DEFAULT_REVEAL_MONTH_VALID = 6

class CRMRevealRule(models.Model):
    _name = 'crm.reveal.rule'
    _description = 'CRM Lead Generation Rules'
    _order = 'sequence'

    name = fields.Char(string='Rule Name', required=True)
    active = fields.Boolean(default=True)

    # Website Traffic Filter
    country_ids = fields.Many2many('res.country', string='Countries', help='Only visitors of following countries will be converted into leads/opportunities (using GeoIP).')
    website_id = fields.Many2one('website', help='Restrict Lead generation to this website.')
    state_ids = fields.Many2many('res.country.state', string='States', help='Only visitors of following states will be converted into leads/opportunities.')
    regex_url = fields.Char(string='URL Expression', help='Regex to track website pages. Leave empty to track the entire website, or / to target the homepage. Example: /page* to track all the pages which begin with /page')
    sequence = fields.Integer(help='Used to order the rules with same URL and countries. '
                                   'Rules with a lower sequence number will be processed first.')

    # Company Criteria Filter
    industry_tag_ids = fields.Many2many('crm.iap.lead.industry', string='Industries', help='Leave empty to always match. Odoo will not create lead if no match')
    filter_on_size = fields.Boolean(string="Filter on Size", default=True, help="Filter companies based on their size.")
    company_size_min = fields.Integer(string='Company Size', default=0)
    company_size_max = fields.Integer(default=1000)

    # Contact Generation Filter
    contact_filter_type = fields.Selection([('role', 'Role'), ('seniority', 'Seniority')], string="Filter On", required=True, default='role')
    preferred_role_id = fields.Many2one('crm.iap.lead.role', string='Preferred Role')
    other_role_ids = fields.Many2many('crm.iap.lead.role', string='Other Roles')
    seniority_id = fields.Many2one('crm.iap.lead.seniority', string='Seniority')
    extra_contacts = fields.Integer(string='Number of Contacts', help='This is the number of contacts to track if their role/seniority match your criteria. Their details will show up in the history thread of generated leads/opportunities. One credit is consumed per tracked contact.', default=1)

    # Lead / Opportunity Data
    lead_for = fields.Selection([('companies', 'Companies'), ('people', 'Companies and their Contacts')], string='Data Tracking', required=True, default='companies', help='Choose whether to track companies only or companies and their contacts')
    lead_type = fields.Selection([('lead', 'Lead'), ('opportunity', 'Opportunity')], string='Type', required=True, default='opportunity')
    suffix = fields.Char(string='Suffix', help='This will be appended in name of generated lead so you can identify lead/opportunity is generated with this rule')
    team_id = fields.Many2one('crm.team', string='Sales Team', ondelete="set null")
    tag_ids = fields.Many2many('crm.tag', string='Tags')
    user_id = fields.Many2one('res.users', string='Salesperson')
    priority = fields.Selection(crm_stage.AVAILABLE_PRIORITIES, string='Priority')
    lead_ids = fields.One2many('crm.lead', 'reveal_rule_id', string='Generated Lead / Opportunity')
    lead_count = fields.Integer(compute='_compute_lead_count', string='Number of Generated Leads')
    opportunity_count = fields.Integer(compute='_compute_lead_count', string='Number of Generated Opportunity')

    # This limits the number of extra contact.
    # Even if more than 5 extra contacts provided service will return only 5 contacts (see service module for more)
    _sql_constraints = [
        ('limit_extra_contacts', 'check(extra_contacts >= 1 and extra_contacts <= 5)', 'Maximum 5 contacts are allowed!'),
    ]

    def _compute_lead_count(self):
        leads = self.env['crm.lead']._read_group([
            ('reveal_rule_id', 'in', self.ids)
        ], groupby=['reveal_rule_id', 'type'], aggregates=['__count'])
        mapping = {(reveal_rule.id, type_crm): count for reveal_rule, type_crm, count in leads}
        for rule in self:
            rule.lead_count = mapping.get((rule.id, 'lead'), 0)
            rule.opportunity_count = mapping.get((rule.id, 'opportunity'), 0)

    @api.constrains('regex_url')
    def _check_regex_url(self):
        try:
            if self.regex_url:
                re.compile(self.regex_url)
        except Exception:
            raise ValidationError(_('Enter Valid Regex.'))

    @api.model_create_multi
    def create(self, vals_list):
        self.env.registry.clear_cache() # Clear the cache in order to recompute _get_active_rules
        return super().create(vals_list)

    def write(self, vals):
        fields_set = {
            'country_ids', 'regex_url', 'active'
        }
        if set(vals.keys()) & fields_set:
            self.env.registry.clear_cache() # Clear the cache in order to recompute _get_active_rules
        return super(CRMRevealRule, self).write(vals)

    def unlink(self):
        self.env.registry.clear_cache() # Clear the cache in order to recompute _get_active_rules
        return super(CRMRevealRule, self).unlink()

    def action_get_lead_tree_view(self):
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_all_leads")
        action['domain'] = [('id', 'in', self.lead_ids.ids), ('type', '=', 'lead')]
        action['context'] = dict(self._context, create=False)
        return action

    def action_get_opportunity_tree_view(self):
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_opportunities")
        action['domain'] = [('id', 'in', self.lead_ids.ids), ('type', '=', 'opportunity')]
        action['context'] = dict(self._context, create=False)
        return action

    @api.model
    @tools.ormcache()
    def _get_active_rules(self):
        """
        Returns informations about the all rules.
        The return is in the form :
        {
            'country_rules': {
                'BE': [0, 1],
                'US': [0]
            },
            'rules': [
            {
                'id': 0,
                'regex': ***,
                'website_id': 1,
                'country_codes': ['BE', 'US'],
                'state_codes': [('BE', False), ('US', 'NY'), ('US', 'CA')]
            },
            {
                'id': 1,
                'regex': ***,
                'website_id': 1,
                'country_codes': ['BE'],
                'state_codes': [('BE', False)]
            }
            ]
        }
        """
        country_rules = {}
        rules_records = self.search([])
        rules = []
        # Fixes for special cases
        for rule in rules_records:
            regex_url = rule['regex_url']
            if not regex_url:
                regex_url = '.*'    # for all pages if url not given
            elif regex_url == '/':
                regex_url = '.*/$'  # for home
            countries = rule.country_ids.mapped('code')

            # First apply rules for any state in countries
            states = [(country_id.code, False) for country_id in rule.country_ids]
            if rule.state_ids:
                for state_id in rule.state_ids:
                    if (state_id.country_id.code, False) in states:
                        # Remove country because rule doesn't apply to any state
                        states.remove((state_id.country_id.code, False))
                    states += [(state_id.country_id.code, state_id.code)]
                
            rules.append({
                'id': rule.id,
                'regex': regex_url,
                'website_id': rule.website_id.id if rule.website_id else False,
                'country_codes': countries,
                'state_codes': states
            })
            for country in countries:
                country_rules = self._add_to_country(country_rules, country, len(rules) - 1)
        return {
            'country_rules': country_rules,
            'rules': rules,
        }

    def _add_to_country(self, country_rules, country, rule_index):
        """
        Add the rule index to the country code in the country_rules
        """
        if country not in country_rules:
            country_rules[country] = []
        country_rules[country].append(rule_index)
        return country_rules

    def _match_url(self, website_id, url, country_code, state_code, rules_excluded):
        """
        Return the matching rule based on the country, the website and URL.
        """
        all_rules = self._get_active_rules()
        rules_id = all_rules['country_rules'].get(country_code, [])

        rules_matched = []
        for rule_index in rules_id:
            rule = all_rules['rules'][rule_index]
            if ((country_code, state_code) in rule['state_codes'] or (country_code, False) in rule['state_codes'])\
                and (not rule['website_id'] or rule['website_id'] == website_id)\
                and str(rule['id']) not in rules_excluded\
                and re.search(rule['regex'], url):
                rules_matched.append(rule)
        return rules_matched

    @api.model
    def _process_lead_generation(self, autocommit=True):
        """ Cron Job for lead generation from page view """
        _logger.info('Start Reveal Lead Generation')
        self.env['crm.reveal.view']._clean_reveal_views()
        self._unlink_unrelevant_reveal_view()
        reveal_views = self._get_reveal_views_to_process()
        view_count = 0
        while reveal_views:
            view_count += len(reveal_views)
            server_payload = self._prepare_iap_payload(dict(reveal_views))
            enough_credit = self._perform_reveal_service(server_payload)
            if autocommit:
                # auto-commit for batch processing
                self._cr.commit()
            if enough_credit:
                reveal_views = self._get_reveal_views_to_process()
            else:
                reveal_views = False
        _logger.info('End Reveal Lead Generation - %s views processed', view_count)

    @api.model
    def _unlink_unrelevant_reveal_view(self):
        """
        We don't want to create the lead if in past (<6 months) we already
        created lead with given IP. So, we unlink crm.reveal.view with same IP
        as a already created lead.
        """
        months_valid = self.env['ir.config_parameter'].sudo().get_param('reveal.lead_month_valid', DEFAULT_REVEAL_MONTH_VALID)
        try:
            months_valid = int(months_valid)
        except ValueError:
            months_valid = DEFAULT_REVEAL_MONTH_VALID
        domain = []
        domain.append(('reveal_ip', '!=', False))
        domain.append(('create_date', '>', fields.Datetime.to_string(datetime.date.today() - relativedelta(months=months_valid))))
        leads = self.env['crm.lead'].with_context(active_test=False).search(domain)
        self.env['crm.reveal.view'].search([('reveal_ip', 'in', [lead.reveal_ip for lead in leads])]).unlink()

    @api.model
    def _get_reveal_views_to_process(self):
        """ Return list of reveal rule ids grouped by IPs """
        batch_limit = DEFAULT_REVEAL_BATCH_LIMIT
        query = """
            SELECT v.reveal_ip, array_agg(v.reveal_rule_id ORDER BY r.sequence)
            FROM crm_reveal_view v
            INNER JOIN crm_reveal_rule r
            ON v.reveal_rule_id = r.id
            WHERE v.reveal_state='to_process'
            GROUP BY v.reveal_ip
            LIMIT %s
            """

        self.env.cr.execute(query, [batch_limit])
        return self.env.cr.fetchall()

    def _prepare_iap_payload(self, pgv):
        """ This will prepare the page view and returns payload
            Payload sample
            {
                ips: {
                    '192.168.1.1': [1,4],
                    '192.168.1.6': [2,4]
                },
                rules: {
                    1: {rule_data},
                    2: {rule_data},
                    4: {rule_data}
                }
            }
        """
        new_list = list(set(itertools.chain.from_iterable(pgv.values())))
        rule_records = self.browse(new_list)
        return {
            'ips': pgv,
            'rules': rule_records._get_rules_payload()
        }

    def _get_rules_payload(self):
        company_country = self.env.company.country_id
        rule_payload = {}
        for rule in self:
            # accumulate all reveal_ids (separated by ',') into one list
            # eg: 3 records with values: "175,176", "177" and "190,191"
            # will become ['175','176','177','190','191']
            reveal_ids = [
                reveal_id.strip()
                for reveal_ids in rule.mapped('industry_tag_ids.reveal_ids')
                for reveal_id in reveal_ids.split(',')
            ]
            data = {
                'rule_id': rule.id,
                'lead_for': rule.lead_for,
                'countries': rule.country_ids.mapped('code'),
                'filter_on_size': rule.filter_on_size,
                'company_size_min': rule.company_size_min,
                'company_size_max': rule.company_size_max,
                'industry_tags': reveal_ids,
                'user_country': company_country and company_country.code or False
            }
            if rule.lead_for == 'people':
                data.update({
                    'contact_filter_type': rule.contact_filter_type,
                    'preferred_role': rule.preferred_role_id.reveal_id or '',
                    'other_roles': rule.other_role_ids.mapped('reveal_id'),
                    'seniority': rule.seniority_id.reveal_id or '',
                    'extra_contacts': rule.extra_contacts - 1
                })
            rule_payload[rule.id] = data
        return rule_payload

    def _perform_reveal_service(self, server_payload):
        result = False
        account_token = self.env['iap.account'].get('reveal')
        params = {
            'account_token': account_token.account_token,
            'data': server_payload
        }
        result = self._iap_contact_reveal(params, timeout=300)

        all_ips, done_ips = list(server_payload['ips'].keys()), []
        for res in result.get('reveal_data', []):
            done_ips.append(res['ip'])
            if not res.get('not_found'):
                lead = self._create_lead_from_response(res)
                self.env['crm.reveal.view'].search([('reveal_ip', '=', res['ip'])]).unlink()
            else:
                views = self.env['crm.reveal.view'].search([('reveal_ip', '=', res['ip'])])
                views.write({'reveal_state': 'not_found'})
                views.flush_recordset()

        if result.get('credit_error'):
            self.env['crm.iap.lead.helpers'].notify_no_more_credit('reveal', self._name, 'reveal.already_notified')
            return False
        else:
            # avoid loops if IAP return result is broken: otherwise some IP may create loops
            views = self.env['crm.reveal.view'].search([
                ('reveal_ip', 'in', [ip for ip in all_ips if ip not in done_ips])
            ])
            views.write({'reveal_state': 'not_found'})
            views.flush_recordset()
            # reset notified parameter to re-send credit notice if appears again
            self.env['ir.config_parameter'].sudo().set_param('reveal.already_notified', False)
        return True

    def _iap_contact_reveal(self, params, timeout=300):
        endpoint = self.env['ir.config_parameter'].sudo().get_param('reveal.endpoint', DEFAULT_ENDPOINT) + '/iap/clearbit/1/reveal'
        return iap_tools.iap_jsonrpc(endpoint, params=params, timeout=timeout)

    def _create_lead_from_response(self, result):
        """ This method will get response from service and create the lead accordingly """
        if result['rule_id']:
            rule = self.browse(result['rule_id'])
        else:
            # Not create a lead if the information match no rule
            # If there is no match, the service still returns all informations
            # in order to let custom code use it.
            return False
        if not result['clearbit_id']:
            return False
        already_created_lead = self.env['crm.lead'].search_count([('reveal_id', '=', result['clearbit_id'])], limit=1)
        if already_created_lead:
            _logger.info('Existing lead for this clearbit_id [%s]', result['clearbit_id'])
            # Does not create a lead if the reveal_id is already known
            return False
        lead_vals = rule._lead_vals_from_response(result)

        lead = self.env['crm.lead'].create(lead_vals)

        template_values = result['reveal_data']
        template_values.update({
            'flavor_text': _("Opportunity created by Odoo Lead Generation"),
            'people_data': result.get('people_data'),
        })
        lead.message_post_with_source(
            'iap_mail.enrich_company',
            render_values=template_values,
            subtype_xmlid='mail.mt_note',
        )

        return lead

    # Methods responsible for format response data in to valid odoo lead data
    def _lead_vals_from_response(self, result):
        self.ensure_one()
        company_data = result['reveal_data']
        people_data = result.get('people_data')
        lead_vals = self.env['crm.iap.lead.helpers'].lead_vals_from_response(self.lead_type, self.team_id.id, self.tag_ids.ids, self.user_id.id, company_data, people_data)

        lead_vals.update({
            'priority': self.priority,
            'reveal_ip': result['ip'],
            'reveal_rule_id': self.id,
            'referred': 'Website Visitor',
            'reveal_iap_credits': result['credit'],
        })

        if self.suffix:
            lead_vals['name'] = '%s - %s' % (lead_vals['name'], self.suffix)

        return lead_vals

```

## File: models\crm_reveal_view.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
from dateutil.relativedelta import relativedelta
from odoo import api, fields, models

DEFAULT_REVEAL_VIEW_WEEKS_VALID = 5

class CRMRevealView(models.Model):
    _name = 'crm.reveal.view'
    _description = 'CRM Reveal View'
    _rec_name = 'reveal_ip'
    _order = 'id desc'

    reveal_ip = fields.Char(string='IP Address')
    reveal_rule_id = fields.Many2one('crm.reveal.rule', string='Lead Generation Rule', index='btree_not_null')
    reveal_state = fields.Selection([('to_process', 'To Process'), ('not_found', 'Not Found')], default='to_process', string="State", index=True)
    create_date = fields.Datetime(index=True)

    def init(self):
        self._cr.execute('SELECT indexname FROM pg_indexes WHERE indexname = %s', ('crm_reveal_view_ip_rule_id',))
        if not self._cr.fetchone():
            self._cr.execute('CREATE UNIQUE INDEX crm_reveal_view_ip_rule_id ON crm_reveal_view (reveal_rule_id,reveal_ip)')
        self._cr.execute('SELECT indexname FROM pg_indexes WHERE indexname = %s', ('crm_reveal_view_state_create_date',))
        if not self._cr.fetchone():
            self._cr.execute('CREATE INDEX crm_reveal_view_state_create_date ON crm_reveal_view (reveal_state,create_date)')


    @api.model
    def _clean_reveal_views(self):
        """ Remove old views (> 1 month) """
        weeks_valid = self.env['ir.config_parameter'].sudo().get_param('reveal.view_weeks_valid', DEFAULT_REVEAL_VIEW_WEEKS_VALID)
        try:
            weeks_valid = int(weeks_valid)
        except ValueError:
            weeks_valid = DEFAULT_REVEAL_VIEW_WEEKS_VALID
        domain = []
        domain.append(('reveal_state', '=', 'not_found'))
        domain.append(('create_date', '<', fields.Datetime.to_string(datetime.date.today() - relativedelta(weeks=weeks_valid))))
        self.search(domain).unlink()

    def _create_reveal_view(self, website_id, url, ip_address, country_code, state_code, rules_excluded):
        # we are avoiding reveal if reveal_view already created for this IP
        rules = self.env['crm.reveal.rule']._match_url(website_id, url, country_code, state_code, rules_excluded)
        if rules:
            query = """
                    INSERT INTO crm_reveal_view (reveal_ip, reveal_rule_id, reveal_state, create_date)
                    VALUES (%s, %s, 'to_process', now() at time zone 'UTC')
                    ON CONFLICT DO NOTHING;
                    """ * len(rules)
            params = []
            for rule in rules:
                params += [ip_address, rule['id']]
                rules_excluded.append(str(rule['id']))
            self.env.cr.execute(query, params)
            return rules_excluded
        return False

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import time

from odoo import models
from odoo.http import request

_logger = logging.getLogger(__name__)

class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _serve_page(cls):
        response = super(IrHttp, cls)._serve_page()
        if response and getattr(response, 'status_code', 0) == 200 and request.env.user._is_public():
            visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
            # We are avoiding to create a reveal_view if a lead is already
            # created from another module, e.g. website_form
            if not (visitor_sudo and visitor_sudo.lead_ids):
                country_code = request.geoip.country_code
                state_code = request.geoip.subdivisions[0].iso_code if request.geoip.subdivisions else None
                if country_code:
                    try:
                        url = request.httprequest.url
                        ip_address = request.httprequest.remote_addr
                        if not ip_address:
                            return response
                        website_id = request.website.id
                        rules_excluded = (request.cookies.get('rule_ids') or '').split(',')
                        before = time.time()
                        new_rules_excluded = request.env['crm.reveal.view'].sudo()._create_reveal_view(website_id, url, ip_address, country_code, state_code, rules_excluded)
                        # even when we match, no view may have been created if this is a duplicate
                        _logger.info('Reveal process time: [%s], match rule: [%s?], country code: [%s], ip: [%s]',
                                     time.time() - before, new_rules_excluded == rules_excluded, country_code,
                                     ip_address)
                        if new_rules_excluded:
                            response.set_cookie('rule_ids', ','.join(new_rules_excluded), expires=None, cookie_type='optional')
                    except Exception:
                        # just in case - we never want to crash a page view
                        _logger.exception("Failed to process reveal rules")
        return response

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import crm_reveal_rule
from . import crm_reveal_view
from . import ir_http

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_crm_reveal_rule_manager,access_crm_reveal_rule_manager,model_crm_reveal_rule,sales_team.group_sale_manager,1,1,1,1
access_crm_reveal_rule_salesman,access_crm_reveal_rule_salesman,model_crm_reveal_rule,sales_team.group_sale_salesman,1,0,0,0
access_crm_reveal_view_manager,access_crm_reveal_view_manager,model_crm_reveal_view,sales_team.group_sale_manager,1,1,1,1
access_crm_reveal_view_salesman,access_crm_reveal_view_salesman,model_crm_reveal_view,sales_team.group_sale_salesman,1,0,0,0

```

## File: security\ir_rules.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="ir_rule_crm_reveal_rule_all" model="ir.rule">
        <field name="name">CRM Reveal Rules: All Rules</field>
        <field name="model_id" ref="model_crm_reveal_rule"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>
    <record id="ir_rule_crm_reveal_rule_salesman" model="ir.rule">
        <field name="name">CRM Reveal Rules: Personal / Global Rules</field>
        <field name="model_id" ref="model_crm_reveal_rule"/>
        <field name="domain_force">['|', ('user_id', '=', user.id), ('user_id', '=', False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="ir_rule_crm_reveal_view_all" model="ir.rule">
        <field name="name">CRM Reveal Views: All Views</field>
        <field name="model_id" ref="model_crm_reveal_view"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>
    <record id="ir_rule_crm_reveal_view_salesman" model="ir.rule">
        <field name="name">CRM Reveal Views: Personal / Global Views</field>
        <field name="model_id" ref="model_crm_reveal_view"/>
        <field name="domain_force">['|', ('reveal_rule_id.user_id', '=', user.id), ('reveal_rule_id.user_id', '=', False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>
</odoo>

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <record id="crm_reveal_lead_opportunity_form" model="ir.ui.view">
        <field name="name">crm.lead.inherited.crm</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='lead']/group" position="inside">
                <group string="Lead Generation Information" invisible="not reveal_ip" groups="base.group_no_one">
                    <field name="reveal_ip"/>
                    <field name="reveal_iap_credits"/>
                    <field name="reveal_rule_id"/>
                </group>
            </xpath>
            <xpath expr="//page[@name='extra']/group" position="inside">
                <group string="Lead Generation Information" invisible="not reveal_ip" groups="base.group_no_one">
                    <field name="reveal_ip"/>
                    <field name="reveal_iap_credits"/>
                    <field name="reveal_rule_id"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_view_pivot" model="ir.ui.view">
        <field name="name">crm.lead.view.pivot.inherit.lead.website</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_pivot"/>
        <field name="arch" type="xml">
            <xpath expr="//pivot" position="inside">
                <field name="reveal_iap_credits" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_view_graph" model="ir.ui.view">
        <field name="name">crm.lead.view.graph.inherit.lead.website</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_graph"/>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="inside">
                <field name="reveal_iap_credits" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_view_graph_report_opportunity" model="ir.ui.view">
        <field name="name">crm.lead.view.graph.report.opportunity.inherit.lead.website</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_opportunity_report_view_graph"/>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="inside">
                <field name="reveal_iap_credits" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_view_graph_report_lead" model="ir.ui.view">
        <field name="name">crm.lead.view.graph.report.lead.inherit.lead.website</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_opportunity_report_view_graph_lead"/>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="inside">
                <field name="reveal_iap_credits" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_view_graph_report_forecast" model="ir.ui.view">
        <field name="name">crm.lead.view.graph.forecast.inherit.website.crm.iap.reveal</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_graph_forecast"/>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="inside">
                <field name="reveal_iap_credits" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_view_pivot_forecast" model="ir.ui.view">
        <field name="name">crm.lead.view.pivot.forecast.inherit.website.crm.iap.reveal</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_pivot_forecast"/>
        <field name="arch" type="xml">
            <xpath expr="//pivot" position="inside">
                <field name="reveal_iap_credits" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="crm_opportunity_report_view_pivot_lead" model="ir.ui.view">
        <field name="name">crm.opportunity.report.view.pivot.lead.inherit.website.crm.iap.reveal</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_opportunity_report_view_pivot_lead"/>
        <field name="arch" type="xml">
            <xpath expr="//pivot" position="inside">
                <field name="reveal_iap_credits" invisible="1"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\crm_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem
        id="crm_reveal_rule_menu_action"
        action="crm_reveal_rule_action"
        parent="crm_iap_mine.crm_menu_lead_generation"
        sequence="5"/>

    <menuitem
        id="crm_reveal_view_menu_action"
        parent="crm.crm_menu_report"
        action="crm_reveal_view_action"
        groups="base.group_no_one"/>
</odoo>

```

## File: views\crm_reveal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_reveal_rule_form" model="ir.ui.view">
        <field name="name">crm.reveal.rule.form</field>
        <field name="model">crm.reveal.rule</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_get_opportunity_tree_view" class="oe_stat_button" type="object" icon="fa-star" invisible="lead_type != 'opportunity' and opportunity_count == 0">
                            <div class="o_stat_info">
                                <field name="opportunity_count"/>
                                <span class="o_stat_text"> Opportunities </span>
                            </div>
                        </button>
                        <button name="action_get_lead_tree_view" class="oe_stat_button" type="object" icon="fa-star" groups="crm.group_use_lead" invisible="lead_type != 'lead' and lead_count == 0">
                            <div class="o_stat_info">
                                <field name="lead_count"/>
                                <span class="o_stat_text"> Leads </span>
                            </div>
                        </button>
                    </div>
                    <div class="oe_title mb8">
                        <label for="name"/>
                        <h1 class="o_row">
                            <field name="name" placeholder="e.g. US Visitors"/>
                        </h1>
                    </div>
                    <div class="row">
                        <div class="col-md-6">
                            <group>
                                <field name="lead_for" widget="radio"/>
                                <field name="active" widget="boolean_toggle"/>
                            </group>
                            <group string="Website Traffic Conditions">
                                <field name="country_ids" widget="many2many_tags" options="{'no_open': True, 'no_create': True}"/>
                                <field name="website_id" options="{'no_open': True, 'no_create_edit': True}" groups="website.group_multi_website"/>
                                <field name="state_ids" widget="many2many_tags" invisible="not country_ids" domain="[('country_id', 'in', country_ids)]"/>
                                <field name="regex_url" widget="website_urls" placeholder="e.g. /page"/>
                                <field name="sequence"/>
                            </group>
                        </div>
                        <div class="col-md-6">
                            <div class="alert alert-info" role="alert">
                                1 credit is consumed per visitor matching the website traffic conditions and whose company can be identified.<br/>
                                <span invisible="lead_for != 'people'">Up to <field name="extra_contacts" readonly="1" class="mb0"/> additional credit(s) are consumed if the company matches this rule.</span>
                            </div>
                            <div class="alert alert-info" role="alert" invisible="lead_for != 'people'">
                                Make sure you know if you have to be GDPR compliant for storing personal data.
                            </div>
                        </div>
                    </div>
                    <group>
                        <group string="Opportunity Generation Conditions">
                            <field name="industry_tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True, 'no_quick_create': True}"/>
                            <field name="filter_on_size"/>
                            <label for="company_size_min" invisible="not filter_on_size"/>
                            <div invisible="not filter_on_size">
                                From <field name="company_size_min" class="oe_inline col-sm-3" />
                                to <field name="company_size_max" class="oe_inline col-sm-3" />
                                employees
                            </div>
                        </group>
                        <group string="Contact Filter" invisible="lead_for != 'people'">
                            <field name="extra_contacts"/>
                            <field name="contact_filter_type" widget="radio"/>
                            <field name="preferred_role_id" invisible="contact_filter_type != 'role'" options="{'no_create_edit': True, 'no_quick_create': True}"/>
                            <field name="other_role_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True, 'no_quick_create': True}" invisible="not preferred_role_id or contact_filter_type != 'role'"/>
                            <field name="seniority_id" invisible="contact_filter_type != 'seniority'" options="{'no_create_edit': True, 'no_quick_create': True}"/>
                        </group>
                    </group>
                    <field name="lead_type" invisible="1"/>
                    <separator invisible="lead_type != 'lead'" string="Lead Data" />
                    <separator invisible="lead_type != 'opportunity'" string="Opportunity Data" />
                    <group>
                        <group>
                            <field name="lead_type" groups="crm.group_use_lead"/>
                            <field name="suffix"/>
                            <field name="team_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                            <field name="user_id" domain="[('share', '=', False)]"/>
                        </group>
                        <group>
                            <field name="tag_ids" widget="many2many_tags"/>
                            <field name="priority" widget="priority"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="crm_reveal_rule_tree" model="ir.ui.view">
        <field name="name">crm.reveal.rule.list</field>
        <field name="model">crm.reveal.rule</field>
        <field name="arch" type="xml">
            <list>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="lead_type"/>
            </list>
        </field>
    </record>

    <record id="crm_reveal_rule_view_search" model="ir.ui.view">
        <field name="name">crm.reveal.rule.view.search</field>
        <field name="model">crm.reveal.rule</field>
        <field name="arch" type="xml">
            <search string="Search CRM Reveal Rule">
                <field string="Rule" name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="crm_reveal_rule_action" model="ir.actions.act_window">
        <field name="name">Visits to Leads Rules</field>
        <field name="res_model">crm.reveal.rule</field>
        <field name="view_mode">list,form</field>
        <field name="search_view_id" ref="crm_reveal_rule_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a conversion rule
            </p><p>
                Create rules to generate B2B leads/opportunities from your website visitors.
            </p>
        </field>
    </record>

    <record id="crm_reveal_view_form" model="ir.ui.view">
        <field name="name">crm.reveal.view.form</field>
        <field name="model">crm.reveal.view</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <field name="reveal_state" widget="statusbar" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                    <group>
                        <field name="reveal_ip"/>
                        <field name="reveal_rule_id"/>
                        <field name="create_date"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="crm_reveal_view_tree" model="ir.ui.view">
        <field name="name">crm.reveal.view.list</field>
        <field name="model">crm.reveal.view</field>
        <field name="arch" type="xml">
            <list>
                <field name="reveal_ip"/>
                <field name="reveal_rule_id"/>
                <field name="reveal_state"/>
                <field name="create_date"/>
            </list>
        </field>
    </record>

    <record id="crm_reveal_view_action" model="ir.actions.act_window">
        <field name="name">Lead Generation Views</field>
        <field name="res_model">crm.reveal.view</field>
        <field name="view_mode">list,form</field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.crm.iap.reveal</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="crm.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="website_crm_iap_reveal_settings" position="inside">
                <widget name="iap_buy_more_credits" service_name="reveal"/>
            </setting>
        </field>
    </record>
</odoo>

```

