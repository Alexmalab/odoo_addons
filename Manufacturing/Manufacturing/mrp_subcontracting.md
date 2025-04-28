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

from odoo import SUPERUSER_ID, api


def uninstall_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
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
    'name': "mrp_subcontracting",
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
        'views/product_views.xml',
        'views/mrp_production_views.xml',
        'views/subcontracting_portal_views.xml',
        'views/subcontracting_portal_templates.xml',
        'views/stock_location_views.xml',
        'wizard/stock_picking_return_views.xml',
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
        ],
        'mrp_subcontracting.webclient': [
            ('include', 'web.assets_backend'),
            ('remove', 'web/static/src/webclient/menus/*.js'),
            'mrp_subcontracting/static/src/subcontracting_portal/*',
            'web/static/src/start.js',
            'web/static/src/legacy/legacy_setup.js',
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
        return request.render("mrp_subcontracting.subcontracting_portal", {'picking_id': picking_id})

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

    @api.depends('name')
    def name_get(self):
        return [
            (record.id, "%s (%s)" % (record.incoming_picking.name, record.name)) if record.bom_id.type == 'subcontract'
            else (record.id, record.name) for record in self
        ]

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
                qty = sum(line.product_uom_id._compute_quantity(line.qty_done, product_id.uom_id) for line in lines)
                move = production._get_move_raw_values(product_id, qty, product_id.uom_id)
                move['additional'] = True
                production.move_raw_ids = [(0, 0, move)]
                production.move_raw_ids.filtered(lambda m: m.product_id == product_id)[:1].move_line_ids = lines

    def write(self, vals):
        if self.env.user.has_group('base.group_portal') and not self.env.su:
            unauthorized_fields = set(vals.keys()) - set(self._get_writeable_fields_portal_user())
            if unauthorized_fields:
                raise AccessError(_("You cannot write on fields %s in mrp.production.", ', '.join(unauthorized_fields)))
        return super().write(vals)

    def action_merge(self):
        if any(production._get_subcontract_move() for production in self):
            raise ValidationError(_("Subcontracted manufacturing orders cannot be merged."))
        return super().action_merge()

    def subcontracting_record_component(self):
        self.ensure_one()
        if not self._get_subcontract_move():
            raise UserError(_("This MO isn't related to a subcontracted move"))
        if float_is_zero(self.qty_producing, precision_rounding=self.product_uom_id.rounding):
            return {'type': 'ir.actions.act_window_close'}
        if self.move_raw_ids and not any(self.move_raw_ids.mapped('quantity_done')):
            raise UserError(_("You must indicate a non-zero amount consumed for at least one of your components"))
        consumption_issues = self._get_consumption_issues()
        if consumption_issues:
            return self._action_generate_consumption_wizard(consumption_issues)

        self._update_finished_move()
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

    def _pre_button_mark_done(self):
        if self._get_subcontract_move():
            return True
        return super()._pre_button_mark_done()

    def _should_postpone_date_finished(self, date_planned_finished):
        return super()._should_postpone_date_finished(date_planned_finished) and not self._get_subcontract_move()

    def _update_finished_move(self):
        """ After producing, set the move line on the subcontract picking. """
        self.ensure_one()
        subcontract_move_id = self._get_subcontract_move().filtered(lambda m: m.state not in ('done', 'cancel'))
        if subcontract_move_id:
            quantity = self.qty_producing
            if self.lot_producing_id:
                move_lines = subcontract_move_id.move_line_ids.filtered(lambda ml: ml.lot_id == self.lot_producing_id or not ml.lot_id)
            else:
                move_lines = subcontract_move_id.move_line_ids.filtered(lambda ml: not ml.lot_id)
            # Update reservation and quantity done
            for ml in move_lines:
                rounding = ml.product_uom_id.rounding
                if float_compare(quantity, 0, precision_rounding=rounding) <= 0:
                    break
                quantity_to_process = min(quantity, ml.reserved_uom_qty - ml.qty_done)
                quantity -= quantity_to_process

                new_quantity_done = (ml.qty_done + quantity_to_process)

                # on which lot of finished product
                if float_compare(new_quantity_done, ml.reserved_uom_qty, precision_rounding=rounding) >= 0:
                    ml.write({
                        'qty_done': new_quantity_done,
                        'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                    })
                else:
                    new_qty_reserved = ml.reserved_uom_qty - new_quantity_done
                    default = {
                        'reserved_uom_qty': new_quantity_done,
                        'qty_done': new_quantity_done,
                        'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                    }
                    ml.copy(default=default)
                    ml.with_context(bypass_reservation_update=True).write({
                        'reserved_uom_qty': new_qty_reserved,
                        'qty_done': 0
                    })

            if float_compare(quantity, 0, precision_rounding=self.product_uom_id.rounding) > 0:
                self.env['stock.move.line'].create({
                    'move_id': subcontract_move_id.id,
                    'picking_id': subcontract_move_id.picking_id.id,
                    'product_id': self.product_id.id,
                    'location_id': subcontract_move_id.location_id.id,
                    'location_dest_id': subcontract_move_id.location_dest_id.id,
                    'reserved_uom_qty': 0,
                    'product_uom_id': self.product_uom_id.id,
                    'qty_done': quantity,
                    'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                })
            if not self._get_quantity_to_backorder():
                ml_reserved = subcontract_move_id.move_line_ids.filtered(lambda ml:
                    float_is_zero(ml.qty_done, precision_rounding=ml.product_uom_id.rounding) and
                    not float_is_zero(ml.reserved_uom_qty, precision_rounding=ml.product_uom_id.rounding))
                ml_reserved.unlink()
                for ml in subcontract_move_id.move_line_ids:
                    ml.reserved_uom_qty = ml.qty_done
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
                raise UserError(_('You must enter a serial number for %s') % production.product_id.name)
            for sml in production.move_raw_ids.move_line_ids:
                if sml.tracking != 'none' and not sml.lot_id:
                    raise UserError(_('You must enter a serial number for each line of %s') % sml.product_id.display_name)
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
            self.env['ir.property']._set_default(
                "property_stock_subcontractor",
                "res.partner",
                subcontracting_location,
                company,
            )
            company.subcontracting_location_id = subcontracting_location

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


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
        results = self.env['mrp.bom'].read_group([('subcontractor_ids.commercial_partner_id', 'in', self.commercial_partner_id.ids)], ['ids:array_agg(id)', 'subcontractor_ids'], ['subcontractor_ids'])
        for partner in self:
            bom_ids = []
            for res in results:
                if partner.id == res['subcontractor_ids'][0] or res['subcontractor_ids'][0] in partner.child_ids.ids:
                    bom_ids += res['ids']
            partner.bom_ids = bom_ids

    def _compute_production_ids(self):
        results = self.env['mrp.production'].read_group([('subcontractor_id.commercial_partner_id', 'in', self.commercial_partner_id.ids)], ['ids:array_agg(id)'], ['subcontractor_id'])
        for partner in self:
            production_ids = []
            for res in results:
                if partner.id == res['subcontractor_id'][0] or res['subcontractor_id'][0] in partner.child_ids.ids:
                    production_ids += res['ids']
            partner.production_ids = production_ids

    def _compute_picking_ids(self):
        results = self.env['stock.picking'].read_group([('partner_id.commercial_partner_id', 'in', self.commercial_partner_id.ids)], ['ids:array_agg(id)'], ['partner_id'])
        for partner in self:
            picking_ids = []
            for res in results:
                if partner.id == res['partner_id'][0] or res['partner_id'][0] in partner.child_ids.ids:
                    picking_ids += res['ids']
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

    @api.depends_context('uid')
    def _compute_is_subcontractor(self):
        """ Check if the user is a subcontractor before giving sudo access
        """
        for partner in self:
            partner.is_subcontractor = (partner.user_has_groups('base.group_portal') and partner.env['mrp.bom'].search_count([
                ('type', '=', 'subcontract'),
                ('subcontractor_ids', 'in', (partner.env.user.partner_id | partner.env.user.partner_id.commercial_partner_id).ids),
            ]))

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
from odoo.exceptions import UserError, AccessError
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
            if not productions or move.has_tracking != 'serial':
                continue
            if productions._has_tracked_component() or productions[:1].consumption != 'strict':
                move.display_assign_serial = False

    def _compute_show_subcontracting_details_visible(self):
        """ Compute if the action button in order to see moves raw is visible """
        self.show_subcontracting_details_visible = False
        for move in self:
            if not move.is_subcontract:
                continue
            if float_is_zero(move.quantity_done, precision_rounding=move.product_uom.rounding):
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
            if self.env.user.has_group('base.group_portal'):
                move.show_details_visible = any(not p._has_been_recorded() for p in move._get_subcontract_production())
                continue
            productions = move._get_subcontract_production()
            if not productions._has_tracked_component() and productions[:1].consumption == 'strict':
                continue
            move.show_details_visible = True
        return res

    def _set_quantity_done(self, qty):
        to_set_moves = self
        for move in self:
            if move.is_subcontract and move._subcontracting_possible_record():
                # If 'done' quantity is changed through the move, record components as if done through the wizard.
                move._auto_record_components(qty)
                to_set_moves -= move
        if to_set_moves:
            super(StockMove, to_set_moves)._set_quantity_done(qty)

    def _quantity_done_set(self):
        to_set_moves = self
        for move in self:
            if move.is_subcontract and move._subcontracting_possible_record():
                delta_qty = move.quantity_done - move._quantity_done_sml()
                if float_compare(delta_qty, 0, precision_rounding=move.product_uom.rounding) > 0:
                    move._auto_record_components(delta_qty)
                    to_set_moves -= move
                elif float_compare(delta_qty, 0, precision_rounding=move.product_uom.rounding) < 0 and not move.picking_id.immediate_transfer:
                    move.with_context(transfer_qty=True)._reduce_subcontract_order_qty(abs(delta_qty))
        if to_set_moves:
            super(StockMove, to_set_moves)._quantity_done_set()

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

    def copy(self, default=None):
        self.ensure_one()
        if not self.is_subcontract or 'location_id' in default:
            return super(StockMove, self).copy(default=default)
        if not default:
            default = {}
        default['location_id'] = self.picking_id.location_id.id
        return super(StockMove, self).copy(default=default)

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
                move.move_orig_ids.production_id.filtered(lambda p: p.state not in ('done', 'cancel')).write({
                    'date_planned_finished': move.date,
                    'date_planned_start': move.date,
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
        elif self.env.user.has_group('base.group_portal'):
            if self.picking_type_id.show_reserved:
                action['views'] = [(self.env.ref('mrp_subcontracting.mrp_subcontracting_view_stock_move_operations').id, 'form')]
            else:
                action['views'] = [(self.env.ref('mrp_subcontracting.mrp_subcontracting_view_stock_move_nosuggest_operations').id, 'form')]
        return action

    def action_show_subcontract_details(self):
        """ Display moves raw for subcontracted product self. """
        moves = self._get_subcontract_production().move_raw_ids.filtered(lambda m: m.state != 'cancel')
        tree_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_move_tree_view')
        form_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_move_form_view')
        ctx = dict(self._context, search_default_by_product=True)
        if self.env.user.has_group('base.group_portal'):
            form_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_portal_move_form_view')
            ctx.update(no_breadcrumbs=False)
        return {
            'name': _('Raw Materials for %s') % (self.product_id.display_name),
            'type': 'ir.actions.act_window',
            'res_model': 'stock.move',
            'views': [(tree_view.id, 'list'), (form_view.id, 'form')],
            'target': 'current',
            'domain': [('id', 'in', moves.ids)],
            'context': ctx
        }

    def _set_quantities_to_reservation(self):
        move_untouchable = self.filtered(lambda m: m.is_subcontract and m._get_subcontract_production()._has_tracked_component())
        return super(StockMove, self - move_untouchable)._set_quantities_to_reservation()

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
            if float_is_zero(move.product_qty, precision_rounding=move.product_uom.rounding) and\
                    move.picking_id.immediate_transfer is True:
                raise UserError(_("To subcontract, use a planned transfer."))
            move.write({
                'is_subcontract': True,
                'location_id': move.picking_id.partner_id.with_company(move.company_id).property_stock_subcontractor.id
            })
            if float_compare(move.product_qty, 0, precision_rounding=move.product_uom.rounding) <= 0:
                # If a subcontracted amount is decreased, don't create a MO that would be for a negative value.
                # We don't care if the MO decreases even when done since everything is handled through picking
                continue
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
        if self.env.user.has_group('base.group_portal'):
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

    def _prepare_extra_move_vals(self, qty):
        vals = super(StockMove, self)._prepare_extra_move_vals(qty)
        vals['location_id'] = self.location_id.id
        return vals

    def _prepare_move_split_vals(self, qty):
        vals = super(StockMove, self)._prepare_move_split_vals(qty)
        vals['location_id'] = self.location_id.id
        return vals

    def _should_bypass_set_qty_producing(self):
        if (self.production_id | self.raw_material_production_id)._get_subcontract_move():
            return False
        return super()._should_bypass_set_qty_producing()

    def _should_bypass_reservation(self, forced_location=False):
        """ If the move is subcontracted then ignore the reservation. """
        should_bypass_reservation = super()._should_bypass_reservation(forced_location=forced_location)
        if not should_bypass_reservation and self.is_subcontract:
            return True
        return should_bypass_reservation

    def _update_subcontract_order_qty(self, new_quantity):
        for move in self:
            quantity_to_remove = move.product_uom_qty - new_quantity
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
        if self.env.user.has_group('base.group_portal') and not self.env.su:
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

    # override existing field domains to prevent suboncontracting production lines from showing in Detailed Operations tab
    move_line_nosuggest_ids = fields.One2many(
        domain=['&', '|', ('location_dest_id.usage', '!=', 'production'), ('move_id.picking_code', '!=', 'outgoing'),
                     '|', ('reserved_qty', '=', 0.0), '&', ('reserved_qty', '!=', 0.0), ('qty_done', '!=', 0.0)])
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
            sm_done_qty = sum(productions._get_subcontract_move().mapped('quantity_done'))
            rounding = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            if float_compare(recorded_qty, sm_done_qty, precision_digits=rounding) >= 0:
                continue
            production = productions - recorded_productions
            if not production:
                continue
            if len(production) > 1:
                raise UserError(_("There shouldn't be multiple productions to record for the same subcontracted move."))
            # Manage additional quantities
            quantity_done_move = move.product_uom._compute_quantity(move.quantity_done, production.product_uom_id)
            if float_compare(production.product_qty, quantity_done_move, precision_rounding=production.product_uom_id.rounding) == -1:
                change_qty = self.env['change.production.qty'].create({
                    'mo_id': production.id,
                    'product_qty': quantity_done_move
                })
                change_qty.with_context(skip_activity=True).change_prod_qty()
            # Create backorder MO for each move lines
            amounts = [move_line.qty_done for move_line in move.move_line_ids]
            len_amounts = len(amounts)
            # _split_production can set the qty_done, but not split it.
            # Remove the qty_done potentially set by a previous split to prevent any issue.
            production.move_line_raw_ids.filtered(lambda sml: sml.state not in ['done', 'cancel']).write({'qty_done': 0})
            productions = production._split_productions({production: amounts}, set_consumed_qty=True)
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
        vals = {
            'company_id': subcontract_move.company_id.id,
            'procurement_group_id': group.id,
            'subcontractor_id': subcontract_move.picking_id.partner_id.commercial_partner_id.id,
            'picking_ids': [subcontract_move.picking_id.id],
            'product_id': product.id,
            'product_uom_id': subcontract_move.product_uom.id,
            'bom_id': bom.id,
            'location_src_id': subcontract_move.picking_id.partner_id.with_company(subcontract_move.company_id).property_stock_subcontractor.id,
            'location_dest_id': subcontract_move.picking_id.partner_id.with_company(subcontract_move.company_id).property_stock_subcontractor.id,
            'product_qty': subcontract_move.product_uom_qty,
            'picking_type_id': warehouse.subcontracting_type_id.id,
            'date_planned_start': subcontract_move.date - relativedelta(days=product.produce_delay)
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
            if float_compare(move.product_qty, 0, precision_rounding=move.product_uom.rounding) <= 0:
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
            finished_move = mo.move_finished_ids.filtered(lambda m: m.product_id == move.product_id)
            finished_move.write({'move_dest_ids': [(4, move.id, False)]})

        all_mo.action_assign()

    @api.onchange('location_id', 'location_dest_id')
    def _onchange_locations(self):
        moves = self.move_ids | self.move_ids_without_package
        moves.filtered(lambda m: m.is_subcontract).update({
            "location_dest_id": self.location_dest_id,
        })
        moves.filtered(lambda m: not m.is_subcontract).update({
            "location_id": self.location_id,
            "location_dest_id": self.location_dest_id,
        })
        if any(line.reserved_qty or line.qty_done for line in self.move_ids.move_line_ids):
            return {'warning': {
                    'title': _("Locations to update"),
                    'message': _("You might want to update the locations of this transfer's operations"),
                }
            }

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
        'stock.rule', 'Subcontracting MTO Rule')
    subcontracting_pull_id = fields.Many2one(
        'stock.rule', 'Subcontracting MTS Rule'
    )

    subcontracting_route_id = fields.Many2one('stock.route', 'Resupply Subcontractor', ondelete='restrict')

    subcontracting_type_id = fields.Many2one(
        'stock.picking.type', 'Subcontracting Operation Type',
        domain=[('code', '=', 'mrp_operation')])
    subcontracting_resupply_type_id = fields.Many2one(
        'stock.picking.type', 'Subcontracting Resupply Operation Type',
        domain=[('code', '=', 'outgoing')])

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

    def _get_global_route_rules_values(self):
        rules = super(StockWarehouse, self)._get_global_route_rules_values()
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
                    'route_id': self._find_or_create_global_route('stock.route_warehouse0_mto', _('Make To Order')).id,
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
                    'route_id': self._find_or_create_global_route('mrp_subcontracting.route_resupply_subcontractor_mto',
                                                        _('Resupply Subcontractor on Order')).id,
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
                'code': 'outgoing',
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
        count = self.env['ir.sequence'].search_count([('prefix', 'like', self.code + '/SBC%/%')])
        values.update({
            'subcontracting_type_id': {
                'name': self.name + ' ' + _('Sequence subcontracting'),
                'prefix': self.code + '/' + (self.subcontracting_type_id.sequence_code or (('SBC' + str(count)) if count else 'SBC')) + '/',
                'padding': 5,
                'company_id': self.company_id.id
            },
            'subcontracting_resupply_type_id': {
                'name': self.name + ' ' + _('Sequence Resupply Subcontractor'),
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
                'barcode': self.code.replace(" ", "").upper() + "-RESUPPLY",
                'active': self.subcontracting_to_resupply and self.active
            },
        })
        return data

    def _get_subcontracting_location(self):
        return self.company_id.subcontracting_location_id

    def _get_subcontracting_locations(self):
        return self.env['stock.location'].search([
            ('company_id', 'in', self.company_id.ids),
            ('is_subcontracting_location', '=', 'True'),
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

    def _get_bom_data(self, bom, warehouse, product=False, line_qty=False, bom_line=False, level=0, parent_bom=False, index=0, product_info=False, ignore_stock=False):
        res = super()._get_bom_data(bom, warehouse, product, line_qty, bom_line, level, parent_bom, index, product_info, ignore_stock)
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
    def _need_special_rules(self, product_info, parent_bom=False, parent_product_id=False):
        if parent_bom and parent_product_id:
            parent_info = product_info.get(parent_product_id, {}).get(parent_bom.id, {})
            return parent_info and parent_info.get('route_type') == 'subcontract'
        return super()._need_special_rules(product_info, parent_bom, parent_product_id)

    @api.model
    def _find_special_rules(self, product, product_info, parent_bom=False, parent_product_id=False):
        res = super()._find_special_rules(product, product_info, parent_bom, parent_product_id)
        # If no rules could be found within the warehouse, check if the product is a component from a subcontracted product.
        parent_info = product_info.get(parent_product_id, {}).get(parent_bom.id, {})
        if parent_info and parent_info.get('route_type') == 'subcontract':
            # Since the product is subcontracted, check the subcontracted location for rules instead of the warehouse.
            subcontracting_loc = parent_info['supplier'].partner_id.property_stock_subcontractor
            return product._get_rules_from_location(subcontracting_loc)
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
            if supplier:
                qty_supplier_uom = product.uom_id._compute_quantity(quantity, supplier.product_uom)
                return {
                    'route_type': 'subcontract',
                    'route_name': subcontract_rules[0].route_id.display_name,
                    'route_detail': supplier.display_name,
                    'lead_time': supplier.delay + rules_delay,
                    'supplier_delay': supplier.delay + rules_delay,
                    'manufacture_delay': product.produce_delay,
                    'supplier': supplier,
                    'route_alert': float_compare(qty_supplier_uom, supplier.min_qty, precision_rounding=product.uom_id.rounding) < 0,
                    'qty_checked': quantity,
                }

        return res

    @api.model
    def _get_quantities_info(self, product, bom_uom, parent_bom, product_info):
        if parent_bom and parent_bom.type == 'subcontract' and product.detailed_type == 'product':
            parent_product_id = self.env.context.get('parent_product_id', False)
            route_info = product_info.get(parent_product_id, {}).get(parent_bom.id, {})
            if route_info and route_info['route_type'] == 'subcontract':
                subcontracting_loc = route_info['supplier'].partner_id.property_stock_subcontractor
                subloc_product = product.with_context(location=subcontracting_loc.id, warehouse=False).read(['free_qty', 'qty_available'])[0]
                stock_loc = f"subcontract_{subcontracting_loc.id}"
                if not product_info[product.id]['consumptions'].get(stock_loc, False):
                    product_info[product.id]['consumptions'][stock_loc] = 0
                return {
                    'free_qty': product.uom_id._compute_quantity(subloc_product['free_qty'], bom_uom),
                    'on_hand_qty': product.uom_id._compute_quantity(subloc_product['qty_available'], bom_uom),
                    'stock_loc': stock_loc,
                }

        return super()._get_quantities_info(product, bom_uom, parent_bom, product_info)

    @api.model
    def _get_resupply_availability(self, route_info, components):
        if route_info.get('route_type') == 'subcontract':
            max_component_delay = self._get_max_component_delay(components)
            if max_component_delay is False:
                return ('unavailable', False)
            produce_delay = route_info.get('manufacture_delay', 0) + max_component_delay
            supplier_delay = route_info.get('supplier_delay', 0)
            return ('estimated', max(produce_delay, supplier_delay))
        return super()._get_resupply_availability(route_info, components)

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

patch(BomOverviewComponentsBlock, "mrp_subcontracting", {
    components: { ...BomOverviewComponentsBlock.components, BomOverviewSpecialLine },
});

```

## File: static\src\components\bom_overview_components_block\mrp_bom_overview_components_block.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="mrp_subcontracting.BomOverviewComponentsBlock" t-inherit="mrp.BomOverviewComponentsBlock" t-inherit-mode="extension" owl="1">
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

patch(BomOverviewLine.prototype, "mrp_subcontracting", {
    /**
     * @override
     */
    async goToRoute(routeType) {
        if (routeType == "subcontract") {
            return this.goToAction(this.data.link_id, this.data.link_model);
        }
        return this._super(...arguments);
    }
});

```

## File: static\src\components\bom_overview_special_line\mrp_bom_overview_special_line.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";
import { BomOverviewSpecialLine } from "@mrp/components/bom_overview_special_line/mrp_bom_overview_special_line";

patch(BomOverviewSpecialLine.prototype, "mrp_subcontracting", {
    setup() {
        this._super.apply();
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

    <t t-name="mrp_subcontracting.BomOverviewSpecialLine" t-inherit="mrp.BomOverviewSpecialLine" t-inherit-mode="extension" owl="1">
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

## File: static\src\subcontracting_portal\main.js

```javascript
/** @odoo-module **/
import { startWebClient } from '@web/start';
import { SubcontractingPortalWebClient } from './subcontracting_portal';

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


class PickingFormController extends FormController {}
PickingFormController.template = "mrp_subcontracting.PickingFormController";

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

    <t t-name="mrp_subcontracting.PickingFormController" t-inherit="web.FormView" t-inherit-mode="primary" owl="1">
        <xpath expr="//ActionMenus" position="replace"/>
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
import { ErrorHandler } from "@web/core/utils/components";
import { session } from '@web/session';
import { LegacyComponent } from "@web/legacy/legacy_component";

const { useEffect} = owl;

export class SubcontractingPortalWebClient extends LegacyComponent {
    setup() {
        window.parent.document.body.style.margin = "0"; // remove the margin in the parent body
        this.actionService = useService('action');
        this.user = useService("user");
        useService("legacy_service_provider");
        useOwnDebugContext({ categories: ["default"] });
        useEffect(
            () => {
                this._showView();
            },
            () => []
        );
    }

    handleComponentError(error, C) {
        // remove the faulty component
        this.Components.splice(this.Components.indexOf(C), 1);
        /**
         * we rethrow the error to notify the user something bad happened.
         * We do it after a tick to make sure owl can properly finish its
         * rendering
         */
        Promise.resolve().then(() => {
            throw error;
        });
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
}

SubcontractingPortalWebClient.components = { ActionContainer, ErrorHandler, MainComponentsContainer };
SubcontractingPortalWebClient.template = 'mrp_subcontracting.SubcontractingPortalWebClient';

```

## File: static\src\subcontracting_portal\subcontracting_portal.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-name="mrp_subcontracting.SubcontractingPortalWebClient" owl="1">
        <ActionContainer/>
        <MainComponentsContainer/>
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
                <field name="subcontractor_ids" widget="many2many_tags" attrs="{'invisible': [('type', '!=', 'subcontract')], 'required': [('type', '=', 'subcontract')]}"/>
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
            <xpath expr="//group[@name='group_extra_info']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//page[@name='operations']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='lot_producing_id']" position="attributes">
                <attribute name="attrs">{'invisible': [('product_tracking', 'in', ('none', False))], 'required': [('product_tracking', 'not in', ('none', False))]}</attribute>
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
                    context="{'tree_view_ref': 'mrp_subcontracting.mrp_subcontracting_stock_move_line_tree_view', 'default_company_id': company_id, 'default_location_id': location_src_id, 'default_location_dest_id': production_location_id, 'bom_product_ids': bom_product_ids}"
                    />
            </xpath>
            <xpath expr="//sheet" position="inside">
                <footer>
                    <button name="subcontracting_record_component" attrs="{'invisible': ['|', ('state', '=', 'to_close'), ('qty_producing', '&lt;', 0.001)]}" string="Continue" type="object" class="oe_highlight" data-hotkey="q"/>
                    <button name="subcontracting_record_component" attrs="{'invisible': [('state', '!=', 'to_close')]}" string="Record Production" type="object" class="oe_highlight" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="z" />
                </footer>
            </xpath>
            <xpath expr="//div[hasclass('oe_chatter')]" position="replace"/>
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
                <attribute name="attrs">{'readonly': True}</attribute>
            </xpath>
            <xpath expr="//field[@name='lot_producing_id']" position="attributes">
                <attribute name="domain">[('id', '=', False)]</attribute>
                <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
            </xpath>
            <xpath expr="//field[@name='move_line_raw_ids']" position="attributes">
                <attribute name="context">{'tree_view_ref': 'mrp_subcontracting.mrp_subcontracting_portal_stock_move_line_tree_view', 'default_company_id': company_id, 'default_location_id': location_src_id, 'default_location_dest_id': production_location_id, 'bom_product_ids': bom_product_ids}</attribute>
            </xpath>
            <xpath expr="//field[@name='forecasted_issue']" position="replace"/>
            <xpath expr="//button[@name='%(mrp.action_change_production_qty)d']" position="replace"/>
            <xpath expr="//button[@name='action_product_forecast_report']" position="replace"/>
            <xpath expr="//button[@name='action_product_forecast_report']" position="replace"/>
        </field>
    </record>

    <record id="mrp_production_subcontracting_tree_view" model="ir.ui.view">
        <field name="name">mrp.production.subcontracting.tree</field>
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

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mrp_subcontracting_product_template_search_view" model="ir.ui.view">
            <field name="name">mrp.subcontracting.product.template.search</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_search_view" />
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='filter_to_purchase']" position="after">
                    <filter string="Can be Subcontracted" name="filter_can_be_subcontracted" domain="[('bom_ids.type', '=', 'subcontract')]" groups="mrp.group_mrp_user"/>
                </xpath>
            </field>
        </record>
    </data>
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
            <xpath expr="//field[@name='return_location']" position='after'>
                <field name="is_subcontracting_location" groups="base.group_no_one" attrs="{'invisible': [('usage', '!=', 'internal')]}"/>
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
        <field name="name">mrp.subcontracting.stock.move.line.tree.view</field>
        <field name="model">stock.move.line</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="company_id" invisible="1"/>
                <field name="product_uom_category_id" invisible="1"/>
                <field name="owner_id" invisible="1"/>
                <field name="tracking" invisible="1"/>
                <field name="package_id" invisible="1"/>
                <field name="result_package_id" invisible="1"/>
                <field name="location_id" invisible="1"/>
                <field name="location_dest_id" invisible="1"/>
                <field name="state" invisible="1"/>
                <!-- Don't put move_id here to avoid that the framework send falsy move_id -->
                <field name="id" invisible="1"/>
                <field name="product_id" required="1" domain="[('id', 'in', context.get('bom_product_ids'))] if context.get('is_subcontracting_portal') else []"/>
                <field name="lot_id" groups="stock.group_production_lot"
                    attrs="{'invisible': [('tracking', 'not in', ('serial', 'lot'))], 'required': [('tracking', 'in', ('serial', 'lot'))]}"
                    context="{'default_product_id': product_id, 'default_company_id': company_id}"/>
                <field name="reserved_uom_qty" readonly="1" force_save="1"/>
                <field name="qty_done"
                    decoration-warning="reserved_uom_qty &lt; qty_done"
                    decoration-success="reserved_uom_qty == qty_done"/>
                <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" string="Unit of Measure" groups="uom.group_uom"
			        attrs="{'readonly': ['|', ('reserved_uom_qty', '!=', 0.0), '&amp;', ('state', '=', 'done'), ('id', '!=', False)]}"/>
            </tree>
        </field>
    </record>
    <record id="mrp_subcontracting_portal_stock_move_line_tree_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.portal.stock.move.line.tree.view</field>
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
        <field name="name">mrp.subcontracting.stock.move.line.operations.tree</field>
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
                <attribute name="context">{'tree_view_ref': 'mrp_subcontracting.mrp_subcontracting_view_stock_move_line_operation_tree', 'default_product_uom_id': product_uom, 'default_picking_id': picking_id, 'default_move_id': id, 'default_product_id': product_id, 'default_location_id': location_id, 'default_location_dest_id': location_dest_id, 'default_company_id': company_id}</attribute>
            </xpath>
        </field>
    </record>
    <record id="mrp_subcontracting_view_stock_move_nosuggest_operations" model="ir.ui.view">
        <field name="name">mrp.subcontracting.stock.move.operations.nosuggest.form</field>
        <field name="model">stock.move</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="stock.view_stock_move_nosuggest_operations" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='move_line_nosuggest_ids']" position="attributes">
                <attribute name="context">{'tree_view_ref': 'mrp_subcontracting.mrp_subcontracting_view_stock_move_line_operation_tree','default_picking_id': picking_id, 'default_move_id': id, 'default_product_id': product_id, 'default_location_id': location_id, 'default_location_dest_id': location_dest_id, 'default_company_id': company_id}</attribute>
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
                    <field name="product_id" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                    <field name="location_id" invisible="1"/>
                    <field name="picking_id" invisible="1"/>
                    <field name="location_dest_id" invisible="1"/>
                    <field name="has_tracking" invisible="1"/>
                    <field name="product_uom" invisible="1"/>
                    <field name="product_uom_qty" invisible="1"/>
                    <group>
                        <field name="order_finished_lot_id"/>
                        <field name="product_uom" groups="uom.group_uom"/>
                        <field name="quantity_done" string="Total Consumed" readonly="1"/>
                    </group>
                    <field name="move_line_ids"
                        attrs="{'readonly': [('state', 'in', ['done', 'cancel'])]}"
                        context="{'default_product_uom_id': product_uom, 'default_picking_id': picking_id, 'default_move_id': id, 'default_product_id': product_id, 'default_location_id': location_id, 'default_location_dest_id': location_dest_id, 'default_company_id': company_id}">
                        <tree editable="bottom" decoration-muted="state in ('done', 'cancel')">
                            <field name="company_id" invisible="1"/>
                            <field name="state" invisible="1"/>
                            <field name="tracking" invisible="1"/>
                            <field name="product_uom_id" invisible="1"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <field name="picking_id" invisible="1"/>
                            <field name="move_id" invisible="1"/>
                            <field name="location_id" invisible="1"/>
                            <field name="location_dest_id" invisible="1"/>
                            <field name="product_id" readonly="1" force_save="1"/>
                            <field name="qty_done"/>
                            <field name="lot_id" attrs="{'column_invisible':[('parent.has_tracking', 'not in', ('serial', 'lot'))], 'required': [('tracking', 'in', ('serial', 'lot'))]}" context="{'default_product_id': product_id, 'default_company_id': company_id}" groups="stock.group_production_lot"/>
                        </tree>
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
            <xpath expr="//tree" position="attributes">
                <attribute name="no_open">1</attribute>
            </xpath>
            <xpath expr="//tree/field[@name='lot_id']" position="attributes">
                <attribute name="options">{'no_create_edit': True, 'no_open': True}</attribute>
            </xpath>
        </field>
    </record>
    <record id="mrp_subcontracting_move_tree_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.move.tree.view</field>
        <field name="model">stock.move</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <tree delete="0" create="0" decoration-muted="is_done" decoration-warning="quantity_done - product_uom_qty &gt; 0.0001" decoration-success="not is_done and quantity_done - product_uom_qty &lt; 0.0001" js_class="subcontracting_portal_move_list_view">
                <field name="company_id" invisible="1"/>
                <field name="sequence" invisible="1"/>
                <field name="product_uom_category_id" invisible="1"/>
                <field name="name" invisible="1"/>
                <field name="unit_factor" invisible="1"/>
                <field name="date" invisible="1"/>
                <field name="picking_type_id" invisible="1"/>
                <field name="has_tracking" invisible="1"/>
                <field name="operation_id" invisible="1"/>
                <field name="is_done" invisible="1"/>
                <field name="bom_line_id" invisible="1"/>
                <field name="location_id" invisible="1"/>
                <field name="warehouse_id" invisible="1"/>
                <field name="product_uom_qty" invisible="1"/>
                <field name="location_dest_id" invisible="1"/>
                <field name="state" invisible="1" force_save="1"/>
                <field name="raw_material_production_id" invisible="1"/>
                <field name="product_id" required="1"/>
                <field name="order_finished_lot_id"/>
                <field name="reserved_availability" attrs="{'invisible': [('is_done', '=', True)]}" string="Reserved"/>
                <field name="quantity_done" string="Consumed" readonly="1"/>
                <field name="product_uom" groups="uom.group_uom"/>
            </tree>
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
                <button name="action_record_components" class="oe_highlight" attrs="{'invisible': [('display_action_record_components', '!=', 'mandatory')]}" string="Record components" type="object" data-hotkey="shift+x"/>
                <button name="action_record_components" attrs="{'invisible': [('display_action_record_components', '!=', 'facultative')]}" string="Record components" type="object" data-hotkey="shift+x"/>
            </xpath>
            <xpath expr="//field[@name='move_ids_without_package']//tree//button[@name='action_show_details']" position="after">
                <field name="show_subcontracting_details_visible" invisible="1"/>
                <button name="action_show_subcontract_details" string="Register components for subcontracted product" type="object" icon="fa-sitemap"
                    width="0.1" attrs="{'invisible': [('show_subcontracting_details_visible', '=', False)]}"/>
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
        <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Productions</t>
                <t t-set="url" t-value="'/my/productions'"/>
                <t t-set="placeholder_count" t-value="'production_count'"/>
            </t>
        </xpath>
    </template>

    <template id="portal_my_home_menu_production" name="Portal layout : production menu entries" inherit_id="portal.portal_breadcrumbs" priority="25">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <li t-if="page_name == 'production' or production" t-attf-class="breadcrumb-item #{'active' if not pickings else ''}">
                <t>Productions</t>
            </li>
        </xpath>
    </template>

    <template id="portal_my_productions" name="My Productions">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Productions</t>
            </t>

            <t t-if="not pickings">
                <p>There are currently no productions for your account.</p>
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
            <t t-set="no_footer" t-value="true"/>
            <t t-call="mrp_subcontracting.subcontracting"/>
        </t>
    </template>

    <template id="subcontracting" name="Subcontracting Portal View">
        <iframe width="100%" height="100%" frameborder="0" t-attf-src="/my/productions/{{ picking_id }}/subcontracting_portal"/>
    </template>

    <template id="subcontracting_portal_embed" name="Subcontracting Portal">
        <t t-call="web.layout">
            <t t-set="head_subcontracting_portal">
                <script type="text/javascript">
                    odoo.__session_info__ = <t t-out="json.dumps(session_info)"/>;
                </script>
                <base target="_parent"/>
                <t t-call-assets="web.assets_common" t-js="false"/>
                <t t-call-assets="web.assets_backend" t-js="false"/>
                <t t-call-assets="web.assets_common" t-css="false"/>
                <t t-call-assets="mrp_subcontracting.webclient" t-css="false"/>
                <t t-call="web.conditional_assets_tests"/>
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
                        <field name="scheduled_date" attrs="{'required': [('id', '!=', False)]}" decoration-warning="state not in ('done', 'cancel') and scheduled_date &lt; now" decoration-danger="state not in ('done', 'cancel') and scheduled_date &lt; current_date" decoration-bf="state not in ('done', 'cancel') and (scheduled_date &lt; current_date or scheduled_date &lt; now)"/>
                        <field name="date_deadline" attrs="{'invisible': ['|', ('state', 'in', ('done', 'cancel')), ('date_deadline', '=', False)]}" decoration-danger="date_deadline and date_deadline &lt; current_date" decoration-bf="date_deadline and date_deadline &lt; current_date"/>
                        <field name="origin" placeholder="e.g. PO0032"/>
                    </group>
                    <notebook>
                        <page string="Operations" name="operations">
                            <field name="move_ids_without_package" mode="tree">
                                <tree no_open="1">
                                    <field name="id" readonly="1" invisible="1"/>
                                    <field name="product_id" readonly="1"/>
                                    <field name="show_details_visible" invisible="1"/>
                                    <field name="description_picking" string="Description" optional="hide"/>
                                    <field name="date" optional="hide"/>
                                    <field name="date_deadline" optional="hide"/>
                                    <field name="product_packaging_id" groups="product.group_stock_packaging"/>
                                    <field name="product_uom_qty" string="Demand" readonly="1"/>
                                    <field name="product_qty" invisible="1" readonly="1"/>
                                    <field name="quantity_done" string="Done"/>
                                    <field name="product_uom" groups="uom.group_uom"/>
                                    <button name="action_show_details" type="object" icon="fa-list" width="0.1" title="Details"
                                                attrs="{'invisible': [('show_details_visible', '=', False)]}" context="{'is_subcontracting_portal': 1}"/>
                                    <field name="show_subcontracting_details_visible" invisible="1"/>
                                    <button name="action_show_subcontract_details" string="Register components for subcontracted product" type="object" icon="fa-sitemap"
                                        width="0.1" attrs="{'invisible': [('show_subcontracting_details_visible', '=', False)]}"/>
                                </tree>
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
        <field name="name">product.supplierinfo.subcontractor.tree.view</field>
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

    @api.model
    def _update_product_qty(self, move, qty):
        res = super()._update_product_qty(move, qty)
        subcontract_moves = move.move_dest_ids.filtered(lambda m: m.is_subcontract and m.from_immediate_transfer)
        if subcontract_moves:
            subcontract_moves[0].with_context(cancel_backorder=False).write({'product_uom_qty': subcontract_moves[0].product_uom_qty + qty})

        return res

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

    subcontract_location_id = fields.Many2one('stock.location', compute='_compute_subcontract_location_id')

    @api.depends('picking_id')
    def _compute_subcontract_location_id(self):
        for record in self:
            record.subcontract_location_id = record.picking_id.partner_id.with_company(
                record.picking_id.company_id
            ).property_stock_subcontractor

    @api.onchange('picking_id')
    def _onchange_picking_id(self):
        res = super(ReturnPicking, self)._onchange_picking_id()
        if any(return_line.quantity > 0 and return_line.move_id.is_subcontract for return_line in self.product_return_moves):
            self.location_id = self.picking_id.partner_id.with_company(self.picking_id.company_id).property_stock_subcontractor
        return res

    def _prepare_move_default_values(self, return_line, new_picking):
        vals = super(ReturnPicking, self)._prepare_move_default_values(return_line, new_picking)
        vals['is_subcontract'] = False
        return vals

```

## File: wizard\stock_picking_return_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_stock_return_picking_form_subcontracting" model="ir.ui.view">
        <field name="name">Return lines</field>
        <field name="model">stock.return.picking</field>
        <field name="inherit_id" ref="stock.view_stock_return_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='location_id']" position="before">
                <field name="subcontract_location_id" invisible="1"/>
                <field name="original_location_id" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='location_id']" position="attributes">
                <attribute name="domain">
                    ['|', '|', ('id', '=', original_location_id), ('return_location', '=', True), ('id', '=', subcontract_location_id)]
                </attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking_return
from . import mrp_consumption_warning
from . import change_production_qty

```

