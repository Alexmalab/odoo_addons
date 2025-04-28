# Odoo Module: pos_self_order

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import controllers
from . import models
from . import tests

def _post_self_order_post_init(env):
    sessions = env['pos.session'].search([('state', '!=', 'closed')])
    if len(sessions) > 0:
        env['pos.session']._create_pos_self_sessions_sequence(sessions)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    "name": "POS Self Order",
    'version': '1.0',
    "summary": "Addon for the POS App that allows customers to view the menu on their smartphone.",
    "category": "Sales/Point Of Sale",
    "depends": ["pos_restaurant", "http_routing"],
    "auto_install": ["pos_restaurant"],
    "demo": [
        "demo/pos_restaurant_demo.xml",
    ],
    "data": [
        "security/ir.model.access.csv",
        "views/pos_self_order.index.xml",
        "views/qr_code.xml",
        "views/pos_config_view.xml",
        "views/pos_session_view.xml",
        "views/custom_link_views.xml",
        "views/pos_restaurant_views.xml",
        "views/product_views.xml",
        "data/init_access.xml",
        "data/custom_link_data.xml",
        "views/res_config_settings_views.xml",
        "views/point_of_sale_dashboard.xml",
        "data/pos_restaurant_data.xml",
    ],
    "assets": {
        # Assets
        'point_of_sale._assets_pos': [
            'pos_self_order/static/src/overrides/**/*',
        ],
        'web.assets_backend': [
            "pos_self_order/static/src/upgrade_selection_field.js",
        ],
        "pos_self_order.assets": [
            "pos_self_order/static/src/app/primary_variables.scss",
            "pos_self_order/static/src/app/bootstrap_overridden.scss",
            "pos_self_order/static/scss/pos_self_order.scss",
            "point_of_sale/static/src/css/pos_receipts.css",
            ("include", "web._assets_helpers"),
            ("include", "web._assets_backend_helpers"),
            ("include", "web._assets_primary_variables"),
            "web/static/src/scss/pre_variables.scss",
            "web/static/lib/bootstrap/scss/_functions.scss",
            "web/static/lib/bootstrap/scss/_variables.scss",
            ("include", "web._assets_bootstrap"),
            ("include", "web._assets_bootstrap_backend"),
            ('include', 'web._assets_core'),
            ("remove", "web/static/src/core/browser/router_service.js"),
            ("remove", "web/static/src/core/debug/**/*"),
            "web/static/src/views/fields/formatters.js",
            "web/static/src/libs/fontawesome/css/font-awesome.css",
            "web/static/lib/odoo_ui_icons/*",
            'web/static/src/legacy/scss/ui.scss',
            "point_of_sale/static/src/utils.js",
            'bus/static/src/services/bus_service.js',
            'bus/static/src/bus_parameters_service.js',
            'bus/static/src/multi_tab_service.js',
            'bus/static/src/workers/*',
            'barcodes/static/src/barcode_service.js',
            'web/static/lib/bootstrap/js/dist/dom/data.js',
            'web/static/lib/bootstrap/js/dist/dom/event-handler.js',
            'web/static/lib/bootstrap/js/dist/dom/manipulator.js',
            'web/static/lib/bootstrap/js/dist/dom/selector-engine.js',
            'web/static/lib/bootstrap/js/dist/base-component.js',
            "web/static/lib/bootstrap/js/dist/carousel.js",
            'web/static/lib/bootstrap/js/dist/scrollspy.js',
            "point_of_sale/static/src/app/store/models/product_custom_attribute.js",
            "point_of_sale/static/src/utils.js",
            'web_editor/static/src/js/editor/odoo-editor/src/base_style.scss',
            'web_editor/static/src/scss/web_editor.common.scss',
            "point_of_sale/static/src/app/generic_components/**/*",
            "point_of_sale/static/src/css/pos_receipts.css",
            "point_of_sale/static/src/app/screens/receipt_screen/receipt/**/*",
            "pos_self_order/static/src/overrides/components/receipt_header/*",
            "point_of_sale/static/src/app/printer/base_printer.js",
            "point_of_sale/static/src/app/printer/printer_service.js",
            'point_of_sale/static/src/app/utils/html-to-image.js',
            "point_of_sale/static/src/app/printer/render_service.js",
            "pos_self_order/static/src/app/**/*",
            "point_of_sale/static/src/app/printer/hw_printer.js",
            "web/static/src/core/utils/render.js",
            "pos_self_order/static/src/app/store/order_change_receipt_template.xml",
        ],
        # Assets tests
        "pos_self_order.assets_tests": [
            "web/static/lib/jquery/jquery.js",
            "web_tour/static/src/tour_pointer/**/*.xml",
            "web_tour/static/src/tour_pointer/**/*.js",
            "web_tour/static/src/tour_service/**/*",
            "pos_self_order/static/tests/**/*",
        ],
    },
    'post_init_hook': '_post_self_order_post_init',
    "license": "LGPL-3",
}

```

## File: controllers\orders.py

```python
# -*- coding: utf-8 -*-
import re
import uuid
from datetime import timedelta
from odoo import http, fields
from odoo.http import request
from odoo.tools import float_round
from werkzeug.exceptions import NotFound, BadRequest, Unauthorized

class PosSelfOrderController(http.Controller):
    @http.route("/pos-self-order/process-new-order/<device_type>/", auth="public", type="json", website=True)
    def process_new_order(self, order, access_token, table_identifier, device_type):
        lines = order.get('lines')
        is_take_away = order.get('take_away')
        pos_config, table = self._verify_authorization(access_token, table_identifier, is_take_away)
        pos_session = pos_config.current_session_id
        ir_sequence_session = pos_config.env['ir.sequence'].with_context(company_id=pos_config.company_id.id).next_by_code(f'pos.order_{pos_session.id}')

        sequence_number = re.findall(r'\d+', ir_sequence_session)[0]
        order_reference = self._generate_unique_id(pos_session.id, pos_config.id, sequence_number, device_type)

        fiscal_position = (
            pos_config.self_ordering_alternative_fp_id
            if is_take_away
            else pos_config.default_fiscal_position_id
        )

        # Create the order without lines and prices computed
        # We need to remap the order because some required fields are not used in the frontend.
        order = {
            'data': {
                'name': order_reference,
                'sequence_number': sequence_number,
                'uuid': order.get('uuid'),
                'take_away': order.get('take_away'),
                'user_id': request.session.uid,
                'access_token': uuid.uuid4().hex,
                'pos_session_id': pos_session.id,
                'table_id': table.id if table else False,
                'partner_id': False,
                'date_order': str(fields.Datetime.now()),
                'fiscal_position_id': fiscal_position.id,
                'statement_ids': [],
                'lines': [],
                'amount_tax': 0,
                'amount_total': 0,
                'amount_paid': 0,
                'amount_return': 0,
                'table_stand_number': order.get('table_stand_number'),
                'ticket_code': order.get('ticket_code'),
                'pricelist_id': pos_config.pricelist_id.id if pos_config.pricelist_id else False
            },
            'to_invoice': False,
            'session_id': pos_session.id,
        }

        # Save the order in the database to get the id
        posted_order_id = pos_config.env['pos.order'].with_context(from_self=True).create_from_ui([order], draft=True)[0].get('id')

        # Process the lines and get their prices computed
        lines = self._process_lines(lines, pos_config, posted_order_id, is_take_away)

        # Compute the order prices
        amount_total, amount_untaxed = self._get_order_prices(lines)

        # Update the order with the computed prices and lines
        order = pos_config.env["pos.order"].browse(posted_order_id)

        classic_lines = []
        combo_lines = []
        for line in lines:
            if line["combo_parent_uuid"]:
                combo_lines.append(line)
            else:
                classic_lines.append(line)

        # combo lines must be created after classic_line, as they need the classic line identifier
        # use user admin to avoid access rights issues
        lines = pos_config.env['pos.order.line'].with_user(pos_config.self_ordering_default_user_id).create(classic_lines)
        lines += pos_config.env['pos.order.line'].with_user(pos_config.self_ordering_default_user_id).create(combo_lines)

        order.write({
            'lines': lines,
            'state': 'paid' if amount_total == 0 else 'draft',
            'amount_tax': amount_total - amount_untaxed,
            'amount_total': amount_total,
        })

        order.send_table_count_notification(order.table_id)
        return order._export_for_self_order()

    @http.route('/pos-self-order/get-orders-taxes', auth='public', type='json', website=True)
    def get_order_taxes(self, order, access_token):
        pos_config = self._verify_pos_config(access_token)
        lines = self._process_lines(order.get('lines'), pos_config, 0, order.get('take_away'))
        amount_total, amount_untaxed = self._get_order_prices(lines)

        return {
            'lines': [{
                'uuid': line.get('uuid'),
                'price_unit': line.get('price_unit'),
                'price_extra': line.get('price_extra'),
                'price_subtotal': line.get('price_subtotal'),
                'price_subtotal_incl': line.get('price_subtotal_incl'),
            } for line in lines],
            'amount_total': amount_total,
            'amount_tax': amount_total - amount_untaxed,
        }

    @http.route('/pos-self-order/update-existing-order', auth="public", type="json", website=True)
    def update_existing_order(self, order, access_token, table_identifier):
        order_id = order.get('id')
        order_access_token = order.get('access_token')
        pos_config, table = self._verify_authorization(access_token, table_identifier, order.get('take_away'))
        session = pos_config.current_session_id

        pos_order = session.order_ids.filtered_domain([
            ('id', '=', order_id),
            ('access_token', '=', order_access_token),
        ])

        if not pos_order:
            raise Unauthorized("Order not found in the server !")
        elif pos_order.state != 'draft':
            raise Unauthorized("Order is not in draft state")

        lines = self._process_lines(order.get('lines'), pos_config, pos_order.id, order.get('take_away'))
        for line in lines:
            if line.get('id'):
                # we need to find by uuid because each time we update the order, id of orderlines changed.
                order_line = pos_order.lines.filtered(lambda l: l.uuid == line.get('uuid'))

                if line.get('qty') < order_line.qty:
                    line.set('qty', order_line.qty)

                if order_line:
                    order_line.write({
                        **line,
                    })
            else:
                pos_order.lines.create(line)

        amount_total, amount_untaxed = self._get_order_prices(lines)
        pos_order.write({
            'amount_tax': amount_total - amount_untaxed,
            'amount_total': amount_total,
            'table_id': table if table else False,
            'table_stand_number': order.get('table_stand_number'),
        })

        pos_order.send_table_count_notification(pos_order.table_id)
        return pos_order._export_for_self_order()

    @http.route('/pos-self-order/get-orders', auth='public', type='json', website=True)
    def get_orders_by_access_token(self, access_token, order_access_tokens):
        pos_config = self._verify_pos_config(access_token)
        session = pos_config.current_session_id
        orders = session.order_ids.filtered_domain([
            ("access_token", "in", order_access_tokens),
            ("date_order", ">=", fields.Datetime.now() - timedelta(days=7)),
        ])

        if not orders:
            raise NotFound("Orders not found")

        orders_for_ui = []
        for order in orders:
            orders_for_ui.append(order._export_for_self_order())

        return orders_for_ui

    @http.route('/pos-self-order/get-tables', auth='public', type='json', website=True)
    def get_tables(self, access_token):
        pos_config = self._verify_pos_config(access_token)
        tables = pos_config.floor_ids.table_ids.filtered(lambda t: t.active).read(['id', 'name', 'identifier', 'floor_id'])

        for table in tables:
            table['floor_name'] = table.get('floor_id')[1]

        return tables


    @http.route('/kiosk/payment/<int:pos_config_id>/<device_type>', auth='public', type='json', website=True)
    def pos_self_order_kiosk_payment(self, pos_config_id, order, payment_method_id, access_token, device_type):
        pos_config = self._verify_pos_config(access_token)
        order_dict = self.process_new_order(order, access_token, None, device_type)

        if not order_dict.get('id'):
            raise BadRequest("Something went wrong")

        # access_token verified in process_new_order
        order_sudo = pos_config.env['pos.order'].browse(order_dict.get('id'))
        payment_method_sudo = pos_config.env["pos.payment.method"].browse(payment_method_id)
        if not order_sudo or not payment_method_sudo or payment_method_sudo not in order_sudo.config_id.payment_method_ids:
            raise NotFound("Order or payment method not found")

        status = payment_method_sudo._payment_request_from_kiosk(order_sudo)

        if not status:
            raise BadRequest("Something went wrong")

        return {'order': order_sudo._export_for_self_order(), 'payment_status': status}

    def _process_lines(self, lines, pos_config, pos_order_id, take_away=False):
        appended_uuid = []
        newLines = []
        pricelist = pos_config.pricelist_id
        sale_price_digits = pos_config.env['decimal.precision'].precision_get('Product Price')

        combo_line_ids = [line['combo_line_id'] for line in lines if line.get('combo_line_id')]
        combo_lines = pos_config.env['pos.combo.line'].search([('id', 'in', combo_line_ids)])
        attribute_value_ids = sum([line.get('attribute_value_ids', []) for line in lines], [])
        fetched_attributes = pos_config.env['product.template.attribute.value'].search([('id', 'in', attribute_value_ids)])

        fiscal_pos = pos_config.default_fiscal_position_id

        if take_away and pos_config.self_ordering_alternative_fp_id:
            fiscal_pos = pos_config.self_ordering_alternative_fp_id

        for line in lines:
            if line.get('uuid') in appended_uuid or not line.get('product_id'):
                continue

            line_qty = line.get('qty')
            product = pos_config.env['product.product'].browse(int(line.get('product_id')))
            lst_price = pricelist._get_product_price(product, quantity=line_qty) if pricelist else product.lst_price
            selected_attributes = fetched_attributes.browse(line.get('attribute_value_ids', []))
            price_extra = sum(attr.price_extra for attr in selected_attributes)
            lst_price += price_extra

            children = [l for l in lines if l.get('combo_parent_uuid') == line.get('uuid')]
            pos_combo_lines = combo_lines.browse([child.get('combo_line_id') for child in children])

            if len(children) > 0:
                original_total = sum(pos_combo_lines.mapped("combo_id.base_price"))
                remaining_total = lst_price
                factor = lst_price / original_total

                for i, child in enumerate(children):
                    child_product = pos_config.env['product.product'].browse(int(child.get('product_id')))
                    pos_combo_line = pos_combo_lines.browse(child.get('combo_line_id'))
                    price_unit = float_round(pos_combo_line.combo_id.base_price * factor, precision_digits=sale_price_digits)
                    remaining_total -= price_unit
                    if i == len(children) - 1:
                        price_unit += remaining_total

                    selected_attributes = fetched_attributes.browse(child.get('attribute_value_ids', []))
                    price_extra_child = sum(attr.price_extra for attr in selected_attributes)
                    price_unit += pos_combo_line.combo_price + price_extra_child

                    price_unit_fp = child_product._get_price_unit_after_fp(price_unit, pos_config.currency_id, fiscal_pos)
                    taxes = fiscal_pos.map_tax(child_product.taxes_id) if fiscal_pos else child_product.taxes_id
                    pdetails = taxes.compute_all(price_unit_fp, pos_config.currency_id, line_qty, child_product)

                    newLines.append({
                        'price_unit': price_unit_fp,
                        'price_subtotal': pdetails.get('total_excluded'),
                        'price_subtotal_incl': pdetails.get('total_included'),
                        'custom_attribute_value_ids': [[0, 0, cAttr] for cAttr in child.get('custom_attribute_value_ids')] if child.get('custom_attribute_value_ids') else [],
                        'id': child.get('id'),
                        'order_id': pos_order_id,
                        'tax_ids': child_product.taxes_id,
                        'uuid': child.get('uuid'),
                        'product_id': child.get('product_id'),
                        'qty': child.get('qty'),
                        'customer_note': child.get('customer_note'),
                        'attribute_value_ids': child.get('attribute_value_ids') or [],
                        'full_product_name': child.get('full_product_name'),
                        'combo_parent_uuid': child.get('combo_parent_uuid'),
                        'combo_id': child.get('combo_id'),
                        'price_extra': price_extra_child
                    })
                    appended_uuid.append(child.get('uuid'))

                lst_price = 0

            price_unit_fp = product._get_price_unit_after_fp(lst_price, pos_config.currency_id, fiscal_pos)
            taxes_after_fp = fiscal_pos.map_tax(product.taxes_id) if fiscal_pos else product.taxes_id
            pdetails = taxes_after_fp.compute_all(price_unit_fp, pos_config.currency_id, line_qty, product)

            newLines.append({
                'price_unit': price_unit_fp,
                'price_subtotal': pdetails.get('total_excluded'),
                'price_subtotal_incl': pdetails.get('total_included'),
                'id': line.get('id'),
                'order_id': pos_order_id,
                'tax_ids': product.taxes_id,
                'uuid': line.get('uuid'),
                'product_id': line.get('product_id'),
                'qty': line_qty,
                'customer_note': line.get('customer_note'),
                'attribute_value_ids': line.get('attribute_value_ids') or [],
                'custom_attribute_value_ids': [[0, 0, cAttr] for cAttr in line.get('custom_attribute_value_ids')] if line.get('custom_attribute_value_ids') else [],
                'full_product_name': line.get('full_product_name'),
                'combo_parent_uuid': line.get('combo_parent_uuid'),
                'combo_id': line.get('combo_id'),
                'price_extra': price_extra
            })
            appended_uuid.append(line.get('uuid'))

        return newLines

    def _get_order_prices(self, lines):
        amount_untaxed = sum([line.get('price_subtotal') for line in lines])
        amount_total = sum([line.get('price_subtotal_incl') for line in lines])
        return amount_total, amount_untaxed

    # The first part will be the session_id of the order.
    # The second part will be the table_id of the order.
    # Last part the sequence number of the order.
    # INFO: This is allow a maximum of 999 tables and 9999 orders per table, so about ~1M orders per session.
    # Example: 'Self-Order 00001-001-0001'
    def _generate_unique_id(self, pos_session_id, config_id, sequence_number, device_type):
        first_part = "{:05d}".format(int(pos_session_id))
        second_part = "{:03d}".format(int(config_id))
        third_part = "{:04d}".format(int(sequence_number))

        device = "Kiosk" if device_type == "kiosk" else "Self-Order"
        return f"{device} {first_part}-{second_part}-{third_part}"

    def _verify_pos_config(self, access_token):
        """
        Finds the pos.config with the given access_token and returns a record with reduced privileges.
        The record is has no sudo access and is in the context of the record's company and current pos.session's user.
        """
        pos_config_sudo = request.env['pos.config'].sudo().search([('access_token', '=', access_token)], limit=1)
        if not pos_config_sudo or (not pos_config_sudo.self_ordering_mode == 'mobile' and not pos_config_sudo.self_ordering_mode == 'kiosk') or not pos_config_sudo.has_active_session:
            raise Unauthorized("Invalid access token")
        company = pos_config_sudo.company_id
        user = pos_config_sudo.self_ordering_default_user_id
        return pos_config_sudo.sudo(False).with_company(company).with_user(user).with_context(allowed_company_ids=company.ids)

    def _verify_authorization(self, access_token, table_identifier, take_away):
        """
        Similar to _verify_pos_config but also looks for the restaurant.table of the given identifier.
        The restaurant.table record is also returned with reduced privileges.
        """
        pos_config = self._verify_pos_config(access_token)
        table_sudo = request.env["restaurant.table"].sudo().search([('identifier', '=', table_identifier)], limit=1)

        if not table_sudo and not pos_config.self_ordering_mode == 'kiosk' and pos_config.self_ordering_service_mode == 'table' and not take_away:
            raise Unauthorized("Table not found")

        company = pos_config.company_id
        user = pos_config.self_ordering_default_user_id
        table = table_sudo.sudo(False).with_company(company).with_user(user).with_context(allowed_company_ids=company.ids)
        return pos_config, table

```

## File: controllers\self_entry.py

```python
# -*- coding: utf-8 -*-
import werkzeug

from odoo import http
from odoo.http import request


class PosSelfKiosk(http.Controller):
    @http.route(["/pos-self/<config_id>", "/pos-self/<config_id>/<path:subpath>"], auth="public", website=True, sitemap=True)
    def start_self_ordering(self, config_id=None, access_token=None, table_identifier=None):
        table_sudo = False

        if not config_id or not config_id.isnumeric():
            raise werkzeug.exceptions.NotFound()

        if access_token:
            config_access_token = True
            pos_config_sudo = request.env["pos.config"].sudo().search([
                ("id", "=", config_id), ('access_token', '=', access_token)], limit=1)
        else:
            config_access_token = False
            pos_config_sudo = request.env["pos.config"].sudo().search([
                ("id", "=", config_id)], limit=1)

        if not pos_config_sudo or pos_config_sudo.self_ordering_mode == 'nothing':
            raise werkzeug.exceptions.NotFound()

        company = pos_config_sudo.company_id
        user = pos_config_sudo.self_ordering_default_user_id
        pos_config = pos_config_sudo.sudo(False).with_company(company).with_user(user).with_context(allowed_company_ids=company.ids)

        if not pos_config:
            raise werkzeug.exceptions.NotFound()

        if pos_config and pos_config.has_active_session and pos_config.self_ordering_mode == 'mobile':
            if config_access_token:
                config_access_token = pos_config.access_token
            table_sudo = table_identifier and (
                request.env["restaurant.table"]
                .sudo()
                .search([("identifier", "=", table_identifier), ("active", "=", True)], limit=1)
            )
        elif pos_config.self_ordering_mode == 'kiosk':
            if config_access_token:
                config_access_token = pos_config.access_token

        table = table_sudo.sudo(False).with_company(company).with_user(user) if table_sudo else False

        return request.render(
                'pos_self_order.index',
                {
                    'session_info': {
                        **request.env["ir.http"].get_frontend_session_info(),
                        'currencies': request.env["ir.http"].get_currencies(),
                        'pos_self_order_data': {
                            'table': table._get_self_order_data() if table else False,
                            'access_token': config_access_token,
                            **pos_config._get_self_ordering_data(),
                        },
                        "base_url": request.env['pos.session'].get_base_url(),
                        "db": request.env.cr.dbname,
                    }
                }
            )

    @http.route(
        "/pos-self/get-category-image/<int:category_id>",
        methods=["GET"],
        type="http",
        auth="public",
    )
    def pos_self_order_get_cat_image(self, category_id: int):
        category = request.env["pos.category"].sudo().browse(category_id)

        if not category.has_image:
            raise werkzeug.exceptions.NotFound()

        return (
            request.env["ir.binary"]
            ._get_image_stream_from(
                category,
                field_name="image_128",
            )
            .get_response()
        )

    @http.route(
        [
            "/menu/get-image/<int:product_id>",
            "/menu/get-image/<int:product_id>/<int:image_size>",
        ],
        methods=["GET"],
        type="http",
        auth="public",
    )
    def pos_self_order_get_image(self, product_id, image_size=128, **kw):
        # This controller is public and does not require an access code (access_token) because the user
        # needs to see the product image in "menu" mode. In this mode, the user has no access_token.
        self.get_any_pos_config_sudo()

        return (
            request.env["ir.binary"]
            ._get_image_stream_from(
                request.env["product.product"].sudo().browse(product_id),
                field_name=f"image_{image_size}",
            )
            .get_response()
        )

    def get_any_pos_config_sudo(self):
        pos_config_sudo = request.env["pos.config"].sudo().search([
            ("self_ordering_mode", "not in", ['nothing'])], limit=1)

        if not pos_config_sudo:
            raise werkzeug.exceptions.NotFound()

        return pos_config_sudo

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import orders
from . import self_entry

```

## File: data\custom_link_data.xml

```xml
<?xml version="1.0"?>
<odoo>
   <data noupdate="1">
      <record id="default_custom_link" model="pos_self_order.custom_link">
         <field name="name">Order Now</field>
         <field name="pos_config_ids" eval="[(4, ref('pos_restaurant.pos_config_main_restaurant'))]"/>
         <field name="url" model="pos.config" eval="'/pos-self/' + str(obj().env.ref('pos_restaurant.pos_config_main_restaurant').id) + '/products'" />
      </record>
   </data>
</odoo>

```

## File: data\init_access.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <function model="restaurant.table" name="_update_identifier" />
        <function model="pos.config" name="_init_access_token" />
    </data>
</odoo>

```

## File: data\pos_restaurant_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <record id="pos_restaurant.pos_config_main_restaurant" model="pos.config">
            <field name="self_ordering_mode">mobile</field>
            <field name="self_ordering_service_mode">table</field>
            <field name="self_ordering_pay_after">meal</field>
        </record>
    </data>
</odoo>

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import models
from odoo.http import request


class IrHttp(models.AbstractModel):
    _inherit = "ir.http"

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super()._get_translation_frontend_modules_name()
        return mods + ["pos_self_order"]

    # With the website module installed, there is an issue where
    # the default website's languages override the kiosk languages.
    # This override works around the issue.
    @classmethod
    def get_nearest_lang(cls, lang_code: str) -> str:
        if lang_code and '/pos-self/' in request.httprequest.path:
            config_id_match = re.search(r'/pos-self/(\d+)', request.httprequest.path)
            if config_id_match:
                pos_config = request.env['pos.config'].sudo().browse(int(config_id_match[1]))
                if pos_config.self_ordering_available_language_ids:
                    self_order_langs = pos_config.self_ordering_available_language_ids.mapped('code')
                    if lang_code in self_order_langs:
                        return lang_code
                    short_code = lang_code.partition('_')[0]
                    matched_code = next((code for code in self_order_langs if code.startswith(short_code)), None)
                    if matched_code:
                        return matched_code

        return super().get_nearest_lang(lang_code)

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import io
import uuid
import base64
from os.path import join as opj
from PIL import Image
from typing import Optional, List, Dict
from werkzeug.urls import url_quote
from odoo.exceptions import UserError
from odoo.tools import image_to_base64

from odoo import api, fields, models, _, service
from odoo.tools import file_open, split_every


class PosConfig(models.Model):
    _inherit = "pos.config"

    _ALLOWED_PAYMENT_METHODS = ['adyen', 'stripe']

    def _self_order_kiosk_default_languages(self):
        return self.env["res.lang"].get_installed()

    def _self_order_default_user(self):
        user_ids = self.env["res.users"].search(['|', ('company_id', '=', self.env.company.id), ('company_id', '=', False)])
        admin_user_ids = user_ids.filtered(lambda u: u.has_group("point_of_sale.group_pos_manager"))

        if admin_user_ids:
            return admin_user_ids[0]

        for user_id in user_ids:
            if user_id.has_group("point_of_sale.group_pos_user"):
                return user_id

    status = fields.Selection(
        [("inactive", "Inactive"), ("active", "Active")],
        string="Status",
        compute="_compute_status",
        store=False,
    )
    self_ordering_url = fields.Char(compute="_compute_self_ordering_url")
    self_ordering_takeaway = fields.Boolean("Takeaway")
    self_ordering_alternative_fp_id = fields.Many2one(
        'account.fiscal.position',
        string='Alternative Fiscal Position',
        help='This is useful for restaurants with onsite and take-away services that imply specific tax rates.',
    )
    self_ordering_mode = fields.Selection(
        [("nothing", "Disable"), ("consultation", "QR menu"), ("mobile", "QR menu + Ordering"), ("kiosk", "Kiosk")],
        string="Self Ordering Mode",
        default="nothing",
        help="Choose the self ordering mode",
        required=True,
    )
    self_ordering_service_mode = fields.Selection(
        [("counter", "Pickup zone"), ("table", "Table")],
        string="Service",
        default="counter",
        help="Choose the kiosk mode",
        required=True,
    )
    self_ordering_default_language_id = fields.Many2one(
        "res.lang",
        string="Default Language",
        help="Default language for the kiosk mode",
        default=lambda self: self.env["res.lang"].search(
            [("code", "=", self.env.lang)], limit=1
        ),
    )
    self_ordering_available_language_ids = fields.Many2many(
        "res.lang",
        string="Available Languages",
        help="Languages available for the kiosk mode",
        default=_self_order_kiosk_default_languages,
    )
    self_ordering_image_home_ids = fields.Many2many(
        'ir.attachment',
        string="Add images",
        help="Image to display on the self order screen",
    )
    self_ordering_default_user_id = fields.Many2one(
        "res.users",
        string="Default User",
        help="Access rights of this user will be used when visiting self order website when no session is open.",
        default=_self_order_default_user,
    )
    self_ordering_pay_after = fields.Selection(
        selection=lambda self: self._compute_selection_pay_after(),
        string="Pay After:",
        default="meal",
        help="Choose when the customer will pay",
        required=True,
    )
    self_ordering_image_brand = fields.Image(
        string="Self Order Kiosk Image Brand",
        help="Image to display on the self order screen",
        max_width=1200,
        max_height=250,
    )
    self_ordering_image_brand_name = fields.Char(
        string="Self Order Kiosk Image Brand Name",
        help="Name of the image to display on the self order screen",
    )
    access_token = fields.Char(
        "Security Token",
        copy=False,
        required=True,
        readonly=True,
        default=lambda self: self._get_access_token(),
    )

    @staticmethod
    def _get_access_token():
        return uuid.uuid4().hex[:16]

    def _update_access_token(self):
        self.access_token = self._get_access_token()
        self.floor_ids.table_ids._update_identifier()

    @api.model
    def _init_access_token(self):
        pos_config_ids = self.env["pos.config"].search([])
        for pos_config_id in pos_config_ids:
            pos_config_id.access_token = self._get_access_token()

    @api.model_create_multi
    def create(self, vals_list):
        pos_config_ids = super().create(vals_list)

        for pos_config_id in pos_config_ids:
            for image_name in ['landing_01.jpg', 'landing_02.jpg', 'landing_03.jpg']:
                image_path = opj("pos_self_order/static/img", image_name)
                attachment = self.env['ir.attachment'].create({
                    'name': image_name,
                    'datas': base64.b64encode(file_open(image_path, "rb").read()),
                    'res_model': 'pos.config',
                    'res_id': pos_config_id.id,
                    'type': 'binary',
                })
                pos_config_id.self_ordering_image_home_ids = [(4, attachment.id)]

            self.env['pos_self_order.custom_link'].create({
                'name': _('Order Now'),
                'url': f'/pos-self/{pos_config_id.id}/products',
                'pos_config_ids': [(4, pos_config_id.id)],
            })

            if pos_config_id.module_pos_restaurant:
                pos_config_id.self_ordering_mode = 'mobile'

        return pos_config_ids

    def _get_allowed_payment_methods(self):
        if self.self_ordering_mode == 'kiosk':
            return self.payment_method_ids.filtered(lambda p: p.use_payment_terminal in self._ALLOWED_PAYMENT_METHODS)
        return []

    def write(self, vals):
        for record in self:
            if vals.get('self_ordering_mode') == 'kiosk' or (vals.get('pos_self_ordering_mode') == 'mobile' and vals.get('pos_self_ordering_service_mode') == 'counter'):
                vals['self_ordering_pay_after'] = 'each'

            if (not vals.get('module_pos_restaurant') and not record.module_pos_restaurant) and vals.get('self_ordering_mode') == 'mobile':
                vals['self_ordering_pay_after'] = 'each'

            if (vals.get('self_ordering_service_mode') == 'counter' or record.self_ordering_service_mode == 'counter') and vals.get('self_ordering_mode') == 'mobile':
                vals['self_ordering_pay_after'] = 'each'

            if vals.get('self_ordering_mode') == 'mobile' and vals.get('self_ordering_pay_after') == 'meal':
                vals['self_ordering_service_mode'] = 'table'
        return super().write(vals)

    @api.depends("module_pos_restaurant")
    def _compute_self_order(self):
        for record in self:
            if not record.module_pos_restaurant and record.self_ordering_mode != 'kiosk':
                record.self_ordering_mode = 'nothing'

    def _compute_selection_pay_after(self):
        selection_each_label = _("Each Order")
        version_info = service.common.exp_version()['server_version_info']
        if version_info[-1] == '':
            selection_each_label = f"{selection_each_label} {_('(require Odoo Enterprise)')}"
        return [("meal", _("Meal")), ("each", selection_each_label)]

    @api.constrains('self_ordering_default_user_id')
    def _check_default_user(self):
        for record in self:
            if record.self_ordering_mode == 'mobile' and not record.self_ordering_default_user_id.has_group("point_of_sale.group_pos_user") and not record.self_ordering_default_user_id.has_group("point_of_sale.group_pos_manager"):
                raise UserError(_("The Self-Order default user must be a POS user"))

    def _get_qr_code_data(self):
        self.ensure_one()

        table_qr_code = []
        if self.self_ordering_mode == 'mobile' and self.module_pos_restaurant and self.self_ordering_service_mode == 'table':
            table_qr_code.extend([{
                    'name': floor.name,
                    'type': 'table',
                    'tables': [
                        {
                            'identifier': table.identifier,
                            'id': table.id,
                            'name': table.name,
                            'url': self._get_self_order_url(table.id),
                        }
                        for table in floor.table_ids.filtered("active")
                    ]
                }
                for floor in self.floor_ids]
            )
        else:
            # Here we use "range" to determine the number of QR codes to generate from
            # this list, which will then be inserted into a PDF.
            table_qr_code.extend([{
                'name': _('Generic'),
                'type': 'default',
                'tables': [{
                    'id': i,
                    'url': self._get_self_order_url(),
                } for i in range(0, 6)]
            }])

        return table_qr_code

    def _get_self_order_route(self, table_id: Optional[int] = None) -> str:
        self.ensure_one()
        base_route = f"/pos-self/{self.id}"
        table_route = ""

        if self.self_ordering_mode == 'consultation':
            return base_route

        if self.self_ordering_mode == 'mobile':
            table = self.env["restaurant.table"].search(
                [("active", "=", True), ("id", "=", table_id)], limit=1
            )

            if table:
                table_route = f"&table_identifier={table.identifier}"

        return f"{base_route}?access_token={self.access_token}{table_route}"

    def _get_self_order_url(self, table_id: Optional[int] = None) -> str:
        self.ensure_one()
        return url_quote(self.get_base_url() + self._get_self_order_route(table_id))

    def preview_self_order_app(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_url",
            "url": self._get_self_order_route(),
            "target": "new",
        }

    def _get_self_order_custom_links(self):
        """
        On the landing page of the app we can have a number of custom links
        that are defined by the restaurant employee in the backend.
        This function returns a list of dictionaries with the attributes of each link
        that is available for the POS with id pos_config_id.
        """
        self.ensure_one()
        return (
            self.env["pos_self_order.custom_link"]
            .search_read(
                [
                    "|",
                    ("pos_config_ids", "in", [self.id]),
                    ("pos_config_ids", "=", False),
                ],
                fields=["name", "url", "style"],
                order="sequence",
            )
        )

    def _get_available_products(self):
        self.ensure_one()
        combo_product_ids = self.env["product.product"].search([
            ("detailed_type", "=", 'combo'),
            *(
                self.limit_categories
                and self.iface_available_categ_ids
                and [("pos_categ_ids", "in", self.iface_available_categ_ids.ids)]
                or []
            ),
        ])
        product_ids = combo_product_ids.combo_ids.combo_line_ids.product_id

        available_product_ids = (
            self.env["product.product"]
            .search(
                [
                    ("id", "not in", product_ids.ids),
                    ("available_in_pos", "=", True),
                    ("self_order_available", "=", True),
                    *(
                        self.limit_categories
                        and self.iface_available_categ_ids
                        and [("pos_categ_ids", "in", self.iface_available_categ_ids.ids)]
                        or []
                    ),
                ],
            )
        )

        return product_ids + available_product_ids

    def _get_available_categories(self):
        self.ensure_one()
        return (
            self.env["pos.category"]
            .search(
                [
                    *(
                        self.limit_categories
                        and self.iface_available_categ_ids
                        and [("id", "in", self.iface_available_categ_ids.ids)]
                        or []
                    ),
                ],
                order="sequence",
            )
        )

    def _get_kitchen_printer(self):
        self.ensure_one()
        printerData = {}
        for printer in self.printer_ids:
            printerData[printer.id] = {
                "printer_type": printer.printer_type,
                "proxy_ip": printer.proxy_ip,
                "product_categories_ids": printer.product_categories_ids.ids,
            }
        return printerData

    def _get_self_ordering_payment_methods_data(self, payment_methods):
        excluded_fields = ['image']
        payment_search_fields = self.current_session_id._loader_params_pos_payment_method()['search_params']['fields']
        filtered_fields = [field for field in payment_search_fields if field not in excluded_fields]
        return payment_methods.read(filtered_fields)

    def _get_self_ordering_data(self):
        self.ensure_one()
        payment_methods = self._get_self_ordering_payment_methods_data(self._get_allowed_payment_methods())
        default_language = self.self_ordering_default_language_id.read(["code", "name", "iso_code", "flag_image_url"])

        return {
            "pos_config_id": self.id,
            "pos_session": self.current_session_id.read(["id", "access_token"])[0] if self.current_session_id and self.current_session_id.state == 'opened' else False,
            "company": {
                **self.company_id.read(["name", "email", "website", "vat", "name", "phone", "point_of_sale_use_ticket_qr_code", "point_of_sale_ticket_unique_code"])[0],
                "partner_id": [None, self.company_id.partner_id.contact_address],
                "country": self.company_id.country_id.read(["vat_label"])[0],
            },
            "base_url": self.get_base_url(),
            "custom_links": self._get_self_order_custom_links(),
            "currency_id": self.currency_id.id,
            "pos_payment_methods": payment_methods if self.self_ordering_mode == "kiosk" else [],
            "currency_decimals": self.currency_id.decimal_places,
            "pos_category": self._get_available_categories().read(["name", "sequence", "has_image"]),
            "products": self._get_available_products()._get_self_order_data(self),
            "combos": self._get_combos_data(),
            "config": {
                "iface_start_categ_id": self.iface_start_categ_id.id,
                "iface_tax_included": self.iface_tax_included == "total",
                "self_ordering_mode": self.self_ordering_mode,
                "self_ordering_takeaway": self.self_ordering_takeaway,
                "self_ordering_service_mode": self.self_ordering_service_mode,
                "self_ordering_default_language_id": default_language[0] if default_language else [],
                "self_ordering_available_language_ids":  self.self_ordering_available_language_ids.read(["code", "display_name", "iso_code", "flag_image_url"]),
                "self_ordering_image_home_ids": self._get_self_ordering_attachment(self.self_ordering_image_home_ids),
                "self_ordering_image_brand": self._get_self_ordering_image(self.self_ordering_image_brand),
                "self_ordering_pay_after": self.self_ordering_pay_after,
                "receipt_header": self.receipt_header,
                "receipt_footer": self.receipt_footer,
            },
            "kitchen_printers": self._get_kitchen_printer(),
        }

    def _get_combos_data(self):
        self.ensure_one()
        combos = self.env["pos.combo"].search([])

        return[{
            'id': combo.id,
            'name': combo.name,
            'combo_line_ids': combo.combo_line_ids.read(['product_id', 'combo_price', 'lst_price', 'combo_id'])
        } for combo in combos]


    def _get_self_ordering_image(self, image):
        image = Image.open(io.BytesIO(base64.b64decode(image))) if image else False
        return image_to_base64(image, 'PNG').decode('utf-8') if image else False

    def _get_self_ordering_attachment(self, images):
        encoded_images = []
        for image in images:
            encoded_images.append({
                'id': image.id,
                'data': image.sudo().datas.decode('utf-8'),
            })
        return encoded_images

    def _split_qr_codes_list(self, floors: List[Dict], cols: int) -> List[Dict]:
        """
        :floors: the list of floors
        :cols: the number of qr codes per row
        """
        self.ensure_one()
        return [
            {
                "name": floor.get("name"),
                "rows_of_tables": list(split_every(cols, floor["tables"], list)),
            }
            for floor in floors
        ]

    def _compute_self_ordering_url(self):
        for record in self:
            record.self_ordering_url = self.get_base_url() + self._get_self_order_route()

    def action_close_kiosk_session(self):
        current_session_id = self.current_session_id

        if current_session_id:
            if current_session_id.order_ids:
                current_session_id.order_ids.filtered(lambda o: o.state not in ['paid', 'invoiced']).unlink()

            self.env['bus.bus']._sendone(f'pos_config-{self.access_token}', 'STATUS', {
                'status': 'closed',
            })

            return current_session_id.action_pos_session_closing_control()

    def _compute_status(self):
        for record in self:
            record.status = 'active' if record.has_active_session else 'inactive'

    def action_open_kiosk(self):
        self.ensure_one()

        return {
            'name': _('Self Kiosk'),
            'type': 'ir.actions.act_url',
            'url': self._get_self_order_route(),
        }

    def action_open_wizard(self):
        self.ensure_one()

        if not self.current_session_id:
            self._check_before_creating_new_session()
            pos_session = self.env['pos.session'].create({'user_id': self.env.uid, 'config_id': self.id})
            pos_session._ensure_access_token()
            self.env['bus.bus']._sendone(f'pos_config-{self.access_token}', 'STATUS', {
                'status': 'open',
                'pos_session': pos_session.read(['id', 'access_token'])[0],
            })

        return {
            'name': _('Self Kiosk'),
            'type': 'ir.actions.act_window',
            'view_type': 'form',
            'view_mode': 'form',
            'res_model': 'pos.config',
            'res_id': self.id,
            'target': 'new',
            'views': [(self.env.ref('pos_self_order.pos_self_order_kiosk_read_only_form_dialog').id, 'form')],
        }

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from typing import Dict

from odoo import models, fields, api


class PosOrderLine(models.Model):
    _inherit = "pos.order.line"

    combo_parent_id = fields.Many2one('pos.order.line', string='Combo Parent')
    combo_line_ids = fields.One2many('pos.order.line', 'combo_parent_id', string='Combo Lines')
    combo_id = fields.Many2one('pos.combo', string='Combo line reference')

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if (vals.get('combo_parent_uuid')):
                vals.update([
                    ('combo_parent_id', self.search([('uuid', '=', vals.get('combo_parent_uuid'))]).id)
                ])
            if 'combo_parent_uuid' in vals:
                del vals['combo_parent_uuid']
        return super().create(vals_list)

    def write(self, vals):
        if (vals.get('combo_parent_uuid')):
            vals.update([
                ('combo_parent_id', self.search([('uuid', '=', vals.get('combo_parent_uuid'))]).id)
            ])
        if 'combo_parent_uuid' in vals:
            del vals['combo_parent_uuid']
        return super().write(vals)

class PosOrder(models.Model):
    _inherit = "pos.order"

    table_stand_number = fields.Char(string="Table Stand Number")
    take_away = fields.Boolean(string="Take Away", default=False)

    def _compute_tax_details(self):
        self.ensure_one()
        taxes = sum([line.tax_ids.with_company(self.company_id).compute_all(line.price_unit, quantity=line.qty, product=line.product_id)['taxes']
               for line in self.lines], [])
        tax_percetanges = {tax['id']: tax['amount'] for tax in self.env['account.tax'].search([]).read(['amount'])}
        merged_tax_details = {}
        for tax_obj in taxes:
            tax_id = tax_obj['id']
            if tax_id not in merged_tax_details:
                merged_tax_details[tax_id] = {
                    'tax': {
                        'id': tax_id,
                        'amount': tax_percetanges[tax_id]
                    },
                    'name': tax_obj['name'],
                    'amount': 0.0,
                    'base': 0.0,
                }
            merged_tax_details[tax_id]['amount'] += tax_obj['amount']
            merged_tax_details[tax_id]['base'] += tax_obj['base']
        return list(merged_tax_details.values())

    @api.model
    def create_from_ui(self, orders, draft=False):
        for order in orders:
            if order['data'].get('server_id'):
                server_id = order['data'].get('server_id')
                old_order = self.env['pos.order'].browse(server_id)
                if old_order.take_away:
                    order['data']['take_away'] = old_order.take_away

        return super().create_from_ui(orders, draft)

    def _process_saved_order(self, draft):
        res = super()._process_saved_order(draft)

        if self.env.context.get('from_self') is not True:
            self._send_notification(self)

        return res

    @api.model
    def remove_from_ui(self, server_ids):
        order_ids = self.env['pos.order'].browse(server_ids)
        order_ids.state = 'cancel'
        self._send_notification(order_ids)

        return super().remove_from_ui(server_ids)

    def _order_fields(self, ui_order):
        fields = super()._order_fields(ui_order)
        fields.update({
            'take_away': ui_order.get('take_away'),
            'table_stand_number': ui_order.get('table_stand_number'),
        })
        return fields

    def _send_notification(self, order_ids):
        for order in order_ids:
            if order.access_token and order.state != 'draft':
                self.env['bus.bus']._sendone(f'self_order-{order.access_token}', 'ORDER_STATE_CHANGED', {
                    'access_token': order.access_token,
                    'state': order.state
                })
            else:
                self.env['bus.bus']._sendone(f'self_order-{order.access_token}', 'ORDER_CHANGED', {
                    'order': order._export_for_self_order()
                })

    def _export_for_self_order(self) -> Dict:
        self.ensure_one()

        return {
            "id": self.id,
            "pos_config_id": self.config_id.id,
            "take_away": self.take_away,
            "pos_reference": self.pos_reference,
            "access_token": self.access_token,
            "table_stand_number": self.table_stand_number,
            "state": self.state,
            "date_order": str(self.date_order),
            "amount_total": self.amount_total,
            "amount_tax": self.amount_tax,
            "lines": [
                {
                    "id": line.id,
                    "price_subtotal": line.price_subtotal,
                    "price_subtotal_incl": line.price_subtotal_incl,
                    "product_id": line.product_id.id,
                    "attribute_value_ids": line.attribute_value_ids.ids,
                    "custom_attribute_value_ids": line.custom_attribute_value_ids.read(['id', 'name', 'custom_product_template_attribute_value_id', 'custom_value'], load=False),
                    "uuid": line.uuid,
                    "qty": line.qty,
                    "customer_note": line.customer_note,
                    "full_product_name": line.full_product_name,
                }
                for line in self.lines
            ],
            "payment_lines": [{
                "name": payment.payment_method_id.name,
                "amount": payment.amount
            } for payment in self.payment_ids],
            "tax_details": self._compute_tax_details(),
        }

    def get_standalone_self_order(self):
        orders = self.env['pos.order'].search([
            *self.env["pos.order"]._check_company_domain(self.env.company),
            ('state', '=', 'draft'),
            '|', ('pos_reference', 'ilike', 'Kiosk'),
            ('pos_reference', 'ilike', 'Self-Order'),
            ('table_id', '=', False),
        ])
        return orders

    @api.model
    def _get_shared_orders(self, config_id):
        orders = super()._get_shared_orders(config_id)
        self_orders = self.get_standalone_self_order()
        return orders | self_orders

    @api.model
    def export_for_ui_table_draft(self, table_ids):
        orders = super().export_for_ui_table_draft(table_ids)
        self_orders = self.get_standalone_self_order().export_for_ui()
        return orders + self_orders

```

## File: models\pos_payment_method.py

```python
from odoo import models


class PosPaymentMethod(models.Model):
    _inherit = "pos.payment.method"

    # will be overridden.
    def _payment_request_from_kiosk(self, order):
        pass

```

## File: models\pos_restaurant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import uuid
from typing import Dict, Callable, List, Optional

from odoo import api, fields, models


class RestaurantTable(models.Model):
    _inherit = "restaurant.table"

    identifier = fields.Char(
        "Security Token",
        copy=False,
        required=True,
        default=lambda self: self._get_identifier(),
    )

    def _get_self_order_data(self) -> Dict:
        self.ensure_one()
        return {
            'name': self.name,
            'identifier': self.identifier,
            'floor_name': self.floor_id.name
        }

    @staticmethod
    def _get_identifier():
        return uuid.uuid4().hex[:8]

    @api.model
    def _update_identifier(self):
        tables = self.env["restaurant.table"].search([])
        for table in tables:
            table.identifier = self._get_identifier()

```

## File: models\pos_self_order_custom_link.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import fields, models, api
from markupsafe import escape


class PosSelfOrderCustomLink(models.Model):
    _name = "pos_self_order.custom_link"
    _description = (
        "Custom links that the restaurant can configure to be displayed on the self order screen"
    )
    name = fields.Char(string="Label", required=True, translate=True)
    url = fields.Char(string="URL", required=True)
    pos_config_ids = fields.Many2many(
        "pos.config",
        string="Points of Sale",
        domain="[('self_ordering_mode', '!=', 'nothing')]",
        help="Select for which points of sale you want to display this link. Leave empty to display it for all points of sale. You have to select among the points of sale that have the 'QR Code Menu' feature enabled.",
    )
    style = fields.Selection(
        [
            ("primary", "Primary"),
            ("secondary", "Secondary"),
            ("success", "Success"),
            ("warning", "Warning"),
            ("danger", "Danger"),
            ("info", "Info"),
            ("light", "Light"),
            ("dark", "Dark"),
        ],
        string="Style",
        default="primary",
        required=True,
    )
    link_html = fields.Html("Preview", compute="_compute_link_html", store=True, readonly=True)
    sequence = fields.Integer("Sequence", default=1)

    @api.depends("name", "style")
    def _compute_link_html(self):
        for link in self:
            if link.name:
                link.link_html = f'<a class="btn btn-{link.style} w-100">{escape(link.name)}</a>'

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, _, fields


class PosSession(models.Model):
    _inherit = 'pos.session'

    @api.model_create_multi
    def create(self, vals_list):
        sessions = super(PosSession, self).create(vals_list)
        sessions = self._create_pos_self_sessions_sequence(sessions)
        return sessions

    @api.model
    def _create_pos_self_sessions_sequence(self, sessions):
        company_id = self.env.company.id

        for session in sessions:
            session.env['ir.sequence'].sudo().create({
                'name': _("PoS Order by Session"),
                'padding': 4,
                'code': f'pos.order_{session.id}',
                'number_next': 1,
                'number_increment': 1,
                'company_id': company_id,
            })

        return sessions

    def _loader_params_product_product(self):
        res = super()._loader_params_product_product()
        res['search_params']['fields'].append('self_order_available')
        return res

    def _pos_data_process(self, loaded_data):
        """
        This is where we need to process the data if we can't do it in the loader/getter
        """
        super()._pos_data_process(loaded_data)
        loaded_data["company_has_self_ordering"] = (
            self.env["pos.config"]
            .sudo()
            .search_count(
                [
                    *self.env["pos.config"]._check_company_domain(self.env.company),
                    '|', ("self_ordering_mode", "=", "kiosk"),
                    ("self_ordering_mode", "=", "mobile"),
                ],
                limit=1,
            )
            > 0
        )

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from __future__ import annotations

from typing import List, Dict, Optional

from odoo import api, models, fields
from copy import deepcopy

from odoo.addons.point_of_sale.models.pos_config import PosConfig


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    self_order_available = fields.Boolean(
        string="Available in Self Order",
        help="If this product is available in the Self Order screens",
        default=True,
    )
    description_self_order = fields.Html(
        string="Product Description for Self Order",
    )

    @api.onchange('available_in_pos')
    def _on_change_available_in_pos(self):
        for record in self:
            if not record.available_in_pos:
                record.self_order_available = False

    def write(self, vals_list):
        if 'available_in_pos' in vals_list:
            if not vals_list['available_in_pos']:
                vals_list['self_order_available'] = False

        res = super().write(vals_list)

        if 'self_order_available' in vals_list:
            for record in self:
                for product in record.product_variant_ids:
                    product._send_availability_status()
        return res

class ProductProduct(models.Model):
    _inherit = "product.product"

    def _get_name(self) -> str:
        """
        Returns the name of the product without the code.
        ex: product_sudo.display_name is '[FURN_7888] Desk Stand with Screen (Red)'
        :return: 'Desk Stand with Screen (Red)' (we remove the [FURN_7888] part)
        """
        self.ensure_one()
        return self.with_context(display_default_code=False).display_name

    def _filter_applicable_attributes(self, attributes_by_ptal_id: Dict) -> List[Dict]:
        """
        The attributes_by_ptal_id is a dictionary that contains all the attributes that have
        [('create_variant', '=', 'no_variant')]
        This method filters out the attributes that are not applicable to the product in self
        """
        self.ensure_one()
        return [
            attributes_by_ptal_id[id]
            for id in self.attribute_line_ids.ids
            if attributes_by_ptal_id.get(id) is not None
        ]

    def _get_attributes(self, pos_config_sudo: PosConfig) -> List[Dict]:
        self.ensure_one()

        attributes = self.env.context.get("cached_attributes_by_ptal_id")

        if attributes is None:
            attributes = self.env["pos.session"]._get_attributes_by_ptal_id()
            attributes = self._filter_applicable_attributes(attributes)
        else:
            # Performance trick to avoid unnecessary calls to _get_attributes_by_ptal_id()
            # Needs to be deep-copied because attributes is potentially mutated
            attributes = deepcopy(self._filter_applicable_attributes(attributes))

        # The image is not JSON serializable
        for attribute in attributes:
            for value in attribute["values"]:
                del value["image"]

        return self._add_price_info_to_attributes(
            attributes,
            pos_config_sudo,
        )

    def _add_price_info_to_attributes(
        self, attributes: List[Dict], pos_config_sudo: PosConfig
    ) -> List[Dict]:
        """
        Here we replace the price_extra of each attribute value with a price_extra
        dictionary that includes the price with taxes included and the price with taxes excluded
        """
        self.ensure_one()
        for attribute in attributes:
            for value in attribute["values"]:
                value.update(
                    {
                        "price_extra": self._get_price_info(
                            pos_config_sudo, value.get("price_extra")
                        )
                    }
                )
        return attributes

    def _get_price_unit_after_fp(self, lst_price, currency, fiscal_position):
        self.ensure_one()

        taxes = self.taxes_id

        mapped_included_taxes = self.env['account.tax']
        new_included_taxes = self.env['account.tax']

        for tax in taxes:
            mapped_taxes = fiscal_position.map_tax(tax)
            if mapped_taxes and any(mapped_taxes.mapped('price_include')):
                new_included_taxes |= mapped_taxes
            if tax.price_include and not (tax in mapped_taxes):
                mapped_included_taxes |= tax

        if mapped_included_taxes:
            if new_included_taxes:
                price_untaxed = mapped_included_taxes.compute_all(
                    lst_price,
                    currency,
                    1,
                    handle_price_include=True,
                )['total_excluded']
                return new_included_taxes.compute_all(
                    price_untaxed,
                    currency,
                    1,
                    handle_price_include=False,
                )['total_included']
            else:
                return mapped_included_taxes.compute_all(
                    lst_price,
                    currency,
                    1,
                    handle_price_include=True,
                )['total_excluded']
        else:
            return lst_price

    # FIXME: this method should be verified about price computation (pricelist taxes....)
    def _get_price_info(
        self, pos_config: PosConfig, price: Optional[float] = None, qty: int = 1
    ) -> Dict[str, float]:
        self.ensure_one()
        # if price == None it means that a price was not passed as a parameter, so we use the product's list price
        # it could happen that a price was passed, but it was 0; in that case we want to use this 0 as the argument,
        # and not the product's list price
        if price is None:
            price = pos_config.pricelist_id._get_product_price(
                self, qty, currency=pos_config.currency_id
            )

        # Declare variables, will be the return values.
        display_price_default = price
        display_price_alternative = price

        taxes_default = pos_config.default_fiscal_position_id.map_tax(self.taxes_id)
        taxes_alternative = pos_config.self_ordering_alternative_fp_id.map_tax(self.taxes_id)

        price_unit_default = self._get_price_unit_after_fp(
            price, pos_config.currency_id, pos_config.default_fiscal_position_id
        )
        price_unit_alternative = self._get_price_unit_after_fp(
            price, pos_config.currency_id, pos_config.self_ordering_alternative_fp_id
        )

        all_prices_default = taxes_default.compute_all(
            price_unit_default, pos_config.currency_id, qty, product=self
        )
        all_prices_alternative = taxes_alternative.compute_all(
            price_unit_alternative, pos_config.currency_id, qty, product=self
        )

        if self.combo_ids:
            display_price_default = self.lst_price
            display_price_alternative = self.lst_price
        else:
            if pos_config.iface_tax_included == 'total':
                display_price_default = all_prices_default["total_included"]
                display_price_alternative = all_prices_alternative["total_included"]
            else:
                display_price_default = all_prices_default["total_excluded"]
                display_price_alternative = all_prices_alternative["total_excluded"]

        return {
            'display_price_default': display_price_default,
            'display_price_alternative': display_price_alternative,
        }

    def _get_product_for_ui(self, pos_config):
        self.ensure_one()
        return {
                "price_info": self._get_price_info(pos_config),
                "has_image": bool(self.product_tmpl_id.image_128 or self.image_variant_128),
                "attributes": self._get_attributes(pos_config),
                "name": self._get_name(),
                "id": self.id,
                "description_self_order": self.description_self_order,
                "pos_categ_ids": self.pos_categ_ids.read(["id", "name"]) or [{"id": 0, "name": "Uncategorised"}],
                "pos_combo_ids": self.combo_ids.mapped("id") or False,
                "is_pos_groupable": self.uom_id.is_pos_groupable,
                "write_date": self.write_date.timestamp(),
                "self_order_available": self.self_order_available,
                "barcode": self.barcode,
            }

    def _get_self_order_data(self, pos_config: PosConfig) -> List[Dict]:
        attributes_by_ptal_id = self.env["pos.session"]._get_attributes_by_ptal_id()
        self = self.with_context(cached_attributes_by_ptal_id=attributes_by_ptal_id)
        return [
            product._get_product_for_ui(pos_config)
            for product in self
        ]

    def write(self, vals_list):
        res = super().write(vals_list)
        if 'self_order_available' in vals_list:
            for record in self:
                record._send_availability_status()
        return res

    def _send_availability_status(self):
        config_self = self.env['pos.config'].sudo().search([('self_ordering_mode', '!=', 'nothing')])
        for config in config_self:
            if config.current_session_id and config.access_token:
                self.env['bus.bus']._sendone(f'pos_config-{config.access_token}', 'PRODUCT_CHANGED', {
                    'product': self._get_product_for_ui(config)
                })

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

import qrcode
import zipfile
from io import BytesIO

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError
from odoo.tools.misc import split_every
from odoo.osv.expression import AND
from werkzeug.urls import url_unquote


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    pos_self_ordering_takeaway = fields.Boolean(related="pos_config_id.self_ordering_takeaway", readonly=False)
    pos_self_ordering_service_mode = fields.Selection(related="pos_config_id.self_ordering_service_mode", readonly=False, required=True)
    pos_self_ordering_mode = fields.Selection(related="pos_config_id.self_ordering_mode", readonly=False, required=True)
    pos_self_ordering_alternative_fp_id = fields.Many2one(related="pos_config_id.self_ordering_alternative_fp_id", readonly=False)
    pos_self_ordering_default_language_id = fields.Many2one(related="pos_config_id.self_ordering_default_language_id", readonly=False)
    pos_self_ordering_available_language_ids = fields.Many2many(related="pos_config_id.self_ordering_available_language_ids", readonly=False)
    pos_self_ordering_image_home_ids = fields.Many2many(related="pos_config_id.self_ordering_image_home_ids", readonly=False)
    pos_self_ordering_image_brand = fields.Image(related="pos_config_id.self_ordering_image_brand", readonly=False)
    pos_self_ordering_image_brand_name = fields.Char(related="pos_config_id.self_ordering_image_brand_name", readonly=False)
    pos_self_ordering_pay_after = fields.Selection(related="pos_config_id.self_ordering_pay_after", readonly=False, required=True)
    pos_self_ordering_default_user_id = fields.Many2one(related="pos_config_id.self_ordering_default_user_id", readonly=False)

    @api.onchange("pos_self_ordering_default_user_id")
    def _onchange_default_user(self):
        self.ensure_one()
        if self.pos_self_ordering_default_user_id and self.pos_self_ordering_mode == 'mobile':
            user_id = self.pos_self_ordering_default_user_id

            if not user_id.has_group("point_of_sale.group_pos_user") and not user_id.has_group("point_of_sale.group_pos_manager"):
                raise ValidationError(_("The user must be a POS user"))

    @api.onchange("pos_self_ordering_service_mode")
    def _onchange_pos_self_order_service_mode(self):
        if self.pos_self_ordering_service_mode == 'counter':
            self.pos_self_ordering_pay_after = "each"

    @api.onchange("pos_self_ordering_default_language_id", "pos_self_ordering_available_language_ids")
    def _onchange_pos_self_order_kiosk_default_language(self):
        if self.pos_self_ordering_default_language_id not in self.pos_self_ordering_available_language_ids:
            self.pos_self_ordering_available_language_ids = self.pos_self_ordering_available_language_ids + self.pos_self_ordering_default_language_id
        if not self.pos_self_ordering_default_language_id and self.pos_self_ordering_available_language_ids:
            self.pos_self_ordering_default_language_id = self.pos_self_ordering_available_language_ids[0]

    @api.onchange("pos_self_ordering_mode", "pos_module_pos_restaurant")
    def _onchange_pos_self_order_kiosk(self):
        if self.pos_self_ordering_mode == 'kiosk':
            self.is_kiosk_mode = True
            self.pos_module_pos_restaurant = False
            self.pos_self_ordering_pay_after = "each"
        else:
            self.is_kiosk_mode = False

            if not self.pos_module_pos_restaurant:
                self.pos_self_ordering_service_mode = 'counter'

    @api.onchange("pos_self_ordering_pay_after", "pos_self_ordering_mode")
    def _onchange_pos_self_order_pay_after(self):
        if self.pos_self_ordering_pay_after == "meal" and self.pos_self_ordering_mode == 'kiosk':
            raise ValidationError(_("Only pay after each is available with kiosk mode."))

        if self.pos_self_ordering_service_mode == 'counter' and self.pos_self_ordering_mode == 'mobile':
            self.pos_self_ordering_pay_after = "each"

        if self.pos_self_ordering_mode not in ['nothing', 'consultation'] and self.pos_self_ordering_pay_after == "each" and not self.module_pos_preparation_display:
            self.module_pos_preparation_display = True

    def custom_link_action(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "res_model": "pos_self_order.custom_link",
            "views": [[False, "tree"]],
            "domain": ['|', ['pos_config_ids', 'in', self.pos_config_id.id], ["pos_config_ids", "=", False]],
        }

    def _generate_single_qr_code(self, url):
        qr = qrcode.QRCode(
            version=1,
            error_correction=qrcode.constants.ERROR_CORRECT_L,
            box_size=10,
            border=4,
        )
        qr.add_data(url)
        qr.make(fit=True)
        return qr.make_image(fill_color="black", back_color="white")

    def generate_qr_codes_zip(self):
        if not self.pos_self_ordering_mode in ['mobile', 'consultation']:
            raise ValidationError(_("QR codes can only be generated in mobile or consultation mode."))

        qr_images = []

        if self.pos_module_pos_restaurant:
            table_ids = self.pos_config_id.floor_ids.table_ids

            if not table_ids:
                raise ValidationError(_("In Self-Order mode, you must have at least one table to generate QR codes"))

            for table in table_ids:
                qr_images.append({
                    'image': self._generate_single_qr_code(url_unquote(self.pos_config_id._get_self_order_url(table.id))),
                    'name': f"{table.floor_id.name} - {table.name}",
                })
        else:
            qr_images.append({
                'image': self._generate_single_qr_code(url_unquote(self.pos_config_id._get_self_order_url())),
                'name': "generic",
            })

        # Create a zip with all images in qr_images
        zip_buffer = BytesIO()
        with zipfile.ZipFile(zip_buffer, "w", 0) as zip_file:
            for index, qr_image in enumerate(qr_images):
                with zip_file.open(f"{qr_image['name']} ({index + 1}).png", "w") as buf:
                    qr_image['image'].save(buf, format="PNG")
        zip_buffer.seek(0)

        # Delete previous attachments
        self.env["ir.attachment"].search([
            ("name", "=", "self_order_qr_code.zip"),
        ]).unlink()

        # Create an attachment with the zip
        attachment_id = self.env["ir.attachment"].create({
            "name": "self_order_qr_code.zip",
            "type": "binary",
            "raw": zip_buffer.read(),
            "res_model": self._name,
            "res_id": self.id,
        })

        return {
            "type": "ir.actions.act_url",
            "url": f"/web/content/{attachment_id.id}",
            "target": "new",
        }

    def generate_qr_codes_page(self):
        """
        Generate the data needed to print the QR codes page
        """
        if self.pos_self_ordering_mode == 'mobile' and self.pos_module_pos_restaurant:
            table_ids = self.pos_config_id.floor_ids.table_ids

            if not table_ids:
                raise ValidationError(_("In Self-Order mode, you must have at least one table to generate QR codes"))

            url = url_unquote(self.pos_config_id._get_self_order_url(table_ids[0].id))
            name = table_ids[0].name
        else:
            url = url_unquote(self.pos_config_id._get_self_order_url())
            name = ""

        return self.env.ref("pos_self_order.report_self_order_qr_codes_page").report_action(
            [], data={
                'pos_name': self.pos_config_id.name,
                'floors': [
                    {
                        "name": floor.get("name"),
                        "type": floor.get("type"),
                        "table_rows": list(split_every(3, floor["tables"], list)),
                    }
                    for floor in self.pos_config_id._get_qr_code_data()
                ],
                'table_mode': self.pos_self_ordering_mode and self.pos_module_pos_restaurant and self.pos_self_ordering_service_mode == 'table',
                'self_order': self.pos_self_ordering_mode == 'mobile',
                'table_example': {
                    'name': name,
                    'decoded_url': url or "",
                }
            }
        )

    def preview_self_order_app(self):
        self.ensure_one()
        return self.pos_config_id.preview_self_order_app()

    def update_access_tokens(self):
        self.ensure_one()
        self.pos_config_id._update_access_token()

    @api.depends('pos_self_ordering_mode')
    def _compute_pos_pricelist_id(self):
        super()._compute_pos_pricelist_id()
        for res_config in self:
            if res_config.pos_self_ordering_mode == 'kiosk':
                currency_id = res_config.pos_journal_id.currency_id.id if res_config.pos_journal_id.currency_id else res_config.pos_config_id.company_id.currency_id.id
                domain = AND([self.env['product.pricelist']._check_company_domain(res_config.pos_config_id.company_id), [('currency_id', '=', currency_id)]])
                res_config.pos_available_pricelist_ids = self.env['product.pricelist'].search(domain)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import ir_http
from . import pos_config
from . import pos_order
from . import pos_restaurant
from . import pos_payment_method
from . import pos_self_order_custom_link
from . import product_product
from . import res_config_settings
from . import pos_session

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_pos_self_order_custom_link_manager,access.pos_self_order.custom_link_manager,model_pos_self_order_custom_link,point_of_sale.group_pos_manager,1,1,1,1
access_pos_self_order_custom_link_user,access.pos_self_order.custom_link_user,model_pos_self_order_custom_link,point_of_sale.group_pos_user,1,0,0,0

```

## File: static\img\eatin.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg id="a" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 610 410"><path d="m218.1691,292.2375c1.8935-1.4784,1.0329-4.5054-1.3549-4.7688-20.4595-2.2569-82.5111-9.0953-82.8429-9.0546-.644.0789-29.2824,19.1389-29.4616,19.2262-2.9932,1.4582-2.8174,4.8905-2.8174,4.8905-1.5448,4.4617,74.2129,19.884,75.9068,19.4397,3.0268-.794,29.2953-20.9297,40.5699-29.733Z" style="fill:#e6e6e6;"/><path d="m132.2009,276.4077c-.3695,4.7894,4.0104,6.3395,5.0107,1.7735l5.9942-49.0233c-2.7003,3.3404-5.4683,6.6008-8.4557,9.6752l-2.5491,37.5746Z" style="fill:#fff;"/><path d="m137.6847,195.5767l-2.9346,43.2563c2.9874-3.0744,5.7554-6.3348,8.4557-9.6752l3.9103-31.9802-9.4315-1.601Z" style="fill:#e6e6e6;"/><path d="m203.6035,234.8449l8.0376,52.9977c1.2406,4.5661,6.6732,3.0159,6.2148-1.7734l-5.2103-61.9188c-2.7886,3.7698-5.7278,7.4001-9.0422,10.6945Z" style="fill:#fff;"/><path d="m199.3561,206.8391l4.2474,28.0058c3.3144-3.2944,6.2536-6.9247,9.0422-10.6945l-1.5914-18.9123-11.6982,1.601Z" style="fill:#e6e6e6;"/><path d="m167.006,227.6952l2.2463,34.5315c3.223-4.6693,6.1813-9.5598,9.6141-14.1536l-.1107-21.1893-11.7498.8114Z" style="fill:#e6e6e6;"/><path d="m172.948,319.0376c1.0433,4.6429,6.5344,3.4606,6.2819-1.3527l-.3634-69.6119c-3.4328,4.5938-6.3911,9.4843-9.6141,14.1536l3.6956,56.811Z" style="fill:#fff;"/><path d="m114.5882,237.3569l2.2457-18.3661-9.4315-1.601-2.0258,29.8596c3.186-3.182,6.2502-6.4858,9.2115-9.8925Z" style="fill:#e6e6e6;"/><path d="m105.3767,247.2494l-3.458,50.9715c-.3695,4.7893,4.0104,6.3395,5.0107,1.7734l7.6589-62.6375c-2.9614,3.4067-6.0256,6.7105-9.2115,9.8925Z" style="fill:#fff;"/><path d="m137.6847,195.5767l-5.4838,80.831c-.3695,4.7893,4.0104,6.3395,5.0107,1.7734l9.9045-81.0035" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m211.0543,205.2381l6.8017,80.831c.4583,4.7893-4.9742,6.3395-6.2149,1.7734l-12.2849-81.0035" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m178.7558,226.8837l.474,90.8012c.2525,4.8133-5.2386,5.9956-6.2819,1.3526l-5.942-91.3425" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m107.4024,217.3898l-5.4838,80.831c-.3695,4.7893,4.0104,6.3395,5.0107,1.7734l9.9045-81.0035" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m207.819,205.1249l-5.3951,2.1302c-9.6183,3.7977-20.8793,10.029-25.2013,12.5291-1.6128.9329-3.4729,1.3097-5.3223,1.0837-27.9721-3.4184-70.2078-9.2944-70.2078-9.2944-1.5448,4.4617,2.1238,10.2163,6.8059,10.8253,11.1047,1.4443,29.5761,3.8358,38.8483,4.9741,12.791,1.5704,15.4887,1.7132,26.715,3.0623,1.7388.2089,3.5.0912,5.194-.3531,3.8769-1.0169,14.2031-5.1165,22.5224-8.4013l5.3951-2.1302c10.6578-4.2082,18.2589-7.2694,18.3671-10.0194.0137-.347.041-.6934.041-1.041.0388-.9871.0777-1.9741.1165-2.9612l.0378-.9602.0378-.9602c.0114-.2905.0267-.6778.0381-.9683.0116-.2953.0271-.6891.0387-.9844.0119-.3026.0278-.7061.0397-1.0087.0123-.3123.0287-.7286.041-1.0409.037-.9418-7.4536,1.3104-18.1114,5.5185Z" style="fill:#e6e6e6;"/><path d="m147.2866,188.5909c-.644.0789-10.1525,2.1954-20.544,6.5952-7.3535,3.1135-22.0537,12.4095-22.2329,12.4968-2.9932,1.4582-3.5228,4.668-.2166,5.0616,16.2604,1.9356,46.9235,5.5953,67.2682,8.0818,2.0659.2525,4.1462-.1839,5.9562-1.2112,4.4383-2.5187,15.5015-8.6466,24.9063-12.36l5.3951-2.1302c9.4011-3.712,16.3379-5.9019,17.8176-5.7205,0,0-77.9527-10.8622-78.35-10.8136Z" style="fill:#fff;"/><path d="m225.6129,199.404c.0067.0007.0173-.0004.0237.0004,0,0-77.9527-10.8622-78.35-10.8135-.644.0789-10.1525,2.1954-20.544,6.5952-7.3535,3.1135-22.0536,12.4095-22.2329,12.4968-2.9932,1.4582-2.8174,4.8905-2.8174,4.8905-1.5448,4.4617,2.1238,9.2163,6.8059,9.8253,11.1047,1.4443,29.5761,3.8358,38.8483,4.9741,12.791,1.5704,15.4886,1.7133,26.715,3.0623,1.7387.2089,3.5.0912,5.194-.3531,3.8769-1.017,14.2031-5.1165,22.5223-8.4013l5.3951-2.1302c10.6578-4.2082,18.2589-7.2694,18.3671-10.0194.0137-.347.041-.6934.041-1.041.0388-.9871.0777-1.9741.1165-2.9612l.0378-.9602.0378-.9602c.0114-.2905.0267-.6778.0381-.9683.0116-.2953.0271-.6891.0387-.9844.0119-.3026.0278-.7061.0397-1.0087.0123-.3123.0287-.7286.041-1.0409.0045-.1147-.1067-.1802-.3174-.2024Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m144.52,128.3687c-1.9854,1.8304-5.0377,3.5921-7.1401,5.2907l2.1368,22.8132c1.594-5.9979,3.5778-11.8893,5.799-17.7257-.2822-3.5135-.5507-6.9998-.7956-10.3782Z" style="fill:#e6e6e6;"/><path d="m145.3156,138.747c-2.2212,5.8364-4.205,11.7278-5.799,17.7257l3.1514,33.6464c1.7011,2.6765,7.6214,1.8163,7.2397-1.1901,0,0-2.6295-25.7501-4.5922-50.182Z" style="fill:#fff;"/><path d="m144.52,128.3688c-1.9854,1.8304-5.0377,3.5921-7.1401,5.2907l5.2882,56.4597c1.7011,2.6765,7.6214,1.8163,7.2397-1.1901,0,0-3.4402-33.6889-5.3878-60.5602Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m110.7565,153.7442c-.043-.1874-1.3683-.0269-2.954.1576.764-.2204-3.2639-.7382-4.6219-1.1625,0,0,.651,11.1796,1.3773,23.2753,1.6221-5.7974,3.9466-11.2741,6.5979-16.5973l-.3993-5.6731Z" style="fill:#e6e6e6;"/><path d="m104.5579,176.0145c.8992,14.9761,1.9139,31.3571,1.9518,29.6684,1.1073,3.9683,8.027,2.3769,7.8354-.9528l-3.1893-45.3129c-2.6513,5.3232-4.9758,10.7999-6.5979,16.5973Z" style="fill:#fff;"/><path d="m114.3451,204.7301l-3.5886-50.9859c-.043-.1874-1.3683-.0269-2.954.1576.7641-.2204-3.2639-.7382-4.6219-1.1626,0,0,3.2607,55.9964,3.3292,52.9437,1.1073,3.9683,8.027,2.3769,7.8354-.9528Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m157.9494,114.021c-.6154-8.4907-1.6078-22.1868-2.2838-31.5153-.3947-5.4457-9.5196-8.0549-7.8785-7.8534-3.2534-.3994-6.514-1.7209-9.7674-2.1203-4.5026-.5529-6.8105-.1174-16.212,3.5947l-16.5223,6.7764c-3.7249,1.4708-7.0264,3.3798-9.7505,5.2467-5.1611,3.537-8.0213,9.5717-7.5602,15.8116.8829,11.9478,3.4468,39.7474,3.4468,39.9705,0,7.2677,6.7855,10.3632,10.8387,11.5653,0,0,0,0-.0001,0,1.3112.6662,2.8659.8848,4.4261.4688,10.5057-2.8007,31.4326-12.4034,40.3396-18.7139,7.4599-5.2853,11.5845-14.1124,10.9236-23.231Z" style="fill:#e6e6e6;"/><path d="m155.6656,82.5056c-.3946-5.4457-3.3338-7.7529-8.5494-6.1377-2.4337.7537-5.2074,1.7048-8.2158,2.8926l-5.3951,2.1302c-6.1856,2.4423-23.066,8.5072-27.657,10.7423-4.9966,2.4325-8.0006,7.6617-7.5989,13.2044l3.2761,45.2052c.2821,3.8931,1.3893,6.4287,5.1608,5.4232,10.5057-2.8007,31.4326-12.4034,40.3396-18.714,7.4599-5.2853,11.5845-14.1124,10.9237-23.231-.6154-8.4906-1.6079-22.1868-2.2839-31.5153Z" style="fill:#fff;"/><path d="m157.9494,114.021c-.6154-8.4907-1.6078-22.1868-2.2838-31.5153-.3071-4.2373-3.6199-7.3811-7.5321-7.8068-1.1157-.1214-1.8067-.2723-2.5372-.4339-2.5254-.5587-5.0532-1.4232-7.5766-1.733-4.5026-.5529-6.8105-.1174-16.212,3.5947l-16.5223,6.7764c-3.7249,1.4708-7.0264,3.3798-9.7505,5.2467-5.1611,3.537-8.0213,9.5717-7.5602,15.8116.8829,11.9478,3.4468,39.7474,3.4468,39.9705,0,7.2677,6.7855,10.3632,10.8387,11.5653,0,0,0,0-.0001,0,1.3112.6662,2.8659.8848,4.4261.4688,10.5057-2.8007,31.4326-12.4034,40.3396-18.7139,7.4599-5.2853,11.5845-14.1124,10.9236-23.231Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m391.8309,292.2375c-1.8935-1.4784-1.0329-4.5054,1.3549-4.7688,20.4595-2.2569,82.5111-9.0953,82.8429-9.0546.644.0789,29.2824,19.1389,29.4616,19.2262,2.9932,1.4582,2.8174,4.8905,2.8174,4.8905,1.5448,4.4617-74.2129,19.884-75.9068,19.4397-3.0268-.794-29.2953-20.9297-40.5699-29.733Z" style="fill:#e6e6e6;"/><path d="m410.6439,206.8391l-11.6982-1.601-1.5914,18.9123-1.3388,15.9103c3.6818-2.8318,7.3041-5.6361,10.8755-8.4768l3.7528-24.7448Z" style="fill:#e6e6e6;"/><path d="m396.0155,240.0607l-3.8715,46.0085c-.4584,4.7894,4.9742,6.3395,6.2148,1.7734l8.0376-52.9977.4946-3.261c-3.5715,2.8407-7.1937,5.645-10.8755,8.4768Z" style="fill:#fff;"/><path d="m431.0692,260.3941l-.2991,57.2908c-.2525,4.8132,5.2386,5.9956,6.2819,1.3527l3.6956-56.811.8206-12.6138c-3.5047,3.621-6.9487,7.261-10.499,10.7812Z" style="fill:#fff;"/><path d="m431.2442,226.8838l-.1107,21.1893-.0643,12.321c3.5503-3.5202,6.9943-7.1602,10.499-10.7812l1.4258-21.9177-11.7498-.8114Z" style="fill:#e6e6e6;"/><path d="m474.8557,233.0234l-2.5405-37.4467-9.4315,1.601,3.9103,31.9802,1.4264,11.6657c2.0241-2.7877,4.2283-5.3952,6.6352-7.8002Z" style="fill:#e6e6e6;"/><path d="m475.2499,238.833l-.3942-5.8096c-2.4069,2.405-4.6111,5.0125-6.6352,7.8002l4.5678,37.3576c1.0002,4.566,5.3802,3.0159,5.0107-1.7735l-2.5491-37.5746Z" style="fill:#fff;"/><path d="m502.5975,217.3899l-9.4315,1.601,2.2457,18.3661.448,3.6638c1.8819-4.4982,4.5867-8.6456,7.4973-12.4366l-.7595-11.1943Z" style="fill:#e6e6e6;"/><path d="m508.0813,298.2209l-3.458-50.9715-1.2663-18.6653c-2.9106,3.791-5.6154,7.9384-7.4973,12.4366l7.2109,58.9736c1.0002,4.566,5.3802,3.0159,5.0107-1.7734Z" style="fill:#fff;"/><path d="m472.3153,195.5767l5.4838,80.831c.3695,4.7893-4.0104,6.3395-5.0107,1.7734l-9.9045-81.0035" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m398.9457,205.2381l-6.8017,80.831c-.4583,4.7893,4.9742,6.3395,6.2149,1.7734l12.2849-81.0035" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m431.2442,226.8837l-.474,90.8012c-.2525,4.8133,5.2386,5.9956,6.2819,1.3526l5.942-91.3425" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m502.5976,217.3898l5.4838,80.831c.3695,4.7893-4.0104,6.3395-5.0107,1.7734l-9.9045-81.0035" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m508.3076,211.5734s-42.2357,5.876-70.2078,9.2944c-1.8494.226-3.7095-.1508-5.3223-1.0837-4.322-2.5001-15.5831-8.7313-25.2014-12.5291l-5.3951-2.1302c-10.6578-4.2081-18.1484-6.4603-18.1114-5.5185.0123.3123.0287.7286.041,1.0409.0119.3026.0278.7061.0397,1.0087.0116.2953.0271.689.0387.9844.0114.2905.0267.6778.0381.9683l.0378.9602.0378.9602c.0388.9871.0776,1.9741.1165,2.9612,0,.3475.0273.694.041,1.041.1082,2.75,7.7092,5.8112,18.3671,10.0194l5.3951,2.1302c8.3193,3.2848,18.6454,7.3843,22.5224,8.4013,1.694.4443,3.4552.562,5.194.3531,11.2263-1.3491,13.924-1.4919,26.715-3.0623,9.2722-1.1384,27.7435-3.5298,38.8483-4.9741,4.6821-.609,8.3507-6.3636,6.8059-10.8253Z" style="fill:#e6e6e6;"/><path d="m462.7134,188.5909c.644.0789,10.1525,2.1954,20.544,6.5952,7.3535,3.1135,22.0537,12.4095,22.2329,12.4968,2.9932,1.4582,3.5228,4.668.2166,5.0616-16.2604,1.9356-46.9235,5.5953-67.2682,8.0818-2.0659.2525-4.1462-.1839-5.9562-1.2112-4.4383-2.5187-15.5015-8.6466-24.9063-12.36l-5.3951-2.1302c-9.4011-3.712-16.3379-5.9019-17.8176-5.7205,0,0,77.9527-10.8622,78.35-10.8136Z" style="fill:#fff;"/><path d="m508.3076,212.5734s.1758-3.4323-2.8174-4.8905c-.1793-.0873-14.8795-9.3833-22.2329-12.4968-10.3915-4.3998-19.9-6.5164-20.544-6.5952-.3973-.0486-78.35,10.8135-78.35,10.8135.0064-.0008.017.0002.0237-.0004-.2108.0222-.322.0877-.3174.2024.0123.3123.0287.7286.041,1.0409.0119.3026.0278.7061.0397,1.0087.0116.2953.0271.689.0387.9844.0114.2905.0267.6778.0381.9683l.0378.9602.0378.9602c.0388.9871.0776,1.9741.1165,2.9612,0,.3475.0273.694.041,1.041.1082,2.75,7.7092,5.8112,18.3671,10.0194l5.3951,2.1302c8.3192,3.2848,18.6454,7.3843,22.5223,8.4013,1.694.4443,3.4553.562,5.194.3531,11.2264-1.3491,13.9241-1.4919,26.715-3.0623,9.2722-1.1384,27.7435-3.5298,38.8483-4.9741,4.6821-.609,8.3507-5.3636,6.8059-9.8253Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m473.1198,129.089c-2.2925,1.4271-5.6203,2.5873-8.0031,3.863l-2.1646,22.8106c2.6869-5.5943,5.7368-11.0112,9.0097-16.3296.3794-3.5043.7672-6.9794,1.1581-10.3441Z" style="fill:#e6e6e6;"/><path d="m471.9617,139.4331c-3.2729,5.3184-6.3228,10.7353-9.0097,16.3296l-3.1925,33.6425c1.1709,2.9472,7.1476,3.2088,7.3345.1839,0,0,2.2295-25.7878,4.8676-50.156Z" style="fill:#fff;"/><path d="m465.48,128.3688c1.9854,1.8304,5.0377,3.5921,7.1401,5.2907l-5.2882,56.4597c-1.7011,2.6765-7.6214,1.8163-7.2397-1.1901,0,0,3.4402-33.6889,5.3878-60.5602Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m506.4369,154.1812c-.02-.1912-1.355-.1923-2.9514-.2012.7851-.1263-3.1505-1.1278-4.4472-1.7135,0,0-.7072,11.1762-1.4505,23.2709,2.312-5.5584,5.2824-10.7134,8.5586-15.6765l.2904-5.6797Z" style="fill:#e6e6e6;"/><path d="m497.5879,175.5374c-.9203,14.9748-1.8961,31.3582-1.6541,29.6865.6188,4.0731,7.6802,3.3311,7.8931.0027l2.3196-45.3657c-3.2762,4.9631-6.2466,10.1181-8.5586,15.6765Z" style="fill:#fff;"/><path d="m495.6549,204.7301l3.5886-50.9859c.043-.1874,1.3683-.0269,2.954.1576-.7641-.2204,3.2639-.7382,4.6219-1.1626,0,0-3.2607,55.9964-3.3292,52.9437-1.1073,3.9683-8.027,2.3769-7.8354-.9528Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m452.0506,114.021c.6154-8.4907,1.6078-22.1868,2.2838-31.5153.3947-5.4457,9.5196-8.0549,7.8785-7.8534,3.2534-.3994,6.514-1.7209,9.7674-2.1203,4.5026-.5529,6.8105-.1174,16.212,3.5947l16.5223,6.7764c3.7249,1.4708,7.0264,3.3798,9.7505,5.2467,5.1611,3.537,8.0213,9.5717,7.5602,15.8116-.8829,11.9478-3.4468,39.7474-3.4468,39.9705,0,7.2677-6.7855,10.3632-10.8387,11.5653,0,0,0,0,.0001,0-1.3112.6662-2.8659.8848-4.4261.4688-10.5057-2.8007-31.4326-12.4034-40.3396-18.7139-7.4599-5.2853-11.5845-14.1124-10.9236-23.231Z" style="fill:#fff;"/><path d="m454.3344,82.5056c.3946-5.4457,3.3338-7.7529,8.5494-6.1377,2.4337.7537,5.2074,1.7048,8.2158,2.8926l5.3951,2.1302c6.1856,2.4423,23.066,8.5072,27.657,10.7423,4.9966,2.4325,8.0006,7.6617,7.5989,13.2044l-3.2761,45.2052c-.2821,3.8931-1.3893,6.4287-5.1608,5.4232-10.5057-2.8007-31.4326-12.4034-40.3396-18.714-7.4599-5.2853-11.5845-14.1124-10.9237-23.231.6154-8.4906,1.6079-22.1868,2.2839-31.5153Z" style="fill:#e6e6e6;"/><path d="m452.0506,114.021c.6154-8.4907,1.6078-22.1868,2.2838-31.5153.3071-4.2373,3.6199-7.3811,7.5321-7.8068,1.1157-.1214,1.8067-.2723,2.5372-.4339,2.5254-.5587,5.0532-1.4232,7.5766-1.733,4.5026-.5529,6.8105-.1174,16.212,3.5947l16.5223,6.7764c3.7249,1.4708,7.0264,3.3798,9.7505,5.2467,5.1611,3.537,8.0213,9.5717,7.5602,15.8116-.8829,11.9478-3.4468,39.7474-3.4468,39.9705,0,7.2677-6.7855,10.3632-10.8387,11.5653,0,0,0,0,.0001,0-1.3112.6662-2.8659.8848-4.4261.4688-10.5057-2.8007-31.4326-12.4034-40.3396-18.7139-7.4599-5.2853-11.5845-14.1124-10.9236-23.231Z" style="fill:none; stroke:#000; stroke-miterlimit:10; stroke-width:3px;"/><path d="m390.7979,322.0233c0-7.7586-37.5259-14.0481-83.8165-14.0481s-83.8165,6.2896-83.8165,14.0481,37.5259,15.6418,83.8165,15.6418,83.8165-7.8832,83.8165-15.6418Z" style="fill:#e6e6e6;"/><path d="m306.9814,307.0203c-21.3225,0-38.6078,3.6838-38.6078,8.2279v3.1438c0,4.5442,17.2853,8.2279,38.6078,8.2279s38.6078-3.6837,38.6078-8.2279v-3.1438c0-4.5441-17.2853-8.2279-38.6078-8.2279Z" style="fill:#fff;"/><path d="m344.4961,313.2154c0-4.1533-16.7893-7.5202-37.4999-7.5202s-37.4999,3.3669-37.4999,7.5202,16.7893,8.5202,37.4999,8.5202,37.4999-4.3669,37.4999-8.5202Z" style="fill:#e6e6e6;"/><path d="m306.9398,307.0613c10.0854,0,19.5612,1.0568,26.6818,2.9757,7.6678,2.0664,9.4342,4.2957,9.4606,4.7366v4.0928c-.0264.4411-1.7928,2.6704-9.4606,4.7368-7.1205,1.9189-16.5963,2.9757-26.6818,2.9757s-19.5612-1.0568-26.6818-2.9757c-7.6678-2.0664-9.4343-4.2957-9.4606-4.7365v-4.0928c.0264-.4411,1.7928-2.6705,9.4606-4.7368,7.1206-1.9189,16.5963-2.9757,26.6818-2.9757m0-3c-21.6177,0-39.1424,4.7961-39.1424,10.7123v4.0931c0,5.9162,17.5247,10.7122,39.1424,10.7122s39.1424-4.796,39.1424-10.7122v-4.0931c0-5.9162-17.5247-10.7123-39.1424-10.7123h0Z"/><path d="m306.9398,169.6468c-59.8092,0-108.2942,10.3329-108.2942,23.0791v8.8183c0,12.7463,48.485,23.0791,108.2942,23.0791s108.2941-10.3328,108.2941-23.0791v-8.8183c0-12.7462-48.4849-23.0791-108.2941-23.0791Z" style="fill:#e6e6e6;"/><path d="m412.1678,190.4881c0-9.7367-47.0936-17.6299-105.1864-17.6299s-105.1864,7.8932-105.1864,17.6299,47.0936,19.6299,105.1864,19.6299,105.1864-9.8932,105.1864-19.6299Z" style="fill:#fff;"/><path d="m306.9398,172.6468c28.7225,0,55.6955,2.3773,75.9502,6.6938,23.1637,4.9365,29.3439,10.7365,29.3439,13.3853v8.8182c0,2.6488-6.1802,8.4488-29.3439,13.3853-20.2546,4.3166-47.2276,6.6938-75.9502,6.6938s-55.6956-2.3773-75.9502-6.6938c-23.1637-4.9365-29.344-10.7365-29.344-13.3853v-8.8182c0-2.6488,6.1803-8.4488,29.344-13.3853,20.2547-4.3166,47.2277-6.6938,75.9502-6.6938m0-3c-59.8092,0-108.2942,10.3329-108.2942,23.0792v8.8182c0,12.7463,48.485,23.0792,108.2942,23.0792s108.2941-10.3329,108.2941-23.0792v-8.8182c0-12.7462-48.4849-23.0792-108.2941-23.0792h0Z"/><path d="m313.5989,245.7856v-22.6949c-2.1022.053-4.2276.0815-6.3756.0815-1.7147,0-3.4146-.0188-5.1005-.0527v23.0148c3.7704.17,7.659-.0504,11.4761-.3487Z" style="fill:#e6e6e6;"/><path d="m303.4594,246.1343v65.4282c3.0175,1.7398,6.3169,1.8293,8.7907,0v-65.7769c-2.9239.2983-5.9026.5187-8.7907.3487Z" style="fill:#fff;"/><path d="m313.5989,305.6952v-82.6045c-2.1022.0529-4.2276.0815-6.3756.0815-1.7147,0-3.4145-.0188-5.1005-.0527v82.5757" style="fill:none; stroke:#000; stroke-linecap:round; stroke-miterlimit:10; stroke-width:3px;"/></svg>
```

## File: static\img\takeAway.svg

```svg
<svg viewBox="0 0 610 410" xmlns="http://www.w3.org/2000/svg"><path d="m357.8337 298.4274c0 4.7842 23.1398 9.6453 51.6842 9.6453s51.6842-4.8611 51.6842-9.6453l-116.5459-81.9214s-16.3593-9.9918-55.5944 1.3767c-16.9279 4.9049-32.8283 14.905-21.81 26.164 24.2404 24.7698 90.5818 54.3808 90.5818 54.3808z" fill="#e6e6e6"/><path d="m349.3248 143.792 15.6315 144.4477v4.6571c0 4.032 7.1584 7.6052 18.1821 9.8239l-2.8192-158.9287z" fill="#e6e6e6"/><path d="m380.3192 143.792 2.8192 158.9287c7.3802 1.4853 16.4904 2.3644 26.3539 2.3644 24.5966 0 44.536-5.4569 44.536-12.1883v-4.6571l15.6315-144.4477h-89.3405z" fill="#fff"/><path d="m364.9562 288.2397v4.6571c0 6.7314 19.9395 12.1883 44.536 12.1883s44.536-5.4569 44.536-12.1883v-4.6571l15.6314-144.4477h-120.335l15.6315 144.4477z" fill="none" stroke="#000" stroke-miterlimit="10" stroke-width="3"/><path d="m409.5179 119.9224c-36.8805 0-66.8037 3.5107-66.8037 17.7421v5.4377c0 7.8598 29.8976 14.2314 66.778 14.2314s66.778-6.3716 66.778-14.2314v-5.4377c0-16.1488-29.8719-17.7421-66.7524-17.7421z" fill="#e6e6e6"/><path d="m474.3796 130.7936c0-6.004-29.0396-10.8712-64.8617-10.8712s-64.8617 4.8672-64.8617 10.8712 29.0396 13.1045 64.8617 13.1045 64.8617-7.1005 64.8617-13.1045z" fill="#fff"/><path d="m409.4923 117.9421c-36.8805 0-66.778 6.3716-66.778 14.2314v11.604c0 7.8598 29.8976 14.2314 66.778 14.2314s66.778-6.3716 66.778-14.2314v-11.604c0-7.8598-29.8975-14.2314-66.778-14.2314z" fill="none" stroke="#000" stroke-miterlimit="10" stroke-width="3"/><path d="m367.0678 130.2501c0 3.9294 19.0055 7.922 42.4501 7.922s42.4501-3.9926 42.4501-7.922v-7.0628h-84.9001v7.0628z" fill="#e6e6e6"/><path d="m451.968 123.0295c0-3.9294-19.0055-7.1149-42.4501-7.1149s-42.4501 3.1854-42.4501 7.1149 19.0055 7.922 42.4501 7.922 42.4501-3.9926 42.4501-7.922z" fill="#fff"/><path d="m452.5719 121.3024c-5.7138-4.5513-23.6029-6.5048-42.5118-6.5048-18.8801 0-37.7135 1.8605-43.8286 6.5348-16.0477 2.5992-23.5173 6.2141-23.5173 10.8411v11.604c0 7.8598 29.8975 14.2314 66.778 14.2314s66.778-6.3716 66.778-14.2314v-11.604c0-4.6194-7.6954-8.2711-23.6983-10.871z" fill="none" stroke="#000" stroke-miterlimit="10" stroke-width="3"/><path d="m414.5693 64.65v57.6991c-3.0061 1.8143-6.0121 1.8143-9.0182 0v-57.8656c0-.65 2.0188-1.7936 4.5091-1.7936s4.5091 1.1436 4.5091 1.7936z" fill="#fff"/><ellipse cx="410.0602" cy="64.3201" fill="#e6e6e6" rx="3.859" ry="1.6302"/><g fill="none" stroke="#000" stroke-miterlimit="10" stroke-width="3"><path d="m414.5693 64.65v57.0175c-3.1931 1.816-6.1882 1.8159-9.0182 0v-57.184c0-.65 2.0188-1.7936 4.5091-1.7936s4.5091 1.1436 4.5091 1.7936z" stroke-linecap="round"/><path d="m355.6709 202.4354s50.4776 17.1743 107.3811 1.6836"/><path d="m361.3371 259.5429s41.759 19.3722 96.367 1.1688"/></g><path d="m369.9382 313.01-213.558-61.0295-82.6504 20.2727 182.6631 67.8408z" fill="#e6e6e6"/><path d="m224.6539 158.3008-19.5266 2.1876 15.1908-40.8899 13.6536 27.6924z" fill="#e6e6e6"/><path d="m224.6539 158.3008-19.5266 2.1876 15.1908-40.8899 13.6536 27.6924z" fill="none" stroke="#000" stroke-linejoin="round" stroke-width="3"/><path d="m233.9717 147.2909-35.0927 41.4657-4.6791 124.1506c35.4459 18.148 62.1927 27.1866 62.1927 27.1866s.0002.0002.0002.0001c.0006-.0151 3.1796-78.8155 3.1127-142.4374l-25.5339-50.3657z" fill="#fff"/><g fill="#e6e6e6"><path d="m250.6709 309.6037-56.471 3.3036 62.193 27.1867z"/><path d="m243.2592 309.6237 9.9652-124.3568 6.2812 12.3897-3.1127 142.4374z"/><path d="m253.2244 185.2669c-13.4693-26.2011-19.2527-37.976-19.2527-37.976l-24.7357 29.2279c16.1213-.3636 27.9022 10.0608 43.9885 8.7481z"/></g><path d="m233.9717 147.2909-35.0927 41.4657-4.6791 124.1506c35.4459 18.148 62.1927 27.1866 62.1927 27.1866s.0002.0002.0002.0001c.0006-.0151 3.1796-78.8155 3.1127-142.4374l-25.5339-50.3657z" fill="none" stroke="#000" stroke-linecap="round" stroke-linejoin="round" stroke-width="3"/><path d="m256.3929 340.094s3.1796-78.8095 3.1127-142.4374l-39.1875-78.0581 95.431-29.5582 45.2378 73.4564c7.5173 39.1821 7.469 128.3103 8.9513 149.5132.3686 5.2724-113.5454 27.084-113.5454 27.084" fill="#fff"/><path d="m256.393 341.5938c-.356 0-.7041-.127-.979-.3633-.3467-.2988-.5381-.7393-.52-1.1973.0317-.7861 3.166-79.2119 3.1123-142.0205l-39.0283-77.7412c-.1982-.394-.2124-.855-.0396-1.2607.1724-.4053.5151-.7144.936-.8447l95.4307-29.5581c.6538-.2061 1.3618.063 1.7212.646l45.2378 73.4561c.0952.1548.1616.3257.1958.5039 5.4858 28.5933 6.9507 83.333 7.9199 119.5557.3643 13.6172.6519 24.373 1.0552 30.1357.1431 2.0459.3208 4.5928-57.8159 17.0898-28.1235 6.0459-56.6592 11.5176-56.9438 11.5723-.0938.0176-.1885.0264-.2822.0264zm-33.9399-221.0859 38.3931 76.4756c.105.208.1592.4385.1597.6709.0605 57.4961-2.5625 128.2217-3.04 140.6104 41.0029-7.8877 105.0054-21.3721 110.4443-25.6191-.3906-5.9258-.6733-16.501-1.0293-29.7959-.9634-36.0107-2.4185-90.3726-7.8159-118.8013l-44.4834-72.231-92.6284 28.6904z"/><path d="m339.2314 192.4307c-.1713.0395-.3412.0899-.5094.1512-5.8373 2.1491-10.7151 7.089-13.7356 13.9123l-1.7841 4.0138c-2.389 5.3971-1.4914 12.6515 2.1353 17.2678l.6722.8581-2.012 4.5454c-7.7146 17.3954-15.9007 33.5547-24.3493 48.0335-.8465 1.4575-1.0157 3.4587-.4466 5.2342l.0763.2311c.7442 2.3246 2.5496 3.7686 4.395 3.5268.0895-.0123.1784-.0287.2661-.049 1.265-.2921 2.3389-1.3984 2.8319-2.9509 5.4082-17.1485 12.0444-34.6357 19.7238-51.9586l2.0186-4.547.8682.4168c1.8091.8646 3.6858 1.0526 5.4453.6464 2.7909-.6443 5.2872-2.7833 6.7534-6.0881l1.7841-4.0138c3.0205-6.8233 3.9594-15.0177 2.6502-23.0615-.3425-2.0857-1.3325-3.9391-2.725-5.0823-1.2391-1.024-2.6886-1.4027-4.0583-1.0865" fill="#fff"/><path d="m303.9393 290.1561c-.0878.0203-.1767.0367-.2661.049-1.8454.2418-3.6507-1.2022-4.395-3.5268l-.0763-.2311c-.5691-1.7755-.4-3.7767.4466-5.2342 8.4486-14.4788 16.6347-30.6381 24.3493-48.0335l2.012-4.5454-.6722-.8581c-3.6267-4.6162-4.5242-11.8707-2.1353-17.2678l1.7841-4.0138c3.0205-6.8233 7.8983-11.7632 13.7356-13.9123.1682-.0614.3381-.1117.5094-.1512 1.3697-.3162 2.8192.0625 4.0583 1.0865 1.3925 1.1432 2.3825 2.9966 2.725 5.0823 1.3091 8.0439.3703 16.2382-2.6502 23.0615l-1.7841 4.0138c-1.4662 3.3048-3.9625 5.4438-6.7534 6.0881-1.7594.4062-3.6362.2182-5.4453-.6464l-.8682-.4168-2.0186 4.547c-7.6794 17.3228-14.3156 34.8101-19.7238 51.9586-.493 1.5524-1.5669 2.6588-2.8319 2.9509m34.768-100.679c-.2448.0565-.4877.1291-.7277.2178-6.3606 2.3436-11.6756 7.7253-14.9775 15.1663l-1.7775 4.0123c-2.7509 6.2175-1.9139 14.4717 1.9363 20.0957l-1.1409 2.5756c-7.671 17.2932-15.8119 33.3592-24.2071 47.752-1.3157 2.2567-1.5776 5.3848-.6907 8.1555l.0696.2327c1.1643 3.6187 3.9791 5.8798 6.8531 5.4926.1488-.0159.296-.0407.435-.0728 1.9723-.4553 3.6317-2.1834 4.3949-4.5981 5.3728-17.039 11.9686-34.4064 19.6044-51.627l1.1393-2.5845c1.9222.7252 3.883.8406 5.737.4126 3.3915-.783 6.4253-3.3848 8.2092-7.4l1.7775-4.0122c3.2953-7.4395 4.3228-16.3636 2.8876-25.1429-.473-2.9216-1.8747-5.5274-3.8293-7.1488-1.7378-1.4287-3.7758-1.9693-5.6932-1.5267"/><path d="m291.6664 203.9537c-.0179.0041-.0358.0087-.0536.0137-.3791.1152-.6817.4245-.8628.8717-.3473.8632-.0685 2.024.6285 2.6001l12.3742 10.215c2.0583 1.6988 2.6875 3.9211 1.878 6.6136-.2647.8625-.8089 1.5133-1.4979 1.7829-.0881.0343-.1775.0619-.2676.0827-.6216.1435-1.2815-.0325-1.8414-.4986l-13.7026-11.3085c-.3499-.2854-.7488-.3775-1.1091-.2944-.3559.0821-.6743.3353-.8714.7424-.2058.4252-.2569.9436-.1592 1.4462.0993.5114.3429.9434.6832 1.2241l13.7535 11.3613c.6461.5233 1.0986 1.3861 1.2513 2.3457.1462.9612-.0249 1.9219-.4671 2.6412-.6373 1.0438-1.3581 1.659-2.1558 1.8432-.9069.2094-1.9131-.1384-3.009-1.0469l-12.3726-10.2062c-.3496-.2889-.747-.3814-1.1063-.2984-.3571.0824-.6765.3381-.8742.7465-.2058.4252-.2651.9363-.1592 1.4462.0993.5114.3429.9434.6832 1.2241l12.4302 10.2574c2.3683 1.9556 5.1119 2.6031 7.6426 2.0188 2.0156-.4653 3.896-1.7113 5.3461-3.7002l.6081-.8313 3.5904 2.9664c19.6983 16.253 34.0076 30.7655 39.2465 36.2531.9691 1.0121 2.2043 1.4317 3.3556 1.1659.2218-.0512.4406-.1279.6537-.2306l.1707-.0855c1.7118-.8282 2.6973-3.1192 2.3917-5.5727-.2366-1.843-1.1882-3.4842-2.4919-4.2886-6.069-3.7484-22.303-14.2238-41.9273-30.411l-3.5904-2.9664.2596-1.1193c1.3982-6.0343-.6811-12.9607-4.9423-16.4723l-12.4236-10.2589c-.3304-.2659-.7038-.3547-1.0617-.2721" fill="#fff"/><path d="m303.8653 226.1333c.0901-.0208.1795-.0483.2676-.0827.689-.2696 1.2332-.9203 1.4979-1.7829.8095-2.6925.1803-4.9148-1.878-6.6136l-12.3742-10.215c-.6971-.576-.9758-1.7369-.6285-2.6001.1812-.4471.4838-.7565.8628-.8717.0178-.005.0357-.0095.0536-.0137.3579-.0826.7313.0062 1.0617.2721l12.4236 10.2589c4.2612 3.5117 6.3405 10.4381 4.9423 16.4723l-.2596 1.1193 3.5904 2.9664c19.6243 16.1872 35.8583 26.6626 41.9273 30.411 1.3037.8045 2.2553 2.4456 2.4919 4.2886.3056 2.4535-.6799 4.7445-2.3917 5.5727l-.1707.0855c-.2131.1028-.4319.1794-.6537.2306-1.1513.2658-2.3865-.1538-3.3556-1.1659-5.2389-5.4876-19.5482-20.0001-39.2465-36.2531l-3.5904-2.9664-.6081.8313c-1.4501 1.9889-3.3305 3.2349-5.3461 3.7002-2.5306.5842-5.2743-.0633-7.6426-2.0188l-12.4302-10.2574c-.3403-.2807-.5839-.7127-.6832-1.2241-.1059-.5098-.0466-1.021.1592-1.4462.1977-.4084.5171-.6641.8742-.7465.3593-.0829.7567.0096 1.1063.2984l12.3726 10.2062c1.0959.9085 2.1021 1.2563 3.009 1.0469.7977-.1842 1.5185-.7994 2.1558-1.8432.4422-.7193.6133-1.68.4671-2.6412-.1528-.9596-.6052-1.8224-1.2513-2.3457l-13.7535-11.3613c-.3403-.2807-.5839-.7127-.6832-1.2241-.0977-.5025-.0466-1.021.1592-1.4462.197-.4071.5154-.6602.8714-.7424.3604-.0832.7592.009 1.1091.2944l13.7026 11.3085c.5599.4661 1.2199.6421 1.8414.4986m-12.7299-25.1292c-.0603.0139-.1206.0298-.1805.0476-.9651.3057-1.7597 1.0972-2.2211 2.2446-.9209 2.2669-.248 5.1884 1.5325 6.6473l12.3742 10.215c1.1919.9869 1.2358 1.6492.8716 2.8387-.0354.1095-.0968.1513-.1481.1724-.0104.0045-.023.009-.0374.0124-.0478.011-.1159.0087-.1904-.0519l-13.7026-11.3086c-.8748-.724-1.8553-.9584-2.7498-.7518-.921.2126-1.7509.8926-2.2805 1.9869-.5255 1.0978-.6723 2.4214-.4209 3.7359.258 1.313.8889 2.4386 1.7717 3.1652l13.7585 11.3509c.0904.0805.1244.1924.1356.2543.0062.0723.0105.1726-.0429.2586-.2982.486-.5565.7946-.8815.8696-.3424.0791-.7589-.1013-1.3738-.6069l-12.3726-10.2062c-.8759-.7249-1.8582-.9611-2.7538-.7543-.9198.2124-1.7483.892-2.2765 1.9894-.5271 1.089-.6806 2.4141-.4225 3.7271.258 1.313.8889 2.4386 1.7733 3.174l12.4286 10.2485c2.8794 2.3804 6.2146 3.1692 9.2907 2.459 2.1628-.4993 4.1976-1.74 5.8547-3.6895l2.0418 1.6842c19.593 16.1667 33.817 30.5884 39.0214 36.0379 1.5137 1.5851 3.4379 2.237 5.2249 1.8245.3508-.081.6869-.2046 1.0233-.3652l.1707-.0855c2.6763-1.2903 4.2098-4.8594 3.7242-8.69-.3592-2.8649-1.8423-5.4243-3.8751-6.6776-6.0311-3.7295-22.1615-14.1275-41.6657-30.2226l-2.0352-1.6857c1.1945-7.0466-1.329-14.8285-6.238-18.8723l-12.4302-10.2574c-.83-.6902-1.7791-.931-2.6985-.7187"/></svg>
```

## File: static\src\upgrade_selection_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { selectionField, SelectionField } from "@web/views/fields/selection/selection_field";
import { useService } from "@web/core/utils/hooks";
import { UpgradeDialog } from "@web/webclient/settings_form_view/fields/upgrade_dialog";

/**
 *  The upgrade selection field is intended to be used in config settings.
 *  When selection changed, an upgrade popup is showed to the user.
 */

export class UpgradeSelectionField extends SelectionField {
    setup() {
        super.setup();
        this.dialogService = useService("dialog");
        this.isEnterprise = odoo.info && odoo.info.isEnterprise;
    }

    async onChange(newValue) {
        if (!this.isEnterprise) {
            this.dialogService.add(
                UpgradeDialog,
                {},
                {
                    onClose: () => {
                        newValue.target.value = '"meal"';
                    },
                }
            );
        } else {
            super.onChange(...arguments);
        }
    }
}

export const upgradeSelectionField = {
    ...selectionField,
    component: UpgradeSelectionField,
    additionalClasses: [...(selectionField.additionalClasses || []), "o_field_selection"],
};

registry.category("fields").add("upgrade_selection", upgradeSelectionField);

```

## File: static\src\app\router.js

```javascript
/** @odoo-module */

import { Component, onWillRender, useState, xml } from "@odoo/owl";
import { escapeRegExp } from "@web/core/utils/strings";
import { zip } from "@web/core/utils/arrays";
import { useService } from "@web/core/utils/hooks";

function parseParams(matches, paramSpecs) {
    return Object.fromEntries(
        zip(matches, paramSpecs).map(([match, paramSpec]) => {
            const { type, name } = paramSpec;
            switch (type) {
                case "int":
                    return [name, parseInt(match)];
                case "string":
                    return [name, match];
                default:
                    throw new Error(`Unknown type ${type}`);
            }
        })
    );
}

export class Router extends Component {
    static props = { slots: Object, pos_config_id: Number };
    static template = xml`<t t-slot="{{activeSlot}}" t-props="slotProps"/>`;

    setup() {
        this.router = useState(useService("router"));
        this.activeSlot = "default";
        this.slotProps = {};
        this.routes = {};

        for (const [routeName, slot] of Object.entries(this.props.slots)) {
            const route = slot.route;
            const paramStrings = route.match(/\{\w+:\w+\}/g);

            if (!paramStrings) {
                this.routes[routeName] = { route, paramSpecs: [], regex: new RegExp(`^${route}$`) };
                continue;
            }

            const paramSpecs = paramStrings.map((paramString) => {
                const [, type, name] = paramString.match(/(\w+):(\w+)/);
                return { type, name };
            });

            const regex = new RegExp(
                `^${route
                    .split(/\{\w+:\w+\}/)
                    .map((part) => escapeRegExp(part))
                    .join("([^/]+)")}$`
            );

            this.routes[routeName] = { route, paramSpecs, regex };
        }

        this.router.registerRoutes(this.routes);

        onWillRender(() => {
            this.matchURL();
        });
    }

    matchURL() {
        const path = this.router.path;

        for (const [routeName, { paramSpecs, regex }] of Object.entries(this.routes)) {
            const match = path.match(regex);
            if (match) {
                const parsedParams = parseParams(match.slice(1), paramSpecs);
                this.router.activeSlot = routeName;
                this.activeSlot = routeName;
                this.slotProps = parsedParams;
                return;
            }
        }

        this.router.activeSlot = "default";
        this.router.navigate("default");
    }
}

```

## File: static\src\app\self_order_bus_service.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { effect } from "@web/core/utils/reactive";
import { batched } from "@web/core/utils/timing";

export class SelfOrderBus {
    static serviceDependencies = ["self_order", "orm", "bus_service"];

    constructor(...args) {
        this.setup(...args);
    }

    setup(env, { self_order, orm, bus_service }) {
        this.selfOrder = self_order;
        this.orm = orm;
        this.bus = bus_service;

        effect(
            batched((orders) => {
                for (const order of orders) {
                    if (order.access_token) {
                        this.bus.addChannel(`self_order-${order.access_token}`);
                    }
                }
            }),
            [this.selfOrder.orders]
        );

        bus_service.addChannel(`pos_config-${this.selfOrder.access_token}`);
        bus_service.addEventListener("notification", ({ detail }) => {
            for (const message of detail) {
                this.dispatch(message);
            }
        });
    }

    dispatch(message) {
        const mode = this.selfOrder.config.self_ordering_mode;
        if (message.type === "ORDER_STATE_CHANGED") {
            this.ws_changeOrderState(message.payload.access_token, message.payload.state);
        } else if (message.type === "ORDER_CHANGED") {
            this.ws_syncOrder(message.payload.order);
        } else if (message.type === "STATUS" && mode === "kiosk") {
            this.ws_status(message);
        } else if (message.type === "PAYMENT_STATUS" && mode === "kiosk") {
            this.ws_paymentStatus(message);
        } else if (message.type === "PRODUCT_CHANGED") {
            this.ws_productChanged(message);
        }
    }

    ws_changeOrderState(access_token, state) {
        this.selfOrder.changeOrderState(access_token, state);
    }

    ws_syncOrder(order, state) {
        this.selfOrder.updateOrdersFromServer([order], [order.access_token]);
    }

    ws_paymentStatus(message) {
        const payload = message.payload;

        if (payload.payment_result === "Success") {
            this.selfOrder.updateOrderFromServer(payload.order);
            this.selfOrder.router.navigate("confirmation", {
                orderAccessToken: payload.order.access_token,
                screenMode: "order",
            });
        } else {
            this.selfOrder.paymentError = true;
        }
    }

    ws_status(message) {
        const payload = message.payload;

        if (payload.status === "closed") {
            this.selfOrder.pos_session = null;
            this.selfOrder.ordering = false;
        } else {
            // reload to get potential new settings
            // more easier than RPC for now
            window.location.reload();
        }
    }

    ws_productChanged(message) {
        this.selfOrder.handleProductChanges(message.payload);
    }
}

export const SelfOrderBusService = {
    dependencies: SelfOrderBus.serviceDependencies,
    async start(env, deps) {
        return new SelfOrderBus(env, deps);
    },
};

registry.category("services").add("self_order_bus_service", SelfOrderBusService);

```

## File: static\src\app\self_order_index.js

```javascript
/** @odoo-module */
import { Component, whenReady, App } from "@odoo/owl";
import { makeEnv, startServices } from "@web/env";
import { templates } from "@web/core/assets";
import { _t } from "@web/core/l10n/translation";
import { MainComponentsContainer } from "@web/core/main_components_container";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { Router } from "@pos_self_order/app/router";
import { LandingPage } from "@pos_self_order/app/pages/landing_page/landing_page";
import { ProductListPage } from "@pos_self_order/app/pages/product_list_page/product_list_page";
import { ComboPage } from "@pos_self_order/app/pages/combo_page/combo_page";
import { ProductPage } from "@pos_self_order/app/pages/product_page/product_page";
import { CartPage } from "@pos_self_order/app/pages/cart_page/cart_page";
import { PaymentPage } from "@pos_self_order/app/pages/payment_page/payment_page";
import { ConfirmationPage } from "@pos_self_order/app/pages/confirmation_page/confirmation_page";
import { EatingLocationPage } from "@pos_self_order/app/pages/eating_location_page/eating_location_page";
import { StandNumberPage } from "@pos_self_order/app/pages/stand_number_page/stand_number_page";
import { OrdersHistoryPage } from "@pos_self_order/app/pages/order_history_page/order_history_page";
import { LoadingOverlay } from "@pos_self_order/app/components/loading_overlay/loading_overlay";

class selfOrderIndex extends Component {
    static template = "pos_self_order.selfOrderIndex";
    static props = [];
    static components = {
        Router,
        CartPage,
        ProductPage,
        OrdersHistoryPage,
        ComboPage,
        PaymentPage,
        ConfirmationPage,
        ProductListPage,
        EatingLocationPage,
        StandNumberPage,
        LandingPage,
        LoadingOverlay,
        MainComponentsContainer,
    };

    setup() {
        this.selfOrder = useSelfOrder();
    }
    get selfIsReady() {
        return Object.values(this.selfOrder.productByIds).length > 0;
    }
}

export async function createPublicRoot() {
    await whenReady();
    const env = makeEnv();
    await startServices(env);
    const app = new App(selfOrderIndex, {
        templates,
        env: env,
        dev: env.debug,
        translateFn: _t,
        translatableAttributes: ["data-tooltip"],
    });
    return app.mount(document.body);
}

createPublicRoot();
export default { selfOrderIndex, createPublicRoot };

```

## File: static\src\app\self_order_index.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.selfOrderIndex">
        <div t-if="!selfOrder.pos_session" class="o-self-closed w-100 m-0 text-center bg-black text-white">
            <p>We're currently closed.</p>
        </div>
        <div t-if="selfOrder.rpcLoading">
            <LoadingOverlay />
        </div>
        <Router t-if="selfIsReady" pos_config_id="selfOrder.pos_config_id">
            <t t-set-slot="default" route="`/pos-self/${selfOrder.pos_config_id}`">
                <LandingPage />
            </t>
            <t t-set-slot="product_list" route="`/pos-self/${selfOrder.pos_config_id}/products`">
                <ProductListPage />
            </t>
            <t t-set-slot="product" route="`/pos-self/${selfOrder.pos_config_id}/product/{int:id}`" t-slot-scope="url">
                <ProductPage product="selfOrder.productByIds[url.id]" />
            </t>
            <t t-set-slot="combo_selection" route="`/pos-self/${selfOrder.pos_config_id}/combo-selection/{int:id}`" t-slot-scope="url">
                <ComboPage product="selfOrder.productByIds[url.id]" />
            </t>
            <t t-set-slot="cart" route="`/pos-self/${selfOrder.pos_config_id}/cart`">
                <CartPage />
            </t>
            <t t-set-slot="payment" route="`/pos-self/${selfOrder.pos_config_id}/payment`">
                <PaymentPage />
            </t>
            <t t-set-slot="confirmation" route="`/pos-self/${selfOrder.pos_config_id}/confirmation/{string:orderAccessToken}/{string:screenMode}`" t-slot-scope="url">
                <ConfirmationPage orderAccessToken="url.orderAccessToken" screenMode="url.screenMode" />
            </t>
            <t t-set-slot="location" route="`/pos-self/${selfOrder.pos_config_id}/location`">
                <EatingLocationPage />
            </t>
            <t t-set-slot="stand_number" route="`/pos-self/${selfOrder.pos_config_id}/stand_number`">
                <StandNumberPage />
            </t>
            <t t-set-slot="orderHistory" route="`/pos-self/${selfOrder.pos_config_id}/orders`">
                <OrdersHistoryPage />
            </t>
        </Router>
        <div t-else="" class="h-100 w-100 d-flex align-items-center justify-content-center text-center">
            Hey, looks like you forgot to create products or add them to pos_config. Please add them before using the Self Order
        </div>
        <MainComponentsContainer />
    </t>
</templates>

```

## File: static\src\app\self_order_router_service.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { Reactive } from "@web/core/utils/reactive";
import { browser } from "@web/core/browser/browser";

export class SelfOrderRouter extends Reactive {
    static serviceDependencies = [];

    constructor(...args) {
        super(...args);
        this.setup(...args);
    }

    setup(env) {
        this.path = window.location.pathname;
        this.registeredRoutes = {};
        this.historyPage = "";
        this.activeSlot = null;
        window.addEventListener("popstate", (event) => {
            this.path = window.location.pathname;
        });
    }

    addTableIdentifier(table) {
        const url = new URL(browser.location.href);
        url.searchParams.set("table_identifier", table.identifier);
        history.replaceState({}, "", url);
    }

    back() {
        if (!this.historyPage.length) {
            // We use the browser history, so if the user arrives on a page with a back button from a link,
            // we don't know the previous page, so we send them back to the beginning of the feed.
            this.navigate("default");
            return;
        }

        history.back();
        this.path = window.location.pathname;
        this.historyPage = window.location.pathname;
    }

    /**
     * Navigate to the given relative route.
     * We use the history API to navigate to it.
     * (this means that we don't make additional requests to the server)
     * @param {string} route
     */
    navigate(routeName, routeParams = {}) {
        const { route } = this.registeredRoutes[routeName];
        const url = new URL(browser.location.href);

        url.pathname = route.replace(/\{\w+:(\w+)\}/g, (match, paramName) => {
            return routeParams[paramName];
        });

        history.pushState({}, "", url);
        this.path = window.location.pathname;
        this.historyPage = this.path;
    }

    registerRoutes(routes) {
        Object.assign(this.registeredRoutes, routes);
    }

    // If the url isn't a valid URL, we assume it's a relative path
    customLink(link) {
        let url = "";

        try {
            url = new URL(link.url);
            window.open(url);
        } catch {
            url = new URL(browser.location.href);
            url.pathname = link.url;

            history.pushState({}, "", url);
            this.path = window.location.pathname;
            this.historyPage = this.path;
        }
    }
}

export const SelfOrderRouterService = {
    dependencies: SelfOrderRouter.serviceDependencies,
    async start(env, deps) {
        return new SelfOrderRouter(env, deps);
    },
};

registry.category("services").add("router", SelfOrderRouterService);

```

## File: static\src\app\self_order_service.js

```javascript
/** @odoo-module */
import { Reactive, effect } from "@web/core/utils/reactive";
import { ConnectionLostError, RPCError } from "@web/core/network/rpc_service";
import { _t } from "@web/core/l10n/translation";
import { formatMonetary } from "@web/views/fields/formatters";
import { Product } from "@pos_self_order/app/models/product";
import { Combo } from "@pos_self_order/app/models/combo";
import { session } from "@web/session";
import { categorySorter, attributeFormatter, attributeFlatter } from "@pos_self_order/app/utils";
import { Order } from "@pos_self_order/app/models/order";
import { batched } from "@web/core/utils/timing";
import { useState, markup } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";
import { cookie } from "@web/core/browser/cookie";
import { formatDateTime } from "@web/core/l10n/dates";
import { printerService } from "@point_of_sale/app/printer/printer_service";
import { constructFullProductName, qrCodeSrc, deduceUrl } from "@point_of_sale/utils";
import { Line } from "./models/line";
import { HWPrinter } from "@point_of_sale/app/printer/hw_printer";
import { renderToElement } from "@web/core/utils/render";
export class SelfOrder extends Reactive {
    constructor(...args) {
        super();
        this.ready = this.setup(...args).then(() => this);
    }

    async setup(env, { rpc, notification, router, printer, barcode }) {
        // services
        this.notification = notification;
        this.router = router;
        this.env = env;
        this.rpc = rpc;
        this.printer = printer;
        this.barcode = barcode;

        // data
        Object.assign(this, {
            ...session.pos_self_order_data,
        });

        this.markupDescriptions();

        this.productsGroupedByCategory = {};
        this.lastEditedProductId = null;
        this.attributeValueById = {};
        this.currentProduct = 0;
        this.attributeById = {};
        this.priceLoading = false;
        this.rpcLoading = false;
        this.productByIds = {};
        this.paymentError = false;
        this.editedOrder = null;
        this.comboByIds = {};
        this.ordering = false;
        this.orders = [];
        this.kitchenPrinters = [];

        this.initData();
        if (this.config.self_ordering_mode === "kiosk") {
            this.initKioskData();
        } else {
            await this.initMobileData();
        }
        barcode.bus.addEventListener("barcode_scanned", (ev) => {
            if (!this.ordering) {
                this.notification.add(_t("We're currently closed"), {
                    type: "danger",
                });
                return;
            }
            const product = Object.values(this.productByIds).filter(
                (p) => p.barcode === ev.detail.barcode
            )?.[0];
            if (!product) {
                this.notification.add(_t("Product not found"), {
                    type: "danger",
                });
                return;
            }
            if (!product.self_order_available) {
                this.notification.add(_t("Product is not available"), {
                    type: "danger",
                });
                return;
            }
            if (product.attributes.length) {
                this.router.navigate("product", { id: product.id });
                return;
            }
            this.addToCart(product, 1, "", {}, {});
            this.router.navigate("cart");
        });
    }

    async addToCart(product, qty, customer_note, selectedValues, customValues) {
        const lineToMerge =
            !product.pos_combo_ids.length &&
            (this.editedLine ||
                this.currentOrder.lines.find(
                    (l) =>
                        JSON.stringify(l.attribute_value_ids.sort()) ===
                            JSON.stringify(attributeFlatter(selectedValues).sort()) &&
                        l.customer_note === customer_note &&
                        l.product_id === product.id
                ));
        if (lineToMerge) {
            lineToMerge.attribute_value_ids = attributeFlatter(selectedValues);
            lineToMerge.customer_note = customer_note;
            lineToMerge.full_product_name = product.name;
            if (this.editedLine) {
                lineToMerge.qty = qty;
            } else {
                lineToMerge.qty += qty;
            }
        } else {
            const mainLine = new Line({
                qty,
                product_id: product.id,
                customer_note,
                custom_attribute_value_ids: Object.values(customValues),
                attribute_value_ids: attributeFlatter(selectedValues),
            });
            mainLine.full_product_name = constructFullProductName(
                mainLine,
                this.attributeValueById,
                product.name
            );
            this.currentOrder.lines.push(mainLine);
        }
        await this.getPricesFromServer();
        // If a command line does not have a quantity greater than 0, we consider it deleted
        this.currentOrder.lines = this.currentOrder.lines.filter((o) => o.qty > 0);
    }
    async confirmationPage(screen_mode, device, access_token = "") {
        this.router.navigate("confirmation", {
            orderAccessToken: access_token || this.currentOrder.access_token,
            screenMode: screen_mode,
        });
        if (device === "kiosk") {
            this.printKioskChanges();
        }
    }

    async confirmOrder() {
        const payAfter = this.config.self_ordering_pay_after; // each, meal
        const device = this.config.self_ordering_mode; // kiosk, mobile
        const service = this.config.self_ordering_service_mode; // table, counter
        const paymentMethods = this.pos_payment_methods; // Stripe, Adyen, Online
        const order = this.currentOrder;

        // Stand number page will recall this function after the stand number is set
        if (
            service === "table" &&
            !order.take_away &&
            device === "kiosk" &&
            !order.table_stand_number
        ) {
            this.router.navigate("stand_number");
            return;
        }

        // if the amount is 0, we don't need to go to the payment page
        // this directive works for both mode each and meal
        if (order.amount_total === 0 && order.lines.length > 0) {
            const order = await this.sendDraftOrderToServer();
            this.confirmationPage("order", device, order.access_token);
            return;
        }

        // When no payment methods redirect to confirmation page
        // the client will be able to pay at counter
        if (paymentMethods.length === 0) {
            let screenMode = "pay";

            if (!order.isSavedOnServer) {
                await this.sendDraftOrderToServer();
                screenMode = payAfter === "meal" ? "order" : "pay";
            }
            this.confirmationPage(screenMode, device);
        } else {
            // In meal mode, first time the customer validate his order, we send it to the server
            // and we redirect him to the confirmation page, the next time he validate his order
            // if the order is already saved on the server, we redirect him to the payment page
            // In each mode, we redirect the customer to the payment page directly
            if (payAfter === "meal" && !order.isSavedOnServer) {
                await this.sendDraftOrderToServer();
                this.confirmationPage("order", device);
            } else {
                this.router.navigate("payment");
            }
        }
    }

    get currentOrder() {
        if (this.editedOrder && this.editedOrder.state === "draft") {
            return this.editedOrder;
        }
        const existingOrder = this.orders.find(
            (o) =>
                o.state === "draft" ||
                (o.state === "paid" &&
                    o.amount_total === 0 &&
                    this.config.self_ordering_mode === "kiosk")
        );
        if (!existingOrder) {
            const newOrder = new Order({
                pos_config_id: this.pos_config_id,
            });

            this.orders.push(newOrder);
            this.editedOrder = newOrder;
        } else {
            this.editedOrder = existingOrder;
        }

        return this.editedOrder;
    }

    markupDescriptions() {
        for (const product of this.products) {
            product.description_self_order = product.description_self_order
                ? markup(product.description_self_order)
                : "";
        }
    }

    initData() {
        this.currentLanguage = this.config.self_ordering_available_language_ids.find(
            (l) => l.code === cookie.get("frontend_lang")
        );

        if (this.config.self_ordering_default_language_id && !this.currentLanguage) {
            this.currentLanguage = this.config.self_ordering_default_language_id;
        }

        cookie.set("frontend_lang", this.currentLanguage.code);

        this.products = this.products.map((p) => {
            const product = new Product(p, this.config.iface_tax_included);
            this.productByIds[product.id] = product;

            if (product.attributes.length > 0) {
                for (const atr of product.attributes) {
                    this.attributeById[atr.id] = atr;
                    for (const val of atr.values) {
                        val.attribute_id = atr.id;
                        this.attributeValueById[val.id] = val;
                    }
                }
            }

            return product;
        });

        this.combos = this.combos.map((c) => {
            const combo = new Combo(c);
            this.comboByIds[combo.id] = combo;
            return combo;
        });

        this.productsGroupedByCategory = this.products.reduce((acc, product) => {
            product.pos_categ_ids.map((categ) => {
                acc[categ.id] = acc[categ.id] || [];
                acc[categ.id].push(product);
            });
            return acc;
        }, {});

        if (this.productsGroupedByCategory[0]) {
            this.pos_category.push({
                has_image: false,
                id: 0,
                name: _t("Uncategorised"),
                sequence: 9999,
            });
        }

        this.categoryList = new Set(
            this.pos_category
                .sort((a, b) => a.sequence - b.sequence)
                .filter((c) => this.productsGroupedByCategory[c.id])
                .sort((a, b) => categorySorter(a, b, this.config.iface_start_categ_id))
        );

        this.currentCategory = this.pos_category.length > 0 ? [...this.categoryList][0] : null;

        for (const printerConfig of Object.values(this.kitchen_printers)) {
            const printer = this.create_printer(printerConfig);
            if (printer) {
                printer.config = printerConfig;
                this.kitchenPrinters.push(printer);
            }
        }
    }

    create_printer(printer) {
        const url = deduceUrl(printer.proxy_ip || "");
        return new HWPrinter({ url });
    }

    _getKioskPrintingCategoriesChanges(categories) {
        return this.currentOrder.lines.filter((orderline) =>
            categories.some((categId) =>
                this.productByIds[orderline["product_id"]].pos_categ_ids
                    .map((categ) => categ.id)
                    .includes(categId)
            )
        );
    }

    async printKioskChanges() {
        const d = new Date();
        let hours = "" + d.getHours();
        hours = hours.length < 2 ? "0" + hours : hours;
        let minutes = "" + d.getMinutes();
        minutes = minutes.length < 2 ? "0" + minutes : minutes;
        for (const printer of this.kitchenPrinters) {
            const orderlines = this._getKioskPrintingCategoriesChanges(
                Object.values(printer.config.product_categories_ids)
            );
            if (orderlines) {
                const printingChanges = {
                    new: orderlines,
                    tracker: this.currentOrder.table_stand_number,
                    trackingNumber: this.currentOrder.trackingNumber || "unknown number",
                    name: this.currentOrder.pos_reference || "unknown order",
                    time: {
                        hours,
                        minutes,
                    },
                };
                const receipt = renderToElement("pos_self_order.OrderChangeReceipt", {
                    changes: printingChanges,
                });
                await printer.printReceipt(receipt);
            }
        }
    }

    initKioskData() {
        if (this.pos_session && this.access_token) {
            this.ordering = true;
        }

        this.idleTimout = false;
        window.addEventListener("click", (event) => {
            this.idleTimout && clearTimeout(this.idleTimout);
            this.idleTimout = setTimeout(() => {
                if (this.router.activeSlot !== "payment") {
                    this.router.navigate("default");
                }
            }, 5 * 1000 * 60);
        });
    }

    async initMobileData() {
        if (this.config.self_ordering_mode !== "qr_code") {
            if (this.pos_session && this.access_token) {
                this.ordering = true;
            }

            if (!this.ordering) {
                return;
            }

            effect(
                batched((state) => this.saveOrderToLocalStorage(state.orders)),
                [this]
            );

            const orders = JSON.parse(localStorage.getItem("orders")) ?? [];
            this.orders.push(
                ...orders.map((o) => {
                    o.lines = o.lines.filter((l) => this.productByIds[l.product_id]);
                    return new Order(o);
                })
            );

            effect((state) => this.saveOrderToLocalStorage(state.orders), [this]);
            await this.getOrdersFromServer();
            await this.getPricesFromServer();
        }
    }

    saveOrderToLocalStorage(orders) {
        Array.isArray(orders) && localStorage.setItem("orders", JSON.stringify(orders));
    }

    cancelOrder() {
        const changes = this.currentOrder.lastChangesSent;
        const lines = this.currentOrder.lines;
        const keptLines = [];

        for (const line of lines) {
            const change = changes[line.uuid];

            if (change) {
                line.qty = change.qty;
                line.customer_note = change.customer_note;
                line.attribute_value_ids = change.attribute_value_ids;
                keptLines.push(line);
            }
        }

        this.currentOrder.lines = keptLines;

        if (this.currentOrder.totalQuantity === 0) {
            this.router.navigate("default");
            this.editedOrder = null;
            this.flushNotSentOrder();
        }
    }

    flushNotSentOrder() {
        this.orders = this.orders.filter((o) => o.isSavedOnServer);
    }

    async sendDraftOrderToServer() {
        if (this.currentOrder.isSavedOnServer || this.currentOrder.lines.length === 0) {
            return this.currentOrder;
        }

        try {
            const rpcUrl = this.currentOrder.isAlreadySent
                ? "/pos-self-order/update-existing-order"
                : `/pos-self-order/process-new-order/${this.config.self_ordering_mode}`;

            const order = await this.rpc(rpcUrl, {
                order: this.currentOrder,
                access_token: this.access_token,
                table_identifier: this.table ? this.table.identifier : null,
            });

            this.editedOrder.access_token = order.access_token;
            this.updateOrdersFromServer([order], [order.access_token]);
            this.editedOrder.updateLastChanges();

            if (this.config.self_ordering_pay_after === "each" && order.amount_total > 0) {
                this.editedOrder = null;
            }

            return order;
        } catch (error) {
            this.handleErrorNotification(error, [this.editedOrder.access_token]);
            return false;
        }
    }

    async getOrdersFromServer() {
        const accessTokens = this.orders.map((order) => order.access_token).filter(Boolean);

        if (accessTokens.length === 0) {
            return;
        }

        try {
            const orders = await this.rpc(`/pos-self-order/get-orders/`, {
                access_token: this.access_token,
                order_access_tokens: accessTokens,
            });

            this.updateOrdersFromServer(orders, accessTokens);
            this.editedOrder = null;
        } catch (error) {
            this.handleErrorNotification(
                error,
                this.orders.map((order) => order.access_token)
            );
        }
    }

    updateOrdersFromServer(ordersFromServer, localAccessToken) {
        const accessTokensFromServer = ordersFromServer.map((order) => order.access_token);

        for (const order of this.orders) {
            if (order.access_token) {
                const orderFromServer = ordersFromServer.find(
                    (o) => o.access_token === order.access_token
                );
                if (orderFromServer) {
                    order.updateDataFromServer(orderFromServer);
                    const newLine = this.currentOrder.lines.find((l) => !l.id);
                    if (!newLine) {
                        order.updateLastChanges();
                    }
                }
            }
        }

        for (const index in this.orders) {
            if (
                !accessTokensFromServer.includes(this.orders[index].access_token) &&
                localAccessToken.includes(this.orders[index].access_token)
            ) {
                this.orders.splice(index, 1);
            }
        }
    }

    changeOrderState(access_token, state) {
        const order = this.orders.filter((o) => o.access_token === access_token);
        let message = _t("Your order status has been changed");

        if (order.length === 0) {
            this.handleErrorNotification(new Error("Warning, no order with this access_token"));
        } else if (order.length !== 1) {
            this.handleErrorNotification(
                new Error("Warning, two orders with the same access_token")
            );
        } else {
            order[0].state = state;
        }

        if (state === "paid") {
            this.editedOrder = null;
            message = _t("Your order has been paid");
        } else if (state === "cancel") {
            this.editedOrder = null;
            message = _t("Your order has been canceled");
        }

        this.notification.add(message, {
            type: "success",
        });

        if (this.router.activeSlot !== "confirmation") {
            this.router.navigate("default");
        }
    }

    updateOrderFromServer(order) {
        this.currentOrder.updateDataFromServer(order);
    }

    isOrder() {
        if (!this.currentOrder || !this.currentOrder.lines.length) {
            this.router.navigate("default");
        }
    }

    async getPricesFromServer() {
        try {
            if (!this.currentOrder) {
                return;
            }
            if (this.priceLoading) {
                this.priceLoading.abort(false);
            }

            this.priceLoading = this.rpc(`/pos-self-order/get-orders-taxes/`, {
                order: this.currentOrder,
                access_token: this.access_token,
            });
            await this.priceLoading.then((taxes) => {
                this.currentOrder.updateDataFromServer(taxes);
                this.priceLoading = false;
            });
        } catch (error) {
            this.handleErrorNotification(error);
        }
    }

    handleErrorNotification(error, accessToken = []) {
        this.rpcLoading = false;

        let message = _t("An error has occurred");
        let cleanOrders = false;

        if (error instanceof RPCError) {
            if (error.data.name === "werkzeug.exceptions.Unauthorized") {
                message = _t("You're not authorized to perform this action");
                cleanOrders = true;
            } else if (error.data.name === "werkzeug.exceptions.NotFound") {
                message = _t("Orders not found on server");
                cleanOrders = true;
            }
        } else if (error instanceof ConnectionLostError) {
            message = _t("Connection lost, please try again later");
        }

        this.notification.add(message, {
            type: "danger",
        });

        if (accessToken && cleanOrders) {
            this.editedOrder = null;

            for (const index in this.orders) {
                if (accessToken.includes(this.orders[index].access_token)) {
                    this.orders.splice(index, 1);
                }
            }
        }
    }

    formatMonetary(price) {
        return formatMonetary(price, { currencyId: this.currency_id });
    }

    handleProductChanges(payload) {
        const product = new Product(payload.product, this.config.iface_tax_included);
        this.productByIds[payload.product.id] = product;
        for (const categ of payload.product.pos_categ_ids) {
            if (!this.pos_category.map((c) => c.id).includes(categ.id)) {
                continue;
            }
            const index = this.productsGroupedByCategory[categ.id].findIndex(
                (p) => p.id === product.id
            );
            if (index >= 0) {
                this.productsGroupedByCategory[categ.id][index] = product;
            } else {
                this.productsGroupedByCategory[categ.id].push(product);
            }
        }
    }

    verifyCart() {
        let result = true;
        for (const line of this.currentOrder.hasNotAllLinesSent()) {
            if (line.combo_parent_uuid) {
                continue;
            }
            const alreadySent = this.currentOrder.lastChangesSent
                ? this.currentOrder.lastChangesSent[line.uuid]
                : false;
            const wrongChild = line.child_lines.find(
                (l) => !this.productByIds[l.product_id]?.self_order_available
            );
            if (wrongChild || !this.productByIds[line.product_id]?.self_order_available) {
                if (alreadySent) {
                    line.qty = alreadySent.qty;
                    line.customer_note = alreadySent.customer_note;
                    line.selected_attributes = alreadySent.selected_attributes;
                } else {
                    this.currentOrder.removeLine(line.uuid);
                }
                this.notification.add(
                    _t(
                        "%s is not available anymore, it has thus been removed from your order. Please review your order and validate it again.",
                        line.full_product_name
                    ),
                    { type: "danger" }
                );
                result = false;
            }
        }

        return result;
    }

    verifyPriceLoading() {
        if (this.priceLoading) {
            this.notification.add(_t("Please wait until the price is loaded"), {
                type: "danger",
            });
            return false;
        }
        return true;
    }

    getProductDisplayPrice(product) {
        if (this.currentOrder.take_away) {
            return product.price_info.display_price_alternative;
        } else {
            return product.price_info.display_price_default;
        }
    }
    getLinePrice(line) {
        return this.config.iface_tax_included ? line.price_subtotal_incl : line.price_subtotal;
    }
    getSelectedAttributes(line) {
        const attributeValues = line.attribute_value_ids;
        const customAttr = line.custom_attribute_value_ids;
        return attributeFormatter(this.attributeById, attributeValues, customAttr);
    }
    getFullProductName(line) {
        const attrs = this.getSelectedAttributes(line);
        const attrsStr = " (" + attrs.map((a) => a.value).join(", ") + ")";
        return line.full_product_name + (attrs.length ? attrsStr : "");
    }
    getLineDisplayData(line) {
        return {
            productName: this.getFullProductName(line),
            price: this.formatMonetary(this.getLinePrice(line)),
            qty: line.qty.toString(),
            unitPrice: this.formatMonetary(this.getLinePrice(line) / line.qty),
            comboParent: line.combo_parent_uuid
                ? this.currentOrder.lines.find((l) => l.uuid === line.combo_parent_uuid)
                      .full_product_name
                : "",
            customerNote: line.customer_note,
        };
    }
    export_for_printing(order) {
        order.receiptDate ||= formatDateTime(luxon.DateTime.now());
        return {
            orderlines: order.lines.map(this.getLineDisplayData.bind(this)),
            amount_total: order.amount_total,
            amount_tax: order.amount_tax,
            total_without_tax: order.amount_total - order.amount_tax,
            total_paid: order.amount_paid,
            paymentlines: order.payment_lines,
            change: 0,
            tax_details: order.tax_details,
            name: order.pos_reference,
            date: order.receiptDate,
            ticket_code: this.company.point_of_sale_ticket_unique_code && order.ticketCode,
            base_url: this.base_url,
            footer: this.config.receipt_footer,
            headerData: {
                header: this.config.receipt_header,
                company: this.company,
                trackingNumber: order.trackingNumber,
                bigTrackingNumber: true,
                pickingService: this.config.self_ordering_service_mode,
                tableTracker: order.table_stand_number ?? false,
            },
            pos_qr_code:
                this.company.point_of_sale_use_ticket_qr_code &&
                qrCodeSrc(
                    `${this.base_url}/pos/ticket/validate?access_token=${order.access_token}`
                ),
        };
    }
}

export const selfOrderService = {
    dependencies: ["rpc", "notification", "router", "printer", "barcode"],
    async start(env, services) {
        return new SelfOrder(env, services).ready;
    },
};

registry.category("services").add("printer", printerService);
registry.category("services").add("self_order", selfOrderService);

export function useSelfOrder() {
    return useState(useService("self_order"));
}

```

## File: static\src\app\utils.js

```javascript
/** @odoo-module */

export const categorySorter = (a, b, start_categ_id) => {
    if (a.id === start_categ_id && b.id !== start_categ_id) {
        return -1; // 'a' should come before 'b'
    }
    if (a.id !== start_categ_id && b.id === start_categ_id) {
        return 1; // 'b' should come before 'a'
    }
    if (a.sequence !== b.sequence) {
        return a.sequence - b.sequence; // sort by sequence
    }
    return a.id - b.id; // sort by id if sequences are the same
};

export const attributeFormatter = (attrById, values, customValues = []) => {
    if (!values) {
        return [];
    }

    const attrVals = {};
    for (const attr of Object.values(attrById)) {
        for (const value of attr.values) {
            attrVals[value.id] = value;
        }
    }

    const selectedValue = Object.values(attrVals)
        .filter((attr) => values.includes(attr.id))
        .reduce((acc, val) => {
            let description = "";
            const attribute = attrById[val.attribute_id];
            const isCustomValue = Object.values(customValues).find(
                (cus) => cus.custom_product_template_attribute_value_id === val.id
            );

            if (isCustomValue && val.is_custom) {
                description = `: ${isCustomValue.custom_value}`;
            }

            if (!acc[attribute.id]) {
                acc[attribute.id] = {
                    id: attribute.id,
                    name: attribute.name,
                    value: val.name + description,
                    valueIds: [val.id],
                };
            } else {
                acc[attribute.id].value += `, ${val.name}${description}`;
                acc[attribute.id].valueIds.push(val.id);
            }

            return acc;
        }, {});

    return Object.values(selectedValue);
};

export const attributeFlatter = (attribute) => {
    return Object.values(attribute)
        .map((v) => {
            if (v instanceof Object) {
                return Object.entries(v)
                    .filter((v) => v[1])
                    .map((v) => v[0]);
            } else {
                return v;
            }
        })
        .flat()
        .map((v) => parseInt(v));
};

```

## File: static\src\app\components\attribute_selection\attribute_selection.js

```javascript
/** @odoo-module */

import { Component, onMounted, useRef, useState } from "@odoo/owl";
import { ProductCustomAttribute } from "@point_of_sale/app/store/models/product_custom_attribute";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { attributeFlatter, attributeFormatter } from "@pos_self_order/app/utils";
import { floatIsZero } from "@web/core/utils/numbers";

export class AttributeSelection extends Component {
    static template = "pos_self_order.AttributeSelection";
    static props = ["product"];

    setup() {
        this.selfOrder = useSelfOrder();
        this.numberOfAttributes = this.props.product.attributes.length;
        this.currentAttribute = 0;

        this.gridsRef = {};
        this.valuesRef = {};
        for (const attr of this.props.product.attributes) {
            this.gridsRef[attr.id] = useRef(`attribute_grid_${attr.id}`);
            this.valuesRef[attr.id] = {};
            for (const value of attr.values) {
                this.valuesRef[attr.id][value.id] = useRef(`value_${attr.id}_${value.id}`);
            }
        }

        this.state = useState({
            showNext: false,
            showCustomInput: false,
        });

        this.selectedValues = useState(this.env.selectedValues);

        this.initAttribute();
        onMounted(this.onMounted);
    }

    onMounted() {
        for (const attr of Object.entries(this.valuesRef)) {
            let classicValue = 0;
            for (const valueRef of Object.values(attr[1])) {
                if (valueRef.el) {
                    const height = valueRef.el.parentNode.offsetHeight;
                    if (classicValue === 0) {
                        classicValue = height;
                    } else {
                        if (height !== classicValue || height > window.innerHeight * 0.18) {
                            this.gridsRef[attr[0]].el.classList.remove(
                                "row-cols-2",
                                "row-cols-sm-3",
                                "row-cols-md-4",
                                "row-cols-xl-5",
                                "row-cols-xxl-6"
                            );
                            this.gridsRef[attr[0]].el.classList.add("row-cols-1");
                            for (const gridValueRef of Object.values(attr[1])) {
                                gridValueRef.el.classList.remove("ratio", "ratio-16x9");
                            }
                            break;
                        }
                    }
                }
            }
        }
    }

    get showNextBtn() {
        for (const attrSelection of Object.values(this.selectedValues)) {
            if (!attrSelection) {
                return false;
            }
        }

        return true;
    }

    get attributeSelected() {
        const flatAttribute = attributeFlatter(this.selectedValues);
        const customAttribute = this.env.customValues;
        return attributeFormatter(this.selfOrder.attributeById, flatAttribute, customAttribute);
    }

    availableAttributeValue(attribute) {
        return this.selfOrder.config.self_ordering_mode === "kiosk"
            ? attribute.values.filter((a) => !a.is_custom)
            : attribute.values;
    }

    initAttribute() {
        const initCustomValue = (value) => {
            let selectedValue = this.selfOrder.editedLine?.custom_attribute_value_ids.find(
                (v) => v.custom_product_template_attribute_value_id === value.id
            );

            if (!selectedValue) {
                selectedValue = new ProductCustomAttribute({
                    custom_product_template_attribute_value_id: value.id,
                });
            }

            return selectedValue;
        };

        const initValue = (value) => {
            if (this.selfOrder.editedLine?.attribute_value_ids.includes(value.id)) {
                return value.id;
            }
            return false;
        };

        for (const attr of this.props.product.attributes) {
            this.selectedValues[attr.id] = {};

            for (const value of attr.values) {
                if (attr.display_type === "multi") {
                    this.selectedValues[attr.id][value.id] = initValue(value);
                } else if (typeof this.selectedValues[attr.id] !== "number") {
                    this.selectedValues[attr.id] = initValue(value);
                }

                if (value.is_custom) {
                    this.env.customValues[value.id] = initCustomValue(value);
                }
            }
        }
    }

    isChecked(attribute, value) {
        return attribute.display_type === "multi"
            ? this.selectedValues[attribute.id][value.id]
            : parseInt(this.selectedValues[attribute.id]) === value.id;
    }

    _getPriceExtra(value) {
        const isTakeAway = this.selfOrder.take_away;
        const priceExtra = isTakeAway
            ? value.price_extra.display_price_default
            : value.price_extra.display_price_alternative;
        return priceExtra;
    }

    shouldShowPriceExtra(value) {
        const priceExtra = this._getPriceExtra(value);
        return !floatIsZero(priceExtra, this.selfOrder.config.currency_decimals);
    }

    getfPriceExtra(value) {
        const priceExtra = this._getPriceExtra(value);
        const sign = priceExtra < 0 ? "- " : "+ ";
        return sign + this.selfOrder.formatMonetary(Math.abs(priceExtra));
    }
}

```

## File: static\src\app\components\attribute_selection\attribute_selection.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.AttributeSelection">
        <div class="self_order_attribute_selection d-flex flex-column flex-grow-1">
            <div class="attribute-selection-content align-items-center justify-content-start px-3 flex-grow-1">
                <div class="d-flex flex-column">
                    <div t-foreach="this.props.product.attributes" t-as="attribute" t-key="attribute.id" class="attribute-row">
                        <h2 t-out="attribute.name"/>
                        <div class="row g-2 g-md-3 g-xl-4 justify-content-between justify-content-md-start mb-5 row-cols-2 row-cols-sm-3 row-cols-md-4 row-cols-xl-5 row-cols-xxl-6"
                            t-ref="attribute_grid_{{attribute.id}}">
                            <t t-foreach="availableAttributeValue(attribute)" t-as="value" t-key="value.id">
                                <div class="col">
                                    <label t-attf-for="{{ attribute.id }}_{{ value.id }}"
                                            t-attf-class="self_order_attribute_selection_option {{ this.isChecked(attribute, value) ? 'text-bg-primary border-primary active' : '' }}
                                            d-flex align-items-center justify-content-center h-100 rounded border ratio ratio-16x9"
                                            t-ref="value_{{attribute.id}}_{{value.id}}">
                                        <div class="name position-relative d-flex flex-column justify-content-center align-items-center flex-grow-1 w-100 p-4 text-center">
                                            <span t-out="value.name"/>
                                            <span t-if="shouldShowPriceExtra(value)">
                                                <t t-esc="getfPriceExtra(value)"/>
                                            </span>
                                        </div>
                                    </label>
                                    <input
                                        type="radio" 
                                        class="d-none"
                                        t-if="attribute.display_type !== 'multi'"
                                        t-att-value="value.id"
                                        t-attf-id="{{ attribute.id }}_{{ value.id }}"
                                        t-model="this.selectedValues[attribute.id]" />
                                    <input
                                        type="checkbox"
                                        class="d-none"
                                        t-else=""
                                        t-att-checked="this.isChecked(attribute, value)"
                                        t-att-value="value.id"
                                        t-model="this.selectedValues[attribute.id][value.id]"
                                        t-attf-id="{{ attribute.id }}_{{ value.id }}" />
                                </div>
                                <div t-if="this.isChecked(attribute, value) and selfOrder.attributeValueById[value.id].is_custom" class="col w-100 order-2">
                                    <input type="text" t-model="this.env.customValues[value.id].custom_value" class="form-control form-control-lg" placeholder="Enter your custom value" />
                                </div>
                            </t>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\components\cancel_popup\cancel_popup.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";

export class CancelPopup extends Component {
    static template = "pos_self_order.CancelPopup";

    setup() {
        this.selfOrder = useSelfOrder();
    }

    confirm() {
        this.props.close();
        this.props.confirm();
    }
}

```

## File: static\src\app\components\cancel_popup\cancel_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.CancelPopup">
        <div class="self_order_cancel_popup o_dialog" t-att-id="id">
            <div role="dialog" class="modal d-block" tabindex="-1">
                  <div class="modal-dialog" role="document">
                    <div class="modal-content rounded">
                        <div class="modal-body p-5">
                            <div class="pb-5 fs-3 text-center">
                                Are you sure you want to cancel this order? <br/>
                                <span t-if="selfOrder.config.self_ordering_mode === 'kiosk'" class="text-muted fs-4">All the items will be removed from the cart.</span>
                                <span t-else="" class="text-muted fs-4">Any items already sent will not be canceled</span>
                            </div>
                            <div class="d-flex align-items-center justify-content-center w-100 gap-3">
                                <button type="button" class="btn btn-primary btn-lg popup_button" t-on-click="() => this.confirm()">Cancel Order</button>
                                <button type="button" class="btn btn-secondary btn-lg popup_button" t-on-click="() => this.props.close()">Discard</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\components\combo_selection\combo_selection.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { AttributeSelection } from "@pos_self_order/app/components/attribute_selection/attribute_selection";

export class ComboSelection extends Component {
    static template = "pos_self_order.ComboSelection";
    static props = ["combo", "comboState", "next"];
    static components = { AttributeSelection };

    setup() {
        this.selfOrder = useSelfOrder();
    }

    productClicked(lineId) {
        // Keep track of the current combo line id.
        // It servers as additional info for each line so that when calculating prices,
        // no need to look for the specific combo line the product belongs to.
        this.env.currentComboLineId.value = lineId;

        const comboLine = this.props.combo.combo_line_ids.find((line) => line.id == lineId);
        const productSelected = this.selfOrder.productByIds[comboLine.product_id[0]];
        if (!productSelected.self_order_available) {
            return;
        }
        this.props.comboState.selectedProduct = productSelected;
        if (productSelected.attributes.length === 0) {
            this.props.next();
            return;
        }
        this.props.comboState.showQtyButtons = true;
    }
}

```

## File: static\src\app\components\combo_selection\combo_selection.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ComboSelection">
        <div t-if="!props.comboState.selectedProduct" class="self_order_attribute_selection p-3 mt-3">
            <h2 class="attribute_name mb-5 mb-md-3"><small class="text-muted">Choose your</small> <strong t-esc="props.combo.name" /></h2>
            <div class="combo-list align-items-center justify-content-start">
                <div class="combo-list-products o-so-products-row">
                    <t t-foreach="props.combo.combo_line_ids" t-as="line" t-key="line.id">
                        <t t-set="isOutOfStock" t-value="!this.selfOrder.productByIds[line.product_id[0]].self_order_available"/>

                        <article
                            t-attf-for="{{ props.combo.id }}_{{ line.id }}"
                            t-on-click="() => this.productClicked(line.id)"
                            class="self_order_product_card d-flex flex-row-reverse flex-md-column align-items-start gap-2 user-select-none"
                            role="button"
                            >
                            <t t-set="product" t-value="this.selfOrder.productByIds[line.product_id[0]]"/>

                            <div
                                class="ratio ratio-1x1 w-25 w-sm-50 w-md-100"
                                t-att-class="{
                                    'd-none d-md-block': !product.has_image
                                }">
                                <div class="placeholder-glow">
                                    <div class="placeholder w-100 h-100 bg-300 rounded"/>
                                </div>
                                <img
                                    class="o_self_order_item_card_image w-100 rounded"
                                    t-attf-src="/menu/get-image/{{ product.id }}/512?unique={{product.write_date}}"
                                    alt="Product image"
                                    loading="lazy"
                                    onerror="this.remove()"/>
                            </div>
                            <div class="product-infos d-flex flex-column justify-content-between text-start flex-grow-1 w-100 lh-1">
                                <span t-esc="product.name" class="fs-4 fw-bold mb-1 mb-sm-2"/>
                                <div class="d-flex justify-content-between gap-3">
                                    <span
                                        t-if="line.combo_price"
                                        class="badge rounded-pill fs-4"
                                        t-att-class="isOutOfStock ? 'text-bg-secondary' : 'text-bg-primary'">
                                        + <t t-out="selfOrder.formatMonetary(line.combo_price)"/>
                                    </span>
                                    <span t-if="isOutOfStock" class="badge text-bg-danger rounded-pill fs-4">
                                        Out of stock
                                    </span>
                                </div>
                            </div>
                        </article>
                    </t>
                </div>
            </div>
        </div>
        <t t-else="">
            <AttributeSelection product="props.comboState.selectedProduct" />
        </t>
    </t>
</templates>

```

## File: static\src\app\components\language_popup\language_popup.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { cookie } from "@web/core/browser/cookie";

export class LanguagePopup extends Component {
    static template = "pos_self_order.LanguagePopup";

    setup() {
        this.selfOrder = useSelfOrder();
    }

    get languages() {
        return this.selfOrder.config.self_ordering_available_language_ids;
    }

    get currentLanguage() {
        return this.selfOrder.currentLanguage;
    }

    onClickLanguage(language) {
        cookie.set("frontend_lang", language.code);
        window.location.reload();
    }
}

```

## File: static\src\app\components\language_popup\language_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.LanguagePopup">
        <div class="self_order_language_popup o_dialog" t-att-id="id">
            <div role="dialog" class="modal d-block" tabindex="-1">
                  <div class="modal-dialog" role="document">
                    <div class="modal-content">
                        <div class="modal-body p-5">
                            <div class="d-grid gap-3">
                                <t t-foreach="languages" t-as="lang" t-key="lang.id" >
                                    <t t-if="lang.id !== currentLanguage.id">
                                        <div class="btn btn-light d-flex flex-row align-items-center rounded border p-4" t-on-click="() => this.onClickLanguage(lang)">
                                            <img class="rounded-2" t-attf-src="{{lang.flag_image_url}}" />
                                            <span class="fs-5 ms-4" t-esc="lang.display_name" />
                                        </div>
                                    </t>
                                </t>
                            </div>
                            <div class="d-flex align-items-center justify-content-center w-100 mt-5">
                                <button type="button" class="btn btn-secondary btn-lg popup_button" t-on-click="() => this.props.close()">Discard</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\components\loading_overlay\loading_overlay.js

```javascript
/** @odoo-module */

import { Component, onMounted, useState } from "@odoo/owl";

export class LoadingOverlay extends Component {
    static template = "pos_self_order.LoadingOverlay";

    setup() {
        this.state = useState({
            loading: false,
        });

        onMounted(() => {
            setTimeout(() => {
                this.state.loading = true;
            }, 200);
        });
    }
}

```

## File: static\src\app\components\loading_overlay\loading_overlay.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.LoadingOverlay">
        <div t-if="state.loading" class="position-absolute top-0 w-100 min-vh-100 d-flex justify-content-center align-items-center bg-black opacity-50" style="z-index: 1000;">
            <span class="spinner-border" />
        </div>
    </t>
</templates>

```

## File: static\src\app\components\order_widget\order_widget.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { useService } from "@web/core/utils/hooks";
import { _t } from "@web/core/l10n/translation";
import { CancelPopup } from "@pos_self_order/app/components/cancel_popup/cancel_popup";

export class OrderWidget extends Component {
    static template = "pos_self_order.OrderWidget";
    static props = ["action", "removeTopClasses?"];

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        this.dialog = useService("dialog");
    }

    cancel() {
        if (this.selfOrder.config.self_ordering_mode === "kiosk") {
            this.dialog.add(CancelPopup, {
                title: _t("Cancel order"),
                confirm: () => {
                    this.selfOrder.cancelOrder();
                },
            });
        } else {
            this.selfOrder.cancelOrder();
        }
    }

    get cancelAvailable() {
        return (
            !this.selfOrder.currentOrder.isSavedOnServer ||
            this.selfOrder.config.self_ordering_mode === "kiosk"
        );
    }

    get buttonToShow() {
        const currentPage = this.router.activeSlot;
        const payAfter = this.selfOrder.config.self_ordering_pay_after;
        const kioskPayment = this.selfOrder.pos_payment_methods;
        const isNoLine = this.selfOrder.currentOrder.lines.length === 0;
        const hasNotAllLinesSent = this.selfOrder.currentOrder.hasNotAllLinesSent();
        const isMobilePayment = this.selfOrder.pos_payment_methods.find((p) => p.is_mobile_payment);

        let label = "";
        let disabled = false;

        if (currentPage === "product_list") {
            label = _t("Order");
            disabled = isNoLine || hasNotAllLinesSent.length == 0;
        } else if (payAfter === "meal" && !this.selfOrder.currentOrder.isSavedOnServer) {
            label = _t("Order");
            disabled = isNoLine;
        } else {
            label = kioskPayment ? _t("Pay") : _t("Order");
            disabled = !kioskPayment && !isMobilePayment;
        }

        return { label, disabled };
    }

    get lineNotSend() {
        const order = this.selfOrder.currentOrder;
        const lineNotSend = order.hasNotAllLinesSent();

        return lineNotSend.reduce(
            (acc, line) => {
                const currentQty = line.qty;
                const lastChange = order.lastChangesSent[line.uuid];
                const qty = !lastChange ? currentQty : currentQty - lastChange.qty;

                acc.count += qty;
                const subtotal = this.selfOrder.config.iface_tax_included
                    ? line.price_subtotal_incl
                    : line.price_subtotal;
                acc.price += (subtotal / currentQty) * qty;

                return acc;
            },
            {
                price: 0,
                count: 0,
            }
        );
    }

    get leftButton() {
        const order = this.selfOrder.currentOrder;
        const back =
            order.isSavedOnServer || this.router.activeSlot === "cart" || order.lines.length === 0;

        return {
            name: back ? _t("Back") : _t("Cancel"),
            icon: back ? "fa fa-arrow-left btn-back" : "btn-close btn-cancel",
        };
    }

    onClickleftButton() {
        const order = this.selfOrder.currentOrder;

        if (
            order.lines.length === 0 ||
            order.isSavedOnServer ||
            this.router.activeSlot === "cart"
        ) {
            this.router.back();
            return;
        } else {
            this.dialog.add(CancelPopup, {
                title: _t("Cancel order"),
                confirm: () => {
                    this.selfOrder.cancelOrder();
                    this.router.navigate("default");
                },
            });
        }
    }
}

```

## File: static\src\app\components\order_widget\order_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.OrderWidget">
        <div
            class="page-buttons d-flex flex-nowrap justify-content-between py-2 px-3 gap-2 gap-md-3 bg-view z-index-1"
            t-att-class="{
                'shadow-lg border-top' : !props.removeTopClasses
            }">
            <button t-attf-class="btn btn-secondary btn-lg h-auto w-auto opacity-75 px-4 d-sm-none" t-att-class="leftButton.icon" t-on-click="onClickleftButton">
                <span class="px-1"/><!-- Spacer -->
            </button>
            <button t-attf-class="btn btn-secondary btn-lg d-none d-sm-inline text-nowrap" t-on-click="onClickleftButton" t-esc="leftButton.name" />
            <div class="d-flex align-items-center justify-content-end flex-grow-1 w-100 w-md-auto">
                <div class="to-order">
                    <span>Your Order</span>
                    <div class="d-flex align-items-center">
                        <span class="o-so-tabular-nums badge text-bg-secondary rounded" t-esc="lineNotSend.count"/>
                        <span class="o-so-tabular-nums mx-2" t-esc="selfOrder.formatMonetary(lineNotSend.price)" />
                    </div>
                </div>
            </div>
            <button t-attf-class="cart btn btn-primary btn-lg flex-grow-1 flex-md-grow-0 {{ buttonToShow.disabled ? 'disabled' : '' }}" t-on-click="props.action" t-esc="buttonToShow.label"/>
        </div>
    </t>
</templates>

```

## File: static\src\app\components\popup_table\popup_table.js

```javascript
/** @odoo-module */

import { Component, onWillStart, useState } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { useService } from "@web/core/utils/hooks";
import { groupBy } from "@web/core/utils/arrays";

export class PopupTable extends Component {
    static template = "pos_self_order.PopupTable";
    static props = { selectTable: Function };

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        this.rpc = useService("rpc");
        this.tables = [];
        this.state = useState({
            selectedTable: null,
        });

        onWillStart(async () => {
            await this.getTable();
        });
    }

    async getTable() {
        try {
            this.tables = await this.rpc("/pos-self-order/get-tables", {
                access_token: this.selfOrder.access_token,
            });
        } catch (e) {
            this.selfOrder.handleErrorNotification(e);
        }

        this.state.selectedTable = this.tables[0]?.id;
    }

    get availableFloor() {
        const groupedFloors = groupBy(this.tables, (t) => t.floor_id[0]);
        return Object.entries(groupedFloors).map(([floorId, tables]) => ({
            id: floorId,
            name: tables[0].floor_id[1],
            tables,
        }));
    }

    setTable() {
        const table = this.tables.find((t) => t.id === parseInt(this.state.selectedTable));
        this.props.selectTable(table);
    }

    close() {
        this.props.selectTable(null);
    }
}

```

## File: static\src\app\components\popup_table\popup_table.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.PopupTable">
        <div class="position-absolute bg-dark bg-opacity-25 w-100 h-100 fixed-top" />
        <div class="self_order_popup_table shadow-lg position-absolute fixed-bottom bg-white w-100 p-4 flex-column d-flex justify-content-between">
            <div class="mb-5 d-flex justify-content-between align-items-start">
                <div>
                    <h3>Table detective time!</h3>
                    <span>Could you please confirm your table number?<br/>Thanks a lot!</span>
                </div>
                <button class="btn btn-close" t-on-click="close"/>
            </div>
            <select class="form-select form-select-lg mb-5" t-model="state.selectedTable">
                <t t-foreach="availableFloor" t-as="floor" t-key="floor.id">
                    <option value="floor" disabled="true">
                        <t t-esc="floor.name" />
                    </option>
                    <option t-foreach="floor.tables" t-as="table" t-key="table.id" t-att-value="table.id">
                        <t t-esc="table.name" />
                    </option>
                </t>
            </select>
            <a
                type="button"
                t-on-click="() => this.setTable()"
                class="btn btn-primary py-3 my-2">
                Confirm
            </a>
        </div>
    </t>
</templates>

```

## File: static\src\app\components\product_card\product_card.js

```javascript
/** @odoo-module */

import { Component, useRef } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { useService, useForwardRefToParent } from "@web/core/utils/hooks";
import { Line } from "@pos_self_order/app/models/line";
import { ProductInfoPopup } from "@pos_self_order/app/components/product_info_popup/product_info_popup";
import { constructFullProductName } from "@point_of_sale/utils";

export class ProductCard extends Component {
    static template = "pos_self_order.ProductCard";
    static props = ["product", "currentProductCard?"];

    selfRef = useRef("selfProductCard");
    currentProductCardRef = useRef("currentProductCard");

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        this.dialog = useService("dialog");

        useForwardRefToParent("currentProductCard");
    }

    flyToCart() {
        const productCardEl = this.selfRef.el;
        if (!productCardEl) {
            return;
        }

        const toOrder = document.querySelector(".to-order");
        if (!toOrder || window.getComputedStyle(toOrder).display === "none") {
            return;
        }

        let pic = this.selfRef.el.querySelector(".o_self_order_item_card_image");
        if (!pic) {
            pic = this.selfRef.el.querySelector(".o_self_order_item_card_no_image");
        }

        const picRect = pic.getBoundingClientRect();
        const clonedPic = pic.cloneNode(true);
        const toOrderRect = toOrder.getBoundingClientRect();

        clonedPic.classList.remove("w-100", "h-100");
        clonedPic.classList.add(
            "position-fixed",
            "border",
            "border-white",
            "border-4",
            "z-index-1"
        );
        clonedPic.style.top = `${picRect.top}px`;
        clonedPic.style.left = `${picRect.left}px`;
        clonedPic.style.width = `${picRect.width}px`;
        clonedPic.style.height = `${picRect.height}px`;
        clonedPic.style.transition = "all 400ms cubic-bezier(0.6, 0, 0.9, 1.000)";

        document.body.appendChild(clonedPic);

        requestAnimationFrame(() => {
            const offsetTop = toOrderRect.top - picRect.top - picRect.height * 0.5;
            const offsetLeft = toOrderRect.left - picRect.left - picRect.width * 0.25;
            clonedPic.style.transform =
                "translateY(" + offsetTop + "px) translateX(" + offsetLeft + "px) scale(0.5)";
            clonedPic.style.opacity = "0"; // Fading out the card
        });

        clonedPic.addEventListener("transitionend", () => {
            clonedPic.remove();
        });
    }

    scaleUpPrice() {
        const priceElement = document.querySelector(".total-price");

        if (!priceElement) {
            return;
        }

        priceElement.classList.add("scale-up");

        setTimeout(() => {
            priceElement.classList.remove("scale-up");
        }, 600);
    }

    async selectProduct(qty = 1) {
        const product = this.props.product;

        if (!product.self_order_available) {
            return;
        }

        if (product.isCombo) {
            this.router.navigate("combo_selection", { id: product.id });
        } else if (product.attributes.length > 0) {
            this.router.navigate("product", { id: product.id });
        } else {
            if (!this.selfOrder.ordering) {
                return;
            }
            this.flyToCart();
            this.scaleUpPrice();
            const isProductInCart = this.selfOrder.currentOrder.lines.find(
                (line) => line.product_id === product.id
            );

            if (isProductInCart) {
                isProductInCart.qty += qty;
            } else {
                const lines = this.selfOrder.currentOrder.lines;
                const line = new Line({
                    id: null,
                    uuid: null,
                    qty: qty,
                    product_id: product.id,
                });
                line.full_product_name = constructFullProductName(
                    line,
                    this.selfOrder.attributeValueById,
                    product.name
                );
                lines.push(line);
            }
            await this.selfOrder.getPricesFromServer();
        }
    }

    showProductInfo() {
        this.dialog.add(ProductInfoPopup, {
            product: this.props.product,
            title: this.props.product.name,
            addToCart: (qty) => {
                this.selectProduct(qty);
            },
        });
    }
}

```

## File: static\src\app\components\product_card\product_card.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ProductCard">
        <article class="self_order_product_card d-flex flex-row-reverse flex-md-column align-items-start gap-2 user-select-none"
            role="button"
            t-att-title="props.product.name"
            t-on-click="() => this.selectProduct()"
            t-ref="selfProductCard">
            <div t-if="props.product.description_self_order" class="product-information-tag" t-on-click.prevent.stop="showProductInfo">
                <i class="product-information-tag-logo fa fa-info fs-4" role="img" aria-label="Product Information" title="Product Information" />
            </div>
            <div
                class="ratio ratio-1x1 w-25 w-sm-50 w-md-100"
                t-att-class="{
                    'd-md-block': !props.product.has_image
                }">
                <div class="placeholder-glow o_self_order_item_card_no_image">
                    <div t-attf-class="{{ props.product.has_image ? 'placeholder' : 'd-flex align-items-center justify-content-center h-100' }} bg-200 w-100 h-100 rounded">
                        <span t-if="!props.product.has_image" t-esc="props.product.name" class="text-center text-white fs-2 fw-bold mb-1 mb-sm-2"/>
                    </div>
                </div>
                <img
                    t-if="props.product.has_image"
                    class="o_self_order_item_card_image w-100 rounded"
                    t-attf-src="/menu/get-image/{{ props.product.id }}/512?unique={{props.product.write_date}}"
                    alt="Product image"
                    loading="lazy"
                    onerror="this.remove()"/>
            </div>
            <div class="product-infos d-flex flex-column justify-content-between text-start flex-grow-1 w-100 lh-1">
                <span t-esc="props.product.name" class="fs-4 fw-bold mb-1 mb-sm-2"/>
                <div class="d-flex justify-content-between align-items-end gap-3">
                    <span t-esc="selfOrder.formatMonetary(selfOrder.getProductDisplayPrice(props.product))" class="o-so-tabular-nums fs-4 text-muted flex-grow-1" />
                    <div class="text-center ms-2 fs-lighter">
                        <div t-if="!props.product.self_order_available" class="fs-lighter bg-secondary rounded">Out of stock</div>
                    </div>
                </div>
            </div>
        </article>
    </t>
</templates>

```

## File: static\src\app\components\product_info_popup\product_info_popup.js

```javascript
/** @odoo-module */

import { Component, useExternalListener, useState } from "@odoo/owl";

export class ProductInfoPopup extends Component {
    static template = "pos_self_order.ProductInfoPopup";
    static props = ["product", "addToCart"];

    setup() {
        useExternalListener(window, "click", this.props.close);
        this.state = useState({
            qty: 1,
        });
    }

    addToCartAndClose() {
        this.props.addToCart(this.state.qty);
        this.props.close();
    }

    changeQuantity(increase) {
        const currentQty = this.state.qty;

        if (!increase && currentQty === 1) {
            return;
        }

        return increase ? this.state.qty++ : this.state.qty--;
    }
}

```

## File: static\src\app\components\product_info_popup\product_info_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ProductInfoPopup">
        <div class="self_order_product_info_popup o_dialog" t-att-id="id">
            <div role="dialog" class="modal d-block" tabindex="-1">
                <div class="modal-dialog" role="document" t-on-click.stop="">
                    <div class="modal-content rounded">
                        <div class="modal-header">
                            <h1 class="modal-title fw-bolder" t-esc="props.product.name"/>
                            <button type="button" class="btn-close" t-on-click.stop="() => this.props.close()"></button>
                        </div>
                        <span class="modal-body o_self_order_main_desc fs-3 p-4 ps-5 overflow-auto" t-out="props.product.description_self_order" />
                        <div class="modal-footer d-flex flex-row-reverse justify-content-between align-items-center">
                            <button type="button" class="btn btn-primary" t-on-click.stop="() => this.addToCartAndClose()">Add to Cart</button>
                            <div t-if="!props.product.isCombo and !props.product.attributes.length > 0" class="o_self_order_incr_button btn-group " role="group" aria-label="Quantity select" >
                                <button type="button"
                                    t-on-click = "() => this.changeQuantity(false)"
                                    class="btn btn-secondary"><span class="fs-2 lh-1 fa-fw d-inline-block">－</span></button>
                                <div class="o-so-tabular-nums d-flex align-items-center px-3 text-bg-200" t-esc="state.qty"/>
                                <button type="button"
                                    t-on-click = "() => this.changeQuantity(true)"
                                    class="btn btn-secondary"><span class="fs-2 lh-1 fa-fw d-inline-block">＋</span></button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\models\combo.js

```javascript
/** @odoo-module **/
import { Reactive } from "@web/core/utils/reactive";
import { ComboLine } from "./combo_line";

export class Combo extends Reactive {
    constructor({ id, name, combo_line_ids }) {
        super();
        this.setup(...arguments);
    }

    setup(combo) {
        // server only data (recovered after first send to server)
        this.id = combo.id || null;
        this.name = combo.name || null;
        this.combo_line_ids = combo.combo_line_ids || [];
        this.initLines();
    }

    initLines() {
        this.combo_line_ids = this.combo_line_ids.map((line) => new ComboLine(line));
    }
}

```

## File: static\src\app\models\combo_line.js

```javascript
/** @odoo-module **/
import { Reactive } from "@web/core/utils/reactive";

export class ComboLine extends Reactive {
    constructor({ id, lst_price, combo_price, product_id, combo_id }) {
        super();
        this.setup(...arguments);
    }

    setup(comboLine) {
        // server only data (recovered after first send to server)
        this.id = comboLine.id || null;
        this.lst_price = comboLine.lst_price || null;
        this.combo_price = comboLine.combo_price || null;
        this.product_id = comboLine.product_id || null;
        this.combo_id = comboLine.combo_id || null;
    }
}

```

## File: static\src\app\models\line.js

```javascript
/** @odoo-module **/
import { Reactive } from "@web/core/utils/reactive";
import { uuidv4 } from "@point_of_sale/utils";
import { ProductCustomAttribute } from "@point_of_sale/app/store/models/product_custom_attribute";

export class Line extends Reactive {
    constructor({
        id,
        uuid,
        product_id,
        qty,
        customer_note,
        full_product_name,
        price_unit,
        price_subtotal_incl,
        price_subtotal,
        attribute_value_ids,
        custom_attribute_value_ids,
        combo_parent_uuid,
        combo_id,
        combo_line_id,
        child_lines,
    }) {
        super();
        this.setup(...arguments);
    }

    setup(line) {
        // server data
        this.id = line.id || null;
        this.uuid = line.uuid || uuidv4();
        this.full_product_name = line.full_product_name || "";
        this.product_id = line.product_id;
        this.qty = line.qty ? line.qty : 0;
        this.customer_note = line.customer_note || "";
        this.price_unit = line.price_unit || 0;
        this.price_subtotal_incl = line.price_subtotal_incl || 0;
        this.price_subtotal = line.price_subtotal || 0;
        this.attribute_value_ids = line.attribute_value_ids || [];
        this.custom_attribute_value_ids = line.custom_attribute_value_ids || [];
        this.combo_parent_uuid = line.combo_parent_uuid || null;
        this.combo_id = line.combo_id || null;
        this.combo_line_id = line.combo_line_id || null;
        this.child_lines = line.child_lines || [];

        this.initCustomAttribute();
    }
    initCustomAttribute() {
        this.custom_attribute_value_ids = this.custom_attribute_value_ids.map(
            (customAttribute) => new ProductCustomAttribute(customAttribute)
        );
    }

    isChange(lastChange) {
        for (const key in lastChange) {
            if (JSON.stringify(lastChange[key]) !== JSON.stringify(this[key])) {
                return true;
            }
        }

        return false;
    }

    get attributes() {
        return Object.entries(this.attribute_value_ids).map(([key, value]) => {
            return { name: key, value };
        });
    }

    updateDataFromServer(data) {
        for (const key in data) {
            let updatedValue = data[key];
            if (key === "attribute_value_ids") {
                updatedValue ||= {};
            }

            this[key] = updatedValue;
        }
    }
}

```

## File: static\src\app\models\order.js

```javascript
/** @odoo-module **/
import { Reactive } from "@web/core/utils/reactive";
import { Line } from "./line";
import { random5Chars } from "@point_of_sale/utils";

export class Order extends Reactive {
    constructor({
        id,
        access_token,
        pos_config_id,
        pos_reference,
        state,
        lines,
        date,
        amount_total,
        amount_tax,
        lastChangesSent,
        take_away,
        tax_details,
    }) {
        super();
        this.setup(...arguments);
    }

    setup(order) {
        // server only data (recovered after first send to server)
        this.id = order.id || null;
        this.pos_config_id = order.pos_config_id;
        this.access_token = order.access_token || null;
        this.pos_reference = order.pos_reference || null;
        this.state = order.state || "draft";
        this.date = order.date || null;
        this.amount_total = order.amount_total || 0;
        this.amount_tax = order.amount_tax || 0;
        this.lines = order.lines || [];
        this.take_away = typeof order.take_away === "boolean" ? order.take_away : null;
        this.ticket_code = random5Chars(); // 5-digits alphanum code shown on the receipt
        this.tax_details = order.tax_details || [];

        // data
        this.lastChangesSent = order.lastChangesSent || {};

        this.initLines();
    }

    get isAlreadySent() {
        return this.id && this.access_token && this.pos_reference ? true : false;
    }

    get totalQuantity() {
        return this.lines.reduce((acc, line) => (acc += line.qty), 0);
    }

    get isSavedOnServer() {
        return this.isAlreadySent && this.hasNotAllLinesSent().length === 0;
    }

    get trackingNumber() {
        if (this.pos_reference) {
            const reference = this.pos_reference;
            const arrRef = reference.split(" ")[1].split("-");
            const sessionID = arrRef[0][4];
            const sequence = arrRef[2].substr(2, 2);
            const trackingNumber = "S" + sessionID + sequence;
            return trackingNumber;
        }
        return null;
    }

    initLines() {
        this.lines = this.lines.map((line) => new Line(line));
    }

    removeLine(lineUuid) {
        this.lines = this.lines.filter((line) => line.uuid !== lineUuid);
        for (const line of this.lines) {
            if (line.combo_parent_uuid === lineUuid) {
                this.removeLine(line.uuid);
            }
        }
    }

    updateLastChanges() {
        for (const changeIdx in this.lastChangesSent) {
            const changeFound = this.lines.find((line) => line.uuid === changeIdx);
            if (!changeFound) {
                delete this.lastChangesSent[changeIdx];
            }
        }

        this.lastChangesSent = this.lines.reduce((acc, line) => {
            acc[line.uuid] = {
                qty: line.qty,
                attribute_value_ids: [...line.attribute_value_ids],
                customer_note: line.customer_note,
            };
            return acc;
        }, {});
    }

    hasNotAllLinesSent() {
        return this.lines.filter((line) => {
            const lastSend = this.lastChangesSent[line.uuid];

            if (!lastSend) {
                return true;
            }

            return lastSend.qty !== line.qty || line.isChange(lastSend);
        });
    }

    updateDataFromServer(data) {
        for (const key in data) {
            if (key !== "lines") {
                this[key] = data[key];
            }
        }

        for (const line of this.lines) {
            const lineFound = data.lines.find((l) => l.uuid === line.uuid);

            if (lineFound) {
                line.updateDataFromServer(lineFound);
            } else if (line.id) {
                this.removeLine(line.uuid);
            }
        }

        for (const line of data.lines) {
            const lineFound = this.lines.find((l) => l.uuid === line.uuid);

            if (!lineFound && line.product_id) {
                this.lines.push(new Line(line));
            }
        }
    }
}

```

## File: static\src\app\models\product.js

```javascript
/** @odoo-module **/
import { Reactive } from "@web/core/utils/reactive";

export class Product extends Reactive {
    constructor(
        {
            price_info,
            has_image,
            attributes,
            name,
            id,
            description_self_order,
            pos_categ_ids,
            pos_combo_ids,
            is_pos_groupable,
            write_date,
            self_order_available,
        },
        showPriceTaxIncluded
    ) {
        super();
        this.setup(...arguments);
    }

    setup(product, showPriceTaxIncluded) {
        // server data
        this.id = product.id;
        this.price_info = product.price_info;
        this.has_image = product.has_image;
        this.attributes = product.attributes;
        this.name = product.name;
        this.description_self_order = product.description_self_order;
        this.pos_categ_ids = product.pos_categ_ids;
        this.pos_combo_ids = product.pos_combo_ids;
        this.is_pos_groupable = product.is_pos_groupable;
        this.write_date = product.write_date;
        this.self_order_available = product.self_order_available;
        this.barcode = product.barcode;

        // data
        this.showPriceTaxIncluded = showPriceTaxIncluded;
    }

    get isCombo() {
        return this.pos_combo_ids;
    }
}

```

## File: static\src\app\pages\cart_page\cart_page.js

```javascript
/** @odoo-module */

import { Component, useState } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { PopupTable } from "@pos_self_order/app/components/popup_table/popup_table";
import { _t } from "@web/core/l10n/translation";
import { OrderWidget } from "@pos_self_order/app/components/order_widget/order_widget";

export class CartPage extends Component {
    static template = "pos_self_order.CartPage";
    static components = { PopupTable, OrderWidget };

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        this.state = useState({
            selectTable: false,
            cancelConfirmation: false,
        });
    }

    get lines() {
        const lines = this.selfOrder.currentOrder.lines;
        return lines ? lines : [];
    }

    get linesToDisplay() {
        const selfOrder = this.selfOrder;
        const order = selfOrder.currentOrder;

        if (selfOrder.config.self_ordering_pay_after === "meal" && !order.isSavedOnServer) {
            return order.hasNotAllLinesSent();
        } else {
            return this.lines;
        }
    }

    getLineChangeQty(line) {
        const currentQty = line.qty;
        const lastChange = this.selfOrder.currentOrder.lastChangesSent[line.uuid];
        return !lastChange ? currentQty : currentQty - lastChange.qty;
    }

    backToMenu() {
        this.router.navigate("product_list");
    }

    async pay() {
        const orderingMode = this.selfOrder.config.self_ordering_service_mode;
        const type = this.selfOrder.config.self_ordering_mode;
        const takeAway = this.selfOrder.currentOrder.take_away;

        if (
            this.selfOrder.rpcLoading ||
            !this.selfOrder.verifyCart() ||
            !this.selfOrder.verifyPriceLoading()
        ) {
            return;
        }

        if (type === "mobile" && orderingMode === "table" && !takeAway && !this.selfOrder.table) {
            this.state.selectTable = true;
            return;
        }

        this.selfOrder.rpcLoading = true;
        await this.selfOrder.confirmOrder();
        this.selfOrder.rpcLoading = false;
    }

    selectTable(table) {
        if (table) {
            this.selfOrder.table = table;
            this.router.addTableIdentifier(table);
            this.pay();
        }

        this.state.selectTable = false;
    }

    getChildLines(line) {
        return this.lines.filter((l) => l.combo_parent_uuid === line.uuid);
    }

    getPrice(line) {
        const childLines = this.getChildLines(line);
        if (childLines.length == 0) {
            return line.price_subtotal_incl;
        } else {
            let price = 0;
            for (const child of childLines) {
                price += child.price_subtotal_incl;
            }
            return price;
        }
    }

    canChangeQuantity(line) {
        const order = this.selfOrder.currentOrder;
        const lastChange = order.lastChangesSent[line.uuid];

        if (!lastChange) {
            return true;
        }

        return lastChange.qty < line.qty;
    }

    canDeleteLine(line) {
        const lastChange = this.selfOrder.currentOrder.lastChangesSent[line.uuid];
        return !lastChange ? true : lastChange.qty !== line.qty;
    }

    async removeLine(line) {
        const lastChange = this.selfOrder.currentOrder.lastChangesSent[line.uuid];

        if (!this.canDeleteLine(line)) {
            return;
        }

        if (lastChange) {
            line.qty = lastChange.qty;
        } else {
            this.selfOrder.currentOrder.removeLine(line.uuid);
        }

        await this.selfOrder.getPricesFromServer();
    }

    async _changeQuantity(line, increase) {
        if (!increase && !this.canChangeQuantity(line)) {
            return;
        }

        if (!increase && line.qty === 1) {
            this.removeLine(line.uuid);
            return;
        }
        increase ? line.qty++ : line.qty--;
        for (const cline of this.selfOrder.currentOrder.lines) {
            if (cline.combo_parent_uuid === line.uuid) {
                this._changeQuantity(cline, increase);
            }
        }
    }

    async changeQuantity(line, increase) {
        await this._changeQuantity(line, increase);
        await this.selfOrder.getPricesFromServer();
    }

    clickOnLine(line) {
        const order = this.selfOrder.currentOrder;
        this.selfOrder.editedLine = line;

        if (order.state === "draft" && !order.lastChangesSent[line.uuid]) {
            this.selfOrder.editedOrder = order;

            if (line.child_lines.length > 0) {
                this.router.navigate("combo_selection", { id: line.product_id });
            } else {
                this.router.navigate("product", { id: line.product_id });
            }
        } else {
            this.selfOrder.notification.add(_t("You cannot edit a posted orderline !"), {
                type: "danger",
            });
        }
    }
}

```

## File: static\src\app\pages\cart_page\cart_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.CartPage">
        <div class="order-cart-content d-flex flex-column flex-grow-1 justify-content-between overflow-y-auto">
            <div class="d-flex align-items-center flex-shrink-0 justify-content-between gap-3 p-3 w-100 bg-view border-bottom overflow-x-auto z-index-1">
                <h1 class="mb-0 fw-bolder text-nowrap">Your Order</h1>
            </div>
            <div class="order-content flex-grow-1 overflow-auto pb-4">
                <t t-foreach="linesToDisplay" t-as="line" t-key="line.uuid">
                    <div t-if="!line.combo_parent_uuid" class="product-card-item py-2 py-lg-4 px-3 bg-view border-bottom">
                        <t t-set="childLines" t-value="getChildLines(line)"/>
                        <t t-set="product" t-value="this.selfOrder.productByIds[line.product_id]"/>

                        <div class="product-wrapper d-flex align-items-start gap-2 gap-lg-3">
                            <div class="product-info d-flex flex-column gap-2 flex-grow-1">
                                <div>
                                    <span t-if="selfOrder.currentOrder.isSavedOnServer"><t t-esc="line.qty" />x </span>
                                    <strong t-esc="selfOrder.productByIds[line.product_id].name"/>
                                </div>
                                <div t-if="this.selfOrder.productByIds[line.product_id]?.attributes.length > 0 || childLines.length > 0" class="d-flex align-items-start gap-2 gap-md-3 my-2">
                                    <button
                                        t-if="!selfOrder.currentOrder.isSavedOnServer"
                                        type="button"
                                        t-on-click="() => this.clickOnLine(line)"
                                        class="btn btn-secondary"> <i class="fa fa-pencil"></i></button>

                                    <div class="vr"/>

                                    <div class="small">
                                        <div t-if="childLines.length === 0" class="product-info d-flex flex-column flex-grow-1 text-muted">
                                            <div t-foreach="selfOrder.getSelectedAttributes(line)" t-as="attribute" t-key="attribute.name">
                                                - <span t-esc="attribute.name" /> : <span t-esc="attribute.value" />
                                            </div>
                                        </div>
                                        <div t-foreach="childLines" t-as="cline" t-key="cline.uuid" class="text-muted">
                                            - <span t-esc="selfOrder.productByIds[cline.product_id].name" /> <span t-if="cline.price_subtotal_incl">(<t t-esc="selfOrder.formatMonetary(cline.price_subtotal_incl)"/>)</span>
                                            <div class="product-info d-flex flex-column flex-grow-1 ms-3">
                                                <div t-foreach="selfOrder.getSelectedAttributes(cline)" t-as="attribute" t-key="attribute.name">
                                                    - <span t-esc="attribute.name" /> : <span t-esc="attribute.value" />
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div class="o_self_order_item_card_image_wrapper flex-shrink-0 ratio ratio-1x1">
                                <img
                                    class="o_self_order_item_card_image w-100 w-100 rounded bg-view"
                                    t-attf-src="/menu/get-image/{{ line.product_id}}/512"
                                    alt="Product image"
                                    loading="lazy"
                                    onerror="this.remove()"/>
                            </div>
                        </div>

                        <div class="product-controllers d-flex flex-wrap flex-grow-1 align-items-center justify-content-between mt-2">
                            <div>
                                <div t-if="!selfOrder.currentOrder.isSavedOnServer" class="btn-group">
                                    <button
                                        t-if="canDeleteLine(line) &amp;&amp; getLineChangeQty(line) > 1"
                                        type="button"
                                        t-on-click= "() => this.changeQuantity(line, false)"
                                        t-attf-class="btn btn-secondary px-3"><span class="fs-2 fa-fw d-inline-block">－</span></button>
                                    <button
                                        t-else=""
                                        type="button"
                                        t-on-click= "() => this.removeLine(line)"
                                        t-attf-class="btn btn-secondary px-3"><i class="fa fa-fw fa-trash-o"/></button>

                                    <div class="o-so-tabular-nums d-flex align-items-center fw-bold px-2 px-md-3 text-bg-200" t-esc="getLineChangeQty(line)"/>
                                    <button type="button"
                                        t-on-click = "() => this.changeQuantity(line, true)"
                                        class="btn btn-secondary px-3"><span class="fs-2 fa-fw d-inline-block">＋</span></button>
                                </div>
                            </div>
                            <div class="line-price o-so-tabular-nums" t-esc="selfOrder.formatMonetary(getPrice(line))"/>
                        </div>

                    </div>
                </t>
            </div>
        </div>
        <div class="order-price d-flex-column flex-grow-0">
            <div class="absolute-content-price d-flex flex-column flex-grow-0 justify-content-center py-2 py-lg-3 px-3 border-bottom border-top bg-view text-end lh-sm">
                <p class="o-so-tabular-nums mb-0 fw-bolder">Total: <t t-esc="selfOrder.formatMonetary(selfOrder.currentOrder.amount_total)" /></p>
                <p class="o-so-tabular-nums mb-0 text-muted">Taxes: <t t-esc="selfOrder.formatMonetary(selfOrder.currentOrder.amount_tax)" /></p>
            </div>
            <OrderWidget t-if="this.selfOrder.ordering" action.bind="pay" removeTopClasses="true"/>
        </div>
        <PopupTable t-if="this.state.selectTable" selectTable.bind="selectTable" />
    </t>
</templates>

```

## File: static\src\app\pages\combo_page\combo_page.js

```javascript
/** @odoo-module */

import { Component, onWillUnmount, useState, useSubEnv } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { ComboSelection } from "@pos_self_order/app/components/combo_selection/combo_selection";
import { useService } from "@web/core/utils/hooks";
import { Line } from "@pos_self_order/app/models/line";
import { attributeFlatter, attributeFormatter } from "@pos_self_order/app/utils";
import { constructFullProductName } from "@point_of_sale/utils";

export class ComboPage extends Component {
    static template = "pos_self_order.ComboPage";
    static props = ["product"];
    static components = { ComboSelection };

    setup() {
        this.selfOrder = useSelfOrder();
        this.selfOrder.lastEditedProductId = this.props.product.id;
        this.router = useService("router");
        useSubEnv({
            selectedValues: {},
            customValues: {},
            editable: this.editableProductLine,
            currentComboLineId: {
                value: null,
            },
        });

        if (!this.props.product) {
            this.router.navigate("product_list");
            return;
        }

        this.state = useState({
            currentComboIndex: 0,
            selectedCombos: [],
            showResume: false,
            selectedProduct: null,
            showQtyButtons: false,
            editMode: false,
            qty: 1,
        });

        onWillUnmount(() => {
            this.selfOrder.editedLine = null;
        });
    }

    get editableProductLine() {
        const order = this.selfOrder.currentOrder;
        return !(
            this.selfOrder.editedLine &&
            this.selfOrder.editedLine.uuid &&
            order.lastChangesSent[this.selfOrder.editedLine.uuid]
        );
    }

    get currentComboId() {
        return this.props.product.pos_combo_ids[this.state.currentComboIndex];
    }

    get currentCombo() {
        return this.selfOrder.comboByIds[this.currentComboId];
    }

    getAttributeSelected(combo) {
        const flatAttribute = attributeFlatter(combo.variants);
        const customAttribute = combo.customValues;
        return attributeFormatter(this.selfOrder.attributeById, flatAttribute, customAttribute);
    }

    resetState() {
        this.state.selectedProduct = null;
        this.state.showQtyButtons = false;

        // Cannot assign to read only property
        for (const key in this.env.selectedValues) {
            delete this.env.selectedValues[key];
        }

        for (const key in this.env.customValues) {
            delete this.env.customValues[key];
        }
    }

    next() {
        const combo = this.currentCombo;
        const index = this.state.selectedCombos.findIndex((c) => c.id === combo.id);
        const selectedCombo = {
            id: combo.id,
            name: combo.name,
            combo_line_id: this.env.currentComboLineId.value,
            product: {
                id: this.state.selectedProduct.id,
                name: this.state.selectedProduct.name,
                variants: { ...this.env.selectedValues },
                customValues: { ...this.env.customValues },
            },
        };
        if (index !== -1) {
            this.state.selectedCombos[index] = selectedCombo;
        } else {
            this.state.selectedCombos.push(selectedCombo);
        }
        this.resetState();
        if (this.state.editMode) {
            this.state.editMode = false;
            this.state.showResume = true;
            this.state.showQtyButtons = true;
            return;
        }
        this.state.currentComboIndex++;
        if (this.state.currentComboIndex == this.props.product.pos_combo_ids.length) {
            this.state.showResume = true;
        }
    }

    back() {
        this.router.navigate("product_list");
    }

    changeQuantity(increase) {
        if (!increase && this.state.qty === 1) {
            return;
        }

        return increase ? this.state.qty++ : this.state.qty--;
    }

    async addToCart() {
        if (this.selfOrder.editedLine) {
            this.selfOrder.currentOrder.removeLine(this.selfOrder.editedLine.uuid);
        }

        const lines = this.selfOrder.currentOrder.lines;
        const parent_line = new Line({
            id: null,
            uuid: null,
            qty: this.state.qty,
            price_unit: this.props.product.price_info.price_without_tax,
            product_id: this.props.product.id,
            full_product_name: this.props.product.name,
            attribute_value_ids: [],
            custom_attribute_value_ids: [],
            combo_parent_uuid: null,
            combo_id: null,
        });
        lines.push(parent_line);
        for (const combo of this.state.selectedCombos) {
            const child_line = new Line({
                id: null,
                uuid: null,
                qty: this.state.qty,
                product_id: combo.product.id,
                attribute_value_ids: attributeFlatter(combo.product.variants),
                custom_attribute_value_ids: Object.values(combo.product.customValues),
                combo_parent_uuid: parent_line.uuid,
                combo_id: combo.id,
                combo_line_id: combo.combo_line_id,
            });
            child_line.full_product_name = constructFullProductName(
                child_line,
                this.selfOrder.attributeValueById,
                combo.product.name
            );
            lines.push(child_line);
            parent_line.child_lines.push(child_line);
        }

        await this.selfOrder.getPricesFromServer();
        this.router.back();
    }

    editCombo(combo_id) {
        this.state.currentComboIndex = this.state.selectedCombos.findIndex(
            (c) => c.id === combo_id
        );
        this.state.showResume = false;
        this.state.editMode = true;
        this.state.showQtyButtons = false;
    }

    get showQtyButtons() {
        return this.state.showQtyButtons && this.props.product.self_order_available;
    }
}

```

## File: static\src\app\pages\combo_page\combo_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ComboPage">
        <div class="d-flex align-items-center flex-shrink-0 justify-content-between gap-3 px-3 py-2 w-100 bg-view border-bottom overflow-x-auto z-index-1">
            <button class="btn btn-secondary btn-lg px-3 text-nowrap" t-on-click="() => this.router.back()">
                <i class="oi oi-chevron-left" aria-hidden="true"/><span class="ms-2 d-none d-md-inline">Discard</span>
            </button>
            <h1 class="mb-0 fw-bolder text-nowrap" t-esc="props.product.name"/>
            <span class="d-none d-md-inline px-5"/> <!-- Spacer -->
        </div>


        <div class="d-flex flex-column overflow-y-auto bg-view flex-grow-1">
            <section t-if="!state.showResume" class="pos_self_order_breadcrumb position-relative px-3 px-md-0 pb-md-4 bg-view">
                <div class="d-flex justify-content-around my-5">
                    <div t-foreach="props.product.pos_combo_ids" t-as="combo_id" t-key="combo_id" class="position-relative">
                        <t t-set="isComboCurrent" t-value="currentComboId === combo_id"/>
                        <t t-set="isComboSelected" t-value="false"/>
                        <t t-set="isNextSelected" t-value="false"/>
                        <t t-foreach="state.selectedCombos"  t-as="selected"  t-key="selected.id">
                            <t t-if="selected.id === combo_id" t-set="isComboSelected" t-value="true"/>
                        </t>

                        <span
                            t-if="!combo_id_first"
                            class="position-absolute end-0 top-50 border-top border-2"
                            t-attf-style="width: calc(100vw / {{
                                props.product.pos_combo_ids.length}})"
                            />

                        <div class="pos_self_order_breadcrumb_pill d-flex align-items-center justify-content-center ratio ratio-1x1 mx-md-auto">
                            <span
                                class="rounded-pill d-flex justify-content-center align-items-center border border-2 z-index-1"
                                t-att-class="{
                                    'text-bg-primary border-primary' : isComboCurrent,
                                    'bg-view' : !isComboCurrent,
                                    'border-success' : isComboSelected &amp;&amp; !isComboCurrent
                                }">
                                    <t t-esc="combo_id_index + 1"/>
                                    <i t-if="isComboSelected &amp;&amp; !isComboCurrent" class="position-absolute top-0 start-100 translate-middle-x fa fa-check fs-4 bg-view text-success"/>
                            </span>
                        </div>
                        <div class="position-absolute start-50 d-none d-md-block mt-3 translate-middle text-nowrap" t-esc="selfOrder.comboByIds[combo_id].name" />
                    </div>
                </div>
            </section>
            <t t-if="state.selectedProduct">
                <div class="o-so-product-details d-flex flex-row align-items-start p-3 gap-3">
                    <div class="o-so-product-details-image ratio ratio-1x1 w-25 flex-shrink-0">
                        <div class="placeholder-glow">
                            <div class="placeholder w-100 h-100 bg-300 rounded"/>
                        </div>
                        <img
                            class="o_self_order_item_card_image w-100 rounded"
                            t-attf-src="/menu/get-image/{{ state.selectedProduct.id }}/512?unique={{state.selectedProduct.write_date}}"
                            alt="Product image"
                            loading="lazy"
                            onerror="this.remove()"/>
                    </div>
                    <div class="o-so-product-details-description">
                        <h2 t-esc="state.selectedProduct.name"/>
                        <small t-if="state.selectedProduct.description_self_order"
                            class="o_self_order_main_desc d-block mb-3 text-muted"
                            t-out="state.selectedProduct.description_self_order"
                        />
                        <span class="fs-3" t-esc="selfOrder.formatMonetary(selfOrder.getProductDisplayPrice(state.selectedProduct))"/>
                    </div>
                </div>
            </t>
            <div t-if="!state.showResume" class="d-flex flex-column flex-grow-1" t-attf-class="{{ state.selectedProduct ? '' : '' }}">
                <ComboSelection combo="currentCombo" comboState="state" next.bind="next"/>
            </div>
            <div t-else="" t-attf-class="o_kiosk-combo d-flex flex-column flex-grow-1 px-3">
                <h2 class="attribute_name mt-5 mb-3 fw-bold">Your Selection</h2>

                <ul class="list-group">
                    <li class="list-group-item d-flex flex-wrap align-items-start gap-3" t-foreach="state.selectedCombos" t-as="combo" t-key="combo.id">
                        <t t-set="product" t-value="this.selfOrder.productByIds[combo.product.id]"/>
                        <div class="d-flex align-items-start gap-3">
                            <div class="pos_self_order_combo_image ratio ratio-1x1">
                                <div class="placeholder-glow">
                                    <div class="placeholder w-100 h-100 bg-300 rounded"/>
                                </div>
                                <img
                                    class="o_self_order_item_card_image w-100 rounded"
                                    t-attf-src="/menu/get-image/{{ combo.product.id }}/512?unique={{product.write_date}}"
                                    alt="Product image"
                                    loading="lazy"
                                    onerror="this.remove()"/>
                            </div>
                            <div>
                                <span class="fs-4" t-esc="combo.product.name"/>
                                <ul t-if="getAttributeSelected(combo.product)">
                                    <li t-foreach="getAttributeSelected(combo.product)" t-as="attribute" t-key="attribute.id">
                                        <span t-esc="attribute.name" /> : <span t-esc="attribute.value" />
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <button class="btn btn-secondary ms-auto" t-on-click="() => this.editCombo(combo.id)">Edit</button>

                    </li>
                </ul>
            </div>
            <t t-if="state.showResume">
                <div class="bg-view p-3 text-end">
                    <div t-if="selfOrder.ordering" class="o_self_order_incr_button btn-group" role="group" aria-label="Quantity select">
                        <button type="button"
                            t-on-click = "() => this.changeQuantity(false)"
                            t-attf-class="btn btn-secondary btn-lg"><span class="fs-2 lh-1 fa-fw d-inline-block">－</span></button>
                        <div class="o-so-tabular-nums d-flex align-items-center px-3 text-bg-200" t-esc="state.qty"/>
                        <button type="button"
                            t-on-click = "() => this.changeQuantity(true)"
                            class="btn btn-secondary btn-lg"><span class="fs-2 lh-1 fa-fw d-inline-block">＋</span></button>
                    </div>
                </div>
            </t>
        </div>
        <div
            t-if="state.showResume || (!state.showResume and showQtyButtons)"
            class="page-buttons d-flex justify-content-end gap-3 p-3 border-top bg-view">
            <button t-if="!state.showResume and showQtyButtons" class="btn btn-primary btn-lg" t-on-click="next">Next</button>
            <button t-if="state.showResume and selfOrder.ordering" class="btn btn-primary btn-lg" t-on-click="addToCart">Add to cart</button>
        </div>
    </t>
</templates>

```

## File: static\src\app\pages\confirmation_page\confirmation_page.js

```javascript
/** @odoo-module */

import { Component, onMounted, useState } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { cookie } from "@web/core/browser/cookie";
import { useService } from "@web/core/utils/hooks";
import { OrderReceipt } from "@point_of_sale/app/screens/receipt_screen/receipt/order_receipt";

export class ConfirmationPage extends Component {
    static template = "pos_self_order.ConfirmationPage";
    static props = ["orderAccessToken", "screenMode"];

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        this.printer = useService("printer");
        this.confirmedOrder = {};
        this.changeToDisplay = [];
        this.state = useState({
            onReload: false,
            payment: this.props.screenMode === "pay",
        });

        onMounted(() => {
            if (this.selfOrder.config.self_ordering_mode === "kiosk") {
                setTimeout(() => {
                    this.setDefautLanguage();
                }, 5000);

                setTimeout(async () => {
                    try {
                        await this.printer.print(OrderReceipt, {
                            data: this.selfOrder.export_for_printing(this.confirmedOrder),
                            formatCurrency: this.selfOrder.formatMonetary,
                        });
                    } catch (e) {
                        console.error(e);
                    }
                }, 500);
            }
        });

        this.initOrder();
    }

    async initOrder() {
        const order = this.selfOrder.orders.find(
            (o) => o.access_token === this.props.orderAccessToken
        );

        const paymentMethods = this.selfOrder.pos_payment_methods.filter(
            (p) => p.is_online_payment
        );

        if (
            !order ||
            (paymentMethods.length > 0 &&
                this.selfOrder.config.self_ordering_mode === "mobile" &&
                this.selfOrder.config.self_ordering_pay_after === "each" &&
                order.state !== "paid")
        ) {
            this.router.navigate("default");
            return;
        }

        this.confirmedOrder = order;
    }

    backToHome() {
        if (!this.setDefautLanguage()) {
            this.router.navigate("default");
        }
    }

    setDefautLanguage() {
        const defaultLanguage = this.selfOrder.config.self_ordering_default_language_id;

        if (
            this.selfOrder.currentLanguage.code !== defaultLanguage.code &&
            !this.state.onReload &&
            this.selfOrder.config.self_ordering_mode === "kiosk"
        ) {
            cookie.set("frontend_lang", defaultLanguage.code);
            window.location.reload();
            this.state.onReload = true;
            return true;
        }

        return this.state.onReload;
    }
}

```

## File: static\src\app\pages\confirmation_page\confirmation_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ConfirmationPage">
        <div t-if="this.state.onReload" class="self_order_success_loader position-absolute vh-100 w-100 d-flex justify-content-center align-items-center opacity-50 bg-dark">
            <div class="page-loader d-flex justify-content-center align-items-center">
                <div class="spinner-border text-primary" role="status">
                    <span class="visually-hidden">Loading...</span>
                </div>
            </div>
        </div>
        <div class="d-flex justify-content-center align-items-center flex-column h-100 px-3 text-center">
            <h1 t-if="state.payment and selfOrder.config.self_ordering_pay_after !== 'each'" class="mb-4">Hope you enjoyed your meal!</h1>
            <h1 t-else="" class="mb-4">We're preparing your order!</h1>
            <h3 t-if="state.payment and confirmedOrder.state !== 'paid'" class="mt-3 text-muted">Pay at the cashier <t t-esc="selfOrder.formatMonetary(confirmedOrder.amount_total)" /></h3>
            <div class="d-inline-flex flex-column border rounded py-4 px-5 bg-view mb-3">
                <span class="fs-2 text-muted">Your order number</span>
                <span class="number lh-1" t-esc="confirmedOrder.trackingNumber" />
            </div>
            <h3 t-if="selfOrder.table.name || confirmedOrder.table_stand_number" class="text-muted mb-3">
                Service at table
                <t t-if="selfOrder.config.self_ordering_mode === 'mobile'" t-esc="selfOrder.table.name + ' (' + selfOrder.table.floor_name + ')'" />
                <t t-else="" t-esc="confirmedOrder.table_stand_number" />
            </h3>
            <h3 t-else="">Order to pick-up at the counter</h3>
        </div>
        <div class="px-3 py-4 text-center">
            <button class="btn btn-primary btn-lg" t-on-click="backToHome">
                <t t-if="this.selfOrder.config.self_ordering_mode === 'kiosk'">Close</t>
                <t t-else="">Ok</t>
            </button>
        </div>
    </t>
</templates>

```

## File: static\src\app\pages\eating_location_page\eating_location_page.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { useService } from "@web/core/utils/hooks";

export class EatingLocationPage extends Component {
    static template = "pos_self_order.EatingLocationPage";

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
    }

    back() {
        this.router.navigate("default");
    }

    selectLocation(loc) {
        this.selfOrder.currentOrder.take_away = loc === "out";
        this.router.navigate("product_list");
    }
}

```

## File: static\src\app\pages\eating_location_page\eating_location_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.EatingLocationPage">
        <div class="o_kiosk_eating_location d-flex flex-column align-items-center flex-grow-1 bg-300 bg-gradient overflow-y-auto text-center">
            <h1 class="pt-3 m-0">Choose your eating location</h1>
            <div class="container d-flex flex-wrap align-items-center justify-content-center gap-3 my-auto">
                <button t-on-click="() => this.selectLocation('in')" role="button" class="o_kiosk_eating_location_box btn btn-light">
                    <img src="/pos_self_order/static/img/eatin.svg" />
                    <h3>Eat In</h3>
                </button>
                <button t-on-click="() => this.selectLocation('out')" role="button" class="o_kiosk_eating_location_box o_kiosk_eating_location_away btn btn-light">
                    <img src="/pos_self_order/static/img/takeAway.svg" />
                    <h3>Take Out</h3>
                </button>
            </div>
        </div>
        <div class="page-buttons shadow-sm p-3 bg-view border-top text-center">
            <button class="btn btn-secondary btn-lg" t-on-click="back">
                <i class="oi oi-chevron-left"/>
                Back
            </button>
        </div>
    </t>
</templates>

```

## File: static\src\app\pages\landing_page\landing_page.js

```javascript
/** @odoo-module */
/* global Carousel */

import { Component, onMounted, onWillStart, onWillUnmount, useRef } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { useService } from "@web/core/utils/hooks";
import { LanguagePopup } from "@pos_self_order/app/components/language_popup/language_popup";

export class LandingPage extends Component {
    static template = "pos_self_order.LandingPage";

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        this.dialog = useService("dialog");
        this.carouselRef = useRef("carousel");
        this.activeSelected = false;
        this.carouselInterval = null;

        onWillStart(() => {
            if (this.selfOrder.config.self_ordering_mode === "kiosk") {
                this.selfOrder.orders = [];
                this.selfOrder.editedOrder = null;
            }
            this.selfOrder.rpcLoading = false;
        });

        onMounted(() => {
            if (this.selfOrder.config.self_ordering_image_home_ids.length > 1) {
                // used to init carousel after components mount / unmount
                const carousel = new Carousel(this.carouselRef.el);

                // prevent traceback when no image is set
                this.carouselInterval = setInterval(() => {
                    carousel.next();
                }, 5000);
            }
        });

        onWillUnmount(() => {
            clearInterval(this.carouselInterval);
        });
    }

    get currentLanguage() {
        return this.selfOrder.currentLanguage;
    }

    get languages() {
        return this.selfOrder.config.self_ordering_available_language_ids;
    }

    get activeImage() {
        if (!this.activeSelected) {
            this.activeSelected = true;
            return "active";
        }
        return "";
    }

    get draftOrder() {
        return this.selfOrder.orders.filter((o) => o.access_token && o.state === "draft");
    }

    hideBtn(link) {
        const arrayLink = link.url.split("/");
        const routeName = arrayLink[arrayLink.length - 1];

        if (routeName !== "products") {
            return;
        }

        return (
            this.draftOrder.length > 0 && this.selfOrder.config.self_ordering_pay_after === "each"
        );
    }

    clickMyOrder() {
        this.router.navigate(this.draftOrder.length > 0 ? "cart" : "orderHistory");
    }

    clickCustomLink(link) {
        const arrayLink = link.url.split("/");
        const routeName = arrayLink[arrayLink.length - 1];

        if (routeName !== "products") {
            this.router.customLink(link);
            return;
        }

        this.start();
    }

    start() {
        if (
            this.draftOrder.length > 0 &&
            this.selfOrder.config.self_ordering_pay_after === "each"
        ) {
            return;
        }

        if (
            this.selfOrder.config.self_ordering_takeaway &&
            this.selfOrder.currentOrder.take_away === null &&
            this.selfOrder.ordering
        ) {
            this.router.navigate("location");
        } else {
            this.router.navigate("product_list");
        }
    }

    openLanguages() {
        this.dialog.add(LanguagePopup);
    }

    showMyOrderBtn() {
        const ordersNotDraft = this.selfOrder.orders.find((o) => o.access_token);
        return this.selfOrder.ordering && ordersNotDraft;
    }
}

```

## File: static\src\app\pages\landing_page\landing_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.LandingPage">
        <div t-if="languages.length > 1" t-on-click="openLanguages" class="self_order_language_selector position-absolute top-0 end-0 m-4 rounded p-4 bg-white shadow-lg">
            <img class="rounded" t-attf-src="{{currentLanguage.flag_image_url}}" />
            <span t-esc="currentLanguage.display_name" class="ms-3"></span>
        </div>
        <div t-if="selfOrder.config.self_ordering_image_home_ids.length > 0" t-on-click="start" class="d-flex flex-column vh-100 align-items-center overflow-hidden">
            <div id="carouselAutoplaying" t-ref="carousel" class="carousel slide w-100 h-100" data-bs-ride="true">
                <div class="carousel-inner h-100 w-100">
                    <div
                        t-foreach="selfOrder.config.self_ordering_image_home_ids"
                        t-as="image"
                        t-key="image.id"
                        t-attf-class="carousel-item object-fit-cover h-100 w-100 {{activeImage}}"
                        t-attf-style="background-image: url('data:image/png;base64,{{image.data}}'); background-size: cover; background-position: center;" />
                </div>
            </div>
        </div>
        <div class="o_pos_landing_footer position-absolute bottom-0 end-0 d-flex w-100 gap-3 p-3">
            <div class="d-flex gap-3" t-att-class="{'flex-grow-1 justify-content-around': !selfOrder.custom_links.length}">
                <t t-if="showMyOrderBtn()">
                    <a
                        type="button"
                        t-on-click="clickMyOrder"
                        class="btn btn-lg btn-primary">
                        <t t-if="draftOrder.length > 0">
                            My Order
                        </t>
                        <t t-else="">
                            My Orders
                        </t>
                    </a>
                </t>
            </div>
            <div t-if="selfOrder.custom_links.length" class="d-flex gap-3 flex-grow-1">
                <t t-foreach="selfOrder.custom_links" t-as="link" t-key="link.id">
                    <a type="button"
                        t-if="!hideBtn(link)"
                        t-on-click="(event) => this.clickCustomLink(link)"
                        t-attf-class="btn btn-lg btn-{{link.style}}">
                        <t t-esc="link.name"/>
                    </a>
                </t>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\pages\order_history_page\order_history_page.js

```javascript
/** @odoo-module */

import { Component, useState } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
export class OrdersHistoryPage extends Component {
    static template = "pos_self_order.OrdersHistoryPage";

    async setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        this.state = useState({
            loadingProgress: true,
        });

        await this.loadOrder();
    }

    async loadOrder() {
        await this.selfOrder.getOrdersFromServer();
        this.state.loadingProgress = false;
    }

    get orders() {
        return this.selfOrder.orders.filter((o) => o.access_token).sort((a, b) => b.id - a.id);
    }

    get lines() {
        return this.order.lines;
    }

    getPrice(line) {
        return this.selfOrder.config.iface_tax_included
            ? line.price_subtotal_incl
            : line.price_subtotal;
    }

    getOrderState(state) {
        return state === "draft" ? _t("Current") : state;
    }

    getNameAndDescription(line) {
        const fullName = line.full_product_name;
        const regex = /\(([^()]+)\)[^(]*$/;
        const matches = fullName.match(regex);

        if (matches && matches.length > 1) {
            const attributes = matches[matches.length - 1].trim();
            const productName = fullName.replace(matches[0], "").trim();
            return { productName, attributes };
        }

        return { productName: fullName, attributes: "" };
    }

    editOrder(order) {
        if (order.state === "draft") {
            this.selfOrder.editedOrder = order;
            this.router.navigate("cart");
        }
    }

    back() {
        this.router.navigate("default");
    }
}

```

## File: static\src\app\pages\order_history_page\order_history_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.OrdersHistoryPage">
        <div class="overflow-auto h-100">
            <t t-if="state.loadingProgress">
                <div class="d-flex align-items-center h-100 justify-content-center">
                    <div class="spinner-border" role="status">
                        <span class="visually-hidden">Loading...</span>
                    </div>
                </div>
            </t>
            <div t-else="" class="d-flex flex-column h-100">
                <div class="overflow-y-auto flex-grow-1 flex-shrink-1">
                    <t t-foreach="orders" t-as="order" t-key="order.access_token">
                        <div class="o_so_order d-flex flex-column flex-grow-1 mb-2 bg-white">
                            <div class="o_so_order_header p-3" t-on-click="() => this.editOrder(order)">
                                <div class="d-flex align-items-center justify-content-between">
                                    <div class="d-flex flex-column">
                                        <h6 class="m-0" t-esc="order.pos_reference"/>
                                        <span class="text-muted">Order number: <t t-esc="order.trackingNumber" /></span>
                                    </div>
                                    <span
                                        class="badge py-2 rounded-pill text-capitalize"
                                        t-att-class="{
                                            'text-bg-success': order.state == 'paid',
                                            'text-bg-primary': order.state != 'paid'
                                        }"
                                        t-esc="getOrderState(order.state)"/>
                                </div>
                                <p class="small m-0 fst-italic text-muted"
                                    t-esc="order.date"/>
                            </div>
                            <div class="o_so_order_body pt-2 border-top">
                                <div
                                    t-foreach="order.lines"
                                    t-as="line"
                                    t-key="line.uuid"
                                    t-attf-class="o_self_order_item_card position-relative d-flex align-items-start w-100 px-3 overflow-hidden"
                                    >
                                    <div class="d-flex w-100 py-1 justify-content-between">
                                        <div t-attf-class="d-flex {{ line.qty ? 'flex-column align-items-start' : 'flex-row align-items-center' }} text-900 fw-bold fs-6">
                                            <t t-set="lineName" t-value="getNameAndDescription(line)" />
                                            <h4 class="mb-0 o_self_product_name" t-esc="lineName.productName" />
                                            <div t-if="line.qty">
                                                <span class="text-primary fw-bolder small" t-esc="`${line.qty}x `" />
                                                <span
                                                    class="flex-grow-1 me-3 small text-muted"
                                                    t-esc="selfOrder.formatMonetary(getPrice(line) / line.qty)"
                                                    />
                                            </div>
                                            <span
                                                t-if="lineName.attributes"
                                                class="m-0 text-muted small break-line"
                                                t-esc="lineName.attributes"
                                                />
                                            <t t-set="comboParent" t-value="line.combo_parent_uuid and order.lines.find(l => l.uuid === line.combo_parent_uuid)" />
                                            <div t-if="comboParent" class="info ms-2 combo-parent-name">
                                                <i class="fa fa-th-large me-2" role="img" aria-label="Combo" title="Combo"/>
                                                <t t-esc="selfOrder.productByIds[comboParent.product_id].name" />
                                            </div>
                                            <div t-if="line.customer_note" class="d-inline-block m-0 text-muted small break-line">
                                                <i class="fa fa-pencil-square-o" aria-hidden="true" />
                                                <span t-esc="line.customer_note" class="customer_note ms-1" />
                                            </div>
                                        </div>
                                        <span t-attf-class="card-text line_price"
                                            t-esc="selfOrder.formatMonetary(getPrice(line))"/>
                                    </div>
                                </div>
                                <div class="d-flex mt-2 px-3">
                                    <div class="ms-auto border-top">
                                        <table class="table table-sm table-borderless mb-0 py-3">
                                            <tbody>
                                                <tr class="text-end text-muted">
                                                    <th class="pt-2 pb-0">Tax:</th>
                                                    <th class="pt-2 pb-0 pe-0" t-if="!selfOrder.priceLoading" t-esc="selfOrder.formatMonetary(order.amount_tax)"/>
                                                    <span t-else="" class="spinner-border"/>
                                                </tr>
                                                <tr class="text-end">
                                                    <th class="pt-0">Total:</th>
                                                    <th class="pt-0 pe-0" t-if="!selfOrder.priceLoading" t-esc="selfOrder.formatMonetary(order.amount_total)"/>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                    <div t-if="orders.length === 0" class="d-flex justify-content-center mt-3">
                        <div>No order found</div>
                    </div>
                </div>
                <div class="bg-view p-3 border-top">
                    <button class="btn btn-secondary btn-lg" t-on-click="back">
                        <i class="oi oi-chevron-left"/>
                        Back
                    </button>
                </div>
            </div>

        </div>
    </t>
</templates>

```

## File: static\src\app\pages\payment_page\payment_page.js

```javascript
/** @odoo-module */

import { Component, onWillStart, onWillUnmount, useState } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { useService } from "@web/core/utils/hooks";

// This component is only use in Kiosk mode
export class PaymentPage extends Component {
    static template = "pos_self_order.PaymentPage";

    setup() {
        this.selfOrder = useSelfOrder();
        this.selfOrder.isOrder();
        this.router = useService("router");
        this.rpc = useService("rpc");
        this.state = useState({
            selection: true,
            paymentMethodId: null,
        });

        onWillUnmount(() => {
            this.selfOrder.paymentError = false;
        });

        onWillStart(async () => {
            const paymentMethods = this.selfOrder.pos_payment_methods;

            if (paymentMethods.length === 1) {
                this.selectMethod(this.selfOrder.pos_payment_methods[0].id);
            }
        });
    }

    get showFooterBtn() {
        return this.selfOrder.paymentError || this.state.selection;
    }

    selectMethod(methodId) {
        this.state.selection = false;
        this.state.paymentMethodId = methodId;
        this.startPayment();
    }

    get selectedPaymentMethod() {
        return this.selfOrder.pos_payment_methods.find((p) => p.id === this.state.paymentMethodId);
    }

    // this function will be override by pos_online_payment_self_order module
    // in mobile is the only available payment method
    async startPayment() {
        this.selfOrder.paymentError = false;
        try {
            const result = await this.rpc(`/kiosk/payment/${this.selfOrder.pos_config_id}/kiosk`, {
                order: this.selfOrder.currentOrder,
                access_token: this.selfOrder.access_token,
                payment_method_id: this.state.paymentMethodId,
            });
            const order = result.order;
            this.selfOrder.updateOrderFromServer(order);
        } catch (error) {
            this.selfOrder.handleErrorNotification(error);
            this.selfOrder.paymentError = true;
        }
    }
}

```

## File: static\src\app\pages\payment_page\payment_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.PaymentPage">
        <div t-if="state.selection" class="h-100 d-flex flex-wrap p-5 gap-5 align-items-center justify-content-center">
            <div
                t-foreach="selfOrder.pos_payment_methods"
                t-as="payment_method"
                t-key="payment_method.id"
                class="o_kiosk-card border rounded d-flex flex-column align-items-center p-5 justify-content-center bg-white"
                t-on-click="() => this.selectMethod(payment_method.id)">
                <div class="h-75 fs-1 p-3 d-flex flex-column justify-content-center align-items-center">
                    <i class="fa fa-credit-card" aria-hidden="true"></i>
                </div>
                <div class="name w-100 d-flex justify-content-center align-items-center h-25">
                    <span t-esc="payment_method.name" />
                </div>
            </div>
        </div>
        <div class="payment-state-container d-flex justify-content-center align-items-center flex-column h-100 px-3 text-center">
            <h1 class="mb-4">Follow instructions on the terminal</h1>
            <div class="d-inline-flex flex-column border rounded p-4 bg-view mb-3">
                <i class="fa fa-credit-card-alt fs-1" aria-hidden="true"></i>
            </div>
        </div>
        <div class="px-3 py-4 d-flex gap-1 justify-content-center">
            <button class="btn btn-primary btn-lg" t-if="state.selection || selectedPaymentMethod.is_online_payment || this.selfOrder.paymentError" t-on-click="() => this.router.back()">Back</button>
            <button class="btn btn-info btn-lg" t-if="!state.selection and selfOrder.paymentError" t-on-click="startPayment">Retry</button>
        </div>
    </t>
</templates>

```

## File: static\src\app\pages\product_list_page\product_list_page.js

```javascript
/** @odoo-module */

import { Component, useEffect, useRef, useState } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { ProductCard } from "@pos_self_order/app/components/product_card/product_card";
import { CancelPopup } from "@pos_self_order/app/components/cancel_popup/cancel_popup";
import { useService, useChildRef } from "@web/core/utils/hooks";
import { OrderWidget } from "@pos_self_order/app/components/order_widget/order_widget";
import { _t } from "@web/core/l10n/translation";
import { fuzzyLookup } from "@web/core/utils/search";

export class ProductListPage extends Component {
    static template = "pos_self_order.ProductListPage";
    static components = { ProductCard, OrderWidget };

    setup() {
        this.selfOrder = useSelfOrder();
        this.dialog = useService("dialog");
        this.router = useService("router");
        this.productsList = useRef("productsList");
        this.categoryList = useRef("categoryList");
        this.searchInput = useRef("searchInput");
        this.currentProductCard = useChildRef();
        this.state = useState({
            search: false,
            searchInput: "",
        });
        this.categoryButton = Object.fromEntries(
            Array.from(this.selfOrder.categoryList).map((category) => {
                return [category.id, useRef(`category_${category.id}`)];
            })
        );

        useEffect(
            () => {
                if (this.state.search) {
                    this.searchInput.el.focus();
                }
            },
            () => [this.state.search]
        );

        useEffect(
            () => {
                if (this.selfOrder.lastEditedProductId) {
                    this.scrollTo(this.currentProductCard, { behavior: "instant" });
                }
                const scrollSpyContentEl = document.getElementById("scrollspy-products");
                const scrollSpy = ScrollSpy.getOrCreateInstance(scrollSpyContentEl);
                const currentCategId = this.selfOrder.currentCategory.id;
                const categ = document.querySelectorAll(`[categId="${currentCategId}"]`);
                if (categ[0]) {
                    categ[0].scrollIntoView();
                }
                const onActivateScrollSpy = () => {
                    const categId = parseInt(scrollSpy._activeTarget.split("_")[1]);
                    this.selfOrder.currentCategory = this.selfOrder.pos_category.find(
                        (categ) => categ.id === categId
                    );
                };
                scrollSpyContentEl.addEventListener("activate.bs.scrollspy", onActivateScrollSpy);
                return () => {
                    scrollSpyContentEl.removeEventListener(
                        "activate.bs.scrollspy",
                        onActivateScrollSpy
                    );
                };
            },
            () => []
        );

        useEffect(
            () => {
                const category = this.selfOrder.currentCategory;
                const categBtn = this.categoryButton[category.name]?.el;

                if (!categBtn) {
                    return;
                }

                this.categoryList.el.scroll({
                    left: categBtn.offsetLeft + categBtn.offsetWidth / 2 - window.innerWidth / 2,
                    behavior: "smooth",
                });
            },
            () => [this.selfOrder.currentCategory]
        );
    }

    focusSearch() {
        this.state.search = !this.state.search;

        if (!this.state.search) {
            this.state.searchInput = "";
        }
    }

    getFilteredProducts(products) {
        return fuzzyLookup(
            this.state.searchInput,
            products,
            (product) => product.name + product.description_sale
        );
    }

    scrollTo(ref = null, { behavior = "smooth" } = {}) {
        this.productsList.el.scroll({
            top: ref?.el ? ref.el.offsetTop - this.productsList.el.offsetTop : 0,
            behavior,
        });
    }

    review() {
        this.router.navigate("cart");
    }

    back() {
        if (this.selfOrder.config.self_ordering_mode !== "kiosk") {
            this.router.navigate("default");
            return;
        }

        this.dialog.add(CancelPopup, {
            title: _t("Cancel order"),
            confirm: () => {
                this.router.navigate("default");
            },
        });
    }
}

```

## File: static\src\app\pages\product_list_page\product_list_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ProductListPage">
        <div class="d-flex flex-column vh-100 overflow-hidden">
            <!-- Categories selector + Search -->
            <div class="navbar-container position-relative d-flex flex-nowrap w-100 bg-view border-bottom z-index-1">
                <nav id="listgroup-categories" class="category-list d-flex flex-grow-1 py-2 px-3 gap-2 gap-md-3 overflow-x-auto" t-ref="categoryList">
                    <a
                        t-foreach="Array.from(selfOrder.categoryList)"
                        t-as="category"
                        t-key="category.id"
                        t-ref="category_{{category.id}}"
                        t-attf-class="nav-link category-item flex-shrink-0 p-0"
                        t-attf-href="#scrollspy_{{category.id}}">
                        <div class="ratio ratio-1x1 mb-1">
                            <div t-att-class="{'placeholder-glow': category.has_image}">
                                <div class="w-100 h-100 bg-200 rounded d-flex align-items-center justify-content-center" t-att-class="{'placeholder': category.has_image}">
                                    <small class="d-block fw-bold text-white text-center" t-esc="category.name"/>
                                </div>
                            </div>
                            <img t-if="category.has_image"  class="rounded w-100 h-100"
                                t-attf-src="/pos-self/get-category-image/{{ category.id }}"
                                alt="Product image"
                                loading="lazy"
                                onerror="this.remove()" />
                        </div>
                        <small class="d-block fw-bold text-center category-name" t-esc="category.name"/>
                    </a>
                </nav>

                <button class="search-button btn btn-light border-0 border-start rounded-0" t-on-click="focusSearch">
                    <i class="oi oi-search"/>
                </button>

                <div t-att-class="{ 'd-none': !this.state.search }" t-attf-class="position-absolute top-0 start-0 w-100 h-100 px-3 pe-4 d-flex align-items-center bg-view">
                    <input type="text"
                        t-ref="searchInput"
                        class="form-control py-3 ps-0 border-0 shadow-none"
                        placeholder="Search product"
                        t-model="state.searchInput" />
                    <button class="btn-close" t-on-click="focusSearch"/>
                </div>
            </div>

            <!-- Products list -->
            <div
                id="scrollspy-products"
                class="product-list position-relative flex-grow-1 overflow-y-auto"
                t-ref="productsList"
                data-bs-spy="scroll"
                data-bs-target="#listgroup-categories"
                data-bs-offset="10"
                tabindex="0">
                <t t-set="nbrItem" t-value="0" />
                <section
                    t-foreach="Array.from(selfOrder.categoryList)"
                    t-as="category"
                    t-key="category.id"
                    t-attf-id="scrollspy_{{category.id}}"
                    t-attf-categId="{{category.id}}"
                    t-ref="productsWithCategory_{{category.id}}"
                    class="product-list-category d-empty-none bg-view px-3 pb-4">
                    <t t-set="products" t-value="selfOrder.productsGroupedByCategory[category.id]" />
                    <t t-set="availableProducts" t-value="!state.searchInput ? products : getFilteredProducts(products)" />
                    <t t-set="nbrItem" t-value="availableProducts.length + nbrItem" />
                    <t t-if="availableProducts.length > 0">
                        <h2 t-esc="category.name" class="pt-4 pb-2 px-3 mb-4 mx-n3 bg-200 display-6 fw-bold"/>
                        <div class="o-so-products-row">
                            <t t-foreach="availableProducts" t-as="product" t-key="product.id">
                                <ProductCard product="product" currentProductCard="product.id === selfOrder.lastEditedProductId and currentProductCard"/>
                            </t>
                        </div>
                    </t>
                </section>
                <p t-if="nbrItem === 0" class="mx-auto mt-3 text-center">No products found</p>
            </div>

            <!-- Page buttons -->
            <OrderWidget t-if="this.selfOrder.ordering" action.bind="review" />
        </div>
    </t>
</templates>

```

## File: static\src\app\pages\product_page\product_page.js

```javascript
/** @odoo-module */

import { Component, onWillUnmount, useState, useSubEnv } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { AttributeSelection } from "@pos_self_order/app/components/attribute_selection/attribute_selection";
import { useService } from "@web/core/utils/hooks";

export class ProductPage extends Component {
    static template = "pos_self_order.ProductPage";
    static props = ["product", "back?", "onValidate?"];
    static components = { AttributeSelection };

    setup() {
        this.selfOrder = useSelfOrder();
        this.router = useService("router");
        useSubEnv({ selectedValues: {}, customValues: {}, editable: this.editableProductLine });

        if (!this.props.product) {
            this.router.navigate("product_list");
            return;
        }

        this.selfOrder.lastEditedProductId = this.props.product.id;
        this.state = useState({
            qty: 1,
            customer_note: "",
            product: this.props.product,
            selectedValues: this.env.selectedValues,
        });

        this.initState();

        onWillUnmount(() => {
            this.selfOrder.editedLine = null;
        });
    }

    get product() {
        return this.props.product;
    }

    get attributes() {
        return this.product.attributes;
    }

    get editableProductLine() {
        const order = this.selfOrder.currentOrder;
        return !(
            this.selfOrder.editedLine &&
            this.selfOrder.editedLine.uuid &&
            order.lastChangesSent[this.selfOrder.editedLine.uuid]
        );
    }

    initState() {
        const editedLine = this.selfOrder.editedLine;

        if (editedLine) {
            this.state.customer_note = editedLine.customer_note;
            this.state.qty = editedLine.qty;
        }

        return 0;
    }

    back() {
        this.router.navigate("product_list");
    }

    changeQuantity(increase) {
        const currentQty = this.state.qty;

        if (!increase && currentQty === 1) {
            return;
        }

        return increase ? this.state.qty++ : this.state.qty--;
    }

    get showQtyButtons() {
        return this.props.product.self_order_available;
    }
    addToCart() {
        this.selfOrder.addToCart(
            this.props.product,
            this.state.qty,
            this.state.customer_note,
            this.env.selectedValues,
            this.env.customValues
        );
        this.router.back();
    }

    isEveryValueSelected() {
        return Object.values(this.state.selectedValues).find((value) => !value) == false;
    }
}

```

## File: static\src\app\pages\product_page\product_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ProductPage">
        <div class="d-flex flex-column bg-view flex-grow-1 h-100">
            <div class="d-flex align-items-center flex-shrink-0 justify-content-between gap-3 px-3 py-2 w-100 bg-view border-bottom overflow-x-auto z-index-1">
                <button class="btn btn-secondary btn-lg px-3 text-nowrap" t-on-click="back">
                    <i class="oi oi-chevron-left" aria-hidden="true"/><span class="ms-2 d-none d-md-inline">Discard</span>
                </button>
                <h1 class="mb-0 text-nowrap"><strong t-esc="product.name"/> options</h1>
                <span class="d-none d-md-inline px-5"/> <!-- Spacer -->
            </div>

            <div class="pos_self_order_product_page_content d-flex flex-column flex-grow-1 overflow-y-auto">
                <div class="o-so-product-details d-flex flex-row align-items-start p-3 gap-3">
                    <div class="o-so-product-details-image ratio ratio-1x1 w-25 flex-shrink-0">
                        <div class="placeholder-glow">
                            <div class="placeholder w-100 h-100 bg-300 rounded"/>
                        </div>
                        <img
                            class="o_self_order_item_card_image w-100 rounded"
                            t-attf-src="/menu/get-image/{{ props.product.id }}/512?unique={{props.product.write_date}}"
                            alt="Product image"
                            loading="lazy"
                            onerror="this.remove()"/>
                    </div>
                    <div class="o-so-product-details-description">
                        <h2 t-esc="product.name"/>
                        <small t-if="product.description_self_order"
                            class="o_self_order_main_desc d-block mb-3 text-muted"
                            t-out="product.description_self_order"
                        />
                        <span class="fs-3" t-esc="selfOrder.formatMonetary(selfOrder.getProductDisplayPrice(product))"/>
                    </div>
                </div>
                <AttributeSelection
                    t-if="this.product.attributes.length"
                    product="product"/>
            </div>

            <div t-if="showQtyButtons and selfOrder.ordering" class="p-3 text-end">
                <div class="o_self_order_incr_button btn-group " role="group" aria-label="Quantity select">
                    <button type="button"
                        t-on-click = "() => this.changeQuantity(false)"
                        t-attf-class="{{ !this.env.editable ? 'disabled' : '' }} btn btn-secondary btn-lg"><span class="fs-2 lh-1 fa-fw d-inline-block">－</span></button>
                    <div class="o-so-tabular-nums d-flex align-items-center px-3 text-bg-200" t-esc="state.qty"/>
                    <button type="button"
                        t-on-click = "() => this.changeQuantity(true)"
                        class="btn btn-secondary btn-lg"><span class="fs-2 lh-1 fa-fw d-inline-block">＋</span></button>
                </div>
            </div>

            <div t-if="showQtyButtons and !props.onValidate" class="page-buttons d-flex justify-content-end p-3 gap-3 bg-view border-top">
                <button t-if="showQtyButtons and !props.onValidate and selfOrder.ordering" class="btn btn-primary btn-lg" t-att-class="{ 'disabled': this.isEveryValueSelected() }" t-on-click="addToCart">Add to cart</button>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\pages\stand_number_page\stand_number_page.js

```javascript
/** @odoo-module */

import { Component, useState } from "@odoo/owl";
import { useSelfOrder } from "@pos_self_order/app/self_order_service";
import { useService } from "@web/core/utils/hooks";

export class StandNumberPage extends Component {
    static template = "pos_self_order.StandNumberPage";

    setup() {
        this.selfOrder = useSelfOrder();
        this.selfOrder.isOrder();
        this.router = useService("router");
        this.state = useState({
            standNumber: "",
        });
    }

    get tableInput() {
        return this.state.standNumber ? this.state.standNumber : "_ _";
    }

    numberClick(event) {
        const key = event.target.attributes.data.value;

        if (key === "reset") {
            this.state.standNumber = this.state.standNumber.slice(0, -1);
        } else if (key === "clear") {
            this.state.standNumber = "";
        } else {
            this.state.standNumber += key;
        }
    }

    confirm() {
        if (this.state.standNumber.length > 0) {
            this.selfOrder.currentOrder.table_stand_number = this.state.standNumber;
            this.selfOrder.confirmOrder();
        }
    }
}

```

## File: static\src\app\pages\stand_number_page\stand_number_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.StandNumberPage">
        <div class="self_order_stand_number d-flex flex-column flex-grow-1 justify-content-between px-3 overflow-y-auto">

            <div class="text-center pt-3">
                <h1>Get a tracker and enter its number here</h1>
                <div class="input-number form-contol form-control-lg text-center">
                    <span t-esc="tableInput" class="display-1"/>
                </div>
            </div>
            <div class="d-flex justify-content-around align-items-center py-4 py-md-5 my-auto">
                <div class="numpad row g-0 row-cols-3 border">
                    <div t-on-click="numberClick" data="1" class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">1</div>
                    <div t-on-click="numberClick" data="2"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">2</div>
                    <div t-on-click="numberClick" data="3"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">3</div>
                    <div t-on-click="numberClick" data="4"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">4</div>
                    <div t-on-click="numberClick" data="5"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">5</div>
                    <div t-on-click="numberClick" data="6"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">6</div>
                    <div t-on-click="numberClick" data="7"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">7</div>
                    <div t-on-click="numberClick" data="8"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">8</div>
                    <div t-on-click="numberClick" data="9"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">9</div>
                    <div t-on-click="numberClick" data="clear"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">
                        <i class="oi oi-close" aria-hidden="true"></i>
                    </div>
                    <div t-on-click="numberClick" data="0"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">0</div>
                    <div t-on-click="numberClick" data="reset"  class="touch-key col btn btn-light d-flex align-items-center justify-content-center border rounded-0 fs-1">&#9003;</div>
                </div>
            </div>
        </div>
        <div class="d-flex justify-content-between p-3 bg-view border-top">
            <button class="btn btn-secondary btn-lg" t-on-click="() => this.router.back()"><i class="oi oi-chevron-left me-2" aria-hidden="true"/>Back</button>
            <button class="btn btn-primary btn-lg" t-att-class="{'disabled': !state.standNumber}" t-on-click="confirm">Pay</button>
        </div>
    </t>
</templates>

```

## File: static\src\app\store\order_change_receipt_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_self_order.OrderChangeReceipt">
        <div class="pos-receipt">
            <div class="pos-receipt-order-data"><t t-esc="changes.name" /></div>
            <t t-if="changes.tracker || changes.trackingNumber">
                <br />
                <div class="pos-receipt-title">
                    <t t-esc="changes.trackingNumber"/>
                    <t t-if="changes.tracker">
                        / Tracker number: <t t-esc="changes.tracker"/>
                    </t>
                </div>
            </t>
            <br />
            <br />
            <div class="pos-receipt-title">
                NEW
                <t t-esc='changes.time.hours'/>:<t t-esc='changes.time.minutes'/>
            </div>
            <br />
            <br />
            <t t-foreach="changes.new" t-as="change" t-key="change_index">
                <div class="multiprint-flex">
                    <span class="product-quantity" t-esc="change.qty"/>
                    <span class="product-name" t-esc="change.full_product_name"/>
                </div>
                <t t-if="change.customer_note">
                    <div>
                        NOTE
                        <span class="pos-receipt-right-align">...</span>
                    </div>
                    <div><span class="pos-receipt-left-padding">--- <t t-esc="change.customer_note" /></span></div>
                    <br/>
                </t>
            </t>
            <br />
            <br />
    </div>
    </t>

</templates>

```

## File: static\src\overrides\components\product_card\product_card.js

```javascript
/** @odoo-module */

import { ProductCard } from "@point_of_sale/app/generic_components/product_card/product_card";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { patch } from "@web/core/utils/patch";

patch(ProductCard.prototype, {
    setup() {
        super.setup();
        this.pos = usePos();
    },
    isSelfOrderAvailable(product_id) {
        return this.pos.db.product_by_id[product_id].self_order_available;
    },
});

```

## File: static\src\overrides\components\product_card\product_card.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ProductCard" t-inherit="point_of_sale.ProductCard" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('product-information-tag')]" position="attributes">
            <attribute name="t-att-class">{'red-tag' : !isSelfOrderAvailable(props.productId)}</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\product_info_popup\product_info_popup.js

```javascript
/** @odoo-module */

import { ProductInfoPopup } from "@point_of_sale/app/screens/product_screen/product_info_popup/product_info_popup";
import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";

patch(ProductInfoPopup.prototype, {
    setup() {
        super.setup();
        this.orm = useService("orm");
    },
    async switchSelfAvailability() {
        await this.orm.write("product.product", [this.props.product.id], {
            self_order_available: !this.props.product.self_order_available,
        });
        this.props.product.self_order_available = !this.props.product.self_order_available;
    },
});

```

## File: static\src\overrides\components\product_info_popup\product_info_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ProductInfoPopup" t-inherit="point_of_sale.ProductInfoPopup" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('section-product-info-title')]" position="attributes">
            <attribute name="class" remove="bg-info" separator=" "/>
            <attribute name="t-attf-class">{{ props.product.self_order_available ? 'bg-success' : 'bg-danger' }}</attribute>
        </xpath>
        <xpath expr="//div[hasclass('section-product-info-title')]" position="after">
            <div class="section-self-order-availability mt-3 mb-4 pb-4 border-bottom text-start">
                <h3 class="section-title">Self-ordering availability:</h3>
                <div class="section-self-order-availability-body">
                    <div class="form-check form-switch">
                        <input class="form-check-input" type="checkbox" t-att-checked="props.product.self_order_available" t-on-click="() => this.switchSelfAvailability()" />
                    </div>
                    <span>
                        <t t-if="props.product.self_order_available">Available</t>
                        <t t-else="">Not available</t>
                    </span>
                </div>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\receipt_header\receipt_header.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order.ReceiptHeader" t-inherit="point_of_sale.ReceiptHeader" t-inherit-mode="extension">
        <xpath expr="//h1[hasclass('tracking-number')]" position="after">
            <div t-if="props.data.pickingService" class="picking-service text-center">
                <span t-if="props.data.pickingService == 'table'" >Service at Table</span>
                <span t-else="">Pickup At Counter</span>
                <br/>
                <br/>
            </div>
            <div t-if="props.data.tableTracker" class="table-tracker text-center">
                Table Tracker:
                <br/>
                <t t-esc="props.data.tableTracker" />
                <br/>
                <br/>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\models\navbar.js

```javascript
/** @odoo-module */

import { Navbar } from "@point_of_sale/app/navbar/navbar";
import { patch } from "@web/core/utils/patch";

patch(Navbar.prototype, {
    _shouldLoadOrders() {
        return super._shouldLoadOrders() || this.pos.company_has_self_ordering;
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
/** @odoo-module */

import { PosStore } from "@point_of_sale/app/store/pos_store";
import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    // @Override
    async _processData(loadedData) {
        await super._processData(...arguments);
        this.company_has_self_ordering = loadedData["company_has_self_ordering"];
    },
});

patch(Order.prototype, {
    setup() {
        super.setup(...arguments);
        if (this.name.startsWith('Self-Order')) {
            this.trackingNumber = "S" + this.trackingNumber
        }
    },

    defaultTableNeeded(options) {
        return (
            super.defaultTableNeeded(...arguments) &&
            !this.name.includes("Kiosk") &&
            !this.name.includes("Self-Order")
        );
    },

    updateSequenceNumber(json){
        if(!json.name.startsWith('Self-Order')) {
            super.updateSequenceNumber(json);
        }
    }
});

```

## File: views\custom_link_views.xml

```xml
<?xml version="1.0"?>
<odoo>
<record model="ir.ui.view" id="custom_link_tree">
    <field name="name">custom.link.tree</field>
    <field name="model">pos_self_order.custom_link</field>
    <field name="arch" type="xml">
        <tree string="Custom Links" editable="bottom">
            <field name="sequence" widget="handle" />
            <field name="name" placeholder="odoo"/>
            <field name="url" placeholder="https://odoo.com"/>
            <field name="pos_config_ids" widget="many2many_tags" placeholder="empty = all points of sale" options="{'no_create': True}"/>
            <field name="style"/>
            <field name="link_html"/>
        </tree>
    </field>
</record>
</odoo>

```

## File: views\point_of_sale_dashboard.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_self_order_kiosk_read_only_form_dialog" model="ir.ui.view">
        <field name="name">pos.self.order.kiosk.read.only.form.dialog</field>
        <field name="model">pos.config</field>
        <field name="arch" type="xml">
            <form>
                <div class="text-center">
                    <div class="fs-2 mb-2">From your Kiosk, open this URL:</div>
                    <field name="self_ordering_url" readonly="1" />
                </div>
                <footer class="justify-content-end">
                    <button class="btn btn-secondary" special="cancel">Close</button>
                    <button class="btn btn-primary" name="action_open_kiosk" type="object">Open in New Tab</button>
                </footer>
            </form>
        </field>
    </record>

    <record id="pos_self_order_search_view" model="ir.ui.view">
        <field name="name">pos.self.order.search.view</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_config_search" />
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='inactive']" position="after">
                <filter string="Kiosk" name="filter_kiosk_only"
                    domain="[('self_ordering_mode', '=', 'kiosk')]" />
            </xpath>
        </field>
    </record>

    <record id="action_pos_self_order_search_view" model="ir.actions.act_window">
        <field name="name">Kiosk</field>
        <field name="res_model">pos.config</field>
        <field name="view_mode">kanban,list,tree</field>
        <field name="search_view_id" ref="pos_self_order_search_view"/>
    </record>

    <record id="pos_self_order_menu_item" model="ir.ui.view">
        <field name="name">pos.config.kanban.view.inherit.self_order</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_config_kanban" />
        <field name="arch" type="xml">
            <xpath
                expr="//div[hasclass('dropdown-pos-config')]/div/div[hasclass('o_kanban_manage_view')]"
                position="inside">
                <field name="self_ordering_mode" invisible="1" />
                <div role="menuitem">
                    <a name="preview_self_order_app"
                        type="object"
                        style="white-space: nowrap;"
                        invisible="not self_ordering_mode == 'mobile'">
                        Mobile Menu
                    </a>
                </div>
            </xpath>
            <xpath expr="//div[hasclass('o_kanban_card_header_title_name')]" position="after">
                <field name="self_ordering_mode" invisible="1" />
                <div class="badge text-bg-info o_kanban_inline_block me-2"
                    invisible="not self_ordering_mode == 'mobile'">
                    Self Ordering Enabled
                </div>
            </xpath>
            <xpath expr="//div[hasclass('o_kanban_primary_left')]" position="after">
                <field name="self_ordering_mode" invisible="1" />
                <field name="current_session_id" invisible="1" />
                <div class="col-6 o_kanban_primary_left d-flex flex-column align-items-start" invisible="not self_ordering_mode == 'kiosk'">
                    <button t-if="!record.current_session_id.raw_value" class="btn btn-primary pos_open_session_btn" name="action_open_wizard" type="object">
                        Start Kiosk
                    </button>
                    <button t-else="" name="action_close_kiosk_session" class="btn btn-secondary" type="object">
                        Close Session
                    </button>
                    <button t-if="record.current_session_id.raw_value" class="btn-link mt-2" name="action_open_wizard" type="object">
                        Open Kiosk
                    </button>
                </div>
            </xpath>
            <xpath expr="//div[hasclass('o_kanban_primary_left')]" position="attributes">
                <field name="self_ordering_mode" invisible="1" />
                <attribute name="invisible">self_ordering_mode == 'kiosk'</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_config_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_self_view_pos_config_tree" model="ir.ui.view">
        <field name="name">pos.self.pos.config.tree.view</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_config_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="inside">
                <field name="status" />
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_restaurant_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_self_order_table_form_view" model="ir.ui.view">
        <field name="name">Restaurant Table</field>
        <field name="model">restaurant.table</field>
        <field name="inherit_id" ref="pos_restaurant.view_restaurant_table_form" />
        <field name="arch" type="xml">
            <xpath expr="//group[@col='2']" position="inside">
                <field name="identifier" groups="base.group_no_one" />
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_self_order.index.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="pos_self_order.index" name="POS Self Order">&lt;!DOCTYPE html&gt;
        <html t-att-lang="lang and lang.replace('_', '-')">
            <head>
                <title>Odoo Self Order</title>
                <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
                <meta http-equiv="content-type" content="text/html, charset=utf-8" />
                <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no"/>
                <meta name="apple-mobile-web-app-capable" content="yes"/>
                <meta name="mobile-web-app-capable" content="yes"/>
                <script type="text/javascript">
                    var odoo = {
                        csrf_token: "<t t-nocache="The csrf token must always be up to date." t-esc="request.csrf_token(None)"/>",
                        debug: "<t t-esc="debug"/>",
                        __session_info__: <t t-esc="json.dumps(session_info)"/>,
                    };
                </script>
                <t t-call-assets="pos_self_order.assets" />
                <t t-if="'tests' in debug or test_mode_enabled" t-call-assets="pos_self_order.assets_tests" />
            </head>
            <body>
            </body>
        </html>
    </template>
</odoo>

```

## File: views\pos_session_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_self_view_pos_session_form" model="ir.ui.view">
        <field name="name">pos.self.session.form.view</field>
        <field name="model">pos.session</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_session_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='open_frontend_cb']" position="attributes">
                <attribute name="invisible">rescue or state not in ['opening_control', 'opened'] or config_id.self_ordering_mode == 'kiosk'</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_search_view_pos" model="ir.ui.view">
        <field name="name">product.template.search.pos.form</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="point_of_sale.product_template_search_view_pos"/>
        <field name="arch" type="xml">
            <filter name="filter_to_availabe_pos" position="after">
                <filter name="filter_to_self_order" string="Available in Self" domain="[('self_order_available', '=', True)]"/>
            </filter>
            <filter name="filter_to_self_order" position="after">
                <filter name="filter_to_not_available_pos" string="Not available in Self" domain="[('self_order_available', '=', False)]"/>
             </filter>
        </field>
    </record>

    <record id="product_template_form_view" model="ir.ui.view">
        <field name="name">product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="point_of_sale.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='pos']" position="inside">
                <field name="self_order_available" invisible="not available_in_pos"/>
            </xpath>
            <group name="upsell" position="after">
                <group string="Product Description for Self Order" name="description_self_order">
                    <field name="description_self_order"
                           placeholder="Information about your product for Self Order and Kiosk"
                           nolabel="1"
                           colspan="2"/>
                </group>
            </group>
        </field>
    </record>

    <record id="product_template_tree_view" model="ir.ui.view">
        <field name="name">product.template.product.tree.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="point_of_sale.product_template_tree_view"/>
        <field name="arch" type="xml">
            <field name="available_in_pos" position="after">
                <field name="self_order_available" groups="point_of_sale.group_pos_user" optional="hide"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\qr_code.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="report_self_order_qr_codes_page" model="ir.actions.report">
            <field name="name">QR Codes</field>
            <field name="model">pos.config</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">pos_self_order.qr_codes_page</field>
            <field name="report_file">pos_self_order.qr_codes_page</field>
            <field name="print_report_name">"QR codes"</field>
            <field name="binding_model_id" ref="model_pos_config"/>
            <field name="binding_type">report</field>
        </record>
        <record id="paperformat_qrcodes" model="report.paperformat">
            <field name="name">QR Codes Page</field>
            <field name="default" eval="True"/>
            <field name="header_line" eval="False"/>
        </record>
    </data>

    <template id="qr_codes_page">
        <!-- This page-break-inside does not seem to work;
            It would be nice to have the qr codes so the title is never on
            a different page then the qr code itself -->
        <t t-set="qr_code_size" t-value="190"/>
        <t t-call="web.basic_layout">
            <div class="w-100" style="text-align: right">
                <span class="fs-4">Point of sale: <t t-esc="pos_name" /></span><span class="fs-4" style="margin-left:20px">Self Order: <t t-if="self_order">Yes</t><t t-else="">No</t></span>
            </div>
            <div class="mb-5" style="border-bottom: 1px solid black;">
                <h2 t-if="table_mode">Make it easy for your customers to explore your menu
                online or order with the QR codes on your tables</h2>
                <h2 t-else="">Make it easy for your customers to explore your menu
                online with the QR codes on your tables</h2>
            </div>

            <div>
                <h3>How to use</h3>
                <t t-if="table_mode">
                    <p>Each table in your floor plan is assigned a unique QR code based on your configuration. For security reasons,
                    both the point of sale and table names are encrypted in the generated URL, as shown in the example below:.</p>
                    <p class="mt-2">Table: <span t-if="table_example" t-esc="table_example['name']" /><br/>
                    URL: <span t-if="table_example" t-esc="table_example['decoded_url']" /></p>
                </t>
                <t t-else="">
                    <p>Feel free to use and print this QR code as many times as needed according to your requirements.</p>
                    <p>URL: <span t-if="table_example" t-esc="table_example['decoded_url']" /></p>
                </t>
            </div>

            <!-- Table with access token -->
            <t t-if="floors" t-foreach="floors" t-as="floor">
                <div class="mb-5">
                    <h4 t-if="floor.get('name')" t-out="floor['name']" class="mb-3 mt-5"/>
                    <table style="border: none; border-collapse: collapse; width: 100%;">
                        <tbody style="border: none;">
                            <tr t-foreach="floor.get('table_rows')" t-as="row" style="border: none;">
                                <td t-foreach="row" t-as="table" style="width: 33%; text-align:center; border: none;">
                                    <div class="mb-3" style="page-break-inside: avoid">
                                        <h4 class="fw-bold text-center" t-if="table.get('name')">
                                            <t t-esc="table['name']"/>
                                        </h4>
                                        <div class="mt-1 mb-1 position-relative top-0 start-0">
                                            <img t-att-src="'/report/barcode/QR/%s?width=%s&amp;height=%s&amp;barLevel=H' %(table['url'], qr_code_size, qr_code_size)" class="position-relative top-0 start-0" />
                                            <svg class="position-absolute top-50 start-50" style="-webkit-transform: translate(-50%, -50%);" t-att-width="qr_code_size/4" t-att-height="qr_code_size/4" viewBox="0 0 120 120" fill="none"
                                                xmlns="http://www.w3.org/2000/svg">
                                                <g>
                                                    <rect width="120" height="120" rx="33" fill="white"/>
                                                    <g>
                                                        <path d="M60 110C87.6142 110 110 87.6142 110 60C110 32.3858 87.6142 10 60 10C32.3858 10 10 32.3858 10 60C10 87.6142 32.3858 110 60 110Z" fill="#9C5789"/>
                                                        <path d="M59.5166 89.6961C75.8029 89.6961 89.0055 76.4935 89.0055 60.2072C89.0055 43.9209 75.8029 30.7182 59.5166 30.7182C43.2303 30.7182 30.0276 43.9209 30.0276 60.2072C30.0276 76.4935 43.2303 89.6961 59.5166 89.6961Z" fill="white"/>
                                                    </g>
                                                </g>
                                            </svg>
                                        </div>
                                    </div>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </t>

            <div>
                <h3>How to customize</h3>
                <p>If you need customized QR codes, start by scanning the relevant QR code to acquire the URL. Then, make
                use of a QR code generator like https://www.qrcode-monkey.com or https://www.qr-code-generator.com</p>
            </div>
        </t>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="res_config_settings_view_form_menu" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_self_order.view</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='payment_methods_new']" position="before">
                <div class="o_notification_alert alert alert-warning" role="alert" invisible="not is_kiosk_mode">
                    <span>Please note that the kiosk only works with Adyen &amp; Stripe terminals</span>
                </div>
            </xpath>
            <block id="restaurant_section" position="after">
                <block title="Mobile self-order &amp; Kiosk" id="self_ordering_section">
                    <setting string="QR menu &amp; Kiosk activation" help="Let your customers order using their mobile or a kiosk.">a
                        <div class="content-group row">
                            <label for="pos_self_ordering_mode" class="col-lg-4" string="Self Ordering"/>
                            <field name="pos_self_ordering_mode" invisible="not pos_config_id"/>
                        </div>
                        <div class="d-flex flex-column align-items-start w-50" invisible="pos_self_ordering_mode == 'nothing'">
                            <button class="btn-link p-0" icon="oi-arrow-right" name="preview_self_order_app" type="object" string="Preview Web interface"/>
                            <button class="btn-link p-0" icon="oi-arrow-right" name="custom_link_action" type="object" string="Home buttons"/>
                            <button class="btn-link p-0" icon="oi-arrow-right" name="generate_qr_codes_page" type="object" string="Print QR Codes" invisible="not pos_self_ordering_mode in ['consultation', 'mobile']"/>
                            <button class="btn-link p-0" icon="oi-arrow-right" name="generate_qr_codes_zip" type="object" string="Download QR Codes" invisible="not pos_self_ordering_mode in ['consultation', 'mobile']"/>
                            <button groups="base.group_no_one" class="btn-link p-0" icon="oi-arrow-right" name="update_access_tokens" type="object" string="Reset QR Codes" invisible="not pos_self_ordering_mode in ['consultation', 'mobile']"/>
                        </div>
                        <div class="content-group row mt-4" invisible="not pos_self_ordering_mode in ['kiosk', 'mobile']">
                            <label for="pos_self_ordering_service_mode" class="col-lg-4" string="Service at" />
                            <field name="pos_self_ordering_service_mode" readonly="not pos_module_pos_restaurant and pos_self_ordering_mode != 'kiosk'" />
                        </div>
                        <div class="content-group row" groups="base.group_no_one" invisible="not pos_self_ordering_mode in ['kiosk', 'mobile']">
                            <label for="pos_self_ordering_default_user_id" class="col-lg-4" string="Default User"/>
                            <field name="pos_self_ordering_default_user_id" />
                        </div>
                        <div id="self-payment-after" class="content-group row" invisible="not pos_self_ordering_mode in ['kiosk', 'mobile']">
                            <label string="Pay after" for="pos_self_ordering_pay_after" class="col-lg-4"/>
                            <field name="pos_self_ordering_pay_after" readonly="pos_self_ordering_mode == 'kiosk' or (pos_self_ordering_service_mode == 'counter' and pos_self_ordering_mode == 'mobile')" widget="upgrade_selection"/>
                        </div>
                    </setting>
                    <setting string="Splash screens" help="Personalize your splash screen by adding one or multiple images to create a slideshow" invisible="pos_self_ordering_mode == 'nothing'">
                        <field name="pos_self_ordering_image_home_ids" class="w-100" widget="many2many_binary" />
                    </setting>
                    <setting string="Language" help="Available interface languages" invisible="pos_self_ordering_mode == 'nothing'">
                        <div class="content-group mt-3">
                            <div class="row">
                                <label for="pos_self_ordering_default_language_id" class="col-lg-3" string="Default"/>
                                <field name="pos_self_ordering_default_language_id" options="{'no_create': True}"/>
                            </div>
                            <div class="row">
                                <label for="pos_self_ordering_available_language_ids" class="col-lg-3" string="Available"/>
                                <field name="pos_self_ordering_available_language_ids" widget="many2many_tags" options="{'no_create': True}"/>
                            </div>
                        </div>
                        <div>
                            <button name="%(base.action_view_base_language_install)d" icon="oi-arrow-right" type="action" string="Add Languages" class="btn-link"/>
                        </div>
                    </setting>
                    <setting string="Customize Header" help="Add an image to brand your header." invisible="pos_self_ordering_mode == 'nothing'">
                        <field name="pos_self_ordering_image_brand_name" invisible ="1"/>
                        <field name="pos_self_ordering_image_brand" class="w-100" filename="pos_self_ordering_image_brand_name"/>
                    </setting>
                    <setting string="Eat in / Take out" help="Adjust the tax rate based on whether customers are dining in or opting for takeout." invisible="not pos_self_ordering_mode in ['mobile', 'kiosk']">
                        <field name="pos_self_ordering_takeaway"/>
                        <div class="content-group" invisible="not pos_self_ordering_takeaway">
                            <label string="" for="pos_self_ordering_alternative_fp_id" class="me-2"/>
                            <field name="pos_self_ordering_alternative_fp_id" placeholder="Alternative Fiscal Position"/>
                            <div>
                                <button name="%(account.action_account_fiscal_position_form)d" icon="oi-arrow-right" type="action" string="Fiscal Positions" class="btn-link"/>
                            </div>
                        </div>
                    </setting>
                </block>
            </block>

            <block id="restaurant_section" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </block>

            <setting id="customer_display" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </setting>

            <setting id="manual_discount" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </setting>

            <setting id="price_control" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </setting>

            <field name="pos_available_pricelist_ids" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </field>

            <xpath expr="//label[@for='pos_available_pricelist_ids']" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </xpath>

            <setting id="multiple_employee_session" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </setting>

            <setting id="margin_and_cost" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </setting>

            <setting id="flexible_taxes" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </setting>
        </field>
    </record>
</odoo>

```

