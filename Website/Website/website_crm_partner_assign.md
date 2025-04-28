# Odoo Module: website_crm_partner_assign

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Resellers',
    'category': 'Website/Website',
    'summary': 'Publish your resellers/partners and forward leads to them',
    'version': '1.0',
    'description': """
This module allows to publish your resellers/partners on your website and to forward incoming leads/opportunities to them.


**Publish a partner**

To publish a partner, set a *Level* in their contact form (in the Partner Assignment section) and click the *Publish* button.

**Forward leads**

Forwarding leads can be done for one or several leads at a time. The action is available in the *Assigned Partner* section of the lead/opportunity form view and in the *Action* menu of the list view.

The automatic assignment is figured from the weight of partner levels and the geolocalization. Partners get leads that are located around them.

    """,
    'depends': ['base_geolocalize', 'crm', 'account',
                'website_partner', 'website_google_map', 'portal'],
    'data': [
        'data/crm_tag_data.xml',
        'data/mail_template_data.xml',
        'security/ir.model.access.csv',
        'security/ir_rule.xml',
        'wizard/crm_forward_to_partner_view.xml',
        'views/res_partner_views.xml',
        'views/crm_lead_views.xml',
        'views/website_crm_partner_assign_templates.xml',
        'report/crm_partner_report_view.xml',
    ],
    'demo': [
        'data/res_partner_demo.xml',
        'data/crm_lead_demo.xml',
        'data/res_partner_grade_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_frontend': [
            'website_crm_partner_assign/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import werkzeug.urls

from collections import OrderedDict
from werkzeug.exceptions import NotFound

from odoo import fields
from odoo import http
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import slug, unslug
from odoo.addons.portal.controllers.portal import CustomerPortal
from odoo.addons.website_partner.controllers.main import WebsitePartnerPage

from odoo.tools.translate import _


class WebsiteAccount(CustomerPortal):

    def get_domain_my_lead(self, user):
        return [
            ('partner_assigned_id', 'child_of', user.commercial_partner_id.id),
            ('type', '=', 'lead')
        ]

    def get_domain_my_opp(self, user):
        return [
            ('partner_assigned_id', 'child_of', user.commercial_partner_id.id),
            ('type', '=', 'opportunity')
        ]

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        CrmLead = request.env['crm.lead']
        if 'lead_count' in counters:
            values['lead_count'] = (
                CrmLead.search_count(self.get_domain_my_lead(request.env.user))
                if CrmLead.check_access_rights('read', raise_exception=False)
                else 0
            )
        if 'opp_count' in counters:
            values['opp_count'] = (
                CrmLead.search_count(self.get_domain_my_opp(request.env.user))
                if CrmLead.check_access_rights('read', raise_exception=False)
                else 0
            )
        return values

    @http.route(['/my/leads', '/my/leads/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_leads(self, page=1, date_begin=None, date_end=None, sortby=None, **kw):
        values = self._prepare_portal_layout_values()
        CrmLead = request.env['crm.lead']
        domain = self.get_domain_my_lead(request.env.user)

        searchbar_sortings = {
            'date': {'label': _('Newest'), 'order': 'create_date desc'},
            'name': {'label': _('Name'), 'order': 'name'},
            'contact_name': {'label': _('Contact Name'), 'order': 'contact_name'},
        }

        # default sort by value
        if not sortby:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # pager
        lead_count = CrmLead.search_count(domain)
        pager = request.website.pager(
            url="/my/leads",
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby},
            total=lead_count,
            page=page,
            step=self._items_per_page
        )
        # content according to pager and archive selected
        leads = CrmLead.search(domain, order=order, limit=self._items_per_page, offset=pager['offset'])

        values.update({
            'date': date_begin,
            'leads': leads,
            'page_name': 'lead',
            'default_url': '/my/leads',
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
        })
        return request.render("website_crm_partner_assign.portal_my_leads", values)

    @http.route(['/my/opportunities', '/my/opportunities/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_opportunities(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, **kw):
        values = self._prepare_portal_layout_values()
        CrmLead = request.env['crm.lead']
        domain = self.get_domain_my_opp(request.env.user)

        today = fields.Date.today()
        this_week_end_date = fields.Date.to_string(fields.Date.from_string(today) + datetime.timedelta(days=7))

        searchbar_filters = {
            'all': {'label': _('Active'), 'domain': []},
            'today': {'label': _('Today Activities'), 'domain': [('activity_date_deadline', '=', today)]},
            'week': {'label': _('This Week Activities'),
                     'domain': [('activity_date_deadline', '>=', today), ('activity_date_deadline', '<=', this_week_end_date)]},
            'overdue': {'label': _('Overdue Activities'), 'domain': [('activity_date_deadline', '<', today)]},
            'won': {'label': _('Won'), 'domain': [('stage_id.is_won', '=', True)]},
            'lost': {'label': _('Lost'), 'domain': [('active', '=', False), ('probability', '=', 0)]},
        }
        searchbar_sortings = {
            'date': {'label': _('Newest'), 'order': 'create_date desc'},
            'name': {'label': _('Name'), 'order': 'name'},
            'contact_name': {'label': _('Contact Name'), 'order': 'contact_name'},
            'revenue': {'label': _('Expected Revenue'), 'order': 'expected_revenue desc'},
            'probability': {'label': _('Probability'), 'order': 'probability desc'},
            'stage': {'label': _('Stage'), 'order': 'stage_id'},
        }

        # default sort by value
        if not sortby:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']
        # default filter by value
        if not filterby:
            filterby = 'all'
        domain += searchbar_filters[filterby]['domain']
        if filterby == 'lost':
            CrmLead = CrmLead.with_context(active_test=False)

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]
        # pager
        opp_count = CrmLead.search_count(domain)
        pager = request.website.pager(
            url="/my/opportunities",
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby, 'filterby': filterby},
            total=opp_count,
            page=page,
            step=self._items_per_page
        )
        # content according to pager
        opportunities = CrmLead.search(domain, order=order, limit=self._items_per_page, offset=pager['offset'])

        values.update({
            'date': date_begin,
            'opportunities': opportunities,
            'page_name': 'opportunity',
            'default_url': '/my/opportunities',
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
        })
        return request.render("website_crm_partner_assign.portal_my_opportunities", values)

    @http.route(['''/my/lead/<model('crm.lead', "[('type','=', 'lead')]"):lead>'''], type='http', auth="user", website=True)
    def portal_my_lead(self, lead, **kw):
        if lead.type != 'lead':
            raise NotFound()
        return request.render("website_crm_partner_assign.portal_my_lead", {'lead': lead})

    @http.route(['''/my/opportunity/<model('crm.lead', "[('type','=', 'opportunity')]"):opp>'''], type='http', auth="user", website=True)
    def portal_my_opportunity(self, opp, **kw):
        if opp.type != 'opportunity':
            raise NotFound()

        return request.render(
            "website_crm_partner_assign.portal_my_opportunity", {
                'opportunity': opp,
                'user_activity': opp.sudo().activity_ids.filtered(lambda activity: activity.user_id == request.env.user)[:1],
                'stages': request.env['crm.stage'].search([('is_won', '!=', True)], order='sequence desc, name desc, id desc'),
                'activity_types': request.env['mail.activity.type'].sudo().search(['|', ('res_model', '=', opp._name), ('res_model', '=', False)]),
                'states': request.env['res.country.state'].sudo().search([]),
                'countries': request.env['res.country'].sudo().search([]),
            })


class WebsiteCrmPartnerAssign(WebsitePartnerPage):
    _references_per_page = 40

    def sitemap_partners(env, rule, qs):
        if not qs or qs.lower() in '/partners':
            yield {'loc': '/partners'}
        base_partner_domain = [
            ('is_company', '=', True),
            ('grade_id', '!=', False),
            ('website_published', '=', True),
            ('grade_id.website_published', '=', True),
            ('grade_id.active', '=', True),
        ]
        grades = env['res.partner'].sudo().read_group(base_partner_domain, fields=['id', 'grade_id'], groupby='grade_id')
        for grade in grades:
            loc = '/partners/grade/%s' % slug(grade['grade_id'])
            if not qs or qs.lower() in loc:
                yield {'loc': loc}
        country_partner_domain = base_partner_domain + [('country_id', '!=', False)]
        countries = env['res.partner'].sudo().read_group(country_partner_domain, fields=['id', 'country_id'], groupby='country_id')
        for country in countries:
            loc = '/partners/country/%s' % slug(country['country_id'])
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

    @http.route([
        '/partners',
        '/partners/page/<int:page>',

        '/partners/grade/<model("res.partner.grade"):grade>',
        '/partners/grade/<model("res.partner.grade"):grade>/page/<int:page>',

        '/partners/country/<model("res.country"):country>',
        '/partners/country/<model("res.country"):country>/page/<int:page>',

        '/partners/grade/<model("res.partner.grade"):grade>/country/<model("res.country"):country>',
        '/partners/grade/<model("res.partner.grade"):grade>/country/<model("res.country"):country>/page/<int:page>',
    ], type='http', auth="public", website=True, sitemap=sitemap_partners)
    def partners(self, country=None, grade=None, page=0, **post):
        country_all = post.pop('country_all', False)
        partner_obj = request.env['res.partner']
        country_obj = request.env['res.country']
        search = post.get('search', '')

        base_partner_domain = [('is_company', '=', True), ('grade_id', '!=', False), ('website_published', '=', True), ('grade_id.active', '=', True)]
        if not request.env['res.users'].has_group('website.group_website_publisher'):
            base_partner_domain += [('grade_id.website_published', '=', True)]
        if search:
            base_partner_domain += ['|', ('name', 'ilike', search), ('website_description', 'ilike', search)]

        # group by grade
        grade_domain = list(base_partner_domain)
        if not country and not country_all:
            country_code = request.session['geoip'].get('country_code')
            if country_code:
                country = country_obj.search([('code', '=', country_code)], limit=1)
        if country:
            grade_domain += [('country_id', '=', country.id)]
        grades = partner_obj.sudo().read_group(
            grade_domain, ["id", "grade_id"],
            groupby="grade_id")
        grades_partners = partner_obj.sudo().search_count(grade_domain)
        # flag active grade
        for grade_dict in grades:
            grade_dict['active'] = grade and grade_dict['grade_id'][0] == grade.id
        grades.insert(0, {
            'grade_id_count': grades_partners,
            'grade_id': (0, _("All Categories")),
            'active': bool(grade is None),
        })

        # group by country
        country_domain = list(base_partner_domain)
        if grade:
            country_domain += [('grade_id', '=', grade.id)]
        countries = partner_obj.sudo().read_group(
            country_domain, ["id", "country_id"],
            groupby="country_id", orderby="country_id")
        countries_partners = partner_obj.sudo().search_count(country_domain)
        # flag active country
        for country_dict in countries:
            country_dict['active'] = country and country_dict['country_id'] and country_dict['country_id'][0] == country.id
        countries.insert(0, {
            'country_id_count': countries_partners,
            'country_id': (0, _("All Countries")),
            'active': bool(country is None),
        })

        # current search
        if grade:
            base_partner_domain += [('grade_id', '=', grade.id)]
        if country:
            base_partner_domain += [('country_id', '=', country.id)]

        # format pager
        if grade and not country:
            url = '/partners/grade/' + slug(grade)
        elif country and not grade:
            url = '/partners/country/' + slug(country)
        elif country and grade:
            url = '/partners/grade/' + slug(grade) + '/country/' + slug(country)
        else:
            url = '/partners'
        url_args = {}
        if search:
            url_args['search'] = search
        if country_all:
            url_args['country_all'] = True

        partner_count = partner_obj.sudo().search_count(base_partner_domain)
        pager = request.website.pager(
            url=url, total=partner_count, page=page, step=self._references_per_page, scope=7,
            url_args=url_args)

        # search partners matching current search parameters
        partner_ids = partner_obj.sudo().search(
            base_partner_domain, order="grade_sequence ASC, implemented_count DESC, display_name ASC, id ASC",
            offset=pager['offset'], limit=self._references_per_page)
        partners = partner_ids.sudo()

        google_map_partner_ids = ','.join(str(p.id) for p in partners)
        google_maps_api_key = request.website.google_maps_api_key

        values = {
            'countries': countries,
            'country_all': country_all,
            'current_country': country,
            'grades': grades,
            'current_grade': grade,
            'partners': partners,
            'google_map_partner_ids': google_map_partner_ids,
            'pager': pager,
            'searches': post,
            'search_path': "%s" % werkzeug.urls.url_encode(post),
            'google_maps_api_key': google_maps_api_key,
        }
        return request.render("website_crm_partner_assign.index", values, status=partners and 200 or 404)


    # Do not use semantic controller due to sudo()
    @http.route(['/partners/<partner_id>'], type='http', auth="public", website=True)
    def partners_detail(self, partner_id, **post):
        current_slug = partner_id
        _, partner_id = unslug(partner_id)
        current_grade, current_country = None, None
        grade_id = post.get('grade_id')
        country_id = post.get('country_id')
        if grade_id:
            current_grade = request.env['res.partner.grade'].browse(int(grade_id)).exists()
        if country_id:
            current_country = request.env['res.country'].browse(int(country_id)).exists()
        if partner_id:
            partner = request.env['res.partner'].sudo().browse(partner_id)
            is_website_publisher = request.env['res.users'].has_group('website.group_website_publisher')
            if partner.exists() and (partner.website_published or is_website_publisher):
                if slug(partner) != current_slug:
                    return request.redirect('/partners/%s' % slug(partner))
                values = {
                    'main_object': partner,
                    'partner': partner,
                    'current_grade': current_grade,
                    'current_country': current_country
                }
                return request.render("website_crm_partner_assign.partner", values)
        raise request.not_found()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\crm_lead_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
      <data noupdate="1">

            <!--  Demo Leads -->
            <record id="crm_case_partner_assign_1" model="crm.lead">
                  <field name="type">lead</field>
                  <field name="name">Specifications and price of your phones</field>
                  <field name="contact_name">Steve Martinez</field>
                  <field name="partner_name"></field>
                  <field name="partner_id" ref=""/>
                  <field name="function">Reseller</field>
                  <field name="country_id" ref="base.uk"/>
                  <field name="city">Edinburgh</field>
                  <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
                  <field name="priority">2</field>
                  <field name="team_id" ref="sales_team.crm_team_1"/>
                  <field name="user_id" ref=""/>
                  <field name="stage_id" ref="crm.stage_lead1"/>
                  <field name="description">Hi,

            Please, can you give me more details about your phones, including their specifications and their prices.

            Regards,
            Steve</field>
                  <field eval="1" name="active"/>
                  <field name="partner_assigned_id" ref="base.partner_demo_portal"/>
                  <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
                  <field name="medium_id" ref="utm.utm_medium_email"/>
                  <field name="source_id" ref="utm.utm_source_newsletter"/>
            </record>
      </data>
</odoo>
```

## File: data\crm_tag_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="crm.tag" id="tag_portal_lead_partner_unavailable">
            <field name="name">No more partner available</field>
            <field name="color">3</field>
        </record>
        <record model="crm.tag" id="tag_portal_lead_is_spam">
            <field name="name">Spam</field>
            <field name="color">3</field>
        </record>
        <record model="crm.tag" id="tag_portal_lead_own_opp">
            <field name="name">Created by Partner</field>
            <field name="color">4</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <!-- Technical template, keep updated -->
        <record id="email_template_lead_forward_mail" model="mail.template">
            <field name="name">Lead Forward: Send to partner</field>
            <field name="model_id" ref="website_crm_partner_assign.model_crm_lead_forward_to_partner" />
            <field name="subject">Fwd: Lead: {{ ctx['partner_id'].name }}</field>
            <field name="email_from">{{ user.email_formatted }}</field>
            <field name="email_to">{{ ctx['partner_id'].email_formatted }}</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your leads</span><br/>
                </td><td valign="middle" align="right">
                    <img t-attf-src="/logo.png?company={{ user.company_id.id }}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="user.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr>
                    <td valign="top" style="font-size: 13px;">
                        <div>
                            Hello,<br/>
                            We have been contacted by those prospects that are in your region. Thus, the following leads have been assigned to <t t-out="ctx['partner_id'].name or ''"></t>:<br/>
                            <ol>
                                <li t-foreach="ctx['partner_leads']" t-as="lead"><a t-att-href="lead['lead_link']" t-out="lead['lead_id'].name or 'Subject Undefined'">Subject Undefined</a>, <t t-out="lead['lead_id'].partner_name or lead['lead_id'].contact_name or 'Contact Name Undefined'">Contact Name Undefined</t>, <t t-out="lead['lead_id'].country_id and lead['lead_id'].country_id.name or 'Country Undefined'">Country Undefined</t>, <t t-out="lead['lead_id'].email_from or 'Email Undefined' or ''">Email Undefined</t>, <t t-out="lead['lead_id'].phone or ''">+1 650-123-4567</t> </li><br/>
                            </ol>
                            <t t-if="ctx.get('partner_in_portal')">
                                Please connect to your <a t-att-href="'%s' % (object.get_base_url())">Partner Portal</a> to get details. On each lead are two buttons on the top left corner that you should press after having contacted the lead: "I'm interested" &amp; "I'm not interested".<br/>
                            </t>
                            <t t-else="">
                                You do not have yet a portal access to our database. Please contact
                                <t t-out="ctx['partner_id'].user_id and ctx['partner_id'].user_id.email and 'your account manager %s (%s)' % (ctx['partner_id'].user_id.name,ctx['partner_id'].user_id.email) or 'us'">us</t>.<br/>
                            </t>
                            The lead will be sent to another partner if you do not contact the lead before 20 days.<br/><br/>
                            Thank you,<br/>
                            <t t-out="ctx['partner_id'].user_id and ctx['partner_id'].user_id.signature or ''"></t>
                            <br/>
                            <t t-if="not ctx['partner_id'].user_id">
                                PS: It looks like you do not have an account manager assigned to you, please contact us.
                            </t>
                        </div>
                    </td>
                </tr>
                <tr>
                    <td style="text-align:center;">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </td>
                </tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                    <t t-out="user.company_id.name or ''">YourCompany</t>
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    <t t-out="user.company_id.phone or ''">+1 650-123-4567</t>
                    <t t-if="user.company_id.phone and (user.company_id.email or user.company_id.website)">|</t>
                    <a t-if="user.company_id.email" t-att-href="'mailto:%s' % user.company_id.email" style="text-decoration:none; color: #454748;" t-out="user.company_id.email or ''">info@yourcompany.com</a>
                    <t t-if="user.company_id.email and user.company_id.website">|</t>
                    <a t-if="user.company_id.website" t-att-href="'%s' % user.company_id.website" style="text-decoration:none; color: #454748;" t-out="user.company_id.website or ''">http://www.example.com</a>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=website" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table>
            </field>
            <field name="lang">{{ ctx['partner_id'].lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\res_partner_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
        <record id="res_partner_grade_platinium" model="res.partner.grade">
            <field name="name">Platinum</field>
            <field name="sequence">4</field>
        </record>
        <record id="res_partner_grade_gold" model="res.partner.grade">
            <field name="name">Gold</field>
            <field name="sequence">3</field>
        </record>
        <record id="res_partner_grade_silver" model="res.partner.grade">
            <field name="name">Silver</field>
            <field name="sequence">2</field>
        </record>
        <record id="res_partner_grade_bronze" model="res.partner.grade">
            <field name="name">Bronze</field>
            <field name="sequence">1</field>
        </record>

        <record id="base.res_partner_3" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_bronze"/>
            <field name="partner_weight">10</field>
        </record>
        <record model="res.partner" id="base.res_partner_2">
            <field name="assigned_partner_id" ref="base.res_partner_3"/>
        </record>
       <record id="base.res_partner_4" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_bronze"/>
            <field name="partner_weight">10</field>
        </record>
        <record id="base.res_partner_12" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_bronze"/>
            <field name="partner_weight">10</field>
        </record>

        <record id="base.res_partner_3" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_silver"/>
            <field name="partner_weight">10</field>
        </record>
        <record id="base.res_partner_12" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_silver"/>
            <field name="partner_weight">10</field>
        </record>
        <record id="base.res_partner_12" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_silver"/>
            <field name="partner_weight">10</field>
        </record>

        <record id="base.res_partner_10" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_gold"/>
            <field name="partner_weight">10</field>
        </record>
        <record id="base.res_partner_18" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_gold"/>
            <field name="partner_weight">10</field>
        </record>
        <record id="base.res_partner_1" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_gold"/>
            <field name="partner_weight">10</field>
        </record>

        <record id="base.res_partner_12" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_platinium"/>
            <field name="partner_weight">10</field>
        </record>
        <record id="base.res_partner_4" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_platinium"/>
            <field name="partner_weight">10</field>
        </record>

        <record model="res.partner" id="base.res_partner_10">
            <field name="assigned_partner_id" ref="base.res_partner_4"/>
        </record>
        <record model="res.partner" id="base.res_partner_12">
            <field name="assigned_partner_id" ref="base.res_partner_10"/>
        </record>
        <record model="res.partner" id="base.res_partner_4">
            <field name="assigned_partner_id" ref="base.res_partner_4"/>
        </record>
        <record model="res.partner" id="base.res_partner_10">
            <field name="assigned_partner_id" ref="base.res_partner_12"/>
        </record>
        <record model="res.partner" id="base.res_partner_3">
            <field name="assigned_partner_id" ref="base.res_partner_3"/>
        </record>
        <record model="res.partner" id="base.res_partner_2">
            <field name="assigned_partner_id" ref="base.res_partner_1"/>
        </record>
        <record model="res.partner" id="base.res_partner_4">
            <field name="assigned_partner_id" ref="base.res_partner_18"/>
        </record>
        <record model="res.partner" id="base.res_partner_4">
            <field name="assigned_partner_id" ref="base.res_partner_3"/>
        </record>
        <record model="res.partner" id="base.res_partner_1">
            <field name="assigned_partner_id" ref="base.res_partner_12"/>
        </record>
        <record model="res.partner" id="base.res_partner_1">
            <field name="assigned_partner_id" ref="base.res_partner_2"/>
        </record>
        <record model="res.partner" id="base.res_partner_1">
            <field name="assigned_partner_id" ref="base.res_partner_2"/>
        </record>
        <record model="res.partner" id="base.res_partner_12">
            <field name="assigned_partner_id" ref="base.res_partner_12"/>
        </record>
</odoo>

```

## File: data\res_partner_grade_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record id="website_crm_partner_assign.res_partner_grade_platinium" model="res.partner.grade">
            <field name="is_published" eval="True" />
        </record>
        <record id="website_crm_partner_assign.res_partner_grade_gold" model="res.partner.grade">
            <field name="is_published" eval="True" />
        </record>
        <record id="website_crm_partner_assign.res_partner_grade_silver" model="res.partner.grade">
            <field name="is_published" eval="True" />
        </record>
        <record id="website_crm_partner_assign.res_partner_grade_bronze" model="res.partner.grade">
            <field name="is_published" eval="True" />
        </record>
        <record id="base.partner_demo_portal" model="res.partner">
            <field name="grade_id" ref="website_crm_partner_assign.res_partner_grade_platinium"/>
            <field name="partner_weight">10</field>
        </record>
    </data>
</odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import random

from odoo import api, fields, models, _
from odoo.exceptions import AccessDenied, AccessError, UserError
from odoo.tools import html_escape



class CrmLead(models.Model):
    _inherit = "crm.lead"

    partner_latitude = fields.Float('Geo Latitude', digits=(10, 7))
    partner_longitude = fields.Float('Geo Longitude', digits=(10, 7))
    partner_assigned_id = fields.Many2one('res.partner', 'Assigned Partner', tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", help="Partner this case has been forwarded/assigned to.", index=True)
    partner_declined_ids = fields.Many2many(
        'res.partner',
        'crm_lead_declined_partner',
        'lead_id',
        'partner_id',
        string='Partner not interested')
    date_partner_assign = fields.Date(
        'Partner Assignment Date', compute='_compute_date_partner_assign',
        copy=True, readonly=False, store=True,
        help="Last date this case was forwarded/assigned to a partner")

    @api.depends("partner_assigned_id")
    def _compute_date_partner_assign(self):
        for lead in self:
            if not lead.partner_assigned_id:
                lead.date_partner_assign = False
            else:
                lead.date_partner_assign = fields.Date.context_today(lead)

    def _merge_get_fields(self):
        fields_list = super(CrmLead, self)._merge_get_fields()
        fields_list += ['partner_latitude', 'partner_longitude', 'partner_assigned_id', 'date_partner_assign']
        return fields_list

    def assign_salesman_of_assigned_partner(self):
        salesmans_leads = {}
        for lead in self:
            if lead.active and lead.probability < 100:
                if lead.partner_assigned_id and lead.partner_assigned_id.user_id != lead.user_id:
                    salesmans_leads.setdefault(lead.partner_assigned_id.user_id.id, []).append(lead.id)

        for salesman_id, leads_ids in salesmans_leads.items():
            leads = self.browse(leads_ids)
            leads.write({'user_id': salesman_id})

    def action_assign_partner(self):
        return self.assign_partner(partner_id=False)

    def assign_partner(self, partner_id=False):
        partner_dict = {}
        res = False
        if not partner_id:
            partner_dict = self.search_geo_partner()
        for lead in self:
            if not partner_id:
                partner_id = partner_dict.get(lead.id, False)
            if not partner_id:
                tag_to_add = self.env.ref('website_crm_partner_assign.tag_portal_lead_partner_unavailable', False)
                if tag_to_add:
                    lead.write({'tag_ids': [(4, tag_to_add.id, False)]})
                continue
            lead.assign_geo_localize(lead.partner_latitude, lead.partner_longitude)
            partner = self.env['res.partner'].browse(partner_id)
            if partner.user_id:
                lead._handle_salesmen_assignment(user_ids=partner.user_id.ids, team_id=partner.team_id.id)
            lead.write({'partner_assigned_id': partner_id})
        return res

    def assign_geo_localize(self, latitude=False, longitude=False):
        if latitude and longitude:
            self.write({
                'partner_latitude': latitude,
                'partner_longitude': longitude
            })
            return True
        # Don't pass context to browse()! We need country name in english below
        for lead in self:
            if lead.partner_latitude and lead.partner_longitude:
                continue
            if lead.country_id:
                result = self.env['res.partner']._geo_localize(
                    lead.street, lead.zip, lead.city,
                    lead.state_id.name, lead.country_id.name
                )
                if result:
                    lead.write({
                        'partner_latitude': result[0],
                        'partner_longitude': result[1]
                    })
        return True

    def _prepare_customer_values(self, partner_name, is_company=False, parent_id=False):
        res = super()._prepare_customer_values(partner_name, is_company=is_company, parent_id=parent_id)
        res.update({
            'partner_latitude': self.partner_latitude,
            'partner_longitude': self.partner_longitude,
        })
        return res

    def search_geo_partner(self):
        Partner = self.env['res.partner']
        res_partner_ids = {}
        self.assign_geo_localize()
        for lead in self:
            partner_ids = []
            if not lead.country_id:
                continue
            latitude = lead.partner_latitude
            longitude = lead.partner_longitude
            if latitude and longitude:
                # 1. first way: in the same country, small area
                partner_ids = Partner.search([
                    ('partner_weight', '>', 0),
                    ('partner_latitude', '>', latitude - 2), ('partner_latitude', '<', latitude + 2),
                    ('partner_longitude', '>', longitude - 1.5), ('partner_longitude', '<', longitude + 1.5),
                    ('country_id', '=', lead.country_id.id),
                    ('id', 'not in', lead.partner_declined_ids.mapped('id')),
                ])

                # 2. second way: in the same country, big area
                if not partner_ids:
                    partner_ids = Partner.search([
                        ('partner_weight', '>', 0),
                        ('partner_latitude', '>', latitude - 4), ('partner_latitude', '<', latitude + 4),
                        ('partner_longitude', '>', longitude - 3), ('partner_longitude', '<', longitude + 3),
                        ('country_id', '=', lead.country_id.id),
                        ('id', 'not in', lead.partner_declined_ids.mapped('id')),
                    ])

                # 3. third way: in the same country, extra large area
                if not partner_ids:
                    partner_ids = Partner.search([
                        ('partner_weight', '>', 0),
                        ('partner_latitude', '>', latitude - 8), ('partner_latitude', '<', latitude + 8),
                        ('partner_longitude', '>', longitude - 8), ('partner_longitude', '<', longitude + 8),
                        ('country_id', '=', lead.country_id.id),
                        ('id', 'not in', lead.partner_declined_ids.mapped('id')),
                    ])

                # 5. fifth way: anywhere in same country
                if not partner_ids:
                    # still haven't found any, let's take all partners in the country!
                    partner_ids = Partner.search([
                        ('partner_weight', '>', 0),
                        ('country_id', '=', lead.country_id.id),
                        ('id', 'not in', lead.partner_declined_ids.mapped('id')),
                    ])

                # 6. sixth way: closest partner whatsoever, just to have at least one result
                if not partner_ids:
                    # warning: point() type takes (longitude, latitude) as parameters in this order!
                    self._cr.execute("""SELECT id, distance
                                  FROM  (select id, (point(partner_longitude, partner_latitude) <-> point(%s,%s)) AS distance FROM res_partner
                                  WHERE active
                                        AND partner_longitude is not null
                                        AND partner_latitude is not null
                                        AND partner_weight > 0
                                        AND id not in (select partner_id from crm_lead_declined_partner where lead_id = %s)
                                        ) AS d
                                  ORDER BY distance LIMIT 1""", (longitude, latitude, lead.id))
                    res = self._cr.dictfetchone()
                    if res:
                        partner_ids = Partner.browse([res['id']])

                if partner_ids:
                    res_partner_ids[lead.id] = random.choices(
                        partner_ids.ids,
                        partner_ids.mapped('partner_weight'),
                    )[0]

        return res_partner_ids

    def partner_interested(self, comment=False):
        message = _('<p>I am interested by this lead.</p>')
        if comment:
            message += '<p>%s</p>' % html_escape(comment)
        for lead in self:
            lead.message_post(body=message)
            lead.sudo().convert_opportunity(lead.partner_id.id)  # sudo required to convert partner data

    def partner_desinterested(self, comment=False, contacted=False, spam=False):
        if contacted:
            message = '<p>%s</p>' % _('I am not interested by this lead. I contacted the lead.')
        else:
            message = '<p>%s</p>' % _('I am not interested by this lead. I have not contacted the lead.')
        partner_ids = self.env['res.partner'].search(
            [('id', 'child_of', self.env.user.partner_id.commercial_partner_id.id)])
        self.message_unsubscribe(partner_ids=partner_ids.ids)
        if comment:
            message += '<p>%s</p>' % html_escape(comment)
        self.message_post(body=message)
        values = {
            'partner_assigned_id': False,
        }

        if spam:
            tag_spam = self.env.ref('website_crm_partner_assign.tag_portal_lead_is_spam', False)
            if tag_spam and tag_spam not in self.tag_ids:
                values['tag_ids'] = [(4, tag_spam.id, False)]
        if partner_ids:
            values['partner_declined_ids'] = [(4, p, 0) for p in partner_ids.ids]
        self.sudo().write(values)

    def update_lead_portal(self, values):
        self.check_access_rights('write')
        for lead in self:
            lead_values = {
                'expected_revenue': values['expected_revenue'],
                'probability': values['probability'] or False,
                'priority': values['priority'],
                'date_deadline': values['date_deadline'] or False,
            }
            # As activities may belong to several users, only the current portal user activity
            # will be modified by the portal form. If no activity exist we create a new one instead
            # that we assign to the portal user.

            user_activity = lead.sudo().activity_ids.filtered(lambda activity: activity.user_id == self.env.user)[:1]
            if values['activity_date_deadline']:
                if user_activity:
                    user_activity.sudo().write({
                        'activity_type_id': values['activity_type_id'],
                        'summary': values['activity_summary'],
                        'date_deadline': values['activity_date_deadline'],
                    })
                else:
                    self.env['mail.activity'].sudo().create({
                        'res_model_id': self.env.ref('crm.model_crm_lead').id,
                        'res_id': lead.id,
                        'user_id': self.env.user.id,
                        'activity_type_id': values['activity_type_id'],
                        'summary': values['activity_summary'],
                        'date_deadline': values['activity_date_deadline'],
                    })
            lead.write(lead_values)

    def update_contact_details_from_portal(self, values):
        self.check_access_rights('write')
        fields = ['partner_name', 'phone', 'mobile', 'email_from', 'street', 'street2',
            'city', 'zip', 'state_id', 'country_id']
        if any([key not in fields for key in values]):
            raise UserError(_("Not allowed to update the following field(s) : %s.") % ", ".join([key for key in values if not key in fields]))
        return self.sudo().write(values)

    @api.model
    def create_opp_portal(self, values):
        if not (self.env.user.partner_id.grade_id or self.env.user.commercial_partner_id.grade_id):
            raise AccessDenied()
        user = self.env.user
        self = self.sudo()
        if not (values['contact_name'] and values['description'] and values['title']):
            return {
                'errors': _('All fields are required !')
            }
        tag_own = self.env.ref('website_crm_partner_assign.tag_portal_lead_own_opp', False)
        values = {
            'contact_name': values['contact_name'],
            'name': values['title'],
            'description': values['description'],
            'priority': '2',
            'partner_assigned_id': user.commercial_partner_id.id,
        }
        if tag_own:
            values['tag_ids'] = [(4, tag_own.id, False)]

        lead = self.create(values)
        lead.assign_salesman_of_assigned_partner()
        lead.convert_opportunity(lead.partner_id.id)
        return {
            'id': lead.id
        }

    #
    #   DO NOT FORWARD PORT IN MASTER
    #   instead, crm.lead should implement portal.mixin
    #
    def get_access_action(self, access_uid=None):
        """ Instead of the classic form view, redirect to the online document for
        portal users or if force_website=True in the context. """
        self.ensure_one()

        user, record = self.env.user, self
        if access_uid:
            try:
                record.check_access_rights('read')
                record.check_access_rule("read")
            except AccessError:
                return super(CrmLead, self).get_access_action(access_uid)
            user = self.env['res.users'].sudo().browse(access_uid)
            record = self.with_user(user)
        if user.share or self.env.context.get('force_website'):
            try:
                record.check_access_rights('read')
                record.check_access_rule('read')
            except AccessError:
                pass
            else:
                return {
                    'type': 'ir.actions.act_url',
                    'url': '/my/opportunity/%s' % record.id,
                }
        return super(CrmLead, self).get_access_action(access_uid)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.addons.http_routing.models.ir_http import slug


class ResPartnerGrade(models.Model):
    _name = 'res.partner.grade'
    _order = 'sequence'
    _inherit = ['website.published.mixin']
    _description = 'Partner Grade'

    sequence = fields.Integer('Sequence')
    active = fields.Boolean('Active', default=lambda *args: 1)
    name = fields.Char('Level Name', translate=True)
    partner_weight = fields.Integer('Level Weight', default=1,
        help="Gives the probability to assign a lead to this partner. (0 means no assignment.)")

    def _compute_website_url(self):
        super(ResPartnerGrade, self)._compute_website_url()
        for grade in self:
            grade.website_url = "/partners/grade/%s" % (slug(grade))

    def _default_is_published(self):
        return True


class ResPartnerActivation(models.Model):
    _name = 'res.partner.activation'
    _order = 'sequence'
    _description = 'Partner Activation'

    sequence = fields.Integer('Sequence')
    name = fields.Char('Name', required=True)


class ResPartner(models.Model):
    _inherit = "res.partner"

    partner_weight = fields.Integer(
        'Level Weight', compute='_compute_partner_weight',
        readonly=False, store=True, tracking=True,
        help="This should be a numerical value greater than 0 which will decide the contention for this partner to take this lead/opportunity.")
    grade_id = fields.Many2one('res.partner.grade', 'Partner Level', tracking=True)
    grade_sequence = fields.Integer(related='grade_id.sequence', readonly=True, store=True)
    activation = fields.Many2one('res.partner.activation', 'Activation', index=True, tracking=True)
    date_partnership = fields.Date('Partnership Date')
    date_review = fields.Date('Latest Partner Review')
    date_review_next = fields.Date('Next Partner Review')
    # customer implementation
    assigned_partner_id = fields.Many2one(
        'res.partner', 'Implemented by',
    )
    implemented_partner_ids = fields.One2many(
        'res.partner', 'assigned_partner_id',
        string='Implementation References',
    )
    implemented_count = fields.Integer(compute='_compute_implemented_partner_count', store=True)

    @api.depends('implemented_partner_ids', 'implemented_partner_ids.website_published', 'implemented_partner_ids.active')
    def _compute_implemented_partner_count(self):
        for partner in self:
            partner.implemented_count = len(partner.implemented_partner_ids.filtered('website_published'))

    @api.depends('grade_id.partner_weight')
    def _compute_partner_weight(self):
        for partner in self:
            partner.partner_weight = partner.grade_id.partner_weight if partner.grade_id else 0

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.addons.http_routing.models.ir_http import url_for


class Website(models.Model):
    _inherit = "website"

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Resellers'), url_for('/partners'), 'website_crm_partner_assign'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import res_partner
from . import website

```

## File: report\crm_partner_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class CrmPartnerReportAssign(models.Model):
    """ CRM Lead Report """
    _name = "crm.partner.report.assign"
    _auto = False
    _description = "CRM Partnership Analysis"

    partner_id = fields.Many2one('res.partner', 'Partner', required=False, readonly=True)
    grade_id = fields.Many2one('res.partner.grade', 'Grade', readonly=True)
    activation = fields.Many2one('res.partner.activation', 'Activation', index=True)
    user_id = fields.Many2one('res.users', 'User', readonly=True)
    date_review = fields.Date('Latest Partner Review')
    date_partnership = fields.Date('Partnership Date')
    country_id = fields.Many2one('res.country', 'Country', readonly=True)
    team_id = fields.Many2one('crm.team', 'Sales Team', readonly=True)
    nbr_opportunities = fields.Integer('# of Opportunity', readonly=True)
    turnover = fields.Float('Turnover', readonly=True)
    date = fields.Date('Invoice Account Date', readonly=True)

    _depends = {
        'account.invoice.report': ['invoice_date', 'partner_id', 'price_subtotal', 'state', 'move_type'],
        'crm.lead': ['partner_assigned_id'],
        'res.partner': ['activation', 'country_id', 'date_partnership', 'date_review',
                        'grade_id', 'parent_id', 'team_id', 'user_id'],
    }

    @property
    def _table_query(self):
        """
            CRM Lead Report
            @param cr: the current row, from the database cursor
        """
        return """
                SELECT
                    COALESCE(2 * i.id, 2 * p.id + 1) AS id,
                    p.id as partner_id,
                    (SELECT country_id FROM res_partner a WHERE a.parent_id=p.id AND country_id is not null limit 1) as country_id,
                    p.grade_id,
                    p.activation,
                    p.date_review,
                    p.date_partnership,
                    p.user_id,
                    p.team_id,
                    (SELECT count(id) FROM crm_lead WHERE partner_assigned_id=p.id) AS nbr_opportunities,
                    i.price_subtotal as turnover,
                    i.invoice_date as date
                FROM
                    res_partner p
                    left join ({account_invoice_report}) i
                        on (i.partner_id=p.id and i.move_type in ('out_invoice','out_refund') and i.state='posted')
            """.format(
                account_invoice_report=self.env['account.invoice.report']._table_query
            )

```

## File: report\crm_partner_report_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!--     Opportunity tree view  -->
        <record id="view_report_crm_partner_assign_filter" model="ir.ui.view">
            <field name="name">crm.partner.report.assign.select</field>
            <field name="model">crm.partner.report.assign</field>
            <field name="arch" type="xml">
                <search string="Partner assigned Analysis">
                    <field name="team_id"/>
                    <field name="user_id"/>
                    <field name="grade_id"/>
                    <field name="activation"/>
                    <filter name="filter_date_partnership" date="date_partnership"/>
                    <filter name="filter_date_review" date="date_review"/>
                    <group  expand="1" string="Group By">
                        <filter string="Salesperson" name="user"
                            context="{'group_by':'user_id'}" />
                        <filter string="Sales Team" name="sales_team"
                            context="{'group_by':'team_id'}"/>
                        <filter string="Partner" name="partner"
                            context="{'group_by':'partner_id'}" />
                        <separator/>
                        <filter string="Date Partnership" name="group_date_partnership"
                            context="{'group_by':'date_partnership'}" />
                        <filter string="Date Review" name="group_date_review"
                            context="{'group_by':'date_review'}" />
                    </group>
                </search>
            </field>
        </record>

       <record id="view_report_crm_partner_assign_graph" model="ir.ui.view">
            <field name="name">crm.partner.assign.report.graph</field>
            <field name="model">crm.partner.report.assign</field>
            <field name="arch" type="xml">
                <graph string="Opportunities Assignment Analysis" sample="1" disable_linking="1">
                    <field name="grade_id"/>
                    <field name="nbr_opportunities" type="measure"/>
                    <field name="turnover" type="measure"/>
                </graph>
            </field>
       </record>

       <!-- Leads by user and team Action -->

       <record id="action_report_crm_partner_assign" model="ir.actions.act_window">
            <field name="name">Partnership Analysis</field>
            <field name="res_model">crm.partner.report.assign</field>
            <field name="context">{'group_by_no_leaf':1,'group_by':[]}</field>
            <field name="view_mode">graph</field>
            <field name="domain">[('grade_id', '!=', False)]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p>
            </field>
        </record>

       <menuitem name="Partnerships" id="menu_report_crm_partner_assign_tree"
           parent="crm.crm_menu_report" action="action_report_crm_partner_assign" sequence="5"/>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_partner_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_crm_partner_report,crm.partner.report.assign.all,model_crm_partner_report_assign,sales_team.group_sale_salesman,1,0,0,0
access_res_partner_grade,res.partner.grade,model_res_partner_grade,sales_team.group_sale_salesman,1,1,1,0
access_res_partner_grade_employee,res.partner.grade,model_res_partner_grade,base.group_user,1,0,0,0
access_res_partner_grade_portal,res.partner.grade,model_res_partner_grade,base.group_portal,1,0,0,0
access_res_partner_grade_public,res.partner.grade,model_res_partner_grade,base.group_public,1,0,0,0
access_res_partner_grade_manager,res.partner.grade.manager,model_res_partner_grade,sales_team.group_sale_manager,1,1,1,1
access_res_partner_activation_user,res.partner.activation.user,model_res_partner_activation,base.group_user,1,0,0,0
access_partner_activation_manager,res.partner.activation.manager,model_res_partner_activation,base.group_partner_manager,1,1,1,1
partner_access_crm_lead,crm.lead,crm.model_crm_lead,base.group_portal,1,1,0,0
access_res_partner_grade_invoicing_payment_readonly,res.partner.grade,model_res_partner_grade,account.group_account_readonly,1,0,0,0
access_res_partner_grade_invoicing_payment,res.partner.grade,model_res_partner_grade,account.group_account_invoice,1,0,0,0
access_crm_lead_forward_to_partner,access.crm.lead.forward.to.partner,model_crm_lead_forward_to_partner,sales_team.group_sale_salesman,1,1,1,0
access_crm_lead_assignation,access.crm.lead.assignation,model_crm_lead_assignation,sales_team.group_sale_salesman,1,1,1,0

```

## File: security\ir_rule.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <!-- CRM Lead portal -->
        <record id="assigned_lead_portal_rule_1" model="ir.rule">
            <field name="name">Portal Graded Partner: read and write assigned leads</field>
            <field name="model_id" ref="crm.model_crm_lead"/>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
            <field name="domain_force">[('partner_assigned_id','child_of',user.commercial_partner_id.id)]</field>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

        <!-- Partner Grade : portal and public -->
        <record id="res_partner_grade_rule_portal_public" model="ir.rule">
            <field name="name">Portal/Public user: read only website published</field>
            <field name="model_id" ref="website_crm_partner_assign.model_res_partner_grade"/>
            <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
            <field name="domain_force">[('website_published','=', True)]</field>
            <field name="perm_read" eval="True"/>
        </record>

    <record id="ir_rule_crm_partner_report_assign_all" model="ir.rule">
         <field name="name">CRM partner assign report: All Assignations</field>
         <field name="model_id" ref="model_crm_partner_report_assign"/>
         <field name="domain_force">[(1, '=', 1)]</field>
         <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
     </record>

     <record id="ir_rule_crm_partner_report_assign_salesman" model="ir.rule">
         <field name="name">CRM partner assign report: Personal / Global Assignations</field>
         <field name="model_id" ref="model_crm_partner_report_assign"/>
         <field name="domain_force">['|', ('user_id', '=', user.id), ('user_id', '=', False)]</field>
         <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
     </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M42.525 39.25H32.346l.228 1.125h9.32c.534 0 .93.503.812 1.03l-.191.854a1.971 1.971 0 0 1 1.096 1.772c0 1.097-.886 1.985-1.973 1.969-1.035-.015-1.887-.866-1.915-1.914a1.975 1.975 0 0 1 .583-1.461h-7.28c.361.357.585.855.585 1.406 0 1.119-.921 2.02-2.037 1.967-.991-.047-1.797-.857-1.849-1.86a1.973 1.973 0 0 1 .974-1.814l-2.44-12.074h-2.426a.839.839 0 0 1-.833-.844v-.562c0-.466.373-.844.833-.844h3.56c.396 0 .737.282.817.675l.318 1.575h13.638c.535 0 .931.503.813 1.03l-1.641 7.313a.836.836 0 0 1-.813.657zm-3.358-5.344H37.5V32.5a.559.559 0 0 0-.556-.563h-.555a.559.559 0 0 0-.556.563v1.406h-1.666a.559.559 0 0 0-.556.563v.562c0 .31.249.563.556.563h1.666V37c0 .31.25.562.556.562h.555A.559.559 0 0 0 37.5 37v-1.406h1.667a.559.559 0 0 0 .555-.563v-.562a.559.559 0 0 0-.555-.563zM26.385 8l-5.9 8.058 9.51 3.036-1.28-3.953a22.208 22.208 0 0 1 7.2-1.203c11.135 0 20.316 8.182 21.904 18.866l3.228-2.94c-2.799-11.282-12.995-19.668-25.132-19.668a25.8 25.8 0 0 0-8.365 1.394L26.385 8zm-8.44 9.452C13.046 22.167 10 28.769 10 36.088c0 7.763 3.434 14.745 8.861 19.496l-2.54 2.807 9.93 1.07-2.12-9.739-2.768 3.074a22.075 22.075 0 0 1-7.6-16.708 22.073 22.073 0 0 1 8.326-17.319l-2.922-.935-1.223-.382zM60.97 32.766l-7.39 6.702 4.068.879c-1.98 10.21-10.933 17.891-21.733 17.891-2.858 0-5.591-.528-8.097-1.508l.916 4.258A25.918 25.918 0 0 0 35.914 62c12.565 0 23.05-8.988 25.4-20.87l3.686.782-4.03-9.146z"/><path id="e" d="M42.525 37.25H32.346l.228 1.125h9.32c.534 0 .93.503.812 1.03l-.191.854a1.971 1.971 0 0 1 1.096 1.772c0 1.097-.886 1.985-1.973 1.969-1.035-.015-1.887-.866-1.915-1.914a1.975 1.975 0 0 1 .583-1.461h-7.28c.361.357.585.855.585 1.406 0 1.119-.921 2.02-2.037 1.967-.991-.047-1.797-.857-1.849-1.86a1.973 1.973 0 0 1 .974-1.814l-2.44-12.074h-2.426a.839.839 0 0 1-.833-.844v-.562c0-.466.373-.844.833-.844h3.56c.396 0 .737.282.817.675l.318 1.575h13.638c.535 0 .931.503.813 1.03l-1.641 7.313a.836.836 0 0 1-.813.657zm-3.358-5.344H37.5V30.5a.559.559 0 0 0-.556-.563h-.555a.559.559 0 0 0-.556.563v1.406h-1.666a.559.559 0 0 0-.556.563v.562c0 .31.249.563.556.563h1.666V35c0 .31.25.562.556.562h.555A.559.559 0 0 0 37.5 35v-1.406h1.667a.559.559 0 0 0 .555-.563v-.562a.559.559 0 0 0-.555-.563zM26.385 6l-5.9 8.058 9.51 3.036-1.28-3.953a22.208 22.208 0 0 1 7.2-1.203c11.135 0 20.316 8.182 21.904 18.866l3.228-2.94C58.248 16.582 48.052 8.196 35.915 8.196A25.8 25.8 0 0 0 27.55 9.59L26.385 6zm-8.44 9.452C13.046 20.167 10 26.769 10 34.088c0 7.763 3.434 14.745 8.861 19.496l-2.54 2.807 9.93 1.07-2.12-9.739-2.768 3.074a22.075 22.075 0 0 1-7.6-16.708 22.073 22.073 0 0 1 8.326-17.319l-2.922-.935-1.223-.382zM60.97 30.766l-7.39 6.702 4.068.879c-1.98 10.21-10.933 17.891-21.733 17.891-2.858 0-5.591-.528-8.097-1.508l.916 4.258A25.918 25.918 0 0 0 35.914 60c12.565 0 23.05-8.988 25.4-20.87l3.686.782-4.03-9.146z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M38.386 69H4c-2 0-4-1-4-4V36.453l14.194-16.398 3.743-4.473 6.001-3.975C38.52 11.29 47.207 12.422 50 15c1.762 1.626 5.428 5.916 10.998 12.868v3.175l3.838 8.813-8.77 9.88-1.372 2.168L38.386 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-opacity=".3" fill-rule="nonzero" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\crm_partner_assign.js

```javascript
odoo.define('crm.partner_assign', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var time = require('web.time');

publicWidget.registry.crmPartnerAssign = publicWidget.Widget.extend({
    selector: '#wrapwrap:has(.interested_partner_assign_form, .desinterested_partner_assign_form, .opp-stage-button, .new_opp_form)',
    events: {
        'click .interested_partner_assign_confirm': '_onInterestedPartnerAssignConfirm',
        'click .desinterested_partner_assign_confirm': '_onDesinterestedPartnerAssignConfirm',
        'click .opp-stage-button': '_onOppStageButtonClick',
        'change .edit_contact_form .country_id': '_onEditContactFormChange',
        'click .edit_contact_confirm': '_onEditContactConfirm',
        'click .new_opp_confirm': '_onNewOppConfirm',
        'click .edit_opp_confirm': '_onEditOppConfirm',
        'change .edit_opp_form .next_activity': '_onChangeNextActivity',
        'click div.input-group.date[data-target-input="nearest"]': '_onCalendarInputGroupClick',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {jQuery} $btn
     * @param {function} callback
     * @returns {Promise}
     */
    _buttonExec: function ($btn, callback) {
        // TODO remove once the automatic system which does this lands in master
        $btn.prop('disabled', true);
        return callback.call(this).guardedCatch(function () {
            $btn.prop('disabled', false);
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _confirmInterestedPartner: function () {
        return this._rpc({
            model: 'crm.lead',
            method: 'partner_interested',
            args: [
                [parseInt($('.interested_partner_assign_form .assign_lead_id').val())],
                $('.interested_partner_assign_form .comment_interested').val()
            ],
        }).then(function () {
            window.location.href = '/my/leads';
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _confirmDesinterestedPartner: function () {
        return this._rpc({
            model: 'crm.lead',
            method: 'partner_desinterested',
            args: [
                [parseInt($('.desinterested_partner_assign_form .assign_lead_id').val())],
                $('.desinterested_partner_assign_form .comment_desinterested').val(),
                $('.desinterested_partner_assign_form .contacted_desinterested').prop('checked'),
                $('.desinterested_partner_assign_form .customer_mark_spam').prop('checked'),
            ],
        }).then(function () {
            window.location.href = '/my/leads';
        });
    },
    /**
     * @private
     * @param {}
     * @returns {Promise}
     */
    _changeOppStage: function (leadID, stageID) {
        return this._rpc({
            model: 'crm.lead',
            method: 'write',
            args: [[leadID], {
                stage_id: stageID,
            }],
            context: _.extend({website_partner_assign: 1}),
        }).then(function () {
            window.location.reload();
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _editContact: function () {
        return this._rpc({
            model: 'crm.lead',
            method: 'update_contact_details_from_portal',
            args: [[parseInt($('.edit_contact_form .opportunity_id').val())], {
                partner_name: $('.edit_contact_form .partner_name').val(),
                phone: $('.edit_contact_form .phone').val(),
                mobile: $('.edit_contact_form .mobile').val(),
                email_from: $('.edit_contact_form .email_from').val(),
                street: $('.edit_contact_form .street').val(),
                street2: $('.edit_contact_form .street2').val(),
                city: $('.edit_contact_form .city').val(),
                zip: $('.edit_contact_form .zip').val(),
                state_id: parseInt($('.edit_contact_form .state_id').find(':selected').attr('value')),
                country_id: parseInt($('.edit_contact_form .country_id').find(':selected').attr('value')),
            }],
        }).then(function () {
            window.location.reload();
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _createOpportunity: function () {
        return this._rpc({
            model: 'crm.lead',
            method: 'create_opp_portal',
            args: [{
                contact_name: $('.new_opp_form .contact_name').val(),
                title: $('.new_opp_form .title').val(),
                description: $('.new_opp_form .description').val(),
            }],
        }).then(function (response) {
            if (response.errors) {
                $('#new-opp-dialog .alert').remove();
                $('#new-opp-dialog div:first').prepend('<div class="alert alert-danger">' + response.errors + '</div>');
                return Promise.reject(response);
            } else {
                window.location = '/my/opportunity/' + response.id;
            }
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _editOpportunity: function () {
        return this._rpc({
            model: 'crm.lead',
            method: 'update_lead_portal',
            args: [[parseInt($('.edit_opp_form .opportunity_id').val())], {
                date_deadline: this._parse_date($('.edit_opp_form .date_deadline').val()),
                expected_revenue: parseFloat($('.edit_opp_form .expected_revenue').val()),
                probability: parseFloat($('.edit_opp_form .probability').val()),
                activity_type_id: parseInt($('.edit_opp_form .next_activity').find(':selected').attr('data')),
                activity_summary: $('.edit_opp_form .activity_summary').val(),
                activity_date_deadline: this._parse_date($('.edit_opp_form .activity_date_deadline').val()),
                priority: $('input[name="PriorityRadioOptions"]:checked').val(),
            }],
        }).then(function () {
            window.location.reload();
        });
    },


    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onInterestedPartnerAssignConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        if ($('.interested_partner_assign_form .comment_interested').val() && $('.interested_partner_assign_form .contacted_interested').prop('checked')) {
            this._buttonExec($(ev.currentTarget), this._confirmInterestedPartner);
        } else {
            $('.interested_partner_assign_form .error_partner_assign_interested').css('display', 'block');
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onDesinterestedPartnerAssignConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec($(ev.currentTarget), this._confirmDesinterestedPartner);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onOppStageButtonClick: function (ev) {
        var $btn = $(ev.currentTarget);
        this._buttonExec(
            $btn,
            this._changeOppStage.bind(this, parseInt($btn.attr('opp')), parseInt($btn.attr('data')))
        );
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onEditContactFormChange: function (ev) {
        var countryID = $('.edit_contact_form .country_id').find(':selected').attr('value');
        $('.edit_contact_form .state[country!=' + countryID + ']').css('display', 'none');
        $('.edit_contact_form .state[country=' + countryID + ']').css('display', 'block');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onEditContactConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec($(ev.currentTarget), this._editContact);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onNewOppConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec($(ev.currentTarget), this._createOpportunity);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onEditOppConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec($(ev.currentTarget), this._editOpportunity);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeNextActivity: function (ev) {
        var $selected = $('.edit_opp_form .next_activity').find(':selected');
        if ($selected.attr('activity_summary')) {
            $('.edit_opp_form .activity_summary').val($selected.attr('activity_summary'));
        }
        if ($selected.attr('delay_count')) {
            var now = moment();
            var date = now.add(parseInt($selected.attr('delay_count')), $selected.attr('delay_unit'));
            $('.edit_opp_form .activity_date_deadline').val(date.format(time.getLangDateFormat()));
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onCalendarInputGroupClick: function (ev) {
        const $calendarInputGroup = $(ev.currentTarget);
        const calendarOptions = {
            format : time.getLangDateFormat(),
            icons: {
                time: 'fa fa-clock-o',
                date: 'fa fa-calendar',
                up: 'fa fa-chevron-up',
                down: 'fa fa-chevron-down',
            },
        };
        // in some screen sizes, the automatic position opens the picker below input,
        // but because #next_activity_div is the last element, we always open the
        // picker on top the input to prevent extra scroll
        if ($calendarInputGroup.is('#next_activity_div')) {
            calendarOptions.widgetPositioning = {vertical: 'top'};
        }
        $calendarInputGroup.datetimepicker(calendarOptions);
    },

    _parse_date: function (value) {
        console.log(value);
        var date = moment(value, time.getLangDateFormat(), true);
        if (date.isValid() && date.year() >= 1900) {
            return time.date_to_str(date.toDate());
        }
        else {
            return false;
        }
    },
});
});

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_crm_lead_opportunity_geo_assign_form" model="ir.ui.view">
            <field name="name">crm.lead.geo_assign.inherit</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_lead_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//notebook/page[last()]" position="after">
                    <page string="Assigned Partner" name="assigned_partner" groups="sales_team.group_sale_salesman">
                        <group col="3">
                            <label for="partner_latitude" string="Geolocation"/>
                            <div class="o_row">
                                <span class="oe_grey">( </span>
                                <field name="partner_latitude"/>
                                <span class="oe_grey" attrs="{'invisible':[('partner_latitude','&lt;=',0)]}">N </span>
                                <span class="oe_grey" attrs="{'invisible':[('partner_latitude','&gt;=',0)]}">S </span>
                                <field name="partner_longitude"/>
                                <span class="oe_grey" attrs="{'invisible':[('partner_longitude','&lt;=',0)]}">E </span>
                                <span class="oe_grey" attrs="{'invisible':[('partner_longitude','&gt;=',0)]}">W </span>
                                <span class="oe_grey">) </span>
                            </div>
                            <button string="Automatic Assignment" name="action_assign_partner" type="object" class="btn-link"/>

                            <field name="partner_assigned_id" domain="[('grade_id','!=',False)]"/>
                            <button string="Send Email" type="action" class="btn-link"
                                attrs="{'invisible':[('partner_assigned_id','=',False)]}"
                                name="%(crm_lead_forward_to_partner_act)d"
                                context="{'default_composition_mode': 'forward','hide_forward_type': 1 , 'default_partner_ids': [partner_assigned_id]}"/>
                        </group>
                    </page>
                </xpath>
            </field>
        </record>

        <record id="view_crm_opportunity_geo_assign_tree" model="ir.ui.view">
            <field name="name">crm.lead.geo_assign.tree.inherit</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_tree_view_oppor"/>
            <field name="arch" type="xml">
                <field name="priority" position="after">
                    <field name="partner_assigned_id" optional="hide"/>
                    <field name="date_partner_assign" invisible="1"/>
                 </field>
            </field>
        </record>

        <record model="ir.ui.view" id="crm_opportunity_partner_filter">
            <field name="name">crm.opportunity.partner.filter.assigned</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.view_crm_case_opportunities_filter"/>
            <field name="arch" type="xml">
                <filter name="stage" position="after">
                    <filter string="Assigned Partner" name="assigned_partner" domain="[]" context="{'group_by':'partner_assigned_id'}"/>
                </filter>
                <filter name="unassigned" position="after">
                    <filter string="My Assigned Partners" name="my_assigned_partners" domain="[('partner_assigned_id.user_id', '=', uid)]"/>
                </filter>
                <field name="phone_mobile_search" position="after">
                    <field name="partner_assigned_id"/>
                </field>
            </field>
        </record>

        <record id="view_crm_lead_geo_assign_tree" model="ir.ui.view">
            <field name="name">crm.lead.lead.geo_assign.tree.inherit</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_tree_view_leads"/>
            <field name="arch" type="xml">
                <field name="partner_id" position="after">
                    <field name="partner_assigned_id" optional="show"/>
                </field>
            </field>
        </record>

        <record model="ir.ui.view" id="crm_lead_partner_filter">
            <field name="name">crm.lead.partner.filter.assigned</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.view_crm_case_leads_filter"/>
            <field name="arch" type="xml">
                <filter name="company" position="after">
                    <filter string="Assigned Partner" name="assigned_partner" domain="[]" context="{'group_by': 'partner_assigned_id'}"/>
                </filter>
                <filter name="unassigned_leads" position="after">
                    <filter string="My Assigned Partners" name="my_assigned_partners" domain="[('partner_assigned_id.user_id', '=', uid)]"/>
                </filter>
                <field name="campaign_id" position="after">
                    <field name="partner_assigned_id"/>
                </field>
            </field>
        </record>

        <record id="crm_lead_view_pivot" model="ir.ui.view">
            <field name="name">crm.lead.view.pivot.inherit.partner.assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_lead_view_pivot"/>
            <field name="arch" type="xml">
                <xpath expr="//pivot" position="inside">
                    <field name="partner_latitude" invisible="1"/>
                    <field name="partner_longitude" invisible="1"/>
                </xpath>
            </field>
        </record>

        <record id="crm_lead_view_graph" model="ir.ui.view">
            <field name="name">crm.lead.view.graph.inherit.partner.assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_lead_view_graph"/>
            <field name="arch" type="xml">
                <xpath expr="//graph" position="inside">
                    <field name="partner_latitude" invisible="1"/>
                    <field name="partner_longitude" invisible="1"/>
                </xpath>
            </field>
        </record>

        <record id="crm_lead_view_graph_report_opportunity" model="ir.ui.view">
            <field name="name">crm.lead.view.graph.report.opportunity.inherit.partner.assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_opportunity_report_view_graph"/>
            <field name="arch" type="xml">
                <xpath expr="//graph" position="inside">
                    <field name="partner_latitude" invisible="1"/>
                    <field name="partner_longitude" invisible="1"/>
                </xpath>
            </field>
        </record>

        <record id="crm_lead_view_graph_report_lead" model="ir.ui.view">
            <field name="name">crm.lead.view.graph.report.lead.inherit.partner.assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_opportunity_report_view_graph_lead"/>
            <field name="arch" type="xml">
                <xpath expr="//graph" position="inside">
                    <field name="partner_latitude" invisible="1"/>
                    <field name="partner_longitude" invisible="1"/>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <menuitem id="crm_menu_resellers" name="Resellers" parent="crm.crm_menu_config" sequence="16"/>

        <!--Partner Activation -->

        <record model="ir.ui.view" id="res_partner_activation_form">
            <field name="name">res.partner.activation.form</field>
            <field name="model">res.partner.activation</field>
            <field name="arch" type="xml">
                <form string="Activation">
                    <sheet>
                        <group col="4">
                            <field name="name" />
                            <field name="sequence" />
                        </group>
                    </sheet>
                </form>
            </field>
        </record>
        <record model="ir.ui.view" id="res_partner_activation_tree">
            <field name="name">res.partner.activation.tree</field>
            <field name="model">res.partner.activation</field>
            <field name="arch" type="xml">
                <tree string="Activation" editable="bottom">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="res_partner_activation_act">
            <field name="name">Partner Activations</field>
            <field name="res_model">res.partner.activation</field>
            <field name="view_mode">tree,form</field>
        </record>

        <menuitem id="res_partner_activation_config_mi" parent="crm_menu_resellers" action="res_partner_activation_act"
            sequence="6"/>

    <!--Partner Level -->

    <record id="view_partner_grade_tree" model="ir.ui.view">
        <field name="name">res.partner.grade.tree</field>
        <field name="model">res.partner.grade</field>
        <field name="arch" type="xml">
            <tree string="Partner Level">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="view_partner_grade_form" model="ir.ui.view">
        <field name="name">res.partner.grade.form</field>
        <field name="model">res.partner.grade</field>
        <field name="arch" type="xml">
            <form string="Partner Level">
                <sheet string="Level">
                    <div class="oe_button_box" name="button_box">
                        <field name="is_published" widget="website_redirect_button"/>
                    </div>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Gold Partner" required="True"/>
                        </h1>
                    </div>
                    <group>
                        <field name="partner_weight"/>
                        <field name="sequence"/>
                        <field name="active" widget="boolean_toggle"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="res_partner_grade_view_search" model="ir.ui.view">
        <field name="name">res.partner.grade.view.search</field>
        <field name="model">res.partner.grade</field>
        <field name="arch" type="xml">
            <search string="Search Partner Grade">
                <field name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="res_partner_grade_action" model="ir.actions.act_window">
        <field name="name">Partner Level</field>
        <field name="res_model">res.partner.grade</field>
        <field name="search_view_id" ref="res_partner_grade_view_search"/>
    </record>

    <menuitem action="res_partner_grade_action" id="menu_res_partner_grade_action"
        parent="crm_menu_resellers"
        sequence="5" />

    <!-- Partner form -->
    <record id="view_res_partner_filter_assign_tree" model="ir.ui.view">
        <field name="name">res.partner.geo.inherit.tree</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_tree"/>
        <field name="arch" type="xml">
            <field name="vat" position="after">
                <field name="date_review_next" optional="hide"/>
                <field name="grade_id" optional="hide"/>
                <field name="activation" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="view_res_partner_filter_assign" model="ir.ui.view">
        <field name="name">res.partner.geo.inherit.search</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_res_partner_filter"/>
        <field name="arch" type="xml">
            <field name="user_id" position="after">
                <field name="grade_id"/>
            </field>
        </field>
    </record>

    <record id="view_crm_partner_assign_form" model="ir.ui.view">
        <field name="name">res.partner.assign.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base_geolocalize.view_crm_partner_geo_form"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//page[@name='geo_location']" position="inside">
                    <group>
                        <group>
                            <separator string="Partner Activation" colspan="2"/>
                            <field name="grade_id" options="{'no_open': True, 'no_create': True}"/>
                            <field name="activation" options="{'no_open': True, 'no_create': True}"/>
                            <field name="partner_weight"/>
                        </group>
                        <group>
                            <separator string="Partner Review" colspan="2"/>
                            <field name="date_review"/>
                            <field name="date_review_next"/>
                            <field name="date_partnership"/>
                        </group>
                    </group>
                </xpath>
                <xpath expr="//group[@name='sale']" position="inside">
                    <field name="assigned_partner_id" groups="base.group_no_one"/>
                </xpath>
            </data>
        </field>
    </record>

</odoo>

```

## File: views\website_crm_partner_assign_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Page -->
<template id="layout" name="Partners Layout">
    <t t-call="website.layout">
        <t t-set="additional_title">Resellers</t>
        <div id="wrap">
            <div class="oe_structure" id="oe_structure_website_crm_partner_assign_layout_1"/>
            <div class="container mt16">
                <div class="row">
                    <t t-out="0" />
                </div>
            </div>
            <div class="oe_structure" id="oe_structure_website_crm_partner_assign_layout_2"/>
        </div>
    </t>
</template>

<template id="index" name="Find Resellers">
    <t t-call="website_crm_partner_assign.layout">
        <div class="col-lg-12">
            <h1 class="text-center">
                Looking For a Local Store?
            </h1><h2 class="text-center text-muted">
                Contact a reseller
            </h2>
        </div>

        <div class="col-lg-3 mb32" id="partner_left">

            <ul id="reseller_grades" class="nav nav-pills flex-column mt16">
                <li class="nav-header nav-item"><h3>Filter by Level</h3></li>
                <t t-foreach="grades" t-as="grade">
                    <li class="nav-item">
                        <a t-attf-href="/partners#{ grade['grade_id'][0] and '/grade/%s' % grade['grade_id'][0] or '' }#{ current_country and '/country/%s' % slug(current_country) or '' }#{ '?' + (search_path or '') + '&amp;' + keep_query('country_all') }"
                           t-attf-class="nav-link#{grade['active'] and ' active' or ''}">
                            <span class="badge badge-pill float-right" t-esc="grade['grade_id_count'] or ''"/>
                            <t t-esc="grade['grade_id'][1]"/>
                        </a>
                    </li>
                </t>
            </ul>

            <ul id="reseller_countries" class="nav nav-pills flex-column mt16">
                <li class="nav-header nav-item"><h3>Filter by Country</h3></li>
                <t t-foreach="countries" t-as="country">
                    <li t-if="country['country_id']" class="nav-item">
                        <a t-attf-href="/partners#{ current_grade and '/grade/%s' % slug(current_grade) or ''}#{country['country_id'][0] and '/country/%s' % country['country_id'][0] or '' }#{ '?' + (search_path or '') + (country['country_id'][0] == 0 and '&amp;country_all=True' or '')}"
                           t-attf-class="nav-link#{country['active'] and ' active' or ''}">
                            <span class="badge badge-pill float-right" t-esc="country['country_id_count'] or ''"/>
                            <t t-esc="country['country_id'][1]"/>
                        </a>
                    </li>
                </t>
            </ul>

        </div>

        <div class="col-lg-8 offset-lg-1" id="ref_content">
            <div class="d-flex p-2">
                <t t-call="website.pager"/>
                <form action="" method="get" class="form-inline ml-auto">
                    <div class="form-group">
                        <input t-if="country_all" type="hidden" name="country_all" value="True" />
                        <input type="text" name="search" class="search-query form-control" placeholder="Search" t-att-value="searches.get('search', '')"/>
                    </div>
                </form>
            </div>
            <div>
                <p t-if="not partners">No result found</p>
                <t t-foreach="partners" t-as="partner">
                    <t t-if="last_grade != partner.grade_id.id">
                        <h3 class="text-center mt-4">
                            <span t-field="partner.grade_id"/>
                            <t t-call="website.publish_management">
                                <t t-set="object" t-value="partner.grade_id"/>
                                <t t-set="publish_edit" t-value="True"/>
                            </t>
                        </h3>
                        <t t-set="last_grade" t-value="partner.grade_id.id"/>
                    </t>
                    <div class="media mt-3">
                        <a t-attf-href="/partners/#{slug(partner)}?#{current_grade and 'grade_id=%s&amp;' % current_grade.id}#{current_country and 'country_id=%s' % current_country.id}"
                           t-field="partner.avatar_128"
                           class="mr-3 text-center o_width_128"
                           t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_128_max"}'
                        ></a>
                        <div class="media-body o_partner_body" style="min-height: 64px;">
                            <a t-attf-href="/partners/#{slug(partner)}?#{current_grade and 'grade_id=%s&amp;' % current_grade.id}#{current_country and 'country_id=%s' % current_country.id}">
                                <span t-field="partner.display_name"/>
                            </a>
                            <div t-field="partner.website_short_description"/>
                            <t t-if="any(p.website_published for p in partner.implemented_partner_ids)">
                                <small><a t-attf-href="/partners/#{slug(partner)}#right_column">
                                    <t t-esc="partner.implemented_count"/> reference(s)
                                </a></small>
                            </t>
                        </div>
                    </div>
                </t>
            </div>
            <div class='navbar'>
                <t t-call="website.pager">
                   <t t-set="classname" t-valuef="float-left"/>
                </t>
            </div>
        </div>
    </t>
</template>

<template id="ref_country" inherit_id="website_crm_partner_assign.index" customize_show="True" name="Left World Map">
    <xpath expr="//ul[@id='reseller_countries']" position="after">
        <t t-if="google_maps_api_key">
            <!-- modal for large map -->
            <div role="dialog" class="modal fade partner_map_modal" tabindex="-1">
              <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">World Map</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                    </header>
                    <iframe t-attf-src="/google_map/?width=898&amp;height=485&amp;partner_ids=#{ google_map_partner_ids }&amp;partner_url=/partners/"
                    style="width:898px; height:485px; border:0; padding:0; margin:0;"></iframe>
                </div>
              </div>
            </div>
            <!-- modal end -->
            <h3>World Map<button class="btn btn-link" data-toggle="modal" data-target=".partner_map_modal"><span class="fa fa-external-link" role="img" aria-label="External link" title="External link"/></button></h3>
            <ul class="nav">
                <iframe t-attf-src="/google_map?width=260&amp;height=240&amp;partner_ids=#{ google_map_partner_ids }&amp;partner_url=/partners/"
                    style="width:260px; height:240px; border:0; padding:0; margin:0;" scrolling="no"></iframe>
            </ul>
        </t>
    </xpath>
</template>

<template id="partner" name="Partner Detail">
    <t t-call="website_crm_partner_assign.layout">
        <div class="col-lg-5">
            <ol t-if="not edit_page" class="breadcrumb">
                <li class="breadcrumb-item"><a t-attf-href="/partners#{current_grade and '/grade/%s' % slug(current_grade)}#{current_country and '/country/%s' % slug(current_country)}">Our Partners</a></li>
                <li class="breadcrumb-item active"><span t-field="partner.display_name"/></li>
            </ol>
        </div>
        <t t-call="website_partner.partner_detail">
            <t t-set="right_column">
                <div id="right_column" class="mb16"><t t-call="website_crm_partner_assign.references_block"/></div>
            </t>
        </t>
    </t>
</template>

<template id="grade_in_detail" inherit_id="website_partner.partner_detail">
  <xpath expr="//*[@id='partner_name']" position="after">
    <h3 class="col-lg-12 text-center text-muted" t-if="partner.grade_id and partner.grade_id.website_published">
      <span t-field="partner.grade_id"/></h3>
  </xpath>
</template>

<template id="references_block" name="Partner References Block">
    <t t-if="any(p.website_published for p in partner.implemented_partner_ids)">
        <h3 id="references">References</h3>
        <div t-foreach="partner.implemented_partner_ids" t-if="reference.website_published" t-as="reference" class="media mt-3">
            <span t-field="reference.avatar_128" class="d-block mr-3 text-center o_width_128" t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_128_max"}'/>
            <div class="media-body" style="min-height: 64px;">
                <span t-field="reference.self"/>
                <div t-field='reference.website_short_description'/>
            </div>
        </div>
    </t>
</template>

<!-- Portal -->
    <template id="portal_my_home_menu_lead" name="Portal layout : lead menu entry" inherit_id="portal.portal_breadcrumbs" priority="15">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <li t-if="page_name == 'lead' or lead" t-attf-class="breadcrumb-item #{'active ' if not lead else ''}">
                <a t-if="lead" t-attf-href="/my/leads?{{ keep_query() }}">
                    Leads
                </a>
                <t t-else="">
                    Leads
                </t>
            </li>
            <li t-if="lead" class="breadcrumb-item active">
                <span t-field="lead.name"/>
            </li>
            <li t-if="page_name == 'opportunity' or opportunity" t-attf-class="breadcrumb-item #{'active ' if not opportunity else ''}">
                <a t-if="opportunity" t-attf-href="/my/opportunities?{{ keep_query() }}">
                    Opportunities
                </a>
                <t t-else="">
                    Opportunities
                </t>
            </li>
            <li t-if="opportunity" class="breadcrumb-item active">
                <span t-field="opportunity.name"/>
            </li>
        </xpath>
    </template>

    <template id="portal_my_home_lead" name="Show Leads / Opps" customize_show="True" inherit_id="portal.portal_my_home" priority="15">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Leads</t>
                <t t-set="url" t-value="'/my/leads'"/>
                <t t-set="placeholder_count" t-value="'lead_count'"/>
            </t>
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Opportunities</t>
                <t t-set="url" t-value="'/my/opportunities'"/>
                <t t-set="placeholder_count" t-value="'opp_count'"/>
            </t>
        </xpath>
    </template>

    <template id="portal_my_leads" name="My Leads">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Leads</t>
            </t>
            <div t-if="not leads" class="alert alert-warning mt8" role="alert">
                There are no leads.
            </div>
            <t t-if="leads" t-call="portal.portal_table">
                <thead>
                    <tr>
                        <th>Date</th>
                        <th class="w-75">Name</th>
                        <th>Contact Name</th>
                    </tr>
                </thead>
                <tbody>
                    <tr t-foreach="leads" t-as="lead">
                        <td><span t-field="lead.create_date" t-options='{"widget": "date"}' /></td>
                        <td>
                            <a t-attf-href="/my/lead/#{lead.id}"><span t-field="lead.name"/></a>
                        </td>
                        <td><span t-field="lead.contact_name" /></td>
                    </tr>
                </tbody>
            </t>
        </t>
    </template>

    <template id="portal_my_opportunities" name="My Opportunities">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Opportunities</t>

                <div class="form-inline ml-lg-4" t-if="request.env.user.partner_id.grade_id or request.env.user.commercial_partner_id.grade_id">
                    <button class="btn btn-success btn-sm" name='new_opp' data-toggle="modal" data-target=".modal_new_opp" title="Add an opportunity" aria-label="Add an opportunity">
                        <i class="fa fa-plus"/> Create New
                    </button>
                </div>
            </t>

            <div class="modal fade modal_new_opp" role="form">
                <div class="modal-dialog">
                    <form method="POST" class="modal-content js_website_submit_form new_opp_form">
                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                        <header class="modal-header">
                            <h4 class="modal-title">New Opportunity</h4>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                        </header>
                        <main class="modal-body" id="new-opp-dialog">
                            <div class="form-group">
                                <label class="col-form-label hdd4" for="contact_name">Contact name</label>
                                <input type='text' name="contact_name" class="form-control contact_name"/>
                            </div>
                            <div class="form-group">
                                <label class="col-form-label h4dd" for="title">Title</label>
                                <input type='text' name="title" class="form-control title"/>
                            </div>
                            <div class="form-group">
                                <label class="col-form-label hdd4" for="description">Description</label>
                                <textarea rows="3" name="description" class="form-control description"></textarea>
                            </div>
                        </main>
                        <footer class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
                            <button t-attf-class="btn btn-primary new_opp_confirm">Confirm</button>
                        </footer>
                    </form>
                </div>
            </div>
            <t t-if="not opportunities">
                <div class="alert alert-warning mt8" role="alert">
                    There are no opportunities.
                </div>
            </t>
            <t t-if="opportunities" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>Date</th>
                        <th class="w-50">Name</th>
                        <th>Contact</th>
                        <th>Expected</th>
                        <th>Stage</th>
                    </tr>
                </thead>
                <tbody>
                    <tr t-foreach="opportunities" t-as="opp">
                        <td><span t-field="opp.create_date" t-options='{"widget": "date"}' /></td>
                        <td>
                            <a t-attf-href="/my/opportunity/#{opp.id}"><span t-field="opp.name"/></a>
                        </td>
                        <td><span t-field="opp.contact_name" /></td>
                        <td>
                            <span t-if="opp.company_currency" class="text-nowrap" t-esc="opp.expected_revenue" t-options="{'widget': 'monetary', 'display_currency': opp.company_currency}"/>
                            <span t-else="" class="text-nowrap" t-esc="opp.expected_revenue"/>
                            <span> at </span>
                            <span t-field="opp.probability" />%
                        </td>
                        <td>
                            <span class="badge badge-info badge-pill" title="Current stage of the opportunity" t-esc="opp.stage_id.name" />
                        </td>
                    </tr>
                </tbody>
            </t>
        </t>
    </template>

    <template id="portal_my_lead" name="My Lead">
        <t t-call="portal.portal_layout">
            <div class="card">
                <div class="card-header">
                    <div class="row">
                        <div class="col-lg-12">
                            <span class="float-right" title="Rating" role="img" t-attf-aria-label="Rating: #{lead.priority} on 3">
                                <t t-foreach="range(1, 4)" t-as="i">
                                    <span t-attf-class="fa fa-lg fa-star#{'' if i &lt;= int(lead.priority) else '-o'}"/>
                                </t>
                            </span>
                            <h4>
                                Lead - <span t-field="lead.name"/>
                            </h4>
                        </div>
                    </div>
                </div>
                <div class="card-body">
                    <div class="row">
                        <div class="col-lg-6">
                            <div class="row" t-if="lead.partner_name or lead.email_from or lead.partner_id">
                                <label class="col-4">Customer</label>
                                <address class="col-8">
                                    <div>
                                        <span t-if="lead.partner_name" itemprop="name" t-field="lead.partner_name" />
                                        <span t-if="not lead.partner_name" itemprop="name" t-field="lead.contact_name"/>
                                    </div>
                                    <div t-if="lead.phone">
                                        <span class="fa fa-phone" role="img" aria-label="Phone" title="Phone"/> <span itemprop="telephone" t-field="lead.phone" />
                                    </div>
                                    <div t-if="lead.mobile">
                                        <span class="fa fa-mobile" role="img" aria-label="Mobile" title="Mobile"/> <span itemprop="telephone" t-field="lead.mobile" />
                                    </div>
                                    <div t-if="lead.email_from">
                                        <span class="fa fa-envelope" role="img" aria-label="Email" title="Email"/> <span itemprop="email" t-field="lead.email_from" />
                                    </div>
                                </address>
                            </div>
                            <div class="row" t-if="lead.street or lead.street2 or lead.city or lead.state_id or lead.country_id">
                                <label class="col-4">Address</label>
                                <address class="col-8">
                                    <div t-if="lead.street"><span t-field="lead.street"/></div>
                                    <div t-if="lead.street2"><span t-field="lead.street2"/></div>
                                    <div t-if="lead.city or lead.zip">
                                        <span t-field="lead.city"/> <span t-field="lead.zip"/>
                                    </div>
                                    <div t-if="lead.state_id or country_id">
                                        <span t-field="lead.state_id"/> <span t-field="lead.country_id"/>
                                    </div>
                                </address>
                            </div>
                            <div class="row" t-if="lead.user_id">
                                <label class="col-4">Salesperson</label>
                                <span class="col-8" t-field="lead.user_id" />
                            </div>
                            <div class="row" t-if="lead.team_id">
                                <label class="col-4">Sales Team</label>
                                <span class="col-8" t-field="lead.team_id" />
                            </div>
                        </div>
                        <div class="col-lg-6">
                            <div class="row" t-if="lead.date_deadline">
                                <label class="col-4">Expected Closing</label>
                                <span class="col-8" t-field="lead.date_deadline" />
                            </div>
                            <div class="row" groups="!base.group_portal" t-if="lead.tag_ids">
                                <label class="col-4">Tags</label>
                                <span class="col-8">
                                    <t t-foreach="lead.tag_ids" t-as="tag">
                                        <span class="badge badge-info" t-esc="tag.name" />
                                    </t>
                                </span>
                            </div>
                            <div class="row mt16" t-if="lead.partner_assigned_id">
                                <label class="col-4">Assigned Partner</label>
                                <address class="col-8" t-field="lead.partner_assigned_id" t-options='{"widget": "contact", "fields": ["name", "email", "phone"], "no_marker": True}'/>
                            </div>
                            <div class="row mt16" t-if="lead.campaign_id">
                                <label class="col-4">Campaign</label>
                                <span class="col-8" t-field="lead.campaign_id" />
                            </div>
                            <div class="row" t-if="lead.medium_id">
                                <label class="col-4">Medium</label>
                                <span class="col-8" t-field="lead.medium_id" />
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class='mt8'>
                <a role="button" href="#" class="btn btn-primary btn" data-toggle="modal" data-target=".modal_partner_assign_interested"><i class="fa fa-file-text-o"/> I'm interested</a>
                <a role="button" href="#" class="btn btn-primary btn" data-toggle="modal" data-target=".modal_partner_assign_desinterested"><i class="fa fa-fw fa-times"/> I'm not interested</a>
                <div class="modal fade modal_partner_assign_interested" role="form">
                    <div class="modal-dialog">
                        <form method="POST" class="js_accept_json modal-content js_website_submit_form interested_partner_assign_form">
                            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                            <input type="hidden" name="lead_id" class="assign_lead_id" t-att-value="lead.id"/>
                            <header class="modal-header">
                                <h4 class="modal-title">Lead Feedback</h4>
                                <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                            </header>
                            <main class="modal-body" id="sign-dialog">
                                <div class="form-group">
                                    <label class="col-form-label" for="comment">What is the next action? When? What is the expected revenue?</label>
                                    <input type="text" name="comment" id="comment" class="form-control comment_interested"/>
                                </div>
                                <div class="form-group">
                                    <label class="col-form-label" for="customer_contacted">I have contacted the customer</label>
                                    <input type="checkbox" name="customer_contacted" id="customer_contacted" class="contacted_interested"/>
                                </div>
                                <div>
                                    <span class="text-danger error_partner_assign_interested" style="display:none;">You need to fill up the next action and contact the customer before accepting the lead</span>
                                </div>
                            </main>
                            <footer class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
                                <button t-attf-class="btn btn-primary interested_partner_assign_confirm">Confirm</button>
                            </footer>
                        </form>
                    </div>
                </div>
                <div role="dialog" class="modal fade modal_partner_assign_desinterested">
                    <div class="modal-dialog">
                        <form method="POST" class="js_accept_json modal-content js_website_submit_form desinterested_partner_assign_form">
                            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                            <input type="hidden" name="lead_id" class="assign_lead_id" t-att-value="lead.id"/>
                            <header class="modal-header">
                                <h4 class="modal-title">Lead Feedback</h4>
                                <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                            </header>
                            <main class="modal-body" id="sign-dialog">
                                <div class="form-group">
                                    <label class="col-form-label" for="comment">Why aren't you interested in this lead?</label>
                                    <input type="text" name="comment" id="comment" class="form-control comment_desinterested"/>
                                </div>
                                <div class="form-group">
                                    <label class="col-form-label" for="contacted_desinterested">I have contacted the customer</label>
                                    <input type="checkbox" name="contacted_desinterested" id="contacted_desinterested" class="contacted_desinterested"/>
                                </div>
                                <div class="form-group">
                                    <label class="col-form-label" for="customer_mark_spam">This lead is a spam</label>
                                    <input type="checkbox" name="customer_mark_spam" id="customer_mark_spam" class="customer_mark_spam"/>
                                </div>
                                <div>
                                    <span class="text-danger error_partner_assign_desinterested" style="display:none;">You need to fill up the next action and contact the customer before accepting the lead</span>
                                </div>
                            </main>
                            <footer class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
                                <button t-attf-class="btn btn-primary desinterested_partner_assign_confirm">Confirm</button>
                            </footer>
                        </form>
                    </div>
                </div>
            </div>
            <div class="row mt32">
                <div class="col-lg-12">
                    <h4><strong>Message and communication history</strong></h4>
                </div>
                <div class="col-lg-10 offset-lg-1 mt16">
                    <t t-call="portal.message_thread">
                        <t t-set="object" t-value="lead"/>
                    </t>
                </div>
            </div>
        </t>
    </template>

    <template id="portal_my_opportunity" name="My Opportunity">
        <t t-call="portal.portal_layout">
            <t t-call="portal.portal_record_layout">
                <t t-set="card_header">
                    <div class="row no-gutters">
                        <div class="col-md">
                            <h5 class="mb-2 mb-md-0">
                                <small class="text-muted">Opportunity - </small>
                                <span t-field="opportunity.name"/>
                            </h5>
                        </div>
                        <div class="col-md text-md-right">
                            <div class="d-inline-block">
                                <small class="mr-2 mt-1 float-left"><b>Stage:</b></small>
                                <div t-foreach="stages[::-1]" t-as="stage" class="float-left">
                                    <i t-if="not stage_first" class="fa fa-chevron-right ml-2 mr-1 small" style="opacity: 0.5"/>
                                    <button type="button" t-att-data="stage.id" t-att-opp="opportunity.id" t-attf-class="btn btn-sm px-2 opp-stage-button #{'btn-secondary' if opportunity.stage_id.name != stage.name else 'btn-primary disabled'}">
                                        <span t-field="stage.name"/>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </t>

                <t t-set="card_body">
                    <div class="row">
                        <div class="col-lg-5 mb-4 mb-lg-0">
                            <div class="border-bottom d-flex justify-content-between py-2 mb-3 align-items-center">
                                <h5 class="mb-0">
                                    <span t-if="opportunity.company_currency" class="text-nowrap" t-esc="opportunity.expected_revenue" t-options="{'widget': 'monetary', 'display_currency': opportunity.company_currency}"/>
                                    <span t-else="" class="text-nowrap" t-esc="opportunity.expected_revenue"/>
                                    <span> at </span>
                                    <span class="badge badge-info badge-pill"><span t-field="opportunity.probability"/>%</span>
                                </h5>
                                <button type="button" data-toggle="modal" data-target=".modal_edit_opp" class="btn btn-link btn-sm"><i class="fa fa-pencil mr-1"/>Edit</button>
                            </div>
                            <div class="row mb-2">
                                <strong class="col-12 col-sm-4">Expected Closing</strong>
                                <div class="col">
                                    <span t-if="opportunity.date_deadline" t-field="opportunity.date_deadline"/>
                                    <span t-else="" class="text-muted"> - </span>
                                </div>
                            </div>
                            <div class="row mb-2">
                                <strong class="col-12 col-sm-4">Priority</strong>
                                <div class="col">
                                    <span class="" title="Rating" role="img" t-attf-aria-label="Rating: #{opportunity.priority} on 4">
                                        <t t-foreach="range(1, 4)" t-as="i">
                                            <span t-attf-class="fa text-warning fa-lg fa-star#{'' if i &lt;= int(opportunity.priority) else '-o'}" role="img" t-att-aria-label="'Star %d of 3' % i"/>
                                        </t>
                                    </span>
                                </div>
                            </div>
                            <div class="row">
                                <strong class="col-12 col-sm-4">Next Activity</strong>
                                <div class="col" t-if="user_activity">
                                    <span t-field="user_activity.activity_type_id"/>
                                    <span t-if="user_activity.date_deadline">&#160;on&#160;</span>
                                    <span t-field="user_activity.date_deadline"/>
                                    <em class="d-block" t-field="user_activity.summary"/>
                                </div>
                                <div class="col" t-else="">
                                    <span class="text-muted"> - </span>
                                </div>
                            </div>
                        </div>

                        <div class="col-lg-6 offset-lg-1 col-xl-5 offset-xl-2">
                            <div class="d-flex justify-content-between py-2 mb-3 border-bottom align-items-center">
                                <h5 class="card-title mb-0">Contact</h5>
                                <button type="button" data-toggle="modal" data-target=".modal_edit_contact" class="btn btn-link btn-sm"><i class="fa fa-pencil mr-1"/>Edit</button>
                            </div>
                            <div class="row mb-3" t-if="opportunity.partner_name or opportunity.email_from or opportunity.contact_name">
                                <strong class="col-12 col-sm-3">Customer</strong>
                                <div class="col">
                                    <div>
                                        <span class="fa fa-user fa-fw" role="img" aria-label="User" title="User"/>
                                        <strong class="font-italic" t-if="opportunity.partner_name" itemprop="name" t-field="opportunity.partner_name" />
                                        <strong class="font-italic" t-else="" itemprop="name" t-field="opportunity.contact_name" />
                                    </div>
                                    <div>
                                        <span class="fa fa-phone fa-fw" role="img" aria-label="Phone" title="Phone"/>
                                        <span t-if="opportunity.phone" itemprop="telephone" t-field="opportunity.phone"/>
                                        <span t-else="" class="text-muted"> - </span>
                                    </div>
                                    <div>
                                        <span class="fa fa-mobile fa-fw" role="img" aria-label="Mobile" title="Mobile"/>
                                        <span t-if="opportunity.mobile" itemprop="mobile" t-field="opportunity.mobile"/>
                                        <span t-else="" class="text-muted"> - </span>
                                    </div>
                                    <div>
                                        <span class="fa fa-envelope fa-fw" role="img" aria-label="Email" title="Email"/>
                                        <span t-if="opportunity.email_from" itemprop="email" t-field="opportunity.email_from"/>
                                        <span t-else="" class="text-muted"> - </span>
                                    </div>
                                </div>
                            </div>
                            <div class="row">
                                <strong class="col-12 col-sm-3">Address</strong>
                                <address class="col d-flex align-items-baseline mb-0">
                                    <span class="fa fa-map-marker fa-fw" role="img" aria-label="Address" title="Address"/>
                                    <div>
                                        <div t-if="opportunity.street"><span t-field="opportunity.street"/></div>
                                        <span t-else="" class="text-muted"> - </span>
                                        <div t-if="opportunity.street2"><span t-field="opportunity.street2"/></div>
                                        <div t-if="opportunity.city or opportunity.zip">
                                            <span t-field="opportunity.city"/> <span t-field="opportunity.zip"/>
                                        </div>
                                        <div t-if="opportunity.state_id or country_id">
                                            <span t-field="opportunity.state_id"/> <span t-field="opportunity.country_id"/>
                                        </div>
                                    </div>
                                </address>
                            </div>
                        </div>

                        <!-- ==== MODALS ==== -->
                        <div>
                            <div role="dialog" class="modal fade modal_edit_opp">
                                <div class="modal-dialog">
                                    <form method="POST" class="js_accept_json modal-content js_website_submit_form edit_opp_form">
                                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                        <input type="hidden" name="opportunity_id" class="opportunity_id" t-att-value="opportunity.id"/>
                                        <header class="modal-header">
                                            <h4 class="modal-title">Edit Opportunity</h4>
                                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                                        </header>
                                        <main class="modal-body" id="sign-dialog">
                                            <div class="form-group">
                                                <div class="form-row align-items-center">
                                                    <div class="col-auto flex-grow-1">
                                                        <div class="input-group">
                                                            <div class="input-group-prepend">
                                                                <div class="input-group-text"><span class="text-nowrap" t-esc="opportunity.company_currency.symbol"/></div>
                                                            </div>
                                                            <input type="text" name="expected_revenue" class="form-control expected_revenue" t-att-value="opportunity.expected_revenue" placeholder="Planned Revenue"/>
                                                        </div>
                                                    </div>
                                                    <div class="col-auto">at</div>
                                                    <div class="col-auto">
                                                        <div class="input-group">
                                                            <input type="text" name="probability" class="form-control probability" t-att-value="opportunity.probability" placeholder="Probability"/>
                                                            <div class="input-group-append">
                                                                <div class="input-group-text">%</div>
                                                            </div>
                                                        </div>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <div class="row">
                                                    <div class="col-md-5 pr-0">
                                                        <label>Priority:</label>
                                                        <div class="input-group">
                                                            <label class="radio-inline">
                                                                <input type="radio" name="PriorityRadioOptions" value="0" t-att-checked="opportunity.priority not in ['1','2','3']" aria-label="Rating: 0 on 3" title="Rating: 0 on 3"/>
                                                                <i class="ml-1 fa fa-star-o"></i>
                                                            </label>
                                                            <label class="radio-inline ml-2">
                                                                <input type="radio" name="PriorityRadioOptions" value="1" t-att-checked="opportunity.priority == '1'" aria-label="Rating: 1 on 3" title="Rating: 1 on 3"/>
                                                                <i class="ml-1 fa text-warning fa-star"></i>
                                                            </label>
                                                            <label class="radio-inline ml-2">
                                                                <input type="radio" name="PriorityRadioOptions" value="2" t-att-checked="opportunity.priority == '2'" aria-label="Rating: 2 on 3" title="Rating: 2 on 3"/>
                                                                <i class="ml-1 fa text-warning fa-star"></i>
                                                                <i class="ml-1 fa text-warning fa-star"></i>
                                                            </label>
                                                            <label class="radio-inline ml-2">
                                                                <input type="radio" name="PriorityRadioOptions" value="3" t-att-checked="opportunity.priority == '3'" aria-label="Rating: 3 on 3" title="Rating: 3 on 3"/>
                                                                <i class="ml-1 fa text-warning fa-star"></i>
                                                                <i class="ml-1 fa text-warning fa-star"></i>
                                                                <i class="ml-1 fa text-warning fa-star"></i>
                                                            </label>
                                                        </div>
                                                    </div>
                                                    <div class="col-md-7">
                                                        <label>Expected Closing:</label>
                                                        <div class="input-group date" id="exp_closing_div" data-target-input="nearest">
                                                            <t t-set='date_formatted'><t t-options='{"widget": "date"}' t-esc="opportunity.date_deadline"/></t>
                                                            <input type="text" name="date_deadline" data-target="#exp_closing_div" t-att-value="date_formatted" class="datetimepicker-input form-control date_deadline" t-att-name="prefix"/>
                                                            <div class="input-group-append" data-target="#exp_closing_div" data-toggle="datetimepicker">
                                                                <span class="input-group-text">
                                                                    <span class="fa fa-calendar" role="img" aria-label="Calendar"></span>
                                                                </span>
                                                            </div>
                                                        </div>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-form-label" for="next_activity">Next Activity</label>
                                                <select class="form-control next_activity" name="next_activity">
                                                    <t t-foreach="activity_types" t-as="activity_type">
                                                        <option t-att-data="activity_type.id" t-att-selected="activity_type.id == user_activity.activity_type_id.id" t-att-name="activity_type.name" t-att-delay_count="activity_type.delay_count" t-att-delay_unit="activity_type.delay_unit" t-att-summary="activity_type.summary"><t t-esc="activity_type.name"/></option>
                                                    </t>
                                                </select>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-form-label" for="activity_summary">Details Next Activity</label>
                                                <textarea rows="3" name="activity_summary" class="form-control activity_summary"><t t-esc="user_activity.summary"/></textarea>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-form-label" for="activity_date_deadline">Next Activity Date</label>
                                                <div class="input-group date" id="next_activity_div" data-target-input="nearest">
                                                    <t t-set='date_formatted'><t t-options='{"widget": "date"}' t-esc="user_activity.date_deadline"/></t>
                                                    <input type="text" name="activity_date_deadline" data-target="#next_activity_div" t-att-value="date_formatted" class="form-control activity_date_deadline datetimepicker-input" t-att-name="prefix"/>
                                                    <div class="input-group-append" data-target="#next_activity_div" data-toggle="datetimepicker">
                                                        <span class="input-group-text">
                                                            <span class="fa fa-calendar" role="img" aria-label="Calendar" title="Calendar"></span>
                                                        </span>
                                                    </div>
                                                </div>
                                            </div>
                                        </main>
                                        <footer class="modal-footer">
                                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
                                            <button t-attf-class="btn btn-primary edit_opp_confirm">Confirm</button>
                                        </footer>
                                    </form>
                                </div>
                            </div>

                            <div role="dialog" class="modal fade modal_edit_contact">
                                <div class="modal-dialog">
                                    <form method="POST" class="js_accept_json modal-content js_website_submit_form edit_contact_form">
                                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                        <input type="hidden" name="opportunity_id" class="opportunity_id" t-att-value="opportunity.id"/>
                                        <header class="modal-header">
                                            <h4 class="modal-title">Edit Contact</h4>
                                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                                        </header>
                                        <main class="modal-body" id="sign-dialog">
                                            <t t-if="opportunity.partner_name">
                                                <div class="form-group">
                                                    <label class="col-form-label" for="partner_name">Customer Name</label>
                                                    <input type="text" name="partner_name" class="form-control partner_name" t-att-value="opportunity.partner_name"/>
                                                </div>
                                            </t>
                                            <t t-if="not opportunity.partner_name">
                                                <div class="form-group">
                                                    <label class="col-form-label" for="partner_name">Customer Name</label>
                                                    <input type="text" name="partner_name" class="form-control partner_name" t-att-value="opportunity.contact_name"/>
                                                </div>
                                            </t>
                                            <div class="form-group">
                                                <label class="col-form-label" for="phone">Phone</label>
                                                <input type="text" name="phone" class="form-control phone" t-att-value="opportunity.phone"/>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-form-label" for="mobile">Mobile</label>
                                                <input type="text" name="mobile" class="form-control mobile" t-att-value="opportunity.mobile"/>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-form-label" for="email_from">Email</label>
                                                <input type="text" name="email_from" class="form-control email_from" t-att-value="opportunity.email_from"/>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-form-label" for="street">Address</label>
                                                <input type="text" name="street" class="form-control street" t-att-value="opportunity.street" placeholder="Street"/>
                                            </div>
                                            <div class="form-group">
                                                <input type="text" name="street2" class="form-control street2" t-att-value="opportunity.street2" placeholder="Street2"/>
                                            </div>
                                            <div class="form-group">
                                                <div class="row">
                                                    <div class="col-md-5">
                                                        <input type="text" name="city" class="form-control city" t-att-value="opportunity.city" placeholder="City"/>
                                                    </div>
                                                    <div class="col-md-5">
                                                        <select name="state_id" class="form-control state_id">
                                                            <option>States...</option>
                                                            <t t-foreach="states or []" t-as="state">
                                                                <option class="state" t-att-value="state.id" t-att-country="state.country_id.id" t-att-selected="state.id == opportunity.state_id.id">
                                                                    <t t-esc="state.name"/>
                                                                </option>
                                                            </t>
                                                        </select>
                                                    </div>
                                                    <div class="col-md-2">
                                                        <input type="text" name="zip" class="form-control zip" t-att-value="opportunity.zip" placeholder="ZIP"/>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <select name="country_id" class="form-control country_id">
                                                    <option>Countries...</option>
                                                    <t t-foreach="countries or []" t-as="country">
                                                        <option t-att-value="country.id" t-att-selected="country.id == opportunity.country_id.id">
                                                            <t t-esc="country.name"/>
                                                        </option>
                                                    </t>
                                                </select>
                                            </div>
                                        </main>
                                        <footer class="modal-footer">
                                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
                                            <button t-attf-class="btn btn-primary edit_contact_confirm">Confirm</button>
                                        </footer>
                                    </form>
                                </div>
                            </div>
                        </div>
                        <!-- === / MODALS === -->
                    </div>
                </t>
            </t>

            <div class="mt32">
                <h4><strong>Message and communication history</strong></h4>
                <div class="mt16">
                    <t t-call="portal.message_thread">
                        <t t-set="object" t-value="opportunity"/>
                    </t>
                </div>
            </div>
        </t>
    </template>

</odoo>

```

## File: wizard\crm_forward_to_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class CrmLeadForwardToPartner(models.TransientModel):
    """ Forward info history to partners. """
    _name = 'crm.lead.forward.to.partner'
    _description = 'Lead forward to partner'

    @api.model
    def _convert_to_assignation_line(self, lead, partner):
        lead_location = []
        partner_location = []
        if lead.country_id:
            lead_location.append(lead.country_id.name)
        if lead.city:
            lead_location.append(lead.city)
        if partner:
            if partner.country_id:
                partner_location.append(partner.country_id.name)
            if partner.city:
                partner_location.append(partner.city)
        return {'lead_id': lead.id,
                'lead_location': ", ".join(lead_location),
                'partner_assigned_id': partner and partner.id or False,
                'partner_location': ", ".join(partner_location),
                'lead_link': self.get_lead_portal_url(lead),
                }

    @api.model
    def default_get(self, fields):
        res = super(CrmLeadForwardToPartner, self).default_get(fields)
        active_ids = self.env.context.get('active_ids')
        if 'body' in fields:
            template = self.env.ref('website_crm_partner_assign.email_template_lead_forward_mail', False)
            if template:
                res['body'] = template.body_html
        if active_ids:
            default_composition_mode = self.env.context.get('default_composition_mode')
            res['assignation_lines'] = []
            leads = self.env['crm.lead'].browse(active_ids)
            if default_composition_mode == 'mass_mail':
                partner_assigned_dict = leads.search_geo_partner()
            else:
                partner_assigned_dict = {lead.id: lead.partner_assigned_id.id for lead in leads}
                res['partner_id'] = leads[0].partner_assigned_id.id
            for lead in leads:
                partner_id = partner_assigned_dict.get(lead.id) or False
                partner = self.env['res.partner'].browse(partner_id)
                res['assignation_lines'].append((0, 0, self._convert_to_assignation_line(lead, partner)))
        return res

    def action_forward(self):
        self.ensure_one()
        template = self.env.ref('website_crm_partner_assign.email_template_lead_forward_mail', False)
        if not template:
            raise UserError(_('The Forward Email Template is not in the database'))
        portal_group = self.env.ref('base.group_portal')

        local_context = self.env.context.copy()
        if not (self.forward_type == 'single'):
            no_email = set()
            for lead in self.assignation_lines:
                if lead.partner_assigned_id and not lead.partner_assigned_id.email:
                    no_email.add(lead.partner_assigned_id.name)
            if no_email:
                raise UserError(_('Set an email address for the partner(s): %s') % ", ".join(no_email))
        if self.forward_type == 'single' and not self.partner_id.email:
            raise UserError(_('Set an email address for the partner %s', self.partner_id.name))

        partners_leads = {}
        for lead in self.assignation_lines:
            partner = self.forward_type == 'single' and self.partner_id or lead.partner_assigned_id
            lead_details = {
                'lead_link': lead.lead_link,
                'lead_id': lead.lead_id,
            }
            if partner:
                partner_leads = partners_leads.get(partner.id)
                if partner_leads:
                    partner_leads['leads'].append(lead_details)
                else:
                    partners_leads[partner.id] = {'partner': partner, 'leads': [lead_details]}

        for partner_id, partner_leads in partners_leads.items():
            in_portal = False
            if portal_group:
                for contact in (partner.child_ids or partner).filtered(lambda contact: contact.user_ids):
                    in_portal = portal_group.id in [g.id for g in contact.user_ids[0].groups_id]

            local_context['partner_id'] = partner_leads['partner']
            local_context['partner_leads'] = partner_leads['leads']
            local_context['partner_in_portal'] = in_portal
            template.with_context(local_context).send_mail(self.id)
            leads = self.env['crm.lead']
            for lead_data in partner_leads['leads']:
                leads |= lead_data['lead_id']
            values = {'partner_assigned_id': partner_id, 'user_id': partner_leads['partner'].user_id.id}
            leads.with_context(mail_auto_subscribe_no_notify=1).write(values)
            self.env['crm.lead'].message_subscribe([partner_id])
        return True

    def get_lead_portal_url(self, lead):
        action = lead.type == 'opportunity' and 'action_portal_opportunities' or 'action_portal_leads'
        action_ref = self.env.ref('website_crm_partner_assign.%s' % (action,), False)
        portal_link = "%s/?db=%s#id=%s&action=%s&view_type=form" % (
            lead.get_base_url(),
            self.env.cr.dbname,
            lead.id,
            action_ref and action_ref.id or False)
        return portal_link

    def get_portal_url(self):
        if self:
            portal_url = self[0].get_base_url()
        else:
            portal_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        portal_link = "%s/?db=%s" % (portal_url, self.env.cr.dbname)
        return portal_link

    forward_type = fields.Selection([
        ('single', 'a single partner: manual selection of partner'),
        ('assigned', "several partners: automatic assignment, using GPS coordinates and partner's grades")
    ], 'Forward selected leads to', default=lambda self: self.env.context.get('forward_type') or 'single')
    partner_id = fields.Many2one('res.partner', 'Forward Leads To')
    assignation_lines = fields.One2many('crm.lead.assignation', 'forward_id', 'Partner Assignment')
    body = fields.Html('Contents', help='Automatically sanitized HTML contents')


class CrmLeadAssignation(models.TransientModel):
    _name = 'crm.lead.assignation'
    _description = 'Lead Assignation'

    forward_id = fields.Many2one('crm.lead.forward.to.partner', 'Partner Assignment')
    lead_id = fields.Many2one('crm.lead', 'Lead')
    lead_location = fields.Char('Lead Location')
    partner_assigned_id = fields.Many2one('res.partner', 'Assigned Partner')
    partner_location = fields.Char('Partner Location')
    lead_link = fields.Char('Link to Lead')

    @api.onchange('lead_id')
    def _onchange_lead_id(self):
        lead = self.lead_id
        if not lead:
            self.lead_location = False
        else:
            lead_location = []
            if lead.country_id:
                lead_location.append(lead.country_id.name)
            if lead.city:
                lead_location.append(lead.city)
            self.lead_location = ", ".join(lead_location)

    @api.onchange('partner_assigned_id')
    def _onchange_partner_assigned_id(self):
        partner = self.partner_assigned_id
        if not partner:
            self.lead_location = False
        else:
            partner_location = []
            if partner.country_id:
                partner_location.append(partner.country_id.name)
            if partner.city:
                partner_location.append(partner.city)
            self.partner_location = ", ".join(partner_location)

```

## File: wizard\crm_forward_to_partner_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
        <record model="ir.ui.view" id="crm_lead_forward_to_partner_form">
            <field name="name">crm_lead_forward_to_partner</field>
            <field name="model">crm.lead.forward.to.partner</field>
            <field name="arch" type="xml">
                <form string="Send Mail">
                    <group>
                        <field name="forward_type" invisible="context.get('hide_forward_type',False)"/>
                    </group>
                    <group>
                        <group>
                            <field name="partner_id" attrs="{'invisible': [('forward_type', 'in', ['assigned',False])], 'required': [('forward_type', '=', 'single')]}"  />
                        </group>
                        <group>
                        </group>
                    </group>
                    <field name="assignation_lines" attrs="{'invisible': [('forward_type', 'in', ['single',False])]}">
                        <tree create="false" editable="bottom">
                            <field name="lead_id" readonly="1" force_save="1" />
                            <field name="lead_location" readonly="1"/>
                            <field name="partner_assigned_id"/>
                            <field name="partner_location" readonly="1"/>
                        </tree>
                    </field>
                    <notebook colspan="4" groups="base.group_no_one">
                        <page string="Email Template" name="email_template">
                            <field name="body" readonly="1" options="{'style-inline': true}"/>
                        </page>
                    </notebook>
                    <footer>
                        <button name="action_forward" string="Send" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" special="cancel" data-hotkey="z" class="btn-secondary"/>
                    </footer>
                </form>
            </field>
        </record>

        <record model="ir.actions.act_window" id="crm_lead_forward_to_partner_act">
            <field name="name">Forward to Partner</field>
            <field name="res_model">crm.lead.forward.to.partner</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="crm_lead_forward_to_partner_form"/>
            <field name="target">new</field>
        </record>

        <record id="action_crm_send_mass_forward" model="ir.actions.act_window">
            <field name="name">Forward to partner</field>
            <field name="res_model">crm.lead.forward.to.partner</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="crm_lead_forward_to_partner_form"/>
            <field name="target">new</field>
            <field name="context">{'default_composition_mode' : 'mass_mail'}</field>
            <field name="binding_model_id" ref="model_crm_lead"/>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_forward_to_partner
```

