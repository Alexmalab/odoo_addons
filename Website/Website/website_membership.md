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
    'depends': ['website_partner', 'website_google_map', 'association', 'website_sale'],
    'data': [
        'data/membership_data.xml',
        'views/product_template_views.xml',
        'views/website_membership_templates.xml',
        'security/ir.model.access.csv',
        'security/website_membership.xml',
    ],
    'demo': ['data/membership_demo.xml'],
    'qweb': ['static/src/xml/*.xml'],
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
from odoo.addons.http_routing.models.ir_http import unslug
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
    ], type='http', auth="public", website=True)
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

        countries = Partner.sudo().read_group(country_domain + [("website_published", "=", True)], ["id", "country_id"], groupby="country_id", orderby="country_id")
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
        if request.website.viewref('website_membership.opt_index_google_map').active:
            google_map_partner_ids = MembershipLine.search(line_domain).get_published_companies(limit=2000)

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
            'search': "?%s" % werkzeug.url_encode(post),
            'search_count': count_members,
            'google_maps_api_key': google_maps_api_key,
        }
        return request.render("website_membership.index", values)

    # Do not use semantic controller due to SUPERUSER_ID
    @http.route(['/members/<partner_id>'], type='http', auth="public", website=True)
    def partners_detail(self, partner_id, **post):
        _, partner_id = unslug(partner_id)
        if partner_id:
            partner = request.env['res.partner'].sudo().browse(partner_id)
            if partner.exists() and partner.website_published:  # TODO should be done with access rules
                values = {}
                values['main_object'] = values['partner'] = partner
                return request.render("website_membership.partner", values)
        return self.members(**post)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\membership_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <data noupdate="1">
        <record id="menu_members" model="website.menu">
            <field name="name">Members</field>
            <field name="url">/members</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence">55</field>
        </record>
    </data>

</odoo>

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

    def get_published_companies(self, limit=None):
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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import membership

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_membership_product_product_public,event.product.product.public,product.model_product_product,base.group_public,1,0,0,0
```

## File: security\website_membership.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CD7690"/><stop offset="100%" stop-color="#CA5377"/></linearGradient><filter id="d" width="103.8%" height="108%" x="-1.9%" y="-2%" filterUnits="objectBoundingBox"><feOffset dy="2" in="SourceAlpha" result="shadowOffsetOuter1"/><feColorMatrix in="shadowOffsetOuter1" result="shadowMatrixOuter1" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.253906 0"/><feMerge><feMergeNode in="shadowMatrixOuter1"/><feMergeNode in="SourceGraphic"/></feMerge></filter></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M48 69H4c-2 0-4-.146-4-4.082V42.266l18.588-19.97C26.037 16.512 34.84 15.413 45 19c6.027 3.156 9.921 9.21 11.683 18.161l-3.95 4.878L61 53.694 48 69z" opacity=".324"/><path d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z" fill="#000" fill-opacity=".383"/></g><g filter="url(#d)" mask="url(#b)"><g transform="translate(10 10)"><path d="M0 0L50 0 50 50 0 50z"/><path fill="#FFF" d="M46.788 27.213C40.83 28.398 38.916 22 34 22c-1.414 0-2.704.378-3.922.927a.726.726 0 0 0 .08-.177c.046-.149.116-.242.21-.279.167.223.362.242.585.056.13-.149.14-.298.028-.447.093-.13.284-.228.572-.293.288-.065.46-.153.516-.265.13.038.205.019.223-.055.019-.075.028-.186.028-.335 0-.15.028-.26.084-.335.074-.093.214-.177.419-.251.204-.075.325-.121.362-.14l.475-.307c.055-.074.055-.111 0-.111a1.01 1.01 0 0 0 .865-.307c.186-.205.13-.39-.168-.558.056-.112.028-.2-.084-.265a1.46 1.46 0 0 0-.418-.154c.056-.018.163-.023.32-.014.159.01.257-.004.294-.042.279-.186.214-.334-.196-.446-.316-.093-.716.019-1.2.335-.037.018-.125.107-.264.265-.14.158-.265.246-.377.265.037 0 .079-.047.125-.14.047-.093.093-.195.14-.306a.856.856 0 0 1 .098-.196c.111-.13.316-.27.613-.418.26-.112.745-.224 1.451-.335.633-.149 1.107-.047 1.423.307-.037-.037.052-.158.265-.363.214-.205.35-.316.405-.335.056-.037.195-.079.419-.125.223-.047.362-.117.418-.21l.056-.613c-.223.018-.386-.047-.488-.196-.103-.149-.163-.344-.182-.586 0 .038-.056.112-.167.224 0-.13-.042-.205-.126-.224a.671.671 0 0 0-.32.028c-.13.037-.215.047-.252.028a1.095 1.095 0 0 1-.418-.21c-.093-.083-.168-.236-.223-.46a7.936 7.936 0 0 0-.112-.418c-.037-.093-.126-.19-.265-.293-.14-.102-.228-.2-.265-.293a3.026 3.026 0 0 1-.07-.153 3.41 3.41 0 0 0-.084-.182.589.589 0 0 0-.111-.153.213.213 0 0 0-.154-.07c-.056 0-.12.047-.195.14a5.451 5.451 0 0 0-.21.279c-.064.093-.106.139-.125.139a.228.228 0 0 0-.167-.042.89.89 0 0 0-.126.028.47.47 0 0 0-.125.084.78.78 0 0 1-.14.097.637.637 0 0 1-.237.084 1.824 1.824 0 0 0-.237.056c.279-.093.27-.195-.028-.307-.186-.074-.335-.102-.447-.084.168-.074.238-.186.21-.334a.671.671 0 0 0-.237-.391h.139c-.019-.075-.098-.154-.237-.237a3.42 3.42 0 0 0-.488-.237 2.402 2.402 0 0 1-.363-.168c-.149-.093-.465-.181-.949-.265-.483-.084-.79-.088-.92-.014-.094.112-.135.21-.126.293.01.084.046.214.111.39.066.177.098.294.098.35.019.111-.032.232-.153.362-.121.13-.182.242-.182.335 0 .13.13.274.391.433.26.158.353.358.279.6-.056.148-.205.297-.446.446-.242.149-.391.26-.447.335-.093.148-.107.32-.042.516.065.195.163.349.293.46.037.037.051.075.042.112-.01.037-.042.079-.098.125a1.249 1.249 0 0 1-.153.112c-.047.028-.107.06-.181.098l-.084.055c-.205.093-.395.038-.572-.167a1.804 1.804 0 0 1-.377-.725c-.13-.465-.279-.744-.446-.837-.428-.15-.698-.14-.81.027-.092-.241-.474-.483-1.143-.725-.465-.167-1.005-.205-1.619-.112.112-.018.112-.158 0-.418-.13-.28-.307-.39-.53-.335a1.4 1.4 0 0 0 .112-.488c.018-.214.028-.34.028-.377.056-.242.167-.456.335-.642a5.35 5.35 0 0 0 .46-.614c.065-.111.07-.167.014-.167.651.074 1.116-.028 1.395-.307.093-.093.2-.25.32-.474.122-.223.22-.381.294-.474.167-.112.297-.163.39-.154.093.01.228.06.405.154.177.093.312.139.405.139.26.019.404-.084.432-.307a.606.606 0 0 0-.21-.558c.224.019.252-.14.085-.474a1.065 1.065 0 0 0-.224-.251c-.223-.075-.474-.028-.753.14-.149.074-.13.148.056.222-.019-.018-.107.08-.265.293a2.36 2.36 0 0 1-.46.489c-.15.111-.298.065-.447-.14-.019-.019-.07-.144-.154-.377-.083-.232-.172-.358-.265-.376-.148 0-.297.14-.446.418.056-.149-.047-.288-.307-.418s-.484-.205-.67-.223c.354-.224.28-.475-.223-.754-.13-.074-.32-.12-.572-.14-.25-.018-.432.02-.544.112-.093.13-.144.237-.153.321-.01.084.037.158.14.223.101.065.2.117.292.154.093.037.2.074.321.111.12.038.2.066.237.084.26.186.335.316.223.39a3.374 3.374 0 0 1-.237.098l-.32.126c-.094.037-.15.074-.168.112-.056.074-.056.204 0 .39.056.186.037.316-.056.39-.093-.092-.177-.255-.251-.487-.074-.233-.14-.386-.195-.46.13.167-.103.222-.698.167l-.279-.028c-.074 0-.223.018-.446.055a2.073 2.073 0 0 1-.572.028.519.519 0 0 1-.377-.223c-.074-.149-.074-.335 0-.558.019-.074.056-.093.112-.056a4.18 4.18 0 0 1-.307-.265c-.13-.12-.223-.2-.28-.237-.855.28-1.73.66-2.622 1.144.112.019.223.01.335-.028.093-.037.214-.097.362-.181a9.41 9.41 0 0 1 .28-.154c.632-.26 1.023-.325 1.171-.195l.14-.14c.26.298.446.53.558.698-.13-.074-.41-.084-.837-.028-.372.112-.577.223-.614.335.13.223.177.39.14.502a4.358 4.358 0 0 1-.321-.279 2.213 2.213 0 0 0-.405-.307c-.13-.074-.27-.12-.418-.14-.298 0-.503.01-.614.029-2.716 1.488-4.902 3.552-6.557 6.194.13.13.242.204.335.223.074.019.12.102.14.251.018.149.041.251.069.307.028.056.135.028.32-.084.168.15.196.326.084.53.02-.018.428.233 1.228.754.354.316.549.511.586.586.056.204-.037.372-.279.502-.019-.037-.102-.12-.251-.251-.149-.13-.233-.167-.251-.112-.056.093-.051.265.014.517.065.25.163.367.293.348-.13 0-.219.15-.265.447-.047.297-.07.628-.07.99 0 .363-.01.582-.028.656l.056.028c-.056.223-.005.544.153.962.158.419.358.6.6.545-.242.055-.056.455.558 1.2.112.148.186.232.223.25.056.038.168.107.335.21.168.102.307.195.419.279.111.083.204.181.279.293.074.093.167.302.279.627.111.326.242.545.39.656-.037.112.052.298.266.558.213.26.311.475.292.642a.136.136 0 0 0-.07.028.136.136 0 0 1-.069.028c.056.13.2.26.432.39.233.13.377.251.433.363.019.056.037.149.056.28.018.13.046.232.083.306.038.074.112.093.224.056.037-.372-.186-.949-.67-1.73a22.924 22.924 0 0 1-.474-.81 1.604 1.604 0 0 1-.154-.432 2.128 2.128 0 0 0-.125-.404c.037 0 .093.014.167.042.074.027.154.06.237.097.084.037.154.075.21.112.055.037.074.065.055.084-.056.13-.037.292.056.488.093.195.205.367.335.516a38.267 38.267 0 0 0 .81.893c.11.111.24.293.39.544.149.251.149.377 0 .377.069 0 .14.015.216.047C17.53 30.278 17 31.865 17 34c0 2-2 4.336-2 5v5.17a21.332 21.332 0 0 1-8.126-7.986C4.958 32.903 4 29.317 4 25.43c0-3.888.958-7.473 2.874-10.757a21.332 21.332 0 0 1 7.798-7.798C17.956 4.958 21.541 4 25.43 4c3.887 0 7.473.958 10.756 2.874a21.332 21.332 0 0 1 7.798 7.798c1.916 3.284 2.874 6.869 2.874 10.757 0 .602-.023 1.196-.069 1.784zM34.5 25.25a5.605 5.605 0 1 1 0 11.21 5.605 5.605 0 0 1 0-11.21zm6.057 11.604l-2.134-.533c-2.625 1.888-5.807 1.466-7.846 0l-2.134.533a3.844 3.844 0 0 0-2.912 3.73v3.244c0 1.062.86 1.922 1.922 1.922h14.094c1.061 0 1.922-.86 1.922-1.922v-3.245c0-1.764-1.2-3.301-2.912-3.729zm5.047.995a3.737 3.737 0 1 0 0-7.474 3.737 3.737 0 0 0 0 7.474zm-22.208 0a3.737 3.737 0 1 0 0-7.474 3.737 3.737 0 0 0 0 7.474zm1.281 5.98v-3.246c0-.883.245-1.72.678-2.434-1.6.786-3.381.465-4.574-.394l-1.423.356a2.562 2.562 0 0 0-1.941 2.486v2.163c0 .708.573 1.282 1.28 1.282h5.99a2.798 2.798 0 0 1-.01-.214zm24.965-5.718l-1.423-.356c-1.49 1.073-3.25 1.027-4.58.386a4.69 4.69 0 0 1 .684 2.442v3.245c0 .072-.004.143-.01.214h5.99c.707 0 1.28-.574 1.28-1.282v-2.163c0-1.176-.8-2.2-1.94-2.486z"/></g></g></g></svg>
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
                       <t t-set="classname">float-left</t>
                    </t>
                    <form action="" method="get" class="navbar-search ml-auto pagination form-inline">
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
                                <div class="media mt-3">
                                    <a t-attf-href="/members/#{slug(partner)}"
                                       t-field="partner.image_128"
                                       t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_64_cover mr-3"}'
                                    ></a>
                                    <div class="media-body" style="min-height: 64px;">
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
        customize_show="True" inherit_id="website_membership.index">
    <xpath expr="//div[@id='left_column']/ul[1]" position="after">
        <ul class="nav nav-pills flex-column mt16">
            <li class="nav-header nav-item"><h3>Location</h3></li>
            <t t-foreach="countries" t-as="country">
                <li t-if="country['country_id']" class="nav-item">
                    <a t-attf-href="/members#{ membership_id and '/association/%s' % membership_id or '' }#{ country['country_id'][0] and '/country/%s' % slug(country['country_id']) or '' }#{ search }"
                        t-attf-class="nav-link#{country['country_id'] and country['country_id'][0] == current_country_id and ' active' or ''}"><t t-esc="country['country_id'][1]"/>
                        <span class="badge badge-pill float-right"><t t-esc="country['country_id_count'] or '0'"/></span>
                    </a>
                </li>
            </t>
        </ul>
    </xpath>
</template>

<!-- Option: index: Left Google Map -->
<template id="opt_index_google_map" name="Left World Map"
        customize_show="True" inherit_id="website_membership.index">
    <xpath expr="//div[@id='left_column']/ul[last()]" position="after">
        <t t-if="google_maps_api_key">
            <!-- modal for large map -->
            <div role="dialog" class="modal fade partner_map_modal" tabindex="-1">
              <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">World Map</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                    </header>
                    <iframe t-attf-src="/google_map/?width=898&amp;height=485&amp;partner_ids=#{ google_map_partner_ids }&amp;partner_url=/members/"
                    style="width:898px; height:485px; border:0; padding:0; margin:0;"></iframe>
                </div>
              </div>
            </div>
            <!-- modal end -->
            <h3>World Map<button class="btn btn-link" data-toggle="modal" data-target=".partner_map_modal"><span class="fa fa-external-link" role="img" aria-label="External link" title="External link"/></button></h3>
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

