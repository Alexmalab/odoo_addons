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
    ],
    'qweb': [],
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
from odoo.addons.http_routing.models.ir_http import unslug, slug
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.tools.translate import _
from odoo.http import request


class WebsiteCustomer(http.Controller):
    _references_per_page = 20

    def sitemap_industry(env, rule, qs):
        if not qs or qs.lower() in '/customers':
            yield {'loc': '/customers'}

        Industry = env['res.partner.industry']
        dom = sitemap_qs2dom(qs, '/customers/industry', Industry._rec_name)
        for industry in Industry.search(dom):
            loc = '/customers/industry/%s' % slug(industry)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

        dom = [('website_published', '=', True), ('assigned_partner_id', '!=', False), ('country_id', '!=', False)]
        dom += sitemap_qs2dom(qs, '/customers/country')
        countries = env['res.partner'].sudo().read_group(dom, ['id', 'country_id'], groupby='country_id')
        for country in countries:
            loc = '/customers/country/%s' % slug(country['country_id'])
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
            tag_id = unslug(tag_id)[1] or 0
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

        if country:
            domain += [('country_id', '=', country.id)]
            if country.id not in (x['country_id'][0] for x in countries if x['country_id']):
                if country.exists():
                    countries.append({
                        'country_id_count': 0,
                        'country_id': (country.id, country.name)
                    })
                    countries.sort(key=lambda d: (d['country_id'] or (0, ""))[1])

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
        google_map_partner_ids = ','.join(str(it) for it in partners.ids)
        google_maps_api_key = request.website.google_maps_api_key

        tags = Tag.search([('website_published', '=', True), ('partner_ids', 'in', partners.ids)], order='classname, name ASC')
        tag = tag_id and Tag.browse(tag_id) or False

        values = {
            'countries': countries,
            'current_country_id': country.id if country else 0,
            'current_country': country or False,
            'industries': industries,
            'current_industry_id': industry.id if industry else 0,
            'current_industry': industry or False,
            'partners': partners,
            'google_map_partner_ids': google_map_partner_ids,
            'pager': pager,
            'post': post,
            'search_path': "?%s" % werkzeug.urls.url_encode(post),
            'tag': tag,
            'tags': tags,
            'google_maps_api_key': google_maps_api_key,
        }
        return request.render("website_customer.index", values)

    # Do not use semantic controller due to SUPERUSER_ID
    @http.route(['/customers/<partner_id>'], type='http', auth="public", website=True)
    def partners_detail(self, partner_id, **post):
        _, partner_id = unslug(partner_id)
        if partner_id:
            partner = request.env['res.partner'].sudo().browse(partner_id)
            if partner.exists() and partner.website_published:
                values = {}
                values['main_object'] = values['partner'] = partner
                return request.render("website_customer.details", values)
        return self.customers(**post)

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
            <field name="website_description" type="xml">
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
            <field name="website_description" type="xml">
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
            <field name="website_description" type="xml">
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
            <field name="website_description" type="xml">
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

    website_tag_ids = fields.Many2many('res.partner.tag', 'res_partner_res_partner_tag_rel', 'partner_id', 'tag_id', string='Website tags')

    def get_backend_menu_id(self):
        return self.env.ref('contacts.menu_contacts').id


class Tags(models.Model):

    _name = 'res.partner.tag'
    _description = 'Partner Tags - These tags can be used on website to find customers by sector, or ...'
    _inherit = 'website.published.mixin'

    @api.model
    def get_selection_class(self):
        classname = ['default', 'primary', 'success', 'warning', 'danger']
        return [(x, str.title(x)) for x in classname]

    name = fields.Char('Category Name', required=True, translate=True)
    partner_ids = fields.Many2many('res.partner', 'res_partner_res_partner_tag_rel', 'tag_id', 'partner_id', string='Partners')
    classname = fields.Selection('get_selection_class', 'Class', default='default', help="Bootstrap class to customize the color", required=True)
    active = fields.Boolean('Active', default=True)

    def _default_is_published(self):
        return True

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
        suggested_controllers.append((_('References'), url_for('/customers'), 'website_customer'))
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
res_partner_tag_sale_manager,res.partner.tag.sale.manager,model_res_partner_tag,,1,0,0,0
res_partner_tag_sale_manager_edition,res.partner.tag.sale.manager.edition,model_res_partner_tag,sales_team.group_sale_manager,1,1,1,1
res_partner_industry_all,res_partner_industry all,base.model_res_partner_industry,,1,0,0,0

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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M28.352 34.871c.482.038 1.02.074 1.648.129.8.07 2-1.196 2-2 0-1.693-1.596-2.82-2-4 1.84-1.809 2-2.806 2-5.304 0-.666-.061-1.317-.18-1.949a11.475 11.475 0 0 1 6.855-2.247c6.235 0 11.29 4.897 11.29 10.938 0 6.04-5.055 10.937-11.29 10.937-4.606 0-8.567-2.672-10.323-6.504zm23.255 7.449c3.015.73 5.13 3.355 5.13 6.366v2.533c0 1.812-1.516 3.281-3.386 3.281H23.999c-1.87 0-3.386-1.469-3.386-3.281v-2.533c0-3.011 2.115-5.636 5.13-6.366l5.032-1.219c4.106 2.862 10.514 3.684 15.8 0l5.032 1.219zM19.151 14.158c6.41 0 11.611 4.09 11.611 9.14 0 2.167-.955 4.153-2.552 5.722.351 1.024.891 1.437 1.496 1.894.448.338 1.298.866.992 1.898a1.45 1.45 0 0 1-1.536 1.024c-2.326-.202-4.483-.843-6.33-1.867-1.157.303-2.395.47-3.677.47-6.414 0-11.611-4.087-11.611-9.14-.004-5.05 5.193-9.141 11.607-9.141zm0 16.171c1.31 0 2.637-.193 3.923-.615 1.302.835 3.173 2.022 5.43 2.373-1.29-1.054-2.29-2.707-2.467-3.884 1.536-1.143 2.79-2.853 2.79-4.905 0-3.116-3.48-7.031-9.676-7.031-6.197 0-9.676 3.915-9.676 7.031 0 3.12 3.48 7.031 9.676 7.031z"/><path id="e" d="M28.352 32.871c.482.038 1.02.074 1.648.129.8.07 2-1.196 2-2 0-1.693-1.596-2.82-2-4 1.84-1.809 2-2.806 2-5.304 0-.666-.061-1.317-.18-1.949a11.475 11.475 0 0 1 6.855-2.247c6.235 0 11.29 4.897 11.29 10.938 0 6.04-5.055 10.937-11.29 10.937-4.606 0-8.567-2.672-10.323-6.504zm23.255 7.449c3.015.73 5.13 3.355 5.13 6.366v2.533c0 1.812-1.516 3.281-3.386 3.281H23.999c-1.87 0-3.386-1.469-3.386-3.281v-2.533c0-3.011 2.115-5.636 5.13-6.366l5.032-1.219c4.106 2.862 10.514 3.684 15.8 0l5.032 1.219zM19.151 12.158c6.41 0 11.611 4.09 11.611 9.14 0 2.167-.955 4.153-2.552 5.722.351 1.024.891 1.437 1.496 1.894.448.338 1.298.866.992 1.898a1.45 1.45 0 0 1-1.536 1.024c-2.326-.202-4.483-.843-6.33-1.867-1.157.303-2.395.47-3.677.47-6.414 0-11.611-4.087-11.611-9.14-.004-5.05 5.193-9.141 11.607-9.141zm0 16.171c1.31 0 2.637-.193 3.923-.615 1.302.835 3.173 2.022 5.43 2.373-1.29-1.054-2.29-2.707-2.467-3.884 1.536-1.143 2.79-2.853 2.79-4.905 0-3.116-3.48-7.031-9.676-7.031-6.197 0-9.676 3.915-9.676 7.031 0 3.12 3.48 7.031 9.676 7.031z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-.145-4-4.054V27.195l7.973-7.858c2.666-5.405 7.36-6.757 12.027-6.08 4.667.675 8 2.702 10 6.08v4.16l1.956-3.643 17.05 12.426-8.914 11.08 16.21 7.48L43.794 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
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
                <tree string="Website Tags" editable="bottom">
                    <field name="name"/>
                    <field name="classname"/>
                    <field name="is_published"/>
                    <field name="active" invisible="1"/>
                </tree>
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
            <field name="type">ir.actions.act_window</field>
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

## File: views\website_customer_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="index" name="Our References">
    <t t-call="website.layout">
        <div id="wrap">
            <div class="oe_structure">
                <section>
                    <h1 class="text-center">
                        Our References
                    </h1><h2 class="text-center text-muted">
                        Trusted by millions worldwide
                    </h2>
                </section>
            </div>
            <div class="container">
                <div class="row">
                    <div class="col-lg-3 mb32" id="ref_left_column">
                    </div>
                    <div class="col-lg-8 offset-lg-1" id="ref_content">
                        <div class='d-flex m-2'>
                            <t t-call="website.pager">
                               <t t-set="classname" t-value="'float-left'"/>
                            </t>
                            <form action="" method="get" class="navbar-search ml-auto pagination form-inline">
                                <div class="form-group">
                                    <input type="text" name="search" class="search-query form-control"
                                        placeholder="Search" t-att-value="post.get('search', '')"/>
                                </div>
                            </form>
                        </div>

                        <div>

                    <p t-if="not partners">No result found</p>
                    <t t-foreach="partners" t-as="partner">
                        <div class="media mt-3">
                            <a t-attf-href="/customers/#{slug(partner)}"
                               t-field="partner.image_128"
                               class="d-block mr-3 text-center o_width_128"
                               t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_128_max"}'
                            ></a>
                            <div class="media-body" style="min-height: 64px;">
                                <a t-attf-href="/customers/#{slug(partner)}">
                                    <span t-field="partner.display_name"/>
                                </a>
                                <t t-if="partner.industry_id">
                                    <a class="badge badge-secondary" t-attf-href="/customers/#{ 'industry/%s/' % slug(partner.industry_id) }#{ current_country_id and 'country/%s' % slug(current_country) or '' }" t-esc="partner.industry_id.name"/>
                                </t>
                                <div t-field="partner.website_short_description"/>
                            </div>
                        </div>
                    </t>
                        </div>
                    </div>

                </div>
            </div>
            <div class="oe_structure"/>
        </div>
    </t>
</template>

<!-- Option: left column: World Map -->
<template id="opt_country" inherit_id="website_customer.index" customize_show="True" name="Show Map">
    <xpath expr="//div[@id='ref_left_column']" position="inside">
        <t t-if="google_maps_api_key">
            <!-- modal for large map -->
            <div role="dialog" class="modal fade customer_map_modal" tabindex="-1">
              <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">World Map</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                    </header>
                    <iframe t-attf-src="/google_map/?width=898&amp;height=485&amp;partner_ids=#{ google_map_partner_ids }&amp;partner_url=/customers/"
                    style="width:898px; height:485px; border:0; padding:0; margin:0;"></iframe>
                </div>
              </div>
            </div>
            <!-- modal end -->
            <h3>World Map<button class="btn btn-link" data-toggle="modal" data-target=".customer_map_modal"><span class="fa fa-external-link" role="img" aria-label="External link" title="External link"/></button></h3>
            <ul class="nav">
                <iframe t-attf-src="/google_map?width=260&amp;height=240&amp;partner_ids=#{ google_map_partner_ids }&amp;partner_url=/customers/"
                    style="width:260px; height:240px; border:0; padding:0; margin:0;" scrolling="no"></iframe>
            </ul>
        </t>
    </xpath>
</template>

<template id="opt_industry_list" inherit_id="website_customer.index" customize_show="True" name="Filter on Industry" priority="20">
    <xpath expr="//div[@id='ref_left_column']" position="inside">
        <h3>References by Industries</h3>
        <ul class="nav nav-pills flex-column mt16 mb32">
            <t t-foreach="industries" t-as="industry_dict">
                <t t-if="industry_dict['industry_id']">
                    <li class="nav-item">
                        <a t-attf-href="/customers/#{ industry_dict['industry_id'][0] and 'industry/%s/' % slug(industry_dict['industry_id']) or '' }#{ current_country_id and 'country/%s' % current_country_id or '' }#{ search_path }"
                           t-attf-class="nav-link#{industry_dict['industry_id'][0] == current_industry_id and ' active' or ''}">
                            <span class="badge badge-pill float-right" t-esc="industry_dict['industry_id_count'] or '0'"/>
                            <t t-esc="industry_dict['industry_id'][1]"/>
                        </a>
                    </li>
                </t>
            </t>
        </ul>
    </xpath>
</template>

<template id="opt_country_list" inherit_id="website_customer.index" customize_show="True" name="Filter on Countries" priority="30">
    <xpath expr="//div[@id='ref_left_column']" position="inside">
        <h3>References by Country</h3>
        <ul class="nav nav-pills flex-column mt16 mb32">
            <t t-foreach="countries" t-as="country_dict">
                <t t-if="country_dict['country_id']">
                    <li class="nav-item">
                        <a t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ country_dict['country_id'][0] and 'country/%s' % slug(country_dict['country_id']) or '' }#{ search_path }"
                           t-attf-class="nav-link#{country_dict['country_id'][0] == current_country_id and ' active' or ''}">
                            <span class="badge badge-pill float-right" t-esc="country_dict['country_id_count'] or '0'"/>
                            <t t-esc="country_dict['country_id'][1]"/>
                        </a>
                    </li>
                </t>
            </t>
        </ul>
    </xpath>
</template>


<template id="opt_tag_list" inherit_id="website_customer.index" customize_show="True" name="Filter on Tags" priority="40">
    <xpath expr="//div[@id='ref_left_column']" position="inside">

        <h3 t-if="len(tags)">References by Tag</h3>
        <ul class="nav nav-pills flex-column mt16 mb32" t-if="len(tags)">
            <li class="nav-item"><a class="nav-link mr8" t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ current_country_id and 'country/%s' % slug(current_country) or '' }">
                <span class="fa fa-1x fa-tags"/> All </a></li>
            <li t-foreach="tags" t-as="o_tag" class="nav-item">
                <a t-attf-class="nav-link badge badge-#{o_tag.classname}" t-esc="o_tag.name" t-att-style="tag and tag.id==o_tag.id and 'text-decoration: underline'"
                    t-attf-href="/customers/#{ current_industry_id and 'industry/%s/' % slug(current_industry) or '' }#{ current_country_id and 'country/%s' % slug(current_country) or '' }?tag_id=#{slug(o_tag)}"/>
            </li>
        </ul>
    </xpath>
</template>

<template id="contact_edit_options" inherit_id="website.user_navbar" name="Edit Customer Options">
    <xpath expr="//li[@id='edit-page-menu']" position="after">
        <t t-if="main_object._name == 'res.partner'" t-set="action" t-value="'contacts.action_contacts'"/>
    </xpath>
</template>

<template id="details" name="Customer Detail">
  <t t-call="website.layout">
    <div id="wrap">
        <div class="oe_structure" id="oe_structure_website_customer_details_1"/>
        <div class="container mt16">
            <div class="row">
                <div class="col-lg-5">
                    <ol t-if="not edit_page" class="breadcrumb">
                        <li class="breadcrumb-item"><a href="/customers">Our References</a></li>
                        <li class="breadcrumb-item active"><span t-field="partner.display_name"/></li>
                    </ol>
                </div>
                <t t-call="website_partner.partner_detail">
                    <t t-set="left_column">
                        <div id="left_column"><t t-call="website_customer.implemented_by_block"/></div>
                    </t>
                    <t t-set="right_column">
                        <div id="right_column"><t t-call="website_customer.references_block"/></div>
                    </t>
                </t>
            </div>
        </div>
        <div class="oe_structure" id="oe_structure_website_customer_details_2"/>
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
    <xpath expr="//address" position="inside">
        <t t-if="partner.industry_id">
            <span class="badge badge-secondary"><t t-esc="partner.industry_id.name"/></span>
        </t>
    </xpath>
</template>

<template id="implemented_by_block" name="Partner Implemented By Block">
        <t t-if="partner.assigned_partner_id and partner.assigned_partner_id.website_published">
            <div class="card">
                <div class="card-header">
                    <h4>Implemented By</h4>
                </div>
                <div class="card-body text-center">
        <h4>
            <a t-attf-href="/partners/#{slug(partner.assigned_partner_id)}">
              <span t-field="partner.assigned_partner_id"/>
              <span class="small"> (<t t-esc="len([p for p in partner.assigned_partner_id.implemented_partner_ids if p.website_published])"/> reference(s))</span>
            </a>
        </h4>
        <div><a t-attf-href="/partners/#{slug(partner.assigned_partner_id)}"
                t-field="partner.assigned_partner_id.image_128"
                class="d-block"
                t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_128_max"}'
             />
        </div>
        <address class="text-left">
             <div t-field="partner.assigned_partner_id" t-options='{
                 "widget": "contact",
                 "fields": ["address", "website", "phone", "email"]
             }'/>
        </address>
                </div>
            </div>
        </t>
</template>

<template id="references_block" name="Partner References Block">
        <t t-if="any(p.website_published for p in partner.implemented_partner_ids)">
            <h3 id="references">References</h3>
            <div t-foreach="partner.implemented_partner_ids" t-as="reference" class="media mt-3">
              <t t-if="reference.website_published">
                <a t-attf-href="/customers/#{slug(reference)}">
                    <span t-field="reference.image_128" class="d-block mr-3 text-center o_width_128" t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_128_max"}'/>
                </a>
                <div class="media-body" style="min-height: 64px;">
                    <a t-attf-href="/customers/#{slug(reference)}">
                        <span t-field="reference.self"/>
                    </a>
                    <t t-if="reference.industry_id">
                        <span class="badge badge-secondary"><t t-esc="reference.industry_id.name"/></span>
                    </t>
                    <div t-field='reference.website_short_description'/>
                </div>
              </t>
            </div>
        </t>
</template>

<template id="references_block_href" inherit_id="website_crm_partner_assign.references_block" name="Partner References Block">
    <xpath expr="//div/span" position="replace">
        <a t-attf-href="/customers/#{slug(reference)}">$0</a>
    </xpath>
    <xpath expr="//div[hasclass('media-body')]/span" position="replace">
        <a t-attf-href="/customers/#{slug(reference)}">$0</a>
    </xpath>
</template>

</odoo>

```

