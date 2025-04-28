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
    'version': '1.2',
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
        'data/crm_lead_merge_template.xml',
        'data/crm_tag_data.xml',
        'data/mail_template_data.xml',
        'data/res_partner_activation_data.xml',
        'data/res_partner_grade_data.xml',
        'security/ir.model.access.csv',
        'security/ir_rule.xml',
        'wizard/crm_forward_to_partner_view.xml',
        'views/res_partner_views.xml',
        'views/res_partner_activation_views.xml',
        'views/res_partner_grade_views.xml',
        'views/crm_lead_views.xml',
        'views/website_crm_partner_assign_templates.xml',
        'views/partner_assign_menus.xml',
        'report/crm_partner_report_view.xml',
        'views/snippets.xml',
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
        'web.assets_tests': [
            'website_crm_partner_assign/static/tests/tours/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug.urls

from werkzeug.exceptions import NotFound

from odoo import fields
from odoo import http
from odoo.http import request
from odoo.addons.portal.controllers.portal import CustomerPortal
from odoo.addons.website_google_map.controllers.main import GoogleMap
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
                if CrmLead.has_access('read')
                else 0
            )
        if 'opp_count' in counters:
            values['opp_count'] = (
                CrmLead.search_count(self.get_domain_my_opp(request.env.user))
                if CrmLead.has_access('read')
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

        searchbar_filters = {
            'all': {'label': _('Active'), 'domain': []},
            'no_activities': {
                'label': _('No Activities'),
                'domain': [('activity_ids', 'not any', [('user_id', '=', request.env.user.id)]), ('stage_id.is_won', '=', False)]
            },
            'overdue': {'label': _('Late Activities'), 'domain': [('activity_date_deadline', '<', today)]},
            'today': {'label': _('Today Activities'), 'domain': [('activity_date_deadline', '=', today)]},
            'future': {'label': _('Future Activities'), 'domain': [('activity_date_deadline', '>', today)]},
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
        # pager: bypass activities access rights for search but still apply access rules
        leads_sudo = CrmLead.sudo()._search(domain)
        domain = [('id', 'in', leads_sudo)]
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
            'searchbar_filters': searchbar_filters,
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
                'stages': request.env['crm.stage'].search([
                    ('is_won', '!=', True), '|', ('team_id', '=', False), ('team_id', '=', opp.team_id.id)
                ], order='sequence desc, name desc, id desc'),
                'activity_types': request.env['mail.activity.type'].sudo().search(['|', ('res_model', '=', opp._name), ('res_model', '=', False)]),
                'states': request.env['res.country.state'].sudo().search([]),
                'countries': request.env['res.country'].sudo().search([]),
            })


class WebsiteCrmPartnerAssign(WebsitePartnerPage, GoogleMap):
    _references_per_page = 40

    def _get_gmap_domains(self, **kw):
        if kw.get('dom', '') != "website_crm_partner_assign.partners":
            return super()._get_gmap_domains(**kw)
        current_grade = kw.get('current_grade')
        current_country = kw.get('current_country')

        domain = [('grade_id', '!=', False), ('is_company', '=', True)]
        if not request.env.user.has_group('website.group_website_restricted_editor'):
            domain += [('grade_id.website_published', '=', True)]

        if current_country:
            domain += [('country_id', '=', int(current_country))]

        if current_grade:
            domain += [('grade_id', '=', int(current_grade))]

        return domain

    def sitemap_partners(env, rule, qs):
        if not qs or qs.lower() in '/partners':
            yield {'loc': '/partners'}

        slug = env['ir.http']._slug
        base_partner_domain = [
            ('is_company', '=', True),
            ('grade_id', '!=', False),
            ('website_published', '=', True),
            ('grade_id.website_published', '=', True),
            ('grade_id.active', '=', True),
        ]
        grades = env['res.partner'].sudo()._read_group(base_partner_domain, groupby=['grade_id'])
        for [grade] in grades:
            loc = '/partners/grade/%s' % slug(grade)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}
        country_partner_domain = base_partner_domain + [('country_id', '!=', False)]
        countries = env['res.partner'].sudo()._read_group(country_partner_domain, groupby=['country_id'])
        for [country] in countries:
            loc = '/partners/country/%s' % slug(country)
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
    ], type='http', auth="public", website=True, sitemap=sitemap_partners, readonly=True)
    def partners(self, country=None, grade=None, page=0, **post):
        country_all = post.pop('country_all', False)
        partner_obj = request.env['res.partner']
        country_obj = request.env['res.country']
        search = post.get('search', '')

        base_partner_domain = [('is_company', '=', True), ('grade_id', '!=', False), ('website_published', '=', True), ('grade_id.active', '=', True)]
        if not request.env.user.has_group('website.group_website_restricted_editor'):
            base_partner_domain += [('grade_id.website_published', '=', True)]
        if search:
            base_partner_domain += ['|', ('name', 'ilike', search), ('website_description', 'ilike', search)]

        # Infer Country
        if not country and not country_all:
            if request.geoip.country_code:
                country = country_obj.search([('code', '=', request.geoip.country_code)], limit=1)

        # Group by country
        country_domain = list(base_partner_domain)
        if grade:
            country_domain += [('grade_id', '=', grade.id)]
        countries = partner_obj.sudo().read_group(
            country_domain + [('country_id', '!=', False)],
            ["id", "country_id"],
            groupby="country_id", orderby="country_id")

        # Fallback on all countries if no partners found for the country and
        # there are matching partners for other countries.
        country_ids = [c['country_id'][0] for c in countries]
        fallback_all_countries = country and country.id not in country_ids
        if fallback_all_countries:
            country = None

        # Group by grade
        grade_domain = list(base_partner_domain)
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
        slug = request.env['ir.http']._slug
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
            base_partner_domain, order="grade_sequence ASC, implemented_partner_count DESC, complete_name ASC, id ASC",
            offset=pager['offset'], limit=self._references_per_page)
        partners = partner_ids.sudo()

        google_maps_api_key = request.website.google_maps_api_key

        values = {
            'countries': countries,
            'country_all': country_all,
            'current_country': country,
            'grades': grades,
            'current_grade': grade,
            'partners': partners,
            'pager': pager,
            'searches': post,
            'search_path': "%s" % werkzeug.urls.url_encode(post),
            'google_maps_api_key': google_maps_api_key,
            'fallback_all_countries': fallback_all_countries,
        }
        return request.render("website_crm_partner_assign.index", values, status=partners and 200 or 404)


    # Do not use semantic controller due to sudo()
    @http.route()
    def partners_detail(self, partner_id, **post):
        current_slug = partner_id
        _, partner_id = request.env['ir.http']._unslug(partner_id)
        current_grade, current_country = None, None
        grade_id = post.get('grade_id')
        country_id = post.get('country_id')
        if grade_id:
            current_grade = request.env['res.partner.grade'].browse(int(grade_id)).exists()
        if country_id:
            current_country = request.env['res.country'].browse(int(country_id)).exists()
        if partner_id:
            partner = request.env['res.partner'].sudo().browse(partner_id)
            is_website_restricted_editor = request.env.user.has_group('website.group_website_restricted_editor')
            if partner.exists() and (partner.website_published or is_website_restricted_editor):
                partner_slug = request.env['ir.http']._slug(partner)
                if partner_slug != current_slug:
                    return request.redirect('/partners/%s' % partner_slug)
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

## File: data\crm_lead_merge_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="crm_lead_merge_summary_inherit_partner_assign" inherit_id="crm.crm_lead_merge_summary">
    <xpath expr="//div[@name='company']" position="after">
        <br t-if="lead.date_partner_assign or lead.partner_assigned_id" />
        <div t-if="lead.partner_assigned_id">
            Assigned Partner: <span t-field="lead.partner_assigned_id"/>
        </div>
        <div name="date_partner_assign" t-if="lead.date_partner_assign">
            Partner Assignment Date: <span t-field="lead.date_partner_assign"/>
        </div>
    </xpath>
    <xpath expr="//div[@name='address']" position="attributes">
        <attribute name="t-if">
            lead.street or lead.street2 or lead.zip or lead.city or lead.state_id or lead.country_id
            or lead.partner_latitude or lead.partner_longitude
        </attribute>
    </xpath>
    <xpath expr="//div[@name='address']/*[last()]" position="after">
        <div name="partner_geolocation" t-if="lead.partner_latitude or lead.partner_longitude">
            Geolocation: <t t-esc="lead.partner_latitude"/> latitude, <t t-esc="lead.partner_longitude"/> longitude
        </div>
    </xpath>
</template>

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
            <field name="description">Sent to partner when a lead has been assigned to him</field>
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
                </td><td valign="middle" align="right" t-if="not user.company_id.uses_default_logo">
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
                                Please connect to your <a t-att-href="'%s?db=%s' % (object.get_base_url(), object.env.cr.dbname)">Partner Portal</a> to get details. On each lead are two buttons on the top left corner that you should press after having contacted the lead: "I'm interested" &amp; "I'm not interested".<br/>
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

## File: data\res_partner_activation_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_partner_activation_data_fully_operational" model="res.partner.activation">
        <field name="name">Fully Operational</field>
        <field name="sequence">1</field>
    </record>
    <record id="res_partner_activation_data_ramp_up" model="res.partner.activation">
        <field name="name">Ramp-up</field>
        <field name="sequence">2</field>
    </record>
    <record id="res_partner_activation_data_first_contact" model="res.partner.activation">
        <field name="name">First Contact</field>
        <field name="sequence">3</field>
    </record>
</odoo>

```

## File: data\res_partner_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

        <record id="base.res_partner_3" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_data_silver"/>
            <field name="partner_weight">10</field>
            <field name="partner_latitude">38.2809274</field>
            <field name="partner_longitude">-121.9505130</field>
            <field name="assigned_partner_id" ref="base.res_partner_3"/>
        </record>

       <record id="base.res_partner_4" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_data_bronze"/>
            <field name="partner_weight">10</field>
            <field name="partner_latitude">37.6952527</field>
            <field name="partner_longitude">-121.3966916</field>
            <field name="assigned_partner_id" ref="base.res_partner_3"/>
        </record>

        <record id="base.res_partner_12" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_data_silver"/>
            <field name="partner_weight">10</field>
            <field name="partner_latitude">37.5304271</field>
            <field name="partner_longitude">-121.9746713</field>
            <field name="assigned_partner_id" ref="base.res_partner_12"/>
        </record>

        <record id="base.res_partner_10" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_data_gold"/>
            <field name="partner_weight">10</field>
            <field name="partner_latitude">37.7019178</field>
            <field name="partner_longitude">-121.4452084</field>
            <field name="assigned_partner_id" ref="base.res_partner_12"/>
        </record>
        <record id="base.res_partner_18" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_data_gold"/>
            <field name="partner_weight">10</field>
            <field name="partner_latitude">37.9663420</field>
            <field name="partner_longitude">-121.2908748</field>
            <field name="assigned_partner_id" ref="base.res_partner_1"/>
        </record>
        <record id="base.res_partner_1" model="res.partner">
            <field name="grade_id" ref="res_partner_grade_data_gold"/>
            <field name="partner_weight">10</field>
            <field name="partner_latitude">37.5055960</field>
            <field name="partner_longitude">-120.8279940</field>
            <field name="assigned_partner_id" ref="base.res_partner_2"/>
        </record>
</odoo>

```

## File: data\res_partner_grade_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_partner_grade_data_gold" model="res.partner.grade">
        <field name="name">Gold</field>
        <field name="sequence">1</field>
    </record>
    <record id="res_partner_grade_data_silver" model="res.partner.grade">
        <field name="name">Silver</field>
        <field name="sequence">2</field>
    </record>
    <record id="res_partner_grade_data_bronze" model="res.partner.grade">
        <field name="name">Bronze</field>
        <field name="sequence">3</field>
    </record>
</odoo>

```

## File: data\res_partner_grade_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record id="website_crm_partner_assign.res_partner_grade_data_gold" model="res.partner.grade">
            <field name="is_published" eval="True" />
        </record>
        <record id="website_crm_partner_assign.res_partner_grade_data_silver" model="res.partner.grade">
            <field name="is_published" eval="True" />
        </record>
        <record id="website_crm_partner_assign.res_partner_grade_data_bronze" model="res.partner.grade">
            <field name="is_published" eval="True" />
        </record>
    </data>
</odoo>

```

## File: models\crm_lead.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import random
from markupsafe import Markup

from odoo import api, fields, models, _
from odoo.exceptions import AccessDenied, AccessError, UserError


class CrmLead(models.Model):
    _inherit = "crm.lead"

    partner_latitude = fields.Float('Geo Latitude', digits=(10, 7))
    partner_longitude = fields.Float('Geo Longitude', digits=(10, 7))
    partner_assigned_id = fields.Many2one('res.partner', 'Assigned Partner', tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", help="Partner this case has been forwarded/assigned to.", index='btree_not_null')
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
        """ While assigning a partner, geo-localization is performed only for leads having country
            set (see method 'assign_geo_localize' and 'search_geo_partner'). So for leads that does not
            have country set, we show the notification, and for the rest, we geo-localize them.
        """
        leads_with_country = self.filtered(lambda lead: lead.country_id)
        leads_without_country = self - leads_with_country
        if leads_without_country:
            self.env.user._bus_send('simple_notification', {
                'type': 'danger',
                'title': _("Warning"),
                'message': _('There is no country set in addresses for %(lead_names)s.', lead_names=', '.join(leads_without_country.mapped('name'))),
            })
        return leads_with_country.assign_partner(partner_id=False)

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
                lead._handle_salesmen_assignment(user_ids=partner.user_id.ids)
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
        message = Markup('<p>%s</p>') % _('I am interested by this lead.')
        if comment:
            message += Markup('<p>%s</p>') % comment
        for lead in self:
            lead.message_post(body=message)
            lead.sudo().convert_opportunity(lead.partner_id)  # sudo required to convert partner data

    def partner_desinterested(self, comment=False, contacted=False, spam=False):
        if contacted:
            message = Markup('<p>%s</p>') % _('I am not interested by this lead. I contacted the lead.')
        else:
            message = Markup('<p>%s</p>') % _('I am not interested by this lead. I have not contacted the lead.')
        partner_ids = self.env['res.partner'].search(
            [('id', 'child_of', self.env.user.partner_id.commercial_partner_id.id)])
        self.message_unsubscribe(partner_ids=partner_ids.ids)
        if comment:
            message += Markup('<p>%s</p>') % comment
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
        self.browse().check_access('write')
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
        self.browse().check_access('write')
        fields = ['partner_name', 'phone', 'mobile', 'email_from', 'street', 'street2',
            'city', 'zip', 'state_id', 'country_id']
        if any([key not in fields for key in values]):
            raise UserError(_("Not allowed to update the following field(s): %s.", ", ".join([key for key in values if not key in fields])))
        return self.sudo().write(values)

    @api.model
    def create_opp_portal(self, values):
        if not (self.env.user.partner_id.grade_id or self.env.user.commercial_partner_id.grade_id):
            raise AccessDenied()
        user = self.env.user
        self = self.sudo()
        if not (values['contact_name'] and values['description'] and values['title']):
            return {
                'errors': _('All fields are required!')
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
        lead.convert_opportunity(lead.partner_id)
        return {
            'id': lead.id
        }

    #
    #   DO NOT FORWARD PORT IN MASTER
    #   instead, crm.lead should implement portal.mixin
    #
    def _get_access_action(self, access_uid=None, force_website=False):
        """ Instead of the classic form view, redirect to the online document for
        portal users or if force_website=True. """
        self.ensure_one()

        user, record = self.env.user, self
        if access_uid:
            try:
                record.check_access("read")
            except AccessError:
                return super(CrmLead, self)._get_access_action(access_uid=access_uid, force_website=force_website)
            user = self.env['res.users'].sudo().browse(access_uid)
            record = self.with_user(user)
        if user.share or force_website:
            try:
                record.check_access('read')
            except AccessError:
                pass
            else:
                return {
                    'type': 'ir.actions.act_url',
                    'url': '/my/opportunity/%s' % record.id,
                }
        return super(CrmLead, self)._get_access_action(access_uid=access_uid, force_website=force_website)

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = "res.partner"

    @api.model
    def default_get(self, fields_list):
        default_vals = super().default_get(fields_list)
        if self.env.context.get('partner_set_default_grade_activation'):
            # sets the lowest grade and activation if no default values given, mainly useful while
            # creating assigned partner on the fly (to make it visible in same m2o again)
            if 'grade_id' in fields_list and not default_vals.get('grade_id'):
                default_vals['grade_id'] = self.env['res.partner.grade'].search([], order='sequence', limit=1).id
            if 'activation' in fields_list and not default_vals.get('activation'):
                default_vals['activation'] = self.env['res.partner.activation'].search([], order='sequence', limit=1).id
        return default_vals

    partner_weight = fields.Integer(
        'Level Weight', compute='_compute_partner_weight',
        readonly=False, store=True, tracking=True,
        help="This should be a numerical value greater than 0 which will decide the contention for this partner to take this lead/opportunity.")
    grade_id = fields.Many2one('res.partner.grade', 'Partner Level', tracking=True)
    grade_sequence = fields.Integer(related='grade_id.sequence', readonly=True, store=True)
    activation = fields.Many2one('res.partner.activation', 'Activation', index='btree_not_null', tracking=True)
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
    implemented_partner_count = fields.Integer(compute='_compute_implemented_partner_count', store=True)

    @api.depends('implemented_partner_ids.is_published', 'implemented_partner_ids.active')
    def _compute_implemented_partner_count(self):
        rg_result = self.env['res.partner']._read_group(
            [('assigned_partner_id', 'in', self.ids),
             ('is_published', '=', True)],
            ['assigned_partner_id'],
            ['__count'],
        )
        rg_data = {assigned_partner.id: count for assigned_partner, count in rg_result}
        for partner in self:
            partner.implemented_partner_count = rg_data.get(partner.id, 0)

    @api.depends('grade_id.partner_weight')
    def _compute_partner_weight(self):
        for partner in self:
            partner.partner_weight = partner.grade_id.partner_weight if partner.grade_id else 0

    def _compute_opportunity_count(self):
        super()._compute_opportunity_count()
        if not self.env.user.has_group('sales_team.group_sale_salesman'):
            return

        opportunity_data = self.env['crm.lead'].with_context(active_test=False)._read_group(
            [('partner_assigned_id', 'in', self.ids)],
            ['partner_assigned_id'], ['__count']
        )
        assign_counts = {partner_assigned.id: count for partner_assigned, count in opportunity_data}
        for partner in self:
            partner.opportunity_count += assign_counts.get(partner.id, 0)

    def action_view_opportunity(self):
        self.ensure_one()  # especially here as we are doing an id, in, IDS domain
        action = super().action_view_opportunity()
        action_domain_origin = action.get('domain')
        action_context_origin = action.get('context') or {}
        action_domain_assign = [('partner_assigned_id', '=', self.id)]
        if not action_domain_origin:
            action['domain'] = action_domain_assign
            return action
        # perform searches independently as having OR with those leaves seems to
        # be counter productive
        Lead = self.env['crm.lead'].with_context(**action_context_origin, active_test=False)
        ids_origin = Lead.search(action_domain_origin).ids
        ids_new = Lead.search(action_domain_assign).ids
        action['domain'] = [('id', 'in', sorted(list(set(ids_origin) | set(ids_new))))]
        return action

```

## File: models\res_partner_activation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartnerActivation(models.Model):
    _name = 'res.partner.activation'
    _order = 'sequence'
    _description = 'Partner Activation'

    sequence = fields.Integer('Sequence')
    name = fields.Char('Name', required=True)
    active = fields.Boolean(default=True)

```

## File: models\res_partner_grade.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


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
            grade.website_url = "/partners/grade/%s" % (self.env['ir.http']._slug(grade))

    def _default_is_published(self):
        return True

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class Website(models.Model):
    _inherit = "website"

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Resellers'), self.env['ir.http']._url_for('/partners'), 'website_crm_partner_assign'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import res_partner
from . import res_partner_activation
from . import res_partner_grade
from . import website

```

## File: report\crm_partner_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools import SQL


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
    nbr_opportunities = fields.Integer('# of Opportunity', readonly=True)
    turnover = fields.Float('Turnover', readonly=True)
    date = fields.Date('Invoice Account Date', readonly=True)

    _depends = {
        'account.invoice.report': ['invoice_date', 'partner_id', 'price_subtotal', 'state', 'move_type'],
        'crm.lead': ['partner_assigned_id'],
        'res.partner': ['activation', 'country_id', 'date_partnership', 'date_review',
                        'grade_id', 'parent_id', 'user_id'],
    }

    @property
    def _table_query(self):
        """
            CRM Lead Report
            @param cr: the current row, from the database cursor
        """
        return SQL(
            """
                SELECT
                    COALESCE(2 * i.id, 2 * p.id + 1) AS id,
                    p.id as partner_id,
                    (SELECT country_id FROM res_partner a WHERE a.parent_id=p.id AND country_id is not null limit 1) as country_id,
                    p.grade_id,
                    p.activation,
                    p.date_review,
                    p.date_partnership,
                    p.user_id,
                    (SELECT count(id) FROM crm_lead WHERE partner_assigned_id=p.id) AS nbr_opportunities,
                    i.price_subtotal as turnover,
                    i.invoice_date as date
                FROM
                    res_partner p
                    left join (%(account_invoice_report)s) i
                        on (i.partner_id=p.id and i.move_type in ('out_invoice','out_refund') and i.state='posted')
            """,
            account_invoice_report=self.env['account.invoice.report']._table_query,
        )

```

## File: report\crm_partner_report_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!--     Opportunity list view  -->
        <record id="view_report_crm_partner_assign_filter" model="ir.ui.view">
            <field name="name">crm.partner.report.assign.select</field>
            <field name="model">crm.partner.report.assign</field>
            <field name="arch" type="xml">
                <search string="Partner assigned Analysis">
                    <field name="user_id"/>
                    <field name="grade_id"/>
                    <field name="activation"/>
                    <filter name="filter_date_partnership" date="date_partnership"/>
                    <filter name="filter_date_review" date="date_review"/>
                    <group  expand="1" string="Group By">
                        <filter string="Salesperson" name="user"
                            context="{'group_by':'user_id'}" />
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
            <field name="context">{'group_by':[]}</field>
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
partner_access_crm_stage,crm.stage,crm.model_crm_stage,base.group_portal,1,0,0,0
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M15.724 6.397C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397L37.236 13H41.9c2.46 0 4.367 2.099 4.07 4.481l-3.106 25C42.613 44.49 40.866 46 38.793 46H11.207c-2.074 0-3.82-1.51-4.07-3.519l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603ZM32.917 13H17.082c0-.56.123-1.134.39-1.691l.956-2C19.102 7.9 20.551 7 22.144 7h5.711c1.593 0 3.042.9 3.716 2.308l.957 2c.266.558.39 1.132.39 1.692Z" fill="#088BF5"/><path fill-rule="evenodd" clip-rule="evenodd" d="M8.514 45.016a3.963 3.963 0 0 1-1.377-2.535l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397l2.59 5.777C35.5 28.256 23.848 41.405 8.515 45.016ZM17.082 13h15.835c0-.56-.123-1.134-.39-1.691l-.956-2C30.897 7.9 29.448 7 27.855 7h-5.711c-1.593 0-3.042.9-3.716 2.308l-.956 2a3.904 3.904 0 0 0-.39 1.692Z" fill="#2EBCFA"/><path d="M33.689 27.924c-.776 2.774-5.49 7.604-9.144 11.031-2.055 1.927-5.403 1.068-6.177-1.585-1.376-4.719-2.937-11.16-2.161-13.933 1.295-4.63 6.258-7.379 11.085-6.14 4.828 1.24 7.692 5.997 6.397 10.627Z" fill="#fff"/></svg>

```

## File: static\src\img\leads.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M48.0478 12.9588C40.1693 8.21713 27.1254 8.08518 18.9135 12.664C12.0258 16.5048 10.725 22.4396 15.1335 27.0518C20.5263 32.6935 25.9189 38.3351 31.3117 43.9769C30.8403 43.4837 30.9793 42.849 31.716 42.4383C32.5941 41.9485 33.9891 41.9627 34.8317 42.4698C35.5147 42.8809 35.625 43.4897 35.1745 43.9664C40.5342 38.2952 45.8938 32.6245 51.2535 26.9534C55.4662 22.4958 54.4345 16.8027 48.0478 12.9588Z" fill="#FBDBD0"/>
<path d="M47.4442 29.8355C39.2324 34.4144 26.1885 34.2825 18.31 29.5406C17.0396 28.7761 15.9811 27.9384 15.1335 27.0517C20.4397 32.6029 25.7457 38.1539 31.0518 43.7051C31.1612 46.2939 31.2705 48.8825 31.3799 51.4714C31.391 51.7342 31.5651 51.9952 31.9041 52.1994C32.625 52.6332 33.8183 52.6452 34.5696 52.2263C34.945 52.017 35.1391 51.7396 35.1492 51.4604C35.2439 48.871 35.3386 46.2821 35.4332 43.6927C40.7067 38.1127 45.9801 32.5333 51.2535 26.9533C50.2694 27.9946 48.9992 28.9683 47.4442 29.8355Z" fill="#C1DBF6"/>
<path d="M48.0478 12.9588C55.9263 17.7005 55.6561 25.2564 47.4442 29.8354C39.2323 34.4144 26.1885 34.2824 18.3099 29.5407C10.4314 24.7989 10.7016 17.243 18.9135 12.6641C27.1254 8.0851 40.1692 8.21705 48.0478 12.9588ZM46.5026 13.8204C39.4428 9.5714 27.7544 9.45319 20.3959 13.5563C13.0375 17.6594 12.7953 24.4301 19.8551 28.6791C26.9149 32.9281 38.6033 33.0463 45.9618 28.9432C53.3202 24.8401 53.5624 18.0694 46.5026 13.8204Z" fill="white"/>
<path d="M26.413 17.2737C23.5893 17.2737 21.2922 14.9763 21.2922 12.1527C21.2922 9.32902 23.5893 7.03192 26.413 7.03192C29.2366 7.03192 31.5337 9.32902 31.5337 12.1527C31.5337 14.9763 29.2366 17.2737 26.413 17.2737Z" fill="white"/>
<path d="M26.413 7.26049C29.115 7.26049 31.3053 9.45089 31.3053 12.1528C31.3053 14.8547 29.115 17.0451 26.413 17.0451C23.7111 17.0451 21.5207 14.8547 21.5207 12.1528C21.5207 9.45089 23.7111 7.26049 26.413 7.26049ZM26.413 6.80334C23.4633 6.80334 21.0636 9.20312 21.0636 12.1528C21.0636 15.1025 23.4633 17.5022 26.413 17.5022C29.3627 17.5022 31.7624 15.1025 31.7624 12.1528C31.7624 9.20312 29.3627 6.80334 26.413 6.80334Z" fill="#374874"/>
<path d="M39.6005 28.0411C36.7768 28.0411 34.4795 25.744 34.4795 22.9204C34.4795 20.0967 36.7768 17.7994 39.6005 17.7994C42.4241 17.7994 44.7212 20.0967 44.7212 22.9204C44.7212 25.744 42.4241 28.0411 39.6005 28.0411Z" fill="white"/>
<path d="M39.6004 18.028C42.3023 18.028 44.4927 20.2183 44.4927 22.9203C44.4927 25.6223 42.3023 27.8125 39.6004 27.8125C36.8985 27.8125 34.7081 25.6222 34.7081 22.9203C34.7081 20.2184 36.8985 18.028 39.6004 18.028ZM39.6004 17.5709C36.6507 17.5709 34.251 19.9706 34.251 22.9203C34.251 25.87 36.6507 28.2696 39.6004 28.2696C42.5501 28.2696 44.9498 25.87 44.9498 22.9203C44.9498 19.9706 42.5501 17.5709 39.6004 17.5709Z" fill="#374874"/>
<path d="M26.413 7.26052C28.1044 7.26052 29.5951 8.1191 30.4737 9.42401C31.3942 9.35213 32.3206 9.3162 33.2471 9.3162C38.64 9.3162 44.0193 10.5342 48.0478 12.9588C48.0496 12.9598 48.0512 12.9609 48.053 12.9621C48.277 13.097 48.4928 13.2347 48.7036 13.3741C48.7715 13.4189 48.8366 13.4647 48.9031 13.51C49.0469 13.6079 49.1883 13.7067 49.3257 13.8066C49.3976 13.8588 49.4682 13.9114 49.5383 13.9641C49.6693 14.0627 49.7967 14.1623 49.9216 14.2626C49.9844 14.3131 50.0481 14.3633 50.1094 14.4143C50.2584 14.538 50.4021 14.6631 50.542 14.7893C50.5719 14.8162 50.6038 14.8427 50.6333 14.8698C50.8002 15.0231 50.9597 15.1783 51.1133 15.3349C51.1573 15.3799 51.1984 15.4254 51.2413 15.4706C51.3495 15.5843 51.4552 15.6986 51.5564 15.8139C51.6064 15.8707 51.6547 15.9281 51.7031 15.9855C51.7939 16.0932 51.8817 16.2015 51.9665 16.3103C52.011 16.3675 52.0558 16.4245 52.0987 16.482C52.1899 16.6042 52.2763 16.7273 52.3601 16.8509C52.389 16.8935 52.42 16.9357 52.4481 16.9785C52.5566 17.1442 52.6587 17.3108 52.7539 17.4785C52.7756 17.5167 52.7944 17.5554 52.8154 17.5937C52.8872 17.7245 52.9559 17.8559 53.0197 17.9879C53.048 18.0465 53.0742 18.1055 53.101 18.1643C53.1532 18.2791 53.2027 18.3943 53.2489 18.5098C53.2733 18.5707 53.2974 18.6319 53.3201 18.6931C53.3654 18.8152 53.4065 18.9375 53.4451 19.0601C53.4614 19.1117 53.4794 19.1632 53.4944 19.2147C53.5449 19.3879 53.5899 19.5614 53.6271 19.7354C53.6314 19.7557 53.634 19.776 53.6381 19.7963C53.6698 19.9504 53.6962 20.105 53.7174 20.2597C53.7252 20.3169 53.7305 20.3741 53.7369 20.4313C53.7504 20.5512 53.7613 20.6712 53.7685 20.7913C53.7723 20.8538 53.7752 20.9163 53.7773 20.979C53.7813 21.0993 53.7815 21.2196 53.7793 21.3401C53.7782 21.3969 53.7783 21.4538 53.7759 21.5106C53.769 21.6694 53.757 21.828 53.7392 21.9866C53.7374 22.0032 53.7369 22.0197 53.735 22.0363C53.7141 22.2111 53.6854 22.3856 53.6513 22.56C53.6411 22.6124 53.628 22.6646 53.6166 22.717C53.5896 22.8402 53.5603 22.9631 53.5268 23.0859C53.51 23.1474 53.4919 23.2087 53.4734 23.2701C53.4381 23.3874 53.3996 23.5044 53.3582 23.6211C53.3376 23.6796 53.3175 23.738 53.2953 23.7962C53.2434 23.9326 53.1865 24.0684 53.1262 24.2038C53.1103 24.2396 53.0964 24.2758 53.0799 24.3116C53.0011 24.482 52.9155 24.6516 52.8236 24.8202C52.8 24.8633 52.7736 24.906 52.7493 24.9491C52.6779 25.0749 52.6038 25.2003 52.5251 25.3251C52.4887 25.3827 52.4506 25.4399 52.4126 25.4973C52.3384 25.6095 52.2613 25.721 52.181 25.8321C52.1397 25.8894 52.0984 25.9465 52.0554 26.0036C51.9649 26.1235 51.8699 26.2426 51.7723 26.3612C51.7363 26.4049 51.7022 26.4491 51.6652 26.4926C51.5336 26.6475 51.3973 26.8013 51.2535 26.9535C45.9801 32.5332 40.7067 38.1127 35.4333 43.6927C35.3386 46.2821 35.2439 48.8711 35.1493 51.4605C35.1391 51.7396 34.9451 52.0171 34.5696 52.2263C34.2025 52.431 33.73 52.5328 33.2584 52.5328C32.765 52.5328 32.2728 52.4213 31.9042 52.1994C31.5651 51.9953 31.391 51.7343 31.3799 51.4714C31.2706 48.8825 31.1612 46.294 31.0519 43.7051C25.7458 38.1539 20.4397 32.6029 15.1336 27.0519C14.843 26.7479 14.5789 26.4378 14.3379 26.1233C14.2978 26.0709 14.2608 26.018 14.222 25.9652C14.1381 25.8514 14.0561 25.7371 13.9786 25.6219C13.9368 25.5597 13.897 25.4974 13.8571 25.435C13.7876 25.3264 13.7208 25.2172 13.657 25.1077C13.62 25.0443 13.5833 24.981 13.5483 24.9173C13.4853 24.8029 13.4265 24.6878 13.3696 24.5725C13.3413 24.5149 13.3113 24.4576 13.2845 24.3999C13.2147 24.2499 13.1506 24.0993 13.0911 23.9482C13.0823 23.9257 13.0716 23.9033 13.063 23.8809C12.9964 23.707 12.9376 23.5322 12.8846 23.357C12.8689 23.3051 12.8565 23.2528 12.842 23.2008C12.8074 23.0769 12.7749 22.9529 12.7471 22.8286C12.7327 22.7639 12.7205 22.6993 12.7079 22.6346C12.6857 22.5208 12.666 22.4067 12.6495 22.2925C12.6398 22.2259 12.6308 22.1594 12.6231 22.0927C12.6096 21.9762 12.5999 21.8597 12.5923 21.7431C12.5883 21.681 12.5831 21.6191 12.5807 21.5571C12.5753 21.4178 12.5754 21.2783 12.5784 21.1391C12.5793 21.101 12.5777 21.0631 12.5792 21.025C12.5859 20.848 12.5998 20.671 12.62 20.4942C12.6254 20.4473 12.6341 20.4008 12.6404 20.354C12.658 20.2236 12.6778 20.0934 12.7027 19.9635C12.7149 19.8998 12.7297 19.8365 12.7437 19.7729C12.7688 19.6587 12.7961 19.5446 12.8269 19.4308C12.8449 19.3643 12.8639 19.2978 12.8839 19.2312C12.918 19.1176 12.9557 19.0042 12.9955 18.8911C13.0176 18.8283 13.0389 18.7656 13.0627 18.703C13.1117 18.5745 13.1657 18.4466 13.222 18.3189C13.2421 18.2734 13.2598 18.2278 13.2808 18.1824C13.3605 18.0103 13.4466 17.8389 13.5397 17.6687C13.5599 17.6318 13.5831 17.5953 13.6039 17.5585C13.6792 17.4251 13.7576 17.2923 13.8413 17.1601C13.8787 17.1011 13.9189 17.0426 13.958 16.9838C14.0312 16.8737 14.1064 16.7639 14.1855 16.6548C14.2309 16.5922 14.2776 16.53 14.3249 16.4676C14.4059 16.3611 14.49 16.2552 14.5765 16.1497C14.6257 16.0899 14.6744 16.03 14.7254 15.9705C14.8235 15.856 14.9264 15.7426 15.0312 15.6295C15.0754 15.5818 15.1175 15.5338 15.1629 15.4862C15.3153 15.3272 15.4735 15.1696 15.6393 15.0138C15.6618 14.9927 15.6864 14.9721 15.7092 14.9509C15.8542 14.8167 16.0037 14.6836 16.1587 14.5521C16.2187 14.5012 16.2816 14.4512 16.3431 14.4007C16.4665 14.2995 16.5917 14.1988 16.7211 14.0993C16.7932 14.0438 16.867 13.989 16.941 13.9341C17.0687 13.8394 17.1993 13.7456 17.3326 13.6526C17.4088 13.5996 17.4848 13.5465 17.5628 13.4939C17.7088 13.3956 17.8593 13.2988 18.0117 13.2028C18.081 13.1589 18.1485 13.1146 18.2191 13.0713C18.4444 12.9333 18.6749 12.7972 18.9136 12.664C19.7556 12.1945 20.6507 11.7774 21.5835 11.4068C21.9436 9.0599 23.9654 7.26052 26.413 7.26052ZM26.413 6.3463C23.7077 6.3463 21.4189 8.16682 20.7839 10.7457C19.9606 11.091 19.1836 11.4667 18.4683 11.8655C18.236 11.9951 17.9984 12.1345 17.7417 12.2916C17.6901 12.3232 17.6403 12.355 17.5904 12.3871L17.5229 12.4301C17.3396 12.5457 17.1897 12.643 17.0521 12.7356C16.9778 12.7857 16.9055 12.8361 16.8333 12.8863L16.8103 12.9022C16.6691 13.0008 16.5313 13.0998 16.3966 13.1996C16.318 13.2579 16.24 13.3159 16.1638 13.3744C16.0392 13.4704 15.9119 13.5719 15.7631 13.6941L15.7056 13.7408C15.6589 13.7788 15.6123 13.8166 15.5672 13.855C15.4078 13.9903 15.2466 14.1332 15.088 14.2802L15.0703 14.296C15.0512 14.3128 15.0323 14.3297 15.0142 14.3466C14.8416 14.5088 14.6746 14.6743 14.5028 14.8537C14.4681 14.89 14.436 14.9251 14.404 14.9605L14.36 15.0087C14.2295 15.1494 14.1247 15.2663 14.031 15.3757C13.9846 15.4298 13.9397 15.4845 13.8948 15.5392L13.8701 15.5693C13.7713 15.6897 13.6821 15.8024 13.597 15.9144C13.5452 15.9827 13.4946 16.0502 13.4453 16.1182C13.3671 16.226 13.2859 16.3435 13.1968 16.4773L13.1634 16.5272C13.1314 16.575 13.0995 16.6227 13.0691 16.6706C12.9836 16.8055 12.8982 16.9488 12.8078 17.1089L12.7905 17.1383C12.7723 17.1689 12.7542 17.1994 12.7374 17.2302C12.637 17.4137 12.5406 17.605 12.4511 17.7983C12.4341 17.8351 12.4183 17.8722 12.4028 17.9093L12.3854 17.9503C12.314 18.1122 12.2576 18.248 12.2083 18.3775C12.1869 18.4338 12.1673 18.4895 12.1478 18.5455L12.1331 18.5876C12.0852 18.7235 12.0444 18.8479 12.0083 18.9679C11.9858 19.0429 11.9646 19.1173 11.9444 19.1917C11.9117 19.3127 11.8811 19.4384 11.8507 19.5766L11.841 19.6204C11.8283 19.6774 11.8158 19.7343 11.8048 19.7914C11.7793 19.9241 11.7569 20.0641 11.7343 20.2317L11.7283 20.2713C11.7221 20.311 11.7162 20.3507 11.7117 20.3904C11.6886 20.592 11.6731 20.7939 11.6656 20.9901C11.6643 21.025 11.6642 21.0595 11.6644 21.0941V21.1194C11.6606 21.2922 11.6615 21.4469 11.6671 21.5922C11.6692 21.6459 11.6728 21.6995 11.6766 21.7532L11.68 21.8026C11.6894 21.9469 11.7008 22.0763 11.7149 22.1978C11.7236 22.2731 11.7338 22.3482 11.7446 22.4235C11.7625 22.5467 11.7847 22.6766 11.8106 22.8096L11.816 22.8377C11.8283 22.901 11.8407 22.9644 11.8548 23.0278C11.8829 23.1535 11.9167 23.2864 11.9613 23.4465L11.9742 23.4945C11.9854 23.5372 11.9968 23.5798 12.0097 23.6223C12.072 23.8283 12.1373 24.02 12.2091 24.2077C12.2187 24.2325 12.2286 24.2562 12.2388 24.2797C12.3078 24.4545 12.3802 24.6236 12.4555 24.7854C12.4773 24.8323 12.5005 24.8788 12.5239 24.9252L12.5497 24.9769C12.6218 25.123 12.6846 25.2442 12.7476 25.3585C12.7858 25.428 12.8262 25.4979 12.8669 25.5679C12.9373 25.6885 13.0113 25.8095 13.0872 25.9282C13.132 25.9982 13.1749 26.0651 13.2197 26.1318C13.2963 26.2457 13.3809 26.365 13.4862 26.5078L13.5216 26.5565C13.5512 26.5975 13.5808 26.6384 13.612 26.6791C13.8787 27.0271 14.1681 27.365 14.4727 27.6836L30.153 44.0879L30.2323 45.965L30.4665 51.5101C30.4911 52.0911 30.8432 52.6279 31.4327 52.9827C31.9303 53.2823 32.5787 53.4471 33.2585 53.4471C33.9028 53.4471 34.5266 53.2972 35.0149 53.0249C35.6585 52.6662 36.0406 52.1082 36.063 51.494L36.2379 46.7103L36.3344 44.0704L46.566 33.2444L51.9181 27.5815C52.0642 27.4269 52.2095 27.2643 52.3622 27.0844C52.3897 27.0521 52.4164 27.0191 52.4429 26.9859L52.4782 26.9423C52.5933 26.8025 52.6938 26.6756 52.7853 26.5542C52.8321 26.4922 52.8766 26.4307 52.9211 26.3691C53.01 26.246 53.095 26.1231 53.1752 26.0018L53.1902 25.9792C53.2268 25.9239 53.2633 25.8686 53.2983 25.8132C53.3785 25.6861 53.459 25.551 53.5446 25.4001L53.5689 25.3584C53.5885 25.3251 53.6079 25.2917 53.6261 25.2583C53.7289 25.0696 53.8243 24.8804 53.9099 24.695C53.924 24.6647 53.9369 24.6341 53.9497 24.6036L53.9615 24.5758C54.0331 24.4147 54.0947 24.2663 54.1498 24.1216C54.1699 24.069 54.1888 24.0154 54.2077 23.9617L54.2204 23.9257C54.2674 23.7928 54.3108 23.6606 54.3491 23.5334C54.3698 23.4643 54.3901 23.3956 54.4089 23.3267C54.4439 23.1983 54.477 23.0626 54.5099 22.9119L54.5205 22.8648C54.5305 22.8217 54.5402 22.7786 54.5487 22.7355C54.5898 22.525 54.6206 22.3317 54.6429 22.1444C54.6455 22.1222 54.6475 22.1001 54.6491 22.0779C54.6681 21.906 54.6817 21.7281 54.6894 21.5501C54.6915 21.5017 54.6922 21.4532 54.6928 21.4049L54.6935 21.3571C54.6962 21.2115 54.6954 21.0779 54.6912 20.9487C54.6888 20.8778 54.6855 20.807 54.6812 20.7364C54.6735 20.6071 54.6618 20.474 54.6456 20.3293L54.6405 20.2823C54.6353 20.2333 54.63 20.1846 54.6233 20.1356C54.5994 19.9613 54.5693 19.7853 54.5339 19.6126C54.5308 19.5927 54.5265 19.5688 54.5214 19.5448C54.4814 19.3575 54.4326 19.1661 54.3723 18.959C54.3599 18.9166 54.3463 18.8746 54.3326 18.8328L54.3172 18.7852C54.2704 18.6364 54.2246 18.5022 54.1774 18.375C54.1519 18.3063 54.1249 18.2379 54.0976 18.1697C54.0479 18.0453 53.9925 17.9158 53.9334 17.7858L53.9166 17.7489C53.8925 17.6956 53.8683 17.6426 53.8427 17.5896C53.7765 17.4524 53.7025 17.3097 53.617 17.1538L53.5997 17.1211C53.5832 17.0895 53.5665 17.0579 53.5487 17.0265C53.4458 16.8453 53.3327 16.6603 53.2129 16.4775C53.1915 16.4447 53.1683 16.4113 53.1449 16.378L53.1168 16.3376C53.0166 16.1898 52.9232 16.0581 52.8315 15.9352C52.7911 15.881 52.7491 15.827 52.707 15.7729L52.688 15.7486C52.5942 15.6284 52.4982 15.5099 52.402 15.3959C52.3497 15.3339 52.297 15.2715 52.2425 15.2095C52.1431 15.0963 52.032 14.9752 51.904 14.8406L51.8612 14.7949C51.8301 14.7615 51.799 14.7283 51.7665 14.6951C51.5963 14.5215 51.4233 14.354 51.252 14.1966C51.2291 14.1755 51.2053 14.1547 51.1813 14.134L51.1534 14.1095C50.9978 13.969 50.847 13.8383 50.6936 13.7109C50.6437 13.6695 50.5917 13.6277 50.5396 13.5862L50.494 13.5497C50.355 13.438 50.2222 13.3345 50.0885 13.2339C50.0138 13.1777 49.9393 13.1223 49.8635 13.0671C49.7264 12.9675 49.5806 12.8651 49.4176 12.7543L49.3546 12.711C49.3058 12.6774 49.257 12.6438 49.2068 12.6108C48.9692 12.4536 48.748 12.3135 48.5319 12.1832C48.5278 12.1806 48.5235 12.178 48.5193 12.1755C44.4762 9.74212 39.0524 8.40193 33.2471 8.40193C32.4684 8.40193 31.6837 8.42705 30.9076 8.47661C29.8092 7.13364 28.166 6.3463 26.413 6.3463Z" fill="#374874"/>
</svg>

```

## File: static\src\img\quotation.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M48.2304 3.28046C49.0383 2.81404 49.6978 3.18771 49.7004 4.11811L49.8128 43.786C49.8155 44.7164 49.1603 45.8512 48.3525 46.3176L21.368 61.8971C20.5583 62.3646 19.8988 61.9888 19.8962 61.0584L19.7837 21.3905C19.7811 20.4601 20.4362 19.3275 21.2459 18.86L48.2304 3.28046Z" fill="white"/>
<path d="M47.7615 6.3496L47.8677 43.7991L21.831 58.7543L21.7331 21.3906L47.7615 6.3496ZM47.7615 5.89246C47.6825 5.89246 47.6036 5.91289 47.5328 5.95385L21.5044 20.9948C21.3627 21.0766 21.2756 21.2281 21.276 21.3918L21.3739 58.7555C21.3743 58.9185 21.4614 59.0688 21.6025 59.1503C21.6732 59.191 21.7521 59.2114 21.831 59.2114C21.9096 59.2114 21.9882 59.1912 22.0587 59.1507L48.0954 44.1956C48.2377 44.1138 48.3253 43.962 48.3249 43.7978L48.2187 6.3483C48.2182 6.18523 48.1309 6.03467 47.9895 5.95332C47.9189 5.91282 47.8402 5.89246 47.7615 5.89246Z" fill="#374874"/>
<path d="M20.3243 61.999L18.2975 60.8211C18.0342 60.668 17.8707 60.3427 17.8694 59.8805L19.8962 61.0584C19.8975 61.5206 20.0609 61.8459 20.3243 61.999Z" fill="#FBDBD0"/>
<path d="M47.2462 2.00083L49.2731 3.17868C49.0065 3.02373 48.6373 3.0456 48.2305 3.28046L46.2037 2.10261C46.6105 1.86775 46.9796 1.8459 47.2462 2.00083Z" fill="#FBDBD0"/>
<path d="M19.8962 61.0584L17.8694 59.8805L17.757 20.2127L19.7838 21.3905L19.8962 61.0584Z" fill="#FBDBD0"/>
<path d="M21.246 18.86L19.2192 17.6821L46.2037 2.10261L48.2305 3.28046L21.246 18.86Z" fill="#FBDBD0"/>
<path d="M19.7838 21.3905L17.757 20.2127C17.7543 19.2823 18.4095 18.1496 19.2192 17.6821L21.246 18.86C20.4363 19.3274 19.7811 20.4601 19.7838 21.3905Z" fill="#FBDBD0"/>
<path d="M37.2801 6.11085L35.2533 4.933C35.6885 4.68176 36.0829 4.65849 36.3677 4.824L38.3945 6.00185C38.1097 5.83634 37.7153 5.85961 37.2801 6.11085Z" fill="#C1DBF6"/>
<path d="M30.604 13.7733L28.5772 12.5955L28.5715 10.5933L30.5983 11.7712L30.604 13.7733Z" fill="#C1DBF6"/>
<path d="M32.1594 9.06731L30.1326 7.88946L35.2533 4.933L37.2801 6.11085L32.1594 9.06731Z" fill="#C1DBF6"/>
<path d="M30.5983 11.7712L28.5715 10.5933C28.5687 9.59961 29.2678 8.38873 30.1326 7.88946L32.1594 9.06731C31.2946 9.5666 30.5955 10.7775 30.5983 11.7712Z" fill="#C1DBF6"/>
<path d="M37.2802 6.11087C38.145 5.61158 38.8486 6.01258 38.8515 7.00627L38.8571 9.00844L39.8945 8.40952C40.7556 7.91235 41.4574 8.31228 41.4602 9.30182L41.4646 10.8263C41.465 10.9925 41.3765 11.1464 41.2325 11.2295L28.71 18.4593C28.4012 18.6377 28.0149 18.4154 28.0139 18.0588L28.0111 17.0667C28.0083 16.0771 28.7056 14.8695 29.5667 14.3723L30.6041 13.7734L30.5984 11.7712C30.5956 10.7775 31.2947 9.56662 32.1594 9.06735L37.2802 6.11087Z" fill="#C1DBF6"/>
<path d="M46.8679 1.899C47.0022 1.899 47.121 1.92947 47.2182 1.98684C47.2182 1.98684 49.27 3.17736 49.2701 3.17736C49.5346 3.32947 49.6991 3.65459 49.7004 4.11809L49.8129 43.786C49.8156 44.7164 49.1604 45.8511 48.3525 46.3177L21.368 61.8971C21.1321 62.0333 20.9091 62.0977 20.7113 62.0977C20.5657 62.0977 20.4337 62.0628 20.3203 61.9958L18.2975 60.8212C18.0341 60.6681 17.8707 60.3428 17.8694 59.8806L17.7569 20.2126C17.7543 19.2823 18.4095 18.1497 19.2192 17.6822L28.5764 12.2797L28.5716 10.5935C28.5688 9.59969 29.2679 8.38876 30.1327 7.88954L35.2534 4.93306C35.5058 4.7873 35.7444 4.71832 35.956 4.71832C36.1092 4.71832 36.2481 4.75459 36.3678 4.82401L38.3943 6.00179C38.3936 6.00134 38.3928 6.00113 38.3921 6.00079C38.542 6.08705 38.6605 6.2268 38.7401 6.41174L46.2037 2.10259C46.4448 1.96342 46.6726 1.899 46.8679 1.899ZM38.3943 6.00179L38.3946 6.00193L38.3943 6.00179ZM46.868 0.984741C46.5039 0.984741 46.1161 1.09747 45.7466 1.31075L38.9199 5.25217C38.907 5.2438 38.8939 5.23564 38.8807 5.22762C38.8718 5.22204 38.8628 5.21658 38.8537 5.21132L36.8272 4.03354C36.5687 3.88344 36.2675 3.80408 35.9561 3.80408C35.5741 3.80408 35.1839 3.91747 34.7963 4.14124L29.6755 7.09784C28.5212 7.76412 27.6536 9.26803 27.6573 10.5961L27.6606 11.7527L18.762 16.8905C17.6642 17.5243 16.8391 18.9536 16.8427 20.2152L16.9551 59.8832C16.9573 60.6568 17.2791 61.2868 17.838 61.6117C17.8384 61.6119 19.7688 62.7328 19.8572 62.7841C20.1103 62.9332 20.4057 63.012 20.7113 63.012C21.0791 63.012 21.4537 62.9033 21.8251 62.689L48.8096 47.1095C49.9064 46.4762 50.7308 45.0462 50.7272 43.7834L50.6147 4.11555C50.6125 3.34033 50.2895 2.7102 49.7283 2.3862C49.7052 2.37281 47.677 1.19603 47.677 1.19603C47.4452 1.0591 47.1634 0.984741 46.868 0.984741Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M39.9177 24.3392C39.9178 24.3392 39.9179 24.3393 39.8187 24.5437L39.9179 24.3393C40.0315 24.3944 40.0788 24.5312 40.0237 24.6447C39.9686 24.7581 39.8322 24.8055 39.7187 24.7507C39.7186 24.7506 39.7184 24.7506 39.7182 24.7505L39.8181 24.5449C39.7182 24.7505 39.7181 24.7504 39.7182 24.7505L39.7165 24.7497C39.7142 24.7486 39.7102 24.7468 39.7045 24.7442C39.693 24.7392 39.6749 24.7314 39.6503 24.7216C39.6011 24.7021 39.5263 24.6746 39.4282 24.6451C39.2319 24.5859 38.9429 24.5185 38.5798 24.4893C37.8552 24.4309 36.8305 24.5237 35.6499 25.1434C34.4535 25.7715 32.9548 26.9463 31.9507 28.3042C30.9413 29.6695 30.4785 31.1486 31.1882 32.4419C31.6049 33.2012 32.2139 33.49 32.9519 33.5488C33.7115 33.6093 34.5889 33.4243 35.5125 33.229L35.5183 33.2277C36.4214 33.0367 37.3696 32.8362 38.2121 32.9079C39.0777 32.9815 39.8464 33.3438 40.3598 34.2709C40.9369 35.3131 40.9157 36.4169 40.4902 37.4453C40.0672 38.4676 39.2457 39.4166 38.2138 40.1836C36.1556 41.7135 33.1879 42.5703 30.6747 41.7892C30.5541 41.7518 30.4868 41.6237 30.5242 41.5031C30.5617 41.3826 30.6898 41.3152 30.8104 41.3527C33.1467 42.0788 35.9645 41.2859 37.9411 39.8168C38.9265 39.0843 39.6847 38.1964 40.0678 37.2706C40.4483 36.3508 40.4597 35.3951 39.9599 34.4924C39.5333 33.7221 38.9159 33.4265 38.1734 33.3634C37.4096 33.2984 36.53 33.481 35.607 33.6762L35.5703 33.684C34.6772 33.8729 33.7435 34.0704 32.9156 34.0045C32.055 33.9359 31.2919 33.5811 30.7874 32.6619C29.9465 31.1294 30.5386 29.4452 31.5832 28.0324C32.6331 26.6124 34.1877 25.3947 35.4374 24.7387C36.7027 24.0744 37.8153 23.9691 38.6165 24.0336C39.0162 24.0658 39.3372 24.1402 39.5601 24.2074C39.6716 24.241 39.7588 24.2728 39.8192 24.2968C39.8494 24.3088 39.8729 24.3189 39.8894 24.3262C39.8977 24.3299 39.9042 24.3328 39.909 24.3351L39.9148 24.3378L39.9167 24.3387L39.9174 24.339L39.9177 24.3392Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M35.5181 22.5457C35.6443 22.5457 35.7466 22.648 35.7466 22.7742V43.5597C35.7466 43.6859 35.6443 43.7883 35.5181 43.7883C35.3918 43.7883 35.2895 43.6859 35.2895 43.5597V22.7742C35.2895 22.648 35.3918 22.5457 35.5181 22.5457Z" fill="#374874"/>
<path d="M49.5169 33.14C53.8497 30.6384 57.3723 32.6475 57.3864 37.6246C57.4005 42.6017 53.9008 48.6651 49.568 51.1667C45.2366 53.6674 41.714 51.6599 41.6999 46.6812C41.6857 41.7025 45.1855 35.6407 49.5169 33.14Z" fill="#374874"/>
<path d="M54.0353 35.226L54.7045 35.6559L47.1187 48.2591L44.3662 46.6905L45.0513 45.5022L47.1348 46.6911L54.0353 35.226Z" fill="white"/>
</svg>

```

## File: static\src\js\crm_partner_assign.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";
import { parseDate, formatDate, serializeDate } from "@web/core/l10n/dates";
const { DateTime } = luxon;

publicWidget.registry.crmPartnerAssign = publicWidget.Widget.extend({
    selector: '#wrapwrap',
    selectorHas: '.interested_partner_assign_form, .desinterested_partner_assign_form, .opp-stage-button, .new_opp_form',
    events: {
        'click .interested_partner_assign_confirm': '_onInterestedPartnerAssignConfirm',
        'click .desinterested_partner_assign_confirm': '_onDesinterestedPartnerAssignConfirm',
        'click .opp-stage-button': '_onOppStageButtonClick',
        'change .edit_contact_form .country_id': '_onEditContactFormChange',
        'click .edit_contact_confirm': '_onEditContactConfirm',
        'click .new_opp_confirm': '_onNewOppConfirm',
        'click .edit_opp_confirm': '_onEditOppConfirm',
        'change .edit_opp_form .next_activity': '_onChangeNextActivity',
        'change #new-opp-dialog .contact_name': '_onChangeContactName',
    },

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Element} btnEl
     * @param {function} callback
     * @returns {Promise}
     */
    _buttonExec: function (btnEl, callback) {
        // TODO remove once the automatic system which does this lands in master
        btnEl.setAttribute("disabled", true);
        return callback.call(this).catch(function (e) {
            btnEl.removeAttribute("disabled");
            if (e instanceof Error) {
                return Promise.reject(e);
            }
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _confirmInterestedPartner: function () {
        return this.orm.call("crm.lead", "partner_interested", [
            [parseInt(document.querySelector(".interested_partner_assign_form .assign_lead_id").value)],
            document.querySelector(".interested_partner_assign_form .comment_interested").value
        ]).then(function () {
            window.location.href = '/my/leads';
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _confirmDesinterestedPartner: function () {
        return this.orm.call("crm.lead", "partner_desinterested", [
            [parseInt(document.querySelector(".desinterested_partner_assign_form .assign_lead_id").value)],
            document.querySelector(".desinterested_partner_assign_form .comment_desinterested").value,
            document.querySelector(".desinterested_partner_assign_form .contacted_desinterested").checked,
            document.querySelector(".desinterested_partner_assign_form .customer_mark_spam").checked,
        ]).then(function () {
            window.location.href = '/my/leads';
        });
    },
    /**
     * @private
     * @param {}
     * @returns {Promise}
     */
    _changeOppStage: function (leadID, stageID) {
        return this.orm.write("crm.lead", [leadID], { stage_id: stageID }, {
            context: Object.assign({website_partner_assign: 1}),
        }).then(function () {
            window.location.reload();
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _editContact: function () {
        return this.orm.call("crm.lead", "update_contact_details_from_portal", [
            [parseInt(document.querySelector(".edit_contact_form .opportunity_id").value)],
            {
                partner_name: document.querySelector(".edit_contact_form .partner_name").value,
                phone: document.querySelector(".edit_contact_form .phone").value,
                mobile: document.querySelector(".edit_contact_form .mobile").value,
                email_from: document.querySelector(".edit_contact_form .email_from").value,
                street: document.querySelector(".edit_contact_form .street").value,
                street2: document.querySelector(".edit_contact_form .street2").value,
                city: document.querySelector(".edit_contact_form .city").value,
                zip: document.querySelector(".edit_contact_form .zip").value,
                state_id: parseInt(document.querySelector(".edit_contact_form .state_id").selectedOptions[0].value),
                country_id: parseInt(document.querySelector(".edit_contact_form .country_id").selectedOptions[0].value),
            },
        ]).then(function () {
            window.location.reload();
        });
    },
    /**
     * @private
     * @returns {Promise}
     */
    _createOpportunity: function () {
        return this.orm.call("crm.lead", "create_opp_portal", [{
            contact_name: document.querySelector(".new_opp_form .contact_name").value,
            title: document.querySelector(".new_opp_form .title").value,
            description: document.querySelector(".new_opp_form .description").value,
        }]).then(function (response) {
            if (response.errors) {
                document.querySelector("#new-opp-dialog .alert")?.remove();
                const alertEl = document.createElement("div");
                alertEl.classList.add("alert", "alert-danger");
                alertEl.textContent = response.errors;
                const parentEl = document.querySelector("#new-opp-dialog");
                parentEl.insertBefore(alertEl, parentEl.firstElementChild);
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
        return this.orm.call("crm.lead", "update_lead_portal", [
            [parseInt(document.querySelector(".edit_opp_form .opportunity_id").value)],
            {
                date_deadline: this._parse_date(document.querySelector(".edit_opp_form .date_deadline").value),
                expected_revenue: parseFloat(document.querySelector(".edit_opp_form .expected_revenue").value),
                probability: parseFloat(document.querySelector(".edit_opp_form .probability").value),
                activity_type_id: parseInt(document.querySelector(".edit_opp_form .next_activity").selectedOptions[0].getAttribute("data")),
                activity_summary: document.querySelector(".edit_opp_form .activity_summary").value,
                activity_date_deadline: this._parse_date(document.querySelector(".edit_opp_form .activity_date_deadline").value),
                priority: document.querySelector("input[name='PriorityRadioOptions']:checked").value,
            },
        ]).then(function () {
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
        if (document.querySelector(".interested_partner_assign_form .comment_interested").value && document.querySelector(".interested_partner_assign_form .contacted_interested").checked) {
            this._buttonExec(ev.currentTarget, this._confirmInterestedPartner);
        } else {
            document.querySelector(".interested_partner_assign_form .error_partner_assign_interested").style.display = "block";
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onDesinterestedPartnerAssignConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec(ev.currentTarget, this._confirmDesinterestedPartner);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onOppStageButtonClick: function (ev) {
        const btnEl = ev.currentTarget;
        this._buttonExec(
            btnEl,
            this._changeOppStage.bind(this, parseInt(btnEl.getAttribute("opp")), parseInt(btnEl.getAttribute("data")))
        );
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onEditContactFormChange: function (ev) {
        var countryID = document.querySelector(".edit_contact_form .country_id").selectedOptions[0].value;
        document.querySelectorAll(".edit_contact_form .state").forEach(state => {
            state.style.display = state.getAttribute("country") != countryID ? "none" : "block";
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onEditContactConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec(ev.currentTarget, this._editContact);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onNewOppConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec(ev.currentTarget, this._createOpportunity);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onEditOppConfirm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this._buttonExec(ev.currentTarget, this._editOpportunity);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeContactName: function (ev) {
        const contactName = ev.currentTarget.value.trim();
        let titleEl = this.el.querySelector('.title');
        if (!titleEl.value.trim()) {
            titleEl.value = contactName ? _t("%s's Opportunity", contactName) : '';
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeNextActivity: function (ev) {
        const selectedEl = document.querySelector(".edit_opp_form .next_activity").selectedOptions[0];
        if (selectedEl.getAttribute("activity_summary")) {
            document.querySelector(".edit_opp_form .activity_summary").value = selectedEl.getAttribute("activity_summary");
        }
        if (selectedEl.getAttribute("delay_count")) {
            const value = +selectedEl.getAttribute("delay_count");
            const unit = selectedEl.getAttribute("delay_unit");
            const date = DateTime.now().plus({ [unit]: value});
            document.querySelector(".edit_opp_form .activity_date_deadline").value = formatDate(date);
        }
    },
    _parse_date: function (value) {
        var date = parseDate(value);
        if (!date.isValid || date.year < 1900) {
            return false;
        }
        return serializeDate(date);
    },
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
                        <group>
                            <group col="2">
                                <label for="partner_latitude" string="Geolocation"/>
                                <div>
                                    <span class="oe_grey">( </span>
                                    <field name="partner_latitude" class="oe_inline o_input_12ch px-1"/>
                                    <span class="oe_grey" invisible="partner_latitude &lt;= 0">N </span>
                                    <span class="oe_grey" invisible="partner_latitude &gt;= 0">S </span>
                                    <field name="partner_longitude" class="oe_inline o_input_12ch ps-2 pe-1"/>
                                    <span class="oe_grey" invisible="partner_longitude &lt;= 0">E </span>
                                    <span class="oe_grey" invisible="partner_longitude &gt;= 0">W </span>
                                    <span class="oe_grey ps-1">) </span>
                                </div>
                                <field name="partner_assigned_id" domain="[('grade_id','!=',False)]" context="{'partner_set_default_grade_activation': 1}"/>
                            </group>
                            <group>
                                <button colspan="2" string="Automatic Assignment" name="action_assign_partner" type="object" class="btn-link pt-0 ps-0"/>
                                <button colspan="2" string="Send Email" type="action" class="btn-link pt-0 ps-0"
                                    invisible="not partner_assigned_id"
                                    name="%(crm_lead_forward_to_partner_act)d"
                                    context="{'default_composition_mode': 'forward','hide_forward_type': 1 , 'default_partner_ids': [partner_assigned_id]}"/>
                            </group>
                        </group>
                    </page>
                </xpath>
            </field>
        </record>

        <record id="view_crm_opportunity_geo_assign_tree" model="ir.ui.view">
            <field name="name">crm.lead.geo_assign.list.inherit</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_tree_view_oppor"/>
            <field name="arch" type="xml">
                <field name="priority" position="after">
                    <field name="partner_assigned_id" optional="hide"/>
                    <field name="date_partner_assign" column_invisible="True"/>
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
            <field name="name">crm.lead.lead.geo_assign.list.inherit</field>
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

        <record id="crm_opportunity_report_view_pivot_lead" model="ir.ui.view">
            <field name="name">crm.opportunity.report.view.pivot.lead.inherit.partner_assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_opportunity_report_view_pivot_lead"/>
            <field name="arch" type="xml">
                <xpath expr="//pivot" position="inside">
                    <field name="partner_latitude" invisible="1"/>
                    <field name="partner_longitude" invisible="1"/>
                </xpath>
            </field>
        </record>

        <record id="crm_lead_view_pivot_forecast" model="ir.ui.view">
            <field name="name">crm.lead.view.pivot.forecast.inherit.website.crm.partner.assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_lead_view_pivot_forecast"/>
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

        <record id="crm_lead_view_graph_forecast" model="ir.ui.view">
            <field name="name">crm.lead.view.graph.forecast.inherit.website.crm.partner.assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_lead_view_graph_forecast"/>
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

        <record id="crm_lead_view_kanban" model="ir.ui.view">
            <field name="name">crm.lead.view.kanban.inherit.website.crm.partner.assign</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_kanban_view_leads"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='partner_id']" position="after">
                    <span class="text-truncate ms-1" t-if="record.partner_assigned_id.value">
                        (<i class="fa fa-long-arrow-right me-1" aria-label="Assigned Partner" title="Assigned Partner"/>
                        <span t-att-title="record.partner_assigned_id.value"><field name="partner_assigned_id"/></span>)
                    </span>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\partner_assign_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem id="crm_menu_resellers"
              name="Resellers"
              parent="crm.crm_menu_config"
              sequence="16"/>

    <menuitem id="menu_res_partner_grade_action"
              action="res_partner_grade_action"
              parent="crm_menu_resellers"
              sequence="1"/>

    <menuitem id="res_partner_activation_config_mi"
              parent="crm_menu_resellers"
              action="res_partner_activation_act"
              sequence="2"/>

</odoo>

```

## File: views\res_partner_activation_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
        <field name="name">res.partner.activation.list</field>
        <field name="model">res.partner.activation</field>
        <field name="arch" type="xml">
            <list string="Activation" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="active" widget="boolean_toggle"/>
            </list>
        </field>
    </record>

    <record id="res_partner_activation_view_search" model="ir.ui.view">
        <field name="name">res.partner.activation.view.search</field>
        <field name="model">res.partner.activation</field>
        <field name="arch" type="xml">
            <search string="Activation">
                <field name="name" string="Partner Activation"/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="res_partner_activation_act">
        <field name="name">Partner Activations</field>
        <field name="res_model">res.partner.activation</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
           <p class="o_view_nocontent_smiling_face">
              Create a Partner Activation
           </p><p>
              Those are used to know where your Partners stand in your onboarding process.
           </p>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_grade_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_grade_tree" model="ir.ui.view">
        <field name="name">res.partner.grade.list</field>
        <field name="model">res.partner.grade</field>
        <field name="arch" type="xml">
            <list string="Partner Level">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </list>
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
        <field name="name">Partner Levels</field>
        <field name="res_model">res.partner.grade</field>
        <field name="search_view_id" ref="res_partner_grade_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Partner Level
            </p><p>
                Partner Levels allow you to rank your Partners based on their performances.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_res_partner_filter_assign_tree" model="ir.ui.view">
        <field name="name">res.partner.geo.inherit.list</field>
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
                <xpath expr="//page[@name='geo_location']/group" position="before">
                    <group name="group_partner_activation_review">
                        <group>
                            <separator string="Partner Activation" colspan="2"/>
                            <field name="grade_id" options="{'no_open': True}"/>
                            <field name="activation" options="{'no_open': True}"/>
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

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Crm Partner Assign Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(#oe_structure_website_crm_partner_assign_layout_1)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Partners Page">
            <we-checkbox string="Show Leads / Opps"
                         data-customize-website-views="website_crm_partner_assign.portal_my_home_lead"
                         data-no-preview="true"
                         data-reload="/"/>
            <t t-if="google_maps_api_key">
                <we-checkbox string="World Map"
                            data-customize-website-views="website_crm_partner_assign.ref_country"
                            data-no-preview="true"
                            data-reload="/"/>
            </t>
            <we-checkbox string="Address"
                         data-customize-website-views="website_crm_partner_assign.o_wcrm_partner_address"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

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
            <t t-set="editor_message">DROP BUILDING BLOCKS HERE TO MAKE THEM AVAILABLE ACROSS ALL RESELLERS</t>
            <div class="oe_structure oe_empty" id="oe_structure_website_crm_partner_assign_layout_1" t-att-data-editor-message="editor_message"/>
            <div class="container my-4">
                <t t-out="0" />
            </div>
            <div class="oe_structure oe_empty" id="oe_structure_website_crm_partner_assign_layout_2" t-att-data-editor-message="editor_message"/>
        </div>
    </t>
</template>

<template id="index" name="Find Resellers">
    <t t-call="website_crm_partner_assign.layout">
        <div class="o_wcrm_filters_top d-flex d-print-none align-items-center justify-content-end flex-wrap gap-2 w-100">
            <h4 class="my-0 me-auto pe-sm-4">
                Find a reseller <t t-foreach="countries" t-as="country"><span t-if="country['active'] and country['country_id'][0] != 0 and country['country_id']">in <t t-out="country['country_id'][1]"/></span></t>
            </h4>
            <div class="dropdown d-none d-lg-block">
                <a role="button" href="#" data-bs-toggle="dropdown" class="dropdown-toggle btn btn-light" aria-expanded="true" aria-label="Open categories dropdown">
                    <t t-foreach="grades" t-as="grade">
                        <span t-if="grade['active']" t-out="grade['grade_id'][1]"/>
                    </t>
                </a>
                <div class="dropdown-menu">
                    <t t-foreach="grades" t-as="grade">
                        <a t-attf-href="/partners#{ grade['grade_id'][0] and '/grade/%s' % slug(grade['grade_id']) or '' }#{ current_country and '/country/%s' % slug(current_country) or '' }#{ '?' + (search_path or '') + '&amp;' + keep_query('country_all') }"
                        class="dropdown-item" t-out="grade['grade_id'][1]"/>
                    </t>
                </div>
            </div>
            <div class="dropdown d-none d-lg-block">
                <a role="button" href="#" data-bs-toggle="dropdown" class="dropdown-toggle btn btn-light" aria-expanded="true" aria-label="Open countries dropdown">
                    <t t-foreach="countries" t-as="country">
                        <span t-if="country['active']" t-out="country['country_id'][1]"/>
                    </t>
                </a>
                <div class="dropdown-menu">
                    <t t-foreach="countries" t-as="country" t-if="country['country_id']">
                        <a t-attf-href="/partners#{ current_grade and '/grade/%s' % slug(current_grade) or ''}#{country['country_id'][0] and '/country/%s' % slug(country['country_id']) or '' }#{ '?' + (search_path or '') + (country['country_id'][0] == 0 and '&amp;country_all=True' or '')}"
                        class="dropdown-item" t-out="country['country_id'][1]"/>
                    </t>
                </div>
            </div>
            <div class="o_wcrm_search d-flex w-100 w-lg-auto">
                <form class="flex-grow-1" action="" method="get">
                    <input t-if="country_all" type="hidden" name="country_all" value="True" />
                    <div class="input-group" role="search">
                        <input type="text" name="search" class="search-query form-control border-0 bg-light" placeholder="Search" t-att-value="searches.get('search', '')"/>
                        <button type="submit" aria-label="Search" title="Search" class="oe_search_button btn btn-light">
                            <i class="oi oi-search"/>
                        </button>
                    </div>
                </form>
                <button class="btn btn-light position-relative ms-2 d-lg-none"
                    data-bs-toggle="offcanvas"
                    data-bs-target="#o_wcrm_offcanvas">
                    <i class="fa fa-sliders"/>
                </button>
            </div>
        </div>
        <!-- Off canvas filters on mobile-->
        <div id="o_wcrm_offcanvas" class="o_website_offcanvas offcanvas offcanvas-end d-lg-none p-0 overflow-visible">
            <div class="offcanvas-header">
                <h5 class="offcanvas-title">Filters</h5>
                <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"/>
            </div>
            <div class="offcanvas-body p-0">
                <div class="accordion accordion-flush">
                    <div class="accordion-item">
                        <h2 class="accordion-header">
                            <button class="accordion-button collapsed"
                                type="button"
                                data-bs-toggle="collapse"
                                data-bs-target=".o_wcrm_offcanvas_grade"
                                aria-expanded="false"
                                aria-controls="o_wcrm_offcanvas_grade">
                                Filter by category
                            </button>
                        </h2>
                        <div class="o_wcrm_offcanvas_grade accordion-collapse collapse">
                            <div class="accordion-body pt-0">
                                <ul class="list-group list-group-flush">
                                    <t t-foreach="grades" t-as="grade">
                                        <li class="list-group-item d-flex justify-content-between align-items-center ps-0 pb-0 border-0">
                                            <a t-attf-href="/partners#{ grade['grade_id'][0] and '/grade/%s' % slug(grade['grade_id']) or '' }#{ current_country and '/country/%s' % slug(current_country) or '' }#{ '?' + (search_path or '') + '&amp;' + keep_query('country_all') }"
                                            class="text-reset" aria-label="See categories filters">
                                                <div class="form-check">
                                                    <input class="form-check-input pe-none" type="radio" t-attf-name="#{grade['grade_id'][1]}" t-att-checked="bool(grade['active'])"/>
                                                    <label class="form-check-label" t-attf-for="#{grade['grade_id'][1]}" t-out="grade['grade_id'][1]"/>
                                                </div>
                                            </a>
                                        </li>
                                    </t>
                                </ul>
                            </div>
                        </div>
                    </div>
                    <div class="accordion-item">
                        <h2 class="accordion-header">
                            <button class="accordion-button border-top collapsed"
                                type="button"
                                data-bs-toggle="collapse"
                                data-bs-target=".o_wcrm_offcanvas_country"
                                aria-expanded="false"
                                aria-controls="o_wcrm_offcanvas_country">
                                Filter by country
                            </button>
                        </h2>
                        <div class="o_wcrm_offcanvas_country accordion-collapse collapse">
                            <div class="accordion-body pt-0">
                                <ul class="list-group list-group-flush">
                                    <t t-foreach="countries" t-as="country">
                                        <li t-if="country['country_id']" class="list-group-item d-flex justify-content-between align-items-center ps-0 pb-0 border-0">
                                            <a t-attf-href="/partners#{ current_grade and '/grade/%s' % slug(current_grade) or ''}#{country['country_id'][0] and '/country/%s' % slug(country['country_id']) or '' }#{ '?' + (search_path or '') + (country['country_id'][0] == 0 and '&amp;country_all=True' or '')}"
                                            class="text-reset" aria-label="See countries filters">
                                                <div class="form-check">
                                                    <input class="form-check-input pe-none" type="radio" t-attf-name="#{country['country_id'][1]}" t-att-checked="bool(country['active'])"/>
                                                    <label class="form-check-label" t-attf-for="#{country['country_id'][1]}" t-out="country['country_id'][1]"/>
                                                </div>
                                            </a>
                                        </li>
                                    </t>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="row mb-4">
            <div class="my-5 py-5 text-center" t-if="not partners">
                <h5>No results found for "<span t-out="searches.get('search', '')"/>"</h5>
                <a href="/partners">See all resellers</a>
            </div>
            <div t-elif="fallback_all_countries" class="mt-4 alert alert-primary alert-dismissible fade show" role="alert">
                <i class="fa fa-info-circle me-2"/>
                There are no matching partners found for the selected country. Displaying results across all countries instead.
                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
            </div>
            <t t-set="last_grade" t-value="None"/>
            <t t-foreach="partners" t-as="partner">
                <t t-if="last_grade != partner.grade_id.id">
                    <h5 class="mt-4 mb-2 col-12">
                        <span t-field="partner.grade_id"/> Resellers
                        <t t-call="website.publish_management">
                            <t t-set="object" t-value="partner.grade_id"/>
                            <t t-set="publish_edit" t-value="True"/>
                        </t>
                    </h5>
                    <t t-set="last_grade" t-value="partner.grade_id.id"/>
                </t>
                <div class="col-md-4 col-xl-3 col-12 mb-4">
                    <div class="card h-100 text-decoration-none">
                        <a class="text-decoration-none" t-attf-href="/partners/#{slug(partner)}?#{current_grade and 'grade_id=%s&amp;' % current_grade.id}#{current_country and 'country_id=%s' % current_country.id}" aria-label="Go to reseller">
                            <div t-field="partner.image_1920"
                                class="card-img-top border-bottom"
                                t-options='{"widget": "image", "qweb_img_responsive": False, "class": "img img-fluid h-100 w-100 mw-100", "style": "max-height: 208px; min-height: 208px; object-fit: cover"}'
                                />
                            <div class="card-body">
                                <h5 class="card-title m-0" t-attf-href="/partners/#{slug(partner)}?#{current_grade and 'grade_id=%s&amp;' % current_grade.id}#{current_country and 'country_id=%s' % current_country.id}" t-field="partner.display_name"/>
                                <small id="o_wcrm_partners_address"/>
                                <small class="o_wcrm_short_description text-muted overflow-hidden">
                                    <b t-if="partner.implemented_partner_count">
                                        <t t-if="partner.implemented_partner_count > 1" t-set='reflabel'>references</t>
                                        <t t-else="" t-set='reflabel'>reference</t>
                                        <t t-out="partner.implemented_partner_count"/> <t t-out='reflabel'/> » </b>
                                    <span t-field="partner.website_short_description"/>
                                </small>
                                <small t-if="not partner.website_short_description" class="css_editable_mode_hidden text-muted fst-italic" groups="website.group_website_restricted_editor">
                                    Edit to add a short description
                                </small>
                            </div>
                        </a>
                    </div>
                </div>
            </t>
            <div class="navbar">
                <t t-call="website.pager">
                   <t t-set="classname" t-valuef="mx-auto"/>
                </t>
            </div>
        </div>
    </t>
</template>

<template id="ref_country" inherit_id="website_crm_partner_assign.index" name="World Map">
    <xpath expr="//div[hasclass('o_wcrm_search')]" position="inside">
        <t t-if="google_maps_api_key">
            <!-- modal for large map -->
            <div role="dialog" class="modal fade partner_map_modal" tabindex="-1">
              <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">World Map</h4>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"/>
                    </header>
                    <iframe loading="lazy" t-attf-src="/google_map?height=485&amp;dom=website_crm_partner_assign.partners&amp;current_grade=#{ current_grade and current_grade.id }&amp;current_country=#{ current_country and current_country.id }&amp;partner_url=/partners/&amp;limit=1000"
                    style="height:485px;"/>
                </div>
              </div>
            </div>
            <!-- modal end -->
            <div class="btn-group ms-2">
                <button class="btn btn-light border-primary active">
                    <i class="fa fa-th-large"/>
                </button>
                <button class="btn btn-light" data-bs-toggle="modal" data-bs-target=".partner_map_modal">
                    <i class="fa fa-map-marker" role="img" aria-label="Open map" title="Open map"/>
                </button>
            </div>
        </t>
    </xpath>
</template>

<template id="o_wcrm_partner_address" inherit_id="website_crm_partner_assign.index" name="Address">
    <xpath expr="//small[@id='o_wcrm_partners_address']" position="inside">
        <div t-field="partner.self" 
            t-options="{
                'widget': 'contact',
                'fields': ['country_id']
            }"
            class="py-2"
        />
    </xpath>
</template>

<template id="partner" name="Partner Detail">
    <t t-call="website_crm_partner_assign.layout">
        <div t-if="not edit_page" class="mb-3">
            <a t-attf-href="/partners#{current_grade and '/grade/%s' % slug(current_grade)}#{current_country and '/country/%s' % slug(current_country)}" aria-label="Back to resellers list"><i class="fa fa-chevron-left me-2"/>Back to resellers</a>
        </div>
        <t t-call="website_partner.partner_detail">
            <t t-set="right_column">
                <div id="right_column"><t t-call="website_crm_partner_assign.references_block"/></div>
            </t>
        </t>
    </t>
</template>

<template id="grade_in_detail" inherit_id="website_partner.partner_detail">
    <xpath expr="//*[@id='partner_name']" position="before">
        <span class="col-lg-12 text-muted" t-if="partner.grade_id and partner.grade_id.website_published">
            <span t-field="partner.grade_id"/>
        </span>
    </xpath>
</template>

<template id="references_block" name="Partner References Block">
    <t t-if="any(p.website_published for p in partner.implemented_partner_ids)">
        <h4 id="references">References</h4>
        <div t-foreach="partner.implemented_partner_ids" t-if="reference.website_published" t-as="reference" class="card mt-3 border-0">
            <div class="row">
                <div class="col-md-2">
                    <span t-field="reference.avatar_128" class="d-flex justify-content-center" t-options='{"widget": "image", "qweb_img_responsive": False, "class": "img-fluid rounded mw-100"}'/>
                </div>
                <div class="card-body col-md-10">
                    <span t-field="reference.self"/>
                    <div t-field='reference.website_short_description'/>
                </div>
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

    <template id="portal_my_home_lead" name="Leads / Opps" inherit_id="portal.portal_my_home" priority="15">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
            <t t-set="portal_vendor_category_enable" t-value="True"/>
        </xpath>
        <div id="portal_vendor_category" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/website_crm_partner_assign/static/src/img/leads.svg'"/>
                <t t-set="text">Follow and convert your leads</t>
                <t t-set="title">Leads</t>
                <t t-set="url" t-value="'/my/leads'"/>
                <t t-set="placeholder_count" t-value="'lead_count'"/>
            </t>
            <t t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/website_crm_partner_assign/static/src/img/quotation.svg'"/>
                <t t-set="text">Follow and convert your opportunities</t>
                <t t-set="title">Opportunities</t>
                <t t-set="url" t-value="'/my/opportunities'"/>
                <t t-set="placeholder_count" t-value="'opp_count'"/>
                <t t-set="config_card" t-value="request.env.user.partner_id.grade_id or request.env.user.commercial_partner_id.grade_id"/>
            </t>
        </div>
    </template>

    <template id="portal_my_leads" name="My Leads">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Leads</t>
            </t>
            <div t-if="not leads" class="alert alert-warning" role="alert">
                There are no leads.
            </div>
            <t t-if="leads" t-call="portal.portal_table">
                <thead>
                    <tr>
                        <th>Date</th>
                        <th class="w-25">Lead</th>
                        <th>Contact Name</th>
                        <th>Email</th>
                        <th>Phone</th>
                    </tr>
                </thead>
                <tbody>
                    <tr t-foreach="leads" t-as="lead">
                        <td><span t-field="lead.create_date" t-options='{"widget": "date"}' /></td>
                        <td class="text-wrap">
                            <a t-attf-href="/my/lead/#{lead.id}"><span t-field="lead.name"/></a>
                        </td>
                        <td><span t-field="lead.contact_name"/></td>
                        <td><span t-field="lead.email_from"/></td>
                        <td><span t-field="lead.phone"/></td>
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

                <div t-if="request.env.user.partner_id.grade_id or request.env.user.commercial_partner_id.grade_id">
                    <button class="btn btn-primary" name='new_opp' data-bs-toggle="modal" data-bs-target=".modal_new_opp" title="Add an opportunity" aria-label="Add an opportunity">
                        Create Opportunity
                    </button>
                </div>
            </t>

            <div class="modal fade modal_new_opp" role="form">
                <div class="modal-dialog">
                    <form method="POST" class="modal-content js_website_submit_form new_opp_form">
                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                        <header class="modal-header">
                            <h4 class="modal-title">New Opportunity</h4>
                            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                        </header>
                        <main class="modal-body" id="new-opp-dialog">
                            <div class="mb-3">
                                <label class="col-form-label hdd4" for="contact_name">Contact name</label>
                                <input type='text' name="contact_name" class="form-control contact_name"/>
                            </div>
                            <div class="mb-3">
                                <label class="col-form-label h4dd" for="title">Opportunity</label>
                                <input type='text' name="title" class="form-control title"/>
                            </div>
                            <div class="mb-3">
                                <label class="col-form-label hdd4" for="description">Description</label>
                                <textarea rows="3" name="description" class="form-control description"></textarea>
                            </div>
                        </main>
                        <footer class="modal-footer">
                            <button type="button" class="btn btn-light" data-bs-dismiss="modal">Cancel</button>
                            <button t-attf-class="btn btn-primary new_opp_confirm">Confirm</button>
                        </footer>
                    </form>
                </div>
            </div>
            <t t-if="not opportunities">
                <div class="alert alert-warning" role="alert">
                    There are no opportunities.
                </div>
            </t>
            <t t-if="opportunities" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>Date</th>
                        <th class="w-25">Opportunity</th>
                        <th>Contact</th>
                        <th>Email</th>
                        <th>Phone</th>
                        <th>Expected</th>
                        <th>Stage</th>
                    </tr>
                </thead>
                <tbody>
                    <tr t-foreach="opportunities" t-as="opp">
                        <td><span t-field="opp.create_date" t-options='{"widget": "date"}' /></td>
                        <td class="text-wrap">
                            <a t-attf-href="/my/opportunity/#{opp.id}"><span t-field="opp.name"/></a>
                        </td>
                        <td><span t-field="opp.contact_name"/></td>
                        <td><span t-field="opp.email_from"/></td>
                        <td><span t-field="opp.phone"/></td>
                        <td>
                            <span t-if="opp.company_currency" class="text-nowrap" t-esc="opp.expected_revenue" t-options="{'widget': 'monetary', 'display_currency': opp.company_currency}"/>
                            <span t-else="" class="text-nowrap" t-esc="opp.expected_revenue"/>
                            <span> at </span>
                            <span t-field="opp.probability" />%
                        </td>
                        <td>
                            <span class="badge text-bg-info" title="Current stage of the opportunity" t-esc="opp.stage_id.name" />
                        </td>
                    </tr>
                </tbody>
            </t>
        </t>
    </template>

    <template id="portal_my_lead" name="My Lead">
        <t t-call="portal.portal_layout">
            <div class="d-flex align-items-center gap-2 mb-4">
                <h4 class="mb-0">Lead -
                    <span t-field="lead.name"/>
                    <span title="Rating" role="img" t-attf-aria-label="Rating: #{lead.priority} on 3" class="fs-6">
                        <t t-foreach="range(1, 4)" t-as="i">
                            <span t-attf-class="fa fa-lg fa-star#{' text-warning' if i &lt;= int(lead.priority) else '-o'}"/>
                        </t>
                    </span>
                </h4>
            </div>
            <table class="table table-borderless">
                <tbody class="text-nowrap">
                    <tr t-if="lead.partner_name or lead.email_from or lead.partner_id">
                        <th class="ps-0 pb-0">Customer:</th>
                        <td>
                            <address>
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
                        </td>
                    </tr>
                    <tr t-if="lead.street or lead.street2 or lead.city or lead.state_id or lead.country_id">
                        <th class="ps-0 pb-0">Address:</th>
                        <td class="w-100 pb-0 text-wrap">
                            <address>
                                <div t-if="lead.street"><span t-field="lead.street"/></div>
                                <div t-if="lead.street2"><span t-field="lead.street2"/></div>
                                <div t-if="lead.city or lead.zip">
                                    <span t-field="lead.city"/> <span t-field="lead.zip"/>
                                </div>
                            </address>
                        </td>
                    </tr>
                    <tr t-if="lead.user_id">
                        <th class="ps-0 pb-0">Salesperson:</th>
                        <td class="w-100 pb-0 text-wrap">
                            <span t-field="lead.user_id"/>
                        </td>
                    </tr>
                    <tr t-if="lead.team_id">
                        <th class="ps-0 pb-0">Sales Team:</th>
                        <td class="w-100 pb-0 text-wrap">
                            <span t-field="lead.team_id"/>
                        </td>
                    </tr>
                    <tr t-if="lead.date_deadline">
                        <th class="ps-0 pb-0">Expected Closing:</th>
                        <td class="w-100 pb-0 text-wrap">
                            <span t-field="lead.date_deadline"/>
                        </td>
                    </tr>
                    <tr groups="!base.group_portal" t-if="lead.tag_ids">
                        <th class="ps-0 pb-0">Tags</th>
                        <td class="w-100 pb-0 text-wrap">
                             <t t-foreach="lead.tag_ids" t-as="tag">
                                <span class="badge text-bg-info" t-esc="tag.name" />
                            </t>
                        </td>
                    </tr>
                    <tr t-if="lead.partner_assigned_id">
                        <th class="ps-0 pb-0">Assigned Partner:</th>
                        <td class="w-100 pb-0 text-wrap">
                            <address t-field="lead.partner_assigned_id" t-options='{"widget": "contact", "fields": ["name", "email", "phone"], "no_marker": True}'/>
                        </td>
                    </tr>
                    <tr t-if="lead.campaign_id">
                        <th class="ps-0 pb-0">Campaign:</th>
                        <td class="w-100 pb-0 text-wrap">
                            <span t-field="lead.campaign_id"/>
                        </td>
                    </tr>
                    <tr t-if="lead.medium_id">
                        <th class="ps-0 pb-0">Medium:</th>
                        <td class="w-100 pb-0 text-wrap">
                            <span t-field="lead.medium_id"/>
                        </td>
                    </tr>
                </tbody>
            </table>
            <div class='d-flex justify-content-center gap-2 mt-3'>
                <a role="button" title="I'm interested" href="#" class="btn btn-primary" data-bs-toggle="modal" data-bs-target=".modal_partner_assign_interested">I'm interested</a>
                <a role="button" title="I'm not interested" href="#" class="btn btn-primary" data-bs-toggle="modal" data-bs-target=".modal_partner_assign_desinterested"> I'm not interested</a>
                <div class="modal fade modal_partner_assign_interested" role="form">
                    <div class="modal-dialog">
                        <form method="POST" class="js_accept_json modal-content js_website_submit_form interested_partner_assign_form">
                            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                            <input type="hidden" name="lead_id" class="assign_lead_id" t-att-value="lead.id"/>
                            <header class="modal-header">
                                <h4 class="modal-title">Lead Feedback</h4>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </header>
                            <main class="modal-body" id="sign-dialog">
                                <div class="mb-3">
                                    <label class="col-form-label" for="comment">What is the next action? When? What is the expected revenue?</label>
                                    <input type="text" name="comment" id="comment" class="form-control comment_interested"/>
                                </div>
                                <div class="mb-3">
                                    <label class="col-form-label" for="customer_contacted">I have contacted the customer</label>
                                    <input type="checkbox" name="customer_contacted" id="customer_contacted" class="contacted_interested"/>
                                </div>
                                <div>
                                    <span class="text-danger error_partner_assign_interested" style="display:none;">You need to fill up the next action and contact the customer before accepting the lead</span>
                                </div>
                            </main>
                            <footer class="modal-footer">
                                <button type="button" class="btn btn-light" data-bs-dismiss="modal">Cancel</button>
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
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </header>
                            <main class="modal-body" id="sign-dialog">
                                <div class="mb-3">
                                    <label class="col-form-label" for="comment">Why aren't you interested in this lead?</label>
                                    <input type="text" name="comment" id="comment" class="form-control comment_desinterested"/>
                                </div>
                                <div class="mb-3">
                                    <label class="col-form-label" for="contacted_desinterested">I have contacted the customer</label>
                                    <input type="checkbox" name="contacted_desinterested" id="contacted_desinterested" class="contacted_desinterested"/>
                                </div>
                                <div class="mb-3">
                                    <label class="col-form-label" for="customer_mark_spam">This lead is a spam</label>
                                    <input type="checkbox" name="customer_mark_spam" id="customer_mark_spam" class="customer_mark_spam"/>
                                </div>
                                <div>
                                    <span class="text-danger error_partner_assign_desinterested" style="display:none;">You need to fill up the next action and contact the customer before accepting the lead</span>
                                </div>
                            </main>
                            <footer class="modal-footer">
                                <button type="button" class="btn btn-light" data-bs-dismiss="modal">Cancel</button>
                                <button t-attf-class="btn btn-primary desinterested_partner_assign_confirm">Confirm</button>
                            </footer>
                        </form>
                    </div>
                </div>
            </div>
            <hr/>
            <div>
                <h3>Communication history</h3>
                <div>
                    <t t-call="portal.message_thread">
                        <t t-set="object" t-value="lead"/>
                    </t>
                </div>
            </div>
        </t>
    </template>

    <template id="portal_my_opportunity" name="My Opportunity">
        <t t-call="portal.portal_layout">
            <div class="d-flex justify-content-between flex-wrap mb-3">
                <h4 class="mb-2 mb-md-0">
                    Opportunity -
                    <span t-field="opportunity.name"/>
                </h4>
                <!-- todo: replace by design system wizard -->
                <div class="d-flex justify-content-between align-items-center gap-2">
                    <p class="mb-0 fw-bold">Stage:</p>
                    <div t-foreach="stages[::-1]" t-as="stage" class="d-flex justify-content-between align-items-center gap-2">
                        <i t-if="not stage_first" class="oi oi-chevron-right small" style="opacity: 0.5"/>
                        <button type="button" t-att-data="stage.id" t-att-opp="opportunity.id" t-attf-class="btn btn-sm px-2 opp-stage-button #{'btn-light' if opportunity.stage_id.name != stage.name else 'btn-primary disabled'}">
                            <span t-field="stage.name"/>
                        </button>
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="col-lg-5 mb-4 mb-lg-0">
                    <div class="border-bottom d-flex justify-content-between py-2 mb-3 align-items-center">
                        <h5 class="d-flex align-items-center justify-content-between gap-2 mb-0 ">
                            <span>
                                <t t-if="opportunity.company_currency" t-out="opportunity.expected_revenue"
                                    t-options="{'widget': 'monetary', 'display_currency': opportunity.company_currency}"/>
                                <t t-else="" t-out="opportunity.expected_revenue"/> at  </span>
                            <span class="badge text-bg-info"><span t-field="opportunity.probability"/>%</span>
                        </h5>
                        <button type="button" data-bs-toggle="modal" data-bs-target=".modal_edit_opp" class="btn btn-link btn-sm"><i class="fa fa-pencil me-1"/>Edit</button>
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
                        <button type="button" data-bs-toggle="modal" data-bs-target=".modal_edit_contact" class="btn btn-link btn-sm"><i class="fa fa-pencil me-1"/>Edit</button>
                    </div>
                    <div class="row mb-3" t-if="opportunity.partner_name or opportunity.email_from or opportunity.contact_name">
                        <strong class="col-12 col-sm-3">Customer</strong>
                        <div class="col">
                            <div class="d-flex justify-content-start align-items-center gap-2 mb-2" t-if="opportunity.partner_name or opportunity.contact_name">
                                <img t-if="opportunity.partner_id.sudo().avatar_1024" class="o_avatar o_portal_contact_img rounded"
                                        t-att-src="image_data_uri(opportunity.partner_id.sudo().avatar_512)"
                                alt="Contact" width="50"/>
                                <div class="d-flex flex-column justify-content-center">
                                    <h5 class="mb-0" t-if="opportunity.partner_name" t-out="opportunity.partner_name"/>
                                    <p class="mb-0 text-muted" t-else="" t-out="opportunity.contact_name"/>
                                </div>
                            </div>
                            <div>
                                <div t-field="opportunity.partner_id" t-options='{"widget": "contact", "fields": ["email", "phone", "mobile"]}'/>
                            </div>
                            <address t-if="opportunity.street or opportunity.street2 or opportunity.city or opportunity.zip or opportunity.state_id or opportunity.country_id"
                            class="col d-flex align-items-baseline mb-0 mt-2">
                                <div class="d-flex flex-nowrap gap-3">
                                    <div class="fa fa-map-marker fa-fw text-muted"></div>
                                    <div>
                                        <div t-if="opportunity.street"><span t-field="opportunity.street"/></div>
                                        <span t-else="" class="text-muted"> - </span>
                                        <div t-if="opportunity.street2"><span t-field="opportunity.street2"/></div>
                                        <div t-if="opportunity.city or opportunity.zip">
                                            <span t-field="opportunity.city"/> <span t-field="opportunity.zip"/>
                                        </div>
                                        <div t-if="opportunity.state_id or opportunity.country_id">
                                            <span t-field="opportunity.state_id"/> <span t-field="opportunity.country_id"/>
                                        </div>
                                    </div>
                                </div>
                            </address>
                        </div>
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
                                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                                </header>
                                <main class="modal-body" id="sign-dialog">
                                    <div class="mb-3">
                                        <div class="row align-items-center">
                                            <div class="col-auto flex-grow-1">
                                                <div class="input-group">
                                                    <div class="input-group-text"><span class="text-nowrap" t-esc="opportunity.company_currency.symbol"/></div>
                                                    <input type="text" name="expected_revenue" class="form-control expected_revenue" t-att-value="opportunity.expected_revenue" placeholder="Planned Revenue"/>
                                                </div>
                                            </div>
                                            <div class="col-auto">at</div>
                                            <div class="col-auto">
                                                <div class="input-group">
                                                    <input type="text" name="probability" class="form-control probability" t-att-value="opportunity.probability" placeholder="Probability"/>
                                                    <div class="input-group-text">%</div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="mb-3">
                                        <div class="row">
                                            <div class="col-md-5 pe-0">
                                                <label>Priority:</label>
                                                <div class="input-group">
                                                    <label class="radio-inline">
                                                        <input type="radio" name="PriorityRadioOptions" value="0" t-att-checked="opportunity.priority not in ['1','2','3']" aria-label="Rating: 0 on 3" title="Rating: 0 on 3"/>
                                                        <i class="ms-1 fa fa-star-o"></i>
                                                    </label>
                                                    <label class="radio-inline ms-2">
                                                        <input type="radio" name="PriorityRadioOptions" value="1" t-att-checked="opportunity.priority == '1'" aria-label="Rating: 1 on 3" title="Rating: 1 on 3"/>
                                                        <i class="ms-1 fa text-warning fa-star"></i>
                                                    </label>
                                                    <label class="radio-inline ms-2">
                                                        <input type="radio" name="PriorityRadioOptions" value="2" t-att-checked="opportunity.priority == '2'" aria-label="Rating: 2 on 3" title="Rating: 2 on 3"/>
                                                        <i class="ms-1 fa text-warning fa-star"></i>
                                                        <i class="ms-1 fa text-warning fa-star"></i>
                                                    </label>
                                                    <label class="radio-inline ms-2">
                                                        <input type="radio" name="PriorityRadioOptions" value="3" t-att-checked="opportunity.priority == '3'" aria-label="Rating: 3 on 3" title="Rating: 3 on 3"/>
                                                        <i class="ms-1 fa text-warning fa-star"></i>
                                                        <i class="ms-1 fa text-warning fa-star"></i>
                                                        <i class="ms-1 fa text-warning fa-star"></i>
                                                    </label>
                                                </div>
                                            </div>
                                            <div class="col-md-7">
                                                <label>Expected Closing:</label>
                                                <div class="input-group date" id="exp_closing_div">
                                                    <t t-set='date_formatted'><t t-options='{"widget": "date"}' t-esc="opportunity.date_deadline"/></t>
                                                    <input type="text" data-widget="datetime-picker" data-widget-type="date" name="date_deadline" t-att-value="date_formatted" class="datetimepicker-input form-control date_deadline" t-att-name="prefix"/>
                                                    <span class="input-group-text o_input_group_date_icon">
                                                        <span class="fa fa-calendar" role="img" aria-label="Calendar"></span>
                                                    </span>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="mb-3">
                                        <div class="row">
                                            <div class="col-md-5">
                                                <label class="col-form-label" for="next_activity">Next Activity</label>
                                                <select class="form-select next_activity" name="next_activity">
                                                    <t t-foreach="activity_types" t-as="activity_type">
                                                        <option t-att-data="activity_type.id" t-att-selected="activity_type.id == user_activity.activity_type_id.id"
                                                            t-att-name="activity_type.name" t-att-delay_count="activity_type.delay_count"
                                                            t-att-delay_unit="activity_type.delay_unit" t-att-summary="activity_type.summary">
                                                        <t t-esc="activity_type.name"/></option>
                                                    </t>
                                                </select>
                                            </div>
                                            <div class="col-md-7">
                                                <label class="col-form-label" for="activity_date_deadline">Next Activity Date</label>
                                                <div class="input-group date" id="next_activity_div" >
                                                    <t t-set='date_formatted'><t t-options='{"widget": "date"}' t-esc="user_activity.date_deadline"/></t>
                                                    <input type="text" data-widget="datetime-picker" data-widget-type="date" name="activity_date_deadline" t-att-value="date_formatted" class="form-control activity_date_deadline datetimepicker-input" t-att-name="prefix"/>
                                                    <span class="input-group-text o_input_group_date_icon">
                                                        <span class="fa fa-calendar" role="img" aria-label="Calendar" title="Calendar"></span>
                                                    </span>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="mb-3">
                                        <label class="col-form-label" for="activity_summary">Details Next Activity</label>
                                        <textarea rows="6" name="activity_summary" class="form-control activity_summary"><t t-esc="user_activity.summary"/></textarea>
                                    </div>
                                </main>
                                <footer class="modal-footer">
                                    <button type="button" class="btn btn-light" data-bs-dismiss="modal">Cancel</button>
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
                                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                                </header>
                                <main class="modal-body" id="sign-dialog">
                                    <t t-if="opportunity.partner_name">
                                        <div class="mb-3">
                                            <label class="col-form-label" for="partner_name">Customer Name</label>
                                            <input type="text" name="partner_name" class="form-control partner_name" t-att-value="opportunity.partner_name"/>
                                        </div>
                                    </t>
                                    <t t-if="not opportunity.partner_name">
                                        <div class="mb-3">
                                            <label class="col-form-label" for="partner_name">Customer Name</label>
                                            <input type="text" name="partner_name" class="form-control partner_name" t-att-value="opportunity.contact_name"/>
                                        </div>
                                    </t>
                                    <div class="mb-3">
                                        <label class="col-form-label" for="phone">Phone</label>
                                        <input type="text" name="phone" class="form-control phone" t-att-value="opportunity.phone"/>
                                    </div>
                                    <div class="mb-3">
                                        <label class="col-form-label" for="mobile">Mobile</label>
                                        <input type="text" name="mobile" class="form-control mobile" t-att-value="opportunity.mobile"/>
                                    </div>
                                    <div class="mb-3">
                                        <label class="col-form-label" for="email_from">Email</label>
                                        <input type="text" name="email_from" class="form-control email_from" t-att-value="opportunity.email_from"/>
                                    </div>
                                    <div class="mb-3">
                                        <label class="col-form-label" for="street">Address</label>
                                        <input type="text" name="street" class="form-control street" t-att-value="opportunity.street" placeholder="Street"/>
                                    </div>
                                    <div class="mb-3">
                                        <input type="text" name="street2" class="form-control street2" t-att-value="opportunity.street2" placeholder="Street2"/>
                                    </div>
                                    <div class="mb-3">
                                        <div class="row">
                                            <div class="col-md-5">
                                                <input type="text" name="city" class="form-control city" t-att-value="opportunity.city" placeholder="City"/>
                                            </div>
                                            <div class="col-md-5">
                                                <select name="state_id" class="form-select state_id">
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
                                    <div class="mb-3">
                                        <select name="country_id" class="form-select country_id">
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
                                    <button type="button" class="btn btn-light" data-bs-dismiss="modal">Cancel</button>
                                    <button t-attf-class="btn btn-primary edit_contact_confirm">Confirm</button>
                                </footer>
                            </form>
                        </div>
                    </div>
                </div>
                <!-- === / MODALS === -->
            </div>

            <hr/>

            <div>
                <h3>Communication history</h3>
                <div>
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
                raise UserError(_('Set an email address for the partner(s): %s', ", ".join(no_email)))
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
        return "%s/my/%s/%s" % (
            lead.get_base_url(),
            lead.type,
            lead.id,
        )

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
                        <field name="forward_type" invisible="context.get('hide_forward_type', False)"/>
                    </group>
                    <group>
                        <group>
                            <field name="partner_id" invisible="forward_type in ['assigned', False]" required="forward_type == 'single'"  />
                        </group>
                        <group>
                        </group>
                    </group>
                    <field name="assignation_lines" invisible="forward_type in ['single', False]">
                        <list create="false" editable="bottom">
                            <field name="lead_id" readonly="1" force_save="1" />
                            <field name="lead_location" readonly="1"/>
                            <field name="partner_assigned_id"/>
                            <field name="partner_location" readonly="1"/>
                            <field name="lead_link" column_invisible="True"/>
                        </list>
                    </field>
                    <notebook colspan="4" groups="base.group_no_one">
                        <page string="Email Template" name="email_template">
                            <field name="body" readonly="1" widget="html_mail"/>
                        </page>
                    </notebook>
                    <footer>
                        <button name="action_forward" string="Send" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" special="cancel" data-hotkey="x" class="btn-secondary"/>
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

