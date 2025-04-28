# Odoo Module: website_membership

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
    'name': 'Online Members Directory',
    'category': 'Website/Website',
    'summary': 'Publish your members directory',
    'version': '1.0',
    'description': """
Publish your members/association directory publicly.
    """,
    'depends': ['website_partner', 'website_google_map', 'membership', 'website_sale'],
    'data': [
        'views/product_template_views.xml',
        'views/website_membership_templates.xml',
        'security/ir.model.access.csv',
        'security/website_membership.xml',
        'views/snippets.xml',
    ],
    'demo': ['data/membership_demo.xml'],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug.urls

from odoo import fields

from odoo import http
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import unslug, slug
from odoo.tools.translate import _


class WebsiteMembership(http.Controller):
    _references_per_page = 20

    @http.route([
        '/members',
        '/members/page/<int:page>',
        '/members/association/<membership_id>',
        '/members/association/<membership_id>/page/<int:page>',

        '/members/country/<int:country_id>',
        '/members/country/<country_name>-<int:country_id>',
        '/members/country/<int:country_id>/page/<int:page>',
        '/members/country/<country_name>-<int:country_id>/page/<int:page>',

        '/members/association/<membership_id>/country/<country_name>-<int:country_id>',
        '/members/association/<membership_id>/country/<int:country_id>',
        '/members/association/<membership_id>/country/<country_name>-<int:country_id>/page/<int:page>',
        '/members/association/<membership_id>/country/<int:country_id>/page/<int:page>',
    ], type='http', auth="public", website=True, sitemap=True)
    def members(self, membership_id=None, country_name=None, country_id=0, page=1, **post):
        Product = request.env['product.product']
        Country = request.env['res.country']
        MembershipLine = request.env['membership.membership_line']
        Partner = request.env['res.partner']

        post_name = post.get('search') or post.get('name', '')
        current_country = None
        today = fields.Date.today()

        # base domain for groupby / searches
        base_line_domain = [
            ("partner.website_published", "=", True), ('state', '=', 'paid'),
            ('date_to', '>=', today), ('date_from', '<=', today)
        ]
        if membership_id and membership_id != 'free':
            membership_id = int(membership_id)
            base_line_domain.append(('membership_id', '=', membership_id))

        if post_name:
            base_line_domain += ['|', ('partner.name', 'ilike', post_name), ('partner.website_description', 'ilike', post_name)]

        # group by country, based on all customers (base domain)
        if membership_id != 'free':
            membership_lines = MembershipLine.sudo().search(base_line_domain)
            country_domain = [('member_lines', 'in', membership_lines.ids)]
            if not membership_id:
                country_domain = ['|', country_domain[0], ('membership_state', '=', 'free')]
        else:
            country_domain = [('membership_state', '=', 'free')]
        if post_name:
            country_domain += ['|', ('name', 'ilike', post_name), ('website_description', 'ilike', post_name)]

        countries = Partner.sudo().read_group(country_domain + [("website_published", "=", True)], ["__count"], groupby="country_id")
        countries_total = sum(country_dict['country_id_count'] for country_dict in countries)

        line_domain = list(base_line_domain)
        if country_id:
            line_domain.append(('partner.country_id', '=', country_id))
            current_country = Country.browse(country_id).read(['id', 'name'])[0]
            if not any(x['country_id'][0] == country_id for x in countries if x['country_id']):
                countries.append({
                    'country_id_count': 0,
                    'country_id': (country_id, current_country["name"])
                })
                countries = [d for d in countries if d['country_id']]
                countries.sort(key=lambda d: d['country_id'][1])

        countries.insert(0, {
            'country_id_count': countries_total,
            'country_id': (0, _("All Countries"))
        })

        # format domain for group_by and memberships
        memberships = Product.search([('membership', '=', True)], order="website_sequence")

        # make sure we don't access to lines with unpublished membershipts
        line_domain.append(('membership_id', 'in', memberships.ids))

        limit = self._references_per_page
        offset = limit * (page - 1)

        count_members = 0
        membership_lines = MembershipLine.sudo()
        # displayed non-free membership lines
        if membership_id != 'free':
            count_members = MembershipLine.sudo().search_count(line_domain)
            if offset <= count_members:
                membership_lines = MembershipLine.sudo().search(line_domain, offset, limit)
        page_partner_ids = set(m.partner.id for m in membership_lines)

        # get google maps localization of partners
        google_map_partner_ids = []
        if request.website.is_view_active('website_membership.opt_index_google_map'):
            google_map_partner_ids = MembershipLine.search(line_domain)._get_published_companies(limit=2000)

        search_domain = [('membership_state', '=', 'free'), ('website_published', '=', True)]
        if post_name:
            search_domain += ['|', ('name', 'ilike', post_name), ('website_description', 'ilike', post_name)]
        if country_id:
            search_domain += [('country_id', '=', country_id)]
        free_partners = Partner.sudo().search(search_domain)

        memberships_data = []
        for membership_record in memberships:
            memberships_data.append({'id': membership_record.id, 'name': membership_record.name})

        memberships_partner_ids = {}
        for line in membership_lines:
            memberships_partner_ids.setdefault(line.membership_id.id, []).append(line.partner.id)

        if free_partners:
            memberships_data.append({'id': 'free', 'name': _('Free Members')})
            if not membership_id or membership_id == 'free':
                if count_members < offset + limit:
                    free_start = max(offset - count_members, 0)
                    free_end = max(offset + limit - count_members, 0)
                    memberships_partner_ids['free'] = free_partners.ids[free_start:free_end]
                    page_partner_ids |= set(memberships_partner_ids['free'])
                google_map_partner_ids += free_partners.ids[:2000-len(google_map_partner_ids)]
                count_members += len(free_partners)

        google_map_partner_ids = ",".join(str(it) for it in google_map_partner_ids)
        google_maps_api_key = request.website.google_maps_api_key

        partners = {p.id: p for p in Partner.sudo().browse(list(page_partner_ids))}

        base_url = '/members%s%s' % ('/association/%s' % membership_id if membership_id else '',
                                     '/country/%s' % country_id if country_id else '')

        # request pager for lines
        pager = request.website.pager(url=base_url, total=count_members, page=page, step=limit, scope=7, url_args=post)

        values = {
            'partners': partners,
            'memberships_data': memberships_data,
            'memberships_partner_ids': memberships_partner_ids,
            'membership_id': membership_id,
            'countries': countries,
            'current_country': current_country and [current_country['id'], current_country['name']] or None,
            'current_country_id': current_country and current_country['id'] or 0,
            'google_map_partner_ids': google_map_partner_ids,
            'pager': pager,
            'post': post,
            'search': "?%s" % werkzeug.urls.url_encode(post),
            'search_count': count_members,
            'google_maps_api_key': google_maps_api_key,
        }
        return request.render("website_membership.index", values)

    # Do not use semantic controller due to SUPERUSER_ID
    @http.route(['/members/<partner_id>'], type='http', auth="public", website=True)
    def partners_detail(self, partner_id, **post):
        current_slug = partner_id
        _, partner_id = unslug(partner_id)
        if partner_id:
            partner = request.env['res.partner'].sudo().browse(partner_id)
            if partner.exists() and partner.website_published:  # TODO should be done with access rules
                if slug(partner) != current_slug:
                    return request.redirect('/members/%s' % slug(partner))
                values = {}
                values['main_object'] = values['partner'] = partner
                return request.render("website_membership.partner", values)
        raise request.not_found()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\membership_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <data noupdate="1">
        <record id="base.res_partner_12" model="res.partner">
            <field name="is_published" eval="True"/>
        </record>
        <record id="base.res_partner_2" model="res.partner">
            <field name="is_published" eval="True"/>
        </record>
    </data>

</odoo>

```

## File: models\membership.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class MembershipLine(models.Model):

    _inherit = 'membership.membership_line'

    def _get_published_companies(self, limit=None):
        if not self.ids:
            return []
        limit_clause = '' if limit is None else ' LIMIT %d' % limit
        self.env.cr.execute("""
            SELECT DISTINCT p.id
            FROM res_partner p INNER JOIN membership_membership_line m
            ON  p.id = m.partner
            WHERE is_published AND is_company AND m.id IN %s """ + limit_clause, (tuple(self.ids),))
        return [partner_id[0] for partner_id in self.env.cr.fetchall()]

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
        suggested_controllers.append((_('Members'), url_for('/members'), 'website_membership'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import membership
from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_membership_product_product_public,event.product.product.public,product.model_product_product,base.group_public,1,0,0,0
access_membership_membership_line_public,membership.membership_line,model_membership_membership_line,base.group_public,1,0,0,0
access_membership_membership_line_portal,membership.membership_line,model_membership_membership_line,base.group_portal,1,0,0,0

```

## File: security\website_membership.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="membership_product_product_public" model="ir.rule">
        <field name="name">Product membership: Public</field>
        <field name="model_id" ref="product.model_product_template"/>
        <field name="domain_force">[('website_published', '=', True), ('product_variant_ids.membership', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="membership_membership_line_public" model="ir.rule">
        <field name="name">Membership line: Public</field>
        <field name="model_id" ref="membership.model_membership_membership_line"/>
        <field name="domain_force">[('partner.website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43 24a5 5 0 1 1-10 0 5 5 0 0 1 10 0Z" fill="#FBB945"/><path d="M28 16c0 5.523-4.477 10-10 10S8 21.523 8 16 12.477 6 18 6s10 4.477 10 10Z" fill="#985184"/><path d="M16 31h26a4 4 0 0 1 4 4v5a4 4 0 0 1-4 4H16V31Z" fill="#FBB945"/><path d="M4 31h16c7.18 0 13 5.82 13 13H17C9.82 44 4 38.18 4 31Z" fill="#985184"/></svg>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_view_form" model="ir.ui.view">
        <field name="name">product.template.view.form.inherit.website_membership</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="membership.membership_products_form"/>
        <field name="arch" type="xml">
            <field name="active" position="after">
                <field name="website_published"/>
                <field name="website_sequence" groups="base.group_no_one"/>
            </field>
        </field>
    </record>

    <record id="product_template_view_tree" model="ir.ui.view">
        <field name="name">product.template.view.tree.inherit.website_membership</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="membership.membership_products_tree"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="website_sequence" widget="handle"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Membership Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(#oe_structure_website_membership_index_1)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Members Page">
            <we-checkbox string="Location"
                         data-customize-website-views="website_membership.opt_index_country"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="World Map"
                         data-customize-website-views="website_membership.opt_index_google_map"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_membership_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="index" name="Members">
    <t t-call="website.layout">
        <t t-set="additional_title">Members</t>
        <div id="wrap">
            <div class="oe_structure">
                <section>
                    <div class="container">
                        <div class="row">
                            <div class="col-lg-12">
                                <h1 class="text-center">Our Members Directory</h1>
                                <h3 class="text-muted text-center">Find a business partner</h3>
                            </div>
                        </div>
                    </div>
                </section>
            </div>
            <div class="container">
                <div class="row">

            <div class="col-lg-3 mb32" id="left_column">
                <ul class="nav nav-pills flex-column mt16">
                    <li class="nav-header nav-item"><h3>Associations</h3></li>
                    <li class="nav-item"><a href="/members" class="nav-link#{'' if membership_id else ' active'}">All</a></li>
                    <t t-foreach="memberships_data" t-as="membership_data">
                        <li class="nav-item">
                            <a t-attf-href="/members/association/#{ membership_data['id'] }#{current_country and '/country/%s' % slug(current_country) or ''}#{ search }"
                                t-attf-class="nav-link#{membership_id and membership_data['id'] == membership_id and ' active' or ''}"><t t-esc="membership_data['name']"/></a>
                        </li>
                    </t>
                </ul>
            </div>
            <div class="col-lg-8" id="ref_content">
                <div class='d-flex m-2'>
                    <t t-call="website.pager">
                       <t t-set="classname" t-valuef="float-start"/>
                    </t>
                    <form action="" method="get" class="navbar-search ms-auto pagination">
                        <t t-call="website.website_search_box">
                            <t t-set="search" t-value="post.get('search', '')"/>
                        </t>
                    </form>
                </div>
                <div>
                    <t t-if="not memberships_partner_ids">
                        <p>No result found.</p>
                    </t>
                    <t t-foreach="memberships_data" t-as="membership_data">
                        <t t-if="memberships_partner_ids.get(membership_data['id'])">
                            <h3 class="text-center"><span t-esc="membership_data['name']"/></h3>
                            <t t-foreach="memberships_partner_ids[membership_data['id']]" t-as="partner_id">
                                <t t-set="partner" t-value="partners[partner_id]"/>
                                <div class="d-flex mt-3">
                                    <a t-attf-href="/members/#{slug(partner)}"
                                       t-field="partner.avatar_128"
                                       t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_64_cover me-3"}'
                                    ></a>
                                    <div class="flex-grow-1" style="min-height: 64px;">
                                        <a t-attf-href="/members/#{slug(partner)}">
                                            <span t-field="partner.display_name"/>
                                        </a>
                                        <div t-field="partner.website_short_description"/>
                                    </div>
                                </div>
                            </t>
                        </t>
                    </t>
                </div>
            </div>

                </div>
            </div>
            <div class="oe_structure" id="oe_structure_website_membership_index_1"/>
        </div>
    </t>
</template>

<template id="opt_index_country" name="Location"
        inherit_id="website_membership.index">
    <xpath expr="//div[@id='left_column']/ul[1]" position="after">
        <ul class="nav nav-pills flex-column mt16">
            <li class="nav-header nav-item"><h3>Location</h3></li>
            <t t-foreach="countries" t-as="country">
                <li t-if="country['country_id']" class="nav-item">
                    <a t-attf-href="/members#{ membership_id and '/association/%s' % membership_id or '' }#{ country['country_id'][0] and '/country/%s' % slug(country['country_id']) or '' }#{ search }"
                        t-attf-class="nav-link#{country['country_id'] and country['country_id'][0] == current_country_id and ' active' or ''}"><t t-esc="country['country_id'][1]"/>
                        <span class="badge rounded-pill float-end"><t t-esc="country['country_id_count'] or '0'"/></span>
                    </a>
                </li>
            </t>
        </ul>
    </xpath>
</template>

<!-- Option: index: Left Google Map -->
<template id="opt_index_google_map" name="Left World Map"
        inherit_id="website_membership.index">
    <xpath expr="//div[@id='left_column']/ul[last()]" position="after">
        <t t-if="google_maps_api_key">
            <!-- modal for large map -->
            <div role="dialog" class="modal fade partner_map_modal" tabindex="-1">
              <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">World Map</h4>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </header>
                    <iframe t-attf-src="/google_map/?width=898&amp;height=485&amp;partner_ids=#{ google_map_partner_ids }&amp;partner_url=/members/"
                    style="width:898px; height:485px; border:0; padding:0; margin:0;"></iframe>
                </div>
              </div>
            </div>
            <!-- modal end -->
            <h3>World Map<button class="btn btn-link" data-bs-toggle="modal" data-bs-target=".partner_map_modal"><span class="fa fa-external-link" role="img" aria-label="External link" title="External link"/></button></h3>
            <ul class="nav">
                <iframe t-attf-src="/google_map/?width=260&amp;height=240&amp;partner_ids=#{ google_map_partner_ids }&amp;partner_url=/members/"
                    style="width:260px; height:240px; border:0; padding:0; margin:0;"></iframe>
            </ul>
        </t>
    </xpath>
</template>

<template id="partner" name="Members">
    <t t-call="website.layout">
        <div id="wrap">
            <div class="oe_structure" id="oe_structure_website_membership_partner_1"/>
            <div class="container">
                <div class="row">
                    <t t-call="website_partner.partner_detail"/>
                </div>
            </div>
            <div class="oe_structure" id="oe_structure_website_membership_partner_2"/>
        </div>
    </t>
</template>

</odoo>

```

