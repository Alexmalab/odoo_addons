# Odoo Module: mrp_subcontracting

Category: Manufacturing/Manufacturing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import report
from . import wizard
from . import controllers


def uninstall_hook(env):
    warehouses = env["stock.warehouse"].search([])
    subcontracting_routes = warehouses.mapped("subcontracting_route_id")
    warehouses.write({"subcontracting_route_id": False})
    companies = env["res.company"].search([])
    subcontracting_locations = companies.mapped("subcontracting_location_id")
    subcontracting_locations.active = False
    companies.write({"subcontracting_location_id": False})
    operations_type_to_remove = (warehouses.subcontracting_resupply_type_id | warehouses.subcontracting_type_id)
    operations_type_to_remove.active = False
    # Fail unlink means that the route is used somewhere (e.g. route_id on stock.rule). In this case
    # we don't try to do anything.
    try:
        with env.cr.savepoint():
            subcontracting_routes.unlink()
            operations_type_to_remove.unlink()
    except:
        pass

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "MRP Subcontracting",
    'version': '0.1',
    'summary': "Subcontract Productions",
    'website': 'https://www.odoo.com/app/manufacturing',
    'category': 'Manufacturing/Manufacturing',
    'depends': ['mrp'],
    'data': [
        'data/mrp_subcontracting_data.xml',
        'security/mrp_subcontracting_security.xml',
        'security/ir.model.access.csv',
        'views/mrp_bom_views.xml',
        'views/res_partner_views.xml',
        'views/stock_warehouse_views.xml',
        'views/stock_move_views.xml',
        'views/stock_quant_views.xml',
        'views/stock_picking_views.xml',
        'views/supplier_info_views.xml',
        'views/mrp_production_views.xml',
        'views/subcontracting_portal_views.xml',
        'views/subcontracting_portal_templates.xml',
        'views/stock_location_views.xml',
    ],
    'demo': [
        'data/mrp_subcontracting_demo.xml',
    ],
    'assets': {
        'web.assets_tests': [
            'mrp_subcontracting/static/tests/tours/subcontracting_portal_tour.js',
        ],
        'web.assets_backend': [
            'mrp_subcontracting/static/src/components/**/*',
            'mrp_subcontracting/static/src/views/**/*',
            'mrp_subcontracting/static/src/subcontracting_portal/move_list_view.js',
        ],
        'web.assets_frontend': [
            'mrp_subcontracting/static/src/scss/subcontracting_portal.scss',
        ],
        'mrp_subcontracting.webclient': [
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_backend_helpers'),

            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
            'web/static/lib/bootstrap/scss/_variables-dark.scss',
            'web/static/lib/bootstrap/scss/_maps.scss',

            'web/static/src/libs/fontawesome/css/font-awesome.css',
            'web/static/lib/odoo_ui_icons/*',
            'web/static/src/webclient/navbar/navbar.scss',
            'web/static/src/scss/animation.scss',
            'web/static/src/core/colorpicker/colorpicker.scss',
            'web/static/src/scss/mimetypes.scss',
            'web/static/src/scss/ui.scss',
            'web/static/src/legacy/scss/ui.scss',
            'web/static/src/views/fields/translation_dialog.scss',
            'web/static/src/scss/fontawesome_overridden.scss',

            'web/static/src/module_loader.js',
            'web/static/src/session.js',

            'web/static/lib/luxon/luxon.js',
            'web/static/lib/owl/owl.js',
            'web/static/lib/owl/odoo_module.js',
            'web/static/lib/jquery/jquery.js',
            'web/static/lib/popper/popper.js',
            'web/static/lib/bootstrap/js/dist/util/index.js',
            'web/static/lib/bootstrap/js/dist/dom/data.js',
            'web/static/lib/bootstrap/js/dist/dom/event-handler.js',
            'web/static/lib/bootstrap/js/dist/dom/manipulator.js',
            'web/static/lib/bootstrap/js/dist/dom/selector-engine.js',
            'web/static/lib/bootstrap/js/dist/util/config.js',
            'web/static/lib/bootstrap/js/dist/util/component-functions.js',
            'web/static/lib/bootstrap/js/dist/util/backdrop.js',
            'web/static/lib/bootstrap/js/dist/util/focustrap.js',
            'web/static/lib/bootstrap/js/dist/util/sanitizer.js',
            'web/static/lib/bootstrap/js/dist/util/scrollbar.js',
            'web/static/lib/bootstrap/js/dist/util/swipe.js',
            'web/static/lib/bootstrap/js/dist/util/template-factory.js',
            'web/static/lib/bootstrap/js/dist/base-component.js',
            'web/static/lib/bootstrap/js/dist/alert.js',
            'web/static/lib/bootstrap/js/dist/button.js',
            'web/static/lib/bootstrap/js/dist/carousel.js',
            'web/static/lib/bootstrap/js/dist/collapse.js',
            'web/static/lib/bootstrap/js/dist/dropdown.js',
            'web/static/lib/bootstrap/js/dist/modal.js',
            'web/static/lib/bootstrap/js/dist/offcanvas.js',
            'web/static/lib/bootstrap/js/dist/tooltip.js',
            'web/static/lib/bootstrap/js/dist/popover.js',
            'web/static/lib/bootstrap/js/dist/scrollspy.js',
            'web/static/lib/bootstrap/js/dist/tab.js',
            'web/static/lib/bootstrap/js/dist/toast.js',
            'web/static/src/libs/bootstrap.js',
            'web/static/src/legacy/js/libs/jquery.js',

            ('include', 'web._assets_bootstrap'),

            'base/static/src/css/modules.css',

            'web/static/src/core/utils/transitions.scss',
            'web/static/src/core/**/*',
            ('remove', 'web/static/src/core/emoji_picker/emoji_data.js'),
            'web/static/src/search/**/*',
            'web/static/src/views/*.js',
            'web/static/src/views/*.xml',
            'web/static/src/views/*.scss',
            'web/static/src/views/fields/**/*',
            'web/static/src/views/form/**/*',
            'web/static/src/views/kanban/**/*',
            'web/static/src/views/list/**/*',
            'web/static/src/model/**/*',
            'web/static/src/views/view_button/**/*',
            'web/static/src/views/view_components/**/*',
            'web/static/src/views/view_dialogs/**/*',
            'web/static/src/views/widgets/**/*',
            'web/static/src/webclient/**/*',
            ('remove', 'web/static/src/webclient/clickbot/clickbot.js'),  # lazy loaded
            ('remove', 'web/static/src/views/form/button_box/*.scss'),
            ('remove', 'web/static/src/webclient/share_target/*'),

            # remove the report code and whitelist only what's needed
            ('remove', 'web/static/src/webclient/actions/reports/**/*'),
            'web/static/src/webclient/actions/reports/*.js',
            'web/static/src/webclient/actions/reports/*.xml',

            'web/static/src/env.js',

            'web/static/src/legacy/scss/fields.scss',

            'base/static/src/scss/res_partner.scss',

            # Form style should be computed before
            'web/static/src/views/form/button_box/*.scss',

            'mrp_subcontracting/static/src/subcontracting_portal/*',
            'web/static/src/start.js',
        ],
    },
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug
from collections import OrderedDict

from odoo import conf, http, _
from odoo.http import request
from odoo.exceptions import AccessError, MissingError
from odoo.addons.portal.controllers import portal
from odoo.addons.portal.controllers.portal import pager as portal_pager


class CustomerPortal(portal.CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        if 'production_count' in counters:
            commercial_partner = request.env.user.partner_id.commercial_partner_id
            values['production_count'] = request.env['stock.picking'].search_count([('partner_id.commercial_partner_id', '=', commercial_partner.id), ('move_ids.is_subcontract', '=', True)])
        return values

    @http.route(['/my/productions', '/my/productions/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_productions(self, page=1, date_begin=None, date_end=None, sortby='date', filterby='all'):
        commercial_partner = request.env.user.partner_id.commercial_partner_id
        StockPicking = request.env['stock.picking']
        domain = [('partner_id.commercial_partner_id', '=', commercial_partner.id), ('move_ids.is_subcontract', '=', True)]

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        searchbar_filters = {
            'all': {'label': _('All'), 'domain': []},
            'done': {'label': _('Done'), 'domain': [('state', '=', 'done')]},
            'ready': {'label': _('Ready'), 'domain': [('state', '=', 'assigned')]},
        }
        domain += searchbar_filters[filterby]['domain']

        searchbar_sortings = {
            'date': {'label': _('Newest'), 'order': 'create_date desc, id desc'},
            'name': {'label': _('Name'), 'order': 'name asc, id asc'},
        }
        order = searchbar_sortings[sortby]['order']
        # count for pager
        count = StockPicking.search_count(domain)
        # make pager
        pager = portal_pager(
            url='/my/productions',
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby},
            total=count,
            page=page,
            step=self._items_per_page
        )
        # search the pickings to display, according to the pager data
        pickings = StockPicking.search(
            domain,
            order=order,
            limit=self._items_per_page,
            offset=pager['offset']
        )

        values = {
            'date': date_begin,
            'pickings': pickings,
            'page_name': 'production',
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
            'default_url': '/my/productions',
        }

        return http.request.render("mrp_subcontracting.portal_my_productions", values)

    @http.route("/my/productions/<int:picking_id>", type="http", auth="user", methods=['GET'], website=True)
    def portal_my_production(self, picking_id):
        try:
            self._document_check_access('stock.picking', picking_id)
        except (AccessError, MissingError):
            raise werkzeug.exceptions.NotFound
        picking = request.env['stock.picking'].browse(picking_id)
        return request.render("mrp_subcontracting.subcontracting_portal", {'picking': picking})

    @http.route("/my/productions/<int:picking_id>/subcontracting_portal", type="http", auth="user", methods=['GET'])
    def render_production_backend_view(self, picking_id):
        try:
            picking = self._document_check_access('stock.picking', picking_id)
        except (AccessError, MissingError):
            raise werkzeug.exceptions.NotFound
        session_info = request.env['ir.http'].session_info()
        user_context = dict(request.env.context) if request.session.uid else {}
        mods = conf.server_wide_modules or []
        lang = user_context.get("lang")
        translation_hash = request.env['ir.http'].get_web_translations_hash(mods, lang)
        cache_hashes = {
            "translations": translation_hash,
        }
        production_company = picking.company_id
        session_info.update(
            cache_hashes=cache_hashes,
            action_name='mrp_subcontracting.subcontracting_portal_view_production_action',
            picking_id=picking.id,
            user_companies={
                'current_company': production_company.id,
                'allowed_companies': {
                    production_company.id: {
                        'id': production_company.id,
                        'name': production_company.name,
                    },
                },
            })

        return request.render(
            'mrp_subcontracting.subcontracting_portal_embed',
            {'session_info': session_info},
        )

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\mrp_subcontracting_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="route_resupply_subcontractor_mto" model='stock.route'>
            <field name="name">Resupply Subcontractor on Order</field>
            <field name="company_id"></field>
            <field name="sequence">15</field>
        </record>
        <function model="res.company" name="_create_missing_subcontracting_location" />
        <function model="stock.warehouse" name="write">
            <value model="stock.warehouse" eval="obj().env['stock.warehouse'].search([]).ids"/>
            <value eval="{'subcontracting_to_resupply': True}"/>
        </function>
    </data>
</odoo>

```

## File: data\mrp_subcontracting_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mrp_bom_subcontract" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_delivery_02"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="type">subcontract</field>
            <field name="subcontractor_ids" eval="[(4, ref('base.res_partner_12')), (4, ref('base.partner_demo_portal'))]"/>
        </record>

        <record id="mrp_bom_line_subcontract" model="mrp.bom.line">
            <field name="product_id" ref="mrp.product_product_computer_desk_screw"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="bom_id" ref="mrp_bom_subcontract"/>
        </record>

        <record id="product_supplierinfo_subcontracting" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product.product_delivery_02_product_template"/>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="sequence">1</field>
            <field name="delay">1</field>
            <field name="min_qty">1</field>
            <field name="price">27</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="route_ids" eval="[(4,ref('stock.route_warehouse0_mto'))]"></field>
        </record>

    </data>
</odoo>

```

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.osv.expression import AND

class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    type = fields.Selection(selection_add=[
        ('subcontract', 'Subcontracting')
    ], ondelete={'subcontract': lambda recs: recs.write({'type': 'normal', 'active': False})})
    subcontractor_ids = fields.Many2many('res.partner', 'mrp_bom_subcontractor', string='Subcontractors', check_company=True)

    def _bom_subcontract_find(self, product, picking_type=None, company_id=False, bom_type='subcontract', subcontractor=False):
        domain = self._bom_find_domain(product, picking_type=picking_type, company_id=company_id, bom_type=bom_type)
        if subcontractor:
            domain = AND([domain, [('subcontractor_ids', 'parent_of', subcontractor.ids)]])
            return self.search(domain, order='sequence, product_id, id', limit=1)
        else:
            return self.env['mrp.bom']

    @api.constrains('operation_ids', 'byproduct_ids', 'type')
    def _check_subcontracting_no_operation(self):
        if self.filtered_domain([('type', '=', 'subcontract'), '|', ('operation_ids', '!=', False), ('byproduct_ids', '!=', False)]):
            raise ValidationError(_('You can not set a Bill of Material with operations or by-product line as subcontracting.'))

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from collections import defaultdict
from odoo import fields, models, _, api
from odoo.exceptions import UserError, ValidationError, AccessError
from odoo.tools.float_utils import float_compare, float_is_zero


class MrpProduction(models.Model):
    _inherit = 'mrp.production'
    _rec_names_search = ['name', 'incoming_picking.name']

    move_line_raw_ids = fields.One2many(
        'stock.move.line', string="Detail Component", readonly=False,
        inverse='_inverse_move_line_raw_ids', compute='_compute_move_line_raw_ids'
    )
    subcontracting_has_been_recorded = fields.Boolean("Has been recorded?", copy=False)
    subcontractor_id = fields.Many2one('res.partner', string="Subcontractor", help="Used to restrict access to the portal user through Record Rules")
    bom_product_ids = fields.Many2many('product.product', compute="_compute_bom_product_ids", help="List of Products used in the BoM, used to filter the list of products in the subcontracting portal view")

    incoming_picking = fields.Many2one(related='move_finished_ids.move_dest_ids.picking_id')

    @api.depends('move_raw_ids.move_line_ids')
    def _compute_move_line_raw_ids(self):
        for production in self:
            production.move_line_raw_ids = production.move_raw_ids.move_line_ids

    def _compute_bom_product_ids(self):
        for production in self:
            production.bom_product_ids = production.bom_id.bom_line_ids.product_id

    def _inverse_move_line_raw_ids(self):
        for production in self:
            line_by_product = defaultdict(lambda: self.env['stock.move.line'])
            for line in production.move_line_raw_ids:
                line_by_product[line.product_id] |= line
            for move in production.move_raw_ids:
                move.move_line_ids = line_by_product.pop(move.product_id, self.env['stock.move.line'])
            for product_id, lines in line_by_product.items():
                qty = sum(line.product_uom_id._compute_quantity(line.quantity, product_id.uom_id) for line in lines)
                move = production._get_move_raw_values(product_id, qty, product_id.uom_id)
                move['additional'] = True
                production.move_raw_ids = [(0, 0, move)]
                production.move_raw_ids.filtered(lambda m: m.product_id == product_id)[:1].move_line_ids = lines

    def write(self, vals):
        if self.env.user._is_portal() and not self.env.su:
            unauthorized_fields = set(vals.keys()) - set(self._get_writeable_fields_portal_user())
            if unauthorized_fields:
                raise AccessError(_("You cannot write on fields %s in mrp.production.", ', '.join(unauthorized_fields)))

        if 'date_start' in vals and self.env.context.get('from_subcontract'):
            date_start = fields.Datetime.to_datetime(vals['date_start'])
            date_start_map = {
                prod: date_start - timedelta(days=prod.bom_id.produce_delay)
                if prod.bom_id else date_start
                for prod in self
            }
            res = True
            for production in self:
                res &= super(MrpProduction, production).write({**vals, 'date_start': date_start_map[production]})
            return res

        return super().write(vals)

    def action_merge(self):
        if any(production._get_subcontract_move() for production in self):
            raise ValidationError(_("Subcontracted manufacturing orders cannot be merged."))
        return super().action_merge()

    def subcontracting_record_component(self):
        self.ensure_one()
        self.move_raw_ids.picked = True
        if not self._get_subcontract_move():
            raise UserError(_("This MO isn't related to a subcontracted move"))
        if float_is_zero(self.qty_producing, precision_rounding=self.product_uom_id.rounding):
            return {'type': 'ir.actions.act_window_close'}

        if self.move_raw_ids and not any(self.move_raw_ids.mapped('quantity')):
            raise UserError(_("You must indicate a non-zero amount consumed for at least one of your components"))
        consumption_issues = self._get_consumption_issues()
        if consumption_issues:
            return self._action_generate_consumption_wizard(consumption_issues)
        self.sudo()._update_finished_move()  # Portal user may need sudo rights to update pickings
        self.subcontracting_has_been_recorded = True

        quantity_issues = self._get_quantity_produced_issues()
        if quantity_issues:
            backorder = self.sudo()._split_productions()[1:]
            # No qty to consume to avoid propagate additional move
            # TODO avoid : stock move created in backorder with 0 as qty
            backorder.move_raw_ids.filtered(lambda m: m.additional).product_uom_qty = 0.0

            backorder.qty_producing = backorder.product_qty
            backorder._set_qty_producing()

            self.product_qty = self.qty_producing
            action = self._get_subcontract_move().filtered(lambda m: m.state not in ('done', 'cancel'))._action_record_components()
            action['res_id'] = backorder.id
            return action
        return {'type': 'ir.actions.act_window_close'}

    def pre_button_mark_done(self):
        if self._get_subcontract_move():
            return super(MrpProduction, self.with_context(skip_consumption=True)).pre_button_mark_done()
        return super().pre_button_mark_done()

    def _should_postpone_date_finished(self, date_finished):
        return super()._should_postpone_date_finished(date_finished) and not self._get_subcontract_move()

    def _update_finished_move(self):
        """ After producing, set the move line on the subcontract picking. """
        self.ensure_one()
        subcontract_move_id = self._get_subcontract_move().filtered(lambda m: m.state not in ('done', 'cancel'))
        if subcontract_move_id:
            quantity = self.qty_producing
            if self.lot_producing_id:
                move_lines = subcontract_move_id.move_line_ids.filtered(lambda ml: not ml.picked and ml.lot_id == self.lot_producing_id or not ml.lot_id)
            else:
                move_lines = subcontract_move_id.move_line_ids.filtered(lambda ml: not ml.picked and not ml.lot_id)
            # Update reservation and quantity done
            for ml in move_lines:
                rounding = ml.product_uom_id.rounding
                if float_compare(quantity, 0, precision_rounding=rounding) <= 0:
                    break
                quantity_to_process = min(quantity, ml.quantity)
                quantity -= quantity_to_process

                # on which lot of finished product
                if float_compare(quantity_to_process, ml.quantity, precision_rounding=rounding) >= 0:
                    ml.write({
                        'quantity': quantity_to_process,
                        'picked': True,
                        'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                    })
                else:
                    ml.write({
                        'quantity': quantity_to_process,
                        'picked': True,
                        'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                    })

            if float_compare(quantity, 0, precision_rounding=self.product_uom_id.rounding) > 0:
                self.env['stock.move.line'].create({
                    'move_id': subcontract_move_id.id,
                    'picking_id': subcontract_move_id.picking_id.id,
                    'product_id': self.product_id.id,
                    'location_id': subcontract_move_id.location_id.id,
                    'location_dest_id': subcontract_move_id.location_dest_id.id,
                    'product_uom_id': self.product_uom_id.id,
                    'quantity': quantity,
                    'picked': True,
                    'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                })
            if not self._get_quantity_to_backorder():
                subcontract_move_id.move_line_ids.filtered(lambda ml: not ml.picked).unlink()
                subcontract_move_id._recompute_state()

    def _subcontracting_filter_to_done(self):
        """ Filter subcontracting production where composant is already recorded and should be consider to be validate """
        def filter_in(mo):
            if mo.state in ('done', 'cancel'):
                return False
            if not mo.subcontracting_has_been_recorded:
                return False
            return True

        return self.filtered(filter_in)

    def _has_been_recorded(self):
        self.ensure_one()
        if self.state in ('cancel', 'done'):
            return True
        return self.subcontracting_has_been_recorded

    def _has_tracked_component(self):
        return any(m.has_tracking != 'none' for m in self.move_raw_ids)

    def _has_workorders(self):
        if self.subcontractor_id:
            return False
        else:
            return super()._has_workorders()

    def _get_subcontract_move(self):
        return self.move_finished_ids.move_dest_ids.filtered(lambda m: m.is_subcontract)

    def _get_writeable_fields_portal_user(self):
        return ['move_line_raw_ids', 'lot_producing_id', 'subcontracting_has_been_recorded', 'qty_producing', 'product_qty']

    def _subcontract_sanity_check(self):
        for production in self:
            if production.product_tracking != 'none' and not self.lot_producing_id:
                raise UserError(_('You must enter a serial number for %s', production.product_id.name))
            for sml in production.move_raw_ids.move_line_ids:
                if sml.tracking != 'none' and not sml.lot_id:
                    raise UserError(_('You must enter a serial number for each line of %s', sml.product_id.display_name))
        return True

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SupplierInfo(models.Model):
    _inherit = 'product.supplierinfo'

    is_subcontractor = fields.Boolean('Subcontracted', compute='_compute_is_subcontractor', help="Choose a vendor of type subcontractor if you want to subcontract the product")

    @api.depends('partner_id', 'product_id', 'product_tmpl_id')
    def _compute_is_subcontractor(self):
        for supplier in self:
            boms = supplier.product_id.variant_bom_ids
            boms |= supplier.product_tmpl_id.bom_ids.filtered(lambda b: not b.product_id or b.product_id in (supplier.product_id or supplier.product_tmpl_id.product_variant_ids))
            supplier.is_subcontractor = supplier.partner_id in boms.subcontractor_ids


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _prepare_sellers(self, params=False):
        if params and params.get('subcontractor_ids'):
            return super()._prepare_sellers(params=params).filtered(lambda s: s.partner_id in params.get('subcontractor_ids'))
        return super()._prepare_sellers(params=params)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ResCompany(models.Model):
    _inherit = 'res.company'

    subcontracting_location_id = fields.Many2one('stock.location')

    @api.model
    def _create_missing_subcontracting_location(self):
        company_without_subcontracting_loc = self.env['res.company'].with_context(active_test=False).search(
            [('subcontracting_location_id', '=', False)])
        company_without_subcontracting_loc._create_subcontracting_location()

    def _create_per_company_locations(self):
        super(ResCompany, self)._create_per_company_locations()
        self._create_subcontracting_location()

    def _create_subcontracting_location(self):
        parent_location = self.env.ref('stock.stock_location_locations', raise_if_not_found=False)
        for company in self:
            subcontracting_location = self.env['stock.location'].create({
                'name': _('Subcontracting Location'),
                'usage': 'internal',
                'location_id': parent_location.id,
                'company_id': company.id,
                'is_subcontracting_location': True,
            })
            self.env['ir.default'].set(
                "res.partner",
                "property_stock_subcontractor",
                subcontracting_location.id,
                company_id=company.id,
            )
            company.subcontracting_location_id = subcontracting_location

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    property_stock_subcontractor = fields.Many2one(
        'stock.location', string="Subcontractor Location", company_dependent=True,
        help="The stock location used as source and destination when sending\
        goods to this contact during a subcontracting process.")
    is_subcontractor = fields.Boolean(
        string="Subcontractor", store=False, search="_search_is_subcontractor", compute="_compute_is_subcontractor")
    bom_ids = fields.Many2many('mrp.bom', compute='_compute_bom_ids', string="BoMs for which the Partner is one of the subcontractors")
    production_ids = fields.Many2many('mrp.production', compute='_compute_production_ids', string="MRP Productions for which the Partner is the subcontractor")
    picking_ids = fields.Many2many('stock.picking', compute='_compute_picking_ids', string="Stock Pickings for which the Partner is the subcontractor")

    def _compute_bom_ids(self):
        results = self.env['mrp.bom']._read_group([('subcontractor_ids.commercial_partner_id', 'in', self.commercial_partner_id.ids)], ['subcontractor_ids'], ['id:array_agg'])
        for partner in self:
            bom_ids = []
            for subcontractor, ids in results:
                if partner.id == subcontractor.id or subcontractor.id in partner.child_ids.ids:
                    bom_ids += ids
            partner.bom_ids = bom_ids

    def _compute_production_ids(self):
        results = self.env['mrp.production']._read_group([('subcontractor_id.commercial_partner_id', 'in', self.commercial_partner_id.ids)], ['subcontractor_id'], ['id:array_agg'])
        for partner in self:
            production_ids = []
            for subcontractor, ids in results:
                if partner.id == subcontractor.id or subcontractor.id in partner.child_ids.ids:
                    production_ids += ids
            partner.production_ids = production_ids

    def _compute_picking_ids(self):
        results = self.env['stock.picking']._read_group([('partner_id.commercial_partner_id', 'in', self.commercial_partner_id.ids)], ['partner_id'], ['id:array_agg'])
        for partner in self:
            picking_ids = []
            for partner_rg, ids in results:
                if partner_rg.id == partner.id or partner_rg.id in partner.child_ids.ids:
                    picking_ids += ids
            partner.picking_ids = picking_ids

    def _search_is_subcontractor(self, operator, value):
        assert operator in ('=', '!=', '<>') and value in (True, False), 'Operation not supported'
        subcontractor_ids = self.env['mrp.bom'].search(
            [('type', '=', 'subcontract')]).subcontractor_ids.ids
        if (operator == '=' and value is True) or (operator in ('<>', '!=') and value is False):
            search_operator = 'in'
        else:
            search_operator = 'not in'
        return [('id', search_operator, subcontractor_ids)]

    def _compute_is_subcontractor(self):
        """ Determine whether the partner is a subcontractor (for giving sudo access) """
        for partner in self:
            partner.is_subcontractor = (
                any(user._is_portal() for user in partner.user_ids)
                and partner.env['mrp.bom'].search_count([
                    ('type', '=', 'subcontract'),
                    ('subcontractor_ids', 'in', (partner | partner.commercial_partner_id).ids),
                ], limit=1)
            )

```

## File: models\stock_location.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class StockLocation(models.Model):
    _inherit = 'stock.location'

    is_subcontracting_location = fields.Boolean(
        "Is a Subcontracting Location?",
        help="Check this box to create a new dedicated subcontracting location for this company. Note that standard subcontracting routes will be adapted so as to take these into account automatically."
    )

    subcontractor_ids = fields.One2many('res.partner', 'property_stock_subcontractor')

    @api.constrains('is_subcontracting_location', 'usage', 'location_id')
    def _check_subcontracting_location(self):
        for location in self:
            if location == location.company_id.subcontracting_location_id:
                raise ValidationError(_("You cannot alter the company's subcontracting location"))
            if location.is_subcontracting_location and location.usage != 'internal':
                raise ValidationError(_("In order to manage stock accurately, subcontracting locations must be type Internal, linked to the appropriate company."))

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        new_subcontracting_locations = res.filtered(lambda l: l.is_subcontracting_location)
        new_subcontracting_locations._activate_subcontracting_location_rules()
        return res

    def write(self, values):
        res = super().write(values)
        if 'is_subcontracting_location' in values:
            if values['is_subcontracting_location']:
                self._activate_subcontracting_location_rules()
            else:
                self._archive_subcontracting_location_rules()
        return res

    def _check_access_putaway(self):
        """ Use sudo mode for subcontractor """
        if self.env.user.partner_id.is_subcontractor:
            return self.sudo()
        else:
            return super()._check_access_putaway()

    def _activate_subcontracting_location_rules(self):
        """ Create or unarchive rules for the 'custom' subcontracting location(s).
        The subcontracting location defined on the company is considered as the 'reference' one.
        All rules defined on this 'reference' location will be replicated on 'custom' subcontracting locations.
        """
        locations_per_company = {}
        for location in self:
            if location.is_subcontracting_location and location != location.company_id.subcontracting_location_id:
                locations_per_company.setdefault(location.company_id, []).extend(location)
        new_rules_vals = []
        rules_to_unarchive = self.env['stock.rule']
        for company, locations in locations_per_company.items():
            reference_location_id = company.subcontracting_location_id
            if reference_location_id:
                reference_rules_from = self.env['stock.rule'].search([('location_src_id', '=', reference_location_id.id)])
                reference_rules_to = self.env['stock.rule'].search([('location_dest_id', '=', reference_location_id.id)])
                for location in locations:
                    existing_rules = {
                        (rule.route_id, rule.picking_type_id, rule.action, rule.location_src_id): rule
                        for rule in self.env['stock.rule'].with_context(active_test=False).search([('location_src_id', '=', location.id)])
                    }
                    for rule in reference_rules_from:
                        if (rule.route_id, rule.picking_type_id, rule.action, location) not in existing_rules:
                            new_rules_vals.append(rule.copy_data({
                                'location_src_id': location.id,
                                'name': rule.name.replace(reference_location_id.name, location.name)
                            })[0])
                        else:
                            existing_rule = existing_rules[(rule.route_id, rule.picking_type_id, rule.action, location)]
                            if not existing_rule.active:
                                rules_to_unarchive += existing_rule
                    existing_rules = {
                        (rule.route_id, rule.picking_type_id, rule.action, rule.location_dest_id): rule
                        for rule in self.env['stock.rule'].with_context(active_test=False).search([('location_dest_id', '=', location.id)])
                    }
                    for rule in reference_rules_to:
                        if (rule.route_id, rule.picking_type_id, rule.action, location) not in existing_rules:
                            new_rules_vals.append(rule.copy_data({
                                'location_dest_id': location.id,
                                'name': rule.name.replace(reference_location_id.name, location.name)
                            })[0])
                        else:
                            existing_rule = existing_rules[(rule.route_id, rule.picking_type_id, rule.action, location)]
                            if not existing_rule.active:
                                rules_to_unarchive += existing_rule
        self.env['stock.rule'].create(new_rules_vals)
        rules_to_unarchive.action_unarchive()

    def _archive_subcontracting_location_rules(self):
        """ Archive subcontracting rules for locations that are no longer 'custom' subcontracting locations."""
        reference_location_ids = self.company_id.subcontracting_location_id
        reference_rules = self.env['stock.rule'].search(['|', ('location_src_id', 'in', reference_location_ids.ids), ('location_dest_id', 'in', reference_location_ids.ids)])
        reference_routes = reference_rules.route_id
        rules_to_archive = self.env['stock.rule'].search(['&', ('route_id', 'in', reference_routes.ids), '|', ('location_src_id', 'in', self.ids), ('location_dest_id', 'in', self.ids)])
        rules_to_archive.action_archive()

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import fields, models, api, _
from odoo.exceptions import AccessError
from odoo.tools.float_utils import float_compare, float_is_zero, float_round
from odoo.tools.misc import OrderedSet


class StockMove(models.Model):
    _inherit = 'stock.move'

    is_subcontract = fields.Boolean('The move is a subcontract receipt')
    show_subcontracting_details_visible = fields.Boolean(
        compute='_compute_show_subcontracting_details_visible'
    )

    def _compute_display_assign_serial(self):
        super(StockMove, self)._compute_display_assign_serial()
        for move in self:
            if not move.is_subcontract:
                continue
            productions = move._get_subcontract_production()
            if not productions or move.has_tracking == 'none':
                continue
            if productions._has_tracked_component() or productions[:1].consumption != 'strict':
                move.display_assign_serial = False

    def _compute_show_subcontracting_details_visible(self):
        """ Compute if the action button in order to see moves raw is visible """
        self.show_subcontracting_details_visible = False
        for move in self:
            if not move.is_subcontract:
                continue
            if not move.picked or float_is_zero(move.quantity, precision_rounding=move.product_uom.rounding):
                continue
            productions = move._get_subcontract_production()
            if not productions or (productions[:1].consumption == 'strict' and not productions[:1]._has_tracked_component()):
                continue
            move.show_subcontracting_details_visible = True

    def _compute_show_details_visible(self):
        """ If the move is subcontract and the components are tracked. Then the
        show details button is visible.
        """
        res = super(StockMove, self)._compute_show_details_visible()
        for move in self:
            if not move.is_subcontract:
                continue
            if self.env.user._is_portal():
                move.show_details_visible = any(not p._has_been_recorded() for p in move._get_subcontract_production())
                continue
            productions = move._get_subcontract_production()
            if not productions._has_tracked_component() and productions[:1].consumption == 'strict':
                continue
            move.show_details_visible = True
        return res

    def _compute_picked(self):
       subcontracted_moves = self.filtered(lambda m: m.is_subcontract)
       super(StockMove, self - subcontracted_moves)._compute_picked()

    def _set_quantity_done(self, qty):
        to_set_moves = self
        for move in self:
            if move.is_subcontract and move._subcontracting_possible_record():
                # If 'done' quantity is changed through the move, record components as if done through the wizard.
                move._auto_record_components(qty)
                to_set_moves -= move
        if to_set_moves:
            super(StockMove, to_set_moves)._set_quantity_done(qty)

    def _set_quantity(self):
        to_set_moves = self
        for move in self:
            if move.is_subcontract and move._subcontracting_possible_record():
                move_line_quantities = sum(move.move_line_ids.filtered(lambda ml: ml.picked).mapped('quantity'))
                delta_qty = move.quantity - move_line_quantities
                if float_compare(delta_qty, 0, precision_rounding=move.product_uom.rounding) > 0:
                    move._auto_record_components(delta_qty)
                    to_set_moves -= move
                elif float_compare(delta_qty, 0, precision_rounding=move.product_uom.rounding) < 0:
                    move.with_context(transfer_qty=True)._reduce_subcontract_order_qty(abs(delta_qty))
        if to_set_moves:
            super(StockMove, to_set_moves)._set_quantity()

    def _auto_record_components(self, qty):
        self.ensure_one()
        subcontracted_productions = self._get_subcontract_production()
        production = subcontracted_productions.filtered(lambda p: not p._has_been_recorded())[-1:]
        if not production:
            # If new quantity is over the already recorded quantity and we have no open production, then create a new one for the missing quantity.
            production = subcontracted_productions[-1:]
            production = production.sudo().with_context(allow_more=True)._split_productions({production: [production.qty_producing, qty]})[-1:]
        qty = self.product_uom._compute_quantity(qty, production.product_uom_id)

        if production.product_tracking == 'serial':
            qty = float_round(qty, precision_digits=0, rounding_method='UP')  # Makes no sense to have partial quantities for serial number
            if float_compare(qty, production.product_qty, precision_rounding=production.product_uom_id.rounding) < 0:
                remaining_qty = production.product_qty - qty
                productions = production.sudo()._split_productions({production: ([1] * int(qty)) + [remaining_qty]})[:-1]
            else:
                productions = production.sudo().with_context(allow_more=True)._split_productions({production: ([1] * int(qty))})

            for production in productions:
                production.qty_producing = 1
                if not production.lot_producing_id:
                    production.action_generate_serial()
                production.with_context(cancel_backorder=False).subcontracting_record_component()
        else:
            production.qty_producing = qty
            if float_compare(production.qty_producing, production.product_qty, precision_rounding=production.product_uom_id.rounding) > 0:
                self.env['change.production.qty'].with_context(skip_activity=True).create({
                    'mo_id': production.id,
                    'product_qty': qty
                }).change_prod_qty()
            if production.product_tracking == 'lot' and not production.lot_producing_id:
                production.action_generate_serial()
            production._set_qty_producing()
            production.with_context(cancel_backorder=False).subcontracting_record_component()

    def copy_data(self, default=None):
        default = dict(default or {})
        vals_list = super().copy_data(default=default)
        for move, vals in zip(self, vals_list):
            if 'location_id' in default or not move.is_subcontract:
                continue
            vals['location_id'] = move.picking_id.location_id.id
        return vals_list

    def write(self, values):
        """ If the initial demand is updated then also update the linked
        subcontract order to the new quantity.
        """
        self._check_access_if_subcontractor(values)
        if 'product_uom_qty' in values and self.env.context.get('cancel_backorder') is not False and not self._context.get('extra_move_mode'):
            self.filtered(
                lambda m: m.is_subcontract and m.state not in ['draft', 'cancel', 'done']
                and float_compare(m.product_uom_qty, values['product_uom_qty'], precision_rounding=m.product_uom.rounding) != 0
            )._update_subcontract_order_qty(values['product_uom_qty'])
        res = super().write(values)
        if 'date' in values:
            for move in self:
                if move.state in ('done', 'cancel') or not move.is_subcontract:
                    continue
                move.move_orig_ids.production_id.with_context(from_subcontract=True).filtered(lambda p: p.state not in ('done', 'cancel')).write({
                    'date_start': move.date,
                    'date_finished': move.date,
                })
        return res

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            self._check_access_if_subcontractor(vals)
        return super().create(vals_list)

    def action_show_details(self):
        """ Open the produce wizard in order to register tracked components for
        subcontracted product. Otherwise use standard behavior.
        """
        self.ensure_one()
        if self.state != 'done' and (self._subcontrating_should_be_record() or self._subcontrating_can_be_record()):
            return self._action_record_components()
        action = super(StockMove, self).action_show_details()
        if self.is_subcontract and all(p._has_been_recorded() for p in self._get_subcontract_production()):
            action['views'] = [(self.env.ref('stock.view_stock_move_operations').id, 'form')]
            action['context'].update({
                'show_lots_m2o': self.has_tracking != 'none',
                'show_lots_text': False,
            })
        elif self.env.user._is_portal():
            action['views'] = [(self.env.ref('mrp_subcontracting.mrp_subcontracting_view_stock_move_operations').id, 'form')]
        return action

    def action_show_subcontract_details(self):
        """ Display moves raw for subcontracted product self. """
        moves = self._get_subcontract_production().move_raw_ids.filtered(lambda m: m.state != 'cancel')
        list_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_move_tree_view')
        form_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_move_form_view')
        ctx = dict(self._context, search_default_by_product=True)
        if self.env.user._is_portal():
            form_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_portal_move_form_view')
            ctx.update(no_breadcrumbs=False)
        return {
            'name': _('Raw Materials for %s', self.product_id.display_name),
            'type': 'ir.actions.act_window',
            'res_model': 'stock.move',
            'views': [(list_view.id, 'list'), (form_view.id, 'form')],
            'target': 'current',
            'domain': [('id', 'in', moves.ids)],
            'context': ctx
        }

    def _action_cancel(self):
        productions_to_cancel_ids = OrderedSet()
        for move in self:
            if move.is_subcontract:
                active_productions = move.move_orig_ids.production_id.filtered(lambda p: p.state not in ('done', 'cancel'))
                moves_todo = self.env.context.get('moves_todo')
                not_todo_productions = active_productions.filtered(lambda p: p not in moves_todo.move_orig_ids.production_id) if moves_todo else active_productions
                if not_todo_productions:
                    productions_to_cancel_ids.update(not_todo_productions.ids)

        if productions_to_cancel_ids:
            productions_to_cancel = self.env['mrp.production'].browse(productions_to_cancel_ids)
            productions_to_cancel.with_context(skip_activity=True).action_cancel()

        return super()._action_cancel()

    def _action_confirm(self, merge=True, merge_into=False):
        subcontract_details_per_picking = defaultdict(list)
        for move in self:
            if move.location_id.usage != 'supplier' or move.location_dest_id.usage == 'supplier':
                continue
            if move.move_orig_ids.production_id:
                continue
            bom = move._get_subcontract_bom()
            if not bom:
                continue
            company = move.company_id
            subcontracting_location = \
                move.picking_id.partner_id.with_company(company).property_stock_subcontractor \
                or company.subcontracting_location_id
            move.write({
                'is_subcontract': True,
                'location_id': subcontracting_location.id
            })
            move._action_assign()  # Re-reserve as the write on location_id will break the link
        res = super()._action_confirm(merge=merge, merge_into=merge_into)
        for move in res:
            if move.is_subcontract:
                subcontract_details_per_picking[move.picking_id].append((move, move._get_subcontract_bom()))
        for picking, subcontract_details in subcontract_details_per_picking.items():
            picking._subcontracted_produce(subcontract_details)

        if subcontract_details_per_picking:
            self.env['stock.picking'].concat(*list(subcontract_details_per_picking.keys())).action_assign()
        return res

    def _action_record_components(self):
        self.ensure_one()
        production = self._get_subcontract_production()[-1:]
        view = self.env.ref('mrp_subcontracting.mrp_production_subcontracting_form_view')
        if self.env.user._is_portal():
            view = self.env.ref('mrp_subcontracting.mrp_production_subcontracting_portal_form_view')
        context = dict(self._context)
        context.pop('skip_consumption', False)
        return {
            'name': _('Subcontract'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mrp.production',
            'views': [(view.id, 'form')],
            'view_id': view.id,
            'target': 'new',
            'res_id': production.id,
            'context': context,
        }

    def _get_subcontract_bom(self):
        self.ensure_one()
        bom = self.env['mrp.bom'].sudo()._bom_subcontract_find(
            self.product_id,
            picking_type=self.picking_type_id,
            company_id=self.company_id.id,
            bom_type='subcontract',
            subcontractor=self.picking_id.partner_id,
        )
        return bom

    def _subcontrating_should_be_record(self):
        return self._get_subcontract_production().filtered(lambda p: not p._has_been_recorded() and p._has_tracked_component())

    def _subcontrating_can_be_record(self):
        return self._get_subcontract_production().filtered(lambda p: not p._has_been_recorded() and p.consumption != 'strict')

    def _subcontracting_possible_record(self):
        return self._get_subcontract_production().filtered(lambda p: p._has_tracked_component() or p.consumption != 'strict')

    def _get_subcontract_production(self):
        return self.filtered(lambda m: m.is_subcontract).move_orig_ids.production_id

    # TODO: To be deleted, use self._get_subcontract_production()._has_tracked_component() instead
    def _has_tracked_subcontract_components(self):
        return any(m.has_tracking != 'none' for m in self._get_subcontract_production().move_raw_ids)

    def _prepare_move_split_vals(self, qty):
        vals = super(StockMove, self)._prepare_move_split_vals(qty)
        vals['location_id'] = self.location_id.id
        return vals

    def _prepare_procurement_values(self):
        res = super()._prepare_procurement_values()
        if self.raw_material_production_id.subcontractor_id:
            res['warehouse_id'] = self.picking_type_id.warehouse_id
        return res

    def _should_bypass_reservation(self, forced_location=False):
        """ If the move is subcontracted then ignore the reservation. """
        should_bypass_reservation = super()._should_bypass_reservation(forced_location=forced_location)
        if not should_bypass_reservation and self.is_subcontract:
            return True
        return should_bypass_reservation

    def _update_subcontract_order_qty(self, new_quantity):
        for move in self:
            quantity_to_remove = move.product_uom_qty - new_quantity
            if not float_is_zero(quantity_to_remove, precision_rounding=move.product_uom.rounding):
                move._reduce_subcontract_order_qty(quantity_to_remove)

    def _reduce_subcontract_order_qty(self, quantity_to_remove):
        self.ensure_one()
        productions = self.move_orig_ids.production_id.filtered(lambda p: p.state not in ('done', 'cancel'))[::-1]
        wip_production = productions[0] if self._context.get('transfer_qty') and len(productions) > 1 else self.env['mrp.production']

        # Transfer removed qty to WIP production
        if wip_production:
            self.env['change.production.qty'].with_context(skip_activity=True).create({
                'mo_id': wip_production.id,
                'product_qty': wip_production.product_qty + quantity_to_remove
            }).change_prod_qty()

        # Cancel productions until reach new_quantity
        for production in (productions - wip_production):
            if quantity_to_remove >= production.product_qty:
                quantity_to_remove -= production.product_qty
                production.with_context(skip_activity=True).action_cancel()
            else:
                if float_is_zero(quantity_to_remove, precision_rounding=production.product_uom_id.rounding):
                    # No need to do change_prod_qty for no change at all.
                    break
                self.env['change.production.qty'].with_context(skip_activity=True).create({
                    'mo_id': production.id,
                    'product_qty': production.product_qty - quantity_to_remove
                }).change_prod_qty()
                break

    def _check_access_if_subcontractor(self, vals):
        if self.env.user._is_portal() and not self.env.su:
            if vals.get('state') == 'done':
                raise AccessError(_("Portal users cannot create a stock move with a state 'Done' or change the current state to 'Done'."))

    def _is_subcontract_return(self):
        self.ensure_one()
        subcontracting_location = self.picking_id.partner_id.with_company(self.company_id).property_stock_subcontractor
        return (
                not self.is_subcontract
                and self.origin_returned_move_id.is_subcontract
                and self.location_dest_id.id == subcontracting_location.id
        )

```

## File: models\stock_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import _, api, models


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    @api.onchange('lot_name', 'lot_id')
    def _onchange_serial_number(self):
        current_location_id = self.location_id
        res = super()._onchange_serial_number()
        if res and not self.lot_name and current_location_id.is_subcontracting_location:
            # we want to avoid auto-updating source location in this case + change the warning message
            self.location_id = current_location_id
            res['warning']['message'] = res['warning']['message'].split("\n\n", 1)[0] + "\n\n" + \
                _("Make sure you validate or adapt the related resupply picking to your subcontractor in order to avoid inconsistencies in your stock.")
        return res

    def write(self, vals):
        for move_line in self:
            if vals.get('lot_id') and move_line.move_id.is_subcontract and move_line.location_id.is_subcontracting_location:
                # Update related subcontracted production to keep consistency between production and reception.
                subcontracted_production = move_line.move_id._get_subcontract_production().filtered(lambda p: p.state not in ('done', 'cancel') and p.lot_producing_id == move_line.lot_id)
                if subcontracted_production:
                    subcontracted_production.lot_producing_id = vals['lot_id']
        return super().write(vals)

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from collections import defaultdict
from datetime import timedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools.float_utils import float_compare
from dateutil.relativedelta import relativedelta


class StockPicking(models.Model):
    _name = 'stock.picking'
    _inherit = 'stock.picking'

    move_line_ids_without_package = fields.One2many(
        domain=['&', '|', ('location_dest_id.usage', '!=', 'production'), ('move_id.picking_code', '!=', 'outgoing'),
                     '|', ('package_level_id', '=', False), ('picking_type_entire_packs', '=', False)])
    display_action_record_components = fields.Selection(
        [('hide', 'Hide'), ('facultative', 'Facultative'), ('mandatory', 'Mandatory')],
        compute='_compute_display_action_record_components')

    @api.depends('state', 'move_ids')
    def _compute_display_action_record_components(self):
        self.display_action_record_components = 'hide'
        for picking in self:
            # Hide if not encoding state or it is not a subcontracting picking
            if picking.state in ('draft', 'cancel', 'done') or not picking._is_subcontract():
                continue
            subcontracted_moves = picking.move_ids.filtered(lambda m: m.is_subcontract)
            if subcontracted_moves._subcontrating_should_be_record():
                picking.display_action_record_components = 'mandatory'
                continue
            if subcontracted_moves._subcontrating_can_be_record():
                picking.display_action_record_components = 'facultative'

    @api.depends('picking_type_id', 'partner_id')
    def _compute_location_id(self):
        super()._compute_location_id()

        for picking in self:
            # If this is a subcontractor resupply transfer, set the destination location
            # to the vendor subcontractor location
            subcontracting_resupply_type_id = picking.picking_type_id.warehouse_id.subcontracting_resupply_type_id
            if picking.picking_type_id == subcontracting_resupply_type_id\
                and picking.partner_id.property_stock_subcontractor:
                picking.location_dest_id = picking.partner_id.property_stock_subcontractor


    # -------------------------------------------------------------------------
    # Action methods
    # -------------------------------------------------------------------------
    def _action_done(self):
        res = super(StockPicking, self)._action_done()
        for move in self.move_ids:
            if not move.is_subcontract:
                continue
            # Auto set qty_producing/lot_producing_id of MO wasn't recorded
            # manually (if the flexible + record_component or has tracked component)
            productions = move._get_subcontract_production()
            recorded_productions = productions.filtered(lambda p: p._has_been_recorded())
            recorded_qty = sum(recorded_productions.mapped('qty_producing'))
            sm_done_qty = sum(productions._get_subcontract_move().filtered(lambda m: m.picked).mapped('quantity'))
            rounding = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            if float_compare(move.product_uom_qty, move.quantity, precision_digits=rounding) > 0 and self._context.get('cancel_backorder'):
                move._update_subcontract_order_qty(move.quantity)
            if float_compare(recorded_qty, sm_done_qty, precision_digits=rounding) >= 0:
                continue
            production = productions - recorded_productions
            if not production:
                continue
            if len(production) > 1:
                raise UserError(_("There shouldn't be multiple productions to record for the same subcontracted move."))
            # Manage additional quantities
            quantity_done_move = move.product_uom._compute_quantity(move.quantity, production.product_uom_id)
            if float_compare(production.product_qty, quantity_done_move, precision_rounding=production.product_uom_id.rounding) == -1:
                change_qty = self.env['change.production.qty'].create({
                    'mo_id': production.id,
                    'product_qty': quantity_done_move
                })
                change_qty.with_context(skip_activity=True).change_prod_qty()
            # Create backorder MO for each move lines
            amounts = [move_line.quantity for move_line in move.move_line_ids]
            len_amounts = len(amounts)
            productions = production._split_productions({production: amounts}, set_consumed_qty=True)
            productions.move_finished_ids.move_line_ids.write({'quantity': 0})
            for production, move_line in zip(productions, move.move_line_ids):
                if move_line.lot_id:
                    production.lot_producing_id = move_line.lot_id
                production.qty_producing = production.product_qty
                production._set_qty_producing()
            productions[:len_amounts].subcontracting_has_been_recorded = True

        for picking in self:
            productions_to_done = picking._get_subcontract_production()._subcontracting_filter_to_done()
            productions_to_done._subcontract_sanity_check()
            if not productions_to_done:
                continue
            productions_to_done = productions_to_done.sudo()
            production_ids_backorder = []
            if not self.env.context.get('cancel_backorder'):
                production_ids_backorder = productions_to_done.filtered(lambda mo: mo.state == "progress").ids
            productions_to_done.with_context(mo_ids_to_backorder=production_ids_backorder).button_mark_done()
            # For concistency, set the date on production move before the date
            # on picking. (Traceability report + Product Moves menu item)
            minimum_date = min(picking.move_line_ids.mapped('date'))
            production_moves = productions_to_done.move_raw_ids | productions_to_done.move_finished_ids
            production_moves.write({'date': minimum_date - timedelta(seconds=1)})
            production_moves.move_line_ids.write({'date': minimum_date - timedelta(seconds=1)})

        return res

    def action_record_components(self):
        self.ensure_one()
        move_subcontracted = self.move_ids.filtered(lambda m: m.is_subcontract)
        for move in move_subcontracted:
            production = move._subcontrating_should_be_record()
            if production:
                return move._action_record_components()
        for move in move_subcontracted:
            production = move._subcontrating_can_be_record()
            if production:
                return move._action_record_components()
        raise UserError(_("Nothing to record"))

    # -------------------------------------------------------------------------
    # Subcontract helpers
    # -------------------------------------------------------------------------
    def _is_subcontract(self):
        self.ensure_one()
        return self.picking_type_id.code == 'incoming' and any(m.is_subcontract for m in self.move_ids)

    def _get_subcontract_production(self):
        return self.move_ids._get_subcontract_production()

    def _get_warehouse(self, subcontract_move):
        return subcontract_move.warehouse_id or self.picking_type_id.warehouse_id or subcontract_move.move_dest_ids.picking_type_id.warehouse_id

    def _prepare_subcontract_mo_vals(self, subcontract_move, bom):
        subcontract_move.ensure_one()
        group = self.env['procurement.group'].create({
            'name': self.name,
            'partner_id': self.partner_id.id,
        })
        product = subcontract_move.product_id
        warehouse = self._get_warehouse(subcontract_move)
        subcontracting_location = \
            subcontract_move.picking_id.partner_id.with_company(subcontract_move.company_id).property_stock_subcontractor \
            or subcontract_move.company_id.subcontracting_location_id
        vals = {
            'company_id': subcontract_move.company_id.id,
            'procurement_group_id': group.id,
            'subcontractor_id': subcontract_move.picking_id.partner_id.commercial_partner_id.id,
            'picking_ids': [subcontract_move.picking_id.id],
            'product_id': product.id,
            'product_uom_id': subcontract_move.product_uom.id,
            'bom_id': bom.id,
            'location_src_id': subcontracting_location.id,
            'location_dest_id': subcontracting_location.id,
            'product_qty': subcontract_move.product_uom_qty or subcontract_move.quantity,
            'picking_type_id': warehouse.subcontracting_type_id.id,
            'date_start': subcontract_move.date - relativedelta(days=bom.produce_delay)
        }
        return vals

    def _subcontracted_produce(self, subcontract_details):
        self.ensure_one()
        group_move = defaultdict(list)
        group_by_company = defaultdict(list)
        for move, bom in subcontract_details:
            # do not create extra production for move that have their quantity updated
            if move.move_orig_ids.production_id:
                continue
            quantity = move.product_qty or move.quantity
            if float_compare(quantity, 0, precision_rounding=move.product_uom.rounding) <= 0:
                # If a subcontracted amount is decreased, don't create a MO that would be for a negative value.
                continue

            mo_subcontract = self._prepare_subcontract_mo_vals(move, bom)
            # Link the move to the id of the MO's procurement group
            group_move[mo_subcontract['procurement_group_id']] = move
            # Group the MO by company
            group_by_company[move.company_id.id].append(mo_subcontract)

        all_mo = set()
        for company, group in group_by_company.items():
            grouped_mo = self.env['mrp.production'].with_company(company).create(group)
            all_mo.update(grouped_mo.ids)

        all_mo = self.env['mrp.production'].browse(sorted(all_mo))
        all_mo.action_confirm()

        for mo in all_mo:
            move = group_move[mo.procurement_group_id.id][0]
            mo.write({'date_finished': move.date})
            finished_move = mo.move_finished_ids.filtered(lambda m: m.product_id == move.product_id)
            finished_move.write({'move_dest_ids': [(4, move.id, False)]})

        all_mo.action_assign()

```

## File: models\stock_quant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import UserError


class StockQuant(models.Model):
    _inherit = 'stock.quant'

    is_subcontract = fields.Boolean(store=False, search='_search_is_subcontract')

    def _search_is_subcontract(self, operator, value):
        if operator not in ['=', '!='] or not isinstance(value, bool):
            raise UserError(_('Operation not supported'))

        return [('location_id.is_subcontracting_location', operator, value)]

```

## File: models\stock_replenish_mixin.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.osv import expression


class ProductReplenishMixin(models.AbstractModel):
    _inherit = 'stock.replenish.mixin'

    def _get_allowed_route_domain(self):
        domains = super()._get_allowed_route_domain()
        route_id = self.env['stock.warehouse']._find_or_create_global_route('mrp_subcontracting.route_resupply_subcontractor_mto', _('Resupply Subcontractor on Order')).id
        return expression.AND([domains, [('id', '!=', route_id)]])

```

## File: models\stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockRule(models.Model):
    _inherit = "stock.rule"

    def _push_prepare_move_copy_values(self, move_to_copy, new_date):
        new_move_vals = super(StockRule, self)._push_prepare_move_copy_values(move_to_copy, new_date)
        new_move_vals["is_subcontract"] = False
        return new_move_vals

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    subcontracting_to_resupply = fields.Boolean(
        'Resupply Subcontractors', default=True)
    subcontracting_mto_pull_id = fields.Many2one(
        'stock.rule', 'Subcontracting MTO Rule', copy=False)
    subcontracting_pull_id = fields.Many2one(
        'stock.rule', 'Subcontracting MTS Rule', copy=False
    )

    subcontracting_route_id = fields.Many2one('stock.route', 'Resupply Subcontractor', ondelete='restrict', copy=False)

    subcontracting_type_id = fields.Many2one(
        'stock.picking.type', 'Subcontracting Operation Type',
        domain=[('code', '=', 'mrp_operation')], copy=False)
    subcontracting_resupply_type_id = fields.Many2one(
        'stock.picking.type', 'Subcontracting Resupply Operation Type',
        domain=[('code', '=', 'internal')], copy=False)

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        res._update_subcontracting_locations_rules()
        # if new warehouse has resupply enabled, enable global route
        if any([vals.get('subcontracting_to_resupply', False) for vals in vals_list]):
            res._update_global_route_resupply_subcontractor()
        return res

    def write(self, vals):
        res = super().write(vals)
        # if all warehouses have resupply disabled, disable global route, until its enabled on a warehouse
        if 'subcontracting_to_resupply' in vals or 'active' in vals:
            if 'subcontracting_to_resupply' in vals:
                # ignore when warehouse archived since it will auto-archive all of its rules
                self._update_resupply_rules()
            self._update_global_route_resupply_subcontractor()
        return res

    def get_rules_dict(self):
        result = super(StockWarehouse, self).get_rules_dict()
        subcontract_location_id = self._get_subcontracting_location()
        for warehouse in self:
            result[warehouse.id].update({
                'subcontract': [
                    self.Routing(warehouse.lot_stock_id, subcontract_location_id, warehouse.subcontracting_resupply_type_id, 'pull'),
                ]
            })
        return result

    def _update_global_route_resupply_subcontractor(self):
        route_id = self._find_or_create_global_route('mrp_subcontracting.route_resupply_subcontractor_mto',
                                           _('Resupply Subcontractor on Order'))
        if not route_id.sudo().rule_ids.filtered(lambda r: r.active):
            route_id.active = False
        else:
            route_id.active = True

    def _get_routes_values(self):
        routes = super(StockWarehouse, self)._get_routes_values()
        routes.update({
            'subcontracting_route_id': {
                'routing_key': 'subcontract',
                'depends': ['subcontracting_to_resupply'],
                'route_create_values': {
                    'product_categ_selectable': False,
                    'warehouse_selectable': True,
                    'product_selectable': False,
                    'company_id': self.company_id.id,
                    'sequence': 10,
                    'name': self._format_routename(name=_('Resupply Subcontractor'))
                },
                'route_update_values': {
                    'active': self.subcontracting_to_resupply,
                },
                'rules_values': {
                    'active': self.subcontracting_to_resupply,
                }
            }
        })
        return routes

    def _generate_global_route_rules_values(self):
        rules = super()._generate_global_route_rules_values()
        subcontract_location_id = self._get_subcontracting_location()
        production_location_id = self._get_production_location()
        rules.update({
            'subcontracting_mto_pull_id': {
                'depends': ['subcontracting_to_resupply'],
                'create_values': {
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_or_create_global_route('stock.route_warehouse0_mto', _('Replenish on Order (MTO)')).id,
                    'name': self._format_rulename(self.lot_stock_id, subcontract_location_id, 'MTO'),
                    'location_dest_id': subcontract_location_id.id,
                    'location_src_id': self.lot_stock_id.id,
                    'picking_type_id': self.subcontracting_resupply_type_id.id
                },
                'update_values': {
                    'active': self.subcontracting_to_resupply
                }
            },
            'subcontracting_pull_id': {
                'depends': ['subcontracting_to_resupply'],
                'create_values': {
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_or_create_global_route('mrp_subcontracting.route_resupply_subcontractor_mto', _('Resupply Subcontractor on Order')).id,
                    'name': self._format_rulename(subcontract_location_id, production_location_id, False),
                    'location_dest_id': production_location_id.id,
                    'location_src_id': subcontract_location_id.id,
                    'picking_type_id': self.subcontracting_resupply_type_id.id
                },
                'update_values': {
                    'active': self.subcontracting_to_resupply
                }
            },
        })
        return rules

    def _get_picking_type_create_values(self, max_sequence):
        data, next_sequence = super(StockWarehouse, self)._get_picking_type_create_values(max_sequence)
        data.update({
            'subcontracting_type_id': {
                'name': _('Subcontracting'),
                'code': 'mrp_operation',
                'use_create_components_lots': True,
                'sequence': next_sequence + 2,
                'sequence_code': 'SBC',
                'company_id': self.company_id.id,
            },
            'subcontracting_resupply_type_id': {
                'name': _('Resupply Subcontractor'),
                'code': 'internal',
                'use_create_lots': False,
                'use_existing_lots': True,
                'default_location_dest_id': self._get_subcontracting_location().id,
                'sequence': next_sequence + 3,
                'sequence_code': 'RES',
                'print_label': True,
                'company_id': self.company_id.id,
            }
        })
        return data, max_sequence + 4

    def _get_sequence_values(self, name=False, code=False):
        values = super(StockWarehouse, self)._get_sequence_values(name=name, code=code)
        count = self.env['ir.sequence'].search_count([('prefix', '=like', self.code + '/SBC%/%')])
        values.update({
            'subcontracting_type_id': {
                'name': _('%(name)s Sequence subcontracting', name=self.name),
                'prefix': self.code + '/' + (self.subcontracting_type_id.sequence_code or (('SBC' + str(count)) if count else 'SBC')) + '/',
                'padding': 5,
                'company_id': self.company_id.id
            },
            'subcontracting_resupply_type_id': {
                'name': _('%(name)s Sequence Resupply Subcontractor', name=self.name),
                'prefix': self.code + '/' + (self.subcontracting_resupply_type_id.sequence_code or (('RES' + str(count)) if count else 'RES')) + '/',
                'padding': 5,
                'company_id': self.company_id.id
            },
        })
        return values

    def _get_picking_type_update_values(self):
        data = super(StockWarehouse, self)._get_picking_type_update_values()
        subcontract_location_id = self._get_subcontracting_location()
        production_location_id = self._get_production_location()
        data.update({
            'subcontracting_type_id': {
                'active': False,
                'default_location_src_id': subcontract_location_id.id,
                'default_location_dest_id': production_location_id.id,
            },
            'subcontracting_resupply_type_id': {
                'default_location_src_id': self.lot_stock_id.id,
                'default_location_dest_id': subcontract_location_id.id,
                'barcode': self.code.replace(" ", "").upper() + "RESUP",
                'active': self.subcontracting_to_resupply and self.active
            },
        })
        return data

    def _get_subcontracting_location(self):
        return self.company_id.subcontracting_location_id

    def _get_subcontracting_locations(self):
        return self.env['stock.location'].search([
            ('company_id', 'in', self.company_id.ids),
            ('is_subcontracting_location', '=', True),
        ])

    def _update_subcontracting_locations_rules(self):
        subcontracting_locations = self._get_subcontracting_locations()
        subcontracting_locations._activate_subcontracting_location_rules()

    def _update_resupply_rules(self):
        '''update (archive/unarchive) any warehouse subcontracting location resupply rules'''
        subcontracting_locations = self._get_subcontracting_locations()
        warehouses_to_resupply = self.filtered(lambda w: w.subcontracting_to_resupply and w.active)
        if warehouses_to_resupply:
            self.env['stock.rule'].with_context(active_test=False).search([
                '&', ('picking_type_id', 'in', warehouses_to_resupply.subcontracting_resupply_type_id.ids),
                '|', ('location_src_id', 'in', subcontracting_locations.ids),
                ('location_dest_id', 'in', subcontracting_locations.ids)]).action_unarchive()

        warehouses_not_to_resupply = self - warehouses_to_resupply
        if warehouses_not_to_resupply:
            self.env['stock.rule'].search([
                '&', ('picking_type_id', 'in', warehouses_not_to_resupply.subcontracting_resupply_type_id.ids),
                '|', ('location_src_id', 'in', subcontracting_locations.ids),
                ('location_dest_id', 'in', subcontracting_locations.ids)]).action_archive()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import mrp_bom
from . import product
from . import res_company
from . import res_partner
from . import stock_location
from . import stock_move
from . import stock_move_line
from . import stock_picking
from . import stock_quant
from . import stock_replenish_mixin
from . import stock_rule
from . import stock_warehouse
from . import mrp_production

```

## File: report\mrp_report_bom_structure.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _, fields
from odoo.tools import float_compare


class ReportBomStructure(models.AbstractModel):
    _inherit = 'report.mrp.report_bom_structure'

    def _get_subcontracting_line(self, bom, seller, level, bom_quantity):
        ratio_uom_seller = seller.product_uom.ratio / bom.product_uom_id.ratio
        price = seller.currency_id._convert(seller.price, self.env.company.currency_id, (bom.company_id or self.env.company), fields.Date.today())
        return {
            'name': seller.partner_id.display_name,
            'partner_id': seller.partner_id.id,
            'quantity': bom_quantity,
            'uom': bom.product_uom_id.name,
            'prod_cost': price / ratio_uom_seller * bom_quantity,
            'bom_cost': price / ratio_uom_seller * bom_quantity,
            'level': level or 0
        }

    def _get_bom_data(self, bom, warehouse, product=False, line_qty=False, bom_line=False, level=0, parent_bom=False, parent_product=False, index=0, product_info=False, ignore_stock=False, simulated_leaves_per_workcenter=False):
        res = super()._get_bom_data(bom, warehouse, product, line_qty, bom_line, level, parent_bom, parent_product, index, product_info, ignore_stock, simulated_leaves_per_workcenter)
        if bom.type == 'subcontract' and not self.env.context.get('minimized', False):
            if not res['product']:
                seller = bom.product_tmpl_id.seller_ids.filtered(lambda s: s.partner_id in bom.subcontractor_ids)[:1]
            else:
                seller = res['product']._select_seller(quantity=res['quantity'], uom_id=bom.product_uom_id, params={'subcontractor_ids': bom.subcontractor_ids})
            if seller:
                res['subcontracting'] = self._get_subcontracting_line(bom, seller, level + 1, res['quantity'])
                if not self.env.context.get('minimized', False):
                    res['bom_cost'] += res['subcontracting']['bom_cost']
        return res

    def _get_bom_array_lines(self, data, level, unfolded_ids, unfolded, parent_unfolded):
        lines = super()._get_bom_array_lines(data, level, unfolded_ids, unfolded, parent_unfolded)

        if data.get('subcontracting'):
            subcontract_info = data['subcontracting']
            lines.append({
                'name': _("Subcontracting: %s", subcontract_info['name']),
                'type': 'subcontract',
                'uom': False,
                'quantity': subcontract_info['quantity'],
                'bom_cost': subcontract_info['bom_cost'],
                'prod_cost': subcontract_info['prod_cost'],
                'level': subcontract_info['level'],
                'visible': level == 1 or unfolded or parent_unfolded
            })
        return lines

    @api.model
    def _need_special_rules(self, product_info, parent_bom=False, parent_product=False):
        if parent_bom and parent_product:
            parent_info = product_info.get(parent_product.id, {}).get(parent_bom.id, {})
            return parent_info and parent_info.get('route_type') == 'subcontract'
        return super()._need_special_rules(product_info, parent_bom, parent_product)

    @api.model
    def _find_special_rules(self, product, product_info, current_bom=False, parent_bom=False, parent_product=False):
        res = super()._find_special_rules(product, product_info, current_bom, parent_bom, parent_product)
        if not parent_bom or not parent_product:
            return res
        # If no rules could be found within the warehouse, check if the product is a component from a subcontracted product.
        parent_info = product_info.get(parent_product.id, {}).get(parent_bom.id, {})
        if parent_info and parent_info.get('route_type') == 'subcontract':
            # Since the product is subcontracted, check the subcontracted location for rules instead of the warehouse.
            subcontracting_loc = parent_info['supplier'].partner_id.property_stock_subcontractor
            found_rules = product._get_rules_from_location(subcontracting_loc)
            if found_rules and self._is_resupply_rules(found_rules, current_bom):
                # We only want to show the effective resupply (i.e. a form of manufacture or buy)
                return found_rules
        return res

    @api.model
    def _format_route_info(self, rules, rules_delay, warehouse, product, bom, quantity):
        res = super()._format_route_info(rules, rules_delay, warehouse, product, bom, quantity)
        subcontract_rules = [rule for rule in rules if rule.action == 'buy' and bom and bom.type == 'subcontract']
        if subcontract_rules:
            supplier = product._select_seller(quantity=quantity, uom_id=product.uom_id, params={'subcontractor_ids': bom.subcontractor_ids})
            if not supplier:
                # If no vendor found for the right quantity, we still want to display a vendor for the lead times
                supplier = product._select_seller(quantity=None, uom_id=product.uom_id, params={'subcontractor_ids': bom.subcontractor_ids})
            # for subcontracting, we can't decide the lead time without component's resupply availability
            # we only return necessary info and calculate the lead time late when we have component's data
            if supplier:
                qty_supplier_uom = product.uom_id._compute_quantity(quantity, supplier.product_uom)
                return {
                    'route_type': 'subcontract',
                    'route_name': subcontract_rules[0].route_id.display_name,
                    'route_detail': supplier.display_name,
                    'lead_time': rules_delay,
                    'supplier': supplier,
                    'route_alert': float_compare(qty_supplier_uom, supplier.min_qty, precision_rounding=product.uom_id.rounding) < 0,
                    'qty_checked': quantity,
                    'bom': bom,
                }

        return res

    @api.model
    def _get_quantities_info(self, product, bom_uom, product_info, parent_bom=False, parent_product=False):
        quantities_info = super()._get_quantities_info(product, bom_uom, product_info, parent_bom, parent_product)
        if parent_product and parent_bom and parent_bom.type == 'subcontract' and product.is_storable:
            route_info = product_info.get(parent_product.id, {}).get(parent_bom.id, {})
            if route_info and route_info['route_type'] == 'subcontract':
                subcontracting_loc = route_info['supplier'].partner_id.property_stock_subcontractor
                subloc_product = product.with_context(location=subcontracting_loc.id, warehouse_id=False)
                subloc_product.fetch(['free_qty', 'qty_available'])
                stock_loc = f"subcontract_{subcontracting_loc.id}"
                if not product_info[product.id]['consumptions'].get(stock_loc, False):
                    product_info[product.id]['consumptions'][stock_loc] = 0
                quantities_info['free_to_manufacture_qty'] = product.uom_id._compute_quantity(subloc_product.free_qty, bom_uom)
                quantities_info['free_qty'] = quantities_info['free_to_manufacture_qty']
                quantities_info['on_hand_qty'] = product.uom_id._compute_quantity(subloc_product.qty_available, bom_uom)
                quantities_info['stock_loc'] = stock_loc

        return quantities_info

    @api.model
    def _get_resupply_availability(self, route_info, components):
        resupply_state, resupply_delay = super()._get_resupply_availability(route_info, components)
        if route_info.get('route_type') == 'subcontract':
            max_component_delay = self._get_max_component_delay(components)
            if max_component_delay is False:
                return ('unavailable', False)
            # Calculate the lead time for subcontracting, keep same as `_get_lead_days`
            vendor_lead_time = route_info['supplier'].delay
            manufacture_lead_time = route_info['bom'].produce_delay
            subcontract_delay = resupply_delay if resupply_delay else 0
            subcontract_delay += max(vendor_lead_time, manufacture_lead_time) + max_component_delay
            route_info['manufacture_delay'] = route_info['lead_time'] + max(vendor_lead_time, manufacture_lead_time)
            route_info['lead_time'] += max(vendor_lead_time, manufacture_lead_time + route_info['bom'].days_to_prepare_mo)
            return ('estimated', subcontract_delay)
        return (resupply_state, resupply_delay)

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_report_bom_structure

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_subcontracting_portal_stock_picking,subcontracting.portal.picking,stock.model_stock_picking,base.group_portal,1,0,0,0
access_subcontracting_portal_picking_type,subcontracting.portal.picking.type,stock.model_stock_picking_type,base.group_portal,1,0,0,0
access_subcontracting_portal_stock_move,subcontracting.portal.move,stock.model_stock_move,base.group_portal,1,1,1,0
access_subcontracting_portal_stock_move_line,subcontracting.portal.move.line,stock.model_stock_move_line,base.group_portal,1,1,1,1
access_subcontracting_portal_warehouse,subcontracting.portal.warehouse,stock.model_stock_warehouse,base.group_portal,1,0,0,0
access_subcontracting_portal_lot,subcontracting.portal.lot,stock.model_stock_lot,base.group_portal,1,0,1,0
access_subcontracting_portal_location,subcontracting.portal.location,stock.model_stock_location,base.group_portal,1,0,0,0
access_subcontracting_portal_production,subcontracting.portal.production,mrp.model_mrp_production,base.group_portal,1,1,0,0
access_subcontracting_portal_bom,subcontracting.portal.bom,mrp.model_mrp_bom,base.group_portal,1,0,0,0
access_subcontracting_portal_bom_line,subcontracting.portal.bom.line,mrp.model_mrp_bom_line,base.group_portal,1,0,0,0
access_subcontracting_portal_consumption_warning,subcontracting.portal.consumption.warning,mrp.model_mrp_consumption_warning,base.group_portal,1,1,1,0
access_subcontracting_portal_consumption_warning_line,subcontracting.portal.consumption.warning.line,mrp.model_mrp_consumption_warning_line,base.group_portal,1,1,1,0
access_subcontracting_portal_product,subcontracting.portal.product,product.model_product_product,base.group_portal,1,0,0,0
access_subcontracting_portal_product_template,subcontracting.portal.product.template,product.model_product_template,base.group_portal,1,0,0,0
access_subcontracting_portal_uom,subcontracting.portal.uom,uom.model_uom_uom,base.group_portal,1,0,0,0
access_subcontracting_portal_barcode_nomenclature_stock_user,subcontracting.portal.barcode.nomenclature,barcodes.model_barcode_nomenclature,base.group_portal,1,0,0,0

```

## File: security\mrp_subcontracting_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.rule" id="production_subcontractor_rule">
        <field name="name">MRP Productions Subcontractor</field>
        <field name="model_id" ref="model_mrp_production"/>
        <field name="domain_force">[('subcontractor_id', '=', user.partner_id.commercial_partner_id.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="bom_subcontractor_rule">
        <field name="name">MRP BoMs Subcontractor</field>
        <field name="model_id" ref="mrp.model_mrp_bom"/>
        <field name="domain_force">[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="bom_line_subcontractor_rule">
        <field name="name">MRP BoM Lines Subcontractor</field>
        <field name="model_id" ref="mrp.model_mrp_bom_line"/>
        <field name="domain_force">[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="consumption_warning_subcontractor_rule">
        <field name="name">MRP Consumption Warnings Subcontractor</field>
        <field name="model_id" ref="mrp.model_mrp_consumption_warning"/>
        <field name="domain_force">[('mrp_production_ids', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="consumption_warning_line_subcontractor_rule">
        <field name="name">MRP Consumption Warning Lines Subcontractor</field>
        <field name="model_id" ref="mrp.model_mrp_consumption_warning_line"/>
        <field name="domain_force">[('mrp_production_id', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>
    
    <record model="ir.rule" id="stock_move_subcontractor_rule">
        <field name="name">Stock Moves Subcontractor</field>
        <field name="model_id" ref="stock.model_stock_move"/>
        <field name="domain_force">[
        '|', 
            '|',
                ('production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),
                ('move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),
            ('raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids)
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="stock_move_line_subcontractor_rule">
        <field name="name">Stock Move Lines Subcontractor</field>
        <field name="model_id" ref="model_stock_move_line"/>
        <field name="domain_force">[
        '|', 
            '|',
                ('move_id.production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),
                ('move_id.move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),
            ('move_id.raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="picking_subcontractor_rule">
        <field name="name">Stock Pickings Subcontractor</field>
        <field name="model_id" ref="model_stock_picking"/>
        <field name="domain_force">[('partner_id.commercial_partner_id', '=', user.partner_id.commercial_partner_id.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="picking_type_subcontractor_rule">
        <field name="name">Stock Picking Types Subcontractor</field>
        <field name="model_id" ref="stock.model_stock_picking_type"/>
        <field name="domain_force">['|', ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.ids), ('id', 'in', user.partner_id.commercial_partner_id.production_ids.picking_type_id.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="stock_location_subcontractor_rule">
        <field name="name">Stock Locations Subcontractor</field>
        <field name="model_id" ref="stock.model_stock_location"/>
        <field name="domain_force">[
            '|',
                '|',
                    '|',
                        '|', 
                            ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids), 
                            ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),
                        '|', 
                            ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids), 
                            ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),
                    '|',
                        ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.view_location_id.ids),
                        ('id', 'in', user.partner_id.commercial_partner_id.production_ids.production_location_id.ids),
                ('id', 'in', user.partner_id.commercial_partner_id.production_ids.move_finished_ids.move_dest_ids.location_id.ids),
        ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="stock_warehouse_subcontractor_rule">
        <field name="name">Warehouses Subcontractor</field>
        <field name="model_id" ref="stock.model_stock_warehouse"/>
        <field name="domain_force">[('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record id="stock_lot_subcontracting_rule" model="ir.rule">
        <field name="name">Stock Lot Subcontractor</field>
        <field name="model_id" ref="stock.model_stock_lot"/>
        <field name="domain_force">[
        '|',
            '|',
                ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.ids),
                ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.product_variant_ids.ids),
            ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.ids),
            ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record id="product_template_subcontracting_rule" model="ir.rule">
        <field name="name">Product Template Subcontractor</field>
        <field name="model_id" ref="product.model_product_template"/>
        <field name="domain_force">[
        '|',
            '|',
                ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.product_tmpl_id.ids),
                ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.ids),
            ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.product_tmpl_id.ids),
            ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record id="uom_subcontracting_rule" model="ir.rule">
        <field name="name">UoM Subcontractor</field>
        <field name="model_id" ref="uom.model_uom_uom"/>
        <field name="domain_force">[
        '|',
            '|',
                ('category_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.product_tmpl_id.uom_id.category_id.ids),
                ('category_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.uom_id.category_id.ids),
            ('category_id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.product_tmpl_id.uom_id.category_id.ids),
            ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

</odoo>

```

## File: static\src\components\bom_overview_components_block\mrp_bom_overview_components_block.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { BomOverviewComponentsBlock } from "@mrp/components/bom_overview_components_block/mrp_bom_overview_components_block";
import { BomOverviewSpecialLine } from "@mrp/components/bom_overview_special_line/mrp_bom_overview_special_line";

patch(BomOverviewComponentsBlock, {
    components: { ...BomOverviewComponentsBlock.components, BomOverviewSpecialLine },
});

```

## File: static\src\components\bom_overview_components_block\mrp_bom_overview_components_block.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="mrp_subcontracting.BomOverviewComponentsBlock" t-inherit="mrp.BomOverviewComponentsBlock" t-inherit-mode="extension">
        <xpath expr="//t[@name='byproducts']" position="after">
            <t t-if="data.subcontracting">
                <BomOverviewSpecialLine
                    type="'subcontracting'"
                    showOptions="props.showOptions"
                    data="props.data"
                    precision="props.precision"
                    />
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\bom_overview_line\mrp_bom_overview_line.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { BomOverviewLine } from "@mrp/components/bom_overview_line/mrp_bom_overview_line";

patch(BomOverviewLine.prototype, {
    /**
     * @override
     */
    async goToRoute(routeType) {
        if (routeType == "subcontract") {
            return this.goToAction(this.data.link_id, this.data.link_model);
        }
        return super.goToRoute(...arguments);
    }
});

```

## File: static\src\components\bom_overview_special_line\mrp_bom_overview_special_line.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";
import { BomOverviewSpecialLine } from "@mrp/components/bom_overview_special_line/mrp_bom_overview_special_line";

patch(BomOverviewSpecialLine.prototype, {
    setup() {
        super.setup();
        this.actionService = useService("action");
    },

    //---- Handlers ----

    async goToSubcontractor() {
        return this.actionService.doAction({
            type: "ir.actions.act_window",
            res_model: "res.partner",
            res_id: this.subcontracting.partner_id,
            views: [[false, "form"]],
            target: "current",
            context: {
                active_id: this.subcontracting.partner_id,
            },
        });
    },

    //---- Getters ----

    get subcontracting() {
        return this.props.data.subcontracting || {};
    },
});

```

## File: static\src\components\bom_overview_special_line\mrp_bom_overview_special_line.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="mrp_subcontracting.BomOverviewSpecialLine" t-inherit="mrp.BomOverviewSpecialLine" t-inherit-mode="extension">
        <xpath expr="//td[@name='td_mrp_bom']" position="inside">
            <t t-if="props.type == 'subcontracting'">Subcontracting: <a href="#" t-on-click.prevent="goToSubcontractor" t-esc="subcontracting.name"/></t>
        </xpath>

        <xpath expr="//td[@name='quantity']" position="inside">
            <span t-if="props.type == 'subcontracting'" t-esc="formatFloat(subcontracting.quantity, {'digits': [false, precision]})"/>
        </xpath>

        <xpath expr="//td[@name='uom']" position="inside">
            <span t-if="props.type == 'subcontracting'" t-esc="subcontracting.uom"/>
        </xpath>

        <xpath expr="//td[@name='bom_cost']" position="inside">
            <span t-if="props.type == 'subcontracting'" t-esc="formatMonetary(subcontracting.bom_cost)"/>
        </xpath>

        <xpath expr="//td[@name='prod_cost']" position="inside">
            <span t-if="props.type == 'subcontracting'" t-esc="formatMonetary(subcontracting.prod_cost)"/>
        </xpath>
    </t>

</templates>

```

## File: static\src\img\manufacturing.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M56.3871 19.5678C56.3008 19.5031 52.5034 17.4931 52.5034 17.4931L53.539 14.6235C54.0137 13.2858 53.9058 11.9696 53.237 11.4734L49.9574 8.99214C49.8927 8.94898 38.932 2.64876 38.932 2.64876C38.2847 2.3467 37.3569 2.7998 36.6449 3.7923L35.5877 5.2379L31.553 2.90768L31.4882 2.8861C31.1214 2.67033 30.6683 2.69191 30.1505 2.99399L25.4901 5.66944C24.5623 6.20885 23.7424 7.52498 23.5051 8.86272L22.7067 13.6742C22.4263 13.9763 22.1458 14.2999 21.8653 14.6235L18.9957 13.9547C18.1974 13.7605 17.1617 14.5372 16.5144 15.7886L13.2133 22.0673C12.5444 23.3403 12.4365 24.7643 12.9112 25.5626L14.6589 28.3891C14.5294 28.8206 14.4215 29.2521 14.3137 29.7052L11.0557 32.6396C10.1063 33.5674 9.54535 34.8404 9.5022 36.1565V42.3704C9.5022 43.2551 9.82583 43.8808 10.3652 44.075L13.9685 46.1463L12.9328 49.0159C12.4366 50.3536 12.566 51.6482 13.2133 52.166L27.3241 60.8828C27.993 61.379 29.007 60.9475 29.8054 59.8471L30.8194 58.4447L35.0484 60.8828L35.0915 60.8396C35.4799 60.9691 35.9114 60.9044 36.2566 60.667L40.9171 57.97C41.8449 57.4306 42.6648 56.1144 42.8805 54.7767L43.6788 49.9868C43.9593 49.6848 44.2398 49.3611 44.5203 49.0375L47.3899 49.7063C48.1882 49.9005 49.2239 49.1238 49.8712 47.8724L53.1507 41.5937C53.8196 40.3208 53.9275 38.8967 53.4528 38.0984L51.7267 35.2935C51.8562 34.862 51.964 34.4089 52.0719 33.9774L55.3299 31.043C56.2792 30.1152 56.8402 28.8422 56.8834 27.5261V21.1429C56.9697 20.3877 56.7539 19.8483 56.3871 19.5678Z" fill="#C1DBF6"/>
<path d="M23.8275 7.08959L23.9277 6.89606C24.2771 6.22083 24.7773 5.68223 25.4146 5.29539L30.0791 2.59585C30.3478 2.42465 30.6749 2.32776 30.9919 2.32776C31.2158 2.32776 31.4459 2.38602 31.6233 2.48735L42.2857 8.6479L42.1179 9.0735C42.0121 9.03823 41.9056 9.02038 41.8016 9.02038C41.5704 9.02038 41.3304 9.09203 41.1257 9.22193L36.4567 11.9027C35.9132 12.2525 35.4556 12.7594 35.1436 13.363L35.0333 13.5759L23.8275 7.08959Z" fill="#FBDBD0"/>
<path d="M30.9919 2.5563C31.1645 2.5563 31.3587 2.59943 31.5098 2.68574L42.19 8.85653C42.0605 8.8134 41.931 8.7918 41.8016 8.7918C41.5211 8.7918 41.2406 8.87813 41.0033 9.02917L36.3428 11.7046C35.7387 12.093 35.264 12.6324 34.9404 13.2581L24.1307 7.00099C24.4544 6.3753 24.9291 5.85747 25.5332 5.49066L30.1936 2.79365C30.431 2.64261 30.7115 2.5563 30.9919 2.5563ZM30.992 2.09918C30.6351 2.09918 30.2679 2.20686 29.9569 2.4025L25.3042 5.095C24.6316 5.50325 24.0883 6.088 23.7247 6.79098L23.5244 7.17822L23.9017 7.39665L34.7114 13.6537L35.1262 13.8939L35.3464 13.4681C35.6384 12.9036 36.0651 12.429 36.5807 12.0952L41.2309 9.42563L41.2399 9.42044L41.2487 9.41486C41.4168 9.30789 41.6132 9.24896 41.8016 9.24896C41.8823 9.24896 41.9621 9.26247 42.0455 9.29027C42.0455 9.29027 42.5783 9.3195 43.0355 9.42563C42.9212 8.86235 42.4187 8.46074 42.4187 8.46074L31.7385 2.28995C31.5256 2.16833 31.2542 2.09918 30.992 2.09918Z" fill="#374874"/>
<path d="M12.5497 25.7742L12.5191 25.7226C11.9928 24.8374 12.1117 23.3429 12.8152 22.0041L16.1163 15.7041C16.7582 14.4891 17.7182 13.7045 18.5626 13.7045C18.6656 13.7045 18.7721 13.7045 18.8719 13.7376L18.894 13.7449L31.6926 21.128L29.537 20.407C29.5077 20.3971 29.4294 20.3971 29.3723 20.3971C28.6954 20.3971 27.8752 21.1018 27.3312 22.1507L24.0513 28.4302C23.433 29.6072 23.3029 30.9822 23.7418 31.7005L24.2071 32.4621L12.5497 25.7742Z" fill="#FBDBD0"/>
<path d="M18.5627 13.9331C18.649 13.9331 18.7353 13.9331 18.8 13.9547L29.6097 20.1902C29.5449 20.1686 29.4586 20.1686 29.3723 20.1686C28.6172 20.1686 27.7325 20.8806 27.1284 22.0457L23.8488 28.3244C23.1799 29.5974 23.0721 31.043 23.5468 31.8197L12.7155 25.6058C12.2409 24.8074 12.3487 23.3834 13.0176 22.1104L16.3187 15.8102C16.9229 14.6666 17.8075 13.9331 18.5627 13.9331ZM18.5627 13.476C17.6203 13.476 16.6056 14.2886 15.9145 15.5967L12.6127 21.8983C11.8738 23.3045 11.7572 24.8885 12.3226 25.8394L12.3839 25.9426L12.4881 26.0023L23.3193 32.2162L24.8677 33.1046L23.9368 31.5813C23.5436 30.9378 23.6797 29.6291 24.2535 28.537L27.5336 22.2573C28.1183 21.1299 28.8782 20.6257 29.3723 20.6257C29.4058 20.6257 29.455 20.6257 29.4777 20.6281L33.7613 22.0573L29.8381 19.7942L19.0284 13.5587L18.9884 13.5356L18.9445 13.521C18.8095 13.476 18.6726 13.476 18.5627 13.476Z" fill="#374874"/>
<path d="M10.0667 44.3034C9.44462 44.059 9.07385 43.3465 9.07385 42.392V36.1996C9.11984 34.9978 9.53266 33.8845 10.2679 32.9711L10.3906 32.8186L21.6003 39.2849L21.434 39.4927C20.7841 40.305 20.4034 41.3454 20.362 42.4224L20.3623 48.6275C20.3623 49.2925 20.5644 49.817 20.903 50.0309L20.6858 50.2857L10.0667 44.3034Z" fill="#FBDBD0"/>
<path d="M10.4459 33.1143L21.2555 39.3498C20.5651 40.2128 20.1767 41.2916 20.1336 42.4136V48.6275C20.1336 49.3827 20.3429 49.8376 20.3429 49.8376L10.1654 44.0965C9.64757 43.9023 9.30236 43.2766 9.30236 42.392V36.1996C9.34551 35.0777 9.73388 33.9989 10.4459 33.1143ZM10.3352 32.5226L10.0898 32.8276C9.32359 33.7795 8.89335 34.9395 8.84556 36.182L8.84521 36.1909V42.392C8.84521 43.4362 9.26352 44.224 9.96661 44.5096L20.8 50.4L21.025 49.8376C20.7571 49.6685 20.5907 49.2048 20.5907 48.6275V42.4226C20.632 41.3972 20.9947 40.4077 21.6125 39.6353L21.945 39.2198L21.484 38.9538L10.6743 32.7183L10.3352 32.5226Z" fill="#374874"/>
<path d="M12.9882 52.4559C12.5874 52.1563 12.3484 51.6378 12.2982 50.9842L12.265 50.5521L23.5762 57.0864L23.5851 57.207C23.6242 57.7333 23.7942 58.1237 24.077 58.3358L27.3573 60.8175L27.0991 61.1942L12.9882 52.4559Z" fill="#FBDBD0"/>
<path d="M12.526 50.9669L23.3572 57.224C23.4004 57.8065 23.5946 58.2597 23.9398 58.5185L27.2193 60.9998L13.1085 52.2615C12.7633 52.0026 12.5691 51.5279 12.526 50.9669ZM12.0037 50.1372L12.0702 51.002C12.1256 51.722 12.3969 52.2992 12.8342 52.6272L12.8505 52.6394L12.8679 52.6501L26.9787 61.3885L27.4952 60.6353L24.2156 58.154C23.9256 57.9365 23.8368 57.51 23.8131 57.1903L23.7953 56.9492L23.5859 56.8282L12.7547 50.5711L12.0037 50.1372Z" fill="#374874"/>
<path d="M28.0144 61.2152C27.7562 61.2152 27.5224 61.1431 27.2997 60.9947L24.0313 58.5213C23.3268 57.9882 23.1931 56.6789 23.6987 55.2635L25.4325 50.4791L25.1104 49.6737L21.9612 50.4554C21.8529 50.4726 21.7372 50.4918 21.6063 50.4918C20.8139 50.4918 20.2814 49.7427 20.2814 48.6276V42.4137C20.3229 41.0439 20.9021 39.7338 21.8705 38.8262L25.1117 35.9135L25.4334 34.6519L23.7212 31.8724C23.2247 31.0581 23.3456 29.5468 24.0095 28.2793L27.2892 22.0006C27.8889 20.8613 28.8221 20.0952 29.611 20.0952C29.6973 20.0952 29.7951 20.1148 29.8896 20.1338L32.6911 20.7889L33.4721 19.9039L34.2596 15.1356C34.5114 13.7519 35.344 12.4057 36.2839 11.8657L40.9451 9.1684C41.2386 9.00054 41.5192 8.91862 41.8016 8.91862C42.4321 8.91862 42.8935 9.37219 43.0672 10.1633L43.8393 13.9559L44.5553 13.9184L47.4027 9.96728C47.9725 9.16147 48.6933 8.68134 49.3317 8.68134C49.594 8.68134 49.821 8.75834 50.0069 8.91014L53.2931 11.397C53.9924 11.9407 54.127 13.2505 53.6259 14.6546L51.8884 19.4492L51.9321 19.5369C51.9502 19.5733 51.9672 19.6282 51.9922 19.703C52.0228 19.7948 52.0616 19.9117 52.1174 20.0566L52.19 20.2446L55.3629 19.4626C55.4748 19.4263 55.5821 19.4262 55.7181 19.4262C56.5107 19.4262 57.0431 20.1753 57.0431 21.2905V27.5046C57.0016 28.8742 56.4223 30.1843 55.454 31.0921L52.2141 34.0035L52.1965 34.0684C52.1358 34.2912 51.9822 34.8816 51.9139 35.1774L51.8929 35.2687L53.6032 38.0457C54.0998 38.8599 53.9788 40.3713 53.315 41.6387L50.0565 47.918C49.457 49.057 48.5239 49.8231 47.7351 49.8231C47.6498 49.8231 47.5558 49.8041 47.4565 49.7843L44.6549 49.1292L43.8739 50.0144L43.0864 54.7827C42.8614 56.1325 42.0101 57.5077 41.0621 58.0524L36.3791 60.7713C36.1154 60.9336 35.8132 61.0211 35.5228 61.0211C34.8922 61.0211 34.4311 60.5675 34.2574 59.7765L33.4873 55.9939H32.7735L29.9433 59.9294C29.3735 60.7352 28.6528 61.2152 28.0144 61.2152Z" fill="white"/>
<path d="M49.3317 8.90985C49.5429 8.90985 49.7164 8.96779 49.8621 9.08699L49.8688 9.09252L49.8758 9.09776L53.1505 11.5753C53.7644 12.0528 53.8714 13.2874 53.4113 14.5756L51.7068 19.2792L51.6393 19.4624L51.7271 19.6381C51.7361 19.6574 51.758 19.7233 51.7757 19.7763C51.8056 19.8659 51.8464 19.9885 51.9041 20.1385L52.0488 20.5147L52.4401 20.4182L55.4176 19.6846L55.4355 19.6802L55.4529 19.6744C55.5119 19.6547 55.5985 19.6547 55.7182 19.6547C56.3842 19.6547 56.8146 20.2968 56.8146 21.2905V27.4975C56.7729 28.8112 56.2203 30.0602 55.3048 30.9187L52.1115 33.7883L52.0114 33.8782L51.976 34.008C51.9143 34.2343 51.7592 34.8306 51.6911 35.1259L51.6489 35.3087L51.7473 35.4685L53.4076 38.1637C53.8656 38.9149 53.7415 40.3317 53.1117 41.5341L49.8549 47.8104C49.2933 48.8774 48.4414 49.5943 47.735 49.5943C47.6741 49.5943 47.5937 49.5786 47.5084 49.5615L44.84 48.9375L44.5739 48.8753L44.3931 49.0802L43.7459 49.8138L43.659 49.9122L43.6376 50.0418L42.861 54.7447C42.6462 56.033 41.8418 57.3407 40.9463 57.8552L36.2643 60.5738L36.2592 60.5767L36.2542 60.5797C36.0346 60.7149 35.768 60.7924 35.5229 60.7924C34.8195 60.7924 34.568 60.1252 34.4821 59.7344L33.7485 56.1311L33.674 55.7651H32.6563L32.5195 55.9554L29.7556 59.7989C29.2296 60.5426 28.5787 60.9866 28.0144 60.9866C27.8067 60.9866 27.6177 60.9292 27.437 60.8114L24.1692 58.339C23.5447 57.8664 23.4421 56.6614 23.9132 55.3424L25.6177 50.6388L25.6771 50.475L25.6123 50.3133L25.3966 49.7738L25.2486 49.404L24.862 49.5L21.9234 50.2293C21.8119 50.2478 21.7139 50.2632 21.6063 50.2632C20.9403 50.2632 20.5099 49.6212 20.5099 48.6275V42.4205C20.5515 41.1069 21.1042 39.8578 22.0197 38.9993L25.213 36.1297L25.3162 36.037L25.3504 35.9026L25.6309 34.8022L25.6787 34.6145L25.5771 34.4495L23.9169 31.7543C23.4588 31.0031 23.583 29.5863 24.2123 28.385L27.4912 22.1076C28.0528 21.0405 28.9047 20.3236 29.611 20.3236C29.672 20.3236 29.7523 20.3394 29.8376 20.3564L32.506 20.9805L32.7721 21.0427L32.9529 20.8378L33.6002 20.1042L33.687 20.0057L33.7084 19.8762L34.4839 15.1799C34.7211 13.8752 35.5261 12.5647 36.3992 12.0632L41.0574 9.36738C41.3168 9.2192 41.5602 9.14713 41.8016 9.14713C42.505 9.14713 42.7565 9.81431 42.8424 10.2052L43.576 13.8085L43.6545 14.1945L44.048 14.1738L44.4579 14.1522L44.6766 14.1406L44.8047 13.9629L47.5904 10.0975C48.1164 9.3538 48.7674 8.90985 49.3317 8.90985ZM49.3316 8.45266C48.6196 8.45266 47.8429 8.94891 47.2172 9.83355L44.4338 13.6957L44.0239 13.7173L43.2903 10.114C43.0961 9.22939 42.5567 8.68999 41.8015 8.68999C41.4563 8.68999 41.1327 8.79787 40.8306 8.97049L36.1702 11.6675C35.1561 12.2501 34.293 13.6741 34.0341 15.0981L33.2574 19.8017L32.6101 20.5353L29.9346 19.9096C29.8267 19.888 29.7189 19.8664 29.611 19.8664C28.7264 19.8664 27.7339 20.6648 27.0866 21.8946L23.807 28.1733C23.095 29.5326 22.9871 31.1076 23.5265 31.9922L25.1879 34.6893L24.9074 35.7896L21.7141 38.6593C20.6784 39.6302 20.0959 40.9895 20.0527 42.4135V48.6274C20.0527 49.8789 20.6784 50.7203 21.6062 50.7203C21.7573 50.7203 21.8867 50.6988 22.0162 50.6772L24.9721 49.9436L25.1879 50.483L23.4833 55.1866C22.9439 56.6969 23.095 58.0994 23.8933 58.7035L27.1728 61.1848C27.4318 61.3574 27.7123 61.4437 28.0143 61.4437C28.7263 61.4437 29.5031 60.9474 30.1288 60.0628L32.8905 56.2223H33.3005L34.0341 59.8255C34.2283 60.7101 34.7676 61.2495 35.5228 61.2495C35.868 61.2495 36.2133 61.1416 36.4937 60.969L41.1758 58.2504C42.1899 57.6679 43.0745 56.2439 43.3118 54.8198L44.0886 50.1162L44.7359 49.3826L47.4113 50.0083C47.5192 50.0299 47.6271 50.0515 47.735 50.0515C48.6196 50.0515 49.6121 49.2531 50.2594 48.0233L53.5174 41.7447C54.2294 40.3853 54.3372 38.8103 53.7979 37.9257L52.1365 35.2287C52.2012 34.9482 52.3522 34.3656 52.417 34.1283L55.6102 31.2586C56.6459 30.2877 57.2285 28.9284 57.2716 27.5044V21.2905C57.2716 20.0391 56.6459 19.1976 55.7181 19.1976C55.5671 19.1976 55.4376 19.1976 55.3082 19.2407L52.3307 19.9743C52.2228 19.6938 52.1796 19.5212 52.1365 19.4349L53.841 14.7313C54.3804 13.221 54.2078 11.8185 53.431 11.2144L50.1515 8.73314C49.9141 8.53895 49.6337 8.45266 49.3316 8.45266Z" fill="#374874"/>
<path d="M44.1519 31.8657C44.1519 35.8041 41.7486 40.3937 38.8025 42.0993C35.8565 43.8049 33.4531 41.9908 33.4531 38.0524C33.4531 34.114 35.8565 29.5243 38.8025 27.8187C41.7486 26.1131 44.1519 27.9273 44.1519 31.8657Z" fill="#FBDBD0"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M42.4441 27.908C41.5687 27.2462 40.3352 27.1954 38.917 28.0165C37.4929 28.841 36.1818 30.3758 35.2255 32.2042C34.2703 34.0306 33.6816 36.1264 33.6816 38.0523C33.6816 39.9835 34.2712 41.3374 35.1608 42.01C36.0362 42.6717 37.2697 42.7225 38.6879 41.9015C40.112 41.077 41.4231 39.5422 42.3794 37.7138C43.3346 35.8874 43.9233 33.7916 43.9233 31.8656C43.9233 29.9345 43.3337 28.5806 42.4441 27.908ZM42.7198 27.5433C43.7683 28.3361 44.3804 29.8584 44.3804 31.8656C44.3804 33.8781 43.7674 36.0463 42.7845 37.9257C41.8025 39.8031 40.439 41.416 38.917 42.2971C37.3891 43.1816 35.948 43.1782 34.8851 42.3747C33.8366 41.5819 33.2245 40.0596 33.2245 38.0523C33.2245 36.0398 33.8375 33.8717 34.8205 31.9923C35.8024 30.1149 37.1659 28.502 38.6879 27.6209C40.2158 26.7364 41.6569 26.7398 42.7198 27.5433Z" fill="#374874"/>
<path d="M38.5004 2.55629C38.6532 2.55629 38.7986 2.58649 38.9319 2.64873C38.9319 2.64873 49.0943 8.25851 49.3316 8.45273L53.4311 11.2145C54.2079 11.8186 54.3805 13.221 53.8411 14.7314L52.7822 17.6533C53.3375 17.9721 55.6331 19.1976 55.7182 19.1976C56.646 19.1976 57.2717 20.0391 57.2717 21.2905V27.5045C57.2285 28.9285 56.646 30.2878 55.6103 31.2587L52.417 34.1283C52.3523 34.3657 52.2012 34.9482 52.1365 35.2287L53.7979 37.9257C54.3373 38.8103 54.2294 40.3854 53.5174 41.7447L50.2594 48.0234C49.6121 49.2532 48.6196 50.0515 47.735 50.0515C47.6271 50.0515 47.5192 50.0299 47.4113 50.0084L44.7359 49.3827L44.0886 50.1163L43.3119 54.8199C43.0745 56.2439 42.1899 57.6679 41.1758 58.2505L36.4938 60.9691C36.2133 61.1416 35.8681 61.2495 35.5229 61.2495C34.9552 61.2495 34.5096 60.9445 34.2408 60.4173L31.1538 58.6375L30.1289 60.0628C29.5031 60.9475 28.7264 61.4437 28.0144 61.4437C27.7123 61.4437 27.4318 61.3574 27.1729 61.1849L25.6161 60.007L13.1085 52.2614C12.7633 52.0025 12.5691 51.5278 12.526 50.9669C12.526 50.9669 12.662 49.7454 12.9327 49.0159L13.9249 46.2666L10.1654 44.0965C9.64756 43.9023 9.30235 43.2766 9.30235 42.3919V36.1996C9.3455 35.0777 9.73387 33.9988 10.4459 33.1142L14.3135 29.7052C14.4214 29.2521 14.5293 28.8206 14.6588 28.389L12.7155 25.6057C12.2409 24.8074 12.3487 23.3834 13.0176 22.1103L16.3187 15.8101C16.9229 14.6666 17.8075 13.933 18.5626 13.933C18.5811 13.933 18.5986 13.9339 18.6168 13.9341C18.618 13.934 18.6195 13.9339 18.6213 13.9339C18.8068 13.934 21.8652 14.6234 21.8652 14.6234C22.1457 14.2998 22.4262 13.9762 22.7066 13.674L23.505 8.8626C23.6187 8.22164 24.1307 7.00091 24.1307 7.00091C24.4543 6.37522 24.929 5.85734 25.5331 5.49058L30.1936 2.79353C30.4309 2.64249 30.7114 2.55622 30.9919 2.55622C31.1645 2.55622 31.3587 2.59935 31.5097 2.68566L35.6884 5.09997L36.6449 3.79218C37.2101 3.00418 37.9114 2.55629 38.5004 2.55629ZM38.5004 1.64189C37.5864 1.64189 36.6393 2.2314 35.902 3.25924L35.435 3.89764L31.9671 1.89398C31.6813 1.73067 31.3363 1.64191 30.9919 1.64191C30.5464 1.64191 30.1075 1.76934 29.721 2.01067L25.0752 4.69919L25.0669 4.70402L25.0587 4.70902C24.3226 5.1559 23.7209 5.80317 23.3186 6.58082L23.3018 6.61341L23.2876 6.64726C23.2311 6.7818 22.7325 7.98255 22.6047 8.70287L21.8491 13.2568C21.7433 13.3738 21.638 13.4927 21.5335 13.6118C18.8882 13.0197 18.7292 13.0197 18.6217 13.0196H18.6186C18.5965 13.019 18.5798 13.0187 18.5626 13.0187C17.4503 13.0187 16.2808 13.9246 15.5103 15.383L12.2077 21.686C11.398 23.2271 11.286 24.9905 11.9297 26.0729L11.9467 26.1016L11.9659 26.129L13.6603 28.556C13.5996 28.7791 13.5442 28.9979 13.4923 29.2102L9.84134 32.4283L9.78269 32.48L9.73366 32.5409C8.90539 33.57 8.44034 34.8229 8.38875 36.1644L8.38806 36.182V42.3919C8.38806 43.5965 8.9125 44.5555 9.76303 44.9199L12.8052 46.6759L12.0727 48.7054C11.7745 49.5092 11.6325 50.7291 11.6173 50.8656L11.6078 50.9511L11.6144 51.0369C11.68 51.8901 12.0158 52.5847 12.56 52.9928L12.5926 53.0172L12.6272 53.0387L25.0985 60.7618L26.6213 61.9139L26.6431 61.9304L26.6659 61.9456C27.0765 62.2192 27.5302 62.358 28.0144 62.358C29.0382 62.358 30.081 61.7138 30.8753 60.5907L31.4134 59.8424C32.1581 60.2718 34.174 61.4518 34.6299 61.7545C35.2986 62.1985 36.2875 62.1649 36.9618 61.7545L41.635 59.0411C42.8905 58.3198 43.9283 56.6829 44.2137 54.9701L44.9479 50.5242L45.0599 50.3973L47.2032 50.8985L47.2176 50.9019L47.2324 50.9049C47.368 50.9319 47.5367 50.9657 47.735 50.9657C48.9742 50.9657 50.2515 50.0014 51.0685 48.4491L54.3289 42.1657C55.1921 40.5179 55.293 38.6214 54.5785 37.4496L53.1153 35.0744C53.1527 34.926 53.194 34.7656 53.2305 34.6264L56.2214 31.9386L56.2286 31.9322L56.2356 31.9256C57.4239 30.8116 58.1347 29.2102 58.1855 27.5321L58.1859 27.5182V21.2904C58.1859 19.6057 57.2229 18.9235 55.9265 18.2919C55.5773 18.1218 54.6126 17.6119 53.9079 17.2302L54.7006 15.0428C55.3802 13.1399 55.1017 11.3554 53.9924 10.4926L53.9678 10.4735L53.9419 10.4561L49.8425 7.69435L49.8415 7.69579C49.5726 7.52132 48.1828 6.71087 39.3738 1.84818L39.3466 1.83318L39.3186 1.82006C39.0652 1.70182 38.7899 1.64189 38.5004 1.64189Z" fill="#374874"/>
</svg>

```

## File: static\src\subcontracting_portal\main.js

```javascript
/** @odoo-module **/
import { startWebClient } from "@web/start";
import { SubcontractingPortalWebClient } from "./subcontracting_portal";
import { registry } from "@web/core/registry";

const servicesToRemove = ["menu"];

const servicesRegistry = registry.category("services");

/**
 * Remove services unsued in subcontracting portal feature.
 *
 * This function is used before starting the webclient
 * to remove the services that we don't want in the registry.
 * In this case, the home_menu service is removed via the assets
 * but the services in web_studio depends on this service and are not removed.
 * Since this module has not web_studio module in this dependencies, this function will remove
 * the services that we don't want instead of create a new module just to remove the services in assets.
 */
export function removeServices() {
    for (const service of servicesToRemove) {
        if (servicesRegistry.contains(service)) {
            servicesRegistry.remove(service);
        }
    }
}

removeServices();
startWebClient(SubcontractingPortalWebClient);

```

## File: static\src\subcontracting_portal\move_list_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";

const MoveListView = {
    ...listView,
    searchMenuTypes: [],
};

registry.category("views").add('subcontracting_portal_move_list_view', MoveListView);

```

## File: static\src\subcontracting_portal\picking_form_controller.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { FormController } from "@web/views/form/form_controller";
import { formView } from "@web/views/form/form_view";


class PickingFormController extends FormController {
    static template = "mrp_subcontracting.PickingFormController";
}

const PickingFormView = {
    ...formView,
    Controller: PickingFormController,
};

registry.category("views").add("subcontracting_portal_picking_form_view", PickingFormView);

```

## File: static\src\subcontracting_portal\picking_form_controller.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-name="mrp_subcontracting.PickingFormController" t-inherit="web.FormView" t-inherit-mode="primary">
        <xpath expr="//t[@t-set-slot='control-panel-additional-actions']" position="replace"/>
    </t>

</templates>

```

## File: static\src\subcontracting_portal\subcontracting_portal.js

```javascript
/** @odoo-module **/

import { useService } from '@web/core/utils/hooks';
import { ActionContainer } from '@web/webclient/actions/action_container';
import { MainComponentsContainer } from "@web/core/main_components_container";
import { useOwnDebugContext } from "@web/core/debug/debug_context";
import { session } from '@web/session';
import { Component, useEffect, useExternalListener } from "@odoo/owl";

export class SubcontractingPortalWebClient extends Component {
    static components = { ActionContainer, MainComponentsContainer };
    static template = "mrp_subcontracting.SubcontractingPortalWebClient";
    static props = {};
    setup() {
        window.parent.document.body.style.margin = "0"; // remove the margin in the parent body
        this.actionService = useService('action');
        useOwnDebugContext({ categories: ["default"] });
        useEffect(
            () => {
                this._showView();
            },
            () => []
        );
        useExternalListener(window, "click", this.onGlobalClick, { capture: true });
    }

    async _showView() {
        const { action_name, picking_id } = session;
        await this.actionService.doAction(
            action_name,
            {
                props: {
                    resId: picking_id,
                    preventEdit: true,
                    preventCreate: true,
                },
                additionalContext: {
                    no_breadcrumbs: true,
                }
            }
        );
    }

    /**
     * @param {MouseEvent} ev
     */
     onGlobalClick(ev) {
        // When a ctrl-click occurs inside an <a href/> element
        // we let the browser do the default behavior and
        // we do not want any other listener to execute.
        if (
            ev.ctrlKey &&
            ((ev.target instanceof HTMLAnchorElement && ev.target.href) ||
                (ev.target instanceof HTMLElement && ev.target.closest("a[href]:not([href=''])")))
        ) {
            ev.stopImmediatePropagation();
            return;
        }
    }
}


```

## File: static\src\subcontracting_portal\subcontracting_portal.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-name="mrp_subcontracting.SubcontractingPortalWebClient">
        <ActionContainer/>
        <MainComponentsContainer/>
    </t>

</templates>

```

## File: static\src\views\stock_move_one2many.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="stock.MovesListRenderer.RecordRow" t-inherit-mode="extension">
        <xpath expr="//t[starts-with(@t-if, 'record.resModel') and .//button[@class='btn btn-link fa fa-list']]" position="attributes">
            <attribute name="t-if">column.type === 'opendetailsop' and !record.data.is_subcontract </attribute>
        </xpath>
    </t>
</templates>

```

## File: views\mrp_bom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_bom_form_view" model="ir.ui.view">
        <field name="name">mrp.bom.form.view</field>
        <field name="model">mrp.bom</field>
        <field name="inherit_id" ref="mrp.mrp_bom_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='type']" position="after">
                <field name="subcontractor_ids" widget="many2many_tags" invisible="type != 'subcontract'" required="type == 'subcontract'"/>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: views\mrp_production_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_production_subcontracting_form_view" model="ir.ui.view">
        <field name="name">mrp.production.subcontracting.form.view</field>
        <field name="model">mrp.production</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//header" position="replace">
                <field name="state" invisible="1"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="replace"/>
            <xpath expr="//div[hasclass('oe_title')]" position="replace"/>
            <xpath expr="//button[@name='action_generate_bom']" position="replace"/>
            <xpath expr="//field[@name='is_outdated_bom']" position="replace"/>
            <xpath expr="//button[@name='action_update_bom']" position="replace"/>
            <xpath expr="//group[@name='group_extra_info']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//page[@name='operations']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='lot_producing_id']" position="attributes">
                <attribute name="invisible">product_tracking in ('none', False)</attribute>
                <attribute name="required">product_tracking not in ('none', False)</attribute>
            </xpath>
            <xpath expr="//page[@name='miscellaneous']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//label[@name='bom_label']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//div[@name='bom_div']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='move_byproduct_ids']" position="replace"/>
            <xpath expr="//field[@name='workorder_ids']" position="replace"/>
            <xpath expr="//button[@name='action_generate_serial']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='move_finished_ids']" position="replace"/>
            <xpath expr="//field[@name='move_raw_ids']" position="replace">
                <field name="bom_product_ids" invisible="1"/>
                <field name="move_line_raw_ids" force_save="1"
                    context="{'list_view_ref': 'mrp_subcontracting.mrp_subcontracting_stock_move_line_tree_view', 'default_company_id': company_id, 'default_location_id': location_src_id, 'default_location_dest_id': production_location_id, 'bom_product_ids': bom_product_ids}"
                    />
            </xpath>
            <xpath expr="//sheet" position="inside">
                <footer>
                    <button name="subcontracting_record_component" invisible="state == 'to_close' or qty_producing &lt; 0.001" string="Continue" type="object" class="oe_highlight" data-hotkey="q"/>
                    <button name="subcontracting_record_component" invisible="state != 'to_close'" string="Record Production" type="object" class="oe_highlight" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="x" />
                </footer>
            </xpath>
            <xpath expr="//chatter" position="replace"/>
        </field>
    </record>

    <record id="mrp_production_subcontracting_portal_form_view" model="ir.ui.view">
        <field name="name">mrp.production.subcontracting.portal.form.view</field>
        <field name="model">mrp.production</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mrp_production_subcontracting_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="attributes">
                <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
            </xpath>
            <xpath expr="//field[@name='product_qty']" position="attributes">
                <attribute name="readonly">True</attribute>
                <attribute name="invisible">False</attribute>
            </xpath>
            <xpath expr="//field[@name='lot_producing_id']" position="attributes">
                <attribute name="domain">[('id', '=', False)]</attribute>
                <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
            </xpath>
            <xpath expr="//field[@name='move_line_raw_ids']" position="attributes">
                <attribute name="context">{'list_view_ref': 'mrp_subcontracting.mrp_subcontracting_portal_stock_move_line_tree_view', 'default_company_id': company_id, 'default_location_id': location_src_id, 'default_location_dest_id': production_location_id, 'bom_product_ids': bom_product_ids}</attribute>
            </xpath>
            <xpath expr="//field[@name='forecasted_issue']" position="replace"/>
            <xpath expr="//button[@name='%(mrp.action_change_production_qty)d']" position="replace"/>
            <xpath expr="//button[@name='action_product_forecast_report']" position="replace"/>
            <xpath expr="//button[@name='action_product_forecast_report']" position="replace"/>
        </field>
    </record>

    <record id="mrp_production_subcontracting_tree_view" model="ir.ui.view">
        <field name="name">mrp.production.subcontracting.list</field>
        <field name="model">mrp.production</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mrp.mrp_production_tree_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="incoming_picking"/>
            </xpath>
        </field>
    </record>

    <record id="mrp_production_subcontracting_filter" model="ir.ui.view">
        <field name="name">mrp.production.subcontracting.select</field>
        <field name="model">mrp.production</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mrp.view_mrp_production_filter" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="name" string="Incoming transfer" filter_domain="[('incoming_picking.name', 'ilike', self)]"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_mrp_subcontracting_form" model="ir.ui.view">
        <field name="name">res.partner.mrp_subcontracting.property.form.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="stock.view_partner_stock_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='property_stock_supplier']" position="after">
                <field name="property_stock_subcontractor"/>
                <separator/>
            </xpath>
        </field>
    </record>
    <record id="view_partner_mrp_subcontracting_filter" model="ir.ui.view">
        <field name="name">res.partner.select.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_res_partner_filter" />
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='inactive']" position="before">
                <filter string="Subcontractors" name="type_subcontractors" domain="[('is_subcontractor', '=', True)]" groups="mrp.group_mrp_user"/>
                <separator groups="mrp.group_mrp_user"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_location_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_location_form" model="ir.ui.view">
        <field name="name">stock.location.form</field>
        <field name="model">stock.location</field>
        <field name="inherit_id" ref="stock.view_location_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='scrap_location']" position='after'>
                <field name="is_subcontracting_location" groups="base.group_no_one" invisible="usage != 'internal'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="mrp_subcontracting_stock_move_line_tree_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.stock.move.line.list.view</field>
        <field name="model">stock.move.line</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <list editable="bottom">
                <field name="company_id" column_invisible="True"/>
                <field name="product_uom_category_id" column_invisible="True"/>
                <field name="owner_id" column_invisible="True"/>
                <field name="tracking" column_invisible="True"/>
                <field name="package_id" column_invisible="True"/>
                <field name="result_package_id" column_invisible="True"/>
                <field name="location_id" column_invisible="True"/>
                <field name="location_dest_id" column_invisible="True"/>
                <field name="state" column_invisible="True"/>
                <!-- Don't put move_id here to avoid that the framework send falsy move_id -->
                <field name="id" column_invisible="True"/>
                <field name="product_id" required="1" domain="[('id', 'in', context.get('bom_product_ids'))] if context.get('is_subcontracting_portal') else []"/>
                <field name="lot_id" groups="stock.group_production_lot"
                    invisible="tracking not in ('serial', 'lot')"
                    required="tracking in ('serial', 'lot')"
                    context="{'default_product_id': product_id}"/>
                <field name="quantity"/>
                <field name="product_uom_id" groups="uom.group_uom"/>
            </list>
        </field>
    </record>
    <record id="mrp_subcontracting_portal_stock_move_line_tree_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.portal.stock.move.line.list.view</field>
        <field name="model">stock.move.line</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mrp_subcontracting_stock_move_line_tree_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lot_id']" position="attributes">
                <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
            </xpath>
            <xpath expr="//field[@name='product_id']" position="attributes">
                <attribute name="options">{'no_open': True}</attribute>
            </xpath>
        </field>
    </record>
    <record id="mrp_subcontracting_view_stock_move_line_operation_tree" model="ir.ui.view">
        <field name="name">mrp.subcontracting.stock.move.line.operations.list</field>
        <field name="model">stock.move.line</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="stock.view_stock_move_line_operation_tree" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lot_id']" position="attributes">
                <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
            </xpath>
        </field>
    </record>
    <record id="mrp_subcontracting_view_stock_move_operations" model="ir.ui.view">
        <field name="name">mrp.subcontracting.stock.move.operations.form</field>
        <field name="model">stock.move</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="stock.view_stock_move_operations" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='move_line_ids']" position="attributes">
                <attribute name="context">{'list_view_ref': 'mrp_subcontracting.mrp_subcontracting_view_stock_move_line_operation_tree', 'default_product_uom_id': product_uom, 'default_picking_id': picking_id, 'default_move_id': id, 'default_product_id': product_id, 'default_location_id': location_id, 'default_location_dest_id': location_dest_id, 'default_company_id': company_id}</attribute>
            </xpath>
        </field>
    </record>

    <record id="mrp_subcontracting_move_form_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.move.form.view</field>
        <field name="model">stock.move</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form create="0" delete="0">
                <header>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <field name="product_uom_category_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <field name="product_id" invisible="1" readonly="state == 'done'"/>
                    <field name="sequence" invisible="1"/>
                    <field name="location_id" invisible="1"/>
                    <field name="picking_id" invisible="1" readonly="state == 'done'"/>
                    <field name="location_dest_id" invisible="1"/>
                    <field name="has_tracking" invisible="1"/>
                    <field name="product_uom" invisible="1"/>
                    <field name="product_uom_qty" invisible="1" readonly="state == 'done'"/>
                    <group>
                        <field name="order_finished_lot_id"/>
                        <field name="product_uom" groups="uom.group_uom"/>
                        <field name="quantity" string="Total Consumed" readonly="1"/>
                    </group>
                    <field name="move_line_ids"
                        readonly="state in ['done', 'cancel']"
                        context="{'default_product_uom_id': product_uom, 'default_picking_id': picking_id, 'default_move_id': id, 'default_product_id': product_id, 'default_location_id': location_id, 'default_location_dest_id': location_dest_id, 'default_company_id': company_id}">
                        <list editable="bottom" decoration-muted="state in ('done', 'cancel')">
                            <field name="company_id" column_invisible="True"/>
                            <field name="state" column_invisible="True"/>
                            <field name="tracking" column_invisible="True"/>
                            <field name="product_uom_id" column_invisible="True"/>
                            <field name="product_uom_category_id" column_invisible="True"/>
                            <field name="picking_id" column_invisible="True"/>
                            <field name="move_id" column_invisible="True"/>
                            <field name="location_id" column_invisible="True"/>
                            <field name="location_dest_id" column_invisible="True"/>
                            <field name="product_id" readonly="1" force_save="1"/>
                            <field name="quantity"/>
                            <field name="lot_id" column_invisible="parent.has_tracking not in ('serial', 'lot')" required="tracking in ('serial', 'lot')" context="{'default_product_id': product_id}" groups="stock.group_production_lot"/>
                        </list>
                    </field>
                </sheet>
            </form>
        </field>
    </record>
    <record id="mrp_subcontracting_portal_move_form_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.portal.move.form.view</field>
        <field name="model">stock.move</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mrp_subcontracting_move_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='order_finished_lot_id']" position="attributes">
                <attribute name="options">{'no_open': True}</attribute>
            </xpath>
            <xpath expr="//list" position="attributes">
                <attribute name="no_open">1</attribute>
            </xpath>
            <xpath expr="//list/field[@name='lot_id']" position="attributes">
                <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
            </xpath>
        </field>
    </record>
    <record id="mrp_subcontracting_move_tree_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.move.list.view</field>
        <field name="model">stock.move</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <list delete="0" create="0" decoration-muted="is_done" decoration-warning="quantity - product_uom_qty &gt; 0.0001" decoration-success="not is_done and quantity - product_uom_qty &lt; 0.0001" js_class="subcontracting_portal_move_list_view">
                <field name="company_id" column_invisible="True"/>
                <field name="sequence" column_invisible="True"/>
                <field name="product_uom_category_id" column_invisible="True"/>
                <field name="name" column_invisible="True"/>
                <field name="unit_factor" column_invisible="True"/>
                <field name="date" column_invisible="True"/>
                <field name="picking_type_id" column_invisible="True"/>
                <field name="has_tracking" column_invisible="True"/>
                <field name="operation_id" column_invisible="True"/>
                <field name="is_done" column_invisible="True"/>
                <field name="bom_line_id" column_invisible="True"/>
                <field name="location_id" column_invisible="True"/>
                <field name="warehouse_id" column_invisible="True"/>
                <field name="product_uom_qty" column_invisible="True" readonly="state == 'done'"/>
                <field name="location_dest_id" column_invisible="True"/>
                <field name="state" column_invisible="True" force_save="1"/>
                <field name="raw_material_production_id" column_invisible="True"/>
                <field name="product_id" required="1" readonly="state == 'done'"/>
                <field name="order_finished_lot_id"/>
                <field name="quantity" string="Consumed" readonly="1"/>
                <field name="product_uom" groups="uom.group_uom"/>
            </list>
        </field>
    </record>
    <record id="view_move_search" model="ir.ui.view">
        <field name="name">stock.move.search</field>
        <field name="model">stock.move</field>
        <field name="inherit_id" ref="stock.view_move_search" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="order_finished_lot_id"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_picking_form_view" model="ir.ui.view">
        <field name="name">stock.picking.form.view</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form" />
        <field name="arch" type="xml">
            <xpath expr="//button[@name='button_validate'][hasclass('o_btn_validate')]" position="before">
                <field name="display_action_record_components" invisible="1"/>
                <button name="action_record_components" class="oe_highlight" invisible="display_action_record_components != 'mandatory'" string="Record components" type="object" data-hotkey="shift+x"/>
                <button name="action_record_components" invisible="display_action_record_components != 'facultative'" string="Record components" type="object" data-hotkey="shift+x"/>
            </xpath>
            <xpath expr="//field[@name='move_ids_without_package']//list" position="inside">
                <field name="show_subcontracting_details_visible" column_invisible="True"/>
                <button name="action_show_subcontract_details" string="Subcontracting" type="object" icon="fa-pencil"
                    invisible="not show_subcontracting_details_visible"/>
            </xpath>
            <xpath expr="//field[@name='move_ids_without_package']//list" position="inside">
                <field name="is_subcontract" column_invisible="True"/>
                <button name="action_show_details" invisible="not is_subcontract" type="object" icon="fa-list" width="0.1" title="Details"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_quant_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="quant_subcontracting_search_view" model="ir.ui.view">
            <field name="name">stock.quant.subcontracting.search</field>
            <field name="model">stock.quant</field>
            <field name="inherit_id" ref="stock.quant_search_view" />
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='transit_loc']" position="after">
                    <filter string="Subcontracting Locations" name="is_subcontract" domain="[('is_subcontract', '=', True)]" groups="mrp.group_mrp_user"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>


```

## File: views\stock_warehouse_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_warehouse_inherit_mrp_subcontracting" model="ir.ui.view">
        <field name="name">Stock Warehouse Inherit Subcontracting</field>
        <field name="model">stock.warehouse</field>
        <field name="inherit_id" ref="mrp.view_warehouse_inherit_mrp"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='manufacture_to_resupply']" position="before">
                <field name="subcontracting_to_resupply" />
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\subcontracting_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="portal_my_home_productions" name="Productions" customize_show="True" inherit_id="portal.portal_my_home" priority="20">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
            <t t-set="portal_vendor_category_enable" t-value="True"/>
        </xpath>
        <div id="portal_vendor_category" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/mrp_subcontracting/static/src/img/manufacturing.svg'"/>
                <t t-set="title">Manufacturing Orders</t>
                <t t-set="text">Follow manufacturing orders you have to fulfill</t>
                <t t-set="url" t-value="'/my/productions'"/>
                <t t-set="placeholder_count" t-value="'production_count'"/>
            </t>
        </div>
    </template>

    <template id="portal_my_home_menu_production" name="Portal layout : production menu entries" inherit_id="portal.portal_breadcrumbs" priority="25">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <t t-if="title">
                <li t-if="page_name == 'production' or picking" t-attf-class="breadcrumb-item #{'active' if breadcrumbs_url else ''}">
                    <a t-if="breadcrumbs_url" t-attf-href="/my/productions{{ keep_query() }}">Productions</a>
                    <t t-else="" t-esc="title"></t>
                </li>
                <li t-if="picking" class="breadcrumb-item active">
                    <t t-out="picking.name"/>
                </li>
            </t>
        </xpath>
    </template>

    <template id="portal_my_productions" name="My Productions">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Productions</t>
            </t>

            <t t-if="not pickings">
                <p class="alert alert-warning">There are currently no productions for your account.</p>
            </t>

            <t t-if="pickings" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>Order</th>
                        <th class="text-end">Source Document</th>
                        <th class="text-end">Scheduled Date</th>
                        <th class="text-end">Deadline Date</th>
                        <th class="text-end">State</th>
                    </tr>
                </thead>
                <t t-foreach="pickings" t-as="picking">
                    <tr>
                        <td><a t-attf-href="/my/productions/#{picking.id}?{{ keep_query() }}"><span t-field="picking.name"/></a></td>
                        <td class="text-end"><span t-field="picking.origin"/></td>
                        <td class="text-end"><span t-field="picking.scheduled_date" t-options='{"widget": "date"}'/></td>
                        <td class="text-end"><span t-field="picking.date_deadline" t-options='{"widget": "date"}'/></td>
                        <td class="text-end"><span t-field="picking.state"/></td>
                    </tr>
                </t>
            </t>
        </t>
    </template>

    <template id="subcontracting_portal" name="Subcontracting View in Portal">
        <t t-call="portal.frontend_layout">
            <div class="o_portal container mt-3">
                <t t-call="mrp_subcontracting.portal_my_home_menu_production">
                    <t t-set="title" t-value="picking.name"/>
                    <t t-set="breadcrumbs_url">/my/productions</t>
                </t>
            </div>
            <div class="o_portal container mt-3 o_subcontracting_portal">
                <div class="o_subcontracting_portal">
                    <t t-set="no_footer" t-value="true"/>
                    <t t-call="mrp_subcontracting.subcontracting"/>
                </div>
            </div>
        </t>
    </template>

    <template id="subcontracting" name="Subcontracting Portal View">
        <iframe width="100%" height="100%" frameborder="0" t-attf-src="/my/productions/{{ picking.id }}/subcontracting_portal" class="border rounded"/>
    </template>

    <template id="subcontracting_portal_embed" name="Subcontracting Portal">
        <t t-call="web.layout">
            <t t-set="head_subcontracting_portal">
                <script type="text/javascript">
                    odoo.__session_info__ = <t t-out="json.dumps(session_info)"/>;
                </script>
                <base target="_parent"/>
                <t t-call-assets="mrp_subcontracting.webclient"/>
            </t>
            <t t-set="head" t-value="head_subcontracting_portal + (head or '')"/>
            <t t-set="body_classname" t-value="'o_web_client o_subcontracting_portal'"/>
        </t>
    </template>

</odoo>

```

## File: views\subcontracting_portal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="subcontracting_portal_production_form_view" model="ir.ui.view">
        <field name="name">subcontracting.portal.production.view.form</field>
        <field name="model">stock.picking</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <form js_class="subcontracting_portal_picking_form_view" string="Manufacturing Orders">
                <sheet>
                    <div class="oe_title">
                        <h1>
                            <field name="name" placeholder="Manufacturing Reference" nolabel="1"/>
                        </h1>
                    </div>
                    <group>
                        <field name="state" invisible='1'/>
                        <field name="scheduled_date" readonly="state in ['cancel', 'done']" required="id" decoration-warning="state not in ('done', 'cancel') and scheduled_date &lt; now" decoration-danger="state not in ('done', 'cancel') and scheduled_date &lt; current_date" decoration-bf="state not in ('done', 'cancel') and (scheduled_date &lt; current_date or scheduled_date &lt; now)"/>
                        <field name="date_deadline" invisible="state in ('done', 'cancel') or not date_deadline" decoration-danger="date_deadline and date_deadline &lt; current_date" decoration-bf="date_deadline and date_deadline &lt; current_date"/>
                        <field name="origin" placeholder="e.g. PO0032" readonly="state in ['cancel', 'done']"/>
                    </group>
                    <notebook>
                        <page string="Operations" name="operations">
                            <field name="move_ids_without_package" mode="list">
                                <list no_open="1">
                                    <field name="id" readonly="1" column_invisible="True"/>
                                    <field name="product_id" readonly="1"/>
                                    <field name="show_details_visible" column_invisible="True"/>
                                    <field name="description_picking" string="Description" optional="hide"/>
                                    <field name="date" optional="hide"/>
                                    <field name="date_deadline" optional="hide"/>
                                    <field name="product_packaging_id" groups="product.group_stock_packaging"/>
                                    <field name="product_uom_qty" string="Demand" readonly="1"/>
                                    <field name="product_qty" readonly="1" column_invisible="True"/>
                                    <field name="quantity" string="Done"/>
                                    <field name="product_uom" groups="uom.group_uom"/>
                                    <button name="action_show_details" type="object" icon="fa-list" title="Details"
                                                invisible="not show_details_visible" context="{'is_subcontracting_portal': 1}"/>
                                    <field name="show_subcontracting_details_visible" column_invisible="True"/>
                                    <button name="action_show_subcontract_details" string="Register components for subcontracted product" type="object" icon="fa-sitemap"
                                        invisible="not show_subcontracting_details_visible"/>
                                </list>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="subcontracting_portal_view_production_action" model="ir.actions.act_window">
        <field name="name">Subcontracting Portal</field>
        <field name="res_model">stock.picking</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="subcontracting_portal_production_form_view"/>
    </record>

</odoo>

```

## File: views\supplier_info_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_supplierinfo_subcontractor_tree_view" model="ir.ui.view">
        <field name="name">product.supplierinfo.subcontractor.list.view</field>
        <field name="model">product.supplierinfo</field>
        <field name="inherit_id" ref="product.product_supplierinfo_tree_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="is_subcontractor" readonly="1" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: wizard\change_production_qty.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ChangeProductionQty(models.TransientModel):
    _inherit = 'change.production.qty'

    @api.model
    def _need_quantity_propagation(self, move, qty):
        res = super()._need_quantity_propagation(move, qty)
        return res and not any(m.is_subcontract for m in move.move_dest_ids)

```

## File: wizard\mrp_consumption_warning.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MrpConsumptionWarning(models.TransientModel):
    _inherit = 'mrp.consumption.warning'

    def action_confirm(self):
        if self.mrp_production_ids._get_subcontract_move():
            return self.mrp_production_ids.with_context(skip_consumption=True).subcontracting_record_component()
        return super().action_confirm()

    def action_cancel(self):
        mo_subcontracted_move = self.mrp_production_ids._get_subcontract_move()
        if mo_subcontracted_move:
            return mo_subcontracted_move.filtered(lambda move: move.state not in ('done', 'cancel'))._action_record_components()
        return super().action_cancel()

```

## File: wizard\stock_picking_return.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class ReturnPicking(models.TransientModel):
    _inherit = 'stock.return.picking'

    def _prepare_picking_default_values(self):
        vals = super()._prepare_picking_default_values()
        if any(return_line.quantity > 0 and return_line.move_id.is_subcontract for return_line in self.product_return_moves):
            vals['location_dest_id'] = self.picking_id.partner_id.with_company(self.picking_id.company_id).property_stock_subcontractor.id
        return vals


class ReturnPickingLine(models.TransientModel):
    _inherit = 'stock.return.picking.line'

    def _prepare_move_default_values(self, new_picking):
        vals = super()._prepare_move_default_values(new_picking)
        vals['is_subcontract'] = False
        return vals

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking_return
from . import mrp_consumption_warning
from . import change_production_qty

```

