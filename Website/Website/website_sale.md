# Odoo Module: website_sale

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import populate
from . import report


def _post_init_hook(env):
    terms_conditions = env['ir.config_parameter'].get_param('account.use_invoice_terms')
    if not terms_conditions:
        env['ir.config_parameter'].set_param('account.use_invoice_terms', True)
    companies = env['res.company'].search([])
    for company in companies:
        company.terms_type = 'html'
    env['website'].search([]).auth_signup_uninvited = 'b2c'

def uninstall_hook(env):
    ''' Need to reenable the `product` pricelist multi-company rule that were
        disabled to be 'overridden' for multi-website purpose
    '''
    pl_rule = env.ref('product.product_pricelist_comp_rule', raise_if_not_found=False)
    pl_item_rule = env.ref('product.product_pricelist_item_comp_rule', raise_if_not_found=False)
    multi_company_rules = pl_rule or env['ir.rule']
    multi_company_rules += pl_item_rule or env['ir.rule']
    multi_company_rules.write({'active': True})

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'eCommerce',
    'category': 'Website/Website',
    'sequence': 50,
    'summary': 'Sell your products online',
    'website': 'https://www.odoo.com/app/ecommerce',
    'version': '1.1',
    'depends': ['website', 'sale', 'website_payment', 'website_mail', 'portal_rating', 'digest', 'delivery'],
    'data': [
        'security/ir.model.access.csv',
        'security/website_sale.xml',

        'data/data.xml',
        'data/mail_template_data.xml',
        'data/product_snippet_template_data.xml',
        'data/digest_data.xml',
        'data/ir_cron_data.xml',

        'report/sale_report_views.xml',

        'views/account_move_views.xml',
        'views/crm_team_views.xml',
        'views/digest_views.xml',
        'views/product_attribute_views.xml',
        'views/product_document_views.xml',
        'views/product_tag_views.xml',
        'views/product_views.xml',
        'views/sale_order_views.xml',
        'views/templates.xml',
        'views/snippets/snippets.xml',
        'views/snippets/s_add_to_cart.xml',
        'views/snippets/s_dynamic_snippet_products.xml',
        'views/snippets/s_popup.xml',
        'views/res_config_settings_views.xml',
        'views/website_sale_visitor_views.xml',
        'views/website_base_unit_views.xml',
        'views/product_product_add.xml',
        'views/website_views.xml',
        'views/website_pages_views.xml',
        'views/website_sale_delivery_templates.xml',
        'views/website_sale_menus.xml',
        'views/website_sale_delivery_views.xml',
        'views/variant_templates.xml',
    ],
    'demo': [
        'data/demo.xml',
    ],
    'installable': True,
    'application': True,
    'post_init_hook': '_post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'website_sale/static/src/js/tours/tour_utils.js',
            'website_sale/static/src/scss/website_sale.scss',
            'website_sale/static/src/scss/website_mail.scss',
            'website_sale/static/src/scss/website_sale_frontend.scss',
            'website_sale/static/src/scss/website_sale_delivery.scss',
            'website/static/lib/multirange/multirange_custom.scss',
            'sale/static/src/scss/sale_portal.scss',
            'website_sale/static/src/js/payment_button.js',
            'website_sale/static/src/js/payment_form.js',
            'website_sale/static/src/js/sale_variant_mixin.js',
            'website_sale/static/src/js/terms_and_conditions_checkbox.js',
            'website_sale/static/src/js/variant_mixin.js',
            'website_sale/static/src/js/website_sale.js',
            'website_sale/static/src/xml/website_sale.xml',
            'website_sale/static/src/js/website_sale_offcanvas.js',
            'website_sale/static/src/js/website_sale_price_range_option.js',
            'website_sale/static/src/js/website_sale_utils.js',
            'website_sale/static/src/xml/website_sale_utils.xml',
            'website_sale/static/src/js/website_sale_recently_viewed.js',
            'website_sale/static/src/js/website_sale_tracking.js',
            'website/static/lib/multirange/multirange_custom.js',
            'website/static/lib/multirange/multirange_instance.js',
            'website_sale/static/src/js/website_sale_category_link.js',
            'website_sale/static/src/xml/website_sale_image_viewer.xml',
            'website_sale/static/src/js/components/website_sale_image_viewer.js',
            'website_sale/static/src/xml/website_sale_reorder_modal.xml',
            'website_sale/static/src/js/website_sale_reorder.js',
            'website_sale/static/src/js/website_sale_delivery.js',
            'website_sale/static/src/scss/product_configurator.scss',
            'website_sale/static/src/js/notification/add_to_cart_notification/add_to_cart_notification.js',
            'website_sale/static/src/js/notification/add_to_cart_notification/add_to_cart_notification.xml',
            'website_sale/static/src/js/notification/cart_notification/cart_notification.js',
            'website_sale/static/src/js/notification/cart_notification/cart_notification.xml',
            'website_sale/static/src/js/notification/warning_notification/warning_notification.js',
            'website_sale/static/src/js/notification/warning_notification/warning_notification.xml',
            'website_sale/static/src/js/notification/notification_service.js',
        ],
        'web._assets_primary_variables': [
            'website_sale/static/src/scss/primary_variables.scss',
        ],
        'web.assets_backend': [
            'website_sale/static/src/js/tours/tour_utils.js',
            'website_sale/static/src/js/website_sale_video_field_preview.js',
            'website_sale/static/src/scss/website_sale_backend.scss',
            'website_sale/static/src/js/tours/website_sale_shop.js',
            'website_sale/static/src/xml/website_sale.xml',
        ],
        'website.assets_wysiwyg': [
            'website_sale/static/src/scss/website_sale.editor.scss',
            'website_sale/static/src/snippets/s_dynamic_snippet_products/options.js',
            'website_sale/static/src/snippets/s_add_to_cart/options.js',
            'website_sale/static/src/js/website_sale.editor.js',
            'website_sale/static/src/js/website_sale_form_editor.js',
        ],
        'website.assets_editor': [
            'website_sale/static/src/js/systray_items/*.js',
            'website_sale/static/src/xml/website_sale_utils.xml',
        ],
        'website.backend_assets_all_wysiwyg': [
            'website_sale/static/src/js/components/wysiwyg_adapter/wysiwyg_adapter.js',
        ],
        'web.assets_tests': [
            'website_sale/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\delivery.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
from odoo import http, _
from odoo.http import request
from odoo.addons.payment import utils as payment_utils
from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.exceptions import UserError


class WebsiteSaleDelivery(WebsiteSale):
    _express_checkout_shipping_route = '/shop/express/shipping_address_change'

    @http.route(['/shop/update_carrier'], type='json', auth='public', methods=['POST'], website=True)
    def update_eshop_carrier(self, **post):
        order = request.website.sale_get_order()
        if not post.get('no_reset_access_point_address'):
            order.access_point_address = {}
        carrier_id = int(post['carrier_id'])
        if order and carrier_id != order.carrier_id.id:
            if any(tx.sudo().state not in ('cancel', 'error', 'draft') for tx in order.transaction_ids):
                raise UserError(_('It seems that there is already a transaction for your order, you can not change the delivery method anymore'))
            order._check_carrier_quotation(force_carrier_id=carrier_id)
        return self._update_website_sale_delivery_return(order, **post)

    @http.route(['/shop/carrier_rate_shipment'], type='json', auth='public', methods=['POST'], website=True)
    def cart_carrier_rate_shipment(self, carrier_id, **kw):
        order = request.website.sale_get_order(force_create=True)

        if not int(carrier_id) in order._get_delivery_methods().ids:
            raise UserError(_('It seems that a delivery method is not compatible with your address. Please refresh the page and try again.'))

        Monetary = request.env['ir.qweb.field.monetary']

        res = {'carrier_id': carrier_id}
        carrier = request.env['delivery.carrier'].sudo().browse(int(carrier_id))
        rate = WebsiteSaleDelivery._get_rate(carrier, order)
        if rate.get('success'):
            res['status'] = True
            res['new_amount_delivery'] = Monetary.value_to_html(rate['price'], {'display_currency': order.currency_id})
            res['is_free_delivery'] = not bool(rate['price'])
            res['error_message'] = rate['warning_message']
        else:
            res['status'] = False
            res['new_amount_delivery'] = Monetary.value_to_html(0.0, {'display_currency': order.currency_id})
            res['error_message'] = rate['error_message']
        return res

    @http.route(
        _express_checkout_shipping_route, type='json', auth='public', methods=['POST'],
        website=True, sitemap=False
    )
    def express_checkout_process_shipping_address(self, partial_shipping_address):
        """ Processes shipping address and returns available carriers.

        Depending on whether the partner is registered and logged in or not, creates a new partner
        or uses an existing partner that matches the partial shipping address received.

        :param dict shipping_address: a dictionary containing part of shipping information sent by
                                      the express payment provider.
        :return dict: all available carriers for `shipping_address` sorted by lowest price.
        """
        order_sudo = request.website.sale_get_order()
        public_partner = request.website.partner_id

        self._include_country_and_state_in_address(partial_shipping_address)
        if order_sudo.partner_id == public_partner:
            # The partner_shipping_id and partner_invoice_id will be automatically computed when
            # changing the partner_id of the SO. This allow website_sale to avoid create duplicates.
            order_sudo.partner_id = self._create_or_edit_partner(
                partial_shipping_address,
                type='delivery',
                name=_('Anonymous express checkout partner for order %s', order_sudo.name),
            )
            # Pricelist are recomputed every time the partner is changed. We don't want to recompute
            # the price with another pricelist at this state since the customer has already accepted
            # the amount and validated the payment.
            order_sudo.env.remove_to_compute(order_sudo._fields['pricelist_id'], order_sudo)
        elif order_sudo.partner_shipping_id.name.endswith(order_sudo.name):
            self._create_or_edit_partner(
                partial_shipping_address,
                edit=True,
                type='delivery',
                partner_id=order_sudo.partner_shipping_id.id,
            )
        elif any(
            partial_shipping_address[k] != order_sudo.partner_shipping_id[k]
            for k in partial_shipping_address
        ):
            # Check if a child partner doesn't already exist with the same informations. The
            # phone isn't always checked because it isn't sent in shipping information with
            # Google Pay.
            child_partner_id = self._find_child_partner(
                order_sudo.partner_id.commercial_partner_id.id, partial_shipping_address
            )
            order_sudo.partner_shipping_id = child_partner_id or self._create_or_edit_partner(
                    partial_shipping_address,
                    type='delivery',
                    parent_id=order_sudo.partner_id.id,
                    name=_('Anonymous express checkout partner for order %s', order_sudo.name),
            )

        # Returns the list of delivery carrier available for the sale order.
        return sorted([{
            'id': carrier.id,
            'name': carrier.name,
            'description': carrier.website_description,
            'minorAmount': payment_utils.to_minor_currency_units(price, order_sudo.currency_id),
        } for carrier, price in WebsiteSaleDelivery._get_carriers_express_checkout(order_sudo).items()
        ], key=lambda carrier: carrier['minorAmount'])

    @staticmethod
    def _get_carriers_express_checkout(order_sudo):
        """ Return available carriers and their prices for the given order.

        :param sale.order order_sudo: The sudoed sales order.
        :rtype: dict
        :return: A dict with a `delivery.carrier` recordset as key, and a rate shipment price as
                 value.
        """
        res = {}
        for carrier in order_sudo._get_delivery_methods():
            rate = WebsiteSaleDelivery._get_rate(carrier, order_sudo, is_express_checkout_flow=True)
            if rate['success']:
                fname = f'{carrier.delivery_type}_use_locations'
                if hasattr(carrier, fname) and getattr(carrier, fname):
                    continue  # Express checkout doesn't allow selecting locations.
                res[carrier] = rate['price']
        return res

    @http.route('/shop/access_point/set', type='json', auth='public', methods=['POST'], website=True, sitemap=False)
    def set_access_point(self, access_point_encoded):
        order = request.website.sale_get_order()
        if hasattr(order.carrier_id, order.carrier_id.delivery_type + '_use_locations'):
            use_location = getattr(order.carrier_id, order.carrier_id.delivery_type + '_use_locations')
            access_point = use_location and (json.loads(access_point_encoded) if access_point_encoded else False) or False
            order.write({'access_point_address': access_point})

    @http.route('/shop/access_point/get', type='json', auth='public', website=True, sitemap=False)
    def get_access_point(self):
        order = request.website.sale_get_order()
        if not order.carrier_id.delivery_type or not order.carrier_id.display_name:
            return {}
        order_location = order.access_point_address
        if not order_location:
            return {}
        address = order_location['address']
        name = order_location['pick_up_point_name']
        return {order.carrier_id.delivery_type + '_access_point': address, 'name': name, 'delivery_name': order.carrier_id.display_name}

    @http.route('/shop/access_point/close_locations', type='json', auth='public', website=True, sitemap=False)
    def get_close_locations(self):
        order = request.website.sale_get_order()
        try:
            error = {'error': _('No pick-up point available for that shipping address')}
            if not hasattr(order.carrier_id, '_' + order.carrier_id.delivery_type + '_get_close_locations'):
                return error
            close_locations = getattr(order.carrier_id, '_' + order.carrier_id.delivery_type + '_get_close_locations')(order.partner_shipping_id)
            partner_address = order.partner_shipping_id
            inline_partner_address = ' '.join((part or '') for part in [partner_address.street, partner_address.street2, partner_address.zip, partner_address.country_id.code])
            if len(close_locations) < 0:
                return error
            for location in close_locations:
                location['address_stringified'] = json.dumps(location)
            return {'close_locations': close_locations, 'partner_address': inline_partner_address}
        except UserError as e:
            return {'error': str(e)}

    @staticmethod
    def _get_rate(carrier, order, is_express_checkout_flow=False):
        """ Compute the price of the order shipment and apply the taxes if relevant

        :param recordset carrier: the carrier for which the rate is to be recovered
        :param recordset order: the order for which the rate is to be recovered
        :param boolean is_express_checkout_flow: Whether the flow is express checkout or not
        :return dict: the rate, as returned in `rate_shipment()`
        """
        # Some delivery carriers check if all the required fields are available before computing the
        # rate, even if those fields aren't required for computing the rate (although they are for
        # delivering the goods). If we only have partial information about the delivery address but
        # still want to compute the rate, this context key will ensure that we only check the
        # required fields for a partial delivery address (city, zip, country_code, state_code).
        rate = carrier.rate_shipment(order.with_context(
            express_checkout_partial_delivery_address=is_express_checkout_flow
        ))
        if rate.get('success'):
            tax_ids = carrier.product_id.taxes_id.filtered(
                lambda t: t.company_id == order.company_id
            )
            if tax_ids:
                fpos = order.fiscal_position_id
                tax_ids = fpos.map_tax(tax_ids)
                taxes = tax_ids.compute_all(
                    rate['price'],
                    currency=order.currency_id,
                    quantity=1.0,
                    product=carrier.product_id,
                    partner=order.partner_shipping_id,
                )
                if not is_express_checkout_flow and request.website.show_line_subtotals_tax_selection == 'tax_excluded':
                    rate['price'] = taxes['total_excluded']
                else:
                    rate['price'] = taxes['total_included']
        return rate

    def _update_website_sale_delivery_return(self, order, **post):
        Monetary = request.env['ir.qweb.field.monetary']
        carrier_id = int(post['carrier_id'])
        currency = order.currency_id
        if order:
            return {
                'status': order.delivery_rating_success,
                'error_message': order.delivery_message,
                'carrier_id': carrier_id,
                'is_free_delivery': not bool(order.amount_delivery),
                'new_amount_delivery': Monetary.value_to_html(order.amount_delivery, {'display_currency': currency}),
                'new_amount_untaxed': Monetary.value_to_html(order.amount_untaxed, {'display_currency': currency}),
                'new_amount_tax': Monetary.value_to_html(order.amount_tax, {'display_currency': currency}),
                'new_amount_total': Monetary.value_to_html(order.amount_total, {'display_currency': currency}),
                'new_amount_total_raw': order.amount_total,
            }
        return {}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
from datetime import datetime

from psycopg2.errors import LockNotAvailable
from werkzeug.exceptions import Forbidden, NotFound
from werkzeug.urls import url_decode, url_encode, url_parse

from odoo import fields, http, SUPERUSER_ID, tools, _
from odoo.exceptions import AccessError, MissingError, UserError, ValidationError
from odoo.fields import Command
from odoo.http import request, route
from odoo.tools import SQL, lazy, str2bool

from odoo.addons.base.models.ir_qweb_fields import nl2br_enclose
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.controllers import portal as payment_portal
from odoo.addons.website.controllers.main import QueryURL
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.addons.portal.controllers.portal import _build_url_w_params
from odoo.addons.website.controllers import main
from odoo.addons.website.controllers.form import WebsiteForm
from odoo.addons.sale.controllers import portal as sale_portal
from odoo.osv import expression
from odoo.tools.json import scriptsafe as json_scriptsafe

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
                'ribbon': p.sudo().website_ribbon_id,
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

    @http.route('/website/form/shop.sale.order', type='http', auth="public", methods=['POST'], website=True)
    def website_form_saleorder(self, **kwargs):
        model_record = request.env.ref('sale.model_sale_order')
        try:
            data = self.extract_data(model_record, kwargs)
        except ValidationError as e:
            return json.dumps({'error_fields': e.args[0]})

        order = request.website.sale_get_order()
        if not order:
            return json.dumps({'error': "No order found; please add a product to your cart."})

        if data['record']:
            order.write(data['record'])

        if data['custom']:
            order._message_log(
                body=nl2br_enclose(data['custom'], 'p'),
                message_type='comment',
            )

        if data['attachments']:
            self.insert_attachment(model_record, order.id, data['attachments'])

        return json.dumps({'id': order.id})


class Website(main.Website):

    def _login_redirect(self, uid, redirect=None):
        # If we are logging in, clear the current pricelist to be able to find
        # the pricelist that corresponds to the user afterwards.
        request.session.pop('website_sale_current_pl', None)
        return super()._login_redirect(uid, redirect=redirect)

    @http.route()
    def autocomplete(self, search_type=None, term=None, order=None, limit=5, max_nb_chars=999, options=None):
        options = options or {}
        if 'display_currency' not in options:
            options['display_currency'] = request.website.currency_id
        return super().autocomplete(search_type, term, order, limit, max_nb_chars, options)

    @http.route()
    def theme_customize_data(self, is_view_data, enable=None, disable=None, reset_view_arch=False):
        super().theme_customize_data(is_view_data, enable, disable, reset_view_arch)
        if any(key in enable or key in disable for key in ['website_sale.products_list_view', 'website_sale.add_grid_or_list_option']):
            request.session.pop('website_sale_shop_layout_mode', None)

    @http.route()
    def get_current_currency(self, **kwargs):
        return {
            'id': request.website.currency_id.id,
            'symbol': request.website.currency_id.symbol,
            'position': request.website.currency_id.position,
        }

    @http.route()
    def change_lang(self, lang, **kwargs):
        order_sudo = request.website.sale_get_order()
        request.env.add_to_compute(
            order_sudo.order_line._fields['name'],
            order_sudo.order_line.with_context(lang=lang),
        )
        return super().change_lang(lang, **kwargs)


class WebsiteSale(payment_portal.PaymentPortal):
    _express_checkout_route = '/shop/express_checkout'
    _express_checkout_shipping_route = '/shop/express/shipping_address_change'

    WRITABLE_PARTNER_FIELDS = [
        'name',
        'email',
        'phone',
        'street',
        'street2',
        'city',
        'zip',
        'country_id',
        'state_id',
    ]

    def _get_search_order(self, post):
        # OrderBy will be parsed in orm and so no direct sql injection
        # id is added to be sure that order is a unique sort key
        order = post.get('order') or request.env['website'].get_current_website().shop_default_sort
        return 'is_published desc, %s, id desc' % order

    def _add_search_subdomains_hook(self, search):
        return []

    def _get_shop_domain(self, search, category, attrib_values, search_in_description=True):
        domains = [request.website.sale_product_domain()]
        if search:
            for srch in search.split(" "):
                subdomains = [
                    [('name', 'ilike', srch)],
                    [('product_variant_ids.default_code', 'ilike', srch)]
                ]
                if search_in_description:
                    subdomains.append([('website_description', 'ilike', srch)])
                    subdomains.append([('description_sale', 'ilike', srch)])
                extra_subdomain = self._add_search_subdomains_hook(srch)
                if extra_subdomain:
                    subdomains.append(extra_subdomain)
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

    def _get_search_options(
        self, category=None, attrib_values=None, tags=None, min_price=0.0, max_price=0.0,
        conversion_rate=1, **post
    ):
        return {
            'displayDescription': True,
            'displayDetail': True,
            'displayExtraDetail': True,
            'displayExtraLink': True,
            'displayImage': True,
            'allowFuzzy': not post.get('noFuzzy'),
            'category': str(category.id) if category else None,
            'tags': tags,
            'min_price': min_price / conversion_rate,
            'max_price': max_price / conversion_rate,
            'attrib_values': attrib_values,
            'display_currency': post.get('display_currency'),
        }

    def _shop_lookup_products(self, attrib_set, options, post, search, website):
        # No limit because attributes are obtained from complete product list
        product_count, details, fuzzy_search_term = website._search_with_fuzzy("products_only", search,
                                                                               limit=None,
                                                                               order=self._get_search_order(post),
                                                                               options=options)
        search_result = details[0].get('results', request.env['product.template']).with_context(bin_size=True)

        return fuzzy_search_term, product_count, search_result

    def _shop_get_query_url_kwargs(
        self, category, search, min_price, max_price, attrib=None, order=None, tags=None, **post
    ):
        return {
            'category': category,
            'search': search,
            'attrib': attrib,
            'tags': tags,
            'min_price': min_price,
            'max_price': max_price,
            'order': order,
        }

    def _get_additional_shop_values(self, values):
        """ Hook to update values used for rendering website_sale.products template """
        return {}

    def _get_additional_extra_shop_values(self, values, **post):
        """ Hook to update values used for rendering website_sale.products template """
        return self._get_additional_shop_values(values)

    @http.route([
        '/shop',
        '/shop/page/<int:page>',
        '/shop/category/<model("product.public.category"):category>',
        '/shop/category/<model("product.public.category"):category>/page/<int:page>',
    ], type='http', auth="public", website=True, sitemap=sitemap_shop)
    def shop(self, page=0, category=None, search='', min_price=0.0, max_price=0.0, ppg=False, **post):
        add_qty = int(post.get('add_qty', 1))
        try:
            min_price = float(min_price)
        except ValueError:
            min_price = 0
        try:
            max_price = float(max_price)
        except ValueError:
            max_price = 0

        Category = request.env['product.public.category']
        if category:
            category = Category.search([('id', '=', int(category))], limit=1)
            if not category or not category.can_access_from_current_website():
                raise NotFound()
        else:
            category = Category

        website = request.env['website'].get_current_website()
        website_domain = website.website_domain()
        if ppg:
            try:
                ppg = int(ppg)
                post['ppg'] = ppg
            except ValueError:
                ppg = False
        if not ppg:
            ppg = website.shop_ppg or 20

        ppr = website.shop_ppr or 4

        request_args = request.httprequest.args
        attrib_list = request_args.getlist('attrib')
        attrib_values = [[int(x) for x in v.split("-")] for v in attrib_list if v]
        attributes_ids = {v[0] for v in attrib_values}
        attrib_set = {v[1] for v in attrib_values}
        if attrib_list:
            post['attrib'] = attrib_list

        filter_by_tags_enabled = website.is_view_active('website_sale.filter_products_tags')
        if filter_by_tags_enabled:
            tags = request_args.getlist('tags')
            # Allow only numeric tag values to avoid internal error.
            if tags and all(tag.isnumeric() for tag in tags):
                post['tags'] = tags
                tags = {int(tag) for tag in tags}
            else:
                post['tags'] = None
                tags = {}

        keep = QueryURL('/shop', **self._shop_get_query_url_kwargs(category and int(category), search, min_price, max_price, **post))

        now = datetime.timestamp(datetime.now())
        pricelist = website.pricelist_id
        if 'website_sale_pricelist_time' in request.session:
            # Check if we need to refresh the cached pricelist
            pricelist_save_time = request.session['website_sale_pricelist_time']
            if pricelist_save_time < now - 60*60:
                request.session.pop('website_sale_current_pl', None)
                website.invalidate_recordset(['pricelist_id'])
                pricelist = website.pricelist_id
                request.session['website_sale_pricelist_time'] = now
                request.session['website_sale_current_pl'] = pricelist.id
        else:
            request.session['website_sale_pricelist_time'] = now
            request.session['website_sale_current_pl'] = pricelist.id

        filter_by_price_enabled = website.is_view_active('website_sale.filter_products_price')
        if filter_by_price_enabled:
            company_currency = website.company_id.sudo().currency_id
            conversion_rate = request.env['res.currency']._get_conversion_rate(
                company_currency, website.currency_id, request.website.company_id, fields.Date.today())
        else:
            conversion_rate = 1

        url = '/shop'
        if search:
            post['search'] = search

        options = self._get_search_options(
            category=category,
            attrib_values=attrib_values,
            min_price=min_price,
            max_price=max_price,
            conversion_rate=conversion_rate,
            display_currency=website.currency_id,
            **post
        )
        fuzzy_search_term, product_count, search_product = self._shop_lookup_products(attrib_set, options, post, search, website)

        filter_by_price_enabled = website.is_view_active('website_sale.filter_products_price')
        if filter_by_price_enabled:
            # TODO Find an alternative way to obtain the domain through the search metadata.
            Product = request.env['product.template'].with_context(bin_size=True)
            domain = self._get_shop_domain(search, category, attrib_values)

            # This is ~4 times more efficient than a search for the cheapest and most expensive products
            query = Product._where_calc(domain)
            Product._apply_ir_rules(query, 'read')
            from_clause, where_clause, where_params = query.get_sql()
            query = f"""
                SELECT COALESCE(MIN(list_price), 0) * {conversion_rate}, COALESCE(MAX(list_price), 0) * {conversion_rate}
                  FROM {from_clause}
                 WHERE {where_clause}
            """
            request.env.cr.execute(query, where_params)
            available_min_price, available_max_price = request.env.cr.fetchone()

            if min_price or max_price:
                # The if/else condition in the min_price / max_price value assignment
                # tackles the case where we switch to a list of products with different
                # available min / max prices than the ones set in the previous page.
                # In order to have logical results and not yield empty product lists, the
                # price filter is set to their respective available prices when the specified
                # min exceeds the max, and / or the specified max is lower than the available min.
                if min_price:
                    min_price = min_price if min_price <= available_max_price else available_min_price
                    post['min_price'] = min_price
                if max_price:
                    max_price = max_price if max_price >= available_min_price else available_max_price
                    post['max_price'] = max_price

        ProductTag = request.env['product.tag']
        if filter_by_tags_enabled and search_product:
            all_tags = ProductTag.search(
                expression.AND([
                    [('product_ids.is_published', '=', True), ('visible_on_ecommerce', '=', True)],
                    website_domain
                ])
            )
        else:
            all_tags = ProductTag

        categs_domain = [('parent_id', '=', False)] + website_domain
        if search:
            search_categories = Category.search(
                [('product_tmpl_ids', 'in', search_product.ids)] + website_domain
            ).parents_and_self
            categs_domain.append(('id', 'in', search_categories.ids))
        else:
            search_categories = Category
        categs = lazy(lambda: Category.search(categs_domain))

        if category:
            url = "/shop/category/%s" % slug(category)

        pager = website.pager(url=url, total=product_count, page=page, step=ppg, scope=5, url_args=post)
        offset = pager['offset']
        products = search_product[offset:offset + ppg]

        ProductAttribute = request.env['product.attribute']
        if products:
            # get all products without limit
            attributes = lazy(lambda: ProductAttribute.search([
                ('product_tmpl_ids', 'in', search_product.ids),
                ('visibility', '=', 'visible'),
            ]))
        else:
            attributes = lazy(lambda: ProductAttribute.browse(attributes_ids))

        layout_mode = request.session.get('website_sale_shop_layout_mode')
        if not layout_mode:
            if website.viewref('website_sale.products_list_view').active:
                layout_mode = 'list'
            else:
                layout_mode = 'grid'
            request.session['website_sale_shop_layout_mode'] = layout_mode

        # Try to fetch geoip based fpos or fallback on partner one
        fiscal_position_sudo = website.fiscal_position_id.sudo()
        products_prices = lazy(lambda: products._get_sales_prices(pricelist, fiscal_position_sudo))

        values = {
            'search': fuzzy_search_term or search,
            'original_search': fuzzy_search_term and search,
            'order': post.get('order', ''),
            'category': category,
            'attrib_values': attrib_values,
            'attrib_set': attrib_set,
            'pager': pager,
            'pricelist': pricelist,
            'fiscal_position': fiscal_position_sudo,
            'add_qty': add_qty,
            'products': products,
            'search_product': search_product,
            'search_count': product_count,  # common for all searchbox
            'bins': lazy(lambda: TableCompute().process(products, ppg, ppr)),
            'ppg': ppg,
            'ppr': ppr,
            'categories': categs,
            'attributes': attributes,
            'keep': keep,
            'search_categories_ids': search_categories.ids,
            'layout_mode': layout_mode,
            'products_prices': products_prices,
            'get_product_prices': lambda product: lazy(lambda: products_prices[product.id]),
            'float_round': tools.float_round,
        }
        if filter_by_price_enabled:
            values['min_price'] = min_price or available_min_price
            values['max_price'] = max_price or available_max_price
            values['available_min_price'] = tools.float_round(available_min_price, 2)
            values['available_max_price'] = tools.float_round(available_max_price, 2)
        if filter_by_tags_enabled:
            values.update({'all_tags': all_tags, 'tags': tags})
        if category:
            values['main_object'] = category
        values.update(self._get_additional_extra_shop_values(values, **post))
        return request.render("website_sale.products", values)

    @http.route(['/shop/<model("product.template"):product>'], type='http', auth="public", website=True, sitemap=True)
    def product(self, product, category='', search='', **kwargs):
        return request.render("website_sale.product", self._prepare_product_values(product, category, search, **kwargs))

    @http.route(
        '/shop/<model("product.template"):product_template>/document/<int:document_id>',
        type='http',
        auth='public',
        website=True,
        sitemap=False,
    )
    def product_document(self, product_template, document_id):
        product_template.check_access_rights('read')

        document = request.env['product.document'].browse(document_id).sudo().exists()
        if not document or not document.active:
            return request.redirect('/shop')

        if not document.shown_on_product_page or not (
            document.res_id == product_template.id
            and document.res_model == 'product.template'
        ):
            return request.redirect('/shop')

        return request.env['ir.binary']._get_stream_from(
            document.ir_attachment_id,
        ).get_response(as_attachment=True)

    @http.route(['/shop/product/<model("product.template"):product>'], type='http', auth="public", website=True, sitemap=False)
    def old_product(self, product, category='', search='', **kwargs):
        # Compatibility pre-v14
        return request.redirect(_build_url_w_params("/shop/%s" % slug(product), request.params), code=301)

    @http.route(['/shop/product/extra-images'], type='json', auth='user', website=True)
    def add_product_images(self, images, product_product_id, product_template_id, combination_ids=None):
        """
        Turns a list of image ids refering to ir.attachments to product.images,
        links all of them to product.
        :raises NotFound : If the user is not allowed to access Attachment model
        """

        if not request.env.user.has_group('website.group_website_restricted_editor'):
            raise NotFound()

        image_ids = request.env["ir.attachment"].browse(i['id'] for i in images)
        image_create_data = [Command.create({
                    'name': image.name,                          # Images uploaded from url do not have any datas. This recovers them manually
                    'image_1920': image.datas if image.datas else request.env['ir.qweb.field.image'].load_remote_url(image.url),
                }) for image in image_ids]

        product_product = request.env['product.product'].browse(int(product_product_id)) if product_product_id else False
        product_template = request.env['product.template'].browse(int(product_template_id)) if product_template_id else False

        if product_product and not product_template:
            product_template = product_product.product_tmpl_id

        if not product_product and product_template and product_template.has_dynamic_attributes():
            combination = request.env['product.template.attribute.value'].browse(combination_ids)
            product_product = product_template._get_variant_for_combination(combination)
            if not product_product:
                product_product = product_template._create_product_variant(combination)
        if product_template.has_configurable_attributes and product_product and not all(pa.create_variant == 'no_variant' for pa in product_template.attribute_line_ids.attribute_id):
            product_product.write({
                'product_variant_image_ids': image_create_data
            })
        else:
            product_template.write({
                'product_template_image_ids': image_create_data
            })

    @http.route(['/shop/product/clear-images'], type='json', auth='user', website=True)
    def clear_product_images(self, product_product_id, product_template_id):
        """
        Unlinks all images from the product.
        """
        if not request.env.user.has_group('website.group_website_restricted_editor'):
            raise NotFound()

        product_product = request.env['product.product'].browse(int(product_product_id)) if product_product_id else False
        product_template = request.env['product.template'].browse(int(product_template_id)) if product_template_id else False

        if product_product and not product_template:
            product_template = product_product.product_tmpl_id

        if product_product and product_product.product_variant_image_ids:
            product_product.product_variant_image_ids.unlink()
        else:
            product_template.product_template_image_ids.unlink()

    @http.route(['/shop/product/resequence-image'], type='json', auth='user', website=True)
    def resequence_product_image(self, image_res_model, image_res_id, move):
        """
        Move the product image in the given direction and update all images' sequence.

        :param str image_res_model: The model of the image. It can be 'product.template',
                                    'product.product', or 'product.image'.
        :param str image_res_id: The record ID of the image to move.
        :param str move: The direction of the move. It can be 'first', 'left', 'right', or 'last'.
        :raises NotFound: If the user does not have the required permissions, if the model of the
                          image is not allowed, or if the move direction is not allowed.
        :raise ValidationError: If the product is not found.
        :raise ValidationError: If the image to move is not found in the product images.
        :raise ValidationError: If a video is moved to the first position.
        :return: None
        """
        if (
            not request.env.user.has_group('website.group_website_restricted_editor')
            or image_res_model not in ['product.product', 'product.template', 'product.image']
            or move not in ['first', 'left', 'right', 'last']
        ):
            raise NotFound()

        image_res_id = int(image_res_id)
        image_to_resequence = request.env[image_res_model].browse(image_res_id)
        if image_res_model == 'product.product':
            product = image_to_resequence
            product_template = product.product_tmpl_id
        elif image_res_model == 'product.template':
            product_template = image_to_resequence
            product = product_template.product_variant_id
        else:
            product = image_to_resequence.product_variant_id
            product_template = product.product_tmpl_id or image_to_resequence.product_tmpl_id

        if not product and not product_template:
            raise ValidationError(_("Product not found"))

        product_images = (product or product_template)._get_images()
        if image_to_resequence not in product_images:
            raise ValidationError(_("Invalid image"))

        image_idx = product_images.index(image_to_resequence)
        new_image_idx = 0
        if move == 'left':
            new_image_idx = max(0, image_idx - 1)
        elif move == 'right':
            new_image_idx = min(len(product_images) - 1, image_idx + 1)
        elif move == 'last':
            new_image_idx = len(product_images) - 1

        # no-op resequences
        if new_image_idx == image_idx:
            return

        # Reorder images locally.
        product_images.insert(new_image_idx, product_images.pop(image_idx))

        # If the main image has been reordered (i.e. it's no longer in first position), use the
        # image that's now in first position as main image instead.
        # Additional images are product.image records. The main image is a product.product or
        # product.template record.
        main_image_idx = next(
            idx for idx, image in enumerate(product_images) if image._name != 'product.image'
        )
        if main_image_idx != 0:
            main_image = product_images[main_image_idx]
            additional_image = product_images[0]
            if additional_image.video_url:
                raise ValidationError(_("You can't use a video as the product's main image."))
            # Swap records.
            product_images[main_image_idx], product_images[0] = additional_image, main_image
            # Swap image data.
            main_image.image_1920, additional_image.image_1920 = (
                additional_image.image_1920, main_image.image_1920
            )
            additional_image.name = main_image.name  # Update image name but not product name.

        # Resequence additional images according to the new ordering.
        for idx, product_image in enumerate(product_images):
            if product_image._name == 'product.image':
                product_image.sequence = idx

    @http.route(['/shop/product/is_add_to_cart_allowed'], type='json', auth="public", website=True)
    def is_add_to_cart_allowed(self, product_id, **kwargs):
        product = request.env['product.product'].browse(product_id)
        return product._is_add_to_cart_allowed()

    def _product_get_query_url_kwargs(self, category, search, attrib=None, **kwargs):
        return {
            'category': category,
            'search': search,
            'attrib': attrib,
            'tags': kwargs.get('tags'),
            'min_price': kwargs.get('min_price'),
            'max_price': kwargs.get('max_price'),
        }

    def _prepare_product_values(self, product, category, search, **kwargs):
        ProductCategory = request.env['product.public.category']

        if category:
            category = ProductCategory.browse(int(category)).exists()

        attrib_list = request.httprequest.args.getlist('attrib')
        attrib_values = [[int(x) for x in v.split("-")] for v in attrib_list if v]
        attrib_set = {v[1] for v in attrib_values}

        keep = QueryURL(
            '/shop',
            **self._product_get_query_url_kwargs(
                category=category and category.id,
                search=search,
                **kwargs,
            ),
        )

        # Needed to trigger the recently viewed product rpc
        view_track = request.website.viewref("website_sale.product").track

        return {
            'search': search,
            'category': category,
            'pricelist': request.website.pricelist_id,
            'attrib_values': attrib_values,
            'attrib_set': attrib_set,
            'keep': keep,
            'categories': ProductCategory.search([('parent_id', '=', False)]),
            'main_object': product,
            'product': product,
            'add_qty': 1,
            'view_track': view_track,
        }

    @http.route(['/shop/change_pricelist/<model("product.pricelist"):pricelist>'], type='http', auth="public", website=True, sitemap=False)
    def pricelist_change(self, pricelist, **post):
        website = request.env['website'].get_current_website()
        redirect_url = request.httprequest.referrer
        if (pricelist.selectable or pricelist == request.env.user.partner_id.property_product_pricelist) \
                and website.is_pricelist_available(pricelist.id):
            if redirect_url and request.website.is_view_active('website_sale.filter_products_price'):
                decoded_url = url_parse(redirect_url)
                args = url_decode(decoded_url.query)
                min_price = args.get('min_price')
                max_price = args.get('max_price')
                if min_price or max_price:
                    previous_price_list = request.website.pricelist_id
                    try:
                        min_price = float(min_price)
                        args['min_price'] = min_price and str(
                            previous_price_list.currency_id._convert(min_price, pricelist.currency_id, request.website.company_id, fields.Date.today(), round=False)
                        )
                    except (ValueError, TypeError):
                        pass
                    try:
                        max_price = float(max_price)
                        args['max_price'] = max_price and str(
                            previous_price_list.currency_id._convert(max_price, pricelist.currency_id, request.website.company_id, fields.Date.today(), round=False)
                        )
                    except (ValueError, TypeError):
                        pass
                    redirect_url = decoded_url.replace(query=url_encode(args)).to_url()
            request.session['website_sale_current_pl'] = pricelist.id
            request.website.sale_get_order(update_pricelist=True)
        return request.redirect(redirect_url or '/shop')

    @http.route(['/shop/pricelist'], type='http', auth="public", website=True, sitemap=False)
    def pricelist(self, promo, **post):
        redirect = post.get('r', '/shop/cart')
        # empty promo code is used to reset/remove pricelist (see `sale_get_order()`)
        if promo:
            pricelist_sudo = request.env['product.pricelist'].sudo().search([('code', '=', promo)], limit=1)
            if not (pricelist_sudo and request.website.is_pricelist_available(pricelist_sudo.id)):
                return request.redirect("%s?code_not_available=1" % redirect)

            request.session['website_sale_current_pl'] = pricelist_sudo.id
            # TODO find the best way to create the order with the correct pricelist directly ?
            # not really necessary, but could avoid one write on SO record
            order_sudo = request.website.sale_get_order(force_create=True)
            order_sudo._cart_update_pricelist(pricelist_id=pricelist_sudo.id)
        else:
            order_sudo = request.website.sale_get_order()
            if order_sudo:
                order_sudo._cart_update_pricelist(update_pricelist=True)
        return request.redirect(redirect)

    def _cart_values(self, **post):
        """
        This method is a hook to pass additional values when rendering the 'website_sale.cart' template (e.g. add
        a flag to trigger a style variation)
        """
        return {}

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
        if order and order.carrier_id:
            # Express checkout is based on the amout of the sale order. If there is already a
            # delivery line, Express Checkout form will display and compute the price of the
            # delivery two times (One already computed in the total amount of the SO and one added
            # in the form while selecting the delivery carrier)
            order._remove_delivery_line()

        request.session['website_sale_cart_quantity'] = order.cart_quantity

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
            order.order_line.filtered(lambda l: l.product_id and not l.product_id.active).unlink()
            values['suggested_products'] = order._cart_accessories()
            values.update(self._get_express_shop_payment_values(order))

        values.update(self._cart_values(**post))
        return request.render("website_sale.cart", values)

    @http.route(['/shop/cart/update'], type='http', auth="public", methods=['POST'], website=True)
    def cart_update(
        self, product_id, add_qty=1, set_qty=0,
        product_custom_attribute_values=None, no_variant_attribute_values=None,
        express=False, **kwargs
    ):
        """This route is called when adding a product to cart (no options)."""
        sale_order = request.website.sale_get_order(force_create=True)
        if sale_order.state != 'draft':
            request.session['sale_order_id'] = None
            sale_order = request.website.sale_get_order(force_create=True)

        if product_custom_attribute_values:
            product_custom_attribute_values = json_scriptsafe.loads(product_custom_attribute_values)

        if no_variant_attribute_values:
            no_variant_attribute_values = json_scriptsafe.loads(no_variant_attribute_values)

        sale_order._cart_update(
            product_id=int(product_id),
            add_qty=add_qty,
            set_qty=set_qty,
            product_custom_attribute_values=product_custom_attribute_values,
            no_variant_attribute_values=no_variant_attribute_values,
            **kwargs
        )

        request.session['website_sale_cart_quantity'] = sale_order.cart_quantity

        if express:
            return request.redirect("/shop/checkout?express=1")

        return request.redirect("/shop/cart")

    @http.route(['/shop/cart/update_json'], type='json', auth="public", methods=['POST'], website=True, csrf=False)
    def cart_update_json(
        self, product_id, line_id=None, add_qty=None, set_qty=None, display=True,
        product_custom_attribute_values=None, no_variant_attribute_values=None, **kw
    ):
        """
        This route is called :
            - When changing quantity from the cart.
            - When adding a product from the wishlist.
            - When adding a product to cart on the same page (without redirection).
        """
        order = request.website.sale_get_order(force_create=True)
        if order.state != 'draft':
            request.website.sale_reset()
            if kw.get('force_create'):
                order = request.website.sale_get_order(force_create=True)
            else:
                return {}

        if product_custom_attribute_values:
            product_custom_attribute_values = json_scriptsafe.loads(product_custom_attribute_values)

        if no_variant_attribute_values:
            no_variant_attribute_values = json_scriptsafe.loads(no_variant_attribute_values)

        values = order._cart_update(
            product_id=product_id,
            line_id=line_id,
            add_qty=add_qty,
            set_qty=set_qty,
            product_custom_attribute_values=product_custom_attribute_values,
            no_variant_attribute_values=no_variant_attribute_values,
            **kw
        )

        values['notification_info'] = self._get_cart_notification_information(order, [values['line_id']])
        values['notification_info']['warning'] = values.pop('warning', '')
        request.session['website_sale_cart_quantity'] = order.cart_quantity

        if not order.cart_quantity:
            request.website.sale_reset()
            return values

        values['cart_quantity'] = order.cart_quantity
        values['minor_amount'] = payment_utils.to_minor_currency_units(
            order.amount_total, order.currency_id
        ),
        values['amount'] = order.amount_total

        if not display:
            return values

        values['cart_ready'] = order._is_cart_ready()
        values['website_sale.cart_lines'] = request.env['ir.ui.view']._render_template(
            "website_sale.cart_lines", {
                'website_sale_order': order,
                'date': fields.Date.today(),
                'suggested_products': order._cart_accessories()
            }
        )
        values['website_sale.total'] = request.env['ir.ui.view']._render_template(
            "website_sale.total", {
                'website_sale_order': order,
            }
        )
        return values

    @http.route('/shop/save_shop_layout_mode', type='json', auth='public', website=True)
    def save_shop_layout_mode(self, layout_mode):
        assert layout_mode in ('grid', 'list'), "Invalid shop layout mode"
        request.session['website_sale_shop_layout_mode'] = layout_mode

    @http.route(['/shop/cart/quantity'], type='json', auth="public", methods=['POST'], website=True, csrf=False)
    def cart_quantity(self):
        if 'website_sale_cart_quantity' not in request.session:
            return request.website.sale_get_order().cart_quantity
        return request.session['website_sale_cart_quantity']

    @http.route(['/shop/cart/clear'], type='json', auth="public", website=True)
    def clear_cart(self):
        order = request.website.sale_get_order()
        for line in order.order_line:
            line.unlink()

    def _get_cart_notification_information(self, order, line_ids):
        """ Get the information about the sale order line to show in the notification.

        :param recordset order: The sale order containing the lines.
        :param list(int) line_ids: The ids of the lines to display in the notification.
        :rtype: dict
        :return: A dict with the following structure:
            {
                'currency_id': int
                'lines': [{
                    'id': int
                    'image_url': int
                    'quantity': float
                    'name': str
                    'description': str
                    'line_price_total': float
                }],
            }
        """
        lines = order.order_line.filtered(lambda line: line.id in line_ids)
        if not lines:
            return {}

        show_tax = order.website_id.show_line_subtotals_tax_selection == 'tax_included'
        return {
            'currency_id': order.currency_id.id,
            'lines': [
                { # For the cart_notification
                    'id': line.id,
                    'image_url': order.website_id.image_url(line.product_id, 'image_128'),
                    'quantity': line._get_displayed_quantity(),
                    'name': line.name_short,
                    'description': line._get_sale_order_line_multiline_description_variants(),
                    'line_price_total': line.price_total if show_tax else line.price_subtotal,
                } for line in lines
            ],
        }

    # ------------------------------------------------------
    # Checkout
    # ------------------------------------------------------

    def checkout_check_address(self, order):
        partner_invoice = order.partner_invoice_id
        if not self._check_billing_partner_mandatory_fields(partner_invoice):
            return request.redirect('/shop/address?partner_id=%d&mode=billing' % partner_invoice.id)

        partner_shipping = order.partner_shipping_id
        if not order.only_services and not self._check_shipping_partner_mandatory_fields(partner_shipping):
            return request.redirect('/shop/address?partner_id=%d&mode=shipping' % partner_shipping.id)

    def checkout_redirection(self, order):
        # must have a draft sales order with lines at this point, otherwise reset
        if not order or order.state != 'draft':
            request.session['sale_order_id'] = None
            request.session['sale_transaction_id'] = None
            return request.redirect('/shop')

        if order and not order.order_line:
            return request.redirect('/shop/cart')

        if request.website.is_public_user() and request.website.account_on_checkout == 'mandatory':
            return request.redirect('/web/login?redirect=/shop/checkout')

        # if transaction pending / done: redirect to confirmation
        tx = request.env.context.get('website_sale_transaction')
        if tx and tx.state != 'draft':
            return request.redirect('/shop/payment/confirmation/%s' % order.id)

    def checkout_values(self, order, **kw):
        order = order or request.website.sale_get_order(force_create=True)
        bill_partners = []
        ship_partners = []
        if not order._is_public_order():
            Partner = order.partner_id.with_context(show_address=1).sudo()
            commercial_partner = order.partner_id.commercial_partner_id
            bill_partners = Partner.search([
                ("id", "child_of", commercial_partner.ids),
                '|', ("type", "in", ["invoice", "other"]), ("id", "=", commercial_partner.id)
            ], order='id desc') | order.partner_id
            ship_partners = Partner.search([
                ("id", "child_of", commercial_partner.ids),
                '|', ("type", "in", ["delivery", "other"]), ("id", "=", commercial_partner.id)
            ], order='id desc') | order.partner_id

            # do not show commercial_partner_id if its mandatory fields are not complete to children
            # as children can not edit (fill) the commercial_partner_id
            if commercial_partner != order.partner_id:
                if not self._check_billing_partner_mandatory_fields(commercial_partner):
                    bill_partners = bill_partners.filtered(lambda p: p.id != commercial_partner.id)

                if not self._check_shipping_partner_mandatory_fields(commercial_partner):
                    ship_partners = ship_partners.filtered(lambda p: p.id != commercial_partner.id)

        return {
            'order': order,
            'website_sale_order': order,
            'shippings': ship_partners,
            'billings': bill_partners,
            'only_services': order and order.only_services or False
        }

    def _check_billing_partner_mandatory_fields(self, partner_id):
        ''' return True if all mandatory fields for billing address are complete '''
        billing_fields_required = self._get_mandatory_fields_billing(partner_id.country_id.id)
        return all(partner_id.read(billing_fields_required)[0].values())

    def _get_mandatory_fields_billing(self, country_id=False):
        req = ["name", "email", "street", "city", "country_id"]
        if country_id:
            country = request.env['res.country'].browse(country_id)
            if country.state_required:
                req += ['state_id']
            if country.zip_required:
                req += ['zip']
        return req

    def _check_shipping_partner_mandatory_fields(self, partner_id):
        ''' return True if all mandatory fields for shipping address are complete '''
        shipping_fields_required = self._get_mandatory_fields_shipping(partner_id.country_id.id)
        return all(partner_id.read(shipping_fields_required)[0].values())

    def _get_mandatory_fields_shipping(self, country_id=False):
        req = ["name", "street", "city", "country_id", "phone"]
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

        partner_su = request.env['res.partner'].sudo()
        if data.get('partner_id'):
            partner_su = request.env['res.partner'].sudo().browse(int(data['partner_id'])).exists()
            if partner_su:
                name_change = 'name' in data and partner_su.name and data['name'] != partner_su.name
                email_change = 'email' in data and partner_su.email and data['email'] != partner_su.email

                # Prevent changing the partner name if invoices have been issued.
                if name_change and not partner_su._can_edit_name():
                    error['name'] = 'error'
                    error_message.append(_(
                        "Changing your name is not allowed once invoices have been issued for your"
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

        update_mode, address_mode = mode
        if address_mode == 'shipping':
            required_fields += self._get_mandatory_fields_shipping(country_id)
        else: # 'billing'
            required_fields += self._get_mandatory_fields_billing(country_id)
            if all_form_values.get('use_same'):
                # If the billing address is also used as shipping one, the phone is required as well
                # because it's required for shipping addresses
                required_fields.append('phone')

            order_sudo = request.website.sale_get_order()
            if (
                # New secondary billing address (SO is not an anonymous cart)
                (update_mode == 'new' and not order_sudo._is_public_order())
                or
                # Editing secondary billing address
                (partner_su and order_sudo.partner_id != partner_su)
            ):
                # Commercial fields managed by the parent partner should not be set or edited
                # through a child billing address.  They should therefore be removed from the
                # required fields.
                for fname in partner_su._commercial_fields():
                    if fname not in data and fname in required_fields:
                        required_fields.remove(fname)

        # error message for empty required fields
        for field_name in required_fields:
            val = data.get(field_name)
            if isinstance(val, str):
                val = val.strip()
            if not val:
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
                partner_dummy.sudo().check_vat()
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

    def values_preprocess(self, values):
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

        update_mode, address_mode = mode
        if update_mode == 'new':
            commercial_partner = order.partner_id.commercial_partner_id
            lang = request.lang.code if request.lang.code in request.website.mapped('language_ids.code') else None
            if lang:
                new_values['lang'] = lang
            new_values['company_id'] = request.website.company_id.id
            new_values['team_id'] = request.website.salesteam_id and request.website.salesteam_id.id
            new_values['user_id'] = request.website.salesperson_id.id

            if address_mode == 'billing':
                is_public_order = order._is_public_order()
                if is_public_order:
                    # New billing address of public customer will be their contact address.
                    new_values['type'] = 'contact'
                elif values.get('use_same'):
                    new_values['type'] = 'other'
                else:
                    new_values['type'] = 'invoice'

                # for public user avoid linking to default archived 'Public user' partner
                if commercial_partner.active:
                    new_values['parent_id'] = commercial_partner.id
            elif address_mode == 'shipping':
                new_values['type'] = 'delivery'
                new_values['parent_id'] = commercial_partner.id
        return new_values, errors, error_msg

    @http.route(['/shop/address'], type='http', methods=['GET', 'POST'], auth="public", website=True, sitemap=False)
    def address(self, **kw):
        Partner = request.env['res.partner'].with_context(show_address=1).sudo()
        order = request.website.sale_get_order()

        redirection = self.checkout_redirection(order)
        if redirection:
            return redirection

        can_edit_vat = False
        values, errors = {}, {}

        partner_id = int(kw.get('partner_id', -1))
        if order._is_public_order():
            mode = ('new', 'billing')
            can_edit_vat = True
        else:  # IF ORDER LINKED TO A PARTNER
            if partner_id > 0:
                if partner_id == order.partner_id.id:
                    # If we modify the main customer of the SO ->
                    # 'billing' bc billing requirements are higher than shipping ones
                    can_edit_vat = order.partner_id.can_edit_vat()
                    mode = ('edit', 'billing')
                else:
                    address_mode = kw.get('mode')
                    if not address_mode:
                        address_mode = 'shipping'
                        if partner_id == order.partner_invoice_id.id:
                            address_mode = 'billing'

                    # Make sure the address exists and belongs to the customer of the SO
                    partner_sudo = Partner.browse(partner_id).exists()
                    partners_sudo = Partner.search(
                        [('id', 'child_of', order.partner_id.commercial_partner_id.ids)]
                    )
                    mode = ('edit', address_mode)
                    if address_mode == 'billing':
                        billing_partners = partners_sudo.filtered(lambda p: p.type != 'delivery')
                        if partner_sudo not in billing_partners:
                            raise Forbidden()
                    else:
                        shipping_partners = partners_sudo.filtered(lambda p: p.type != 'invoice')
                        if partner_sudo not in shipping_partners:
                            raise Forbidden()

                    can_edit_vat = partner_sudo.can_edit_vat()

                if mode and partner_id != -1:
                    values = Partner.browse(partner_id)
            elif partner_id == -1:
                mode = ('new', kw.get('mode') or 'shipping')
            else: # no mode - refresh without post?
                return request.redirect('/shop/checkout')

        # IF POSTED
        if 'submitted' in kw and request.httprequest.method == "POST":
            pre_values = self.values_preprocess(kw)
            errors, error_msg = self.checkout_form_validate(mode, kw, pre_values)
            post, errors, error_msg = self.values_postprocess(order, mode, pre_values, errors, error_msg)

            if errors:
                errors['error_message'] = error_msg
                values = kw
            else:
                update_mode, address_mode = mode
                partner_id = self._checkout_form_save(mode, post, kw)
                # We need to validate _checkout_form_save return, because when partner_id not in shippings
                # it returns Forbidden() instead the partner_id
                if isinstance(partner_id, Forbidden):
                    return partner_id

                fpos_before = order.fiscal_position_id
                update_values = {}
                if update_mode == 'new':  # New address
                    if order._is_public_order():
                        update_values['partner_id'] = partner_id

                    if address_mode == 'billing':
                        update_values['partner_invoice_id'] = partner_id
                        if kw.get('use_same'):
                            update_values['partner_shipping_id'] = partner_id
                        elif (
                            order._is_public_order()
                            and not kw.get('callback')
                            and not order.only_services
                        ):
                            # Now that the billing is set, if shipping is necessary
                            # request the customer to fill the shipping address
                            kw['callback'] = '/shop/address'
                    elif address_mode == 'shipping':
                        update_values['partner_shipping_id'] = partner_id
                elif update_mode == 'edit':  # Updating an existing address
                    if order.partner_id.id == partner_id:
                        # Editing the main partner of the SO --> also trigger a partner update to
                        # recompute fpos & any partner-related fields
                        update_values['partner_id'] = partner_id

                    if address_mode == 'billing':
                        update_values['partner_invoice_id'] = partner_id
                        if not kw.get('callback') and not order.only_services:
                            kw['callback'] = '/shop/checkout'
                    elif address_mode == 'shipping':
                        update_values['partner_shipping_id'] = partner_id

                order.write(update_values)

                if order.fiscal_position_id != fpos_before:
                    # Recompute taxes on fpos change
                    # TODO recompute all prices too to correctly manage price_include taxes ?
                    order._recompute_taxes()

                if 'partner_id' in update_values:
                    # Force recomputation of pricelist on main customer address update
                    request.website.sale_get_order(update_pricelist=True)

                # TDE FIXME: don't ever do this
                # -> TDE: you are the guy that did what we should never do in commit e6f038a
                order.message_partner_ids = [(4, order.partner_id.id), (3, request.website.partner_id.id)]
                if not errors:
                    return request.redirect(kw.get('callback') or '/shop/confirm_order')

        is_public_user = request.website.is_public_user()
        render_values = {
            'website_sale_order': order,
            'partner_id': partner_id,
            'mode': mode,
            'checkout': values,
            'can_edit_vat': can_edit_vat,
            'error': errors,
            'callback': kw.get('callback'),
            'only_services': order and order.only_services,
            'account_on_checkout': request.website.account_on_checkout,
            'is_public_user': is_public_user,
            'is_public_order': order._is_public_order(),
            'use_same': is_public_user or ('use_same' in kw and str2bool(kw.get('use_same') or '0')),
        }
        render_values.update(self._get_country_related_render_values(kw, render_values))
        return request.render("website_sale.address", render_values)

    @http.route(
        _express_checkout_route, type='json', methods=['POST'], auth="public", website=True,
        sitemap=False
    )
    def process_express_checkout(
            self, billing_address, shipping_address=None, shipping_option=None, **kwargs
        ):
        """ Records the partner information on the order when using express checkout flow.

        Depending on whether the partner is registered and logged in, either creates a new partner
        or uses an existing one that matches all received data.

        :param dict billing_address: Billing information sent by the express payment form.
        :param dict shipping_address: Shipping information sent by the express payment form.
        :param dict shipping_option: Carrier information sent by the express payment form.
        :param dict kwargs: Optional data. This parameter is not used here.
        :return int: The order's partner id.
        """

        order_sudo = request.website.sale_get_order()
        public_partner = request.website.partner_id

        # Update the partner with all the information
        self._include_country_and_state_in_address(billing_address)
        if order_sudo.partner_id == public_partner:
            billing_partner_id = self._create_or_edit_partner(billing_address, type='invoice')
            order_sudo.partner_id = billing_partner_id
            # Pricelist are recomputed every time the partner is changed. We don't want to recompute
            # the price with another pricelist at this state since the customer has already accepted
            # the amount and validated the payment.
            order_sudo.env.remove_to_compute(
                order_sudo.env['sale.order']._fields['pricelist_id'], order_sudo
            )
            order_sudo.message_partner_ids = request.env['res.partner'].browse(billing_partner_id)
        elif any(billing_address[k] != order_sudo.partner_invoice_id[k] for k in billing_address):
            # Check if a child partner doesn't already exist with the same informations. The
            # phone isn't always checked because it isn't sent in shipping information with
            # Google Pay.
            child_partner_id = self._find_child_partner(
                order_sudo.partner_id.commercial_partner_id.id, billing_address
            )
            order_sudo.partner_invoice_id = child_partner_id or self._create_or_edit_partner(
                billing_address, type='invoice', parent_id=order_sudo.partner_id.id
            )

        # In a non-express flow, `sale_last_order_id` would be added in the session before the
        # payment. As we skip all the steps with the express checkout, `sale_last_order_id` must be
        # assigned to ensure the right behavior from `shop_payment_confirmation()`.
        request.session['sale_last_order_id'] = order_sudo.id

        if shipping_address:
            #in order to not override shippig address, it's checked separately from shipping option
            self._include_country_and_state_in_address(shipping_address)

            if order_sudo.partner_shipping_id.name.endswith(order_sudo.name):
                # The existing partner was created by `process_express_checkout_delivery_choice`, it
                # means that the partner is missing information, so we update it.
                order_sudo.partner_shipping_id = self._create_or_edit_partner(
                    shipping_address,
                    edit=True,
                    type='delivery',
                    partner_id=order_sudo.partner_shipping_id.id,
                )
            elif any(
                shipping_address[k] != order_sudo.partner_shipping_id[k] for k in shipping_address
            ):
                # The sale order's shipping partner's address is different from the one received. If
                # all the sale order's child partners' address differs from the one received, we
                # create a new partner. The phone isn't always checked because it isn't sent in
                # shipping information with Google Pay.
                child_partner_id = self._find_child_partner(
                    order_sudo.partner_id.commercial_partner_id.id, shipping_address
                )
                order_sudo.partner_shipping_id = child_partner_id or self._create_or_edit_partner(
                    shipping_address, type='delivery', parent_id=order_sudo.partner_id.id
                )
            # Process the delivery carrier
            if shipping_option:
                order_sudo._check_carrier_quotation(force_carrier_id=int(shipping_option['id']))

        return order_sudo.partner_id.id

    def _find_child_partner(self, commercial_partner_id, address):
        """ Find a child partner for a specified address

        Compare all keys in the `address` dict with the same keys on the partner object and return
        the id of the first partner that have the same value than in the dict for all the keys.

        :param int commercial_partner_id: commercial partner for whom we need to find his children.
        :param dict address: dictionary of address fields.
        :return int: id of the first child partner that match the criteria, if any.
        """
        partners_sudo = request.env['res.partner'].with_context(show_address=1).sudo().search([
            ('id', 'child_of', commercial_partner_id),
        ])
        for partner_sudo in partners_sudo:
            if all(address[k] == partner_sudo[k] for k in address):
                return partner_sudo.id
        return False

    def _include_country_and_state_in_address(self, address):
        """ This function is used to include country_id and state_id in address.

        Fetch country and state and include the records in address. The object is included to
        simplify the comparison of addresses.

        :param dict address: An address with country and state defined in ISO 3166.
        :return None:
        """
        country = request.env["res.country"].search([
            ('code', '=', address.pop('country')),
        ], limit=1)
        state = request.env["res.country.state"].search([
            ('code', '=', address.pop('state', '')),
            ('country_id', '=', country.id),
        ], limit=1)
        address.update(country_id=country, state_id=state)

    def _create_or_edit_partner(self, partner_details, edit=False, **custom_values):
        """ Create or update a partner

        To create a partner, this controller usually calls `values_preprocess()`, then
        `checkout_form_validate()`, then `values_postprocess()` and finally `_checkout_form_save()`.
        Since these methods are very specific to the checkout form, this method makes it possible to
        create  a partner for more specific flows like express payment, which does not require all
        the checks carried out by the previous methods. Parts of code in this method come from those.

        :param dict partner_details: The values needed to create the partner or to edit the partner.
        :param bool edit: Whether edit an existing partner or create one, defaults to False.
        :param dict custom_values: Optional custom values for the creation or edition.
        :return int: The id of the partner created or edited
        """
        request.update_env(context=request.website.env.context)
        values = self.values_preprocess(partner_details)

        # Ensure that we won't write on unallowed fields.
        sanitized_values = {
            k: v for k, v in values.items() if k in self.WRITABLE_PARTNER_FIELDS
        }
        sanitized_custom_values = {
            k: v for k, v in custom_values.items()
            if k in self.WRITABLE_PARTNER_FIELDS + ['partner_id', 'parent_id', 'type']
        }

        if request.website.specific_user_account:
            sanitized_values['website_id'] = request.website.id

        lang = request.lang.code if request.lang.code in request.website.mapped(
            'language_ids.code'
        ) else None
        if lang:
            sanitized_values['lang'] = lang

        partner_id = sanitized_custom_values.get('partner_id')
        if edit and partner_id:
            request.env['res.partner'].browse(partner_id).sudo().write(sanitized_values)
        else:
            sanitized_values = dict(sanitized_values, **{
                'company_id': request.website.company_id.id,
                'team_id': request.website.salesteam_id and request.website.salesteam_id.id,
                'user_id': request.website.salesperson_id.id,
                **sanitized_custom_values
            })
            partner_id = request.env['res.partner'].sudo().with_context(
                tracking_disable=True
            ).create(sanitized_values).id
        return partner_id

    def _get_country_related_render_values(self, kw, render_values):
        """ Provide the fields related to the country to render the website sale form """
        values = render_values['checkout']
        mode = render_values['mode']
        order = render_values['website_sale_order']

        def_country_id = order.partner_id.country_id
        if order._is_public_order():
            if request.geoip.country_code:
                def_country_id = request.env['res.country'].search([('code', '=', request.geoip.country_code)], limit=1)
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
        order_sudo = request.website.sale_get_order()
        request.session['sale_last_order_id'] = order_sudo.id
        redirection = self.checkout_redirection(order_sudo)
        if redirection:
            return redirection

        if order_sudo._is_public_order():
            return request.redirect('/shop/address')

        redirection = self.checkout_check_address(order_sudo)
        if redirection:
            return redirection

        if post.get('express'):
            return request.redirect('/shop/confirm_order')

        values = self.checkout_values(order_sudo, **post)

        # Avoid useless rendering if called in ajax
        if post.get('xhr'):
            return 'ok'
        return request.render("website_sale.checkout", values)

    @route('/shop/cart/update_address', type='http', auth='public', methods=['POST'], website=True)
    def update_cart_address(self, partner_id, mode='billing', **kw):
        partner_id = int(partner_id)

        order_sudo = request.website.sale_get_order()
        if not order_sudo:
            return

        ResPartner = request.env['res.partner'].sudo()
        partner_sudo = ResPartner.browse(partner_id).exists()
        children = ResPartner._search([
            ('id', 'child_of', order_sudo.partner_id.commercial_partner_id.id),
            ('type', 'in', ('invoice', 'delivery', 'other')),
        ])
        if (
            partner_sudo != order_sudo.partner_id
            and partner_sudo != order_sudo.partner_id.commercial_partner_id
            and partner_sudo.id not in children
        ):
            raise Forbidden()

        fpos_before = order_sudo.fiscal_position_id
        if (
            mode == 'billing'
            and partner_sudo != order_sudo.partner_invoice_id
        ):
            order_sudo.partner_invoice_id = partner_id
        elif (
            mode == 'shipping'
            and partner_sudo != order_sudo.partner_shipping_id
        ):
            order_sudo.partner_shipping_id = partner_id
            if order_sudo.carrier_id:
                # update carrier rates on shipping address change
                order_sudo._check_carrier_quotation(force_carrier_id=order_sudo.carrier_id.id)
        else:
            # TODO someday we should gracefully handle invalid addresses
            return

        if fpos_before != order_sudo.fiscal_position_id:
            # TODO recompute full cart amounts to correctly handle price_include taxes stuff ?
            order_sudo._recompute_taxes()

    @http.route(['/shop/confirm_order'], type='http', auth="public", website=True, sitemap=False)
    def confirm_order(self, **post):
        order = request.website.sale_get_order()

        redirection = self.checkout_redirection(order) or self.checkout_check_address(order)
        if redirection:
            return redirection

        order.order_line._compute_tax_id()
        request.website.sale_get_order(update_pricelist=True)
        extra_step = request.website.viewref('website_sale.extra_info')
        if extra_step.active:
            return request.redirect("/shop/extra_info")

        return request.redirect("/shop/payment")

    # ------------------------------------------------------
    # Extra step
    # ------------------------------------------------------
    @http.route(['/shop/extra_info'], type='http', auth="public", website=True, sitemap=False)
    def extra_info(self, **post):
        # Check that this option is activated
        extra_step = request.website.viewref('website_sale.extra_info')
        if not extra_step.active:
            return request.redirect("/shop/payment")

        # check that cart is valid
        order = request.website.sale_get_order()
        redirection = self.checkout_redirection(order)
        open_editor = request.params.get('open_editor') == 'true'
        # Do not redirect if it is to edit
        # (the information is transmitted via the "open_editor" parameter in the url)
        if not open_editor and redirection:
            return redirection

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

    def _get_express_shop_payment_values(self, order, **kwargs):
        payment_form_values = sale_portal.CustomerPortal._get_payment_values(
            self, order, website_id=request.website.id, is_express_checkout=True
        )
        payment_form_values.update({
            'payment_access_token': payment_form_values.pop('access_token'),  # Rename the key.
            'minor_amount': payment_utils.to_minor_currency_units(
                order.amount_total, order.currency_id
            ),
            'merchant_name': request.website.name,
            'transaction_route': f'/shop/payment/transaction/{order.id}',
            'express_checkout_route': self._express_checkout_route,
            'landing_route': '/shop/payment/validate',
            'payment_method_unknown_id': request.env.ref('payment.payment_method_unknown').id,
            'shipping_info_required': not order.only_services,
            'shipping_address_update_route': self._express_checkout_shipping_route,
        })
        if request.website.is_public_user():
            payment_form_values['partner_id'] = -1
        return payment_form_values

    def _get_shop_payment_values(self, order, **kwargs):
        checkout_page_values = {
            'website_sale_order': order,
            'errors': self._get_shop_payment_errors(order),
            'partner': order.partner_invoice_id,
            'order': order,
            'submit_button_label': _("Pay now"),
            'payment_action_id': request.env.ref('payment.action_payment_provider').id,
            'action_activate_stripe_id': request.env.ref(
                'website_payment.action_activate_stripe'
            ).id,
        }
        payment_form_values = {
            **sale_portal.CustomerPortal._get_payment_values(
                self, order, website_id=request.website.id
            ),
            'display_submit_button': False,  # The submit button is re-added outside the form.
            'transaction_route': f'/shop/payment/transaction/{order.id}',
            'landing_route': '/shop/payment/validate',
            'sale_order_id': order.id,  # Allow Stripe to check if tokenization is required.
        }
        values = {**checkout_page_values, **payment_form_values}
        if request.website.enabled_delivery:
            has_storable_products = any(
                line.product_id.type in ['consu', 'product'] for line in order.order_line
            )
            if has_storable_products:
                if order.carrier_id and not order.delivery_rating_success:
                    order._remove_delivery_line()
                    order._check_carrier_quotation()
                values['deliveries'] = order._get_delivery_methods().sudo()

            values['delivery_has_storable'] = has_storable_products
            values['delivery_action_id'] = request.env.ref(
                'delivery.action_delivery_carrier_form'
            ).id

        return values

    def _get_shop_payment_errors(self, order):
        """ Check that there is no error that should block the payment.

        :param sale.order order: The sales order to pay
        :return: A list of errors (error_title, error_message)
        :rtype: list[tuple]
        """
        errors = []

        if not order.only_services and not order._get_delivery_methods():
            errors.append((
                _('Sorry, we are unable to ship your order'),
                _('No shipping method is available for your current order and shipping address. '
                   'Please contact us for more information.'),
            ))
        return errors

    @http.route('/shop/payment', type='http', auth='public', website=True, sitemap=False)
    def shop_payment(self, **post):
        """ Payment step. This page proposes several payment means based on available
        payment.provider. State at this point :

         - a draft sales order with lines; otherwise, clean context / session and
           back to the shop
         - no transaction in context / session, or only a draft one, if the customer
           did go to a payment.provider website but closed the tab without
           paying / canceling
        """
        order = request.website.sale_get_order()

        if order and not order.only_services and (request.httprequest.method == 'POST' or not order.carrier_id):
            # Update order's carrier_id (will be the one of the partner if not defined)
            # If a carrier_id is (re)defined, redirect to "/shop/payment" (GET method to avoid infinite loop)
            carrier_id = post.get('carrier_id')
            keep_carrier = post.get('keep_carrier', False)
            if keep_carrier:
                keep_carrier = bool(int(keep_carrier))
            if carrier_id:
                carrier_id = int(carrier_id)
            order._check_carrier_quotation(force_carrier_id=carrier_id, keep_carrier=keep_carrier)
            if carrier_id:
                return request.redirect("/shop/payment")

        redirection = self.checkout_redirection(order) or self.checkout_check_address(order)
        if redirection:
            return redirection

        render_values = self._get_shop_payment_values(order, **post)
        render_values['only_services'] = order and order.only_services or False

        if render_values['errors']:
            render_values.pop('payment_methods_sudo', '')
            render_values.pop('tokens_sudo', '')

        return request.render("website_sale.payment", render_values)

    @http.route('/shop/payment/validate', type='http', auth="public", website=True, sitemap=False)
    def shop_payment_validate(self, sale_order_id=None, **post):
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

        errors = self._get_shop_payment_errors(order)
        if errors:
            first_error = errors[0]  # only display first error
            error_msg = f"{first_error[0]}\n{first_error[1]}"
            raise ValidationError(error_msg)

        tx_sudo = order.get_portal_last_transaction() if order else order.env['payment.transaction']

        if not order or (order.amount_total and not tx_sudo):
            return request.redirect('/shop')

        if order and not order.amount_total and not tx_sudo:
            if order.state != 'sale':
                order.with_context(send_email=True).with_user(SUPERUSER_ID).action_confirm()
            request.website.sale_reset()
            return request.redirect(order.get_portal_url())

        # clean context and session, then redirect to the confirmation page
        request.website.sale_reset()
        if tx_sudo and tx_sudo.state == 'draft':
            return request.redirect('/shop')

        return request.redirect('/shop/confirmation')

    @http.route(['/shop/confirmation'], type='http', auth="public", website=True, sitemap=False)
    def shop_payment_confirmation(self, **post):
        """ End of checkout process controller. Confirmation is basically seing
        the status of a sale.order. State at this point :

         - should not have any context / session info: clean them
         - take a sale.order id, because we request a sale.order and are not
           session dependant anymore
        """
        sale_order_id = request.session.get('sale_last_order_id')
        if sale_order_id:
            order = request.env['sale.order'].sudo().browse(sale_order_id)
            values = self._prepare_shop_payment_confirmation_values(order)
            return request.render("website_sale.confirmation", values)
        else:
            return request.redirect('/shop')

    def _prepare_shop_payment_confirmation_values(self, order):
        """
        This method is called in the payment process route in order to prepare the dict
        containing the values to be rendered by the confirmation template.
        """
        return {
            'order': order,
            'website_sale_order': order,
            'order_tracking_info': self.order_2_return_dict(order),
        }

    @http.route(['/shop/print'], type='http', auth="public", website=True, sitemap=False)
    def print_saleorder(self, **kwargs):
        sale_order_id = request.session.get('sale_last_order_id')
        if sale_order_id:
            pdf, _ = request.env['ir.actions.report'].sudo()._render_qweb_pdf('sale.action_report_saleorder', [sale_order_id])
            pdfhttpheaders = [('Content-Type', 'application/pdf'), ('Content-Length', u'%s' % len(pdf))]
            return request.make_response(pdf, headers=pdfhttpheaders)
        else:
            return request.redirect('/shop')

    # ------------------------------------------------------
    # Edit
    # ------------------------------------------------------

    @http.route(['/shop/config/product'], type='json', auth='user')
    def change_product_config(self, product_id, **options):
        if not request.env.user.has_group('website.group_website_restricted_editor'):
            raise NotFound()

        product = request.env['product.template'].browse(product_id)
        if "sequence" in options:
            sequence = options["sequence"]
            if sequence == "top":
                product.set_sequence_top()
            elif sequence == "bottom":
                product.set_sequence_bottom()
            elif sequence == "up":
                product.set_sequence_up()
            elif sequence == "down":
                product.set_sequence_down()
        if {"x", "y"} <= set(options):
            product.write({'website_size_x': options["x"], 'website_size_y': options["y"]})

    @http.route(['/shop/config/attribute'], type='json', auth='user')
    def change_attribute_config(self, attribute_id, **options):
        if not request.env.user.has_group('website.group_website_restricted_editor'):
            raise NotFound()

        attribute = request.env['product.attribute'].browse(attribute_id)
        if 'display_type' in options:
            attribute.write({'display_type': options['display_type']})
            request.env.registry.clear_cache('templates')

    @http.route(['/shop/config/website'], type='json', auth='user')
    def _change_website_config(self, **options):
        if not request.env.user.has_group('website.group_website_restricted_editor'):
            raise NotFound()

        current_website = request.env['website'].get_current_website()
        # Restrict options we can write to.
        writable_fields = {
            'shop_ppg', 'shop_ppr', 'shop_default_sort',
            'product_page_image_layout', 'product_page_image_width',
            'product_page_grid_columns', 'product_page_image_spacing'
        }
        # Default ppg to 1.
        if 'ppg' in options and not options['ppg']:
            options['ppg'] = 1
        if 'product_page_grid_columns' in options:
            options['product_page_grid_columns'] = int(options['product_page_grid_columns'])

        write_vals = {k: v for k, v in options.items() if k in writable_fields}
        if write_vals:
            current_website.write(write_vals)

    def order_lines_2_google_api(self, order_lines):
        """ Transforms a list of order lines into a dict for google analytics """
        ret = []
        for line in order_lines.filtered(lambda line: not line.is_delivery):
            product = line.product_id
            ret.append({
                'item_id': product.barcode or product.id,
                'item_name': product.name or '-',
                'item_category': product.categ_id.name or '-',
                'price': line.price_unit,
                'quantity': line.product_uom_qty,
            })
        return ret

    def order_2_return_dict(self, order):
        """ Returns the tracking_cart dict of the order for Google analytics basically defined to be inherited """
        tracking_cart_dict = {
            'transaction_id': order.id,
            'affiliation': order.company_id.name,
            'value': order.amount_total,
            'tax': order.amount_tax,
            'currency': order.currency_id.name,
            'items': self.order_lines_2_google_api(order.order_line),
        }
        delivery_line = order.order_line.filtered('is_delivery')
        if delivery_line:
            tracking_cart_dict['shipping'] = delivery_line.price_unit
        return tracking_cart_dict

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
    # Products Recently Viewed
    # --------------------------------------------------------------------------
    @http.route('/shop/products/recently_viewed_update', type='json', auth='public', website=True)
    def products_recently_viewed_update(self, product_id, **kwargs):
        res = {}
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request(force_create=True)
        visitor_sudo._add_viewed_product(product_id)
        return res

    @http.route('/shop/products/recently_viewed_delete', type='json', auth='public', website=True)
    def products_recently_viewed_delete(self, product_id, **kwargs):
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            request.env['website.track'].sudo().search([('visitor_id', '=', visitor_sudo.id), ('product_id', '=', product_id)]).unlink()
        return {}


class PaymentPortal(payment_portal.PaymentPortal):

    def _validate_transaction_for_order(self, transaction, sale_order_id):
        """
        Perform final checks against the transaction & sale_order.
        Override me to apply payment unrelated checks & processing
        """
        return

    @http.route(
        '/shop/payment/transaction/<int:order_id>', type='json', auth='public', website=True
    )
    def shop_payment_transaction(self, order_id, access_token, **kwargs):
        """ Create a draft transaction and return its processing values.

        :param int order_id: The sales order to pay, as a `sale.order` id
        :param str access_token: The access token used to authenticate the request
        :param dict kwargs: Locally unused data passed to `_create_transaction`
        :return: The mandatory values for the processing of the transaction
        :rtype: dict
        :raise: UserError if the order has already been paid or has an ongoing transaction
        :raise: ValidationError if the invoice id or the access token is invalid
        """
        # Check the order id and the access token
        # Then lock it during the transaction to prevent concurrent payments
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token)
            request.env.cr.execute(
                SQL('SELECT 1 FROM sale_order WHERE id = %s FOR NO KEY UPDATE NOWAIT', order_id)
            )
        except MissingError as error:
            raise error
        except AccessError:
            raise ValidationError(_("The access token is invalid."))
        except LockNotAvailable:
            raise UserError(_("Payment is already being processed."))

        if order_sudo.state == "cancel":
            raise ValidationError(_("The order has been canceled."))

        order_sudo._check_cart_is_ready_to_be_paid()

        self._validate_transaction_kwargs(kwargs)
        kwargs.update({
            'partner_id': order_sudo.partner_invoice_id.id,
            'currency_id': order_sudo.currency_id.id,
            'sale_order_id': order_id,  # Include the SO to allow Subscriptions to tokenize the tx
        })
        if not kwargs.get('amount'):
            kwargs['amount'] = order_sudo.amount_total

        compare_amounts = order_sudo.currency_id.compare_amounts
        if compare_amounts(kwargs['amount'], order_sudo.amount_total):
            raise ValidationError(_("The cart has been updated. Please refresh the page."))
        if compare_amounts(order_sudo.amount_paid, order_sudo.amount_total) == 0:
            raise UserError(_("The cart has already been paid. Please refresh the page."))

        tx_sudo = self._create_transaction(
            custom_create_values={'sale_order_ids': [Command.set([order_id])]}, **kwargs,
        )

        # Store the new transaction into the transaction list and if there's an old one, we remove
        # it until the day the ecommerce supports multiple orders at the same time.
        request.session['__website_sale_last_tx_id'] = tx_sudo.id

        self._validate_transaction_for_order(tx_sudo, order_id)

        return tx_sudo._get_processing_values()


class CustomerPortal(sale_portal.CustomerPortal):

    def _get_payment_values(self, order_sudo, website_id=None, **kwargs):
        """ Override of `sale` to inject the `website_id` into the kwargs.

        :param sale.order order_sudo: The sales order being paid.
        :param int website_id: The website on which the order was made, if any, as a `website` id.
        :param dict kwargs: Locally unused keywords arguments.
        :return: The payment-specific values.
        :rtype: dict
        """
        website_id = website_id or order_sudo.website_id.id
        return super()._get_payment_values(order_sudo, website_id=website_id, **kwargs)

    def _sale_reorder_get_line_context(self):
        return {}

    @http.route('/my/orders/reorder_modal_content', type='json', auth='public', website=True)
    def my_orders_reorder_modal_content(self, order_id, access_token):
        try:
            sale_order = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        currency = request.env['website'].get_current_website().currency_id
        result = {
            'currency': currency.id,
            'products': [],
        }
        for line in sale_order.order_line:
            if line.display_type:
                continue
            if line._is_delivery():
                continue
            combination = line.product_id.product_template_attribute_value_ids | line.product_no_variant_attribute_value_ids
            res = {
                'product_template_id': line.product_id.product_tmpl_id.id,
                'product_id': line.product_id.id,
                'combination': combination.ids,
                'no_variant_attribute_values': [
                    { # Same input format as provided by product configurator
                        'value': ptav.id,
                    } for ptav in line.product_no_variant_attribute_value_ids
                ],
                'product_custom_attribute_values': [
                    { # Same input format as provided by product configurator
                        'custom_product_template_attribute_value_id': pcav.custom_product_template_attribute_value_id.id,
                        'custom_value': pcav.custom_value,
                    } for pcav in line.product_custom_attribute_value_ids
                ],
                'type': line.product_id.type,
                'name': line.name_short,
                'description_sale': line.product_id.description_sale or '' + line._get_sale_order_line_multiline_description_variants(),
                'qty': line.product_uom_qty,
                'add_to_cart_allowed': line.with_user(request.env.user).sudo()._is_reorder_allowed(),
                'has_image': bool(line.product_id.image_128),
            }
            if res['add_to_cart_allowed']:
                res['combinationInfo'] = line.product_id.product_tmpl_id.with_context(
                    **self._sale_reorder_get_line_context()
                )._get_combination_info(combination, res['product_id'], res['qty'])
            else:
                res['combinationInfo'] = {}
            result['products'].append(res)
        return result

```

## File: controllers\variant.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details

import json

from odoo.http import request, route, Controller


class WebsiteSaleVariantController(Controller):

    @route('/website_sale/get_combination_info', type='json', auth='public', methods=['POST'], website=True)
    def get_combination_info_website(
        self, product_template_id, product_id, combination, add_qty, parent_combination=None,
        **kwargs
    ):
        product_template = request.env['product.template'].browse(
            product_template_id and int(product_template_id))

        combination_info = product_template._get_combination_info(
            combination=request.env['product.template.attribute.value'].browse(combination),
            product_id=product_id and int(product_id),
            add_qty=add_qty and float(add_qty) or 1.0,
            parent_combination=request.env['product.template.attribute.value'].browse(parent_combination),
        )

        # Pop data only computed to ease server-side computations.
        for key in ('product_taxes', 'taxes', 'currency', 'date'):
            combination_info.pop(key)

        if request.website.product_page_image_width != 'none' and not request.env.context.get('website_sale_no_images', False):
            combination_info['carousel'] = request.env['ir.ui.view']._render_template(
                'website_sale.shop_product_images',
                values={
                    'product': product_template,
                    'product_variant': request.env['product.product'].browse(combination_info['product_id']),
                    'website': request.env['website'].get_current_website(),
                },
            )
        return combination_info

    @route('/sale/create_product_variant', type='json', auth='public', methods=['POST'])
    def create_product_variant(self, product_template_id, product_template_attribute_value_ids, **kwargs):
        """Old product configurator logic, only used by frontend configurator, will be deprecated soon"""
        return request.env['product.template'].browse(
            int(product_template_id)
        ).create_product_variant(json.loads(product_template_attribute_value_ids))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery
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

        <record id="product_attribute_brand" model="product.attribute">
            <field name="name">Brand</field>
            <field name="sequence">0</field>
        </record>

        <record id="website_sale.sale_ribbon" model="product.ribbon">
            <field name="html">Sale</field>
            <field name="html_class">o_ribbon_left</field>
            <field name="bg_color">rgb(40, 167, 69)</field>
            <field name="text_color">white</field>
        </record>

        <record id="website_sale.sold_out_ribbon" model="product.ribbon">
            <field name="html">Sold out</field>
            <field name="html_class">o_ribbon_left</field>
            <field name="bg_color">rgb(220, 53, 69)</field>
            <field name="text_color">white</field>
        </record>

        <record id="website_sale.out_of_stock_ribbon" model="product.ribbon">
            <field name="html">Out of stock</field>
            <field name="html_class">o_ribbon_left</field>
            <field name="bg_color">rgb(255, 193, 7)</field>
            <field name="text_color">black</field>
        </record>

        <record id="website_sale.new_ribbon" model="product.ribbon">
            <field name="html">New!</field>
            <field name="html_class">o_ribbon_left</field>
            <field name="bg_color">rgb(0, 123, 255)</field>
            <field name="text_color">white</field>
        </record>

        <record id="sales_team.salesteam_website_sales" model="crm.team">
            <field name="active" eval="True"/>
        </record>

        <record model="website" id="website.default_website">
            <field name="salesteam_id" ref="sales_team.salesteam_website_sales"/>
        </record>

        <record id="delivery.free_delivery_carrier" model="delivery.carrier" forcecreate="False">
            <field name="is_published" eval="True"/>
        </record>

    </data>
    <data>
        <!-- Filters for Dynamic Filter -->
        <record id="dynamic_snippet_newest_products_filter" model="ir.filters">
            <field name="name">Newest Products</field>
            <field name="model_id">product.product</field>
            <field name="user_id" eval="False" />
            <field name="domain">[('website_published', '=', True)]</field>
            <field name="context">{'display_default_code': False, 'add2cart_rerender': False}</field>
            <field name="sort">["create_date desc"]</field>
            <field name="action_id" ref="website.action_website"/>
        </record>
        <!-- Action Server for Dynamic Filter -->
        <record id="dynamic_snippet_latest_sold_products_action" model="ir.actions.server">
            <field name="name">Recently Sold Products</field>
            <field name="model_id" ref="model_product_product"/>
            <field name="state">code</field>
            <field name="code">
DynamicFilter = model.env['website.snippet.filter']
response = DynamicFilter._get_products('latest_sold', model.env.context)
            </field>
        </record>
        <record id="dynamic_snippet_latest_viewed_products_action" model="ir.actions.server">
            <field name="name">Recently Viewed Products</field>
            <field name="model_id" ref="model_product_product"/>
            <field name="state">code</field>
            <field name="code">
DynamicFilter = model.env['website.snippet.filter']
res_products = DynamicFilter._get_products('latest_viewed', model.env.context)
for data in res_products:
    data['_latest_viewed'] = True
response = res_products
            </field>
        </record>
        <record id="dynamic_snippet_accessories_action" model="ir.actions.server">
            <field name="name">Product Accessories</field>
            <field name="model_id" ref="model_product_product"/>
            <field name="state">code</field>
            <field name="code">
DynamicFilter = model.env['website.snippet.filter']
model.env.context['product_template_id'] = request.params.get('productTemplateId')
response = DynamicFilter._get_products('accessories', model.env.context)
            </field>
        </record>
        <record id="dynamic_snippet_recently_sold_with_action" model="ir.actions.server">
            <field name="name">Products Recently Sold With</field>
            <field name="model_id" ref="model_product_product"/>
            <field name="state">code</field>
            <field name="code">
DynamicFilter = model.env['website.snippet.filter']
model.env.context['product_template_id'] = request.params.get('productTemplateId')
response = DynamicFilter._get_products('recently_sold_with', model.env.context)
            </field>
        </record>
        <record id="dynamic_snippet_alternative_products" model="ir.actions.server">
            <field name="name">Alternative Products</field>
            <field name="model_id" ref="model_product_product"/>
            <field name="state">code</field>
            <field name="code">
DynamicFilter = model.env['website.snippet.filter']
model.env.context['product_template_id'] = request.params.get('productTemplateId')
response = DynamicFilter._get_products('alternative_products', model.env.context)
            </field>
        </record>
        <!-- Dynamic Filter -->
        <record id="dynamic_filter_newest_products" model="website.snippet.filter">
            <field name="filter_id" ref="website_sale.dynamic_snippet_newest_products_filter"/>
            <field name="field_names">display_name,description_sale,image_512</field>
            <field name="limit" eval="16"/>
            <field name="name">Newest Products</field>
        </record>
        <record id="dynamic_filter_latest_sold_products" model="website.snippet.filter">
            <field name="action_server_id" ref="website_sale.dynamic_snippet_latest_sold_products_action"/>
            <field name="field_names">display_name,description_sale,image_512</field>
            <field name="limit" eval="16"/>
            <field name="name">Recently Sold Products</field>
        </record>
        <record id="dynamic_filter_latest_viewed_products" model="website.snippet.filter">
            <field name="action_server_id" ref="website_sale.dynamic_snippet_latest_viewed_products_action"/>
            <field name="field_names">display_name,description_sale,image_512</field>
            <field name="limit" eval="16"/>
            <field name="name">Recently Viewed Products</field>
        </record>
        <record id="dynamic_filter_cross_selling_accessories" model="website.snippet.filter">
            <field name="action_server_id" ref="website_sale.dynamic_snippet_accessories_action"/>
            <field name="field_names">display_name,description_sale,image_512</field>
            <field name="limit" eval="16"/>
            <field name="name">Accessories for Product</field>
            <field name="product_cross_selling">True</field>
        </record>
        <record id="dynamic_filter_cross_selling_recently_sold_with" model="website.snippet.filter">
            <field name="action_server_id" ref="website_sale.dynamic_snippet_recently_sold_with_action"/>
            <field name="field_names">display_name,description_sale,image_512</field>
            <field name="limit" eval="16"/>
            <field name="name">Products Recently Sold With Product</field>
            <field name="product_cross_selling">True</field>
        </record>
        <record id="dynamic_filter_cross_selling_alternative_products" model="website.snippet.filter">
            <field name="action_server_id" ref="website_sale.dynamic_snippet_alternative_products"/>
            <field name="field_names">display_name,description_sale,image_512</field>
            <field name="limit" eval="16"/>
            <field name="name">Alternative Products</field>
            <field name="product_cross_selling">True</field>
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

        <record id="product.product_attribute_2" model="product.attribute">
            <field name="visibility">hidden</field>
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
            <field name="website_sequence">9950</field>
            <field name="website_description" type="html">
                <section class="s_text_image pt32 pb32 o_colored_level o_cc o_cc1" data-snippet="s_text_image" data-name="Text - Image">
                    <div class="container">
                        <div class="row align-items-center">
                            <div class="pt16 pb16 col-lg-6">
                                <h2>Ergonomic</h2>
                                <p>Press a button and watch your desk glide effortlessly from sitting to standing height in seconds.</p>
                                <p>The minimum height is 65 cm, and for standing work the maximum height position is 125 cm.</p>
                            </div>
                            <div class="pt16 pb16 col-lg-6">
                                <img src="/website/static/src/img/snippets_demo/s_text_image.jpg" class="img img-fluid mx-auto" alt=""/>
                            </div>
                        </div>
                    </div>
                </section>
                <section class="s_text_image pt32 pb32 o_colored_level o_cc o_cc1" data-snippet="s_image_text" data-name="Image - Text">
                    <div class="container">
                        <div class="row align-items-center">
                            <div class="pt16 pb16 col-lg-6">
                                <img src="/website_sale/static/src/img/carpentry.jpg" class="img img-fluid mx-auto" alt=""/>
                            </div>
                            <div class="pt16 pb16 col-lg-6">
                                <h2>Locally handmade</h2>
                                <p>We pay special attention to detail, which is why our desks are of a superior quality.</p>
                                <p>Looking for a custom bamboo stain to match existing furniture? Contact us for a quote.</p>
                                <p><a href="/contactus" class="mb-2 btn btn-primary">Contact Us</a></p>
                            </div>
                        </div>
                    </div>
                </section>
            </field>
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

    <!-- product.public.category -->

        <record id="public_category_desks" model="product.public.category">
          <field name="name">Desks</field>
          <field name="sequence">15</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/desks.jpg"/>
        </record>
        <record id="public_category_furnitures" model="product.public.category">
          <field name="name">Furnitures</field>
          <field name="sequence">17</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/furnitures.jpg"/>
        </record>
        <record id="public_category_boxes" model="product.public.category">
            <field name="name">Boxes</field>
            <field name="sequence">20</field>
            <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/boxes.jpg"/>
        </record>
        <record id="public_category_drawers" model="product.public.category">
          <field name="name">Drawers</field>
          <field name="sequence">21</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/drawers.jpg"/>
        </record>
        <record id="public_category_cabinets" model="product.public.category">
          <field name="name">Cabinets</field>
          <field name="sequence">22</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/cabinets.jpg"/>
        </record>
        <record id="public_category_bins" model="product.public.category">
          <field name="name">Bins</field>
          <field name="sequence">23</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/bins.jpg"/>
        </record>
        <record id="public_category_lamps" model="product.public.category">
          <field name="name">Lamps</field>
          <field name="sequence">24</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/lamps.jpg"/>
        </record>
        <record id="services" model="product.public.category">
          <field name="name">Services</field>
          <field name="sequence">25</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/warranty.jpg"/>
        </record>
        <record id="public_category_multimedia" model="product.public.category">
          <field name="name">Multimedia</field>
          <field name="sequence">26</field>
          <field name="image_1920" type="base64" file="product/static/img/product_product_43-image.jpg"/>
        </record>

        <!-- subcategories -->
        <record id="public_category_desks_components" model="product.public.category">
          <field name="parent_id" eval="ref('public_category_desks')"/>
          <field name="name">Components</field>
          <field name="sequence">16</field>
          <field name="image_1920" type="base64" file="website_sale/static/src/img/categories/desk_components.jpg"/>
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

        <record id="benelux" model="res.country.group">
            <field name="name">BeNeLux</field>
            <field name="country_ids" eval="[(6,0,[
                ref('base.be'),ref('base.lu'),ref('base.nl')])]"/>
        </record>

        <!-- Since we are adding pricelists, we activate the feature -->
        <record id="base.group_user" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('product.group_product_pricelist'))]"/>
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
            <field name="team_id" ref="sales_team.salesteam_website_sales"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(hours=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="website_sale_order_line_9" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_9"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="website_sale_order_line_10" model="sale.order.line">
            <field name="order_id" ref="website_sale_order_9"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <!-- Active Carts -->
        <record id="website_sale_order_10" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
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
            <field name="price_unit">33</field>
        </record>

        <!-- Abandoned Carts -->
        <record id="website_sale_order_11" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="website_id" ref="website.default_website"/>
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
            <field name="image_1920" type="base64" file="website_sale/static/src/img/warranty.jpg"/>
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

        <record id="delivery.delivery_carrier" model="delivery.carrier">
            <field name="is_published" eval="False" />
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

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_send_availability_email" model="ir.cron">
        <field name="name">eCommerce: send email to customers about their abandoned cart</field>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
        <field name="model_id" ref="model_website"/>
        <field name="code">model._send_abandoned_cart_email()</field>
        <field name="state">code</field>
    </record>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_sale_cart_recovery" model="mail.template">
            <field name="name">Ecommerce: Cart Recovery</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="subject">You left items in your cart!</field>
            <field name="email_from">{{ (object.user_id.email_formatted or object.company_id.email_formatted or user.email_formatted or '') }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">If the setting is set, sent to authenticated visitors who abandoned their cart</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 0px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 0px 0px 0px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 13px;">
                    <h1 style="color:#A9A9A9;">THERE'S SOMETHING IN YOUR CART.</h1>
                    Would you like to complete your purchase?<br/><br/>
                    <t t-if="object.order_line">
                        <t t-foreach="object.website_order_line" t-as="line">
                            <hr/>
                            <table width="100%">
                                <tr>
                                    <td style="padding: 10px; width:150px;">
                                        <img t-attf-src="/web/image/product.product/{{ line.product_id.id }}/image_128" style="width: 100px; height: 100px; object-fit: contain;" alt="Product image"></img>
                                    </td>
                                    <td>
                                        <strong t-out="line.product_id.display_name or ''">[FURN_7800] Desk Combination</strong><br/><t t-out="line.name or ''">[FURN_7800] Desk Combination Desk combination, black-brown: chair + desk + drawer.</t>
                                    </td>
                                    <td width="100px" align="right">
                                        <t t-out="int(line.product_uom_qty) or ''">10000</t> <t t-out="line.product_uom.name or ''">Units</t>
                                    </td>
                                </tr>
                            </table>
                        </t>
                        <hr/>
                    </t>
                    <div style="text-align: center; padding: 16px 0px 16px 0px; font-size: 14px;">
                        <a t-attf-href="{{ object.get_base_url() }}/shop/cart?access_token={{ object.access_token }}"
                            target="_blank"
                            style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                            Resume order
                        </a>
                    </div>
                    <t t-set="company" t-value="object.company_id or object.user_id.company_id or user.company_id"/>
                    <div style="text-align: center;"><strong>Thank you for shopping with <t t-out="company.name or ''">My Company (San Francisco)</t>!</strong></div>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: data\product_snippet_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Templates for Dynamic Snippet -->
        <template id="dynamic_filter_template_product_product_add_to_cart" name="Classic Card">
            <t t-foreach="records" t-as="data" data-thumb="/website_sale/static/src/img/snippets_options/product_add_to_cart.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card card h-100 w-100" t-att-data-add2cart-rerender="data.get('_add2cart_rerender')">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <a class="o_carousel_product_img_link o_dynamic_product_hovered overflow-hidden" t-att-href="record.website_url">
                        <img class="card-img-top o_img_product_square o_img_product_cover h-auto" loading="lazy" t-att-src="data['image_512']"
                            t-att-alt="record.display_name"/>
                    </a>
                    <i t-if="data.get('_latest_viewed')" class="fa fa-trash o_carousel_product_remove js_remove"/>
                    <div class="o_carousel_product_card_body card-body d-flex flex-wrap">
                        <a t-att-href="record.website_url" class="text-decoration-none d-block w-100">
                            <div class="h6 card-title mb-0" t-field="record.display_name"/>
                        </a>
                        <div class="mt-2">
                            <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                                <t t-set="rating_avg" t-value="record.rating_avg"/>
                                <t t-set="rating_count" t-value="record.rating_count"/>
                            </t>
                        </div>
                        <div class="w-100 d-flex flex-wrap flex-md-column flex-lg-row align-items-center align-self-end justify-content-between mt-3">
                            <div class="py-2">
                                <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                            </div>
                            <div class="o_dynamic_snippet_btn_wrapper" t-if="record._website_show_quick_add()">
                                <button type="button" role="button" class="btn btn-primary js_add_cart ms-auto" title="Add to Cart">
                                    <i class="fa fa-fw fa-shopping-cart"/>
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_view_detail" name="Classic Card - Detailed">
            <t t-foreach="records" t-as="data" data-number-of-elements="3" data-thumb="/website_sale/static/src/img/snippets_options/product_view_detail.svg">
                <t t-set="record" t-value="data['_record']" data-arrow-position="bottom"/>
                <div class="o_carousel_product_card card h-100 w-100" t-att-data-add2cart-rerender="data.get('_add2cart_rerender')">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <a class="o_carousel_product_img_link o_dynamic_product_hovered overflow-hidden" t-att-href="record.website_url">
                        <img class="card-img-top o_img_product_square o_img_product_cover h-auto" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                    </a>
                    <div class="o_carousel_product_card_body card-body d-flex flex-column justify-content-between">
                        <div class="card-title h5" t-field="record.display_name"/>
                        <div class="card-text flex-grow-1 text-muted h6" t-field="record.description_sale"/>
                        <div class="mt-2">
                            <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                                <t t-set="rating_avg" t-value="record.rating_avg"/>
                                <t t-set="rating_count" t-value="record.rating_count"/>
                            </t>
                        </div>
                        <div class="d-flex justify-content-between flex-wrap flex-md-column flex-lg-row align-items-center align-self-end w-100 mt-2 pt-3 border-top">
                            <div class="pb-2">
                                <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                            </div>
                            <a class="btn btn-primary" t-att-href="record.website_url">
                                View product
                            </a>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_mini_image" name="Image only">
            <t t-foreach="records" t-as="data" data-number-of-elements="4" data-number-of-elements-sm="1" data-thumb="/website_sale/static/src/img/snippets_options/product_image_only.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="card h-100 border-0 w-100 rounded-0 bg-transparent" t-att-data-url="record.website_url">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <a class="o_carousel_product_img_link o_dynamic_product_hovered overflow-hidden" t-att-href="record.website_url">
                        <img class="card-img-top h-auto o_img_product_square o_img_product_cover rounded" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                    </a>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_mini_price" name="Image with price">
            <t t-foreach="records" t-as="data" data-thumb="/website_sale/static/src/img/snippets_options/product_image_with_price.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="card h-100 border-0 w-100 rounded-0 bg-transparent o_dynamic_product_hovered" t-att-data-url="record.website_url">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <a class="o_carousel_product_img_link o_dynamic_product_hovered overflow-hidden" t-att-href="record.website_url">
                        <img class="card-img-top h-auto o_img_product_square o_img_product_cover rounded" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                    </a>
                    <div class="o_carousel_product_card_body mt-2 d-flex justify-content-between">
                        <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                            <t t-set="rating_style_compressed" t-value="true"/>
                            <t t-set="rating_avg" t-value="record.rating_avg"/>
                            <t t-set="rating_count" t-value="record.rating_count"/>
                        </t>
                        <div class="ms-auto">
                            <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_mini_name" name="Image with name">
            <t t-foreach="records" t-as="data" data-thumb="/website_sale/static/src/img/snippets_options/product_image_with_name.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="card h-100 border-0 w-100 rounded-0 bg-transparent o_dynamic_product_hovered" t-att-data-url="record.website_url">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <a class="o_carousel_product_img_link overflow-hidden" t-att-href="record.website_url">
                        <img class="card-img-top h-auto o_img_product_square o_img_product_cover rounded" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                    </a>
                    <div class="h6 text-center mt-2 p-2" t-field="record.display_name"/>
                    <div class="text-center">
                        <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                            <t t-set="rating_avg" t-value="record.rating_avg"/>
                            <t t-set="rating_count" t-value="record.rating_count"/>
                        </t>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_centered" name="Centered Product">
            <t t-foreach="records" t-as="data" data-arrow-position="bottom" data-thumb="/website_sale/static/src/img/snippets_options/product_centered.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card card w-100" t-att-data-add2cart-rerender="data.get('_add2cart_rerender')">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <a class="o_carousel_product_img_link position-absolute mx-auto" t-att-href="record.website_url">
                        <img class="card-img-top" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                    </a>
                    <div class="o_carousel_product_card_body card-body d-flex flex-column justify-content-between">
                        <div class="card-title h5 text-center" t-field="record.display_name"/>
                        <div class="text-center">
                            <div class="h5">
                                <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                            </div>
                            <div class="h6 mb-0">
                                <t t-if="is_view_active('website_sale.product_comment')">
                                    <t t-call="portal_rating.rating_widget_stars_static">
                                        <t t-set="rating_avg" t-value="record.rating_avg"/>
                                        <t t-set="rating_count" t-value="record.rating_count"/>
                                    </t>
                                </t>
                            </div>
                        </div>
                    </div>
                    <div class="o_carousel_product_card_footer d-flex align-items-center justify-content-center pb-4">
                        <a class="btn btn-primary d-block" t-att-href="record.website_url">
                            View Product
                        </a>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_borderless_1" name="Borderless Product n°1">
            <t t-foreach="records" t-as="data" data-thumb="/website_sale/static/src/img/snippets_options/product_borderless_1.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card bg-transparent w-100 card border-0">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <a class="o_carousel_product_img_link o_dynamic_product_hovered stretched-link" t-att-href="record.website_url">
                        <div class="overflow-hidden rounded">
                            <img class="card-img-top o_img_product_square o_img_product_cover h-auto" loading="lazy" t-att-src="data['image_512']"
                            t-att-alt="record.display_name"/>
                        </div>
                    </a>
                    <div class="o_carousel_product_card_body d-flex flex-wrap flex-column justify-content-between h-100 p-3">
                        <div class="h6 card-title" t-field="record.display_name"/>
                        <div>
                            <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                                <t t-set="rating_avg" t-value="record.rating_avg"/>
                                <t t-set="rating_count" t-value="record.rating_count"/>
                            </t>
                            <div class="mt-2">
                                <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_borderless_2" name="Borderless Product n°2">
            <t t-foreach="records" t-as="data" data-thumb="/website_sale/static/src/img/snippets_options/product_borderless_2.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card card w-100 border-0 bg-transparent" t-att-data-add2cart-rerender="data.get('_add2cart_rerender')">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <a class="o_carousel_product_img_link o_dynamic_product_hovered" t-att-href="record.website_url">
                        <div class="overflow-hidden rounded">
                            <img class="card-img-top o_img_product_square o_img_product_cover h-auto" loading="lazy" t-att-src="data['image_512']"
                            t-att-alt="record.display_name"/>
                        </div>
                    </a>
                    <div class="o_carousel_product_card_body h-100 p-3 d-flex flex-column justify-content-between">
                        <div class="d-flex justify-content-between align-items-center flex-wrap mb-2">
                            <div class="h5 mb-0 me-4">
                                <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                            </div>
                            <div class="h6 mb-0">
                                <t t-if="is_view_active('website_sale.product_comment')">
                                    <t t-call="portal_rating.rating_widget_stars_static">
                                        <t t-set="rating_style_compressed" t-value="true"/>
                                        <t t-set="rating_avg" t-value="record.rating_avg"/>
                                        <t t-set="rating_count" t-value="record.rating_count"/>
                                    </t>
                                </t>
                            </div>
                        </div>
                        <div class="card-title h6 flex-grow-1 w-100 mt-2 mb-3" t-field="record.display_name"/>
                        <div class="text-end o_dynamic_snippet_btn_wrapper" t-if="record._website_show_quick_add()">
                            <button type="button" role="button" class="btn btn-primary js_add_cart w-100" title="Add to Cart">
                                Add to Cart
                            </button>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_banner" name="Large Banner">
            <t t-foreach="records" t-as="data" data-number-of-elements="1" data-number-of-elements-sm="1" data-thumb="/website_sale/static/src/img/snippets_options/product_banner.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card card w-100" t-att-data-add2cart-rerender="data.get('_add2cart_rerender')">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <div class="row flex-row-reverse">
                        <div class="col-lg-6 d-flex align-items-center justify-content-center justify-content-lg-end o_wrap_product_img position-relative">
                            <img class="img img-fluid position-absolute o_img_product_cover w-100 h-100" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                        </div>
                        <div class="col-lg-6 px-5 d-flex align-items-center">
                            <div class="o_carousel_product_card_body card-body p-5">
                                <div class="card-title h1" t-field="record.display_name"/>
                                <div class="d-flex align-items-center my-4">
                                    <div class="h4 mb-0 me-3">
                                        <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                                    </div>
                                    <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                                        <t t-set="rating_avg" t-value="record.rating_avg"/>
                                        <t t-set="rating_count" t-value="record.rating_count"/>
                                    </t>
                                </div>
                                <div class="card-text text-muted" t-field="record.description_sale"/>
                                <div class="mt-4">
                                    <button t-if="record._website_show_quick_add()" type="button" role="button" class="btn btn-primary js_add_cart mt-1" title="Add to Cart">
                                        Add to Cart
                                    </button>
                                    <a class="btn btn-link me-1 mt-1" t-att-href="record.website_url">
                                        View Product
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_horizontal_card" name="Horizontal Card">
            <t t-foreach="records" t-as="data"
                data-number-of-elements="3"
                data-number-of-elements-sm="1"
                data-row-per-slide="2"
                data-arrow-position="bottom"
                data-extra-classes="o_carousel_multiple_rows"
                data-thumb="/website_sale/static/src/img/snippets_options/product_horizontal_card.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card card w-100 border-0 bg-light p-3" t-att-data-add2cart-rerender="data.get('_add2cart_rerender')">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <div class="row h-100 p-0">
                        <div class="col-lg-4 position-static">
                            <a class="stretched-link o_dynamic_product_hovered" t-att-href="record.website_url">
                                <img class="img img-fluid mx-auto o_img_product_square" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                            </a>
                        </div>
                        <div class="o_carousel_product_card_body col-lg-8 d-flex flex-column justify-content-between">
                            <div>
                                <div class="card-title h6" t-field="record.display_name"/>
                            </div>
                            <div>
                                <div class="mb-1">
                                    <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                                        <t t-set="rating_avg" t-value="record.rating_avg"/>
                                        <t t-set="rating_count" t-value="record.rating_count"/>
                                    </t>
                                </div>
                                <div class="d-flex align-items-center flex-wrap">
                                    <div class="my-2">
                                        <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                                    </div>
                                    <div t-if="record._website_show_quick_add()" class="o_dynamic_snippet_btn_wrapper ms-auto">
                                        <button type="button" role="button" class="btn btn-primary js_add_cart" title="Add to Cart">
                                            <i class="fa fa-fw fa-shopping-cart"/>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>
        <template id="dynamic_filter_template_product_product_horizontal_card_2" name="Horizontal Card width covered image">
            <t t-foreach="records" t-as="data"
                data-row-per-slide="2"
                data-arrow-position="bottom"
                data-number-of-elements="2"
                data-number-of-elements-sm="1"
                data-extra-classes="o_carousel_multiple_rows"
                data-thumb="/website_sale/static/src/img/snippets_options/product_horizontal_card_2.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card card w-100 border-0 o_dynamic_product_hovered o_cc o_cc5">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <a class="stretched-link" t-att-href="record.website_url">
                        <img class="img img-fluid position-absolute w-100 h-100 o_img_product_cover" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                    </a>
                    <div class="o_carousel_product_card_body d-flex flex-column justify-content-between h-100 bg-black-50 p-3 position-relative">
                        <div class="mb-3">
                            <div class="card-title h5" t-field="record.display_name"/>
                            <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                                    <t t-set="rating_avg" t-value="record.rating_avg"/>
                                    <t t-set="rating_count" t-value="record.rating_count"/>
                                </t>
                        </div>
                        <div class="card-text h6 flex-grow-1" t-field="record.description_sale"/>
                        <div class="d-flex justify-content-between align-items-center flex-wrap mt-3">
                            <div class="h5 mb-0 me-2">
                                <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                            </div>
                            <div t-if="record._website_show_quick_add()" class="o_dynamic_snippet_btn_wrapper">
                                <button type="button" role="button" class="btn btn-primary js_add_cart" title="Add to Cart">
                                    Add to Cart
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="dynamic_filter_template_product_product_card_group" name="Card group">
            <t t-foreach="records" t-as="data"
                data-row-per-slide="2"
                data-number-of-elements="2"
                data-number-of-elements-sm="1"
                data-arrow-position="bottom"
                data-extra-classes="o_card_group rounded"
                data-thumb="/website_sale/static/src/img/snippets_options/product_card_group.svg">
                <t t-set="record" t-value="data['_record']"/>
                <div class="o_carousel_product_card card w-100 rounded-0 border-top-0 border-start-0" t-att-data-url="record.website_url">
                    <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                    <input type="hidden" name="product-id" t-att-data-product-id="record.id"/>
                    <div class="o_carousel_product_card_body card-body justify-content-between h-100 p-3">
                        <div class="row h-100">
                            <div class="col-8 d-flex flex-column">
                                <div class="card-title h5" t-field="record.display_name"/>
                                <div class="card-text h6 text-muted" t-field="record.description_sale"/>
                                <div class="d-flex justify-content-between align-items-center flex-wrap w-100 mt-auto">
                                    <div class="h5 text-primary mb-0 me-2">
                                        <t t-call="website_sale.price_dynamic_filter_template_product_product"/>
                                    </div>
                                    <t t-if="is_view_active('website_sale.product_comment')" t-call="portal_rating.rating_widget_stars_static">
                                        <t t-set="rating_avg" t-value="record.rating_avg"/>
                                        <t t-set="rating_count" t-value="record.rating_count"/>
                                    </t>
                                </div>
                            </div>
                            <div class="col-4 position-static">
                                <div class="overflow-hidden position-static">
                                    <a class="stretched-link o_dynamic_product_hovered" t-att-href="record.website_url">
                                        <img class="img img-fluid o_img_product_square o_img_product_cover h-auto" loading="lazy" t-att-src="data['image_512']" t-att-alt="record.display_name"/>
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>
        <template id="price_dynamic_filter_template_product_product" name="Dynamic Product Filter Price">
            <t t-set="record_price" t-value="record._get_contextual_price_tax_selection()"/>
            <t t-if="not website.prevent_zero_price_sale or record_price">
                <span t-esc="record_price" class="fw-bold"
                      t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                <del t-if="data.get('has_discounted_price')" class="text-danger ms-1 h6" style="white-space: nowrap;"
                     t-esc="data['list_price']"
                     t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
            </t>
            <t t-else="">
                <span t-field="website.prevent_zero_price_sale_text"/>
            </t>
        </template>

        <!-- Assets -->
        <record id="website_sale.s_dynamic_snippet_products_000_scss" model="ir.asset">
            <field name="name">Dynamic snippet products 000 SCSS</field>
            <field name="bundle">web.assets_frontend</field>
            <field name="path">website_sale/static/src/snippets/s_dynamic_snippet_products/000.scss</field>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column


class AccountMove(models.Model):
    _inherit = 'account.move'

    website_id = fields.Many2one(
        'website', compute='_compute_website_id', string='Website',
        help='Website through which this invoice was created for eCommerce orders.',
        store=True, readonly=True, tracking=True)

    def _auto_init(self):
        if not column_exists(self.env.cr, "account_move", "website_id"):
            # Creating the column via `_auto_init` prevents a MemoryError in databases where many
            # invoices exist when `website_sale` is installed, as it skips the computation of the
            # `website_id` field.
            create_column(self.env.cr, "account_move", "website_id", "int4")
        super()._auto_init()

    def preview_invoice(self):
        action = super().preview_invoice()
        if action['url'].startswith('/'):
            # URL should always be relative, safety check
            action['url'] = f'/@{action["url"]}'
        return action

    @api.depends('partner_id')  # Dummy depends to trigger compute, will be dropped in master
    def _compute_website_id(self):
        for move in self:
            source_websites = move.line_ids.sale_line_ids.order_id.website_id
            if len(source_websites) == 1:
                move.website_id = source_websites
            else:
                move.website_id = False

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

    website_ids = fields.One2many('website', 'salesteam_id', string='Websites')
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
        website_teams = self.filtered(lambda team: team.website_ids)
        abandoned_carts_data = self.env['sale.order']._read_group([
            ('is_abandoned_cart', '=', True),
            ('cart_recovery_email_sent', '=', False),
            ('team_id', 'in', website_teams.ids),
        ], ['team_id'], ['amount_total:sum', '__count'])
        counts = {team.id: count for team, __, count in abandoned_carts_data}
        amounts = {team.id: amount_total_sum for team, amount_total_sum, __ in abandoned_carts_data}
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

## File: models\delivery_carrier.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class DeliveryCarrier(models.Model):
    _name = 'delivery.carrier'
    _inherit = ['delivery.carrier', 'website.published.multi.mixin']

    website_description = fields.Text(related='product_id.description_sale', string='Description for Online Quotations', readonly=False)

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

        self._calculate_company_based_kpi(
            'sale.report',
            'kpi_website_sale_total_value',
            date_field='date',
            additional_domain=[('state', 'not in', ['draft', 'cancel', 'sent']), ('website_id', '!=', False)],
            sum_field='price_subtotal',
        )

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
    def _pre_dispatch(cls, rule, args):
        super()._pre_dispatch(rule, args)
        affiliate_id = request.httprequest.args.get('affiliate_id')
        if affiliate_id:
            request.session['affiliate_id'] = int(affiliate_id)

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    def _get_available_tokens(self, *args, is_express_checkout=False, **kwargs):
        """ Override of `payment` not to return the tokens in case of express checkout.

        :param dict args: Locally unused arguments.
        :param bool is_express_checkout: Whether the payment is made through express checkout.
        :param dict kwargs: Locally unused keywords arguments.
        :return: The available tokens.
        :rtype: payment.token
        """
        if is_express_checkout:
            return self.env['payment.token']

        return super()._get_available_tokens(*args, **kwargs)

```

## File: models\product_attribute.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ProductAttribute(models.Model):
    _inherit = 'product.attribute'

    visibility = fields.Selection(
        selection=[('visible', "Visible"), ('hidden', "Hidden")],
        default='visible')

```

## File: models\product_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ProductDocument(models.Model):
    _inherit = 'product.document'

    shown_on_product_page = fields.Boolean(string="Show on product page")

    @api.constrains('res_model', 'shown_on_product_page')
    def _unsupported_product_product_document_on_ecommerce(self):
        # Not supported for now because product page is dynamic and it would require a lot of work
        # to update documents shown according to combination. It'll wait for planned tasks
        # rebuilding the product page & variant mixin.
        for document in self:
            if document.res_model == 'product.product' and document.shown_on_product_page:
                raise ValidationError(
                    _("Documents shown on product page cannot be restricted to a specific variant"))

```

## File: models\product_image.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError

from odoo.addons.web_editor.tools import get_video_embed_code, get_video_thumbnail


class ProductImage(models.Model):
    _name = 'product.image'
    _description = "Product Image"
    _inherit = ['image.mixin']
    _order = 'sequence, id'

    name = fields.Char("Name", required=True)
    sequence = fields.Integer(default=10)

    image_1920 = fields.Image()

    product_tmpl_id = fields.Many2one('product.template', "Product Template", index=True, ondelete='cascade')
    product_variant_id = fields.Many2one('product.product', "Product Variant", index=True, ondelete='cascade')
    video_url = fields.Char('Video URL',
                            help='URL of a video for showcasing your product.')
    embed_code = fields.Html(compute="_compute_embed_code", sanitize=False)

    can_image_1024_be_zoomed = fields.Boolean("Can Image 1024 be zoomed", compute='_compute_can_image_1024_be_zoomed', store=True)

    @api.depends('image_1920', 'image_1024')
    def _compute_can_image_1024_be_zoomed(self):
        for image in self:
            image.can_image_1024_be_zoomed = image.image_1920 and tools.is_image_size_above(image.image_1920, image.image_1024)

    @api.onchange('video_url')
    def _onchange_video_url(self):
        if not self.image_1920:
            thumbnail = get_video_thumbnail(self.video_url)
            self.image_1920 = thumbnail and base64.b64encode(thumbnail) or False

    @api.depends('video_url')
    def _compute_embed_code(self):
        for image in self:
            image.embed_code = get_video_embed_code(image.video_url) or False

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

## File: models\product_pricelist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, UserError
from odoo.addons.website.models import ir_http


class ProductPricelist(models.Model):
    _inherit = "product.pricelist"

    def _default_website(self):
        """ Find the first company's website, if there is one. """
        company_id = self.env.company.id

        if self._context.get('default_company_id'):
            company_id = self._context.get('default_company_id')

        domain = [('company_id', '=', company_id)]
        return self.env['website'].search(domain, limit=1)

    website_id = fields.Many2one(
        comodel_name='website',
        string="Website",
        ondelete='restrict',
        default=_default_website,
        domain="[('company_id', '=?', company_id)]",
        tracking=20,
    )
    code = fields.Char(string='E-commerce Promotional Code', groups="base.group_user")
    selectable = fields.Boolean(help="Allow the end user to choose this price list")

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('company_id') and not vals.get('website_id'):
                # l10n modules install will change the company currency, creating a
                # pricelist for that currency. Do not use user's company in that
                # case as module install are done with OdooBot (company 1)
                # YTI FIXME: The fix is not at the correct place
                # It be set when we actually create the pricelist
                self = self.with_context(default_company_id=vals['company_id'])
        pricelists = super().create(vals_list)
        if pricelists:
            self.env.registry.clear_cache()
        return pricelists

    def write(self, data):
        res = super(ProductPricelist, self).write(data)
        self and self.env.registry.clear_cache()
        return res

    def unlink(self):
        res = super(ProductPricelist, self).unlink()
        self and self.env.registry.clear_cache()
        return res

    def _get_partner_pricelist_multi_search_domain_hook(self, company_id):
        domain = super()._get_partner_pricelist_multi_search_domain_hook(company_id)
        website = ir_http.get_request_website()
        if website:
            domain += self._get_website_pricelists_domain(website)
        return domain

    def _get_partner_pricelist_multi_filter_hook(self):
        res = super()._get_partner_pricelist_multi_filter_hook()
        website = ir_http.get_request_website()
        if website:
            res = res.filtered(lambda pl: pl._is_available_on_website(website))
        return res

    def _is_available_on_website(self, website):
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
        if self.company_id and self.company_id != website.company_id:
            return False
        return self.active and self.website_id.id == website.id or (not self.website_id and (self.selectable or self.sudo().code))

    def _is_available_in_country(self, country_code):
        self.ensure_one()
        if not country_code or not self.country_group_ids:
            return True
        return country_code in self.country_group_ids.country_ids.mapped('code')

    def _get_website_pricelists_domain(self, website):
        ''' Check above `_is_available_on_website` for explanation.
        Change in this method should be reflected in `_is_available_on_website`.
        '''
        return [
            ('active', '=', True),
            ('company_id', 'in', [False, website.company_id.id]),
            '|', ('website_id', '=', website.id),
            '&', ('website_id', '=', False),
            '|', ('selectable', '=', True), ('code', '!=', False),
        ]

    @api.constrains('company_id', 'website_id')
    def _check_websites_in_company(self):
        '''Prevent misconfiguration multi-website/multi-companies.
           If the record has a company, the website should be from that company.
        '''
        for record in self.filtered(lambda pl: pl.website_id and pl.company_id):
            if record.website_id.company_id != record.company_id:
                raise ValidationError(_("""Only the company's websites are allowed.\nLeave the Company field empty or select a website from that company."""))

```

## File: models\product_product.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_join

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class Product(models.Model):
    _inherit = "product.product"
    _mail_post_access = 'read'

    website_id = fields.Many2one(related='product_tmpl_id.website_id', readonly=False)

    product_variant_image_ids = fields.One2many('product.image', 'product_variant_id', string="Extra Variant Images")

    website_url = fields.Char('Website URL', compute='_compute_product_website_url', help='The full URL to access the document through the website.')
    ribbon_id = fields.Many2one(string="Variant Ribbon", comodel_name='product.ribbon')

    base_unit_count = fields.Float('Base Unit Count', required=True, default=1, help="Display base unit price on your eCommerce pages. Set to 0 to hide it for this product.")
    base_unit_id = fields.Many2one('website.base.unit', string='Custom Unit of Measure', help="Define a custom unit to display in the price per unit of measure field.")
    base_unit_price = fields.Monetary("Price Per Unit", currency_field="currency_id", compute="_compute_base_unit_price")
    base_unit_name = fields.Char(compute='_compute_base_unit_name', help='Displays the custom unit for the products if defined or the selected unit of measure otherwise.')

    def _get_base_unit_price(self, price):
        self.ensure_one()
        return self.base_unit_count and price / self.base_unit_count

    @api.depends('lst_price', 'base_unit_count')
    def _compute_base_unit_price(self):
        for product in self:
            if not product.id:
                product.base_unit_price = 0
            else:
                product.base_unit_price = product._get_base_unit_price(product.lst_price)

    @api.depends('uom_name', 'base_unit_id')
    def _compute_base_unit_name(self):
        for product in self:
            product.base_unit_name = product.base_unit_id.name or product.uom_name

    @api.constrains('base_unit_count')
    def _check_base_unit_count(self):
        if any(product.base_unit_count < 0 for product in self):
            raise ValidationError(_('The value of Base Unit Count must be greater than 0. Use 0 to hide the price per unit on this product.'))

    @api.depends_context('lang')
    @api.depends('product_tmpl_id.website_url', 'product_template_attribute_value_ids')
    def _compute_product_website_url(self):
        for product in self:
            attributes = ','.join(str(x) for x in product.product_template_attribute_value_ids.ids)
            url = product.product_tmpl_id.website_url
            if attributes:
                url = url_join(url, f"#attr={attributes}")
            product.website_url = url

    def _prepare_variant_values(self, combination):
        variant_dict = super()._prepare_variant_values(combination)
        variant_dict['base_unit_count'] = self.base_unit_count
        return variant_dict

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

        It contains in this order: the main image of the variant (which will fall back on the main
        image of the template, if unset), the Variant Extra Images, and the Template Extra Images.
        """
        self.ensure_one()
        variant_images = list(self.product_variant_image_ids)
        template_images = list(self.product_tmpl_id.product_template_image_ids)
        return [self] + variant_images + template_images

    def _get_combination_info_variant(self, **kwargs):
        """Return the variant info based on its combination.
        See `_get_combination_info` for more information.
        """
        self.ensure_one()
        return self.product_tmpl_id._get_combination_info(
            combination=self.product_template_attribute_value_ids,
            product_id=self.id,
            **kwargs)

    def _website_show_quick_add(self):
        website = self.env['website'].get_current_website()
        return self.sale_ok and (not website.prevent_zero_price_sale or self._get_contextual_price())

    def _is_add_to_cart_allowed(self):
        self.ensure_one()
        return self.user_has_groups('base.group_system') or (self.active and self.sale_ok and self.website_published)

    def _get_contextual_price_tax_selection(self):
        self.ensure_one()
        website = self.env['website'].get_current_website()
        fiscal_position_sudo = website.sudo().fiscal_position_id
        product_taxes = self.sudo().taxes_id._filter_taxes_by_company(self.env.company)
        return self.env['product.template']._apply_taxes_to_price(
            self._get_contextual_price(),
            website.currency_id,
            product_taxes,
            fiscal_position_sudo.map_tax(product_taxes),
            self,
        )

```

## File: models\product_public_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.translate import html_translate


class ProductPublicCategory(models.Model):
    _name = "product.public.category"
    _inherit = [
        'website.seo.metadata',
        'website.multi.mixin',
        'website.searchable.mixin',
        'image.mixin',
    ]
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
    parent_path = fields.Char(index=True, unaccent=False)
    child_id = fields.One2many('product.public.category', 'parent_id', string='Children Categories')
    parents_and_self = fields.Many2many('product.public.category', compute='_compute_parents_and_self')
    sequence = fields.Integer(help="Gives the sequence order when displaying a list of product categories.", index=True, default=_default_sequence)
    website_description = fields.Html('Category Description', sanitize_overridable=True, sanitize_attributes=False, translate=html_translate, sanitize_form=False)
    product_tmpl_ids = fields.Many2many('product.template', relation='product_public_category_product_template_rel')

    @api.constrains('parent_id')
    def check_parent_id(self):
        if not self._check_recursion():
            raise ValueError(_('Error! You cannot create recursive categories.'))

    @api.depends('parents_and_self')
    def _compute_display_name(self):
        for category in self:
            category.display_name = " / ".join(category.parents_and_self.mapped(
                lambda cat: cat.name or _("New")
            ))

    @api.depends('parent_path')
    def _compute_parents_and_self(self):
        for category in self:
            if category.parent_path:
                category.parents_and_self = self.env['product.public.category'].browse([int(p) for p in category.parent_path.split('/')[:-1]])
            else:
                category.parents_and_self = category

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        search_fields = ['name']
        fetch_fields = ['id', 'name']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'url', 'type': 'text', 'truncate': False},
        }
        if with_description:
            search_fields.append('website_description')
            fetch_fields.append('website_description')
            mapping['description'] = {'name': 'website_description', 'type': 'text', 'match': True, 'html': True}
        return {
            'model': 'product.public.category',
            'base_domain': [website.website_domain()],
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-folder-o',
            'order': 'name desc, id desc' if 'name desc' in order else 'name asc, id desc',
        }

    def _search_render_results(self, fetch_fields, mapping, icon, limit):
        results_data = super()._search_render_results(fetch_fields, mapping, icon, limit)
        for data in results_data:
            data['url'] = '/shop/category/%s' % data['id']
        return results_data

```

## File: models\product_ribbon.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class ProductRibbon(models.Model):
    _name = "product.ribbon"
    _description = 'Product ribbon'

    @api.depends('html')
    def _compute_display_name(self):
        for ribbon in self:
            ribbon.display_name = f'{tools.html2plaintext(ribbon.html)} (#{ribbon.id})'

    html = fields.Html(string='Ribbon html', required=True, translate=True, sanitize=False)
    bg_color = fields.Char(string='Ribbon background color', required=False)
    text_color = fields.Char(string='Ribbon text color', required=False)
    html_class = fields.Char(string='Ribbon class', required=True, default='')

```

## File: models\product_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ProductTag(models.Model):
    _name = 'product.tag'
    _inherit = ['website.multi.mixin', 'product.tag']

    visible_on_ecommerce = fields.Boolean(
        string="Visible on eCommerce",
        help="Whether the tag is displayed on the eCommerce.",
        default=True,
    )
    image = fields.Image(string="Image", max_width=200, max_height=200)

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import api, fields, models
from odoo.addons.http_routing.models.ir_http import slug, unslug
from odoo.addons.website.models import ir_http
from odoo.tools import float_is_zero, is_html_empty
from odoo.tools.translate import html_translate
from odoo.osv import expression

_logger = logging.getLogger(__name__)

class ProductTemplate(models.Model):
    _inherit = [
        'rating.mixin',
        "product.template",
        "website.seo.metadata",
        'website.published.multi.mixin',
        'website.searchable.mixin',
    ]
    _name = 'product.template'
    _mail_post_access = 'read'
    _check_company_auto = True

    #=== DEFAULT METHODS ===#

    @api.model
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

    #=== FIELDS ===#

    website_description = fields.Html(
        string="Description for the website",
        translate=html_translate,
        sanitize_overridable=True,
        sanitize_attributes=False,
        sanitize_form=False,
    )
    description_ecommerce = fields.Html(
        string="eCommerce Description",
        translate=html_translate,
        sanitize_overridable=True,
        sanitize_attributes=False,
        sanitize_form=False,
    )

    alternative_product_ids = fields.Many2many(
        string="Alternative Products",
        comodel_name='product.template',
        relation='product_alternative_rel',
        column1='src_id', column2='dest_id',
        check_company=True,
        help="Suggest alternatives to your customer (upsell strategy). Those products show up on the product page.",
    )
    accessory_product_ids = fields.Many2many(
        string="Accessory Products",
        comodel_name='product.product',
        relation='product_accessory_rel',
        column1='src_id', column2='dest_id',
        check_company=True,
        help="Accessories show up when the customer reviews the cart before payment (cross-sell strategy).",
    )

    website_size_x = fields.Integer(string="Size X", default=1)
    website_size_y = fields.Integer(string="Size Y", default=1)
    website_ribbon_id = fields.Many2one(string="Ribbon", comodel_name='product.ribbon')
    website_sequence = fields.Integer(
        string="Website Sequence",
        default=_default_website_sequence,
        copy=False,
        index=True,
        help="Determine the display order in the Website E-commerce",
    )
    public_categ_ids = fields.Many2many(
        string="Website Product Category",
        comodel_name='product.public.category',
        relation='product_public_category_product_template_rel',
        help="The product will be available in each mentioned eCommerce category. Go to Shop > Edit "
             "Click on the page and enable 'Categories' to view all eCommerce categories.",
    )

    product_template_image_ids = fields.One2many(
        string="Extra Product Media",
        comodel_name='product.image',
        inverse_name='product_tmpl_id',
        copy=True,
    )

    base_unit_count = fields.Float(
        string="Base Unit Count",
        compute='_compute_base_unit_count',
        inverse='_set_base_unit_count',
        store=True,
        required=True,
        default=0,
        help="Display base unit price on your eCommerce pages. Set to 0 to hide it for this product.")
    base_unit_id = fields.Many2one(
        string="Custom Unit of Measure",
        comodel_name='website.base.unit',
        compute='_compute_base_unit_id',
        inverse='_set_base_unit_id',
        store=True,
        help="Define a custom unit to display in the price per unit of measure field.")
    base_unit_price = fields.Monetary(string="Price Per Unit", compute="_compute_base_unit_price")
    base_unit_name = fields.Char(
        compute='_compute_base_unit_name',
        help="Displays the custom unit for the products if defined or the selected unit of measure otherwise.")

    compare_list_price = fields.Monetary(
        string="Compare to Price",
        help="The amount will be displayed strikethroughed on the eCommerce product page")

    #=== COMPUTE METHODS ===#

    @api.depends('product_variant_ids', 'product_variant_ids.base_unit_count')
    def _compute_base_unit_count(self):
        self.base_unit_count = 0
        for template in self.filtered(lambda template: len(template.product_variant_ids) == 1):
            template.base_unit_count = template.product_variant_ids.base_unit_count

    def _set_base_unit_count(self):
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.product_variant_ids.base_unit_count = template.base_unit_count

    @api.depends('product_variant_ids', 'product_variant_ids.base_unit_count')
    def _compute_base_unit_id(self):
        self.base_unit_id = self.env['website.base.unit']
        for template in self.filtered(lambda template: len(template.product_variant_ids) == 1):
            template.base_unit_id = template.product_variant_ids.base_unit_id

    def _set_base_unit_id(self):
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.product_variant_ids.base_unit_id = template.base_unit_id

    def _get_base_unit_price(self, price):
        self.ensure_one()
        return self.base_unit_count and price / self.base_unit_count

    @api.depends('list_price', 'base_unit_count')
    def _compute_base_unit_price(self):
        for template in self:
            template.base_unit_price = template._get_base_unit_price(template.list_price)

    @api.depends('uom_name', 'base_unit_id.name')
    def _compute_base_unit_name(self):
        for template in self:
            template.base_unit_name = template.base_unit_id.name or template.uom_name

    def _compute_website_url(self):
        super()._compute_website_url()
        for product in self:
            if product.id:
                product.website_url = "/shop/%s" % slug(product)

    #=== CRUD METHODS ===#

    def write(self, vals):
        # Clear empty ecommerce description content to avoid side-effects on product pages
        # when there is no content to display anyway.
        if vals.get('description_ecommerce') and is_html_empty(vals['description_ecommerce']):
            vals['description_ecommerce'] = ''
        return super().write(vals)

    #=== BUSINESS METHODS ===#

    def _prepare_variant_values(self, combination):
        variant_dict = super()._prepare_variant_values(combination)
        variant_dict['base_unit_count'] = self.base_unit_count
        return variant_dict

    def _get_website_accessory_product(self):
        domain = self.env['website'].sale_product_domain()
        if not self.env.user._is_internal():
            domain = expression.AND([domain, [('is_published', '=', True)]])
        return self.accessory_product_ids.filtered_domain(domain)

    def _get_website_alternative_product(self):
        domain = self.env['website'].sale_product_domain()
        return self.alternative_product_ids.filtered_domain(domain)

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

    def _get_sales_prices(self, pricelist, fiscal_position):
        if not self:
            return {}

        pricelist and pricelist.ensure_one()
        pricelist = pricelist or self.env['product.pricelist']
        currency = pricelist.currency_id or self.env.company.currency_id
        date = fields.Date.context_today(self)

        sales_prices = pricelist._get_products_price(self, 1.0)
        show_discount = pricelist and pricelist.discount_policy == 'without_discount'
        show_strike_price = self.env.user.has_group('website_sale.group_product_price_comparison')

        base_sales_prices = self._price_compute('list_price', currency=currency)

        res = {}
        for template in self:
            price_reduce = sales_prices[template.id]

            product_taxes = template.sudo().taxes_id._filter_taxes_by_company(self.env.company)
            taxes = fiscal_position.map_tax(product_taxes)

            base_price = None
            price_list_contains_template = currency.compare_amounts(price_reduce, base_sales_prices[template.id]) != 0

            if template.compare_list_price and show_strike_price:
                # The base_price becomes the compare list price and the price_reduce becomes the price
                base_price = template.compare_list_price
                if not price_list_contains_template:
                    price_reduce = base_sales_prices[template.id]

                if template.currency_id != currency:
                    base_price = template.currency_id._convert(
                        base_price,
                        currency,
                        self.env.company,
                        date,
                        round=False
                    )

            elif show_discount and price_list_contains_template:
                base_price = base_sales_prices[template.id]

                # Compare_list_price are never tax included
                base_price = self._apply_taxes_to_price(
                    base_price, currency, product_taxes, taxes, template,
                )

            price_reduce = self._apply_taxes_to_price(
                price_reduce, currency, product_taxes, taxes, template,
            )

            template_price_vals = {
                'price_reduce': price_reduce,
            }
            if base_price:
                template_price_vals['base_price'] = base_price

            res[template.id] = template_price_vals

        return res

    def _can_be_added_to_cart(self):
        """
        Pre-check to `_is_add_to_cart_possible` to know if product can be sold.
        """
        return self.sale_ok

    def _is_add_to_cart_possible(self, parent_combination=None):
        """
        It's possible to add to cart (potentially after configuration) if
        there is at least one possible combination.

        :param parent_combination: the combination from which `self` is an
            optional or accessory product.
        :type parent_combination: recordset `product.template.attribute.value`

        :return: True if it's possible to add to cart, else False
        :rtype: bool
        """
        self.ensure_one()
        if not self.active or not self._can_be_added_to_cart():
            # for performance: avoid calling `_get_possible_combinations`
            return False
        return next(self._get_possible_combinations(parent_combination), False) is not False

    def _get_combination_info(
        self, combination=False, product_id=False, add_qty=1.0,
        parent_combination=False, only_template=False,
    ):
        """ Return info about a given combination.

        Note: this method does not take into account whether the combination is
        actually possible.

        :param combination: recordset of `product.template.attribute.value`

        :param int product_id: `product.product` id. If no `combination`
            is set, the method will try to load the variant `product_id` if
            it exists instead of finding a variant based on the combination.

            If there is no combination, that means we definitely want a
            variant and not something that will have no_variant set.

        :param float add_qty: the quantity for which to get the info,
            indeed some pricelist rules might depend on it.

        :param parent_combination: if no combination and no product_id are
            given, it will try to find the first possible combination, taking
            into account parent_combination (if set) for the exclusion rules.

        :param only_template: boolean, if set to True, get the info for the
            template only: ignore combination and don't try to find variant

        :return: dict with product/combination info:

            - product_id: the variant id matching the combination (if it exists)

            - product_template_id: the current template id

            - display_name: the name of the combination

            - price: the computed price of the combination, take the catalog
                price if no pricelist is given

            - price_extra: the computed extra price of the combination

            - list_price: the catalog price of the combination, but this is
                not the "real" list_price, it has price_extra included (so
                it's actually more closely related to `lst_price`), and it
                is converted to the pricelist currency (if given)

            - has_discounted_price: True if the pricelist discount policy says
                the price does not include the discount and there is actually a
                discount applied (price < list_price), else False
        """
        self.ensure_one()

        combination = combination or self.env['product.template.attribute.value']
        parent_combination = parent_combination or self.env['product.template.attribute.value']
        website = self.env['website'].get_current_website().with_context(self.env.context)

        if not product_id and not combination and not only_template:
            combination = self._get_first_possible_combination(parent_combination)

        if only_template:
            product = self.env['product.product']
        elif product_id:
            product = self.env['product.product'].browse(product_id)
            if (combination - product.product_template_attribute_value_ids):
                # If the combination is not fully represented in the given product
                #   make sure to fetch the right product for the given combination
                product = self._get_variant_for_combination(combination)
        else:
            product = self._get_variant_for_combination(combination)

        product_or_template = product or self
        combination = combination or product.product_template_attribute_value_ids

        display_name = product_or_template.display_name
        if not product:
            combination_name = combination._get_combination_name()
            if combination_name:
                display_name = f"{display_name} ({combination_name})"

        price_context = product_or_template._get_product_price_context(combination)
        product_or_template = product_or_template.with_context(**price_context)

        combination_info = {
            'product_id': product.id,
            'product_template_id': self.id,
            'display_name': display_name,
            'display_image': bool(product_or_template.image_128),
            'is_combination_possible': self._is_combination_possible(combination=combination, parent_combination=parent_combination),
            'parent_exclusions': self._get_parent_attribute_exclusions(parent_combination=parent_combination),

            **self._get_additionnal_combination_info(
                product_or_template=product_or_template,
                quantity=add_qty or 1.0,
                date=fields.Date.context_today(self),
                website=website,
            )
        }

        if website.google_analytics_key:
            combination_info['product_tracking_info'] = self._get_google_analytics_data(
                product,
                combination_info,
            )

        return combination_info

    def _get_additionnal_combination_info(self, product_or_template, quantity, date, website):
        """Computes additional combination info, based on given parameters

        :param product_or_template: `product.product` or `product.template` record
            as variant values must take precedence over template values (when we have a variant)
        :param float quantity:
        :param date date: today's date, avoids useless calls to today/context_today and harmonize
            behavior
        :param website: `website` record holding the current website of the request (if any),
            or the contextual website (tests, ...)
        :returns: additional product/template information
        :rtype: dict
        """
        pricelist = website.pricelist_id
        currency = website.currency_id

        compare_list_price = product_or_template.compare_list_price if self.env.user.has_group(
            'website_sale.group_product_price_comparison'
        ) else None
        list_price = product_or_template._price_compute('list_price')[product_or_template.id]
        price_extra = product_or_template._get_attributes_extra_price()
        if product_or_template.currency_id != currency:
            price_extra = product_or_template.currency_id._convert(
                from_amount=price_extra,
                to_currency=currency,
                company=self.env.company,
                date=date,
            )
            list_price = product_or_template.currency_id._convert(
                from_amount=list_price,
                to_currency=currency,
                company=self.env.company,
                date=date,
            )
            compare_list_price = product_or_template.currency_id._convert(
                from_amount=compare_list_price,
                to_currency=currency,
                company=self.env.company,
                date=date,
                round=False)

        # Pricelist price doesn't have to be converted
        pricelist_price = pricelist._get_product_price(
            product=product_or_template,
            quantity=quantity,
            target_currency=currency,
        )

        if pricelist.discount_policy == 'without_discount':
            has_discounted_price = currency.compare_amounts(list_price, pricelist_price) == 1
        else:
            has_discounted_price = False

        combination_info = {
            'price_extra': price_extra,
            'price': pricelist_price,
            'list_price': list_price,
            'has_discounted_price': has_discounted_price,
            'compare_list_price': compare_list_price,
        }

        # Apply taxes
        fiscal_position = website.fiscal_position_id.sudo()

        product_taxes = product_or_template.sudo().taxes_id._filter_taxes_by_company(self.env.company)
        taxes = self.env['account.tax']
        if product_taxes:
            taxes = fiscal_position.map_tax(product_taxes)
            # We do not apply taxes on the compare_list_price value because it's meant to be
            # a strict value displayed as is.
            for price_key in ('price', 'list_price', 'price_extra'):
                combination_info[price_key] = self._apply_taxes_to_price(
                    combination_info[price_key],
                    currency,
                    product_taxes,
                    taxes,
                    product_or_template,
                )

        combination_info.update({
            'prevent_zero_price_sale': website.prevent_zero_price_sale and float_is_zero(
                combination_info['price'],
                precision_rounding=currency.rounding,
            ),

            'base_unit_name': product_or_template.base_unit_name,
            'base_unit_price': product_or_template._get_base_unit_price(combination_info['price']),

            # additional info to simplify overrides
            'currency': currency,  # displayed currency
            'date': date,
            'product_taxes': product_taxes,  # taxes before fpos mapping
            'taxes': taxes,  # taxes after fpos mapping
        })

        if combination_info['prevent_zero_price_sale']:
            combination_info['compare_list_price'] = 0

        if pricelist.discount_policy != 'without_discount':
            # Leftover from before cleanup, different behavior between ecommerce & backend configurator
            # probably to keep product sales price hidden from customers ?
            combination_info['list_price'] = combination_info['price']

        if website.is_view_active('website_sale.product_tags') and product_or_template.is_product_variant:
            combination_info['product_tags'] = self.env['ir.ui.view']._render_template(
                'website_sale.product_tags', values={
                    'all_product_tags': product_or_template.all_product_tag_ids.filtered('visible_on_ecommerce')
                }
            )

        return combination_info

    @api.model
    def _apply_taxes_to_price(
        self, price, currency, product_taxes, taxes, product_or_template,
    ):
        website = self.env['website'].get_current_website()
        price = self.env['product.product']._get_tax_included_unit_price_from_price(
            price,
            currency,
            product_taxes,
            product_taxes_after_fp=taxes,
        )
        show_tax = website.show_line_subtotals_tax_selection
        tax_display = 'total_excluded' if show_tax == 'tax_excluded' else 'total_included'

        # The list_price is always the price of one.
        return taxes.compute_all(
            price, currency, 1, product_or_template, self.env.user.partner_id
        )[tax_display]

    def create_product_variant(self, product_template_attribute_value_ids):
        """ Create if necessary and possible and return the id of the product
        variant matching the given combination for this template.

        Note AWA: Known "exploit" issues with this method:

        - This method could be used by an unauthenticated user to generate a
            lot of useless variants. Unfortunately, after discussing the
            matter with ODO, there's no easy and user-friendly way to block
            that behavior.

            We would have to use captcha/server actions to clean/... that
            are all not user-friendly/overkill mechanisms.

        - This method could be used to try to guess what product variant ids
            are created in the system and what product template ids are
            configured as "dynamic", but that does not seem like a big deal.

        The error messages are identical on purpose to avoid giving too much
        information to a potential attacker:
            - returning 0 when failing
            - returning the variant id whether it already existed or not

        :param product_template_attribute_value_ids: the combination for which
            to get or create variant
        :type product_template_attribute_value_ids: list of id
            of `product.template.attribute.value`

        :return: id of the product variant matching the combination or 0
        :rtype: int
        """
        combination = self.env['product.template.attribute.value'].browse(
            product_template_attribute_value_ids)

        return self._create_product_variant(combination, log_warning=True).id or 0

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

    def _get_suitable_image_size(self, columns, x_size, y_size):
        if x_size == 1 and y_size == 1 and columns >= 3:
            return 'image_512'
        return 'image_1024'

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
            self.env.cr.execute_values(query, values_args)
        else:
            super(ProductTemplate, self)._init_column(column_name)

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
        res = super()._default_website_meta()
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = self.description_sale
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = self.env['website'].image_url(self, 'image_1024')
        res['default_meta_description'] = self.description_sale
        return res

    @api.model
    def _get_alternative_product_filter(self):
        return self.env.ref('website_sale.dynamic_filter_cross_selling_alternative_products').id

    @api.model
    def _get_product_types_allow_zero_price(self):
        """
        Returns a list of detailed types (`product.template.detailed_type`) that can ignore the
        `prevent_zero_price_sale` rule when buying products on a website.
        """
        return []

    # ---------------------------------------------------------
    # Rating Mixin API
    # ---------------------------------------------------------

    def _rating_domain(self):
        """ Only take the published rating into account to compute avg and count """
        domain = super()._rating_domain()
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

    @api.model
    def _search_get_detail(self, website, order, options):
        with_image = options['displayImage']
        with_description = options['displayDescription']
        with_category = options['displayExtraLink']
        with_price = options['displayDetail']
        domains = [website.sale_product_domain()]
        category = options.get('category')
        tags = options.get('tags')
        min_price = options.get('min_price')
        max_price = options.get('max_price')
        attrib_values = options.get('attrib_values')
        if category:
            domains.append([('public_categ_ids', 'child_of', unslug(category)[1])])
        if tags:
            if isinstance(tags, str):
                tags = tags.split(',')
            domains.append([('product_variant_ids.all_product_tag_ids', 'in', tags)])
        if min_price:
            domains.append([('list_price', '>=', min_price)])
        if max_price:
            domains.append([('list_price', '<=', max_price)])
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
        search_fields = ['name', 'default_code', 'product_variant_ids.default_code']
        fetch_fields = ['id', 'name', 'website_url']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'default_code': {'name': 'default_code', 'type': 'text', 'match': True},
            'product_variant_ids.default_code': {'name': 'product_variant_ids.default_code', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
        }
        if with_image:
            mapping['image_url'] = {'name': 'image_url', 'type': 'html'}
        if with_description:
            # Internal note is not part of the rendering.
            search_fields.append('description')
            fetch_fields.append('description')
            search_fields.append('description_sale')
            fetch_fields.append('description_sale')
            mapping['description'] = {'name': 'description_sale', 'type': 'text', 'match': True}
        if with_price:
            mapping['detail'] = {'name': 'price', 'type': 'html', 'display_currency': options['display_currency']}
            mapping['detail_strike'] = {'name': 'list_price', 'type': 'html', 'display_currency': options['display_currency']}
        if with_category:
            mapping['extra_link'] = {'name': 'category', 'type': 'html'}
        return {
            'model': 'product.template',
            'base_domain': domains,
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-shopping-cart',
        }

    def _search_render_results(self, fetch_fields, mapping, icon, limit):
        with_image = 'image_url' in mapping
        with_category = 'extra_link' in mapping
        with_price = 'detail' in mapping
        results_data = super()._search_render_results(fetch_fields, mapping, icon, limit)
        current_website = self.env['website'].get_current_website()
        for product, data in zip(self, results_data):
            categ_ids = product.public_categ_ids.filtered(lambda c: not c.website_id or c.website_id == current_website)
            if with_price:
                combination_info = product._get_combination_info(only_template=True)
                data['price'], list_price = self._search_render_results_prices(
                    mapping, combination_info
                )
                if list_price:
                    data['list_price'] = list_price

            if with_image:
                data['image_url'] = '/web/image/product.template/%s/image_128' % data['id']
            if with_category and categ_ids:
                data['category'] = self.env['ir.ui.view'].sudo()._render_template(
                    "website_sale.product_category_extra_link",
                    {'categories': categ_ids, 'slug': slug}
                )
        return results_data

    def _search_render_results_prices(self, mapping, combination_info):
        if combination_info.get('prevent_zero_price_sale'):
            website = self.env['website'].get_current_website()
            return website.prevent_zero_price_sale_text, None

        monetary_options = {'display_currency': mapping['detail']['display_currency']}
        price = self.env['ir.qweb.field.monetary'].value_to_html(
            combination_info['price'], monetary_options
        )
        list_price = None
        if combination_info['has_discounted_price']:
            list_price = self.env['ir.qweb.field.monetary'].value_to_html(
                combination_info['list_price'], monetary_options
            )
        if combination_info['compare_list_price']:
            list_price = self.env['ir.qweb.field.monetary'].value_to_html(
                combination_info['compare_list_price'], monetary_options
            )

        return price, list_price

    def _get_google_analytics_data(self, product, combination_info):
        self.ensure_one()
        return {
            'item_id': product.barcode or product.id,
            'item_name': combination_info['display_name'],
            'item_category': self.categ_id.name,
            'currency': combination_info['currency'].name,
            'price': combination_info['list_price'],
        }

    def _get_contextual_pricelist(self):
        """ Override to fallback on website current pricelist """
        pricelist = super()._get_contextual_pricelist()
        if not pricelist:
            website = ir_http.get_request_website()
            if website:
                return website.pricelist_id
        return pricelist

    def _website_show_quick_add(self):
        website = self.env['website'].get_current_website()
        return self.sale_ok and (not website.prevent_zero_price_sale or self._get_contextual_price())

```

## File: models\product_template_attribute_line.py

```python
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
        single_value_lines = self.filtered(
            lambda ptal: len(ptal.value_ids) == 1 and ptal.attribute_id.display_type != 'multi'
        )
        single_value_attributes = OrderedDict([(pa, self.env['product.template.attribute.line']) for pa in single_value_lines.attribute_id])
        for ptal in single_value_lines:
            single_value_attributes[ptal.attribute_id] |= ptal
        return single_value_attributes

```

## File: models\product_template_attribute_value.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ProductTemplateAttributeValue(models.Model):
    _inherit = 'product.template.attribute.value'

    def _get_extra_price(self, combination_info):
        self.ensure_one()
        if not self.price_extra:
            return 0.0

        price_extra = self.price_extra
        if not price_extra:
            return price_extra

        product_template = self.product_tmpl_id
        currency = combination_info['currency']
        if currency != product_template.currency_id:
            price_extra = self.currency_id._convert(
                from_amount=price_extra,
                to_currency=currency,
                company=self.env.company,
                date=combination_info['date'],
            )

        product_taxes = combination_info['product_taxes']
        if product_taxes:
            price_extra = self.env['product.template']._apply_taxes_to_price(
                price_extra,
                combination_info['currency'],
                product_taxes,
                combination_info['taxes'],
                self.product_tmpl_id,
            )

        return price_extra

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _get_default_pricelist_vals(self):
        """ Override of product. Called at company creation or activation of the pricelist setting.

        We don't want the default website from the current company to be applied on every company

        Note: self.ensure_one()

        :rtype: dict
        """
        values = super()._get_default_pricelist_vals()
        values['website_id'] = False
        return values

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    salesperson_id = fields.Many2one('res.users', related='website_id.salesperson_id', string='Salesperson', readonly=False, domain="[('share', '=', False)]")
    salesteam_id = fields.Many2one('crm.team', related='website_id.salesteam_id', string='Sales Team', readonly=False)
    group_delivery_invoice_address = fields.Boolean(string="Shipping Address", implied_group='account.group_delivery_invoice_address', group='base.group_portal,base.group_user,base.group_public')
    group_show_uom_price = fields.Boolean(default=False, string="Base Unit Price", implied_group="website_sale.group_show_uom_price", group='base.group_portal,base.group_user,base.group_public')
    group_product_price_comparison = fields.Boolean(
        string="Comparison Price",
        implied_group="website_sale.group_product_price_comparison",
        group='base.group_portal,base.group_user,base.group_public')

    module_website_sale_wishlist = fields.Boolean("Wishlists")
    module_website_sale_comparison = fields.Boolean("Product Comparison Tool")
    module_website_sale_autocomplete = fields.Boolean('Address Autocomplete')

    module_account = fields.Boolean("Invoicing")
    module_website_sale_picking = fields.Boolean('On Site Payments & Picking')

    cart_recovery_mail_template = fields.Many2one('mail.template', string='Cart Recovery Email', domain="[('model', '=', 'sale.order')]",
                                                  related='website_id.cart_recovery_mail_template_id', readonly=False)
    cart_abandoned_delay = fields.Float(string="Send After", related='website_id.cart_abandoned_delay', readonly=False)
    send_abandoned_cart_email = fields.Boolean('Abandoned Email', related='website_id.send_abandoned_cart_email', readonly=False)
    add_to_cart_action = fields.Selection(related='website_id.add_to_cart_action', readonly=False)

    module_delivery_mondialrelay = fields.Boolean("Mondial Relay Connector")
    group_product_pricelist = fields.Boolean(
        compute='_compute_group_product_pricelist', store=True, readonly=False)

    enabled_extra_checkout_step = fields.Boolean(string="Extra Step During Checkout", compute='_compute_checkout_process_steps', readonly=False, store=True)
    enabled_buy_now_button = fields.Boolean(string="Buy Now", compute='_compute_checkout_process_steps', readonly=False, store=True)

    account_on_checkout = fields.Selection(
        string="Customer Accounts",
        selection=[
            ("optional", "Optional"),
            ("disabled", "Disabled (buy as guest)"),
            ("mandatory", "Mandatory (no guest checkout)"),
        ],
        compute="_compute_account_on_checkout",
        inverse="_inverse_account_on_checkout",
        readonly=False, required=True)
    website_sale_prevent_zero_price_sale = fields.Boolean(string="Prevent Sale of Zero Priced Product", related='website_id.prevent_zero_price_sale', readonly=False)
    website_sale_contact_us_button_url = fields.Char(string="Button URL", related='website_id.contact_us_button_url', readonly=False)
    website_sale_enabled_portal_reorder_button = fields.Boolean(string="Re-order From Portal", related='website_id.enabled_portal_reorder_button', readonly=False)
    show_line_subtotals_tax_selection = fields.Selection(
        readonly=False,
        related='website_id.show_line_subtotals_tax_selection',
    )

    def set_values(self):
        super().set_values()
        if self.website_id:
            website = self.with_context(website_id=self.website_id.id).website_id
            extra_step_view = website.viewref('website_sale.extra_info')
            buy_now_view = website.viewref('website_sale.product_buy_now')

            if extra_step_view.active != self.enabled_extra_checkout_step:
                extra_step_view.active = self.enabled_extra_checkout_step
            if buy_now_view.active != self.enabled_buy_now_button:
                buy_now_view.active = self.enabled_buy_now_button

    @api.depends('group_discount_per_so_line')
    def _compute_group_product_pricelist(self):
        self.filtered(lambda w: w.group_discount_per_so_line).update({
            'group_product_pricelist': True,
        })

    @api.depends('website_id.account_on_checkout')
    def _compute_account_on_checkout(self):
        for record in self:
            record.account_on_checkout = record.website_id.account_on_checkout or 'disabled'

    @api.depends('website_id')
    def _compute_checkout_process_steps(self):
        """
        Computing the extra info step and buy now settings when changing
        the website in the res.config.settings page to show the correct value
        in the checkbox.
        """
        for record in self:
            website = record.with_context(website_id=record.website_id.id).website_id
            record.enabled_extra_checkout_step = website.is_view_active(
                'website_sale.extra_info'
            )
            record.enabled_buy_now_button = website.is_view_active(
                'website_sale.product_buy_now'
            )

    def _inverse_account_on_checkout(self):
        for record in self:
            if not record.website_id:
                continue
            record.website_id.account_on_checkout = record.account_on_checkout
            # account_on_checkout implies different values for `auth_signup_uninvited`
            if record.account_on_checkout in ['optional', 'mandatory']:
                record.website_id.auth_signup_uninvited = 'b2c'
            else:
                record.website_id.auth_signup_uninvited = 'b2b'

    def action_open_extra_info(self):
        self.ensure_one()
        # Add the "edit" parameter in the url to tell the controller
        # that we want to edit even if we are not in a payment flow
        return self.env["website"].get_client_action('/shop/extra_info?open_editor=true', True, self.website_id.id)

    def action_open_sale_mail_templates(self):
        return {
            'name': _('Customize Email Templates'),
            'type': 'ir.actions.act_window',
            'domain': [('model', '=', 'sale.order')],
            'res_model': 'mail.template',
            'view_id': False,
            'view_mode': 'tree,form',
        }

    def action_open_abandoned_cart_mail_template(self):
        return {
            'name': _('Customize Email Templates'),
            'type': 'ir.actions.act_window',
            'res_model': 'mail.template',
            'view_id': False,
            'view_mode': 'form',
            'res_id': self.env['ir.model.data']._xmlid_to_res_id("website_sale.mail_template_sale_cart_recovery"),
        }

```

## File: models\res_country.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResCountry(models.Model):
    _inherit = 'res.country'

    def get_website_sale_countries(self, mode='billing'):
        res = self.sudo().search([])
        if mode == 'shipping':
            countries = self.env['res.country']

            delivery_carriers = self.env['delivery.carrier'].sudo().search([('website_published', '=', True)])
            for carrier in delivery_carriers:
                if not carrier.country_ids and not carrier.state_ids:
                    countries = res
                    break
                countries |= carrier.country_ids

            res = res & countries
        return res

    def get_website_sale_states(self, mode='billing'):
        res = self.sudo().state_ids
        if mode == 'shipping':
            states = self.env['res.country.state']
            dom = ['|', ('country_ids', 'in', self.id), ('country_ids', '=', False), ('website_published', '=', True)]
            delivery_carriers = self.env['delivery.carrier'].sudo().search(dom)

            for carrier in delivery_carriers:
                if not carrier.country_ids or not carrier.state_ids:
                    states = res
                    break
                states |= carrier.state_ids
            res = res & states
        return res

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models

from odoo.addons.website.models import ir_http


class ResPartner(models.Model):
    _inherit = 'res.partner'

    last_website_so_id = fields.Many2one('sale.order', compute='_compute_last_website_so_id', string='Last Online Sales Order')

    def _compute_last_website_so_id(self):
        SaleOrder = self.env['sale.order']
        for partner in self:
            is_public = partner.is_public
            website = ir_http.get_request_website()
            if website and not is_public:
                partner.last_website_so_id = SaleOrder.search([
                    ('partner_id', '=', partner.id),
                    ('pricelist_id', '=', partner.property_product_pricelist.id),
                    ('website_id', '=', website.id),
                    ('state', '=', 'draft'),
                ], order='write_date desc', limit=1)
            else:
                partner.last_website_so_id = SaleOrder  # Not in a website context or public User

    @api.onchange('property_product_pricelist')
    def _onchange_property_product_pricelist(self):
        open_order = self.env['sale.order'].sudo().search([
            ('partner_id', '=', self._origin.id),
            ('pricelist_id', '=', self._origin.property_product_pricelist.id),
            ('pricelist_id', '!=', self.property_product_pricelist.id),
            ('website_id', '!=', False),
            ('state', '=', 'draft'),
        ], limit=1)

        if open_order:
            return {'warning': {
                'title': _('Open Sale Orders'),
                'message': _(
                    "This partner has an open cart. "
                    "Please note that the pricelist will not be updated on that cart. "
                    "Also, the cart might not be visible for the customer until you update the pricelist of that cart."
                ),
            }}

    def _can_be_edited_by_current_customer(self, sale_order, mode):
        self.ensure_one()
        children_partner_ids = self.env['res.partner']._search([
            ('id', 'child_of', sale_order.partner_id.commercial_partner_id.id),
            ('type', 'in', ('invoice', 'delivery', 'other')),
        ])
        if (
            self == sale_order.partner_id
            or self.id in children_partner_ids
        ):
            # address belongs to the customer
            if mode == 'billing':
                # All addresses are editable as billing
                return True
            elif mode == 'shipping' and self.type == 'delivery':
                # Only delivery addresses are editable as delivery
                return True

        return False

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import random
from datetime import datetime
from dateutil.relativedelta import relativedelta

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError
from odoo.http import request
from odoo.osv import expression
from odoo.tools import float_is_zero


class SaleOrder(models.Model):
    _inherit = "sale.order"

    website_order_line = fields.One2many(
        'sale.order.line',
        compute='_compute_website_order_line',
        string='Order Lines displayed on Website',
    ) # should not be used for computation purpose.',
    cart_quantity = fields.Integer(compute='_compute_cart_info', string='Cart Quantity')
    only_services = fields.Boolean(compute='_compute_cart_info', string='Only Services')
    is_abandoned_cart = fields.Boolean('Abandoned Cart', compute='_compute_abandoned_cart', search='_search_abandoned_cart')
    cart_recovery_email_sent = fields.Boolean('Cart recovery email already sent')
    website_id = fields.Many2one('website', string='Website', readonly=True,
                                 help='Website through which this order was placed for eCommerce orders.')
    shop_warning = fields.Char('Warning')
    amount_delivery = fields.Monetary(
        string="Delivery Amount",
        compute='_compute_amount_delivery',
        help="Tax included or excluded depending on the website configuration.",
    )
    access_point_address = fields.Json("Delivery Point Address")

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('website_id'):
                website = self.env['website'].browse(vals['website_id'])
                if 'company_id' in vals:
                    company = self.env['res.company'].browse(vals['company_id'])
                    if website.company_id.id != company.id:
                        raise ValueError(_("The company of the website you are trying to sale from (%s) is different than the one you want to use (%s)", website.company_id.name, company.name))
                else:
                    vals['company_id'] = website.company_id.id
        return super().create(vals_list)

    def _compute_user_id(self):
        """Do not assign self.env.user as salesman for e-commerce orders
        Leave salesman empty if no salesman is specified on partner or website

        c/p of the logic in Website._prepare_sale_order_values
        """
        website_orders = self.filtered('website_id')
        super(SaleOrder, self - website_orders)._compute_user_id()
        for order in website_orders:
            if not order.user_id:
                order.user_id = order.website_id.salesperson_id or order.partner_id.parent_id.user_id.id or order.partner_id.user_id.id

    @api.model
    def _get_note_url(self):
        website_id = self._context.get('website_id')
        if website_id:
            return self.env['website'].browse(website_id).get_base_url()
        return super()._get_note_url()

    @api.depends('order_line')
    def _compute_website_order_line(self):
        for order in self:
            order.website_order_line = order.order_line.filtered(lambda l: l._show_in_cart())

    @api.depends('order_line.product_uom_qty', 'order_line.product_id')
    def _compute_cart_info(self):
        for order in self:
            order.cart_quantity = int(sum(order.mapped('website_order_line.product_uom_qty')))
            order.only_services = all(l.product_id.type == 'service' for l in order.website_order_line)

    @api.depends('order_line.price_total', 'order_line.price_subtotal')
    def _compute_amount_delivery(self):
        self.amount_delivery = 0.0
        for order in self.filtered('website_id'):
            delivery_lines = order.order_line.filtered('is_delivery')
            if order.website_id.show_line_subtotals_tax_selection == 'tax_excluded':
                order.amount_delivery = sum(delivery_lines.mapped('price_subtotal'))
            else:
                order.amount_delivery = sum(delivery_lines.mapped('price_total'))

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

    @api.depends('partner_id')
    def _compute_payment_term_id(self):
        super()._compute_payment_term_id()
        for order in self:
            if order.website_id:
                order.payment_term_id = order.website_id.with_company(order.company_id).sale_get_payment_term(order.partner_id)

    def _search_abandoned_cart(self, operator, value):
        website_ids = self.env['website'].search_read(fields=['id', 'cart_abandoned_delay', 'partner_id'])
        deadlines = [[
            '&', '&',
            ('website_id', '=', website_id['id']),
            ('date_order', '<=', fields.Datetime.to_string(datetime.utcnow() - relativedelta(hours=website_id['cart_abandoned_delay'] or 1.0))),
            ('partner_id', '!=', website_id['partner_id'][0])
        ] for website_id in website_ids]
        abandoned_domain = [
            ('state', '=', 'draft'),
            ('order_line', '!=', False)
        ]
        abandoned_domain.extend(expression.OR(deadlines))
        abandoned_domain = expression.normalize_domain(abandoned_domain)
        # is_abandoned domain possibilities
        if (operator not in expression.NEGATIVE_TERM_OPERATORS and value) or (operator in expression.NEGATIVE_TERM_OPERATORS and not value):
            return abandoned_domain
        return expression.distribute_not(['!'] + abandoned_domain)  # negative domain

    def _cart_update_order_line(self, product_id, quantity, order_line, **kwargs):
        self.ensure_one()

        if order_line and quantity <= 0:
            # Remove zero or negative lines
            order_line.unlink()
            order_line = self.env['sale.order.line']
        elif order_line:
            # Update existing line
            update_values = self._prepare_order_line_update_values(order_line, quantity, **kwargs)
            if update_values:
                self._update_cart_line_values(order_line, update_values)
        elif quantity > 0:
            # Create new line
            order_line_values = self._prepare_order_line_values(product_id, quantity, **kwargs)
            order_line = self.env['sale.order.line'].sudo().create(order_line_values)
        return order_line

    def _cart_update_pricelist(self, pricelist_id=None, update_pricelist=False):
        self.ensure_one()

        previous_pricelist_id = self.pricelist_id.id

        if pricelist_id:
            self.pricelist_id = pricelist_id

        if update_pricelist:
            self._compute_pricelist_id()

        if update_pricelist or previous_pricelist_id != self.pricelist_id.id:
            self._recompute_prices()

    def _cart_update(self, product_id, line_id=None, add_qty=0, set_qty=0, **kwargs):
        """ Add or set product quantity, add_qty can be negative """
        self.ensure_one()
        self = self.with_company(self.company_id)

        if self.state != 'draft':
            request.session.pop('sale_order_id', None)
            request.session.pop('website_sale_cart_quantity', None)
            raise UserError(_('It is forbidden to modify a sales order which is not in draft status.'))

        product = self.env['product.product'].browse(product_id).exists()
        if add_qty and (not product or not product._is_add_to_cart_allowed()):
            raise UserError(_("The given product does not exist therefore it cannot be added to cart."))

        if line_id is not False:
            order_line = self._cart_find_product_line(product_id, line_id, **kwargs)[:1]
        else:
            order_line = self.env['sale.order.line']

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
        if set_qty:
            quantity = set_qty
        elif add_qty is not None:
            if order_line:
                quantity = order_line.product_uom_qty + (add_qty or 0)
            else:
                quantity = add_qty or 0

        if quantity > 0:
            quantity, warning = self._verify_updated_quantity(
                order_line,
                product_id,
                quantity,
                **kwargs,
            )
        else:
            # If the line will be removed anyway, there is no need to verify
            # the requested quantity update.
            warning = ''

        self._remove_delivery_line()

        order_line = self._cart_update_order_line(product_id, quantity, order_line, **kwargs)

        if (
            order_line
            and order_line.price_unit == 0
            and self.website_id.prevent_zero_price_sale
            and product.detailed_type not in self.env['product.template']._get_product_types_allow_zero_price()
        ):
            raise UserError(_(
                "The given product does not have a price therefore it cannot be added to cart.",
            ))

        return {
            'line_id': order_line.id,
            'quantity': quantity,
            'option_ids': list(set(order_line.option_line_ids.filtered(lambda l: l.order_id == order_line.order_id).ids)),
            'warning': warning,
        }

    def _cart_find_product_line(self, product_id, line_id=None, **kwargs):
        """Find the cart line matching the given parameters.

        If a product_id is given, the line will match the product only if the
        line also has the same special attributes: `no_variant` attributes and
        `is_custom` values.
        """
        self.ensure_one()
        SaleOrderLine = self.env['sale.order.line']

        if not self.order_line:
            return SaleOrderLine

        product = self.env['product.product'].browse(product_id)
        if not line_id and (
            product.product_tmpl_id.has_dynamic_attributes()
            or product.product_tmpl_id._has_no_variant_attributes()
        ):
            return SaleOrderLine

        domain = [('order_id', '=', self.id), ('product_id', '=', product_id)]
        if line_id:
            domain += [('id', '=', line_id)]
        else:
            domain += [('product_custom_attribute_value_ids', '=', False)]

        return SaleOrderLine.search(domain)

    # hook to be overridden
    def _verify_updated_quantity(self, order_line, product_id, new_qty, **kwargs):
        return new_qty, ''

    def _prepare_order_line_values(
        self, product_id, quantity, linked_line_id=False,
        no_variant_attribute_values=None, product_custom_attribute_values=None,
        **kwargs
    ):
        self.ensure_one()
        product = self.env['product.product'].browse(product_id)

        no_variant_attribute_values = no_variant_attribute_values or []
        received_no_variant_values = product.env['product.template.attribute.value'].browse([
            int(ptav['value'])
            for ptav in no_variant_attribute_values
        ])
        received_combination = product.product_template_attribute_value_ids | received_no_variant_values
        product_template = product.product_tmpl_id

        # handle all cases where incorrect or incomplete data are received
        combination = product_template._get_closest_possible_combination(received_combination)

        # get or create (if dynamic) the correct variant
        product = product_template._create_product_variant(combination)

        if not product:
            raise UserError(_("The given combination does not exist therefore it cannot be added to cart."))

        values = {
            'product_id': product.id,
            'product_uom_qty': quantity,
            'order_id': self.id,
            'linked_line_id': linked_line_id,
        }

        # add no_variant attributes that were not received
        for ptav in combination.filtered(
            lambda ptav: ptav.attribute_id.create_variant == 'no_variant' and ptav not in received_no_variant_values
        ):
            no_variant_attribute_values.append({
                'value': ptav.id,
            })

        if no_variant_attribute_values:
            values['product_no_variant_attribute_value_ids'] = [
                fields.Command.set([int(attribute['value']) for attribute in no_variant_attribute_values])
            ]

        # add is_custom attribute values that were not received
        custom_values = product_custom_attribute_values or []
        received_custom_values = product.env['product.template.attribute.value'].browse([
            int(ptav['custom_product_template_attribute_value_id'])
            for ptav in custom_values
        ])

        for ptav in combination.filtered(lambda ptav: ptav.is_custom and ptav not in received_custom_values):
            custom_values.append({
                'custom_product_template_attribute_value_id': ptav.id,
                'custom_value': '',
            })

        if custom_values:
            values['product_custom_attribute_value_ids'] = [
                fields.Command.create({
                    'custom_product_template_attribute_value_id': custom_value['custom_product_template_attribute_value_id'],
                    'custom_value': custom_value['custom_value'],
                }) for custom_value in custom_values
            ]

        return values

    def _prepare_order_line_update_values(
        self, order_line, quantity, linked_line_id=False, **kwargs
    ):
        self.ensure_one()
        values = {}

        if quantity != order_line.product_uom_qty:
            values['product_uom_qty'] = quantity
        if linked_line_id and linked_line_id != order_line.linked_line_id.id:
            values['linked_line_id'] = linked_line_id

        return values

    # hook to be overridden
    def _update_cart_line_values(self, order_line, update_values):
        self.ensure_one()
        order_line.write(update_values)

    def _is_cart_ready(self):
        """Whether the cart is valid and can be confirmed (and paid for)

        :rtype: bool
        """
        return True

    def _check_cart_is_ready_to_be_paid(self):
        """"Whether the cart is valid and the user can proceed to the payment

        :rtype: bool
        """
        if not self._is_cart_ready():
            raise ValidationError(_(
                "Your cart is not ready to be paid, please verify previous steps."
            ))

        if not self.only_services and not self.carrier_id:
            raise ValidationError(_("No shipping method is selected."))

    def _cart_accessories(self):
        """ Suggest accessories based on 'Accessory Products' of products in cart """
        product_ids = set(self.website_order_line.product_id.ids)
        all_accessory_products = self.env['product.product']
        for line in self.website_order_line.filtered('product_id'):
            accessory_products = line.product_id.product_tmpl_id._get_website_accessory_product()
            if accessory_products:
                # Do not read ptavs if there is no accessory products to filter
                combination = line.product_id.product_template_attribute_value_ids + line.product_no_variant_attribute_value_ids
                all_accessory_products |= accessory_products.filtered(lambda product:
                    product.id not in product_ids
                    and product.filtered_domain(self.env['product.product']._check_company_domain(line.company_id))
                    and product._is_variant_possible(parent_combination=combination)
                    and (
                        not self.website_id.prevent_zero_price_sale
                        or product._get_contextual_price()
                    )
                )

        return random.sample(all_accessory_products, len(all_accessory_products))

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
                'default_email_layout_xmlid': 'mail.mail_notification_layout_with_responsible_signature',
                'default_res_ids': self.ids,
                'default_model': 'sale.order',
                'default_template_id': template_id,
                'website_sale_send_recovery_email': True,
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
        Similar method to action_recovery_email_send, made to be called in automation rules.
        Contrary to the former, it will use the website-specific template for each order."""
        sent_orders = self.env['sale.order']
        for order in self:
            template = order._get_cart_recovery_template()
            if template:
                order._portal_ensure_token()
                template.send_mail(order.id)
                sent_orders |= order
        sent_orders.write({'cart_recovery_email_sent': True})

    def _message_mail_after_hook(self, mails):
        """ After sending recovery cart emails, update orders to avoid sending
        it again. """
        if self.env.context.get('website_sale_send_recovery_email'):
            self.filtered_domain([
                ('cart_recovery_email_sent', '=', False),
                ('is_abandoned_cart', '=', True)
            ]).cart_recovery_email_sent = True
        return super()._message_mail_after_hook(mails)

    def _message_post_after_hook(self, message, msg_vals):
        """ After sending recovery cart emails, update orders to avoid sending
        it again. """
        if self.env.context.get('website_sale_send_recovery_email'):
            self.cart_recovery_email_sent = True
        return super(SaleOrder, self)._message_post_after_hook(message, msg_vals)

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ In case of cart recovery email, update link to redirect directly
        to the cart (like ``mail_template_sale_cart_recovery`` template). """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        self.ensure_one()
        customer_portal_group = next((group for group in groups if group[0] == 'portal_customer'), None)
        if customer_portal_group:
            access_opt = customer_portal_group[2].setdefault('button_access', {})
            if self._context.get('website_sale_send_recovery_email'):
                access_opt['title'] = _('Resume Order')
                access_opt['url'] = '%s/shop/cart?access_token=%s' % (self.get_base_url(), self.access_token)
        return groups

    def _action_confirm(self):
        for order in self:
            order_location = order.access_point_address

            if not order_location:
                continue

            # retrieve all the data :
            # name, street, city, state, zip, country
            name = order.partner_shipping_id.name
            street = order_location['pick_up_point_address']
            city = order_location['pick_up_point_town']
            zip_code = order_location['pick_up_point_postal_code']
            country = order.env['res.country'].search([('code', '=', order_location['pick_up_point_country'])]).id
            state = order.env['res.country.state'].search(['&', ('code', '=', order_location['pick_up_point_state']), ('country_id', '=', country)]).id if (order_location['pick_up_point_state'] and country) else None
            parent_id = order.partner_shipping_id.id
            email = order.partner_shipping_id.email
            phone = order.partner_shipping_id.phone

            # we can check if the current partner has a partner of type "delivery" that has the same address
            existing_partner = order.env['res.partner'].search(['&', '&', '&', '&', '&',
                                                                ('street', '=', street),
                                                                ('city', '=', city),
                                                                ('state_id', '=', state),
                                                                ('country_id', '=', country),
                                                                ('parent_id', '=', parent_id),
                                                                ('type', '=', 'delivery')], limit=1)

            if existing_partner:
                order.partner_shipping_id = existing_partner
            else:
                # if not, we create that res.partner
                order.partner_shipping_id = order.env['res.partner'].create({
                    'parent_id': parent_id,
                    'type': 'delivery',
                    'name': name,
                    'street': street,
                    'city': city,
                    'state_id': state,
                    'zip': zip_code,
                    'country_id': country,
                    'email': email,
                    'phone': phone
                })
        return super()._action_confirm()

    def _get_shop_warning(self, clear=True):
        self.ensure_one()
        warn = self.shop_warning
        if clear:
            self.shop_warning = ''
        return warn

    def _is_reorder_allowed(self):
        self.ensure_one()
        return self.state == 'sale' and any(line._is_reorder_allowed() for line in self.order_line if not line.display_type)

    def _filter_can_send_abandoned_cart_mail(self):
        self.website_id.ensure_one()
        abandoned_datetime = datetime.utcnow() - relativedelta(hours=self.website_id.cart_abandoned_delay)

        sales_after_abandoned_date = self.env['sale.order'].search([
            ('state', '=', 'sale'),
            ('partner_id', 'in', self.partner_id.ids),
            ('create_date', '>=', abandoned_datetime),
            ('website_id', '=', self.website_id.id),
        ])
        latest_create_date_per_partner = dict()
        for sale in self:
            if sale.partner_id not in latest_create_date_per_partner:
                latest_create_date_per_partner[sale.partner_id] = sale.create_date
            else:
                latest_create_date_per_partner[sale.partner_id] = max(latest_create_date_per_partner[sale.partner_id], sale.create_date)
        has_later_sale_order = dict()
        for sale in sales_after_abandoned_date:
            if has_later_sale_order.get(sale.partner_id, False):
                continue
            has_later_sale_order[sale.partner_id] = latest_create_date_per_partner[sale.partner_id] <= sale.date_order

        # Customer needs to be signed in otherwise the mail address is not known.
        # We therefore consider only sales with a known mail address.

        # If a payment processing error occurred when the customer tried to complete their checkout,
        # then the email won't be sent.

        # If all the products in the checkout are free, and the customer does not visit the shipping page to add a
        # shipping fee or the shipping fee is also free, then the email won't be sent.

        # If a potential customer creates one or more abandoned sale order and then completes a sale order before
        # the recovery email gets sent, then the email won't be sent.

        return self.filtered(
            lambda abandoned_sale_order:
            abandoned_sale_order.partner_id.email
            and not any(transaction.sudo().state == 'error' for transaction in abandoned_sale_order.transaction_ids)
            and any(not float_is_zero(line.price_unit, precision_rounding=line.currency_id.rounding) for line in abandoned_sale_order.order_line)
            and not has_later_sale_order.get(abandoned_sale_order.partner_id, False)
        )

    def action_preview_sale_order(self):
        action = super().action_preview_sale_order()
        if action['url'].startswith('/'):
            # URL should always be relative, safety check
            action['url'] = f'/@{action["url"]}'
        return action

    def _check_carrier_quotation(self, force_carrier_id=None, keep_carrier=False):
        self.ensure_one()
        DeliveryCarrier = self.env['delivery.carrier']

        if self.only_services:
            self._remove_delivery_line()
            return True
        else:
            self = self.with_company(self.company_id)
            # attempt to use partner's preferred carrier
            if not force_carrier_id and self.partner_shipping_id.property_delivery_carrier_id and not keep_carrier:
                force_carrier_id = self.partner_shipping_id.property_delivery_carrier_id.id

            carrier = force_carrier_id and DeliveryCarrier.browse(force_carrier_id) or self.carrier_id
            available_carriers = self._get_delivery_methods()
            if carrier:
                if carrier not in available_carriers:
                    carrier = DeliveryCarrier
                else:
                    # set the forced carrier at the beginning of the list to be verfied first below
                    available_carriers -= carrier
                    available_carriers = carrier + available_carriers
            if force_carrier_id or not carrier or carrier not in available_carriers:
                for delivery in available_carriers:
                    verified_carrier = delivery._match_address(self.partner_shipping_id)
                    if verified_carrier:
                        carrier = delivery
                        break
                self.write({'carrier_id': carrier.id})
            self._remove_delivery_line()
            if carrier:
                res = carrier.rate_shipment(self)
                if res.get('success'):
                    self.set_delivery_line(carrier, res['price'])
                    self.delivery_rating_success = True
                    self.delivery_message = res['warning_message']
                else:
                    self.set_delivery_line(carrier, 0.0)
                    self.delivery_rating_success = False
                    self.delivery_message = res['error_message']

        return bool(carrier)

    def _get_delivery_methods(self):
        # searching on website_published will also search for available website (_search method on computed field)
        return self.env['delivery.carrier'].sudo().search([
            ('website_published', '=', True),
        ]).filtered(lambda carrier: carrier._is_available_for_order(self))

    #=== TOOLING ===#

    def _is_public_order(self):
        self.ensure_one()
        return self.partner_id.id == request.website.user_id.sudo().partner_id.id

    def _get_lang(self):
        res = super()._get_lang()

        if self.website_id and request and request.is_frontend:
            # Use request lang as cart lang if request comes from frontend
            return request.env.lang

        return res

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    linked_line_id = fields.Many2one('sale.order.line', string='Linked Order Line', domain="[('order_id', '=', order_id)]", ondelete='cascade', copy=False, index=True)
    option_line_ids = fields.One2many('sale.order.line', 'linked_line_id', string='Options Linked')

    name_short = fields.Char(compute="_compute_name_short")

    shop_warning = fields.Char('Warning')

    #=== COMPUTE METHODS ===#

    @api.depends('linked_line_id', 'option_line_ids')
    def _compute_name(self):
        """Override to add the compute dependency.

        The custom name logic can be found below in _get_sale_order_line_multiline_description_sale.
        """
        super()._compute_name()

    @api.depends('product_id.display_name')
    def _compute_name_short(self):
        """ Compute a short name for this sale order line, to be used on the website where we don't have much space.
            To keep it short, instead of using the first line of the description, we take the product name without the internal reference.
        """
        for record in self:
            record.name_short = record.product_id.with_context(display_default_code=False).display_name

    #=== BUSINESS METHODS ===#

    def _get_sale_order_line_multiline_description_sale(self):
        description = super()._get_sale_order_line_multiline_description_sale()
        if self.linked_line_id:
            description += "\n" + _("Option for: %s", self.linked_line_id.product_id.display_name)
        if self.option_line_ids:
            description += "\n" + '\n'.join([
                _("Option: %s", option_line.product_id.display_name)
                for option_line in self.option_line_ids
            ])
        return description

    def get_description_following_lines(self):
        return self.name.splitlines()[1:]

    def _get_order_date(self):
        self.ensure_one()
        if self.order_id.website_id and self.state == 'draft':
            # cart prices must always be computed based on the current time, not on the order
            # creation date.
            return fields.Datetime.now()
        return super()._get_order_date()

    def _get_pricelist_price_before_discount(self):
        """On ecommerce orders, the base price must always be the sales price."""
        self.ensure_one()
        self.product_id.ensure_one()

        if self.order_id.website_id:
            return self.env['product.pricelist.item']._compute_price_before_discount(
                product=self.product_id.with_context(**self._get_product_price_context()),
                quantity=self.product_uom_qty or 1.0,
                uom=self.product_uom,
                date=self._get_order_date(),
                currency=self.currency_id,
            )

        return super()._get_pricelist_price_before_discount()

    def _get_shop_warning(self, clear=True):
        self.ensure_one()
        warn = self.shop_warning
        if clear:
            self.shop_warning = ''
        return warn

    def _get_displayed_unit_price(self):
        show_tax = self.order_id.website_id.show_line_subtotals_tax_selection
        tax_display = 'total_excluded' if show_tax == 'tax_excluded' else 'total_included'

        return self.tax_id.compute_all(
            self.price_unit, self.currency_id, 1, self.product_id, self.order_partner_id,
        )[tax_display]

    def _get_displayed_quantity(self):
        rounded_uom_qty = round(self.product_uom_qty,
                                self.env['decimal.precision'].precision_get('Product Unit of Measure'))
        return int(rounded_uom_qty) == rounded_uom_qty and int(rounded_uom_qty) or rounded_uom_qty

    def _show_in_cart(self):
        self.ensure_one()
        # Exclude delivery & section/note lines from showing up in the cart
        return not self.is_delivery and not bool(self.display_type)

    def _is_reorder_allowed(self):
        self.ensure_one()
        return self.product_id._is_add_to_cart_allowed()

```

## File: models\website.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import SUPERUSER_ID, _, _lt, api, fields, models, tools
from odoo.http import request
from odoo.osv import expression

from odoo.addons.http_routing.models.ir_http import url_for


class Website(models.Model):
    _inherit = 'website'

    def _default_salesteam_id(self):
        team = self.env.ref('sales_team.salesteam_website_sales', False)
        if team and team.active:
            return team.id
        else:
            return None

    salesperson_id = fields.Many2one('res.users', string='Salesperson')
    salesteam_id = fields.Many2one('crm.team',
        string='Sales Team', ondelete="set null",
        default=_default_salesteam_id)
    show_line_subtotals_tax_selection = fields.Selection(
        selection=[
            ('tax_excluded', "Tax Excluded"),
            ('tax_included', "Tax Included"),
        ],
        string="Line Subtotals Tax Display",
        required=True, default='tax_excluded',
    )

    fiscal_position_id = fields.Many2one(
        'account.fiscal.position', compute='_compute_fiscal_position_id')
    pricelist_id = fields.Many2one(
        'product.pricelist', compute='_compute_pricelist_id', string="Default Pricelist if any")
    currency_id = fields.Many2one(
        'res.currency', compute='_compute_currency_id', string="Default Currency")
    pricelist_ids = fields.One2many('product.pricelist', compute="_compute_pricelist_ids",
                                    string='Price list available for this Ecommerce/Website')
    # Technical: Used to recompute pricelist_ids
    all_pricelist_ids = fields.One2many('product.pricelist', 'website_id', string='All pricelists')

    def _default_recovery_mail_template(self):
        try:
            return self.env.ref('website_sale.mail_template_sale_cart_recovery').id
        except ValueError:
            return False

    cart_recovery_mail_template_id = fields.Many2one('mail.template', string='Cart Recovery Email', default=_default_recovery_mail_template, domain="[('model', '=', 'sale.order')]")
    cart_abandoned_delay = fields.Float(string="Abandoned Delay", default=10.0)
    send_abandoned_cart_email = fields.Boolean(string="Send email to customers who abandoned their cart.")

    shop_ppg = fields.Integer(default=20, string="Number of products in the grid on the shop")
    shop_ppr = fields.Integer(default=4, string="Number of grid columns on the shop")

    @staticmethod
    def _get_product_sort_mapping():
        return [
            ('website_sequence asc', _('Featured')),
            ('create_date desc', _('Newest Arrivals')),
            ('name asc', _('Name (A-Z)')),
            ('list_price asc', _('Price - Low to High')),
            ('list_price desc', _('Price - High to Low')),
        ]
    shop_default_sort = fields.Selection(selection='_get_product_sort_mapping', default='website_sequence asc', required=True)

    shop_extra_field_ids = fields.One2many('website.sale.extra.field', 'website_id', string='E-Commerce Extra Fields')

    add_to_cart_action = fields.Selection(
        selection=[
            ('stay', 'Stay on Product Page'),
            ('go_to_cart', 'Go to cart'),
        ],
        default='stay')
    auth_signup_uninvited = fields.Selection(default='b2c')
    account_on_checkout = fields.Selection(
        string="Customer Accounts",
        selection=[
            ('optional', 'Optional'),
            ('disabled', 'Disabled (buy as guest)'),
            ('mandatory', 'Mandatory (no guest checkout)'),
        ],
        default='optional')

    product_page_image_layout = fields.Selection([
        ('carousel', 'Carousel'),
        ('grid', 'Grid'),
        ], default='carousel', required=True,
    )
    product_page_grid_columns = fields.Integer(default=2)
    product_page_image_width = fields.Selection([
        ('none', 'Hidden'),
        ('50_pc', '50 %'),
        ('66_pc', '66 %'),
        ('100_pc', '100 %'),
        ], default='50_pc', required=True,
    )
    product_page_image_spacing = fields.Selection([
        ('none', 'None'),
        ('small', 'Small'),
        ('medium', 'Medium'),
        ('big', 'Big'),
        ], default='small', required=True,
    )

    prevent_zero_price_sale = fields.Boolean(string="Hide 'Add To Cart' when price = 0")
    prevent_zero_price_sale_text = fields.Char(string="Text to show instead of price", translate=True,
                                               default="Not Available For Sale")
    contact_us_button_url = fields.Char(string="Contact Us Button URL", translate=True, default="/contactus")
    enabled_portal_reorder_button = fields.Boolean(string="Re-order From Portal")
    enabled_delivery = fields.Boolean(string="Enable Shipping", compute='_compute_enabled_delivery')

    @api.depends('all_pricelist_ids')
    def _compute_pricelist_ids(self):
        for website in self:
            website = website.with_company(website.company_id)
            ProductPricelist = website.env['product.pricelist']  # with correct company in env
            website.pricelist_ids = ProductPricelist.sudo().search(
                ProductPricelist._get_website_pricelists_domain(website)
            )

    def _compute_pricelist_id(self):
        for website in self:
            website.pricelist_id = website._get_current_pricelist()

    def _compute_fiscal_position_id(self):
        for website in self:
            website.fiscal_position_id = website._get_current_fiscal_position()

    @api.depends('all_pricelist_ids', 'pricelist_id', 'company_id')
    def _compute_currency_id(self):
        for website in self:
            website.currency_id = website.pricelist_id.currency_id or website.company_id.currency_id

    def _compute_enabled_delivery(self):
        for website in self:
            website.enabled_delivery = bool(website.env['delivery.carrier'].sudo().search_count(
                [('website_id', 'in', (False, website.id)), ('is_published', '=', True)], limit=1
            ))

    # This method is cached, must not return records! See also #8795
    @tools.ormcache(
        'country_code', 'show_visible',
        'current_pl_id', 'website_pricelist_ids',
        'partner_pl_id', 'order_pl_id',
    )
    def _get_pl_partner_order(
        self, country_code, show_visible, current_pl_id, website_pricelist_ids,
        partner_pl_id=False, order_pl_id=False
    ):
        """ Return the list of pricelists that can be used on website for the current user.

        :param str country_code: code iso or False, If set, we search only price list available for this country
        :param bool show_visible: if True, we don't display pricelist where selectable is False (Eg: Code promo)
        :param int current_pl_id: The current pricelist used on the website
            (If not selectable but currently used anyway, e.g. pricelist with promo code)
        :param tuple website_pricelist_ids: List of ids of pricelists available for this website
        :param int partner_pl_id: the partner pricelist
        :param int order_pl_id: the current cart pricelist
        :returns: list of product.pricelist ids
        :rtype: list
        """
        self.ensure_one()
        pricelists = self.env['product.pricelist']

        if show_visible:
            # Only show selectable or currently used pricelist (cart or session)
            check_pricelist = lambda pl: pl.selectable or pl.id in (current_pl_id, order_pl_id)
        else:
            check_pricelist = lambda _pl: True

        # Note: 1. pricelists from all_pl are already website compliant (went through
        #          `_get_website_pricelists_domain`)
        #       2. do not read `property_product_pricelist` here as `_get_pl_partner_order`
        #          is cached and the result of this method will be impacted by that field value.
        #          Pass it through `partner_pl_id` parameter instead to invalidate the cache.

        # If there is a GeoIP country, find a pricelist for it
        if country_code:
            pricelists |= self.env['res.country.group'].search(
                [('country_ids.code', '=', country_code)]
            ).pricelist_ids.filtered(
                lambda pl: pl._is_available_on_website(self) and check_pricelist(pl)
            )

        # no GeoIP or no pricelist for this country
        if not pricelists:
            pricelists = pricelists.browse(website_pricelist_ids).filtered(check_pricelist)

        # if logged in, add partner pl (which is `property_product_pricelist`, might not be website compliant)
        if not self.env.user._is_public():
            # keep partner_pricelist only if website compliant
            partner_pricelist = pricelists.browse(partner_pl_id).filtered(
                lambda pl:
                    pl._is_available_on_website(self)
                    and check_pricelist(pl)
                    and pl._is_available_in_country(country_code)
            )
            pricelists |= partner_pricelist

        # This method is cached, must not return records! See also #8795
        # sudo is needed to ensure no records rules are applied during the sorted call,
        # we only want to reorder the records on hand, not filter them.
        return pricelists.sudo().sorted().ids

    def get_pricelist_available(self, show_visible=False):
        """ Return the list of pricelists that can be used on website for the current user.
        Country restrictions will be detected with GeoIP (if installed).
        :param bool show_visible: if True, we don't display pricelist where selectable is False (Eg: Code promo)
        :returns: pricelist recordset
        """
        self.ensure_one()

        country_code = self._get_geoip_country_code()
        website = self.with_company(self.company_id)

        partner_sudo = website.env.user.partner_id
        is_user_public = self.env.user._is_public()
        if not is_user_public:
            last_order_pricelist = partner_sudo.last_website_so_id.pricelist_id
            ctx = {'country_code': country_code} if country_code else {}
            partner_pricelist = partner_sudo.with_context(**ctx).property_product_pricelist
        else:  # public user: do not compute partner pl (not used)
            last_order_pricelist = self.env['product.pricelist']
            partner_pricelist = self.env['product.pricelist']
        website_pricelists = website.sudo().pricelist_ids

        current_pricelist_id = self._get_cached_pricelist_id()

        pricelist_ids = website._get_pl_partner_order(
            country_code,
            show_visible,
            current_pl_id=current_pricelist_id,
            website_pricelist_ids=tuple(website_pricelists.ids),
            partner_pl_id=partner_pricelist.id,
            order_pl_id=last_order_pricelist.id)

        return self.env['product.pricelist'].browse(pricelist_ids)

    def is_pricelist_available(self, pl_id):
        """ Return a boolean to specify if a specific pricelist can be manually set on the website.
        Warning: It check only if pricelist is in the 'selectable' pricelists or the current pricelist.
        :param int pl_id: The pricelist id to check
        :returns: Boolean, True if valid / available
        """
        return pl_id in self.get_pricelist_available(show_visible=False).ids

    def _get_geoip_country_code(self):
        return request and request.geoip.country_code or False

    def _get_cached_pricelist_id(self):
        return request and request.session.get('website_sale_current_pl') or None

    def _get_current_pricelist(self):
        """
        :returns: The current pricelist record
        """
        self = self.with_company(self.company_id)
        ProductPricelist = self.env['product.pricelist']

        pricelist = ProductPricelist
        if request and request.session.get('website_sale_current_pl'):
            # `website_sale_current_pl` is set only if the user specifically chose it:
            #  - Either, he chose it from the pricelist selection
            #  - Either, he entered a coupon code
            pricelist = ProductPricelist.browse(request.session['website_sale_current_pl']).exists().sudo()
            country_code = self._get_geoip_country_code()
            if not pricelist or not pricelist._is_available_on_website(self) or not pricelist._is_available_in_country(country_code):
                request.session.pop('website_sale_current_pl')
                pricelist = ProductPricelist

        if not pricelist:
            partner_sudo = self.env.user.partner_id

            # If the user has a saved cart, it take the pricelist of this last unconfirmed cart
            pricelist = partner_sudo.last_website_so_id.pricelist_id
            if not pricelist:
                # The pricelist of the user set on its partner form.
                # If the user is not signed in, it's the public user pricelist
                pricelist = partner_sudo.property_product_pricelist

            # The list of available pricelists for this user.
            # If the user is signed in, and has a pricelist set different than the public user pricelist
            # then this pricelist will always be considered as available
            available_pricelists = self.get_pricelist_available()
            if available_pricelists and pricelist not in available_pricelists:
                # If there is at least one pricelist in the available pricelists
                # and the chosen pricelist is not within them
                # it then choose the first available pricelist.
                # This can only happen when the pricelist is the public user pricelist and this pricelist is not in the available pricelist for this localization
                # If the user is signed in, and has a special pricelist (different than the public user pricelist),
                # then this special pricelist is amongs these available pricelists, and therefore it won't fall in this case.
                pricelist = available_pricelists[0]

        return pricelist

    def sale_product_domain(self):
        website_domain = self.get_current_website().website_domain()
        if not self.env.user._is_internal():
            website_domain = expression.AND([website_domain, [('is_published', '=', True)]])
        return expression.AND([self._product_domain(), website_domain])

    def _product_domain(self):
        return [('sale_ok', '=', True)]

    def sale_get_order(self, force_create=False, update_pricelist=False):
        """ Return the current sales order after mofications specified by params.

        :param bool force_create: Create sales order if not already existing
        :param bool update_pricelist: Force to recompute all the lines from sales order to adapt the price with the current pricelist.
        :returns: record for the current sales order (might be empty)
        :rtype: `sale.order` recordset
        """
        self.ensure_one()

        self = self.with_company(self.company_id)
        SaleOrder = self.env['sale.order'].sudo()

        sale_order_id = request.session.get('sale_order_id')

        if sale_order_id:
            sale_order_sudo = SaleOrder.browse(sale_order_id).exists()
        elif self.env.user and not self.env.user._is_public():
            sale_order_sudo = self.env.user.partner_id.last_website_so_id
            if sale_order_sudo:
                available_pricelists = self.get_pricelist_available()
                so_pricelist_sudo = sale_order_sudo.pricelist_id
                if so_pricelist_sudo and so_pricelist_sudo not in available_pricelists:
                    # Do not reload the cart of this user last visit
                    # if the cart uses a pricelist no longer available.
                    sale_order_sudo = SaleOrder
                else:
                    # Do not reload the cart of this user last visit
                    # if the Fiscal Position has changed.
                    fpos = sale_order_sudo.env['account.fiscal.position'].with_company(
                        sale_order_sudo.company_id
                    )._get_fiscal_position(
                        sale_order_sudo.partner_id,
                        delivery=sale_order_sudo.partner_shipping_id
                    )
                    if fpos.id != sale_order_sudo.fiscal_position_id.id:
                        sale_order_sudo = SaleOrder
        else:
            sale_order_sudo = SaleOrder

        # Ignore the current order if a payment has been initiated. We don't want to retrieve the
        # cart and allow the user to update it when the payment is about to confirm it.
        if sale_order_sudo and sale_order_sudo.get_portal_last_transaction().state in (
            'pending', 'authorized', 'done'
        ):
            sale_order_sudo = None

        if not (sale_order_sudo or force_create):
            # Do not create a SO record unless needed
            if request.session.get('sale_order_id'):
                request.session.pop('sale_order_id')
                request.session.pop('website_sale_cart_quantity', None)
            return self.env['sale.order']

        # Only set when neeeded
        pricelist_id = False

        partner_sudo = self.env.user.partner_id

        # cart creation was requested
        if not sale_order_sudo:
            so_data = self._prepare_sale_order_values(partner_sudo)
            sale_order_sudo = SaleOrder.with_user(SUPERUSER_ID).create(so_data)

            request.session['sale_order_id'] = sale_order_sudo.id
            request.session['website_sale_cart_quantity'] = sale_order_sudo.cart_quantity
            # The order was created with SUPERUSER_ID, revert back to request user.
            sale_order_sudo = sale_order_sudo.with_user(self.env.user).sudo()
            return sale_order_sudo

        # Existing Cart:
        #   * For logged user
        #   * In session, for specified partner

        # case when user emptied the cart
        if not request.session.get('sale_order_id'):
            request.session['sale_order_id'] = sale_order_sudo.id
            request.session['website_sale_cart_quantity'] = sale_order_sudo.cart_quantity

        # check for change of partner_id ie after signup
        if sale_order_sudo.partner_id.id != partner_sudo.id and request.website.partner_id.id != partner_sudo.id:
            previous_fiscal_position = sale_order_sudo.fiscal_position_id
            previous_pricelist = sale_order_sudo.pricelist_id

            # Reset the session pricelist according to logged partner pl
            request.session.pop('website_sale_current_pl', None)
            # Force recomputation of the website pricelist after reset
            self.invalidate_recordset(['pricelist_id'])
            pricelist_id = self.pricelist_id.id
            request.session['website_sale_current_pl'] = pricelist_id

            # change the partner, and trigger the computes (fpos)
            sale_order_sudo.write({
                'partner_id': partner_sudo.id,
                'payment_term_id': self.sale_get_payment_term(partner_sudo),
                # Must be specified to ensure it is not recomputed when it shouldn't
                'pricelist_id': pricelist_id,
            })

            if sale_order_sudo.fiscal_position_id != previous_fiscal_position:
                sale_order_sudo.order_line._compute_tax_id()

            if sale_order_sudo.pricelist_id != previous_pricelist:
                update_pricelist = True
        elif update_pricelist:
            # Only compute pricelist if needed
            pricelist_id = self.pricelist_id.id

        # update the pricelist
        if update_pricelist:
            request.session['website_sale_current_pl'] = pricelist_id
            sale_order_sudo.write({'pricelist_id': pricelist_id})
            sale_order_sudo._recompute_prices()

        return sale_order_sudo

    def _prepare_sale_order_values(self, partner_sudo):
        self.ensure_one()
        addr = partner_sudo.address_get(['delivery', 'invoice'])
        if not request.website.is_public_user():
            last_sale_order = self.env['sale.order'].sudo().search(
                [('partner_id', '=', partner_sudo.id), ('website_id', '=', self.id)],
                limit=1,
                order="date_order desc, id desc",
            )
            if last_sale_order:
                partner_shipping = last_sale_order.partner_shipping_id
                if (
                    partner_shipping.active
                    and partner_shipping.commercial_partner_id == partner_sudo
                ):
                    addr['delivery'] = partner_shipping.id
                partner_invoice = last_sale_order.partner_invoice_id
                if (
                    partner_invoice.active
                    and partner_invoice.commercial_partner_id == partner_sudo
                ):
                    addr['invoice'] = partner_invoice.id

        affiliate_id = request.session.get('affiliate_id')
        salesperson_user_sudo = self.env['res.users'].sudo().browse(affiliate_id).exists()
        if not salesperson_user_sudo:
            salesperson_user_sudo = self.salesperson_id or partner_sudo.parent_id.user_id or partner_sudo.user_id

        values = {
            'company_id': self.company_id.id,

            'fiscal_position_id': self.fiscal_position_id.id,
            'partner_id': partner_sudo.id,
            'partner_invoice_id': addr['invoice'],
            'partner_shipping_id': addr['delivery'],

            'pricelist_id': self.pricelist_id.id,
            'payment_term_id': self.sale_get_payment_term(partner_sudo),

            'team_id': self.salesteam_id.id or partner_sudo.parent_id.team_id.id or partner_sudo.team_id.id,
            'user_id': salesperson_user_sudo.id,
            'website_id': self.id,
        }

        return values

    @api.model
    def sale_get_payment_term(self, partner):
        pt = self.env.ref('account.account_payment_term_immediate', False)
        if pt:
            pt = pt.sudo()
            pt = (not pt.company_id.id or self.company_id.id == pt.company_id.id) and pt
        return (
            partner.property_payment_term_id or
            pt or
            self.env['account.payment.term'].sudo().search([('company_id', '=', self.company_id.id)], limit=1)
        ).id

    def _get_current_fiscal_position(self):
        AccountFiscalPosition = self.env['account.fiscal.position'].sudo()
        fpos = AccountFiscalPosition
        partner_sudo = self.env.user.partner_id

        # If the current user is the website public user, the fiscal position
        # is computed according to geolocation.
        if request and request.geoip.country_code and self.partner_id.id == partner_sudo.id:
            country = self.env['res.country'].search(
                [('code', '=', request.geoip.country_code)],
                limit=1,
            )
            fpos = AccountFiscalPosition._get_fpos_by_region(country.id)

        if not fpos:
            fpos = AccountFiscalPosition._get_fiscal_position(partner_sudo)

        return fpos

    def sale_reset(self):
        request.session.pop('sale_order_id', None)
        request.session.pop('website_sale_current_pl', None)
        request.session.pop('website_sale_cart_quantity', None)

    @api.model
    def action_dashboard_redirect(self):
        if self.env.user.has_group('sales_team.group_sale_salesman'):
            return self.env["ir.actions.actions"]._for_xml_id("website.backend_dashboard")
        return super(Website, self).action_dashboard_redirect()

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('eCommerce'), url_for('/shop'), 'website_sale'))
        return suggested_controllers

    def _search_get_details(self, search_type, order, options):
        result = super()._search_get_details(search_type, order, options)
        if search_type in ['products', 'product_categories_only', 'all']:
            result.append(self.env['product.public.category']._search_get_detail(self, order, options))
        if search_type in ['products', 'products_only', 'all']:
            result.append(self.env['product.template']._search_get_detail(self, order, options))
        return result

    def _get_product_page_proportions(self):
        """
        Returns the number of columns (css) that both the images and the product details should take.
        """
        self.ensure_one()

        return {
            'none': (0, 12),
            '50_pc': (6, 6),
            '66_pc': (8, 4),
            '100_pc': (12, 12),
        }.get(self.product_page_image_width)

    def _get_product_page_grid_image_classes(self):
        spacing_map = {
            'none': 'p-0',
            'small': 'p-2',
            'medium': 'p-3',
            'big': 'p-4',
        }
        columns_map = {
            1: 'col-12',
            2: 'col-6',
            3: 'col-4',
        }
        return spacing_map.get(self.product_page_image_spacing) + ' ' +\
                columns_map.get(self.product_page_grid_columns)

    @api.model
    def _send_abandoned_cart_email(self):
        for website in self.search([]):
            if not website.send_abandoned_cart_email:
                continue
            all_abandoned_carts = self.env['sale.order'].search([
                ('is_abandoned_cart', '=', True),
                ('cart_recovery_email_sent', '=', False),
                ('website_id', '=', website.id),
            ])
            if not all_abandoned_carts:
                continue

            abandoned_carts = all_abandoned_carts._filter_can_send_abandoned_cart_mail()
            # Mark abandoned carts that failed the filter as sent to avoid rechecking them again and again.
            (all_abandoned_carts - abandoned_carts).cart_recovery_email_sent = True
            for sale_order in abandoned_carts:
                template = self.env.ref('website_sale.mail_template_sale_cart_recovery')
                template.send_mail(sale_order.id, email_values=dict(email_to=sale_order.partner_id.email))
                sale_order.cart_recovery_email_sent = True

    def _display_partner_b2b_fields(self):
        """ This method is to be inherited by localizations and return
        True if localization should always displayed b2b fields """
        self.ensure_one()

        return self.is_view_active('website_sale.address_b2b')

    def _get_checkout_step_list(self):
        """ Return an ordered list of steps according to the current template rendered.

        :rtype: list
        :return: A list with the following structure:
            [
                [xmlid],
                {
                    'name': str,
                    'current_href': str,
                    'main_button': str,
                    'main_button_href': str,
                    'back_button': str,
                    'back_button_href': str
                }
            ]
        """
        self.ensure_one()
        is_extra_step_active = self.viewref('website_sale.extra_info').active
        redirect_to_sign_in = self.account_on_checkout == 'mandatory' and self.is_public_user()

        steps = [(['website_sale.cart'], {
            'name': _lt("Review Order"),
            'current_href': '/shop/cart',
            'main_button': _lt("Sign In") if redirect_to_sign_in else _lt("Checkout"),
            'main_button_href': f'{"/web/login?redirect=" if redirect_to_sign_in else ""}/shop/checkout?express=1',
            'back_button':  _lt("Continue shopping"),
            'back_button_href': '/shop',
        }), (['website_sale.checkout', 'website_sale.address'], {
            'name': _lt("Shipping"),
            'current_href': '/shop/checkout',
            'main_button': _lt("Confirm"),
            'main_button_href': f'{"/shop/extra_info" if is_extra_step_active else "/shop/confirm_order"}',
            'back_button':  _lt("Back to cart"),
            'back_button_href': '/shop/cart',
        })]
        if is_extra_step_active:
            steps.append((['website_sale.extra_info'], {
                'name': _lt("Extra Info"),
                'current_href': '/shop/extra_info',
                'main_button': _lt("Continue checkout"),
                'main_button_href': '/shop/confirm_order',
                'back_button':  _lt("Return to shipping"),
                'back_button_href': '/shop/checkout',
            }))
        steps.append((['website_sale.payment'], {
            'name': _lt("Payment"),
            'current_href': '/shop/payment',
            'back_button':  _lt("Back to cart"),
            'back_button_href': '/shop/cart',
        }))
        return steps

    def _get_checkout_steps(self, current_step=None):
        """ Return an ordered list of steps according to the current template rendered.
        If `current_step` is provided, returns only the corresponding step.
        Note: self.ensure_one()
        :param str current_step: The xmlid of the current step, defaults to None.
        :rtype: list
        :return: A list containing the steps generated by :meth:`_get_checkout_step_list`.
        """
        self.ensure_one()

        steps = self._get_checkout_step_list()

        if current_step:
            return next(step for step in steps if current_step in step[0])[1]
        else:
            return steps

class WebsiteSaleExtraField(models.Model):
    _name = 'website.sale.extra.field'
    _description = 'E-Commerce Extra Info Shown on product page'
    _order = 'sequence'

    website_id = fields.Many2one('website')
    sequence = fields.Integer(default=10)
    field_id = fields.Many2one(
        'ir.model.fields',
        domain=[('model_id.model', '=', 'product.template'), ('ttype', 'in', ['char', 'binary'])],
        required=True,
        ondelete='cascade'
    )
    label = fields.Char(related='field_id.field_description')
    name = fields.Char(related='field_id.name')

```

## File: models\website_base_unit.py

```python
from odoo import fields, models


class WebsiteBaseUnit(models.Model):
    _name = "website.base.unit"
    _description = "Unit of Measure for price per unit on eCommerce products."
    _order = "name"

    name = fields.Char(help="Define a custom unit to display in the price per unit of measure field.",
                       required=True, translate=True)

```

## File: models\website_snippet_filter.py

```python
# -*- coding: utf-8 -*-

from collections import Counter

from odoo import models, fields, api, _
from odoo.osv import expression


class WebsiteSnippetFilter(models.Model):
    _inherit = 'website.snippet.filter'

    product_cross_selling = fields.Boolean(string="About cross selling products", default=False,
        help="True only for product filters that require a product_id because they relate to cross selling")

    @api.model
    def _get_website_currency(self):
        website = self.env['website'].get_current_website()
        return website.currency_id

    def _get_hardcoded_sample(self, model):
        samples = super()._get_hardcoded_sample(model)
        if model._name == 'product.product':
            data = [{
                'image_512': b'/product/static/img/product_chair.jpg',
                'display_name': _('Chair'),
                'description_sale': _('Sit comfortably'),
            }, {
                'image_512': b'/product/static/img/product_lamp.png',
                'display_name': _('Lamp'),
                'description_sale': _('Lightbulb sold separately'),
            }, {
                'image_512': b'/product/static/img/product_product_20-image.png',
                'display_name': _('Whiteboard'),
                'description_sale': _('With three feet'),
            }, {
                'image_512': b'/product/static/img/product_product_27-image.jpg',
                'display_name': _('Drawer'),
                'description_sale': _('On wheels'),
            }, {
                'image_512': b'/product/static/img/product_product_7-image.png',
                'display_name': _('Box'),
                'description_sale': _('Reinforced for heavy loads'),
            }, {
                'image_512': b'/product/static/img/product_product_9-image.jpg',
                'display_name': _('Bin'),
                'description_sale': _('Pedal-based opening system'),
            }]
            merged = []
            for index in range(0, max(len(samples), len(data))):
                merged.append({**samples[index % len(samples)], **data[index % len(data)]})
                # merge definitions
            samples = merged
        return samples

    def _filter_records_to_values(self, records, is_sample=False):
        res_products = super()._filter_records_to_values(records, is_sample)
        if self.model_name == 'product.product':
            for res_product in res_products:
                product = res_product.get('_record')
                if not is_sample:
                    res_product.update(product._get_combination_info_variant())
                    if records.env.context.get('add2cart_rerender'):
                        res_product['_add2cart_rerender'] = True
        return res_products

    @api.model
    def _get_products(self, mode, context):
        dynamic_filter = context.get('dynamic_filter')
        handler = getattr(self, '_get_products_%s' % mode, self._get_products_latest_sold)
        website = self.env['website'].get_current_website()
        search_domain = context.get('search_domain')
        limit = context.get('limit')
        domain = expression.AND([
            [('website_published', '=', True)] if self.env.user._is_public() or self.env.user._is_portal() else [],
            website.website_domain(),
            [('company_id', 'in', [False, website.company_id.id])],
            search_domain or [],
        ])
        products = handler(website, limit, domain, context)
        return dynamic_filter._filter_records_to_values(products, False)

    def _get_products_latest_sold(self, website, limit, domain, context):
        products = []
        sale_orders = self.env['sale.order'].sudo().search([
            ('website_id', '=', website.id),
            ('state', '=', 'sale'),
        ], limit=8, order='date_order DESC')
        if sale_orders:
            sold_products = [p.product_id.id for p in sale_orders.order_line]
            products_ids = [id for id, _ in Counter(sold_products).most_common()]
            if products_ids:
                domain = expression.AND([
                    domain,
                    [('id', 'in', products_ids)],
                ])
                products = self.env['product.product'].with_context(display_default_code=False).search(domain)
                products = products.sorted(key=lambda p: products_ids.index(p.id))[:limit]
        return products

    def _get_products_latest_viewed(self, website, limit, domain, context):
        products = []
        visitor = self.env['website.visitor']._get_visitor_from_request()
        if visitor:
            excluded_products = website.sale_get_order().order_line.product_id.ids
            tracked_products = self.env['website.track'].sudo()._read_group(
                [('visitor_id', '=', visitor.id), ('product_id', '!=', False), ('product_id.website_published', '=', True), ('product_id', 'not in', excluded_products)],
                ['product_id'], limit=limit, order='visit_datetime:max DESC')
            products_ids = [product.id for [product] in tracked_products]
            if products_ids:
                domain = expression.AND([
                    domain,
                    [('id', 'in', products_ids)],
                ])
                filtered_ids = set(self.env['product.product']._search(domain, limit=limit))
                # `search` will not keep the order of tracked products; however, we want to keep
                # that order (latest viewed first).
                products = self.env['product.product'].with_context(
                    display_default_code=False, add2cart_rerender=True,
                ).browse([product_id for product_id in products_ids if product_id in filtered_ids])

        return products

    def _get_products_recently_sold_with(self, website, limit, domain, context):
        products = []
        current_id = context.get('product_template_id')
        if current_id:
            current_id = int(current_id)
            sale_orders = self.env['sale.order'].sudo().search([
                ('website_id', '=', website.id),
                ('state', '=', 'sale'),
                ('order_line.product_id.product_tmpl_id', '=', current_id),
            ], limit=8, order='date_order DESC')
            if sale_orders:
                current_template = self.env['product.template'].browse(current_id)
                excluded_products = website.sale_get_order().order_line.product_id.product_tmpl_id.product_variant_ids.ids
                excluded_products.extend(current_template.product_variant_ids.ids)
                included_products = []
                for sale_order in sale_orders:
                    included_products.extend(sale_order.order_line.product_id.ids)
                products_ids = list(set(included_products) - set(excluded_products))
                if products_ids:
                    domain = expression.AND([
                        domain,
                        [('id', 'in', products_ids)],
                    ])
                    products = self.env['product.product'].with_context(display_default_code=False).search(domain, limit=limit)
        return products

    def _get_products_accessories(self, website, limit, domain, context):
        products = []
        current_id = context.get('product_template_id')
        if current_id:
            current_id = int(current_id)
            current_template = self.env['product.template'].browse(current_id)
            if current_template.exists():
                excluded_products = website.sale_get_order().order_line.product_id.ids
                excluded_products.extend(current_template.product_variant_ids.ids)
                included_products = current_template._get_website_accessory_product().ids
                products_ids = list(set(included_products) - set(excluded_products))
                if products_ids:
                    domain = expression.AND([
                        domain,
                        [('id', 'in', products_ids)],
                    ])
                    products = self.env['product.product'].with_context(display_default_code=False).search(domain, limit=limit)
        return products

    def _get_products_alternative_products(self, website, limit, domain, context):
        products = self.env['product.product']
        current_id = context.get('product_template_id')
        if not current_id:
            return products
        current_template = self.env['product.template'].browse(int(current_id))
        if current_template.exists():
            excluded_products = website.sale_get_order().order_line.product_id
            excluded_products |= current_template.product_variant_ids
            included_products = current_template.alternative_product_ids.product_variant_ids
            products = included_products - excluded_products
            if products:
                domain = expression.AND([
                    domain,
                    [('id', 'in', products.ids)],
                ])
                products = self.env['product.product'].with_context(display_default_code=False).search(domain, limit=limit)
        return products

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta

from odoo import fields, models, api

class WebsiteTrack(models.Model):
    _inherit = 'website.track'

    product_id = fields.Many2one('product.product', ondelete='cascade', readonly=True, index='btree_not_null')


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    visitor_product_count = fields.Integer('Product Views', compute="_compute_product_statistics", help="Total number of views on products")
    product_ids = fields.Many2many('product.product', string="Visited Products", compute="_compute_product_statistics")
    product_count = fields.Integer('Products Views', compute="_compute_product_statistics", help="Total number of product viewed")

    @api.depends('website_track_ids')
    def _compute_product_statistics(self):
        results = self.env['website.track']._read_group([
            ('visitor_id', 'in', self.ids), ('product_id', '!=', False),
            ('product_id', 'any', self.env['product.product']._check_company_domain(self.env.companies)),
        ], ['visitor_id'], ['product_id:array_agg', '__count'])
        mapped_data = {
            visitor.id: {'product_count': count, 'product_ids': product_ids}
            for visitor, product_ids, count in results
        }

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
            website_track_values = {'product_id': product_id}
            self._add_tracking(domain, website_track_values)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import crm_team
from . import delivery_carrier
from . import digest
from . import ir_http
from . import payment_token
from . import product_attribute
from . import product_document
from . import product_image
from . import product_pricelist
from . import product_product
from . import product_public_category
from . import product_ribbon
from . import product_tag
from . import product_template
from . import product_template_attribute_line
from . import product_template_attribute_value
from . import res_company
from . import res_config_settings
from . import res_country
from . import res_partner
from . import sale_order
from . import sale_order_line
from . import website
from . import website_base_unit
from . import website_snippet_filter
from . import website_visitor

```

## File: populate\product_attribute.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import populate


class ProductAttribute(models.Model):
    _inherit = 'product.attribute'

    def _populate_factories(self):
        return super()._populate_factories() + [
            ('visibility', populate.randomize(['visible', 'hidden'], [6, 3])),
        ]

```

## File: populate\product_product.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import populate


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _populate_get_product_factories(self):
        """Populate the invoice_policy of product.product & product.template models."""
        return super()._populate_get_product_factories() + [
            ('is_published', populate.randomize([True, False], [8, 2]))]

```

## File: populate\product_public_category.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
import logging

from odoo import models
from odoo.fields import Command
from odoo.tools import populate

_logger = logging.getLogger(__name__)


class ProductPublicCategory(models.Model):
    _inherit = 'product.public.category'
    _populate_sizes = {'small': 20, 'medium': 100, 'large': 1_500}
    _populate_dependencies = ['product.template']

    def _populate_factories(self):

        p_tmpl_ids = self.env.registry.populated_models['product.template']
        max_products_in_category = min(len(p_tmpl_ids), 500)
        def get_products(iterator, field_name, model_name):
            random = populate.Random('product_public_category_products')
            for values in iterator:
                # Fixed price, percentage, formula
                number_of_products = random.randint(1, max_products_in_category)
                product_tmpl_ids = set()
                for _i in range(number_of_products):
                    product_tmpl_ids.add(
                        random.choice(p_tmpl_ids)
                    )
                values['product_tmpl_ids'] = [Command.set(product_tmpl_ids)]
                yield values

        return [
            ('name', populate.constant('PC_{counter}')),
            ('sequence', populate.randomize([False] + [i for i in range(1, 101)])),
            ('_products', get_products),
        ]

    def _populate(self, size):
        categories = super()._populate(size)
        # Set parent/child relation
        self._populate_set_parents(categories, size)
        return categories

    def _populate_set_parents(self, categories, size):
        _logger.info('Set parent/child relation of product categories')
        parent_ids = []
        rand = populate.Random('product.public.category+parent_generator')

        for category in categories:
            if rand.random() < 0.25:
                parent_ids.append(category.id)

        categories -= self.browse(parent_ids)  # Avoid recursion in parent-child relations.
        parent_childs = defaultdict(lambda: self.env['product.public.category'])
        for category in categories:
            if rand.random() < 0.25:  # 1/4 of remaining categories have a parent.
                parent_childs[rand.choice(parent_ids)] |= category

        for count, (parent, children) in enumerate(parent_childs.items()):
            if (count + 1) % 1000 == 0:
                _logger.info('Setting parent: %s/%s', count + 1, len(parent_childs))
            children.write({'parent_id': parent})

```

## File: populate\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_attribute
from . import product_product
from . import product_public_category

```

## File: report\sale_report.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class SaleReport(models.Model):
    _inherit = 'sale.report'

    website_id = fields.Many2one('website', readonly=True)
    is_abandoned_cart = fields.Boolean(string="Abandoned Cart", readonly=True)

    def _select_additional_fields(self):
        res = super()._select_additional_fields()
        res['website_id'] = "s.website_id"
        res['is_abandoned_cart'] = """
            s.date_order <= (timezone('utc', now()) - ((COALESCE(w.cart_abandoned_delay, '1.0') || ' hour')::INTERVAL))
            AND s.website_id IS NOT NULL
            AND s.state = 'draft'
            AND s.partner_id != %s""" % self.env.ref('base.public_partner').id
        return res

    def _from_sale(self):
        res = super()._from_sale()
        res += """
            LEFT JOIN website w ON w.id = s.website_id"""
        return res

    def _group_by_sale(self):
        res = super()._group_by_sale()
        res += """,
            s.website_id,
            w.cart_abandoned_delay"""
        return res

```

## File: report\sale_report_views.xml

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
                <filter string="Confirmed Orders" name="confirmed" domain="[('state', '=', 'sale')]"/>
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
                    <filter string="Last Week" invisible="1" name="week" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=7)).strftime('%Y-%m-%d'))]"/>
                    <filter string="Last Month" invisible="1" name="month" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                    <filter string="Last Year" invisible="1"  name="year" domain="[('date','&gt;=', (context_today() - datetime.timedelta(days=365)).strftime('%Y-%m-%d'))]"/>
                </group>
            </search>
        </field>
    </record>

    <record id="sale_report_view_pivot_website" model="ir.ui.view">
        <field name="name">sale.report.view.pivot.website</field>
        <field name="model">sale.report</field>
        <field name="arch" type="xml">
            <pivot string="Sales Analysis" sample="1">
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
            <graph string="Sale Analysis" sample="1">
                <field name="date"/>
                <field name="price_subtotal" type='measure'/>
            </graph>
        </field>
    </record>

    <record id="sale_report_view_tree" model="ir.ui.view">
        <field name="name">sale.report.view.tree.inherit.website.sale</field>
        <field name="model">sale.report</field>
        <field name="inherit_id" ref="sale.sale_report_view_tree"/>
        <field name="arch" type="xml">
             <field name="order_reference" position="after">
                <field name="website_id" optional="hide"/>
            </field>
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

## File: report\__init__.py

```python
# coding: utf-8
from . import sale_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_product_public_public,product.product.public,product.model_product_product,base.group_public,1,0,0,0
access_product_product_public_portal,product.product.public,product.model_product_product,base.group_portal,1,0,0,0
access_product_product_public_employee,product.product.public,product.model_product_product,base.group_user,1,0,0,0
access_product_template_public_public,product.template.public,product.model_product_template,base.group_public,1,0,0,0
access_product_template_public_portal,product.template.public,product.model_product_template,base.group_portal,1,0,0,0
access_product_template_public_employee,product.template.public,product.model_product_template,base.group_user,1,0,0,0
access_product_category_public_public,product.category.public,product.model_product_category,base.group_public,1,0,0,0
access_product_category_public_portal,product.category.public,product.model_product_category,base.group_portal,1,0,0,0
access_product_category_public_employee,product.category.public,product.model_product_category,base.group_user,1,0,0,0
access_product_tag_public_public,product.tag.public,product.model_product_tag,base.group_public,1,0,0,0
access_product_tag_public_portal,product.tag.public,product.model_product_tag,base.group_portal,1,0,0,0
access_product_tag_public_employee,product.tag.public,product.model_product_tag,base.group_user,1,0,0,0
access_product_category_pos_manager,product.public.category manager,model_product_public_category,sales_team.group_sale_manager,1,1,1,1
access_product_public_category_public_public,product.category.public,model_product_public_category,base.group_public,1,0,0,0
access_product_public_category_public_portal,product.category.public,model_product_public_category,base.group_portal,1,0,0,0
access_product_public_category_public_employee,product.category.public,model_product_public_category,base.group_user,1,0,0,0
access_product_pricelist_public_public,product.pricelist.public,product.model_product_pricelist,base.group_public,1,0,0,0
access_product_pricelist_public_portal,product.pricelist.public,product.model_product_pricelist,base.group_portal,1,0,0,0
access_product_pricelist_public_employee,product.pricelist.public,product.model_product_pricelist,base.group_user,1,0,0,0
access_product_pricelist_item_public_public,product.pricelist.item.public,product.model_product_pricelist_item,base.group_public,1,0,0,0
access_product_pricelist_item_public_portal,product.pricelist.item.public,product.model_product_pricelist_item,base.group_portal,1,0,0,0
access_product_pricelist_item_public_employee,product.pricelist.item.public,product.model_product_pricelist_item,base.group_user,1,0,0,0
access_product_ribbon_public,product.ribbon.public,website_sale.model_product_ribbon,base.group_user,1,0,0,0
access_product_ribbon_sale_manager,product.ribbon.sale_manager,website_sale.model_product_ribbon,sales_team.group_sale_manager,1,1,1,1
access_product_attribute_public_public,product.attribute public,product.model_product_attribute,base.group_public,1,0,0,0
access_product_attribute_public_portal,product.attribute public,product.model_product_attribute,base.group_portal,1,0,0,0
access_product_attribute_public_employee,product.attribute public,product.model_product_attribute,base.group_user,1,0,0,0
access_product_attribute_value_public_public,product.attribute value public,product.model_product_attribute_value,base.group_public,1,0,0,0
access_product_attribute_value_public_portal,product.attribute value public,product.model_product_attribute_value,base.group_portal,1,0,0,0
access_product_attribute_value_public_employee,product.attribute value public,product.model_product_attribute_value,base.group_user,1,0,0,0
access_product_product_attribute_public,product.template.attribute value public,product.model_product_template_attribute_value,base.group_public,1,0,0,0
access_product_product_attribute_portal,product.template.attribute value public,product.model_product_template_attribute_value,base.group_portal,1,0,0,0
access_product_product_attribute_employee,product.template.attribute value public,product.model_product_template_attribute_value,base.group_user,1,0,0,0
access_product_product_attribute_custom_value_public,product.attribute.custom value,sale.model_product_attribute_custom_value,base.group_public,1,0,0,0
access_product_product_attribute_custom_value_portal,product.attribute.custom value,sale.model_product_attribute_custom_value,base.group_portal,1,0,0,0
access_product_product_attribute_custom_value_employee,product.attribute.custom value,sale.model_product_attribute_custom_value,base.group_user,1,0,0,0
access_product_template_attribute_exclusion_public,product.template.attribute exclusion public,product.model_product_template_attribute_exclusion,base.group_public,1,0,0,0
access_product_template_attribute_exclusion_portal,product.template.attribute exclusion public,product.model_product_template_attribute_exclusion,base.group_portal,1,0,0,0
access_product_template_attribute_exclusion_employee,product.template.attribute exclusion public,product.model_product_template_attribute_exclusion,base.group_user,1,0,0,0
access_product_template_attribute_line_public_public,product.template.attribute line public,product.model_product_template_attribute_line,base.group_public,1,0,0,0
access_product_template_attribute_line_public_portal,product.template.attribute line public,product.model_product_template_attribute_line,base.group_portal,1,0,0,0
access_product_template_attribute_line_public_employee,product.template.attribute line public,product.model_product_template_attribute_line,base.group_user,1,0,0,0
access_fiscal_position_public,fiscal position public,account.model_account_fiscal_position,base.group_portal,1,0,0,0
access_payment_term,payment term public,account.model_account_payment_term,base.group_portal,1,0,0,0
access_account_tax_user,account.tax,account.model_account_tax,base.group_public,1,0,0,0
access_product_image_public_public,product.image public,model_product_image,base.group_public,1,0,0,0
access_product_image_public_portal,product.image public,model_product_image,base.group_portal,1,0,0,0
access_product_image_public_employee,product.image public,model_product_image,base.group_user,1,0,0,0
access_product_image_restricted_editor,product.image wbesite restricted_editor,model_product_image,website.group_website_restricted_editor,1,1,1,1
access_product_image_sale,product.image sale,model_product_image,sales_team.group_sale_manager,1,1,1,1
access_ecom_extra_fields_public_public,access_ecom_extra_field public,model_website_sale_extra_field,base.group_public,1,0,0,0
access_ecom_extra_fields_public_portal,access_ecom_extra_field public,model_website_sale_extra_field,base.group_portal,1,0,0,0
access_ecom_extra_fields_public_employee,access_ecom_extra_field public,model_website_sale_extra_field,base.group_user,1,0,0,0
access_ecom_extra_fields_restricted_editor,access_ecom_extra_field restricted_editor,model_website_sale_extra_field,website.group_website_restricted_editor,1,1,1,1
access_website_base_unit_public_public,website.base.unit public,model_website_base_unit,base.group_public,1,0,0,0
access_website_base_unit_public_portal,website.base.unit public,model_website_base_unit,base.group_portal,1,0,0,0
access_website_base_unit_public_employee,website.base.unit public,model_website_base_unit,base.group_user,1,0,0,0
access_website_base_unit_sale_manager,website.base.unit sale manager,model_website_base_unit,sales_team.group_sale_manager,1,1,1,1

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

    <record id="group_show_uom_price" model="res.groups">
        <field name="name">UOM Price Display for eCommerce</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_product_price_comparison" model="res.groups">
        <field name="name">Comparison Price</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="sales_team.group_sale_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_restricted_editor'))]"/>
    </record>

    <record id="base.group_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('account.group_delivery_invoice_address'))]"/>
    </record>

    <record id="base.group_public" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('account.group_delivery_invoice_address'))]"/>
    </record>

    <record id="base.group_portal" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('account.group_delivery_invoice_address'))]"/>
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M15.724 6.397C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397L37.236 13H41.9c2.46 0 4.367 2.099 4.07 4.481l-3.106 25C42.613 44.49 40.866 46 38.793 46H11.207c-2.074 0-3.82-1.51-4.07-3.519l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603ZM32.917 13H17.082c0-.56.123-1.134.39-1.691l.956-2C19.102 7.9 20.551 7 22.144 7h5.711c1.593 0 3.042.9 3.716 2.308l.957 2c.266.558.39 1.132.39 1.692Z" fill="#712258"/><path fill-rule="evenodd" clip-rule="evenodd" d="M8.514 45.016a3.963 3.963 0 0 1-1.377-2.535l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397l2.59 5.777C35.5 28.256 23.848 41.405 8.515 45.016ZM17.082 13h15.835c0-.56-.123-1.134-.39-1.691l-.956-2C30.897 7.9 29.448 7 27.855 7h-5.711c-1.593 0-3.042.9-3.716 2.308l-.956 2a3.904 3.904 0 0 0-.39 1.692Z" fill="#985184"/></svg>

```

## File: static\src\img\snippets_options\product_add_to_cart.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_add_to_cart_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_add_to_cart_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_add_to_cart_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_add_to_cart_item">
            <rect width="48" height="60" fill="#ffffff"/>
            <rect x="5" y="43" width="27" height="2" fill="#222222"/>
            <rect x="36" y="47" width="7" height="7" rx="2" fill="#3aadaa"/>
            <rect x="11" y="50" width="15" height="2" fill="#595959"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(4.5 63)" fill="#595959"/>
            <rect width="48" height="37" fill="#9ccde4"/>
        </g>
    </defs> 
    <use href="#product_add_to_cart_item" y="5" x="5"/>
    <use href="#product_add_to_cart_shirt" y="11" x="15"/>
    <use href="#product_add_to_cart_item" y="5" x="66"/>
    <use href="#product_add_to_cart_pants" y="11" x="83"/>
    <use href="#product_add_to_cart_item" y="5" x="126"/>
    <use href="#product_add_to_cart_shirt" y="11" x="137"/>
    <use href="#product_add_to_cart_item" y="5" x="187"/>
    <use href="#product_add_to_cart_watch" y="11" x="205"/>
</svg>

```

## File: static\src\img\snippets_options\product_banner.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_banner_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_banner_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_banner_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_banner_item">
            <rect width="230" height="60" fill="#fff"/>
            <rect width="115" height="60" x="115" fill="#9ccde4"/>
            <rect width="67" height="2" x="10" y="6" fill="#222"/>
            <rect width="31" height="8" rx="4" x="10" y="16" fill="#3aadaa"/>
            <rect width="15" height="2" x="21" y="19" fill="#fff"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(13 33)" fill="#fff"/>
            <rect width="75" height="1" x="10" y="32" fill="#666"/>
            <rect width="66" height="1" x="10" y="38" fill="#666"/>
            <rect width="62" height="1" x="10" y="35" fill="#666"/>
            <rect width="46" height="1" x="10" y="41" fill="#666"/>
            <rect width="28" height="7" x="10" y="48" rx="2" fill="#3aadaa"/>
            <rect width="12" height="1" x="18" y="51" fill="#fff"/>
            <rect width="12" height="1" x="47" y="51" fill="#3aadaa"/>
            <use href="#product_banner_shirt" y="18" x="132"/>
            <use href="#product_banner_watch" y="18" x="170"/>
            <use href="#product_banner_pants" y="18" x="195"/>
        </g>
    </defs> 
    <use href="#product_banner_item" y="5" x="5"/>
</svg>

```

## File: static\src\img\snippets_options\product_borderless_1.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_borderless_1_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_borderless_1_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_borderless_1_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_borderless_1_item">
            <rect width="48" height="44" fill="#9ccde4"/>
            <rect width="42" height="2" transform="translate(3 49.514)" fill="#c7c7c7"/>
            <rect width="15" height="2" transform="translate(9.024 56.117)" fill="#fff"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(3 70)" fill="#fff"/>
        </g>
    </defs> 
    <use href="#product_borderless_1_item" y="5" x="5"/>
    <use href="#product_borderless_1_shirt" y="15" x="15"/>
    <use href="#product_borderless_1_item" y="5" x="66"/>
    <use href="#product_borderless_1_pants" y="15" x="83"/>
    <use href="#product_borderless_1_item" y="5" x="126"/>
    <use href="#product_borderless_1_shirt" y="15" x="137"/>
    <use href="#product_borderless_1_item" y="5" x="187"/>
    <use href="#product_borderless_1_watch" y="15" x="205"/>
</svg>

```

## File: static\src\img\snippets_options\product_borderless_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_borderless_2_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_borderless_2_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_borderless_2_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_borderless_2_item">
            <rect width="48" height="37" fill="#9ccde4"/>
            <rect width="42" height="2" x="3" y="47" fill="#c7c7c7"/>
            <rect width="42" height="7" rx="2" x="3" y="53" fill="#3aadaa"/>
            <rect width="16" height="1" x="16" y="56" fill="#ffffff"/>
            <rect width="15" height="2" x="9" y="42" fill="#ffffff"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(3 56)" fill="#ffffff"/>
        </g>
    </defs> 
    <use href="#product_borderless_2_item" y="5" x="5"/>
    <use href="#product_borderless_2_shirt" y="12" x="15"/>
    <use href="#product_borderless_2_item" y="5" x="66"/>
    <use href="#product_borderless_2_pants" y="12" x="83"/>
    <use href="#product_borderless_2_item" y="5" x="126"/>
    <use href="#product_borderless_2_shirt" y="12" x="137"/>
    <use href="#product_borderless_2_item" y="5" x="187"/>
    <use href="#product_borderless_2_watch" y="12" x="205"/>
</svg>

```

## File: static\src\img\snippets_options\product_card_group.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_card_group_shirt" transform="scale(.5)">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_card_group_watch" transform="scale(.5)">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_card_group_pants" transform="scale(.5)">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_card_group_item">
            <rect width="115" height="30" fill="#fff" stroke="#c7c7c7" stroke-width="1"/>
            <rect width="20" height="20" fill="#9ccde4" x="90" y="5"/>
            <rect width="67" height="2" x="5" y="7" fill="#222"/>
            <rect width="52" height="1" x="5" y="12" fill="#666"/>
            <rect width="37" height="1" x="5" y="15" fill="#666"/>
            <rect width="15" height="2" x="11" y="21" fill="#3aadaa"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(4 35)" fill="#3aadaa"/>
        </g>
    </defs> 
    <use href="#product_card_group_item" y="5" x="5"/>
    <use href="#product_card_group_shirt" y="14" x="99"/>
    <use href="#product_card_group_item" y="5" x="120"/>
    <use href="#product_card_group_watch" y="14" x="217"/>
    <use href="#product_card_group_item" y="35" x="5"/>
    <use href="#product_card_group_pants" y="44" x="102"/>
    <use href="#product_card_group_item" y="35" x="120"/>
    <use href="#product_card_group_shirt" y="44" x="214"/>
</svg>

```

## File: static\src\img\snippets_options\product_centered.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_centered_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_centered_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_centered_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_centered_item">
            <rect width="48" height="43" y="17" fill="#fff"/>
            <rect width="26" height="2" x="11" y="35" fill="#c7c7c7"/>
            <rect width="15" height="2" x="18" y="42" fill="#595959"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(10 56)" fill="#595959"/>
            <rect width="34" height="7" rx="2" x="7" y="49" fill="#3aadaa"/>
            <rect width="16" height="1" x="16" y="52" fill="#fff"/>
            <rect width="34" height="32" fill="#9ccde4" x="7"/>
        </g>
    </defs> 
    <use href="#product_centered_item" y="5" x="5"/>
    <use href="#product_centered_shirt" y="9" x="16"/>
    <use href="#product_centered_item" y="5" x="66"/>
    <use href="#product_centered_pants" y="9" x="83"/>
    <use href="#product_centered_item" y="5" x="126"/>
    <use href="#product_centered_shirt" y="9" x="137"/>
    <use href="#product_centered_item" y="5" x="187"/>
    <use href="#product_centered_watch" y="9" x="205"/>
</svg>

```

## File: static\src\img\snippets_options\product_horizontal_card.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_horizontal_card_shirt" transform="scale(.5)">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_horizontal_card_watch" transform="scale(.5)">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_horizontal_card_pants" transform="scale(.5)">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_horizontal_card_item">
            <rect width="70" height="26" rx="3" fill="#fff"/>
            <rect width="36" height="2" x="29" y="5" fill="#222"/>
            <rect width="15" height="2" x="34" y="16" fill="#595959"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(27.5 30)" fill="#595959"/>
            <rect width="7" height="7" rx="2" x="58" y="14" fill="#3aadaa"/>
            <rect width="20" height="20" fill="#9ccde4" x="5" y="3"/>
        </g>
    </defs> 
    <use href="#product_horizontal_card_item" y="5" x="5"/>
    <use href="#product_horizontal_card_shirt" y="12" x="13"/>
    <use href="#product_horizontal_card_item" y="5" x="85"/>
    <use href="#product_horizontal_card_watch" y="12" x="97"/>
    <use href="#product_horizontal_card_item" y="5" x="165"/>
    <use href="#product_horizontal_card_pants" y="12" x="176"/>
    <use href="#product_horizontal_card_item" y="39" x="5"/>
    <use href="#product_horizontal_card_pants" y="46" x="16"/>
    <use href="#product_horizontal_card_item" y="39" x="85"/>
    <use href="#product_horizontal_card_shirt" y="46" x="94"/>
    <use href="#product_horizontal_card_item" y="39" x="165"/>
    <use href="#product_horizontal_card_watch" y="46" x="177"/>
</svg>

```

## File: static\src\img\snippets_options\product_horizontal_card_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_horizontal_card_2_item">
            <rect width="110" height="26" fill="#9ccde4"/>
            <g transform="translate(10 4) scale(.75)">
                <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
                <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            </g>
            <g transform="translate(51 4) scale(.75)">
                <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            </g>
            <g transform="translate(83 4) scale(.75)">
                <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
                <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            </g>
            <rect width="110" height="26" fill="#000" opacity="0.5"/>
            <rect width="67" height="2" x="5" y="5" fill="#fff"/>
            <rect width="52" height="1" x="5" y="10" fill="#fff"/>
            <rect width="15" height="2" x="11" y="17" fill="#fff"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(4.5 31)" fill="#fff"/>
            <rect width="28" height="7" rx="2" x="77" y="14" fill="#3aadaa"/>
            <rect width="12" height="1" x="85" y="17" fill="#fff"/>
        </g>
    </defs> 
    <use href="#product_horizontal_card_2_item" y="5" x="5"/>
    <use href="#product_horizontal_card_2_item" y="5" x="125"/>
    <use href="#product_horizontal_card_2_item" y="39" x="5"/>
    <use href="#product_horizontal_card_2_item" y="39" x="125"/>
</svg>

```

## File: static\src\img\snippets_options\product_image_only.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_image_only_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_only_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_only_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <rect id="product_image_only_item" width="50" height="60" fill="#9ccde4"/>
    </defs> 
    <use href="#product_image_only_item" y="5" x="5"/>
    <use href="#product_image_only_item" y="5" x="65"/>
    <use href="#product_image_only_item" y="5" x="125"/>
    <use href="#product_image_only_item" y="5" x="185"/>
    <use href="#product_image_only_shirt" y="23" x="17"/>
    <use href="#product_image_only_pants" y="23" x="83"/>
    <use href="#product_image_only_shirt" y="23" x="137"/>
    <use href="#product_image_only_watch" y="23" x="205"/>
</svg>

```

## File: static\src\img\snippets_options\product_image_with_name.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_image_with_name_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_with_name_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_with_name_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_with_name_item">
            <rect width="50" height="50" fill="#9ccde4"/>
            <rect width="40" height="2" x="5" y="58" fill="#fff"/>
        </g>
    </defs> 
    <use href="#product_image_with_name_item" y="5" x="5"/>
    <use href="#product_image_with_name_item" y="5" x="65"/>
    <use href="#product_image_with_name_item" y="5" x="125"/>
    <use href="#product_image_with_name_item" y="5" x="185"/>
    <use href="#product_image_with_name_shirt" y="18" x="17"/>
    <use href="#product_image_with_name_pants" y="18" x="83"/>
    <use href="#product_image_with_name_shirt" y="18" x="137"/>
    <use href="#product_image_with_name_watch" y="18" x="205"/>
</svg>

```

## File: static\src\img\snippets_options\product_image_with_price.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_image_with_price_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_with_price_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_with_price_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_image_with_price_item">
            <rect width="50" height="50" fill="#9ccde4"/>
            <rect width="15" height="2" x="30" y="58" fill="#fff"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(23.5 72)" fill="#fff"/>
        </g>
    </defs> 
    <use href="#product_image_with_price_item" y="5" x="5"/>
    <use href="#product_image_with_price_item" y="5" x="65"/>
    <use href="#product_image_with_price_item" y="5" x="125"/>
    <use href="#product_image_with_price_item" y="5" x="185"/>
    <use href="#product_image_with_price_shirt" y="18" x="17"/>
    <use href="#product_image_with_price_pants" y="18" x="83"/>
    <use href="#product_image_with_price_shirt" y="18" x="137"/>
    <use href="#product_image_with_price_watch" y="18" x="205"/>
</svg>

```

## File: static\src\img\snippets_options\product_view_detail.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <g id="product_view_detail_shirt">
            <path d="M12.9.4H5.417L.4,6.928l2.86,3.683h1.7V22.9h15.86V10.611h1.726L25.4,6.928,20.383.4Z" fill="#e89849"/>
            <path d="M4.954,10.612h-1.7L.4,6.928,5.417.4H20.383L25.4,6.928l-2.86,3.684H20.814" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M20.814,8.526V22.9H4.954V5.437" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M16.718,1.648a4.032,4.032,0,0,1-7.283.839" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="2.465" transform="translate(18.349 18.115)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line x1="6.068" transform="translate(14.746 19.742)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_view_detail_watch">
            <rect width="4.743" height="7.122" x="3.4" y="15.7" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <rect width="4.743" height="7.122" x="3.4" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M11.124,11.642A5.361,5.361,0,1,1,5.762,6.28,5.362,5.362,0,0,1,11.124,11.642Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="3.183" transform="translate(5.762 8.459)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <line y1="1.313" x2="1.961" transform="translate(5.762 10.329)" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_view_detail_pants">
            <path d="M13.848,22.9H8.884L7.123,7.094,5.361,22.9H.4V.4H13.848Z" fill="#e89849" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M2.231,4.648h2.46V8.157L2.566,9.215.4,8.077" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
            <path d="M13.81,4.648H9.52V8.157l2.125,1.058" fill="none" stroke="#89440b" stroke-linecap="round" stroke-linejoin="round" stroke-width="1"/>
        </g>
        <g id="product_view_detail_item">
            <rect width="70" height="60" fill="#fff"/>
            <rect width="15" height="2" x="11" y="52" fill="#595959"/>
            <path d="M1.116-11.744H.5a1.827,1.827,0,0,0,.486,1.192,1.818,1.818,0,0,0,1.152.5v.594h.4v-.594a2.2,2.2,0,0,0,.638-.145,1.559,1.559,0,0,0,.5-.315A1.432,1.432,0,0,0,4-10.983a1.57,1.57,0,0,0,.12-.63A1.207,1.207,0,0,0,4-12.186a1.393,1.393,0,0,0-.308-.4,1.571,1.571,0,0,0-.377-.25,1.975,1.975,0,0,0-.341-.127l-.435-.116V-14.86a.964.964,0,0,1,.812.841h.616a2.074,2.074,0,0,0-.493-1,1.5,1.5,0,0,0-.935-.435v-.486h-.4v.478a1.794,1.794,0,0,0-.583.138,1.48,1.48,0,0,0-.471.315,1.48,1.48,0,0,0-.315.471,1.508,1.508,0,0,0-.116.6,1.508,1.508,0,0,0,.087.544.929.929,0,0,0,.272.38,1.7,1.7,0,0,0,.464.268,4.95,4.95,0,0,0,.663.207v1.92a1.3,1.3,0,0,1-.721-.355A1.079,1.079,0,0,1,1.116-11.744Zm1.42,1.123V-12.44q.2.058.37.127a1.1,1.1,0,0,1,.3.17.762.762,0,0,1,.2.243.765.765,0,0,1,.072.351,1.07,1.07,0,0,1-.069.4.742.742,0,0,1-.2.283.861.861,0,0,1-.3.17A1.483,1.483,0,0,1,2.536-10.621Zm-.4-4.261v1.7A2.683,2.683,0,0,1,1.8-13.3a1.123,1.123,0,0,1-.265-.156.613.613,0,0,1-.174-.225.768.768,0,0,1-.062-.322.791.791,0,0,1,.243-.627A1.031,1.031,0,0,1,2.138-14.882Z" transform="translate(4.5 65)" fill="#595959"/>
            <rect width="60" height="2" x="5" y="34" fill="#222"/>
            <rect width="54" height="1" x="5" y="39" fill="#666"/>
            <rect width="37" height="1" x="5" y="42" fill="#666"/>
            <rect width="28" height="7" rx="2" x="37" y="50" fill="#3aadaa"/>
            <rect width="12" height="1" x="45" y="53" fill="#fff"/>
            <rect width="70" height="30" fill="#9ccde4"/>
        </g>
    </defs> 
    <use href="#product_view_detail_item" y="5" x="5"/>
    <use href="#product_view_detail_item" y="5" x="85"/>
    <use href="#product_view_detail_item" y="5" x="165"/>
    <use href="#product_view_detail_shirt" y="8" x="27"/>
    <use href="#product_view_detail_watch" y="8" x="114"/>
    <use href="#product_view_detail_pants" y="8" x="193"/>
</svg>

```

## File: static\src\img\snippets_thumbs\s_dynamic_products.svg

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

## File: static\src\img\snippet_options\image-width-100.svg

```svg
<svg width="24" height="8" viewBox="0 0 24 8" fill="none" xmlns="http://www.w3.org/2000/svg">
<path fill-rule="evenodd" clip-rule="evenodd" d="M23.3594 0V8H0.359375V0H23.3594ZM18.3594 1V3.5H15.3594V4.5H18.3594V7L21.3594 4L18.3594 1ZM5.35938 1L2.35938 4L5.35938 7V4.5H8.35938V3.5H5.35938V1Z" fill="white"/>
</svg>

```

## File: static\src\img\snippet_options\image-width-50.svg

```svg
<svg width="23" height="8" viewBox="0 0 23 8" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M21 0H12V8H21V0Z" fill="#CDCDCD"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M2 0V8H11V0H2Z" fill="white"/>
</svg>

```

## File: static\src\img\snippet_options\image-width-66.svg

```svg
<svg width="24" height="8" viewBox="0 0 24 8" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M21.3594 0H15.3594V8H21.3594V0Z" fill="#CDCDCD"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M2.35938 0V8H13.3594V0H2.35938Z" fill="white"/>
</svg>

```

## File: static\src\img\snippet_options\image-width-none.svg

```svg
<svg width="24" height="8" viewBox="0 0 24 8" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M2 0V8H21V0H2ZM14.36 5.56C14.45 5.65 14.5 5.75 14.5 5.87C14.5 5.99 14.45 6.1 14.36 6.18L13.69 6.8C13.6 6.89 13.49 6.93 13.35 6.93C13.21 6.93 13.11 6.89 13.01 6.8L11.56 5.46L10.11 6.8C10.02 6.89 9.91 6.93 9.77 6.93C9.63 6.93 9.53 6.89 9.43 6.8L8.76 6.18C8.67 6.09 8.62 5.99 8.62 5.87C8.62 5.75 8.67 5.64 8.76 5.56L10.21 4.22L8.76 2.88C8.67 2.79 8.62 2.69 8.62 2.57C8.62 2.45 8.67 2.34 8.76 2.26L9.43 1.64C9.52 1.55 9.63 1.51 9.77 1.51C9.91 1.51 10.01 1.55 10.11 1.64L11.56 2.98L13.01 1.64C13.1 1.55 13.21 1.51 13.35 1.51C13.49 1.51 13.59 1.55 13.69 1.64L14.36 2.26C14.45 2.35 14.5 2.45 14.5 2.57C14.5 2.69 14.45 2.8 14.36 2.88L12.91 4.22L14.36 5.56Z" fill="white"/>
</svg>

```

## File: static\src\js\payment_button.js

```javascript
/** @odoo-module **/

import paymentButton from '@payment/js/payment_button';

paymentButton.include({

    /**
     * Verify that the payment button is ready to be enabled.
     *
     * The conditions are that:
     * - a delivery carrier is selected and ready (the price is computed) if deliveries are enabled;
     * - the "Terms and Conditions" checkbox is ticked if it is present.
     *
     * @override from @payment/js/payment_button
     * @return {boolean}
     */
    _canSubmit() {
        return this._super(...arguments) && this._isCarrierReady() && this._isTCCheckboxReady();
    },

    /**
     * Check if the delivery carrier is selected and if its price is computed.
     *
     * @private
     * @return {boolean}
     */
    _isCarrierReady() {
        const carriers = document.querySelectorAll('.o_delivery_carrier_select');
        if (carriers.length === 0) { // No carrier is available.
            return true; // Ignore the check.
        }

        const checkedCarriers = document.querySelectorAll('input[name="delivery_type"]:checked');
        if (checkedCarriers.length === 0) { // No carrier is selected.
            return false; // Nothing else to check.
        }
        const carriersContainer = checkedCarriers[0].closest('.o_delivery_carrier_select');
        if (carriersContainer.querySelector('.o_wsale_delivery_carrier_error')) {
            // Rate shipment error.
            return false;
        }
        const isPickUpPointRequired = carriersContainer.querySelector('.o_show_pickup_locations');
        if (isPickUpPointRequired) {
            const address = carriersContainer.querySelector(
                '.o_order_location_address'
            ).innerText;
            return address !== '';  // A pickup point is required but not selected.
        }
        return true;
    },

    /**
     * Check if the "Terms and Conditions" checkbox is ticked, if present.
     *
     * @private
     * @return {boolean}
     */
    _isTCCheckboxReady() {
        const checkbox = document.querySelector('#website_sale_tc_checkbox');
        if (!checkbox) { // The checkbox is not present.
            return true;  // Ignore the check.
        }

        return checkbox.checked;
    },

});

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module **/

import PaymentForm from '@payment/js/payment_form';

PaymentForm.include({

     /**
      * Create an event listener for the payment button located outside the payment form.
      * @override
     */
     async start() {
         const submitButton = document.querySelector('[name="o_payment_submit_button"]');
         submitButton.addEventListener('click', ev => this._submitForm(ev));
         return await this._super(...arguments);
     }

});

```

## File: static\src\js\sale_variant_mixin.js

```javascript
/** @odoo-module **/

import { KeepLast } from "@web/core/utils/concurrency";
import { memoize, uniqueId } from "@web/core/utils/functions";
import { throttleForAnimation } from "@web/core/utils/timing";
import { insertThousandsSep } from "@web/core/utils/numbers";
import { _t } from "@web/core/l10n/translation";
import { localization } from "@web/core/l10n/localization";
import { jsonrpc } from "@web/core/network/rpc_service";

var VariantMixin = {
    events: {
        'change .css_attribute_color input': '_onChangeColorAttribute',
        'click .o_variant_pills': '_onChangePillsAttribute',
        'change .main_product:not(.in_cart) input.js_quantity': 'onChangeAddQuantity',
        'change [data-attribute_exclusions]': 'onChangeVariant'
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * When a variant is changed, this will check:
     * - If the selected combination is available or not
     * - The extra price if applicable
     * - The display name of the product ("Customizable desk (White, Steel)")
     * - The new total price
     * - The need of adding a "custom value" input
     *   If the custom value is the only available value
     *   (defined by its data 'is_single_and_custom'),
     *   the custom value will have it's own input & label
     *
     * 'change' events triggered by the user entered custom values are ignored since they
     * are not relevant
     *
     * @param {MouseEvent} ev
     */
    onChangeVariant: function (ev) {
        var $parent = $(ev.target).closest('.js_product');
        if (!$parent.data('uniqueId')) {
            $parent.data('uniqueId', uniqueId());
        }
        this._throttledGetCombinationInfo(this, $parent.data('uniqueId'))(ev);
    },
    /**
     * @see onChangeVariant
     *
     * @private
     * @param {Event} ev
     * @returns {Deferred}
     */
    _getCombinationInfo: function (ev) {
        if ($(ev.target).hasClass('variant_custom_value')) {
            return Promise.resolve();
        }

        const $parent = $(ev.target).closest('.js_product');
        if(!$parent.length){
            return Promise.resolve();
        }
        const combination = this.getSelectedVariantValues($parent);
        let parentCombination;

        if ($parent.hasClass('main_product')) {
            parentCombination = $parent.find('ul[data-attribute_exclusions]').data('attribute_exclusions').parent_combination;
            const $optProducts = $parent.parent().find(`[data-parent-unique-id='${$parent.data('uniqueId')}']`);

            for (const optionalProduct of $optProducts) {
                const $currentOptionalProduct = $(optionalProduct);
                const childCombination = this.getSelectedVariantValues($currentOptionalProduct);
                const productTemplateId = parseInt($currentOptionalProduct.find('.product_template_id').val());
                jsonrpc('/website_sale/get_combination_info', {
                    'product_template_id': productTemplateId,
                    'product_id': this._getProductId($currentOptionalProduct),
                    'combination': childCombination,
                    'add_qty': parseInt($currentOptionalProduct.find('input[name="add_qty"]').val()),
                    'parent_combination': combination,
                    'context': this.context,
                    ...this._getOptionalCombinationInfoParam($currentOptionalProduct),
                }).then((combinationData) => {
                    if (this._shouldIgnoreRpcResult()) {
                        return;
                    }
                    this._onChangeCombination(ev, $currentOptionalProduct, combinationData);
                    this._checkExclusions($currentOptionalProduct, childCombination, combinationData.parent_exclusions);
                });
            }
        } else {
            parentCombination = this.getSelectedVariantValues(
                $parent.parent().find('.js_product.in_cart.main_product')
            );
        }

        return jsonrpc('/website_sale/get_combination_info', {
            'product_template_id': parseInt($parent.find('.product_template_id').val()),
            'product_id': this._getProductId($parent),
            'combination': combination,
            'add_qty': parseInt($parent.find('input[name="add_qty"]').val()),
            'parent_combination': parentCombination,
            'context': this.context,
            ...this._getOptionalCombinationInfoParam($parent),
        }).then((combinationData) => {
            if (this._shouldIgnoreRpcResult()) {
                return;
            }
            this._onChangeCombination(ev, $parent, combinationData);
            this._checkExclusions($parent, combination, combinationData.parent_exclusions);
        });
    },

    /**
     * Hook to add optional info to the combination info call.
     *
     * @param {$.Element} $product
     */
    _getOptionalCombinationInfoParam($product) {
        return {};
    },

    /**
     * Will add the "custom value" input for this attribute value if
     * the attribute value is configured as "custom" (see product_attribute_value.is_custom)
     *
     * @private
     * @param {MouseEvent} ev
     */
    handleCustomValues: function ($target) {
        var $variantContainer;
        var $customInput = false;
        if ($target.is('input[type=radio]') && $target.is(':checked')) {
            $variantContainer = $target.closest('ul').closest('li');
            $customInput = $target;
        } else if ($target.is('select')) {
            $variantContainer = $target.closest('li');
            $customInput = $target
                .find('option[value="' + $target.val() + '"]');
        }

        if ($variantContainer) {
            if ($customInput && $customInput.data('is_custom') === 'True') {
                var attributeValueId = $customInput.data('value_id');
                var attributeValueName = $customInput.data('value_name');

                if ($variantContainer.find('.variant_custom_value').length === 0
                        || $variantContainer
                              .find('.variant_custom_value')
                              .data('custom_product_template_attribute_value_id') !== parseInt(attributeValueId)) {
                    $variantContainer.find('.variant_custom_value').remove();

                    const previousCustomValue = $customInput.attr("previous_custom_value");
                    var $input = $('<input>', {
                        type: 'text',
                        'data-custom_product_template_attribute_value_id': attributeValueId,
                        'data-attribute_value_name': attributeValueName,
                        class: 'variant_custom_value form-control mt-2'
                    });

                    $input.attr('placeholder', attributeValueName);
                    $input.addClass('custom_value_radio');
                    $variantContainer.append($input);
                    if (previousCustomValue) {
                        $input.val(previousCustomValue);
                    }
                }
            } else {
                $variantContainer.find('.variant_custom_value').remove();
            }
        }
    },

    /**
     * Hack to add and remove from cart with json
     *
     * @param {MouseEvent} ev
     */
    onClickAddCartJSON: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        var $input = $link.closest('.input-group').find("input");
        var min = parseFloat($input.data("min") || 0);
        var max = parseFloat($input.data("max") || Infinity);
        var previousQty = parseFloat($input.val() || 0, 10);
        var quantity = ($link.has(".fa-minus").length ? -1 : 1) + previousQty;
        var newQty = quantity > min ? (quantity < max ? quantity : max) : min;

        if (newQty !== previousQty) {
            $input.val(newQty).trigger('change');
        }
        return false;
    },

    /**
     * When the quantity is changed, we need to query the new price of the product.
     * Based on the price list, the price might change when quantity exceeds X
     *
     * @param {MouseEvent} ev
     */
    onChangeAddQuantity: function (ev) {
        var $parent;

        if ($(ev.currentTarget).closest('.oe_advanced_configurator_modal').length > 0){
            $parent = $(ev.currentTarget).closest('.oe_advanced_configurator_modal');
        } else if ($(ev.currentTarget).closest('form').length > 0){
            $parent = $(ev.currentTarget).closest('form');
        }  else {
            $parent = $(ev.currentTarget).closest('.o_product_configurator');
        }

        this.triggerVariantChange($parent);
    },

    /**
     * Triggers the price computation and other variant specific changes
     *
     * @param {$.Element} $container
     */
    triggerVariantChange: function ($container) {
        $container.find('ul[data-attribute_exclusions]').trigger('change');
        $container.find('input.js_variant_change:checked, select.js_variant_change').each(function () {
            VariantMixin.handleCustomValues($(this));
        });
    },

    /**
     * Will look for user custom attribute values
     * in the provided container
     *
     * @param {$.Element} $container
     * @returns {Array} array of custom values with the following format
     *   {integer} custom_product_template_attribute_value_id
     *   {string} attribute_value_name
     *   {string} custom_value
     */
    getCustomVariantValues: function ($container) {
        var variantCustomValues = [];
        $container.find('.variant_custom_value').each(function (){
            var $variantCustomValueInput = $(this);
            if ($variantCustomValueInput.length !== 0){
                variantCustomValues.push({
                    'custom_product_template_attribute_value_id': $variantCustomValueInput.data('custom_product_template_attribute_value_id'),
                    'attribute_value_name': $variantCustomValueInput.data('attribute_value_name'),
                    'custom_value': $variantCustomValueInput.val(),
                });
            }
        });

        return variantCustomValues;
    },

    /**
     * Will look for attribute values that do not create product variant
     * (see product_attribute.create_variant "dynamic")
     *
     * @param {$.Element} $container
     * @returns {Array} array of attribute values with the following format
     *   {integer} custom_product_template_attribute_value_id
     *   {string} attribute_value_name
     *   {integer} value
     *   {string} attribute_name
     *   {boolean} is_custom
     */
    getNoVariantAttributeValues: function ($container) {
        var noVariantAttributeValues = [];
        var variantsValuesSelectors = [
            'input.no_variant.js_variant_change:checked',
            'select.no_variant.js_variant_change'
        ];

        $container.find(variantsValuesSelectors.join(',')).each(function (){
            var $variantValueInput = $(this);
            var singleNoCustom = $variantValueInput.data('is_single') && !$variantValueInput.data('is_custom');

            if ($variantValueInput.is('select')){
                $variantValueInput = $variantValueInput.find('option[value=' + $variantValueInput.val() + ']');
            }

            if ($variantValueInput.length !== 0 && !singleNoCustom){
                noVariantAttributeValues.push({
                    'custom_product_template_attribute_value_id': $variantValueInput.data('value_id'),
                    'attribute_value_name': $variantValueInput.data('value_name'),
                    'value': $variantValueInput.val(),
                    'attribute_name': $variantValueInput.data('attribute_name'),
                    'is_custom': $variantValueInput.data('is_custom')
                });
            }
        });

        return noVariantAttributeValues;
    },

    /**
     * Will return the list of selected product.template.attribute.value ids
     * For the modal, the "main product"'s attribute values are stored in the
     * "unchanged_value_ids" data
     *
     * @param {$.Element} $container the container to look into
     */
    getSelectedVariantValues: function ($container) {
        var values = [];
        var unchangedValues = $container
            .find('div.oe_unchanged_value_ids')
            .data('unchanged_value_ids') || [];

        var variantsValuesSelectors = [
            'input.js_variant_change:checked',
            'select.js_variant_change'
        ];
        $container.find(variantsValuesSelectors.join(', ')).toArray().forEach((el) => {
            values.push(+$(el).val());
        });

        return values.concat(unchangedValues);
    },

    /**
     * Will return a promise:
     *
     * - If the product already exists, immediately resolves it with the product_id
     * - If the product does not exist yet ("dynamic" variant creation), this method will
     *   create the product first and then resolve the promise with the created product's id
     *
     * @param {$.Element} $container the container to look into
     * @param {integer} productId the product id
     * @param {integer} productTemplateId the corresponding product template id
     * @param {boolean} useAjax wether the rpc call should be done using jsonrpc or using _rpc
     * @returns {Promise} the promise that will be resolved with a {integer} productId
     */
    selectOrCreateProduct: function ($container, productId, productTemplateId, useAjax) {
        productId = parseInt(productId);
        productTemplateId = parseInt(productTemplateId);
        var productReady = Promise.resolve();
        if (productId) {
            productReady = Promise.resolve(productId);
        } else {
            var params = {
                product_template_id: productTemplateId,
                product_template_attribute_value_ids:
                    JSON.stringify(VariantMixin.getSelectedVariantValues($container)),
            };

            var route = '/sale/create_product_variant';
            if (useAjax) {
                productReady = jsonrpc(route, params);
            } else {
                productReady = this.rpc(route, params);
            }
        }

        return productReady;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Will disable attribute value's inputs based on combination exclusions
     * and will disable the "add" button if the selected combination
     * is not available
     *
     * This will check both the exclusions within the product itself and
     * the exclusions coming from the parent product (meaning that this product
     * is an option of the parent product)
     *
     * It will also check that the selected combination does not exactly
     * match a manually archived product
     *
     * @private
     * @param {$.Element} $parent the parent container to apply exclusions
     * @param {Array} combination the selected combination of product attribute values
     * @param {Array} parentExclusions the exclusions induced by the variant selection of the parent product
     * For example chair cannot have steel legs if the parent Desk doesn't have steel legs
     */
    _checkExclusions: function ($parent, combination, parentExclusions) {
        var self = this;
        var combinationData = $parent
            .find('ul[data-attribute_exclusions]')
            .data('attribute_exclusions');

        if (parentExclusions && combinationData.parent_exclusions) {
            combinationData.parent_exclusions = parentExclusions;
        }
        $parent
            .find('option, input, label, .o_variant_pills')
            .removeClass('css_not_available')
            .attr('title', function () { return $(this).data('value_name') || ''; })
            .data('excluded-by', '');

        // exclusion rules: array of ptav
        // for each of them, contains array with the other ptav they exclude
        if (combinationData.exclusions) {
            // browse all the currently selected attributes
            Object.values(combination).forEach((current_ptav) => {
                if (combinationData.exclusions.hasOwnProperty(current_ptav)) {
                    // for each exclusion of the current attribute:
                    Object.values(combinationData.exclusions[current_ptav]).forEach((excluded_ptav) => {
                        // disable the excluded input (even when not already selected)
                        // to give a visual feedback before click
                        self._disableInput(
                            $parent,
                            excluded_ptav,
                            current_ptav,
                            combinationData.mapped_attribute_names
                        );
                    });
                }
            });
        }
        // combination exclusions: array of array of ptav
        // for example a product with 3 variation and one specific variation is disabled (archived)
        //  requires the first 2 to be selected for the third to be disabled
        if (combinationData.archived_combinations) {
            combinationData.archived_combinations.forEach((excludedCombination) => {
                const ptavCommon = excludedCombination.filter((ptav) => combination.includes(ptav));
                if (
                    !!ptavCommon
                    && (combination.length === excludedCombination.length)
                    && (ptavCommon.length === combination.length)
                ) {
                    // Selected combination is archived, all attributes must be disabled from each other
                    combination.forEach((ptav) => {
                        combination.forEach((ptavOther) => {
                            if (ptav === ptavOther) {
                                return;
                            }
                            self._disableInput(
                                $parent,
                                ptav,
                                ptavOther,
                                combinationData.mapped_attribute_names,
                            );
                        })
                    })
                } else if (
                    !!ptavCommon
                    && (combination.length === excludedCombination.length)
                    && (ptavCommon.length === (combination.length - 1))
                ) {
                    // In this case we only need to disable the remaining ptav
                    const disabledPtav = excludedCombination.find((ptav) => !combination.includes(ptav));
                    excludedCombination.forEach((ptav) => {
                        if (ptav === disabledPtav) {
                            return;
                        }
                        self._disableInput(
                            $parent,
                            disabledPtav,
                            ptav,
                            combinationData.mapped_attribute_names,
                        )
                    });
                }
            });
        }

        // parent exclusions (tell which attributes are excluded from parent)
        for (const [excluded_by, exclusions] of Object.entries(
            combinationData.parent_exclusions || {}
        )) {
            // check that the selected combination is in the parent exclusions
            exclusions.forEach((ptav) => {
                // disable the excluded input (even when not already selected)
                // to give a visual feedback before click
                self._disableInput(
                    $parent,
                    ptav,
                    excluded_by,
                    combinationData.mapped_attribute_names,
                    combinationData.parent_product_name
                );
            });
        }
    },
    /**
     * Extracted to a method to be extendable by other modules
     *
     * @param {$.Element} $parent
     */
    _getProductId: function ($parent) {
        return parseInt($parent.find('.product_id').val());
    },
    /**
     * Will disable the input/option that refers to the passed attributeValueId.
     * This is used for showing the user that some combinations are not available.
     *
     * It will also display a message explaining why the input is not selectable.
     * Based on the "excludedBy" and the "productName" params.
     * e.g: Not available with Color: Black
     *
     * @private
     * @param {$.Element} $parent
     * @param {integer} attributeValueId
     * @param {integer} excludedBy The attribute value that excludes this input
     * @param {Object} attributeNames A dict containing all the names of the attribute values
     *   to show a human readable message explaining why the input is disabled.
     * @param {string} [productName] The parent product. If provided, it will be appended before
     *   the name of the attribute value that excludes this input
     *   e.g: Not available with Customizable Desk (Color: Black)
     */
    _disableInput: function ($parent, attributeValueId, excludedBy, attributeNames, productName) {
        var $input = $parent
            .find('option[value=' + attributeValueId + '], input[value=' + attributeValueId + ']');
        $input.addClass('css_not_available');
        $input.closest('label').addClass('css_not_available');
        $input.closest('.o_variant_pills').addClass('css_not_available');

        if (excludedBy && attributeNames) {
            var $target = $input.is('option') ? $input : $input.closest('label').add($input);
            var excludedByData = [];
            if ($target.data('excluded-by')) {
                excludedByData = JSON.parse($target.data('excluded-by'));
            }

            var excludedByName = attributeNames[excludedBy];
            if (productName) {
                excludedByName = productName + ' (' + excludedByName + ')';
            }
            excludedByData.push(excludedByName);

            $target.attr('title', _t('Not available with %s', excludedByData.join(', ')));
            $target.data('excluded-by', JSON.stringify(excludedByData));
        }
    },
    /**
     * @see onChangeVariant
     *
     * @private
     * @param {MouseEvent} ev
     * @param {$.Element} $parent
     * @param {Array} combination
     */
    _onChangeCombination: function (ev, $parent, combination) {
        var self = this;
        var $price = $parent.find(".oe_price:first .oe_currency_value");
        var $default_price = $parent.find(".oe_default_price:first .oe_currency_value");
        var $optional_price = $parent.find(".oe_optional:first .oe_currency_value");
        $price.text(self._priceToStr(combination.price));
        $default_price.text(self._priceToStr(combination.list_price));

        var isCombinationPossible = true;
        if (typeof combination.is_combination_possible !== "undefined") {
            isCombinationPossible = combination.is_combination_possible;
        }
        this._toggleDisable($parent, isCombinationPossible);

        if (combination.has_discounted_price && !combination.compare_list_price) {
            $default_price
                .closest('.oe_website_sale')
                .addClass("discount");
            $optional_price
                .closest('.oe_optional')
                .removeClass('d-none')
                .css('text-decoration', 'line-through');
            $default_price.parent().removeClass('d-none');
        } else {
            $default_price
                .closest('.oe_website_sale')
                .removeClass("discount");
            $optional_price.closest('.oe_optional').addClass('d-none');
            $default_price.parent().addClass('d-none');
        }

        var rootComponentSelectors = [
            'tr.js_product',
            '.oe_website_sale',
            '.o_product_configurator'
        ];

        // update images only when changing product
        // or when either ids are 'false', meaning dynamic products.
        // Dynamic products don't have images BUT they may have invalid
        // combinations that need to disable the image.
        if (!combination.product_id ||
            !this.last_product_id ||
            combination.product_id !== this.last_product_id) {
            this.last_product_id = combination.product_id;
            self._updateProductImage(
                $parent.closest(rootComponentSelectors.join(', ')),
                combination.display_image,
                combination.product_id,
                combination.product_template_id,
                combination.carousel,
                isCombinationPossible
            );
        }

        $parent
            .find('.product_id')
            .first()
            .val(combination.product_id || 0)
            .trigger('change');

        $parent
            .find('.product_display_name')
            .first()
            .text(combination.display_name);

        $parent
            .find('.js_raw_price')
            .first()
            .text(combination.price)
            .trigger('change');

        $parent
            .find('.o_product_tags')
            .first()
            .html(combination.product_tags);

        this.handleCustomValues($(ev.target));
    },

    /**
     * returns the formatted price
     *
     * @private
     * @param {float} price
     */
    _priceToStr: function (price) {
        var precision = 2;

        if ($('.decimal_precision').length) {
            precision = parseInt($('.decimal_precision').last().data('precision'));
        }
        var formatted = price.toFixed(precision).split(".");
        const { thousandsSep, decimalPoint, grouping } = localization;
        formatted[0] = insertThousandsSep(formatted[0], thousandsSep, grouping);
        return formatted.join(decimalPoint);
    },
    /**
     * Returns a throttled `_getCombinationInfo` with a leading and a trailing
     * call, which is memoized per `uniqueId`, and for which previous results
     * are dropped.
     *
     * The uniqueId is needed because on the configurator modal there might be
     * multiple elements triggering the rpc at the same time, and we need each
     * individual product rpc to be executed, but only once per individual
     * product.
     *
     * The leading execution is to keep good reactivity on the first call, for
     * a better user experience. The trailing is because ultimately only the
     * information about the last selected combination is useful. All
     * intermediary rpc can be ignored and are therefore best not done at all.
     *
     * The keepLast is to make sure we only consider the result of the last call, when several
     * (asynchronous) calls are done in parallel.
     *
     * @private
     * @param {string} uniqueId
     * @returns {function}
     */
    _throttledGetCombinationInfo: memoize(function (self, uniqueId) {
        const keepLast = new KeepLast();
        var _getCombinationInfo = throttleForAnimation(self._getCombinationInfo.bind(self));
        return (ev, params) => keepLast.add(_getCombinationInfo(ev, params));
    }),
    /**
     * Toggles the disabled class depending on the $parent element
     * and the possibility of the current combination.
     *
     * @private
     * @param {$.Element} $parent
     * @param {boolean} isCombinationPossible
     */
    _toggleDisable: function ($parent, isCombinationPossible) {
        $parent.toggleClass('css_not_available', !isCombinationPossible);
        if ($parent.hasClass('in_cart')) {
            const primaryButton = $parent.parents('.modal-content').find('.modal-footer .btn-primary');
            primaryButton.prop('disabled', !isCombinationPossible);
            primaryButton.toggleClass('disabled', !isCombinationPossible);
        }
    },
    /**
     * Updates the product image.
     * This will use the productId if available or will fallback to the productTemplateId.
     *
     * @private
     * @param {$.Element} $productContainer
     * @param {boolean} displayImage will hide the image if true. It will use the 'invisible' class
     *   instead of d-none to prevent layout change
     * @param {integer} product_id
     * @param {integer} productTemplateId
     */
    _updateProductImage: function ($productContainer, displayImage, productId, productTemplateId) {
        var model = productId ? 'product.product' : 'product.template';
        var modelId = productId || productTemplateId;
        var imageUrl = '/web/image/{0}/{1}/' + (this._productImageField ? this._productImageField : 'image_1024');
        var imageSrc = imageUrl
            .replace("{0}", model)
            .replace("{1}", modelId);

        var imagesSelectors = [
            'span[data-oe-model^="product."][data-oe-type="image"] img:first',
            'img.product_detail_img',
            'span.variant_image img',
            'img.variant_image',
        ];

        var $img = $productContainer.find(imagesSelectors.join(', '));

        if (displayImage) {
            $img.removeClass('invisible').attr('src', imageSrc);
        } else {
            $img.addClass('invisible');
        }
    },

    /**
     * Highlight selected color
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onChangeColorAttribute: function (ev) {
        var $parent = $(ev.target).closest('.js_product');
        $parent.find('.css_attribute_color')
            .removeClass("active")
            .filter(':has(input:checked)')
            .addClass("active");
    },

    _onChangePillsAttribute: function (ev) {
        const radio = ev.target.closest('.o_variant_pills').querySelector("input");
        radio.click();  // Trigger onChangeVariant.
        var $parent = $(ev.target).closest('.js_product');
        $parent.find('.o_variant_pills')
            .removeClass("active")
            .filter(':has(input:checked)')
            .addClass("active");
    },

    /**
     * Return true if the current object has been destroyed.
     * This function has been added as a fix to know if the result of a rpc
     * should be handled.
     *
     * @private
     */
    _shouldIgnoreRpcResult() {
        return (typeof this.isDestroyed === "function" && this.isDestroyed());
    },

    /**
     * Extension point for website_sale
     *
     * @private
     * @param {string} uri The uri to adapt
     */
    _getUri: function (uri) {
        return uri;
    }
};

export default VariantMixin;

```

## File: static\src\js\terms_and_conditions_checkbox.js

```javascript
/** @odoo-module **/

import { Component } from '@odoo/owl';
import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.TermsAndConditionsCheckbox = publicWidget.Widget.extend({
        selector: 'div[name="website_sale_terms_and_conditions_checkbox"]',
        events: {
            'change #website_sale_tc_checkbox': '_onClickTCCheckbox',
        },

        async start() {
            this.checkbox = this.el.querySelector('#website_sale_tc_checkbox');
            return this._super(...arguments);
        },

        /**
         * Enable/disabled the payment button when the "Terms and Conditions" checkbox is
         * checked/unchecked.
         *
         * @private
         * @return {void}
         */
        _onClickTCCheckbox() {
            if (this.checkbox.checked) {
                Component.env.bus.trigger('enablePaymentButton');
            } else {
                Component.env.bus.trigger('disablePaymentButton');
            }
        },

});

export default publicWidget.registry.TermsAndConditionsCheckbox;

```

## File: static\src\js\variant_mixin.js

```javascript
/** @odoo-module **/

import VariantMixin from "@website_sale/js/sale_variant_mixin";

const originalOnChangeCombination = VariantMixin._onChangeCombination;
VariantMixin._onChangeCombination = function (ev, $parent, combination) {
    const $pricePerUom = $parent.find(".o_base_unit_price:first .oe_currency_value");
    if ($pricePerUom) {
        if (combination.is_combination_possible !== false && combination.base_unit_price != 0) {
            $pricePerUom.parents(".o_base_unit_price_wrapper").removeClass("d-none");
            $pricePerUom.text(this._priceToStr(combination.base_unit_price));
            $parent.find(".oe_custom_base_unit:first").text(combination.base_unit_name);
        } else {
            $pricePerUom.parents(".o_base_unit_price_wrapper").addClass("d-none");
        }
    }

    // Triggers a new JS event with the correct payload, which is then handled
    // by the google analytics tracking code.
    // Indeed, every time another variant is selected, a new view_item event
    // needs to be tracked by google analytics.
    if ('product_tracking_info' in combination) {
        const $product = $('#product_detail');
        $product.data('product-tracking-info', combination['product_tracking_info']);
        $product.trigger('view_item_event', combination['product_tracking_info']);
    }
    const addToCart = $parent.find('#add_to_cart_wrap');
    const contactUsButton = $parent.find('#contact_us_wrapper');
    const productPrice = $parent.find('.product_price');
    const quantity = $parent.find('.css_quantity');
    const product_unavailable = $parent.find('#product_unavailable');
    if (combination.prevent_zero_price_sale) {
        productPrice.removeClass('d-inline-block').addClass('d-none');
        quantity.removeClass('d-inline-flex').addClass('d-none');
        addToCart.removeClass('d-inline-flex').addClass('d-none');
        contactUsButton.removeClass('d-none').addClass('d-flex');
        product_unavailable.removeClass('d-none').addClass('d-flex')
    } else {
        productPrice.removeClass('d-none').addClass('d-inline-block');
        quantity.removeClass('d-none').addClass('d-inline-flex');
        addToCart.removeClass('d-none').addClass('d-inline-flex');
        contactUsButton.removeClass('d-flex').addClass('d-none');
        product_unavailable.removeClass('d-flex').addClass('d-none')
    }
    originalOnChangeCombination.apply(this, [ev, $parent, combination]);
};

const originalToggleDisable = VariantMixin._toggleDisable;
/**
 * Toggles the disabled class depending on the $parent element
 * and the possibility of the current combination. This override
 * allows us to disable the secondary button in the website
 * sale product configuration modal.
 *
 * @private
 * @param {$.Element} $parent
 * @param {boolean} isCombinationPossible
 */
VariantMixin._toggleDisable = function ($parent, isCombinationPossible) {
    if ($parent.hasClass('in_cart')) {
        const secondaryButton = $parent.parents('.modal-content').find('.modal-footer .btn-secondary');
        secondaryButton.prop('disabled', !isCombinationPossible);
        secondaryButton.toggleClass('disabled', !isCombinationPossible);
    }
    originalToggleDisable.apply(this, [$parent, isCombinationPossible]);
};

export default VariantMixin;

```

## File: static\src\js\website_sale.editor.js

```javascript
/** @odoo-module **/

import options from "@web_editor/js/editor/snippets.options";
import { MediaDialog } from "@web_editor/components/media_dialog/media_dialog";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import "@website/js/editor/snippets.options";
import { renderToElement } from "@web/core/utils/render";
import { useChildSubEnv } from "@odoo/owl";
import weUtils from '@web_editor/js/common/utils';

options.registry.WebsiteSaleGridLayout = options.Class.extend({
    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    start: function () {
        this.ppg = parseInt(this.$target.closest('[data-ppg]').data('ppg'));
        this.ppr = parseInt(this.$target.closest('[data-ppr]').data('ppr'));
        this.default_sort = this.$target.closest('[data-default-sort]').data('default-sort');
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
        return this.rpc('/shop/config/website', { 'shop_ppg': this.ppg });
    },
    /**
     * @see this.selectClass for params
     */
    setPpr: function (previewMode, widgetValue, params) {
        this.ppr = parseInt(widgetValue);
        return this.rpc('/shop/config/website', { 'shop_ppr': this.ppr });
    },
    /**
     * @see this.selectClass for params
     */
    setDefaultSort: function (previewMode, widgetValue, params) {
        this.default_sort = widgetValue;
        return this.rpc('/shop/config/website', { 'shop_default_sort': this.default_sort });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async updateUIVisibility() {
        await this._super(...arguments);
        const pprSelector = this.el.querySelector('.o_wsale_ppr_submenu.d-none');
        this.el.querySelector('.o_wsale_ppr_by').classList.toggle('d-none', pprSelector);
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
            case 'setDefaultSort': {
                return this.default_sort;
            }
        }
        return this._super(...arguments);
    },
});

options.registry.WebsiteSaleProductsItem = options.Class.extend({
    events: Object.assign({}, options.Class.prototype.events || {}, {
        'mouseenter .o_wsale_soptions_menu_sizes table': '_onTableMouseEnter',
        'mouseleave .o_wsale_soptions_menu_sizes table': '_onTableMouseLeave',
        'mouseover .o_wsale_soptions_menu_sizes td': '_onTableItemMouseEnter',
        'click .o_wsale_soptions_menu_sizes td': '_onTableItemClick',
    }),

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    willStart: async function () {
        const _super = this._super.bind(this);
        this.ppr = this.$target.closest('[data-ppr]').data('ppr');
        this.productTemplateID = parseInt(this.$target.find('[data-oe-model="product.template"]').data('oe-id'));
        this.ribbons = await new Promise(resolve => this.trigger_up('get_ribbons', {callback: resolve}));
        this.$ribbon = this.$target.find('.o_ribbon');
        return _super(...arguments);
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
        this.ribbonEditMode = false;
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async selectStyle(previewMode, widgetValue, params) {
        const proms = [this._super(...arguments)];
        if (params.cssProperty === 'background-color' && params.colorNames.includes(widgetValue)) {
            // Reset text-color when choosing a background-color class, so it uses the automatic text-color of the class.
            proms.push(this.selectStyle(previewMode, '', {cssProperty: 'color'}));
        }
        await Promise.all(proms);
        if (!previewMode) {
            await this._saveRibbon();
        }
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
        if (!previewMode) {
            this.ribbonEditMode = false;
        }
        await this._setRibbon(widgetValue);
    },
    /**
     * @see this.selectClass for params
     */
    editRibbon(previewMode, widgetValue, params) {
        this.ribbonEditMode = !this.ribbonEditMode;
    },
    /**
     * @see this.selectClass for params
     */
    async createRibbon(previewMode, widgetValue, params) {
        await this._setRibbon(false);
        this.$ribbon.text(_t('Badge Text'));
        this.$ribbon.addClass('o_ribbon_left');
        this.ribbonEditMode = true;
        await this._saveRibbon(true);
    },
    /**
     * @see this.selectClass for params
     */
    async deleteRibbon(previewMode, widgetValue, params) {
        const save = await new Promise(resolve => {
            this.dialog.add(ConfirmationDialog, {
                body: _t('Are you sure you want to delete this badge?'),
                confirm: () => resolve(true),
                cancel: () => resolve(false),
            });
        });
        if (!save) {
            return;
        }
        const {ribbonId} = this.$target[0].dataset;
        this.trigger_up('delete_ribbon', {id: ribbonId});
        this.ribbons = await new Promise(resolve => this.trigger_up('get_ribbons', {callback: resolve}));
        this.rerender = true;
        await this._setRibbon(ribbonId);
        this.ribbonEditMode = false;
    },
    /**
     * @see this.selectClass for params
     */
    async setRibbonHtml(previewMode, widgetValue, params) {
        this.$ribbon.html(widgetValue);
        if (!previewMode) {
            await this._saveRibbon();
        }
    },
    /**
     * @see this.selectClass for params
     */
    async setRibbonMode(previewMode, widgetValue, params) {
        this.$ribbon[0].className = this.$ribbon[0].className.replace(/o_(ribbon|tag)_(left|right)/, `o_${widgetValue}_$2`);
        await this._saveRibbon();
    },
    /**
     * @see this.selectClass for params
     */
    async setRibbonPosition(previewMode, widgetValue, params) {
        this.$ribbon[0].className = this.$ribbon[0].className.replace(/o_(ribbon|tag)_(left|right)/, `o_$1_${widgetValue}`);
        await this._saveRibbon();
    },
    /**
     * @see this.selectClass for params
     */
    changeSequence: function (previewMode, widgetValue, params) {
        // TODO this should be awaited
        this.rpc('/shop/config/product', {
            product_id: this.productTemplateID,
            sequence: widgetValue,
        }).then(() => this._reloadEditable());
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
        // TODO: update this once updateUIVisibility can be used to compute visibility
        // of arbitrary DOM elements and not just widgets.
        await this._super(...arguments);
        this.$el.find('[data-name="ribbon_customize_opt"]').toggleClass('d-none', !this.ribbonEditMode);
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
        const classes = this.$ribbon[0].className;
        this.$ribbon[0].className = '';
        const defaultTextColor = window.getComputedStyle(this.$ribbon[0]).color;
        this.$ribbon[0].className = classes;
        Object.values(this.ribbons).forEach(ribbon => {
            const colorClasses = ribbon.html_class
                .split(' ')
                .filter(className => !/^o_(ribbon|tag)_(left|right)$/.test(className))
                .join(' ');
            $select.append(renderToElement('website_sale.ribbonSelectItem', {
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
     * @override
     */
    async _computeWidgetVisibility(widgetName, params) {
        if (widgetName === 'create_ribbon_opt') {
            return !this.ribbonEditMode;
        }
        return this._super(...arguments);
    },
    /**
     * Saves the ribbons.
     *
     * @private
     * @param {Boolean} [isNewRibbon=false]
     */
    async _saveRibbon(isNewRibbon = false) {
        const text = this.$ribbon.html().trim();
        const ribbon = {
            'html': text,
            'bg_color': this.$ribbon[0].style.backgroundColor,
            'text_color': this.$ribbon[0].style.color,
            'html_class': this.$ribbon.attr('class').split(' ').filter(c => !['o_ribbon'].includes(c)).join(' '),
        };
        ribbon.id = isNewRibbon ? Date.now() : parseInt(this.$target.closest('.oe_product')[0].dataset.ribbonId);
        this.trigger_up('set_ribbon', {ribbon: ribbon});
        this.ribbons = await new Promise(resolve => this.trigger_up('get_ribbons', {callback: resolve}));
        this.rerender = true;
        await this._setRibbon(ribbon.id);
    },
    /**
     * Sets the ribbon.
     *
     * @private
     * @param {integer|false} ribbonId
     */
    async _setRibbon(ribbonId) {
        this.$target[0].dataset.ribbonId = ribbonId;
        this.trigger_up('set_product_ribbon', {
            templateId: this.productTemplateID,
            ribbonId: ribbonId || false,
        });
        const ribbon = this.ribbons[ribbonId] || {html: '', bg_color: '', text_color: '', html_class: ''};
        // This option also manages other products' ribbon, therefore we need a
        // way to access all of them at once. With the content being in an iframe,
        // this is the simplest way.
        const $editableDocument = $(this.$target[0].ownerDocument.body);
        const $ribbons = $editableDocument.find(`[data-ribbon-id="${ribbonId}"] .o_ribbon`);
        $ribbons.empty().append(ribbon.html);
        let htmlClasses;
        this.trigger_up('get_ribbon_classes', {callback: classes => htmlClasses = classes});
        $ribbons.removeClass(htmlClasses);

        $ribbons.addClass(ribbon.html_class || '');
        $ribbons.css('background-color', ribbon.bg_color || '');
        $ribbons.css('color', ribbon.text_color || '');

        if (!this.ribbons[ribbonId]) {
            $editableDocument.find(`[data-ribbon-id="${ribbonId}"]`).each((index, product) => delete product.dataset.ribbonId);
        }

        // The ribbon does not have a savable parent, so we need to trigger the
        // saving process manually by flagging the ribbon as dirty.
        this.$ribbon.addClass('o_dirty');
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
        var y = $td.parent().index() + 1
        // TODO this should be awaited somehow
        this.rpc('/shop/config/product', {
            product_id: this.productTemplateID,
            x: x,
            y: y,
        }).then(() => this._reloadEditable());
    },
    _reloadEditable() {
        return this.trigger_up('request_save', {reload: true, optionSelector: `.oe_product:has(span[data-oe-id=${this.productTemplateID}])`});
    }
});

// Small override of the MediaDialog to retrieve the attachment ids instead of img elements
class AttachmentMediaDialog extends MediaDialog {
    setup() {
        super.setup();
        useChildSubEnv({ addFieldImage: true });
    }
    /**
     * @override
     */
    async save() {
        await super.save();
        const selectedMedia = this.selectedMedia[this.state.activeTab];
        if (selectedMedia.length) {
            await this.props.extraImageSave(selectedMedia);
        }
        this.props.close();
    }
}

options.registry.WebsiteSaleProductPage = options.Class.extend({
    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
        this.orm = this.bindService("orm");
        this.notification = this.bindService("notification");
    },

    /**
     * @override
     */
    async willStart() {
        let productProduct = this.$target[0].querySelector('[data-oe-model="product.product"]');
        let productTemplate = this.$target[0].querySelector('[data-oe-model="product.template"]');
        this.productProductID = productProduct ? productProduct.dataset.oeId : null;
        this.productTemplateID = productTemplate ? productTemplate.dataset.oeId : null;
        this.mode = "product.template";
        if (this.productProductID) {
            this.mode = "product.product"
        }

        // Different targets
        this.productDetailMain = this.$target[0].querySelector('#product_detail_main');
        this.productPageCarousel = this.$target[0].querySelector("#o-carousel-product");
        this.productPageGrid = this.$target[0].querySelector("#o-grid-product");
        return this._super(...arguments);
    },

    _updateWebsiteConfig(params) {
        // TODO: Remove the request_save in master, it's already done by the
        // data-page-options set to true in the template.
        return this.rpc('/shop/config/website', params).then(() => this.trigger_up('request_save', {reload: true, optionSelector: this.data.selector}));
    },

    _getZoomOptionData() {
        return this._userValueWidgets.find(widget => {
            return widget.options && widget.options.dataAttributes && widget.options.dataAttributes.name === "o_wsale_zoom_mode";
        });
    },

    /**
     * @override
     */
    async setImageWidth(previewMode, widgetValue, params) {
        const zoomOption = this._getZoomOptionData();
        const updateWidth = this._updateWebsiteConfig.bind(this, { product_page_image_width: widgetValue });
        if (!zoomOption || widgetValue !== "100_pc") {
            await updateWidth();
        } else {
            const defaultZoomOption = "website_sale.product_picture_magnify_click";
            await this._customizeWebsiteData(defaultZoomOption, { possibleValues: zoomOption._methodsParams.optionsPossibleValues["customizeWebsiteViews"] }, true);
            await updateWidth();
        }
    },

    /**
     * @override
     */
    async setImageLayout(previewMode, widgetValue, params) {
        const zoomOption = this._getZoomOptionData();
        const updateLayout = this._updateWebsiteConfig.bind(this, { product_page_image_layout: widgetValue });
        if (!zoomOption) {
            await updateLayout();
        } else {
            const imageWidthOption = this.productDetailMain.dataset.image_width;
            let defaultZoomOption = widgetValue === "grid" ? "website_sale.product_picture_magnify_click" : "website_sale.product_picture_magnify_hover";
            if (imageWidthOption === "100_pc" && defaultZoomOption === "website_sale.product_picture_magnify_hover") {
                defaultZoomOption = "website_sale.product_picture_magnify_click";
            }
            await this._customizeWebsiteData(defaultZoomOption, { possibleValues: zoomOption._methodsParams.optionsPossibleValues["customizeWebsiteViews"] }, true);
            await updateLayout();
        }
    },

    /**
     * Emulate click on the main image of the carousel.
     */
    replaceMainImage: function () {
        const image = this.productDetailMain.querySelector(`[data-oe-model="${this.mode}"][data-oe-field=image_1920] img`);
        image.dispatchEvent(new Event('dblclick', {bubbles: true}));
    },

    _getSelectedVariantValues($container) {
        const combination = $container.find('input.js_product_change:checked').data('combination');

        if (combination) {
            return combination;
        }
        const values = [];

        const variantsValuesSelectors = [
            'input.js_variant_change:checked',
            'select.js_variant_change'
        ];
        $container
            .find(variantsValuesSelectors.join(", "))
            .toArray()
            .forEach((el) => {
                values.push(+$(el).val());
            });

        return values;
    },

    /**
     * Prompts the user for images, then saves the new images.
     */
    addImages: function () {
        if(this.mode === 'product.template'){
            this.notification.add(
                'Pictures will be added to the main image. Use "Instant" attributes to set pictures on each variants',
                { type: 'info' }
            );
        }
        let extraImageEls;
        this.call("dialog", "add", AttachmentMediaDialog, {
            multiImages: true,
            onlyImages: true,
            // Kinda hack-ish but the regular save does not get the information we need
            save: async (imgEls) => {
                extraImageEls = imgEls;
            },
            extraImageSave: async (attachments) => {
                for (const index in attachments) {
                    const attachment = attachments[index];
                    if (attachment.mimetype.startsWith("image/")) {
                        if (["image/gif", "image/svg+xml"].includes(attachment.mimetype)) {
                            continue;
                        }
                        await this._convertAttachmentToWebp(attachment, extraImageEls[index]);
                    }
                }
                this.rpc(`/shop/product/extra-images`, {
                    images: attachments,
                    product_product_id: this.productProductID,
                    product_template_id: this.productTemplateID,
                    combination_ids: this._getSelectedVariantValues(this.$target.find('.js_add_cart_variants')),
                }).then(() => {
                    this.trigger_up('request_save', {reload: true, optionSelector: this.data.selector});
                });
            }
        });
    },

    async _convertAttachmentToWebp(attachment, imageEl) {
        // This method is widely adapted from onFileUploaded in ImageField.
        // Upon change, make sure to verify whether the same change needs
        // to be applied on both sides.
        if (await weUtils.isImageCorsProtected(imageEl)) {
            // The image is CORS protected; do not transform it into webp
            return;
        }
        // Generate alternate sizes and format for reports.
        const imgEl = document.createElement("img");
        imgEl.src = imageEl.src;
        await new Promise(resolve => imgEl.addEventListener("load", resolve));
        const originalSize = Math.max(imgEl.width, imgEl.height);
        const smallerSizes = [1024, 512, 256, 128].filter(size => size < originalSize);
        const extension = attachment.name.match(/\.(jpe?|pn)g$/i)?.[0] ?? ".jpeg";
        const webpName = attachment.name.replace(extension, ".webp");
        const format = extension.substr(1).toLowerCase().replace(/^jpg$/, 'jpeg');
        const mimetype = `image/${format}`;
        let referenceId = undefined;
        for (const size of [originalSize, ...smallerSizes]) {
            const ratio = size / originalSize;
            const canvas = document.createElement("canvas");
            canvas.width = imgEl.width * ratio;
            canvas.height = imgEl.height * ratio;
            const ctx = canvas.getContext("2d");
            ctx.fillStyle = 'transparent';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(imgEl, 0, 0, imgEl.width, imgEl.height, 0, 0, canvas.width, canvas.height);
            const [resizedId] = await this.orm.call("ir.attachment", "create_unique", [[{
                name: webpName,
                description: size === originalSize ? "" : `resize: ${size}`,
                datas: canvas.toDataURL("image/webp", 0.75).split(",")[1],
                res_id: referenceId,
                res_model: "ir.attachment",
                mimetype: "image/webp",
            }]]);
            if (size === originalSize) {
                attachment.original_id = attachment.id;
                attachment.id = resizedId;
                attachment.image_src = `/web/image/${resizedId}-autowebp/${attachment.name}`;
                attachment.mimetype = "image/webp";
            }
            referenceId = referenceId || resizedId; // Keep track of original.
            await this.orm.call("ir.attachment", "create_unique", [[{
                name: attachment.name,
                description: `format: ${format}`,
                datas: canvas.toDataURL(mimetype, 0.75).split(",")[1],
                res_id: resizedId,
                res_model: "ir.attachment",
                mimetype: mimetype,
            }]]);
        }
    },

    /**
     * Removes all extra-images from the product.
     */
    clearImages: function () {
        // TODO this should be awaited
        this.rpc(`/shop/product/clear-images`, {
            model: this.mode,
            product_product_id: this.productProductID,
            product_template_id: this.productTemplateID,
            combination_ids: this._getSelectedVariantValues(this.$target.find('.js_add_cart_variants')),
        }).then(() => {
            this.trigger_up('request_save', {reload: true, optionSelector: this.data.selector});
        });
    },

    /**
     * @override
     */
    setSpacing(previewMode, widgetValue, params) {
        const spacing = {
            0: 'none',
            1: 'small',
            2: 'medium',
            3: 'big',
        }[widgetValue];
        this.productPageGrid.dataset.image_spacing = spacing;
        // TODO: Remove the request_save in master, it's already done by the
        // data-page-options set to true in the template.
        return this.rpc('/shop/config/website', {
            'product_page_image_spacing': spacing,
        }).then(() => this.trigger_up('request_save', {reload: true, optionSelector: this.data.selector}));
    },

    setColumns(previewMode, widgetValue, params) {
        this.productPageGrid.dataset.grid_columns = widgetValue;
        // TODO: Remove the request_save in master, it's already done by the
        // data-page-options set to true in the template.
        return this.rpc('/shop/config/website', {
            'product_page_grid_columns': widgetValue,
        }).then(() => this.trigger_up('request_save', {reload: true, optionSelector: this.data.selector}));
    },

    /**
     * @override
     */
    async _computeWidgetState(methodName, params) {
        switch (methodName) {
            case 'setImageWidth':
                return this.productDetailMain.dataset.image_width;
            case 'setImageLayout':
                return this.productDetailMain.dataset.image_layout;
            case 'setSpacing':
                if (!this.productPageGrid) return 0;
                return {
                    'none': 0,
                    'small': 1,
                    'medium': 2,
                    'big': 3,
                }[this.productPageGrid.dataset.image_spacing];
            case 'setColumns':
                return this.productPageGrid && this.productPageGrid.dataset.grid_columns || 1;
        }
        return this._super(...arguments);
    },

    async _computeWidgetVisibility(widgetName, params) {
        const hasImages = this.productDetailMain.dataset.image_width != 'none';
        const isFullImage = this.productDetailMain.dataset.image_width == '100_pc';
        switch (widgetName) {
            case 'o_wsale_thumbnail_pos':
                return Boolean(this.productPageCarousel) && hasImages;
            case 'o_wsale_grid_spacing':
            case 'o_wsale_grid_columns':
                return Boolean(this.productPageGrid) && hasImages;
            case 'o_wsale_image_layout':
            case 'o_wsale_zoom_click':
            case 'o_wsale_zoom_none':
            case 'o_wsale_replace_main_image':
            case 'o_wsale_add_extra_images':
            case 'o_wsale_clear_extra_images':
            case 'o_wsale_zoom_mode':
                return hasImages;
            case 'o_wsale_zoom_hover':
            case 'o_wsale_zoom_both':
                return hasImages && !isFullImage;
        }
        return this._super(widgetName, params);
    }
});

options.registry.WebsiteSaleProductAttribute = options.Class.extend({
    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
     willStart: async function () {
        this.attributeID = this.$target.closest('[data-attribute_id]').data('attribute_id');
        return this._super(...arguments);
    },

    /**
     * @see this.selectClass for params
     */
    setDisplayType: function (previewMode, widgetValue, params) {
        // TODO this should be awaited
        this.rpc('/shop/config/attribute', {
            attribute_id: this.attributeID,
            display_type: widgetValue,
        }).then(() => this.trigger_up('request_save', {reload: true, optionSelector: this.data.selector}));
    },

    /**
     * @override
     */
    async _computeWidgetState(methodName, params) {
        switch (methodName) {
            case 'setDisplayType':
                return this.$target.closest('[data-attribute_display_type]').data('attribute_display_type');
        }
        return this._super(methodName, params);
    },
});

// Disable save for alternative products snippet
options.registry.SnippetSave.include({
    /**
     * @override
     */
    async _computeVisibility() {
        return await this._super(...arguments)
            && !this.$target.hasClass('o_wsale_alternative_products');
    }
});

options.registry.ReplaceMedia.include({
    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },
    /**
     * @override
     */
    async willStart() {
        const parent = this.$target.parent();
        this.isProductPageImage = this.$target.closest('.o_wsale_product_images').length > 0;
        // Product Page images may be the product's image or a record of `product.image`
        this.recordModel = parent.data('oe-model');
        this.recordId = parent.data('oe-id');
        return this._super(...arguments);
    },
    /**
     * Removes the image in the back-end
     */
    async removeMedia() {
        if (this.recordModel === "product.image") {
            // Unlink the "product.image" record as it is not the main product
            // image.
            await this.orm.unlink("product.image", [this.recordId]);
        }
        this.$target[0].remove();
        this.trigger_up("request_save", {reload: true, optionSelector: "#product_detail_main"});
    },
    /**
     * Change sequence of product page images
     *
     */
    async setPosition(previewMode, widgetValue, params) {
        // TODO this should be awaited
        this.rpc('/shop/product/resequence-image', {
            image_res_model: this.recordModel,
            image_res_id: this.recordId,
            move: widgetValue,
        }).then(() => this.trigger_up('request_save', {reload: true, optionSelector: '#product_detail_main'}));
    },
    /**
     * @override
     */
    async _computeWidgetVisibility(widgetName, params) {
        if (['media_wsale_resequence', 'media_wsale_remove'].includes(widgetName)) {
            // Only include these if we are inside of the product's page images
            return this.isProductPageImage;
        }
        return this._super(...arguments);
    }
});

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import VariantMixin from "@website_sale/js/variant_mixin";
import wSaleUtils from "@website_sale/js/website_sale_utils";
const cartHandlerMixin = wSaleUtils.cartHandlerMixin;
import "@website/libs/zoomodoo/zoomodoo";
import { browser } from "@web/core/browser/browser";
import {extraMenuUpdateCallbacks} from "@website/js/content/menu";
import { ProductImageViewer } from "@website_sale/js/components/website_sale_image_viewer";
import { jsonrpc } from "@web/core/network/rpc_service";
import { debounce, throttleForAnimation } from "@web/core/utils/timing";
import { listenSizeChange, SIZES, utils as uiUtils } from "@web/core/ui/ui_service";
import { isBrowserFirefox, hasTouch } from "@web/core/browser/feature_detection";
import { Component } from "@odoo/owl";

export const WebsiteSale = publicWidget.Widget.extend(VariantMixin, cartHandlerMixin, {
    selector: '.oe_website_sale',
    events: Object.assign({}, VariantMixin.events || {}, {
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
        'submit .o_wsale_products_searchbar_form': '_onSubmitSaleSearch',
        'change select[name="country_id"]': '_onChangeCountry',
        'change #shipping_use_same': '_onChangeShippingUseSame',
        'click .toggle_summary': '_onToggleSummary',
        'click #add_to_cart, .o_we_buy_now, #products_grid .o_wsale_product_btn .a-submit': 'async _onClickAdd',
        'click input.js_product_change': 'onChangeVariant',
        'change .js_main_product [data-attribute_exclusions]': 'onChangeVariant',
        'change oe_advanced_configurator_modal [data-attribute_exclusions]': 'onChangeVariant',
        'click .o_product_page_reviews_link': '_onClickReviewsLink',
        'mousedown .o_wsale_filmstip_wrapper': '_onMouseDown',
        'mouseleave .o_wsale_filmstip_wrapper': '_onMouseLeave',
        'mouseup .o_wsale_filmstip_wrapper': '_onMouseUp',
        'mousemove .o_wsale_filmstip_wrapper': '_onMouseMove',
        'click .o_wsale_filmstip_wrapper' : '_onClickHandler',
        'submit': '_onClickConfirmOrder',
        "change select[name='state_id']": "_onChangeState",
    }),

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);

        this._changeCartQuantity = debounce(this._changeCartQuantity.bind(this), 500);
        this._changeCountry = debounce(this._changeCountry.bind(this), 500);

        this.isWebsite = true;
        this.filmStripStartX = 0;
        this.filmStripIsDown = false;
        this.filmStripScrollLeft = 0;
        this.filmStripMoved = false;

        delete this.events['change .main_product:not(.in_cart) input.js_quantity'];
        delete this.events['change [data-attribute_exclusions]'];

        this.rpc = this.bindService("rpc");
    },
    /**
     * @override
     */
    start() {
        const def = this._super(...arguments);

        this._applyHashFromSearch();

        this.$("div.js_product")
            .toArray()
            .forEach((product) => {
            $('input.js_product_change', product).first().trigger('change');
        });

        // This has to be triggered to compute the "out of stock" feature and the hash variant changes
        this.triggerVariantChange(this.$el);

        this.$('select[name="country_id"]').change();

        listenSizeChange(() => {
            if (uiUtils.getSize() === SIZES.XL) {
                $('.toggle_summary_div').addClass('d-none d-xl-block');
            }
        })

        this._startZoom();

        window.addEventListener('hashchange', () => {
            this._applyHash();
            this.triggerVariantChange(this.$el);
        });

        // This allows conditional styling for the filmstrip
        const filmstripContainer = this.el.querySelector('.o_wsale_filmstip_container');
        const filmstripContainerWidth = filmstripContainer
            ? filmstripContainer.getBoundingClientRect().width : 0;
        const filmstripWrapper = this.el.querySelector('.o_wsale_filmstip_wrapper');
        const filmstripWrapperWidth = filmstripWrapper
            ? filmstripWrapper.getBoundingClientRect().width : 0;
        const isFilmstripScrollable = filmstripWrapperWidth < filmstripContainerWidth
        if (isBrowserFirefox() || hasTouch() || isFilmstripScrollable) {
            filmstripContainer?.classList.add('o_wsale_filmstip_fancy_disabled');
        }

        this.getRedirectOption();
        return def;
    },
    destroy() {
        this._super.apply(this, arguments);
        this._cleanupZoom();
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

    _onMouseDown: function (ev) {
        this.filmStripIsDown = true;
        this.filmStripStartX = ev.pageX - ev.currentTarget.offsetLeft;
        this.filmStripScrollLeft = ev.currentTarget.scrollLeft;
        this.formerTarget = ev.target;
        this.filmStripMoved = false;
    },
    _onMouseLeave: function (ev) {
        if (!this.filmStripIsDown) {
            return;
        }
        ev.currentTarget.classList.remove('activeDrag');
        this.filmStripIsDown = false
    },
    _onMouseUp: function (ev) {
        this.filmStripIsDown = false;
        ev.currentTarget.classList.remove('activeDrag');
    },
    _onMouseMove: function (ev) {
        if (!this.filmStripIsDown) {
            return;
        }
        ev.preventDefault();
        ev.currentTarget.classList.add('activeDrag');
        this.filmStripMoved = true;
        const x = ev.pageX - ev.currentTarget.offsetLeft;
        const walk = (x - this.filmStripStartX) * 2;
        ev.currentTarget.scrollLeft = this.filmStripScrollLeft - walk;
    },
    _onClickHandler: function(ev) {
        if(this.filmStripMoved) {
            ev.stopPropagation();
            ev.preventDefault();
        }
    },
    _applyHash: function () {
        const params = new URLSearchParams(window.location.hash.substring(1));
        if (params.get("attr")) {
            var attributeIds = params.get("attr").split(',');
            var $inputs = this.$('input.js_variant_change, select.js_variant_change option');
            attributeIds.forEach((id) => {
                var $toSelect = $inputs.filter('[data-value_id="' + id + '"]');
                if ($toSelect.is('input[type="radio"]')) {
                    $toSelect.prop('checked', true);
                } else if ($toSelect.is('option')) {
                    $toSelect.prop('selected', true);
                }
            });
            this._changeAttribute(['.css_attribute_color', '.o_variant_pills']);
        }
    },

    /**
     * Sets the url hash from the selected product options.
     *
     * @private
     */
    _setUrlHash: function ($parent) {
        var $attributes = $parent.find('input.js_variant_change:checked, select.js_variant_change option:selected');
        if (!$attributes.length) {
            return;
        }
        var attributeIds = $attributes.toArray().map((elem) => $(elem).data("value_id"));
        window.location.replace('#attr=' + attributeIds.join(','));
    },
    /**
     * Set the checked values active.
     *
     * @private
     * @param {Array} valueSelectors Selectors
     */
    _changeAttribute: function (valueSelectors) {
        valueSelectors.forEach((selector) => {
            $(selector).removeClass("active").filter(":has(input:checked)").addClass("active");
        });
    },
    /**
     * @private
     */
    _changeCartQuantity: function ($input, value, $dom_optional, line_id, productIDs) {
        $($dom_optional).toArray().forEach((elem) => {
            $(elem).find('.js_quantity').text(value);
            productIDs.push($(elem).find('span[data-product-id]').data('product-id'));
        });
        $input.data('update_change', true);

        this.rpc("/shop/cart/update_json", {
            line_id: line_id,
            product_id: parseInt($input.data('product-id'), 10),
            set_qty: value,
            display: true,
        }).then((data) => {
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
                // Ensures last cart removal is recorded
                browser.sessionStorage.setItem('website_sale_cart_quantity', 0);
                return window.location = '/shop/cart';
            }
            $input.val(data.quantity);
            $('.js_quantity[data-line-id='+line_id+']').val(data.quantity).text(data.quantity);

            wSaleUtils.updateCartNavBar(data);
            wSaleUtils.showWarning(data.notification_info.warning);
            // Propagating the change to the express checkout forms
            Component.env.bus.trigger('cart_amount_changed', [data.amount, data.minor_amount]);
        });
    },
    /**
     * @private
     */
    _changeCountry: function () {
        if (!$("#country_id").val()) {
            return;
        }
        return this.rpc("/shop/country_infos/" + $("#country_id").val(), {
            mode: $("#country_id").attr('mode'),
        }).then(function (data) {
            // placeholder phone_code
            $("input[name='phone']").attr('placeholder', data.phone_code !== 0 ? '+'+ data.phone_code : '');

            // populate states and display
            var selectStates = $("select[name='state_id']");
            // dont reload state at first loading (done in qweb)
            if (selectStates.data('init')===0 || selectStates.find('option').length===1) {
                if (data.states.length || data.state_required) {
                    selectStates.html('');
                    data.states.forEach((x) => {
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
                all_fields.forEach((field) => {
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
    _getProductImageLayout: function () {
        return document.querySelector("#product_detail_main").dataset.image_layout;
    },
    _getProductImageWidth: function () {
        return document.querySelector("#product_detail_main").dataset.image_width;
    },
    _getProductImageContainerSelector: function () {
        return {
            'carousel': "#o-carousel-product",
            'grid': "#o-grid-product",
        }[this._getProductImageLayout()];
    },
    _getProductImageContainer: function () {
        return document.querySelector(this._getProductImageContainerSelector());
    },
    _isEditorEnabled() {
        return document.body.classList.contains("editor_enable");
    },
    /**
     * @private
     */
    _startZoom: function () {
        const salePage = document.querySelector(".o_wsale_product_page");
        if (!salePage || this._getProductImageWidth() === "none") {
            return;
        }
        this._cleanupZoom();
        this.zoomCleanup = [];
        // Zoom on hover (except on mobile)
        if (salePage.dataset.ecomZoomAuto && !uiUtils.isSmall()) {
            const images = salePage.querySelectorAll("img[data-zoom]");
            for (const image of images) {
                const $image = $(image);
                const callback = () => {
                    $image.zoomOdoo({
                        event: "mouseenter",
                        attach: this._getProductImageContainerSelector(),
                        preventClicks: salePage.dataset.ecomZoomClick,
                        attachToTarget: this._getProductImageLayout() === "grid",
                    });
                    image.dataset.zoom = 1;
                };
                image.addEventListener('load', callback);
                this.zoomCleanup.push(() => {
                    image.removeEventListener('load', callback);
                    const zoomOdoo = $image.data("zoomOdoo");
                    if (zoomOdoo) {
                        zoomOdoo.hide();
                        $image.unbind();
                    }
                });
                if (image.complete) {
                    callback();
                }
            }
        }
        // Zoom on click
        if (salePage.dataset.ecomZoomClick) {
            // In this case we want all the images not just the ones that are "zoomables"
            const images = salePage.querySelectorAll(".product_detail_img");
            for (const image of images ) {
                const handler = () => {
                    if (salePage.dataset.ecomZoomAuto) {
                        // Remove any flyout
                        const flyouts = document.querySelectorAll(".zoomodoo-flyout");
                        for (const flyout of flyouts) {
                            flyout.remove();
                        }
                    }
                    this.call("dialog", "add", ProductImageViewer, {
                        selectedImageIdx: [...images].indexOf(image),
                        images,
                    });
                };
                image.addEventListener("click", handler);
                this.zoomCleanup.push(() => {
                    image.removeEventListener("click", handler);
                });
            }
        }
    },
    _cleanupZoom() {
        if (!this.zoomCleanup || !this.zoomCleanup.length) {
            return;
        }
        for (const cleanup of this.zoomCleanup) {
            cleanup();
        }
        this.zoomCleanup = undefined;
    },
    /**
     * On website, we display a carousel instead of only one image
     *
     * @override
     * @private
     */
    _updateProductImage: function ($productContainer, displayImage, productId, productTemplateId, newImages, isCombinationPossible) {
        let $images = $productContainer.find(this._getProductImageContainerSelector());
        // When using the web editor, don't reload this or the images won't
        // be able to be edited depending on if this is done loading before
        // or after the editor is ready.
        if ($images.length && !this._isEditorEnabled()) {
            const $newImages = $(newImages);
            $images.after($newImages);
            $images.remove();
            $images = $newImages;
            if ($images.attr('id') === 'o-carousel-product') {
                $images.carousel(0);
            }
            this._startZoom();
            // fix issue with carousel height
            this.trigger_up('widgets_start_request', {$target: $images});
        }
        $images.toggleClass('css_not_available', !isCombinationPossible);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickAdd: function (ev) {
        ev.preventDefault();
        var def = () => {
            this.getCartHandlerOptions(ev);
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
            self._updateRootProduct($form, productId);
            return self._onProductReady();
        });
    },

    _onProductReady: function () {
        return this._submitForm();
    },

    /**
     * Add custom variant values and attribute values that do not generate variants
     * in the params to submit form if 'stay on page' option is disabled, or call
     * '_addToCartInPage' otherwise.
     *
     * @private
     * @returns {Promise}
     */
    _submitForm: function () {
        const params = this.rootProduct;

        const $product = $('#product_detail');
        const productTrackingInfo = $product.data('product-tracking-info');
        if (productTrackingInfo) {
            productTrackingInfo.quantity = params.quantity;
            $product.trigger('add_to_cart_event', [productTrackingInfo]);
        }

        params.add_qty = params.quantity;
        params.product_custom_attribute_values = JSON.stringify(params.product_custom_attribute_values);
        params.no_variant_attribute_values = JSON.stringify(params.no_variant_attribute_values);
        delete params.quantity;
        return this.addToCart(params);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickAddCartJSON: function (ev) {
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
        if ($aSubmit.hasClass('a-submit-disable')) {
            $aSubmit.addClass("disabled");
        }
        if ($aSubmit.hasClass('a-submit-loading')) {
            var loading = '<span class="fa fa-cog fa-spin"/>';
            var fa_span = $aSubmit.find('span[class*="fa"]');
            if (fa_span.length) {
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
            const productGrid = this.el.querySelector(".o_wsale_products_grid_table_wrapper");
            if (productGrid) {
                productGrid.classList.add("opacity-50");
            }
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
    _onSubmitSaleSearch: function (ev) {
        if (!this.$('.dropdown_sorty_by').length) {
            return;
        }
        var $this = $(ev.currentTarget);
        if (!ev.isDefaultPrevented() && !$this.is(".disabled")) {
            ev.preventDefault();
            var oldurl = $this.attr('action');
            oldurl += (oldurl.indexOf("?")===-1) ? "?" : "";
            if ($this.find('[name=noFuzzy]').val() === "true") {
                oldurl += '&noFuzzy=true';
            }
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
        return this._changeCountry();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeState: function (ev) {
        return Promise.resolve();
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
        $parent.find(".o_we_buy_now").toggleClass('disabled', !isCombinationPossible);
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
        const params =  new URL(window.location).searchParams;
        if (params.get("attrib")) {
            const dataValueIds = [];
            for (const attrib of [].concat(params.get("attrib"))) {
                const attribSplit = attrib.split('-');
                const attribValueSelector = `.js_variant_change[name="ptal-${attribSplit[0]}"][value="${attribSplit[1]}"]`;
                const attribValue = this.el.querySelector(attribValueSelector);
                if (attribValue !== null) {
                    dataValueIds.push(attribValue.dataset.value_id);
                }
            }
            if (dataValueIds.length) {
                window.location.hash = `attr=${dataValueIds.join(',')}`;
            }
        }
        this._applyHash();
    },
    /**
     * @private
     */
    _onClickReviewsLink: function () {
        $('#o_product_page_reviews_content').collapse('show');
    },
    /**
     * Prevent multiclicks on confirm button when the form is submitted
     *
     * @private
     */
    _onClickConfirmOrder: function () {
        const submitFormButton = $('form[name="o_wsale_confirm_order"]').find('button[type="submit"]');
        submitFormButton.attr('disabled', true);
        setTimeout(() => submitFormButton.attr('disabled', false), 5000);
    },

    // -------------------------------------
    // Utils
    // -------------------------------------
    /**
     * Update the root product during an Add process.
     *
     * @private
     * @param {Object} $form
     * @param {Number} productId
     */
    _updateRootProduct($form, productId) {
        this.rootProduct = {
            product_id: productId,
            quantity: parseFloat($form.find('input[name="add_qty"]').val() || 1),
            product_custom_attribute_values: this.getCustomVariantValues($form.find('.js_product')),
            variant_values: this.getSelectedVariantValues($form.find('.js_product')),
            no_variant_attribute_values: this.getNoVariantAttributeValues($form.find('.js_product'))
        };
    },
});

publicWidget.registry.WebsiteSale = WebsiteSale

publicWidget.registry.WebsiteSaleLayout = publicWidget.Widget.extend({
    selector: '.oe_website_sale',
    disabledInEditableMode: false,
    events: {
        'change .o_wsale_apply_layout input': '_onApplyShopLayoutChange',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onApplyShopLayoutChange: function (ev) {
        const wysiwyg = this.options.wysiwyg;
        if (wysiwyg) {
            wysiwyg.odooEditor.observerUnactive('_onApplyShopLayoutChange');
        }
        var clickedValue = $(ev.target).val();
        var isList = clickedValue === 'list';
        if (!this.editableMode) {
            jsonrpc('/shop/save_shop_layout_mode', {
                'layout_mode': isList ? 'list' : 'grid',
            });
        }

        const activeClasses = ev.target.parentElement.dataset.activeClasses.split(' ');
        ev.target.parentElement.querySelectorAll('.btn').forEach((btn) => {
            activeClasses.map(c => btn.classList.toggle(c));
        });

        var $grid = this.$('#products_grid');
        // Disable transition on all list elements, then switch to the new
        // layout then reenable all transitions after having forced a redraw
        // TODO should probably be improved to allow disabling transitions
        // altogether with a class/option.
        $grid.find('*').css('transition', 'none');
        $grid.toggleClass('o_wsale_layout_list', isList);
        void $grid[0].offsetWidth;
        $grid.find('*').css('transition', '');
        if (wysiwyg) {
            wysiwyg.odooEditor.observerActive('_onApplyShopLayoutChange');
        }
    },
});

publicWidget.registry.websiteSaleCart = publicWidget.Widget.extend({
    selector: '.oe_website_sale .oe_cart',
    events: {
        'click .js_change_billing': '_onClickChangeBilling',
        'click .js_change_shipping': '_onClickChangeShipping',
        'click .js_edit_address': '_onClickEditAddress',
        'click .js_delete_product': '_onClickDeleteProduct',
    },

    /**
     * @override
     */
    async start() {
        document.querySelector('.o_cta_navigation_placeholder')?.classList.remove('d-none')
        const ctaContainer = document.querySelector('.o_cta_navigation_container');
        if (ctaContainer) {
            const placeholder = document.querySelector('.o_cta_navigation_placeholder');
            placeholder.style.height = `${ctaContainer.offsetHeight}px`;
            ctaContainer.style.top = `calc(100% - ${ctaContainer.offsetHeight}px)`;
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onClickChangeBilling: function (ev) {
        this._onClickChangeAddress(ev, 'all_billing', 'js_change_billing');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickChangeShipping: function (ev) {
        this._onClickChangeAddress(ev, 'all_shipping', 'js_change_shipping');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickChangeAddress: function (ev, rowAddrClass, cardClass) {
        var $old = $(`.${rowAddrClass}`).find('.card.border.border-primary');
        $old.find('.btn-addr').toggle();
        $old.addClass(cardClass);
        $old.removeClass('bg-primary border border-primary');

        var $new = $(ev.currentTarget).parent('div.one_kanban').find('.card');
        $new.find('.btn-addr').toggle();
        $new.removeClass(cardClass);
        $new.addClass('bg-primary border border-primary');

        // TODO this should not be a form, but a clean rpc to /shop/cart/update_address
        var $form = $(ev.currentTarget).parent('div.one_kanban').find('form.d-none');
        $.post($form.attr('action'), $form.serialize()+'&xhr=1');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickEditAddress: function (ev) {
        // Do not trigger _onClickChangeBilling or _onClickChangeShipping when customer
        // clicks on the pencil to update the address
        ev.stopPropagation();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickDeleteProduct: function (ev) {
        ev.preventDefault();
        $(ev.currentTarget).closest('.o_cart_product').find('.js_quantity').val(0).trigger('change');
    },
});

publicWidget.registry.websiteSaleCarouselProduct = publicWidget.Widget.extend({
    selector: '#o-carousel-product',
    disabledInEditableMode: false,
    events: {
        'wheel .o_carousel_product_indicators': '_onMouseWheel',
    },

    /**
     * @override
     */
    async start() {
        await this._super(...arguments);
        this._updateCarouselPosition();
        this.throttleOnResize = throttleForAnimation(this._onSlideCarouselProduct.bind(this));
        extraMenuUpdateCallbacks.push(this._updateCarouselPosition.bind(this));
        if (this.$el.find('.carousel-indicators').length > 0) {
            this.$el.on('slide.bs.carousel.carousel_product_slider', this._onSlideCarouselProduct.bind(this));
            $(window).on('resize.carousel_product_slider', this.throttleOnResize);
            this._updateJustifyContent();
        }
    },
    /**
     * @override
     */
    destroy() {
        this.$el.css('top', '');
        this.$el.off('.carousel_product_slider');
        if (this.throttleOnResize) {
            this.throttleOnResize.cancel();
        }
        this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _updateCarouselPosition() {
        let size = 5;
        for (const el of document.querySelectorAll('.o_top_fixed_element')) {
            size += $(el).outerHeight();
        }
        this.$el.css('top', size);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Center the selected indicator to scroll the indicators list when it
     * overflows.
     *
     * @private
     * @param {Event} ev
     */
    _onSlideCarouselProduct: function (ev) {
        const isReversed = this.$el.css('flex-direction') === "column-reverse";
        const isLeftIndicators = this.$el.hasClass('o_carousel_product_left_indicators');
        const $indicatorsDiv = isLeftIndicators ? this.$el.find('.o_carousel_product_indicators') : this.$el.find('.carousel-indicators');
        let indicatorIndex = $(ev.relatedTarget).index();
        indicatorIndex = indicatorIndex > -1 ? indicatorIndex : this.$el.find('li.active').index();
        const $indicator = $indicatorsDiv.find('[data-bs-slide-to=' + indicatorIndex + ']');
        const indicatorsDivSize = isLeftIndicators && !isReversed ? $indicatorsDiv.outerHeight() : $indicatorsDiv.outerWidth();
        const indicatorSize = isLeftIndicators && !isReversed ? $indicator.outerHeight() : $indicator.outerWidth();
        const indicatorPosition = isLeftIndicators && !isReversed ? $indicator.position().top : $indicator.position().left;
        const scrollSize = isLeftIndicators && !isReversed ? $indicatorsDiv[0].scrollHeight : $indicatorsDiv[0].scrollWidth;
        let indicatorsPositionDiff = (indicatorPosition + (indicatorSize/2)) - (indicatorsDivSize/2);
        indicatorsPositionDiff = Math.min(indicatorsPositionDiff, scrollSize - indicatorsDivSize);
        this._updateJustifyContent();
        const indicatorsPositionX = isLeftIndicators && !isReversed ? '0' : '-' + indicatorsPositionDiff;
        const indicatorsPositionY = isLeftIndicators && !isReversed ? '-' + indicatorsPositionDiff : '0';
        const translate3D = indicatorsPositionDiff > 0 ? "translate3d(" + indicatorsPositionX + "px," + indicatorsPositionY + "px,0)" : '';
        $indicatorsDiv.css("transform", translate3D);
    },
    /**
     * @private
     */
     _updateJustifyContent: function () {
        const $indicatorsDiv = this.$el.find('.carousel-indicators');
        $indicatorsDiv.css('justify-content', 'start');
        if (uiUtils.getSize() <= SIZES.MD) {
            if (($indicatorsDiv.children().last().position().left + this.$el.find('li').outerWidth()) < $indicatorsDiv.outerWidth()) {
                $indicatorsDiv.css('justify-content', 'center');
            }
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onMouseWheel: function (ev) {
        ev.preventDefault();
        if (ev.originalEvent.deltaY > 0) {
            this.$el.carousel('next');
        } else {
            this.$el.carousel('prev');
        }
    },
});

publicWidget.registry.websiteSaleProductPageReviews = publicWidget.Widget.extend({
    selector: '#o_product_page_reviews',
    disabledInEditableMode: false,

    /**
     * @override
     */
    async start() {
        await this._super(...arguments);
        this._updateChatterComposerPosition();
        extraMenuUpdateCallbacks.push(this._updateChatterComposerPosition.bind(this));
    },
    /**
     * @override
     */
    destroy() {
        this.$el.find('.o_portal_chatter_composer').css('top', '');
        this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _updateChatterComposerPosition() {
        let size = 20;
        for (const el of document.querySelectorAll('.o_top_fixed_element')) {
            size += $(el).outerHeight();
        }
        this.$el.find('.o_portal_chatter_composer').css('top', size);
    },
});

export default {
    WebsiteSale: publicWidget.registry.WebsiteSale,
    WebsiteSaleLayout: publicWidget.registry.WebsiteSaleLayout,
    websiteSaleCart: publicWidget.registry.websiteSaleCart,
    WebsiteSaleCarouselProduct: publicWidget.registry.websiteSaleCarouselProduct,
    WebsiteSaleProductPageReviews: publicWidget.registry.websiteSaleProductPageReviews,
};

```

## File: static\src\js\website_sale_category_link.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget'

publicWidget.registry.ProductCategoriesLinks = publicWidget.Widget.extend({
    selector: '.o_wsale_products_page',
    events: {
        'click [data-link-href]': '_openLink',
    },

    _openLink: function (ev) {
        const productsDiv = this.el.querySelector('.o_wsale_products_grid_table_wrapper');
        if (productsDiv) {
            productsDiv.classList.add('opacity-50');
        }
        window.location.href = ev.currentTarget.getAttribute('data-link-href');
    },
});

```

## File: static\src\js\website_sale_delivery.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";
import { renderToElement } from "@web/core/utils/render";
import { KeepLast } from "@web/core/utils/concurrency";
import { Component } from "@odoo/owl";

publicWidget.registry.websiteSaleDelivery = publicWidget.Widget.extend({
    selector: '.oe_website_sale',
    events: {
        'change select[name="shipping_id"]': '_onSetAddress',
        'click .o_delivery_carrier_select': '_onCarrierClick',
        "click .o_address_select": "_onClickLocation",
        "click .o_remove_order_location": "_onClickRemoveLocation",
        "click .o_show_pickup_locations": "_onClickShowLocations",
        "click .o_payment_option_card": "_onClickPaymentMethod"
    },

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    start: async function () {
        this.carriers = Array.from(document.querySelectorAll('input[name="delivery_type"]'));
        this.keepLast = new KeepLast();
        // Workaround to:
        // - update the amount/error on the label at first rendering
        // - prevent clicking on 'Pay Now' if the shipper rating fails
        if (this.carriers.length > 0) {
            const carrierChecked = this.carriers.filter(e =>e.checked)
            if (carrierChecked.length === 0) {
                this._disablePayButton();
            } else {
                this.forceClickCarrier = true;
                await this._getCurrentLocation();
                carrierChecked[0].click();
            }
        }

        await this.carriers.forEach(async (carrierInput) => {
            this._showLoading((carrierInput));
            await this._getCarrierRateShipment(carrierInput);
        });
        if (this._super && typeof(this._super.apply)==="function") {
          return this._super.apply(this, arguments);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * @private
     */
    _getCurrentLocation: async function () {
        const data = await this.rpc("/shop/access_point/get");
        const carriers = document.querySelectorAll('.o_delivery_carrier_select')
        for (let carrier of carriers) {
            const deliveryType = carrier.querySelector('input[type="radio"]').getAttribute("delivery_type");
            const deliveryName = carrier.querySelector('label').innerText;
            const showLoc = carrier.querySelector(".o_show_pickup_locations");
            if (!showLoc) {
                continue;
            }
            const orderLoc = carrier.querySelector(".o_order_location");
            if (data[deliveryType + '_access_point'] && data.delivery_name == deliveryName) {
                orderLoc.querySelector(".o_order_location_name").innerText = data.name
                orderLoc.querySelector(".o_order_location_address").innerText = data[deliveryType + '_access_point']
                orderLoc.parentElement.classList.remove("d-none");
                showLoc.classList.add("d-none");
                // Prevent force clicking a carrier since it is already set.
                this.forceClickCarrier = false;
                break;
            } else {
                orderLoc.parentElement.classList.add("d-none");
                showLoc.classList.remove("d-none");
            }
        }
    },

    /**
     * @private
     * @param {Element} docCarrier //carrier element from document
     */
    _specificDropperDisplay: function (docCarrier) {
        if(!docCarrier?.closest("li").getElementsByTagName("input")[0].getAttribute("delivery_type")){
            return;
        }
        while (docCarrier.firstChild) {
            docCarrier.lastChild.remove();
        }
        const currentCarrierChecked = docCarrier.closest("li").getElementsByTagName("input")[0].checked;
        const span = document.createElement("em");
        if (!currentCarrierChecked || this.carriers.length == 1) {
            span.textContent = _t("select to see available Pick-Up Locations");
            span.classList.add("text-muted");
        }
        docCarrier.appendChild(span);
    },
    /**
     * @private
     * @param {Element} carrierInput
     */
    _showLoading: function (carrierInput) {
        const priceTag = carrierInput.parentNode.querySelector('.o_wsale_delivery_badge_price')
        while (priceTag.firstChild) {
            priceTag.removeChild(priceTag.lastChild);
        }
        const loadingCircle = priceTag.appendChild(document.createElement('span'));
        loadingCircle.classList.add("fa", "fa-circle-o-notch", "fa-spin");
    },
    /**
     * Update the total cost according to the selected shipping method
     *
     * @private
     * @param {float} amount : The new total amount of to be paid
     */
    _updateShippingCost: function(amount) {
        Component.env.bus.trigger('update_shipping_cost', amount);
    },
     /**
     * Get the rate shipment of a carrier
     *
     * @private
     * @params {Object} carrier: The carrier element
     */
    _getCarrierRateShipment: async function(carrierInput) {
      const result = await this.rpc('/shop/carrier_rate_shipment', {
            'carrier_id': carrierInput.value,
      });
      this._handleCarrierUpdateResultBadge(result);
    },
    /**
     * @private
     * @param {Object} result
     */
    _handleCarrierUpdateResult: async function (carrierInput) {
        const result = await this.rpc('/shop/update_carrier', {
            'carrier_id': carrierInput.value,
            'no_reset_access_point_address': this.forceClickCarrier,
        })
        this.result = result;
        this._handleCarrierUpdateResultBadge(result);
        if (carrierInput.checked) {
            var amountDelivery = document.querySelector('#order_delivery .monetary_field');
            var amountUntaxed = document.querySelector('#order_total_untaxed .monetary_field');
            var amountTax = document.querySelector('#order_total_taxes .monetary_field');
            var amountTotal = document.querySelectorAll('#order_total .monetary_field, #amount_total_summary.monetary_field');

            amountDelivery.innerHTML = result.new_amount_delivery;
            amountUntaxed.innerHTML = result.new_amount_untaxed;
            amountTax.innerHTML = result.new_amount_tax;
            amountTotal.forEach(total => total.innerHTML = result.new_amount_total);
            // we need to check if it's the carrier that is selected
            if (result.new_amount_total_raw !== undefined) {
                this._updateShippingCost(result.new_amount_total_raw);
                // reload page only when amount_total switches between zero and not zero
                const hasPaymentMethod = document.querySelector(
                    "div[name='o_website_sale_free_cart']"
                ) === null;
                const shouldDisplayPaymentMethod = result.new_amount_total_raw !== 0;
                if (hasPaymentMethod !==  shouldDisplayPaymentMethod) {
                    location.reload(false);
                }
            }
            this._updateShippingCost(result.new_amount_delivery);
        }
        this._enableButton(result.status);
        let currentId = result.carrier_id
        const showLocations = document.querySelectorAll(".o_show_pickup_locations");

        for (const showLoc of showLocations) {
            const currentCarrierId = showLoc.closest("li").getElementsByTagName("input")[0].value;
            if (currentCarrierId == currentId) {
                this._specificDropperDisplay(showLoc);
                break;
            }
        }
    },

    /**
     * @private
     * @param {Object} result
     */
    _handleCarrierUpdateResultBadge: function (result) {
        var $carrierBadge = $('#delivery_carrier input[name="delivery_type"][value=' + result.carrier_id + '] ~ .o_wsale_delivery_badge_price');

        if (result.status === true) {
             // if free delivery (`free_over` field), show 'Free', not '$0'
             if (result.is_free_delivery) {
                 $carrierBadge.text(_t('Free'));
             } else {
                 $carrierBadge.html(result.new_amount_delivery);
             }
             $carrierBadge.removeClass('o_wsale_delivery_carrier_error');
        } else {
            $carrierBadge.addClass('o_wsale_delivery_carrier_error');
            $carrierBadge.text(result.error_message);
        }
    },

    /**
     * Disable the payment button.
     *
     * @private
     * @return {void}
     */
    _disablePayButton: function (){
        Component.env.bus.trigger('disablePaymentButton');
    },

    _disablePayButtonNoPickupPoint : function (ev){
        const selectedCarrierEl = ev.currentTarget.closest('.o_delivery_carrier_select');
        const address = selectedCarrierEl.querySelector('.o_order_location_address').innerText
        const orderLocationContainer = selectedCarrierEl.querySelector('.o_order_location').parentNode;
        const hasPickUpLocations = selectedCarrierEl.querySelector('.o_list_pickup_locations');

        document.querySelectorAll('.error_no_pick_up_point').forEach(el => el.remove());

        if (hasPickUpLocations && (address == "" || orderLocationContainer.classList.contains("d-none"))) {
            this._disablePayButton();
            const errorNode = document.createElement("i");
            errorNode.classList.add("small", "error_no_pick_up_point","ms-2");
            errorNode.textContent = _t("Select a pick-up point");
            errorNode.style = "color:red;";
            selectedCarrierEl.insertBefore(errorNode, selectedCarrierEl.querySelector("label").nextElementSibling);
        }
    },

    _checkCarrier: async function (ev, carrier_id) {
        ev.stopPropagation();
        await this.keepLast.add(this.rpc('/shop/update_carrier', {
            carrier_id: carrier_id,
        }))
        var closestDocElement = ev.currentTarget.closest('.o_delivery_carrier_select');
        var radio = closestDocElement.querySelector('input[type="radio"]');
        radio.checked = true;
        this._disablePayButtonNoPickupPoint(ev)
    },

    _onClickPaymentMethod: async function (ev) {
        const carriers = Array.from(document.querySelectorAll('.o_delivery_carrier_select'))
        if(carriers.length === 0){
            return;
        }
        this._disablePayButton();
        let carrierChecked = null;
        carriers.forEach((carrier) => {
            if (carrier.querySelector('input').checked){
                carrierChecked = carrier;
            }
        })
        if (!carrierChecked) {
            return;
        }
        const carrier_id = carrierChecked?.querySelector('input')?.value;
        const result = await this.rpc('/shop/update_carrier', {
            'carrier_id': carrier_id,
            'no_reset_access_point_address': true,
        })
        this._enableButton(result.status);
    },
    /**
     * Enable the payment button if the rate_shipment request succeeded.
     *
     * @private
     * @param {boolean} status - The status of the rate_shipment request.
     * @return {void}
     */
    _enableButton(status){
        if (status) {
            Component.env.bus.trigger('enablePaymentButton');
        }
        else {
            this._disablePayButton();
        }
    },

    _isPickupLocationSelected: function (ev) {
        return !ev.currentTarget.closest('.o_delivery_carrier_select').querySelector(".o_order_location").parentElement.classList.contains("d-none");
    },

    _shouldDisplayPickupLocations: function (ev) {
        const pickupPointsAreNeeded = ev.currentTarget.querySelector('.o_show_pickup_locations');
        const pickupPointsAreDisplayed = ev.currentTarget.querySelector('.o_list_pickup_locations')?.hasChildNodes();
        return pickupPointsAreNeeded && !pickupPointsAreDisplayed && !this._isPickupLocationSelected(ev);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickRemoveLocation: async function (ev) {
        ev.stopPropagation();
        await this.rpc("/shop/access_point/set", {
            access_point_encoded: null,
        })
        const deliveryTypeInput = ev.currentTarget.closest(".o_delivery_carrier_select").querySelector('input[name="delivery_type"]');
        const deliveryTypeId = deliveryTypeInput.value;
        await Promise.all([this._getCurrentLocation(),this._checkCarrier(ev,deliveryTypeId)])
        await this._onClickShowLocations(ev);
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onClickShowLocations: async function (ev) {
        // This checks if there is a pick up point already select with that carrier
        if (this._isPickupLocationSelected(ev)) {
            return;
        }
        const showPickupLocations = ev.currentTarget.closest('.o_delivery_carrier_select').querySelector('.o_show_pickup_locations');
        const modal = showPickupLocations?.nextElementSibling;
        if (!modal) {
            return;
        }

        while (modal.firstChild) {
            modal.lastChild.remove();
        }
        const deliveryTypeInput = ev.currentTarget.closest(".o_delivery_carrier_select").querySelector('input[name="delivery_type"]');
        const deliveryType = deliveryTypeInput.getAttribute("delivery_type");
        const deliveryTypeId = deliveryTypeInput.value;
        await this._checkCarrier(ev,deliveryTypeId)
        $(renderToElement(deliveryType + "_pickup_location_loading")).appendTo($(modal));
        const data = await this.rpc("/shop/access_point/close_locations");
        if (modal.firstChild){
            modal.firstChild.remove();
        }
        if (data.error || (data.close_locations.length === 0)) {
            const errorMessage = document.createElement("em");
            errorMessage.classList.add("text-error");
            errorMessage.innerText = data.error ? data.error : "No available Pick-Up Locations";
            modal.appendChild(errorMessage);
            return;
        }

        var listToRender = deliveryType + "_pickup_location_list";
        var dataToRender = {partner_address: data.partner_address};
        dataToRender[deliveryType + "_pickup_locations"] = data.close_locations;
        $(renderToElement(listToRender, dataToRender)).appendTo($(modal));

        const showLocations = document.querySelectorAll(".o_show_pickup_locations");
        if (!ev.currentTarget.closest(".o_delivery_carrier_select")) {
            return;
        }
        for (const showLoc of showLocations) {
            this._specificDropperDisplay(showLoc);
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onCarrierClick: async function (ev) {
        const radio = ev.currentTarget.closest('.o_delivery_carrier_select').querySelector(
            'input[type="radio"]'
        );
        if (radio.checked && !this._shouldDisplayPickupLocations(ev) && !this.forceClickCarrier) {
            return;
        }
        this.forceClickCarrier = false;

        // Clear order locations on carrier change.
        const orderLocs = document.querySelectorAll('.o_order_location');
        orderLocs.forEach(loc => {
            loc.querySelector('.o_order_location_name').textContent = '';
            loc.querySelector('.o_order_location_address').textContent = '';
            const divDNone = loc.parentElement;
            if (!divDNone.classList.contains('d-none')) {
                divDNone.classList.add('d-none');
            }
        });

        this._disablePayButton();
        this._showLoading(radio);
        radio.checked = true;
        await this._onClickShowLocations(ev);
        await this._handleCarrierUpdateResult(radio);
        this._disablePayButtonNoPickupPoint(ev);
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onClickLocation: async function (ev) {
        const carrierId = ev.currentTarget.closest(".o_delivery_carrier_select").childNodes[1].value;
        await this._checkCarrier(ev,carrierId)
        const modal = ev.target.closest(".o_list_pickup_locations");
        const encodedLocation = ev.target.previousElementSibling.innerText;
        await this.rpc("/shop/access_point/set", {
            access_point_encoded: encodedLocation,
        })
        while (modal.firstChild) {
            modal.lastChild.remove();
        }
        await this._getCurrentLocation();
        document.querySelectorAll('.error_no_pick_up_point').forEach(el => el.remove());
        const result = await this.rpc('/shop/update_carrier', {
            'carrier_id': carrierId,
            'no_reset_access_point_address': true,
        })
        this._enableButton(result.status);
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onSetAddress: function (ev) {
        var value = $(ev.currentTarget).val();
        var $providerFree = $('select[name="country_id"]:not(.o_provider_restricted), select[name="state_id"]:not(.o_provider_restricted)');
        var $providerRestricted = $('select[name="country_id"].o_provider_restricted, select[name="state_id"].o_provider_restricted');
        if (value === 0) {
            // Ship to the same address : only show shipping countries available for billing
            $providerFree.hide().attr('disabled', true);
            $providerRestricted.show().attr('disabled', false).change();
        } else {
            // Create a new address : show all countries available for billing
            $providerFree.show().attr('disabled', false).change();
            $providerRestricted.hide().attr('disabled', true);
        }
    },
});

```

## File: static\src\js\website_sale_form_editor.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import FormEditorRegistry from "@website/js/form_editor_registry";

FormEditorRegistry.add('create_customer', {
    formFields: [{
        type: 'char',
        modelRequired: true,
        name: 'name',
        fillWith: 'name',
        string: _t('Your Name'),
    }, {
        type: 'email',
        required: true,
        fillWith: 'email',
        name: 'email',
        string: _t('Your Email'),
    }, {
        type: 'tel',
        fillWith: 'phone',
        name: 'phone',
        string: _t('Phone Number'),
    }, {
        type: 'char',
        name: 'company_name',
        fillWith: 'commercial_company_name',
        string: _t('Company Name'),
    }],
});

```

## File: static\src\js\website_sale_offcanvas.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.websiteSaleOffcanvas = publicWidget.Widget.extend({
    selector: '#o_wsale_offcanvas',
    events: {
        'show.bs.offcanvas': '_toggleFilters',
        'hidden.bs.offcanvas': '_toggleFilters',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Unfold active filters, fold inactive ones
     *
     * @private
     * @param {Event} ev
     */
    _toggleFilters: function (ev) {
        for (const btn of this.el.querySelectorAll('button[data-status]')) {
            if(btn.classList.contains('collapsed') && btn.dataset.status == "active" || ! btn.classList.contains('collapsed') && btn.dataset.status == "inactive" ) {
                btn.click();
            }
        }
    },
});

```

## File: static\src\js\website_sale_price_range_option.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.multirangePriceSelector = publicWidget.Widget.extend({
    selector: '.o_wsale_products_page',
    events: {
        'newRangeValue #o_wsale_price_range_option input[type="range"]': '_onPriceRangeSelected',
    },

    //----------------------------------------------------------------------
    // Handlers
    //----------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onPriceRangeSelected(ev) {
        const range = ev.currentTarget;
        const searchParams = new URLSearchParams(window.location.search);
        searchParams.delete("min_price");
        searchParams.delete("max_price");
        if (parseFloat(range.min) !== range.valueLow) {
            searchParams.set("min_price", range.valueLow);
        }
        if (parseFloat(range.max) !== range.valueHigh) {
            searchParams.set("max_price", range.valueHigh);
        }
        let product_list_div = this.el.querySelector('.o_wsale_products_grid_table_wrapper');
        if (product_list_div) {
            product_list_div.classList.add('opacity-50');
        }
        window.location.search = searchParams.toString();
    },
});

```

## File: static\src\js\website_sale_recently_viewed.js

```javascript
/** @odoo-module **/

import { debounce } from "@web/core/utils/timing";
import publicWidget from "@web/legacy/js/public/public_widget";
import { cookie } from "@web/core/browser/cookie";;

publicWidget.registry.productsRecentlyViewedUpdate = publicWidget.Widget.extend({
    selector: '#product_detail',
    events: {
        'change input.product_id[name="product_id"]': '_onProductChange',
    },
    debounceValue: 500,

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        this._onProductChange = debounce(this._onProductChange, this.debounceValue);
        this.rpc = this.bindService("rpc");
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
        if (cookie.get(cookieName)) {
            return; // Already tracked in the last 30min
        }
        if ($(this.el).find('.js_product.css_not_available').length) {
            return; // Variant not possible
        }
        this.rpc('/shop/products/recently_viewed_update', {
            product_id: productId,
        }).then(function (res) {
            cookie.set(cookieName, productId, 30 * 60, 'optional');
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

```

## File: static\src\js\website_sale_reorder.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { debounce as debounceFn } from "@web/core/utils/timing";
import publicWidget from "@web/legacy/js/public/public_widget";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { formatCurrency } from "@web/core/currency";

// Widget responsible for openingn the modal (giving out the sale order id)

publicWidget.registry.SaleOrderPortalReorderWidget = publicWidget.Widget.extend({
    selector: ".o_portal_sidebar",
    events: {
        "click .o_wsale_reorder_button": "_onReorder",
    },

    _onReorder(ev) {
        const orderId = parseInt(ev.currentTarget.dataset.saleOrderId);
        const urlSearchParams = new URLSearchParams(window.location.search);
        if (!orderId || !urlSearchParams.has("access_token")) {
            return;
        }
        // Open the modal
        this.call("dialog", "add", ReorderDialog, {
            orderId: orderId,
            accessToken: urlSearchParams.get("access_token"),
        });
    },
});

import { useService } from "@web/core/utils/hooks";
import { Dialog } from "@web/core/dialog/dialog";
import { Component, onWillStart } from "@odoo/owl";

// Reorder Dialog

export class ReorderConfirmationDialog extends ConfirmationDialog {}
ReorderConfirmationDialog.template = "website_sale.ReorderConfirmationDialog";

export class ReorderDialog extends Component {
    setup() {
        this.rpc = useService("rpc");
        this.orm = useService("orm");
        this.dialogService = useService("dialog");
        this.formatCurrency = formatCurrency;

        onWillStart(this.onWillStartHandler.bind(this));
    }

    async onWillStartHandler() {
        // Cart Qty should not change while the dialog is opened.
        this.cartQty = parseInt(sessionStorage.getItem("website_sale_cart_quantity"));
        if (!this.cartQty) {
            this.cartQty = await this.rpc("/shop/cart/quantity");
        }
        // Get required information about the order
        this.content = await this.rpc("/my/orders/reorder_modal_content", {
            order_id: this.props.orderId,
            access_token: this.props.accessToken,
        });
        // Get required information about each products
        for (const product of this.content.products) {
            product.debouncedLoadProductCombinationInfo = debounceFn(() => {
                this.loadProductCombinationInfo(product).then(this.render.bind(this));
            }, 200);
        }
    }

    get total() {
        return this.content.products.reduce((total, product) => {
            if (product.add_to_cart_allowed) {
                total += product.combinationInfo.price * product.qty;
            }
            return total;
        }, 0);
    }

    get hasBuyableProducts() {
        return this.content.products.some((product) => product.add_to_cart_allowed);
    }

    async loadProductCombinationInfo(product) {
        product.combinationInfo = await this.rpc("/website_sale/get_combination_info", {
            product_template_id: product.product_template_id,
            product_id: product.product_id,
            combination: product.combination,
            add_qty: product.qty,
            context: {
                website_sale_no_images: true,
            },
        });
    }

    getWarningForProduct(product) {
        if (!product.add_to_cart_allowed) {
            return _t("This product is not available for purchase.");
        }
        return false;
    }

    changeProductQty(product, newQty) {
        const productNewQty = Math.max(0, newQty);
        const qtyChanged = productNewQty !== product.qty;
        product.qty = productNewQty;
        this.render(true);
        if (!qtyChanged) {
            return;
        }
        product.debouncedLoadProductCombinationInfo();
    }

    onChangeProductQtyInput(ev, product) {
        const newQty = parseFloat(ev.target.value) || product.qty;
        this.changeProductQty(product, newQty);
    }

    async confirmReorder(ev) {
        if (this.confirmed) {
            return;
        }
        this.confirmed = true;
        const onConfirm = async () => {
            await this.addProductsToCart();
            window.location = "/shop/cart";
        };
        if (this.cartQty) {
            // Open confirmation modal
            this.dialogService.add(ReorderConfirmationDialog, {
                body: _t("Do you wish to clear your cart before adding products to it?"),
                confirm: async () => {
                    await this.rpc("/shop/cart/clear");
                    await onConfirm();
                },
                cancel: onConfirm,
                dismiss: () => {}, // Prevents fallback of 'cancel' from Confirmation Dialog
            });
        } else {
            await onConfirm();
        }
    }

    async addProductsToCart() {
        for (const product of this.content.products) {
            if (!product.add_to_cart_allowed) {
                continue;
            }
            await this.rpc("/shop/cart/update_json", {
                product_id: product.product_id,
                add_qty: product.qty,
                no_variant_attribute_values: JSON.stringify(product.no_variant_attribute_values),
                product_custom_attribute_values: JSON.stringify(product.product_custom_attribute_values),
                display: false,
            });
        }
    }
}
ReorderDialog.props = {
    close: Function,
    orderId: Number,
    accessToken: String,
};
ReorderDialog.components = {
    Dialog,
};
ReorderDialog.template = "website_sale.ReorderModal";

```

## File: static\src\js\website_sale_tracking.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.websiteSaleTracking = publicWidget.Widget.extend({
    selector: '.oe_website_sale',
    events: {
        'click form[action="/shop/cart/update"] a.a-submit': '_onAddProductIntoCart',
        'click a[href^="/shop/checkout"]': '_onCheckoutStart',
        'click a[href^="/web/login?redirect"][href*="/shop/checkout"]': '_onCustomerSignin',
        'click form[action="/shop/confirm_order"] a.a-submit': '_onOrder',
        'click form[target="_self"] button[type=submit]': '_onOrderPayment',
        'view_item_event': '_onViewItem',
        'add_to_cart_event': '_onAddToCart',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;

        // ...
        const $confirmation = this.$('div.oe_website_sale_tx_status');
        if ($confirmation.length) {
            const orderID = $confirmation.data('order-id');
            const json = $confirmation.data('order-tracking-info');
            this._vpv('/stats/ecom/order_confirmed/' + orderID);
            self._trackGA('event', 'purchase', json);
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
        const websiteGA = window.gtag || function () {};
        websiteGA.apply(this, arguments);
    },
    /**
     * @private
     */
    _vpv: function (page) { //virtual page view
        this._trackGA('event', 'page_view', {
            'page_path': page,
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onViewItem(event, productTrackingInfo) {
        const trackingInfo = {
            'currency': productTrackingInfo['currency'],
            'value': productTrackingInfo['price'],
            'items': [productTrackingInfo],
        };
        this._trackGA('event', 'view_item', trackingInfo);
    },

    /**
     * @private
     */
    _onAddToCart(event, ...productsTrackingInfo) {
        const trackingInfo = {
            'currency': productsTrackingInfo[0]['currency'],
            'value': productsTrackingInfo.reduce((acc, val) => acc + val['price'] * val['quantity'], 0),
            'items': productsTrackingInfo,
        };
        this._trackGA('event', 'add_to_cart', trackingInfo);
    },

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
        if ($('header#top [href="/web/login"]').length) {
            this._vpv('/stats/ecom/customer_signup');
        }
        this._vpv('/stats/ecom/order_checkout');
    },
    /**
     * @private
     */
    _onOrderPayment: function () {
        var method = $('#payment_method input[name=provider]:checked').nextAll('span:first').text();
        this._vpv('/stats/ecom/order_payment/' + method);
    },
});

export default publicWidget.registry.websiteSaleTracking;

```

## File: static\src\js\website_sale_utils.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import wUtils from "@website/js/utils";

export const cartHandlerMixin = {
    getRedirectOption() {
        const html = document.documentElement;
        this.stayOnPageOption = html.dataset.add2cartRedirect === '1';
        this.forceDialog = html.dataset.add2cartRedirect === '2';
    },
    getCartHandlerOptions(ev) {
        this.isBuyNow = ev.currentTarget.classList.contains('o_we_buy_now');
        const targetSelector = ev.currentTarget.dataset.animationSelector || 'img';
        this.$itemImgContainer = this.$(ev.currentTarget).closest(`:has(${targetSelector})`);
    },
    /**
     * Used to add product depending on stayOnPageOption value.
     */
    addToCart(params) {
        if (this.isBuyNow) {
            params.express = true;
        } else if (this.stayOnPageOption) {
            return this._addToCartInPage(params);
        }
        return wUtils.sendRequest('/shop/cart/update', params);
    },
    /**
     * @private
     */
    async _addToCartInPage(params) {
        const data = await this.rpc("/shop/cart/update_json", {
            ...params,
            display: false,
            force_create: true,
        });
        if (data.cart_quantity && (data.cart_quantity !== parseInt($(".my_cart_quantity").text()))) {
            updateCartNavBar(data);
        };
        showCartNotification(this.call.bind(this), data.notification_info);
        return data;
    },
};

function animateClone($cart, $elem, offsetTop, offsetLeft) {
    if (!$cart.length) {
        return Promise.resolve();
    }
    $cart.removeClass('d-none').find('.o_animate_blink').addClass('o_red_highlight o_shadow_animation').delay(500).queue(function () {
        $(this).removeClass("o_shadow_animation").dequeue();
    }).delay(2000).queue(function () {
        $(this).removeClass("o_red_highlight").dequeue();
    });
    return new Promise(function (resolve, reject) {
        if(!$elem) resolve();
        var $imgtodrag = $elem.find('img').eq(0);
        if ($imgtodrag.length) {
            var $imgclone = $imgtodrag.clone()
                .offset({
                    top: $imgtodrag.offset().top,
                    left: $imgtodrag.offset().left
                })
                .removeClass()
                .addClass('o_website_sale_animate')
                .appendTo(document.body)
                .css({
                    // Keep the same size on cloned img.
                    width: $imgtodrag.width(),
                    height: $imgtodrag.height(),
                })
                .animate({
                    top: $cart.offset().top + offsetTop,
                    left: $cart.offset().left + offsetLeft,
                    width: 75,
                    height: 75,
                }, 500);

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
    sessionStorage.setItem('website_sale_cart_quantity', data.cart_quantity);
    $(".my_cart_quantity")
        .parents('li.o_wsale_my_cart').removeClass('d-none').end()
        .toggleClass('d-none', data.cart_quantity === 0)
        .addClass('o_mycart_zoom_animation').delay(300)
        .queue(function () {
            $(this)
                .toggleClass('fa fa-warning', !data.cart_quantity)
                .attr('title', data.warning)
                .text(data.cart_quantity || '')
                .removeClass('o_mycart_zoom_animation')
                .dequeue();
        });

    $(".js_cart_lines").first().before(data['website_sale.cart_lines']).end().remove();
    $("#cart_total").replaceWith(data['website_sale.total']);
    if (data.cart_ready) {
        document.querySelector("a[name='website_sale_main_button']")?.classList.remove('disabled');
    } else {
        document.querySelector("a[name='website_sale_main_button']")?.classList.add('disabled');
    }
}

function showCartNotification(callService, props, options = {}) {
    // Show the notification about the cart
    if (props.lines) {
        callService("cartNotificationService", "add", _t("Item(s) added to your cart"), {
            lines: props.lines,
            currency_id: props.currency_id,
            ...options,
        });
    }
    if (props.warning) {
        callService("cartNotificationService", "add", _t("Warning"), {
            warning: props.warning,
            ...options,
        });
    }
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
                '<button type="button" class="btn-close" data-bs-dismiss="alert"></button> ' +
                '<span></span>' +
            '</div>').prependTo($page);
    }
    cart_alert.children('span:last-child').text(message);
}

export default {
    animateClone: animateClone,
    updateCartNavBar: updateCartNavBar,
    cartHandlerMixin: cartHandlerMixin,
    showCartNotification: showCartNotification,
    showWarning: showWarning,
};

```

## File: static\src\js\website_sale_video_field_preview.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { Component } from "@odoo/owl";

export class FieldVideoPreview extends Component {}
FieldVideoPreview.template = 'website_sale.FieldVideoPreview';

export const fieldVideoPreview = {
    component: FieldVideoPreview,
};

registry.category("fields").add("video_preview", fieldVideoPreview);

```

## File: static\src\js\components\website_sale_image_viewer.js

```javascript
/** @odoo-module **/

import { Dialog } from "@web/core/dialog/dialog";
import { useHotkey } from "@web/core/hotkeys/hotkey_hook";
import { onRendered, useRef, useEffect, useState } from "@odoo/owl";

const ZOOM_STEP = 0.1;

export class ProductImageViewer extends Dialog {
    setup() {
        super.setup();
        this.imageContainerRef = useRef("imageContainer");
        this.images = [...this.props.images].map(image => {
            return {
                src: image.dataset.zoomImage || image.src,
                thumbnailSrc: image.src.replace('/image_1024/', '/image_128/'),
            };
        });
        this.state = useState({
            selectedImageIdx: this.props.selectedImageIdx || 0,
            imageScale: 1,
        });
        this.isDragging = false;
        this.dragStartPos = { x: 0, y: 0 };
        // Doing a full render for the translate is too slow.
        this.imageTranslate = { x: 0, y: 0 };
        useHotkey("arrowleft", this.previousImage.bind(this));
        useHotkey("arrowright", this.nextImage.bind(this));
        useHotkey("r", () => {
            this.imageTranslate = { x: 0, y: 0 };
            this.isDragging = false;
            this.state.imageScale = 1;
            this.updateImage();
        });

        // Not using a t-on-click on purpose because we want to be able to cancel the drag
        // when we go outside of the window.
        useEffect(
            (document) => {
                const onGlobalClick = this.onGlobalClick.bind(this);
                document.addEventListener("click", onGlobalClick);
                return () => {document.removeEventListener("click", onGlobalClick)};
            },
            () => [document],
        );
        // For some reason the styling does not always update properly.
        onRendered(() => {
            this.updateImage();
        })
    }

    get selectedImage() {
        return this.images[this.state.selectedImageIdx];
    }

    set selectedImage(image) {
        this.state.imageScale = 1;
        this.imageTranslate = { x: 0, y: 0 };
        this.state.selectedImageIdx = this.images.indexOf(image);
    }

    get imageStyle() {
        return `transform:
            scale3d(${this.state.imageScale}, ${this.state.imageScale}, 1);
        `;
    }

    get imageContainerStyle() {
        return `transform: translate(${this.imageTranslate.x}px, ${this.imageTranslate.y}px);`;
    }

    previousImage() {
        this.selectedImage = this.images[(this.state.selectedImageIdx - 1 + this.images.length) % this.images.length];
    }

    nextImage() {
        this.selectedImage = this.images[(this.state.selectedImageIdx + 1) % this.images.length];
    }

    updateImage() {
        if (!this.imageContainerRef || !this.imageContainerRef.el) {
            return;
        }
        this.imageContainerRef.el.style = this.imageContainerStyle;
    }

    onGlobalClick(ev) {
        if (ev.target.tagName === "IMG") {
            // Only zoom if the image did not move
            if (this.dragStartPos.clientX === ev.clientX && this.dragStartPos.clientY === ev.clientY) {
                this.zoomIn(ZOOM_STEP * 3);
            }
        }
        if (ev.target.classList.contains('o_wsale_image_viewer_void') && !this.isDragging) {
            ev.stopPropagation();
            ev.preventDefault();
            this.data.close();
        } else {
            this.isDragging = false;
        }
    }

    zoomIn(step=undefined) {
        this.state.imageScale += step || ZOOM_STEP;
    }

    zoomOut(step=undefined) {
        this.state.imageScale = Math.max(0.5, this.state.imageScale - (step || ZOOM_STEP));
    }

    onWheelImage(ev) {
        if (ev.deltaY > 0) {
            this.zoomOut();
        } else {
            this.zoomIn();
        }
    }

    onMousedownImage(ev) {
        this.isDragging = true;
        this.dragStartPos = {
            x: ev.clientX - this.imageTranslate.x,
            y: ev.clientY - this.imageTranslate.y,
            clientX: ev.clientX,
            clientY: ev.clientY,
        };
    }

    onGlobalMousemove(ev) {
        if (!this.isDragging) {
            return;
        }
        this.imageTranslate.x = ev.clientX - this.dragStartPos.x;
        this.imageTranslate.y = ev.clientY - this.dragStartPos.y;
        this.updateImage();
    }
}
ProductImageViewer.props = {
    ...Dialog.props,
    images: { type: NodeList, required: true },
    selectedImageIdx: { type: Number, optional: true },
    close: Function,
};
delete ProductImageViewer.props.slots;
ProductImageViewer.template = "website_sale.ProductImageViewer";

```

## File: static\src\js\components\wysiwyg_adapter\wysiwyg_adapter.js

```javascript
/** @odoo-module **/

import { WysiwygAdapterComponent } from '@website/components/wysiwyg_adapter/wysiwyg_adapter';
import { patch } from "@web/core/utils/patch";
import { markup } from "@odoo/owl";

patch(WysiwygAdapterComponent.prototype, {
    /**
     * @override
     */
    async init() {
        await super.init(...arguments);

        let ribbons = [];
        if (this._isProductListPage()) {
            ribbons = await this.orm.searchRead(
                'product.ribbon',
                [],
                ['id', 'html', 'bg_color', 'text_color', 'html_class'],
            );
        }
        this.ribbons = Object.fromEntries(ribbons.map(ribbon => {
            ribbon.html = markup(ribbon.html);
            return [ribbon.id, ribbon];
        }));
        this.originalRibbons = Object.assign({}, this.ribbons);
        this.productTemplatesRibbons = [];
        this.deletedRibbonClasses = '';
    },
    /**
     * @override
     */
    async _saveViewBlocks() {
        await this._saveRibbons();
        return super._saveViewBlocks(...arguments);
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
            proms.push(this.orm.create(
                'product.ribbon',
                created.map(ribbon => {
                    ribbon = Object.assign({}, ribbon);
                    delete ribbon.id;
                    return ribbon;
                }),
            ).then(ids => createdRibbonIds = ids));
        }

        modified.forEach(ribbon => proms.push(this.orm.write(
            'product.ribbon',
            [ribbon.id],
            ribbon,
        )));

        if (deletedIds.length > 0) {
            proms.push(this.orm.unlink(
                'product.ribbon',
                deletedIds,
            ));
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
            }).map(([ribbonId, templateIds]) => this.orm.write(
                'product.template',
                templateIds,
                {'website_ribbon_id': localToServer[ribbonId].id},
            ));
        return Promise.all(setProductTemplateRibbons);
    },
    /**
     * Checks whether the current page is the product list.
     *
     * @private
     */
    _isProductListPage() {
        return this.options.editable && this.options.editable.find('#products_grid').length !== 0;
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
    /**
     * @override
     */
    _trigger_up(ev) {
        const methods = {
            get_ribbons: this._onGetRibbons.bind(this),
            get_ribbon_classes: this._onGetRibbonClasses.bind(this),
            delete_ribbon: this._onDeleteRibbon.bind(this),
            set_ribbon: this._onSetRibbon.bind(this),
            set_product_ribbon: this._onSetProductRibbon.bind(this),
        }
        if (methods[ev.name]) {
            return methods[ev.name](ev);
        } else {
            return super._trigger_up(...arguments);
        }
    },
    // TODO this whole patch actually seems unnecessary. The bug it solved seems
    // to stay solved if this is removed. To investigate.
    /**
     * @override
     */
     _getContentEditableAreas() {
        const array = super._getContentEditableAreas(...arguments);
        return array.filter(el => {
            // TODO should really review this system of "ContentEditableAreas +
            // ReadOnlyAreas", here the "products_header" stuff is duplicated in
            // both but this system is also duplicated with o_not_editable and
            // maybe even other systems (like preserving contenteditable="false"
            // with oe-keep-contenteditable).
            return !el.closest('.oe_website_sale .products_header');
        });
    },
    /**
     * @override
     */
    _getReadOnlyAreas() {
        const readOnlyEls = super._getReadOnlyAreas(...arguments);
        return [...readOnlyEls].concat(
            $(this.websiteService.pageDocument).find("#wrapwrap").find('.oe_website_sale .products_header, .oe_website_sale .products_header a').toArray()
        );
    },
});

```

## File: static\src\js\notification\notification_service.js

```javascript
/** @odoo-module **/

import { xml } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { notificationService } from "@web/core/notifications/notification_service";
import { NotificationContainer } from "@web/core/notifications/notification_container";
import { CartNotification } from "@website_sale/js/notification/cart_notification/cart_notification";


export class CartNotificationContainer extends NotificationContainer {
    static components = {
        ...NotificationContainer.components,
        Notification: CartNotification,
    }
    static template = xml`
    <div class="position-absolute w-100 h-100 top-0 pe-none">
        <div class="d-flex flex-column container align-items-end">
            <t t-foreach="notifications" t-as="notification" t-key="notification">
                <Transition leaveDuration="0" name="'o_notification_fade'" t-slot-scope="transition">
                    <Notification t-props="notification_value.props" className="(notification_value.props.className || '') + ' ' + transition.className"/>
                </Transition>
            </t>
        </div>
    </div>`;
}

export const cartNotificationService = {
    ...notificationService,
    notificationContainer: CartNotificationContainer,
}

registry.category("services").add("cartNotificationService", cartNotificationService);

```

## File: static\src\js\notification\add_to_cart_notification\add_to_cart_notification.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { formatCurrency } from "@web/core/currency";

export class AddToCartNotification extends Component {
    static template = "website_sale.addToCartNotification";
    static props = {
        lines: {
            type: Array,
            element: {
                type: Object,
                shape: {
                    id: Number,
                    image_url: String,
                    quantity: Number,
                    name: String,
                    description: { type: String, optional: true },
                    line_price_total: Number,
                },
            },
        },
        currency_id: Number,
    }

    /**
     * Return the price, in the format of the sale order currency.
     *
     * @param {Object} line - The line element for which to return the formatted price.
     * @return {String} - The price, in the format of the sale order currency.
     */
    getFormattedPrice(line) {
        return formatCurrency(line.line_price_total, this.props.currency_id);
    }

    /**
     * Return the product summary based on the line information.
     *
     * The product summary is computed based on the line quantity and name, separated by the symbol
     * 'x' (e.g.: 1 x Chair Floor Protection).
     *
     * @param {Object} line - The line element for which to return the product summary.
     * @return {String} - The product summary.
     */
    getProductSummary(line) {
        return line.quantity + " x " + line.name;
    }
}

```

## File: static\src\js\notification\add_to_cart_notification\add_to_cart_notification.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_sale.addToCartNotification">
        <div class="row g-2 mb-2" t-foreach="props.lines" t-as="line" t-key="line.id">
            <div class="col-3">
                <img class="img o_image_64_max rounded mb-2 img-fluid"
                    t-att-src="line.image_url"
                    t-att-alt="line.name"/>
            </div>
            <div class="col-6 d-flex flex-column align-items-start">
                <span t-out="getProductSummary(line)"/>
                <span class="text-muted small"
                    t-if="line.description"
                    t-out="line.description"/>
            </div>
            <div class="col-3 d-flex flex-column align-items-end gap-1"
                t-out="getFormattedPrice(line)"/>
        </div>
        <a role="button" class="w-100 btn btn-primary" href="/shop/cart">
            View cart
        </a>
    </t>

</templates>

```

## File: static\src\js\notification\cart_notification\cart_notification.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { AddToCartNotification } from "../add_to_cart_notification/add_to_cart_notification";
import { WarningNotification } from "../warning_notification/warning_notification";

export class CartNotification extends Component {
    static components = { AddToCartNotification, WarningNotification };
    static template = "website_sale.cartNotification";
    static props = {
        message: [String, { toString: Function }],
        warning: {type : [String, { toString: Function }],optional: true},
        lines: {
            type: Array,
            optional: true,
            element: {
                type: Object,
                shape: {
                    id: Number,
                    image_url: String,
                    quantity: Number,
                    name: String,
                    description: { type: String, optional: true },
                    line_price_total: Number,
                },
            },
        },
        currency_id: {type: Number, optional: true},
        className: String,
        close: Function,
        refresh: Function,
        freeze: Function,
    }

    /**
     * Get the top position (in px) of the notification based on the navbar height.
     *
     * This prevents the notification from being shown in front of the navbar.
     */
    get positionOffset() {
        return (document.querySelector('header.o_top_fixed_element')?.offsetHeight || 0) + 'px';
    }
}

```

## File: static\src\js\notification\cart_notification\cart_notification.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_sale.cartNotification">
        <div t-attf-class="toast show o_cc1 position-relative start-0 mt-2 {{props.className}}"
             t-attf-style="top: {{positionOffset}};"
             role="alert"
             aria-live="assertive"
             aria-atomic="true">
            <div class="toast-header justify-content-between">
                <strong t-if="props.message" t-out="props.message"/>
                <button class="btn-close"
                        type="button"
                        t-on-click="props.close"
                        aria-label="Close"/>
            </div>
            <div class="toast-body">
                <WarningNotification t-if="this.props.warning" warning="this.props.warning"/>
                <AddToCartNotification
                    t-elif="this.props.lines.length"
                    lines="this.props.lines"
                    currency_id="this.props.currency_id"/>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\js\notification\warning_notification\warning_notification.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";

export class WarningNotification extends Component {
    static template = "website_sale.warningNotification";
    static props = {
        warning: [String, { toString: Function }],
    }
}

```

## File: static\src\js\notification\warning_notification\warning_notification.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_sale.warningNotification">
        <div class="alert alert-warning" role="alert">
            <i class="fa fa-warning"/>
            <t t-out="this.props.warning"/>
        </div>
    </t>

</templates>

```

## File: static\src\js\systray_items\new_content.js

```javascript
/** @odoo-module **/

import { NewContentModal, MODULE_STATUS } from '@website/systray_items/new_content';
import { patch } from "@web/core/utils/patch";

patch(NewContentModal.prototype, {
    setup() {
        super.setup();

        const newProductElement = this.state.newContentElements.find(element => element.moduleXmlId === 'base.module_website_sale');
        newProductElement.createNewContent = () => this.onAddContent(
            'website_sale.product_product_action_add',
            true,
            {default_is_published: true});
        newProductElement.status = MODULE_STATUS.INSTALLED;
        newProductElement.model = 'product.product';
    },
});

```

## File: static\src\js\tours\tour_utils.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import wTourUtils from "@website/js/tours/tour_utils";

function addToCart({productName, search = true, productHasVariants = false}) {
    const steps = [];
    if (search) {
        steps.push(...searchProduct(productName));
    }
    steps.push(wTourUtils.clickOnElement(productName, `a:contains(${productName})`));
    steps.push(wTourUtils.clickOnElement('Add to cart', '#add_to_cart'));
    if (productHasVariants) {
        steps.push(wTourUtils.clickOnElement('Continue Shopping', 'button:contains("Continue Shopping")'));
    }
    return steps;
}

function assertCartAmounts({taxes = false, untaxed = false, total = false, delivery = false}) {
    let steps = [];
    if (taxes) {
        steps.push({
            content: 'Check if the tax is correct',
            trigger: `tr#order_total_taxes .oe_currency_value:containsExact(${taxes})`,
            run: function () {},  // it's a check
        });
    }
    if (untaxed) {
        steps.push({
            content: 'Check if the tax is correct',
            trigger: `tr#order_total_untaxed .oe_currency_value:containsExact(${untaxed})`,
            run: function () {},  // it's a check
        });
    }
    if (total) {
        steps.push({
            content: 'Check if the tax is correct',
            trigger: `tr#order_total .oe_currency_value:containsExact(${total})`,
            run: function () {},  // it's a check
        });
    }
    if (delivery) {
        steps.push({
            content: 'Check if the tax is correct',
            trigger: `tr#order_delivery .oe_currency_value:containsExact(${delivery})`,
            run: function () {},  // it's a check
        });
    }
    return steps
}

function assertCartContains({productName, backend, notContains = false} = {}) {
    let trigger = `a:contains(${productName})`;

    if (notContains) {
        trigger = `:not(${trigger})`;
    }
    return {
        content: `Checking if ${productName} is in the cart`,
        trigger: `${backend ? "iframe" : ""} ${trigger}`,
        run: () => {}
    };
}

/**
 * Used to assert if the price attribute of a given product is correct on the /shop view
 */
function assertProductPrice(attribute, value, productName) {
    return {
        content: `The ${attribute} of the ${productName} is ${value}`,
        trigger: `div:contains("${productName}") [data-oe-expression="template_price_vals['${attribute}']"] .oe_currency_value:contains("${value}")`,
        run: () => {}
    };
}

function fillAdressForm(adressParams = {
    name: "John Doe",
    phone: "123456789",
    email: "johndoe@gmail.com",
    street: "1 rue de la paix",
    city: "Paris",
    zip: "75000"
}) {
    let steps = [];
    steps.push({
        content: "Address filling",
        trigger: 'select[name="country_id"]',
        run: () => {
            $('input[name="name"]').val(adressParams.name);
            $('input[name="phone"]').val(adressParams.phone);
            $('input[name="email"]').val(adressParams.email);
            $('input[name="street"]').val(adressParams.street);
            $('input[name="city"]').val(adressParams.city);
            $('input[name="zip"]').val(adressParams.zip);
            $('#country_id option:eq(1)').attr('selected', true);
        }
    });
    steps.push({
        content: "Continue checkout",
        trigger: '.oe_cart .btn:contains("Continue checkout")',
    });
    return steps;
}

function goToCart({quantity = 1, position = "bottom", backend = false} = {}) {
    return {
        content: _t("Go to cart"),
        trigger: `${backend ? "iframe" : ""} a sup.my_cart_quantity:containsExact(${quantity})`,
        position: position,
        run: "click",
    };
}

function goToCheckout() {
    return {
        content: 'Checkout your order',
        trigger: 'a[href^="/shop/checkout"]',
        run: 'click',
    };
}

function pay() {
    return {
        content: 'Pay',
        //Either there are multiple payment methods, and one is checked, either there is only one, and therefore there are no radio inputs
        // extra_trigger: '#payment_method input:checked,#payment_method:not(:has("input:radio:visible"))',
        trigger: 'button[name="o_payment_submit_button"]:visible:not(:disabled)'
    };
}

function payWithDemo() {
    return [{
        content: 'eCommerce: select Test payment provider',
        trigger: 'input[name="o_payment_radio"][data-payment-method-code="demo"]'
    }, {
        content: 'eCommerce: add card number',
        trigger: 'input[name="customer_input"]',
        run: 'text 4242424242424242'
    },
    pay(),
    {
        content: 'eCommerce: check that the payment is successful',
        trigger: '.oe_website_sale_tx_status:contains("Your payment has been successfully processed.")',
        run: function () {}
    }]
}

function payWithTransfer(redirect=false) {
    const first_step = {
        content: "Select `Wire Transfer` payment method",
        trigger: 'input[name="o_payment_radio"][data-payment-method-code="wire_transfer"]',
    }
    if (!redirect) {
        return [
        first_step,
        pay(),
        {
            content: "Last step",
            trigger: '.oe_website_sale_tx_status:contains("Please use the following transfer details")',
            timeout: 30000,
            isCheck: true,
        }]
    } else {
        return [
            first_step,
            pay(),
            {
                content: "Last step",
                trigger: '.oe_website_sale_tx_status:contains("Please use the following transfer details")',
                timeout: 30000,
                run: () => {
                    window.location.href = '/contactus'; // Redirect in JS to avoid the RPC loop (20x1sec)
                },
            }, {
                content: "wait page loaded",
                trigger: 'h1:contains("Contact us")',
                run: function () {}, // it's a check
            }
        ]
    }
}

function searchProduct(productName) {
    return [
        wTourUtils.clickOnElement('Shop', 'a:contains("Shop")'),
        {
            content: "Search for the product",
            trigger: 'form input[name="search"]',
            run: `text ${productName}`
        },
        wTourUtils.clickOnElement('Search', 'form:has(input[name="search"]) .oe_search_button'),
    ];
}

/**
 * Used to select a pricelist on the /shop view
 */
function selectPriceList(pricelist) {
    return [
        {
            content: "Click on pricelist dropdown",
            trigger: "div.o_pricelist_dropdown a[data-bs-toggle=dropdown]",
        },
        {
            content: "Click on pricelist",
            trigger: `span:contains(${pricelist})`,
        },
    ];
}

export default {
    addToCart,
    assertCartAmounts,
    assertCartContains,
    assertProductPrice,
    fillAdressForm,
    goToCart,
    goToCheckout,
    pay,
    payWithDemo,
    payWithTransfer,
    selectPriceList,
    searchProduct,
};

```

## File: static\src\js\tours\website_sale_shop.js

```javascript
/** @odoo-module **/

    import { _t } from "@web/core/l10n/translation";
    import wTourUtils from "@website/js/tours/tour_utils";

    import { markup } from "@odoo/owl";

    wTourUtils.registerWebsitePreviewTour("shop", {
        url: '/shop',
        sequence: 130,
    }, () => [{
        trigger: ".o_menu_systray .o_new_content_container > a",
        content: _t("Let's create your first product."),
        extra_trigger: "iframe .js_sale",
        consumeVisibleOnly: true,
        position: "bottom",
    }, {
        trigger: "a[data-module-xml-id='base.module_website_sale']",
        content: markup(_t("Select <b>New Product</b> to create it and manage its properties to boost your sales.")),
        position: "bottom",
    }, {
        trigger: ".modal-dialog input[type=text]",
        content: _t("Enter a name for your new product"),
        position: "left",
    }, {
        trigger: ".modal-footer button.btn-primary",
        content: markup(_t("Click on <em>Save</em> to create the product.")),
        position: "right",
    }, {
        trigger: "iframe .product_price .oe_currency_value:visible",
        extra_trigger: "#oe_snippets.o_loaded",
        content: _t("Edit the price of this product by clicking on the amount."),
        position: "bottom",
        run: "text 1.99",
        timeout: 30000,
    }, {
        trigger: "iframe #wrap img.product_detail_img",
        extra_trigger: "iframe .product_price .o_dirty .oe_currency_value:not(:containsExact(1.00))",
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
        auto: true,
    },
    wTourUtils.goBackToBlocks(),
    {
        trigger: "#snippet_structure .oe_snippet:eq(3) .oe_snippet_thumbnail",
        extra_trigger: "body:not(.modal-open)",
        content: _t("Drag this website block and drop it in your page."),
        position: "bottom",
        run: "drag_and_drop_native iframe #wrapwrap > main",
    }, {
        trigger: "button[data-action=save]",
        content: markup(_t("Once you click on <b>Save</b>, your product is updated.")),
        position: "bottom",
        // Wait until the drag and drop is resolved (causing a history step)
        // before clicking save.
        extra_trigger: ".o_we_external_history_buttons button[data-action=undo]:not([disabled])",
    }, {
        trigger: ".o_menu_systray_item .o_switch_danger_success",
        extra_trigger: "iframe body:not(.editor_enable)",
        content: _t("Click on this button so your customers can see it."),
        position: "bottom",
    }, {
        trigger: "button[data-menu-xmlid='website.menu_reporting']",
        content: _t("Click here to open the reporting menu"),
        position: "bottom",
    }, {
        trigger: "a[data-menu-xmlid='website.menu_website_dashboard'], a[data-menu-xmlid='website.menu_website_analytics']",
        content: _t("Let's now take a look at your eCommerce dashboard to get your eCommerce website ready in no time."),
        position: "bottom",
        // Just check during test mode. Otherwise, clicking it will result to random error on loading the Chart.js script.
        run: () => {},
    }]);

```

## File: static\src\snippets\s_add_to_cart\000.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { cartHandlerMixin } from '@website_sale/js/website_sale_utils';
import { WebsiteSale } from '@website_sale/js/website_sale';
import { _t } from "@web/core/l10n/translation";

publicWidget.registry.AddToCartSnippet = WebsiteSale.extend(cartHandlerMixin, {
    selector: '.s_add_to_cart_btn',
    events: {
        'click': '_onClickAddToCartButton',
    },

    init() {
        this._super(...arguments);
        this.notification = this.bindService("notification");
    },

    _onClickAddToCartButton: async function (ev) {
        const dataset = ev.currentTarget.dataset;

        const visitorChoice = dataset.visitorChoice === 'true';
        const action = dataset.action;
        const productId = parseInt(dataset.productVariantId);

        if (!productId) {
            return;
        }

        if (visitorChoice) {
            this._handleAdd($(ev.currentTarget.closest('div')));
        } else {
            const isAddToCartAllowed = await this.rpc(`/shop/product/is_add_to_cart_allowed`, {
                product_id: productId,
            });
            if (!isAddToCartAllowed) {
                this.notification.add(
                    _t('This product does not exist therefore it cannot be added to cart.'),
                    { title: 'User Error', type: 'warning' }
                );
                return;
            }
            this.isBuyNow = action === 'buy_now';
            this.stayOnPageOption = !this.isBuyNow;
            this.addToCart({product_id: productId, add_qty: 1});
        }
    },
});

export default publicWidget.registry.AddToCartSnippet;

```

## File: static\src\snippets\s_add_to_cart\options.js

```javascript
/** @odoo-module **/

import options from '@web_editor/js/editor/snippets.options';
import { _t } from "@web/core/l10n/translation";

const Many2oneUserValueWidget = options.userValueWidgetsRegistry['we-many2one'];

const Many2oneDefaultMessageWidget = Many2oneUserValueWidget.extend({

    // defaultMessage: default message to display when no records are selected
    configAttributes: [...Many2oneUserValueWidget.prototype.configAttributes, 'defaultMessage'],

    /**
     * @override
     */
    async setValue(value, methodName) {
        await this._super(...arguments);

        if (value === '') {
            this.menuTogglerEl.textContent = this.options.defaultMessage;
        }
    },
});

options.userValueWidgetsRegistry['we-many2one-default-message'] = Many2oneDefaultMessageWidget;

options.registry.AddToCart = options.Class.extend({
    events: Object.assign({}, options.Class.prototype.events || {}, {
        'click .reset-variant-picker': '_onClickResetVariantPicker',
        'click .reset-product-picker': '_onClickResetProductPicker',
    }),

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    async updateUI() {
        if (this.rerender) {
            this.rerender = false;
            await this._rerenderXML();
            return;
        }
        return this._super.apply(this, arguments);
    },

    _setButtonDisabled: function (isDisabled) {
        const buttonEl = this._buttonEl();

        if (isDisabled) {
            buttonEl.classList.add('disabled');
        } else {
            buttonEl.classList.remove('disabled');
        }
    },

    async setProductTemplate(previewMode, widgetValue, params) {
        this.$target[0].dataset.productTemplate = widgetValue;
        this._resetVariantChoice();
        this._resetAction();
        this._setButtonDisabled(false);

        await this._fetchVariants(widgetValue);
        this.rerender = true;
        this._updateButton();

    },

    setProductVariant(previewMode, widgetValue, params) {
        this.$target[0].dataset.productVariant = widgetValue;
        this._updateButton();
    },

    setAction(previewMode, widgetValue, params) {
        this.$target[0].dataset.action = widgetValue;
        this._updateButton();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _onClickResetVariantPicker() {
        this._resetVariantChoice();
        this._resetAction();
        this._updateButton();
    },

    _onClickResetProductPicker() {
        this._resetProductChoice();
        this._resetVariantChoice();
        this._resetAction();
        this._updateButton();
    },

    /**
     * Fetches the variants ids from the server
     */
    async _fetchVariants(productTemplateId) {
        const response = await this.orm.searchRead(
            "product.product", [["product_tmpl_id", "=", parseInt(productTemplateId)]], ["id"]
        );
        this.$target[0].dataset.variants = response.map(variant => variant.id);
    },


    _resetProductChoice() {
        this.$target[0].dataset.productTemplate = '';
        this._buttonEl().classList.add('disabled');
    },


    _resetVariantChoice() {
        this.$target[0].dataset.productVariant = '';
    },

    _resetAction: function () {
        this.$target[0].dataset.action = "add_to_cart";
    },

    /**
     * Returns an array of variant ids from the dom
     */
    _variantIds() {
        return this.$target[0].dataset.variants.split(',').map(stringId => parseInt(stringId));
    },

    _buttonEl() {
        const buttonEl = this.$target[0].querySelector('.s_add_to_cart_btn');
        // In case the button was deleted somehow, we rebuild it.
        if (!buttonEl) {
            return this._buildButtonEl();
        }
        return buttonEl;
    },

    _buildButtonEl() {
        const buttonEl = document.createElement('button');
        buttonEl.classList.add("s_add_to_cart_btn", "btn", "btn-secondary", "mb-2");
        this.$target[0].append(buttonEl);
        return buttonEl;
    },

    /**
     * Updates the button's html
     */
    _updateButton() {
        const variantIds = this._variantIds();
        const buttonEl = this._buttonEl();

        let productVariantId = variantIds[0];
        buttonEl.dataset.visitorChoice = false;

        if (variantIds.length > 1) {
            // If there is more than 1 variant, that means that there are variants for the product template
            // and we check if there is one selected and assign it. If not, visitorChoice is set to true
            if (this.$target[0].dataset.productVariant) {
                productVariantId = this.$target[0].dataset.productVariant;
            } else {
                buttonEl.dataset.visitorChoice = true;
            }
        }
        buttonEl.dataset.productVariantId = productVariantId;
        buttonEl.dataset.action = this.$target[0].dataset.action;
        this._updateButtonContent();
        this._createHiddenFormInput(productVariantId);
    },

    _updateButtonContent() {
        let iconEl = document.createElement('i');
        const buttonContent = {
            add_to_cart: {classList: "fa fa-cart-plus me-2", text: _t("Add to Cart")},
            buy_now: {classList: "fa fa-credit-card me-2", text: _t("Buy now")},
        };
        let buttonContentElement = buttonContent[this.$target[0].dataset.action];

        iconEl.classList = buttonContentElement.classList;

        this._buttonEl().replaceChildren(iconEl, buttonContentElement.text);
    },
    /**
     * Because sale_product_configurator._handleAdd() requires a hidden input to retrieve the productId,
     * this method creates a hidden input in the form of the button to make the modal behaviour possible.
     */
    _createHiddenFormInput(productVariantId) {
        const inputEl = this._buttonEl().querySelector('input[type="hidden"][name="product_id"]');
        if (inputEl) {
            // If the input already exists, we change its value
            inputEl.setAttribute('value', productVariantId);
        } else {
            // Otherwise, we create the input element
            let inputEl = document.createElement('input');
            inputEl.setAttribute('type', 'hidden');
            inputEl.setAttribute('name', 'product_id');
            inputEl.setAttribute('value', productVariantId);
            this._buttonEl().append(inputEl);
        }
    },

    /**
     * Called when the template is chosen and that we want to update the m2o variant widget with the right variants.
     */
    async _renderCustomXML(uiFragment) {
        if (this.$target[0].dataset.productTemplate) {
            // That means that a template was selected and we want to update the content of the variant picker based on the template id
            const productVariantPickerEl = uiFragment.querySelector('we-many2one-default-message[data-name="product_variant_picker_opt"]');
            productVariantPickerEl.dataset.domain = `[["product_tmpl_id", "=", ${this.$target[0].dataset.productTemplate}]]`;
        }
    },

    /**
     * @override
     */
    _computeWidgetState(methodName, params) {
        switch (methodName) {
            case 'setProductTemplate': {
                return this.$target[0].dataset.productTemplate || '';
            }
            case 'setProductVariant': {
                return this.$target[0].dataset.productVariant || '';
            }
            case 'setAction': {
                return this.$target[0].dataset.action;
            }
        }
        return this._super(...arguments);
    },

    /**
     * @override
     */
    async _computeWidgetVisibility(widgetName, params) {
        switch (widgetName) {
            case 'product_variant_picker_opt': {
                return this.$target[0].dataset.productTemplate && this._variantIds().length > 1;
            }
            case 'product_variant_reset_opt': {
                return this.$target[0].dataset.productVariant;
            }

            case 'product_template_reset_opt': {
                return this.$target[0].dataset.productTemplate;
            }
            case 'action_picker_opt': {
                if (this.$target[0].dataset.productTemplate) {
                    if (this._variantIds().length > 1) {
                        return this.$target[0].dataset.productVariant;
                    }
                    return true;
                }
                return false;
            }
        }
        return this._super(...arguments);
    },
});
export default {
    AddToCart: options.registry.AddToCart,
};

```

## File: static\src\snippets\s_dynamic_snippet_products\000.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import DynamicSnippetCarousel from "@website/snippets/s_dynamic_snippet_carousel/000";
import wSaleUtils from "@website_sale/js/website_sale_utils";

const DynamicSnippetProducts = DynamicSnippetCarousel.extend({
    selector: '.s_dynamic_snippet_products',

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Gets the category search domain
     *
     * @private
     */
    _getCategorySearchDomain() {
        const searchDomain = [];
        let productCategoryId = this.$el.get(0).dataset.productCategoryId;
        if (productCategoryId && productCategoryId !== 'all') {
            if (productCategoryId === 'current') {
                productCategoryId = undefined;
                const productCategoryField = $("#product_details").find(".product_category_id");
                if (productCategoryField && productCategoryField.length) {
                    productCategoryId = parseInt(productCategoryField[0].value);
                }
                if (!productCategoryId) {
                    this.trigger_up('main_object_request', {
                        callback: function (value) {
                            if (value.model === "product.public.category") {
                                productCategoryId = value.id;
                            }
                        },
                    });
                }
                if (!productCategoryId) {
                    // Try with categories from product, unfortunately the category hierarchy is not matched with this approach
                    const productTemplateId = $("#product_details").find(".product_template_id");
                    if (productTemplateId && productTemplateId.length) {
                        searchDomain.push(['public_categ_ids.product_tmpl_ids', '=', parseInt(productTemplateId[0].value)]);
                    }
                }
            }
            if (productCategoryId) {
                searchDomain.push(['public_categ_ids', 'child_of', parseInt(productCategoryId)]);
            }
        }
        return searchDomain;
    },
    /**
     * Gets the tag search domain
     *
     * @private
     */
    _getTagSearchDomain() {
        const searchDomain = [];
        let productTagIds = this.$el.get(0).dataset.productTagIds;
        productTagIds = productTagIds ? JSON.parse(productTagIds) : [];
        if (productTagIds.length) {
            searchDomain.push(['all_product_tag_ids', 'in', productTagIds.map(productTag => productTag.id)]);
        }
        return searchDomain;
    },
    /**
     * Method to be overridden in child components in order to provide a search
     * domain if needed.
     * @override
     * @private
     */
    _getSearchDomain: function () {
        const searchDomain = this._super.apply(this, arguments);
        searchDomain.push(...this._getCategorySearchDomain());
        searchDomain.push(...this._getTagSearchDomain());
        const productNames = this.$el.get(0).dataset.productNames;
        if (productNames) {
            const nameDomain = [];
            for (const productName of productNames.split(',')) {
                // Ignore empty names
                if (!productName.length) {
                    continue;
                }
                // Search on name, internal reference and barcode.
                if (nameDomain.length) {
                    nameDomain.unshift('|');
                }
                nameDomain.push(...[
                    '|', '|', ['name', 'ilike', productName],
                              ['default_code', '=', productName],
                              ['barcode', '=', productName],
                ]);
            }
            searchDomain.push(...nameDomain);
        }
        return searchDomain;
    },
    /**
     * Add `productTemplateId` for product snippets (Accessories, Alternatives and Recently sold).
     *
     * See `dynamic_snippet_accessories_action`, `dynamic_snippet_recently_sold_with_action` and
     * `dynamic_snippet_alternative_products`.
     *
     * @override
     * @private
     */
    _getRpcParameters: function () {
        const productTemplateId = $("#product_details").find(".product_template_id");
        return Object.assign(this._super.apply(this, arguments), {
            productTemplateId: productTemplateId && productTemplateId.length ? productTemplateId[0].value : undefined,
        });
    },
});

const DynamicSnippetProductsCard = publicWidget.Widget.extend({
    selector: '.o_carousel_product_card',
    read_events: {
        'click .js_add_cart': '_onClickAddToCart',
        'click .js_remove': '_onRemoveFromRecentlyViewed',
    },

    init(root, options) {
        const parent = options.parent || root;
        this._super(parent, options);
        this.rpc = this.bindService("rpc");
    },

    start() {
        this.add2cartRerender = this.el.dataset.add2cartRerender === 'True';
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Event triggered by a click on the Add to cart button
     *
     * @param {OdooEvent} ev
     */
    async _onClickAddToCart(ev) {
        const $card = $(ev.currentTarget).closest('.card');
        const data = await this.rpc("/shop/cart/update_json", {
            product_id: $card.find('input[data-product-id]').data('product-id'),
            add_qty: 1,
            display: false,
        });
        wSaleUtils.updateCartNavBar(data);
        wSaleUtils.showCartNotification(this.call.bind(this), data.notification_info);
        if (this.add2cartRerender) {
            this.trigger_up('widgets_start_request', {
                $target: this.$el.closest('.s_dynamic'),
            });
        }
    },
    /**
     * Event triggered by a click on the remove button on a "recently viewed"
     * template.
     *
     * @param {OdooEvent} ev
     */
    async _onRemoveFromRecentlyViewed(ev) {
        const $card = $(ev.currentTarget).closest('.card');
        await this.rpc("/shop/products/recently_viewed_delete", {
            product_id: $card.find('input[data-product-id]').data('product-id'),
        });
        this.trigger_up('widgets_start_request', {
            $target: this.$el.closest('.s_dynamic'),
        });
    },
});

publicWidget.registry.dynamic_snippet_products_cta = DynamicSnippetProductsCard;
publicWidget.registry.dynamic_snippet_products = DynamicSnippetProducts;

export default DynamicSnippetProducts;

```

## File: static\src\snippets\s_dynamic_snippet_products\options.js

```javascript
/** @odoo-module **/

import options from "@web_editor/js/editor/snippets.options";
import s_dynamic_snippet_carousel_options from "@website/snippets/s_dynamic_snippet_carousel/options";

import wUtils from "@website/js/utils";

const alternativeSnippetRemovedOptions = [
    'filter_opt', 'product_category_opt', 'product_tag_opt', 'product_names_opt',
]

const dynamicSnippetProductsOptions = s_dynamic_snippet_carousel_options.extend({

    /**
     *
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.modelNameFilter = 'product.product';
        // Directly calling $() will not work in this case since we are querying something
        // in an iframe
        const productTemplateId = this.$target.closest("#wrapwrap").find("input.product_template_id");
        this.hasProductTemplateId = productTemplateId.val();
        if (!this.hasProductTemplateId) {
            this.contextualFilterDomain.push(['product_cross_selling', '=', false]);
        }
        this.productCategories = {};
        this.isAlternativeProductSnippet = this.$target.hasClass('o_wsale_alternative_products');

        this.orm = this.bindService("orm");
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
     _computeWidgetVisibility(widgetName, params) {
        if (this.isAlternativeProductSnippet && alternativeSnippetRemovedOptions.includes(widgetName)) {
            return false;
        }
        return this._super(...arguments);
    },
    /**
     * Fetches product categories.
     * @private
     * @returns {Promise}
     */
    _fetchProductCategories: function () {
        return this.orm.searchRead("product.public.category", wUtils.websiteDomain(this), ["id", "name"]);
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
     * @override
     * @private
     */
    _setOptionsDefaultValues: function () {
        this._setOptionValue('productCategoryId', 'all');
        this._super.apply(this, arguments);
    },
});

options.registry.dynamic_snippet_products = dynamicSnippetProductsOptions;

export default dynamicSnippetProductsOptions;

```

## File: static\src\snippets\s_popup\000.js

```javascript
/** @odoo-module **/

import PopupWidget from '@website/snippets/s_popup/000';

PopupWidget.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Checks if the given primary button should allow or not to close the
     * modal.
     *
     * @override
     */
    _canBtnPrimaryClosePopup(primaryBtnEl) {
        return (
            this._super(...arguments)
            && !primaryBtnEl.classList.contains("js_add_cart")
        );
    },
});

export default PopupWidget;

```

## File: static\src\xml\website_sale.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="website_sale.FieldVideoPreview">
        <div class="ratio ratio-16x9 mt-2" t-if="props.record.data[props.name]">
            <t t-out="props.record.data[props.name]"/>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_sale_image_viewer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="website_sale.ProductImageViewer">
        <div class="o_dialog" t-att-id="id" t-att-class="{ o_inactive_modal: !data.isActive }">
            <div role="dialog" class="modal" t-ref="modalRef">
                <div class="o_wsale_image_viewer flex-column align-items-center d-flex w-100 h-100" t-on-mousemove="onGlobalMousemove">
                    <!-- Header -->
                    <div class="o_wsale_image_viewer_header d-flex w-100 text-white">
                        <div class="flex-grow-1"/>
                        <div class="d-flex align-items-center mb-0 px-3 h4 text-reset cursor-pointer">
                            <span class="fa fa-times" t-on-click="data.close"/>
                        </div>
                    </div>
                    <!-- Content -->
                    <div class="o_wsale_image_viewer_image position-absolute top-0 bottom-0 start-0 end-0 align-items-center justify-content-center d-flex o_with_img overflow-hidden">
                        <div class="o_wsale_image_viewer_void position-absolute align-items-center justify-content-center d-flex w-100 h-100" t-ref="imageContainer" t-att-style="imageContainerStyle">
                            <img
                                alt="Viewer"
                                class="mw-100 mh-100 transition-base"
                                draggable="false"
                                t-att-src="selectedImage.src"
                                t-att-style="imageStyle"
                                t-on-mousedown="onMousedownImage"
                                t-on-wheel.stop="onWheelImage"
                            />
                        </div>
                    </div>
                    <t t-if="images.length > 1">
                        <!-- Footer -->
                        <div class="o_wsale_image_viewer_carousel position-absolute bottom-0 d-flex" role="toolbar">
                            <ol class="d-flex justify-content-start ps-0 pt-2 pt-lg-0 mx-auto my-0 text-start">
                                <t t-foreach="images" t-as="image" t-key="image.thumbnailSrc">
                                    <li t-attf-class="align-top position-relative px-1 pb-1 {{image === selectedImage ? 'active' : ''}}" t-on-click="() => this.selectedImage = image">
                                        <div>
                                            <img t-att-src="image.thumbnailSrc" t-attf-class="img o_wsale_image_viewer_thumbnail {{image === selectedImage ? 'active' : ''}}" t-att-alt="props.title" loading="lazy"/>
                                        </div>
                                    </li>
                                </t>
                            </ol>
                        </div>
                        <!-- Controls -->
                        <div class="o_wsale_image_viewer_control o_wsale_image_viewer_previous btn btn-dark position-absolute top-0 bottom-0 start-0 align-items-center justify-content-center d-flex my-auto ms-3 rounded-circle" t-on-click="previousImage" title="Previous (Left-Arrow)" role="button">
                            <span class="oi oi-chevron-left" role="img"/>
                        </div>
                        <div class="o_wsale_image_viewer_control o_wsale_image_viewer_next btn btn-dark position-absolute top-0 bottom-0 end-0 align-items-center justify-content-center d-flex my-auto me-3 rounded-circle" t-on-click="nextImage" title="Next (Right-Arrow)" role="button">
                            <span class="oi oi-chevron-right" role="img"/>
                        </div>
                    </t>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website_sale_reorder_modal.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="website_sale.ReorderModal">
        <t t-set="reorder">Re-Order</t>
        <Dialog title="reorder">
            <table id="o_wsale_reorder_table" class="table table-sm">
                <thead class="bg-100">
                    <tr id="o_wsale_reorder_header">
                        <!-- Product Image -->
                        <th class="text-start td-img">Product</th>
                        <!-- Product name + description -->
                        <th/>
                        <!-- Product Quantity Selector -->
                        <th class="text-center td-qty">Quantity</th>
                        <!-- Product price (per unit) -->
                        <th class="text-end td-price">Price</th>
                    </tr>
                </thead>
                <tbody id="o_wsale_reorder_body" class="sale_tbody">
                    <t t-foreach="content.products" t-as="product" t-key="product_index">
                        <tr class="js_product">
                            <td t-if="product.has_image" class="td-img">
                                <img class="product_detail_img" t-att-alt="product.name" t-attf-src="/web/image/product.product/{{product.product_id}}/image_128"/>
                            </td>
                            <td t-att-colspan="product.has_image ? '1' : '2'">
                                <h5><t t-esc="product.name"/></h5>
                                <span class="text-muted d-none d-md-inline-block" t-if="product.description_sale" t-out="product.description_sale"/>
                            </td>
                            <t t-if="product.add_to_cart_allowed">
                                <td class="text-center td-qty">
                                    <div class="css_quantity input-group input-group-sm justify-content-center">
                                        <a href="#" class="btn btn-link d-none d-md-inline-block" aria-label="Remove one" title="Remove one" t-on-click.stop.prevent="() => this.changeProductQty(product, product.qty - 1)">
                                            <i class="fa fa-minus"/>
                                        </a>
                                        <input type="text" class="js_quantity text-center form-control quantity" t-on-change="(ev) => this.onChangeProductQtyInput(ev, product)"
                                            t-att-value="product.qty"/>
                                        <a href="#" class="btn btn-link d-none d-md-inline-block" aria-label="Add one" title="Add one" t-on-click.stop.prevent="() => this.changeProductQty(product, product.qty + 1)">
                                            <i class="fa fa-plus"/>
                                        </a>
                                    </div>
                                    <div t-if="product.qty_warning and product.qty_warning !== ''" class="text-warning fw-bold">
                                        <i class="fa fa-exclamation-triangle"/>
                                        <span t-esc="product.qty_warning"/>
                                    </div>
                                </td>
                                <td class="text-end td-price">
                                    <span t-esc="formatCurrency(product.combinationInfo.price, content.currency)"/>
                                </td>
                            </t>
                            <t t-else="">
                                <td class="text-center" colspan="2">
                                    <div class="text-warning fw-bold">
                                        <i class="fa fa-exclamation-triangle"/>
                                        <span t-esc="getWarningForProduct(product)"/>
                                    </div>
                                </td>
                            </t>
                        </tr>
                    </t>
                </tbody>
            </table>
            <div id="o_wsale_reorder_total" class="row" name="total" style="page-break-inside: avoid;">
                <div class="col-sm-7 col-md-6 ms-auto">
                    <table class="table table-sm">
                        <tr class="border-black o_total">
                            <td>
                                <strong>Total</strong>
                            </td>
                            <td class="text-end">
                                <span t-out="formatCurrency(total, content.currency)"/>
                            </td>
                        </tr>
                    </table>
                </div>
            </div>
            <t t-set-slot="footer">
                <button class="btn btn-primary o_wsale_reorder_confirm" t-att-disabled="!hasBuyableProducts" t-on-click="confirmReorder">
                    Add To Cart
                </button>
                <button class="btn btn-secondary o_wsale_reorder_cancel" t-on-click.stop.prevent="props.close">
                    Discard
                </button>
            </t>
        </Dialog>
    </t>

    <t t-name="website_sale.ReorderConfirmationDialog" t-inherit="web.ConfirmationDialog" t-inherit-mode="primary">
        <xpath expr="//button[1]" position="replace">
            <button class="btn btn-primary" t-on-click="_confirm">
              Yes
            </button>
        </xpath>
        <xpath expr="//button[2]" position="replace">
            <button class="btn btn-secondary" t-on-click="_cancel">
              No
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\website_sale_utils.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<!-- Products Search Bar autocomplete item -->
<we-button t-name="website_sale.ribbonSelectItem" t-att-data-set-ribbon="ribbon.id">
    <t t-out="ribbon.html"/>
    <span t-attf-class="fa fa-#{isTag ? 'tag' : 'bookmark'} ms-auto"></span>
    <span t-attf-class="fa fa-arrow-#{isLeft ? 'left' : 'right'} ms-1"></span>
    <span t-attf-class="o_wsale_color_preview #{colorClasses} ms-1" t-attf-style="background-color: #{ribbon.bg_color}"></span>
    <span t-attf-class="o_wsale_color_preview #{colorClasses} ms-1" t-attf-style="background-color: #{textColor} !important;"></span>
</we-button>

</templates>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_move_view_form" model="ir.ui.view">
        <field name="name">account.move.form.inherit.website_sale</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <group name="sale_info_group" position="inside">
                <field name="website_id"
                       invisible="not website_id"
                       groups="website.group_multi_website"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="crm_team_view_kanban_dashboard" model="ir.ui.view"> 
        <field name="name">crm.team.view.kanban.dashboard.inherit.website.sale</field>
        <field name="model">crm.team</field>
        <field name="inherit_id" ref="sales_team.crm_team_view_kanban_dashboard"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//t[@name='third_options']" position="after">
                    <div class="row" t-if="record.abandoned_carts_count.raw_value">
                        <div class="col-8">
                            <div>
                                <a name="get_abandoned_carts" type="object">
                                    <field name="abandoned_carts_count" class="me-1"/>
                                    Abandoned Carts to Recover
                                </a>
                            </div>
                        </div>
                        <div class="col-4 text-end">
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
        <field name="priority">10</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form" />
        <field name="arch" type="xml">
            <group name="kpi_sales" position="attributes">
                <attribute name="string">Sales</attribute>
                <attribute name="groups">sales_team.group_sale_salesman_all_leads</attribute>
            </group>
            <group name="kpi_sales" position="inside">
                <field name="kpi_website_sale_total"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\product_attribute_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_attribute_view_form" model="ir.ui.view">
        <field name="name">product.attribute.view.form</field>
        <field name="model">product.attribute</field>
        <field name="inherit_id" ref="product.product_attribute_view_form"/>
        <field name="arch" type="xml">
            <group name="main_fields" position="inside">
                <group name="ecommerce_main_fields">
                    <field name="visibility" string="eCommerce Filter Visibility" widget="radio"/>
                </group>
            </group>
        </field>
    </record>

    <record id="attribute_tree_view" model="ir.ui.view">
        <field name="name">product.attribute.tree</field>
        <field name="model">product.attribute</field>
        <field name="inherit_id" ref="product.attribute_tree_view"/>
        <field name="arch" type="xml">
            <field name="create_variant" position="after">
                <field name="visibility" string="eCommerce Filter Visibility"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\product_document_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="product_document_form" model="ir.ui.view">
        <field name="name">product.document.form.website_sale</field>
        <field name="model">product.document</field>
        <field name="inherit_id" ref="sale.product_document_form"/>
        <field name="arch" type="xml">
            <sheet position="inside">
                <group name="website_sale" string="E-commerce">
                    <field name="shown_on_product_page"/>
                </group>
            </sheet>
        </field>
    </record>

    <record id="product_document_kanban" model="ir.ui.view">
        <field name="name">product.document.kanban.website_sale</field>
        <field name="model">product.document</field>
        <field name="inherit_id" ref="sale.product_document_kanban"/>
        <field name="arch" type="xml">
            <div name="bottom" position="inside">
                <div class="mt-2">
                    <span>Show on product page</span>
                    <field name="shown_on_product_page" class="ms-2" widget="boolean_toggle"/>
                </div>
            </div>
        </field>
    </record>

    <record id="product_document_list" model="ir.ui.view">
        <field name="name">product.document.list.website_sale</field>
        <field name="model">product.document</field>
        <field name="inherit_id" ref="sale.product_document_list"/>
        <field name="arch" type="xml">
            <field name="attached_on" position="after">
                <field name="shown_on_product_page"/>
            </field>
        </field>
    </record>

    <record id="product_document_search" model="ir.ui.view">
        <field name="name">product.document.search.sale</field>
        <field name="model">product.document</field>
        <field name="inherit_id" ref="sale.product_document_search"/>
        <field name="arch" type="xml">
            <search position="inside">
                <separator/>
                <filter name="e_commerce" string="Show on Ecommerce" domain="[('shown_on_product_page', '=', True)]"/>
            </search>
        </field>
    </record>
</odoo>

```

## File: views\product_product_add.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="product_product_view_form_add" model="ir.ui.view">
    <field name="name">product.product.view.form.add</field>
    <field name="model">product.product</field>
    <field name="arch" type="xml">
        <form js_class="website_new_content_form">
            <group name="pricing">
                <field name="website_url" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="currency_id" invisible='1'/>
                <field name="cost_currency_id" invisible="1"/>
                <field name="name" placeholder="e.g. Cheese Burger" string="Product Name"/>
                <label for="list_price" class="mt-1"/>
                <div name="Sales Price">
                    <field name="list_price" class="oe_inline" widget='monetary'
                    options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                    <field name="tax_string"/>
                </div>
                <field name="taxes_id" widget="many2many_tags" context="{'default_type_tax_use':'sale', 'search_default_sale': 1}" options="{'create': false, 'create_edit': false}"/>
            </group>
        </form>
    </field>
</record>

<record id="product_product_action_add" model="ir.actions.act_window">
    <field name="name">New Product</field>
    <field name="res_model">product.product</field>
    <field name="view_mode">form</field>
    <field name="target">new</field>
    <field name="view_id" ref="product_product_view_form_add"/>
</record>

</odoo>

```

## File: views\product_tag_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Product Tags -->
    <record id="product_tag_form_view_inherit_website_sale" model="ir.ui.view">
        <field name="name">product.tag.form.inherit.website.sale</field>
        <field name="model">product.tag</field>
        <field name="inherit_id" ref="product.product_tag_form_view"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="visible_on_ecommerce"/>
                <field name="color"
                       widget='color'
                       invisible="not visible_on_ecommerce"/>
                <field name="image"
                       widget="image"
                       options="{'size': [0, 55]}"
                       invisible="not visible_on_ecommerce"/>
                <p class="text-muted" colspan="2" invisible="not visible_on_ecommerce">
                    If an image is set, the color will not be used on eCommerce.
                </p>
            </field>
        </field>
    </record>

    <record id="product_tag_tree_view_inherit_website_sale" model="ir.ui.view">
        <field name="name">product.tag.tree.inherit.website.sale</field>
        <field name="model">product.tag</field>
        <field name="inherit_id" ref="product.product_tag_tree_view"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="visible_on_ecommerce" optional="show"/>
                <field name="color"
                       widget="color"
                       optional="hide"
                       readonly="1"
                       invisible="not visible_on_ecommerce"/>
                <field name="image"
                       widget="image"
                       options="{'size': [0, 30]}"
                       optional="hide"
                       invisible="not visible_on_ecommerce"/>
            </field>
        </field>
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
            <filter name="consumable" position="after">
                <separator/>
                <filter string="Published" name="published" domain="[('is_published', '=', True)]"/>
            </filter>
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
            <field name="additional_product_tag_ids" position="after">
                <field name="ribbon_id" options="{'no_quick_create': True}" optional="hide"/>
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
            <tree position="attributes">
              <attribute name="default_order">website_sequence</attribute>
            </tree>
            <field name="priority" position="before">
                <field name="website_sequence" widget="handle"/>
            </field>
            <field name="website_id" position="after">
                <field name="public_categ_ids" widget="many2many_tags" string="Categories" optional="show"/>
                <field name="is_published" string="Is Published" optional="hide"/>
            </field>
        </field>
    </record>

    <record model="ir.ui.view" id="product_template_view_kanban_website_sale">
        <field name="name">product.template.view.kanban.website_sale</field>
        <field name="mode">primary</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_kanban_view"/>
        <field name="arch" type="xml">
            <kanban position="attributes">
              <attribute name="default_order">website_sequence</attribute>
            </kanban>
            <field name="id" position="after">
                <field name="website_sequence"/>
            </field>
        </field>
    </record>

    <record id="product_template_action_website" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">kanban,tree,form,activity</field>
        <field name="view_id"/>
        <field name="search_view_id" ref="product_template_search_view_website"/>
        <field name="context">{'search_default_published': 1, 'tree_view_ref':'website_sale.product_template_view_tree_website_sale', 'kanban_view_ref':'website_sale.product_template_view_kanban_website_sale'}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new product
            </p><p>
                A product can be either a physical product or a service that you sell to your customers.
            </p>
        </field>
    </record>

    <record model="ir.ui.view" id="product_template_only_website_form_view">
        <field name="name">product.template.product.only.website.form</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_only_form_view"/>
        <field name="arch" type="xml">
            <div name="pricing" position="after">
                <label for="compare_list_price" groups="website_sale.group_product_price_comparison"/>
                <div class="o_row" groups="website_sale.group_product_price_comparison">
                    <del class="oe_read_only">
                        <field name="compare_list_price" nolabel="1" widget="monetary" options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                    </del>
                    <field name="compare_list_price" nolabel="1" widget="monetary" options="{'currency_field': 'currency_id', 'field_digits': True}" class="oe_edit_only"/>
                </div>
            </div>
            <field name="uom_id" position="after">
                <label for="base_unit_count" groups="website_sale.group_show_uom_price"/>
                <div name="base_unit_price" groups="website_sale.group_show_uom_price" class="d-flex flex-row">
                    <field name="base_unit_count" invisible="product_variant_count &gt; 1" style="width: 4rem;"/>
                    <field name="base_unit_id" options="{'no_open': True}" invisible="product_variant_count &gt; 1" placeholder="Specify unit" style="width: 10rem;"/>
                    <div class="d-flex flex-row" invisible="base_unit_price == 0 or product_variant_count &gt; 1">
                        (<field name="base_unit_price" class="oe_inline"/> / <field name="base_unit_name" class="oe_inline"/>)
                    </div>
                    <span class='text-muted' invisible="product_variant_count &lt;= 1">Based on variants</span>
                </div>
            </field>
        </field>
    </record>

    <record model="ir.ui.view" id="product_product_normal_website_form_view">
        <field name="name">product.product.normal.view.website</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_normal_form_view"/>
        <field name="arch" type="xml">
            <field name="uom_id" position="after">
                <label for="base_unit_count" groups="website_sale.group_show_uom_price"/>
                <div name="base_unit_price" groups="website_sale.group_show_uom_price" class="d-flex flex-row">
                    <field name="base_unit_count" style="width: 4rem;"/>
                    <field name="base_unit_id" options="{'no_open': True}" placeholder="Specify unit" style="width: 10rem;"/>
                    <div class="d-flex flex-row" invisible="base_unit_price == 0">
                        (<field name="base_unit_price" class="oe_inline"/> / <field name="base_unit_name" class="oe_inline"/>)
                    </div>
                </div>
            </field>
        </field>
    </record>

    <record model="ir.ui.view" id="product_template_form_view">
        <field name="name">product.template.product.website.form</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <!-- add state field in header -->
            <div name="button_box" position="inside">
                <field name="is_published" widget="website_redirect_button" invisible="not sale_ok"/>
            </div>
            <group name="upsell" position="attributes">
                <attribute name="invisible">0</attribute>
            </group>
            <group name="upsell" position="inside">
                <field name="accessory_product_ids" widget="many2many_tags" invisible="not sale_ok"
                       placeholder="Suggested accessories in the eCommerce cart"/>
                <field name="alternative_product_ids" widget="many2many_tags"
                       domain="[('id', '!=', id), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"
                       invisible="not sale_ok"
                       placeholder="Displayed in bottom of product pages"/>
            </group>
            <xpath expr="//page[@name='sales']/group[@name='sale']" position="inside">
                <group string="eCommerce Shop" name="shop" invisible="not sale_ok">
                    <field name="website_url" invisible="1"/>
                    <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                    <field name="website_sequence" groups="base.group_no_one"/>
                    <field name="public_categ_ids" widget="many2many_tags" string="Categories"/>
                    <field name="website_ribbon_id" groups="base.group_no_one" options="{'no_quick_create': True}"/>
                </group>
                <group name="product_template_images" string="Extra Product Media" invisible="not sale_ok">
                    <field name="product_template_image_ids" class="o_website_sale_image_list" context="{'default_name': name}" mode="kanban" add-label="Add a Media" nolabel="1"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="product_product_view_form_easy_inherit_website_sale" model="ir.ui.view">
        <field name="name">product.product.view.form.easy.inherit.website_sale</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_variant_easy_edit_view"/>
        <field name="arch" type="xml">
            <group name="pricing" position="inside">
                <label for="base_unit_count" groups="website_sale.group_show_uom_price"/>
                <div name="base_unit_price" groups="website_sale.group_show_uom_price" class="d-flex flex-row">
                    <field name="base_unit_count" style="width: 4rem;"/>
                    <field name="base_unit_id" options="{'no_open': True}" placeholder="Specify unit" style="width: 10rem;"/>
                    <div class="d-flex flex-row" invisible="base_unit_price == 0">
                        (<field name="base_unit_price" class="oe_inline"/> / <field name="base_unit_name" class="oe_inline"/>)
                    </div>
                </div>
            </group>
            <group name="packaging" position="after">
                <group name="product_variant_images" string="Extra Variant Media">
                    <field name="product_variant_image_ids" class="o_website_sale_image_list" context="{'default_name': name}" mode="kanban" add-label="Add a Media" nolabel="1"/>
                </group>
                <group name="sales" string="Sales" groups="base.group_no_one">
                    <field name="ribbon_id" options="{'no_quick_create': True}"/>
                </group>
            </group>
        </field>
    </record>

    <!-- Product ribbon -->
    <record id="product_ribbon_form_view" model="ir.ui.view">
        <field name="name">product.ribbon form view</field>
        <field name="model">product.ribbon</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <group>
                            <field name="html" widget="char"/>
                            <field name="text_color"/>
                        </group>
                        <group>
                            <field name="html_class"/>
                            <field name="bg_color"/>
                        </group>
                    </group>
                </sheet>
            </form>
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
                    <div class="float-start">
                        <group class="col-md-6 pe-3">
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
            <group name="pricelist_availability" position="after">
                <group name="pricelist_website" string="Website">
                    <field name="company_id" invisible="1"/>
                    <field name="website_id" options="{'no_create': True}"/>
                    <field name="selectable"/>
                    <field name="code"/>
                </group>
            </group>
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
                            <div class="col" invisible="video_url in ['', False]">
                                <div class="o_video_container p-2">
                                    <span>Video Preview</span>
                                    <field name="embed_code" class="mt-2" widget="video_preview"/>
                                    <h4 class="o_invalid_warning text-muted text-center" invisible="embed_code">
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
                                <t t-set="size_status" t-value="'text-bg-success'"/>
                                <t t-set="message">Acceptable file size</t>
                            </t>
                            <!-- below 1000 Kb: decent -->
                            <t t-elif="record.image_1920.raw_value.length &lt; 1000*1000">
                                <t t-set="size_status" t-value="'text-bg-warning'" />
                                <t t-set="message">Huge file size. The image should be optimized/reduced.</t>
                            </t>
                            <!-- above 1000 Kb: bad -->
                            <t t-else="1">
                                <t t-set="size_status" t-value="'text-bg-danger'"/>
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

    <record id="res_config_settings_view_form_inherit_sale" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <!-- Remove customer accounts setting from sales settings tab -->
            <!-- It must not be in the view at all to make sure settings can be saved
                (because auth_signup_uninvited is specified as required) -->
            <setting id="auth_signup_documents" position="replace"/>
        </field>
    </record>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <block id="website_info_settings" position="after">
                <block title="Shop - Checkout Process" id="website_shop_checkout">
                    <setting id="cart_redirect_setting" string="Add to Cart" help="What should be done on &quot;Add to Cart&quot;?">
                        <div class="content-group">
                            <div class="row mt16 ms-4">
                                <field name="add_to_cart_action" widget="radio"/>
                            </div>
                        </div>
                    </setting>
                    <setting help="Instant checkout, instead of adding to cart">
                        <field name="enabled_buy_now_button"/>
                    </setting>
                    <setting help="Add a customizable form during checkout (after address)">
                        <field name="enabled_extra_checkout_step"/>
                        <div class="row mt8 ms-4" invisible="not enabled_extra_checkout_step">
                            <button type="object" name="action_open_extra_info" string="Configure Form " class="btn-link" icon="oi-arrow-right"/>
                        </div>
                    </setting>
                    <setting string="Assignment" help="Assignment of online orders">
                        <div class="content-group">
                            <div class="row mt16">
                                <label class="o_light_label col-lg-3" string="Sales Team" for="salesteam_id"/>
                                <field name="salesteam_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                            </div>
                            <div class="row">
                                <label class="o_light_label col-lg-3" for="salesperson_id"/>
                                <field name="salesperson_id"/>
                            </div>
                        </div>
                    </setting>
                    <setting help="Allow your customer to add products from previous order in their cart.">
                        <field name="website_sale_enabled_portal_reorder_button"/>
                    </setting>
                </block>

                <block title="Shop - Products" id="sale_product_catalog_settings">
                    <setting id="website_tax_inclusion_setting">
                        <label for="show_line_subtotals_tax_selection" string="Display Product Prices"/>
                        <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                        <div class="text-muted">
                            Prices displayed on your eCommerce
                        </div>
                        <div class="content-group">
                            <div class="row mt16">
                                <field name="show_line_subtotals_tax_selection" class="o_light_label" widget="radio"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="pricelists_setting" title="With the first mode you can set several prices in the product config form (from Sales tab). With the second one, you set prices and computation rules from Pricelists." help="Manage pricelists to apply specific prices per country, customer, products, etc">
                        <field name="group_product_pricelist"/>
                        <div class="content-group mt16" invisible="not group_product_pricelist">
                            <field name="group_sale_pricelist" invisible="1"/>
                            <field name="group_product_pricelist" invisible="1"/>
                            <field name="product_pricelist_setting" class="o_light_label w-75" widget="radio"/>
                        </div>
                        <div invisible="not group_product_pricelist">
                            <button type="action" name="%(product.product_pricelist_action2)d" string="Pricelists" class="btn-link" icon="oi-arrow-right"/>
                        </div>
                    </setting>
                    <setting help="Add a strikethrough price, as a comparison">
                        <field name="group_product_price_comparison"/>
                    </setting>
                    <setting id="ecom_uom_price_option_setting" string="Product Reference Price" help="Add a reference price per UoM on products (i.e $/kg), in addition to the sale price">
                        <field name="group_show_uom_price"/>
                    </setting>
                    <setting id="product_attributes_setting" string="Product Variants" help="One product might have different attributes (size, color, ...)">
                        <field name="group_product_variant"/>
                        <div class="content-group" invisible="not group_product_variant">
                            <div class="mt8">
                                <button type="action" name="%(product.attribute_action)d" string="Attributes" class="btn-link" icon="oi-arrow-right"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="promotion_coupon_programs" title="Boost your sales with multiple kinds of programs: Coupons, Promotions, Gift Card, Loyalty. Specific conditions can be set (products, customers, minimum purchase amount, period). Rewards can be discounts (% or amount) or free products." string="Discounts, Loyalty &amp; Gift Card" help="Manage Promotions, coupons, loyalty cards, Gift cards &amp; eWallet">
                        <field name="module_loyalty" />
                    </setting>
                    <setting id="wishlist_option_setting" help="Allow signed-in users to save product in a wishlist">
                        <field name="module_website_sale_wishlist"/>
                    </setting>
                    <setting id="comparator_option_setting" string="Product Comparison Tool" help="Allow shoppers to compare products based on their attributes">
                        <field name="module_website_sale_comparison"/>
                    </setting>
                    <setting id="hide_add_to_cart_setting" help="If product price equals 0, replace 'Add to Cart' by 'Contact us'.">
                        <field name="website_sale_prevent_zero_price_sale"/>
                        <div class="content-group" invisible="not website_sale_prevent_zero_price_sale">
                            <div class="row mt16">
                                <label class="o_light_label col-lg-3" string="Button url" for="website_sale_contact_us_button_url"/>
                                <field name="website_sale_contact_us_button_url"/>
                            </div>
                        </div>
                    </setting>
                </block>

                <block title="Shipping" id="sale_shipping_settings">
                    <setting id="shipping_address_setting" help="Let the customer enter a shipping address">
                        <field name="group_delivery_invoice_address"/>
                    </setting>
                    <setting id="delivery_method_setting" string="Shipping Costs" help="Compute shipping costs on orders"
                             documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html">
                        <div class="content-group">
                            <div class="mt16">
                                <button type="action" name="%(delivery.action_delivery_carrier_form)d" string="Shipping Methods" class="btn-link" icon="oi-arrow-right"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="ups_provider_setting" string="UPS" help="Compute shipping costs and ship with UPS"
                             documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html">
                        <field name="module_delivery_ups" widget="upgrade_boolean"/>
                    </setting>
                    <setting id="shipping_provider_dhl_setting" string="DHL Express Connector" help="Compute shipping costs and ship with DHL"
                             documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html">
                        <field name="module_delivery_dhl" widget="upgrade_boolean"/>
                        <div class="content-group">
                            <div class="mt8" invisible="not module_delivery_dhl">
                                <button name="%(delivery.action_delivery_carrier_form)d" icon="oi-arrow-right" type="action" string="DHL Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'dhl'}"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="shipping_provider_fedex_setting" string="FedEx" help="Compute shipping costs and ship with FedEx"
                             documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html">
                        <field name="module_delivery_fedex" widget="upgrade_boolean"/>
                        <div class="content-group">
                            <div class="mt8" invisible="not module_delivery_fedex">
                                <button name="%(delivery.action_delivery_carrier_form)d" icon="oi-arrow-right" type="action" string="FedEx Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'fedex'}"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="shipping_provider_usps_setting" string="USPS" help="Compute shipping costs and ship with USPS"
                             documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html">
                        <field name="module_delivery_usps" widget="upgrade_boolean"/>
                        <div class="content-group">
                            <div class="mt8" invisible="not module_delivery_usps">
                                <button name="%(delivery.action_delivery_carrier_form)d" icon="oi-arrow-right" type="action" string="USPS Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'usps'}"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="shipping_provider_bpost_setting" string="bpost" help="Compute shipping costs and ship with bpost"
                             documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html">
                        <field name="module_delivery_bpost" widget="upgrade_boolean"/>
                        <div class="content-group">
                            <div class="mt8" invisible="not module_delivery_bpost">
                                <button name="%(delivery.action_delivery_carrier_form)d" icon="oi-arrow-right" type="action" string="bpost Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'bpost'}"/>
                            </div>
                        </div>
                    </setting>
                    <setting id="shipping_provider_easypost_setting" string="Easypost" help="Compute shipping cost and ship with Easypost"
                             documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html">
                        <field name="module_delivery_easypost" widget="upgrade_boolean"/>
                        <div class="content-group">
                            <div class="mt8" invisible="not module_delivery_easypost">
                                <button name="%(delivery.action_delivery_carrier_form)d" icon="oi-arrow-right" type="action" string="Easypost Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'easypost'}"/>
                            </div>
                        </div>
                    </setting>
                    <setting
                        id="shipping_provider_shiprocket_setting"
                        string="Shiprocket"
                        help="Compute shipping cost and ship with Shiprocket"
                        documentation="/applications/inventory_and_mrp/inventory/shipping/setup/third_party_shipper.html"
                    >
                        <field name="module_delivery_shiprocket" widget="upgrade_boolean"/>
                            <div class="content-group">
                                <div class="mt8" invisible="not module_delivery_shiprocket">
                                    <button
                                        name="%(delivery.action_delivery_carrier_form)d"
                                        icon="oi-arrow-right"
                                        type="action"
                                        string="Shiprocket Shipping Methods"
                                        class="btn-link"
                                        context="{'search_default_delivery_type': 'shiprocket'}"/>
                                </div>
                            </div>
                    </setting>
                    <setting id="shipping_provider_mondialrelay_setting" string="Mondial Relay" help="Let the customer select a Mondial Relay shipping point">
                        <field name="module_delivery_mondialrelay"/>
                    </setting>
                    <setting id="onsite_payment_setting" help="Allow customers to pay in person at your stores">
                        <field name="module_website_sale_picking"/>
                    </setting>
                </block>

                <field name='module_account' invisible="1"/>
                    <block title="Invoicing" id="sale_invoicing_settings" invisible="not module_account">
                        <setting id="invoicing_policy_setting" title="The mode selected here applies as invoicing policy of any new product created but not of products already existing." string="Invoicing Policy" help="Issue invoices to customers">
                            <div class="content-group">
                                <div class="mt16">
                                    <field name="default_invoice_policy" class="o_light_label" widget="radio"/>
                                </div>
                            </div>
                        </setting>
                        <setting id="automatic_invoice_generation" invisible="default_invoice_policy == 'delivery'" help="Generate the invoice automatically when the online payment is confirmed">
                            <field name="automatic_invoice"/>
                            <div  invisible="not automatic_invoice">
                                <label for="invoice_mail_template_id" class="o_light_label me-2"/>
                                <field name="invoice_mail_template_id" class="oe_inline"/>
                            </div>
                        </setting>
                    </block>
            </block>

            <setting id="cart_redirect_setting" position="after">
                <setting id="website_checkout_registration" title=" To send invitations in B2B mode, open a contact or select several ones in list view and click on 'Portal Access Management' option in the dropdown menu *Action*."
                         string="Sign in/up at checkout" help="&quot;Optional&quot; allows guests to register from the order confirmation email to track their order.">
                    <field name="account_on_checkout" class="w-75" widget="radio"/>
                </setting>
            </setting>

            <setting id="website_marketing_automation" position="after">
                <setting
                    id="abandoned_carts_setting"
                    title="Customer needs to be signed in otherwise the mail address is not known.
    &#10;&#10;- If a potential customer creates one or more abandoned checkouts and then completes a sale before the recovery email gets sent, then the email won't be sent.
    &#10;&#10;- If user has manually sent a recovery email, the mail will not be sent a second time
    &#10;&#10;- If a payment processing error occurred when the customer tried to complete their checkout, then the email won't be sent.
    &#10;&#10;- If your shop does not support shipping to the customer's address, then the email won't be sent.
    &#10;&#10;- If none of the products in the checkout are available for purchase (empty inventory, for example), then the email won't be sent.
    &#10;&#10;- If all the products in the checkout are free, and the customer does not visit the shipping page to add a shipping fee or the shipping fee is also free, then the email won't be sent."
                    string="Automatically send abandoned checkout emails"
                    help="Mail only sent to signed in customers with items available for sale in their cart.">
                    <field name="send_abandoned_cart_email"/>

                    <div invisible="not send_abandoned_cart_email" class="content-group" title="Carts are flagged as abandoned after this delay.">
                        <div class="row mt16">
                            <div class="col-12">
                                <label for="cart_abandoned_delay" string="Send after" class="o_light_label"/>
                                <field class="col-2" name="cart_abandoned_delay" widget="float_time" /> Hours.
                            </div>
                        </div>
                    </div>
                    <div invisible="not send_abandoned_cart_email" class="mt8">
                        <button type="object" name="action_open_abandoned_cart_mail_template" string="Customize Abandoned Email Template" class="btn-link" icon="oi-arrow-right"/>
                    </div>
                </setting>
            </setting>

            <setting id="google_analytics_setting" position="after">
                <setting id="autocomplete_googleplaces_setting" help="Use Google Places API to validate addresses entered by your visitors">
                    <field name="module_website_sale_autocomplete"/>
                </setting>
            </setting>
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
            <filter name="my_sale_orders_filter" position="before">
                <filter string="Confirmed" name="order_confirmed" domain="[('state', '=', 'sale')]"/>
                <filter string="Unpaid" name="order_unpaid" domain="[('state', '=', 'sent')]"/>
                <filter string="Abandoned" name="order_abandoned" domain="[('is_abandoned_cart', '=', True)]"/>
                <separator/>
                <filter string="Order Date" name="order_date" date="date_order"/>
                <separator/>
                <filter string="From Website" name="from_website" domain="[('website_id', '!=', False)]"/>
                <separator/>
                <!-- Dashboard filter - used by context -->
                <filter string="Last Week" invisible="1" name="week" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=7)).strftime('%Y-%m-%d'))]"/>
                <filter string="Last Month" invisible="1" name="month" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                <filter string="Last Year" invisible="1"  name="year" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=365)).strftime('%Y-%m-%d'))]"/>
            </filter>
        </field>
    </record>

    <record id="view_sales_order_filter_ecommerce_unpaid" model="ir.ui.view">
        <field name="name">sale.order.ecommerce.search.unpaid.view</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_sales_order_filter"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <filter name="my_sale_orders_filter" position="attributes">
                <attribute name="invisible">1</attribute>
            </filter>
            <filter name="my_sale_orders_filter" position="before">
                <filter string="Order Date" name="order_date" date="date_order"/>
                <separator/>
            </filter>
        </field>
    </record>

    <record id="sale_order_view_form_cart_recovery" model="ir.ui.view">
        <field name="name">sale.order.form.abandoned.cart</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <field name="team_id" position="after">
                <field name="is_abandoned_cart" invisible="1"/>
                <field name="cart_recovery_email_sent" invisible="1"/>
            </field>
            <button name="action_quotation_send" id="send_by_email_primary" position="attributes">
                <attribute name="invisible">state != 'draft' or (is_abandoned_cart and not cart_recovery_email_sent)</attribute>
            </button>
            <button name="action_quotation_send" position="after">
                <button name="action_recovery_email_send" type="object" id="send_recovery" data-hotkey="l"
                    string="Send a Recovery Email"
                    class="btn-primary"
                    invisible="not is_abandoned_cart or cart_recovery_email_sent"/>
            </button>
            <button name="action_quotation_send" id="send_by_email" position="after">
                <button name="action_quotation_send" id="send_by_email_bis" string="Send by Email" type="object"
                    invisible="not is_abandoned_cart or cart_recovery_email_sent or state != 'draft'"/>
            </button>
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
                <filter string="Last Week" invisible="1" name="week" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=7)).strftime('%Y-%m-%d'))]"/>
                <filter string="Last Month" invisible="1" name="month" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                <filter string="Last Year" invisible="1"  name="year" domain="[('date_order','&gt;', (context_today() - datetime.timedelta(days=365)).strftime('%Y-%m-%d'))]"/>
            </search>
        </field>
    </record>

    <!-- Dashboard Action -->
    <record id="sale_order_action_to_invoice" model="ir.actions.act_window">
        <field name="name">Orders To Invoice</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">tree,form,kanban</field>
        <field name="domain">[('state', '=', 'sale'), ('order_line', '!=', False), ('invoice_status', '=', 'to invoice'), ('website_id', '!=', False)]</field>
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
                <field name="website_id" invisible="not website_id" groups="website.group_multi_website"/>
            </field>
        </field>
    </record>

    <record id="sale_order_tree" model="ir.ui.view">
        <field name="name">sale.order.tree.inherit.website.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.sale_order_tree"/>
        <field name="arch" type="xml">
            <field name="user_id" position="before">
                <field name="website_id" groups="website.group_multi_website" optional="show"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="header_cart_link" name="Header Cart Link">
        <t t-nocache="The number of products is dynamic, this rendering cannot be cached."
           t-nocache-_icon="_icon"
           t-nocache-_text="_text"
           t-nocache-_badge="_badge"
           t-nocache-_badge_class="_badge_class"
           t-nocache-_icon_wrap_class="_icon_wrap_class"
           t-nocache-_text_class="_text_class"
           t-nocache-_item_class="_item_class"
           t-nocache-_link_class="_link_class">
            <t t-set="website_sale_cart_quantity" t-value="request.session['website_sale_cart_quantity'] if 'website_sale_cart_quantity' in request.session else website.sale_get_order().cart_quantity or 0"/>
            <t t-set="show_cart" t-value="true"/>
            <li t-attf-class="#{_item_class} divider d-none"/> <!-- Make sure the cart and related menus are not folded (see autohideMenu) -->
            <li t-attf-class="o_wsale_my_cart #{not show_cart and 'd-none'} #{_item_class}">
                <a href="/shop/cart" t-attf-class="#{_link_class}" aria-label="eCommerce cart">
                    <div t-attf-class="#{_icon_wrap_class}">
                        <i t-if="_icon" class="fa fa-shopping-cart fa-stack"/>
                        <sup t-attf-class="my_cart_quantity badge text-bg-primary #{_badge_class} #{'d-none' if (website_sale_cart_quantity == 0) else ''}" t-esc="website_sale_cart_quantity" t-att-data-order-id="request.session.get('sale_order_id', '')"/>
                    </div>
                    <span t-if="_text" t-attf-class="#{_text_class}">My Cart</span>
                </a>
            </li>
        </t>
    </template>

    <template id="header_hide_empty_cart_link" inherit_id="website_sale.header_cart_link" name="Header Hide Empty Cart link" active="False">
        <xpath expr="//t[@t-set='show_cart']" position="after">
            <t t-set="show_cart" t-value="website_sale_cart_quantity" />
        </xpath>
    </template>

    <template id="template_header_mobile" inherit_id="website.template_header_mobile">
        <xpath expr="//ul[hasclass('o_header_mobile_buttons_wrap')]//li" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_link_class" t-value="'o_navlink_background_hover btn position-relative rounded-circle border-0 p-1 text-reset'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded-pill'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_default" inherit_id="website.template_header_default">
        <xpath expr="//t[@t-call='website.placeholder_header_search_box']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_link_class" t-value="'o_navlink_background btn position-relative rounded-circle p-1 text-center text-reset'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded-pill'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_hamburger" inherit_id="website.template_header_hamburger">
        <xpath expr="//t[@t-call='portal.placeholder_user_sign_in']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_link_class" t-value="'o_navlink_background_hover btn position-relative rounded-pill p-1 text-reset'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_stretch" inherit_id="website.template_header_stretch">
        <xpath expr="//t[@t-call='website.placeholder_header_social_links']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'border-start o_border_contrast'"/>
                <t t-set="_link_class" t-value="'o_navlink_background_hover btn position-relative d-flex align-items-center h-100 rounded-0 p-2 text-reset'"/>
                <t t-set="_badge_class" t-value="'rounded'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_vertical" inherit_id="website.template_header_vertical">
        <xpath expr="//t[@t-call='portal.placeholder_user_sign_in']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_link_class" t-value="'o_navlink_background btn position-relative rounded-circle p-1 text-reset'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded-pill'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_search" inherit_id="website.template_header_search">
        <xpath expr="//t[@t-call='portal.placeholder_user_sign_in']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'border-start o_border_contrast'"/>
                <t t-set="_link_class" t-value="'o_navlink_background_hover btn btn-sm d-flex align-items-center gap-1 h-100 rounded-0 p-2 text-reset'"/>
                <t t-set="_badge_class" t-value="'rounded'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_sales_one" inherit_id="website.template_header_sales_one">
        <xpath expr="//t[@t-call='portal.user_dropdown']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_link_class" t-value="'btn position-relative rounded-circle p-1 text-reset o_navlink_background'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded-pill'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_sales_two" inherit_id="website.template_header_sales_two">
        <xpath expr="//t[@t-call='portal.placeholder_user_sign_in']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_text" t-value="True"/>
                <t t-set="_icon_wrap_class" t-value="'position-relative me-2 rounded-circle border p-2 bg-o-color-3 o_border_contrast'"/>
                <t t-set="_link_class" t-value="'btn d-flex align-items-center fw-bold text-reset o_navlink_background_hover'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded-pill'"/>
                <t t-set="_text_class" t-value="'small'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_sales_three" inherit_id="website.template_header_sales_three">
        <xpath expr="//t[@t-call='website.placeholder_header_language_selector']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'position-relative'"/>
                <t t-set="_link_class" t-value="'nav-link btn btn-sm d-flex flex-row-reverse align-items-center text-uppercase fw-bold'"/>
                <t t-set="_icon_wrap_class" t-value="'d-contains'"/>
                <t t-set="_badge_class" t-value="'top-0 d-block ms-2'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_sales_four" inherit_id="website.template_header_sales_four">
        <xpath expr="//t[@t-call='website.placeholder_header_call_to_action']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_link_class" t-value="'o_navlink_background_hover btn position-relative rounded-pill p-1 text-reset'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded-pill'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_sidebar" inherit_id="website.template_header_sidebar">
        <xpath expr="//t[@t-call='website.placeholder_header_brand']" position="after">
            <div class="d-flex ms-auto mb-0">
                <t t-call="website_sale.header_cart_link">
                    <t t-set="_icon" t-value="True"/>
                    <t t-set="_link_class" t-value="'o_navlink_background_hover btn position-relative p-1 rounded-circle text-reset'"/>
                    <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 rounded-pill mt-n1 me-n1'"/>
                </t>
            </div>
        </xpath>
    </template>

    <template id="template_header_boxed" inherit_id="website.template_header_boxed">
        <xpath expr="//t[@t-call='website.placeholder_header_search_box']" position="before">
            <t t-call="website_sale.header_cart_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_link_class" t-value="'o_navlink_background btn position-relative rounded-circle p-1 text-center text-reset'"/>
                <t t-set="_badge_class" t-value="'position-absolute top-0 end-0 mt-n1 me-n1 rounded-pill'"/>
            </t>
        </xpath>
    </template>

    <!-- Search Bar input-group template -->
    <template id="search" name="Search Box" active="True">
        <t t-call="website.website_search_box_input">
            <t t-set="_form_classes" t-valuef="o_wsale_products_searchbar_form me-auto flex-grow-1 {{_form_classes}}"/>
            <t t-set="_submit_classes" t-valuef="btn btn-{{navClass}}"/>
            <t t-set="_input_classes" t-valuef="border-0 text-bg-{{navClass}}"/>
            <t t-set="search_type" t-valuef="products"/>
            <t t-set="action" t-value="keep('/shop'+ ('/category/'+slug(category)) if category else None, search=0) or '/shop'"/>
            <t t-set="display_image" t-valuef="true"/>
            <t t-set="display_description" t-valuef="true"/>
            <t t-set="display_extra_link" t-valuef="true"/>
            <t t-set="display_detail" t-valuef="true"/>
            <t t-if="attrib_values">
                <t t-foreach="attrib_values" t-as="a">
                    <input type="hidden" name="attrib" t-att-value="'%s-%s' % (a[0], a[1])" />
                </t>
            </t>
        </t>
    </template>

    <template id="products_item" name="Products item">
        <form action="/shop/cart/update" method="post" class="oe_product_cart h-100 d-flex"
            t-att-data-publish="product.website_published and 'on' or 'off'"
            itemscope="itemscope" itemtype="http://schema.org/Product">

            <t t-set="product_href" t-value="keep(product.website_url, page=(pager['page']['num'] if pager['page']['num']&gt;1 else None))" />
            <t t-set="image_type" t-value="product._get_suitable_image_size(ppr, td_product['x'], td_product['y'])"/>

            <div class="oe_product_image position-relative h-100 flex-grow-0 overflow-hidden">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" t-nocache="The csrf token must always be up to date."/>
                <a t-att-href="product_href" class="oe_product_image_link d-block h-100 position-relative" itemprop="url" contenteditable="false">
                    <t t-set="image_holder" t-value="product._get_image_holder()"/>
                    <span t-field="image_holder.image_1920"
                        t-options="{'widget': 'image', 'preview_image': image_type, 'itemprop': 'image', 'class': 'h-100 w-100 position-absolute'}"
                        class="oe_product_image_img_wrapper d-flex h-100 justify-content-center align-items-center position-absolute"/>

                    <t t-set="bg_color" t-value="td_product['ribbon']['bg_color'] or ''"/>
                    <t t-set="text_color" t-value="td_product['ribbon']['text_color']"/>
                    <t t-set="bg_class" t-value="td_product['ribbon']['html_class']"/>
                    <span t-attf-class="o_ribbon o_not_editable #{bg_class}" t-attf-style="#{text_color and ('color: %s; ' % text_color)}#{bg_color and 'background-color:' + bg_color}" t-out="td_product['ribbon']['html'] or ''"/>
                </a>
            </div>
            <div class="o_wsale_product_information position-relative d-flex flex-column flex-grow-1 flex-shrink-1">
                <div class="o_wsale_product_information_text flex-grow-1">
                    <h6 class="o_wsale_products_item_title mb-2">
                        <a class="text-primary text-decoration-none" itemprop="name" t-att-href="product_href" t-att-content="product.name" t-field="product.name" />
                        <a t-if="not product.website_published" role="button" t-att-href="product_href" class="btn btn-sm btn-danger" title="This product is unpublished.">
                            Unpublished
                        </a>
                    </h6>
                </div>
                <div class="o_wsale_product_sub d-flex justify-content-between align-items-end gap-2 flex-wrap pb-1">
                    <t t-set="template_price_vals" t-value="get_product_prices(product)"/>
                    <div class="o_wsale_product_btn"/>
                    <div class="product_price" itemprop="offers" itemscope="itemscope" itemtype="http://schema.org/Offer">
                        <t t-if="'base_price' in template_price_vals and (template_price_vals['base_price'] &gt; template_price_vals['price_reduce']) and (template_price_vals['price_reduce'] or not website.prevent_zero_price_sale)">
                            <del t-attf-class="text-muted me-1 h6 mb-0" style="white-space: nowrap;">
                                <em class="small" t-esc="template_price_vals['base_price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}" />
                            </del>
                        </t>
                        <span class="h6 mb-0" t-if="template_price_vals['price_reduce'] or not website.prevent_zero_price_sale" t-esc="template_price_vals['price_reduce']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                        <span class="h6 mb-0" t-elif="any(ptav.price_extra for ptav in product.attribute_line_ids.product_template_value_ids)">&amp;nbsp;</span>
                        <span class="h6 mb-0" t-else="" t-field="website.prevent_zero_price_sale_text"/>
                        <span itemprop="price" style="display:none;" t-esc="template_price_vals['price_reduce']" />
                        <span itemprop="priceCurrency" style="display:none;" t-esc="website.currency_id.name" />
                    </div>
                </div>
            </div>
        </form>
    </template>

    <template id="products_description" inherit_id="website_sale.products_item" active="False" name="Product Description">
        <xpath expr="//*[hasclass('o_wsale_products_item_title')]" position="after">
            <div class="oe_subdescription mb-2 text-muted small" contenteditable="false">
                <div itemprop="description" t-field="product.description_sale"/>
            </div>
        </xpath>
    </template>

    <template id="products_add_to_cart" inherit_id="website_sale.products_item" active="False" name="Add to Cart">
        <xpath expr="//div[hasclass('o_wsale_product_btn')]" position="inside">
            <t t-set="product_variant_id" t-value="product._get_first_possible_variant_id()"/>
            <input name="product_id" t-att-value="product_variant_id" type="hidden"/>
            <t t-if="product_variant_id and template_price_vals['price_reduce'] or not website.prevent_zero_price_sale">
                <a t-if="product._website_show_quick_add()"
                   href="#" role="button" class="btn btn-primary a-submit" aria-label="Shopping cart" title="Shopping cart">
                    <span class="fa fa-shopping-cart"/>
                </a>
            </t>
        </xpath>
    </template>

    <template id="pricelist_list" name="Pricelists Dropdown">
        <div t-attf-class="o_pricelist_dropdown dropdown #{_classes if hasPricelistDropdown else 'd-none'}">
            <t t-set="curr_pl" t-value="website.pricelist_id" />

            <a role="button" href="#" t-attf-class="dropdown-toggle btn btn-{{navClass}}" data-bs-toggle="dropdown">
                <t t-esc="curr_pl and curr_pl.name or ' - '" />
            </a>
            <div class="dropdown-menu" role="menu">
                <t t-foreach="website_sale_pricelists" t-as="pl">
                    <a role="menuitem" t-att-href="'/shop/change_pricelist/%s' % pl.id" class="dropdown-item">
                        <span class="switcher_pricelist small" t-att-data-pl_id="pl.id" t-esc="pl.name" />
                    </a>
                </t>
            </div>
        </div>
    </template>

    <template id="products_breadcrumb" name="Products Breadcrumb">
        <ol t-if="category" t-attf-class="breadcrumb #{_classes}">
            <li class="breadcrumb-item">
                <a href="/shop">Products</a>
            </li>
            <t t-foreach="category.parents_and_self" t-as="cat">
                <li t-if="cat == category" class="breadcrumb-item">
                    <span class="d-inline-block" t-field="cat.name"/>
                </li>
                <li t-else="" class="breadcrumb-item">
                    <a t-att-href="keep('/shop/category/%s' % slug(cat), category=0)" t-field="cat.name"/>
                </li>
            </t>
        </ol>
    </template>

    <!-- /shop product listing -->
    <template id="products" name="Products">
        <t t-call="website.layout">
            <t t-set="additional_title">Shop</t>
            <t t-set="grid_block_name">Grid</t>
            <t t-set="product_block_name">Product</t>

            <!-- Qweb variable defining the class suffix for navbar items.
                 Change accordingly to the derired visual result (eg. `primary`, `dark`...)-->
            <t t-set="navClass" t-valuef="light"/>

            <!-- Check for active options: the stored value may be used in sub-templates too  -->
            <t t-set="opt_wsale_categories" t-value="is_view_active('website_sale.products_categories')"/>
            <t t-set="opt_wsale_attributes" t-value="is_view_active('website_sale.products_attributes')"/>
            <t t-set="opt_wsale_filter_price" t-value="is_view_active('website_sale.filter_products_price')"/>
            <t t-set="opt_wsale_filter_tags" t-value="is_view_active('website_sale.filter_products_tags')"/>

            <t t-set="opt_wsale_categories_top" t-value="is_view_active('website_sale.products_categories_top')"/>
            <t t-set="opt_wsale_attributes_top" t-value="is_view_active('website_sale.products_attributes_top')"/>

            <t t-set="website_sale_pricelists" t-value="website.get_pricelist_available(show_visible=True)" />
            <t t-set="website_sale_sortable" t-value="website._get_product_sort_mapping()"/>

            <t t-set="hasLeftColumn" t-value="opt_wsale_categories or opt_wsale_attributes"/>

            <t t-set="isFilteringByPrice" t-if="opt_wsale_filter_price" t-value="float_round(available_min_price, 2) != float_round(min_price, 2) or float_round(available_max_price, 2) != float_round(max_price, 2)"/>
            <t t-set="hasPricelistDropdown" t-value="website_sale_pricelists and len(website_sale_pricelists)&gt;1"/>
            <t t-set="isSortingBy" t-value="[sort for sort in website_sale_sortable if sort[0]==request.params.get('order', '')]"/>

            <div id="wrap" class="js_sale o_wsale_products_page">
                <div class="oe_structure oe_empty oe_structure_not_nearest" id="oe_structure_website_sale_products_1"/>
                <div class="container oe_website_sale pt-2">
                    <div class="row o_wsale_products_main_row align-items-start flex-nowrap">
                        <aside t-if="hasLeftColumn" id="products_grid_before" class="d-none d-lg-block position-sticky col-3 px-3 clearfix">
                            <div class="o_wsale_products_grid_before_rail vh-100 ms-n2 mt-n2 pt-2 pe-lg-2 pb-lg-5 ps-2 overflow-y-scroll">
                                <div t-if="opt_wsale_categories" class="products_categories mb-3">
                                    <t t-call="website_sale.products_categories_list"/>
                                </div>
                                <div class="products_attributes_filters"/>
                                <t t-if="opt_wsale_filter_price and opt_wsale_attributes"
                                   t-call="website_sale.filter_products_price"/>
                            </div>
                        </aside>
                        <div id="products_grid"
                             t-attf-class="#{'o_wsale_layout_list' if layout_mode == 'list' else ''} {{'col-lg-9' if hasLeftColumn else 'col-12'}}">
                            <t t-call="website_sale.products_breadcrumb">
                                <t t-set="_classes" t-valuef="d-none d-lg-flex w-100 p-0 small"/>
                            </t>
                            <div class="products_header btn-toolbar flex-nowrap align-items-center justify-content-between gap-3 mb-3">
                                <t t-if="is_view_active('website_sale.search')" t-call="website_sale.search">
                                    <t t-set="search" t-value="original_search or search"/>
                                    <t t-set="_form_classes" t-valuef="d-lg-inline {{'d-inline' if not category else 'd-none'}}"/>
                                </t>

                                <t t-call="website_sale.pricelist_list" t-cache="pricelist">
                                    <t t-set="_classes" t-valuef="d-none d-lg-inline"/>
                                </t>

                                <t t-if="is_view_active('website_sale.sort')" t-call="website_sale.sort">
                                    <t t-set="_classes" t-valuef="d-none me-auto d-lg-inline-block"/>
                                </t>

                                <div t-if="category" class="d-flex align-items-center d-lg-none me-auto">
                                    <t t-if="not category.parent_id" t-set="backUrl" t-valuef="/shop"/>
                                    <t t-else="" t-set="backUrl" t-value="keep('/shop/category/' + slug(category.parent_id), category=0)"/>

                                    <a t-attf-class="btn btn-{{navClass}} me-2" t-att-href="category.parent_id and keep('/shop/category/' + slug(category.parent_id), category=0) or '/shop'">
                                        <i class="fa fa-angle-left"/>
                                    </a>
                                    <h4 t-out="category.name" class="mb-0 me-auto"/>
                                </div>

                                <t t-if="is_view_active('website_sale.add_grid_or_list_option')" t-call="website_sale.add_grid_or_list_option">
                                    <t t-set="_classes" t-valuef="d-flex"/>
                                </t>

                                <button t-if="is_view_active('website_sale.sort') or opt_wsale_categories or opt_wsale_attributes or opt_wsale_attributes_top"
                                        t-attf-class="btn btn-{{navClass}} position-relative {{not opt_wsale_attributes_top and 'd-lg-none'}}"
                                        data-bs-toggle="offcanvas"
                                        data-bs-target="#o_wsale_offcanvas">
                                    <i class="fa fa-sliders"/>
                                    <span t-if="isFilteringByPrice or attrib_set or tags" t-attf-class="position-absolute top-0 start-100 translate-middle border border-{{navClass}} rounded-circle bg-danger p-1"><span class="visually-hidden">filters active</span></span>
                                </button>
                            </div>

                            <t t-if="opt_wsale_categories_top" t-call="website_sale.filmstrip_categories"/>

                            <div t-if="original_search and products" class="alert alert-warning mt8">
                                No results found for '<span t-esc="original_search"/>'. Showing results for '<span t-esc="search"/>'.
                            </div>

                            <t t-if="category">
                                <t t-set='editor_msg'>Drag building blocks here to customize the header for "<t t-esc='category.name'/>" category.</t>
                                <div class="mb16" id="category_header" t-att-data-editor-message="editor_msg" t-field="category.website_description"/>
                            </t>

                            <div t-if="products" class="o_wsale_products_grid_table_wrapper pt-3 pt-lg-0">
                                <table class="table table-borderless h-100 m-0" t-att-data-ppg="ppg" t-att-data-ppr="ppr" t-att-data-default-sort="website.shop_default_sort" t-att-data-name="grid_block_name">
                                    <colgroup t-ignore="true">
                                        <!-- Force the number of columns (useful when only one row of (x < ppr) products) -->
                                        <col t-foreach="ppr" t-as="p"/>
                                    </colgroup>
                                    <tbody>
                                        <tr t-foreach="bins" t-as="tr_product">
                                            <t t-foreach="tr_product" t-as="td_product">
                                                <t t-if="td_product">
                                                    <!-- We use t-attf-class here to allow easier customization -->
                                                    <td t-att-colspan="td_product['x'] != 1 and td_product['x']"
                                                        t-att-rowspan="td_product['y'] != 1 and td_product['y']"
                                                        t-attf-class="oe_product"
                                                        t-att-data-ribbon-id="td_product['ribbon'].id"
                                                        t-att-data-name="product_block_name">
                                                        <div t-attf-class="o_wsale_product_grid_wrapper position-relative h-100 o_wsale_product_grid_wrapper_#{td_product['x']}_#{td_product['y']}">
                                                            <t t-call="website_sale.products_item">
                                                                <t t-set="product" t-value="td_product['product']"/>
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
                            <div t-nocache="get the actual search" t-else="" class="text-center text-muted mt128 mb256">
                                <t t-if="not search">
                                    <h3 class="mt8">No product defined</h3>
                                    <p t-if="category">No product defined in this category.</p>
                                </t>
                                <t t-else="">
                                    <h3 class="mt8">No results</h3>
                                    <p>No results for "<strong t-esc='search'/>"<t t-if="category"> in category "<strong t-esc="category.display_name"/>"</t>.</p>
                                </t>
                                <p t-ignore="true" groups="sales_team.group_sale_manager">Click <i>'New'</i> in the top-right corner to create your first product.</p>
                            </div>
                            <div class="products_pager d-flex justify-content-center pt-5 pb-3">
                                <t t-call="website.pager"/>
                            </div>
                        </div>
                    </div>

                    <t t-call="website_sale.o_wsale_offcanvas"/>
                </div>
                <div class="oe_structure oe_empty oe_structure_not_nearest" id="oe_structure_website_sale_products_2"/>
            </div>
        </t>
    </template>

    <!-- Add the fiscal position in the t-cache key after all overrides -->
    <template id="products_fiscal_position" inherit_id="website_sale.products" priority="99">
        <!-- TODO: Remove this template in master -->
    </template>

    <!-- (Option) Products: Enable "Card" or "Thumbnails" designs -->
    <template id="products_design_card" name="Card Design" inherit_id="website_sale.products" active="False">
        <xpath expr="//table" position="attributes">
            <attribute name="class" add="o_wsale_design_cards" separator=" "/>
        </xpath>
    </template>
    <template id="products_design_thumbs" name="Thumbnails Design" inherit_id="website_sale.products" active="False">
        <xpath expr="//table" position="attributes">
            <attribute name="class" add="o_wsale_design_thumbs" separator=" "/>
        </xpath>
    </template>
    <template id="products_design_grid" name="Grid Design" inherit_id="website_sale.products" active="False">
        <xpath expr="//table" position="attributes">
            <attribute name="class" add="o_wsale_design_grid" separator=" "/>
        </xpath>
    </template>

    <!-- (Options) Products: Define products' default image ratio -->
    <template id="products_thumb_4_3" name="thumb_4_3" inherit_id="website_sale.products" active="False">
        <xpath expr="//table" position="attributes">
            <attribute name="class" add="o_wsale_context_thumb_4_3" separator=" "/>
        </xpath>
    </template>
    <template id="products_thumb_4_5" name="thumb_4_5" inherit_id="website_sale.products" active="False">
        <xpath expr="//table" position="attributes">
            <attribute name="class" add="o_wsale_context_thumb_4_5" separator=" "/>
        </xpath>
    </template>
    <template id="products_thumb_2_3" name="thumb_2_3" inherit_id="website_sale.products" active="False">
        <xpath expr="//table" position="attributes">
            <attribute name="class" add="o_wsale_context_thumb_2_3" separator=" "/>
        </xpath>
    </template>

    <!-- (Option) Products: Define products' images filling mode -->
    <template id="products_thumb_cover" name="thumb_cover" inherit_id="website_sale.products" active="True">
        <xpath expr="//table" position="attributes">
            <attribute name="class" add="o_wsale_context_thumb_cover" separator=" "/>
        </xpath>
    </template>

    <template id="website_sale.sort" name="Sort-by Template">
        <div t-attf-class="o_sortby_dropdown dropdown dropdown_sorty_by {{_classes}}">
            <small class="d-none d-lg-inline text-muted">Sort By:</small>
            <a role="button" href="#" t-attf-class="dropdown-toggle btn btn-{{navClass}}" data-bs-toggle="dropdown">
                <span class="d-none d-lg-inline">
                    <t t-if="isSortingBy" t-out="isSortingBy[0][1]"/>
                    <span t-else="1" t-field="website.shop_default_sort"/>
                </span>
                <i class="fa fa-sort-amount-asc d-lg-none"/>
            </a>
            <div class="dropdown-menu dropdown-menu-end" role="menu">
                <t t-foreach="website_sale_sortable" t-as="sortby">
                    <a role="menuitem" rel="noindex,nofollow" t-att-href="keep('/shop', order=sortby[0])" class="dropdown-item">
                        <span t-out="sortby[1]"/>
                    </a>
                </t>
            </div>
        </div>
    </template>

    <template id="website_sale.add_grid_or_list_option" active="True" name="Grid or List button">
        <t t-set="_activeClasses" t-translation="off">active</t>
        <div t-attf-class="o_wsale_apply_layout btn-group {{_classes}}" t-att-data-active-classes="_activeClasses">
            <input type="radio" class="btn-check" name="wsale_products_layout" id="o_wsale_apply_grid"  t-att-checked="'checked' if layout_mode != 'list' else None" value="grid"/>
            <label t-attf-class="btn btn-{{navClass}} #{_activeClasses if layout_mode != 'list' else None} o_wsale_apply_grid" title="Grid" for="o_wsale_apply_grid">
                <i class="fa fa-th-large"/>
            </label>
            <input type="radio" class="btn-check" name="wsale_products_layout" id="o_wsale_apply_list" t-att-checked="'checked' if layout_mode == 'list' else None" value="list"/>
            <label t-attf-class="btn btn-{{navClass}} #{_activeClasses if layout_mode == 'list' else None} o_wsale_apply_list" title="List" for="o_wsale_apply_list">
                <i class="oi oi-view-list"/>
            </label>
        </div>
    </template>

    <template id="website_sale.products_categories" active="False" name="Categories in Left Side "/>
    <template id="website_sale.products_categories_top" active="True" name="Categories in top-nav"/>
    <template id="website_sale.products_attributes_top" active="False" name="Attributes in top-nav"/>

    <template id="o_wsale_offcanvas_color_attribute" name="Color type attribute in filter">
        <t t-foreach="a.value_ids" t-as="v">
            <t t-set="img_style"
               t-value="'background:url(/web/image/product.attribute.value/%s/image); background-size:cover;' % v.id if v.image else ''"
            />
            <t t-set="color_style"
               t-value="'background: ' + str(v.html_color or v.name if not v.is_custom else '')"
            />
            <label t-attf-style="#{img_style or color_style}"
                   t-attf-class="css_attribute_color mb-1 #{'active' if v.id in attrib_set else ''}"
            >
                <input type="checkbox"
                       name="attrib"
                       t-att-value="'%s-%s' % (a.id, v.id)"
                       t-att-checked="'checked' if v.id in attrib_set else None"
                       t-att-title="v.name"
                />
            </label>
        </t>
    </template>

    <!-- OffCanvas Nav -->
    <template id="website_sale.o_wsale_offcanvas" name="Offcanvas">
        <aside id="o_wsale_offcanvas"
               class="o_website_offcanvas offcanvas offcanvas-end p-0">
            <div class="offcanvas-header justify-content-end">
                <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"/>
            </div>
            <div t-if="category" class="offcanvas-body d-lg-none flex-grow-0 overflow-visible">
                <t t-call="website_sale.search">
                    <t t-set="search" t-value="original_search or search"/>
                    <t t-set="_s_searchbar_autocomplete_classes" t-valuef="bg-primary"> </t>
                </t>
            </div>
            <div id="o_wsale_offcanvas_content" class="accordion accordion-flush flex-grow-1 overflow-auto">
                <div class="d-block d-lg-none accordion-item" t-if="hasPricelistDropdown">
                    <h2 id="o_wsale_offcanvas_orderby_header" class="accordion-header mb-0">
                        <button class="o_wsale_offcanvas_title accordion-button rounded-0 collapsed"
                            type="button"
                            data-bs-toggle="collapse"
                            data-bs-target="#o_wsale_offcanvas_pricelist"
                            aria-expanded="false"
                            aria-controls="o_wsale_offcanvas_pricelist">
                            <b>Pricelist</b>
                        </button>
                    </h2>
                    <t t-set="curr_pl" t-value="website.pricelist_id"/>
                    <div id="o_wsale_offcanvas_pricelist"
                        class="accordion-collapse collapse"
                        aria-labelledby="o_wsale_offcanvas_orderby_header">
                        <div class="accordion-body pt-0">
                            <div class="list-group list-group-flush">
                                <a t-foreach="website_sale_pricelists" t-as="pl"
                                    role="menuitem"
                                    rel="noindex,nofollow"
                                    t-att-href="'/shop/change_pricelist/%s' % pl.id"
                                    class="list-group-item border-0 ps-0 pb-0"
                                    >
                                    <div class="form-check d-inline-block">
                                        <input type="radio"
                                            t-attf-onclick="location.href='/shop/change_pricelist/#{pl.id}';"
                                            class="form-check-input o_not_editable"
                                            name="wsale_pricelist_radios_offcanvas"
                                            t-att-checked="curr_pl == pl">
                                            <label class="form-check-label fw-normal" t-out="pl.name"/>
                                        </input>
                                    </div>
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
                <div t-if="is_view_active('website_sale.sort')" class="accordion-item">
                    <t t-if="isSortingBy" t-set="isSortingBy" t-value="isSortingBy[0][1]"/>
                    <t t-else="" t-set="isSortingBy" t-value="website.shop_default_sort"/>
                    <h2 id="o_wsale_offcanvas_orderby_header" class="accordion-header mb-0">
                        <button class="o_wsale_offcanvas_title accordion-button rounded-0 collapsed"
                                type="button"
                                data-bs-toggle="collapse"
                                data-bs-target="#o_wsale_offcanvas_orderby"
                                aria-expanded="false"
                                aria-controls="o_wsale_offcanvas_orderby">
                                <b>Sort By</b>
                        </button>
                    </h2>
                    <div id="o_wsale_offcanvas_orderby"
                         class="accordion-collapse collapse"
                         aria-labelledby="o_wsale_offcanvas_orderby_header">
                        <div class="accordion-body pt-0">
                            <div class="list-group list-group-flush">
                                <a t-foreach="website_sale_sortable" t-as="sortby"
                                   role="menuitem"
                                   rel="noindex,nofollow"
                                   t-att-href="keep('/shop', order=sortby[0])"
                                   class="list-group-item border-0 ps-0 pb-0">
                                    <div class="form-check d-inline-block">
                                        <input type="radio"
                                               t-attf-onclick="location.href='#{keep('/shop', order=sortby[0])}';"
                                               class="form-check-input o_not_editable"
                                               name="wsale_sortby_radios_offcanvas"
                                               t-att-checked="isSortingBy and isSortingBy == sortby[1]">
                                            <label class="form-check-label fw-normal" t-out="sortby[1]"/>
                                        </input>
                                    </div>
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
                <div t-if="opt_wsale_categories"
                     class="accordion-item">
                    <h2 id="o_wsale_offcanvas_categories_header" class="accordion-header mb-0">
                        <button class="o_wsale_offcanvas_title accordion-button rounded-0 collapsed"
                                type="button"
                                data-bs-toggle="collapse"
                                data-bs-target="#o_wsale_offcanvas_categories"
                                aria-expanded="false"
                                aria-controls="o_wsale_offcanvas_categories">
                                <b>Categories</b>
                        </button>
                    </h2>
                    <div id="o_wsale_offcanvas_categories"
                         class="accordion-collapse collapse"
                         aria-labelledby="o_wsale_offcanvas_categories_header">
                        <div class="accordion-body pt-0">
                            <t t-call="website_sale.products_categories_list">
                                <t t-set="isOffcanvas" t-value="true"/>
                                <t t-set="_titleClasses" t-valuef="d-none"/>
                                <t t-set="_radioGroup" t-valuef="_offcanvas"/>
                            </t>
                        </div>
                    </div>
                </div>

                <form t-if="opt_wsale_attributes or opt_wsale_attributes_top"
                      t-attf-class="js_attributes d-flex flex-column"
                      method="get">
                    <input t-if="category" type="hidden" name="category" t-att-value="category.id"/>
                    <input type="hidden" name="search" t-att-value="search"/>

                    <t t-foreach="attributes" t-as="a">
                        <t t-cache="a,attrib_set">
                            <t t-set="_status" t-value="'inactive'"/>
                            <t t-foreach="a.value_ids" t-as="v" t-if="v.id in attrib_set" t-set="_status" t-value="'active'"/>

                            <div t-if="a.value_ids and len(a.value_ids) &gt; 1"
                                 t-attf-class="accordion-item border-top-0 {{(_status == 'active') and 'order-1' or 'order-2'}}">
                                <h2 class="accordion-header mb-0" t-attf-id="o_wsale_offcanvas_attribute_{{a.id}}_header">
                                    <button t-attf-class="o_wsale_offcanvas_title accordion-button rounded-0 {{ not attrib_values and 'collapsed'}}"
                                            type="button"
                                            t-att-data-status="_status"
                                            data-bs-toggle="collapse"
                                            t-attf-data-bs-target="#o_wsale_offcanvas_attribute_{{a.id}}"
                                            t-att-aria-expanded="_status == 'active' and 'True' or 'False'"
                                            t-attf-aria-controls="o_wsale_offcanvas_attribute_{{a.id}}">
                                            <b t-out="a.name"/>
                                    </button>
                                </h2>
                                <div t-attf-id="o_wsale_offcanvas_attribute_{{a.id}}"
                                     t-attf-class="accordion-collapse collapse {{ (_status == 'active') and 'show'}}"
                                     t-att-aria-expanded="(_status == 'active') and 'True' or 'False'"
                                     t-attf-aria-labelledby="o_wsale_offcanvas_attribute_{{a.id}}_header">

                                    <div class="accordion-body pt-0">
                                        <div t-if="a.display_type == 'color'" class="pt-1 pb-3">
                                            <t t-call="website_sale.o_wsale_offcanvas_color_attribute"/>
                                        </div>
                                        <div t-elif="a.display_type in ('radio', 'pills', 'select', 'multi')"
                                             class="list-group list-group-flush">
                                            <div t-foreach="a.value_ids" t-as="v" class="list-group-item border-0 ps-0 pb-0">
                                                <div class="form-check mb-1">
                                                    <input type="checkbox"
                                                           name="attrib"
                                                           class="form-check-input"
                                                           t-att-id="'%s-%s' % (a.id,v.id)"
                                                           t-att-value="'%s-%s' % (a.id,v.id)"
                                                           t-att-checked="'checked' if v.id in attrib_set else None"/>
                                                    <label class="form-check-label fw-normal" t-att-for="'%s-%s' % (a.id,v.id)" t-field="v.name"/>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </t>
                    <t t-if="opt_wsale_filter_tags and (opt_wsale_attributes or opt_wsale_attributes_top)">
                        <t t-set="_status" t-value="'inactive'"/>
                        <t t-foreach="all_tags" t-as="v" t-if="v.id in tags" t-set="_status" t-value="'active'"/>
                        <div t-if="all_tags">
                            <h2 class="accordion-header mb-0" t-attf-id="o_wsale_offcanvas_tags_header">
                                <button t-attf-class="o_wsale_offcanvas_title accordion-button border-top rounded-0 {{ not tags and 'collapsed'}}"
                                        type="button"
                                        t-att-data-status="_status"
                                        data-bs-toggle="collapse"
                                        t-attf-data-bs-target="#o_wsale_offcanvas_tags"
                                        t-att-aria-expanded="_status == 'active' and 'True' or 'False'"
                                        t-attf-aria-controls="o_wsale_offcanvas_tags"
                                >
                                    <b>Tags</b>
                                </button>
                            </h2>
                            <div t-attf-id="o_wsale_offcanvas_tags"
                                 t-attf-class="accordion-collapse collapse {{ (_status == 'active') and 'show'}}"
                                 t-att-aria-expanded="(_status == 'active') and 'True' or 'False'"
                                 t-attf-aria-labelledby="o_wsale_offcanvas_tags_header"
                            >
                                <div class="accordion-body pt-0">
                                    <div class="list-group list-group-flush">
                                        <t t-call="website_sale.filter_products_tags_list">
                                            <t t-set="all_tags" t-value="all_tags"/>
                                        </t>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </form>

                <t t-if="opt_wsale_filter_price and (opt_wsale_attributes or opt_wsale_attributes_top)"
                   t-call="website_sale.filter_products_price">
                    <t t-set="_classes" t-valuef="o_wsale_offcanvas_title px-4 border-top"/>
                    <t t-set="_classes_title" t-valuef="ms-n1 pt-3 pb-2"/>
                </t>
            </div>
            <div class="offcanvas-body d-flex justify-content-between flex-grow-0 border-top overflow-hidden">
                <a t-attf-class="btn btn-{{navClass}} d-flex py-1 mb-2 {{(not attrib_values and not isFilteringByPrice and not tags) and 'disabled' }}"
                   t-att-aria-disabled="(not attrib_values and not isFilteringByPrice and not tags) and 'true' or 'false'"
                   href="/shop"
                   title="Clear Filters">
                    Clear Filters
                </a>
            </div>
        </aside>
    </template>


    <!-- Top-Nav Categories -->
    <template id="website_sale.filmstrip_categories" name="Categories Filmstrip">
        <t t-if="category.id">
            <t t-set="entries" t-value="not search and category.child_id or category.child_id.filtered(lambda c: category.id in search_categories_ids)"/>

            <t t-if="not entries">
                <t t-set="parent" t-value="category.parent_id"/>
                <t t-set="entries" t-value="not search and parent.child_id or parent.child_id.filtered(lambda c: parent.id in search_categories_ids)"/>
            </t>
        </t>
        <t t-else="">
            <t t-set="entries" t-value="categories"/>
        </t>

        <div t-if="entries" class="o_wsale_filmstip_container d-flex align-items-stretch mb-2 overflow-hidden">
            <div class="o_wsale_filmstip_wrapper pb-1 overflow-auto">
                <ul class="o_wsale_filmstip d-flex align-items-stretch mb-0 list-unstyled overflow-visible">
                    <t t-foreach="entries" t-as="c" t-if="c.image_128" t-set="atLeastOneImage" t-value="True"/>
                    <t t-if="category.parent_id" t-set="backUrl" t-value="keep('/shop/category/' + slug(category.parent_id), category=0)"/>
                    <t t-else="" t-set="backUrl" t-value="'/shop'"/>

                    <li t-foreach="entries" t-as="c"
                        t-attf-class="d-flex {{'pe-3' if not c_last else ''}}"
                        t-att-data-link-href="keep('/shop/category/' + slug(c), category=0)">
                        <input type="radio" t-attf-name="wsale_categories_top_radios_{{parentCategoryId}}" class="btn-check pe-none" t-att-id="c.id" t-att-value="c.id" t-att-checked="'true' if c.id == category.id else None"/>

                        <div t-attf-class=" btn btn-{{navClass}} d-flex align-items-center {{'ps-2 pe-3' if c.image_128 else 'px-4'}} fs-6 fw-normal {{ 'border-primary' if c.id == category.id else '' }}" t-att-for="c.id">
                            <div t-if="c.image_128"
                                 t-attf-style="background-image:url('data:image/png;base64,#{c.image_128}')"
                                 class="o_image_40_cover oe_img_bg o_bg_img_center rounded-3 me-3"
                                 t-att-alt="c.name "/>
                            <span t-field="c.name"/>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </template>

    <!-- Add to cart button-->
    <template id="categories_recursive" name="Category list">
        <li class="nav-item mb-1">
            <t t-call="website_sale.categorie_link"/>
            <ul t-if="c.child_id" class="nav flex-column nav-hierarchy mt-1 ps-3">
                <t t-foreach="c.child_id" t-as="c">
                    <t t-if="not search or c.id in search_categories_ids">
                        <t t-call="website_sale.categories_recursive" />
                    </t>
                </t>
            </ul>
        </li>
    </template>

    <template id="categorie_link" name="Category Link">
        <t t-if="isOffcanvas" t-set="parentCategoryId" t-valuef="offcanvas"/>

        <div t-att-data-link-href="keep('/shop/category/' + slug(c), category=0)" class="form-check d-inline-block">
            <input type="radio" t-attf-name="wsale_categories_radios_{{parentCategoryId}}" class="form-check-input pe-none" t-att-id="c.id" t-att-value="c.id" t-att-checked="'true' if c.id == category.id else None"/>
            <label class="form-check-label fw-normal" t-att-for="c.id" t-field="c.name"/>
        </div>
    </template>

    <template id="website_sale.products_categories_list" active="True" name="eCommerce Categories">
        <h6 t-attf-class="o_categories_collapse_title mb-3 {{_titleClasses}}"><b>Categories</b></h6>

        <div class="wsale_products_categories_list">
            <ul class="nav d-flex flex-column my-2">
                <li class="nav-item mb-1">
                    <div t-att-data-link-href="keep('/shop', category=0)" class="form-check d-inline-block">
                        <input type="radio" t-attf-name="wsale_categories_radios{{_radioGroup}}" class="form-check-input pe-none o_not_editable" t-att-id="all_products" t-att-value="all_products" t-att-checked="'true' if not category else None"/>
                        <label class="form-check-label fw-normal" t-att-for="all_products">All Products</label>
                    </div>
                </li>
                <t t-foreach="categories" t-as="c">
                    <t t-call="website_sale.categories_recursive"/>
                </t>
            </ul>
        </div>
    </template>

    <template id="option_collapse_categories_recursive" name="Collapse Category Recursive">
        <t t-set="children" t-value="not search and c.child_id or c.child_id.filtered(lambda c: c.id in search_categories_ids)"/>

        <t t-if="children">
            <t t-set="isOpen" t-value="c.id in category.parents_and_self.ids"/>

            <li class="nav-item">
                <div class="accordion-header d-flex mb-1">
                    <t t-call="website_sale.categorie_link"/>
                    <button t-attf-id="o_wsale_cat_accordion_title_{{c.id}}"
                            t-attf-class="accordion-button p-0 ms-3 {{not isOpen and 'collapsed'}} w-auto flex-grow-1 bg-transparent shadow-none"
                            t-attf-data-bs-target="#o_wsale_cat_accordion_{{c.id}}"
                            t-att-aria-expanded="isOpen and 'true' or 'false'"
                            t-attf-aria-controls="o_wsale_cat_accordion_{{c.id}}"
                            data-bs-toggle="collapse"
                            type="button"/>
                </div>
                <ul t-attf-id="o_wsale_cat_accordion_{{c.id}}"
                    t-attf-class="accordion-collapse list-unstyled ps-2 pb-2 collapse {{isOpen and 'show'}}"
                    t-attf-aria-labelledby="o_wsale_cat_accordion_title_{{c.id}}">
                    <t t-set="parentCategoryId" t-value="c.id"/>
                    <t t-if="isOffcanvas" t-set="parentCategoryId" t-valuef="offcanvas_{{c.id}}"/>

                    <t t-foreach="children" t-as="c">
                        <t t-call="website_sale.option_collapse_categories_recursive"/>
                    </t>
                </ul>
            </li>
        </t>

        <li t-else="" class="nav-item mb-1">
            <t t-if="isOffcanvas" t-set="parentCategoryId" t-valuef="offcanvas"/>
            <div class="d-flex flex-wrap justify-content-between align-items-center">
                <t t-call="website_sale.categorie_link"/>
            </div>
        </li>
    </template>

    <template id="option_collapse_products_categories" name="Collapsible Category List" inherit_id="website_sale.products_categories_list" active="False">
        <xpath expr="//div[hasclass('wsale_products_categories_list')]" position="attributes">
            <attribute name="class" add="o_shop_collapse_category" separator=" "/>
        </xpath>
        <xpath expr="//t[@t-call='website_sale.categories_recursive']" position="attributes">
            <attribute name="t-call">website_sale.option_collapse_categories_recursive</attribute>
        </xpath>
    </template>

    <template id="products_attributes" inherit_id="website_sale.products" active="True" name="Attributes &amp; Variants filters">
        <xpath expr="//div[hasclass('products_attributes_filters')]" position="inside">
            <div id="wsale_products_attributes_collapse"
                 class=" position-relative">
                <form t-if="attributes or all_tags" class="js_attributes position-relative mb-2" method="get">
                    <input t-if="category" type="hidden" name="category" t-att-value="category.id" />
                    <input type="hidden" name="search" t-att-value="search" />
                    <input type="hidden" name="order" t-att-value="order"/>
                    <a t-if="attrib_values or tags" t-att-href="keep('/shop'+ ('/category/'+slug(category)) if category else None, attrib=0, tags=0)" t-attf-class="btn btn-{{navClass}} d-flex align-items-center py-1 mb-2">
                        <small class="mx-auto"><b>Clear Filters</b></small>
                        <i class="oi oi-close"/>
                    </a>
                    <t t-foreach="attributes" t-as="a">
                        <t t-cache="a,attrib_set">
                            <div class="accordion-item nav-item mb-1 border-0" t-if="a.value_ids and len(a.value_ids) &gt; 1">
                                <h6 class="mb-3">
                                    <b class="o_products_attributes_title d-none d-lg-block" t-field="a.name"/>
                                </h6>
                                <div t-attf-id="o_products_attributes_{{a.id}}" class="">
                                    <t t-if="a.display_type == 'select'">
                                        <select class="form-select css_attribute_select mb-2" name="attrib">
                                            <option value="" selected="true">-</option>
                                            <t t-foreach="a.value_ids" t-as="v">
                                                <option t-att-value="'%s-%s' % (a.id,v.id)" t-esc="v.name" t-att-selected="v.id in attrib_set" />
                                            </t>
                                        </select>
                                    </t>
                                    <div t-elif="a.display_type == 'color'" class="mb-3">
                                        <t t-call="website_sale.o_wsale_offcanvas_color_attribute"/>
                                    </div>
                                    <div t-elif="a.display_type in ('radio', 'pills', 'multi')" class="flex-column mb-3">
                                        <t t-foreach="a.value_ids" t-as="v">
                                            <div class="form-check mb-1">
                                                <input type="checkbox"
                                                       name="attrib"
                                                       class="form-check-input"
                                                       t-att-id="'%s-%s' % (a.id,v.id)"
                                                       t-att-value="'%s-%s' % (a.id,v.id)"
                                                       t-att-checked="'checked' if v.id in attrib_set else None"/>
                                                <label class="form-check-label fw-normal" t-att-for="'%s-%s' % (a.id,v.id)" t-field="v.name"/>
                                            </div>
                                        </t>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </t>
                    <t t-if="opt_wsale_filter_tags and opt_wsale_attributes"
                       t-call="website_sale.filter_products_tags"
                    >
                        <t t-set="all_tags" t-value="all_tags"/>
                    </t>
                </form>
            </div>
        </xpath>
    </template>

    <template id="filter_products_price" name="Filter by Prices" active="False">
        <t t-set="isDisabled" t-value="available_min_price == available_max_price"/>
        <div id="o_wsale_price_range_option"
             t-attf-class="position-relative {{_classes}} {{isDisabled and 'opacity-75 pe-none user-select-none'}}">
            <label t-attf-class="m-0 h6 o_products_attributes_title {{_classes_title}}">
                <b>Price Range</b>
            </label>
            <input type="range" multiple="multiple"
                   t-attf-class="form-range range-with-input {{_classes_input}}"
                   t-att-data-currency="website.currency_id.symbol"
                   t-att-data-currency-position="website.currency_id.position"
                   t-att-step="website.currency_id.rounding" t-att-min="'%f' % (available_min_price)"
                   t-att-max="'%f' % (available_max_price)" t-att-value="'%f,%f' % (min_price, max_price)"/>
        </div>
    </template>

    <template id="filter_products_tags" name="Filter by Tags" active="True">
        <div t-if="all_tags">
            <h6 class="mb-3">
                <b>Tags</b>
            </h6>
            <div class="flex-column mb-3">
                <t t-call="website_sale.filter_products_tags_list">
                    <t t-set="all_tags" t-value="all_tags"/>
                </t>
            </div>
        </div>
    </template>

    <template id="filter_products_tags_list">
        <t t-foreach="all_tags" t-as="tag" class="list-group-item border-0 ps-0 pb-0">
            <div class="form-check mb-1">
                <input type="checkbox"
                       name="tags"
                       class="form-check-input"
                       t-attf-id="tag_#{tag.id}"
                       t-att-value="tag.id"
                       t-att-checked="'checked' if tag.id in tags else None"
                />
                <label class="form-check-label fw-normal" t-attf-for="tag_#{tag.id}" t-field="tag.name"/>
            </div>
        </t>
    </template>

    <template id="products_list_view" inherit_id="website_sale.products" active="False" name="List View (by default)">
        <xpath expr="//div[@id='products_grid']" position="after">
            <!-- Nothing to do, this view is only meant to allow the server -->
            <!-- to know if the list view layout should be used -->
        </xpath>
    </template>

    <!-- /shop/product page -->
    <template id="base_unit_price" name="Product Base unit price">
        (<span class="o_base_unit_price" t-esc="combination_info['base_unit_price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
         / <span class="oe_custom_base_unit" t-field="product.base_unit_name"/>)
    </template>

    <template id="product" name="Product" track="1">
        <!-- Qweb variable defining the class suffix for navbar items.
             Change accordingly to the derired visual result (eg. `primary`, `dark`...)-->
        <t t-set="navClass" t-valuef="light"/>

    <t t-cache="pricelist,product,fiscal_position">
        <!-- TODO drop _get_first_possible_combination here -->
        <t t-set="combination" t-value="product._get_first_possible_combination()"/>
        <t t-set="combination_info" t-value="product._get_combination_info(combination, add_qty=add_qty)"/>
        <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

        <t t-call="website.layout">
            <t t-set="additional_title" t-value="product.name" />
            <div itemscope="itemscope" itemtype="http://schema.org/Product" id="wrap" class="js_sale o_wsale_product_page">
                <div class="oe_structure oe_empty oe_structure_not_nearest" id="oe_structure_website_sale_product_1" data-editor-message="DROP BUILDING BLOCKS HERE TO MAKE THEM AVAILABLE ACROSS ALL PRODUCTS"/>
                <section id="product_detail"
                         t-attf-class="container py-4 oe_website_sale #{'discount' if combination_info['has_discounted_price'] else ''}"
                         t-att-data-view-track="view_track and '1' or '0'"
                         t-att-data-product-tracking-info="'product_tracking_info' in combination_info and json.dumps(combination_info['product_tracking_info'])"
                >
                    <div class="row align-items-center">
                        <div class="col-lg-6 d-flex align-items-center">
                            <div class="d-flex justify-content-between w-100">
                                <t t-if="is_view_active('website_sale.search')" t-call="website_sale.search">
                                    <t t-set="search" t-value="False"/>
                                    <t t-set="_form_classes" t-valuef="mb-2 mb-lg-0"/>
                                    <t t-set="_classes" t-value="'me-sm-2'"/>
                                </t>
                                <t t-set="website_sale_pricelists" t-value="website.get_pricelist_available(show_visible=True)" />
                                <t t-set="hasPricelistDropdown" t-value="website_sale_pricelists and len(website_sale_pricelists)&gt;1"/>
                                <t t-call="website_sale.pricelist_list">
                                    <t t-set="_classes" t-valuef="d-lg-inline ms-2"/>
                                </t>
                            </div>
                        </div>
                        <div class="col-lg-6 d-flex align-items-center">
                            <ol class="breadcrumb p-0 mb-2 m-lg-0">
                                <li class="breadcrumb-item o_not_editable">
                                    <a t-att-href="keep(category=0)">All Products</a>
                                </li>
                                <li t-nocache="The category does not have to be cached, as the product can be accessed via different paths."
                                    t-if="category" class="breadcrumb-item">
                                    <a t-att-href="keep('/shop/category/%s' % slug(category), category=0)" t-field="category.name" />
                                </li>
                                <li class="breadcrumb-item active">
                                    <span t-field="product.name" />
                                </li>
                            </ol>
                        </div>
                    </div>
                    <div class="row" id="product_detail_main" data-name="Product Page"
                        t-att-data-image_width="website.product_page_image_width"
                        t-att-data-image_layout="website.product_page_image_layout">
                        <t t-set="image_cols" t-value="website._get_product_page_proportions()"/>
                        <div t-attf-class="col-lg-#{image_cols[0]} mt-lg-4 o_wsale_product_images position-relative" t-if="website.product_page_image_width != 'none'">
                            <t t-call="website_sale.shop_product_images"/>
                        </div>
                        <div t-attf-class="col-lg-#{image_cols[1]} mt-md-4" id="product_details">
                            <t t-set="base_url" t-value="website.get_base_url()"/>
                            <!-- TODO: remove next line in master. Stable fix not to break custos. -->
                            <t t-if="False" t-set="base_url" t-value="product.get_base_url()"/>
                            <h1 itemprop="name" t-field="product.name">Product Name</h1>
                            <span itemprop="url" style="display:none;" t-esc="base_url + product.website_url"/>
                            <span itemprop="image" style="display:none;" t-esc="base_url + website.image_url(product, 'image_1920')" />
                            <t t-if="is_view_active('website_sale.product_comment')">
                                <a href="#o_product_page_reviews" class="o_product_page_reviews_link text-decoration-none">
                                    <t t-call="portal_rating.rating_widget_stars_static">
                                        <t t-set="rating_avg" t-value="product.rating_avg"/>
                                        <t t-set="trans_text_plural">%s reviews</t>
                                        <t t-set="trans_text_singular">%s review</t>
                                        <t t-set="rating_count" t-value="(trans_text_plural if product.rating_count > 1 else trans_text_singular) % product.rating_count"/>
                                    </t>
                                </a>
                            </t>
                            <p t-field="product.description_sale" class="text-muted my-2" placeholder="A short description that will also appear on documents." />
                            <div t-field="product.description_ecommerce" class="oe_structure"
                                placeholder="A detailed, formatted description to promote your product on this page. Use '/' to discover more features."/>
                            <form t-if="product._is_add_to_cart_possible()" action="/shop/cart/update" method="POST">
                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" t-nocache="The csrf token must always be up to date."/>
                                <div class="js_product js_main_product mb-3">
                                    <div>
                                        <t t-call="website_sale.product_price"/>
                                        <small t-if="'base_unit_price' in combination_info"
                                               class="ms-1 text-muted o_base_unit_price_wrapper d-none" groups="website_sale.group_show_uom_price">
                                            <t t-call='website_sale.base_unit_price'/>
                                        </small>
                                    </div>
                                    <t t-placeholder="select">
                                        <input type="hidden" class="product_id" name="product_id" t-att-value="product_variant.id" />
                                        <input type="hidden" class="product_template_id" name="product_template_id" t-att-value="product.id" />
                                        <input t-if="product.public_categ_ids.ids" type="hidden" class="product_category_id" name="product_category_id" t-att-value="product.public_categ_ids.ids[0]" />
                                        <t t-call="website_sale.variants">
                                            <t t-set="ul_class" t-valuef="flex-column" />
                                            <t t-set="parent_combination" t-value="None" />
                                        </t>
                                    </t>
                                    <p t-if="True" class="css_not_available_msg alert alert-warning">This combination does not exist.</p>
                                    <div id="o_wsale_cta_wrapper" class="d-flex flex-wrap align-items-center">
                                        <t t-set="hasQuantities" t-value="false"/>
                                        <t t-set="hasBuyNow" t-value="false"/>
                                        <!-- TODO: remove line below in master -->
                                        <t t-set="ctaSizeBig" t-value="not hasQuantities or not hasBuyNow"/>

                                        <div id="add_to_cart_wrap" t-attf-class="{{'d-none' if combination_info['prevent_zero_price_sale'] else 'd-inline-flex'}} align-items-center mb-2 me-auto">
                                            <a data-animation-selector=".o_wsale_product_images" role="button" id="add_to_cart" t-attf-class="btn btn-primary js_check_product a-submit flex-grow-1" href="#">
                                                <i class="fa fa-shopping-cart me-2"/>
                                                Add to cart
                                            </a>
                                        </div>
                                        <div id="product_option_block" class="d-flex flex-wrap w-100"/>
                                    </div>
                                    <div id="contact_us_wrapper"
                                         t-attf-class="{{'d-flex' if combination_info['prevent_zero_price_sale'] else 'd-none'}} oe_structure oe_structure_solo #{_div_classes}">
                                        <section class="s_text_block" data-snippet="s_text_block" data-name="Text">
                                            <div class="container">
                                                <a t-att-href="website.contact_us_button_url"
                                                   class="btn btn-primary btn_cta">Contact Us
                                                </a>
                                            </div>
                                        </section>
                                    </div>
                                    <t t-if="is_view_active('website_sale.product_tags')" t-call="website_sale.product_tags">
                                        <t t-set="all_product_tags" t-value="product_variant.all_product_tag_ids"/>
                                    </t>
                                </div>
                            </form>
                            <p t-elif="not product.active" class="alert alert-warning">This product is no longer available.</p>
                            <p t-else="" class="alert alert-warning">This product has no valid combination.</p>
                            <div id="product_attributes_simple">
                                <t t-set="single_value_attributes" t-value="product.valid_product_template_attribute_line_ids._prepare_single_value_for_display()"/>
                                <table t-attf-class="table table-sm text-muted {{'' if single_value_attributes else 'd-none'}}">
                                    <t t-foreach="single_value_attributes" t-as="attribute">
                                        <tr>
                                            <td>
                                                <span t-field="attribute.name"/>:
                                                <t t-foreach="single_value_attributes[attribute]" t-as="ptal">
                                                    <span t-field="ptal.product_template_value_ids._only_active().name"/><t t-if="not ptal_last">, </t>
                                                </t>
                                            </td>
                                        </tr>
                                    </t>
                                </table>
                            </div>
                            <t t-set="product_documents" t-value="product.sudo().product_document_ids.filtered(lambda doc: doc.shown_on_product_page)"/>
                            <div id="product_documents" class="my-2" t-if="product_documents">
                                <h5>Documents</h5>
                                <t t-foreach="product_documents" t-as="document_sudo">
                                    <t t-set="attachment_sudo" t-value="document_sudo.ir_attachment_id"/>
                                    <t t-set="target" t-value="attachment_sudo.type == 'url' and '_blank' or '_self'"/>
                                    <t t-set="icon" t-value="attachment_sudo.type == 'url' and 'fa-link' or 'fa-download'"/>
                                    <div>
                                        <a t-att-href="'/shop/' + slug(product) + '/document/' + str(document_sudo.id)" t-att-target="target">
                                            <i t-att-class="'fa ' + icon"/>
                                            <t t-out="attachment_sudo.name"/>
                                        </a>
                                    </div>
                                </t>
                            </div>
                            <div id="o_product_terms_and_share" class="d-flex justify-content-between flex-column flex-md-row align-items-md-end mb-3">
                            </div>
                        </div>
                    </div>
                </section>
                <div itemprop="description" t-field="product.website_description" class="oe_structure oe_empty mt16" id="product_full_description"/>
                <div class="oe_structure oe_empty oe_structure_not_nearest mt16" id="oe_structure_website_sale_product_2" data-editor-message="DROP BUILDING BLOCKS HERE TO MAKE THEM AVAILABLE ACROSS ALL PRODUCTS"/>
            </div>
        </t>
    </t>
    </template>

    <template id="product_tags" name="Product Tags" active="True">
        <div class="o_product_tags o_field_tags d-flex flex-wrap align-items-center gap-2">
            <t t-foreach="all_product_tags" t-as="tag">
                <t t-if="tag.visible_on_ecommerce">
                    <span t-if="tag.image"
                          class="order-0"
                          t-field="tag.image"
                          t-options="{'widget': 'image', 'class': 'o_product_tag_img rounded'}"
                    />
                    <span t-else="" class="position-relative order-1 py-1 px-2">
                        <span class="position-absolute top-0 start-0 w-100 h-100 rounded"
                              t-attf-style="background-color: #{tag.color}; opacity: .2;"
                        />
                        <span class="text-nowrap small"
                              t-attf-style="color: #{tag.color}"
                              t-field="tag.name"
                        />
                    </span>
                </t>
            </t>
        </div>
    </template>

    <template id="alternative_products" name="Alternative Products" inherit_id="website_sale.product" active="True">
        <div itemprop="description" position="after">
            <div class="oe_structure oe_structure_solo oe_unremovable oe_unmovable" id="oe_structure_website_sale_recommended_products" t-ignore="true" t-if="product.alternative_product_ids">
                <section data-snippet="s_dynamic_snippet_products"
                    class="oe_unmovable oe_unremovable s_dynamic_snippet_products o_wsale_alternative_products s_dynamic pt32 pb32 o_colored_level s_product_product_borderless_1"
                    data-name="Alternative Products" style="background-image: none;" t-att-data-filter-id="product._get_alternative_product_filter()"
                    data-template-key="website_sale.dynamic_filter_template_product_product_borderless_1" data-product-category-id="all" data-number-of-elements="4"
                    data-number-of-elements-small-devices="1" data-number-of-records="16" data-carousel-interval="5000" data-bs-original-title="" title="">
                    <div class="container o_not_editable">
                        <div class="css_non_editable_mode_hidden">
                            <div class="missing_option_warning alert alert-info rounded-0 fade show d-none d-print-none o_default_snippet_text">
                                Your Dynamic Snippet will be displayed here...
                                This message is displayed because youy did not provide both a filter and a template to use.
                            </div>
                        </div>
                        <div class="dynamic_snippet_template"></div>
                    </div>
                </section>
            </div>
        </div>
    </template>

    <template id="product_custom_text" inherit_id="website_sale.product" customize_show="True" active="True" name="Terms and Conditions" priority="21">
        <xpath expr="//div[@id='o_product_terms_and_share']" position="inside">
            <p class="text-muted mb-0">
                <a href="/terms" class="text-muted"><u>Terms and Conditions</u></a><br/>
                30-day money-back guarantee<br/>
                Shipping: 2-3 Business Days
            </p>
        </xpath>
    </template>

    <template id="product_share_buttons" inherit_id="website_sale.product" active="True" name="Share Buttons" priority="22">
        <xpath expr="//div[@id='o_product_terms_and_share']" position="inside">
            <div class="h4 mt-3 mb-0 d-flex justify-content-md-end flex-shrink-0" contenteditable="false">
                <t t-snippet-call="website.s_share">
                    <t t-set="_exclude_share_links" t-value="['whatsapp', 'linkedin']"/>
                    <t t-set="_no_title" t-value="True"/>
                    <t t-set="_classes" t-valuef="text-lg-end"/>
                    <t t-set="_link_classes" t-valuef="mx-1 my-0"/>
                </t>
            </div>
        </xpath>
    </template>

    <!-- Product options: Zoom -->
    <template inherit_id='website_sale.product' id="product_picture_magnify_hover" name="Automatic Image Zoom">
        <xpath expr='//div[hasclass("o_wsale_product_page")]' position='attributes'>
            <attribute name="data-ecom-zoom-auto">1</attribute>
            <attribute name="class" separator=" " add="ecom-zoomable zoomodoo-next" />
        </xpath>
    </template>

    <template inherit_id='website_sale.product' id="product_picture_magnify_click" active="False" name="Image Zoom On Click">
        <xpath expr='//div[hasclass("o_wsale_product_page")]' position='attributes'>
            <attribute name="data-ecom-zoom-click">1</attribute>
            <attribute name="class" separator=" " add="ecom-zoomable zoomodoo-next" />
        </xpath>
    </template>

    <template inherit_id='website_sale.product' id="product_picture_magnify_both" active="False" name="Automatic Image Zoom And On Click">
        <xpath expr='//div[hasclass("o_wsale_product_page")]' position='attributes'>
            <attribute name="data-ecom-zoom-auto">1</attribute>
            <attribute name="data-ecom-zoom-click">1</attribute>
            <attribute name="class" separator=" " add="ecom-zoomable zoomodoo-next" />
        </xpath>
    </template>

    <!-- Product options: OpenChatter -->
    <template id="product_comment" inherit_id="website_sale.product" active="False" name="Discussion and Rating" priority="15">
        <xpath expr="//div[@t-field='product.website_description']" position="after">
            <div class="o_shop_discussion_rating" data-anchor='true'>
                <section id="o_product_page_reviews" class="container pt32 pb32" data-anchor='true'>
                    <a class="o_product_page_reviews_title d-flex justify-content-between text-decoration-none collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#o_product_page_reviews_content" aria-expanded="false" aria-controls="o_product_page_reviews_content">
                        <h3 class="mb32">Customer Reviews</h3>
                        <i class="fa align-self-start"/>
                    </a>
                    <div id="o_product_page_reviews_content" class="collapse">
                        <t t-call="portal.message_thread">
                            <t t-set="object" t-value="product"/>
                            <t t-set="display_rating" t-value="True"/>
                            <t t-set="message_per_page" t-value="5"/>
                            <t t-set="two_columns" t-value="true"/>
                        </t>
                    </div>
                </section>
            </div>
        </xpath>
    </template>

    <template id="product_quantity" inherit_id="website_sale.product" name="Select Quantity">
        <xpath expr="//t[@t-set='hasQuantities']" position="attributes">
            <attribute name="t-value" remove="false" add="true" separator=" "/>
        </xpath>
      <xpath expr="//div[@id='add_to_cart_wrap']" position="before">
        <div t-attf-class="css_quantity input-group {{'d-none' if combination_info['prevent_zero_price_sale'] else 'd-inline-flex'}} me-2 mb-2 align-middle" contenteditable="false">
            <a t-attf-href="#" class="btn btn-link js_add_cart_json" aria-label="Remove one" title="Remove one">
                <i class="fa fa-minus"></i>
            </a>
            <input type="text" class="form-control quantity text-center" data-min="1" name="add_qty" t-att-value="add_qty or 1"/>
            <a t-attf-href="#" class="btn btn-link float_left js_add_cart_json" aria-label="Add one" title="Add one">
                <i class="fa fa-plus"></i>
            </a>
        </div>
      </xpath>
    </template>

    <template id="product_buy_now" inherit_id="website_sale.product" active="False" name="Buy Now Button">
        <xpath expr="//t[@t-set='hasBuyNow']" position="attributes">
            <attribute name="t-value" remove="false" add="true" separator=" "/>
        </xpath>
        <xpath expr="//a[@id='add_to_cart']" position="after">
            <a role="button" class="btn btn-outline-primary o_we_buy_now ms-1" href="#">
                <i class="fa fa-bolt me-2"/>
                Buy now
            </a>
        </xpath>
    </template>

    <template id="website_sale.tax_indication" active="False">
        <span t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'" class="h6 text-muted">
            VAT Excluded
        </span>
        <span t-else="" class="h6 text-muted">
            VAT Included
        </span>
    </template>

    <template id="product_price">
        <div
            itemprop="offers"
            itemscope="itemscope"
            itemtype="http://schema.org/Offer"
            t-attf-class="product_price mt-2 mb-3 {{'d-none' if combination_info['prevent_zero_price_sale'] else 'd-inline-block'}}"
        >
            <h3 class="css_editable_mode_hidden">
                <span class="oe_price"
                      style="white-space: nowrap;"
                      t-out="combination_info['price']"
                      t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                <span itemprop="price" style="display:none;" t-out="combination_info['price']"/>
                <span itemprop="priceCurrency" style="display:none;" t-esc="website.currency_id.name"/>
                <span t-attf-class="text-danger oe_default_price ms-1 h5 {{'' if combination_info['has_discounted_price'] and not combination_info['compare_list_price'] else 'd-none'}}"
                      style="text-decoration: line-through; white-space: nowrap;"
                      t-esc="combination_info['list_price']"
                      t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"
                    itemprop="listPrice"
                />
                <t t-if="is_view_active('website_sale.tax_indication')" t-call="website_sale.tax_indication"/>
                <del t-if="combination_info['compare_list_price'] and (combination_info['compare_list_price'] &gt; combination_info['price'])">
                    <bdi dir="inherit">
                    <span t-esc="combination_info['compare_list_price']"
                          groups="website_sale.group_product_price_comparison"
                          t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                    </bdi>
                </del>
            </h3>
            <h3
                t-if="editable"
                class="css_non_editable_mode_hidden decimal_precision"
                t-att-data-precision="str(website.currency_id.decimal_places)"
            >
                <span t-field="product.list_price"
                      t-options="{'widget': 'monetary', 'display_currency': product.currency_id}"/>
                <t t-if="is_view_active('website_sale.tax_indication')" t-call="website_sale.tax_indication"/>
                <del t-if="combination_info['compare_list_price'] and (combination_info['compare_list_price'] &gt; combination_info['price'])">
                    <bdi dir="inherit">
                    <span t-field="product.compare_list_price"
                          groups="website_sale.group_product_price_comparison"
                          t-options="{'widget': 'monetary', 'display_currency': product.currency_id}"/>
                    </bdi>
                </del>
            </h3>
        </div>
        <div id="product_unavailable" t-attf-class="{{'d-flex' if combination_info['prevent_zero_price_sale'] else 'd-none'}}">
            <h3 class="fst-italic" t-field="website.prevent_zero_price_sale_text"/>
        </div>
    </template>

    <template id="product_variants" inherit_id="website_sale.product" active="False" name="List View of Variants">
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
                <ul class="d-none js_add_cart_variants mb-0" t-att-data-attribute_exclusions="json.dumps(attribute_exclusions)"/>
                <input type="hidden" class="product_template_id" t-att-value="product.id"/>
                <input type="hidden" t-if="len(filtered_sorted_variants) == 1" class="product_id" name="product_id" t-att-value="filtered_sorted_variants[0].id"/>
                <t t-if="len(filtered_sorted_variants) &gt; 1">
                    <div class="mb-4">
                        <div t-foreach="filtered_sorted_variants" t-as="variant_id" class="form-check mb-1">
                            <t t-set="template_combination_info" t-value="product._get_combination_info(only_template=True, add_qty=add_qty)"/>
                            <t t-set="combination_info" t-value="variant_id._get_combination_info_variant(add_qty=add_qty)"/>
                            <input type="radio"
                                   name="product_id"
                                   class="form-check-input product_id js_product_change"
                                   t-att-checked="'checked' if variant_id_index == 0 else None"
                                   t-attf-id="radio_variant_#{variant_id.id}"
                                   t-att-value="variant_id.id"
                                   t-att-data-price="combination_info['price']"
                                   t-att-data-combination="variant_id.product_template_attribute_value_ids.ids"/>
                            <label t-attf-for="radio_variant_#{variant_id.id}" label-default="label-default" class="form-check-label fw-normal">
                                <span t-out="combination_info['display_name']"/>
                                <t t-set="diff_price" t-value="website.currency_id.compare_amounts(combination_info['price'], template_combination_info['price'])"/>
                                <span t-attf-class="badge rounded-pill text-bg-{{navClass}} border" t-if="diff_price != 0">
                                    <span class="sign_badge_price_extra" t-out="diff_price > 0 and '+' or '-'"/>
                                    <span t-out="abs(combination_info['price'] - template_combination_info['price'])"
                                          t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"
                                          class="text-muted fst-italic"/>
                                </span>
                            </label>
                        </div>
                    </div>
                </t>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </template>

    <template id="wizard_checkout" name="Wizard Checkout">
        <t t-call="website.step_wizard">
            <t t-set="wizard_step" t-value="website._get_checkout_steps()"/>
        </t>
    </template>

    <!-- /shop/extra_info route -->
    <template id="extra_info" name="Checkout Extra Info" active="False">
        <t t-call="website_sale.checkout_layout">
            <t t-set="show_navigation_button" t-value="False"/>
            <t t-set="redirect" t-valuef="/shop/extra_info"/>
            <t t-set="oe_structure">
                <!-- This is the drag-and-drop area for website building blocs at the end of each
                     checkout page. This is append at the of the page in `checkout_layout`. The
                     templates created in the database to store blocs are hooked using XPath on the
                     `oe_struture` element ID. Therefore, we can't use dynamic IDs (like with
                     t-att-id) and each template needs to define a div element. -->
                <div class="oe_structure" id="oe_structure_website_sale_extra_info_1"/>
            </t>

            <h3 class="mb-4">Extra info</h3>
            <section class="s_website_form" data-vcss="001" data-snippet="s_website_form">
                <div class="container">
                    <form action="/website/form/" method="post" enctype="multipart/form-data" class="o_mark_required s_website_form_no_recaptcha" data-mark="*" data-force_action="shop.sale.order" data-model_name="sale.order" data-success-mode="redirect" data-success-page="/shop/payment" hide-change-model="true">
                        <div class="s_website_form_rows s_col_no_bgcolor row">
                            <div class="s_website_form_field col-12 py-2 mb-0" data-type="char" data-name="Field">
                                <div class="s_col_no_resize s_col_no_bgcolor row">
                                    <label class="s_website_form_label col-form-label col-sm-auto" style="width: 200px" for="sale1">
                                        <span class="s_website_form_label_content">Your Reference</span>
                                    </label>
                                    <div class="col-sm">
                                        <input id="sale1" type="text" class="s_website_form_input form-control" name="client_order_ref"/>
                                    </div>
                                </div>
                            </div>
                            <div class="s_website_form_field s_website_form_custom col-12 py-2 mb-0" data-type="text" data-name="Field">
                                <div class="s_col_no_resize s_col_no_bgcolor row">
                                    <label class="s_website_form_label col-form-label col-sm-auto" style="width: 200px" for="sale2">
                                        <span class="s_website_form_label_content">Give us your feedback</span>
                                    </label>
                                    <div class="col-sm">
                                        <textarea id="sale2" class="s_website_form_input form-control" name="Give us your feedback" />
                                    </div>
                                </div>
                            </div>
                            <div class="s_website_form_field s_website_form_custom col-12 py-2 mb-0" data-type="binary" data-name="Field">
                                <div class="s_col_no_resize s_col_no_bgcolor row">
                                    <label class="s_website_form_label col-form-label col-sm-auto" style="width: 200px" for="sale3">
                                        <span class="s_website_form_label_content">Upload a document</span>
                                    </label>
                                    <div class="col-sm">
                                        <input id="sale3" type="file" class="s_website_form_input form-control" name="a_document" />
                                    </div>
                                </div>
                            </div>
                            <div class="s_website_form_submit s_website_form_no_submit_option d-flex flex-column flex-lg-row align-items-lg-center pt-4">
                                <a role="button"
                                   name="website_sale_main_button"
                                   class="s_website_form_send btn btn-primary order-lg-3 w-100 w-lg-auto ms-lg-auto"
                                   href="/shop/confirm_order">
                                    Continue checkout
                                    <i class="fa fa-angle-right ms-2 fw-light"/>
                                </a>

                                <div class="position-relative d-flex d-lg-none w-100 justify-content-center align-items-center my-2 opacity-75">
                                    <hr class="w-100"/>
                                    <span class="px-3">or</span>
                                    <hr class="w-100"/>
                                </div>

                                <a href="/shop/checkout" class="text-center">
                                    <i class="fa fa-angle-left me-2 fw-light"/>
                                    Return to shipping
                                </a>
                                <span id="s_website_form_result"/>
                            </div>
                        </div>
                    </form>
                </div>
            </section>
        </t>
    </template>

    <!-- Encapsulate the content in a `a` tag with a link to the product page. Override this
         template to change or remove the product link. Called in `website_sale.cart_lines`. -->
    <template id="cart_line_product_link" name="Shopping Cart Line Product Link">
        <a t-att-href="line.product_id.website_url">
            <t t-out="0"/>
        </a>
    </template>

    <!-- This template displays all the lines following the first one on the description of the sale
         order line, with a muted style. For typical products this content will be the product
         description_sale. Called in `website_sale.cart_lines`. -->
    <template id="cart_line_description_following_lines" name="Shopping Cart Line Description Following Lines">
        <t t-set="description_lines" t-value="line.get_description_following_lines()"/>
        <div t-if="description_lines" t-attf-class="text-muted {{div_class}} small">
            <t t-foreach="description_lines" t-as="name_line">
                <span t-if="name_line" class="d-block" t-out="name_line"/>
            </t>
        </div>
    </template>

    <!-- Lines that show items in the cart. Called in `website_sale.cart`. -->
    <template id="cart_lines" name="Shopping Cart Lines">
        <div t-if="not website_sale_order or not website_sale_order.website_order_line" class="js_cart_lines alert alert-info">
            Your cart is empty!
        </div>
        <t t-if='website_sale_order'>
            <div t-if='website_sale_order._get_shop_warning(clear=False)' class="alert alert-warning js_cart_lines" role="alert">
                <strong>Warning!</strong> <t t-esc='website_sale_order._get_shop_warning()'/>
            </div>
        </t>
        <div id="cart_products"
             t-if="website_sale_order and website_sale_order.website_order_line"
             class="js_cart_lines d-flex flex-column mb32">
            <t t-set="show_qty" t-value="is_view_active('website_sale.product_quantity')"/>
            <div t-foreach="website_sale_order.website_order_line"
                 t-as="line"
                 t-attf-class="o_cart_product d-flex align-items-stretch gap-3 #{line.linked_line_id and 'optional_product info'} #{not line_last and 'border-bottom pb-4'} #{line_index &gt; 0 and 'pt-4'}"
                 t-attf-data-product-id="#{line.product_id and line.product_id.id}">
                <t t-if="line.product_id">
                    <div style="width: 64px">
                        <!--
                            Unsellable lines can have unpublished products, but portal users have no
                            access to unpublished product images. To ensure product images are
                            always shown for unsellable lines, we use the raw image data as src
                            (which doesn't require access, unlike the image URL).
                        -->
                        <img
                            t-if="line._is_not_sellable_line() and line.product_id.image_128"
                            t-att-src="image_data_uri(line.product_id.image_128)"
                            class="o_image_64_max img rounded"
                            t-att-alt="line.name_short"
                        />
                        <div
                            t-else=""
                            t-field="line.product_id.image_128"
                            t-options="{
                                'widget': 'image',
                                'qweb_img_responsive': False,
                                'class': 'o_image_64_max rounded',
                            }"
                        />
                    </div>
                    <div class="flex-grow-1">
                        <t t-call="website_sale.cart_line_product_link">
                            <h6 t-field="line.name_short" class="d-inline align-top h6 fw-bold"/>
                        </t>
                        <t t-call="website_sale.cart_line_description_following_lines">
                            <t t-set="div_class" t-value="''"/>
                        </t>
                        <div>
                            <a href='#'
                               class="js_delete_product d-none d-md-inline-block small"
                               aria-label="Remove from cart"
                               title="Remove from cart">Remove</a>
                            <button class="js_delete_product btn btn-light d-inline-block d-md-none"
                                    title="remove">
                                <i class="fa fa-trash-o"/>
                            </button>
                        </div>
                    </div>
                    <div class="d-flex flex-column align-items-end">
                        <div t-attf-class="css_quantity input-group mb-2"
                             name="website_sale_cart_line_quantity">
                            <t t-if="not line._is_not_sellable_line()">
                                <t t-if="show_qty">
                                    <a href="#"
                                       class="js_add_cart_json btn btn-link d-inline-block border-end-0"
                                       aria-label="Remove one"
                                       title="Remove one">
                                        <i class="position-relative z-index-1 fa fa-minus"/>
                                    </a>
                                    <input type="text"
                                           class="js_quantity quantity form-control border-start-0 border-end-0"
                                           t-att-data-line-id="line.id"
                                           t-att-data-product-id="line.product_id.id"
                                           t-att-value="line._get_displayed_quantity()"/>
                                    <t t-if="line._get_shop_warning(clear=False)">
                                        <a href="#" class="btn btn-link">
                                        <i class='fa fa-warning text-warning'
                                           t-att-title="line._get_shop_warning()"
                                           role="img"
                                           aria-label="Warning"/>
                                        </a>
                                    </t>
                                    <a t-else=""
                                       href="#"
                                       class="js_add_cart_json d-inline-block float_left btn btn-link border-start-0"
                                       aria-label="Add one"
                                       title="Add one">
                                        <i class="fa fa-plus position-relative z-index-1"/>
                                    </a>
                                </t>
                                <t t-else="">
                                    <input type="hidden"
                                           class="js_quantity form-control quantity"
                                           t-att-data-line-id="line.id"
                                           t-att-data-product-id="line.product_id.id"
                                           t-att-value="line._get_displayed_quantity()"/>
                                </t>
                            </t>
                            <t t-else="">
                                <span class="w-100 text-muted" t-esc="int(line.product_uom_qty)"/>
                                <input type="hidden"
                                       class="js_quantity quantity form-control"
                                       t-att-data-line-id="line.id"
                                       t-att-data-product-id="line.product_id.id"
                                       t-att-value="line._get_displayed_quantity()"/>
                            </t>
                        </div>
                        <div class="mb-0 h6 fw-bold text-end" name="website_sale_cart_line_price">
                            <t t-if="line.discount">
                                <del t-attf-class="#{'text-danger mr8'}"
                                     style="white-space: nowrap;"
                                     t-out="line._get_displayed_unit_price() * line.product_uom_qty"
                                     t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                            </t>
                            <t t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                               t-set='product_price'
                               t-value='line.price_subtotal'/>
                            <t t-else=""
                               t-set='product_price'
                               t-value='line.price_total'/>
                            <span t-out="product_price" style="white-space: nowrap;"
                                  t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}"/>
                            <small t-if="not line._is_not_sellable_line() and line.product_id.base_unit_price"
                                   class="cart_product_base_unit_price d-block text-muted"
                                   groups="website_sale.group_show_uom_price">
                                <t t-call='website_sale.base_unit_price'>
                                    <t t-set='product' t-value='line.product_id'/>
                                    <t t-set='combination_info'
                                       t-value="{'base_unit_price': product._get_base_unit_price(product_price/line.product_uom_qty)}"/>
                                </t>
                            </small>
                        </div>
                    </div>
                </t>
            </div>
        </div>
    </template>

    <!-- /shop/cart route -->
    <template id="cart" name="Shopping Cart">
        <t t-call="website_sale.checkout_layout">
            <t t-set="show_shorter_cart_summary" t-value="True"/>
            <t t-set="show_footer" t-value="True"/>
            <t t-set="oe_structure">
                <!-- This is the drag-and-drop area for website building blocs at the end of each
                     checkout page. This is append at the of the page in `checkout_layout`. The
                     templates created in the database to store blocs are hooked using XPath on the
                     `oe_struture` element ID. Therefore, we can't use dynamic IDs (like with
                     t-att-id) and each template needs to define a div element. -->
                <div class="oe_structure" id="oe_structure_website_sale_cart_2"/>
            </t>

            <div class="col">
                <h3 class="mb-4">Order overview</h3>
                <div t-if="abandoned_proceed or access_token" class="alert alert-info mt8 mb8" role="alert"> <!-- abandoned cart choices -->
                    <t t-if="abandoned_proceed">
                        <p>Your previous cart has already been completed.</p>
                        <p t-if="website_sale_order">Please proceed your current cart.</p>
                    </t>
                    <t t-if="access_token">
                        <p>This is your current cart.</p>
                        <p>
                            <strong>
                                <a t-attf-href="/shop/cart/?access_token=#{access_token}&amp;revive=squash">Click here</a>
                            </strong> if you want to restore your previous cart. Your current cart will be replaced with your previous cart.
                        </p>
                        <p>
                            <strong>
                                <a t-attf-href="/shop/cart/?access_token=#{access_token}&amp;revive=merge">Click here</a>
                            </strong> if you want to merge your previous cart into current cart.
                        </p>
                    </t>
                </div>
                <t t-call="website_sale.cart_lines"/>
                <div class="clearfix" />
                <div class="oe_structure" id="oe_structure_website_sale_cart_1"/>
            </div>
        </t>
    </template>

    <!-- Deactivatable through the website editor. -->
    <template id="suggested_products_list" inherit_id="website_sale.cart_lines" name="Accessory Products in my cart">
        <xpath expr="//div[@id='cart_products']" position="inside">
            <h5 t-attf-class="mt32 mb-3" t-if="suggested_products">Suggested accessories</h5>
            <div t-if="suggested_products"
                 id="suggested_products"
                 class="d-flex flex-column align-items-stretch mb32">
                <div t-foreach="suggested_products"
                     t-as="product"
                     t-attf-class="d-flex gap-3 #{not product_last and 'border-bottom pb-4'} #{product_index &gt; 0 and 'pt-4'}"
                     t-att-data-publish="product.website_published and 'on' or 'off'">
                    <div style="width: 64px">
                        <a t-att-href="product.website_url">
                            <span t-field="product.image_128" t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'o_image_64_max rounded'}"/>
                        </a>
                    </div>
                    <div class="o_cart_suggested_product_name flex-grow-1">
                        <div>
                            <a t-att-href="product.website_url">
                                <strong t-out="product.with_context(display_default_code=False).display_name"/>
                            </a>
                        </div>
                        <div class="d-none d-md-block text-muted" t-field="product.description_sale"/>
                    </div>
                    <div class="d-flex flex-column align-items-end">
                        <input class="js_quantity" name="product_id" t-att-data-product-id="product.id" type="hidden"/>
                        <a t-if="product._website_show_quick_add()"
                           role="button"
                           class="js_add_suggested_products btn btn-md btn-outline-primary text-nowrap">
                            <span class="d-md-none fa fa-shopping-cart"/>
                            <span class="d-none d-md-inline">Add to cart</span>
                        </a>
                        <div class="mb-0 h-6 fw-bold text-end d-flex"
                             name="website_sale_suggested_product_price">
                            <t t-set="combination_info"
                               t-value="product._get_combination_info_variant()"/>
                            <del t-attf-class="text-danger mr8 {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                                 t-esc="combination_info['list_price']"
                                 t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"
                                 style="white-space: nowrap;"/>
                            <span t-esc="combination_info['price']"
                                  t-options="{'widget': 'monetary','display_currency': website.currency_id}"
                                  style="white-space: nowrap;"/>
                        </div>
                    </div>
                </div>
            </div>
        </xpath>
    </template>

    <!-- Called in `website_sale.reduction_code`. -->
    <template id='coupon_form' name='Coupon form'>
        <!-- Checkout context:
            - redirect: The route to redirect to when a customer enters a coupon; default: `None`.
            - website_sale_order: The current order.
        -->
        <form t-attf-action="/shop/pricelist#{redirect and '?r=' + redirect or ''}"
            method="post" name="coupon_code">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" t-nocache="The csrf token must always be up to date."/>
            <div class="input-group w-100 my-2">
                <input name="promo" class="form-control" type="text" placeholder="Discount code..." t-att-value="website_sale_order.pricelist_id.code or None"/>
                <a href="#" role="button" class="btn btn-secondary a-submit ps-2">Apply</a>
            </div>
        </form>
        <t t-if="request.params.get('code_not_available')" name="code_not_available">
            <div class="alert alert-danger text-start" role="alert">This promo code is not available.</div>
        </t>
    </template>

    <!-- Called in `website_sale.checkout_layout`. -->
    <template id="navigation_buttons" name="Navigation buttons">
        <!-- Layout customization parameters:
            - _cta_classes: CSS classes to append on the primary navigation button; default `None`.
            - _form_send_navigation: Whether the primary button serves as a submit button for a
                                     form; default `None`.
            - hide_payment_button: Whether the payment button should be hidden; default: False.
        -->
        <!-- Checkout context:
            - website_sale_order: The current order.
            - xmlid: The id of the xml templated rendered by the controller.
        -->
        <t t-set="step_specific_values" t-value="website._get_checkout_steps(xmlid)"/>
        <div t-attf-class="#{_container_classes} d-flex #{_form_send_navigation and 'flex-column flex-lg-row align-items-lg-center' or 'flex-column'} mb-5 mb-lg-0 pt-4">
            <t t-if="website_sale_order and website_sale_order.website_order_line">
                <t t-if="xmlid == 'website_sale.payment'">
                    <div t-if="not errors and not website_sale_order.amount_total"
                         name="o_website_sale_free_cart">
                        <form name="o_wsale_confirm_order"
                              class="d-flex flex-column"
                              target="_self"
                              action="/shop/payment/validate"
                              method="post">
                            <input type="hidden"
                                   name="csrf_token"
                                   t-att-value="request.csrf_token()"
                                   t-nocache="The csrf token must always be up to date."/>
                            <t t-if="not hide_payment_button" t-call="payment.submit_button">
                                <t t-set="submit_button_label">Confirm Order</t>
                            </t>
                        </form>
                    </div>
                    <t t-elif="not hide_payment_button" t-call="payment.submit_button"/>
                </t>
                <t t-else="">
                    <a role="button" name="website_sale_main_button"
                        t-attf-class="#{_cta_classes} btn btn-primary #{not website_sale_order._is_cart_ready() and 'disabled'} #{_form_send_navigation and 'order-lg-3 w-100 w-lg-auto ms-lg-auto' or 'w-100'}"
                        t-att-href="step_specific_values['main_button_href']">
                        <t t-out="step_specific_values['main_button']"/>
                        <i class="fa fa-angle-right ms-2 fw-light"/>
                    </a>
                </t>
            </t>
            <div t-if="not hide_payment_button" t-attf-class="position-relative #{_form_send_navigation and 'd-flex d-lg-none' or 'd-flex'} w-100 justify-content-center align-items-center my-2 opacity-75">
                <hr class="w-100"/>
                <span class="px-3">or</span>
                <hr class="w-100"/>
            </div>
            <a t-att-href="step_specific_values['back_button_href']" class="text-center">
                <i class="fa fa-angle-left me-2 fw-light"/>
                <t t-out="step_specific_values['back_button']"/>
            </a>
        </div>
    </template>

    <!-- /shop/checkout route -->
    <template id="checkout">
        <t t-call="website_sale.checkout_layout">
            <t t-set="additional_title">Shop - Checkout</t>
            <t t-set="redirect" t-valuef="/shop/checkout"/>
            <t t-set="same_shipping" t-value="bool(order.partner_shipping_id==order.partner_invoice_id or only_services)" />
            <div class="row">
                <div class="col-lg-12">
                    <h3 class="mb-4">Address</h3>
                    <h4>Billing</h4>
                </div>
            </div>
            <t t-call="website_sale.row_addresses">
                <t t-set="order" t-value="order"/>
                <t t-set="is_invoice" t-value="True"/>
                <t t-set="addresses" t-value="billings"/>
                <t t-set="selected_address" t-value="order.partner_invoice_id"/>
            </t>
            <t t-if="not only_services" groups="account.group_delivery_invoice_address">
                <div class="row">
                    <div class="col-lg-12">
                        <h4>Shipping</h4>
                    </div>
                </div>
                <t t-call="website_sale.row_addresses">
                    <t t-set="order" t-value="order"/>
                    <t t-set="is_invoice" t-value="False"/>
                    <t t-set="addresses" t-value="shippings"/>
                    <t t-set="selected_address" t-value="order.partner_shipping_id"/>
                </t>
            </t>
        </t>
    </template>

    <template id="row_addresses">
        <div t-attf-class="{{'all_billing' if is_invoice else 'all_shipping'}} row row-cols-md-2 row-cols-lg-3 g-3 flex-nowrap flex-md-wrap mb32">
            <div t-foreach="addresses" t-as="addr"
                    class="one_kanban col-md">
                <t t-call="website_sale.address_kanban">
                    <t t-set="contact" t-value="addr"/>
                    <t t-set="selected" t-value="len(addresses) == 1 or addr == selected_address"/>
                </t>
            </div>
            <div t-if="not is_invoice or not order.website_id.is_public_user()" class="one_kanban col-md">
                <!-- We do not allow public users to have multiple billing addresses -->
                <t t-if="is_invoice">
                    <t t-set="new_address_href" t-valuef="/shop/address?mode=billing"/>
                </t>
                <t t-else="">
                    <t t-set="new_address_href" t-valuef="/shop/address?mode=shipping"/>
                </t>
                <a role="button" t-att-href="new_address_href" class="o_wsale_add_address d-flex align-items-center justify-content-center h-100 px-4 border rounded mx-auto no-decoration">
                    <i class="fa fa-plus me-md-2"/><span class="d-none d-md-inline">Add address</span>
                </a>
            </div>
        </div>
    </template>

    <!-- Card view of addresses. Called in `website_sale.checkout`. -->
    <template id="address_kanban" name="Kanban address">
        <t t-set="mode" t-value="is_invoice and 'billing' or 'shipping'"/>
        <form action="/shop/cart/update_address" method="POST" class="d-none">
            <input type="hidden"
                   name="csrf_token"
                   t-att-value="request.csrf_token()"
                   t-nocache="The csrf token must always be up to date."/>
            <input type="hidden" name="partner_id" t-att-value="contact.id"/>
            <input type="hidden" name="mode" t-att-value="mode"/>
            <input type="submit"/>
        </form>
        <div t-attf-class="card position-relative h-100 #{selected and 'bg-primary border border-primary' or is_invoice and 'js_change_billing' or 'js_change_shipping'}">
            <div class="card-body d-flex flex-column align-items-start">
                <t t-esc="contact" t-options="dict(widget='contact', fields=['name', 'address'], no_marker=True)"/>
                <t t-if="contact._can_be_edited_by_current_customer(website_sale_order, mode)">
                    <t t-set="new_address_href" t-value="'/shop/address?mode=' + mode"/>
                    <a
                        t-att-href="new_address_href + '&amp;partner_id=' + str(contact.id)"
                        class="js_edit_address btn btn-link p-0 mt-auto"
                        role="button"
                        title="Edit this address"
                        aria-label="Edit this address">
                        <i class="fa fa-pencil me-1"/>Edit
                    </a>
                </t>
            </div>
        </div>
    </template>

    <!-- /shop/address route -->
    <template id="address" name="Address Management">
        <t t-set="no_footer" t-value="1"/>
        <t t-call="website.layout">
            <div id="wrap">
                <div class="oe_website_sale o_wsale_address_fill container py-2">
                    <div class="row">
                        <div class="col-12">
                            <t t-call="website_sale.wizard_checkout"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="oe_cart col-12 col-lg-8">
                            <div>
                                <t t-set="address_mode" t-value="mode[1]"/>
                                <t t-if="is_public_order">
                                    <h3 class="mb-3">
                                        <span t-if="account_on_checkout != 'mandatory'">Fill in your address</span>
                                        <small class="text-muted" t-if="account_on_checkout == 'optional'"> or </small>
                                        <a t-if="account_on_checkout != 'disabled'" role="button" href='/web/login?redirect=/shop/checkout'  style="margin-top: -11px"> Sign in</a>
                                    </h3>
                                </t>
                                <t t-elif="address_mode == 'billing'">
                                    <h3 class="mb-3">Billing address</h3>
                                </t>
                                <t t-else="">
                                    <h3 class="mb-3">Shipping address</h3>
                                </t>
                                <t t-if="partner_id == website_sale_order.partner_shipping_id.id == website_sale_order.partner_invoice_id.id">
                                    <div class="alert alert-warning" role="alert" t-if="not only_services" groups="account.group_delivery_invoice_address">
                                        <h4 class="alert-heading">Be aware!</h4>
                                        <p>
                                            You are editing your <b>billing and shipping</b> addresses at the same time!<br/>
                                            If you want to modify your shipping address, create a <a href='/shop/address'>new address</a>.
                                        </p>
                                    </div>
                                </t>
                                <t t-if="error" t-foreach="error.get('error_message', [])" t-as="err">
                                    <h5 class="text-danger" t-esc="err" />
                                </t>
                                <form t-if="account_on_checkout != 'mandatory' or not is_public_user" action="/shop/address" method="post" class="checkout_autoformat">
                                    <div class="row">
                                        <div t-attf-class="#{error.get('name') and 'o_has_error'} div_name col-lg-12 mb-2">
                                            <label class="col-form-label" for="name">Full name</label>
                                            <input type="text" name="name" t-attf-class="form-control #{error.get('name') and 'is-invalid' or ''}" t-att-value="'name' in checkout and checkout['name']" />
                                        </div>
                                        <div class="w-100"/>
                                        <div t-attf-class="#{error.get('email') and 'o_has_error'} col-lg-6 mb-2" id="div_email">
                                            <label t-attf-class="col-form-label #{mode[1] == 'shipping' and 'label-optional' or ''}" for="email">Email</label>
                                            <input type="email" name="email" t-attf-class="form-control #{error.get('email') and 'is-invalid' or ''}" t-att-value="'email' in checkout and checkout['email']" />
                                        </div>
                                        <div t-attf-class="#{error.get('phone') and 'o_has_error'} col-lg-6 mb-2" id="div_phone">
                                            <label class="col-form-label" for="phone">Phone</label>
                                            <input type="tel" name="phone" t-attf-class="form-control #{error.get('phone') and 'is-invalid' or ''}" t-att-value="'phone' in checkout and checkout['phone']" />
                                        </div>
                                        <t t-if="website._display_partner_b2b_fields()">
                                            <div class="w-100"/>
                                            <t t-set='vat_warning' t-value="'vat' in checkout and checkout['vat'] and not can_edit_vat" />
                                            <t t-if="(mode == ('new', 'billing') and is_public_order
                                                or mode == ('edit', 'billing') and partner_id == website_sale_order.partner_id.id)
                                                and (can_edit_vat or 'vat' in checkout and checkout['vat'])"
                                            >
                                                <div t-attf-class="#{error.get('company_name') and 'o_has_error'} col-lg-6 mb-2">
                                                    <label class="col-form-label fw-normal label-optional" for="company_name">Company Name</label>
                                                    <input type="text" name="company_name" t-attf-class="form-control #{error.get('company_name') and 'is-invalid' or ''}" t-att-value="'commercial_company_name' in checkout and checkout['commercial_company_name'] or 'company_name' in checkout and checkout['company_name']" t-att-readonly="'1' if vat_warning else None" />
                                                    <small t-if="vat_warning" class="form-text text-muted d-block d-lg-none">Changing company name is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</small>
                                                </div>
                                                <div t-attf-class="#{error.get('vat') and 'o_has_error'} div_vat col-lg-6 mb-2">
                                                    <label class="col-form-label fw-normal label-optional" for="vat">VAT</label>
                                                    <input type="text" name="vat" t-attf-class="form-control #{error.get('vat') and 'is-invalid' or ''}" t-att-value="'vat' in checkout and checkout['vat']" t-att-readonly="'1' if vat_warning else None"/>
                                                    <small t-if="vat_warning" class="form-text text-muted d-block d-lg-none">Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</small>
                                                </div>
                                                <div t-if="vat_warning" class="col-12 d-none d-lg-block mb-1">
                                                    <small class="form-text text-muted">Changing company name or VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</small>
                                                </div>
                                            </t>
                                        </t>
                                        <div t-attf-class="#{error.get('street') and 'o_has_error'} div_street col-lg-12 mb-2">
                                            <label class="col-form-label" for="street">Street and Number</label>
                                            <input type="text" name="street" t-attf-class="form-control #{error.get('street') and 'is-invalid' or ''}" t-att-value="'street' in checkout and checkout['street']" />
                                        </div>
                                        <div t-attf-class="mb-2 #{error.get('street2') and 'o_has_error' or ''} col-lg-12 div_street2">
                                            <label class="col-form-label label-optional" for="street2">Apartment, suite, etc.</label>
                                            <input type="text" name="street2" t-attf-class="form-control #{error.get('street2') and 'is-invalid' or ''}" t-att-value="'street2' in checkout and checkout['street2']" />
                                        </div>
                                        <div class="w-100"/>
                                        <t t-set='zip_city' t-value='country and [x for x in country.get_address_fields() if x in ["zip", "city"]] or ["city", "zip"]'/>
                                        <t t-if="'zip' in zip_city and zip_city.index('zip') &lt; zip_city.index('city')">
                                            <div t-attf-class="#{error.get('zip') and 'o_has_error'} div_zip col-md-4 mb-2">
                                                <label class="col-form-label label-optional" for="zip">Zip Code</label>
                                                <input type="text" name="zip" t-attf-class="form-control #{error.get('zip') and 'is-invalid' or ''}" t-att-value="'zip' in checkout and checkout['zip']" />
                                            </div>
                                        </t>
                                        <div t-attf-class="#{error.get('city') and 'o_has_error' or ''} div_city col-md-8 mb-2">
                                            <label class="col-form-label" for="city">City</label>
                                            <input type="text" name="city" t-attf-class="form-control #{error.get('city') and 'is-invalid' or ''}" t-att-value="'city' in checkout and checkout['city']" />
                                        </div>
                                        <t t-if="'zip' in zip_city and zip_city.index('zip') &gt; zip_city.index('city')">
                                            <div t-attf-class="#{error.get('zip') and 'o_has_error'} div_zip col-md-4 mb-2">
                                                <label class="col-form-label label-optional" for="zip">Zip Code</label>
                                                <input type="text" name="zip" t-attf-class="form-control #{error.get('zip') and 'is-invalid' or ''}" t-att-value="'zip' in checkout and checkout['zip']" />
                                            </div>
                                        </t>
                                        <div class="w-100"/>
                                        <div t-attf-class="#{error.get('country_id') and 'o_has_error'} div_country col-lg-6 mb-2">
                                            <label class="col-form-label" for="country_id">Country</label>
                                            <select id="country_id" name="country_id" t-attf-class="form-select #{error.get('country_id') and 'is-invalid' or ''}" t-att-mode="mode[1]">
                                                <option value="">Country...</option>
                                                <t t-foreach="countries" t-as="c">
                                                    <option t-att-value="c.id" t-att-selected="c.id == (country and country.id or -1)">
                                                        <t t-esc="c.name" />
                                                    </option>
                                                </t>
                                            </select>
                                        </div>
                                        <div t-attf-class="#{error.get('state_id') and 'o_has_error'} div_state col-lg-6 mb-2" t-att-style="(not country or not country.state_ids) and 'display: none'">
                                            <label class="col-form-label" for="state_id">State / Province</label>
                                            <select name="state_id" t-attf-class="form-select #{error.get('state_id') and 'is-invalid' or ''}" data-init="1">
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
                                                <div class="form-check form-switch mt-2 mb-3">
                                                    <label>
                                                        <input
                                                            type="checkbox"
                                                            id="shipping_use_same"
                                                            class="form-check-input mr8"
                                                            name="use_same"
                                                            value="1"
                                                            t-att-checked="use_same"/>Ship to the same address
                                                        <span
                                                            t-if="is_public_user"
                                                            class="form-check-label ship_to_other text-muted"
                                                            style="display: none">
                                                            &amp;nbsp;(<i>Your shipping address will be requested later)</i>
                                                        </span>
                                                    </label>
                                                </div>
                                            </div>
                                        </t>
                                        <t t-else="">
                                            <input type="hidden" name="use_same" t-att-value="partner_id == website_sale_order.partner_shipping_id.id == website_sale_order.partner_invoice_id.id"/>
                                        </t>
                                    </div>

                                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" t-nocache="The csrf token must always be up to date."/>
                                    <input type="hidden" name="submitted" value="1" />
                                    <input type="hidden" name="partner_id" t-att-value="partner_id or '0'" />
                                    <input type="hidden" name="mode" t-att-value="mode[1]"/>
                                    <input type="hidden" name="callback" t-att-value="callback" />

                                    <!-- Example -->
                                    <input type="hidden" name="field_required" t-att-value="'name,street'" />

                                    <div class="d-flex flex-column flex-md-row align-items-center justify-content-between mt32 mb32">
                                        <a role="button" t-att-href="mode == ('new', 'billing') and '/shop/cart' or '/shop/checkout'" class="btn btn-outline-secondary w-100 w-md-auto order-md-1 order-3">
                                            <i class="fw-light fa fa-angle-left me-2"/>Discard
                                        </a>
                                        <div class="position-relative w-100 d-flex d-md-none justify-content-center align-items-center order-2 my-2 opacity-75">
                                            <hr class="w-100"/>
                                            <span class="px-3">or</span>
                                            <hr class="w-100"/>
                                        </div>
                                        <a role="button" href="#" class="a-submit a-submit-disable a-submit-loading btn btn-primary w-100 w-md-auto order-1 order-md-3">
                                            <t t-if="mode == ('new', 'billing')">
                                                Continue checkout
                                            </t>
                                            <t t-else="">
                                                Save address
                                            </t>
                                            <i class="fw-light fa fa-angle-right ms-2"/>
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

    <!-- Deactivatable through the website editor. -->
    <template id="address_b2b" inherit_id="address" name="Show b2b fields" />

    <!-- Called in `website_sale.payment` and `website_sale.confirmation`. -->
    <template id="address_on_payment" name="Address on payment">
        <div class="card o_not_editable">
            <div class="card-body" id="shipping_and_billing">
                <a t-if="not disable_edit" class="float-end no-decoration" href="/shop/checkout"><i class="fa fa-pencil me-1"/>Edit</a>
                <t t-set="same_shipping" t-value="bool(order.partner_shipping_id==order.partner_invoice_id or only_services)" />
                <t t-set="order_address" t-value="order.with_context(show_address=True)"/>
                <div>
                    <b>Billing<t t-if="same_shipping and not only_services"> &amp; Shipping</t>: </b>
                    <span
                        t-esc="', '.join(order_address.partner_invoice_id.display_name.split('\n')[1:])"
                        class="address-inline lh-sm o_address_font_sm"/>
                </div>
                <div t-if="not same_shipping and not only_services" groups="account.group_delivery_invoice_address">
                    <b>Shipping: </b>
                    <span
                        id="shipping_on_payment_details"
                        t-esc="', '.join(order_address.partner_shipping_id.display_name.split('\n')[1:])"
                        class="address-inline lh-sm o_address_font_sm"/>
                </div>
            </div>
        </div>
    </template>

    <!-- /shop/payment route -->
    <template id="payment" name="Payment">
        <t t-call="website_sale.checkout_layout">
            <t t-set="additional_title">Shop - Select Payment Method</t>
            <t t-set='redirect' t-valuef="/shop/payment"/>
            <t t-set="oe_structure">
                <!-- This is the drag-and-drop area for website building blocs at the end of each
                     checkout page. This is append at the of the page in `checkout_layout`. The
                     templates created in the database to store blocs are hooked using XPath on the
                     `oe_struture` element ID. Therefore, we can't use dynamic IDs (like with
                     t-att-id) and each template needs to define a div element. -->
                <div class="oe_structure" id="oe_structure_website_sale_payment_2"/>
            </t>
            <div class="col-12" t-if="errors">
                <t t-set="hide_payment_button" t-value="True"/>
                <t t-foreach="errors" t-as="error">
                    <div class="alert alert-danger" t-if="error" role="alert">
                        <h4>
                            <t t-esc="error[0]" />
                        </h4>
                        <t t-esc="error[1]" />
                    </div>
                </t>
            </div>
            <h3 class="mb-4">Confirm order</h3>
            <div id="address_on_payment" class="mb-4">
                <t t-call="website_sale.address_on_payment"/>
            </div>
            <div class="oe_structure clearfix mt-3" id="oe_structure_website_sale_payment_1"/>

            <t t-if="not errors and website_sale_order.amount_total" name="website_sale_non_free_cart">
                <div t-if="payment_methods_sudo or tokens_sudo"
                        id="payment_method"
                        class="o_not_editable mt-4">
                    <t t-call="payment.form"/>
                </div>
                <div t-else="" class="alert alert-warning mt-4">
                    <t t-set="hide_payment_button" t-value="True"/>
                    <strong>No suitable payment option could be found.</strong><br/>
                    <div t-if="not request.env.is_admin()">
                        If you believe that it is an error, please contact the website administrator.
                    </div>
                    <div class="mt-2" groups="base.group_system">
                        <a t-if="request.env.company.country_id.is_stripe_supported_country"
                            t-attf-href="/web#action=#{action_activate_stripe_id}"
                            role="button"
                            class="btn btn-primary"
                            t-out="'ACTIVATE STRIPE'"
                        />
                        <div t-else=""
                                class="d-inline"
                                title="Stripe Connect is not available in your country, please use another payment provider."
                        >
                            <button t-out="'ACTIVATE STRIPE'" class="btn btn-primary" disabled="true"/>
                        </div>
                        <a role="button"
                            class="btn-link alert-warning ps-2"
                            t-attf-href="/web#action=#{payment_action_id}"
                        >
                            <strong><i class="oi oi-arrow-right"></i> View alternatives</strong>
                        </a>
                    </div>
                </div>
            </t>
        </t>
    </template>

    <!-- Activatable through the website editor. -->
    <template id="accept_terms_and_conditions"
              inherit_id="navigation_buttons"
              name="Accept Terms &amp; Conditions"
              active="False">
        <xpath expr="//div[@name='o_website_sale_free_cart']" position="before">
            <div name="website_sale_terms_and_conditions_checkbox" class="form-check mb-2">
                <input type="checkbox" id="website_sale_tc_checkbox" class="form-check-input"/>
                <label for="website_sale_tc_checkbox" class="form-check-label">
                    I agree to the <a target="_BLANK" href="/terms">terms &amp; conditions</a>
                </label>
            </div>
        </xpath>
    </template>

    <!-- Template of the checkout pages. Should be called in every page of the checkout flow. -->
    <template id="checkout_layout" name="Checkout layout page">
        <!-- Layout customization parameters:
            - show_footer: Whether to show the website footer; default: `False`.
            - show_navigation_button: Whether to show the navigation buttons; default: `True`.
            - show_wizard_checkout: Whether to show the wizard checkout; default: `True`.
            - show_shorter_cart_summary: Whether to show the shorter cart_summary (without items
                                         summary and with express checkout buttons) or the full;
                                         default: `None`.
            - oe_structure: The structure element to append at the bottom of the page;
                            default: `None`.
        -->
        <!-- Checkout context (non exhaustive):
            - redirect: The route to redirect to when a customer enters a coupon; default: `None`.
            - website_sale_order: The current order.
        -->
        <t t-call="website.layout">
            <t t-set="no_footer" t-value="True if show_footer is None else not show_footer"/>
            <t t-set="show_navigation_button" t-value="True if show_navigation_button is None else show_navigation_button"/>
            <t t-set="show_wizard_checkout" t-value="True if show_wizard_checkout is None else show_wizard_checkout"/>
            <div id="wrap">
                <div class="oe_website_sale o_website_sale_checkout container py-2">
                    <div t-attf-class="row #{show_navigation_button and 'position-relative'} #{not show_wizard_checkout and 'mt32'} mb32">
                        <div t-if="show_wizard_checkout" class="col-12">
                            <t t-call="website_sale.wizard_checkout"/>
                        </div>
                        <div t-if="show_shorter_cart_summary"
                             class="offset-xl-1 col-lg-5 col-xl-4 order-2"
                             id="o_cart_summary">
                            <div class="o_total_card card sticky-lg-top"
                                 t-if="website_sale_order and website_sale_order.website_order_line">
                                <div class="card-body p-0 p-lg-4">
                                    <t t-call="website_sale.total"/>
                                    <t
                                        t-if="website.account_on_checkout != 'mandatory' or
                                              not website.is_public_user()"
                                        t-call="payment.express_checkout"
                                    />
                                    <t t-call="website_sale.navigation_buttons"/>
                                </div>
                            </div>
                        </div>
                        <div t-else=""
                             class="o_wsale_accordion accordion sticky-lg-top offset-xl-1 col-12 col-lg-5 col-xl-4 order-lg-2 rounded"
                             id="o_wsale_total_accordion">
                            <div class="o_total_card sticky-lg-top">
                                <div id="o_wsale_total_accordion_item" class="accordion-item p-lg-4 border-0">
                                    <div class="accordion-header d-block align-items-center mb-4">
                                        <button class="accordion-button px-0 collapsed"
                                                data-bs-toggle="collapse"
                                                data-bs-target="#o_wsale_accordion_item"
                                                aria-expanded="false"
                                                aria-controls="o_wsale_accordion_item">
                                            <div class="d-flex flex-wrap">
                                                <b class="w-100">Order summary</b>
                                                <span t-out="str(website_sale_order.cart_quantity)"/>
                                                &amp;nbsp;item(s)&amp;nbsp;-&amp;nbsp;
                                                <span id="amount_total_summary"
                                                    class="monetary_field ms-1"
                                                    t-field="website_sale_order.amount_total"
                                                    t-options='{"widget": "monetary", "display_currency": website_sale_order.currency_id}'/>
                                            </div>
                                        </button>
                                    </div>
                                    <div name="cart_summary_info" t-if="not website_sale_order or not website_sale_order.website_order_line" class="alert alert-info">
                                        Your cart is empty!
                                    </div>
                                    <div id="o_wsale_accordion_item"
                                        class="accordion-collapse collapse mb-4 mb-lg-0"
                                        data-bs-parent="#o_wsale_total_accordion">
                                        <div t-att-class="len(website_sale_order.website_order_line) &gt; 3 and 'o_wsale_scrollable_table mt-n4 me-n4 pt-4 pe-4'">
                                            <table t-if="website_sale_order and website_sale_order.website_order_line"
                                                class="table accordion-body mb-0"
                                                id="cart_products">
                                                <tbody>
                                                    <tr t-foreach="website_sale_order.website_order_line" t-as="line" t-att-class="line_last and 'border-transparent'">
                                                        <t t-set="o_cart_sum_padding_top"
                                                        t-value="'pt-3' if line_size &gt; 1 and not line_first else 'pt-0'"/>
                                                        <td t-if="not line.product_id" colspan="2"/>
                                                        <t t-else="">
                                                            <td t-attf-class="td-img ps-0 #{o_cart_sum_padding_top}">
                                                                <span t-if="line._is_not_sellable_line() and line.product_id.image_128">
                                                                    <img t-att-src="image_data_uri(line.product_id.image_128)" class="o_image_64_max img rounded" t-att-alt="line.name_short"/>
                                                                </span>
                                                                <span t-else=""
                                                                    t-field="line.product_id.image_128"
                                                                    t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'o_image_64_max rounded'}"
                                                                />
                                                            </td>
                                                            <td t-attf-class="#{o_cart_sum_padding_top} td-product_name td-qty w-100"
                                                                name='website_sale_cart_summary_product_name'>
                                                                <h6>
                                                                    <t t-out="int(line.product_uom_qty)" />
                                                                    <t t-if="line._get_shop_warning(clear=False)">
                                                                        <i class="fa fa-warning text-warning"
                                                                        role="img"
                                                                        t-att-title="line._get_shop_warning()"
                                                                        aria-label="Warning"/>
                                                                    </t>
                                                                    x
                                                                    <t t-out="line.name_short"/>
                                                                </h6>
                                                            </td>
                                                        </t>
                                                        <td t-attf-class="#{o_cart_sum_padding_top} td-price pe-0 text-end"
                                                            name="website_sale_cart_summary_line_price">
                                                            <span t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                                                                t-field="line.price_subtotal" style="white-space: nowrap;"
                                                                t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}"/>
                                                            <span t-else=""
                                                                t-field="line.price_total" style="white-space: nowrap;"
                                                                t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}"/>
                                                        </td>
                                                    </tr>
                                                </tbody>
                                            </table>
                                        </div>
                                        <t t-if='website_sale_order'>
                                            <t t-set='warning' t-value='website_sale_order._get_shop_warning(clear=False)' />
                                            <div t-if='warning' class="alert alert-warning" role="alert">
                                                <strong>Warning!</strong> <t t-esc='website_sale_order._get_shop_warning()'/>
                                            </div>
                                        </t>
                                    </div>
                                    <t t-call="website_sale.total">
                                        <t t-set="_cart_total_classes" t-valuef="border-top pt-3"/>
                                    </t>
                                    <div t-if="show_navigation_button" class="o_cta_navigation_container position-absolute position-lg-static start-0 bottom-0 col-12">
                                        <t t-call="website_sale.navigation_buttons"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div t-attf-class="oe_cart col-12 col-lg-7">
                            <t t-out="0"/>
                        </div>
                        <!-- This div serves as an anchor for the navigation buttons on the mobile
                             view. -->
                        <div t-if="not show_shorter_cart_summary and show_navigation_button"
                             class="o_cta_navigation_placeholder d-block d-none d-lg-none order-lg-4"/>
                    </div>
                </div>
                <!-- This is the drag-and-drop area for website building blocs at the end of each
                     checkout page. The templates created in the database to store blocs are hooked
                     using XPath on the `oe_struture` element ID. Therefore, we can't use dynamic
                     IDs (like with t-att-id) and each template needs to define a div element. -->
                <t t-out="oe_structure"/>
            </div>
        </t>
    </template>

    <!-- /shop/confirmation route -->
    <template id="confirmation">
        <t t-call="website_sale.checkout_layout">
            <t t-set="show_wizard_checkout" t-value="False"/>
            <t t-set="show_navigation_button" t-value="False"/>
            <t t-set="show_footer" t-value="True"/>
            <t t-set="additional_title">Shop - Confirmed</t>
            <t t-set="hide_promotions" t-value="True"/>
            <t t-set="oe_structure">
                <!-- This is the drag-and-drop area for website building blocs at the end of each
                     checkout page. This is append at the of the page in `checkout_layout`. The
                     templates created in the database to store blocs are hooked using XPath on the
                     `oe_struture` element ID. Therefore, we can't use dynamic IDs (like with
                     t-att-id) and each template needs to define a div element. -->
                <div class="oe_structure" id="oe_structure_website_sale_confirmation_3"/>
            </t>

            <t t-set="tx_sudo" t-value="order.get_portal_last_transaction()"/>
            <div t-if="tx_sudo.state in ['pending', 'done']" class="d-flex justify-content-between align-items-center">
                <h3>Thank you for your order.</h3>
                <a role="button" class="d-none d-md-inline-block btn btn-primary ms-auto" href="/shop/print" target="_blank" aria-label="Print" title="Print"><i class="fa fa-print me-2"></i>Print</a>
            </div>
            <t t-if="tx_sudo.state == 'done'">
                <div class="mb-4">
                    <h5>
                        <em>
                            <span>Order</span>
                            <span t-field="order.name" />
                            <t t-if="order.state == 'sale'">
                                <i class="fa fa-check-circle ms-1"/>
                            </t>
                        </em>
                    </h5>
                </div>
            </t>
            <t t-if="tx_sudo.state != 'done'">
                <div class="mb-4">
                    <h5>
                        <em>
                            <span>Order</span>
                            <span t-field="order.name" />
                            <t t-if="order.state == 'sale'">
                                <i class="fa fa-check-circle ms-1"/>
                            </t>
                        </em>
                    </h5>
                </div>
            </t>
            <t t-if="request.env['res.users']._get_signup_invitation_scope() == 'b2c' and request.website.is_public_user()">
                <p class="alert alert-info mt-3" role="status">
                    <a role="button" t-att-href="order.partner_id.signup_prepare() and order.partner_id.with_context(relative_url=True).signup_url" class="btn btn-primary">Sign Up</a>
                    to follow your order.
                </p>
            </t>
            <div class="oe_structure clearfix mt-3" id="oe_structure_website_sale_confirmation_1"/>
            <h4 class="text-start mt-3">Payment Information</h4>
            <table class="table">
                <tbody>
                    <tr>
                        <td colspan="2" class="ps-0">
                            <t t-esc="tx_sudo.provider_id.name" />
                        </td>
                        <td class="text-end pe-0" width="100">
                            <strong>Total:</strong>
                        </td>
                        <td class="text-end pe-0" width="100">
                            <strong t-field="tx_sudo.amount" t-options="{'widget': 'monetary', 'display_currency': order.currency_id}" />
                        </td>
                    </tr>
                </tbody>
            </table>
            <t t-call="website_sale.payment_confirmation_status"/>
            <div id="address_on_confirmation" class="mt-3">
                <t t-call="website_sale.address_on_payment">
                    <t t-set="disable_edit" t-value="True"/>
                </t>
            </div>
            <div class="oe_structure mt-3" id="oe_structure_website_sale_confirmation_2"/>
            <input t-if='website.plausible_shared_key' type='hidden' class='js_plausible_push' data-event-name='Shop' t-attf-data-event-params='{"CTA": "Order Confirmed", "amount": "#{"%3s-%3s" % (max(0, round(website_sale_order.amount_total/100)*100 - 50), round(website_sale_order.amount_total/100)*100 + 50)}"}' />
        </t>
    </template>

    <!-- Called in `website_sale.checkout_layout`. -->
    <template id="total">
        <div id="cart_total" t-if="website_sale_order and website_sale_order.website_order_line" t-att-class="_cart_total_classes">
            <table class="table mb-0">
                <tr id="order_total_untaxed">
                    <td id="cart_total_subtotal"
                        class="border-0 pb-2 ps-0 pt-0 text-start text-muted"
                        colspan="2">
                        Subtotal
                    </td>
                    <td class="text-end border-0 pb-2 pe-0 pt-0">
                        <span t-field="website_sale_order.amount_untaxed"
                              class="monetary_field"
                              style="white-space: nowrap;"
                              t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}"/>
                    </td>
                </tr>
                <tr id="order_total_taxes">
                    <td colspan="2" class="text-muted border-0 ps-0 pt-0 pb-3">Taxes</td>
                    <td class="text-end border-0 pe-0 pt-0 pb-3">
                        <span t-field="website_sale_order.amount_tax"
                              class="monetary_field"
                              style="white-space: nowrap;"
                              t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}"/>
                    </td>
                </tr>
                <tr id="order_total" class="border-top">
                    <td colspan="2" class="border-0 ps-0 pt-3"><strong>Total</strong></td>
                    <td class="text-end border-0 px-0 pt-3">
                        <strong t-field="website_sale_order.amount_total"
                                class="monetary_field text-end p-0"
                                t-options="{'widget': 'monetary', 'display_currency': website_sale_order.currency_id}"/>
                    </td>
                </tr>
            </table>
        </div>
    </template>

    <!-- Deactivatable through the website editor. -->
    <template id="reduction_code" inherit_id="website_sale.total" name="Promo Code">
        <!-- Checkout context:
            - hide_promotions: Whether to hide promotion input; default: `None`.
            - redirect: The route to redirect to when a customer enters a coupon; default: `None`.
            - website_sale_order: The current order.
        -->
        <xpath expr="//div[@id='cart_total']//table/tr[last()]" position="after">
            <tr t-if="not hide_promotions">
                <td colspan="3" class="text-end text-xl-end border-0 p-0">
                <span>
                    <t t-set="force_coupon" t-value="website_sale_order.pricelist_id.code"/>
                    <div t-if="not force_coupon" class="coupon_form">
                        <t t-call="website_sale.coupon_form"/>
                    </div>
                </span>
                </td>
            </tr>
        </xpath>
    </template>

    <!-- Called in `website_sale.confirmation`. -->
    <template id="payment_confirmation_status">
        <div class="oe_website_sale_tx_status mt-3" t-att-data-order-id="order.id" t-att-data-order-tracking-info="json.dumps(order_tracking_info)">
            <t t-set="tx_sudo" t-value="order.get_portal_last_transaction()"/>
            <div t-attf-class="card #{
                (tx_sudo.state == 'pending' and 'bg-info') or
                (tx_sudo.state == 'done' and order.amount_total == tx_sudo.amount and 'alert-success') or
                (tx_sudo.state == 'done' and order.amount_total != tx_sudo.amount and 'bg-warning') or
                (tx_sudo.state == 'authorized' and 'alert-success') or
                'bg-danger'}">
                <div class="card-header">
                    <a role="button" groups="base.group_system" class="btn btn-sm btn-link text-white float-end" target="_blank" aria-label="Edit" title="Edit"
                            t-attf-href="/web#model=payment.provider&amp;id=#{tx_sudo.provider_id.id}&amp;action=payment.action_payment_provider&amp;view_type=form">
                        <i class="fa fa-pencil"></i>
                    </a>
                    <t t-if="tx_sudo.state == 'pending'">
                        <t t-out="tx_sudo.provider_id.sudo().pending_msg"/>
                    </t>
                    <t t-if="tx_sudo.state == 'done'">
                        <span t-if='tx_sudo.provider_id.sudo().done_msg' t-out="tx_sudo.provider_id.sudo().done_msg"/>
                    </t>
                    <t t-if="tx_sudo.state == 'done' and order.amount_total != tx_sudo.amount">
                        <span>Unfortunately your order can not be confirmed as the amount of your payment does not match the amount of your cart.
                        Please contact the responsible of the shop for more information.</span>
                    </t>
                    <t t-if="tx_sudo.state == 'cancel'">
                        <t t-out="tx_sudo.provider_id.sudo().cancel_msg"/>
                    </t>
                    <t t-if="tx_sudo.state == 'authorized'">
                        <t t-if="tx_sudo.provider_id.sudo().auth_msg" t-out="tx_sudo.provider_id.sudo().auth_msg"/>
                        <span t-else="">Your payment has been authorized.</span>
                    </t>
                    <t t-if="tx_sudo.state == 'error'">
                        <span t-esc="tx_sudo.state_message"/>
                    </t>
                </div>
                <t t-if="tx_sudo.provider_code == 'custom'">
                    <div t-if="order.reference" class="card-body">
                        <b>Communication: </b><span t-esc='order.reference'/>
                    </div>
                    <div t-if="tx_sudo.provider_id.sudo().qr_code">
                        <t t-set="qr_code" t-value="tx_sudo.company_id.partner_id.bank_ids[:1].build_qr_code_base64(order.amount_total,tx_sudo.reference, None, tx_sudo.currency_id, tx_sudo.partner_id)"/>
                        <div class="card-body" t-if="qr_code">
                            <h3>Or scan me with your banking app.</h3>
                            <img class="border border-dark rounded" t-att-src="qr_code"/>
                        </div>
                    </div>
                </t>
            </div>
        </div>
    </template>

    <template id="website_sale.brand_promotion" inherit_id="website.brand_promotion">
        <xpath expr="//t[@t-call='web.brand_promotion_message']" position="replace">
            <t t-call="web.brand_promotion_message">
                <t t-set="_message">
                    The #1 <a target="_blank" href="http://www.odoo.com/app/ecommerce?utm_source=db&amp;utm_medium=website">Open Source eCommerce</a>
                </t>
                <t t-set="_utm_medium" t-valuef="website"/>
            </t>
        </xpath>
    </template>

    <template id="sale_order_portal_content_inherit_website_sale" name="Orders Followup Products Links" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//section[@id='details']//div[hasclass('table-responsive')]" position="attributes">
            <attribute name="class" remove="table-responsive" separator=" "/>
        </xpath>
        <xpath expr="//section[@id='details']//td[@id='product_name']/*" position="replace">
            <a t-if="line.product_id.website_published" t-att-href="line.product_id.website_url" class="d-block text-wrap" style="max-width: 35vw">
                <span t-field="line.name" />
            </a>
            <t t-if="not line.product_id.website_published">
                <span t-field="line.name" class="d-block text-wrap" style="max-width: 35vw"/>
            </t>
        </xpath>
    </template>

    <!-- Product page images -->
    <template id="website_sale.shop_product_images" name="Shop Product Images">
        <t t-set="product_images" t-value="product_variant._get_images() if product_variant else product._get_images()"/>
        <t t-set="ribbon" t-value="product_variant.sudo().ribbon_id or product.sudo().website_ribbon_id"/>
        <t t-set="bg_color" t-value="ribbon['bg_color'] or ''"/>
        <t t-set="text_color" t-value="ribbon['text_color']"/>
        <t t-set="bg_class" t-value="ribbon['html_class']"/>
        <t t-call="website_sale.shop_product_#{website.product_page_image_layout}"/>
    </template>

    <template id="website_sale.shop_product_image">
        <div t-if="product_image._name == 'product.image' and product_image.embed_code" t-att-class="image_classes + ' ratio ratio-16x9'">
            <t t-out="product_image.embed_code"/>
        </div>
        <div t-elif="len(product_images) == 1 and website.product_page_image_layout != 'grid'"
             class="position-relative d-inline-flex overflow-hidden m-auto h-100"
        >
            <span t-attf-class="o_ribbon #{ribbon['html_class']} z-index-1"
                  t-attf-style="#{text_color and ('color: %s; ' % text_color)}#{bg_color and 'background-color:' + bg_color}"
                  t-out="ribbon['html'] or ''"
            />
            <div t-field="product_image.image_1920"
                 class="d-flex align-items-start justify-content-center h-100 oe_unmovable"
                 t-options='{"widget": "image", "preview_image": "image_1024", "class": "oe_unmovable product_detail_img mh-100", "alt-field": "name", "zoom": product_image.can_image_1024_be_zoomed and "image_1920"}'
            />
        </div>
        <div t-else="" t-field="product_image.image_1920" t-att-class="image_classes + ' oe_unmovable'" t-options='{"widget": "image", "preview_image": "image_1024", "class": "oe_unmovable product_detail_img mh-100", "alt-field": "name", "zoom": product_image.can_image_1024_be_zoomed and "image_1920"}'/>
    </template>

    <!-- Product page images: Carousel -->
    <template id="website_sale.shop_product_carousel" name="Shop Product Carousel">
        <t t-set="product_carousel_block_name">Product Carousel</t>
        <div id="o-carousel-product" class="carousel slide position-sticky mb-3 overflow-hidden" data-bs-ride="carousel" data-bs-interval="0" t-att-data-name="product_carousel_block_name">
            <div class="o_carousel_product_outer carousel-outer position-relative flex-grow-1 overflow-hidden">
                <span t-if="len(product_images) > 1"
                      t-attf-class="o_ribbon #{ribbon['html_class']} z-index-1"
                      t-attf-style="#{text_color and ('color: %s; ' % text_color)}#{bg_color and 'background-color:' + bg_color}"
                      t-out="ribbon['html'] or ''"
                />
                <div class="carousel-inner h-100">
                    <t t-set="image_classes" t-value="'d-flex align-items-center justify-content-center h-100'"/>
                    <t t-foreach="product_images" t-as="product_image">
                        <div t-attf-class="carousel-item h-100 text-center#{' active' if product_image_first else ''}">
                            <t t-call="website_sale.shop_product_image"/>
                        </div>
                    </t>
                </div>
                <t t-if="len(product_images) > 1">
                    <a class="carousel-control-prev" href="#o-carousel-product" role="button" data-bs-slide="prev">
                        <span class="oi oi-chevron-left fa-2x oe_unmovable" role="img" aria-label="Previous" title="Previous"/>
                    </a>
                    <a class="carousel-control-next" href="#o-carousel-product" role="button" data-bs-slide="next">
                        <span class="oi oi-chevron-right fa-2x oe_unmovable" role="img" aria-label="Next" title="Next"/>
                    </a>
                </t>
            </div>
        </div>
    </template>

    <template id="carousel_product_indicators_bottom" inherit_id="website_sale.shop_product_carousel" name="Carousel Product Indicators Bottom">
        <xpath expr="//div[hasclass('o_carousel_product_outer')]" position="after">
            <t t-call="website_sale.carousel_product_indicators">
                <t t-set="indicators_div_class" t-value="'pt-2 overflow-hidden'"/>
            </t>
        </xpath>
    </template>

    <template id="carousel_product_indicators_left" inherit_id="website_sale.shop_product_carousel" name="Carousel Product Indicators Left" active="False">
        <xpath expr="//div[hasclass('o_carousel_product_outer')]" position="before">
            <t t-call="website_sale.carousel_product_indicators">
                <t t-set="indicators_list_class" t-value="'d-flex d-lg-block pe-2'"/>
            </t>
        </xpath>
        <xpath expr="//div[@id='o-carousel-product']" position="attributes">
            <attribute name="class" add="o_carousel_product_left_indicators d-flex" separator=" "/>
        </xpath>
    </template>

    <template id="carousel_product_indicators" name="Carousel Product">
        <div t-ignore="True" t-attf-class="o_carousel_product_indicators {{indicators_div_class}}">
            <ol t-if="len(product_images) > 1" t-attf-class="carousel-indicators {{indicators_list_class}} position-static pt-2 pt-lg-0 mx-auto my-0 text-start">
                <li t-foreach="product_images" t-as="product_image"
                    t-attf-class="align-top position-relative {{'active' if product_image_first else ''}}"
                    data-bs-target="#o-carousel-product"
                    t-att-data-bs-slide-to="str(product_image_index)">
                    <div t-field="product_image.image_128" t-options='{"widget": "image", "qweb_img_responsive": False, "class": "o_image_64_cover", "alt-field": "name"}'/>
                    <i t-if="product_image._name == 'product.image' and product_image.embed_code" class="fa fa-2x fa-play-circle o_product_video_thumb bg-black-50 text-center"/>
                </li>
            </ol>
        </div>
    </template>

    <!-- Product page images: Grid -->
    <template id="website_sale.shop_product_grid" name="Shop Product Grid">
        <div id="o-grid-product" class="o_wsale_product_page_grid mb-3" data-name="Product Grid" t-att-data-image_spacing="website.product_page_image_spacing" t-att-data-grid_columns="website.product_page_grid_columns">
            <div class="container position-relative overflow-hidden">
                <span t-attf-class="o_ribbon #{ribbon['html_class']}"
                      t-attf-style="#{text_color and ('color: %s; ' % text_color)}#{bg_color and 'background-color:' + bg_color}"
                      t-out="ribbon['html'] or ''"
                />
                <!-- One row for every two images -->
                <t t-set="image_classes" t-value="'w-100'"/>
                <t t-set="col_classes" t-value="website._get_product_page_grid_image_classes()"/>
                <t t-foreach="range(ceil(len(product_images) / website.product_page_grid_columns))" t-as="row_idx">
                    <div class="row m-0">
                        <t t-foreach="range(row_idx * website.product_page_grid_columns, (row_idx + 1) * website.product_page_grid_columns)" t-as="image_idx">
                            <t t-set="product_image" t-value="image_idx &lt; len(product_images) and product_images[image_idx] or False"/>
                            <div t-if="product_image" t-att-class="col_classes">
                                <t t-call="website_sale.shop_product_image"/>
                            </div>
                        </t>
                    </div>
                </t>
            </div>
        </div>
    </template>

    <template id="ecom_show_extra_fields" inherit_id="website_sale.product" active="True" name="Show Extra Fields">
        <xpath expr="//div[@id='product_details']" position="inside">
            <t t-if="any([product[field.name] for field in website.shop_extra_field_ids])">
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
            </t>
        </xpath>
    </template>

    <template id="add_to_cart_redirect" inherit_id="website.layout" name="Cart Redirection" priority="1">
        <xpath expr="//html" position="before">
            <t t-set="html_data" t-value="dict(html_data, **{'data-add2cart-redirect': '1' if website.add_to_cart_action == 'stay' else '0' if website.add_to_cart_action == 'go_to_cart' else '2'})"/>
        </xpath>
    </template>

    <template id="product_category_extra_link" name="Product Category Extra Link">
        <button  class="btn btn-link btn-sm pe-0" disabled="disabled">
            <t t-if="len(categories) == 1">Category:</t>
            <t t-else="">Categories:</t>
        </button>
        <t t-foreach="categories" t-as="category">
            <button class="btn btn-link btn-sm p-0 text-wrap" t-out="category.name"
                t-attf-onclick="location.href='/shop/category/#{slug(category)}';return false;"/>
        </t>
    </template>

    <template id="sale_order_re_order_btn" inherit_id="sale.sale_order_portal_template" name="Sale Order Order Again">
        <xpath expr="//t[@t-set='entries']/div/div/div[hasclass('o_download_pdf')]" position="before">
            <t t-if="request.website.enabled_portal_reorder_button and sale_order.with_user(request.env.user).sudo()._is_reorder_allowed()">
                <button class="btn btn-primary o_wsale_reorder_button w-100" t-att-data-sale-order-id="sale_order.id">
                    <i class="fa fa-rotate-right me-1"/>
                    Order Again
                </button>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\variant_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="variants">
        <t t-set="attribute_exclusions" t-value="product._get_attribute_exclusions(parent_combination)"/>
        <ul t-attf-class="list-unstyled js_add_cart_variants mb-0 #{ul_class}" t-att-data-attribute_exclusions="json.dumps(attribute_exclusions)">
            <t t-foreach="product.valid_product_template_attribute_line_ids" t-as="ptal">
                <!-- Attributes selection is hidden if there is only one value available and it's not a custom value -->
                <li t-att-data-attribute_id="ptal.attribute_id.id"
                    t-att-data-attribute_name="ptal.attribute_id.name"
                    t-att-data-attribute_display_type="ptal.attribute_id.display_type"
                    t-attf-class="variant_attribute #{
                        'd-none' if len(ptal.product_template_value_ids._only_active()) == 1
                        and not ptal.product_template_value_ids._only_active()[0].is_custom
                        and not ptal.attribute_id.display_type == 'multi' else ''
                    }">

                    <!-- Used to customize layout if the only available attribute value is custom -->
                    <t t-set="single" t-value="len(ptal.product_template_value_ids._only_active()) == 1"/>
                    <t t-set="single_and_custom" t-value="single and ptal.product_template_value_ids._only_active()[0].is_custom" />
                    <strong t-field="ptal.attribute_id.name" class="attribute_name"/>

                    <t t-if="ptal.attribute_id.display_type == 'select'">
                        <select
                            t-att-data-attribute_id="ptal.attribute_id.id"
                            t-attf-class="form-select css_attribute_select o_wsale_product_attribute js_variant_change #{ptal.attribute_id.create_variant} #{'d-none' if single_and_custom else ''}"
                            t-att-name="'ptal-%s' % ptal.id">
                            <t t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav">
                                <option t-att-value="ptav.id"
                                    t-att-data-value_id="ptav.id"
                                    t-att-data-value_name="ptav.name"
                                    t-att-data-attribute_name="ptav.attribute_id.name"
                                    t-att-data-is_custom="ptav.is_custom"
                                    t-att-selected="ptav in combination"
                                    t-att-data-is_single="single"
                                    t-att-data-is_single_and_custom="single_and_custom">
                                    <span t-field="ptav.name"/>
                                    <t t-call="website_sale.badge_extra_price"/>
                                </option>
                            </t>
                        </select>
                    </t>

                    <t t-elif="ptal.attribute_id.display_type in ('radio', 'multi')">
                        <ul t-att-data-attribute_id="ptal.attribute_id.id" t-attf-class="list-inline list-unstyled o_wsale_product_attribute #{'d-none' if single_and_custom else ''}">
                            <t t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav">
                                <li class="list-inline-item mb-3 js_attribute_value" style="margin: 0;">
                                    <label class="col-form-label">
                                        <div class="form-check">
                                            <input t-att-type="'radio' if ptal.attribute_id.display_type == 'radio' else 'checkbox'"
                                                t-attf-class="form-check-input js_variant_change #{ptal.attribute_id.create_variant}"
                                                t-att-checked="ptav in combination"
                                                t-att-name="'ptal-%s' % ptal.id"
                                                t-att-value="ptav.id"
                                                t-att-data-value_id="ptav.id"
                                                t-att-data-value_name="ptav.name"
                                                t-att-data-attribute_name="ptav.attribute_id.name"
                                                t-att-data-is_custom="ptav.is_custom"
                                                t-att-data-is_single="single if ptal.attribute_id.display_type != 'multi' else False"
                                                t-att-data-is_single_and_custom="single_and_custom"/>
                                            <div class="radio_input_value form-check-label">
                                                <span t-field="ptav.name"/>
                                                <t t-call="website_sale.badge_extra_price"/>
                                            </div>
                                        </div>
                                    </label>
                                </li>
                            </t>
                        </ul>
                    </t>

                    <t t-elif="ptal.attribute_id.display_type == 'pills'">
                        <ul t-att-data-attribute_id="ptal.attribute_id.id"
                            t-attf-class="btn-group-toggle list-inline list-unstyled o_wsale_product_attribute #{'d-none' if single_and_custom else ''}"
                            data-bs-toggle="buttons">
                            <t t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav">
                                <li t-attf-class="o_variant_pills btn btn-primary mb-1 list-inline-item js_attribute_value #{'active' if ptav in combination else ''}">
                                    <input type="radio"
                                        t-attf-class="js_variant_change #{ptal.attribute_id.create_variant}"
                                        t-att-checked="ptav in combination"
                                        t-att-name="'ptal-%s' % ptal.id"
                                        t-att-value="ptav.id"
                                        t-att-data-value_id="ptav.id"
                                        t-att-id="ptav.id"
                                        t-att-data-value_name="ptav.name"
                                        t-att-data-attribute_name="ptav.attribute_id.name"
                                        t-att-data-is_custom="ptav.is_custom"
                                        t-att-data-is_single_and_custom="single_and_custom"
                                        t-att-autocomplete="off"/>
                                    <label class="radio_input_value o_variant_pills_input_value"
                                           t-att-for="ptav.id">
                                        <span t-field="ptav.name"/>
                                        <t t-call="website_sale.badge_extra_price"/>
                                    </label>
                                </li>
                            </t>
                        </ul>
                    </t>

                    <t t-elif="ptal.attribute_id.display_type == 'color'">
                        <ul t-att-data-attribute_id="ptal.attribute_id.id" t-attf-class="list-inline o_wsale_product_attribute #{'d-none' if single_and_custom else ''}">
                            <li t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav" class="list-inline-item me-1">
                                <t t-set="img_style"
                                   t-value="'background:url(/web/image/product.template.attribute.value/%s/image); background-size:cover;' % ptav.id if ptav.image else ''"
                                />
                                <t t-set="color_style"
                                   t-value="'background:' + str(ptav.html_color or ptav.name if not ptav.is_custom else '')"
                                />
                                <label t-attf-style="#{img_style or color_style}"
                                       t-attf-class="css_attribute_color #{'active' if ptav in combination else ''} #{'custom_value' if ptav.is_custom else ''} #{'transparent' if (not ptav.is_custom and not ptav.html_color) else ''}"
                                >
                                      <input type="radio"
                                        t-attf-class="js_variant_change  #{ptal.attribute_id.create_variant}"
                                        t-att-checked="ptav in combination"
                                        t-att-name="'ptal-%s' % ptal.id"
                                        t-att-value="ptav.id"
                                        t-att-title="ptav.name"
                                        t-att-data-value_id="ptav.id"
                                        t-att-data-value_name="ptav.name"
                                        t-att-data-attribute_name="ptav.attribute_id.name"
                                        t-att-data-is_custom="ptav.is_custom"
                                        t-att-data-is_single="single"
                                        t-att-data-is_single_and_custom="single_and_custom"/>
                                </label>
                            </li>
                        </ul>
                    </t>
                </li>
            </t>
        </ul>
    </template>
    <template id="badge_extra_price" name="Badge Extra Price">
        <t t-set="price_extra" t-value="ptav._get_extra_price(combination_info)"/>
        <span t-if="price_extra" class="badge rounded-pill text-bg-light border">
            <!--
                price_extra is displayed as catalog price instead of
                price after pricelist because it is impossible to
                compute. Indeed, the pricelist rule might depend on the
                selected variant, so the price_extra will be different
                depending on the selected combination. The price of an
                attribute is therefore variable and it's not very
                accurate to display it.
            -->
            <span class="sign_badge_price_extra" t-out="price_extra > 0 and '+' or '-'"/>
            <span t-out="abs(price_extra)"
                class="variant_price_extra text-muted fst-italic"
                style="white-space: nowrap;"
                t-options='{
                    "widget": "monetary",
                    "display_currency": (pricelist or product.env.company).currency_id
                }'/>
        </span>
    </template>
</odoo>

```

## File: views\website_base_unit_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="base_unit_action" model="ir.actions.act_window">
        <field name="name">Base Units</field>
        <field name="res_model">website.base.unit</field>
        <field name="view_mode">tree,form</field>
    </record>

</odoo>

```

## File: views\website_pages_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<record id="product_pages_tree_view" model="ir.ui.view">
    <field name="name">Product Pages Tree</field>
    <field name="model">product.template</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="product_template_view_tree_website_sale"/>
    <field name="arch" type="xml">
        <xpath expr="//tree" position="attributes">
            <attribute name="js_class">website_pages_list</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
        </xpath>

        <field name="is_published" position="replace"/>

        <field name="default_code" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>
        <field name="product_tag_ids" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>
        <field name="standard_price" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>

        <field name="name" position="after">
            <field name="website_url"/>
        </field>
        <xpath expr="//tree">
            <field name="is_seo_optimized"/>
            <field name="is_published"/>

            <field name="website_id" position="move"/>
        </xpath>
    </field>
</record>

<record id="product_pages_kanban_view" model="ir.ui.view">
    <field name="name">Product Pages Kanban</field>
    <field name="model">product.template</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="product_template_view_kanban_website_sale"/>
    <field name="arch" type="xml">
        <kanban position="attributes">
            <attribute name="js_class">website_pages_kanban</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
        </kanban>
        <kanban position="inside">
            <field name="website_url"/>
        </kanban>
        <field name="name" position="after">
            <div class="text-muted" t-if="record.website_id.value" groups="website.group_multi_website">
                <i class="fa fa-globe me-1" title="Website"/>
                <field name="website_id"/>
            </div>
            <div class="text-primary" t-esc="record.website_url.value"/>
        </field>
        <xpath expr="//div[hasclass('oe_kanban_details')]" position="inside">
            <div class="o_kanban_footer">
                <div class="position-absolute bottom-0 end-0 m-1">
                    <t t-if="record.is_published.raw_value">Published </t>
                    <t t-else="">Not Published </t>
                    <field name="is_published" widget="boolean_toggle"/>
                </div>
            </div>
        </xpath>
    </field>
</record>

<record id="action_product_pages_list" model="ir.actions.act_window">
    <field name="name">Product Pages</field>
    <field name="res_model">product.template</field>
    <field name="view_mode">tree,kanban</field>
    <field name="view_id" ref="product_pages_tree_view"/>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'tree', 'sequence': 1, 'view_id': ref('product_pages_tree_view')}),
        (0, 0, {'view_mode': 'kanban', 'sequence': 2, 'view_id': ref('product_pages_kanban_view')}),
    ]"/>
    <field name="context">{'create_action': 'website_sale.product_product_action_add'}</field>
</record>

</odoo>

```

## File: views\website_sale_delivery_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="cart_delivery" name="Delivery Costs" inherit_id="website_sale.total">
        <tr id="order_total_untaxed" position="before">
            <tr id="order_delivery" t-if="website_sale_order and website_sale_order.carrier_id">
                <td colspan="2" class="ps-0 pt-0 pb-2 border-0 text-muted"
                    title="Delivery will be updated after choosing a new delivery method">
                    Delivery
                </td>
                <td class="text-end pe-0 pt-0 pb-2 border-0 text-muted">
                    <span t-field="website_sale_order.amount_delivery"
                          class="monetary_field"
                          style="white-space: nowrap;"
                          t-options='{"widget": "monetary", "display_currency": website_sale_order.currency_id}'/>
                </td>
            </tr>
        </tr>
    </template>

    <template id="payment_delivery_methods">
        <input class="pe-none" t-att-value="delivery.id" t-att-id="'delivery_%i' % delivery.id" t-att-delivery_type="delivery.delivery_type" type="radio" name="delivery_type" t-att-checked="order.carrier_id and order.carrier_id.id == delivery.id and 'checked' or False"/>
        <label class="label-optional" t-field="delivery.name"/>
        <span class="o_wsale_delivery_badge_price float-end fw-bold" name="price">Select to compute delivery rate</span>
        <t t-set='delivery_method' t-value="delivery.delivery_type+'_use_locations'" />
        <div class="small">
            <div class="d-none">
                <span class="o_order_location">
                    <b class="o_order_location_name"/>
                    <br/>
                    <i class="o_order_location_address"/>
                </span>
                <span class="fa fa-times ms-2 o_remove_order_location" aria-label="Remove this location" title="Remove this location"/>
            </div>
            <t t-if="delivery_method in delivery.fields_get() and delivery[delivery_method]">
                <div class="o_show_pickup_locations"/>
                <div class="o_list_pickup_locations"/>
            </t>
        </div>
        <t t-if="delivery.website_description">
            <div t-field="delivery.website_description" class="text-muted mt8"/>
        </t>
    </template>

    <template id="payment_delivery" name="Delivery Costs" inherit_id="website_sale.payment">
        <!-- //t[@t-if='website_sale_order.amount_total'] should be removed in master -->
        <xpath expr="//div[@name='website_sale_non_free_cart'] | //t[@name='website_sale_non_free_cart'] | //t[@t-if='website_sale_order.amount_total']" position="before">
            <div t-if="deliveries" id="delivery_carrier">
                <t t-set="delivery_nb" t-value="len(deliveries)"/>
                <h4 class="fs-6 small text-uppercase fw-bolder">Choose a delivery method</h4>
                <div class="card border-0" id="delivery_method">
                    <ul class="list-group">
                    <t t-foreach="deliveries" t-as="delivery">
                        <li class="list-group-item o_delivery_carrier_select">
                            <t t-call="website_sale.payment_delivery_methods"/>
                        </li>
                    </t>
                    </ul>
                </div>
            </div>
        </xpath>
    </template>

</odoo>

```

## File: views\website_sale_delivery_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_delivery_carrier_form_website_delivery" model="ir.ui.view">
        <field name="name">delivery.carrier.website.form</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_form"/>
        <field name="arch" type="xml">
            <field name="company_id" position='after'>
                <field name="website_id" groups="website.group_multi_website" options="{'no_open': True, 'no_create_edit': True}"/>
            </field>
            <field name="carrier_description" position='before'>
                <field name="website_description"  placeholder="Description displayed on the eCommerce and on online quotations."/>
            </field>
            <xpath expr="//button[@name='toggle_prod_environment']" position='before'>
                <button name="website_publish_button" type="object" class="oe_stat_button" icon="fa-globe">
                    <field name="is_published" widget="website_publish_button"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="view_delivery_carrier_tree" model="ir.ui.view">
        <field name="name">delivery.carrier.tree.inherit</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_tree"/>
        <field name="arch" type="xml">
            <field name="delivery_type" position="after">
                <field name="is_published"/>
                <field name="website_id" groups="website.group_multi_website"/>
            </field>
        </field>
    </record>

    <record id="view_delivery_carrier_search" model="ir.ui.view">
        <field name="name">delivery.carrier.search.inherit</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_search"/>
        <field name="arch" type="xml">
            <filter name="inactive" position="after">
                <filter string="Published" name="is_published" domain="[('is_published','=',True)]"/>
            </filter>
        </field>
    </record>

</odoo>

```

## File: views\website_sale_menus.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <menuitem id="menu_ecommerce"
        name="eCommerce"
        parent="website.menu_website_configuration"
        groups="sales_team.group_sale_salesman"
        sequence="20">

        <menuitem id="menu_orders"
            name="Orders"
            sequence="2">

            <menuitem id="menu_orders_orders"
                name="Orders"
                action="action_orders_ecommerce"
                sequence="1"/>
            <menuitem id="menu_orders_unpaid_orders"
                name="Unpaid Orders"
                action="action_view_unpaid_quotation_tree"
                sequence="2"/>
            <menuitem id="menu_orders_abandoned_orders"
                name="Abandoned Carts"
                action="action_view_abandoned_tree"
                sequence="3"/>
            <menuitem id="menu_orders_customers"
                name="Customers"
                action="base.action_partner_customer_form"
                sequence="4"/>

        </menuitem>

        <menuitem id="menu_catalog"
            name="Products"
            sequence="3">

            <menuitem id="menu_catalog_products"
                name="Products"
                action="product_template_action_website"
                sequence="1"/>
            <menuitem id="menu_catalog_pricelists"
                name="Pricelists"
                action="product.product_pricelist_action2"
                groups="product.group_product_pricelist"
                sequence="3"/>
            <menuitem id="menu_catalog_categories"
                action="product_public_category_action"
                sequence="4"/>
            <menuitem id="menu_product_attribute_action"
                action="product.attribute_action"
                groups="product.group_product_variant"
                sequence="5"/>
            <menuitem id="product_catalog_product_tags"
                name="Product Tags"
                action="product.product_tag_action"/>

        </menuitem>
    </menuitem>

    <menuitem id="menu_ecommerce_settings"
        name="eCommerce"
        parent="website.menu_website_global_configuration"
        sequence="50">

        <menuitem id="menu_ecommerce_payment_providers"
                  name="Payment Providers"
                  action="payment.action_payment_provider"
                  sequence="10"
        />
        <menuitem id="menu_ecommerce_payment_methods"
                  action="payment.action_payment_method"
                  name="Payment Methods"
                  sequence="20"
        />
        <menuitem id="menu_ecommerce_payment_tokens"
                  action="payment.action_payment_token"
                  groups="base.group_no_one"
                  sequence="30"
        />
        <menuitem id="menu_ecommerce_payment_transactions"
                  action="payment.action_payment_transaction"
                  groups="base.group_no_one"
                  sequence="40"
        />
        <menuitem id="menu_ecommerce_delivery"
                  action="delivery.action_delivery_carrier_form"
                  sequence="90"
        />
        <menuitem id="menu_delivery_zip_prefix"
                  action="delivery.action_delivery_zip_prefix_list"
                  groups="base.group_no_one"
                  sequence="100"
        />

    </menuitem>

    <!-- Reporting sub-menus -->
    <menuitem id="menu_report_sales" name="Online Sales"
        action="sale_report_action_dashboard"
        parent="website.menu_reporting"
        groups="sales_team.group_sale_manager"
        sequence="30"/>

    <menuitem id="menu_product_pages"
        name="Products"
        action="action_product_pages_list"
        parent="website.menu_content"
        sequence="30"/>

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
            <field name="url" position="after">
                <field name="product_id"/>
            </field>
            <filter name="type_url" position="after">
                <filter string="Products" name="type_product" domain="[('product_id', '!=', False)]"/>
            </filter>
            <filter name="group_by_url" position="after">
                <filter string="Product" name="group_by_product" domain="[]" context="{'group_by': 'product_id'}"/>
            </filter>
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
                    <field name="visitor_product_count" widget="statinfo" string="Product Views"/>
                </button>
            </xpath>
            <xpath expr="//group[@id='visits']/field[@name='page_ids']" position="after">
                <field name="product_ids" string="Products" widget="many2many_tags"/>
            </xpath>
        </field>
    </record>

    <record id="website_sale_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <field name="page_ids" position="after">
                <field name="product_ids" widget="many2many_tags" string="Products"/>
            </field>
        </field>
    </record>

    <record id="website_sale_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_kanban"/>
        <field name="arch" type="xml">
            <field name="page_ids" position="after">
                <field name="product_ids"/>
            </field>
            <xpath expr="//div[@id='o_page_count']" position="after">
                 <div id="o_product_count">Visited Products<span class="float-end fw-bold"><field name="product_count"/></span></div>
            </xpath>
        </field>
    </record>

    <!-- website track views -->
    <record id="website_sale_visitor_track_view_tree" model="ir.ui.view">
        <field name="name">website.track.view.tree</field>
        <field name="model">website.track</field>
        <field name="inherit_id" ref="website.website_visitor_track_view_tree"/>
        <field name="arch" type="xml">
            <field name="url" position="after">
                <field name="product_id"/>
            </field>
        </field>
    </record>

    <record id="website_sale_visitor_track_view_graph" model="ir.ui.view">
        <field name="name">website.track.view.graph</field>
        <field name="model">website.track</field>
        <field name="inherit_id" ref="website.website_visitor_track_view_graph"/>
        <field name="arch" type="xml">
            <field name="url" position="after">
                <field name="product_id"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\website_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="view_website_sale_website_form" model="ir.ui.view">
        <field name="name">website_sale.website.form</field>
        <field name="model">website</field>
        <field name="inherit_id" ref="website.view_website_form"/>
        <field name="arch" type="xml">
            <notebook position="inside">
                <page string="Product Page Extra Fields" name="page_product_page_extra_fields" groups="base.group_no_one">
                    <field name="shop_extra_field_ids" context="{'default_website_id': id}">
                        <tree editable="bottom">
                            <field name="sequence" widget="handle"/>
                            <field name="field_id" required="1" options="{'no_create': True}"/>
                        </tree>
                    </field>
                </page>
            </notebook>
        </field>
    </record>

</odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippets" inherit_id="website.snippets" name="e-commerce snippets">
    <xpath expr="//t[@id='sale_products_hook']" position="replace">
        <t t-snippet="website_sale.s_dynamic_snippet_products" string="Products" t-thumbnail="/website_sale/static/src/img/snippets_thumbs/s_dynamic_products.svg"/>
    </xpath>
    <xpath expr="//t[@id='snippet_add_to_cart_hook']" position="replace">
        <t t-snippet="website_sale.s_add_to_cart" string="Add to Cart Button"  t-thumbnail="/website/static/src/img/snippets_thumbs/s_add_to_cart.svg"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options" name="e-commerce snippet options">
    <xpath expr="." position="inside">
        <!-- All products page -->
        <div data-js="WebsiteSaleGridLayout" data-page-options="true" groups="website.group_website_designer" data-selector="main:has(.o_wsale_products_page)" data-no-check="true"
            string="Products Page" data-target="#products_grid .o_wsale_products_grid_table_wrapper > table">
            <we-select string="Layout" data-no-preview="true" data-reload="/">
                <we-button data-customize-website-views="" data-name="grid_view_opt">Grid</we-button>
                <we-button data-customize-website-views="website_sale.products_list_view">List</we-button>
            </we-select>
            <we-row string="Size" class="o_we_sublevel_1">
                <we-input data-set-ppg="" data-step="1" data-no-preview="true" data-reload="/"/>
                <span class="mx-2 o_wsale_ppr_by">by</span>
                <we-select class="o_wsale_ppr_submenu" data-dependencies="grid_view_opt" data-no-preview="true" data-reload="/">
                    <we-button data-set-ppr="2">2</we-button>
                    <we-button data-set-ppr="3">3</we-button>
                    <we-button data-set-ppr="4">4</we-button>
                </we-select>
            </we-row>
            <we-select string="Style" class="o_we_sublevel_1">
                <we-button data-select-class=""
                           data-customize-website-views="">
                           Default
                </we-button>
                <we-button data-select-class="o_wsale_design_cards"
                           data-customize-website-views="website_sale.products_design_card">
                           Cards
                </we-button>
                <we-button data-select-class="o_wsale_design_thumbs"
                           data-customize-website-views="website_sale.products_design_thumbs">
                           Thumbnails
                </we-button>
                <we-button data-select-class="o_wsale_design_grid"
                           data-customize-website-views="website_sale.products_design_grid">
                           Grid
                </we-button>
            </we-select>
            <we-select string="Images Size" class="o_we_sublevel_1">
                <we-button data-select-class="o_wsale_context_thumb_4_3"
                           data-customize-website-views="website_sale.products_thumb_4_3">
                           Landscape (4/3)
                </we-button>
                <we-button data-select-class=""
                           data-customize-website-views="">
                           Default (1/1)
                </we-button>
                <we-button data-select-class="o_wsale_context_thumb_4_5"
                           data-customize-website-views="website_sale.products_thumb_4_5">
                           Portrait (4/5)
                </we-button>
                <we-button data-select-class="o_wsale_context_thumb_2_3"
                           data-customize-website-views="website_sale.products_thumb_2_3">
                           Vertical (2/3)
                </we-button>
            </we-select>
            <we-button-group string="Fill" class="o_we_sublevel_2" data-variable="thumb_size">
                <we-button data-select-class=""
                           data-img="/website/static/src/img/snippets_options/content_width_normal.svg"
                           data-customize-website-views="">
                </we-button>
                <we-button data-select-class="o_wsale_context_thumb_cover"
                           data-name="thumb_cover"
                           data-variable="thumb_cover"
                           data-img="/website/static/src/img/snippets_options/content_width_full.svg"
                           data-customize-website-views="website_sale.products_thumb_cover">
                </we-button>
            </we-button-group>
            <we-checkbox string="Search bar"
                         data-customize-website-views="website_sale.search"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Prod. Desc."
                         data-customize-website-views="website_sale.products_description"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-row id="o_wsale_grid_left_panel" string="Categories" data-variable="filmstrip">
                <we-button string="Left"
                           data-customize-website-views="website_sale.products_categories"
                           data-name="categories_opt"
                           data-no-preview="true"
                           data-reload="/"/>
                <we-button string="Top"
                           data-customize-website-views="website_sale.products_categories_top"
                           data-name="categories_opt_top"
                           data-no-preview="true"
                           data-reload="/"/>
            </we-row>
            <we-checkbox id="collapse_category_recursive" string="Collapse Category Recursive"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_sale.option_collapse_products_categories"
                         data-dependencies="categories_opt"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-row string="Attributes" class="o_we_full_row">
                <we-button string="Left"
                           data-customize-website-views="website_sale.products_attributes"
                           data-name="attributes_opt"
                           data-no-preview="true"
                           data-reload="/"/>
                <we-button string="Top"
                           data-customize-website-views="website_sale.products_attributes_top"
                           data-name="attributes_opt_top"
                           data-no-preview="true"
                           data-reload="/"/>
            </we-row>
            <we-checkbox string="Price Filter"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_sale.filter_products_price"
                         data-dependencies="attributes_opt, attributes_opt_top"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Product Tags Filter"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_sale.filter_products_tags"
                         data-dependencies="attributes_opt, attributes_opt_top"
                         data-no-preview="true"
                         data-reload="/"
            />
            <we-row string="Top Bar" class="o_we_full_row">
                <we-button string="Sort by"
                           data-customize-website-views="website_sale.sort"
                           data-no-preview="true"
                           data-reload="/"/>
                <we-button string="Layout"
                           data-customize-website-views="website_sale.add_grid_or_list_option"
                           data-no-preview="true"
                           data-reload="/"/>
            </we-row>
            <we-select string="Default Sort" class="o_wsale_sort_submenu" data-no-preview="true" data-reload="/">
                <t t-foreach="request.env['website']._get_product_sort_mapping()" t-as="query_and_label">
                    <we-button t-att-data-set-default-sort="query_and_label[0]"><t t-esc="query_and_label[1]"/></we-button>
                </t>
            </we-select>
            <we-row string="Buttons" class="o_we_full_row">
                <we-button title="Add to Cart" class="fa fa-fw fa-shopping-cart o_we_add_to_cart_btn"
                           data-customize-website-views="website_sale.products_add_to_cart"
                           data-no-preview="true"
                           data-reload="/"/>
            </we-row>
        </div>
        <!-- Product -->
        <div data-js="WebsiteSaleProductsItem"
            data-selector="#products_grid .oe_product"
            data-no-check="true">
            <div class="o_wsale_soptions_menu_sizes">
                <we-row string="Size">
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
                </we-row>
            </div>

            <we-row string="Re-order" data-no-preview="true">
                <we-button title="Push to top" data-change-sequence="top" class="fa fa-fw fa-angle-double-left"/>
                <we-button title="Push up" data-change-sequence="up" class="fa fa-fw fa-angle-left"/>
                <we-button title="Push down" data-change-sequence="down" class="fa fa-fw fa-angle-right"/>
                <we-button title="Push to bottom" data-change-sequence="bottom" class="fa fa-fw fa-angle-double-right"/>
            </we-row>

            <we-row>
                <we-select string="Badge" class="o_wsale_ribbon_select">
                    <we-button data-set-ribbon="" data-name="no_ribbon_opt">None</we-button>
                    <!-- Ribbons are filled in JS -->
                </we-select>
                <we-button data-edit-ribbon="" title="Edit" class="fa fa-edit" data-no-preview="true" data-dependencies="!no_ribbon_opt"/>
                <we-button data-create-ribbon="" data-name="create_ribbon_opt" title="Create" class="fa fa-plus text-success" data-no-preview="true"/>
            </we-row>
            <div class="d-none" data-name="ribbon_customize_opt">
                <we-input string="Content" class="o_we_sublevel_1 o_we_large"
                          data-set-ribbon-html="Badge Text" data-apply-to=".o_ribbon"/>
                <we-colorpicker string="Background" class="o_we_sublevel_1"
                                title="" data-select-style="" data-css-property="background-color" data-color-prefix="text-bg-" data-apply-to=".o_ribbon"/>
                <we-colorpicker string="Text" class="o_we_sublevel_1"
                                title="" data-select-style="" data-css-property="color" data-apply-to=".o_ribbon"/>
                <we-select string="Style" class="o_we_sublevel_1">
                    <we-button data-set-ribbon-mode="ribbon">Slanted</we-button>
                    <we-button data-set-ribbon-mode="tag">Tag</we-button>
                </we-select>
                <we-select string="Position" class="o_we_sublevel_1">
                    <we-button data-set-ribbon-position="left">Left</we-button>
                    <we-button data-set-ribbon-position="right">Right</we-button>
                </we-select>
                <we-row string=" ">
                    <we-button class="o_we_bg_danger" data-delete-ribbon="" data-no-preview="true">Delete Badge</we-button>
                </we-row>
            </div>
        </div>
        <div data-selector="#wrapwrap > header"
            data-no-check="true"
            groups="website.group_website_designer">
            <we-row string="Show Empty" class="o_we_full_row">
                <div class="d-flex gap-1 mb-1 w-100">
                    <we-button title="Show/hide shopping cart" class="o_btn_show_empty_cart fa fa-shopping-cart d-flex justify-content-center flex-grow-1"
                            data-customize-website-views="website_sale.header_hide_empty_cart_link|"
                            data-no-preview="true"
                            data-reload="/"/>
                </div>
            </we-row>
        </div>
        <!-- Product image -->
        <div data-js="WebsiteSaleProductAttribute" data-selector="#product_detail .o_wsale_product_attribute" data-no-check="true">
            <we-select string="Display Type" data-no-preview="true">
                <we-button data-set-display-type="radio">Radio</we-button>
                <we-button data-set-display-type="pills">Pills</we-button>
                <we-button data-set-display-type="select">Select</we-button>
                <we-button data-set-display-type="color">Color</we-button>
            </we-select>
        </div>
        <!-- Product page -->
        <div data-js="WebsiteSaleProductPage" data-selector="main:has(.o_wsale_product_page)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Product Page">
            <we-row string="Customers" class="o_we_full_row">
                <we-button string="Rating"
                           data-customize-website-views="website_sale.product_comment"
                           data-no-preview="true"
                           data-reload="/"/>
                <we-button string="Share"
                           data-name="attributes_opt"
                           data-customize-website-views="website_sale.product_share_buttons"
                           data-no-preview="true"
                           data-reload="/"/>
            </we-row>
            <we-checkbox string="Select Quantity"
                         data-customize-website-views="website_sale.product_quantity"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Tax Indication"
                         data-customize-website-views="website_sale.tax_indication"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-select data-name="variants_opt" groups="product.group_product_variant" string="Variants" data-no-preview="true" data-reload="/">
                <we-button data-name="variants_options_opt" data-customize-website-views="">Options</we-button>
                <we-button data-name="variants_products_list_opt" data-customize-website-views="website_sale.product_variants">Products List</we-button>
            </we-select>
            <we-checkbox string="Product Tags"
                         data-customize-website-views="website_sale.product_tags"
                         data-no-preview="true"
                         data-reload="/"
            />
            <we-row string="Cart" class="o_we_full_row" data-name="o_wsale_buy_now_opt">
                <we-button title="Buy Now" class="o_we_buy_now_btn"
                           data-customize-website-views="website_sale.product_buy_now"
                           data-no-preview="true"
                           data-reload="/">
                    <i class="fa fa-fw fa-bolt"/>
                    Buy Now
                </we-button>
            </we-row>
            <!-- Image config -->
            <we-button-group string="Images Width" data-no-preview="true" data-reload="/">
                <we-button data-set-image-width="none" data-img="/website_sale/static/src/img/snippet_options/image-width-none.svg" title="None"/>
                <we-button data-set-image-width="50_pc" data-img="/website_sale/static/src/img/snippet_options/image-width-50.svg" title="50 percent"/>
                <we-button data-set-image-width="66_pc" data-img="/website_sale/static/src/img/snippet_options/image-width-66.svg" title="66 percent"/>
                <we-button data-set-image-width="100_pc" data-img="/website_sale/static/src/img/snippet_options/image-width-100.svg" title="100 percent"/>
            </we-button-group>
            <we-select string="Layout" data-name="o_wsale_image_layout" data-no-preview="true" data-reload="/">
                <we-button data-set-image-layout="carousel">Carousel</we-button>
                <we-button data-set-image-layout="grid">Grid</we-button>
            </we-select>
            <we-select string="Image Zoom" class="o_we_sublevel_1" data-name="o_wsale_zoom_mode" data-no-preview="true" data-reload="/">
                <we-button data-name="o_wsale_zoom_hover" data-customize-website-views="website_sale.product_picture_magnify_hover">Magnifier on hover</we-button>
                <we-button data-name="o_wsale_zoom_click" data-customize-website-views="website_sale.product_picture_magnify_click">Pop-up on Click</we-button>
                <we-button data-name="o_wsale_zoom_both" data-customize-website-views="website_sale.product_picture_magnify_both">Both</we-button>
                <we-button data-name="o_wsale_zoom_none" data-customize-website-views="">None</we-button>
            </we-select>
            <!-- Carousel config -->
            <we-button-group string="Thumbnails" class="o_we_sublevel_1" data-name="o_wsale_thumbnail_pos" data-no-preview="true" data-reload="/">
                <we-button class="fa fa-fw fa-long-arrow-left" title="Left" data-customize-website-views="website_sale.carousel_product_indicators_left"/>
                <we-button class="fa fa-fw fa-long-arrow-down" title="Bottom" data-customize-website-views="website_sale.carousel_product_indicators_bottom"/>
            </we-button-group>
            <!-- Grid config -->
            <we-range string="Image Spacing" class="o_we_sublevel_1" data-name="o_wsale_grid_spacing" data-no-preview="true" data-reload="/" data-max="3" data-step="1" data-set-spacing=""/>
            <we-select string="Columns" class="o_we_sublevel_1" data-name="o_wsale_grid_columns"  data-no-preview="true" data-reload="/">
                <we-button data-set-columns="1">1</we-button>
                <we-button data-set-columns="2">2</we-button>
                <we-button data-set-columns="3">3</we-button>
            </we-select>
            <we-row string="Main image">
                <we-button class="o_we_bg_success" data-name="o_wsale_replace_main_image" data-replace-main-image="true" data-no-preview="true">Replace</we-button>
            </we-row>
            <we-row string="Extra Images">
                <we-button class="o_we_bg_success" data-name="o_wsale_add_extra_images" data-add-images="true" data-no-preview="true">Add</we-button>
                <we-button class="o_we_bg_danger" data-name="o_wsale_clear_extra_images" data-clear-images="true" data-no-preview="true">Remove all</we-button>
            </we-row>
        </div>
        <!-- Checkout page  -->
        <div data-selector="main:has(.oe_website_sale .o_wizard)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Checkout Pages">
            <we-checkbox string="Extra Step"
                         data-customize-website-views="website_sale.extra_info"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Suggested Accessories"
                         data-customize-website-views="website_sale.suggested_products_list"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Promo Code"
                         data-customize-website-views="website_sale.reduction_code"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Accept Terms &amp; Conditions"
                         data-customize-website-views="website_sale.accept_terms_and_conditions"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Show b2b Fields"
                         data-customize-website-views="website_sale.address_b2b"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

<template id="snippets_options_web_editor" inherit_id="web_editor.snippet_options" name="e-commerce base snippet options">
    <xpath expr="//div[@data-js='ReplaceMedia']" position="inside">
        <we-row string="Re-order">
            <we-button class="fa fa-fw fa-angle-double-left" data-no-preview="true" title="Move to first" data-set-position="first" data-name="media_wsale_resequence"/>
            <we-button class="fa fa-fw fa-angle-left" data-no-preview="true" title="Move to previous" data-set-position="left" data-name="media_wsale_resequence"/>
            <we-button class="fa fa-fw fa-angle-right" data-no-preview="true" title="Move to next" data-set-position="right" data-name="media_wsale_resequence"/>
            <we-button class="fa fa-fw fa-angle-double-right" data-no-preview="true" title="Move to last" data-set-position="last" data-name="media_wsale_resequence"/>
        </we-row>
    </xpath>
    <xpath expr="//div[@data-js='ReplaceMedia']/we-row" position="inside">
        <we-button class="o_we_bg_danger" data-remove-media="true" data-no-preview="true" data-name="media_wsale_remove">Remove</we-button>
    </xpath>
</template>

<template id="product_searchbar_input_snippet_options" inherit_id="website.searchbar_input_snippet_options" name="product search bar snippet options">
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='scope_opt']" position="inside">
        <we-button data-set-search-type="products" data-select-data-attribute="products" data-name="search_products_opt" data-form-action="/shop">Products</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='order_opt']" position="inside">
        <t t-foreach="request.env['website']._get_product_sort_mapping()" t-as="query_and_label">
            <!-- name asc is already part of the general sorting methods of this snippet. -->
            <we-button t-if="query_and_label[0] != 'name asc'" t-att-data-set-order-by="query_and_label[0]" t-att-data-select-data-attribute="query_and_label[0]" data-dependencies="search_products_opt"><t t-out="query_and_label[1]"/></we-button>
        </t>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/div[@data-dependencies='limit_opt']" position="inside">
        <we-checkbox string="Description" data-dependencies="search_products_opt" data-select-data-attribute="true" data-attribute-name="displayDescription"
            data-apply-to=".search-query"/>
        <we-checkbox string="Category" data-dependencies="search_products_opt" data-select-data-attribute="true" data-attribute-name="displayExtraLink"
            data-apply-to=".search-query"/>
        <we-checkbox string="Price" data-dependencies="search_products_opt" data-select-data-attribute="true" data-attribute-name="displayDetail"
            data-apply-to=".search-query"/>
        <we-checkbox string="Image" data-dependencies="search_products_opt" data-select-data-attribute="true" data-attribute-name="displayImage"
            data-apply-to=".search-query"/>
    </xpath>
</template>

</odoo>

```

## File: views\snippets\s_add_to_cart.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template name="Add to Cart Button" id="s_add_to_cart">
        <div class="s_add_to_cart">
            <button class="s_add_to_cart_btn disabled btn btn-secondary mb-2">
                <i class="fa fa-cart-plus me-2"/>Add to Cart
            </button>
        </div>
    </template>
    <template id="s_add_to_cart_options" inherit_id="website.snippet_options">
        <xpath expr="." position="inside">
            <div data-js="AddToCart"
                 data-selector=".s_add_to_cart">
                <we-row>
                    <we-many2one string="Product"
                                 data-model="product.template"
                                 data-set-product-template=""
                                 data-name="product_template_picker_opt"
                                 data-no-preview="true"
                                 data-domain='[["is_published", "=", true], ["sale_ok", "=", true]]'
                    />
                    <we-button data-name="product_template_reset_opt"
                               class="reset-product-picker align-self-end fa fa-fw fa-times">
                    </we-button>
                </we-row>
                <we-row>
                    <we-many2one-default-message string="Variant" class="o_we_sublevel_1"
                                 data-model="product.product"
                                 data-set-product-variant=""
                                 data-name="product_variant_picker_opt"
                                 data-no-preview="true"
                                 data-default-message="Visitor's Choice"
                    />
                    <we-button data-name="product_variant_reset_opt"
                               class="reset-variant-picker align-self-end fa fa-fw fa-times">
                    </we-button>
                </we-row>
                <we-select data-name="action_picker_opt" string="Action" data-no-preview="true">
                    <we-button data-set-action="add_to_cart">Add to Cart</we-button>
                    <we-button data-set-action="buy_now">Buy Now</we-button>
                </we-select>
            </div>
        </xpath>
    </template>
    <record id="website_sale.s_add_to_cart_000_js" model="ir.asset">
        <field name="name">Add to Cart 000 JS</field>
        <field name="bundle">web.assets_frontend</field>
        <field name="path">website_sale/static/src/snippets/s_add_to_cart/000.js</field>
    </record>
</odoo>

```

## File: views\snippets\s_dynamic_snippet_products.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_dynamic_snippet_products" name="Products">
        <t t-call="website.s_dynamic_snippet_template">
            <t t-set="snippet_name" t-value="'s_dynamic_snippet_products'"/>
        </t>
    </template>
    <template id="s_dynamic_snippet_products_options" inherit_id="website.snippet_options">
        <xpath expr="." position="inside">
            <t t-call="website.dynamic_snippet_carousel_options_template">
                <t t-set="snippet_name" t-value="'dynamic_snippet_products'"/>
                <t t-set="snippet_selector" t-value="'.s_dynamic_snippet_products'"/>
            </t>
        </xpath>
    </template>
    <template id="s_dynamic_snippet_products_template_options" inherit_id="website.s_dynamic_snippet_options_template">
        <xpath expr="//we-select[@data-name='filter_opt']" position="after">
            <t t-if="snippet_name == 'dynamic_snippet_products'">
                <we-select string="Category" data-name="product_category_opt" data-attribute-name="productCategoryId" data-no-preview="true">
                    <we-button data-select-data-attribute="all">All Products</we-button>
                    <we-button data-select-data-attribute="current">Current Category or All</we-button>
                </we-select>
                <we-many2many string="Tags"
                    data-name="product_tag_opt"
                    data-attribute-name="productTagIds"
                    data-no-preview="true"
                    data-model="product.tag"
                    data-allow-delete="true"
                    data-fakem2m="true"
                    data-select-data-attribute=""/>
                <we-input string="Product names" class="o_we_large" data-name="product_names_opt"
                    data-attribute-name="productNames" data-no-preview="true" data-select-data-attribute=""
                    placeholder="e.g. lamp,bin" title="Comma-separated list of parts of product names, barcodes or internal reference"/>
            </t>
        </xpath>
    </template>

    <record id="website_sale.s_dynamic_snippet_products_000_js" model="ir.asset">
        <field name="name">Dynamic snippet products 000 JS</field>
        <field name="bundle">web.assets_frontend</field>
        <field name="path">website_sale/static/src/snippets/s_dynamic_snippet_products/000.js</field>
    </record>

</odoo>

```

## File: views\snippets\s_popup.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="website_sale.s_popup_000_js" model="ir.asset">
    <field name="name">Popup 000 JS Website Sale Override</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_sale/static/src/snippets/s_popup/000.js</field>
</record>

</odoo>

```

