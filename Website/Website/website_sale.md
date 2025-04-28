# Odoo Module: website_sale

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID, _
from . import controllers
from . import models
from . import wizard
from . import report


def uninstall_hook(cr, registry):
    """ Need to reenable the `product` pricelist multi-company rule that were
        disabled to be 'overridden' for multi-website purpose
    """
    env = api.Environment(cr, SUPERUSER_ID, {})
    pl_rule = env.ref('product.product_pricelist_comp_rule', raise_if_not_found=False)
    pl_item_rule = env.ref('product.product_pricelist_item_comp_rule', raise_if_not_found=False)
    multi_company_rules = pl_rule or env['ir.rule']
    multi_company_rules += pl_item_rule or env['ir.rule']
    multi_company_rules.write({'active': True})


def post_init_hook(cr, registry):
    """ Need to add product filters to previously created website """
    # Do the same as in _bootstrap_snippet_filters()
    env = api.Environment(cr, SUPERUSER_ID, {})
    filter = env.ref('website_sale.dynamic_filter_demo_products', raise_if_not_found=False)
    if filter:
        for website in env['website'].search([('id', '!=', filter.website_id.id)]):
            filter.copy({'website_id': website.id})

```

## File: __manifest__.py

```python
{
    'name': 'eCommerce',
    'category': 'Website/Website',
    'sequence': 50,
    'summary': 'Sell your products online',
    'website': 'https://www.odoo.com/page/e-commerce',
    'version': '1.0',
    'description': "",
    'depends': ['website', 'sale', 'website_payment', 'website_mail', 'website_form', 'portal_rating', 'digest'],
    'data': [
        'security/ir.model.access.csv',
        'security/website_sale.xml',
        'data/data.xml',
        'data/mail_template_data.xml',
        'data/digest_data.xml',
        'views/product_views.xml',
        'views/account_views.xml',
        'views/onboarding_views.xml',
        'views/sale_report_views.xml',
        'views/sale_order_views.xml',
        'views/crm_team_views.xml',
        'views/templates.xml',
        'views/snippets/snippets.xml',
        'views/snippets/s_dynamic_snippet_products.xml',
        'views/snippets/s_products_searchbar.xml',
        'views/res_config_settings_views.xml',
        'views/digest_views.xml',
        'views/website_sale_visitor_views.xml',
    ],
    'demo': [
        'data/demo.xml',
    ],
    'qweb': ['static/src/xml/*.xml'],
    'installable': True,
    'application': True,
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\backend.py

```python
# -*- coding: utf-8 -*-

import babel.dates

from datetime import datetime, timedelta, time

from odoo import fields, http, _
from odoo.addons.website.controllers.backend import WebsiteBackend
from odoo.http import request
from odoo.tools.misc import get_lang


class WebsiteSaleBackend(WebsiteBackend):

    @http.route()
    def fetch_dashboard_data(self, website_id, date_from, date_to):
        Website = request.env['website']
        current_website = website_id and Website.browse(website_id) or Website.get_current_website()

        results = super(WebsiteSaleBackend, self).fetch_dashboard_data(website_id, date_from, date_to)

        date_date_from = fields.Date.from_string(date_from)
        date_date_to = fields.Date.from_string(date_to)
        date_diff_days = (date_date_to - date_date_from).days
        datetime_from = datetime.combine(date_date_from, time.min)
        datetime_to = datetime.combine(date_date_to, time.max)

        sales_values = dict(
            graph=[],
            best_sellers=[],
            summary=dict(
                order_count=0, order_carts_count=0, order_unpaid_count=0,
                order_to_invoice_count=0, order_carts_abandoned_count=0,
                payment_to_capture_count=0, total_sold=0,
                order_per_day_ratio=0, order_sold_ratio=0, order_convertion_pctg=0,
            )
        )

        results['dashboards']['sales'] = sales_values

        results['groups']['sale_salesman'] = request.env['res.users'].has_group('sales_team.group_sale_salesman')

        if not results['groups']['sale_salesman']:
            return results

        results['dashboards']['sales']['utm_graph'] = self.fetch_utm_data(datetime_from, datetime_to)
        # Product-based computation
        sale_report_domain = [
            ('website_id', '=', current_website.id),
            ('state', 'in', ['sale', 'done']),
            ('date', '>=', datetime_from),
            ('date', '<=', fields.Datetime.now())
        ]
        report_product_lines = request.env['sale.report'].read_group(
            domain=sale_report_domain,
            fields=['product_tmpl_id', 'product_uom_qty', 'price_subtotal'],
            groupby='product_tmpl_id', orderby='product_uom_qty desc', limit=5)
        for product_line in report_product_lines:
            product_tmpl_id = request.env['product.template'].browse(product_line['product_tmpl_id'][0])
            sales_values['best_sellers'].append({
                'id': product_tmpl_id.id,
                'name': product_tmpl_id.name,
                'qty': product_line['product_uom_qty'],
                'sales': product_line['price_subtotal'],
            })

        # Sale-based results computation
        sale_order_domain = [
            ('website_id', '=', current_website.id),
            ('date_order', '>=', fields.Datetime.to_string(datetime_from)),
            ('date_order', '<=', fields.Datetime.to_string(datetime_to))]
        so_group_data = request.env['sale.order'].read_group(sale_order_domain, fields=['state'], groupby='state')
        for res in so_group_data:
            if res.get('state') == 'sent':
                sales_values['summary']['order_unpaid_count'] += res['state_count']
            elif res.get('state') in ['sale', 'done']:
                sales_values['summary']['order_count'] += res['state_count']
            sales_values['summary']['order_carts_count'] += res['state_count']

        report_price_lines = request.env['sale.report'].read_group(
            domain=[
                ('website_id', '=', current_website.id),
                ('state', 'in', ['sale', 'done']),
                ('date', '>=', datetime_from),
                ('date', '<=', datetime_to)],
            fields=['team_id', 'price_subtotal'],
            groupby=['team_id'],
        )
        sales_values['summary'].update(
            order_to_invoice_count=request.env['sale.order'].search_count(sale_order_domain + [
                ('state', 'in', ['sale', 'done']),
                ('order_line', '!=', False),
                ('partner_id', '!=', request.env.ref('base.public_partner').id),
                ('invoice_status', '=', 'to invoice'),
            ]),
            order_carts_abandoned_count=request.env['sale.order'].search_count(sale_order_domain + [
                ('is_abandoned_cart', '=', True),
                ('cart_recovery_email_sent', '=', False)
            ]),
            payment_to_capture_count=request.env['payment.transaction'].search_count([
                ('state', '=', 'authorized'),
                # that part perform a search on sale.order in order to comply with access rights as tx do not have any
                ('sale_order_ids', 'in', request.env['sale.order'].search(sale_order_domain + [('state', '!=', 'cancel')]).ids),
            ]),
            total_sold=sum(price_line['price_subtotal'] for price_line in report_price_lines)
        )

        # Ratio computation
        sales_values['summary']['order_per_day_ratio'] = round(float(sales_values['summary']['order_count']) / date_diff_days, 2)
        sales_values['summary']['order_sold_ratio'] = round(float(sales_values['summary']['total_sold']) / sales_values['summary']['order_count'], 2) if sales_values['summary']['order_count'] else 0
        sales_values['summary']['order_convertion_pctg'] = 100.0 * sales_values['summary']['order_count'] / sales_values['summary']['order_carts_count'] if sales_values['summary']['order_carts_count'] else 0

        # Graphes computation
        if date_diff_days == 7:
            previous_sale_label = _('Previous Week')
        elif date_diff_days > 7 and date_diff_days <= 31:
            previous_sale_label = _('Previous Month')
        else:
            previous_sale_label = _('Previous Year')

        sales_values['graph'] += [{
            'values': self._compute_sale_graph(date_date_from, date_date_to, sale_report_domain),
            'key': 'Untaxed Total',
        }, {
            'values': self._compute_sale_graph(date_date_from - timedelta(days=date_diff_days), date_date_from, sale_report_domain, previous=True),
            'key': previous_sale_label,
        }]

        return results

    def fetch_utm_data(self, date_from, date_to):
        sale_utm_domain = [
            ('website_id', '!=', False),
            ('state', 'in', ['sale', 'done']),
            ('date_order', '>=', date_from),
            ('date_order', '<=', date_to)
        ]

        orders_data_groupby_campaign_id = request.env['sale.order'].read_group(
            domain=sale_utm_domain + [('campaign_id', '!=', False)],
            fields=['amount_total', 'id', 'campaign_id'],
            groupby='campaign_id')

        orders_data_groupby_medium_id = request.env['sale.order'].read_group(
            domain=sale_utm_domain + [('medium_id', '!=', False)],
            fields=['amount_total', 'id', 'medium_id'],
            groupby='medium_id')

        orders_data_groupby_source_id = request.env['sale.order'].read_group(
            domain=sale_utm_domain + [('source_id', '!=', False)],
            fields=['amount_total', 'id', 'source_id'],
            groupby='source_id')

        return {
            'campaign_id': self.compute_utm_graph_data('campaign_id', orders_data_groupby_campaign_id),
            'medium_id': self.compute_utm_graph_data('medium_id', orders_data_groupby_medium_id),
            'source_id': self.compute_utm_graph_data('source_id', orders_data_groupby_source_id),
        }

    def compute_utm_graph_data(self, utm_type, utm_graph_data):
        return [{
            'utm_type': data[utm_type][1],
            'amount_total': data['amount_total']
        } for data in utm_graph_data]

    def _compute_sale_graph(self, date_from, date_to, sales_domain, previous=False):
        days_between = (date_to - date_from).days
        date_list = [(date_from + timedelta(days=x)) for x in range(0, days_between + 1)]

        daily_sales = request.env['sale.report'].read_group(
            domain=sales_domain,
            fields=['date', 'price_subtotal'],
            groupby='date:day')

        daily_sales_dict = {p['date:day']: p['price_subtotal'] for p in daily_sales}

        sales_graph = [{
            '0': fields.Date.to_string(d) if not previous else fields.Date.to_string(d + timedelta(days=days_between)),
            # Respect read_group format in models.py
            '1': daily_sales_dict.get(babel.dates.format_date(d, format='dd MMM yyyy', locale=get_lang(request.env).code), 0)
        } for d in date_list]

        return sales_graph

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json
import logging
from datetime import datetime
from werkzeug.exceptions import Forbidden, NotFound

from odoo import fields, http, SUPERUSER_ID, tools, _
from odoo.http import request
from odoo.addons.base.models.ir_qweb_fields import nl2br
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.payment.controllers.portal import PaymentProcessing
from odoo.addons.website.controllers.main import QueryURL
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.exceptions import ValidationError
from odoo.addons.portal.controllers.portal import _build_url_w_params
from odoo.addons.website.controllers.main import Website
from odoo.addons.website_form.controllers.main import WebsiteForm
from odoo.osv import expression
_logger = logging.getLogger(__name__)


class TableCompute(object):

    def __init__(self):
        self.table = {}

    def _check_place(self, posx, posy, sizex, sizey, ppr):
        res = True
        for y in range(sizey):
            for x in range(sizex):
                if posx + x >= ppr:
                    res = False
                    break
                row = self.table.setdefault(posy + y, {})
                if row.setdefault(posx + x) is not None:
                    res = False
                    break
            for x in range(ppr):
                self.table[posy + y].setdefault(x, None)
        return res

    def process(self, products, ppg=20, ppr=4):
        # Compute products positions on the grid
        minpos = 0
        index = 0
        maxy = 0
        x = 0
        for p in products:
            x = min(max(p.website_size_x, 1), ppr)
            y = min(max(p.website_size_y, 1), ppr)
            if index >= ppg:
                x = y = 1

            pos = minpos
            while not self._check_place(pos % ppr, pos // ppr, x, y, ppr):
                pos += 1
            # if 21st products (index 20) and the last line is full (ppr products in it), break
            # (pos + 1.0) / ppr is the line where the product would be inserted
            # maxy is the number of existing lines
            # + 1.0 is because pos begins at 0, thus pos 20 is actually the 21st block
            # and to force python to not round the division operation
            if index >= ppg and ((pos + 1.0) // ppr) > maxy:
                break

            if x == 1 and y == 1:   # simple heuristic for CPU optimization
                minpos = pos // ppr

            for y2 in range(y):
                for x2 in range(x):
                    self.table[(pos // ppr) + y2][(pos % ppr) + x2] = False
            self.table[pos // ppr][pos % ppr] = {
                'product': p, 'x': x, 'y': y,
                'ribbon': p.website_ribbon_id,
            }
            if index <= ppg:
                maxy = max(maxy, y + (pos // ppr))
            index += 1

        # Format table according to HTML needs
        rows = sorted(self.table.items())
        rows = [r[1] for r in rows]
        for col in range(len(rows)):
            cols = sorted(rows[col].items())
            x += len(cols)
            rows[col] = [r[1] for r in cols if r[1]]

        return rows


class WebsiteSaleForm(WebsiteForm):

    @http.route('/website_form/shop.sale.order', type='http', auth="public", methods=['POST'], website=True)
    def website_form_saleorder(self, **kwargs):
        model_record = request.env.ref('sale.model_sale_order')
        try:
            data = self.extract_data(model_record, kwargs)
        except ValidationError as e:
            return json.dumps({'error_fields': e.args[0]})

        order = request.website.sale_get_order()
        if data['record']:
            order.write(data['record'])

        if data['custom']:
            values = {
                'body': nl2br(data['custom']),
                'model': 'sale.order',
                'message_type': 'comment',
                'no_auto_thread': False,
                'res_id': order.id,
            }
            request.env['mail.message'].with_user(SUPERUSER_ID).create(values)

        if data['attachments']:
            self.insert_attachment(model_record, order.id, data['attachments'])

        return json.dumps({'id': order.id})


class Website(Website):
    @http.route()
    def get_switchable_related_views(self, key):
        views = super(Website, self).get_switchable_related_views(key)
        if key == 'website_sale.product':
            if not request.env.user.has_group('product.group_product_variant'):
                view_product_variants = request.website.viewref('website_sale.product_variants')
                views = [v for v in views if v['id'] != view_product_variants.id]
        return views

    @http.route()
    def toggle_switchable_view(self, view_key):
        super(Website, self).toggle_switchable_view(view_key)
        if view_key in ('website_sale.products_list_view', 'website_sale.add_grid_or_list_option'):
            request.session.pop('website_sale_shop_layout_mode', None)


class WebsiteSale(http.Controller):

    def _get_pricelist_context(self):
        pricelist_context = dict(request.env.context)
        pricelist = False
        if not pricelist_context.get('pricelist'):
            pricelist = request.website.get_current_pricelist()
            pricelist_context['pricelist'] = pricelist.id
        else:
            pricelist = request.env['product.pricelist'].browse(pricelist_context['pricelist'])

        return pricelist_context, pricelist

    def _get_search_order(self, post):
        # OrderBy will be parsed in orm and so no direct sql injection
        # id is added to be sure that order is a unique sort key
        order = post.get('order') or 'website_sequence ASC'
        return 'is_published desc, %s, id desc' % order

    def _get_search_domain(self, search, category, attrib_values, search_in_description=True):
        domains = [request.website.sale_product_domain()]
        if search:
            for srch in search.split(" "):
                subdomains = [
                    [('name', 'ilike', srch)],
                    [('product_variant_ids.default_code', 'ilike', srch)]
                ]
                if search_in_description:
                    subdomains.append([('description', 'ilike', srch)])
                    subdomains.append([('description_sale', 'ilike', srch)])
                domains.append(expression.OR(subdomains))

        if category:
            domains.append([('public_categ_ids', 'child_of', int(category))])

        if attrib_values:
            attrib = None
            ids = []
            for value in attrib_values:
                if not attrib:
                    attrib = value[0]
                    ids.append(value[1])
                elif value[0] == attrib:
                    ids.append(value[1])
                else:
                    domains.append([('attribute_line_ids.value_ids', 'in', ids)])
                    attrib = value[0]
                    ids = [value[1]]
            if attrib:
                domains.append([('attribute_line_ids.value_ids', 'in', ids)])

        return expression.AND(domains)

    def sitemap_shop(env, rule, qs):
        if not qs or qs.lower() in '/shop':
            yield {'loc': '/shop'}

        Category = env['product.public.category']
        dom = sitemap_qs2dom(qs, '/shop/category', Category._rec_name)
        dom += env['website'].get_current_website().website_domain()
        for cat in Category.search(dom):
            loc = '/shop/category/%s' % slug(cat)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

    @http.route([
        '''/shop''',
        '''/shop/page/<int:page>''',
        '''/shop/category/<model("product.public.category"):category>''',
        '''/shop/category/<model("product.public.category"):category>/page/<int:page>'''
    ], type='http', auth="public", website=True, sitemap=sitemap_shop)
    def shop(self, page=0, category=None, search='', ppg=False, **post):
        add_qty = int(post.get('add_qty', 1))
        Category = request.env['product.public.category']
        if category:
            category = Category.search([('id', '=', int(category))], limit=1)
            if not category or not category.can_access_from_current_website():
                raise NotFound()
        else:
            category = Category

        if ppg:
            try:
                ppg = int(ppg)
                post['ppg'] = ppg
            except ValueError:
                ppg = False
        if not ppg:
            ppg = request.env['website'].get_current_website().shop_ppg or 20

        ppr = request.env['website'].get_current_website().shop_ppr or 4

        attrib_list = request.httprequest.args.getlist('attrib')
        attrib_values = [[int(x) for x in v.split("-")] for v in attrib_list if v]
        attributes_ids = {v[0] for v in attrib_values}
        attrib_set = {v[1] for v in attrib_values}

        domain = self._get_search_domain(search, category, attrib_values)

        keep = QueryURL('/shop', category=category and int(category), search=search, attrib=attrib_list, order=post.get('order'))

        pricelist_context, pricelist = self._get_pricelist_context()

        request.context = dict(request.context, pricelist=pricelist.id, partner=request.env.user.partner_id)

        url = "/shop"
        if search:
            post["search"] = search
        if attrib_list:
            post['attrib'] = attrib_list

        Product = request.env['product.template'].with_context(bin_size=True)

        search_product = Product.search(domain, order=self._get_search_order(post))
        website_domain = request.website.website_domain()
        categs_domain = [('parent_id', '=', False)] + website_domain
        if search:
            search_categories = Category.search([('product_tmpl_ids', 'in', search_product.ids)] + website_domain).parents_and_self
            categs_domain.append(('id', 'in', search_categories.ids))
        else:
            search_categories = Category
        categs = Category.search(categs_domain)

        if category:
            url = "/shop/category/%s" % slug(category)

        product_count = len(search_product)
        pager = request.website.pager(url=url, total=product_count, page=page, step=ppg, scope=5, url_args=post)
        offset = pager['offset']
        products = search_product[offset: offset + ppg]

        ProductAttribute = request.env['product.attribute']
        if products:
            # get all products without limit
            attributes = ProductAttribute.search([('product_tmpl_ids', 'in', search_product.ids)])
        else:
            attributes = ProductAttribute.browse(attributes_ids)

        layout_mode = request.session.get('website_sale_shop_layout_mode')
        if not layout_mode:
            if request.website.viewref('website_sale.products_list_view').active:
                layout_mode = 'list'
            else:
                layout_mode = 'grid'

        values = {
            'search': search,
            'order': post.get('order', ''),
            'category': category,
            'attrib_values': attrib_values,
            'attrib_set': attrib_set,
            'pager': pager,
            'pricelist': pricelist,
            'add_qty': add_qty,
            'products': products,
            'search_count': product_count,  # common for all searchbox
            'bins': TableCompute().process(products, ppg, ppr),
            'ppg': ppg,
            'ppr': ppr,
            'categories': categs,
            'attributes': attributes,
            'keep': keep,
            'search_categories_ids': search_categories.ids,
            'layout_mode': layout_mode,
        }
        if category:
            values['main_object'] = category
        return request.render("website_sale.products", values)

    @http.route(['/shop/<model("product.template"):product>'], type='http', auth="public", website=True, sitemap=True)
    def product(self, product, category='', search='', **kwargs):
        if not product.can_access_from_current_website():
            raise NotFound()

        return request.render("website_sale.product", self._prepare_product_values(product, category, search, **kwargs))

    @http.route(['/shop/product/<model("product.template"):product>'], type='http', auth="public", website=True, sitemap=False)
    def old_product(self, product, category='', search='', **kwargs):
        # Compatibility pre-v14
        return request.redirect(_build_url_w_params("/shop/%s" % slug(product), request.params), code=301)

    def _prepare_product_values(self, product, category, search, **kwargs):
        add_qty = int(kwargs.get('add_qty', 1))

        product_context = dict(request.env.context, quantity=add_qty,
                               active_id=product.id,
                               partner=request.env.user.partner_id)
        ProductCategory = request.env['product.public.category']

        if category:
            category = ProductCategory.browse(int(category)).exists()

        attrib_list = request.httprequest.args.getlist('attrib')
        attrib_values = [[int(x) for x in v.split("-")] for v in attrib_list if v]
        attrib_set = {v[1] for v in attrib_values}

        keep = QueryURL('/shop', category=category and category.id, search=search, attrib=attrib_list)

        categs = ProductCategory.search([('parent_id', '=', False)])

        pricelist = request.website.get_current_pricelist()

        if not product_context.get('pricelist'):
            product_context['pricelist'] = pricelist.id
            product = product.with_context(product_context)

        # Needed to trigger the recently viewed product rpc
        view_track = request.website.viewref("website_sale.product").track

        return {
            'search': search,
            'category': category,
            'pricelist': pricelist,
            'attrib_values': attrib_values,
            'attrib_set': attrib_set,
            'keep': keep,
            'categories': categs,
            'main_object': product,
            'product': product,
            'add_qty': add_qty,
            'view_track': view_track,
        }

    @http.route(['/shop/change_pricelist/<model("product.pricelist"):pl_id>'], type='http', auth="public", website=True, sitemap=False)
    def pricelist_change(self, pl_id, **post):
        if (pl_id.selectable or pl_id == request.env.user.partner_id.property_product_pricelist) \
                and request.website.is_pricelist_available(pl_id.id):
            request.session['website_sale_current_pl'] = pl_id.id
            request.website.sale_get_order(force_pricelist=pl_id.id)
        return request.redirect(request.httprequest.referrer or '/shop')

    @http.route(['/shop/pricelist'], type='http', auth="public", website=True, sitemap=False)
    def pricelist(self, promo, **post):
        redirect = post.get('r', '/shop/cart')
        # empty promo code is used to reset/remove pricelist (see `sale_get_order()`)
        if promo:
            pricelist = request.env['product.pricelist'].sudo().search([('code', '=', promo)], limit=1)
            if (not pricelist or (pricelist and not request.website.is_pricelist_available(pricelist.id))):
                return request.redirect("%s?code_not_available=1" % redirect)

        request.website.sale_get_order(code=promo)
        return request.redirect(redirect)

    @http.route(['/shop/cart'], type='http', auth="public", website=True, sitemap=False)
    def cart(self, access_token=None, revive='', **post):
        """
        Main cart management + abandoned cart revival
        access_token: Abandoned cart SO access token
        revive: Revival method when abandoned cart. Can be 'merge' or 'squash'
        """
        order = request.website.sale_get_order()
        if order and order.state != 'draft':
            request.session['sale_order_id'] = None
            order = request.website.sale_get_order()
        values = {}
        if access_token:
            abandoned_order = request.env['sale.order'].sudo().search([('access_token', '=', access_token)], limit=1)
            if not abandoned_order:  # wrong token (or SO has been deleted)
                raise NotFound()
            if abandoned_order.state != 'draft':  # abandoned cart already finished
                values.update({'abandoned_proceed': True})
            elif revive == 'squash' or (revive == 'merge' and not request.session.get('sale_order_id')):  # restore old cart or merge with unexistant
                request.session['sale_order_id'] = abandoned_order.id
                return request.redirect('/shop/cart')
            elif revive == 'merge':
                abandoned_order.order_line.write({'order_id': request.session['sale_order_id']})
                abandoned_order.action_cancel()
            elif abandoned_order.id != request.session.get('sale_order_id'):  # abandoned cart found, user have to choose what to do
                values.update({'access_token': abandoned_order.access_token})

        values.update({
            'website_sale_order': order,
            'date': fields.Date.today(),
            'suggested_products': [],
        })
        if order:
            order.order_line.filtered(lambda l: not l.product_id.active).unlink()
            _order = order
            if not request.env.context.get('pricelist'):
                _order = order.with_context(pricelist=order.pricelist_id.id)
            values['suggested_products'] = _order._cart_accessories()

        if post.get('type') == 'popover':
            # force no-cache so IE11 doesn't cache this XHR
            return request.render("website_sale.cart_popover", values, headers={'Cache-Control': 'no-cache'})

        return request.render("website_sale.cart", values)

    @http.route(['/shop/cart/update'], type='http', auth="public", methods=['POST'], website=True)
    def cart_update(self, product_id, add_qty=1, set_qty=0, **kw):
        """This route is called when adding a product to cart (no options)."""
        sale_order = request.website.sale_get_order(force_create=True)
        if sale_order.state != 'draft':
            request.session['sale_order_id'] = None
            sale_order = request.website.sale_get_order(force_create=True)

        product_custom_attribute_values = None
        if kw.get('product_custom_attribute_values'):
            product_custom_attribute_values = json.loads(kw.get('product_custom_attribute_values'))

        no_variant_attribute_values = None
        if kw.get('no_variant_attribute_values'):
            no_variant_attribute_values = json.loads(kw.get('no_variant_attribute_values'))

        sale_order._cart_update(
            product_id=int(product_id),
            add_qty=add_qty,
            set_qty=set_qty,
            product_custom_attribute_values=product_custom_attribute_values,
            no_variant_attribute_values=no_variant_attribute_values
        )

        if kw.get('express'):
            return request.redirect("/shop/checkout?express=1")

        return request.redirect("/shop/cart")

    @http.route(['/shop/cart/update_json'], type='json', auth="public", methods=['POST'], website=True, csrf=False)
    def cart_update_json(self, product_id, line_id=None, add_qty=None, set_qty=None, display=True):
        """This route is called when changing quantity from the cart or adding
        a product from the wishlist."""
        order = request.website.sale_get_order(force_create=1)
        if order.state != 'draft':
            request.website.sale_reset()
            return {}

        value = order._cart_update(product_id=product_id, line_id=line_id, add_qty=add_qty, set_qty=set_qty)

        if not order.cart_quantity:
            request.website.sale_reset()
            return value

        order = request.website.sale_get_order()
        value['cart_quantity'] = order.cart_quantity

        if not display:
            return value

        value['website_sale.cart_lines'] = request.env['ir.ui.view']._render_template("website_sale.cart_lines", {
            'website_sale_order': order,
            'date': fields.Date.today(),
            'suggested_products': order._cart_accessories()
        })
        value['website_sale.short_cart_summary'] = request.env['ir.ui.view']._render_template("website_sale.short_cart_summary", {
            'website_sale_order': order,
        })
        return value

    @http.route('/shop/save_shop_layout_mode', type='json', auth='public', website=True)
    def save_shop_layout_mode(self, layout_mode):
        assert layout_mode in ('grid', 'list'), "Invalid shop layout mode"
        request.session['website_sale_shop_layout_mode'] = layout_mode

    # ------------------------------------------------------
    # Checkout
    # ------------------------------------------------------

    def checkout_check_address(self, order):
        billing_fields_required = self._get_mandatory_fields_billing(order.partner_id.country_id.id)
        if not all(order.partner_id.read(billing_fields_required)[0].values()):
            return request.redirect('/shop/address?partner_id=%d' % order.partner_id.id)

        shipping_fields_required = self._get_mandatory_fields_shipping(order.partner_shipping_id.country_id.id)
        if not all(order.partner_shipping_id.read(shipping_fields_required)[0].values()):
            return request.redirect('/shop/address?partner_id=%d' % order.partner_shipping_id.id)

    def checkout_redirection(self, order):
        # must have a draft sales order with lines at this point, otherwise reset
        if not order or order.state != 'draft':
            request.session['sale_order_id'] = None
            request.session['sale_transaction_id'] = None
            return request.redirect('/shop')

        if order and not order.order_line:
            return request.redirect('/shop/cart')

        # if transaction pending / done: redirect to confirmation
        tx = request.env.context.get('website_sale_transaction')
        if tx and tx.state != 'draft':
            return request.redirect('/shop/payment/confirmation/%s' % order.id)

    def checkout_values(self, **kw):
        order = request.website.sale_get_order(force_create=1)
        shippings = []
        if order.partner_id != request.website.user_id.sudo().partner_id:
            Partner = order.partner_id.with_context(show_address=1).sudo()
            shippings = Partner.search([
                ("id", "child_of", order.partner_id.commercial_partner_id.ids),
                '|', ("type", "in", ["delivery", "other"]), ("id", "=", order.partner_id.commercial_partner_id.id)
            ], order='id desc')
            if shippings:
                if kw.get('partner_id') or 'use_billing' in kw:
                    if 'use_billing' in kw:
                        partner_id = order.partner_id.id
                    else:
                        partner_id = int(kw.get('partner_id'))
                    if partner_id in shippings.mapped('id'):
                        order.partner_shipping_id = partner_id

        values = {
            'order': order,
            'shippings': shippings,
            'only_services': order and order.only_services or False
        }
        return values

    def _get_mandatory_billing_fields(self):
        # deprecated for _get_mandatory_fields_billing which handle zip/state required
        return ["name", "email", "street", "city", "country_id"]

    def _get_mandatory_shipping_fields(self):
        # deprecated for _get_mandatory_fields_shipping which handle zip/state required
        return ["name", "street", "city", "country_id"]

    def _get_mandatory_fields_billing(self, country_id=False):
        req = self._get_mandatory_billing_fields()
        if country_id:
            country = request.env['res.country'].browse(country_id)
            if country.state_required:
                req += ['state_id']
            if country.zip_required:
                req += ['zip']
        return req

    def _get_mandatory_fields_shipping(self, country_id=False):
        req = self._get_mandatory_shipping_fields()
        if country_id:
            country = request.env['res.country'].browse(country_id)
            if country.state_required:
                req += ['state_id']
            if country.zip_required:
                req += ['zip']
        return req

    def checkout_form_validate(self, mode, all_form_values, data):
        # mode: tuple ('new|edit', 'billing|shipping')
        # all_form_values: all values before preprocess
        # data: values after preprocess
        error = dict()
        error_message = []

        if data.get('partner_id'):
            partner_su = request.env['res.partner'].sudo().browse(int(data['partner_id'])).exists()
            name_change = partner_su and 'name' in data and data['name'] != partner_su.name
            email_change = partner_su and 'email' in data and data['email'] != partner_su.email

            # Prevent changing the billing partner name if invoices have been issued.
            if mode[1] == 'billing' and name_change and not partner_su.can_edit_vat():
                error['name'] = 'error'
                error_message.append(_(
                    "Changing your name is not allowed once documents have been issued for your"
                    " account. Please contact us directly for this operation."
                ))

            # Prevent change the partner name or email if it is an internal user.
            if (name_change or email_change) and not all(partner_su.user_ids.mapped('share')):
                error.update({
                    'name': 'error' if name_change else None,
                    'email': 'error' if email_change else None,
                })
                error_message.append(_(
                    "If you are ordering for an external person, please place your order via the"
                    " backend. If you wish to change your name or email address, please do so in"
                    " the account settings or contact your administrator."
                ))

        # Required fields from form
        required_fields = [f for f in (all_form_values.get('field_required') or '').split(',') if f]

        # Required fields from mandatory field function
        country_id = int(data.get('country_id', False))
        required_fields += mode[1] == 'shipping' and self._get_mandatory_fields_shipping(country_id) or self._get_mandatory_fields_billing(country_id)

        # error message for empty required fields
        for field_name in required_fields:
            if not data.get(field_name):
                error[field_name] = 'missing'

        # email validation
        if data.get('email') and not tools.single_email_re.match(data.get('email')):
            error["email"] = 'error'
            error_message.append(_('Invalid Email! Please enter a valid email address.'))

        # vat validation
        Partner = request.env['res.partner']
        if data.get("vat") and hasattr(Partner, "check_vat"):
            if country_id:
                data["vat"] = Partner.fix_eu_vat_number(country_id, data.get("vat"))
            partner_dummy = Partner.new(self._get_vat_validation_fields(data))
            try:
                partner_dummy.check_vat()
            except ValidationError as exception:
                error["vat"] = 'error'
                error_message.append(exception.args[0])

        if [err for err in error.values() if err == 'missing']:
            error_message.append(_('Some required fields are empty.'))

        return error, error_message

    def _get_vat_validation_fields(self, data):
        return {
            'vat': data['vat'],
            'country_id': int(data['country_id']) if data.get('country_id') else False,
        }

    def _checkout_form_save(self, mode, checkout, all_values):
        Partner = request.env['res.partner']
        if mode[0] == 'new':
            partner_id = Partner.sudo().with_context(tracking_disable=True).create(checkout).id
        elif mode[0] == 'edit':
            partner_id = int(all_values.get('partner_id', 0))
            if partner_id:
                # double check
                order = request.website.sale_get_order()
                shippings = Partner.sudo().search([("id", "child_of", order.partner_id.commercial_partner_id.ids)])
                if partner_id not in shippings.mapped('id') and partner_id != order.partner_id.id:
                    return Forbidden()
                Partner.browse(partner_id).sudo().write(checkout)
        return partner_id

    def values_preprocess(self, order, mode, values):
        new_values = dict()
        partner_fields = request.env['res.partner']._fields

        for k, v in values.items():
            # Convert the values for many2one fields to integer since they are used as IDs
            if k in partner_fields and partner_fields[k].type == 'many2one':
                new_values[k] = bool(v) and int(v)
            # Store empty fields as `False` instead of empty strings `''` for consistency with other applications like
            # Contacts.
            elif v == '':
                new_values[k] = False
            else:
                new_values[k] = v

        return new_values

    def values_postprocess(self, order, mode, values, errors, error_msg):
        new_values = {}
        authorized_fields = request.env['ir.model']._get('res.partner')._get_form_writable_fields()
        for k, v in values.items():
            # don't drop empty value, it could be a field to reset
            if k in authorized_fields and v is not None:
                new_values[k] = v
            else:  # DEBUG ONLY
                if k not in ('field_required', 'partner_id', 'callback', 'submitted'): # classic case
                    _logger.debug("website_sale postprocess: %s value has been dropped (empty or not writable)" % k)

        if request.website.specific_user_account:
            new_values['website_id'] = request.website.id

        if mode[0] == 'new':
            new_values['company_id'] = request.website.company_id.id
            new_values['team_id'] = request.website.salesteam_id and request.website.salesteam_id.id
            new_values['user_id'] = request.website.salesperson_id.id

        lang = request.lang.code if request.lang.code in request.website.mapped('language_ids.code') else None
        if lang:
            new_values['lang'] = lang
        if mode == ('edit', 'billing') and order.partner_id.type == 'contact':
            new_values['type'] = 'other'
        if mode[1] == 'shipping':
            new_values['parent_id'] = order.partner_id.commercial_partner_id.id
            new_values['type'] = 'delivery'

        return new_values, errors, error_msg

    @http.route(['/shop/address'], type='http', methods=['GET', 'POST'], auth="public", website=True, sitemap=False)
    def address(self, **kw):
        Partner = request.env['res.partner'].with_context(show_address=1).sudo()
        order = request.website.sale_get_order()

        redirection = self.checkout_redirection(order)
        if redirection:
            return redirection

        mode = (False, False)
        can_edit_vat = False
        values, errors = {}, {}

        partner_id = int(kw.get('partner_id', -1))

        # IF PUBLIC ORDER
        if order.partner_id.id == request.website.user_id.sudo().partner_id.id:
            mode = ('new', 'billing')
            can_edit_vat = True
        # IF ORDER LINKED TO A PARTNER
        else:
            if partner_id > 0:
                if partner_id == order.partner_id.id:
                    mode = ('edit', 'billing')
                    can_edit_vat = order.partner_id.can_edit_vat()
                else:
                    shippings = Partner.search([('id', 'child_of', order.partner_id.commercial_partner_id.ids)])
                    if order.partner_id.commercial_partner_id.id == partner_id:
                        mode = ('new', 'shipping')
                        partner_id = -1
                    elif partner_id in shippings.mapped('id'):
                        mode = ('edit', 'shipping')
                    else:
                        return Forbidden()
                if mode and partner_id != -1:
                    values = Partner.browse(partner_id)
            elif partner_id == -1:
                mode = ('new', 'shipping')
            else: # no mode - refresh without post?
                return request.redirect('/shop/checkout')

        # IF POSTED
        if 'submitted' in kw and request.httprequest.method == "POST":
            pre_values = self.values_preprocess(order, mode, kw)
            errors, error_msg = self.checkout_form_validate(mode, kw, pre_values)
            post, errors, error_msg = self.values_postprocess(order, mode, pre_values, errors, error_msg)

            if errors:
                errors['error_message'] = error_msg
                values = kw
            else:
                partner_id = self._checkout_form_save(mode, post, kw)
                # We need to validate _checkout_form_save return, because when partner_id not in shippings
                # it returns Forbidden() instead the partner_id
                if isinstance(partner_id, Forbidden):
                    return partner_id
                if mode[1] == 'billing':
                    order.partner_id = partner_id
                    order.with_context(not_self_saleperson=True).onchange_partner_id()
                    # This is the *only* thing that the front end user will see/edit anyway when choosing billing address
                    order.partner_invoice_id = partner_id
                    if not kw.get('use_same'):
                        kw['callback'] = kw.get('callback') or \
                            (not order.only_services and (mode[0] == 'edit' and '/shop/checkout' or '/shop/address'))
                    # We need to update the pricelist(by the one selected by the customer), because onchange_partner reset it
                    # We only need to update the pricelist when it is not redirected to /confirm_order
                    if kw.get('callback', False) != '/shop/confirm_order':
                        request.website.sale_get_order(update_pricelist=True)
                elif mode[1] == 'shipping':
                    order.partner_shipping_id = partner_id

                # TDE FIXME: don't ever do this
                order.message_partner_ids = [(4, partner_id), (3, request.website.partner_id.id)]
                if not errors:
                    return request.redirect(kw.get('callback') or '/shop/confirm_order')

        render_values = {
            'website_sale_order': order,
            'partner_id': partner_id,
            'mode': mode,
            'checkout': values,
            'can_edit_vat': can_edit_vat,
            'error': errors,
            'callback': kw.get('callback'),
            'only_services': order and order.only_services,
        }
        render_values.update(self._get_country_related_render_values(kw, render_values))
        return request.render("website_sale.address", render_values)

    def _get_country_related_render_values(self, kw, render_values):
        '''
        This method provides fields related to the country to render the website sale form
        '''
        values = render_values['checkout']
        mode = render_values['mode']
        order = render_values['website_sale_order']

        def_country_id = order.partner_id.country_id
        # IF PUBLIC ORDER
        if order.partner_id.id == request.website.user_id.sudo().partner_id.id:
            country_code = request.session['geoip'].get('country_code')
            if country_code:
                def_country_id = request.env['res.country'].search([('code', '=', country_code)], limit=1)
            else:
                def_country_id = request.website.user_id.sudo().country_id

        country = 'country_id' in values and values['country_id'] != '' and request.env['res.country'].browse(int(values['country_id']))
        country = country and country.exists() or def_country_id

        res = {
            'country': country,
            'country_states': country.get_website_sale_states(mode=mode[1]),
            'countries': country.get_website_sale_countries(mode=mode[1]),
        }
        return res

    @http.route(['/shop/checkout'], type='http', auth="public", website=True, sitemap=False)
    def checkout(self, **post):
        order = request.website.sale_get_order()

        redirection = self.checkout_redirection(order)
        if redirection:
            return redirection

        if order.partner_id.id == request.website.user_id.sudo().partner_id.id:
            return request.redirect('/shop/address')

        redirection = self.checkout_check_address(order)
        if redirection:
            return redirection

        values = self.checkout_values(**post)

        if post.get('express'):
            return request.redirect('/shop/confirm_order')

        values.update({'website_sale_order': order})

        # Avoid useless rendering if called in ajax
        if post.get('xhr'):
            return 'ok'
        return request.render("website_sale.checkout", values)

    @http.route(['/shop/confirm_order'], type='http', auth="public", website=True, sitemap=False)
    def confirm_order(self, **post):
        order = request.website.sale_get_order()

        redirection = self.checkout_redirection(order) or self.checkout_check_address(order)
        if redirection:
            return redirection

        order.onchange_partner_shipping_id()
        order.order_line._compute_tax_id()
        request.session['sale_last_order_id'] = order.id
        request.website.sale_get_order(update_pricelist=True)
        extra_step = request.website.viewref('website_sale.extra_info_option')
        if extra_step.active:
            return request.redirect("/shop/extra_info")

        return request.redirect("/shop/payment")

    # ------------------------------------------------------
    # Extra step
    # ------------------------------------------------------
    @http.route(['/shop/extra_info'], type='http', auth="public", website=True, sitemap=False)
    def extra_info(self, **post):
        # Check that this option is activated
        extra_step = request.website.viewref('website_sale.extra_info_option')
        if not extra_step.active:
            return request.redirect("/shop/payment")

        # check that cart is valid
        order = request.website.sale_get_order()
        redirection = self.checkout_redirection(order)
        if redirection:
            return redirection

        # if form posted
        if 'post_values' in post:
            values = {}
            for field_name, field_value in post.items():
                if field_name in request.env['sale.order']._fields and field_name.startswith('x_'):
                    values[field_name] = field_value
            if values:
                order.write(values)
            return request.redirect("/shop/payment")

        values = {
            'website_sale_order': order,
            'post': post,
            'escape': lambda x: x.replace("'", r"\'"),
            'partner': order.partner_id.id,
            'order': order,
        }

        return request.render("website_sale.extra_info", values)

    # ------------------------------------------------------
    # Payment
    # ------------------------------------------------------

    def _get_shop_payment_values(self, order, **kwargs):
        values = dict(
            website_sale_order=order,
            errors=[],
            partner=order.partner_id.id,
            order=order,
            payment_action_id=request.env.ref('payment.action_payment_acquirer').id,
            return_url= '/shop/payment/validate',
            bootstrap_formatting= True
        )
        if order.partner_id.company_id and order.partner_id.company_id.id != order.company_id.id:
            values['errors'].append(
                (_('Sorry, we are unable to find a Payment Method'),
                 _('No payment method is available for your current order. '
                   'Please contact us for more information.')))
            return values

        domain = expression.AND([
            ['&', ('state', 'in', ['enabled', 'test']), ('company_id', '=', order.company_id.id)],
            ['|', ('website_id', '=', False), ('website_id', '=', request.website.id)],
            ['|', ('country_ids', '=', False), ('country_ids', 'in', [order.partner_id.country_id.id])]
        ])
        acquirers = request.env['payment.acquirer'].search(domain)

        values['access_token'] = order.access_token
        values['acquirers'] = [acq for acq in acquirers if (acq.payment_flow == 'form' and acq.view_template_id) or
                                    (acq.payment_flow == 's2s' and acq.registration_view_template_id)]
        values['tokens'] = request.env['payment.token'].search([
            ('acquirer_id', 'in', acquirers.ids),
            ('partner_id', 'child_of', order.partner_id.commercial_partner_id.id)])

        if order:
            values['acq_extra_fees'] = acquirers.get_acquirer_extra_fees(order.amount_total, order.currency_id, order.partner_id.country_id.id)
        return values

    @http.route(['/shop/payment'], type='http', auth="public", website=True, sitemap=False)
    def payment(self, **post):
        """ Payment step. This page proposes several payment means based on available
        payment.acquirer. State at this point :

         - a draft sales order with lines; otherwise, clean context / session and
           back to the shop
         - no transaction in context / session, or only a draft one, if the customer
           did go to a payment.acquirer website but closed the tab without
           paying / canceling
        """
        order = request.website.sale_get_order()
        redirection = self.checkout_redirection(order) or self.checkout_check_address(order)
        if redirection:
            return redirection

        render_values = self._get_shop_payment_values(order, **post)
        render_values['only_services'] = order and order.only_services or False

        if render_values['errors']:
            render_values.pop('acquirers', '')
            render_values.pop('tokens', '')

        return request.render("website_sale.payment", render_values)

    @http.route(['/shop/payment/transaction/',
        '/shop/payment/transaction/<int:so_id>',
        '/shop/payment/transaction/<int:so_id>/<string:access_token>'], type='json', auth="public", website=True)
    def payment_transaction(self, acquirer_id, save_token=False, so_id=None, access_token=None, token=None, **kwargs):
        """ Json method that creates a payment.transaction, used to create a
        transaction when the user clicks on 'pay now' button. After having
        created the transaction, the event continues and the user is redirected
        to the acquirer website.

        :param int acquirer_id: id of a payment.acquirer record. If not set the
                                user is redirected to the checkout page
        """
        # Ensure a payment acquirer is selected
        if not acquirer_id:
            return False

        try:
            acquirer_id = int(acquirer_id)
        except:
            return False

        # Retrieve the sale order
        if so_id:
            env = request.env['sale.order']
            domain = [('id', '=', so_id)]
            if access_token:
                env = env.sudo()
                domain.append(('access_token', '=', access_token))
            order = env.search(domain, limit=1)
        else:
            order = request.website.sale_get_order()

        # Ensure there is something to proceed
        if not order or (order and not order.order_line):
            return False

        assert order.partner_id.id != request.website.partner_id.id

        # Create transaction
        vals = {'acquirer_id': acquirer_id,
                'return_url': '/shop/payment/validate'}

        if save_token:
            vals['type'] = 'form_save'
        if token:
            vals['payment_token_id'] = int(token)

        transaction = order._create_payment_transaction(vals)

        # store the new transaction into the transaction list and if there's an old one, we remove it
        # until the day the ecommerce supports multiple orders at the same time
        last_tx_id = request.session.get('__website_sale_last_tx_id')
        last_tx = request.env['payment.transaction'].browse(last_tx_id).sudo().exists()
        if last_tx:
            PaymentProcessing.remove_payment_transaction(last_tx)
        PaymentProcessing.add_payment_transaction(transaction)
        request.session['__website_sale_last_tx_id'] = transaction.id
        return transaction.render_sale_button(order)

    @http.route('/shop/payment/token', type='http', auth='public', website=True, sitemap=False)
    def payment_token(self, pm_id=None, **kwargs):
        """ Method that handles payment using saved tokens

        :param int pm_id: id of the payment.token that we want to use to pay.
        """
        order = request.website.sale_get_order()
        # do not crash if the user has already paid and try to pay again
        if not order:
            return request.redirect('/shop/?error=no_order')

        assert order.partner_id.id != request.website.partner_id.id

        try:
            pm_id = int(pm_id)
        except ValueError:
            return request.redirect('/shop/?error=invalid_token_id')

        # We retrieve the token the user want to use to pay
        if not request.env['payment.token'].sudo().search_count([('id', '=', pm_id)]):
            return request.redirect('/shop/?error=token_not_found')

        # Create transaction
        vals = {'payment_token_id': pm_id, 'return_url': '/shop/payment/validate'}

        tx = order._create_payment_transaction(vals)
        PaymentProcessing.add_payment_transaction(tx)
        return request.redirect('/payment/process')

    @http.route('/shop/payment/get_status/<int:sale_order_id>', type='json', auth="public", website=True)
    def payment_get_status(self, sale_order_id, **post):
        order = request.env['sale.order'].sudo().browse(sale_order_id).exists()
        if order.id != request.session.get('sale_last_order_id'):
            # either something went wrong or the session is unbound
            # prevent recalling every 3rd of a second in the JS widget
            return {}

        return {
            'recall': order.get_portal_last_transaction().state == 'pending',
            'message': request.env['ir.ui.view']._render_template("website_sale.payment_confirmation_status", {
                'order': order
            })
        }

    @http.route('/shop/payment/validate', type='http', auth="public", website=True, sitemap=False)
    def payment_validate(self, transaction_id=None, sale_order_id=None, **post):
        """ Method that should be called by the server when receiving an update
        for a transaction. State at this point :

         - UDPATE ME
        """
        if sale_order_id is None:
            order = request.website.sale_get_order()
            if not order and 'sale_last_order_id' in request.session:
                # Retrieve the last known order from the session if the session key `sale_order_id`
                # was prematurely cleared. This is done to prevent the user from updating their cart
                # after payment in case they don't return from payment through this route.
                last_order_id = request.session['sale_last_order_id']
                order = request.env['sale.order'].sudo().browse(last_order_id).exists()
        else:
            order = request.env['sale.order'].sudo().browse(sale_order_id)
            assert order.id == request.session.get('sale_last_order_id')

        if transaction_id:
            tx = request.env['payment.transaction'].sudo().browse(transaction_id)
            assert tx in order.transaction_ids()
        elif order:
            tx = order.get_portal_last_transaction()
        else:
            tx = None

        if not order or (order.amount_total and not tx):
            return request.redirect('/shop')

        if order and not order.amount_total and not tx:
            order.with_context(send_email=True).action_confirm()
            return request.redirect(order.get_portal_url())

        # clean context and session, then redirect to the confirmation page
        request.website.sale_reset()
        if tx and tx.state == 'draft':
            return request.redirect('/shop')

        PaymentProcessing.remove_payment_transaction(tx)
        return request.redirect('/shop/confirmation')

    @http.route(['/shop/terms'], type='http', auth="public", website=True, sitemap=True)
    def terms(self, **kw):
        return request.render("website_sale.terms")

    @http.route(['/shop/confirmation'], type='http', auth="public", website=True, sitemap=False)
    def payment_confirmation(self, **post):
        """ End of checkout process controller. Confirmation is basically seing
        the status of a sale.order. State at this point :

         - should not have any context / session info: clean them
         - take a sale.order id, because we request a sale.order and are not
           session dependant anymore
        """
        sale_order_id = request.session.get('sale_last_order_id')
        if sale_order_id:
            order = request.env['sale.order'].sudo().browse(sale_order_id)
            return request.render("website_sale.confirmation", {'order': order})
        else:
            return request.redirect('/shop')

    @http.route(['/shop/print'], type='http', auth="public", website=True, sitemap=False)
    def print_saleorder(self, **kwargs):
        sale_order_id = request.session.get('sale_last_order_id')
        if sale_order_id:
            pdf, _ = request.env.ref('sale.action_report_saleorder').with_user(SUPERUSER_ID)._render_qweb_pdf([sale_order_id])
            pdfhttpheaders = [('Content-Type', 'application/pdf'), ('Content-Length', u'%s' % len(pdf))]
            return request.make_response(pdf, headers=pdfhttpheaders)
        else:
            return request.redirect('/shop')

    @http.route(['/shop/tracking_last_order'], type='json', auth="public")
    def tracking_cart(self, **post):
        """ return data about order in JSON needed for google analytics"""
        ret = {}
        sale_order_id = request.session.get('sale_last_order_id')
        if sale_order_id:
            order = request.env['sale.order'].sudo().browse(sale_order_id)
            ret = self.order_2_return_dict(order)
        return ret

    # ------------------------------------------------------
    # Edit
    # ------------------------------------------------------

    @http.route(['/shop/add_product'], type='json', auth="user", methods=['POST'], website=True)
    def add_product(self, name=None, category=None, **post):
        product = request.env['product.product'].create({
            'name': name or _("New Product"),
            'public_categ_ids': category,
            'website_id': request.website.id,
        })
        return "%s?enable_editor=1" % product.product_tmpl_id.website_url

    @http.route(['/shop/change_sequence'], type='json', auth='user')
    def change_sequence(self, id, sequence):
        product_tmpl = request.env['product.template'].browse(id)
        if sequence == "top":
            product_tmpl.set_sequence_top()
        elif sequence == "bottom":
            product_tmpl.set_sequence_bottom()
        elif sequence == "up":
            product_tmpl.set_sequence_up()
        elif sequence == "down":
            product_tmpl.set_sequence_down()

    @http.route(['/shop/change_size'], type='json', auth='user')
    def change_size(self, id, x, y):
        product = request.env['product.template'].browse(id)
        return product.write({'website_size_x': x, 'website_size_y': y})

    @http.route(['/shop/change_ppg'], type='json', auth='user')
    def change_ppg(self, ppg):
        request.env['website'].get_current_website().shop_ppg = ppg

    @http.route(['/shop/change_ppr'], type='json', auth='user')
    def change_ppr(self, ppr):
        request.env['website'].get_current_website().shop_ppr = ppr

    def order_lines_2_google_api(self, order_lines):
        """ Transforms a list of order lines into a dict for google analytics """
        ret = []
        for line in order_lines:
            product = line.product_id
            ret.append({
                'id': line.order_id.id,
                'sku': product.barcode or product.id,
                'name': product.name or '-',
                'category': product.categ_id.name or '-',
                'price': line.price_unit,
                'quantity': line.product_uom_qty,
            })
        return ret

    def order_2_return_dict(self, order):
        """ Returns the tracking_cart dict of the order for Google analytics basically defined to be inherited """
        return {
            'transaction': {
                'id': order.id,
                'affiliation': order.company_id.name,
                'revenue': order.amount_total,
                'tax': order.amount_tax,
                'currency': order.currency_id.name
            },
            'lines': self.order_lines_2_google_api(order.order_line)
        }

    @http.route(['/shop/country_infos/<model("res.country"):country>'], type='json', auth="public", methods=['POST'], website=True)
    def country_infos(self, country, mode, **kw):
        return dict(
            fields=country.get_address_fields(),
            states=[(st.id, st.name, st.code) for st in country.get_website_sale_states(mode=mode)],
            phone_code=country.phone_code,
            zip_required=country.zip_required,
            state_required=country.state_required,
        )

    # --------------------------------------------------------------------------
    # Products Search Bar
    # --------------------------------------------------------------------------

    @http.route('/shop/products/autocomplete', type='json', auth='public', website=True)
    def products_autocomplete(self, term, options={}, **kwargs):
        """
        Returns list of products according to the term and product options

        Params:
            term (str): search term written by the user
            options (dict)
                - 'limit' (int), default to 5: number of products to consider
                - 'display_description' (bool), default to True
                - 'display_price' (bool), default to True
                - 'order' (str)
                - 'max_nb_chars' (int): max number of characters for the
                                        description if returned

        Returns:
            dict (or False if no result)
                - 'products' (list): products (only their needed field values)
                        note: the prices will be strings properly formatted and
                        already containing the currency
                - 'products_count' (int): the number of products in the database
                        that matched the search query
        """
        ProductTemplate = request.env['product.template']

        display_description = options.get('display_description', True)
        display_price = options.get('display_price', True)
        order = self._get_search_order(options)
        max_nb_chars = options.get('max_nb_chars', 999)

        category = options.get('category')
        attrib_values = options.get('attrib_values')

        domain = self._get_search_domain(term, category, attrib_values, display_description)
        products = ProductTemplate.search(
            domain,
            limit=min(20, options.get('limit', 5)),
            order=order
        )

        fields = ['id', 'name', 'website_url']
        if display_description:
            fields.append('description_sale')

        res = {
            'products': products.read(fields),
            'products_count': ProductTemplate.search_count(domain),
        }

        if display_description:
            for res_product in res['products']:
                desc = res_product['description_sale']
                if desc and len(desc) > max_nb_chars:
                    res_product['description_sale'] = "%s..." % desc[:(max_nb_chars - 3)]

        if display_price:
            FieldMonetary = request.env['ir.qweb.field.monetary']
            monetary_options = {
                'display_currency': request.website.get_current_pricelist().currency_id,
            }
            for res_product, product in zip(res['products'], products):
                combination_info = product._get_combination_info(only_template=True)
                res_product.update(combination_info)
                res_product['list_price'] = FieldMonetary.value_to_html(res_product['list_price'], monetary_options)
                res_product['price'] = FieldMonetary.value_to_html(res_product['price'], monetary_options)

        return res

    # --------------------------------------------------------------------------
    # Products Recently Viewed
    # --------------------------------------------------------------------------
    @http.route('/shop/products/recently_viewed', type='json', auth='public', website=True)
    def products_recently_viewed(self, **kwargs):
        return self._get_products_recently_viewed()

    def _get_products_recently_viewed(self):
        """
        Returns list of recently viewed products according to current user
        """
        max_number_of_product_for_carousel = 12
        visitor = request.env['website.visitor']._get_visitor_from_request()
        if visitor:
            excluded_products = request.website.sale_get_order().mapped('order_line.product_id.id')
            products = request.env['website.track'].sudo().read_group(
                [('visitor_id', '=', visitor.id), ('product_id', '!=', False), ('product_id.website_published', '=', True), ('product_id', 'not in', excluded_products)],
                ['product_id', 'visit_datetime:max'], ['product_id'], limit=max_number_of_product_for_carousel, orderby='visit_datetime DESC')
            products_ids = [product['product_id'][0] for product in products]
            if products_ids:
                viewed_products = request.env['product.product'].with_context(display_default_code=False).search([('id', 'in', products_ids)])

                FieldMonetary = request.env['ir.qweb.field.monetary']
                monetary_options = {
                    'display_currency': request.website.get_current_pricelist().currency_id,
                }
                rating = request.website.viewref('website_sale.product_comment').active
                res = {'products': []}
                for product in viewed_products:
                    combination_info = product._get_combination_info_variant()
                    res_product = product.read(['id', 'name', 'website_url'])[0]
                    res_product.update(combination_info)
                    res_product['price'] = FieldMonetary.value_to_html(res_product['price'], monetary_options)
                    if rating:
                        res_product['rating'] = request.env["ir.ui.view"]._render_template('portal_rating.rating_widget_stars_static', values={
                            'rating_avg': product.rating_avg,
                            'rating_count': product.rating_count,
                        })
                    res['products'].append(res_product)

                return res
        return {}

    @http.route('/shop/products/recently_viewed_update', type='json', auth='public', website=True)
    def products_recently_viewed_update(self, product_id, **kwargs):
        res = {}
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request(force_create=True)
        if visitor_sudo:
            if request.httprequest.cookies.get('visitor_uuid', '') != visitor_sudo.access_token:
                res['visitor_uuid'] = visitor_sudo.access_token
            visitor_sudo._add_viewed_product(product_id)
        return res

    @http.route('/shop/products/recently_viewed_delete', type='json', auth='public', website=True)
    def products_recently_viewed_delete(self, product_id, **kwargs):
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            request.env['website.track'].sudo().search([('visitor_id', '=', visitor_sudo.id), ('product_id', '=', product_id)]).unlink()
        return self._get_products_recently_viewed()

    # --------------------------------------------------------------------------
    # Website Snippet Filters
    # --------------------------------------------------------------------------

    @http.route('/website_sale/snippet/options_filters', type='json', auth='user', website=True)
    def get_dynamic_snippet_filters(self):
        domain = expression.AND([
            request.website.website_domain(),
            ['|', ('filter_id.model_id', '=', 'product.product'), ('action_server_id.model_id.model', '=', 'product.product')]
        ])
        filters = request.env['website.snippet.filter'].sudo().search_read(
            domain, ['id']
        )
        return filters

```

## File: controllers\variant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import http
from odoo.http import request

from odoo.addons.sale.controllers.variant import VariantController

class WebsiteSaleVariantController(VariantController):
    @http.route(['/sale/get_combination_info_website'], type='json', auth="public", methods=['POST'], website=True)
    def get_combination_info_website(self, product_template_id, product_id, combination, add_qty, **kw):
        """Special route to use website logic in get_combination_info override.
        This route is called in JS by appending _website to the base route.
        """
        kw.pop('pricelist_id')
        res = self.get_combination_info(product_template_id, product_id, combination, add_qty, request.website.get_current_pricelist(), **kw)

        carousel_view = request.env['ir.ui.view']._render_template('website_sale.shop_product_carousel',
            values={
                'product': request.env['product.template'].browse(res['product_template_id']),
                'product_variant': request.env['product.product'].browse(res['product_id']),
            })
        res['carousel'] = carousel_view
        return res

    @http.route(auth="public")
    def create_product_variant(self, product_template_id, product_template_attribute_value_ids, **kwargs):
        """Override because on the website the public user must access it."""
        return super(WebsiteSaleVariantController, self).create_product_variant(product_template_id, product_template_attribute_value_ids, **kwargs)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import backend
from . import main
from . import variant

```

## File: data\data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="menu_shop" model="website.menu">
            <field name="name">Shop</field>
            <field name="url">/shop</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">20</field>
        </record>
        <record id="action_open_website" model="ir.actions.act_url">
            <field name="name">Website Shop</field>
            <field name="target">self</field>
            <field name="url">/shop</field>
        </record>
        <record id="base.open_menu" model="ir.actions.todo">
            <field name="action_id" ref="action_open_website"/>
            <field name="state">open</field>
        </record>

        <record id="website_sale.sale_ribbon" model="product.ribbon">
            <field name="html">Sale</field>
            <field name="html_class">bg-success o_ribbon_left</field>
        </record>

        <record id="website_sale.sold_out_ribbon" model="product.ribbon">
            <field name="html">Sold out</field>
            <field name="html_class">bg-danger o_ribbon_left</field>
        </record>

        <record id="website_sale.new_ribbon" model="product.ribbon">
            <field name="html">New!</field>
            <field name="html_class">bg-primary o_ribbon_left</field>
        </record>

        <record id="sales_team.salesteam_website_sales" model="crm.team">
            <field name="active" eval="True"/>
        </record>

        <record model="website" id="website.default_website">
            <field name="salesteam_id" ref="sales_team.salesteam_website_sales"/>
        </record>

        <record model="product.pricelist" id="product.list0">
            <field name="selectable" eval="True" />
            <field name="website_id" eval="False"/>
        </record>

    </data>
    <data>
        <!-- Action Server for Dynamic Filter -->
        <record id="dynamic_snippet_products_action" model="ir.actions.server">
            <field name="name">Products Dynamic Snippet</field>
            <field name="model_id" ref="model_product_product"/>
            <field name="state">code</field>
            <field name="code">
ProductProduct = model.env['product.product']
FieldMonetary = model.env['ir.qweb.field.monetary']

website = request.website.get_current_website()
dynamic_filter = model.env.context.get('dynamic_filter')
limit = model.env.context.get('limit')
search_domain = model.env.context.get('search_domain')
get_rendering_data_structure = model.env.context.get('get_rendering_data_structure')
escape = dynamic_filter.escape_falsy_as_empty

domain = [('website_published', '=', True)] + website.website_domain() + (search_domain or [])
products = ProductProduct.search(domain, limit=limit)
_ = products.mapped('name')

monetary_options = {
    'display_currency': request.website.get_current_pricelist().currency_id,
}

max_nb_chars = 100
res_products = []
for product in products:
    res_product = product._read_format(['id', 'name', 'website_url', 'description_sale'])[0]
    res_product.update(product._get_combination_info_variant())

    if res_product['description_sale'] and len(res_product['description_sale']) > max_nb_chars:
        res_product['description_sale'] = "%s ..." % res_product['description_sale'][:max_nb_chars]
    res_product['list_price'] = FieldMonetary.value_to_html(res_product['price'], monetary_options)
    data = get_rendering_data_structure()
    for field_name in dynamic_filter.field_names.split(","):
        field = ProductProduct._fields.get(field_name)
        if field and field.type == 'binary':
            data['image_fields'][field_name] = escape(website.image_url(product, field_name))
        elif field_name == 'list_price':
            data['fields'][field_name] = res_product[field_name]
        else:
            data['fields'][field_name] = escape(res_product[field_name])
    data['fields']['call_to_action_url'] = escape(product['website_url'])
    res_products.append(data)

response = res_products
            </field>
        </record>
        <!-- Dynamic Filter -->
        <record id="dynamic_filter_demo_products" model="website.snippet.filter">
            <field name="action_server_id" ref="website_sale.dynamic_snippet_products_action"/>
            <field name="field_names">display_name,description_sale,image_512,list_price</field>
            <field name="limit" eval="16"/>
            <field name="name">Products</field>
            <field name="website_id" ref="website.default_website"/>
        </record>
        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>sale.order</value>
            <value eval="[
                'client_order_ref',
            ]"/>
        </function>

        <record id="base.model_res_partner" model="ir.model">
            <field name="website_form_key">create_customer</field>
            <field name="website_form_access">True</field>
            <field name="website_form_label">Create a Customer</field>
        </record>
        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>res.partner</value>
            <value eval="[
                'name', 'phone', 'email',
                'city', 'zip', 'street', 'street2', 'state_id', 'country_id',
                'vat', 'company_name'
            ]"/>
        </function>
    </data>
</odoo>

```

## File: data\demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <record model="website" id="website.website2">
            <field name="salesteam_id" ref="sales_team.salesteam_website_sales"/>
        </record>

        <record id="product.product_product_24" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_5" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_12" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_10" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_13" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_25" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.consu_delivery_02" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_delivery_01" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_3" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_22" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.consu_delivery_03" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_27" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_delivery_02" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_product_16" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.consu_delivery_01" model="product.product">
            <field name="is_published" eval="True"/>
        </record>
        <record id="product.product_order_01" model="product.product">
            <field name="is_published" eval="True"/>
        </record>

        <record id="product.product_product_4" model="product.product">
            <field name="is_published" eval="True"/>
            <field name="website_size_x">2</field>
            <field name="website_size_y">2</field>
            <field name="website_sequence">9950</field>
        </record>

        <record id="product.product_product_6" model="product.product">
            <field name="is_published" eval="True"/>
        </record>

        <record id="product.product_product_7" model="product.product">
            <field name="is_published" eval="True"/>
        </record>

        <record id="product.product_product_8" model="product.product">
            <field name="is_published" eval="True"/>
        </record>

        <record id="product.product_product_9" model="product.product">
            <field name="is_published" eval="True"/>
        </record>

        <record id="product.product_product_11" model="product.product">
            <field name="is_published" eval="True"/>
            <field name="accessory_product_ids" eval="[(6, 0, [ref('product.product_product_7')])]"/>
        </record>

        <record id="item1" model="product.pricelist.item">
            <field name="base">list_price</field>
            <field name="applied_on">1_product</field>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="product_tmpl_id" ref="product.product_product_4_product_template"/>
            <field name="price_discount">20</field>
            <field name="min_quantity">2</field>
            <field name="compute_price">formula</field>
        </record>

    <!-- product.public.category -->

        <record id="public_category_desks" model="product.public.category">
          <field name="name">Desks</field>
          <field name="sequence">15</field>
        </record>
        <record id="public_category_furnitures" model="product.public.category">
          <field name="name">Furnitures</field>
          <field name="sequence">17</field>
        </record>
        <record id="public_category_boxes" model="product.public.category">
            <field name="name">Boxes</field>
            <field name="sequence">20</field>
        </record>
        <record id="public_category_drawers" model="product.public.category">
          <field name="name">Drawers</field>
          <field name="sequence">21</field>
        </record>
        <record id="public_category_cabinets" model="product.public.category">
          <field name="name">Cabinets</field>
          <field name="sequence">22</field>
        </record>
        <record id="public_category_bins" model="product.public.category">
          <field name="name">Bins</field>
          <field name="sequence">23</field>
        </record>
        <record id="public_category_lamps" model="product.public.category">
          <field name="name">Lamps</field>
          <field name="sequence">24</field>
        </record>
        <record id="services" model="product.public.category">
          <field name="name">Services</field>
          <field name="sequence">25</field>
        </record>
        <record id="public_category_multimedia" model="product.public.category">
          <field name="name">Multimedia</field>
          <field name="sequence">26</field>
        </record>

        <!-- subcategories -->
        <record id="public_category_desks_components" model="product.public.category">
          <field name="parent_id" eval="ref('public_category_desks')"/>
          <field name="name">Components</field>
          <field name="sequence">16</field>
        </record>
        <record id="public_category_furnitures_chairs" model="product.public.category">
          <field name="parent_id" eval="ref('public_category_furnitures')"/>
          <field name="name">Chairs</field>
          <field name="sequence">18</field>
        </record>
        <record id="public_category_furnitures_couches" model="product.public.category">
          <field name="parent_id" eval="ref('public_category_furnitures')"/>
          <field name="name">Couches</field>
          <field name="sequence">19</field>
        </record>

        <record id="product.product_product_1_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('services')])]"/>
        </record>
        <record id="product.product_product_2_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('services')])]"/>
        </record>
        <record id="product.product_product_3_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks_components')])]"/>
        </record>
        <record id="product.consu_delivery_03_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks')])]"/>
        </record>
        <record id="product.product_product_4_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks')])]"/>
        </record>
        <record id="product.product_product_5_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks')])]"/>
        </record>
        <record id="product.product_product_6_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_cabinets')])]"/>
        </record>
        <record id="product.product_product_7_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_boxes')])]"/>
        </record>
        <record id="product.product_product_8_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks')])]"/>
        </record>
        <record id="product.product_product_9_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_bins')])]"/>
        </record>
        <record id="product.product_product_10_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_cabinets')])]"/>
        </record>
        <record id="product.product_product_11_product_template" model="product.template">
            <field name="website_sequence">9990</field>
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_furnitures_chairs')])]"/>
        </record>
        <record id="product.product_product_12_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_furnitures_chairs')])]"/>
        </record>
        <record id="product.product_product_13_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks')])]"/>
        </record>
        <record id="product.product_product_16_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_drawers')])]"/>
        </record>
        <record id="product.product_product_20_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks_components')])]"/>
        </record>
        <record id="product.product_product_22_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks_components')])]"/>
        </record>
        <record id="product.product_product_25_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks_components')])]"/>
        </record>
        <record id="product.product_product_27_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_drawers')])]"/>
        </record>
        <record id="product.product_order_01_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_multimedia')])]"/>
        </record>
        <record id="product.consu_delivery_01_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_furnitures_couches')])]"/>
        </record>
        <record id="product.consu_delivery_02_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks')])]"/>
        </record>
        <record id="product.consu_delivery_03_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_desks')])]"/>
        </record>
        <record id="product.product_delivery_01_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_furnitures_chairs')])]"/>
        </record>
        <record id="product.product_delivery_02_product_template" model="product.template">
            <field name="public_categ_ids" eval="[(6,0,[ref('public_category_lamps')])]"/>
        </record>

        <record model="product.pricelist" id="product.list0">
            <field name="selectable" eval="True" />
            <field name="sequence">3</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="benelux" model="res.country.group">
            <field name="name">BeNeLux</field>
            <field name="country_ids" eval="[(6,0,[
                ref('base.be'),ref('base.lu'),ref('base.nl')])]"/>
        </record>

        <record id="list_christmas" model="product.pricelist">
            <field name="name">Christmas</field>
            <field name="selectable" eval="False" />
            <field name="website_id" ref="website.default_website" />
            <field name="country_group_ids" eval="[(6,0,[ref('base.europe')])]" />
            <field name="sequence">20</field>
        </record>
        <record id="item_christmas" model="product.pricelist.item">
            <field name="pricelist_id" ref="list_christmas"/>
            <field name="compute_price">formula</field>
            <field name="base">list_price</field>
            <field name="price_discount">20</field>
        </record>

        <record id="list_benelux" model="product.pricelist">
            <field name="name">Benelux</field>
            <field name="selectable" eval="False" />
            <field name="website_id" ref="website.default_website" />
            <field name="country_group_ids" eval="[(6,0,[ref('benelux')])]" />
            <field name="sequence">2</field>
        </record>
        <record id="item_benelux" model="product.pricelist.item">
            <field name="pricelist_id" ref="list_benelux"/>
            <field name="compute_price">percentage</field>
            <field name="base">list_price</field>
            <field name="percent_price">10</field>
            <field name="currency_id" ref="base.EUR"/>
        </record>


        <record id="list_europe" model="product.pricelist">
            <field name="name">EUR</field>
            <field name="selectable" eval="True" />
            <field name="website_id" ref="website.default_website" />
            <field name="country_group_ids" eval="[(6,0,[ref('base.europe')])]" />
            <field name="sequence">3</field>
            <field name="currency_id" ref="base.EUR"/>
        </record>
        <record id="item_europe" model="product.pricelist.item">
            <field name="pricelist_id" ref="list_europe"/>
            <field name="compute_price">formula</field>
            <field name="base">list_price</field>
        </record>

        <record id="item_us" model="product.pricelist.item">
            <field name="pricelist_id" ref="product.list0"/>
            <field name="compute_price">formula</field>
            <field name="base">list_price</field>
        </record>

        <!-- Add demo-data for pretty website sales graph (for the sales dashboard) -->
        <record id="website_sale_order_1" model="sale.order">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=7)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_1" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_1"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_6').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_6"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">599.0</field>
        </record>

        <record id="website_sale_order_2" model="sale.order">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=6)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_2" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_2"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_4').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_4"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">900</field>
        </record>

        <record id="website_sale_order_3" model="sale.order">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor2'))]"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_3" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_3"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_4').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_4"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">750</field>
        </record>

        <record id="website_sale_order_4" model="sale.order">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=4)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_4" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_4"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_8').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_8"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">1199.0</field>
        </record>

        <record id="website_sale_order_5" model="sale.order">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_5" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_5"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_4').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_4"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">349.0</field>
        </record>

        <record id="website_sale_order_6" model="sale.order">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_6" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_6"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_8').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_8"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">1599.00</field>
        </record>

        <record id="website_sale_order_7" model="sale.order">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_7" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_7"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_8').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_8"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">1349.00</field>
        </record>

        <record id="website_sale_order_8" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="datetime.now()"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor1'))]"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_8" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_8"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_8').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_8"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">1799.00</field>
        </record>

        <record id="website_sale_order_9" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(hours=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="website_sale_order_line_9" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_9"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">2950.00</field>
        </record>

        <record id="website_sale_order_line_10" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_9"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">12.50</field>
        </record>

        <!-- Active Carts -->
        <record id="website_sale_order_10" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="datetime.now()"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor5'))]"/>
        </record>

        <record id="website_sale_order_line_11" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_10"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_11').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_11"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">16.50</field>
        </record>

        <!-- Abandoned Carts -->
        <record id="website_sale_order_11" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-timedelta(hours=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="website_sale_order_line_12" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_11"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_9').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_9"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">47.0</field>
        </record>

        <!-- Payments to Capture -->
        <record id="website_sale_order_13" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="payment_term_id" ref="account.account_payment_term_immediate"/>
            <field name="date_order" eval="(datetime.now()-timedelta(hours=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="state">sent</field>
        </record>

        <record id="website_sale_order_line_14" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_13"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_8').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_8"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">1799.0</field>
        </record>

        <!-- Order to Invoice -->
        <record id="website_sale_order_14" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
        </record>

        <record id="website_sale_order_line_15" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_14"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_16').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_16"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">25.0</field>
        </record>

        <record id="website_sale_order_16" model="sale.order">
            <field name="create_date" eval="datetime.now() - relativedelta(months=1)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="datetime.now()-relativedelta(months=1)"/>
            <field name="state">sale</field>
        </record>

        <record id="website_sale_order_line_16" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_16"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_8').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_8"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">1799.0</field>
        </record>

        <record id="website_sale_order_17" model="sale.order">
            <field name="create_date" eval="datetime.now() - relativedelta(months=1, days=2)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="datetime.now()-relativedelta(months=1, days=2)"/>
        </record>

        <record id="website_sale_order_line_17" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_17"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_9').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_9"/>
            <field name="product_uom_qty">7</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">47.0</field>
            <field name="invoice_status">to invoice</field>
        </record>

        <record id="website_sale_order_18" model="sale.order">
            <field name="create_date" eval="datetime.now() - relativedelta(months=2)"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="datetime.now()-relativedelta(months=2)"/>
        </record>

        <record id="website_sale_order_line_18" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_18"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_9').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_9"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">47.0</field>
            <field name="invoice_status">to invoice</field>
        </record>


        <!-- action_confirm for confirmation date -->
        <function model="sale.order" name="action_confirm" eval="[[ref('website_sale_order_14')]]"/>

        <record id="product_product_1_product_template" model="product.template">
            <field name="name">Warranty</field>
            <field name="list_price">20.0</field>
            <field name="website_sequence">9980</field>
            <field name="is_published" eval="True"/>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Warranty, issued to the purchaser of an article by its manufacturer, promising to repair or replace it if necessary within a specified period of time.</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="invoice_policy">delivery</field>
            <field name="public_categ_ids" eval="[(6, 0, [ref('website_sale.services')])]"/>
        </record>

        <record id="product_1_attribute_3_product_template_attribute_line" model="product.template.attribute.line">
            <field name="product_tmpl_id" ref="website_sale.product_product_1_product_template"/>
            <field name="attribute_id" ref="product.product_attribute_3"/>
            <field name="value_ids" eval="[(6,0,[ref('product.product_attribute_value_5'), ref('product.product_attribute_value_6')])]"/>
        </record>

        <!-- Handle automatically created product.template.attribute.value -->
        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'website_sale.product_1_attribute_3_value_1',
                'record': obj().env.ref('website_sale.product_1_attribute_3_product_template_attribute_line').product_template_value_ids[0],
                'noupdate': True,
            }, {
                'xml_id': 'website_sale.product_1_attribute_3_value_2',
                'record': obj().env.ref('website_sale.product_1_attribute_3_product_template_attribute_line').product_template_value_ids[1],
                'noupdate': True,
            }]"/>
        </function>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'website_sale.product_product_1',
                'record': obj().env.ref('website_sale.product_product_1_product_template')._get_variant_for_combination(obj().env.ref('website_sale.product_1_attribute_3_value_1')),
                'noupdate': True,
            }, {
                'xml_id': 'website_sale.product_product_1b',
                'record': obj().env.ref('website_sale.product_product_1_product_template')._get_variant_for_combination(obj().env.ref('website_sale.product_1_attribute_3_value_2')),
                'noupdate': True,
            },]"/>
        </function>

        <record id="product_product_1" model="product.product">
            <field name="default_code">SERV_125889</field>
        </record>
        <record id="product_product_1b" model="product.product">
            <field name="default_code">SERV_125890</field>
        </record>

        <record id="website_sale.product_1_attribute_3_value_2" model="product.template.attribute.value">
            <field name="price_extra">18.00</field>
        </record>

        <record id="website_sale_activity_1" model="mail.activity">
            <field name="res_id" ref="website_sale.website_sale_order_3"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')" />
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="website_sale_activity_2" model="mail.activity">
            <field name="res_id" ref="website_sale.website_sale_order_8"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Follow-up on satisfaction</field>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="website_sale_activity_3" model="mail.activity">
            <field name="res_id" ref="website_sale.website_sale_order_9"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Confirm quote</field>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="website_sale_activity_5" model="mail.activity">
            <field name="res_id" ref="website_sale.website_sale_order_11"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Send updated pricelist</field>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>

</odoo>

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest.digest_digest_default" model="digest.digest">
            <field name="kpi_website_sale_total">True</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_sale_cart_recovery" model="mail.template">
            <field name="name">Sales Order: Cart Recovery Email</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="subject">You left items in your cart!</field>
            <field name="email_from">${(object.user_id.email_formatted or user.email_formatted or '') | safe}</field>
            <field name="partner_to">${object.partner_id.id}</field>
            <field name="body_html" type="xml">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    % set company = object.company_id or object.user_id.company_id or user.company_id
                    <span style="font-size: 10px;">Your Cart</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        ${object.name}
                    </span>
                </td><td valign="middle" align="right">
                    <img src="/logo.png?company=${company.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${company.name}"/>
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
                <tr><td valign="top" style="font-size: 13px;">
                    <h1 style="color:#A9A9A9;">THERE'S SOMETHING IN YOUR CART.</h1>
                    Would you like to complete your purchase?<br/><br/>
                    % if object.order_line:
                        % for line in object.website_order_line:
                            <hr/>
                            <table width="100%">
                                <tr>
                                    <td style="padding: 10px; width:150px;">
                                        <img src="/web/image/product.product/${line.product_id.id}/image_128" style="width: 100px; height: 100px; object-fit: contain;" alt="Product image"></img>
                                    </td>
                                    <td>
                                        <strong>${line.product_id.display_name}</strong><br/>${line.name}
                                    </td>
                                    <td width="100px" align="right">
                                        ${(line.product_uom_qty) | int} ${(line.product_uom.name)}
                                    </td>
                                </tr>
                            </table>
                        % endfor
                        <hr/>
                    % endif
                    <div style="text-align: center; margin: 16px 0px 16px 0px; font-size: 14px;">
                        <a href="${object.get_base_url()}/shop/cart?access_token=${object.access_token}"
                            target="_blank"
                            style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                            Resume order
                        </a>
                    </div>
                    <div style="text-align: center;"><strong>Thank you for shopping with ${company.name}!</strong></div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                    ${company.name}
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    ${company.phone}
                    % if company.email
                        | <a href="'mailto:%s' % ${company.email}" style="text-decoration:none; color: #454748;">${company.email}</a>
                    % endif
                    % if company.website
                        | <a href="'%s' % ${company.website}" style="text-decoration:none; color: #454748;">${company.website}</a>
                    % endif
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
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class AccountMove(models.Model):
    _inherit = 'account.move'

    website_id = fields.Many2one('website', related='partner_id.website_id', string='Website',
                                 help='Website through which this invoice was created.',
                                 store=True, readonly=True, tracking=True)

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime
from dateutil.relativedelta import relativedelta

from odoo import fields,api, models, _
from odoo.exceptions import UserError, ValidationError


class CrmTeam(models.Model):
    _inherit = "crm.team"

    website_ids = fields.One2many('website', 'salesteam_id', string='Websites', help="Websites using this Sales Team")
    abandoned_carts_count = fields.Integer(
        compute='_compute_abandoned_carts',
        string='Number of Abandoned Carts', readonly=True)
    abandoned_carts_amount = fields.Integer(
        compute='_compute_abandoned_carts',
        string='Amount of Abandoned Carts', readonly=True)

    def _compute_abandoned_carts(self):
        # abandoned carts to recover are draft sales orders that have no order lines,
        # a partner other than the public user, and created over an hour ago
        # and the recovery mail was not yet sent
        counts = {}
        amounts = {}
        website_teams = self.filtered(lambda team: team.website_ids)
        if website_teams:
            abandoned_carts_data = self.env['sale.order'].read_group([
                ('is_abandoned_cart', '=', True),
                ('cart_recovery_email_sent', '=', False),
                ('team_id', 'in', website_teams.ids),
            ], ['amount_total', 'team_id'], ['team_id'])
            counts = {data['team_id'][0]: data['team_id_count'] for data in abandoned_carts_data}
            amounts = {data['team_id'][0]: data['amount_total'] for data in abandoned_carts_data}
        for team in self:
            team.abandoned_carts_count = counts.get(team.id, 0)
            team.abandoned_carts_amount = amounts.get(team.id, 0)

    def get_abandoned_carts(self):
        self.ensure_one()
        return {
            'name': _('Abandoned Carts'),
            'type': 'ir.actions.act_window',
            'view_mode': 'tree,form',
            'domain': [('is_abandoned_cart', '=', True)],
            'search_view_id': [self.env.ref('sale.sale_order_view_search_inherit_sale').id],
            'context': {
                'search_default_team_id': self.id,
                'default_team_id': self.id,
                'search_default_recovery_email': 1,
                'create': False
            },
            'res_model': 'sale.order',
            'help': _('''<p class="o_view_nocontent_smiling_face">
                        You can find all abandoned carts here, i.e. the carts generated by your website's visitors from over an hour ago that haven't been confirmed yet.</p>
                        <p>You should send an email to the customers to encourage them!</p>
                    '''),
        }

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import AccessError


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_website_sale_total = fields.Boolean('eCommerce Sales')
    kpi_website_sale_total_value = fields.Monetary(compute='_compute_kpi_website_sale_total_value')

    def _compute_kpi_website_sale_total_value(self):
        if not self.env.user.has_group('sales_team.group_sale_salesman_all_leads'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            confirmed_website_sales = self.env['sale.order'].search([
                ('date_order', '>=', start),
                ('date_order', '<', end),
                ('state', 'not in', ['draft', 'cancel', 'sent']),
                ('website_id', '!=', False),
                ('company_id', '=', company.id)
            ])
            record.kpi_website_sale_total_value = sum(confirmed_website_sales.mapped('amount_total'))

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_website_sale_total'] = 'website.backend_dashboard&menu_id=%s' % self.env.ref('website.menu_website_configuration').id
        return res

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.http import request


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _dispatch(cls):
        affiliate_id = request.httprequest.args.get('affiliate_id')
        if affiliate_id:
            request.session['affiliate_id'] = int(affiliate_id)
        return super(IrHttp, cls)._dispatch()

```

## File: models\mail_compose_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class MailComposeMessage(models.TransientModel):
    _inherit = 'mail.compose.message'

    def send_mail(self, auto_commit=False):
        context = self._context
        # TODO TDE: clean that brole one day
        if context.get('website_sale_send_recovery_email') and self.model == 'sale.order' and context.get('active_ids'):
            self.env['sale.order'].search([
                ('id', 'in', context.get('active_ids')),
                ('cart_recovery_email_sent', '=', False),
                ('is_abandoned_cart', '=', True)
            ]).write({'cart_recovery_email_sent': True})
            self = self.with_context(mail_post_autofollow=True)
        return super(MailComposeMessage, self).send_mail(auto_commit=auto_commit)

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError, UserError
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website.models import ir_http
from odoo.tools.translate import html_translate
from odoo.osv import expression
from psycopg2.extras import execute_values

_logger = logging.getLogger(__name__)


class ProductRibbon(models.Model):
    _name = "product.ribbon"
    _description = 'Product ribbon'

    def name_get(self):
        return [(ribbon.id, '%s (#%d)' % (tools.html2plaintext(ribbon.html), ribbon.id)) for ribbon in self]

    html = fields.Char(string='Ribbon html', required=True, translate=True)
    bg_color = fields.Char(string='Ribbon background color', required=False)
    text_color = fields.Char(string='Ribbon text color', required=False)
    html_class = fields.Char(string='Ribbon class', required=True, default='')


class ProductPricelist(models.Model):
    _inherit = "product.pricelist"

    def _default_website(self):
        """ Find the first company's website, if there is one. """
        company_id = self.env.company.id

        if self._context.get('default_company_id'):
            company_id = self._context.get('default_company_id')

        domain = [('company_id', '=', company_id)]
        return self.env['website'].search(domain, limit=1)

    website_id = fields.Many2one('website', string="Website", ondelete='restrict', default=_default_website, domain="[('company_id', '=?', company_id)]")
    code = fields.Char(string='E-commerce Promotional Code', groups="base.group_user")
    selectable = fields.Boolean(help="Allow the end user to choose this price list")

    def clear_cache(self):
        # website._get_pl_partner_order() is cached to avoid to recompute at each request the
        # list of available pricelists. So, we need to invalidate the cache when
        # we change the config of website price list to force to recompute.
        website = self.env['website']
        website._get_pl_partner_order.clear_cache(website)

    @api.model
    def create(self, data):
        if data.get('company_id') and not data.get('website_id'):
            # l10n modules install will change the company currency, creating a
            # pricelist for that currency. Do not use user's company in that
            # case as module install are done with OdooBot (company 1)
            self = self.with_context(default_company_id=data['company_id'])
        res = super(ProductPricelist, self).create(data)
        self.clear_cache()
        return res

    def write(self, data):
        res = super(ProductPricelist, self).write(data)
        if data.keys() & {'code', 'active', 'website_id', 'selectable', 'company_id'}:
            self._check_website_pricelist()
        self.clear_cache()
        return res

    def unlink(self):
        res = super(ProductPricelist, self).unlink()
        self._check_website_pricelist()
        self.clear_cache()
        return res

    def _get_partner_pricelist_multi_search_domain_hook(self, company_id):
        domain = super(ProductPricelist, self)._get_partner_pricelist_multi_search_domain_hook(company_id)
        website = ir_http.get_request_website()
        if website:
            domain += self._get_website_pricelists_domain(website.id)
        return domain

    def _get_partner_pricelist_multi_filter_hook(self):
        res = super(ProductPricelist, self)._get_partner_pricelist_multi_filter_hook()
        website = ir_http.get_request_website()
        if website:
            res = res.filtered(lambda pl: pl._is_available_on_website(website.id))
        return res

    def _check_website_pricelist(self):
        for website in self.env['website'].search([]):
            # sudo() to be able to read pricelists/website from another company
            if not website.sudo().pricelist_ids:
                raise UserError(_("With this action, '%s' website would not have any pricelist available.") % (website.name))

    def _is_available_on_website(self, website_id):
        """ To be able to be used on a website, a pricelist should either:
        - Have its `website_id` set to current website (specific pricelist).
        - Have no `website_id` set and should be `selectable` (generic pricelist)
          or should have a `code` (generic promotion).
        - Have no `company_id` or a `company_id` matching its website one.

        Note: A pricelist without a website_id, not selectable and without a
              code is a backend pricelist.

        Change in this method should be reflected in `_get_website_pricelists_domain`.
        """
        self.ensure_one()
        if self.company_id and self.company_id != self.env["website"].browse(website_id).company_id:
            return False
        return self.website_id.id == website_id or (not self.website_id and (self.selectable or self.sudo().code))

    def _get_website_pricelists_domain(self, website_id):
        ''' Check above `_is_available_on_website` for explanation.
        Change in this method should be reflected in `_is_available_on_website`.
        '''
        company_id = self.env["website"].browse(website_id).company_id.id
        return [
            '&', ('company_id', 'in', [False, company_id]),
            '|', ('website_id', '=', website_id),
            '&', ('website_id', '=', False),
            '|', ('selectable', '=', True), ('code', '!=', False),
        ]

    def _get_partner_pricelist_multi(self, partner_ids, company_id=None):
        ''' If `property_product_pricelist` is read from website, we should use
            the website's company and not the user's one.
            Passing a `company_id` to super will avoid using the current user's
            company.
        '''
        website = ir_http.get_request_website()
        if not company_id and website:
            company_id = website.company_id.id
        return super(ProductPricelist, self)._get_partner_pricelist_multi(partner_ids, company_id)

    @api.constrains('company_id', 'website_id')
    def _check_websites_in_company(self):
        '''Prevent misconfiguration multi-website/multi-companies.
           If the record has a company, the website should be from that company.
        '''
        for record in self.filtered(lambda pl: pl.website_id and pl.company_id):
            if record.website_id.company_id != record.company_id:
                raise ValidationError(_("""Only the company's websites are allowed.\nLeave the Company field empty or select a website from that company."""))


class ProductPublicCategory(models.Model):
    _name = "product.public.category"
    _inherit = ["website.seo.metadata", "website.multi.mixin", 'image.mixin']
    _description = "Website Product Category"
    _parent_store = True
    _order = "sequence, name, id"

    def _default_sequence(self):
        cat = self.search([], limit=1, order="sequence DESC")
        if cat:
            return cat.sequence + 5
        return 10000

    name = fields.Char(required=True, translate=True)
    parent_id = fields.Many2one('product.public.category', string='Parent Category', index=True, ondelete="cascade")
    parent_path = fields.Char(index=True)
    child_id = fields.One2many('product.public.category', 'parent_id', string='Children Categories')
    parents_and_self = fields.Many2many('product.public.category', compute='_compute_parents_and_self')
    sequence = fields.Integer(help="Gives the sequence order when displaying a list of product categories.", index=True, default=_default_sequence)
    website_description = fields.Html('Category Description', sanitize_attributes=False, translate=html_translate, sanitize_form=False)
    product_tmpl_ids = fields.Many2many('product.template', relation='product_public_category_product_template_rel')

    @api.constrains('parent_id')
    def check_parent_id(self):
        if not self._check_recursion():
            raise ValueError(_('Error ! You cannot create recursive categories.'))

    def name_get(self):
        res = []
        for category in self:
            res.append((category.id, " / ".join(category.parents_and_self.mapped('name'))))
        return res

    def _compute_parents_and_self(self):
        for category in self:
            if category.parent_path:
                category.parents_and_self = self.env['product.public.category'].browse([int(p) for p in category.parent_path.split('/')[:-1]])
            else:
                category.parents_and_self = category


class ProductTemplate(models.Model):
    _inherit = ["product.template", "website.seo.metadata", 'website.published.multi.mixin', 'rating.mixin']
    _name = 'product.template'
    _mail_post_access = 'read'
    _check_company_auto = True

    website_description = fields.Html('Description for the website', sanitize_attributes=False, translate=html_translate, sanitize_form=False)
    alternative_product_ids = fields.Many2many(
        'product.template', 'product_alternative_rel', 'src_id', 'dest_id', check_company=True,
        string='Alternative Products', help='Suggest alternatives to your customer (upsell strategy). '
                                            'Those products show up on the product page.')
    accessory_product_ids = fields.Many2many(
        'product.product', 'product_accessory_rel', 'src_id', 'dest_id', string='Accessory Products', check_company=True,
        help='Accessories show up when the customer reviews the cart before payment (cross-sell strategy).')
    website_size_x = fields.Integer('Size X', default=1)
    website_size_y = fields.Integer('Size Y', default=1)
    website_ribbon_id = fields.Many2one('product.ribbon', string='Ribbon')
    website_sequence = fields.Integer('Website Sequence', help="Determine the display order in the Website E-commerce",
                                      default=lambda self: self._default_website_sequence(), copy=False)
    public_categ_ids = fields.Many2many(
        'product.public.category', relation='product_public_category_product_template_rel',
        string='Website Product Category',
        help="The product will be available in each mentioned eCommerce category. Go to Shop > "
             "Customize and enable 'eCommerce categories' to view all eCommerce categories.")

    product_template_image_ids = fields.One2many('product.image', 'product_tmpl_id', string="Extra Product Media", copy=True)

    def _has_no_variant_attributes(self):
        """Return whether this `product.template` has at least one no_variant
        attribute.

        :return: True if at least one no_variant attribute, False otherwise
        :rtype: bool
        """
        self.ensure_one()
        return any(a.create_variant == 'no_variant' for a in self.valid_product_template_attribute_line_ids.attribute_id)

    def _has_is_custom_values(self):
        self.ensure_one()
        """Return whether this `product.template` has at least one is_custom
        attribute value.

        :return: True if at least one is_custom attribute value, False otherwise
        :rtype: bool
        """
        return any(v.is_custom for v in self.valid_product_template_attribute_line_ids.product_template_value_ids._only_active())

    def _get_possible_variants_sorted(self, parent_combination=None):
        """Return the sorted recordset of variants that are possible.

        The order is based on the order of the attributes and their values.

        See `_get_possible_variants` for the limitations of this method with
        dynamic or no_variant attributes, and also for a warning about
        performances.

        :param parent_combination: combination from which `self` is an
            optional or accessory product
        :type parent_combination: recordset `product.template.attribute.value`

        :return: the sorted variants that are possible
        :rtype: recordset of `product.product`
        """
        self.ensure_one()

        def _sort_key_attribute_value(value):
            # if you change this order, keep it in sync with _order from `product.attribute`
            return (value.attribute_id.sequence, value.attribute_id.id)

        def _sort_key_variant(variant):
            """
                We assume all variants will have the same attributes, with only one value for each.
                    - first level sort: same as "product.attribute"._order
                    - second level sort: same as "product.attribute.value"._order
            """
            keys = []
            for attribute in variant.product_template_attribute_value_ids.sorted(_sort_key_attribute_value):
                # if you change this order, keep it in sync with _order from `product.attribute.value`
                keys.append(attribute.product_attribute_value_id.sequence)
                keys.append(attribute.id)
            return keys

        return self._get_possible_variants(parent_combination).sorted(_sort_key_variant)

    def _get_combination_info(self, combination=False, product_id=False, add_qty=1, pricelist=False, parent_combination=False, only_template=False):
        """Override for website, where we want to:
            - take the website pricelist if no pricelist is set
            - apply the b2b/b2c setting to the result

        This will work when adding website_id to the context, which is done
        automatically when called from routes with website=True.
        """
        self.ensure_one()

        current_website = False

        if self.env.context.get('website_id'):
            current_website = self.env['website'].get_current_website()
            if not pricelist:
                pricelist = current_website.get_current_pricelist()

        combination_info = super(ProductTemplate, self)._get_combination_info(
            combination=combination, product_id=product_id, add_qty=add_qty, pricelist=pricelist,
            parent_combination=parent_combination, only_template=only_template)

        if self.env.context.get('website_id'):
            partner = self.env.user.partner_id
            company_id = current_website.company_id
            product = self.env['product.product'].browse(combination_info['product_id']) or self

            tax_display = self.user_has_groups('account.group_show_line_subtotals_tax_excluded') and 'total_excluded' or 'total_included'
            fpos = self.env['account.fiscal.position'].sudo().get_fiscal_position(partner.id)
            taxes = fpos.map_tax(product.sudo().taxes_id.filtered(lambda x: x.company_id == company_id), product, partner)

            # The list_price is always the price of one.
            quantity_1 = 1
            combination_info['price'] = self.env['account.tax']._fix_tax_included_price_company(combination_info['price'], product.sudo().taxes_id, taxes, company_id)
            price = taxes.compute_all(combination_info['price'], pricelist.currency_id, quantity_1, product, partner)[tax_display]
            if pricelist.discount_policy == 'without_discount':
                combination_info['list_price'] = self.env['account.tax']._fix_tax_included_price_company(combination_info['list_price'], product.sudo().taxes_id, taxes, company_id)
                list_price = taxes.compute_all(combination_info['list_price'], pricelist.currency_id, quantity_1, product, partner)[tax_display]
            else:
                list_price = price
            combination_info['price_extra'] = self.env['account.tax']._fix_tax_included_price_company(combination_info['price_extra'], product.sudo().taxes_id, taxes, company_id)
            price_extra = taxes.compute_all(combination_info['price_extra'], pricelist.currency_id, quantity_1, product, partner)[tax_display]
            has_discounted_price = pricelist.currency_id.compare_amounts(list_price, price) == 1

            combination_info.update(
                price=price,
                list_price=list_price,
                price_extra=price_extra,
                has_discounted_price=has_discounted_price,
            )

        return combination_info

    def _get_image_holder(self):
        """Returns the holder of the image to use as default representation.
        If the product template has an image it is the product template,
        otherwise if the product has variants it is the first variant

        :return: this product template or the first product variant
        :rtype: recordset of 'product.template' or recordset of 'product.product'
        """
        self.ensure_one()
        if self.image_128:
            return self
        variant = self.env['product.product'].browse(self._get_first_possible_variant_id())
        # if the variant has no image anyway, spare some queries by using template
        return variant if variant.image_variant_128 else self

    def _get_current_company_fallback(self, **kwargs):
        """Override: if a website is set on the product or given, fallback to
        the company of the website. Otherwise use the one from parent method."""
        res = super(ProductTemplate, self)._get_current_company_fallback(**kwargs)
        website = self.website_id or kwargs.get('website')
        return website and website.company_id or res

    def _init_column(self, column_name):
        # to avoid generating a single default website_sequence when installing the module,
        # we need to set the default row by row for this column
        if column_name == "website_sequence":
            _logger.debug("Table '%s': setting default value of new column %s to unique values for each row", self._table, column_name)
            self.env.cr.execute("SELECT id FROM %s WHERE website_sequence IS NULL" % self._table)
            prod_tmpl_ids = self.env.cr.dictfetchall()
            max_seq = self._default_website_sequence()
            query = """
                UPDATE {table}
                SET website_sequence = p.web_seq
                FROM (VALUES %s) AS p(p_id, web_seq)
                WHERE id = p.p_id
            """.format(table=self._table)
            values_args = [(prod_tmpl['id'], max_seq + i * 5) for i, prod_tmpl in enumerate(prod_tmpl_ids)]
            execute_values(self.env.cr._obj, query, values_args)
        else:
            super(ProductTemplate, self)._init_column(column_name)

    def _default_website_sequence(self):
        ''' We want new product to be the last (highest seq).
        Every product should ideally have an unique sequence.
        Default sequence (10000) should only be used for DB first product.
        As we don't resequence the whole tree (as `sequence` does), this field
        might have negative value.
        '''
        self._cr.execute("SELECT MAX(website_sequence) FROM %s" % self._table)
        max_sequence = self._cr.fetchone()[0]
        if max_sequence is None:
            return 10000
        return max_sequence + 5

    def set_sequence_top(self):
        min_sequence = self.sudo().search([], order='website_sequence ASC', limit=1)
        self.website_sequence = min_sequence.website_sequence - 5

    def set_sequence_bottom(self):
        max_sequence = self.sudo().search([], order='website_sequence DESC', limit=1)
        self.website_sequence = max_sequence.website_sequence + 5

    def set_sequence_up(self):
        previous_product_tmpl = self.sudo().search([
            ('website_sequence', '<', self.website_sequence),
            ('website_published', '=', self.website_published),
        ], order='website_sequence DESC', limit=1)
        if previous_product_tmpl:
            previous_product_tmpl.website_sequence, self.website_sequence = self.website_sequence, previous_product_tmpl.website_sequence
        else:
            self.set_sequence_top()

    def set_sequence_down(self):
        next_prodcut_tmpl = self.search([
            ('website_sequence', '>', self.website_sequence),
            ('website_published', '=', self.website_published),
        ], order='website_sequence ASC', limit=1)
        if next_prodcut_tmpl:
            next_prodcut_tmpl.website_sequence, self.website_sequence = self.website_sequence, next_prodcut_tmpl.website_sequence
        else:
            return self.set_sequence_bottom()

    def _default_website_meta(self):
        res = super(ProductTemplate, self)._default_website_meta()
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = self.description_sale
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = self.env['website'].image_url(self, 'image_1024')
        res['default_meta_description'] = self.description_sale
        return res

    def _compute_website_url(self):
        super(ProductTemplate, self)._compute_website_url()
        for product in self:
            if product.id:
                product.website_url = "/shop/%s" % slug(product)

    # ---------------------------------------------------------
    # Rating Mixin API
    # ---------------------------------------------------------

    def _rating_domain(self):
        """ Only take the published rating into account to compute avg and count """
        domain = super(ProductTemplate, self)._rating_domain()
        return expression.AND([domain, [('is_internal', '=', False)]])

    def _get_images(self):
        """Return a list of records implementing `image.mixin` to
        display on the carousel on the website for this template.

        This returns a list and not a recordset because the records might be
        from different models (template and image).

        It contains in this order: the main image of the template and the
        Template Extra Images.
        """
        self.ensure_one()
        return [self] + list(self.product_template_image_ids)


class Product(models.Model):
    _inherit = "product.product"

    website_id = fields.Many2one(related='product_tmpl_id.website_id', readonly=False)

    product_variant_image_ids = fields.One2many('product.image', 'product_variant_id', string="Extra Variant Images")

    website_url = fields.Char('Website URL', compute='_compute_product_website_url', help='The full URL to access the document through the website.')

    @api.depends_context('lang')
    @api.depends('product_tmpl_id.website_url', 'product_template_attribute_value_ids')
    def _compute_product_website_url(self):
        for product in self:
            attributes = ','.join(str(x) for x in product.product_template_attribute_value_ids.ids)
            product.website_url = "%s#attr=%s" % (product.product_tmpl_id.website_url, attributes)

    def website_publish_button(self):
        self.ensure_one()
        return self.product_tmpl_id.website_publish_button()

    def open_website_url(self):
        self.ensure_one()
        res = self.product_tmpl_id.open_website_url()
        res['url'] = self.website_url
        return res

    def _get_images(self):
        """Return a list of records implementing `image.mixin` to
        display on the carousel on the website for this variant.

        This returns a list and not a recordset because the records might be
        from different models (template, variant and image).

        It contains in this order: the main image of the variant (if set), the
        Variant Extra Images, and the Template Extra Images.
        """
        self.ensure_one()
        variant_images = list(self.product_variant_image_ids)
        if self.image_variant_1920:
            # if the main variant image is set, display it first
            variant_images = [self] + variant_images
        else:
            # If the main variant image is empty, it will fallback to template
            # image, in this case insert it after the other variant images, so
            # that all variant images are first and all template images last.
            variant_images = variant_images + [self]
        # [1:] to remove the main image from the template, we only display
        # the template extra images here
        return variant_images + self.product_tmpl_id._get_images()[1:]

    def _is_add_to_cart_allowed(self):
        self.ensure_one()
        return self.user_has_groups('base.group_system') or (self.sale_ok and self.website_published)

```

## File: models\product_attribute.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import OrderedDict

from odoo import models


class ProductTemplateAttributeLine(models.Model):
    _inherit = 'product.template.attribute.line'

    def _prepare_single_value_for_display(self):
        """On the product page group together the attribute lines that concern
        the same attribute and that have only one value each.

        Indeed those are considered informative values, they do not generate
        choice for the user, so they are displayed below the configurator.

        The returned attributes are ordered as they appear in `self`, so based
        on the order of the attribute lines.
        """
        single_value_lines = self.filtered(lambda ptal: len(ptal.value_ids) == 1)
        single_value_attributes = OrderedDict([(pa, self.env['product.template.attribute.line']) for pa in single_value_lines.attribute_id])
        for ptal in single_value_lines:
            single_value_attributes[ptal.attribute_id] |= ptal
        return single_value_attributes

```

## File: models\product_image.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError

from odoo.addons.website.tools import get_video_embed_code


class ProductImage(models.Model):
    _name = 'product.image'
    _description = "Product Image"
    _inherit = ['image.mixin']
    _order = 'sequence, id'

    name = fields.Char("Name", required=True)
    sequence = fields.Integer(default=10, index=True)

    image_1920 = fields.Image(required=True)

    product_tmpl_id = fields.Many2one('product.template', "Product Template", index=True, ondelete='cascade')
    product_variant_id = fields.Many2one('product.product', "Product Variant", index=True, ondelete='cascade')
    video_url = fields.Char('Video URL',
                            help='URL of a video for showcasing your product.')
    embed_code = fields.Char(compute="_compute_embed_code")

    can_image_1024_be_zoomed = fields.Boolean("Can Image 1024 be zoomed", compute='_compute_can_image_1024_be_zoomed', store=True)

    @api.depends('image_1920', 'image_1024')
    def _compute_can_image_1024_be_zoomed(self):
        for image in self:
            image.can_image_1024_be_zoomed = image.image_1920 and tools.is_image_size_above(image.image_1920, image.image_1024)

    @api.depends('video_url')
    def _compute_embed_code(self):
        for image in self:
            image.embed_code = get_video_embed_code(image.video_url)

    @api.constrains('video_url')
    def _check_valid_video_url(self):
        for image in self:
            if image.video_url and not image.embed_code:
                raise ValidationError(_("Provided video URL for '%s' is not valid. Please enter a valid video URL.", image.name))

    @api.model_create_multi
    def create(self, vals_list):
        """
            We don't want the default_product_tmpl_id from the context
            to be applied if we have a product_variant_id set to avoid
            having the variant images to show also as template images.
            But we want it if we don't have a product_variant_id set.
        """
        context_without_template = self.with_context({k: v for k, v in self.env.context.items() if k != 'default_product_tmpl_id'})
        normal_vals = []
        variant_vals_list = []

        for vals in vals_list:
            if vals.get('product_variant_id') and 'default_product_tmpl_id' in self.env.context:
                variant_vals_list.append(vals)
            else:
                normal_vals.append(vals)

        return super().create(normal_vals) + super(ProductImage, context_without_template).create(variant_vals_list)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    website_sale_onboarding_payment_acquirer_state = fields.Selection([('not_done', "Not done"), ('just_done', "Just done"), ('done', "Done")], string="State of the website sale onboarding payment acquirer step", default='not_done')

    @api.model
    def action_open_website_sale_onboarding_payment_acquirer(self):
        """ Called by onboarding panel above the quotation list."""
        self.env.company.get_chart_of_accounts_or_fail()
        action = self.env["ir.actions.actions"]._for_xml_id("website_sale.action_open_website_sale_onboarding_payment_acquirer_wizard")
        return action

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, models, fields


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    salesperson_id = fields.Many2one('res.users', related='website_id.salesperson_id', string='Salesperson', readonly=False)
    salesteam_id = fields.Many2one('crm.team', related='website_id.salesteam_id', string='Sales Team', readonly=False)
    module_website_sale_delivery = fields.Boolean("eCommerce Shipping Costs")
    # field used to have a nice radio in form view, resuming the 2 fields above
    sale_delivery_settings = fields.Selection([
        ('none', 'No shipping management on website'),
        ('internal', "Delivery methods are only used internally: the customer doesn't pay for shipping costs"),
        ('website', "Delivery methods are selectable on the website: the customer pays for shipping costs"),
    ], string="Shipping Management")

    group_delivery_invoice_address = fields.Boolean(string="Shipping Address", implied_group='sale.group_delivery_invoice_address', group='base.group_portal,base.group_user,base.group_public')

    module_website_sale_digital = fields.Boolean("Digital Content")
    module_website_sale_wishlist = fields.Boolean("Wishlists")
    module_website_sale_comparison = fields.Boolean("Product Comparison Tool")
    module_website_sale_stock = fields.Boolean("Inventory", help='Installs the "Website Delivery Information" application')

    module_account = fields.Boolean("Invoicing")

    cart_recovery_mail_template = fields.Many2one('mail.template', string='Cart Recovery Email', domain="[('model', '=', 'sale.order')]",
                                                  related='website_id.cart_recovery_mail_template_id', readonly=False)
    cart_abandoned_delay = fields.Float("Abandoned Delay", help="Number of hours after which the cart is considered abandoned.",
                                        related='website_id.cart_abandoned_delay', readonly=False)

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()

        sale_delivery_settings = 'none'
        if self.env['ir.module.module'].search([('name', '=', 'delivery')], limit=1).state in ('installed', 'to install', 'to upgrade'):
            sale_delivery_settings = 'internal'
            if self.env['ir.module.module'].search([('name', '=', 'website_sale_delivery')], limit=1).state in ('installed', 'to install', 'to upgrade'):
                sale_delivery_settings = 'website'

        res.update(
            sale_delivery_settings=sale_delivery_settings,
        )
        return res

    @api.onchange('sale_delivery_settings')
    def _onchange_sale_delivery_settings(self):
        if self.sale_delivery_settings == 'none':
            self.update({
                'module_delivery': False,
                'module_website_sale_delivery': False,
            })
        elif self.sale_delivery_settings == 'internal':
            self.update({
                'module_delivery': True,
                'module_website_sale_delivery': False,
            })
        else:
            self.update({
                'module_delivery': True,
                'module_website_sale_delivery': True,
            })

    @api.onchange('group_discount_per_so_line')
    def _onchange_group_discount_per_so_line(self):
        if self.group_discount_per_so_line:
            self.update({
                'group_product_pricelist': True,
            })

```

## File: models\res_country.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResCountry(models.Model):
    _inherit = 'res.country'

    def get_website_sale_countries(self, mode='billing'):
        return self.sudo().search([])

    def get_website_sale_states(self, mode='billing'):
        return self.sudo().state_ids

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.addons.website.models import ir_http


class ResPartner(models.Model):
    _inherit = 'res.partner'

    last_website_so_id = fields.Many2one('sale.order', compute='_compute_last_website_so_id', string='Last Online Sales Order')

    def _compute_last_website_so_id(self):
        SaleOrder = self.env['sale.order']
        for partner in self:
            is_public = any(u._is_public() for u in partner.with_context(active_test=False).user_ids)
            website = ir_http.get_request_website()
            if website and not is_public:
                partner.last_website_so_id = SaleOrder.search([
                    ('partner_id', '=', partner.id),
                    ('website_id', '=', website.id),
                    ('state', '=', 'draft'),
                ], order='write_date desc', limit=1)
            else:
                partner.last_website_so_id = SaleOrder  # Not in a website context or public User

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import random
from datetime import datetime
from dateutil.relativedelta import relativedelta

from odoo import api, models, fields, _
from odoo.http import request
from odoo.osv import expression
from odoo.exceptions import UserError, ValidationError

_logger = logging.getLogger(__name__)


class SaleOrder(models.Model):
    _inherit = "sale.order"

    website_order_line = fields.One2many(
        'sale.order.line',
        compute='_compute_website_order_line',
        string='Order Lines displayed on Website',
        help='Order Lines to be displayed on the website. They should not be used for computation purpose.',
    )
    cart_quantity = fields.Integer(compute='_compute_cart_info', string='Cart Quantity')
    only_services = fields.Boolean(compute='_compute_cart_info', string='Only Services')
    is_abandoned_cart = fields.Boolean('Abandoned Cart', compute='_compute_abandoned_cart', search='_search_abandoned_cart')
    cart_recovery_email_sent = fields.Boolean('Cart recovery email already sent')
    website_id = fields.Many2one('website', string='Website', readonly=True,
                                 help='Website through which this order was placed.')

    @api.depends('order_line')
    def _compute_website_order_line(self):
        for order in self:
            order.website_order_line = order.order_line

    @api.depends('order_line.product_uom_qty', 'order_line.product_id')
    def _compute_cart_info(self):
        for order in self:
            order.cart_quantity = int(sum(order.mapped('website_order_line.product_uom_qty')))
            order.only_services = all(l.product_id.type in ('service', 'digital') for l in order.website_order_line)

    @api.depends('website_id', 'date_order', 'order_line', 'state', 'partner_id')
    def _compute_abandoned_cart(self):
        for order in self:
            # a quotation can be considered as an abandonned cart if it is linked to a website,
            # is in the 'draft' state and has an expiration date
            if order.website_id and order.state == 'draft' and order.date_order:
                public_partner_id = order.website_id.user_id.partner_id
                # by default the expiration date is 1 hour if not specified on the website configuration
                abandoned_delay = order.website_id.cart_abandoned_delay or 1.0
                abandoned_datetime = datetime.utcnow() - relativedelta(hours=abandoned_delay)
                order.is_abandoned_cart = bool(order.date_order <= abandoned_datetime and order.partner_id != public_partner_id and order.order_line)
            else:
                order.is_abandoned_cart = False

    @api.onchange('partner_id')
    def onchange_partner_id(self):
        super().onchange_partner_id()
        for order in self:
            if order.website_id:
                order.payment_term_id = order.website_id.with_company(order.company_id).sale_get_payment_term(order.partner_id)

    def _search_abandoned_cart(self, operator, value):
        abandoned_delay = self.website_id and self.website_id.cart_abandoned_delay or 1.0
        abandoned_datetime = fields.Datetime.to_string(datetime.utcnow() - relativedelta(hours=abandoned_delay))
        abandoned_domain = expression.normalize_domain([
            ('date_order', '<=', abandoned_datetime),
            ('website_id', '!=', False),
            ('state', '=', 'draft'),
            ('partner_id', '!=', self.env.ref('base.public_partner').id),
            ('order_line', '!=', False)
        ])
        # is_abandoned domain possibilities
        if (operator not in expression.NEGATIVE_TERM_OPERATORS and value) or (operator in expression.NEGATIVE_TERM_OPERATORS and not value):
            return abandoned_domain
        return expression.distribute_not(['!'] + abandoned_domain)  # negative domain

    def _cart_find_product_line(self, product_id=None, line_id=None, **kwargs):
        """Find the cart line matching the given parameters.

        If a product_id is given, the line will match the product only if the
        line also has the same special attributes: `no_variant` attributes and
        `is_custom` values.
        """
        self.ensure_one()
        product = self.env['product.product'].browse(product_id)

        # split lines with the same product if it has untracked attributes
        if product and (product.product_tmpl_id.has_dynamic_attributes() or product.product_tmpl_id._has_no_variant_attributes()) and not line_id and not kwargs.get('force_search', False):
            return self.env['sale.order.line']

        domain = [('order_id', '=', self.id), ('product_id', '=', product_id)]
        if line_id:
            domain += [('id', '=', line_id)]
        else:
            domain += [('product_custom_attribute_value_ids', '=', False)]

        return self.env['sale.order.line'].sudo().search(domain)

    def _website_product_id_change(self, order_id, product_id, qty=0):
        order = self.sudo().browse(order_id)
        product_context = dict(self.env.context)
        product_context.setdefault('lang', order.partner_id.lang)
        product_context.update({
            'partner': order.partner_id,
            'quantity': qty,
            'date': order.date_order,
            'pricelist': order.pricelist_id.id,
        })
        product = self.env['product.product'].with_context(product_context).with_company(order.company_id.id).browse(product_id)
        discount = 0

        if order.pricelist_id.discount_policy == 'without_discount':
            # This part is pretty much a copy-paste of the method '_onchange_discount' of
            # 'sale.order.line'.
            price, rule_id = order.pricelist_id.with_context(product_context).get_product_price_rule(product, qty or 1.0, order.partner_id)
            pu, currency = request.env['sale.order.line'].with_context(product_context)._get_real_price_currency(product, rule_id, qty, product.uom_id, order.pricelist_id.id)
            if order.pricelist_id and order.partner_id:
                order_line = order._cart_find_product_line(product.id)
                if order_line:
                    price = product._get_tax_included_unit_price(
                        self.company_id,
                        order.currency_id,
                        order.date_order,
                        'sale',
                        fiscal_position=order.fiscal_position_id,
                        product_price_unit=price,
                        product_currency=order.currency_id
                    )
                    pu = product._get_tax_included_unit_price(
                        self.company_id,
                        order.currency_id,
                        order.date_order,
                        'sale',
                        fiscal_position=order.fiscal_position_id,
                        product_price_unit=pu,
                        product_currency=order.currency_id
                    )
            if pu != 0:
                if order.pricelist_id.currency_id != currency:
                    # we need new_list_price in the same currency as price, which is in the SO's pricelist's currency
                    date = order.date_order or fields.Date.today()
                    pu = currency._convert(pu, order.pricelist_id.currency_id, order.company_id, date)
                discount = (pu - price) / pu * 100
                if discount < 0:
                    # In case the discount is negative, we don't want to show it to the customer,
                    # but we still want to use the price defined on the pricelist
                    discount = 0
                    pu = price
            else:
                # In case the price_unit equal 0 and therefore not able to calculate the discount,
                # we fallback on the price defined on the pricelist.
                pu = price
        else:
            pu = product.price
            if order.pricelist_id and order.partner_id:
                order_line = order._cart_find_product_line(product.id, force_search=True)
                if order_line:
                    pu = product._get_tax_included_unit_price(
                        self.company_id,
                        order.currency_id,
                        order.date_order,
                        'sale',
                        fiscal_position=order.fiscal_position_id,
                        product_price_unit=product.price,
                        product_currency=order.currency_id
                    )

        return {
            'product_id': product_id,
            'product_uom_qty': qty,
            'order_id': order_id,
            'product_uom': product.uom_id.id,
            'price_unit': pu,
            'discount': discount,
        }

    def _cart_update(self, product_id=None, line_id=None, add_qty=0, set_qty=0, **kwargs):
        """ Add or set product quantity, add_qty can be negative """
        self.ensure_one()
        product_context = dict(self.env.context)
        product_context.setdefault('lang', self.sudo().partner_id.lang)
        SaleOrderLineSudo = self.env['sale.order.line'].sudo().with_context(product_context)
        # change lang to get correct name of attributes/values
        product_with_context = self.env['product.product'].with_context(product_context)
        product = product_with_context.browse(int(product_id)).exists()

        if not product or (not line_id and not product._is_add_to_cart_allowed()):
            raise UserError(_("The given product does not exist therefore it cannot be added to cart."))

        try:
            if add_qty:
                add_qty = int(add_qty)
        except ValueError:
            add_qty = 1
        try:
            if set_qty:
                set_qty = int(set_qty)
        except ValueError:
            set_qty = 0
        quantity = 0
        order_line = False
        if self.state != 'draft':
            request.session['sale_order_id'] = None
            raise UserError(_('It is forbidden to modify a sales order which is not in draft status.'))
        if line_id is not False:
            order_line = self._cart_find_product_line(product_id, line_id, **kwargs)[:1]

        # Create line if no line with product_id can be located
        if not order_line:
            no_variant_attribute_values = kwargs.get('no_variant_attribute_values') or []
            received_no_variant_values = product.env['product.template.attribute.value'].browse([int(ptav['value']) for ptav in no_variant_attribute_values])
            received_combination = product.product_template_attribute_value_ids | received_no_variant_values
            product_template = product.product_tmpl_id

            # handle all cases where incorrect or incomplete data are received
            combination = product_template._get_closest_possible_combination(received_combination)

            # get or create (if dynamic) the correct variant
            product = product_template._create_product_variant(combination)

            if not product:
                raise UserError(_("The given combination does not exist therefore it cannot be added to cart."))

            product_id = product.id

            values = self._website_product_id_change(self.id, product_id, qty=1)

            # add no_variant attributes that were not received
            for ptav in combination.filtered(lambda ptav: ptav.attribute_id.create_variant == 'no_variant' and ptav not in received_no_variant_values):
                no_variant_attribute_values.append({
                    'value': ptav.id,
                })

            # save no_variant attributes values
            if no_variant_attribute_values:
                values['product_no_variant_attribute_value_ids'] = [
                    (6, 0, [int(attribute['value']) for attribute in no_variant_attribute_values])
                ]

            # add is_custom attribute values that were not received
            custom_values = kwargs.get('product_custom_attribute_values') or []
            received_custom_values = product.env['product.template.attribute.value'].browse([int(ptav['custom_product_template_attribute_value_id']) for ptav in custom_values])

            for ptav in combination.filtered(lambda ptav: ptav.is_custom and ptav not in received_custom_values):
                custom_values.append({
                    'custom_product_template_attribute_value_id': ptav.id,
                    'custom_value': '',
                })

            # save is_custom attributes values
            if custom_values:
                values['product_custom_attribute_value_ids'] = [(0, 0, {
                    'custom_product_template_attribute_value_id': custom_value['custom_product_template_attribute_value_id'],
                    'custom_value': custom_value['custom_value']
                }) for custom_value in custom_values]

            # create the line
            order_line = SaleOrderLineSudo.create(values)

            try:
                order_line._compute_tax_id()
            except ValidationError as e:
                # The validation may occur in backend (eg: taxcloud) but should fail silently in frontend
                _logger.debug("ValidationError occurs during tax compute. %s" % (e))
            if add_qty:
                add_qty -= 1

        # compute new quantity
        if set_qty:
            quantity = set_qty
        elif add_qty is not None:
            quantity = order_line.product_uom_qty + (add_qty or 0)

        # Remove zero of negative lines
        if quantity <= 0:
            linked_line = order_line.linked_line_id
            order_line.unlink()
            if linked_line:
                # update description of the parent
                linked_product = product_with_context.browse(linked_line.product_id.id)
                linked_line.name = linked_line.get_sale_order_line_multiline_description_sale(linked_product)
        else:
            # update line
            no_variant_attributes_price_extra = [ptav.price_extra for ptav in order_line.product_no_variant_attribute_value_ids]
            values = self.with_context(no_variant_attributes_price_extra=tuple(no_variant_attributes_price_extra))._website_product_id_change(self.id, product_id, qty=quantity)
            order = self.sudo().browse(self.id)
            if self.pricelist_id.discount_policy == 'with_discount' and not self.env.context.get('fixed_price'):
                product_context.update({
                    'partner': order.partner_id,
                    'quantity': quantity,
                    'date': order.date_order,
                    'pricelist': order.pricelist_id.id,
                })
            product_with_context = self.env['product.product'].with_context(product_context).with_company(order.company_id.id)
            product = product_with_context.browse(product_id)

            order_line.write(values)

            # link a product to the sales order
            if kwargs.get('linked_line_id'):
                linked_line = SaleOrderLineSudo.browse(kwargs['linked_line_id'])
                order_line.write({
                    'linked_line_id': linked_line.id,
                })
                linked_product = product_with_context.browse(linked_line.product_id.id)
                linked_line.name = linked_line.get_sale_order_line_multiline_description_sale(linked_product)
            # Generate the description with everything. This is done after
            # creating because the following related fields have to be set:
            # - product_no_variant_attribute_value_ids
            # - product_custom_attribute_value_ids
            # - linked_line_id
            order_line.name = order_line.get_sale_order_line_multiline_description_sale(product)

        option_lines = self.order_line.filtered(lambda l: l.linked_line_id.id == order_line.id)

        return {'line_id': order_line.id, 'quantity': quantity, 'option_ids': list(set(option_lines.ids))}

    def _cart_accessories(self):
        """ Suggest accessories based on 'Accessory Products' of products in cart """
        for order in self:
            products = order.website_order_line.mapped('product_id')
            accessory_products = self.env['product.product']
            for line in order.website_order_line.filtered(lambda l: l.product_id):
                combination = line.product_id.product_template_attribute_value_ids + line.product_no_variant_attribute_value_ids
                accessory_products |= line.product_id.accessory_product_ids.filtered(lambda product:
                    product.website_published and
                    product not in products and
                    product._is_variant_possible(parent_combination=combination) and
                    (product.company_id == line.company_id or not product.company_id)
                )

            return random.sample(accessory_products, len(accessory_products))

    def action_recovery_email_send(self):
        for order in self:
            order._portal_ensure_token()
        composer_form_view_id = self.env.ref('mail.email_compose_message_wizard_form').id

        template_id = self._get_cart_recovery_template().id

        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'view_id': composer_form_view_id,
            'target': 'new',
            'context': {
                'default_composition_mode': 'mass_mail' if len(self.ids) > 1 else 'comment',
                'default_res_id': self.ids[0],
                'default_model': 'sale.order',
                'default_use_template': bool(template_id),
                'default_template_id': template_id,
                'website_sale_send_recovery_email': True,
                'active_ids': self.ids,
            },
        }

    def _get_cart_recovery_template(self):
        """
        Return the cart recovery template record for a set of orders.
        If they all belong to the same website, we return the website-specific template;
        otherwise we return the default template.
        If the default is not found, the empty ['mail.template'] is returned.
        """
        websites = self.mapped('website_id')
        template = websites.cart_recovery_mail_template_id if len(websites) == 1 else False
        template = template or self.env.ref('website_sale.mail_template_sale_cart_recovery', raise_if_not_found=False)
        return template or self.env['mail.template']

    def _cart_recovery_email_send(self):
        """Send the cart recovery email on the current recordset,
        making sure that the portal token exists to avoid broken links, and marking the email as sent.
        Similar method to action_recovery_email_send, made to be called in automated actions.
        Contrary to the former, it will use the website-specific template for each order."""
        sent_orders = self.env['sale.order']
        for order in self:
            template = order._get_cart_recovery_template()
            if template:
                order._portal_ensure_token()
                template.send_mail(order.id)
                sent_orders |= order
        sent_orders.write({'cart_recovery_email_sent': True})

    def action_confirm(self):
        res = super(SaleOrder, self).action_confirm()
        for order in self:
            if not order.transaction_ids and not order.amount_total and self._context.get('send_email'):
                order._send_order_confirmation_mail()
        return res


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    name_short = fields.Char(compute="_compute_name_short")

    linked_line_id = fields.Many2one('sale.order.line', string='Linked Order Line', domain="[('order_id', '=', order_id)]", ondelete='cascade', copy=False, index=True)
    option_line_ids = fields.One2many('sale.order.line', 'linked_line_id', string='Options Linked')

    def get_sale_order_line_multiline_description_sale(self, product):
        description = super(SaleOrderLine, self).get_sale_order_line_multiline_description_sale(product)
        if self.linked_line_id:
            description += "\n" + _("Option for: %s", self.linked_line_id.product_id.display_name)
        if self.option_line_ids:
            description += "\n" + '\n'.join([_("Option: %s", option_line.product_id.display_name) for option_line in self.option_line_ids])
        return description

    @api.depends('product_id.display_name')
    def _compute_name_short(self):
        """ Compute a short name for this sale order line, to be used on the website where we don't have much space.
            To keep it short, instead of using the first line of the description, we take the product name without the internal reference.
        """
        for record in self:
            record.name_short = record.product_id.with_context(display_default_code=False).display_name

    def get_description_following_lines(self):
        return self.name.splitlines()[1:]

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, tools, SUPERUSER_ID, _

from odoo.http import request
from odoo.addons.website.models import ir_http
from odoo.addons.http_routing.models.ir_http import url_for

_logger = logging.getLogger(__name__)


class Website(models.Model):
    _inherit = 'website'

    pricelist_id = fields.Many2one('product.pricelist', compute='_compute_pricelist_id', string='Default Pricelist')
    currency_id = fields.Many2one('res.currency',
        related='pricelist_id.currency_id', depends=(), related_sudo=False,
        string='Default Currency', readonly=False)
    salesperson_id = fields.Many2one('res.users', string='Salesperson')

    def _get_default_website_team(self):
        try:
            team = self.env.ref('sales_team.salesteam_website_sales')
            return team if team.active else None
        except ValueError:
            return None

    salesteam_id = fields.Many2one('crm.team',
        string='Sales Team',
        default=_get_default_website_team)
    pricelist_ids = fields.One2many('product.pricelist', compute="_compute_pricelist_ids",
                                    string='Price list available for this Ecommerce/Website')
    all_pricelist_ids = fields.One2many('product.pricelist', 'website_id', string='All pricelists',
                                        help='Technical: Used to recompute pricelist_ids')

    def _default_recovery_mail_template(self):
        try:
            return self.env.ref('website_sale.mail_template_sale_cart_recovery').id
        except ValueError:
            return False

    cart_recovery_mail_template_id = fields.Many2one('mail.template', string='Cart Recovery Email', default=_default_recovery_mail_template, domain="[('model', '=', 'sale.order')]")
    cart_abandoned_delay = fields.Float("Abandoned Delay", default=1.0)

    shop_ppg = fields.Integer(default=20, string="Number of products in the grid on the shop")
    shop_ppr = fields.Integer(default=4, string="Number of grid columns on the shop")

    shop_extra_field_ids = fields.One2many('website.sale.extra.field', 'website_id', string='E-Commerce Extra Fields')

    @api.depends('all_pricelist_ids')
    def _compute_pricelist_ids(self):
        Pricelist = self.env['product.pricelist']
        for website in self:
            website.pricelist_ids = Pricelist.search(
                Pricelist._get_website_pricelists_domain(website.id)
            )

    def _compute_pricelist_id(self):
        for website in self:
            website.pricelist_id = website.with_context(website_id=website.id).get_current_pricelist()

    # This method is cached, must not return records! See also #8795
    @tools.ormcache('self.env.uid', 'country_code', 'show_visible', 'website_pl', 'current_pl', 'all_pl', 'partner_pl', 'order_pl')
    def _get_pl_partner_order(self, country_code, show_visible, website_pl, current_pl, all_pl, partner_pl=False, order_pl=False):
        """ Return the list of pricelists that can be used on website for the current user.
        :param str country_code: code iso or False, If set, we search only price list available for this country
        :param bool show_visible: if True, we don't display pricelist where selectable is False (Eg: Code promo)
        :param int website_pl: The default pricelist used on this website
        :param int current_pl: The current pricelist used on the website
                               (If not selectable but the current pricelist we had this pricelist anyway)
        :param list all_pl: List of all pricelist available for this website
        :param int partner_pl: the partner pricelist
        :param int order_pl: the current cart pricelist
        :returns: list of pricelist ids
        """
        def _check_show_visible(pl):
            """ If `show_visible` is True, we will only show the pricelist if
            one of this condition is met:
            - The pricelist is `selectable`.
            - The pricelist is either the currently used pricelist or the
            current cart pricelist, we should consider it as available even if
            it might not be website compliant (eg: it is not selectable anymore,
            it is a backend pricelist, it is not active anymore..).
            """
            return (not show_visible or pl.selectable or pl.id in (current_pl, order_pl))

        # Note: 1. pricelists from all_pl are already website compliant (went through
        #          `_get_website_pricelists_domain`)
        #       2. do not read `property_product_pricelist` here as `_get_pl_partner_order`
        #          is cached and the result of this method will be impacted by that field value.
        #          Pass it through `partner_pl` parameter instead to invalidate the cache.

        # If there is a GeoIP country, find a pricelist for it
        self.ensure_one()
        pricelists = self.env['product.pricelist']
        if country_code:
            for cgroup in self.env['res.country.group'].search([('country_ids.code', '=', country_code)]):
                pricelists |= cgroup.pricelist_ids.filtered(
                    lambda pl: pl._is_available_on_website(self.id) and _check_show_visible(pl)
                )

        # no GeoIP or no pricelist for this country
        if not country_code or not pricelists:
            pricelists |= all_pl.filtered(lambda pl: _check_show_visible(pl))

        # if logged in, add partner pl (which is `property_product_pricelist`, might not be website compliant)
        is_public = self.user_id.id == self.env.user.id
        if not is_public:
            # keep partner_pl only if website compliant
            partner_pl = pricelists.browse(partner_pl).filtered(lambda pl: pl._is_available_on_website(self.id) and _check_show_visible(pl))
            if country_code:
                # keep partner_pl only if GeoIP compliant in case of GeoIP enabled
                partner_pl = partner_pl.filtered(
                    lambda pl: pl.country_group_ids and country_code in pl.country_group_ids.mapped('country_ids.code') or not pl.country_group_ids
                )
            pricelists |= partner_pl

        # This method is cached, must not return records! See also #8795
        return pricelists.ids

    def _get_pricelist_available(self, req, show_visible=False):
        """ Return the list of pricelists that can be used on website for the current user.
        Country restrictions will be detected with GeoIP (if installed).
        :param bool show_visible: if True, we don't display pricelist where selectable is False (Eg: Code promo)
        :returns: pricelist recordset
        """
        website = ir_http.get_request_website()
        if not website:
            if self.env.context.get('website_id'):
                website = self.browse(self.env.context['website_id'])
            else:
                # In the weird case we are coming from the backend (https://github.com/odoo/odoo/issues/20245)
                website = len(self) == 1 and self or self.search([], limit=1)
        isocountry = req and req.session.geoip and req.session.geoip.get('country_code') or False
        partner = self.env.user.partner_id
        last_order_pl = partner.last_website_so_id.pricelist_id
        partner_pl = partner.property_product_pricelist
        pricelists = website._get_pl_partner_order(isocountry, show_visible,
                                                   website.user_id.sudo().partner_id.property_product_pricelist.id,
                                                   req and req.session.get('website_sale_current_pl') or None,
                                                   website.pricelist_ids,
                                                   partner_pl=partner_pl and partner_pl.id or None,
                                                   order_pl=last_order_pl and last_order_pl.id or None)
        return self.env['product.pricelist'].browse(pricelists)

    def get_pricelist_available(self, show_visible=False):
        return self._get_pricelist_available(request, show_visible)

    def is_pricelist_available(self, pl_id):
        """ Return a boolean to specify if a specific pricelist can be manually set on the website.
        Warning: It check only if pricelist is in the 'selectable' pricelists or the current pricelist.
        :param int pl_id: The pricelist id to check
        :returns: Boolean, True if valid / available
        """
        return pl_id in self.get_pricelist_available(show_visible=False).ids

    def get_current_pricelist(self):
        """
        :returns: The current pricelist record
        """
        # The list of available pricelists for this user.
        # If the user is signed in, and has a pricelist set different than the public user pricelist
        # then this pricelist will always be considered as available
        available_pricelists = self.get_pricelist_available()
        pl = None
        partner = self.env.user.partner_id
        if request and request.session.get('website_sale_current_pl'):
            # `website_sale_current_pl` is set only if the user specifically chose it:
            #  - Either, he chose it from the pricelist selection
            #  - Either, he entered a coupon code
            pl = self.env['product.pricelist'].browse(request.session['website_sale_current_pl'])
            if pl not in available_pricelists:
                pl = None
                request.session.pop('website_sale_current_pl')
        if not pl:
            # If the user has a saved cart, it take the pricelist of this last unconfirmed cart
            pl = partner.last_website_so_id.pricelist_id
            if not pl:
                # The pricelist of the user set on its partner form.
                # If the user is not signed in, it's the public user pricelist
                pl = partner.property_product_pricelist
            if available_pricelists and pl not in available_pricelists:
                # If there is at least one pricelist in the available pricelists
                # and the chosen pricelist is not within them
                # it then choose the first available pricelist.
                # This can only happen when the pricelist is the public user pricelist and this pricelist is not in the available pricelist for this localization
                # If the user is signed in, and has a special pricelist (different than the public user pricelist),
                # then this special pricelist is amongs these available pricelists, and therefore it won't fall in this case.
                pl = available_pricelists[0]

        if not pl:
            _logger.error('Fail to find pricelist for partner "%s" (id %s)', partner.name, partner.id)
        return pl

    def sale_product_domain(self):
        return [("sale_ok", "=", True)] + self.get_current_website().website_domain()

    @api.model
    def sale_get_payment_term(self, partner):
        pt = self.env.ref('account.account_payment_term_immediate', False).sudo()
        if pt:
            pt = (not pt.company_id.id or self.company_id.id == pt.company_id.id) and pt
        return (
            partner.property_payment_term_id or
            pt or
            self.env['account.payment.term'].sudo().search([('company_id', '=', self.company_id.id)], limit=1)
        ).id

    def _prepare_sale_order_values(self, partner, pricelist):
        self.ensure_one()
        affiliate_id = request.session.get('affiliate_id')
        salesperson_id = affiliate_id if self.env['res.users'].sudo().browse(affiliate_id).exists() else request.website.salesperson_id.id
        addr = partner.address_get(['delivery'])
        if not request.website.is_public_user():
            last_sale_order = self.env['sale.order'].sudo().search([('partner_id', '=', partner.id)], limit=1, order="date_order desc, id desc")
            if last_sale_order and last_sale_order.partner_shipping_id.active:  # first = me
                addr['delivery'] = last_sale_order.partner_shipping_id.id
        default_user_id = partner.parent_id.user_id.id or partner.user_id.id
        values = {
            'partner_id': partner.id,
            'pricelist_id': pricelist.id,
            'payment_term_id': self.sale_get_payment_term(partner),
            'team_id': self.salesteam_id.id or partner.parent_id.team_id.id or partner.team_id.id,
            'partner_invoice_id': partner.id,
            'partner_shipping_id': addr['delivery'],
            'user_id': salesperson_id or self.salesperson_id.id or default_user_id,
            'website_id': self._context.get('website_id'),
        }
        company = self.company_id or pricelist.company_id
        if company:
            values['company_id'] = company.id
        return values

    def sale_get_order(self, force_create=False, code=None, update_pricelist=False, force_pricelist=False):
        """ Return the current sales order after mofications specified by params.
        :param bool force_create: Create sales order if not already existing
        :param str code: Code to force a pricelist (promo code)
                         If empty, it's a special case to reset the pricelist with the first available else the default.
        :param bool update_pricelist: Force to recompute all the lines from sales order to adapt the price with the current pricelist.
        :param int force_pricelist: pricelist_id - if set,  we change the pricelist with this one
        :returns: browse record for the current sales order
        """
        self.ensure_one()
        partner = self.env.user.partner_id
        sale_order_id = request.session.get('sale_order_id')
        check_fpos = False
        if not sale_order_id and not self.env.user._is_public():
            last_order = partner.last_website_so_id
            if last_order:
                available_pricelists = self.get_pricelist_available()
                # Do not reload the cart of this user last visit if the cart uses a pricelist no longer available.
                sale_order_id = last_order.pricelist_id in available_pricelists and last_order.id
                check_fpos = True

        # Test validity of the sale_order_id
        sale_order = self.env['sale.order'].with_company(request.website.company_id.id).sudo().browse(sale_order_id).exists() if sale_order_id else None

        # Ignore the current order if a payment has been initiated. We don't want to retrieve the
        # cart and allow the user to update it when the payment is about to confirm it.
        if sale_order and sale_order.get_portal_last_transaction().state in ('pending', 'authorized', 'done'):
            sale_order = None

        # Do not reload the cart of this user last visit if the Fiscal Position has changed.
        if check_fpos and sale_order:
            fpos_id = (
                self.env['account.fiscal.position'].sudo()
                .with_company(sale_order.company_id.id)
                .get_fiscal_position(sale_order.partner_id.id, delivery_id=sale_order.partner_shipping_id.id)
            ).id
            if sale_order.fiscal_position_id.id != fpos_id:
                sale_order = None

        if not (sale_order or force_create or code):
            if request.session.get('sale_order_id'):
                request.session['sale_order_id'] = None
            return self.env['sale.order']

        if self.env['product.pricelist'].browse(force_pricelist).exists():
            pricelist_id = force_pricelist
            request.session['website_sale_current_pl'] = pricelist_id
            update_pricelist = True
        else:
            pricelist_id = request.session.get('website_sale_current_pl') or self.get_current_pricelist().id

        if not self._context.get('pricelist'):
            self = self.with_context(pricelist=pricelist_id)

        # cart creation was requested (either explicitly or to configure a promo code)
        if not sale_order:
            # TODO cache partner_id session
            pricelist = self.env['product.pricelist'].browse(pricelist_id).sudo()
            so_data = self._prepare_sale_order_values(partner, pricelist)
            sale_order = self.env['sale.order'].with_company(request.website.company_id.id).with_user(SUPERUSER_ID).create(so_data)

            # set fiscal position
            if request.website.partner_id.id != partner.id:
                sale_order.onchange_partner_shipping_id()
            else: # For public user, fiscal position based on geolocation
                country_code = request.session['geoip'].get('country_code')
                if country_code:
                    country_id = request.env['res.country'].search([('code', '=', country_code)], limit=1).id
                    sale_order.fiscal_position_id = request.env['account.fiscal.position'].sudo().with_company(request.website.company_id.id)._get_fpos_by_region(country_id)
                else:
                    # if no geolocation, use the public user fp
                    sale_order.onchange_partner_shipping_id()

            request.session['sale_order_id'] = sale_order.id

            # The order was created with SUPERUSER_ID, revert back to request user.
            sale_order = sale_order.with_user(self.env.user).sudo()

        # case when user emptied the cart
        if not request.session.get('sale_order_id'):
            request.session['sale_order_id'] = sale_order.id

        # check for change of pricelist with a coupon
        pricelist_id = pricelist_id or partner.property_product_pricelist.id

        # check for change of partner_id ie after signup
        if sale_order.partner_id.id != partner.id and request.website.partner_id.id != partner.id:
            flag_pricelist = False
            if pricelist_id != sale_order.pricelist_id.id:
                flag_pricelist = True
            fiscal_position = sale_order.fiscal_position_id.id

            # change the partner, and trigger the onchange
            sale_order.write({'partner_id': partner.id})
            sale_order.with_context(not_self_saleperson=True).onchange_partner_id()
            sale_order.write({'partner_invoice_id': partner.id})
            sale_order.onchange_partner_shipping_id() # fiscal position
            sale_order['payment_term_id'] = self.sale_get_payment_term(partner)

            # check the pricelist : update it if the pricelist is not the 'forced' one
            values = {}
            if sale_order.pricelist_id:
                if sale_order.pricelist_id.id != pricelist_id:
                    values['pricelist_id'] = pricelist_id
                    update_pricelist = True

            # if fiscal position, update the order lines taxes
            if sale_order.fiscal_position_id:
                sale_order._compute_tax_id()

            # if values, then make the SO update
            if values:
                sale_order.write(values)

            # check if the fiscal position has changed with the partner_id update
            recent_fiscal_position = sale_order.fiscal_position_id.id
            # when buying a free product with public user and trying to log in, SO state is not draft
            if (flag_pricelist or recent_fiscal_position != fiscal_position) and sale_order.state == 'draft':
                update_pricelist = True

        if code and code != sale_order.pricelist_id.code:
            code_pricelist = self.env['product.pricelist'].sudo().search([('code', '=', code)], limit=1)
            if code_pricelist:
                pricelist_id = code_pricelist.id
                update_pricelist = True
        elif code is not None and sale_order.pricelist_id.code and code != sale_order.pricelist_id.code:
            # code is not None when user removes code and click on "Apply"
            pricelist_id = partner.property_product_pricelist.id
            update_pricelist = True

        # update the pricelist
        if update_pricelist:
            request.session['website_sale_current_pl'] = pricelist_id
            values = {'pricelist_id': pricelist_id}
            sale_order.write(values)
            for line in sale_order.order_line:
                if line.exists():
                    sale_order._cart_update(product_id=line.product_id.id, line_id=line.id, add_qty=0)

        return sale_order

    def sale_reset(self):
        request.session.update({
            'sale_order_id': False,
            'website_sale_current_pl': False,
        })

    @api.model
    def action_dashboard_redirect(self):
        if self.env.user.has_group('sales_team.group_sale_salesman'):
            return self.env["ir.actions.actions"]._for_xml_id("website.backend_dashboard")
        return super(Website, self).action_dashboard_redirect()

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('eCommerce'), url_for('/shop'), 'website_sale'))
        return suggested_controllers

    def _bootstrap_snippet_filters(self):
        super(Website, self)._bootstrap_snippet_filters()
        # The same behavior is done in the post_init hook
        action = self.env.ref('website_sale.dynamic_snippet_products_action', raise_if_not_found=False)
        if action:
            self.env['website.snippet.filter'].create({
                'action_server_id': action.id,
                'field_names': 'display_name,description_sale,image_512,list_price',
                'limit': 16,
                'name': _('Products'),
                'website_id': self.id,
            })


class WebsiteSaleExtraField(models.Model):
    _name = 'website.sale.extra.field'
    _description = 'E-Commerce Extra Info Shown on product page'
    _order = 'sequence'

    website_id = fields.Many2one('website')
    sequence = fields.Integer(default=10)
    field_id = fields.Many2one(
        'ir.model.fields',
        domain=[('model_id.model', '=', 'product.template'), ('ttype', 'in', ['char', 'binary'])]
    )
    label = fields.Char(related='field_id.field_description')
    name = fields.Char(related='field_id.name')

```

## File: models\website_page.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.http import request


class WabsitePage(models.AbstractModel):
    _inherit = 'website.page'

    def _get_cache_key(self, req):
        cart = request.website.sale_get_order()
        cache_key = (cart and cart.cart_quantity or 0,)
        cache_key += super()._get_cache_key(req)
        return cache_key

```

## File: models\website_snippet_filter.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api, _


class WebsiteSnippetFilter(models.Model):
    _inherit = 'website.snippet.filter'

    @api.model
    def _get_website_currency(self):
        pricelist = self.env['website'].get_current_website().get_current_pricelist()
        return pricelist.currency_id

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta

from odoo import fields, models, api

class WebsiteTrack(models.Model):
    _inherit = 'website.track'

    product_id = fields.Many2one('product.product', index=True, ondelete='cascade', readonly=True)


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    visitor_product_count = fields.Integer('Product Views', compute="_compute_product_statistics", help="Total number of views on products")
    product_ids = fields.Many2many('product.product', string="Visited Products", compute="_compute_product_statistics")
    product_count = fields.Integer('Products Views', compute="_compute_product_statistics", help="Total number of product viewed")

    @api.depends('website_track_ids')
    def _compute_product_statistics(self):
        results = self.env['website.track'].read_group(
            [('visitor_id', 'in', self.ids), ('product_id', '!=', False),
             '|', ('product_id.company_id', 'in', self.env.companies.ids), ('product_id.company_id', '=', False)],
            ['visitor_id', 'product_id'], ['visitor_id', 'product_id'],
            lazy=False)
        mapped_data = {}
        for result in results:
            visitor_info = mapped_data.get(result['visitor_id'][0], {'product_count': 0, 'product_ids': set()})
            visitor_info['product_count'] += result['__count']
            visitor_info['product_ids'].add(result['product_id'][0])
            mapped_data[result['visitor_id'][0]] = visitor_info

        for visitor in self:
            visitor_info = mapped_data.get(visitor.id, {'product_ids': [], 'product_count': 0})

            visitor.product_ids = [(6, 0, visitor_info['product_ids'])]
            visitor.visitor_product_count = visitor_info['product_count']
            visitor.product_count = len(visitor_info['product_ids'])

    def _add_viewed_product(self, product_id):
        """ add a website_track with a page marked as viewed"""
        self.ensure_one()
        if product_id and self.env['product.product'].browse(product_id)._is_variant_possible():
            domain = [('product_id', '=', product_id)]
            website_track_values = {'product_id': product_id, 'visit_datetime': datetime.now()}
            self._add_tracking(domain, website_track_values)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import crm_team
from . import ir_http
from . import mail_compose_message
from . import product_image
from . import product
from . import res_country
from . import res_partner
from . import sale_order
from . import website
from . import res_config_settings
from . import digest
from . import res_company
from . import product_attribute
from . import website_page
from . import website_visitor
from . import website_snippet_filter

```

## File: report\sale_report.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class SaleReport(models.Model):
    _inherit = 'sale.report'

    website_id = fields.Many2one('website', readonly=True)

    def _group_by_sale(self, groupby=''):
        res = super()._group_by_sale(groupby)
        res += """,s.website_id"""
        return res

    def _select_additional_fields(self, fields):
        fields['website_id'] = ", s.website_id as website_id"
        return super()._select_additional_fields(fields)

```

## File: report\__init__.py

```python
# coding: utf-8
from . import sale_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_product_public,product.product.public,product.model_product_product,,1,0,0,0
access_product_template_public,product.template.public,product.model_product_template,,1,0,0,0
access_product_category_public,product.category.public,product.model_product_category,,1,0,0,0
access_product_category_pos_manager,product.public.category manager,model_product_public_category,sales_team.group_sale_manager,1,1,1,1
access_product_public_category_public,product.category.public,model_product_public_category,,1,0,0,0
access_product_pricelist_public,product.pricelist.public,product.model_product_pricelist,,1,0,0,0
access_product_pricelist_item_public,product.pricelist.item.public,product.model_product_pricelist_item,,1,0,0,0
access_product_ribbon_public,product.ribbon.public,website_sale.model_product_ribbon,,1,0,0,0
access_product_ribbon_sale_manager,product.ribbon.sale_manager,website_sale.model_product_ribbon,sales_team.group_sale_manager,1,1,1,1
access_product_attribute_public,product.attribute public,product.model_product_attribute,,1,0,0,0
access_product_attribute_value_public,product.attribute value public,product.model_product_attribute_value,,1,0,0,0
access_product_product_attribute,product.template.attribute value public,product.model_product_template_attribute_value,,1,0,0,0
access_product_product_attribute_custom_value,product.attribute.custom value,sale.model_product_attribute_custom_value,,1,0,0,0
access_product_template_attribute_exclusion,product.template.attribute exclusion public,product.model_product_template_attribute_exclusion,,1,0,0,0
access_product_template_attribute_line_public,product.template.attribute line public,product.model_product_template_attribute_line,,1,0,0,0
access_fiscal_position_public,fiscal position public,account.model_account_fiscal_position,base.group_portal,1,0,0,0
access_payment_term,payment term public,account.model_account_payment_term,base.group_portal,1,0,0,0
access_account_tax_user,account.tax,account.model_account_tax,base.group_public,1,0,0,0
access_product_image_public,product.image public,model_product_image,,1,0,0,0
access_product_image_publisher,product.image wbesite publisher,model_product_image,website.group_website_publisher,1,1,1,1
access_product_image_sale,product.image sale,model_product_image,sales_team.group_sale_manager,1,1,1,1
access_website_sale_payment_acquirer_onboarding_wizard,access.website.sale.payment.acquirer.onboarding.wizard,model_website_sale_payment_acquirer_onboarding_wizard,base.group_system,1,1,1,0
access_ecom_extra_fields_public,access_ecom_extra_field public,model_website_sale_extra_field,,1,0,0,0
access_ecom_extra_fields_publisher,access_ecom_extra_field publisher,model_website_sale_extra_field,website.group_website_publisher,1,1,1,1

```

## File: security\website_sale.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="product_template_public" model="ir.rule">
        <field name="name">Public product template</field>
        <field name="model_id" ref="product.model_product_template"/>
        <field name="domain_force">[('website_published', '=', True), ("sale_ok", "=", True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="sales_team.group_sale_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_publisher'))]"/>
    </record>

    <record id="base.group_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('sale.group_delivery_invoice_address'))]"/>
    </record>

    <record id="base.group_public" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('sale.group_delivery_invoice_address'))]"/>
    </record>

    <record id="base.group_portal" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('sale.group_delivery_invoice_address'))]"/>
    </record>

    <!--
        Multi-company/Multi-website compliant:
        We can't add a condition on domain_force without losing `product`
        ir.rule domain_force. It is better to disabled them to be able to
        reenable them on `website_sale` uninstall.
        Don't override domain_force or we will need to hardcode the original
        domain in `uninstall_hook` rather than just reenabling records.
    -->
    <record id="product.product_pricelist_comp_rule" model="ir.rule">
        <field name="active" eval="False"/>
    </record>
    <record id="product.product_pricelist_item_comp_rule" model="ir.rule">
        <field name="active" eval="False"/>
    </record>
    <record id="product_pricelist_comp_rule" model="ir.rule">
        <field name="name">product pricelist company rule</field>
        <field name="model_id" ref="product.model_product_pricelist"/>
        <field name="domain_force">['|', ('company_id', 'in', [False,website.company_id.id]), ('company_id', 'in', company_ids)]</field>
    </record>
    <record id="product_pricelist_item_comp_rule" model="ir.rule">
        <field name="name">product pricelist item company rule</field>
        <field name="model_id" ref="product.model_product_pricelist_item"/>
        <field name="domain_force">['|', ('company_id', 'in', [False,website.company_id.id]), ('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M50.363 41a1.636 1.636 0 0 1-1.59 1.286L27 43l1 4h19c1 0 1 2 0 2H26l-6-24h-2v1c0 .667-.333 1-1 1s-1-.333-1-1v-2c.066-.667.4-1 1-1h4c.517 0 .85.333 1 1l1 3.281h28.45c1.048 0 1.824.985 1.592 2.019L50.363 41zM45.5 55a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm-19 0a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5z"/><path id="e" d="M50.363 39a1.636 1.636 0 0 1-1.59 1.286L27 41l1 4h19c1 0 1 2 0 2H26l-6-24h-2v1c0 .667-.333 1-1 1s-1-.333-1-1v-2c.066-.667.4-1 1-1h4c.517 0 .85.333 1 1l1 3.281h28.45c1.048 0 1.824.985 1.592 2.019L50.363 39zM45.5 53a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm-19 0a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M35 69H4c-2 0-4-1-4-4V37.785l16.292-16.467L21 21l2 5h29l-1.963 13.743L45 45h2l.542 1.59-3.3 3.164 3.3 1.837L35 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\img\snippets_thumbs\s_products_recently_viewed.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-2" d="M16 19v1H8v-1h8zm17 0v1h-9v-1h9zm16 0v1h-9v-1h9z"/>
    <filter id="filter-3" width="102.4%" height="300%" x="-1.2%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <polygon id="path-4" points="0 8.954 5.571 11.28 5.571 4.714 0 2.571"/>
    <filter id="filter-5" width="117.9%" height="123%" x="-9%" y="-5.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <polygon id="path-6" points="6.429 11.28 12 8.954 12 2.571 6.429 4.714"/>
    <filter id="filter-7" width="117.9%" height="123%" x="-9%" y="-5.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <polygon id="path-8" points="0 8.954 5.571 11.28 5.571 4.714 0 2.571"/>
    <filter id="filter-9" width="117.9%" height="123%" x="-9%" y="-5.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <polygon id="path-10" points="6.429 11.28 12 8.954 12 2.571 6.429 4.714"/>
    <filter id="filter-11" width="117.9%" height="123%" x="-9%" y="-5.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <polygon id="path-12" points="0 8.954 5.571 11.28 5.571 4.714 0 2.571"/>
    <filter id="filter-13" width="117.9%" height="123%" x="-9%" y="-5.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <polygon id="path-14" points="6.429 11.28 12 8.954 12 2.571 6.429 4.714"/>
    <filter id="filter-15" width="117.9%" height="123%" x="-9%" y="-5.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_products_recently_viewed">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(13 20)">
        <path fill="url(#linearGradient-1)" d="M17.154 15v2H7v-2h10.154zm16.923 0v2h-11v-2h11zM51 15v2H38.308v-2H51z" class="combined_shape"/>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-2"/>
        </g>
        <g class="box_solid" transform="translate(6)">
          <rect width="12" height="11.143" class="rectangle"/>
          <polygon fill="#FFF" fill-opacity=".78" points="6 .429 0 2.061 6 4.286 12 2.061" class="path"/>
          <g class="path">
            <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-4"/>
          </g>
          <g class="path">
            <use fill="#000" filter="url(#filter-7)" xlink:href="#path-6"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-6"/>
          </g>
        </g>
        <g class="box_solid" transform="translate(38)">
          <rect width="12" height="11.143" class="rectangle"/>
          <polygon fill="#FFF" fill-opacity=".78" points="6 .429 0 2.061 6 4.286 12 2.061" class="path"/>
          <g class="path">
            <use fill="#000" filter="url(#filter-9)" xlink:href="#path-8"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-8"/>
          </g>
          <g class="path">
            <use fill="#000" filter="url(#filter-11)" xlink:href="#path-10"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-10"/>
          </g>
        </g>
        <g class="box_solid" transform="translate(22)">
          <rect width="12" height="11.143" class="rectangle"/>
          <polygon fill="#FFF" fill-opacity=".78" points="6 .429 0 2.061 6 4.286 12 2.061" class="path"/>
          <g class="path">
            <use fill="#000" filter="url(#filter-13)" xlink:href="#path-12"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-12"/>
          </g>
          <g class="path">
            <use fill="#000" filter="url(#filter-15)" xlink:href="#path-14"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-14"/>
          </g>
        </g>
        <path fill="#FFF" stroke="#FFF" d="M1.5 4.793v4.414L-.707 7 1.5 4.793zm53-1L56.707 6 54.5 8.207V3.793z" class="combined_shape"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_products_searchbar.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-2" d="M62 13.5c0-.55-.196-1.022-.587-1.413A1.926 1.926 0 0 0 60 11.5c-.55 0-1.022.196-1.413.587A1.926 1.926 0 0 0 58 13.5c0 .55.196 1.022.587 1.413.391.391.862.587 1.413.587.55 0 1.022-.196 1.413-.587.391-.391.587-.862.587-1.413zm2 3.462a.517.517 0 0 1-.16.378.517.517 0 0 1-.378.16.5.5 0 0 1-.38-.16l-1.442-1.439a2.88 2.88 0 0 1-1.678.522 2.91 2.91 0 0 1-1.151-.233 2.961 2.961 0 0 1-.947-.631 2.961 2.961 0 0 1-.63-.947 2.91 2.91 0 0 1-.234-1.15c0-.402.078-.785.233-1.151a2.98 2.98 0 0 1 .631-.947c.266-.265.581-.475.947-.63a2.91 2.91 0 0 1 1.15-.234c.402 0 .785.078 1.151.233.366.156.682.366.947.631.265.266.475.581.63.947.156.366.234.75.234 1.15a2.88 2.88 0 0 1-.522 1.679l1.443 1.443c.104.104.156.23.156.379z"/>
    <filter id="filter-3" width="114.3%" height="128.6%" x="-7.1%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-4" d="M55 10v8H34v-8h21zm-1 1H35v6h19v-6z"/>
    <filter id="filter-5" width="104.8%" height="125%" x="-2.4%" y="-6.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_products_searchbar">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(0 16)">
        <rect width="82" height="28" fill="url(#linearGradient-1)" class="rectangle" opacity=".4"/>
        <g class="box_solid" transform="translate(18 9.5)">
          <rect width="11" height="10" class="rectangle"/>
          <path fill="#FFF" fill-opacity=".95" d="M0 2.174L5.077 4.1V10L0 7.91V2.174zm11 0V7.91L5.923 10V4.1L11 2.174zM5.5 0L11 1.472 5.5 3.478 0 1.472 5.5 0z" class="combined_shape"/>
        </g>
        <g class="search">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-4"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_products_searchbar_inline.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <path id="path-1" d="M46 4.5c0-.55-.196-1.022-.587-1.413A1.926 1.926 0 0 0 44 2.5c-.55 0-1.022.196-1.413.587A1.926 1.926 0 0 0 42 4.5c0 .55.196 1.022.587 1.413.391.391.862.587 1.413.587.55 0 1.022-.196 1.413-.587.391-.391.587-.862.587-1.413zm2 3.462a.517.517 0 0 1-.16.378.517.517 0 0 1-.378.16.5.5 0 0 1-.38-.16L45.64 6.901a2.88 2.88 0 0 1-1.678.522 2.91 2.91 0 0 1-1.151-.233 2.961 2.961 0 0 1-.947-.631 2.961 2.961 0 0 1-.63-.947A2.91 2.91 0 0 1 41 4.462c0-.402.078-.785.233-1.151a2.98 2.98 0 0 1 .631-.947c.266-.265.581-.475.947-.63a2.91 2.91 0 0 1 1.15-.234c.402 0 .785.078 1.151.233.366.156.682.366.947.631.265.266.475.581.63.947.156.366.234.75.234 1.15a2.88 2.88 0 0 1-.522 1.679l1.443 1.443c.104.104.156.23.156.379z"/>
    <filter id="filter-2" width="114.3%" height="128.6%" x="-7.1%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-3" d="M37 1v8H16V1h21zm-1 1H17v6h19V2z"/>
    <filter id="filter-4" width="104.8%" height="125%" x="-2.4%" y="-6.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_products_searchbar_inline">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(17 25)">
        <g class="box_solid" transform="translate(0 .5)">
          <rect width="11" height="10" class="rectangle"/>
          <path fill="#FFF" fill-opacity=".95" d="M0 2.174L5.077 4.1V10L0 7.91V2.174zm11 0V7.91L5.923 10V4.1L11 2.174zM5.5 0L11 1.472 5.5 3.478 0 1.472 5.5 0z" class="combined_shape"/>
        </g>
        <g class="search">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-1"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-3"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\js\variant_mixin.js

```javascript
odoo.define('website_sale.VariantMixin', function (require) {
'use strict';

var VariantMixin = require('sale.VariantMixin');

/**
 * Website behavior is slightly different from backend so we append
 * "_website" to URLs to lead to a different route
 *
 * @private
 * @param {string} uri The uri to adapt
 */
VariantMixin._getUri = function (uri) {
    if (this.isWebsite){
        return uri + '_website';
    } else {
        return uri;
    }
};

return VariantMixin;

});

```

## File: static\src\js\website_sale.editor.js

```javascript
odoo.define('website_sale.add_product', function (require) {
'use strict';

var core = require('web.core');
var wUtils = require('website.utils');
var WebsiteNewMenu = require('website.newMenu');

var _t = core._t;

WebsiteNewMenu.include({
    actions: _.extend({}, WebsiteNewMenu.prototype.actions || {}, {
        new_product: '_createNewProduct',
    }),

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Asks the user information about a new product to create, then creates it
     * and redirects the user to this new product.
     *
     * @private
     * @returns {Promise} Unresolved if there is a redirection
     */
    _createNewProduct: function () {
        var self = this;
        return wUtils.prompt({
            id: "editor_new_product",
            window_title: _t("New Product"),
            input: _t("Name"),
        }).then(function (result) {
            if (!result.val) {
                return;
            }
            return self._rpc({
                route: '/shop/add_product',
                params: {
                    name: result.val,
                },
            }).then(function (url) {
                window.location.href = url;
                return new Promise(function () {});
            });
        });
    },
});
});

//==============================================================================

odoo.define('website_sale.editor', function (require) {
'use strict';

var options = require('web_editor.snippets.options');
var publicWidget = require('web.public.widget');
const {Class: EditorMenuBar} = require('web_editor.editor');
const {qweb} = require('web.core');

EditorMenuBar.include({
    custom_events: Object.assign(EditorMenuBar.prototype.custom_events, {
        get_ribbons: '_onGetRibbons',
        get_ribbon_classes: '_onGetRibbonClasses',
        delete_ribbon: '_onDeleteRibbon',
        set_ribbon: '_onSetRibbon',
        set_product_ribbon: '_onSetProductRibbon',
    }),

    /**
     * @override
     */
    async willStart() {
        const _super = this._super.bind(this);
        let ribbons = [];
        if (this._isProductListPage()) {
            ribbons = await this._rpc({
                model: 'product.ribbon',
                method: 'search_read',
                fields: ['id', 'html', 'bg_color', 'text_color', 'html_class'],
            });
        }
        this.ribbons = Object.fromEntries(ribbons.map(ribbon => [ribbon.id, ribbon]));
        this.originalRibbons = Object.assign({}, this.ribbons);
        this.productTemplatesRibbons = [];
        this.deletedRibbonClasses = '';
        return _super(...arguments);
    },
    /**
     * @override
     */
    async save() {
        const _super = this._super.bind(this);
        await this._saveRibbons();
        return _super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Saves the ribbons in the database.
     *
     * @private
     */
    async _saveRibbons() {
        if (!this._isProductListPage()) {
            return;
        }
        const originalIds = Object.keys(this.originalRibbons).map(id => parseInt(id));
        const currentIds = Object.keys(this.ribbons).map(id => parseInt(id));

        const ribbons = Object.values(this.ribbons);
        const created = ribbons.filter(ribbon => !originalIds.includes(ribbon.id));
        const deletedIds = originalIds.filter(id => !currentIds.includes(id));
        const modified = ribbons.filter(ribbon => {
            if (created.includes(ribbon)) {
                return false;
            }
            const original = this.originalRibbons[ribbon.id];
            return Object.entries(ribbon).some(([key, value]) => value !== original[key]);
        });

        const proms = [];
        let createdRibbonIds;
        if (created.length > 0) {
            proms.push(this._rpc({
                method: 'create',
                model: 'product.ribbon',
                args: [created.map(ribbon => {
                    ribbon = Object.assign({}, ribbon);
                    delete ribbon.id;
                    return ribbon;
                })],
            }).then(ids => createdRibbonIds = ids));
        }

        modified.forEach(ribbon => proms.push(this._rpc({
            method: 'write',
            model: 'product.ribbon',
            args: [[ribbon.id], ribbon],
        })));

        if (deletedIds.length > 0) {
            proms.push(this._rpc({
                method: 'unlink',
                model: 'product.ribbon',
                args: [deletedIds],
            }));
        }

        await Promise.all(proms);
        const localToServer = Object.assign(
            this.ribbons,
            Object.fromEntries(created.map((ribbon, index) => [ribbon.id, {id: createdRibbonIds[index]}])),
            {'false': {id: false}},
        );

        // Building the final template to ribbon-id map
        const finalTemplateRibbons = this.productTemplatesRibbons.reduce((acc, {templateId, ribbonId}) => {
            acc[templateId] = ribbonId;
            return acc;
        }, {});
        // Inverting the relationship so that we have all templates that have the same ribbon to reduce RPCs
        const ribbonTemplates = Object.entries(finalTemplateRibbons).reduce((acc, [templateId, ribbonId]) => {
            if (!acc[ribbonId]) {
                acc[ribbonId] = [];
            }
            acc[ribbonId].push(parseInt(templateId));
            return acc;
        }, {});
        const setProductTemplateRibbons = Object.entries(ribbonTemplates)
            // If the ribbonId that the template had no longer exists, remove the ribbon (id = false)
            .map(([ribbonId, templateIds]) => {
                const id = currentIds.includes(parseInt(ribbonId)) ? ribbonId : false;
                return [id, templateIds];
            }).map(([ribbonId, templateIds]) => this._rpc({
                method: 'write',
                model: 'product.template',
                args: [templateIds, {'website_ribbon_id': localToServer[ribbonId].id}],
            }));
        return Promise.all(setProductTemplateRibbons);
    },
    /**
     * Checks whether the current page is the product list.
     *
     * @private
     */
    _isProductListPage() {
        return $('#products_grid').length !== 0;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Returns a copy of this.ribbons through a callback.
     *
     * @private
     */
    _onGetRibbons(ev) {
        ev.data.callback(Object.assign({}, this.ribbons));
    },
    /**
     * Returns all ribbon classes, current and deleted, so they can be removed.
     *
     * @private
     */
    _onGetRibbonClasses(ev) {
        const classes = Object.values(this.ribbons).reduce((classes, ribbon) => {
            return classes + ` ${ribbon.html_class}`;
        }, '') + this.deletedRibbonClasses;
        ev.data.callback(classes);
    },
    /**
     * Deletes a ribbon.
     *
     * @private
     */
    _onDeleteRibbon(ev) {
        this.deletedRibbonClasses += ` ${this.ribbons[ev.data.id].html_class}`;
        delete this.ribbons[ev.data.id];
    },
    /**
     * Sets a ribbon;
     *
     * @private
     */
    _onSetRibbon(ev) {
        const {ribbon} = ev.data;
        const previousRibbon = this.ribbons[ribbon.id];
        if (previousRibbon) {
            this.deletedRibbonClasses += ` ${previousRibbon.html_class}`;
        }
        this.ribbons[ribbon.id] = ribbon;
    },
    /**
     * Sets which ribbon is used by a product template.
     *
     * @private
     */
    _onSetProductRibbon(ev) {
        const {templateId, ribbonId} = ev.data;
        this.productTemplatesRibbons.push({templateId, ribbonId});
    },
});

publicWidget.registry.websiteSaleCurrency = publicWidget.Widget.extend({
    selector: '.oe_website_sale',
    disabledInEditableMode: false,
    edit_events: {
        'click .oe_currency_value:o_editable': '_onCurrencyValueClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onCurrencyValueClick: function (ev) {
        $(ev.currentTarget).selectContent();
    },
});

function reload() {
    if (window.location.href.match(/\?enable_editor/)) {
        window.location.reload();
    } else {
        window.location.href = window.location.href.replace(/\?(enable_editor=1&)?|#.*|$/, '?enable_editor=1&');
    }
}

options.registry.WebsiteSaleGridLayout = options.Class.extend({

    /**
     * @override
     */
    start: function () {
        this.ppg = parseInt(this.$target.closest('[data-ppg]').data('ppg'));
        this.ppr = parseInt(this.$target.closest('[data-ppr]').data('ppr'));
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    onFocus: function () {
        var listLayoutEnabled = this.$target.closest('#products_grid').hasClass('o_wsale_layout_list');
        this.$el.filter('.o_wsale_ppr_submenu').toggleClass('d-none', listLayoutEnabled);
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for params
     */
    setPpg: function (previewMode, widgetValue, params) {
        const PPG_LIMIT = 10000;
        const ppg = parseInt(widgetValue);
        if (!ppg || ppg < 1) {
            return false;
        }
        this.ppg = Math.min(ppg, PPG_LIMIT);
        return this._rpc({
            route: '/shop/change_ppg',
            params: {
                'ppg': this.ppg,
            },
        }).then(() => reload());
    },
    /**
     * @see this.selectClass for params
     */
    setPpr: function (previewMode, widgetValue, params) {
        this.ppr = parseInt(widgetValue);
        this._rpc({
            route: '/shop/change_ppr',
            params: {
                'ppr': this.ppr,
            },
        }).then(reload);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState: function (methodName, params) {
        switch (methodName) {
            case 'setPpg': {
                return this.ppg;
            }
            case 'setPpr': {
                return this.ppr;
            }
        }
        return this._super(...arguments);
    },
});

options.registry.WebsiteSaleProductsItem = options.Class.extend({
    xmlDependencies: (options.Class.prototype.xmlDependencies || []).concat(['/website_sale/static/src/xml/website_sale_utils.xml']),
    events: _.extend({}, options.Class.prototype.events || {}, {
        'mouseenter .o_wsale_soptions_menu_sizes table': '_onTableMouseEnter',
        'mouseleave .o_wsale_soptions_menu_sizes table': '_onTableMouseLeave',
        'mouseover .o_wsale_soptions_menu_sizes td': '_onTableItemMouseEnter',
        'click .o_wsale_soptions_menu_sizes td': '_onTableItemClick',
    }),

    /**
     * @override
     */
    willStart: async function () {
        const _super = this._super.bind(this);
        this.ppr = this.$target.closest('[data-ppr]').data('ppr');
        this.productTemplateID = parseInt(this.$target.find('[data-oe-model="product.template"]').data('oe-id'));
        this.ribbons = await new Promise(resolve => this.trigger_up('get_ribbons', {callback: resolve}));
        return _super(...arguments);
    },
    /**
     * @override
     */
    start: function () {
        this._resetRibbonDummy();
        return this._super(...arguments);
    },
    /**
     * @override
     */
    onFocus: function () {
        var listLayoutEnabled = this.$target.closest('#products_grid').hasClass('o_wsale_layout_list');
        this.$el.find('.o_wsale_soptions_menu_sizes')
            .toggleClass('d-none', listLayoutEnabled);
        // Ribbons may have been edited or deleted in another products' option, need to make sure they're up to date
        this.rerender = true;
    },
    /**
     * @override
     */
    onBlur: function () {
        // Since changes will not be saved unless they are validated, reset the
        // previewed ribbon onBlur to communicate that to the user
        this._resetRibbonDummy();
        this._toggleEditingUI(false);
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    selectStyle(previewMode, widgetValue, params) {
        const proms = [this._super(...arguments)];
        if (params.cssProperty === 'background-color' && params.colorNames.includes(widgetValue)) {
            // Reset text-color when choosing a background-color class, so it uses the automatic text-color of the class.
            proms.push(this.selectStyle(previewMode, '', {applyTo: '.o_wsale_ribbon_dummy', cssProperty: 'color'}));
        }
        return Promise.all(proms);
    },
    /**
     * @see this.selectClass for params
     */
    async setRibbon(previewMode, widgetValue, params) {
        if (previewMode === 'reset') {
            widgetValue = this.prevRibbonId;
        } else {
            this.prevRibbonId = this.$target[0].dataset.ribbonId;
        }
        this.$target[0].dataset.ribbonId = widgetValue;
        this.trigger_up('set_product_ribbon', {
            templateId: this.productTemplateID,
            ribbonId: widgetValue || false,
        });
        const ribbon = this.ribbons[widgetValue] || {html: '', bg_color: '', text_color: '', html_class: ''};
        const $ribbons = $(`[data-ribbon-id="${widgetValue}"] .o_ribbon:not(.o_wsale_ribbon_dummy)`);
        $ribbons.html(ribbon.html);
        let htmlClasses;
        this.trigger_up('get_ribbon_classes', {callback: classes => htmlClasses = classes});
        $ribbons.removeClass(htmlClasses);

        $ribbons.addClass(ribbon.html_class || '');
        $ribbons.css('color', ribbon.text_color || '');
        $ribbons.css('background-color', ribbon.bg_color || '');

        if (!this.ribbons[widgetValue]) {
            $(`[data-ribbon-id="${widgetValue}"]`).each((index, product) => delete product.dataset.ribbonId);
        }
        this._resetRibbonDummy();
        this._toggleEditingUI(false);
    },
    /**
     * @see this.selectClass for params
     */
    editRibbon(previewMode, widgetValue, params) {
        this.saveMethod = 'modify';
        this._toggleEditingUI(true);
    },
    /**
     * @see this.selectClass for params
     */
    createRibbon(previewMode, widgetValue, params) {
        this.saveMethod = 'create';
        this.setRibbon(false);
        this.$ribbon.html('Ribbon text');
        this.$ribbon.addClass('bg-primary o_ribbon_left');
        this._toggleEditingUI(true);
        this.isCreating = true;
    },
    /**
     * @see this.selectClass for params
     */
    async deleteRibbon(previewMode, widgetValue, params) {
        if (this.isCreating) {
            // Ribbon doesn't exist yet, simply discard.
            this.isCreating = false;
            this._resetRibbonDummy();
            return this._toggleEditingUI(false);
        }
        const {ribbonId} = this.$target[0].dataset;
        this.trigger_up('delete_ribbon', {id: ribbonId});
        this.ribbons = await new Promise(resolve => this.trigger_up('get_ribbons', {callback: resolve}));
        this.rerender = true;
        await this.setRibbon(false, ribbonId);
    },
    /**
     * @see this.selectClass for params
     */
    async saveRibbon(previewMode, widgetValue, params) {
        const text = this.$ribbon.html().trim();
        if (!text) {
            return;
        }
        const ribbon = {
            'html': text,
            'bg_color': this.$ribbon[0].style.backgroundColor,
            'text_color': this.$ribbon[0].style.color,
            'html_class': this.$ribbon.attr('class').split(' ')
                .filter(c => !['d-none', 'o_wsale_ribbon_dummy', 'o_ribbon'].includes(c))
                .join(' '),
        };
        ribbon.id = this.saveMethod === 'modify' ? parseInt(this.$target[0].dataset.ribbonId) : Date.now();
        this.trigger_up('set_ribbon', {ribbon: ribbon});
        this.ribbons = await new Promise(resolve => this.trigger_up('get_ribbons', {callback: resolve}));
        this.rerender = true;
        await this.setRibbon(false, ribbon.id);
    },
    /**
     * @see this.selectClass for params
     */
    setRibbonHtml(previewMode, widgetValue, params) {
        this.$ribbon.html(widgetValue);
    },
    /**
     * @see this.selectClass for params
     */
    setRibbonMode(previewMode, widgetValue, params) {
        this.$ribbon[0].className = this.$ribbon[0].className.replace(/o_(ribbon|tag)_(left|right)/, `o_${widgetValue}_$2`);
    },
    /**
     * @see this.selectClass for params
     */
    setRibbonPosition(previewMode, widgetValue, params) {
        this.$ribbon[0].className = this.$ribbon[0].className.replace(/o_(ribbon|tag)_(left|right)/, `o_$1_${widgetValue}`);
    },
    /**
     * @see this.selectClass for params
     */
    changeSequence: function (previewMode, widgetValue, params) {
        this._rpc({
            route: '/shop/change_sequence',
            params: {
                id: this.productTemplateID,
                sequence: widgetValue,
            },
        }).then(reload);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    updateUI: async function () {
        await this._super.apply(this, arguments);

        var sizeX = parseInt(this.$target.attr('colspan') || 1);
        var sizeY = parseInt(this.$target.attr('rowspan') || 1);

        var $size = this.$el.find('.o_wsale_soptions_menu_sizes');
        $size.find('tr:nth-child(-n + ' + sizeY + ') td:nth-child(-n + ' + sizeX + ')')
             .addClass('selected');

        // Adapt size array preview to fit ppr
        $size.find('tr td:nth-child(n + ' + parseInt(this.ppr + 1) + ')').hide();
        if (this.rerender) {
            this.rerender = false;
            return this._rerenderXML();
        }
    },
    /**
     * @override
     */
    updateUIVisibility: async function () {
        // Main updateUIVisibility will remove the d-none class because there are visible widgets
        // inside of it. TODO: update this once updateUIVisibility can be used to compute visibility
        // of arbitrary DOM elements and not just widgets.
        const isEditing = this.$el.find('[data-name="ribbon_options"]').hasClass('d-none');
        await this._super(...arguments);
        this._toggleEditingUI(isEditing);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async _renderCustomXML(uiFragment) {
        const $select = $(uiFragment.querySelector('.o_wsale_ribbon_select'));
        this.ribbons = await new Promise(resolve => this.trigger_up('get_ribbons', {callback: resolve}));
        if (!this.$ribbon) {
            this._resetRibbonDummy();
        }
        const classes = this.$ribbon[0].className;
        this.$ribbon[0].className = '';
        const defaultTextColor = window.getComputedStyle(this.$ribbon[0]).color;
        this.$ribbon[0].className = classes;
        Object.values(this.ribbons).forEach(ribbon => {
            const colorClasses = ribbon.html_class
                .split(' ')
                .filter(className => !/^o_(ribbon|tag)_(left|right)$/.test(className))
                .join(' ');
            $select.append(qweb.render('website_sale.ribbonSelectItem', {
                ribbon,
                colorClasses,
                isTag: /o_tag_(left|right)/.test(ribbon.html_class),
                isLeft: /o_(tag|ribbon)_left/.test(ribbon.html_class),
                textColor: ribbon.text_color || (colorClasses ? 'currentColor' : defaultTextColor),
            }));
        });
    },
    /**
     * @override
     */
    async _computeWidgetState(methodName, params) {
        const classList = this.$ribbon[0].classList;
        switch (methodName) {
            case 'setRibbon':
                return this.$target.attr('data-ribbon-id') || '';
            case 'setRibbonHtml':
                return this.$ribbon.html();
            case 'setRibbonMode': {
                if (classList.contains('o_ribbon_left') || classList.contains('o_ribbon_right')) {
                    return 'ribbon';
                }
                return 'tag';
            }
            case 'setRibbonPosition': {
                if (classList.contains('o_tag_left') || classList.contains('o_ribbon_left')) {
                    return 'left';
                }
                return 'right';
            }
        }
        return this._super(methodName, params);
    },
    /**
     * Toggles the UI mode between select and create/edit mode.
     *
     * @private
     * @param {Boolean} state true to activate editing UI, false to deactivate.
     */
    _toggleEditingUI(state) {
        this.$el.find('[data-name="ribbon_options"]').toggleClass('d-none', state);
        this.$el.find('[data-name="ribbon_customize_opt"]').toggleClass('d-none', !state);
        this.$('.o_ribbon:not(.o_wsale_ribbon_dummy)').toggleClass('d-none', state);
        this.$ribbon.toggleClass('d-none', !state);
    },
    /**
     * Creates a copy of current ribbon to manipulate for edition/creation.
     *
     * @private
     */
    _resetRibbonDummy() {
        if (this.$ribbon) {
            this.$ribbon.remove();
        }
        const $original = this.$('.o_ribbon');
        this.$ribbon = $original.clone().addClass('d-none o_wsale_ribbon_dummy').appendTo($original.parent());
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onTableMouseEnter: function (ev) {
        $(ev.currentTarget).addClass('oe_hover');
    },
    /**
     * @private
     */
    _onTableMouseLeave: function (ev) {
        $(ev.currentTarget).removeClass('oe_hover');
    },
    /**
     * @private
     */
    _onTableItemMouseEnter: function (ev) {
        var $td = $(ev.currentTarget);
        var $table = $td.closest("table");
        var x = $td.index() + 1;
        var y = $td.parent().index() + 1;

        var tr = [];
        for (var yi = 0; yi < y; yi++) {
            tr.push("tr:eq(" + yi + ")");
        }
        var $selectTr = $table.find(tr.join(","));
        var td = [];
        for (var xi = 0; xi < x; xi++) {
            td.push("td:eq(" + xi + ")");
        }
        var $selectTd = $selectTr.find(td.join(","));

        $table.find("td").removeClass("select");
        $selectTd.addClass("select");
    },
    /**
     * @private
     */
    _onTableItemClick: function (ev) {
        var $td = $(ev.currentTarget);
        var x = $td.index() + 1;
        var y = $td.parent().index() + 1;
        this._rpc({
            route: '/shop/change_size',
            params: {
                id: this.productTemplateID,
                x: x,
                y: y,
            },
        }).then(reload);
    },
});
});

```

## File: static\src\js\website_sale.js

```javascript
odoo.define('website_sale.cart', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var core = require('web.core');
var _t = core._t;

var timeout;

publicWidget.registry.websiteSaleCartLink = publicWidget.Widget.extend({
    // TODO in master: remove the second selector.
    selector: '#top a[href$="/shop/cart"]:not(.js_change_lang), #top_menu a[href$="/shop/cart"]:not(.js_change_lang)',
    events: {
        'mouseenter': '_onMouseEnter',
        'mouseleave': '_onMouseLeave',
        'click': '_onClick',
    },

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        this._popoverRPC = null;
    },
    /**
     * @override
     */
    start: function () {
        this.$el.popover({
            trigger: 'manual',
            animation: true,
            html: true,
            title: function () {
                return _t("My Cart");
            },
            container: 'body',
            placement: 'auto',
            template: '<div class="popover mycart-popover" role="tooltip"><div class="arrow"></div><h3 class="popover-header"></h3><div class="popover-body"></div></div>'
        });
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onMouseEnter: function (ev) {
        var self = this;
        clearTimeout(timeout);
        $(this.selector).not(ev.currentTarget).popover('hide');
        timeout = setTimeout(function () {
            if (!self.$el.is(':hover') || $('.mycart-popover:visible').length) {
                return;
            }
            self._popoverRPC = $.get("/shop/cart", {
                type: 'popover',
            }).then(function (data) {
                self.$el.data("bs.popover").config.content = data;
                self.$el.popover("show");
                $('.popover').on('mouseleave', function () {
                    self.$el.trigger('mouseleave');
                });
            });
        }, 300);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onMouseLeave: function (ev) {
        var self = this;
        setTimeout(function () {
            if ($('.popover:hover').length) {
                return;
            }
            if (!self.$el.is(':hover')) {
               self.$el.popover('hide');
            }
        }, 1000);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClick: function (ev) {
        // When clicking on the cart link, prevent any popover to show up (by
        // clearing the related setTimeout) and, if a popover rpc is ongoing,
        // wait for it to be completed before going to the link's href. Indeed,
        // going to that page may perform the same computation the popover rpc
        // is already doing.
        clearTimeout(timeout);
        if (this._popoverRPC && this._popoverRPC.state() === 'pending') {
            ev.preventDefault();
            var href = ev.currentTarget.href;
            this._popoverRPC.then(function () {
                window.location.href = href;
            });
        }
    },
});
});

odoo.define('website_sale.website_sale_category', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.websiteSaleCategory = publicWidget.Widget.extend({
    selector: '#o_shop_collapse_category',
    events: {
        'click .fa-chevron-right': '_onOpenClick',
        'click .fa-chevron-down': '_onCloseClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onOpenClick: function (ev) {
        var $fa = $(ev.currentTarget);
        $fa.parent().siblings().find('.fa-chevron-down:first').click();
        $fa.parents('li').find('ul:first').show('normal');
        $fa.toggleClass('fa-chevron-down fa-chevron-right');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onCloseClick: function (ev) {
        var $fa = $(ev.currentTarget);
        $fa.parent().find('ul:first').hide('normal');
        $fa.toggleClass('fa-chevron-down fa-chevron-right');
    },
});
});

odoo.define('website_sale.website_sale', function (require) {
'use strict';

var core = require('web.core');
var config = require('web.config');
var publicWidget = require('web.public.widget');
var VariantMixin = require('sale.VariantMixin');
var wSaleUtils = require('website_sale.utils');
const wUtils = require('website.utils');
require("web.zoomodoo");


publicWidget.registry.WebsiteSale = publicWidget.Widget.extend(VariantMixin, {
    selector: '.oe_website_sale',
    events: _.extend({}, VariantMixin.events || {}, {
        'change form .js_product:first input[name="add_qty"]': '_onChangeAddQuantity',
        'mouseup .js_publish': '_onMouseupPublish',
        'touchend .js_publish': '_onMouseupPublish',
        'change .oe_cart input.js_quantity[data-product-id]': '_onChangeCartQuantity',
        'click .oe_cart a.js_add_suggested_products': '_onClickSuggestedProduct',
        'click a.js_add_cart_json': '_onClickAddCartJSON',
        'click .a-submit': '_onClickSubmit',
        'change form.js_attributes input, form.js_attributes select': '_onChangeAttribute',
        'mouseup form.js_add_cart_json label': '_onMouseupAddCartLabel',
        'touchend form.js_add_cart_json label': '_onMouseupAddCartLabel',
        'click .show_coupon': '_onClickShowCoupon',
        'submit .o_wsale_products_searchbar_form': '_onSubmitSaleSearch',
        'change select[name="country_id"]': '_onChangeCountry',
        'change #shipping_use_same': '_onChangeShippingUseSame',
        'click .toggle_summary': '_onToggleSummary',
        'click #add_to_cart, #buy_now, #products_grid .o_wsale_product_btn .a-submit': 'async _onClickAdd',
        'click input.js_product_change': 'onChangeVariant',
        'change .js_main_product [data-attribute_exclusions]': 'onChangeVariant',
        'change oe_optional_products_modal [data-attribute_exclusions]': 'onChangeVariant',
    }),

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);

        this._changeCartQuantity = _.debounce(this._changeCartQuantity.bind(this), 500);
        this._changeCountry = _.debounce(this._changeCountry.bind(this), 500);

        this.isWebsite = true;

        delete this.events['change .main_product:not(.in_cart) input.js_quantity'];
        delete this.events['change [data-attribute_exclusions]'];
    },
    /**
     * @override
     */
    start() {
        const def = this._super(...arguments);

        this._applyHashFromSearch();

        _.each(this.$('div.js_product'), function (product) {
            $('input.js_product_change', product).first().trigger('change');
        });

        // This has to be triggered to compute the "out of stock" feature and the hash variant changes
        this.triggerVariantChange(this.$el);

        this.$('select[name="country_id"]').change();

        core.bus.on('resize', this, function () {
            if (config.device.size_class === config.device.SIZES.XL) {
                $('.toggle_summary_div').addClass('d-none d-xl-block');
            }
        });

        this._startZoom();

        window.addEventListener('hashchange', () => {
            this._applyHash();
            this.triggerVariantChange(this.$el);
        });

        return def;
    },
    /**
     * The selector is different when using list view of variants.
     *
     * @override
     */
    getSelectedVariantValues: function ($container) {
        var combination = $container.find('input.js_product_change:checked')
            .data('combination');

        if (combination) {
            return combination;
        }
        return VariantMixin.getSelectedVariantValues.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _applyHash: function () {
        var hash = window.location.hash.substring(1);
        if (hash) {
            var params = $.deparam(hash);
            if (params['attr']) {
                var attributeIds = params['attr'].split(',');
                var $inputs = this.$('input.js_variant_change, select.js_variant_change option');
                _.each(attributeIds, function (id) {
                    var $toSelect = $inputs.filter('[data-value_id="' + id + '"]');
                    if ($toSelect.is('input[type="radio"]')) {
                        $toSelect.prop('checked', true);
                    } else if ($toSelect.is('option')) {
                        $toSelect.prop('selected', true);
                    }
                });
                this._changeColorAttribute();
            }
        }
    },

    /**
     * Sets the url hash from the selected product options.
     *
     * @private
     */
    _setUrlHash: function ($parent) {
        var $attributes = $parent.find('input.js_variant_change:checked, select.js_variant_change option:selected');
        var attributeIds = _.map($attributes, function (elem) {
            return $(elem).data('value_id');
        });
        history.replaceState(undefined, undefined, '#attr=' + attributeIds.join(','));
    },
    /**
     * Set the checked color active.
     *
     * @private
     */
    _changeColorAttribute: function () {
        $('.css_attribute_color').removeClass("active")
                                 .filter(':has(input:checked)')
                                 .addClass("active");
    },
    /**
     * @private
     */
    _changeCartQuantity: function ($input, value, $dom_optional, line_id, productIDs) {
        _.each($dom_optional, function (elem) {
            $(elem).find('.js_quantity').text(value);
            productIDs.push($(elem).find('span[data-product-id]').data('product-id'));
        });
        $input.data('update_change', true);

        this._rpc({
            route: "/shop/cart/update_json",
            params: {
                line_id: line_id,
                product_id: parseInt($input.data('product-id'), 10),
                set_qty: value
            },
        }).then(function (data) {
            $input.data('update_change', false);
            var check_value = parseInt($input.val() || 0, 10);
            if (isNaN(check_value)) {
                check_value = 1;
            }
            if (value !== check_value) {
                $input.trigger('change');
                return;
            }
            if (!data.cart_quantity) {
                return window.location = '/shop/cart';
            }
            $input.val(data.quantity);

            wSaleUtils.updateCartNavBar(data);
            wSaleUtils.showWarning(data.warning);
        });
    },
    /**
     * @private
     */
    _changeCountry: function () {
        if (!$("#country_id").val()) {
            return;
        }
        this._rpc({
            route: "/shop/country_infos/" + $("#country_id").val(),
            params: {
                mode: $("#country_id").attr('mode'),
            },
        }).then(function (data) {
            // placeholder phone_code
            $("input[name='phone']").attr('placeholder', data.phone_code !== 0 ? '+'+ data.phone_code : '');

            // populate states and display
            var selectStates = $("select[name='state_id']");
            // dont reload state at first loading (done in qweb)
            if (selectStates.data('init')===0 || selectStates.find('option').length===1) {
                if (data.states.length || data.state_required) {
                    selectStates.html('');
                    _.each(data.states, function (x) {
                        var opt = $('<option>').text(x[1])
                            .attr('value', x[0])
                            .attr('data-code', x[2]);
                        selectStates.append(opt);
                    });
                    selectStates.parent('div').show();
                } else {
                    selectStates.val('').parent('div').hide();
                }
                selectStates.data('init', 0);
            } else {
                selectStates.data('init', 0);
            }

            // manage fields order / visibility
            if (data.fields) {
                if ($.inArray('zip', data.fields) > $.inArray('city', data.fields)){
                    $(".div_zip").before($(".div_city"));
                } else {
                    $(".div_zip").after($(".div_city"));
                }
                var all_fields = ["street", "zip", "city", "country_name"]; // "state_code"];
                _.each(all_fields, function (field) {
                    $(".checkout_autoformat .div_" + field.split('_')[0]).toggle($.inArray(field, data.fields)>=0);
                });
            }

            if ($("label[for='zip']").length) {
                $("label[for='zip']").toggleClass('label-optional', !data.zip_required);
                $("label[for='zip']").get(0).toggleAttribute('required', !!data.zip_required);
            }
            if ($("label[for='zip']").length) {
                $("label[for='state_id']").toggleClass('label-optional', !data.state_required);
                $("label[for='state_id']").get(0).toggleAttribute('required', !!data.state_required);
            }
        });
    },
    /**
     * This is overridden to handle the "List View of Variants" of the web shop.
     * That feature allows directly selecting the variant from a list instead of selecting the
     * attribute values.
     *
     * Since the layout is completely different, we need to fetch the product_id directly
     * from the selected variant.
     *
     * @override
     */
    _getProductId: function ($parent) {
        if ($parent.find('input.js_product_change').length !== 0) {
            return parseInt($parent.find('input.js_product_change:checked').val());
        }
        else {
            return VariantMixin._getProductId.apply(this, arguments);
        }
    },
    /**
     * @private
     */
    _startZoom: function () {
        // Do not activate image zoom for mobile devices, since it might prevent users from scrolling the page
        if (!config.device.isMobile) {
            var autoZoom = $('.ecom-zoomable').data('ecom-zoom-auto') || false,
            attach = '#o-carousel-product';
            _.each($('.ecom-zoomable img[data-zoom]'), function (el) {
                onImageLoaded(el, function () {
                    var $img = $(el);
                    $img.zoomOdoo({event: autoZoom ? 'mouseenter' : 'click', attach: attach});
                    $img.attr('data-zoom', 1);
                });
            });
        }

        function onImageLoaded(img, callback) {
            // On Chrome the load event already happened at this point so we
            // have to rely on complete. On Firefox it seems that the event is
            // always triggered after this so we can rely on it.
            //
            // However on the "complete" case we still want to keep listening to
            // the event because if the image is changed later (eg. product
            // configurator) a new load event will be triggered (both browsers).
            $(img).on('load', function () {
                callback();
            });
            if (img.complete) {
                callback();
            }
        }
    },
    /**
     * On website, we display a carousel instead of only one image
     *
     * @override
     * @private
     */
    _updateProductImage: function ($productContainer, displayImage, productId, productTemplateId, newCarousel, isCombinationPossible) {
        var $carousel = $productContainer.find('#o-carousel-product');
        // When using the web editor, don't reload this or the images won't
        // be able to be edited depending on if this is done loading before
        // or after the editor is ready.
        if (window.location.search.indexOf('enable_editor') === -1) {
            var $newCarousel = $(newCarousel);
            $carousel.after($newCarousel);
            $carousel.remove();
            $carousel = $newCarousel;
            $carousel.carousel(0);
            this._startZoom();
            // fix issue with carousel height
            this.trigger_up('widgets_start_request', {$target: $carousel});
        }
        $carousel.toggleClass('css_not_available', !isCombinationPossible);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickAdd: function (ev) {
        ev.preventDefault();
        var def = () => {
            this.isBuyNow = $(ev.currentTarget).attr('id') === 'buy_now';
            return this._handleAdd($(ev.currentTarget).closest('form'));
        };
        if ($('.js_add_cart_variants').children().length) {
            return this._getCombinationInfo(ev).then(() => {
                return !$(ev.target).closest('.js_product').hasClass("css_not_available") ? def() : Promise.resolve();
            });
        }
        return def();
    },
    /**
     * Initializes the optional products modal
     * and add handlers to the modal events (confirm, back, ...)
     *
     * @private
     * @param {$.Element} $form the related webshop form
     */
    _handleAdd: function ($form) {
        var self = this;
        this.$form = $form;

        var productSelector = [
            'input[type="hidden"][name="product_id"]',
            'input[type="radio"][name="product_id"]:checked'
        ];

        var productReady = this.selectOrCreateProduct(
            $form,
            parseInt($form.find(productSelector.join(', ')).first().val(), 10),
            $form.find('.product_template_id').val(),
            false
        );

        return productReady.then(function (productId) {
            $form.find(productSelector.join(', ')).val(productId);

            self.rootProduct = {
                product_id: productId,
                quantity: parseFloat($form.find('input[name="add_qty"]').val() || 1),
                product_custom_attribute_values: self.getCustomVariantValues($form.find('.js_product')),
                variant_values: self.getSelectedVariantValues($form.find('.js_product')),
                no_variant_attribute_values: self.getNoVariantAttributeValues($form.find('.js_product'))
            };

            return self._onProductReady();
        });
    },

    _onProductReady: function () {
        return this._submitForm();
    },

    /**
     * Add custom variant values and attribute values that do not generate variants
     * in the form data and trigger submit.
     *
     * @private
     * @returns {Promise} never resolved
     */
    _submitForm: function () {
        let params = this.rootProduct;
        params.add_qty = params.quantity;

        params.product_custom_attribute_values = JSON.stringify(params.product_custom_attribute_values);
        params.no_variant_attribute_values = JSON.stringify(params.no_variant_attribute_values);

        if (this.isBuyNow) {
            params.express = true;
        }

        return wUtils.sendRequest('/shop/cart/update', params);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickAddCartJSON: function (ev){
        this.onClickAddCartJSON(ev);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeAddQuantity: function (ev) {
        this.onChangeAddQuantity(ev);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onMouseupPublish: function (ev) {
        $(ev.currentTarget).parents('.thumbnail').toggleClass('disabled');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeCartQuantity: function (ev) {
        var $input = $(ev.currentTarget);
        if ($input.data('update_change')) {
            return;
        }
        var value = parseInt($input.val() || 0, 10);
        if (isNaN(value)) {
            value = 1;
        }
        var $dom = $input.closest('tr');
        // var default_price = parseFloat($dom.find('.text-danger > span.oe_currency_value').text());
        var $dom_optional = $dom.nextUntil(':not(.optional_product.info)');
        var line_id = parseInt($input.data('line-id'), 10);
        var productIDs = [parseInt($input.data('product-id'), 10)];
        this._changeCartQuantity($input, value, $dom_optional, line_id, productIDs);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickSuggestedProduct: function (ev) {
        $(ev.currentTarget).prev('input').val(1).trigger('change');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickSubmit: function (ev, forceSubmit) {
        if ($(ev.currentTarget).is('#add_to_cart, #products_grid .a-submit') && !forceSubmit) {
            return;
        }
        var $aSubmit = $(ev.currentTarget);
        if (!ev.isDefaultPrevented() && !$aSubmit.is(".disabled")) {
            ev.preventDefault();
            $aSubmit.closest('form').submit();
        }
        if ($aSubmit.hasClass('a-submit-disable')){
            $aSubmit.addClass("disabled");
        }
        if ($aSubmit.hasClass('a-submit-loading')){
            var loading = '<span class="fa fa-cog fa-spin"/>';
            var fa_span = $aSubmit.find('span[class*="fa"]');
            if (fa_span.length){
                fa_span.replaceWith(loading);
            } else {
                $aSubmit.append(loading);
            }
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeAttribute: function (ev) {
        if (!ev.isDefaultPrevented()) {
            ev.preventDefault();
            $(ev.currentTarget).closest("form").submit();
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onMouseupAddCartLabel: function (ev) { // change price when they are variants
        var $label = $(ev.currentTarget);
        var $price = $label.parents("form:first").find(".oe_price .oe_currency_value");
        if (!$price.data("price")) {
            $price.data("price", parseFloat($price.text()));
        }
        var value = $price.data("price") + parseFloat($label.find(".badge span").text() || 0);

        var dec = value % 1;
        $price.html(value + (dec < 0.01 ? ".00" : (dec < 1 ? "0" : "") ));
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickShowCoupon: function (ev) {
        $(ev.currentTarget).hide();
        $('.coupon_form').removeClass('d-none');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onSubmitSaleSearch: function (ev) {
        if (!this.$('.dropdown_sorty_by').length) {
            return;
        }
        var $this = $(ev.currentTarget);
        if (!ev.isDefaultPrevented() && !$this.is(".disabled")) {
            ev.preventDefault();
            var oldurl = $this.attr('action');
            oldurl += (oldurl.indexOf("?")===-1) ? "?" : "";
            var search = $this.find('input.search-query');
            window.location = oldurl + '&' + search.attr('name') + '=' + encodeURIComponent(search.val());
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeCountry: function (ev) {
        if (!this.$('.checkout_autoformat').length) {
            return;
        }
        this._changeCountry();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeShippingUseSame: function (ev) {
        $('.ship_to_other').toggle(!$(ev.currentTarget).prop('checked'));
    },
    /**
     * Toggles the add to cart button depending on the possibility of the
     * current combination.
     *
     * @override
     */
    _toggleDisable: function ($parent, isCombinationPossible) {
        VariantMixin._toggleDisable.apply(this, arguments);
        $parent.find("#add_to_cart").toggleClass('disabled', !isCombinationPossible);
        $parent.find("#buy_now").toggleClass('disabled', !isCombinationPossible);
    },
    /**
     * Write the properties of the form elements in the DOM to prevent the
     * current selection from being lost when activating the web editor.
     *
     * @override
     */
    onChangeVariant: function (ev) {
        var $component = $(ev.currentTarget).closest('.js_product');
        $component.find('input').each(function () {
            var $el = $(this);
            $el.attr('checked', $el.is(':checked'));
        });
        $component.find('select option').each(function () {
            var $el = $(this);
            $el.attr('selected', $el.is(':selected'));
        });

        this._setUrlHash($component);

        return VariantMixin.onChangeVariant.apply(this, arguments);
    },
    /**
     * @private
     */
    _onToggleSummary: function () {
        $('.toggle_summary_div').toggleClass('d-none');
        $('.toggle_summary_div').removeClass('d-xl-block');
    },
    /**
     * @private
     */
    _applyHashFromSearch() {
        const params = $.deparam(window.location.search.slice(1));
        if (params.attrib) {
            const dataValueIds = [];
            for (const attrib of [].concat(params.attrib)) {
                const attribSplit = attrib.split('-');
                const attribValueSelector = `.js_variant_change[name="ptal-${attribSplit[0]}"][value="${attribSplit[1]}"]`;
                const attribValue = this.el.querySelector(attribValueSelector);
                if (attribValue !== null) {
                    dataValueIds.push(attribValue.dataset.value_id);
                }
            }
            if (dataValueIds.length) {
                history.replaceState(undefined, undefined, `#attr=${dataValueIds.join(',')}`);
            }
        }
        this._applyHash();
    },
});

publicWidget.registry.WebsiteSaleLayout = publicWidget.Widget.extend({
    selector: '.oe_website_sale',
    disabledInEditableMode: false,
    events: {
        'change .o_wsale_apply_layout': '_onApplyShopLayoutChange',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onApplyShopLayoutChange: function (ev) {
        var switchToList = $(ev.currentTarget).find('.o_wsale_apply_list input').is(':checked');
        if (!this.editableMode) {
            this._rpc({
                route: '/shop/save_shop_layout_mode',
                params: {
                    'layout_mode': switchToList ? 'list' : 'grid',
                },
            });
        }
        var $grid = this.$('#products_grid');
        // Disable transition on all list elements, then switch to the new
        // layout then reenable all transitions after having forced a redraw
        // TODO should probably be improved to allow disabling transitions
        // altogether with a class/option.
        $grid.find('*').css('transition', 'none');
        $grid.toggleClass('o_wsale_layout_list', switchToList);
        void $grid[0].offsetWidth;
        $grid.find('*').css('transition', '');
    },
});

publicWidget.registry.websiteSaleCart = publicWidget.Widget.extend({
    selector: '.oe_website_sale .oe_cart',
    events: {
        'click .js_change_shipping': '_onClickChangeShipping',
        'click .js_edit_address': '_onClickEditAddress',
        'click .js_delete_product': '_onClickDeleteProduct',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onClickChangeShipping: function (ev) {
        var $old = $('.all_shipping').find('.card.border.border-primary');
        $old.find('.btn-ship').toggle();
        $old.addClass('js_change_shipping');
        $old.removeClass('border border-primary');

        var $new = $(ev.currentTarget).parent('div.one_kanban').find('.card');
        $new.find('.btn-ship').toggle();
        $new.removeClass('js_change_shipping');
        $new.addClass('border border-primary');

        var $form = $(ev.currentTarget).parent('div.one_kanban').find('form.d-none');
        $.post($form.attr('action'), $form.serialize()+'&xhr=1');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickEditAddress: function (ev) {
        ev.preventDefault();
        $(ev.currentTarget).closest('div.one_kanban').find('form.d-none').attr('action', '/shop/address').submit();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickDeleteProduct: function (ev) {
        ev.preventDefault();
        $(ev.currentTarget).closest('tr').find('.js_quantity').val(0).trigger('change');
    },
});
});

```

## File: static\src\js\website_sale_backend.js

```javascript
odoo.define('website_sale.backend', function (require) {
"use strict";

var WebsiteBackend = require('website.backend.dashboard');
var COLORS = ['#875a7b', '#21b799', '#E4A900', '#D5653E', '#5B899E', '#E46F78', '#8F8F8F'];

WebsiteBackend.include({
    jsLibs: [
        '/web/static/lib/Chart/Chart.js',
    ],

    events: _.defaults({
        'click tr.o_product_template': 'on_product_template',
        'click .js_utm_selector': '_onClickUtmButton',
    }, WebsiteBackend.prototype.events),

    init: function (parent, context) {
        this._super(parent, context);

        this.graphs.push({'name': 'sales', 'group': 'sale_salesman'});
    },

    /**
     * @override method from website backendDashboard
     * @private
     */
    render_graphs: function() {
        this._super();
        this.utmGraphData = this.dashboards_data.sales.utm_graph;
        this.utmGraphData && this._renderUtmGraph();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Method used to generate Pie chart, depending on user selected UTM option(campaign, medium, source)
     *
     * @private
     */
    _renderUtmGraph: function() {
        var self = this;
        this.$(".utm_button_name").html(this.btnName); // change drop-down button name
        var utmDataType = this.utmType || 'campaign_id';
        var graphData = this.utmGraphData[utmDataType];
        if (graphData.length) {
            this.$(".o_utm_no_data_img").hide();
            this.$(".o_utm_data_graph").empty().show();
            var $canvas = $('<canvas/>');
            this.$(".o_utm_data_graph").append($canvas);
            var context = $canvas[0].getContext('2d');
            console.log(graphData);

            var data = [];
            var labels = [];
            graphData.forEach(function(pt) {
                data.push(pt.amount_total);
                labels.push(pt.utm_type);
            });
            var config = {
                type: 'pie',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: COLORS,
                    }]
                },
                options: {
                    tooltips: {
                        callbacks: {
                            label: function(tooltipItem, data) {
                                var label = data.labels[tooltipItem.index] || '';
                                if (label) {
                                    label += ': ';
                                }
                                var amount = data.datasets[0].data[tooltipItem.index];
                                amount = self.render_monetary_field(amount, self.data.currency);
                                label += amount;
                                return label;
                            }
                        }
                    },
                    legend: {display: false}
                }
            };
            new Chart(context, config);
        } else {
            this.$(".o_utm_no_data_img").show();
            this.$(".o_utm_data_graph").hide();
        }
    },

    //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * Onchange on UTM dropdown button, this method is called.
         *
         * @private
         */
    _onClickUtmButton: function(ev) {
        this.utmType = $(ev.currentTarget).attr('name');
        this.btnName = $(ev.currentTarget).text();
        this._renderUtmGraph();
    },

    on_product_template: function (ev) {
        ev.preventDefault();

        var product_tmpl_id = $(ev.currentTarget).data('productId');
        this.do_action({
            type: 'ir.actions.act_window',
            res_model: 'product.template',
            res_id: product_tmpl_id,
            views: [[false, 'form']],
            target: 'current',
        }, {
            on_reverse_breadcrumb: this.on_reverse_breadcrumb,
        });
    },
});
return WebsiteBackend;

});

```

## File: static\src\js\website_sale_form_editor.js

```javascript
odoo.define('website_sale.form', function (require) {
'use strict';

var FormEditorRegistry = require('website_form.form_editor_registry');

FormEditorRegistry.add('create_customer', {
    formFields: [{
        type: 'char',
        modelRequired: true,
        name: 'name',
        string: 'Your Name',
    }, {
        type: 'email',
        required: true,
        name: 'email',
        string: 'Your Email',
    }, {
        type: 'tel',
        name: 'phone',
        string: 'Phone Number',
    }, {
        type: 'char',
        name: 'company_name',
        string: 'Company Name',
    }],
});

});

```

## File: static\src\js\website_sale_payment.js

```javascript
odoo.define('website_sale.payment', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.WebsiteSalePayment = publicWidget.Widget.extend({
    selector: '#wrapwrap:has(#checkbox_cgv)',
    events: {
        'change #checkbox_cgv': '_onCGVCheckboxClick',
    },

    /**
     * @override
     */
    start: function () {
        this.$checkbox = this.$('#checkbox_cgv');
        this.$payButton = $('button#o_payment_form_pay');
        this.$checkbox.trigger('change');
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _adaptPayButton: function () {
        var disabledReasons = this.$payButton.data('disabled_reasons') || {};
        disabledReasons.cgv = !this.$checkbox.prop('checked');
        this.$payButton.data('disabled_reasons', disabledReasons);

        this.$payButton.prop('disabled', _.contains(disabledReasons, true));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onCGVCheckboxClick: function () {
        this._adaptPayButton();
    },
});
});

```

## File: static\src\js\website_sale_recently_viewed.js

```javascript
odoo.define('website_sale.recently_viewed', function (require) {

var concurrency = require('web.concurrency');
var config = require('web.config');
var core = require('web.core');
var publicWidget = require('web.public.widget');
var utils = require('web.utils');
var wSaleUtils = require('website_sale.utils');

var qweb = core.qweb;

publicWidget.registry.productsRecentlyViewedSnippet = publicWidget.Widget.extend({
    selector: '.s_wsale_products_recently_viewed',
    xmlDependencies: ['/website_sale/static/src/xml/website_sale_recently_viewed.xml'],
    disabledInEditableMode: false,
    read_events: {
        'click .js_add_cart': '_onAddToCart',
        'click .js_remove': '_onRemove',
    },

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        this._dp = new concurrency.DropPrevious();
        this.uniqueId = _.uniqueId('o_carousel_recently_viewed_products_');
        this._onResizeChange = _.debounce(this._addCarousel, 100);
    },
    /**
     * @override
     */
    start: function () {
        this._dp.add(this._fetch()).then(this._render.bind(this));
        $(window).resize(() => {
            this._onResizeChange();
        });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this._super(...arguments);
        this.$el.addClass('d-none');
        this.$el.find('.slider').html('');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _fetch: function () {
        return this._rpc({
            route: '/shop/products/recently_viewed',
        }).then(res => {
            var products = res['products'];

            // In edit mode, if the current visitor has no recently viewed
            // products, use demo data.
            if (this.editableMode && (!products || !products.length)) {
                return {
                    'products': [{
                        id: 0,
                        website_url: '#',
                        display_name: 'Product 1',
                        price: '$ <span class="oe_currency_value">750.00</span>',
                    }, {
                        id: 0,
                        website_url: '#',
                        display_name: 'Product 2',
                        price: '$ <span class="oe_currency_value">750.00</span>',
                    }, {
                        id: 0,
                        website_url: '#',
                        display_name: 'Product 3',
                        price: '$ <span class="oe_currency_value">750.00</span>',
                    }, {
                        id: 0,
                        website_url: '#',
                        display_name: 'Product 4',
                        price: '$ <span class="oe_currency_value">750.00</span>',
                    }],
                };
            }

            return res;
        });
    },
    /**
     * @private
     */
    _render: function (res) {
        var products = res['products'];
        var mobileProducts = [], webProducts = [], productsTemp = [];
        _.each(products, function (product) {
            if (productsTemp.length === 4) {
                webProducts.push(productsTemp);
                productsTemp = [];
            }
            productsTemp.push(product);
            mobileProducts.push([product]);
        });
        if (productsTemp.length) {
            webProducts.push(productsTemp);
        }

        this.mobileCarousel = $(qweb.render('website_sale.productsRecentlyViewed', {
            uniqueId: this.uniqueId,
            productFrame: 1,
            productsGroups: mobileProducts,
        }));
        this.webCarousel = $(qweb.render('website_sale.productsRecentlyViewed', {
            uniqueId: this.uniqueId,
            productFrame: 4,
            productsGroups: webProducts,
        }));
        this._addCarousel();
        this.$el.toggleClass('d-none', !(products && products.length));
    },
    /**
     * Add the right carousel depending on screen size.
     * @private
     */
    _addCarousel: function () {
        var carousel = config.device.size_class <= config.device.SIZES.SM ? this.mobileCarousel : this.webCarousel;
        this.$('.slider').html(carousel).css('display', ''); // Removing display is kept for compatibility (it was hidden before)
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Add product to cart and reload the carousel.
     * @private
     * @param {Event} ev
     */
    _onAddToCart: function (ev) {
        var self = this;
        var $card = $(ev.currentTarget).closest('.card');
        this._rpc({
            route: "/shop/cart/update_json",
            params: {
                product_id: $card.find('input[data-product-id]').data('product-id'),
                add_qty: 1
            },
        }).then(function (data) {
            wSaleUtils.updateCartNavBar(data);
            var $navButton = $('header .o_wsale_my_cart').first();
            var fetch = self._fetch();
            var animation = wSaleUtils.animateClone($navButton, $(ev.currentTarget).parents('.o_carousel_product_card'), 25, 40);
            Promise.all([fetch, animation]).then(function (values) {
                self._render(values[0]);
            });
        });
    },

    /**
     * Remove product from recently viewed products.
     * @private
     * @param {Event} ev
     */
    _onRemove: function (ev) {
        var self = this;
        var $card = $(ev.currentTarget).closest('.card');
        this._rpc({
            route: "/shop/products/recently_viewed_delete",
            params: {
                product_id: $card.find('input[data-product-id]').data('product-id'),
            },
        }).then(function (data) {
            self._render(data);
        });
    },
});

publicWidget.registry.productsRecentlyViewedUpdate = publicWidget.Widget.extend({
    selector: '#product_detail',
    events: {
        'change input.product_id[name="product_id"]': '_onProductChange',
    },
    debounceValue: 8000,

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        this._onProductChange = _.debounce(this._onProductChange, this.debounceValue);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Debounced method that wait some time before marking the product as viewed.
     * @private
     * @param {HTMLInputElement} $input
     */
    _updateProductView: function ($input) {
        var productId = parseInt($input.val());
        var cookieName = 'seen_product_id_' + productId;
        if (! parseInt(this.el.dataset.viewTrack, 10)) {
            return; // Is not tracked
        }
        if (utils.get_cookie(cookieName)) {
            return; // Already tracked in the last 30min
        }
        if ($(this.el).find('.js_product.css_not_available').length) {
            return; // Variant not possible
        }
        this._rpc({
            route: '/shop/products/recently_viewed_update',
            params: {
                product_id: productId,
            }
        }).then(function (res) {
            if (res && res.visitor_uuid) {
                utils.set_cookie('visitor_uuid', res.visitor_uuid);
            }
            utils.set_cookie(cookieName, productId, 30 * 60);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Call debounced method when product change to reset timer.
     * @private
     * @param {Event} ev
     */
    _onProductChange: function (ev) {
        this._updateProductView($(ev.currentTarget));
    },
});
});

```

## File: static\src\js\website_sale_tracking.js

```javascript
odoo.define('website_sale.tracking', function (require) {

var publicWidget = require('web.public.widget');

publicWidget.registry.websiteSaleTracking = publicWidget.Widget.extend({
    selector: '.oe_website_sale',
    events: {
        'click form[action="/shop/cart/update"] a.a-submit': '_onAddProductIntoCart',
        'click a[href="/shop/checkout"]': '_onCheckoutStart',
        'click div.oe_cart a[href^="/web?redirect"][href$="/shop/checkout"]': '_onCustomerSignin',
        'click form[action="/shop/confirm_order"] a.a-submit': '_onOrder',
        'click form[target="_self"] button[type=submit]': '_onOrderPayment',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;

        // Watching a product
        if (this.$el.is('#product_detail')) {
            var productID = this.$('input[name="product_id"]').attr('value');
            this._vpv('/stats/ecom/product_view/' + productID);
        }

        // ...
        if (this.$('div.oe_website_sale_tx_status').length) {
            this._trackGA('require', 'ecommerce');

            var orderID = this.$('div.oe_website_sale_tx_status').data('order-id');
            this._vpv('/stats/ecom/order_confirmed/' + orderID);

            this._rpc({
                route: '/shop/tracking_last_order/',
            }).then(function (o) {
                self._trackGA('ecommerce:clear');

                if (o.transaction && o.lines) {
                    self._trackGA('ecommerce:addTransaction', o.transaction);
                    _.forEach(o.lines, function (line) {
                        self._trackGA('ecommerce:addItem', line);
                    });
                }
                self._trackGA('ecommerce:send');
            });
        }

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _trackGA: function () {
        var websiteGA = window.ga || function () {};
        websiteGA.apply(this, arguments);
    },
    /**
     * @private
     */
    _vpv: function (page) { //virtual page view
        this._trackGA('send', 'pageview', {
          'page': page,
          'title': document.title,
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onAddProductIntoCart: function () {
        var productID = this.$('input[name="product_id"]').attr('value');
        this._vpv('/stats/ecom/product_add_to_cart/' + productID);
    },
    /**
     * @private
     */
    _onCheckoutStart: function () {
        this._vpv('/stats/ecom/customer_checkout');
    },
    /**
     * @private
     */
    _onCustomerSignin: function () {
        this._vpv('/stats/ecom/customer_signin');
    },
    /**
     * @private
     */
    _onOrder: function () {
        if ($('#top_menu [href="/web/login"]').length) {
            this._vpv('/stats/ecom/customer_signup');
        }
        this._vpv('/stats/ecom/order_checkout');
    },
    /**
     * @private
     */
    _onOrderPayment: function () {
        var method = $('#payment_method input[name=acquirer]:checked').nextAll('span:first').text();
        this._vpv('/stats/ecom/order_payment/' + method);
    },
});
});

```

## File: static\src\js\website_sale_utils.js

```javascript
odoo.define('website_sale.utils', function (require) {
'use strict';

function animateClone($cart, $elem, offsetTop, offsetLeft) {
    if (!$cart.length) {
        return Promise.resolve();
    }
    $cart.find('.o_animate_blink').addClass('o_red_highlight o_shadow_animation').delay(500).queue(function () {
        $(this).removeClass("o_shadow_animation").dequeue();
    }).delay(2000).queue(function () {
        $(this).removeClass("o_red_highlight").dequeue();
    });
    return new Promise(function (resolve, reject) {
        var $imgtodrag = $elem.find('img').eq(0);
        if ($imgtodrag.length) {
            var $imgclone = $imgtodrag.clone()
                .offset({
                    top: $imgtodrag.offset().top,
                    left: $imgtodrag.offset().left
                })
                .addClass('o_website_sale_animate')
                .appendTo(document.body)
                .animate({
                    top: $cart.offset().top + offsetTop,
                    left: $cart.offset().left + offsetLeft,
                    width: 75,
                    height: 75,
                }, 1000, 'easeInOutExpo');

            $imgclone.animate({
                width: 0,
                height: 0,
            }, function () {
                resolve();
                $(this).detach();
            });
        } else {
            resolve();
        }
    });
}

/**
 * Updates both navbar cart
 * @param {Object} data
 */
function updateCartNavBar(data) {
    $(".my_cart_quantity")
        .parents('li.o_wsale_my_cart').removeClass('d-none').end()
        .toggleClass('fa fa-warning', !data.cart_quantity)
        .attr('title', data.warning)
        .text(data.cart_quantity || '')
        .hide()
        .fadeIn(600);

    $(".js_cart_lines").first().before(data['website_sale.cart_lines']).end().remove();
    $(".js_cart_summary").first().before(data['website_sale.short_cart_summary']).end().remove();
}

/**
 * Displays `message` in an alert box at the top of the page if it's a
 * non-empty string.
 *
 * @param {string | null} message
 */
function showWarning(message) {
    if (!message) {
        return;
    }
    var $page = $('.oe_website_sale');
    var cart_alert = $page.children('#data_warning');
    if (!cart_alert.length) {
        cart_alert = $(
            '<div class="alert alert-danger alert-dismissible" role="alert" id="data_warning">' +
                '<button type="button" class="close" data-dismiss="alert">&times;</button> ' +
                '<span></span>' +
            '</div>').prependTo($page);
    }
    cart_alert.children('span:last-child').text(message);
}

return {
    animateClone: animateClone,
    updateCartNavBar: updateCartNavBar,
    showWarning: showWarning,
};
});

```

## File: static\src\js\website_sale_validate.js

```javascript
odoo.define('website_sale.validate', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var core = require('web.core');
var _t = core._t;

publicWidget.registry.websiteSaleValidate = publicWidget.Widget.extend({
    selector: 'div.oe_website_sale_tx_status[data-order-id]',

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        this._poll_nbr = 0;
        this._paymentTransationPollStatus();
        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _paymentTransationPollStatus: function () {
        var self = this;
        this._rpc({
            route: '/shop/payment/get_status/' + parseInt(this.$el.data('order-id')),
        }).then(function (result) {
            self._poll_nbr += 1;
            if (result.recall) {
                if (self._poll_nbr < 20) {
                    setTimeout(function () {
                        self._paymentTransationPollStatus();
                    }, Math.ceil(self._poll_nbr / 3) * 1000);
                } else {
                    var $message = $(result.message);
                    var $warning =  $("<i class='fa fa-warning' style='margin-right:10px;'>");
                    $warning.attr("title", _t("We are waiting for confirmation from the bank or the payment provider"));
                    $message.find('span:first').prepend($warning);
                    result.message = $message.html();
                }
            }
            self.$el.html(result.message);
        });
    },
});
});

```

## File: static\src\js\website_sale_video_field_preview.js

```javascript
odoo.define('website_sale.video_field_preview', function (require) {
"use strict";


var AbstractField = require('web.AbstractField');
var core = require('web.core');
var fieldRegistry = require('web.field_registry');

var QWeb = core.qweb;

/**
 * Displays preview of the video showcasing product.
 */
var FieldVideoPreview = AbstractField.extend({
    className: 'd-block o_field_video_preview',

    _render: function () {
        this.$el.html(QWeb.render('productVideo', {
            embedCode: this.value,
        }));
    },
});

fieldRegistry.add('video_preview', FieldVideoPreview);

return FieldVideoPreview;

});

```

## File: static\src\js\tours\website_sale_shop.js

```javascript
odoo.define("website_sale.tour_shop", function (require) {
    "use strict";

    var core = require("web.core");
    var _t = core._t;

    // return the steps, used for backend and frontend

    return [{
        trigger: "body:has(#o_new_content_menu_choices.o_hidden) #new-content-menu > a",
        content: _t("Let's create your first product."),
        extra_trigger: ".js_sale",
        consumeVisibleOnly: true,
        position: "bottom",
    }, {
        trigger: "a[data-action=new_product]",
        content: _t("Select <b>New Product</b> to create it and manage its properties to boost your sales."),
        position: "bottom",
    }, {
        trigger: ".modal-dialog #editor_new_product input[type=text]",
        content: _t("Enter a name for your new product"),
        position: "right",
    }, {
        trigger: ".modal-footer button.btn-primary.btn-continue",
        content: _t("Click on <em>Continue</em> to create the product."),
        position: "right",
    }, {
        trigger: ".product_price .oe_currency_value:visible",
        extra_trigger: ".editor_enable",
        content: _t("Edit the price of this product by clicking on the amount."),
        position: "bottom",
        run: "text 1.99",
    }, {
        trigger: "#wrap img.product_detail_img",
        extra_trigger: ".product_price .o_dirty .oe_currency_value:not(:containsExact(1.00))",
        content: _t("Double click here to set an image describing your product."),
        position: "top",
        run: function (actions) {
            actions.dblclick();
        },
    }, {
        trigger: ".o_select_media_dialog .o_upload_media_button",
        content: _t("Upload a file from your local library."),
        position: "bottom",
        run: function (actions) {
            actions.auto(".modal-footer .btn-secondary");
        },
    }, {
        trigger: "button.o_we_add_snippet_btn",
        auto: true,
    }, {
        trigger: "#snippet_structure .oe_snippet:eq(3) .oe_snippet_thumbnail",
        extra_trigger: "body:not(.modal-open)",
        content: _t("Drag this website block and drop it in your page."),
        position: "bottom",
        run: "drag_and_drop",
    }, {
        trigger: "button[data-action=save]",
        content: _t("Once you click on <b>Save</b>, your product is updated."),
        position: "bottom",
    }, {
        trigger: ".js_publish_management .js_publish_btn .css_publish",
        extra_trigger: "body:not(.editor_enable)",
        content: _t("Click on this button so your customers can see it."),
        position: "bottom",
    }, {
        trigger: ".o_main_navbar .o_menu_toggle, #oe_applications .dropdown-toggle",
        content: _t("Let's now take a look at your administration dashboard to get your eCommerce website ready in no time."),
        position: "bottom",
    }, { // backend
        trigger: '.o_apps > a[data-menu-xmlid="website.menu_website_configuration"], #oe_main_menu_navbar a[data-menu-xmlid="website.menu_website_configuration"]',
        content: _t("Open your website app here."),
        extra_trigger: ".o_apps,#oe_applications",
        position: "bottom",
    }];
});

```

## File: static\src\js\tours\website_sale_shop_backend.js

```javascript
odoo.define("website_sale.tour_shop_backend", function (require) {
"use strict";

var tour = require("web_tour.tour");
var steps = require("website_sale.tour_shop");
tour.register("shop", {url: "/shop"}, steps);

});

```

## File: static\src\js\tours\website_sale_shop_frontend.js

```javascript
odoo.define("website_sale.tour_shop_frontend", function (require) {
"use strict";

var tour = require("web_tour.tour");
var steps = require("website_sale.tour_shop");
tour.register("shop", {
    url: "/shop",
    sequence: 130,
}, steps);

});

```

## File: static\src\snippets\s_dynamic_snippet_products\000.js

```javascript
odoo.define('website_sale.s_dynamic_snippet_products', function (require) {
'use strict';

const config = require('web.config');
const core = require('web.core');
const publicWidget = require('web.public.widget');
const DynamicSnippetCarousel = require('website.s_dynamic_snippet_carousel');

const DynamicSnippetProducts = DynamicSnippetCarousel.extend({
    selector: '.s_dynamic_snippet_products',

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Method to be overridden in child components if additional configuration elements
     * are required in order to fetch data.
     * @override
     * @private
     */
    _isConfigComplete: function () {
        return this._super.apply(this, arguments) && this.$el.get(0).dataset.productCategoryId !== undefined;
    },
    /**
     *
     * @override
     * @private
     */
    _mustMessageWarningBeHidden: function() {
        const isInitialDrop = this.$el.get(0).dataset.templateKey === undefined;
        // This snippet has default values obtained after the initial start and render after drop.
        // Because of this there is an initial refresh happening right after.
        // We want to avoid showing the incomplete config message before this refresh.
        // Since the refreshed call will always happen with a defined templateKey,
        // if it is not set yet, we know it is the drop call and we can avoid showing the message.
        return isInitialDrop || this._super.apply(this, arguments);
    },
    /**
     * Method to be overridden in child components in order to provide a search
     * domain if needed.
     * @override
     * @private
     */
    _getSearchDomain: function () {
        const searchDomain = this._super.apply(this, arguments);
        const productCategoryId = parseInt(this.$el.get(0).dataset.productCategoryId);
        if (productCategoryId >= 0) {
            searchDomain.push(['public_categ_ids', 'child_of', productCategoryId]);
        }
        return searchDomain;
    },

});
publicWidget.registry.dynamic_snippet_products = DynamicSnippetProducts;

return DynamicSnippetProducts;
});

```

## File: static\src\snippets\s_dynamic_snippet_products\000.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_sale.s_dynamic_snippet.products" inherit_id="website.s_dynamic_snippet.carousel">
        <xpath expr="//div" position="attributes">
            <attribute name="data-filter-id" add=""/>
        </xpath>
    </t>
</templates>

```

## File: static\src\snippets\s_dynamic_snippet_products\options.js

```javascript
odoo.define('website_sale.s_dynamic_snippet_products_options', function (require) {
'use strict';

const options = require('web_editor.snippets.options');
const s_dynamic_snippet_carousel_options = require('website.s_dynamic_snippet_carousel_options');

const dynamicSnippetProductsOptions = s_dynamic_snippet_carousel_options.extend({

    /**
     *
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.productCategories = {};
    },
    /**
     *
     * @override
     */
    onBuilt: function () {
        this._super.apply(this, arguments);
        // TODO Remove in master.
        this.$target[0].dataset['snippet'] = 's_dynamic_snippet_products';
        this._rpc({
            route: '/website_sale/snippet/options_filters'
        }).then((data) => {
            if (data.length) {
                this.$target.get(0).dataset.filterId = data[0].id;
                this.$target.get(0).dataset.numberOfRecords = this.dynamicFilters[data[0].id].limit;
                this._refreshPublicWidgets();
                // Refresh is needed because default values are obtained after start()
            }
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     *
     * @override
     * @private
     */
    _computeWidgetVisibility: function (widgetName, params) {
        if (widgetName === 'filter_opt') {
            return false;
        }
        return this._super.apply(this, arguments);
    },
    /**
     * Fetches product categories.
     * @private
     * @returns {Promise}
     */
    _fetchProductCategories: function () {
        return this._rpc({
            model: 'product.public.category',
            method: 'search_read',
            kwargs: {
                domain: [],
                fields: ['id', 'name'],
            }
        });
    },
    /**
     *
     * @override
     * @private
     */
    _renderCustomXML: async function (uiFragment) {
        await this._super.apply(this, arguments);
        await this._renderProductCategorySelector(uiFragment);
    },
    /**
     * Renders the product categories option selector content into the provided uiFragment.
     * @private
     * @param {HTMLElement} uiFragment
     */
    _renderProductCategorySelector: async function (uiFragment) {
        const productCategories = await this._fetchProductCategories();
        for (let index in productCategories) {
            this.productCategories[productCategories[index].id] = productCategories[index];
        }
        const productCategoriesSelectorEl = uiFragment.querySelector('[data-name="product_category_opt"]');
        return this._renderSelectUserValueWidgetButtons(productCategoriesSelectorEl, this.productCategories);
    },
    /**
     * Sets default options values.
     * @override
     * @private
     */
    _setOptionsDefaultValues: function () {
        this._super.apply(this, arguments);
        const templateKeys = this.$el.find("we-select[data-attribute-name='templateKey'] we-selection-items we-button");
        if (templateKeys.length > 0) {
            this._setOptionValue('templateKey', templateKeys.attr('data-select-data-attribute'));
        }
        const productCategories = this.$el.find("we-select[data-attribute-name='productCategoryId'] we-selection-items we-button");
        if (productCategories.length > 0) {
            this._setOptionValue('productCategoryId', productCategories.attr('data-select-data-attribute'));
        }
    },

});

options.registry.dynamic_snippet_products = dynamicSnippetProductsOptions;

return dynamicSnippetProductsOptions;
});

```

## File: static\src\snippets\s_products_searchbar\000.js

```javascript
odoo.define('website_sale.s_products_searchbar', function (require) {
'use strict';

const concurrency = require('web.concurrency');
const publicWidget = require('web.public.widget');

const { qweb } = require('web.core');

/**
 * @todo maybe the custom autocomplete logic could be extract to be reusable
 */
publicWidget.registry.productsSearchBar = publicWidget.Widget.extend({
    selector: '.o_wsale_products_searchbar_form',
    xmlDependencies: ['/website_sale/static/src/xml/website_sale_utils.xml'],
    events: {
        'input .search-query': '_onInput',
        'focusout': '_onFocusOut',
        'keydown .search-query': '_onKeydown',
    },
    autocompleteMinWidth: 300,

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);

        this._dp = new concurrency.DropPrevious();

        this._onInput = _.debounce(this._onInput, 400);
        this._onFocusOut = _.debounce(this._onFocusOut, 100);
    },
    /**
     * @override
     */
    start: function () {
        this.$input = this.$('.search-query');

        this.order = this.$('.o_wsale_search_order_by').val();
        this.limit = parseInt(this.$input.data('limit'));
        this.displayDescription = !!this.$input.data('displayDescription');
        this.displayPrice = !!this.$input.data('displayPrice');
        this.displayImage = !!this.$input.data('displayImage');

        if (this.limit) {
            this.$input.attr('autocomplete', 'off');
        }

        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy() {
        this._super(...arguments);
        this._render(null);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _adaptToScrollingParent() {
        const bcr = this.el.getBoundingClientRect();
        this.$menu[0].style.setProperty('position', 'fixed', 'important');
        this.$menu[0].style.setProperty('top', `${bcr.bottom}px`, 'important');
        this.$menu[0].style.setProperty('left', `${bcr.left}px`, 'important');
        this.$menu[0].style.setProperty('max-width', `${bcr.width}px`, 'important');
        this.$menu[0].style.setProperty('max-height', `${document.body.clientHeight - bcr.bottom - 16}px`, 'important');
    },
    /**
     * @private
     */
    _fetch: function () {
        return this._rpc({
            route: '/shop/products/autocomplete',
            params: {
                'term': this.$input.val(),
                'options': {
                    'order': this.order,
                    'limit': this.limit,
                    'display_description': this.displayDescription,
                    'display_price': this.displayPrice,
                    'max_nb_chars': Math.round(Math.max(this.autocompleteMinWidth, parseInt(this.$el.width())) * 0.22),
                },
            },
        });
    },
    /**
     * @private
     */
    _render: function (res) {
        if (this._scrollingParentEl) {
            this._scrollingParentEl.removeEventListener('scroll', this._menuScrollAndResizeHandler);
            window.removeEventListener('resize', this._menuScrollAndResizeHandler);
            delete this._scrollingParentEl;
            delete this._menuScrollAndResizeHandler;
        }

        var $prevMenu = this.$menu;
        this.$el.toggleClass('dropdown show', !!res);
        if (res) {
            var products = res['products'];
            this.$menu = $(qweb.render('website_sale.productsSearchBar.autocomplete', {
                products: products,
                hasMoreProducts: products.length < res['products_count'],
                currency: res['currency'],
                widget: this,
            }));

            // TODO adapt directly in the template in master
            const mutedItemTextEl = this.$menu.find('span.dropdown-item-text.text-muted')[0];
            if (mutedItemTextEl) {
                const newItemTextEl = document.createElement('span');
                newItemTextEl.classList.add('dropdown-item-text');
                mutedItemTextEl.after(newItemTextEl);
                mutedItemTextEl.classList.remove('dropdown-item-text');
                newItemTextEl.appendChild(mutedItemTextEl);
            }

            this.$menu.css('min-width', this.autocompleteMinWidth);

            // Handle the case where the searchbar is in a mega menu by making
            // it position:fixed and forcing its size. Note: this could be the
            // default behavior or at least needed in more cases than the mega
            // menu only (all scrolling parents). But as a stable fix, it was
            // easier to fix that case only as a first step, especially since
            // this cannot generically work on all scrolling parent.
            const megaMenuEl = this.el.closest('.o_mega_menu');
            if (megaMenuEl) {
                const navbarEl = this.el.closest('.navbar');
                const navbarTogglerEl = navbarEl ? navbarEl.querySelector('.navbar-toggler') : null;
                if (navbarTogglerEl && navbarTogglerEl.clientWidth < 1) {
                    this._scrollingParentEl = megaMenuEl;
                    this._menuScrollAndResizeHandler = () => this._adaptToScrollingParent();
                    this._scrollingParentEl.addEventListener('scroll', this._menuScrollAndResizeHandler);
                    window.addEventListener('resize', this._menuScrollAndResizeHandler);

                    this._adaptToScrollingParent();
                }
            }

            this.$el.append(this.$menu);
        }

        if ($prevMenu) {
            $prevMenu.remove();
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onInput: function () {
        if (!this.limit) {
            return;
        }
        this._dp.add(this._fetch()).then(this._render.bind(this));
    },
    /**
     * @private
     */
    _onFocusOut: function () {
        if (!this.$el.has(document.activeElement).length) {
            this._render();
        }
    },
    /**
     * @private
     */
    _onKeydown: function (ev) {
        switch (ev.which) {
            case $.ui.keyCode.ESCAPE:
                this._render();
                break;
            case $.ui.keyCode.UP:
            case $.ui.keyCode.DOWN:
                ev.preventDefault();
                if (this.$menu) {
                    let $element = ev.which === $.ui.keyCode.UP ? this.$menu.children().last() : this.$menu.children().first();
                    $element.focus();
                }
                break;
        }
    },
});
});

```

## File: static\src\xml\website_sale.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="productVideo">
        <div class="embed-responsive embed-responsive-16by9 mt-2" t-if="embedCode">
            <t t-raw="embedCode"/>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_sale_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-extend="website.dashboard_header">
        <t t-jquery="div.o_dashboard_common" t-operation="append">
            <div class="col-12 o_box" t-if="widget.dashboards_data.sales.summary.order_unpaid_count || widget.dashboards_data.sales.summary.order_to_invoice_count || widget.dashboards_data.sales.summary.payment_to_capture_count || widget.dashboards_data.sales.summary.order_carts_abandoned_count">
                <div t-if="widget.dashboards_data.sales.summary.order_unpaid_count" class="o_inner_box o_dashboard_action" title="Confirm orders when you get paid." name="website_sale.action_unpaid_orders_ecommerce">
                    <div class="o_highlight"><t t-esc="widget.dashboards_data.sales.summary.order_unpaid_count"/></div>
                    Unpaid Orders
                </div>
                <div t-if="widget.dashboards_data.sales.summary.order_to_invoice_count" class="o_inner_box o_dashboard_action" title="Generate an invoice from orders ready for invoicing." name="website_sale.sale_order_action_to_invoice">
                    <div class="o_highlight"><t t-esc="widget.dashboards_data.sales.summary.order_to_invoice_count"/></div>
                    Orders to Invoice
                </div>
                <div t-if="widget.dashboards_data.sales.summary.payment_to_capture_count" class="o_inner_box o_dashboard_action" title="Capture order payments when the delivery is completed." name="website_sale.payment_transaction_action_payments_to_capture">
                    <div class="o_highlight"><t t-esc="widget.dashboards_data.sales.summary.payment_to_capture_count"/></div>
                    Payments to Capture
                </div>
                <div t-if="widget.dashboards_data.sales.summary.order_carts_abandoned_count" class="o_inner_box o_dashboard_action" title="Send a recovery email to visitors who haven't completed their order." name="website_sale.action_view_abandoned_tree">
                    <div class="o_highlight"><t t-esc="widget.dashboards_data.sales.summary.order_carts_abandoned_count"/></div>
                    Abandoned Carts
                </div>
            </div>
        </t>
    </t>

    <t t-extend="website.dashboard_content">
        <t t-jquery="div.o_website_dashboard_content" t-operation="prepend">
            <div t-if="widget.groups.sale_salesman" class="row o_dashboard_sales">
                <div class="col-12 row o_box">
                    <t t-if="widget.dashboards_data.sales.summary.order_count">
                        <h2 class="col-lg-7 col-12">
                            <t t-if="widget.date_range=='week'">
                                Sales Since Last Week
                            </t>
                            <t t-elif="widget.date_range=='month'">
                                Sales Since Last Month
                            </t>
                            <t t-elif="widget.date_range=='year'">
                                Sales Since Last Year
                            </t>
                            <t t-else="">Sales</t>
                        </h2>
                        <h4 class='col-lg-5 col-12'>AT A GLANCE</h4>
                        <div class="col-lg-7 col-12">
                            <div class="o_graph_sales" data-type="sales"/>
                        </div>
                        <div class="col-lg-5 col-12">
                            <t t-call="website_sale.products_table"/>
                        </div>
                    </t>
                    <t t-if="! widget.dashboards_data.sales.summary.order_count">
                        <t t-if="widget.date_range=='week'">
                            <h2>Sales Since Last Week</h2>
                        </t>
                        <t t-elif="widget.date_range=='month'">
                            <h2>Sales Since Last Month</h2>
                        </t>
                        <t t-elif="widget.date_range=='year'">
                            <h2>Sales Since Last Year</h2>
                        </t>
                        <t t-else=""><h2>Sales</h2></t>
                        <div class="col-lg-12 col-12">
                            <div class="o_demo_background">
                            </div>
                            <div class="o_demo_message">
                                <h3>There is no recent confirmed order.</h3>
                            </div>
                        </div>
                    </t>
                </div>
            </div>
        </t>
    </t>

    <t t-name="website_sale.products_table">
        <div class="row">
            <a href="#" class="col-md-4 o_dashboard_action" name="website_sale.sale_report_action_dashboard">
                <div class="o_link_enable" title="Orders">
                    <div class="o_highlight">
                        <t t-esc="widget.dashboards_data.sales.summary.order_count"/>
                    </div>
                    Orders
                </div>
            </a>
            <a href="#" class="col-md-4 o_dashboard_action" name="website_sale.sale_report_action_dashboard">
                <div class="o_link_enable" title="Untaxed Total Sold">
                    <div class="o_highlight">
                        <t t-esc="widget.render_monetary_field(widget.dashboards_data.sales.summary.total_sold, widget.data.currency)"/>
                    </div>
                    Sold
                </div>
            </a>
            <a href="#" class="col-md-4 o_dashboard_action" name="website_sale.sale_report_action_carts">
                <div class="o_link_enable o_invisible_border" title="Carts">
                    <div class="o_highlight"><t t-esc="widget.dashboards_data.sales.summary.order_carts_count"/></div>
                    Carts
                </div>
            </a>
            <div class="col-md-4 o_link_disable" title="Orders/Day">
                <div class="o_highlight"><t t-esc="widget.dashboards_data.sales.summary.order_per_day_ratio"/></div>
                Orders/Day
            </div>
            <div class="col-md-4 o_link_disable" title="Average Order">
                <div class="o_highlight"><t t-esc="widget.render_monetary_field(widget.dashboards_data.sales.summary.order_sold_ratio, widget.data.currency)"/></div>
                Average Order
            </div>
            <div class="col-md-4 o_link_disable o_invisible_border" title="Conversion">
                <div class="o_highlight"><t t-esc="widget.format_number(widget.dashboards_data.sales.summary.order_convertion_pctg, 'float', [3, 2], '%')"/></div>
                Conversion
            </div>
        </div>
        <div class="col-lg-12 col-12 o_top_margin">
            <div class="row">
                <div class="col-lg-12 col-12">
                    <h4>Best Sellers</h4>
                    <table class="table table-responsive table-hover">
                        <tr>
                            <th>Product</th>
                            <th>Quantity</th>
                            <th>Sold</th>
                        </tr>
                        <tr class="o_product_template" t-foreach="widget.dashboards_data.sales.best_sellers" t-as="product" t-att-data-product-id="product.id">
                            <td><t t-esc="product.name"/></td>
                            <td><t t-esc="product.qty"/></td>
                            <td><t t-esc="widget.render_monetary_field(product.sales, widget.data.currency)"/></td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
    </t>

    <t t-extend="website_sale.products_table">
        <t t-jquery=".o_top_margin .row .col-12" t-operation="attributes">
            <attribute name="class" value="col-lg-6 col-12" />
        </t>
        <t t-jquery=".o_top_margin .row" t-operation="append">
            <div class="col-lg-6 col-12 o_dashboard_utms">
                <div>
                    <h4 class="float-left">REVENUE BY</h4>
                    <t t-call="website_sale.LinkTrackersDropDown"/>
                </div>
                <div class="o_utm_no_data_img">
                    <img src="website_sale/static/src/img/website_sale_chart_demo.png" alt="There isn't any UTM tag detected in orders" class="utm_chart_image image-responsive mt8"/>
                </div>
                <div class="o_utm_data_graph"/>
            </div>
        </t>
    </t>

    <t t-name="website_sale.LinkTrackersDropDown">
        <div class="dropdown">
            <button class="btn btn-secondary dropdown-toggle utm_dropdown ml4" type="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="true"><span class="utm_button_name">Campaigns</span>
            </button>
            <div class="dropdown-menu" role="menu" aria-labelledby="utm_dropdown">
                <a name="campaign_id" class="dropdown-item js_utm_selector" role="menuitem">Campaigns</a>
                <a name="medium_id" class="dropdown-item js_utm_selector" role="menuitem">Medium</a>
                <a name="source_id" class="dropdown-item js_utm_selector" role="menuitem">Sources</a>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website_sale_recently_viewed.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <!-- Recently Viewed Products Carousel -->
    <t t-name="website_sale.productsRecentlyViewed">
        <div t-att-id="uniqueId" class="carousel slide o_not_editable" data-interval="false">
            <div class="carousel-inner">
                <t t-foreach="productsGroups" t-as="products">
                    <div t-attf-class="carousel-item #{!products_index and 'active' or ''}">
                        <div class="row">
                            <t t-foreach="products" t-as="product">
                                <div t-attf-class="o_carousel_product_card_wrap col-md-#{12 / productFrame}">
                                    <div class="o_carousel_product_card card h-100">
                                        <input type="hidden" name="product-id" t-att-data-product-id="product.id"/>
                                        <a class="o_carousel_product_img_link" t-att-href="product.website_url">
                                            <img class="o_carousel_product_card_img_top card-img-top" t-attf-src="/web/image/product.product/#{product.id}#{productFrame == 1 ? '/image_256' : '/image_512'}" t-att-alt="product.display_name"/>
                                        </a>
                                        <i class="fa fa-trash o_carousel_product_remove js_remove"></i>
                                        <div class="o_carousel_product_card_body card-body border-top">
                                            <a t-att-href="product.website_url" class="text-decoration-none">
                                                <h6 class="card-title mb-0 text-truncate" t-raw="product.display_name"/>
                                            </a>
                                            <t t-if="product.rating" t-raw="product.rating"/>
                                        </div>
                                        <div class="o_carousel_product_card_footer card-footer d-flex align-items-center">
                                            <div class="d-block font-weight-bold" t-raw="product.price"/>
                                            <button type="button" role="button" class="btn btn-primary js_add_cart ml-auto" title="Add to Cart">
                                                <i class="fa fa-fw fa-shopping-cart"/>
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </t>
                        </div>
                    </div>
                </t>
            </div>
            <t t-if='productsGroups.length > 1'>
                <a class="o_carousel_product_control carousel-control-prev" t-att-href="'#' + uniqueId" role="button" data-slide="prev">
                    <span class="carousel-control-prev-icon"></span>
                    <span class="sr-only">Previous</span>
                </a>
                <a class="o_carousel_product_control carousel-control-next" t-att-href="'#' + uniqueId" role="button" data-slide="next">
                    <span class="carousel-control-next-icon"></span>
                    <span class="sr-only">Next</span>
                </a>
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website_sale_utils.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<!-- Products Search Bar autocomplete item -->
<div t-name="website_sale.productsSearchBar.autocomplete"
    class="dropdown-menu show w-100">
    <t t-if="!products.length">
        <!-- TODO adapt in master, this is patched in JS so that text-muted -->
        <!-- is not on the same element as dropdown-item-text -->
        <span class="dropdown-item-text text-muted">No results found. Please try another search.</span>
    </t>
    <a t-foreach="products" t-as="product"
        t-att-href="product['website_url']" class="dropdown-item p-2 text-wrap">
        <div class="media align-items-center o_search_product_item">
            <t t-if="widget.displayImage">
                <img t-attf-src="/web/image/product.template/#{product['id']}/image_128"
                    class="flex-shrink-0 o_image_64_contain"/>
            </t>
            <div class="media-body px-3">
                <t t-set="description" t-value="widget.displayDescription and product['description_sale']"/>
                <h6 t-attf-class="font-weight-bold #{description ? '' : 'mb-0'}" t-esc="product['name']"/>
                <p t-if="description" class="mb-0" t-esc="description"/>
            </div>
            <div t-if="widget.displayPrice" class="flex-shrink-0">
                <t t-if="product['has_discounted_price']">
                    <span class="text-danger text-nowrap" style="text-decoration: line-through;">
                        <t t-raw="product['list_price']"/>
                    </span>
                    <br/>
                </t>
                <b class="text-nowrap">
                    <t t-raw="product['price']"/>
                </b>
            </div>
        </div>
    </a>
    <t t-if="hasMoreProducts">
        <button type="submit" class="dropdown-item text-center text-primary">All results</button>
    </t>
</div>

<!-- Products Search Bar autocomplete item -->
<we-button t-name="website_sale.ribbonSelectItem" t-att-data-set-ribbon="ribbon.id">
    <t t-raw="ribbon.html"/>
    <span t-attf-class="fa fa-#{isTag ? 'tag' : 'bookmark'} ml-auto"></span>
    <span t-attf-class="fa fa-arrow-#{isLeft ? 'left' : 'right'} ml-1"></span>
    <span t-attf-class="o_wsale_color_preview #{colorClasses} ml-1" t-attf-style="background-color: #{ribbon.bg_color}"></span>
    <span t-attf-class="o_wsale_color_preview #{colorClasses} ml-1" t-attf-style="background-color: #{textColor} !important;"></span>
</we-button>

</templates>

```

## File: views\account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_invoices_ecommerce" model="ir.actions.act_window">
        <field name="name">Invoices</field>
        <field name="res_model">account.move</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('team_id.website_ids', '!=', False)]</field>
        <field name="view_id" ref="account.view_invoice_tree"/>
        <field name="context">{'move_type':'out_invoice'}</field>
        <field name="search_view_id" ref="account.view_account_invoice_filter"/>
    </record>

    <record id="website_product_pricelist3" model="ir.actions.act_window">
        <field name="name">Pricelists</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.pricelist</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('website_id', '!=', False)]</field>
        <field name="search_view_id" ref="product.product_pricelist_view_search" />
        <field name="context">{"default_base":'list_price'}</field>
    </record>

    <record id="account_move_view_form" model="ir.ui.view">
        <field name="name">account.move.form.inherit.website_sale</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='sale_info_group']" position="inside">
                <field name="website_id" groups="website.group_multi_website" attrs="{'invisible': [('website_id', '=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="crm_team_salesteams_view_kanban_inherit_website_sale" model="ir.ui.view"> 
        <field name="name">crm.team.kanban</field>
        <field name="model">crm.team</field>
        <field name="inherit_id" ref="sales_team.crm_team_salesteams_view_kanban"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//t[@name='third_options']" position="after">
                    <div class="row" t-if="record.abandoned_carts_count.raw_value">
                        <div class="col-8">
                            <div>
                                <a name="get_abandoned_carts" type="object">
                                    <field name="abandoned_carts_count"/>
                                    Abandoned Carts to Recover
                                </a>
                            </div>
                        </div>
                        <div class="col-4 text-right">
                            <field name="abandoned_carts_amount" widget="monetary"/>
                        </div>
                    </div>
                </xpath>
            </data>
        </field>
    </record>

</odoo>

```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form.inherit.website.sale.order</field>
        <field name="model">digest.digest</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpi_sales']" position="attributes">
                <attribute name="string">Sales</attribute>
                <attribute name="groups">sales_team.group_sale_salesman_all_leads</attribute>
            </xpath>
            <xpath expr="//group[@name='kpi_sales']" position="inside">
                <field name="kpi_website_sale_total"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\onboarding_views.xml

```xml
<odoo>
    <!-- PAYMENT ACQUIRER -->
    <template id="website_sale_onboarding_payment_acquirer_step" primary="True"
              inherit_id="payment.onboarding_payment_acquirer_step">
        <xpath expr="//t[@t-set='method']" position="replace">
            <t t-set="method" t-value="'action_open_website_sale_onboarding_payment_acquirer'" />
        </xpath>
        <xpath expr="//t[@t-set='state']" position="replace">
            <t t-set="state" t-value="state.get('website_sale_onboarding_payment_acquirer_state')" />
        </xpath>
    </template>

    <record id="action_open_website_sale_onboarding_payment_acquirer_wizard" model="ir.actions.act_window">
        <field name="name">Choose a payment method</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">website.sale.payment.acquirer.onboarding.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="payment.payment_acquirer_onboarding_wizard_form" />
        <field name="target">new</field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_search_view_website" model="ir.ui.view">
        <field name="name">product.template.search.published</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_search_view"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='consumable']" position="after">
                <separator/>
                <filter string="Published" name="published" domain="[('is_published', '=', True)]"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="product_product_website_tree_view">
        <field name="name">product.product.website.tree</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_product_tree_view"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="website_id" groups="website.group_multi_website" optional="show"/>
                <field name="is_published" string="Is Published" optional="hide"/>
            </field>
        </field>
    </record>

    <!-- We want website_id to be shown outside of website module like other models -->
    <record model="ir.ui.view" id="product_template_view_tree">
        <field name="name">product.template.view.tree.inherit.website_sale</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_tree_view"/>
        <field name="arch" type="xml">
            <field name="default_code" position="after">
                <field name="website_id" groups="website.group_multi_website" optional="hide"/>
            </field>
        </field>
    </record>

    <!-- only website module template view should use the website_sequence -->
    <record model="ir.ui.view" id="product_template_view_tree_website_sale">
        <field name="name">product.template.view.tree.website_sale</field>
        <field name="mode">primary</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="website_sale.product_template_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="/tree" position="attributes">
              <attribute name="default_order">website_sequence</attribute>
            </xpath>
            <field name="sequence" position="replace">
                <field name="website_sequence" widget="handle"/>
            </field>
            <field name="website_id" position="after">
                <field name="is_published" string="Is Published" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="product_template_action_website" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">kanban,tree,form,activity</field>
        <field name="view_id" ref="product_template_view_tree_website_sale"/>
        <field name="search_view_id" ref="product_template_search_view_website"/>
        <field name="context">{'search_default_published': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new product
            </p><p>
                A product can be either a physical product or a service that you sell to your customers.
            </p>
        </field>
    </record>

    <record model="ir.ui.view" id="product_template_form_view_invoice_policy">
        <field name="name">product.template.invoice.policy</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="sale.product_template_form_view_invoice_policy"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='invoicing']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="product_template_form_view">
        <field name="name">product.template.product.website.form</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <!-- add state field in header -->
            <div name="button_box" position="inside">
                <field name="is_published" widget="website_redirect_button" attrs="{'invisible': [('sale_ok','=',False)]}"/>
            </div>
            <xpath expr="//page[@name='sales']" position="after">
                <page name="shop" string="eCommerce" attrs="{'invisible': [('sale_ok','=',False)]}">
                    <group name="shop">
                        <group string="Shop">
                            <field name="website_url" invisible="1"/>
                            <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                            <field name="website_sequence" groups="base.group_no_one"/>
                            <field name="public_categ_ids" widget="many2many_tags" string="Categories"/>
                            <field name="alternative_product_ids" widget="many2many_tags" domain="[('id', '!=', active_id), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"/>
                            <field name="accessory_product_ids" widget="many2many_tags"/>
                            <field name="website_ribbon_id" groups="base.group_no_one" options="{'no_quick_create': True}"/>
                        </group>
                    </group>
                    <group name="product_template_images" string="Extra Product Media">
                        <field name="product_template_image_ids" class="o_website_sale_image_list" context="{'default_name': name}" mode="kanban" options="{'create_text':'Add a Media'}" nolabel="1"/>
                    </group>
                </page>
            </xpath>
        </field>
    </record>

    <record id="product_product_view_form_easy_inherit_website_sale" model="ir.ui.view">
        <field name="name">product.product.view.form.easy.inherit.website_sale</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_variant_easy_edit_view"/>
        <field name="arch" type="xml">
            <sheet position="inside">
                <group name="product_variant_images" string="Extra Variant Media">
                    <field name="product_variant_image_ids" class="o_website_sale_image_list" context="{'default_name': name}" mode="kanban" options="{'create_text':'Add a Media'}" nolabel="1"/>
                </group>
            </sheet>
        </field>
    </record>

    <!-- Product Public Categories -->
    <record id="product_public_category_form_view" model="ir.ui.view">
        <field name="name">product.public.category.form</field>
        <field name="model">product.public.category</field>
        <field name="arch" type="xml">
            <form string="Website Public Categories">
                <sheet>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_left">
                        <group>
                            <field name="name"/>
                            <field name="parent_id"/>
                            <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                            <field name="sequence" groups="base.group_no_one"/>
                        </group>
                    </div>
                </sheet>
            </form>
        </field>
    </record>
    <record id="product_public_category_tree_view" model="ir.ui.view">
        <field name="name">product.public.category.tree</field>
        <field name="model">product.public.category</field>
        <field name="field_parent" eval="False"/>
        <field name="arch" type="xml">
            <tree string="Product Public Categories">
                <field name="sequence" widget="handle"/>
                <field name="display_name"/>
                <field name="website_id" groups="website.group_multi_website"/>
            </tree>
        </field>
    </record>
    <record id="product_public_category_action" model="ir.actions.act_window">
        <field name="name">eCommerce Categories</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.public.category</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" eval="False"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Define a new category
          </p><p>
            Categories are used to browse your products through the
            touchscreen interface.
          </p>
        </field>
    </record>

    <record id="product_ribbon_view_tree" model="ir.ui.view">
        <field name="name">product.ribbon.tree</field>
        <field name="model">product.ribbon</field>
        <field name="arch" type="xml">
            <tree string="Products Ribbon">
                <field name="html" string="Name"/>
            </tree>
        </field>
    </record>

    <record id="website_sale_pricelist_form_view" model="ir.ui.view">
        <field name="name">website_sale.pricelist.form</field>
        <field name="inherit_id" ref="product.product_pricelist_view" />
        <field name="model">product.pricelist</field>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='pricelist_availability']" position="after">
                <group name="pricelist_website" string="Website">
                    <field name="website_id" options="{'no_create': True}"/>
                    <field name="selectable"/>
                    <field name="code"/>
                </group>
            </xpath>
          </field>
    </record>

    <record id="website_sale_pricelist_tree_view" model="ir.ui.view">
        <field name="name">product.pricelist.tree.inherit.product</field>
        <field name="model">product.pricelist</field>
        <field name="inherit_id" ref="product.product_pricelist_view_tree"/>
        <field name="arch" type="xml">
            <field name="currency_id" position="after">
                <field name="selectable" />
                <field name="website_id" groups="website.group_multi_website"/>
            </field>
        </field>
    </record>

    <!-- This view should only be used from the product o2m because the required field product_tmpl_id has to be automatically set. -->
    <record id="view_product_image_form" model="ir.ui.view">
        <field name="name">product.image.view.form</field>
        <field name="model">product.image</field>
        <field name="arch" type="xml">
            <form string="Product Images">
                <field name="sequence" invisible="1"/>
                <div class="row o_website_sale_image_modal">
                    <div class="col-md-6 col-xl-5">
                        <label for="name" string="Image Name"/>
                        <h2><field name="name" placeholder="Image Name"/></h2>
                        <label for="video_url" string="Video URL"/><br/>
                        <field name="video_url"/><br/>
                    </div>
                    <div class="col-md-6 col-xl-7 text-center o_website_sale_image_modal_container">
                        <div class="row">
                            <div class="col">
                                <field name="image_1920" widget="image"/>
                            </div>
                            <div class="col" attrs="{'invisible': [('video_url', 'in', ['', False])]}">
                                <div class="o_video_container p-2">
                                    <span>Video Preview</span>
                                    <field name="embed_code" class="mt-2" widget="video_preview"/>
                                    <h4 class="o_invalid_warning text-muted text-center" attrs="{'invisible': [('embed_code', '!=', False)]}">
                                        Please enter a valid Video URL.
                                    </h4>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </field>
    </record>
    <record id="product_image_view_kanban" model="ir.ui.view">
        <field name="name">product.image.view.kanban</field>
        <field name="model">product.image</field>
        <field name="arch" type="xml">
            <kanban string="Product Images" default_order="sequence">
                <field name="id"/>
                <field name="name"/>
                <field name="image_1920"/>
                <field name="sequence" widget="handle"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="card oe_kanban_global_click p-0">
                            <div class="o_squared_image">
                                <img class="card-img-top" t-att-src="kanban_image('product.image', 'image_1920', record.id.raw_value)" t-att-alt="record.name.value"/>
                            </div>
                            <div class="card-body p-0">
                                <h4 class="card-title p-2 m-0 bg-200">
                                    <small><field name="name"/></small>
                                </h4>
                            </div>
                            <!-- below 100 Kb: good -->
                            <t t-if="record.image_1920.raw_value.length &lt; 100*1000">
                                <t t-set="size_status" t-value="'badge-success'"/>
                                <t t-set="message">Acceptable file size</t>
                            </t>
                            <!-- below 1000 Kb: decent -->
                            <t t-elif="record.image_1920.raw_value.length &lt; 1000*1000">
                                <t t-set="size_status" t-value="'badge-warning'" />
                                <t t-set="message">Huge file size. The image should be optimized/reduced.</t>
                            </t>
                            <!-- above 1000 Kb: bad -->
                            <t t-else="1">
                                <t t-set="size_status" t-value="'badge-danger'"/>
                                <t t-set="message">Optimization required! Reduce the image size or increase your compression settings.</t>
                            </t>
                            <span t-attf-class="badge #{size_status} o_product_image_size" t-esc="record.image_1920.value" t-att-title="message"/>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="webmaster_settings" position="after">
                <h2>Products</h2>
                <div class="row mt16 o_settings_container" id="sale_product_catalog_settings">
                    <div class="col-12 col-lg-6 o_setting_box" id="product_attributes_setting">
                        <div class="o_setting_left_pane">
                            <field name="group_product_variant"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="group_product_variant"/>
                            <div class="text-muted">
                                Sell variants of a product using attributes (size, color, etc.)
                            </div>
                            <div class="content-group" attrs="{'invisible': [('group_product_variant', '=', False)]}">
                                <div class="mt8">
                                    <button type="action" name="%(product.attribute_action)d" string="Attributes" class="btn-link" icon="fa-arrow-right"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="prompt_optional_products">
                        <div class="o_setting_left_pane">
                            <field name="module_sale_product_configurator" string="Optional Products"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="module_sale_product_configurator" string="Optional Products"/>
                            <div class="text-muted">
                                Display a prompt with optional products when adding to cart
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="digital_content_setting" title="Provide customers with product-specific links or downloadable content in the confirmation page of the checkout process if the payment gets through. To do so, attach some files to a product using the new Files button and publish them.">
                        <div class="o_setting_left_pane">
                            <field name="module_website_sale_digital"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="module_website_sale_digital"/>
                            <div class="text-muted">
                                Sell content to download or URL links
                            </div>
                        </div>
                    </div>

                    <div class="col-12 col-lg-6 o_setting_box" id="wishlist_option_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_website_sale_wishlist"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="module_website_sale_wishlist"/>
                            <div class="text-muted">
                                Let returning shoppers save products in a wishlist
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="comparator_option_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_website_sale_comparison"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="module_website_sale_comparison"/>
                            <div class="text-muted">
                                Allow shoppers to compare products based on their attributes
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="product_availability_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_website_sale_stock"/>
                        </div>
                        <div class="o_setting_right_pane" name="stock_inventory_availability">
                            <label for="module_website_sale_stock"/>
                            <div class="text-muted">
                                Manage availability of products
                            </div>
                        </div>
                    </div>
                </div>

                <h2>Pricing</h2>
                <div class="row mt16 o_settings_container" id="sale_pricing_settings">
                    <div class="col-12 col-lg-6 o_setting_box" id="website_tax_inclusion_setting">
                        <div class="o_setting_right_pane">
                            <label string="Product Prices" for="show_line_subtotals_tax_selection"/>
                            <div class="text-muted">
                                Product prices displaying in web catalog
                            </div>
                            <div class="content-group">
                                <div class="mt16">
                                    <field name="show_line_subtotals_tax_selection" class="o_light_label" widget="radio"/>
                                    <field name="group_show_line_subtotals_tax_included" invisible="1"/>
                                    <field name="group_show_line_subtotals_tax_excluded" invisible="1"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="pricelists_setting"  title="With the first mode you can set several prices in the product config form (from Sales tab). With the second one, you set prices and computation rules from Pricelists.">
                        <div class="o_setting_left_pane">
                            <field name="group_product_pricelist"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="group_product_pricelist"/>
                            <div class="text-muted">
                                Apply specific prices per country, discounts, etc.
                            </div>
                            <div class="content-group mt16" attrs="{'invisible': [('group_product_pricelist', '=', False)]}">
                                <field name="group_sale_pricelist" invisible="1"/>
                                <field name="group_product_pricelist" invisible="1"/>
                                <field name="product_pricelist_setting" class="o_light_label" widget="radio"/>
                            </div>
                            <div class="mt8" attrs="{'invisible': [('group_product_pricelist', '=', False)]}">
                                <button type="action" name="%(product.product_pricelist_action2)d" string="Pricelists" class="btn-link" icon="fa-arrow-right"/>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="discount_setting" title="Apply manual discounts on sales order lines or display discounts computed from pricelists (option to activate in the pricelist configuration).">
                        <div class="o_setting_left_pane">
                            <field name="group_discount_per_so_line"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="group_discount_per_so_line"/>
                            <div class="text-muted">
                                Grant discounts on sales order lines
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="multicurrencies_setting" title="This adds the choice of a currency on pricelists.">
                        <div class="o_setting_left_pane">
                            <field name="group_multi_currency"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="group_multi_currency"/>
                            <div class="text-muted">
                                Sell in several currencies
                            </div>
                            <div class="content-group" attrs="{'invisible': [('group_multi_currency', '=', False)]}">
                                <div class="mt8">
                                    <button type="action" name="%(base.action_currency_form)d" string="Currencies" class="btn-link" icon="fa-arrow-right"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box"
                        id="promotion_coupon_programs"
                        title="Boost your sales with two kinds of discount programs: promotions and coupon codes. Specific conditions can be set (products, customers, minimum purchase amount, period). Rewards can be discounts (% or amount) or free products.">
                        <div class="o_setting_left_pane">
                            <field name="module_sale_coupon" />
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="module_sale_coupon"/>
                            <div class="text-muted" id="website_sale_coupon">
                                Manage promotion &amp; coupon programs
                            </div>
                        </div>
                    </div>
                </div>

                <h2>Shipping</h2>
                <div class="row mt16 o_settings_container" id="sale_shipping_settings">
                    <div class="col-12 col-lg-6 o_setting_box" id="shipping_address_setting">
                        <div class="o_setting_left_pane">
                            <field name="group_delivery_invoice_address"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="group_delivery_invoice_address"/>
                            <div class="text-muted">
                                Let the customer enter a shipping address
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="delivery_method_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_website_sale_delivery"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="Shipping Costs" for="module_website_sale_delivery"/>
                            <div class="text-muted" id="msg_delivery_method_setting">
                                Compute shipping costs on orders
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="ups_provider_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_delivery_ups" widget="upgrade_boolean"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="UPS" for="module_delivery_ups"/>
                            <div class="text-muted" id="website_delivery_ups">
                                Compute shipping costs and ship with UPS
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="shipping_provider_dhl_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_delivery_dhl" widget="upgrade_boolean"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="DHL Express Connector" for="module_delivery_dhl"/>
                            <div class="text-muted" id="website_delivery_dhl">
                                Compute shipping costs and ship with DHL
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="shipping_provider_fedex_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_delivery_fedex" widget="upgrade_boolean"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="FedEx" for="module_delivery_fedex"/>
                            <div class="text-muted" id="website_delivery_fedex">
                                Compute shipping costs and ship with FedEx
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="shipping_provider_usps_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_delivery_usps" widget="upgrade_boolean"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="USPS" for="module_delivery_usps"/>
                            <div class="text-muted" id="website_delivery_usps">
                                Compute shipping costs and ship with USPS
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="shipping_provider_bpost_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_delivery_bpost" widget="upgrade_boolean"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="bpost" for="module_delivery_bpost"/>
                            <div class="text-muted" id="website_delivery_bpost">
                                Compute shipping costs and ship with bpost
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="shipping_provider_easypost_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_delivery_easypost" widget="upgrade_boolean"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label string="Easypost" for="module_delivery_easypost"/>
                            <div class="text-muted" id="website_delivery_easypost">
                                Compute shipping cost and ship with Easypost
                            </div>
                        </div>
                    </div>
                </div>

                <field name='module_account' invisible="1"/>
                <div attrs="{'invisible': [('module_account', '=', False)]}">
                    <h2>Invoicing</h2>
                    <div class="row mt16 o_settings_container" id="sale_invoicing_settings">
                       <div class="col-12 col-lg-6 o_setting_box" id="invoicing_policy_setting" title="The mode selected here applies as invoicing policy of any new product created but not of products already existing.">
                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Invoicing Policy</span>
                                <div class="text-muted">
                                    Issue invoices to customers
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="default_invoice_policy" class="o_light_label" widget="radio"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="automatic_invoice_generation"
                            attrs="{'invisible': [('default_invoice_policy', '=', 'delivery')]}">
                            <div class="o_setting_left_pane">
                                <field name="automatic_invoice" nolabel="1"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="automatic_invoice"/>
                                <div class="text-muted">
                                    Generate the invoice automatically when the online payment is confirmed
                                </div>
                                <div  attrs="{'invisible': [('automatic_invoice','=',False)]}">
                                    <label for="template_id" class="o_light_label"/>
                                    <field name="template_id" class="oe_inline"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <h2>Orders Followup</h2>
                <div class="row mt16 o_settings_container" id="sale_checkout_settings">
                    <div class="col-12 col-lg-6 o_setting_box" id="checkout_assignation_setting">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Assignment</span>
                            <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                            <div class="text-muted">
                                Assignment of online orders
                            </div>
                            <div class="content-group">
                                <div class="row mt16">
                                    <label class="o_light_label col-lg-3" string="Sales Team" for="salesteam_id"/>
                                    <field name="salesteam_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s" />
                                </div>
                                <div class="row">
                                    <label class="o_light_label col-lg-3" for="salesperson_id"/>
                                    <field name="salesperson_id"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="confirmation_email_setting">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Confirmation Email</span>
                            <div class="text-muted">
                                Email sent to the customer after the checkout
                            </div>
                            <div class="row mt16">
                                <label for="confirmation_template_id" string="Email Template" class="col-lg-4 o_light_label"/>
                                <field name="confirmation_template_id" class="oe_inline"/>
                            </div>
                        </div>
                    </div>
                    <div class="col-xs-12 col-lg-6 o_setting_box" id="abandoned_carts_setting" title="Abandoned carts are all carts left unconfirmed by website visitors. You can find them in *Website > Orders > Abandoned Carts*. From there you can send recovery emails to visitors who entered their contact details.">
                        <div class="o_setting_left_pane"/>
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Abandoned Carts</span>
                            <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                            <div class="text-muted">
                                Send a recovery email when a cart is abandoned
                            </div>
                            <div class="content-group" title="This email template is suggested by default when you send a recovery email.">
                                <div class="row mt16">
                                    <label for="cart_recovery_mail_template" string="Email Template" class="col-lg-4 o_light_label"/>
                                    <field name="cart_recovery_mail_template" class="oe_inline"/>
                                </div>
                            </div>
                            <div class="content-group" title="Carts are flagged as abandoned after this delay.">
                                <div class="row mt16">
                                    <div class="col-12">
                                      <label for="cart_abandoned_delay" string="Cart is abandoned after" class="o_light_label"/>
                                      <field class="col-2" name="cart_abandoned_delay"/> hours.
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_sales_order_filter_ecommerce" model="ir.ui.view">
        <field name="name">sale.order.ecommerce.search.view</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_sales_order_filter"/>
        <field name="mode">primary</field> 
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='my_sale_orders_filter']" position="before">
                <filter string="Confirmed" name="order_confirmed" domain="[('state', 'in', ('sale', 'done'))]"/>
                <filter string="Unpaid" name="order_unpaid" domain="[('state', '=', 'sent'), ('website_id', '!=', False)]"/>
                <filter string="Abandoned" name="order_abandoned" domain="[('is_abandoned_cart', '=', True)]"/>
                <separator/>
                <filter string="Order Date" name="order_date" date="date_order"/>
                <separator/>
                <filter string="From Website" name="from_website" domain="[('website_id', '!=', False)]"/>
                <separator/>
                <!-- Dashboard filter - used by context -->
                <filter string="Last Week" invisible="1" name="week" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=7)).strftime('%%Y-%%m-%%d'))]"/>
                <filter string="Last Month" invisible="1" name="month" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=30)).strftime('%%Y-%%m-%%d'))]"/>
                <filter string="Last Year" invisible="1"  name="year" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=365)).strftime('%%Y-%%m-%%d'))]"/>
            </xpath>
        </field>
    </record>

    <record id="view_sales_order_filter_ecommerce_unpaid" model="ir.ui.view">
        <field name="name">sale.order.ecommerce.search.unpaid.view</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_sales_order_filter"/>
        <field name="mode">primary</field> 
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='my_sale_orders_filter']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_sale_orders_filter']" position="before">
                <filter string="Order Date" name="order_date" date="date_order"/>
                <separator/>
            </xpath>
        </field>
    </record>

    <record id="sale_order_view_form_cart_recovery" model="ir.ui.view">
        <field name="name">sale.order.form.abandoned.cart</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='team_id']" position="after">
                <field name="is_abandoned_cart" invisible="1"/>
                <field name="cart_recovery_email_sent" invisible="1"/>
            </xpath>
            <xpath expr="//button[@name='action_quotation_send' and @states='draft']" position="attributes">
                <!-- The '| and the '&amp' opertors are necessary because draft state of the parent concatenate the domain -->
                <attribute name="attrs">{'invisible': ['|','&amp;',('is_abandoned_cart', '=', True), ('cart_recovery_email_sent', '=', False)]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_quotation_send']" position="after">
                <button name="action_recovery_email_send"
                    string="Send a Recovery Email"
                    type="object"
                    class="btn-primary"
                    attrs="{'invisible': ['|', ('is_abandoned_cart', '=', False), ('cart_recovery_email_sent', '=', True)]}"/>
            </xpath>
        </field>
    </record>

    <record id="action_orders_ecommerce" model="ir.actions.act_window">
        <field name="name">Orders</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">tree,form,kanban,activity</field>
        <field name="domain">[]</field>
        <field name="context">{'show_sale': True, 'search_default_order_confirmed': 1, 'search_default_from_website': 1}</field>
        <field name="search_view_id" ref="view_sales_order_filter_ecommerce"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There is no confirmed order from the website
            </p>
        </field>
    </record>

    <!-- Dashboard Action -->
    <record id="action_unpaid_orders_ecommerce" model="ir.actions.act_window">
        <field name="name">Unpaid Orders</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">tree,form,kanban,activity</field>
        <field name="domain">[('state', '=', 'sent'), ('website_id', '!=', False)]</field>
        <field name="context">{'show_sale': True, 'create': False}</field>
        <field name="search_view_id" ref="view_sales_order_filter_ecommerce"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There is no unpaid order from the website yet
            </p><p>
                Process the order once the payment is received.
            </p>
        </field>
    </record>

    <record id="view_sales_order_filter_ecommerce_abondand" model="ir.ui.view">
        <field name="name">sale.order.ecommerce.abondand.view</field>
        <field name="model">sale.order</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <search string="Search Abandoned Sales Orders">
                <field name="name"/>
                <filter string="Creation Date" name="creation_date" date="create_date"/>
                <separator/>
                <filter string="Recovery Email to Send" name="recovery_email" domain="[('cart_recovery_email_sent', '=', False)]" />
                <filter string="Recovery Email Sent" name="recovery_email_set" domain="[('cart_recovery_email_sent', '=', True)]" />
                <group expand="0" string="Group By">
                    <filter string="Order Date" name="order_date" domain="[]" context="{'group_by':'date_order'}"/>
                </group>
                <!-- Dashboard filter - used by context -->
                <filter string="Last Week" invisible="1" name="week" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=7)).strftime('%%Y-%%m-%%d'))]"/>
                <filter string="Last Month" invisible="1" name="month" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=30)).strftime('%%Y-%%m-%%d'))]"/>
                <filter string="Last Year" invisible="1"  name="year" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=365)).strftime('%%Y-%%m-%%d'))]"/>
            </search>
        </field>
    </record>

    <!-- Dashboard Action -->
    <record id="sale_order_action_to_invoice" model="ir.actions.act_window">
        <field name="name">Orders To Invoice</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">tree,form,kanban</field>
        <field name="domain">[('state', 'in', ('sale', 'done')), ('order_line', '!=', False), ('invoice_status', '=', 'to invoice'), ('website_id', '!=', False)]</field>
        <field name="context">{'show_sale': True, 'search_default_order_confirmed': 1, 'create': False}</field>
        <field name="search_view_id" ref="view_sales_order_filter_ecommerce"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                You don't have any order to invoice from the website
            </p>
        </field>
    </record>

    <!-- Server action to send multiple recovery email-->
    <record id="ir_actions_server_sale_cart_recovery_email" model="ir.actions.server">
        <field name="name">Send a Cart Recovery Email</field>
        <field name="type">ir.actions.server</field>
        <field name="model_id" ref="model_sale_order"/>
        <field name="state">code</field>
        <field name="code">
            if records:
                action = records.action_recovery_email_send()
        </field>
        <field name="binding_model_id" ref="sale.model_sale_order"/>
        <field name="binding_view_types">list,form</field>
    </record>

    <record id="action_view_unpaid_quotation_tree" model="ir.actions.act_window">
        <field name="name">Unpaid Orders</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">tree,kanban,form,activity</field>
        <field name="domain">[('state', '=', 'sent'), ('website_id', '!=', False)]</field>
        <field name="context" eval="{'show_sale': True, 'create': False}"/>
        <field name="view_id" ref="sale.view_quotation_tree"/>
        <field name="search_view_id" ref="view_sales_order_filter_ecommerce_unpaid"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There is no unpaid order from the website yet
            </p><p>
                Process the order once the payment is received.
            </p>
        </field>
    </record>

    <record id="action_view_abandoned_tree" model="ir.actions.act_window">
        <field name="name">Abandoned Carts</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">tree,kanban,form,activity</field>
        <field name="domain">[('is_abandoned_cart', '=', 1)]</field>
        <field name="context" eval="{'show_sale': True, 'create': False, 'public_partner_id': ref('base.public_partner'), 'search_default_recovery_email': True}"/>
        <field name="view_id" ref="sale.view_quotation_tree"/>
        <field name="search_view_id" ref="view_sales_order_filter_ecommerce_abondand"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No abandoned carts found
            </p><p>
                You'll find here all the carts abandoned by your visitors.
                If they completed their address, you should send them a recovery email!
            </p><p>
                The time to mark a cart as abandoned can be changed in the settings.
            </p>
        </field>
    </record>

    <!-- Main website sale menu items -->
    <menuitem id="menu_orders" name="Orders"
        parent="website.menu_website_configuration" sequence="2"
        groups="sales_team.group_sale_salesman"/>
    <menuitem id="menu_catalog" name="Products"
        parent="website.menu_website_configuration" sequence="3"
        groups="sales_team.group_sale_salesman"/>
    <menuitem id="menu_reporting" name="Reporting"
        parent="website.menu_website_configuration" sequence="99"
        groups="sales_team.group_sale_manager"/>
    <menuitem id="website.menu_website_global_configuration" name="Configuration"
        parent="website.menu_website_configuration" sequence="100"
        groups="base.group_system,sales_team.group_sale_manager"/>

    <menuitem id="menu_ecommerce_settings" name="eCommerce" sequence="50"
        parent="website.menu_website_global_configuration"/>
    <menuitem id="menu_product_settings" name="Products" sequence="80"
        parent="website.menu_website_global_configuration"/>

    <!-- Orders sub-menus -->
    <menuitem id="menu_orders_orders" name="Orders"
        action="action_orders_ecommerce"
        parent="menu_orders" sequence="1"/>
    <menuitem id="menu_orders_unpaid_orders" name="Unpaid Orders"
        action="action_view_unpaid_quotation_tree"
        parent="menu_orders" sequence="2"/>
    <menuitem id="menu_orders_abandoned_orders" name="Abandoned Carts"
        action="action_view_abandoned_tree"
        parent="menu_orders" sequence="3"/>
    <menuitem id="menu_orders_customers" name="Customers"
        action="base.action_partner_customer_form"
        parent="menu_orders" sequence="4"/>


    <!-- Catalog sub-menus -->
    <menuitem id="menu_catalog_products" name="Products"
        action="product_template_action_website"
        parent="menu_catalog" sequence="1"/>
    <menuitem id="product_catalog_variants" name="Product Variants"
        action="product.product_normal_action"
        parent="menu_catalog" groups="product.group_product_variant" sequence="2"/>
    <menuitem id="menu_catalog_pricelists" name="Pricelists"
        action="product.product_pricelist_action2"
        parent="menu_catalog" groups="product.group_product_pricelist" sequence="4"/>

    <!-- Reporting sub-menus -->
    <menuitem id="menu_report_sales" name="Online Sales"
        action="sale_report_action_dashboard"
        parent="menu_reporting" sequence="1"/>

    <!-- Configuration sub-menus -->
    <menuitem id="menu_ecommerce_payment_acquirers"
        action="payment.action_payment_acquirer"
        parent="menu_ecommerce_settings" name="Payment Acquirers"/>
    <menuitem id="menu_ecommerce_payment_tokens"
        action="payment.payment_token_action"
        groups="base.group_no_one"
        parent="menu_ecommerce_settings"/>
    <menuitem id="menu_ecommerce_payment_icons"
        action="payment.action_payment_icon"
        groups="base.group_no_one"
        parent="menu_ecommerce_settings"/>
    <menuitem id="menu_ecommerce_payment_transactions"
        action="payment.action_payment_transaction"
        groups="base.group_no_one"
        parent="menu_ecommerce_settings"/>
    <menuitem id="menu_catalog_categories"
        action="product_public_category_action"
        parent="menu_product_settings" sequence="1"/>
    <menuitem id="menu_product_attribute_action"
        action="product.attribute_action"
        parent="menu_product_settings"  groups="product.group_product_variant" sequence="2"/>

    <record id="sale_order_view_form" model="ir.ui.view">
        <field name="name">sale.order.form</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="attributes">
                <attribute name="context">{
                    'display_website': True,
                    'res_partner_search_mode': 'customer',
                    'show_address': 1,
                    'show_vat': True,
                }</attribute>
            </field>
            <field name="team_id" position="after">
                <field name="website_id" attrs="{'invisible': [('website_id', '=', False)]}" groups="website.group_multi_website"/>
            </field>
        </field>
    </record>

    <record id="view_quotation_tree" model="ir.ui.view">
        <field name="name">sale.order.tree.inherit.website.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_quotation_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="before">
                <field name="website_id" optional="hide"/>
            </xpath>
        </field>
    </record>

    <record id="view_order_tree" model="ir.ui.view">
        <field name="name">sale.order.tree.inherit.website.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="before">
                <field name="website_id" optional="show"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\sale_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_report_view_search_website" model="ir.ui.view">
        <field name="name">sale.report.search</field>
        <field name="model">sale.report</field>
        <field name="arch" type="xml">
            <search string="Sales">
                <field name="website_id" groups="website.group_multi_website"/>
                <field name="product_id"/>
                <field name="categ_id"/>
                <field name="partner_id"/>
                <field name="country_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <filter string="Confirmed Orders" name="confirmed" domain="[('state', 'in', ['sale', 'done'])]"/>
                <separator/>
                <filter name="filter_date" date="date" default_period="this_month"/>
                <group expand="0" string="Group By">
                    <filter string="Website" name="groupby_website" context="{'group_by':'website_id'}" groups="website.group_multi_website"/>
                    <filter string="Product" name="groupby_product" context="{'group_by':'product_id'}"/>
                    <filter string="Product Category" name="groupby_product_category" context="{'group_by':'categ_id'}"/>
                    <filter string="Customer" name="groupby_customer" context="{'group_by':'partner_id'}"/>
                    <filter string="Customer Country" name="groupby_country" context="{'group_by':'country_id'}"/>
                    <filter string="Status" name="groupby_status" context="{'group_by':'state'}"/>
                    <separator orientation="vertical"/>
                    <filter string="Order Date" name="groupby_order_date" context="{'group_by':'date'}"/>
                    <!-- Dashboard filter - used by context -->
                    <filter string="Last Week" invisible="1" name="week" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=7)).strftime('%%Y-%%m-%%d'))]"/>
                    <filter string="Last Month" invisible="1" name="month" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=30)).strftime('%%Y-%%m-%%d'))]"/>
                    <filter string="Last Year" invisible="1"  name="year" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=365)).strftime('%%Y-%%m-%%d'))]"/>
                </group>
            </search>
        </field>
    </record>

    <record id="sale_report_view_pivot_website" model="ir.ui.view">
        <field name="name">sale.report.view.pivot.website</field>
        <field name="model">sale.report</field>
        <field name="arch" type="xml">
            <pivot string="Sales Report" disable_linking="True" sample="1">
                <field name="date" type="row"/>
                <field name="state" type="col"/>
                <field name="price_subtotal" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="sale_report_view_graph_website" model="ir.ui.view">
        <field name="name">sale.report.view.graph.website</field>
        <field name="model">sale.report</field>
        <field name="arch" type="xml">
            <graph string="Sale Report" type="bar" sample="1" disable_linking="1">
                <field name="date"/>
                <field name="price_subtotal" type='measure'/>
            </graph>
        </field>
    </record>

    <record id="sale_report_action_dashboard" model="ir.actions.act_window">
        <field name="name">Online Sales Analysis</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">pivot,graph</field>
        <field name="domain">[('website_id', '!=', False)]</field>
        <field name="context">{'search_default_confirmed': 1}</field>
        <field name="search_view_id" ref="sale_report_view_search_website"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                You don't have any order from the website
            </p>
        </field>
    </record>

    <record id="sale_report_action_view_pivot_website" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="sale_report_view_pivot_website"/>
        <field name="act_window_id" ref="sale_report_action_dashboard"/>
    </record>

    <record id="sale_report_action_view_graph_website" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="sale_report_view_graph_website"/>
        <field name="act_window_id" ref="sale_report_action_dashboard"/>
    </record>

    <record id="sale_report_action_carts" model="ir.actions.act_window">
        <field name="name">Sales</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">pivot,graph</field>
        <field name="domain">[('website_id', '!=', False)]</field>
        <field name="search_view_id" ref="sale_report_view_search_website"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                You don't have any order from the website
            </p>
        </field>
    </record>

    <record id="sale_report_action_view_pivot_carts" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="sale_report_view_pivot_website"/>
        <field name="act_window_id" ref="sale_report_action_carts"/>
    </record>

    <record id="sale_report_action_view_graph_carts" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="sale_report_view_graph_website"/>
        <field name="act_window_id" ref="sale_report_action_carts"/>
    </record>
</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="_assets_primary_variables" inherit_id="website._assets_primary_variables">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/website_sale/static/src/scss/primary_variables.scss"/>
        </xpath>
    </template>

    <template id="assets_backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_video_field_preview.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_backend.js"></script>
            <link rel="stylesheet" type="text/scss" href="/website_sale/static/src/scss/website_sale_dashboard.scss"/>
            <link rel="stylesheet" type="text/scss" href="/website_sale/static/src/scss/website_sale_backend.scss"/>
            <script type="text/javascript" src="/website_sale/static/src/js/tours/website_sale_shop_backend.js"></script>
        </xpath>
    </template>

    <template id="assets_frontend" inherit_id="website.assets_frontend">
        <xpath expr="link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/website_sale/static/src/scss/website_sale.scss" />
            <link rel="stylesheet" type="text/scss" href="/website_sale/static/src/scss/website_mail.scss" />
            <link rel="stylesheet" type="text/scss" href="/website_sale/static/src/scss/website_sale_frontend.scss"/>
            <link rel="stylesheet" type="text/scss" href="/sale/static/src/scss/sale_portal.scss"/>
            <link rel="stylesheet" type="text/scss" href="/sale/static/src/scss/product_configurator.scss"/>
        </xpath>
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/sale/static/src/js/variant_mixin.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/variant_mixin.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_utils.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_payment.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_validate.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_recently_viewed.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_tracking.js"></script>
        </xpath>
    </template>

    <template id="assets_wysiwyg" inherit_id="website.assets_wysiwyg" name="Shop Wysiwyg Editor">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/website_sale/static/src/scss/website_sale.editor.scss"/>
            <script type="text/javascript" src="/website_sale/static/src/snippets/s_dynamic_snippet_products/options.js"/>
        </xpath>
    </template>

    <template id="assets_editor" inherit_id="website.assets_editor" name="Shop Editor">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale.editor.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/website_sale_form_editor.js"></script>
            <script type="text/javascript" src="/website_sale/static/src/js/tours/website_sale_shop_frontend.js"></script>
        </xpath>
    </template>

    <template id="assets_common" name="tour" inherit_id="web.assets_common">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/website_sale/static/src/js/tours/website_sale_shop.js"></script>
        </xpath>
    </template>

    <template id="assets_tests" name="Website Sale Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_buy.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_complete_flow.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_cart_recovery.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_mail.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_customize.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_custom_attribute_value.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_zoom.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_dynamic_variants.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_deleted_archived_variants.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_list_view_b2c.js"></script>
            <script type="text/javascript" src="/website_sale/static/tests/tours/website_sale_shop_no_variant_attribute.js"></script>
        </xpath>
    </template>

    <template id="header_cart_link" name="Header Cart Link">
        <t t-set="website_sale_order" t-value="website.sale_get_order()" />
        <t t-set="show_cart" t-value="true"/>
        <li t-attf-class="#{_item_class} divider d-none"/> <!-- Make sure the cart and related menus are not folded (see autohideMenu) -->
        <li t-attf-class="o_wsale_my_cart #{not show_cart and 'd-none'} #{_item_class}">
            <a href="/shop/cart" t-attf-class="#{_link_class}">
                <i t-if="_icon" class="fa fa-shopping-cart"/>
                <span t-if="_text">My Cart</span>
                <sup class="my_cart_quantity badge badge-primary" t-esc="website_sale_order and website_sale_order.cart_quantity or '0'" t-att-data-order-id="website_sale_order and website_sale_order.id or ''"/>
            </a>
        </li>
    </template>

    <template id="header_hide_empty_cart_link" inherit_id="website_sale.header_cart_link" name="Header Hide Empty Cart link" active="False">
        <xpath expr="//t[@t-set='show_cart']" position="after">
            <t t-set="show_cart" t-value="show_cart and website_sale_order and website_sale_order.cart_quantity" />
        </xpath>
    </template>

    <template id="template_header_default" inherit_id="website.template_header_default">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item mx-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_hamburger" inherit_id="website.template_header_hamburger">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_vertical" inherit_id="website.template_header_vertical">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ml-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_sidebar" inherit_id="website.template_header_sidebar">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_slogan" inherit_id="website.template_header_slogan">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ml-lg-auto'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_contact" inherit_id="website.template_header_contact">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ml-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_minimalist" inherit_id="website.template_header_minimalist">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link ml-3'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_boxed" inherit_id="website.template_header_boxed">
        <xpath expr="//t[@t-call='portal.placeholder_user_sign_in']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_centered_logo" inherit_id="website.template_header_centered_logo">
        <xpath expr="//t[@t-call='portal.placeholder_user_sign_in']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_image" inherit_id="website.template_header_image">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ml-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_hamburger_full" inherit_id="website.template_header_hamburger_full">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ml-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_magazine" inherit_id="website.template_header_magazine">
        <xpath expr="//t[@t-foreach='website.menu_id.child_id']" position="after">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <!-- Products Search Bar input-group template -->
    <template id="website_sale_products_search_box" inherit_id="website.website_search_box" primary="True">
        <xpath expr="//input[@name='search']" position="attributes">
            <attribute name="data-limit">5</attribute>
            <attribute name="data-display-description">true</attribute>
            <attribute name="data-display-price">true</attribute>
            <attribute name="data-display-image">true</attribute>
        </xpath>
        <xpath expr="//div[@role='search']" position="attributes">
            <attribute name="t-attf-class" remove="#{_classes}" separator=" "/>
        </xpath>
        <xpath expr="//div[@role='search']" position="replace">
            <form t-attf-class="o_wsale_products_searchbar_form o_wait_lazy_js #{_classes}" t-att-action="action if action else '/shop'" method="get" t-att-data-snippet="_snippet">
                <t>$0</t>
                <input name="order" type="hidden" class="o_wsale_search_order_by" value=""/>
                <t t-raw="0"/>
            </form>
        </xpath>
    </template>

    <template id="search" name="Search Box">
        <t t-call="website_sale.website_sale_products_search_box">
            <t t-set="action" t-value="keep('/shop'+ ('/category/'+slug(category)) if category else None, search=0)"/>
            <t t-if="attrib_values">
                <t t-foreach="attrib_values" t-as="a">
                    <input type="hidden" name="attrib" t-att-value="'%s-%s' % (a[0], a[1])" />
                </t>
            </t>
        </t>
    </template>

    <template id="search_count_box" inherit_id="website.website_search_box" active="False" customize_show="True" name="Show # found">
        <xpath expr="//button[hasclass('oe_search_button')]" position="inside">
            <span t-if='search' class='oe_search_found'> <small>(<t t-esc="search_count"/> found)</small></span>
        </xpath>
    </template>

    <template id="products_item" name="Products item">
        <t t-set="product_href" t-value="keep(product.website_url, page=(pager['page']['num'] if pager['page']['num']&gt;1 else None))" />

        <t t-set="combination_info" t-value="product._get_combination_info(only_template=True, add_qty=add_qty or 1, pricelist=pricelist)"/>

        <form action="/shop/cart/update" method="post" class="card oe_product_cart"
            t-att-data-publish="product.website_published and 'on' or 'off'"
            itemscope="itemscope" itemtype="http://schema.org/Product">
            <div class="card-body p-1 oe_product_image">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                <a t-att-href="product_href" class="d-block h-100" itemprop="url">
                    <t t-set="image_holder" t-value="product._get_image_holder()"/>
                    <span t-field="image_holder.image_1920"
                        t-options="{'widget': 'image', 'preview_image': 'image_1024' if product_image_big else 'image_256', 'itemprop': 'image'}"
                        class="d-flex h-100 justify-content-center align-items-center"/>
                </a>
            </div>
            <div class="card-body p-0 text-center o_wsale_product_information">
                <div class="p-2 o_wsale_product_information_text">
                    <h6 class="o_wsale_products_item_title">
                        <a itemprop="name" t-att-href="product_href" t-att-content="product.name" t-field="product.name" />
                        <a role="button" t-if="not product.website_published" t-att-href="product_href" class="btn btn-sm btn-danger" title="This product is unpublished.">Unpublished</a>
                    </h6>
                    <div class="product_price" itemprop="offers" itemscope="itemscope" itemtype="http://schema.org/Offer">
                        <del t-attf-class="text-danger mr-2 {{'' if combination_info['has_discounted_price'] else 'd-none'}}" style="white-space: nowrap;" t-esc="combination_info['list_price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}" />
                        <span t-if="combination_info['price']" t-esc="combination_info['price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                        <span itemprop="price" style="display:none;" t-esc="combination_info['price']" />
                        <span itemprop="priceCurrency" style="display:none;" t-esc="website.currency_id.name" />
                    </div>
                </div>
                <div class="o_wsale_product_btn"/>
            </div>
            <t t-set="bg_color" t-value="td_product['ribbon']['bg_color'] or ''"/>
            <t t-set="text_color" t-value="td_product['ribbon']['text_color']"/>
            <t t-set="bg_class" t-value="td_product['ribbon']['html_class']"/>
            <span t-attf-class="o_ribbon #{bg_class}" t-attf-style="#{text_color and ('color: %s; ' % text_color)}#{bg_color and 'background-color:' + bg_color}" t-raw="td_product['ribbon']['html'] or ''"/>
        </form>
    </template>

    <template id="products_description" inherit_id="website_sale.products_item" active="False" customize_show="True" name="Product Description">
        <xpath expr="//*[hasclass('product_price')]" position="after">
            <div class="oe_subdescription" contenteditable="false">
                <div itemprop="description" t-field="product.description_sale"/>
            </div>
        </xpath>
    </template>

    <template id="products_add_to_cart" inherit_id="website_sale.products_item" active="False" customize_show="True" name="Add to Cart">
        <xpath expr="//div[hasclass('o_wsale_product_btn')]" position="inside">
            <t t-set="product_variant_id" t-value="product._get_first_possible_variant_id()"/>
            <input name="product_id" t-att-value="product_variant_id" type="hidden"/>
            <t t-if="product_variant_id">
                <a href="#" role="button" class="btn btn-secondary a-submit" aria-label="Shopping cart" title="Shopping cart">
                    <span class="fa fa-shopping-cart"/>
                </a>
            </t>
        </xpath>
    </template>

    <template id="pricelist_list" name="Pricelists Dropdown">
        <t t-set="website_sale_pricelists" t-value="website.get_pricelist_available(show_visible=True)" />
        <div t-attf-class="dropdown#{'' if website_sale_pricelists and len(website_sale_pricelists)&gt;1 else ' d-none'} #{_classes}">
            <t t-set="curr_pl" t-value="website.get_current_pricelist()" />
            <a role="button" href="#" class="dropdown-toggle btn btn-secondary" data-toggle="dropdown">
                <t t-esc="curr_pl and curr_pl.name or ' - '" />
            </a>
            <div class="dropdown-menu" role="menu">
                <t t-foreach="website_sale_pricelists" t-as="pl">
                    <a role="menuitem" t-att-href="'/shop/change_pricelist/%s' % pl.id" class="dropdown-item">
                        <span class="switcher_pricelist" t-att-data-pl_id="pl.id" t-esc="pl.name" />
                    </a>
                </t>
            </div>
        </div>
    </template>

    <!-- /shop product listing -->
    <template id="products" name="Products">
        <t t-call="website.layout">
            <t t-set="additional_title">Shop</t>
            <div id="wrap" class="js_sale">
                <div class="oe_structure oe_empty" id="oe_structure_website_sale_products_1"/>
                <div class="container oe_website_sale">
                    <div class="products_pager form-inline flex-md-nowrap justify-content-between justify-content-md-center">
                        <t t-call="website_sale.search">
                            <t t-set="_classes" t-valuef="w-100 w-md-auto mt-2"/>
                        </t>
                        <t t-call="website_sale.pricelist_list">
                            <t t-set="_classes" t-valuef="mt-2 ml-md-2"/>
                        </t>
                        <t t-call="website.pager">
                            <t t-set="_classes" t-valuef="mt-2 ml-md-2"/>
                        </t>
                    </div>
                    <div class="row o_wsale_products_main_row">
                        <div t-if="enable_left_column" id="products_grid_before" class="col-lg-3"/>
                        <div id="products_grid" t-attf-class="col #{'o_wsale_layout_list' if layout_mode == 'list' else ''}">
                            <t t-if="category">
                                <t t-set='editor_msg'>Drag building blocks here to customize the header for "<t t-esc='category.name'/>" category.</t>
                                <div class="mb16" id="category_header" t-att-data-editor-message="editor_msg" t-field="category.website_description"/>
                            </t>
                            <div t-if="bins" class="o_wsale_products_grid_table_wrapper">
                                <table class="table table-borderless m-0" t-att-data-ppg="ppg" t-att-data-ppr="ppr">
                                    <colgroup t-ignore="true">
                                        <!-- Force the number of columns (useful when only one row of (x < ppr) products) -->
                                        <col t-foreach="ppr" t-as="p"/>
                                    </colgroup>
                                    <tbody>
                                        <tr t-foreach="bins" t-as="tr_product">
                                            <t t-foreach="tr_product" t-as="td_product">
                                                <t t-if="td_product">
                                                    <t t-set="product" t-value="td_product['product']" />
                                                    <!-- We use t-attf-class here to allow easier customization -->
                                                    <td t-att-colspan="td_product['x'] != 1 and td_product['x']"
                                                        t-att-rowspan="td_product['y'] != 1 and td_product['y']"
                                                        t-attf-class="oe_product"
                                                        t-att-data-ribbon-id="td_product['ribbon'].id">
                                                        <div t-attf-class="o_wsale_product_grid_wrapper o_wsale_product_grid_wrapper_#{td_product['x']}_#{td_product['y']}">
                                                            <t t-call="website_sale.products_item">
                                                                <t t-set="product_image_big" t-value="td_product['x'] + td_product['y'] &gt; 2"/>
                                                            </t>
                                                        </div>
                                                    </td>
                                                </t>
                                                <td t-else=""/>
                                            </t>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                            <t t-else="">
                                <div class="text-center text-muted">
                                    <t t-if="not search">
                                        <h3 class="mt8">No product defined</h3>
                                        <p t-if="category">No product defined in category "<strong t-esc="category.display_name"/>".</p>
                                    </t>
                                    <t t-else="">
                                        <h3 class="mt8">No results</h3>
                                        <p>No results for "<strong t-esc='search'/>"<t t-if="category"> in category "<strong t-esc="category.display_name"/>"</t>.</p>
                                    </t>
                                    <p t-ignore="true" groups="sales_team.group_sale_manager">Click <i>'New'</i> in the top-right corner to create your first product.</p>
                                </div>
                            </t>
                        </div>
                    </div>
                    <div class="products_pager form-inline justify-content-center py-3">
                        <t t-call="website.pager"/>
                    </div>
                </div>
                <div class="oe_structure oe_empty" id="oe_structure_website_sale_products_2"/>
            </div>
        </t>
    </template>

    <!-- (Option) Products: Enable image full for all products : Images Full -->
    <template id="products_images_full" name="Images Full" inherit_id="website_sale.products" active="False" customize_show="True">
        <xpath expr="//t[@t-foreach='tr_product']//td" position="attributes">
            <attribute name="t-attf-class" add="oe_image_full" separator=" "/>
        </xpath>
    </template>

    <template id="sort" inherit_id="website_sale.products" customize_show="True" name="Show Sort by">
        <xpath expr="//div[hasclass('products_pager')]/t[@t-call][last()]" position="after">
            <t t-set="list_price_desc_label">Catalog price: High to Low</t>
            <t t-set="list_price_asc_label">Catalog price: Low to High</t>
            <t t-set="name_asc_label">Name: A to Z</t>
            <t t-set="name_desc_label">Name: Z to A</t>
            <t t-set="website_sale_sortable" t-value="[
                (list_price_desc_label, 'list_price desc'),
                (list_price_asc_label, 'list_price asc'),
                (name_asc_label, 'name asc'),
                (name_desc_label, 'name desc')
            ]"/>
            <t t-set="website_sale_sortable_current" t-value="[sort for sort in website_sale_sortable if sort[1]==request.params.get('order', '')]"/>
            <div class="dropdown mt-2 ml-md-2 dropdown_sorty_by">
                <a role="button" href="#" class="dropdown-toggle btn btn-secondary" data-toggle="dropdown">
                    <span class="d-none d-lg-inline">
                        <t t-if='len(website_sale_sortable_current)'>
                            Sorting by : <t t-raw='website_sale_sortable_current[0][0]'/>
                        </t>
                        <t t-else='1'>
                            Sort by
                        </t>
                    </span>
                    <i class="fa fa-sort-amount-asc d-lg-none"/>
                </a>
                <div class="dropdown-menu dropdown-menu-right" role="menu">
                    <t t-foreach="website_sale_sortable" t-as="sortby">
                        <a role="menuitem" rel="noindex,nofollow" t-att-href="keep('/shop', order=sortby[1])" class="dropdown-item">
                            <span t-raw="sortby[0]"/>
                        </a>
                    </t>
                </div>
            </div>
        </xpath>
    </template>

    <template id="add_grid_or_list_option" inherit_id="website_sale.products" active="True" customize_show="True" name="Grid or List button">
        <xpath expr="//div[hasclass('products_pager')]/t[@t-call][last()]" position="after">
            <div class="btn-group btn-group-toggle mt-2 ml-md-2 d-none d-sm-inline-flex o_wsale_apply_layout" data-toggle="buttons">
                <label t-attf-class="btn btn-secondary #{'active' if layout_mode != 'list' else None} fa fa-th-large o_wsale_apply_grid" title="Grid">
                    <input type="radio" name="wsale_products_layout" t-att-checked="'checked' if layout_mode != 'list' else None"/>
                </label>
                <label t-attf-class="btn btn-secondary #{'active' if layout_mode == 'list' else None} fa fa-th-list o_wsale_apply_list" title="List">
                    <input type="radio" name="wsale_products_layout" t-att-checked="'checked' if layout_mode == 'list' else None"/>
                </label>
            </div>
        </xpath>
    </template>

    <!-- Add to cart button-->
    <template id="categories_recursive" name="Category list">
        <li class="nav-item">
            <a t-att-href="keep('/shop/category/' + slug(c), category=0)" t-attf-class="nav-link #{'active' if c.id == category.id else ''}">
                <span t-field="c.name"/>
            </a>
            <ul t-if="c.child_id" class="nav nav-pills flex-column nav-hierarchy">
                <t t-foreach="c.child_id" t-as="c">
                    <t t-if="not search or c.id in search_categories_ids">
                        <t t-call="website_sale.categories_recursive" />
                    </t>
                </t>
            </ul>
        </li>
    </template>

    <template id="products_categories" inherit_id="website_sale.products" active="False" customize_show="True" name="eCommerce Categories">
        <xpath expr="//div[@id='products_grid_before']" position="before">
            <t t-set="enable_left_column" t-value="True"/>
        </xpath>
        <xpath expr="//div[@id='products_grid_before']" position="inside">
            <button type="button" class="btn btn-link d-lg-none"
                data-target="#wsale_products_categories_collapse" data-toggle="collapse">
                Show categories
            </button>
            <div class="collapse d-lg-block" id="wsale_products_categories_collapse">
                <ul class="nav nav-pills flex-column mb-2">
                    <li class="nav-item">
                        <a t-att-href="keep('/shop', category=0)" t-attf-class="nav-link #{'' if category else 'active'} o_not_editable">All Products</a>
                    </li>
                    <t t-foreach="categories" t-as="c">
                        <t t-call="website_sale.categories_recursive" />
                    </t>
                </ul>
            </div>
        </xpath>
    </template>

    <template id="option_collapse_categories_recursive" name="Collapse Category Recursive">
        <li class="nav-item">
            <t t-set="children" t-value="not search and c.child_id or c.child_id.filtered(lambda c: c.id in search_categories_ids)"/>
            <i t-if="children" t-attf-class="text-primary fa #{'fa-chevron-down' if c.id in category.parents_and_self.ids else 'fa-chevron-right'}"
               t-attf-title="#{'Unfold' if c.id in category.parents_and_self.ids else 'Fold'}"
               t-attf-aria-label="#{'Unfold' if c.id in category.parents_and_self.ids else 'Fold'}" role="img"/>
            <a t-att-href="keep('/shop/category/' + slug(c), category=0)" t-attf-class="nav-link #{'active' if c.id == category.id else ''}" t-field="c.name"></a>
            <ul t-if="children" class="nav nav-pills flex-column nav-hierarchy" t-att-style="'display:block;' if c.id in category.parents_and_self.ids else 'display:none;'">
                <t t-foreach="children" t-as="c">
                    <t t-call="website_sale.option_collapse_categories_recursive"/>
                </t>
            </ul>
        </li>
    </template>

    <template id="option_collapse_products_categories" name="Collapsible Category List" inherit_id="website_sale.products_categories" active="False" customize_show="True">
        <xpath expr="//div[@id='wsale_products_categories_collapse']/ul" position="attributes">
            <attribute name="id">o_shop_collapse_category</attribute>
        </xpath>
        <xpath expr="//t[@t-call='website_sale.categories_recursive']" position="attributes">
            <attribute name="t-call">website_sale.option_collapse_categories_recursive</attribute>
        </xpath>
    </template>

    <template id="products_attributes" inherit_id="website_sale.products" active="False" customize_show="True" name="Product Attribute's Filters">
        <xpath expr="//div[@id='products_grid_before']" position="before">
            <t t-set="enable_left_column" t-value="True"/>
        </xpath>
        <xpath expr="//div[@id='products_grid_before']" position="inside">
            <button type="button" class="btn btn-link d-lg-none"
                data-target="#wsale_products_attributes_collapse" data-toggle="collapse">
                Show options
            </button>
            <div class="collapse d-lg-block" id="wsale_products_attributes_collapse">
                <form class="js_attributes mb-2" method="get">
                    <input t-if="category" type="hidden" name="category" t-att-value="category.id" />
                    <input type="hidden" name="search" t-att-value="search" />
                    <input type="hidden" name="order" t-att-value="order"/>
                    <ul class="nav nav-pills flex-column">
                        <t t-foreach="attributes" t-as="a">
                            <li t-if="a.value_ids and len(a.value_ids) &gt; 1" class="nav-item">
                                <div>
                                    <strong t-field="a.name" />
                                </div>
                                <t t-if="a.display_type == 'select'">
                                    <select class="form-control" name="attrib">
                                        <option value="" />
                                        <t t-foreach="a.value_ids" t-as="v">
                                            <option t-att-value="'%s-%s' % (a.id,v.id)" t-esc="v.name" t-att-selected="v.id in attrib_set" />
                                        </t>
                                    </select>
                                </t>
                                <t t-if="a.display_type == 'radio'">
                                    <ul class="nav nav-pills flex-column">
                                        <t t-foreach="a.value_ids" t-as="v">
                                            <li class="nav-item">
                                                <label style="padding: 0; margin: 0" t-attf-class="nav-link#{' active' if v.id in attrib_set else ''}">
                                                    <input type="checkbox" name="attrib" t-att-value="'%s-%s' % (a.id,v.id)" t-att-checked="'checked' if v.id in attrib_set else None" />
                                                    <span style="font-weight: normal" t-field="v.name" />
                                                </label>
                                            </li>
                                        </t>
                                    </ul>
                                </t>
                                <t t-if="a.display_type == 'color'">
                                    <t t-foreach="a.value_ids" t-as="v">
                                        <label t-attf-style="background-color:#{v.html_color or v.name}" t-attf-class="css_attribute_color #{'active' if v.id in attrib_set else ''}">
                                            <input type="checkbox" name="attrib" t-att-value="'%s-%s' % (a.id,v.id)" t-att-checked="'checked' if v.id in attrib_set else None" t-att-title="v.name" />
                                        </label>
                                    </t>
                                </t>
                            </li>
                        </t>
                    </ul>
                </form>
            </div>
        </xpath>
    </template>

    <template id="products_list_view" inherit_id="website_sale.products" active="False" customize_show="True" name="List View (by default)">
        <xpath expr="//div[@id='products_grid']" position="after">
            <!-- Nothing to do, this view is only meant to allow the server -->
            <!-- to know if the list view layout should be used -->
        </xpath>
    </template>

    <!-- /shop/product page -->
    <template id="product_edit_options" inherit_id="website.user_navbar" name="Edit Product Options">
        <xpath expr="//li[@id='edit-page-menu']" position="after">
            <t t-if="main_object._name == 'product.template'" t-set="action" t-value="'website_sale.product_template_action_website'" />
        </xpath>
    </template>

    <template id="product" name="Product" track="1">

        <t t-set="combination" t-value="product._get_first_possible_combination()"/>
        <t t-set="combination_info" t-value="product._get_combination_info(combination, add_qty=add_qty or 1, pricelist=pricelist)"/>
        <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

        <t t-call="website.layout">
            <t t-set="additional_title" t-value="product.name" />
            <div itemscope="itemscope" itemtype="http://schema.org/Product" id="wrap" class="js_sale">
                <section t-attf-class="container py-2 oe_website_sale #{'discount' if combination_info['has_discounted_price'] else ''}" id="product_detail" t-att-data-view-track="view_track and '1' or '0'">
                    <div class="row">
                        <div class="col-md-4">
                            <ol class="breadcrumb">
                                <li class="breadcrumb-item">
                                    <a t-att-href="keep(category=0)">Products</a>
                                </li>
                                <li t-if="category" class="breadcrumb-item">
                                    <a t-att-href="keep('/shop/category/%s' % slug(category), category=0)" t-field="category.name" />
                                </li>
                                <li class="breadcrumb-item active">
                                    <span t-field="product.name" />
                                </li>
                            </ol>
                        </div>
                        <div class="col-md-8">
                            <div class="form-inline justify-content-end">
                                <t t-call="website_sale.search">
                                    <t t-set="search" t-value="False"/>
                                </t>
                                <t t-call="website_sale.pricelist_list">
                                    <t t-set="_classes" t-valuef="ml-2"/>
                                </t>
                            </div>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-xl-8">
                            <t t-call="website_sale.shop_product_carousel"/>
                        </div>
                        <div class="col-md-6 col-xl-4" id="product_details">
                            <t t-set="base_url" t-value="product.get_base_url()"/>
                            <h1 itemprop="name" t-field="product.name">Product Name</h1>
                            <span itemprop="url" style="display:none;" t-esc="base_url + product.website_url"/>
                            <span itemprop="image" style="display:none;" t-esc="base_url + website.image_url(product, 'image_1920')" />
                            <form t-if="product._is_add_to_cart_possible()" action="/shop/cart/update" method="POST">
                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                                <div class="js_product js_main_product">
                                    <t t-placeholder="select">
                                        <input type="hidden" class="product_id" name="product_id" t-att-value="product_variant.id" />
                                        <input type="hidden" class="product_template_id" name="product_template_id" t-att-value="product.id" />
                                        <t t-if="combination" t-call="sale.variants">
                                            <t t-set="ul_class" t-value="'flex-column'" />
                                            <t t-set="parent_combination" t-value="None" />
                                        </t>
                                        <t t-else="">
                                            <ul class="d-none js_add_cart_variants" t-att-data-attribute_exclusions="{'exclusions: []'}"/>
                                        </t>
                                    </t>
                                    <t t-call="website_sale.product_price" />
                                    <p t-if="True" class="css_not_available_msg alert alert-warning">This combination does not exist.</p>
                                    <a role="button" id="add_to_cart" class="btn btn-primary btn-lg mt16 js_check_product a-submit d-block d-sm-inline-block" href="#"><i class="fa fa-shopping-cart"/> Add to Cart</a>
                                    <div id="product_option_block"/>
                                </div>
                            </form>
                            <p t-elif="not product.active" class="alert alert-warning">This product is no longer available.</p>
                            <p t-else="" class="alert alert-warning">This product has no valid combination.</p>
                            <hr t-if="product.description_sale" />
                            <div>
                                <p t-field="product.description_sale" class="text-muted mt-3" placeholder="A short description that will also appear on documents." />
                                <div id="product_attributes_simple">
                                    <hr t-if="sum([(1 if len(l.value_ids)==1 else 0) for l in product.attribute_line_ids])"/>
                                    <p class="text-muted">
                                        <t t-set="single_value_attributes" t-value="product.valid_product_template_attribute_line_ids._prepare_single_value_for_display()"/>
                                        <t t-foreach="single_value_attributes" t-as="attribute">
                                            <span t-field="attribute.name"/>:
                                            <t t-foreach="single_value_attributes[attribute]" t-as="ptal">
                                                <span t-field="ptal.product_template_value_ids._only_active().name"/><t t-if="not ptal_last">, </t>
                                            </t>
                                            <br/>
                                        </t>
                                    </p>
                                </div>
                            </div>
                            <hr />
                        </div>
                    </div>
                </section>
                <div itemprop="description" t-field="product.website_description" class="oe_structure oe_empty mt16" id="product_full_description"/>
            </div>
        </t>
    </template>

    <template id="product_custom_text" inherit_id="website_sale.product" active="True" customize_show="True" name="Terms and Conditions">
        <xpath expr="//div[@id='product_details']" position="inside">
            <p class="text-muted">
                <a href="/shop/terms">Terms and Conditions</a><br/>
                30-day money-back guarantee<br/>
                Shipping: 2-3 Business Days
            </p>
        </xpath>
    </template>

    <template inherit_id='website_sale.product' id="product_picture_magnify" customize_show="True" name="Image Zoom">
        <xpath expr='//div[hasclass("js_sale")]' position='attributes'>
            <attribute name="class" separator=" " add="ecom-zoomable zoomodoo-next" />
        </xpath>
    </template>

    <template inherit_id='website_sale.product' id="product_picture_magnify_auto" active="False" customize_show="True" name="Automatic Image Zoom">
        <xpath expr='//div[hasclass("js_sale")]' position='attributes'>
            <attribute name="data-ecom-zoom-auto">1</attribute>
            <attribute name="class" separator=" " add="ecom-zoomable zoomodoo-next" />

        </xpath>
    </template>

    <template id="recommended_products" inherit_id="website_sale.product" customize_show="True" name="Alternative Products">
        <xpath expr="//div[@id='product_full_description']" position="after">
            <div class="container mt32" t-if="product.alternative_product_ids">
                <h3>Alternative Products:</h3>
                <div class="row mt16" style="">
                    <t t-foreach="product.alternative_product_ids" t-as="alt_product">
                        <div class="col-lg-2" style="width: 170px; height:130px; float:left; display:inline; margin-right: 10px; overflow:hidden;">
                            <div class="mt16 text-center" style="height: 100%;">
                                <t t-set="combination_info" t-value="alt_product._get_combination_info()"/>
                                <t t-set="product_variant" t-value="alt_product.env['product.product'].browse(combination_info['product_id'])"/>
                                <div t-if="product_variant" t-field="product_variant.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded shadow o_alternative_product o_image_64_max' }" />
                                <div t-else="" t-field="alt_product.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded shadow o_alternative_product o_image_64_max' }" />
                                <h6>
                                    <a t-att-href="alt_product.website_url" style="display: block">
                                        <span t-att-title="alt_product.name" t-field="alt_product.name" class="o_text_overflow" style="display: block;" />
                                    </a>
                                </h6>
                            </div>
                        </div>
                    </t>
                </div>
            </div>
        </xpath>
    </template>

    <template id="recently_viewed_products_product" inherit_id="website_sale.product" active="False" customize_show="True" name="Recently Viewed Products" priority="16">
        <xpath expr="//div[@t-field='product.website_description']" position="after">
            <t t-snippet-call="website_sale.s_products_recently_viewed"/>
        </xpath>
    </template>

    <!-- Product options: OpenChatter -->
    <template id="product_comment" inherit_id="website_sale.product" active="False" customize_show="True" name="Discussion and Rating" priority="15">
        <xpath expr="//div[@t-field='product.website_description']" position="after">
            <div class="o_shop_discussion_rating">
                <section class="container mt16 mb16">
                    <hr/>
                    <div class="row">
                        <div class="col-lg-8 offset-lg-2">
                            <t t-call="portal.message_thread">
                                <t t-set="object" t-value="product"/>
                                <t t-set="display_rating" t-value="True"/>
                            </t>
                        </div>
                    </div>
                </section>
            </div>
        </xpath>
    </template>

    <template id="product_quantity" inherit_id="website_sale.product" customize_show="True" name="Select Quantity">
      <xpath expr="//a[@id='add_to_cart']" position="before">
        <div class="css_quantity input-group" contenteditable="false">
            <div class="input-group-prepend">
                <a t-attf-href="#" class="btn btn-secondary js_add_cart_json" aria-label="Remove one" title="Remove one">
                    <i class="fa fa-minus"></i>
                </a>
            </div>
            <input type="text" class="form-control quantity" data-min="1" name="add_qty" t-att-value="add_qty or 1"/>
            <div class="input-group-append">
                <a t-attf-href="#" class="btn btn-secondary float_left js_add_cart_json" aria-label="Add one" title="Add one">
                    <i class="fa fa-plus"></i>
                </a>
            </div>
        </div>
      </xpath>
    </template>

    <template id="product_buy_now" inherit_id="website_sale.product" active="False" customize_show="True" name="Buy Now Button">
        <xpath expr="//a[@id='add_to_cart']" position="after">
            <a role="button" id="buy_now" class="btn btn-outline-primary btn-lg mt16 d-block d-sm-inline-block" href="#"><i class="fa fa-bolt"/> Buy Now</a>
        </xpath>
    </template>

    <template id="product_price">
      <div itemprop="offers" itemscope="itemscope" itemtype="http://schema.org/Offer" class="product_price mt16">
          <h4 class="oe_price_h4 css_editable_mode_hidden">
              <span t-attf-class="text-danger oe_default_price {{'' if combination_info['has_discounted_price'] else 'd-none'}}" style="text-decoration: line-through; white-space: nowrap;"
                  t-esc="combination_info['list_price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"
              />
              <b class="oe_price" style="white-space: nowrap;" t-esc="combination_info['price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
              <span itemprop="price" style="display:none;" t-esc="combination_info['price']"/>
              <span itemprop="priceCurrency" style="display:none;" t-esc="website.currency_id.name"/>
          </h4>
          <h4 class="css_non_editable_mode_hidden decimal_precision" t-att-data-precision="str(website.currency_id.decimal_places)">
            <span t-field="product.list_price"
                t-options='{
                   "widget": "monetary",
                   "display_currency": product.currency_id,
               }'/>
          </h4>
      </div>
    </template>

    <template id="product_variants" inherit_id="website_sale.product" active="False" customize_show="True" name="List View of Variants">
        <xpath expr="//t[@t-placeholder='select']" position="replace">
            <!--
                Using this setting with dynamic variants is not supported.
                Indeed the variants that have yet to exist will not show on the
                list and will never be selectable to be created...

                We also don't use the feature with no_variant because these
                attributes have to be selected manually.

                Finally we don't use the feature with is_custom values because
                they need to be set by the user.
            -->
            <t t-if="not product.has_dynamic_attributes() and not product._has_no_variant_attributes() and not product._has_is_custom_values()">
                <t t-set="attribute_exclusions" t-value="product._get_attribute_exclusions()"/>
                <t t-set="filtered_sorted_variants" t-value="product._get_possible_variants_sorted()"/>
                <ul class="d-none js_add_cart_variants" t-att-data-attribute_exclusions="json.dumps(attribute_exclusions)"/>
                <input type="hidden" class="product_template_id" t-att-value="product.id"/>
                <input type="hidden" t-if="len(filtered_sorted_variants) == 1" class="product_id" name="product_id" t-att-value="filtered_sorted_variants[0].id"/>
                <t t-if="len(filtered_sorted_variants) &gt; 1">
                    <div t-foreach="filtered_sorted_variants" t-as="variant_id" class="custom-control custom-radio">
                        <t t-set="template_combination_info" t-value="product._get_combination_info(only_template=True, add_qty=add_qty, pricelist=pricelist)"/>
                        <t t-set="combination_info" t-value="variant_id._get_combination_info_variant(add_qty=add_qty, pricelist=pricelist)"/>
                        <input type="radio" name="product_id" class="custom-control-input product_id js_product_change" t-att-checked="'checked' if variant_id_index == 0 else None" t-attf-id="radio_variant_#{variant_id.id}" t-att-value="variant_id.id" t-att-data-price="combination_info['price']" t-att-data-combination="variant_id.product_template_attribute_value_ids.ids"/>
                        <label t-attf-for="radio_variant_#{variant_id.id}" label-default="label-default" class="custom-control-label">
                            <span t-esc="combination_info['display_name']"/>
                            <t t-set="diff_price" t-value="website.currency_id.compare_amounts(combination_info['price'], template_combination_info['price'])"/>
                            <span class="badge badge-pill badge-secondary" t-if="diff_price != 0">
                                <t t-esc="diff_price > 0 and '+' or '-'"/><span t-esc="abs(combination_info['price'] - template_combination_info['price'])" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                            </span>
                        </label>
                    </div>
                </t>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </template>

    <template id="wizard_checkout" name="Wizard Checkout">
        <t t-set="website_sale_order" t-value="website.sale_get_order()"/>

        <div class="row">
            <div class="col-xl">
                <div class="wizard">
                    <div class="progress-wizard">
                        <a class="no-decoration" t-att-href="step&gt;=10 and '/shop/cart' or '#'">
                          <div id="wizard-step10" t-att-class="'progress-wizard-step %s' % (step == 10 and 'active' or step&gt;10 and 'complete' or 'disabled')">
                            <div class="progress-wizard-bar d-none d-md-block"/>
                            <span class="progress-wizard-dot d-none d-md-inline-block"></span>
                            <div class="text-center progress-wizard-steplabel">Review Order</div>
                          </div>
                        </a>
                        <a class="no-decoration" t-att-href="step&gt;=20 and '/shop/checkout' or '#'">
                          <div id="wizard-step20" t-att-class="'progress-wizard-step %s' % (step == 20 and 'active' or step&gt;20 and 'complete' or 'disabled')">
                            <div class="progress-wizard-bar d-none d-md-block"/>
                            <span class="progress-wizard-dot d-none d-md-inline-block"></span>
                            <div class="text-center progress-wizard-steplabel">Address</div>
                          </div>
                        </a>
                        <a class="no-decoration" t-att-href="step&gt;=40 and '/shop/payment' or '#'">
                          <div id="wizard-step40" t-att-class="'progress-wizard-step %s' % (step == 40 and 'active' or step&gt;40 and 'complete' or 'disabled')">
                            <div class="progress-wizard-bar d-none d-md-block"/>
                            <span class="progress-wizard-dot d-none d-md-inline-block"></span>
                            <div class="text-center progress-wizard-steplabel">Confirm Order</div>
                          </div>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="extra_info" name="Checkout Extra Info">
        <t t-call="website.layout">
            <t t-set="no_footer" t-value="1"/>
            <div id="wrap">
                <div class="container oe_website_sale py-2">
                    <div class="row">
                        <div class="col-12">
                            <t t-call="website_sale.wizard_checkout">
                                <t t-set="step" t-value="30"/>
                            </t>
                        </div>
                        <div class="col-12 col-xl-auto order-xl-2 d-none d-xl-block">
                            <t t-call="website_sale.cart_summary">
                                <t t-set="redirect" t-valuef="/shop/extra_info"/>
                            </t>
                        </div>
                        <div class="col-12 col-xl order-xl-1 oe_cart">
                            <section class="s_website_form" data-vcss="001" data-snippet="s_website_form">
                                <div class="container">
                                    <form action="/website_form/" method="post" enctype="multipart/form-data" class="o_mark_required s_website_form_no_recaptcha" data-mark="*" data-force_action="shop.sale.order" data-model_name="sale.order" data-success-mode="redirect" data-success-page="/shop/payment" hide-change-model="true">
                                        <div class="s_website_form_rows row s_col_no_bgcolor">
                                            <div class="form-group col-12 s_website_form_field" data-type="char" data-name="Field">
                                                <div class="row s_col_no_resize s_col_no_bgcolor">
                                                    <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="sale1">
                                                        <span class="s_website_form_label_content">Your Reference</span>
                                                    </label>
                                                    <div class="col-sm">
                                                        <input id="sale1" type="text" class="form-control s_website_form_input" name="client_order_ref"/>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="form-group col-12 s_website_form_field s_website_form_custom" data-type="text" data-name="Field">
                                                <div class="row s_col_no_resize s_col_no_bgcolor">
                                                    <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="sale2">
                                                        <span class="s_website_form_label_content">Give us your feedback</span>
                                                    </label>
                                                    <div class="col-sm">
                                                        <textarea id="sale2" class="form-control s_website_form_input" name="Give us your feedback" />
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="form-group col-12 s_website_form_field s_website_form_custom" data-type="binary" data-name="Field">
                                                <div class="row s_col_no_resize s_col_no_bgcolor">
                                                    <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="sale3">
                                                        <span class="s_website_form_label_content">A document to provide</span>
                                                    </label>
                                                    <div class="col-sm">
                                                        <input id="sale3" type="file" class="form-control-file s_website_form_input" name="a_document" />
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="form-group col-12 s_website_form_submit s_website_form_no_submit_options">
                                                <div style="width: 200px;" class="s_website_form_label"/>
                                                <a role="button" href="/shop/checkout" class="btn btn-secondary float-left"><span class="fa fa-chevron-left"/> Previous</a>
                                                <a role="button" class="btn btn-primary float-right s_website_form_send" href="/shop/confirm_order">Next <span class="fa fa-chevron-right" /></a>
                                                <span id="s_website_form_result"></span>
                                            </div>
                                        </div>
                                    </form>
                                </div>
                            </section>
                        </div>
                    </div>
                </div>
            </div>
            <div class="oe_structure" id="oe_structure_website_sale_extra_info_1"/>
        </t>
    </template>

    <template id="extra_info_option" name="Extra Step Option" inherit_id="wizard_checkout" active="False" customize_show="True">
        <!-- Add a "flag element" to trigger style variation -->
        <xpath expr="//div[hasclass('wizard')]/div" position="before">
            <span class="o_wizard_has_extra_step d-none"/>
        </xpath>
        <xpath expr="//div[@id='wizard-step20']" position="after">
            <a class="no-decoration" t-att-href="step&gt;=30 and '/shop/extra_info' or '#'">
                <div id="wizard-step30" t-att-class="'progress-wizard-step %s' % (step == 30 and 'active' or step&gt;30 and 'complete' or 'disabled')">
                    <div class="progress-wizard-bar d-none d-md-block"/>
                    <span class="progress-wizard-dot d-none d-md-inline-block"></span>
                    <div class="text-center progress-wizard-steplabel">Extra Info</div>
                </div>
            </a>
        </xpath>
    </template>

    <!-- We use this template where we want to give the user a link to the product of a sale order line. -->
    <template id="cart_line_product_link" name="Shopping Cart Line Product Link">
        <a t-att-href="line.product_id.website_url">
            <t t-raw="0"/>
        </a>
    </template>

    <!-- This template displays all the lines following the first one on the description of the sale order line, with a muted style. For typical products this content will be the product description_sale. -->
    <template id="cart_line_description_following_lines" name="Shopping Cart Line Description Following Lines">
        <div t-if="line.get_description_following_lines()" t-attf-class="text-muted {{div_class}} small">
            <t t-foreach="line.get_description_following_lines()" t-as="name_line">
                <span><t t-esc="name_line"/></span>
                <br t-if="not name_line_last" />
            </t>
        </div>
    </template>

    <!-- This template is the one at the "Review order" step (the first one) on the checkout workflow. -->
    <template id="cart_lines" name="Shopping Cart Lines">
        <div t-if="not website_sale_order or not website_sale_order.website_order_line" class="js_cart_lines alert alert-info">
          Your cart is empty!
        </div>
        <table class="mb16 table table-striped table-sm js_cart_lines" id="cart_products" t-if="website_sale_order and website_sale_order.website_order_line">
            <thead>
                <tr>
                    <th class="td-img">Product</th>
                    <th></th>
                    <th class="text-center td-qty">Quantity</th>
                    <th class="text-center td-price">Price</th>
                    <th class="text-center td-action"></th>
                </tr>
            </thead>
            <tbody>
                <t t-foreach="website_sale_order.website_order_line" t-as="line">
                    <tr t-att-class="'optional_product info' if line.linked_line_id else None">
                        <td colspan="2" t-if="not line.product_id.product_tmpl_id" class='td-img'></td>
                        <td align="center" t-if="line.product_id.product_tmpl_id" class='td-img'>
                            <span t-field="line.product_id.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded o_image_64_max'}" />
                        </td>
                        <td t-if="line.product_id.product_tmpl_id" class='td-product_name'>
                            <div>
                                <t t-call="website_sale.cart_line_product_link">
                                    <strong t-field="line.name_short" />
                                </t>
                            </div>
                            <t t-call="website_sale.cart_line_description_following_lines">
                                <t t-set="div_class" t-value="'d-none d-md-block'"/>
                            </t>
                        </td>
                        <td class="text-center td-qty">
                            <div class="css_quantity input-group mx-auto">
                                <div class="input-group-prepend">
                                    <a t-attf-href="#" class="btn btn-link js_add_cart_json d-none d-md-inline-block" aria-label="Remove one" title="Remove one">
                                        <i class="fa fa-minus"></i>
                                    </a>
                                </div>
                                <input type="text" class="js_quantity form-control quantity" t-att-data-line-id="line.id" t-att-data-product-id="line.product_id.id" t-att-value="int(line.product_uom_qty) == line.product_uom_qty and int(line.product_uom_qty) or line.product_uom_qty" />
                                <div class="input-group-append">
                                    <a t-attf-href="#" class="btn btn-link float_left js_add_cart_json d-none d-md-inline-block" aria-label="Add one" title="Add one">
                                        <i class="fa fa-plus"></i>
                                    </a>
                                </div>
                            </div>
                        </td>
                        <td class="text-center td-price" name="price">
                            <t t-set="combination" t-value="line.product_id.product_template_attribute_value_ids + line.product_no_variant_attribute_value_ids"/>
                            <t t-set="combination_info" t-value="line.product_id.product_tmpl_id._get_combination_info(combination, pricelist=website_sale_order.pricelist_id)"/>

                            <t t-set="list_price_converted" t-value="website.currency_id._convert(combination_info['list_price'], website_sale_order.currency_id, website_sale_order.company_id, date)"/>
                            <t groups="account.group_show_line_subtotals_tax_excluded" t-if="(website_sale_order.pricelist_id.discount_policy == 'without_discount' and website_sale_order.currency_id.compare_amounts(list_price_converted, line.price_reduce_taxexcl) == 1) or website_sale_order.currency_id.compare_amounts(line.price_unit, line.price_reduce) == 1" name="order_line_discount">
                                <del t-attf-class="#{'text-danger mr8'}" style="white-space: nowrap;" t-esc="list_price_converted" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}" />
                            </t>
                            <span t-field="line.price_reduce_taxexcl" style="white-space: nowrap;" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}" groups="account.group_show_line_subtotals_tax_excluded" />
                            <t groups="account.group_show_line_subtotals_tax_included" t-if="(website_sale_order.pricelist_id.discount_policy == 'without_discount' and website_sale_order.currency_id.compare_amounts(list_price_converted, line.price_reduce_taxinc) == 1) or website_sale_order.currency_id.compare_amounts(line.price_unit, line.price_reduce) == 1" name="order_line_discount">
                                <del t-attf-class="#{'text-danger mr8'}" style="white-space: nowrap;" t-esc="list_price_converted" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}" />
                            </t>
                            <span t-field="line.price_reduce_taxinc" style="white-space: nowrap;" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}" groups="account.group_show_line_subtotals_tax_included" />
                        </td>
                        <td class="td-action">
                            <a href='#' aria-label="Remove from cart" title="Remove from cart" class='js_delete_product no-decoration'> <small><i class='fa fa-trash-o'></i></small></a>
                        </td>
                    </tr>
                </t>
            </tbody>
        </table>

    </template>

    <template id="cart" name="Shopping Cart">
        <t t-call="website.layout">
            <div id="wrap">
                <div class="container oe_website_sale py-2">
                    <div class="row">
                        <div class="col-12">
                            <t t-call="website_sale.wizard_checkout">
                                <t t-set="step" t-value="10" />
                            </t>
                        </div>
                        <div class="col-12 col-xl-8 oe_cart">
                            <div class="row">
                                <div class="col-lg-12">
                                    <div t-if="abandoned_proceed or access_token" class="mt8 mb8 alert alert-info" role="alert"> <!-- abandoned cart choices -->
                                        <t t-if="abandoned_proceed">
                                            <p>Your previous cart has already been completed.</p>
                                            <p t-if="website_sale_order">Please proceed your current cart.</p>
                                        </t>
                                        <t t-if="access_token">
                                            <p>This is your current cart.</p>
                                            <p>
                                                <strong><a t-attf-href="/shop/cart/?access_token=#{access_token}&amp;revive=squash">Click here</a></strong> if you want to restore your previous cart. Your current cart will be replaced with your previous cart.</p>
                                            <p>
                                                <strong><a t-attf-href="/shop/cart/?access_token=#{access_token}&amp;revive=merge">Click here</a></strong> if you want to merge your previous cart into current cart.
                                            </p>
                                        </t>
                                    </div>
                                    <t t-call="website_sale.cart_lines" />
                                    <div class="clearfix" />
                                    <a role="button" href="/shop" class="btn btn-secondary mb32 d-none d-xl-inline-block">
                                        <span class="fa fa-chevron-left" />
                                        <span class="">Continue Shopping</span>
                                    </a>
                                    <a role="button" t-if="website_sale_order and website_sale_order.website_order_line" class="btn btn-primary float-right d-none d-xl-inline-block" href="/shop/checkout?express=1">
                                        <span class="">Process Checkout</span>
                                        <span class="fa fa-chevron-right" />
                                    </a>
                                    <div class="oe_structure" id="oe_structure_website_sale_cart_1"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-xl-4" id="o_cart_summary">
                            <t t-call="website_sale.short_cart_summary"/>
                            <div class="d-xl-none mt8">
                                <a role="button" href="/shop" class="btn btn-secondary mb32">
                                    <span class="fa fa-chevron-left" />
                                    Continue<span class="d-none d-md-inline"> Shopping</span>
                                </a>
                                <a role="button" t-if="website_sale_order and website_sale_order.website_order_line" class="btn btn-primary float-right" href="/shop/checkout?express=1">
                                    <span class="">Process Checkout</span>
                                    <span class="fa fa-chevron-right" />
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="oe_structure" id="oe_structure_website_sale_cart_2"/>
            </div>
        </t>
    </template>

    <!-- this template is the one when we mouse over "My Cart" on the top right -->
    <template id="cart_popover" name="Cart Popover">
        <div t-if="not website_sale_order or not website_sale_order.website_order_line" class="alert alert-info">
          Your cart is empty!
        </div>
        <t t-if="website_sale_order and website_sale_order.website_order_line">
            <t t-foreach="website_sale_order.website_order_line" t-as="line">
                <div class="row mb8 cart_line">
                    <div class="col-3 text-center">
                        <span t-field="line.product_id.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded o_image_64_max mb-2'}" />
                    </div>
                    <div class="col-9">
                        <div>
                            <t t-call="website_sale.cart_line_product_link">
                                <span class="h6" t-esc="line.name_short" />
                            </t>
                        </div>
                        Qty: <t t-esc="int(line.product_uom_qty) == line.product_uom_qty and int(line.product_uom_qty) or line.product_uom_qty" />
                    </div>
                </div>
            </t>
            <div class="text-center">
                <span class="h6">
                    <t t-call="website_sale.total">
                        <t t-set="hide_coupon" t-value="True"/>
                    </t>
                </span>
                <a role="button" class="btn btn-primary" href="/shop/cart">
                       View Cart (<t t-esc="website_sale_order.cart_quantity" /> item(s))
                     </a>
            </div>
        </t>
    </template>

    <template id="suggested_products_list" inherit_id="website_sale.cart_lines" customize_show="True" name="Accessory Products in my cart">
        <xpath expr="//table[@id='cart_products']" position="after">
            <h5 class='text-muted js_cart_lines' t-if="suggested_products">Suggested Accessories:</h5>
            <table t-if="suggested_products" id="suggested_products" class="js_cart_lines table table-striped table-sm">
                <tbody>
                    <tr t-foreach="suggested_products" t-as="product">
                        <t t-set="combination_info" t-value="product._get_combination_info_variant(pricelist=website_sale_order.pricelist_id)"/>
                        <td class='td-img text-center'>
                            <a t-att-href="product.website_url">
                                <span t-field="product.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded o_image_64_max'}" />
                            </a>
                        </td>
                        <td class='td-product_name'>
                            <div>
                                <a t-att-href="product.website_url">
                                    <strong t-esc="combination_info['display_name']" />
                                </a>
                            </div>
                            <div class="text-muted d-none d-md-block" t-field="product.description_sale" />
                        </td>
                        <td class='td-price'>
                            <del t-attf-class="text-danger mr8 {{'' if combination_info['has_discounted_price'] else 'd-none'}}" style="white-space: nowrap;" t-esc="combination_info['list_price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                            <span t-esc="combination_info['price']" style="white-space: nowrap;" t-options="{'widget': 'monetary','display_currency': website.currency_id}"/>
                        </td>
                        <td class="w-25 text-center">
                            <input class="js_quantity" name="product_id" t-att-data-product-id="product.id" type="hidden" />
                            <a role="button" class="btn btn-link js_add_suggested_products">
                                <strong>Add to Cart</strong>
                            </a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </xpath>
    </template>

    <template id='coupon_form' name='Coupon form'>
        <form t-att-action="'/shop/pricelist%s' % (redirect and '?r=' + redirect or '')"
            method="post" name="coupon_code">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
            <div class="input-group w-100">
                <input name="promo" class="form-control" type="text" placeholder="code..." t-att-value="website_sale_order.pricelist_id.code or None"/>
                <div class="input-group-append">
                    <a href="#" role="button" class="btn btn-secondary a-submit">Apply</a>
                </div>
            </div>
        </form>
        <t t-if="request.params.get('code_not_available')" name="code_not_available">
            <div class="alert alert-danger text-left" role="alert">This promo code is not available.</div>
        </t>
    </template>

    <template id="checkout">
        <t t-call="website.layout">
            <t t-set="additional_title">Shop - Checkout</t>
            <t t-set="no_footer" t-value="1"/>
            <div id="wrap">
                <div class="container oe_website_sale py-2">
                    <t t-set="same_shipping" t-value="bool(order.partner_shipping_id==order.partner_id or only_services)" />
                    <div class="row">
                        <div class="col-12">
                            <t t-call="website_sale.wizard_checkout">
                                <t t-set="step" t-value="20" />
                            </t>
                        </div>
                        <div class="col-12 col-xl-auto order-xl-2 d-none d-xl-block">
                            <t t-call="website_sale.cart_summary">
                                <t t-set="redirect" t-valuef="/shop/checkout"/>
                            </t>
                        </div>
                        <div class="col-12 col-xl order-xl-1 oe_cart">
                            <div class="row">
                                <div class="col-lg-12">
                                    <h3 class="o_page_header mt8">Billing Address</h3>
                                </div>
                                <div class="col-lg-6 one_kanban">
                                    <t t-call="website_sale.address_kanban">
                                        <t t-set='contact' t-value="order.partner_id"/>
                                        <t t-set='selected' t-value="1"/>
                                        <t t-set='readonly' t-value="1"/>
                                    </t>
                                </div>
                            </div>
                            <t t-if="not only_services" groups="sale.group_delivery_invoice_address">
                                <div class="row">
                                    <div class="col-lg-12">
                                        <h3 class="o_page_header mt16 mb4">Shipping Address</h3>
                                    </div>
                                </div>
                                <div class="row all_shipping">
                                    <div class="col-lg-12">
                                        <div class="row mt8">
                                            <div class="col-md-12 col-lg-12 one_kanban">
                                                <form action="/shop/address" method="post" class=''>
                                                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                                                    <a role="button" href="#" class='a-submit btn btn-secondary mb16 btn-block'>
                                                        <i class="fa fa-plus-square"/>
                                                        <span>Add an address</span>
                                                    </a>
                                                </form>
                                            </div>
                                            <t t-foreach="shippings" t-as="ship">
                                                <div class="col-md-12 col-lg-6 one_kanban">
                                                    <t t-call="website_sale.address_kanban">
                                                        <t t-set="actual_partner" t-value="order.partner_id" />
                                                        <t t-set='contact' t-value="ship"/>
                                                        <t t-set='selected' t-value="order.partner_shipping_id==ship"/>
                                                        <t t-set='readonly' t-value="bool(len(shippings)==1)"/>
                                                        <t t-set='edit_billing' t-value="bool(ship==order.partner_id)"/>
                                                    </t>
                                                </div>
                                            </t>
                                        </div>
                                    </div>
                                </div>
                            </t>
                            <div class="d-flex justify-content-between mt-3">
                                <a role="button" href="/shop/cart" class="btn btn-secondary mb32">
                                    <i class="fa fa-chevron-left"/>
                                    <span>Return to Cart</span>
                                </a>
                                <a role="button" href="/shop/confirm_order" class="btn btn-primary mb32">
                                    <span>Confirm</span>
                                    <i class="fa fa-chevron-right"/>
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="address_kanban" name="Kanban address">
            <form action="/shop/checkout" method="POST" class="d-none">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                <input type="hidden" name="partner_id" t-att-value="contact.id" />
                <t t-if='edit_billing'>
                    <input type="hidden" name="callback" value="/shop/checkout?use_billing" />
                </t>
                <input type='submit'/>
            </form>
            <div t-attf-class="card #{selected and 'border border-primary' or 'js_change_shipping'}">
                <div class='card-body' style='min-height: 130px;'>
                    <a t-if="not actual_partner or (ship.id in actual_partner.child_ids.ids)" href="#" class="btn btn-link float-right p-0 js_edit_address no-decoration" role="button" title="Edit this address" aria-label="Edit this address"><i class='fa fa-edit'/></a>
                    <t t-esc="contact" t-options="dict(widget='contact', fields=['name', 'address'], no_marker=True)"/>
                </div>
                <div class='card-footer' t-if='not readonly'>
                    <span class='btn-ship' t-att-style="'' if selected else 'display:none;'">
                        <a role="button" href='#' class="btn btn-block btn-primary">
                            <i class='fa fa-check'></i> Ship to this address
                        </a>
                    </span>
                    <span class='btn-ship' t-att-style="'' if not selected else 'display:none;'">
                        <a role="button" href='#' class="btn btn-block btn-secondary">
                            Select this address
                        </a>
                    </span>
                </div>
            </div>
    </template>

    <template id="address" name="Address Management">
        <t t-set="no_footer" t-value="1"/>
        <t t-call="website.layout">
            <div id="wrap">
                <div class="container oe_website_sale py-2">
                    <div class="row">
                        <div class="col-12">
                            <t t-call="website_sale.wizard_checkout">
                                <t t-set="step" t-value="20" />
                            </t>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-12 col-xl-auto order-xl-2 d-none d-xl-block">
                            <t t-call="website_sale.cart_summary">
                                <t t-set="hide_coupon">True</t>
                                <t t-set="redirect" t-valuef="/shop/address"/>
                            </t>
                        </div>
                        <div class="col-12 col-xl order-xl-1 oe_cart">
                            <div>
                                <t t-if="mode == ('new', 'billing')">
                                    <h2 class="o_page_header mt8">Your Address
                                        <small> or </small>
                                        <a role="button" href='/web/login?redirect=/shop/checkout' class='btn btn-primary' style="margin-top: -11px">Log In</a>
                                    </h2>
                                </t>
                                <t t-if="mode == ('edit', 'billing')">
                                    <h2 class="o_page_header mt8">Your Address</h2>
                                </t>
                                <t t-if="mode[1] == 'shipping'">
                                    <h2 class="o_page_header mt8">Shipping Address </h2>
                                </t>
                                <t t-if="partner_id == website_sale_order.partner_shipping_id.id == website_sale_order.partner_invoice_id.id">
                                    <div class="alert alert-warning" role="alert" t-if="not only_services">
                                        <h4 class="alert-heading">Be aware!</h4>
                                        <p  groups="sale.group_delivery_invoice_address">
                                            You are editing your <b>billing and shipping</b> addresses at the same time!<br/>
                                            If you want to modify your shipping address, create a <a href='/shop/address'>new address</a>.
                                        </p>
                                    </div>
                                </t>
                                <t t-if="error" t-foreach="error.get('error_message', [])" t-as="err">
                                    <h5 class="text-danger" t-esc="err" />
                                </t>
                                <form action="/shop/address" method="post" class="checkout_autoformat">
                                    <div class="form-row">
                                        <div t-attf-class="form-group #{error.get('name') and 'o_has_error' or ''} col-lg-12 div_name">
                                            <label class="col-form-label" for="name">Name</label>
                                            <input type="text" name="name" t-attf-class="form-control #{error.get('name') and 'is-invalid' or ''}" t-att-value="'name' in checkout and checkout['name']" />
                                        </div>
                                        <div class="w-100"/>
                                        <t t-if="mode[1] == 'billing'">
                                            <div t-attf-class="form-group #{error.get('email') and 'o_has_error' or ''} col-lg-6" id="div_email">
                                                <label class="col-form-label" for="email">Email</label>
                                                <input type="email" name="email" t-attf-class="form-control #{error.get('email') and 'is-invalid' or ''}" t-att-value="'email' in checkout and checkout['email']" />
                                            </div>
                                        </t>
                                        <div t-attf-class="form-group #{error.get('phone') and 'o_has_error' or ''} col-lg-6" id="div_phone">
                                            <label class="col-form-label" for="phone">Phone</label>
                                            <input type="tel" name="phone" t-attf-class="form-control #{error.get('phone') and 'is-invalid' or ''}" t-att-value="'phone' in checkout and checkout['phone']" />
                                        </div>
                                        <div class="w-100"/>
                                        <div t-attf-class="form-group #{error.get('street') and 'o_has_error' or ''} col-lg-12 div_street">
                                            <label class="col-form-label" for="street">Street <span class="d-none d-md-inline"> and Number</span></label>
                                            <input type="text" name="street" t-attf-class="form-control #{error.get('street') and 'is-invalid' or ''}" t-att-value="'street' in checkout and checkout['street']" />
                                        </div>
                                        <div t-attf-class="form-group #{error.get('street2') and 'o_has_error' or ''} col-lg-12 div_street2">
                                            <label class="col-form-label label-optional" for="street2">Street 2</label>
                                            <input type="text" name="street2" t-attf-class="form-control #{error.get('street2') and 'is-invalid' or ''}" t-att-value="'street2' in checkout and checkout['street2']" />
                                        </div>
                                        <div class="w-100"/>
                                        <t t-set='zip_city' t-value='country and [x for x in country.get_address_fields() if x in ["zip", "city"]] or ["city", "zip"]'/>
                                        <t t-if="'zip' in zip_city and zip_city.index('zip') &lt; zip_city.index('city')">
                                            <div t-attf-class="form-group #{error.get('zip') and 'o_has_error' or ''} col-md-4 div_zip">
                                                <label class="col-form-label label-optional" for="zip">Zip Code</label>
                                                <input type="text" name="zip" t-attf-class="form-control #{error.get('zip') and 'is-invalid' or ''}" t-att-value="'zip' in checkout and checkout['zip']" />
                                            </div>
                                        </t>
                                        <div t-attf-class="form-group #{error.get('city') and 'o_has_error' or ''} col-md-8 div_city">
                                            <label class="col-form-label" for="city">City</label>
                                            <input type="text" name="city" t-attf-class="form-control #{error.get('city') and 'is-invalid' or ''}" t-att-value="'city' in checkout and checkout['city']" />
                                        </div>
                                        <t t-if="'zip' in zip_city and zip_city.index('zip') &gt; zip_city.index('city')">
                                            <div t-attf-class="form-group #{error.get('zip') and 'o_has_error' or ''} col-md-4 div_zip">
                                                <label class="col-form-label label-optional" for="zip">Zip Code</label>
                                                <input type="text" name="zip" t-attf-class="form-control #{error.get('zip') and 'is-invalid' or ''}" t-att-value="'zip' in checkout and checkout['zip']" />
                                            </div>
                                        </t>
                                        <div class="w-100"/>
                                        <div t-attf-class="form-group #{error.get('country_id') and 'o_has_error' or ''} col-lg-6 div_country">
                                            <label class="col-form-label" for="country_id">Country</label>
                                            <select id="country_id" name="country_id" t-attf-class="form-control #{error.get('country_id') and 'is-invalid' or ''}" t-att-mode="mode[1]">
                                                <option value="">Country...</option>
                                                <t t-foreach="countries" t-as="c">
                                                    <option t-att-value="c.id" t-att-selected="c.id == (country and country.id or -1)">
                                                        <t t-esc="c.name" />
                                                    </option>
                                                </t>
                                            </select>
                                        </div>
                                        <div t-attf-class="form-group #{error.get('state_id') and 'o_has_error' or ''} col-lg-6 div_state" t-att-style="(not country or not country.state_ids) and 'display: none'">
                                            <label class="col-form-label" for="state_id">State / Province</label>
                                            <select name="state_id" t-attf-class="form-control #{error.get('state_id') and 'is-invalid' or ''}" data-init="1">
                                                <option value="">State / Province...</option>
                                                <t t-foreach="country_states" t-as="s">
                                                    <option t-att-value="s.id" t-att-selected="s.id == ('state_id' in checkout and country and checkout['state_id'] != '' and int(checkout['state_id']))">
                                                        <t t-esc="s.name" />
                                                    </option>
                                                </t>
                                            </select>
                                        </div>
                                        <div class="w-100"/>
                                        <t t-if="mode == ('new', 'billing') and not only_services">
                                            <div class="col-lg-12">
                                                <div class="checkbox">
                                                  <label>
                                                    <input type="checkbox" id='shipping_use_same' class="mr8" name='use_same' value="1" checked='checked'/>Ship to the same address
                                                    <span class='ship_to_other text-muted' style="display: none">&amp;nbsp;(<i>Your shipping address will be requested later) </i></span>
                                                    </label>
                                                </div>
                                            </div>
                                        </t>
                                    </div>

                                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                                    <input type="hidden" name="submitted" value="1" />
                                    <input type="hidden" name="partner_id" t-att-value="partner_id or '0'" />
                                    <input type="hidden" name="callback" t-att-value="callback" />
                                    <!-- Example -->
                                    <input type="hidden" name="field_required" t-att-value="'phone,name'" />

                                    <div class="d-flex justify-content-between">
                                        <a role="button" t-att-href="mode == ('new', 'billing') and '/shop/cart' or '/shop/checkout'" class="btn btn-secondary mb32">
                                            <i class="fa fa-chevron-left"/>
                                            <span>Back</span>
                                        </a>
                                        <a role="button" href="#" class="btn btn-primary mb32 a-submit a-submit-disable a-submit-loading">
                                            <span>Next</span>
                                            <i class="fa fa-chevron-right"/>
                                        </a>
                                    </div>
                                </form>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="address_b2b" inherit_id="address" name="Show b2b fields" customize_show="True">
        <xpath expr="//div[@id='div_phone']" position="after">
            <div class="w-100"/>
            <t t-if="mode == ('new', 'billing') or (mode == ('edit', 'billing') and (can_edit_vat or 'vat' in checkout and checkout['vat']))">
                <div t-attf-class="form-group #{error.get('company_name') and 'o_has_error' or ''} col-lg-6">
                    <label class="col-form-label font-weight-normal label-optional" for="company_name">Company Name</label>
                    <t t-set="company_name_not_editable_message">Changing company name is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</t>
                    <input type="text" name="company_name" t-attf-class="form-control #{error.get('company_name') and 'is-invalid' or ''}" t-att-value="'commercial_company_name' in checkout and checkout['commercial_company_name'] or 'company_name' in checkout and checkout['company_name']" t-att-readonly="'1' if 'vat' in checkout and checkout['vat'] and not can_edit_vat else None" t-att-title="company_name_not_editable_message if 'vat' in checkout and checkout['vat'] and not can_edit_vat else None" />
                </div>
                <div t-attf-class="form-group #{error.get('vat') and 'o_has_error' or ''} col-lg-6 div_vat">
                    <label class="col-form-label font-weight-normal label-optional" for="vat">VAT</label>
                    <t t-set="vat_not_editable_message">Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</t>
                    <input type="text" name="vat" t-attf-class="form-control #{error.get('vat') and 'is-invalid' or ''}" t-att-value="'vat' in checkout and checkout['vat']" t-att-readonly="'1' if 'vat' in checkout and checkout['vat'] and not can_edit_vat else None" t-att-title="vat_not_editable_message if 'vat' in checkout and checkout['vat'] and not can_edit_vat else None" />
                </div>
            </t>
        </xpath>
    </template>

    <template id="payment" name="Payment">
        <t t-call="website.layout">
            <t t-set="additional_title">Shop - Select Payment Acquirer</t>
            <t t-set="no_footer" t-value="1"/>

            <div id="wrap">
                <div class="container oe_website_sale py-2">
                    <div class="row">
                        <div class='col-12'>
                            <t t-call="website_sale.wizard_checkout">
                                <t t-set="step" t-value="40" />
                            </t>
                        </div>
                        <div class="col-12" t-if="errors">
                            <t t-foreach="errors" t-as="error">
                                <div class="alert alert-danger" t-if="error" role="alert">
                                    <h4>
                                        <t t-esc="error[0]" />
                                    </h4>
                                    <t t-esc="error[1]" />
                                </div>
                            </t>
                        </div>
                        <div class="col-12 col-xl-auto order-xl-2">
                            <t t-call="website_sale.cart_summary"/>
                        </div>
                        <div class="col-12 col-xl order-xl-1 oe_cart">
                            <div class="card">
                                <div class="card-body" id="shipping_and_billing">
                                    <a class='float-right no-decoration' href='/shop/checkout'><i class="fa fa-edit"/> Edit</a>
                                    <t t-set="same_shipping" t-value="bool(order.partner_shipping_id==order.partner_id or only_services)" />
                                    <div><b>Billing<t t-if="same_shipping and not only_services"> &amp; Shipping</t>: </b><span t-esc='order.partner_id' t-options="dict(widget='contact', fields=['address'], no_marker=True, separator=', ')" class="address-inline"/></div>
                                    <div t-if="not same_shipping and not only_services" groups="sale.group_delivery_invoice_address"><b>Shipping: </b><span t-esc='order.partner_shipping_id' t-options="dict(widget='contact', fields=['address'], no_marker=True, separator=', ')"  class="address-inline"/></div>
                                </div>
                            </div>

                            <div class="oe_structure clearfix mt-3" id="oe_structure_website_sale_payment_1"/>

                            <div id="payment_method" class="mt-3" t-if="(acquirers or tokens) and website_sale_order.amount_total">
                                <h3 class="mb24">Pay with </h3>
                                <t t-call="payment.payment_tokens_list">
                                    <t t-set="mode" t-value="'payment'"/>
                                    <t t-set="submit_txt">Pay Now</t>
                                    <t t-set="icon_right" t-value="1"/>
                                    <t t-set="icon_class" t-value="'fa-chevron-right'"/>
                                    <t t-set="submit_class" t-value="'btn btn-primary'"/>
                                    <t t-set="pms" t-value="tokens"/>
                                    <t t-set="form_action" t-value="'/shop/payment/token'"/>
                                    <t t-set="prepare_tx_url" t-value="'/shop/payment/transaction/'"/>
                                    <t t-set="partner_id" t-value="partner"/>

                                    <t t-set="back_button_icon_class" t-value="'fa-chevron-left'"/>
                                    <t t-set="back_button_txt">Return to Cart</t>
                                    <t t-set="back_button_class" t-value="'btn btn-secondary'"/>
                                    <t t-set="back_button_link" t-value="'/shop/cart'"/>
                                </t>
                            </div>
                            <div t-if="not (acquirers or tokens)" class="alert alert-warning">
                                <strong>No suitable payment option could be found.</strong><br/>
                                If you believe that it is an error, please contact the website administrator.
                            </div>

                            <div t-if="not acquirers" class="mt-2">
                                <a role="button" class="btn-link"
                                    groups="base.group_system"
                                    t-attf-href="/web#action=#{payment_action_id}">
                                        <i class="fa fa-arrow-right"></i> Add payment acquirers
                                </a>
                            </div>
                            <div class="js_payment mt-3" t-if="not website_sale_order.amount_total" id="payment_method">
                                <form target="_self" action="/shop/payment/validate" method="post" class="float-right">
                                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                                    <a role="button" class="btn btn-primary a-submit" href="#">
                                        <span t-if="order.amount_total &gt; 0">Pay Now <span class="fa fa-chevron-right"></span></span>
                                        <span t-if="order.amount_total == 0">Confirm Order <span class="fa fa-chevron-right"></span></span>
                                    </a>
                                </form>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="oe_structure" id="oe_structure_website_sale_payment_2"/>
            </div>
        </t>
    </template>


    <template id="short_cart_summary" name="Short Cart right column">
        <div class="card js_cart_summary" t-if="website_sale_order and website_sale_order.website_order_line" >
            <div class="card-body">
                <h4 class="d-none d-xl-block">Order Total</h4>
                <hr class="d-none d-xl-block"/>
                <div>
                    <t t-call="website_sale.total">
                        <t t-set="no_rowspan" t-value="1"/>
                    </t>
                    <a role="button" t-if="website_sale_order and website_sale_order.website_order_line" class="btn btn-secondary float-right d-none d-xl-inline-block" href="/shop/checkout?express=1">
                        <span>Process Checkout</span>
                    </a>
                </div>
            </div>
        </div>
    </template>

    <!-- This template is the one present on the right during the payment process.
        Here it is important to not show too much information to the user, because we want him to pay!
        We shouldn't display link to products or long descriptions.
    -->
    <template id="cart_summary" name="Cart right column">
        <div class="card">
            <div class="card-body p-xl-0">
                <div class="toggle_summary d-xl-none">
                    <b>Your order: </b> <span id="amount_total_summary" class="monetary_field" t-field="website_sale_order.amount_total" t-options='{"widget": "monetary", "display_currency": website_sale_order.pricelist_id.currency_id}'/>
                    <span class='fa fa-chevron-down fa-border float-right' role="img" aria-label="Details" title="Details"></span>
                </div>
                <div t-if="not website_sale_order or not website_sale_order.website_order_line" class="alert alert-info">
                    Your cart is empty!
                </div>
                <div class="toggle_summary_div d-none d-xl-block">
                    <table class="table table-striped table-sm" id="cart_products" t-if="website_sale_order and website_sale_order.website_order_line">
                        <thead>
                            <tr>
                                <th class="border-top-0 td-img">Product</th>
                                <th class="border-top-0"></th>
                                <th class="border-top-0 td-qty">Quantity</th>
                                <th class="border-top-0 text-center td-price">Price</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr t-foreach="website_sale_order.website_order_line" t-as="line">
                                <td class='' colspan="2" t-if="not line.product_id.product_tmpl_id"></td>
                                <td class='td-img text-center' t-if="line.product_id.product_tmpl_id">
                                    <span t-field="line.product_id.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'rounded o_image_64_max'}" />
                                </td>
                                <td class='td-product_name' t-if="line.product_id.product_tmpl_id">
                                    <div>
                                        <strong t-field="line.name_short" />
                                    </div>
                                </td>
                                <td class='td-qty'>
                                    <div t-esc="line.product_uom_qty" />
                                </td>
                                <td class="text-center td-price">
                                    <span t-field="line.price_reduce_taxexcl" style="white-space: nowrap;" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}" groups="account.group_show_line_subtotals_tax_excluded" />
                                    <span t-field="line.price_reduce_taxinc" style="white-space: nowrap;" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}" groups="account.group_show_line_subtotals_tax_included" />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                    <t t-call="website_sale.total">
                        <t t-set='redirect' t-value="redirect or '/shop/payment'"></t>
                    </t>
                </div>
            </div>
        </div>
    </template>

    <template id="payment_sale_note" inherit_id="payment" name="Accept Terms &amp; Conditions" customize_show="True" active="False">
        <xpath expr="//div[@id='payment_method'][hasclass('js_payment')]" position="after">
            <div class="custom-control custom-checkbox float-right mt-2 oe_accept_cgv_button">
                <input type="checkbox" id="checkbox_cgv" class="custom-control-input"/>
                <label for="checkbox_cgv" class="custom-control-label">
                    I agree to the <a target="_BLANK" href="/shop/terms">terms &amp; conditions</a>
                </label>
            </div>
        </xpath>
    </template>

    <template id="confirmation">
        <t t-call="website.layout">
            <t t-set="additional_title">Shop - Confirmed</t>
            <div id="wrap">
                <div class="container oe_website_sale py-2">
                    <h1><span>Order</span> <em t-field="order.name" /> <t t-if="order.state == 'sale'"><span>Confirmed</span></t></h1>

                    <div class="row">
                        <div class="col-12 col-xl">
                            <div class="oe_cart">
                                <t t-set="payment_tx_id" t-value="order.get_portal_last_transaction()"/>
                                <t t-if="payment_tx_id.state == 'done'">
                                    <div class="thanks_msg">
                                        <h2>Thank you for your order.
                                            <a role="button" class="btn btn-primary d-none d-md-inline-block" href="/shop/print" target="_blank" aria-label="Print" title="Print"><i class="fa fa-print"></i> Print</a>
                                        </h2>
                                    </div>
                                </t>
                                <t t-if="request.env['res.users']._get_signup_invitation_scope() == 'b2c' and request.website.is_public_user()">
                                    <p class="alert alert-info mt-3" role="status">
                                        <a role="button" t-att-href='order.partner_id.signup_prepare() and order.partner_id.with_context(relative_url=True).signup_url' class='btn btn-primary'>Sign Up</a>
                                         to follow your order.
                                    </p>
                                </t>
                                <div class="oe_structure clearfix mt-3" id="oe_structure_website_sale_confirmation_1"/>
                                <h3 class="text-left mt-3">
                                    <strong>Payment Information:</strong>
                                </h3>
                                <table class="table">
                                    <tbody>
                                        <tr>
                                            <td colspan="2">
                                                <t t-set="last_transaction" t-value="order.get_portal_last_transaction()"/>
                                                <t t-esc="last_transaction.acquirer_id.display_as or last_transaction.acquirer_id.name" />
                                            </td>
                                            <td class="text-right" width="100">
                                                <strong>Total:</strong>
                                            </td>
                                            <td class="text-right" width="100">
                                                <strong t-field="last_transaction.amount" t-options="{'widget': 'monetary', 'display_currency': last_transaction.currency_id}" />
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                                <t t-call="website_sale.payment_confirmation_status"/>
                                <div class="card mt-3">
                                  <div class="card-body">
                                    <t t-set="same_shipping" t-value="bool(order.partner_shipping_id==order.partner_id or only_services)" />
                                    <div><b>Billing <t t-if="same_shipping and not only_services"> &amp; Shipping</t>: </b><span t-esc='order.partner_id' t-options="dict(widget='contact', fields=['address'], no_marker=True, separator=', ')" class="address-inline"/></div>
                                    <div t-if="not same_shipping and not only_services" groups="sale.group_delivery_invoice_address"><b>Shipping: </b><span t-esc='order.partner_shipping_id' t-options="dict(widget='contact', fields=['address'], no_marker=True, separator=', ')"  class="address-inline"/></div>
                                  </div>
                                </div>
                                <div class="oe_structure mt-3" id="oe_structure_website_sale_confirmation_2"/>
                            </div>
                        </div>
                        <div class="col-12 col-xl-auto">
                            <t t-set="website_sale_order" t-value="order"/>
                            <t t-call="website_sale.cart_summary">
                                <t t-set="hide_coupon" t-value="1"/>
                            </t>
                        </div>
                    </div>
                </div>
                <div class="oe_structure" id="oe_structure_website_sale_confirmation_3"/>
            </div>
        </t>
    </template>

    <template id="total">
        <div id="cart_total" t-att-class="extra_class or ''" t-if="website_sale_order and website_sale_order.website_order_line">
            <table class="table">
                  <tr id="empty">
                      <t t-if='not no_rowspan'><td rowspan="10" class="border-0"/></t>
                      <td class="col-md-2 col-3 border-0"></td>
                      <td class="col-md-2 col-3 border-0" ></td>
                  </tr>
                  <tr id="order_total_untaxed">
                      <td class="text-right border-0">Subtotal:</td>
                      <td class="text-xl-right border-0" >
                          <span t-field="website_sale_order.amount_untaxed" class="monetary_field" style="white-space: nowrap;" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}"/>
                      </td>
                  </tr>
                  <tr id="order_total_taxes">
                      <td class="text-right border-0">Taxes:</td>
                      <td class="text-xl-right border-0">
                           <span t-field="website_sale_order.amount_tax" class="monetary_field" style="white-space: nowrap;" t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}" />
                      </td>
                  </tr>
                  <tr id="order_total">
                      <td class="text-right"><strong>Total:</strong></td>
                      <td class="text-xl-right">
                          <strong t-field="website_sale_order.amount_total" class="monetary_field"
                              t-options='{"widget": "monetary", "display_currency": website_sale_order.pricelist_id.currency_id}'/>
                      </td>
                  </tr>
            </table>
        </div>
    </template>

    <template id="reduction_code" inherit_id="website_sale.total" customize_show="True" name="Promo Code">
        <xpath expr="//div[@id='cart_total']//table/tr[last()]" position="after">
            <tr t-if="not hide_coupon">
                <td colspan="3" class="text-center text-xl-right border-0">
                <span class=''>
                    <t t-set='force_coupon' t-value="website_sale_order.pricelist_id.code or request.params.get('code_not_available')"/>
                    <t t-if="not force_coupon">
                        <a href="#" class="show_coupon">I have a promo code</a>
                    </t>
                    <div t-attf-class="coupon_form #{not force_coupon and 'd-none'}">
                        <t t-call="website_sale.coupon_form"/>
                    </div>
                </span>
                </td>
            </tr>
        </xpath>
    </template>

    <template id="payment_confirmation_status">
        <div class="oe_website_sale_tx_status mt-3" t-att-data-order-id="order.id">
            <t t-set="payment_tx_id" t-value="order.get_portal_last_transaction()"/>
            <div t-attf-class="card #{
                (payment_tx_id.state == 'pending' and 'bg-info') or
                (payment_tx_id.state == 'done' and 'alert-success') or
                (payment_tx_id.state == 'authorized' and 'alert-success') or
                'bg-danger'}">
                <div class="card-header">
                    <a role="button" groups="base.group_system" class="btn btn-sm btn-link text-white float-right" target="_blank" aria-label="Edit" title="Edit"
                            t-att-href="'/web#model=%s&amp;id=%s&amp;action=%s&amp;view_type=form' % ('payment.acquirer', payment_tx_id.acquirer_id.id, 'payment.action_payment_acquirer')">
                        <i class="fa fa-pencil"></i>
                    </a>
                    <t t-if="payment_tx_id.state == 'pending'">
                        <t t-raw="payment_tx_id.acquirer_id.pending_msg"/>
                    </t>
                    <t t-if="payment_tx_id.state == 'done'">
                        <span t-if='payment_tx_id.acquirer_id.done_msg' t-raw="payment_tx_id.acquirer_id.done_msg"/>
                    </t>
                    <t t-if="payment_tx_id.state == 'cancel'">
                        <t t-raw="payment_tx_id.acquirer_id.cancel_msg"/>
                    </t>
                    <t t-if="payment_tx_id.state == 'authorized'">
                        <span>Your payment has been authorized.</span>
                    </t>
                    <t t-if="payment_tx_id.state == 'error'">
                        <span t-esc="payment_tx_id.state_message"/>
                    </t>
                </div>
                <div t-if="payment_tx_id.acquirer_id.provider == 'transfer' and order.reference" class="card-body">
                    <b>Communication: </b><span t-esc='order.reference'/>
                </div>

                <div t-if="payment_tx_id.acquirer_id.qr_code and (payment_tx_id.acquirer_id.provider == 'transfer')">
                    <t t-set="qr_code" t-value="payment_tx_id.acquirer_id.journal_id.bank_account_id.build_qr_code_base64(order.amount_total,payment_tx_id.reference, None, payment_tx_id.currency_id, payment_tx_id.partner_id)"/>
                    <div class="card-body" t-if="qr_code">
                        <h3>Or scan me with your banking app.</h3>
                        <img class="border border-dark rounded" t-att-src="qr_code"/>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="website_sale.brand_promotion" inherit_id="website.brand_promotion">
        <xpath expr="//t[@t-call='web.brand_promotion_message']" position="replace">
            <t t-call="web.brand_promotion_message">
                <t t-set="_message">
                    The #1 <a target="_blank" href="http://www.odoo.com/page/e-commerce?utm_source=db&amp;utm_medium=website">Open Source eCommerce</a>
                </t>
                <t t-set="_utm_medium" t-valuef="website"/>
            </t>
        </xpath>
    </template>

    <!-- User Navbar -->
    <template id="user_navbar_inherit_website_sale" inherit_id="website.user_navbar">
        <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_sale']" position="attributes">
            <attribute name="name"/>
            <attribute name="t-att-data-module-id"/>
            <attribute name="t-att-data-module-shortdesc"/>
            <attribute name="groups">sales_team.group_sale_manager</attribute>
        </xpath>
    </template>

    <template id="sale_order_portal_content_inherit_website_sale" name="Orders Followup Products Links" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//section[@id='details']//td[@id='product_name']/*" position="replace">
            <a t-if="line.product_id.website_published" t-att-href="line.product_id.website_url">
                <span t-field="line.name" />
            </a>
            <t t-if="not line.product_id.website_published">
                <span t-field="line.name" />
            </t>
        </xpath>
    </template>

    <template id="terms" name="Terms &amp; Conditions">
        <t t-call="website.layout">
          <div id="wrap">
              <div class="oe_structure" id="oe_structure_website_sale_terms_1">
                <section class="s_title" data-vcss="001">
                  <div class="container">
                    <div class="row">
                      <div class="col-lg-12">
                        <h1 class="text-center">Terms &amp;amp; Conditions</h1>
                        <div class="card s_well clearfix">
                            <div class="card-body">
                                <ul>
                                    <li>The <b>Intellectual Property</b> disclosure will inform users that the contents, logo and other visual media you created is your property and is protected by copyright laws.</li>
                                    <li>A <b>Termination</b> clause will inform that users’ accounts on your website and mobile app or users’ access to your website and mobile (if users can’t have an account with you) can be terminated in case of abuses or at your sole discretion.</li>
                                    <li>A <b>Governing Law</b> will inform users which laws govern the agreement. This should the country in which your company is headquartered or the country from which you operate your web site and mobile app.</li>
                                    <li>A <b>Links To Other Web Sites</b> clause will inform users that you are not responsible for any third party web sites that you link to. This kind of clause will generally inform users that they are responsible for reading and agreeing (or disagreeing) with the Terms and Conditions or Privacy Policies of these third parties.</li>
                                    <li>If your website or mobile apps allows users to create content and make that content public to other users, a <b>Content</b> section will inform users that they own the rights to the content they have created.<br/>The “Content” clause usually mentions that users must give you (the website or mobile app developer) a license so that you can share this content on your website/mobile app and to make it available to other users.<br/>Because the content created by users is public to other users, a DMCA notice clause (or Copyright Infringement ) section is helpful to inform users and copyright authors that, if any content is found to be a copyright infringement, you will respond to any DMCA take down notices received and you will take down the content.</li>
                                    <li>A <b>Limit What Users Can Do</b> clause can inform users that by agreeing to use your service, they’re also agreeing to not do certain things. This can be part of a very long and thorough list in your Terms and Conditions agreements so as to encompass the most amount of negative uses.</li>
                               </ul>
                               <small class="text-muted float-right">Source: https://termsfeed.com/blog/sample-terms-and-conditions-template</small>
                           </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </section>
                <section class="s_text_block" data-snippet="s_text_block">
                  <div class="container">
                    <div class="row">
                      <div class="col-lg-12 mb16 mt16">
                        <p style='white-space:pre' t-esc="website.company_id.invoice_terms"/>
                      </div>
                    </div>
                  </div>
                </section>
              </div>
              <div class="oe_structure" id="oe_structure_website_sale_terms_2"/>
          </div>
        </t>
    </template>

    <template id="website_sale.shop_product_carousel" name="Shop Product Carousel">
        <t t-set="product_images" t-value="product_variant._get_images() if product_variant else product._get_images()"/>
        <div id="o-carousel-product" class="carousel slide" data-ride="carousel" data-interval="0">
            <div class="carousel-outer position-relative">
                <div class="carousel-inner h-100">
                    <t t-foreach="product_images" t-as="product_image">
                        <div t-attf-class="carousel-item h-100#{' active' if product_image_first else ''}">
                            <div t-if="product_image._name == 'product.image' and product_image.embed_code" class="d-flex align-items-center justify-content-center h-100 embed-responsive embed-responsive-16by9">
                                <t t-raw="product_image.embed_code"/>
                            </div>
                            <div  t-else="" t-field="product_image.image_1920" class="d-flex align-items-center justify-content-center h-100" t-options='{"widget": "image", "preview_image": "image_1024", "class": "product_detail_img mh-100", "alt-field": "name", "zoom": product_image.can_image_1024_be_zoomed and "image_1920"}'/>
                        </div>
                    </t>
                </div>
                <t t-if="len(product_images) > 1">
                    <a class="carousel-control-prev" href="#o-carousel-product" role="button" data-slide="prev">
                        <span class="fa fa-chevron-left p-2" role="img" aria-label="Previous" title="Previous"/>
                    </a>
                    <a class="carousel-control-next" href="#o-carousel-product" role="button" data-slide="next">
                        <span class="fa fa-chevron-right p-2" role="img" aria-label="Next" title="Next"/>
                    </a>
                </t>
            </div>
            <div t-ignore="True" class="d-none d-md-block text-center">
                <ol t-if="len(product_images) > 1" class="carousel-indicators d-inline-block position-static mx-auto my-0 p-1 text-left">
                    <t t-foreach="product_images" t-as="product_image"><li t-attf-class="d-inline-block m-1 align-top {{'active' if product_image_first else ''}}" data-target="#o-carousel-product" t-att-data-slide-to="str(product_image_index)">
                        <div t-field="product_image.image_128" t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_64_contain", "alt-field": "name"}'/>
                        <i t-if="product_image._name == 'product.image' and product_image.embed_code" class="fa fa-2x fa-play-circle-o o_product_video_thumb"/>
                    </li></t>
                </ol>
            </div>
        </div>
    </template>


    <record id="view_website_sale_website_form" model="ir.ui.view">
        <field name="name">website_sale.website.form</field>
        <field name="model">website</field>
        <field name="inherit_id" ref="website.view_website_form"/>
        <field name="arch" type="xml">
            <xpath expr="//notebook" position="inside">
                <page string="Product Page Extra Fields" groups="base.group_no_one">
                    <field name="shop_extra_field_ids" context="{'default_website_id': active_id}">
                        <tree editable="bottom">
                            <field name="sequence" widget="handle"/>
                            <field name="field_id" required="1"/>
                        </tree>
                    </field>
                </page>
            </xpath>
        </field>
    </record>

    <template id="ecom_show_extra_fields" inherit_id="website_sale.product" active="False" customize_show="True" name="Show Extra Fields">
        <xpath expr="//div[@id='product_details']" position="inside">
            <hr/>
            <p class="text-muted">
                <t t-foreach='website.shop_extra_field_ids' t-as='field' t-if='product[field.name]'>
                    <b><t t-esc='field.label'/>: </b>
                    <t t-if='field.field_id.ttype != "binary"'>
                        <span t-esc='product[field.name]' t-options="{'widget': field.field_id.ttype}"/>
                    </t>
                    <t t-else=''>
                        <a target='_blank' t-attf-href='/web/content/product.template/#{product.id}/#{field.name}?download=1'>
                            <i class='fa fa-file'></i>
                        </a>
                    </t>
                    <br/>
                </t>
            </p>
        </xpath>
    </template>

</odoo>

```

## File: views\website_sale_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--product history-->
    <record id="website_sale_visitor_page_view_tree" model="ir.ui.view">
        <field name="name">website.track.view.tree</field>
        <field name="model">website.track</field>
        <field name="arch" type="xml">
            <tree string="Visitor Product Views History" create="0">
                <field name="visitor_id"/>
                <field name="product_id"/>
                <field name="visit_datetime"/>
            </tree>
        </field>
    </record>

    <record id="website_sale_visitor_page_view_graph" model="ir.ui.view">
        <field name="name">website.track.view.graph</field>
        <field name="model">website.track</field>
        <field name="arch" type="xml">
            <graph string="Visitor Product Views" sample="1">
                <field name="product_id"/>
            </graph>
        </field>
    </record>

    <record id="website_sale_visitor_product_action" model="ir.actions.act_window">
        <field name="name">Product Views History</field>
        <field name="res_model">website.track</field>
        <field name="view_mode">tree</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('website_sale_visitor_page_view_tree')}),
            (0, 0, {'view_mode': 'graph', 'view_id': ref('website_sale_visitor_page_view_graph')}),
        ]"/>
        <field name="domain">[('visitor_id', '=', active_id), ('product_id', '!=', False)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
              No product views yet for this visitor
            </p>
        </field>
    </record>

    <record id="website_sale_visitor_page_view_search" model="ir.ui.view">
        <field name="name">website.track.view.search</field>
        <field name="model">website.track</field>
        <field name="inherit_id" ref="website.website_visitor_page_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='url']" position="after">
                <field name="product_id"/>
            </xpath>
            <xpath expr="//filter[@name='type_url']" position="after">
                <filter string="Products" name="type_product" domain="[('product_id', '!=', False)]"/>
            </xpath>
            <xpath expr="//filter[@name='group_by_url']" position="after">
                <filter string="Product" name="group_by_product" domain="[]" context="{'group_by': 'product_id'}"/>
            </xpath>
        </field>
    </record>

    <!-- website visitor views -->
    <record id="website_sale_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(website.website_visitor_page_action)d']" position="after">
                <button name="%(website_sale.website_sale_visitor_product_action)d" type="action"
                    class="oe_stat_button"
                    icon="fa-tags">
                    <field name="visitor_product_count" widget="statinfo" string="Product views"/>
                </button>
            </xpath>
            <xpath expr="//group[@id='visits']/field[@name='page_ids']" position="after">
                <field name="product_ids" widget="many2many_tags"/>
            </xpath>
        </field>
    </record>

    <record id="website_sale_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='page_ids']" position="after">
                <field name="product_ids" widget="many2many_tags"/>
            </xpath>
        </field>
    </record>

    <record id="website_sale_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='page_ids']" position="after">
                <field name="product_ids"/>
            </xpath>
            <xpath expr="//div[@id='o_page_count']" position="after">
                 <div id="o_product_count">Visited Products<span class="float-right font-weight-bold"><field name="product_count"/></span></div>
            </xpath>
        </field>
    </record>

    <!-- website track views -->
    <record id="website_sale_visitor_track_view_tree" model="ir.ui.view">
        <field name="name">website.track.view.tree</field>
        <field name="model">website.track</field>
        <field name="inherit_id" ref="website.website_visitor_track_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='url']" position="after">
                <field name="product_id"/>
            </xpath>
        </field>
    </record>

    <record id="website_sale_visitor_track_view_graph" model="ir.ui.view">
        <field name="name">website.track.view.graph</field>
        <field name="model">website.track</field>
        <field name="inherit_id" ref="website.website_visitor_track_view_graph"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='url']" position="after">
                <field name="product_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_products_recently_viewed" name="Viewed Products">
    <section class="s_wsale_products_recently_viewed d-none pt24 pb24" style="min-height: 400px;">
        <div class="container">
            <div class="alert alert-info alert-dismissible rounded-0 fade show d-print-none css_non_editable_mode_hidden o_not_editable">
                This is a preview of the recently viewed products by the user.<br/>
                Once the user has seen at least one product this snippet will be visible.
                <button type="button" class="close" data-dismiss="alert" aria-label="Close"> × </button>
            </div>

            <h3 class="text-center mb32">Recently viewed Products</h3>
            <div class="slider o_not_editable"/>
        </div>
    </section>
</template>

<template id="snippets" inherit_id="website.snippets" name="e-commerce snippets">
    <xpath expr="//t[@id='sale_products_hook']" position="replace">
        <t t-snippet="website_sale.s_dynamic_snippet_products" t-thumbnail="/website_sale/static/src/img/snippets_thumbs/s_products_recently_viewed.svg"/>
    </xpath>
    <xpath expr="//t[@id='sale_recently_viewed_product_hook']" position="replace">
        <t t-snippet="website_sale.s_products_recently_viewed" t-thumbnail="/website_sale/static/src/img/snippets_thumbs/s_products_recently_viewed.svg"/>
    </xpath>
    <xpath expr="//t[@id='sale_product_search_section_hook']" position="replace">
        <!-- This snippet cannot be used in sanitized fields because it -->
        <!-- contains an input that would be removed -->
        <t t-snippet="website_sale.s_products_searchbar" t-thumbnail="/website_sale/static/src/img/snippets_thumbs/s_products_searchbar.svg" t-forbid-sanitize="form"/>
    </xpath>
    <xpath expr="//t[@id='sale_product_search_input_hook']" position="replace">
        <!-- This snippet cannot be used in sanitized fields because it -->
        <!-- contains an input that would be removed -->
        <t t-snippet="website_sale.s_products_searchbar_input" t-thumbnail="/website_sale/static/src/img/snippets_thumbs/s_products_searchbar_inline.svg" t-forbid-sanitize="form"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options" name="e-commerce snippet options">
    <xpath expr="." position="inside">
        <div data-js="WebsiteSaleGridLayout"
            data-selector="#products_grid .o_wsale_products_grid_table_wrapper > table"
            data-no-check="true">
            <we-input string="Number of products" data-set-ppg="" data-no-preview="true"/>
            <we-select string="Number of Columns" class="o_wsale_ppr_submenu" data-no-preview="true">
                <we-button data-set-ppr="2">2</we-button>
                <we-button data-set-ppr="3">3</we-button>
                <we-button data-set-ppr="4">4</we-button>
            </we-select>
        </div>

        <div data-js="WebsiteSaleProductsItem"
            data-selector="#products_grid .oe_product"
            data-no-check="true">
            <div class="o_wsale_soptions_menu_sizes">
                <we-title>Size</we-title>
                <table>
                    <tr>
                        <td/><td/><td/><td/>
                    </tr>
                    <tr>
                        <td/><td/><td/><td/>
                    </tr>
                    <tr>
                        <td/><td/><td/><td/>
                    </tr>
                    <tr>
                        <td/><td/><td/><td/>
                    </tr>
                </table>
            </div>

            <we-row data-name="ribbon_options">
                <we-select string="Ribbon" class="o_wsale_ribbon_select">
                    <we-button data-set-ribbon="" data-name="no_ribbon_opt">None</we-button>
                    <!-- Ribbons are filled in JS -->
                </we-select>
                <we-button data-edit-ribbon="" class="fa fa-edit" data-no-preview="true" data-dependencies="!no_ribbon_opt"/>
                <we-button data-create-ribbon="" class="fa fa-plus text-success" data-no-preview="true"/>
            </we-row>
            <div class="d-none" data-name="ribbon_customize_opt">
                <we-row string="Ribbon">
                    <we-input data-set-ribbon-html="" class="o_we_large_input" data-apply-to=".o_wsale_ribbon_dummy"/>
                    <we-button class="fa fa-check" data-save-ribbon="" title="Validate" data-no-preview="true"/>
                    <we-button class="fa fa-trash" data-delete-ribbon="" title="Delete" data-no-preview="true"/>
                </we-row>
                <we-colorpicker string="⌙ Background" title="" data-select-style="" data-apply-to=".o_wsale_ribbon_dummy" data-css-property="background-color" data-color-prefix="bg-"/>
                <we-colorpicker string="⌙ Text" title="" data-select-style="" data-apply-to=".o_wsale_ribbon_dummy" data-css-property="color"/>
                <we-select string="⌙ Mode">
                    <we-button data-set-ribbon-mode="ribbon">Slanted</we-button>
                    <we-button data-set-ribbon-mode="tag">Tag</we-button>
                </we-select>
                <we-select string="⌙ Position">
                    <we-button data-set-ribbon-position="left">Left</we-button>
                    <we-button data-set-ribbon-position="right">Right (only on grid view)</we-button>
                </we-select>
            </div>

            <div name="reordering" data-no-preview="true">
                <we-button data-change-sequence="top">Push to top</we-button>
                <we-button data-change-sequence="up">Push up</we-button>
                <we-button data-change-sequence="down">Push down</we-button>
                <we-button data-change-sequence="bottom">Push to bottom</we-button>
            </div>
        </div>
        <div data-selector="#wrapwrap > header"
            data-no-check="true"
            groups="website.group_website_designer">
            <we-checkbox string="Show Empty Cart"
                        data-customize-website-views="website_sale.header_hide_empty_cart_link|"
                        data-no-preview="true"
                        data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\snippets\s_dynamic_snippet_products.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_dynamic_snippet_products" name="Dynamic Products">
        <t t-call="website.s_dynamic_snippet_template">
            <t t-set="snippet_name" t-value="'s_dynamic_snippet_products'"/>
        </t>
    </template>
    <template id="s_dynamic_snippet_products_options"  inherit_id="website.snippet_options">
        <xpath expr="." position="inside">
            <t t-call="website.dynamic_snippet_carousel_options_template">
                <t t-set="snippet_name" t-value="'dynamic_snippet_products'"/>
                <t t-set="snippet_selector" t-value="'.s_dynamic_snippet_products'"/>
                <we-select string="Product Category" data-name="product_category_opt" data-attribute-name="productCategoryId" data-no-preview="true">
                    <we-button data-select-data-attribute="-1">All Products</we-button>
                </we-select>
            </t>
        </xpath>
    </template>
    <template id="assets_snippet_s_dynamic_snippet_products_js_000" inherit_id="website.assets_snippet_s_dynamic_snippet_carousel_js_000">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/website_sale/static/src/snippets/s_dynamic_snippet_products/000.js"/>
        </xpath>
    </template>
</odoo>

```

## File: views\snippets\s_products_searchbar.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_products_searchbar_input" name="Products Search">
    <t t-call="website_sale.website_sale_products_search_box">
        <t t-set="_classes" t-value="'s_wsale_products_searchbar_input'"/>
        <t t-set="_snippet" t-value="'s_products_searchbar_input'"/>
    </t>
</template>
<template id="s_products_searchbar" name="Products Search">
    <section class="s_wsale_products_searchbar bg-200 pt48 pb48" data-vxml="001">
        <div class="container">
            <div class="row">
                <div class="col-lg-8 offset-lg-2">
                    <h2>Search for a product</h2>
                    <p>We have amazing products in our shop, check them now !</p>
                    <t t-snippet-call="website_sale.s_products_searchbar_input"/>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="searchbar_input_snippet_options" inherit_id="website_sale.snippet_options" name="search bar snippet options">
    <xpath expr="." position="inside">
        <div data-selector=".s_wsale_products_searchbar_input">
            <we-select string="Order by" id="order_by" data-attribute-name="value" data-apply-to=".o_wsale_search_order_by">
                <we-button data-select-attribute="">default</we-button>
                <we-button data-select-attribute="name asc">name (A-Z)</we-button>
                <we-button data-select-attribute="name desc">name (Z-A)</we-button>
                <we-button data-select-attribute="list_price asc">price (low to high)</we-button>
                <we-button data-select-attribute="list_price desc">price (high to low)</we-button>
            </we-select>
            <t t-set="unit">products</t>
            <we-input string="Suggestions" data-name="product_limit_opt" data-attribute-name="limit"
                      data-apply-to=".search-query" data-select-data-attribute="" t-att-data-unit="unit"/>
            <we-checkbox string="Description" data-dependencies="product_limit_opt" data-select-data-attribute="true" data-attribute-name="displayDescription"
                         data-apply-to=".search-query"/>
            <we-checkbox string="Price" data-dependencies="product_limit_opt" data-select-data-attribute="true" data-attribute-name="displayPrice"
                         data-apply-to=".search-query"/>
            <we-checkbox string="Image" data-dependencies="product_limit_opt" data-select-data-attribute="true" data-attribute-name="displayImage"
                         data-apply-to=".search-query"/>
        </div>
    </xpath>
    <xpath expr="//*[@t-set='so_content_addition_selector']" position="inside">, .s_wsale_products_searchbar_input</xpath>
</template>

<template id="assets_snippet_s_products_searchbar_js_000" inherit_id="website_sale.assets_frontend">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_sale/static/src/snippets/s_products_searchbar/000.js"/>
    </xpath>
</template>

</odoo>

```

## File: wizard\payment_acquirer_onboarding_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PaymentWizard(models.TransientModel):
    _inherit = 'payment.acquirer.onboarding.wizard'
    _name = 'website.sale.payment.acquirer.onboarding.wizard'
    _description = 'Website Payment acquire onboarding wizard'

    def _set_payment_acquirer_onboarding_step_done(self):
        """ Override. """
        self.env.company.sudo().set_onboarding_step_done('website_sale_onboarding_payment_acquirer_state')

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_acquirer_onboarding_wizard

```

