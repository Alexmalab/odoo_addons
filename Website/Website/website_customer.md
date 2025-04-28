# Odoo Module: website_customer

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
    'name': 'Customer References',
    'category': 'Website/Website',
    'summary': 'Publish your customer references',
    'version': '1.0',
    'description': """
Publish your customers as business references on your website to attract new potential prospects.
    """,
    'depends': [
        'website_crm_partner_assign',
        'website_partner',
        'website_google_map',
    ],
    'demo': [
        'data/res_partner_demo.xml',
    ],
    'data': [
        'views/website_customer_templates.xml',
        'views/res_partner_views.xml',
        'security/ir.model.access.csv',
        'security/ir_rule.xml',
        'views/snippets.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug.urls

from odoo import http
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.addons.website_google_map.controllers.main import GoogleMap
from odoo.tools.translate import _
from odoo.http import request


class WebsiteCustomer(GoogleMap):
    _references_per_page = 20

    def _get_gmap_domains(self, **kw):
        if kw.get('dom', '') != "website_customer.customers":
            return super()._get_gmap_domains(**kw)

        current_industry = kw.get('current_industry')
        current_country = kw.get('current_country')

        domain = [('assigned_partner_id', '!=', False)]

        if current_country and current_country != '0':
            domain += [('country_id', '=', int(current_country))]

        if current_industry and current_industry != '0':
            domain += [('industry_id', '=', int(current_industry))]

        return domain

    def sitemap_industry(env, rule, qs):
        if not qs or qs.lower() in '/customers':
            yield {'loc': '/customers'}

        Industry = env['res.partner.industry']
        dom = sitemap_qs2dom(qs, '/customers/industry', Industry._rec_name)
        for industry in Industry.search(dom):
            loc = '/customers/industry/%s' % env['ir.http']._slug(industry)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

        dom = [('website_published', '=', True), ('assigned_partner_id', '!=', False), ('country_id', '!=', False)]
        dom += sitemap_qs2dom(qs, '/customers/country')
        countries = env['res.partner'].sudo()._read_group(dom, ['country_id'])
        for [country] in countries:
            loc = '/customers/country/%s' % env['ir.http']._slug(country)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

    @http.route([
        '/customers',
        '/customers/page/<int:page>',
        '/customers/country/<model("res.country"):country>',
        '/customers/country/<model("res.country"):country>/page/<int:page>',
        '/customers/industry/<model("res.partner.industry"):industry>',
        '/customers/industry/<model("res.partner.industry"):industry>/page/<int:page>',
        '/customers/industry/<model("res.partner.industry"):industry>/country/<model("res.country"):country>',
        '/customers/industry/<model("res.partner.industry"):industry>/country/<model("res.country"):country>/page/<int:page>',
    ], type='http', auth="public", website=True, sitemap=sitemap_industry)
    def customers(self, country=None, industry=None, page=0, **post):
        Tag = request.env['res.partner.tag']
        Partner = request.env['res.partner']
        search_value = post.get('search')

        domain = [('website_published', '=', True), ('assigned_partner_id', '!=', False)]
        if search_value:
            domain += [
                '|', '|',
                ('name', 'ilike', search_value),
                ('website_description', 'ilike', search_value),
                ('industry_id.name', 'ilike', search_value),
            ]

        tag_id = post.get('tag_id')
        if tag_id:
            tag_id = request.env['ir.http']._unslug(tag_id)[1] or 0
            domain += [('website_tag_ids', 'in', tag_id)]

        # group by industry, based on customers found with the search(domain)
        industries = Partner.sudo().read_group(domain, ["id", "industry_id"], groupby="industry_id", orderby="industry_id")
        partners_count = Partner.sudo().search_count(domain)

        if industry:
            domain.append(('industry_id', '=', industry.id))
            if industry.id not in (x['industry_id'][0] for x in industries if x['industry_id']):
                if industry.exists():
                    industries.append({
                        'industry_id_count': 0,
                        'industry_id': (industry.id, industry.name)
                    })

        industries.sort(key=lambda d: (d.get('industry_id') or (0, ''))[1])

        industries.insert(0, {
            'industry_id_count': partners_count,
            'industry_id': (0, _("All Industries"))
        })

        # group by country, based on customers found with the search(domain)
        countries = Partner.sudo().read_group(domain, ["id", "country_id"], groupby="country_id", orderby="country_id")
        country_count = Partner.sudo().search_count(domain)

        fallback_all_countries = False
        if country:
            if country_count > 0 and country.id not in (x['country_id'][0] for x in countries if x['country_id']):
                # fallback on all countries if no customer found for the country
                # and there are matching customers for other countries
                fallback_all_countries = True
                country = None
            else:
                domain += [('country_id', '=', country.id)]

        countries.insert(0, {
            'country_id_count': country_count,
            'country_id': (0, _("All Countries"))
        })

        # search customers to display
        partner_count = Partner.sudo().search_count(domain)

        # pager
        url = '/customers'
        if industry:
            url += '/industry/%s' % industry.id
        if country:
            url += '/country/%s' % country.id
        pager = request.website.pager(
            url=url, total=partner_count, page=page, step=self._references_per_page,
            scope=7, url_args=post
        )

        partners = Partner.sudo().search(domain, offset=pager['offset'], limit=self._references_per_page)
        google_maps_api_key = request.website.google_maps_api_key

        tags = Tag.search([('website_published', '=', True), ('partner_ids', 'in', partners.ids)], order='classname, name ASC')
        tag = tag_id and Tag.browse(tag_id) or False

        values = {
            'countries': countries,
            'current_country_id': country.id if country and partners else 0,
            'current_country': country if partners and country else False,
            'industries': industries,
            'current_industry_id': industry.id if industry else 0,
            'current_industry': industry or False,
            'partners': partners,
            'pager': pager,
            'post': post,
            'search_path': "?%s" % werkzeug.urls.url_encode(post),
            'tag': tag,
            'tags': tags,
            'google_maps_api_key': google_maps_api_key,
            'fallback_all_countries': fallback_all_countries,
        }
        return request.render("website_customer.index", values)

    # Do not use semantic controller due to SUPERUSER_ID
    @http.route(['/customers/<partner_id>'], type='http', auth="public", website=True)
    def customers_detail(self, partner_id, **post):
        current_slug = partner_id
        _, partner_id = request.env['ir.http']._unslug(partner_id)
        if partner_id:
            partner = request.env['res.partner'].sudo().browse(partner_id)
            if partner.exists() and partner.website_published:
                if request.env['ir.http']._slug(partner) != current_slug:
                    return request.redirect('/customers/%s' % request.env['ir.http']._slug(partner))
                values = {}
                values['main_object'] = values['partner'] = partner
                return request.render("website_customer.details", values)
        raise request.not_found()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\res_partner_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="base.res_partner_2" model="res.partner">
            <field name="is_published" eval="True"/>
            <field name="website_short_description">Deco Addict designs, develops, integrates and supports HR and Supply Chain processes in order to make our customers more productive, responsive and profitable.</field>
            <field name="website_description" type="html">
                <p>
                Deco Addict designs, develops, integrates and supports HR and Supply
                Chain processes in order to make our customers more productive,
                responsive and profitable.
                </p><p>
                Our experts invent, imagine and develop solutions which meet
                your business requirements. They build a new technical
                environment for your company, but they always take the already
                installed IT software into account. That is why Idealis
                Consulting delivers excellence in HR and SC Management.
                </p><p>
                Deco Addict integrates ERP for Global Companies and supports PME
                with Open Sources software to manage their businesses. Our
                consultants are experts in the following areas:
                </p>
                <ul>
                    <li>Sales and Distribution</li>
                    <li>Materials Management</li>
                    <li>Inventory and Warehouse management</li>
                    <li>Customer Relationship Management</li>
                    <li>Personnel Administration</li>
                    <li>Talent Management</li>
                    <li>Reporting</li>
                </ul>
            </field>
        </record>
        <record id="base.res_partner_3" model="res.partner">
            <field name="is_published" eval="True"/>
            <field name="website_short_description">A non-profit international educational and scientific organisation, hosting three departments (aeronautics and aerospace, environmental and applied fluid dynamics, and turbomachinery and propulsion).</field>
            <field name="website_description" type="html">
                <p>
                    A non-profit international educational and scientific
                    organisation, hosting three departments (aeronautics and
                    aerospace, environmental and applied fluid dynamics, and
                    turbomachinery and propulsion).
                </p><p>
                    It provides post-graduate education in fluid dynamics
                    (research master in fluid dynamics, former "Diploma
                    Course", doctoral program, stagiaire program and lecture
                    series) and encourages "training in research through
                    research".
                </p><p>
                    It undertakes and promotes research in the field of fluid
                    dynamics. It possesses about fifty different wind tunnels,
                    turbomachinery and other specialized test facilities, some
                    of which are unique or the largest in the world. Extensive
                    research on experimental, computational and theoretical
                    aspects of gas and liquid flows is carried out under the
                    direction of the faculty and research engineers, sponsored
                    mainly by governmental and international agencies as well
                    as industries.
                </p>
            </field>
        </record>
        <record id="base.res_partner_4" model="res.partner">
            <field name="is_published" eval="True"/>
            <field name="website_short_description">Deco Addict designs, develops, integrates and supports HR and Supply Chain processes in order to make our customers more productive, responsive and profitable.</field>
            <field name="website_description" type="html">
                <p>
                Deco Addict designs, develops, integrates and supports HR and Supply
                Chain processes in order to make our customers more productive,
                responsive and profitable.
                </p><p>
                Our experts invent, imagine and develop solutions which meet
                your business requirements. They build a new technical
                environment for your company, but they always take the already
                installed IT software into account. That is why Idealis
                Consulting delivers excellence in HR and SC Management.
                </p><p>
                Deco Addict integrates ERP for Global Companies and supports PME
                with Open Sources software to manage their businesses. Our
                consultants are experts in the following areas:
                </p>
                <ul>
                    <li>Sales and Distribution</li>
                    <li>Materials Management</li>
                    <li>Inventory and Warehouse management</li>
                    <li>Customer Relationship Management</li>
                    <li>Personnel Administration</li>
                    <li>Talent Management</li>
                    <li>Reporting</li>
                </ul>
            </field>
        </record>
        <record id="base.res_partner_3" model="res.partner">
            <field name="is_published" eval="True"/>
            <field name="website_short_description">A non-profit international educational and scientific organisation, hosting three departments (aeronautics and aerospace, environmental and applied fluid dynamics, and turbomachinery and propulsion).</field>
            <field name="website_description" type="html">
                <p>
                    A non-profit international educational and scientific
                    organisation, hosting three departments (aeronautics and
                    aerospace, environmental and applied fluid dynamics, and
                    turbomachinery and propulsion).
                </p><p>
                    It provides post-graduate education in fluid dynamics
                    (research master in fluid dynamics, former "Diploma
                    Course", doctoral program, stagiaire program and lecture
                    series) and encourages "training in research through
                    research".
                </p><p>
                    It undertakes and promotes research in the field of fluid
                    dynamics. It possesses about fifty different wind tunnels,
                    turbomachinery and other specialized test facilities, some
                    of which are unique or the largest in the world. Extensive
                    research on experimental, computational and theoretical
                    aspects of gas and liquid flows is carried out under the
                    direction of the faculty and research engineers, sponsored
                    mainly by governmental and international agencies as well
                    as industries.
                </p>
            </field>
        </record>
    </data>
</odoo>

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Partner(models.Model):

    _inherit = 'res.partner'

    website_tag_ids = fields.Many2many(
        'res.partner.tag',
        'res_partner_res_partner_tag_rel',
        'partner_id',
        'tag_id',
        string='Website tags',
        help="Filter published customers on the .../customers website page",
    )

    def get_backend_menu_id(self):
        return self.env.ref('contacts.menu_contacts').id


class Tags(models.Model):

    _name = 'res.partner.tag'
    _description = 'Partner Tags - These tags can be used on website to find customers by sector, or ...'
    _inherit = 'website.published.mixin'

    @api.model
    def get_selection_class(self):
        classname = ['info', 'primary', 'success', 'warning', 'danger']
        return [(x, str.title(x)) for x in classname]

    name = fields.Char('Category Name', required=True, translate=True)
    partner_ids = fields.Many2many('res.partner', 'res_partner_res_partner_tag_rel', 'tag_id', 'partner_id', string='Partners')
    classname = fields.Selection('get_selection_class', 'Class', default='info', help="Bootstrap class to customize the color", required=True)
    active = fields.Boolean('Active', default=True)

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
        suggested_controllers.append((_('References'), self.env['ir.http']._url_for('/customers'), 'website_customer'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
res_partner_tag_sale_manager_public,res.partner.tag.sale.manager,model_res_partner_tag,base.group_public,1,0,0,0
res_partner_tag_sale_manager_portal,res.partner.tag.sale.manager,model_res_partner_tag,base.group_portal,1,0,0,0
res_partner_tag_sale_manager_employee,res.partner.tag.sale.manager,model_res_partner_tag,base.group_user,1,0,0,0
res_partner_tag_sale_manager_edition,res.partner.tag.sale.manager.edition,model_res_partner_tag,sales_team.group_sale_manager,1,1,1,1
res_partner_industry_public,res_partner_industry all,base.model_res_partner_industry,base.group_public,1,0,0,0
res_partner_industry_portal,res_partner_industry all,base.model_res_partner_industry,base.group_portal,1,0,0,0

```

## File: security\ir_rule.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <record model="ir.rule" id="website_customer_res_partner_tag_public">
            <field name="name">Partner Tag: published only</field>
            <field name="model_id" ref="model_res_partner_tag"/>
            <field name="domain_force">[('website_published', '=', True)]</field>
            <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43 24a5 5 0 1 1-10 0 5 5 0 0 1 10 0Z" fill="#1AD3BB"/><path d="M28 16c0 5.523-4.477 10-10 10S8 21.523 8 16 12.477 6 18 6s10 4.477 10 10Z" fill="#985184"/><path d="M16 31h26a4 4 0 0 1 4 4v5a4 4 0 0 1-4 4H16V31Z" fill="#1AD3BB"/><path d="M4 31h16c7.18 0 13 5.82 13 13H17C9.82 44 4 38.18 4 31Z" fill="#985184"/><path d="M13.95 21.331 18 18.397l4.052 2.934-1.607-4.716 4.053-2.934h-4.96L18 9l-1.537 4.68h-4.96l4.017 2.935-1.572 4.716Z" fill="#fff"/></svg>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="view_partners_form_website" model="ir.ui.view">
            <field name="name">view.res.partner.form.website.tags</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="website_partner.view_partners_form_website" />
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//field[@name='website_id']" position="after">
                        <field name="website_tag_ids" widget="many2many_tags" string="Website Tags"/>
                    </xpath>
                </data>
            </field>
        </record>

        <record id="view_partner_tag_form" model="ir.ui.view">
            <field name="name">Website Tags</field>
            <field name="model">res.partner.tag</field>
            <field name="arch" type="xml">
                <form string="Partner Tag">
                <sheet>
                    <group col="4">
                        <field name="name"/>
                        <field name="classname"/>
                        <field name="is_published"/>
                        <field name="active" widget="boolean_toggle"/>
                    </group>
                </sheet>
                </form>
            </field>
        </record>

        <record id="view_partner_tag_list" model="ir.ui.view">
            <field name="name">Website Tags</field>
            <field name="model">res.partner.tag</field>
            <field eval="6" name="priority"/>
            <field name="arch" type="xml">
                <list string="Website Tags" editable="bottom">
                    <field name="name"/>
                    <field name="classname"/>
                    <field name="is_published"/>
                    <field name="active" column_invisible="True"/>
                </list>
            </field>
        </record>

        <record id="res_partner_tag_view_search" model="ir.ui.view">
            <field name="name">res.partner.tag.view.search</field>
            <field name="model">res.partner.tag</field>
            <field name="arch" type="xml">
                <search string="Search Partner Tag">
                    <field name="name"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="action_partner_tag_form" model="ir.actions.act_window">
            <field name="name">Website Tags</field>
            <field name="res_model">res.partner.tag</field>
            <field name="search_view_id" ref="res_partner_tag_view_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new contact tag
              </p><p>
                Manage contact tags to better classify them for tracking and analysis purposes.
              </p>
            </field>
        </record>

        <menuitem
            action="action_partner_tag_form"
            id="menu_partner_tag_form"
            name="Website Tags"
            sequence="2"
            parent="contacts.res_partner_menu_config"
        />

</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Customer Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector=".o_wcrm_filters_top" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="References Page">
            <we-checkbox string="Countries Filter"
                         data-customize-website-views="website_customer.opt_country_list"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Industry Filter"
                         data-customize-website-views="website_customer.opt_industry_list"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Tags Filter"
                         data-customize-website-views="website_customer.opt_tag_list"
                         data-no-preview="true"
                         data-reload="/"/>
            <t t-if="google_maps_api_key">
                <we-checkbox string="Show Map"
                            data-customize-website-views="website_customer.opt_country"
                            data-no-preview="true"
                            data-reload="/"/>
            </t>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_customer_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="index" name="Our References">
    <t t-call="website.layout">
        <div id="wrap">
            <div class="container my-4">
                <div class="o_wcrm_filters_top d-flex d-print-none align-items-center justify-content-end flex-wrap gap-2 w-100">
                    <h4 class="my-0 me-auto pe-sm-4">Our references</h4>
                    <div class="o_wcrm_search d-flex w-100 w-lg-auto">
                        <form action="" method="get" class="flex-grow-1">
                            <div class="input-group" role="search">
                                <input type="text" name="search" class="search-query form-control border-0 bg-light" placeholder="Search" t-att-value="post.get('search', '')"/>
                                <button type="submit" aria-label="Search" title="Search" class="oe_search_button btn btn-light">
                                    <i class="oi oi-search"/>
                                </button>
                            </div>
                        </form>
                        <button class="btn btn-light position-relative ms-2 d-lg-none"
                            data-bs-toggle="offcanvas"
                            data-bs-target="#o_wc_offcanvas">
                            <i class="fa fa-sliders"/>
                        </button>
                    </div>
                </div>
                <!-- Off canvas filters on mobile-->
                <div id="o_wc_offcanvas" class="o_website_offcanvas offcanvas offcanvas-end d-lg-none p-0 overflow-visible">
                    <div class="offcanvas-header">
                        <h5 class="offcanvas-title">Filters</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"/>
                    </div>
                    <div class="offcanvas-body p-0">
                        <div class="accordion accordion-flush">
                            <div class="accordion-item">
                                <h2 class="accordion-header">
                                    <button class="accordion-button border-top collapsed"
                                        type="button"
                                        data-bs-toggle="collapse"
                                        data-bs-target=".o_wc_offcanvas_industry"
                                        aria-expanded="false"
                                        aria-controls="wc_offcanvas_industry">
                                        Filter by Industry
                                    </button>
                                </h2>
                                <div class="o_wc_offcanvas_industry accordion-collapse collapse">
                                    <div class="accordion-body pt-0">
                                        <ul class="list-group list-group-flush">
                                            <t t-foreach="industries" t-as="industry_dict">
                                                <t t-if="industry_dict['industry_id']">
                                                    <li class="list-group-item d-flex justify-content-between align-items-center ps-0 pb-0 border-0">
                                                        <a t-attf-href="/customers/#{ industry_dict['industry_id'][0] and 'industry/%s/' % slug(industry_dict['industry_id']) or '' }#{ current_country_id and 'country/%s' % current_country_id or '' }#{ search_path }" 
                                                            class="text-reset" aria-label="See industries filters">
                                                            <div class="form-check">
                                                                <input class="form-check-input pe-none" type="radio" t-attf-name="#{industry_dict['industry_id'][1]}" t-att-checked="industry_dict['industry_id'][0] == current_industry_id and true or false"/>
                                                                <label class="form-check-label" t-attf-for="#{industry_dict['industry_id'][1]}" t-out="industry_dict['industry_id'][1]"/>
                                                            </div>
                                                        </a>
                                                    </li>
                                                </t>
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
                                        data-bs-target=".o_wc_offcanvas_country"
                                        aria-expanded="false"
                                        aria-controls="offcanvas_country">
                                        Filter by Country
                                    </button>
                                </h2>
                                <div class="o_wc_offcanvas_country accordion-collapse collapse">
                                    <div class="accordion-body pt-0">
                                        <ul class="list-group list-group-flush ">
                                            <t t-foreach="countries" t-as="country_dict">
                                                <t t-if="country_dict['country_id']">
                                                    <li class="list-group-item d-flex justify-content-between align-items-center ps-0 pb-0 border-0">
                                                        <a t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ country_dict['country_id'][0] and 'country/%s' % slug(country_dict['country_id']) or '' }#{ search_path }"
                                                            class="text-reset" aria-label="See countries filters">
                                                            <div class="form-check">
                                                                <input class="form-check-input pe-none" type="radio" t-attf-name="{country_dict['country_id'][1]}" t-att-checked="country_dict['country_id'][0] == current_country_id and true or false"/>
                                                                <label class="form-check-label" t-attf-for="{country_dict['country_id'][1]}" t-out="country_dict['country_id'][1]"/>
                                                            </div>
                                                        </a>
                                                    </li>
                                                </t>
                                            </t>
                                        </ul>
                                    </div>
                                </div>
                            </div>
                            <div class="accordion-item" t-if="len(tags)">
                                <h2 class="accordion-header">
                                    <button class="accordion-button border-top collapsed"
                                        type="button"
                                        data-bs-toggle="collapse"
                                        data-bs-target=".o_wc_offcanvas_tags"
                                        aria-expanded="false"
                                        aria-controls="o_wc_offcanvas_tags">
                                        Filter by Tags
                                    </button>
                                </h2>
                                <div class="o_wc_offcanvas_tags accordion-collapse collapse">
                                    <div class="accordion-body">
                                        <div class="d-flex flex-wrap align-items-center gap-1 mb-4" t-if="len(tags)">
                                            <a class="badge text-bg-info" t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ current_country_id and 'country/%s' % slug(current_country) or '' }">
                                                <span class="fa fa-1x fa-tags"/> All </a>
                                            <t t-foreach="tags" t-as="o_tag">
                                                <a t-attf-class="badge text-bg-#{o_tag.classname}" t-out="o_tag.name" t-att-style="tag and tag.id==o_tag.id and 'text-decoration: underline'"
                                                t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ current_country_id and 'country/%s' % slug(current_country) or '' }?tag_id=#{slug(o_tag)}"/>
                                            </t>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="row mt-4">
                    <div class="my-5 py-5 text-center" t-if="not partners">
                        <h5>No results found for "<span t-out="post.get('search', '')"/>"</h5>
                        <a href="/customers">See all customers</a>
                    </div>
                    <div t-elif="fallback_all_countries" class="alert alert-primary alert-dismissible fade show" role="alert">
                        <i class="fa fa-info-circle me-2"/>
                        There are no matching customers found for the selected country. Displaying results across all countries instead.
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                    </div>
                    <t t-foreach="partners" t-as="partner">
                        <div class="col-md-4 col-xl-3 col-12 mb-4">
                            <div class="card h-100 text-decoration-none">
                                <a class="text-decoration-none" t-attf-href="/customers/#{slug(partner)}" aria-label="Go to customer">
                                    <div t-field="partner.avatar_1920"
                                        class="card-img-top border-bottom"
                                        t-options='{"widget": "image", "qweb_img_responsive": False, "class": "img img-fluid h-100 w-100 mw-100 object-fit-cover", "style": "max-height: 208px; object-fit: cover"}'
                                    />
                                    <div class="card-body">
                                        <h5 class="card-title" t-field="partner.display_name"/>
                                        <small class="o_wcrm_short_description text-muted overflow-hidden" t-field="partner.website_short_description"/>
                                        <small t-if="not partner.website_short_description" class="css_editable_mode_hidden text-muted fst-italic" groups="website.group_website_restricted_editor">
                                            Enter a short description
                                        </small>
                                        <t t-if="partner.industry_id">
                                            <a class="badge mt-3 text-bg-secondary" t-attf-href="/customers/#{ 'industry/%s/' % slug(partner.industry_id) }#{ current_country_id and 'country/%s' % slug(current_country) or '' }" t-out="partner.industry_id.name"/>
                                        </t>
                                    </div>
                                </a>
                            </div>
                        </div>
                    </t>
                </div>
                <div class="navbar">
                    <t t-call="website.pager">
                        <t t-set="classname" t-value="'mx-auto'"/>
                    </t>
                </div>
            </div>
        </div>
    </t>
</template>

<!-- Option: top filters: World Map -->
<template id="opt_country" inherit_id="website_customer.index" name="Show Map">
    <xpath expr="//div[hasclass('o_wcrm_search')]" position="inside">
        <t t-if="google_maps_api_key">
            <!-- modal for large map -->
            <div role="dialog" class="modal fade customer_map_modal" tabindex="-1">
              <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">World Map</h4>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"/>
                    </header>
                    <iframe loading="lazy" t-attf-src="/google_map?width=898&amp;height=485&amp;dom=website_customer.customers&amp;current_industry=#{current_industry_id}&amp;current_country=#{current_country_id}&amp;partner_url=/customers/&amp;limit=1000"
                    style="height:485px;"/>
                </div>
              </div>
            </div>
            <!-- modal end -->
            <div class="btn-group ms-2">
                <button class="btn btn-light border-primary active">
                    <i class="fa fa-th-large"/>
                </button>
                <button class="btn btn-light" data-bs-toggle="modal" data-bs-target=".customer_map_modal">
                    <i class="fa fa-map-marker" role="img" aria-label="Open map" title="Open map"/>
                </button>
            </div>
        </t>
    </xpath>
</template>

<template id="opt_industry_list" inherit_id="website_customer.index" name="Filter on Industry" priority="20">
    <xpath expr="//div[hasclass('o_wcrm_search')]" position="before">
        <div class="dropdown d-none d-lg-block">
            <a role="button" href="#" data-bs-toggle="dropdown" t-attf-class="dropdown-toggle btn btn-light" aria-expanded="true" aria-label="Open industries dropdown">
                <t t-foreach="industries" t-as="industry_dict">
                    <label class="cursor-pointer" t-if="industry_dict['industry_id'] and industry_dict['industry_id'][0] == current_industry_id and true or false" t-out="industry_dict['industry_id'][1]"/>
                </t>
            </a>
            <div class="dropdown-menu">
                <t t-foreach="industries" t-as="industry_dict">
                    <t t-if="industry_dict['industry_id']">
                        <a t-attf-href="/customers/#{ industry_dict['industry_id'][0] and 'industry/%s/' % slug(industry_dict['industry_id']) or '' }#{ current_country_id and 'country/%s' % current_country_id or '' }#{ search_path }"
                        class="dropdown-item" t-out="industry_dict['industry_id'][1]"/>
                    </t>
                </t>
            </div>
        </div>
    </xpath>
</template>

<template id="opt_country_list" inherit_id="website_customer.index" name="Filter on Countries" priority="30">
    <xpath expr="//div[hasclass('o_wcrm_search')]" position="before">
        <div class="dropdown d-none d-lg-block">
            <a role="button" href="#" data-bs-toggle="dropdown" t-attf-class="dropdown-toggle btn btn-light" aria-expanded="true" aria-label="Open countries dropdown">
                <t t-foreach="countries" t-as="country_dict">
                    <label class="cursor-pointer" t-if="country_dict['country_id'] and current_country_id == country_dict['country_id'][0]">
                        <t t-out="country_dict['country_id'][1]"/>
                    </label>
                </t>
            </a>
            <div class="dropdown-menu">
                <t t-foreach="countries" t-as="country_dict">
                    <t t-if="country_dict['country_id']">
                        <a t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ country_dict['country_id'][0] and 'country/%s' % slug(country_dict['country_id']) or '' }#{ search_path }"
                        class="dropdown-item" t-out="country_dict['country_id'][1]"/>
                    </t>
                </t>
            </div>
        </div>
    </xpath>
</template>


<template id="opt_tag_list" inherit_id="website_customer.index" name="Filter on Tags" priority="40">
    <xpath expr="//div[hasclass('o_wcrm_filters_top')]" position="after">
        <div class="d-flex flex-wrap align-items-center gap-2 my-4" t-if="len(tags)">
            <a class="badge text-bg-info" t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ current_country_id and 'country/%s' % slug(current_country) or '' }">
                <span class="fa fa-1x fa-tags"/> All
            </a>
            <t t-foreach="tags" t-as="o_tag">
                <a t-attf-class="text-bg-#{o_tag.classname} badge" t-out="o_tag.name" t-att-style="tag and tag.id==o_tag.id and 'text-decoration: underline'"
                t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ current_country_id and 'country/%s' % slug(current_country) or '' }?tag_id=#{slug(o_tag)}"/>
            </t>
        </div>
    </xpath>
</template>

<template id="details" name="Customer Detail">
  <t t-call="website.layout">
    <div id="wrap">
        <t t-set="editor_message">DROP BUILDING BLOCKS HERE TO MAKE THEM AVAILABLE ACROSS ALL CUSTOMERS</t>
        <div class="oe_structure oe_empty" id="oe_structure_website_customer_details_1" t-att-data-editor-message="editor_message"/>
        <div class="container my-4">
            <div class="row">
                <div class="mb-3" t-if="not edit_page">
                    <a t-attf-href="/customers" aria-label="Back to references list"><i class="fa fa-chevron-left me-2"/>Back to references</a>
                </div>
                <t t-call="website_partner.partner_detail">
                    <t t-set="right_column">
                        <div id="right_column"><t t-call="website_customer.references_block"/></div>
                    </t>
                </t>
            </div>
        </div>
        <div class="oe_structure oe_empty" id="oe_structure_website_customer_details_2" t-att-data-editor-message="editor_message"/>
    </div>
  </t>
</template>

<template id="partner_details" inherit_id="website_partner.partner_page" name="Partner Detail Columns">
 <xpath expr="//t[@t-call='website_partner.partner_detail']" position="inside">
    <t t-set="left_column"><div id="left_column"><t t-call="website_customer.implemented_by_block"/></div></t>
    <t t-set="right_column"><div id="right_column"><t t-call="website_customer.references_block"/></div></t>
 </xpath>
</template>

<template id="partner_detail" inherit_id="website_partner.partner_detail" name="Partner Details">
    <xpath expr="//div[hasclass('o_wcrm_contact_details')]" position="inside">
        <t t-if="partner.industry_id">
            <span class="badge text-bg-secondary"><t t-out="partner.industry_id.name"/></span>
        </t>
    </xpath>
</template>

<template id="implemented_by_block" name="Partner Implemented By Block">
        <t t-if="partner.assigned_partner_id and partner.assigned_partner_id.website_published">
            <div class="d-flex align-items-center">
                <div>
                    <a t-attf-href="/partners/#{slug(partner.assigned_partner_id)}"
                    t-field="partner.assigned_partner_id.avatar_128"
                    class="d-block me-2 p-1 border rounded-circle shadow-sm"
                    style="width: 42px; height: 42px"
                    t-options='{"widget": "image", "qweb_img_responsive": False, "class": "img-fluid h-100 w-100 rounded-circle"}'
                    />
                </div>
                <div>
                    <span class="small text-muted">Implemented by</span>
                    <div>
                        <a t-attf-href="/partners/#{slug(partner.assigned_partner_id)}">
                            <span t-field="partner.assigned_partner_id"/>
                        </a>
                    </div>
                </div>
            </div>
        </t>
</template>

<template id="references_block" name="Partner References Block">
        <t t-if="any(p.website_published for p in partner.implemented_partner_ids)">
            <h3 id="references">References</h3>
            <div t-foreach="partner.implemented_partner_ids" t-as="reference" class="card mt-3 border-0">
                <t t-if="reference.website_published">
                    <div class="row">
                        <div class="col-md-2">
                            <span t-field="reference.avatar_128"
                                class="d-flex justify-content-center"
                                t-options='{"widget": "image", "qweb_img_responsive": False, "class": "img-fluid rounded mw-100"}'/>
                        </div>
                        <div class="card-body col-md-10">
                            <a t-attf-href="/customers/#{slug(reference)}">
                                <span t-field="reference.self"/>
                            </a>
                            <t t-if="reference.industry_id">
                                <span class="badge ms-1 text-bg-secondary"><t t-out="reference.industry_id.name"/></span>
                            </t>
                            <div t-field='reference.website_short_description'/>
                        </div>
                    </div>
                </t>
            </div>
        </t>
</template>

<template id="references_block_href" inherit_id="website_crm_partner_assign.references_block" name="Partner References Block">
    <xpath expr="//div/span" position="replace">
        <a t-attf-href="/customers/#{slug(reference)}">$0</a>
    </xpath>
    <xpath expr="//div[hasclass('card-body')]/span" position="replace">
        <a t-attf-href="/customers/#{slug(reference)}">$0</a>
    </xpath>
</template>

</odoo>

```

