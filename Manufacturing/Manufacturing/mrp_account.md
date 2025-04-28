# Odoo Module: mrp_account

Category: Manufacturing/Manufacturing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import report


def _configure_journals(env):
    # if we already have a coa installed, create journal and set property field
    for company in env['res.company'].search([('chart_template', '!=', False)], order="parent_path"):
        ChartTemplate = env['account.chart.template'].with_company(company)
        template_code = company.chart_template
        template_data = ChartTemplate._get_chart_template_data(template_code)['template_data']
        if 'property_stock_account_production_cost_id' in template_data:
            data = {'property_stock_account_production_cost_id': template_data['property_stock_account_production_cost_id']}
            ChartTemplate._post_load_data(template_code, company, data)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Accounting - MRP',
    'version': '1.0',
    'category': 'Manufacturing/Manufacturing',
    'summary': 'Analytic accounting in Manufacturing',
    'description': """
Analytic Accounting in MRP
==========================

* Cost structure report

Also, allows to compute the cost of the product based on its BoM, using the costs of its components and work center operations.
It adds a button on the product itself but also an action in the list view of the products.
If the automated inventory valuation is active, the necessary accounting entries will be created.

""",
    'website': 'https://www.odoo.com/app/manufacturing',
    'depends': ['mrp', 'stock_account'],
    "data": [
        'security/ir.model.access.csv',
        "views/product_views.xml",
        "views/mrp_bom_views.xml",
        "views/mrp_production_views.xml",
        "views/analytic_account_views.xml",
        "report/report_mrp_templates.xml"
    ],
    'demo': [
        'data/mrp_account_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'post_init_hook': '_configure_journals',
    'license': 'LGPL-3',
}

```

## File: data\mrp_account_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Create extra MOs that will have stock.valuation.layers + can be used to populate reports -->
    <record id="mrp_production_drawer_2" model="mrp.production">
        <field name="product_id" ref="product.product_product_27"/>
        <field name="product_uom_id" ref="uom.product_uom_unit"/>
        <field name="product_qty">5</field>
        <field name="location_src_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_stock"/>
        <field name="bom_id" ref="mrp.mrp_bom_drawer"/>
        <field name="create_date" eval="(datetime.now() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="date_start" eval="(datetime.now() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>

    <record id="mrp_production_drawer_3" model="mrp.production">
        <field name="product_id" ref="product.product_product_27"/>
        <field name="product_uom_id" ref="uom.product_uom_unit"/>
        <field name="product_qty">3</field>
        <field name="location_src_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_stock"/>
        <field name="bom_id" ref="mrp.mrp_bom_drawer"/>
        <field name="create_date" eval="(datetime.now() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="date_start" eval="(datetime.now() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>

    <function model="mrp.production" name="action_confirm" eval="[[
        ref('mrp_account.mrp_production_drawer_2'),
        ref('mrp_account.mrp_production_drawer_3'),
    ]]"/>

    <function model="mrp.production" name="write">
        <value eval="[ref('mrp_account.mrp_production_drawer_2')]"/>
        <value eval="{'qty_producing': 5, 'lot_producing_id': ref('mrp.lot_product_27_0')}"/>
    </function>
    <function model="mrp.production" name="write">
        <value eval="[ref('mrp_account.mrp_production_drawer_3')]"/>
        <value eval="{'qty_producing': 3, 'lot_producing_id': ref('mrp.lot_product_27_0')}"/>
    </function>

    <function model="mrp.production" name="action_assign">
        <value eval="[ref('mrp_account.mrp_production_drawer_2'),
                        ref('mrp_account.mrp_production_drawer_3')]"/>
    </function>

    <function model="stock.move" name="write">
        <value model="stock.move" eval="obj().env['stock.move'].search([('raw_material_production_id', '=', obj().env.ref('mrp_account.mrp_production_drawer_2').id)]).ids"/>
        <value eval="{'quantity': 5}"/>
    </function>
    <function model="stock.move" name="write">
        <value model="stock.move" eval="obj().env['stock.move'].search([('raw_material_production_id', '=', obj().env.ref('mrp_account.mrp_production_drawer_3').id)]).ids"/>
        <value eval="{'quantity': 4}"/>
    </function>

    <function model="mrp.production" name="_post_inventory">
        <value eval="[ref('mrp_account.mrp_production_drawer_2'),
                        ref('mrp_account.mrp_production_drawer_3')]"/>
    </function>

    <function model="mrp.production" name="button_mark_done">
        <value eval="[ref('mrp_account.mrp_production_drawer_2'),
                        ref('mrp_account.mrp_production_drawer_3')]"/>
    </function>

    <!-- Rewrite history so they finished in the past so it looks more interesting in graphs -->
    <function model="mrp.production" name="write">
        <value eval="[ref('mrp_account.mrp_production_drawer_2')]"/>
        <value eval="{'date_finished': (datetime.now() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M:%S')}"/>
    </function>
    <function model="mrp.production" name="write">
        <value eval="[ref('mrp_account.mrp_production_drawer_3')]"/>
        <value eval="{'date_finished': (datetime.now() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M:%S')}"/>
    </function>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

from collections import defaultdict


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def _get_invoiced_qty_per_product(self):
        # Replace the kit-type products with their components
        qties = defaultdict(float)
        res = super()._get_invoiced_qty_per_product()
        invoiced_products = self.env['product.product'].concat(*res.keys())
        bom_kits = self.env['mrp.bom']._bom_find(invoiced_products, company_id=self.company_id[:1].id, bom_type='phantom')
        for product, qty in res.items():
            bom_kit = bom_kits[product]
            if bom_kit:
                invoiced_qty = product.uom_id._compute_quantity(qty, bom_kit.product_uom_id, round=False)
                factor = invoiced_qty / bom_kit.product_qty
                dummy, bom_sub_lines = bom_kit.explode(product, factor)
                for bom_line, bom_line_data in bom_sub_lines:
                    qties[bom_line.product_id] += bom_line.product_uom_id._compute_quantity(bom_line_data['qty'], bom_line.product_id.uom_id)
            else:
                qties[product] += qty
        return qties

```

## File: models\analytic_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class AccountAnalyticAccount(models.Model):
    _inherit = 'account.analytic.account'
    _description = 'Analytic Account'

    production_ids = fields.Many2many('mrp.production')
    production_count = fields.Integer("Manufacturing Orders Count", compute='_compute_production_count')
    bom_ids = fields.Many2many('mrp.bom')
    bom_count = fields.Integer("BoM Count", compute='_compute_bom_count')
    workcenter_ids = fields.Many2many('mrp.workcenter')
    workorder_count = fields.Integer("Work Order Count", compute='_compute_workorder_count')

    @api.depends('production_ids')
    def _compute_production_count(self):
        for account in self:
            account.production_count = len(account.production_ids)

    @api.depends('bom_ids')
    def _compute_bom_count(self):
        for account in self:
            account.bom_count = len(account.bom_ids)

    @api.depends('workcenter_ids.order_ids', 'production_ids.workorder_ids')
    def _compute_workorder_count(self):
        for account in self:
            account.workorder_count = len(account.workcenter_ids.order_ids | account.production_ids.workorder_ids)

    def action_view_mrp_production(self):
        self.ensure_one()
        result = {
            "type": "ir.actions.act_window",
            "res_model": "mrp.production",
            "domain": [['id', 'in', self.production_ids.ids]],
            "name": _("Manufacturing Orders"),
            'view_mode': 'tree,form',
            "context": {'default_analytic_account_id': self.id},
        }
        if len(self.production_ids) == 1:
            result['view_mode'] = 'form'
            result['res_id'] = self.production_ids.id
        return result

    def action_view_mrp_bom(self):
        self.ensure_one()
        result = {
            "type": "ir.actions.act_window",
            "res_model": "mrp.bom",
            "domain": [['id', 'in', self.bom_ids.ids]],
            "name": _("Bills of Materials"),
            'view_mode': 'tree,form',
            "context": {'default_analytic_account_id': self.id},
        }
        if self.bom_count == 1:
            result['view_mode'] = 'form'
            result['res_id'] = self.bom_ids.id
        return result

    def action_view_workorder(self):
        self.ensure_one()
        result = {
            "type": "ir.actions.act_window",
            "res_model": "mrp.workorder",
            "domain": [['id', 'in', (self.workcenter_ids.order_ids | self.production_ids.workorder_ids).ids]],
            "context": {"create": False},
            "name": _("Work Orders"),
            'view_mode': 'tree',
        }
        return result


class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    category = fields.Selection(selection_add=[('manufacturing_order', 'Manufacturing Order')])

class AccountAnalyticApplicability(models.Model):
    _inherit = 'account.analytic.applicability'
    _description = "Analytic Plan's Applicabilities"

    business_domain = fields.Selection(
        selection_add=[
            ('manufacturing_order', 'Manufacturing Order'),
        ],
        ondelete={'manufacturing_order': 'cascade'},
    )

```

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json

from odoo import models, api, fields


class MrpBom(models.Model):
    _name = 'mrp.bom'
    _inherit = ['mrp.bom', 'analytic.mixin']

    # Company dependent JSON fields are not yet supported
    analytic_distribution_text = fields.Text(company_dependent=True)
    analytic_distribution = fields.Json(inverse="_inverse_analytic_distribution", store=False, precompute=False)
    analytic_account_ids = fields.Many2many('account.analytic.account', compute="_compute_analytic_account_ids", copy=True)

    @api.depends_context('company')
    @api.depends('analytic_distribution_text')
    def _compute_analytic_distribution(self):
        for record in self:
            record.analytic_distribution = json.loads(record.analytic_distribution_text or '{}')

    def _inverse_analytic_distribution(self):
        for record in self:
            record.analytic_distribution_text = json.dumps(record.analytic_distribution)

    @api.depends('analytic_distribution')
    def _compute_analytic_account_ids(self):
        for record in self:
            record.analytic_account_ids = bool(record.analytic_distribution) and self.env['account.analytic.account'].browse(
                list({int(account_id) for ids in record.analytic_distribution for account_id in ids.split(",")})
            ).exists()

    @api.onchange('product_id')
    def _onchange_analytic_distribution(self):
        for record in self:
            if record.product_id:
                record.analytic_distribution = record.env['account.analytic.distribution.model'].sudo()._get_distribution({
                    "product_id": record.product_id.id,
                    "product_categ_id": record.product_id.categ_id.id,
                    "company_id": record.company_id.id,
                })

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from ast import literal_eval

from odoo import api, fields, models, _
from odoo.tools import float_round


class MrpProduction(models.Model):
    _name = 'mrp.production'
    _inherit = ['mrp.production', 'analytic.mixin']

    extra_cost = fields.Float(copy=False, string='Extra Unit Cost')
    show_valuation = fields.Boolean(compute='_compute_show_valuation')
    analytic_account_ids = fields.Many2many('account.analytic.account', compute='_compute_analytic_account_ids', store=True)

    def _compute_show_valuation(self):
        for order in self:
            order.show_valuation = any(m.state == 'done' for m in order.move_finished_ids)

    @api.depends('bom_id', 'product_id')
    def _compute_analytic_distribution(self):
        for record in self:
            if record.bom_id.analytic_distribution:
                record.analytic_distribution = record.bom_id.analytic_distribution
            else:
                record.analytic_distribution = record.env['account.analytic.distribution.model']._get_distribution({
                    "product_id": record.product_id.id,
                    "product_categ_id": record.product_id.categ_id.id,
                    "company_id": record.company_id.id,
                })

    @api.depends('analytic_distribution')
    def _compute_analytic_account_ids(self):
        for record in self:
            record.analytic_account_ids = bool(record.analytic_distribution) and self.env['account.analytic.account'].browse(
                list({int(account_id) for ids in record.analytic_distribution for account_id in ids.split(",")})
            ).exists()

    @api.constrains('analytic_distribution')
    def _check_analytic(self):
        for record in self:
            params = {'business_domain': 'manufacturing_order', 'company_id': record.company_id.id}
            if record.product_id:
                params['product'] = record.product_id.id
            record.with_context({'validate_analytic': True})._validate_distribution(**params)

    def write(self, vals):
        res = super().write(vals)
        for production in self:
            if vals.get('name'):
                production.move_raw_ids.analytic_account_line_ids.ref = production.display_name
                for workorder in production.workorder_ids:
                    workorder.mo_analytic_account_line_ids.ref = production.display_name
                    workorder.mo_analytic_account_line_ids.name = _("[WC] %s", workorder.display_name)
            if 'analytic_distribution' in vals and production.state != 'draft':
                production.move_raw_ids._account_analytic_entry_move()
                production.workorder_ids._create_or_update_analytic_entry()
        return res

    def action_view_stock_valuation_layers(self):
        self.ensure_one()
        domain = [('id', 'in', (self.move_raw_ids + self.move_finished_ids + self.scrap_ids.move_ids).stock_valuation_layer_ids.ids)]
        action = self.env["ir.actions.actions"]._for_xml_id("stock_account.stock_valuation_layer_action")
        context = literal_eval(action['context'])
        context.update(self.env.context)
        context['no_at_date'] = True
        context['search_default_group_by_product_id'] = False
        return dict(action, domain=domain, context=context)

    def action_view_analytic_accounts(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "res_model": "account.analytic.account",
            'domain': [('id', 'in', self.analytic_account_ids.ids)],
            "name": _("Analytic Accounts"),
            'view_mode': 'tree,form',
        }

    def _cal_price(self, consumed_moves):
        """Set a price unit on the finished move according to `consumed_moves`.
        """
        super(MrpProduction, self)._cal_price(consumed_moves)
        work_center_cost = 0
        finished_move = self.move_finished_ids.filtered(
            lambda x: x.product_id == self.product_id and x.state not in ('done', 'cancel') and x.quantity > 0)
        if finished_move:
            finished_move.ensure_one()
            for work_order in self.workorder_ids:
                work_center_cost += work_order._cal_cost()
            quantity = finished_move.product_uom._compute_quantity(
                finished_move.quantity, finished_move.product_id.uom_id)
            extra_cost = self.extra_cost * quantity
            total_cost = - sum(consumed_moves.sudo().stock_valuation_layer_ids.mapped('value')) + work_center_cost + extra_cost
            byproduct_moves = self.move_byproduct_ids.filtered(lambda m: m.state not in ('done', 'cancel') and m.quantity > 0)
            byproduct_cost_share = 0
            for byproduct in byproduct_moves:
                if byproduct.cost_share == 0:
                    continue
                byproduct_cost_share += byproduct.cost_share
                if byproduct.product_id.cost_method in ('fifo', 'average'):
                    byproduct.price_unit = total_cost * byproduct.cost_share / 100 / byproduct.product_uom._compute_quantity(byproduct.quantity, byproduct.product_id.uom_id)
            if finished_move.product_id.cost_method in ('fifo', 'average'):
                finished_move.price_unit = total_cost * float_round(1 - byproduct_cost_share / 100, precision_rounding=0.0001) / quantity
        return True

    def _get_backorder_mo_vals(self):
        res = super()._get_backorder_mo_vals()
        res['extra_cost'] = self.extra_cost
        return res

```

## File: models\mrp_routing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MrpRoutingWorkcenter(models.Model):
    _inherit = 'mrp.routing.workcenter'

    def _total_cost_per_hour(self):
        self.ensure_one()
        return self.workcenter_id.costs_hour

```

## File: models\mrp_workcenter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class MrpWorkcenter(models.Model):
    _name = 'mrp.workcenter'
    _inherit = ['mrp.workcenter', 'analytic.mixin']

    costs_hour_account_ids = fields.Many2many('account.analytic.account', compute="_compute_costs_hour_account_ids", store=True)

    @api.depends('analytic_distribution')
    def _compute_costs_hour_account_ids(self):
        for record in self:
            record.costs_hour_account_ids = bool(record.analytic_distribution) and self.env['account.analytic.account'].browse(
                list({int(account_id) for ids in record.analytic_distribution for account_id in ids.split(",")})
            ).exists()

```

## File: models\mrp_workorder.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class MrpWorkorder(models.Model):
    _inherit = 'mrp.workorder'

    mo_analytic_account_line_ids = fields.Many2many('account.analytic.line', 'mrp_workorder_mo_analytic_rel', copy=False)
    wc_analytic_account_line_ids = fields.Many2many('account.analytic.line', 'mrp_workorder_wc_analytic_rel', copy=False)

    def _compute_duration(self):
        res = super()._compute_duration()
        self._create_or_update_analytic_entry()
        return res

    def _set_duration(self):
        res = super()._set_duration()
        self._create_or_update_analytic_entry()
        return res

    def action_cancel(self):
        (self.mo_analytic_account_line_ids | self.wc_analytic_account_line_ids).unlink()
        return super().action_cancel()

    def _prepare_analytic_line_values(self, account_field_values, amount, unit_amount):
        self.ensure_one()
        return {
            'name': _("[WC] %s", self.display_name),
            'amount': amount,
            **account_field_values,
            'unit_amount': unit_amount,
            'product_id': self.product_id.id,
            'product_uom_id': self.env.ref('uom.product_uom_hour').id,
            'company_id': self.company_id.id,
            'ref': self.production_id.name,
            'category': 'manufacturing_order',
        }

    def _create_or_update_analytic_entry(self):
        for wo in self.filtered(lambda wo: wo.id and (wo.production_id.analytic_distribution or wo.workcenter_id.analytic_distribution or wo.wc_analytic_account_line_ids or wo.mo_analytic_account_line_ids)):
            hours = wo.duration / 60.0
            value = -hours * wo.workcenter_id.costs_hour

            mo_analytic_line_vals = self.env['account.analytic.account']._perform_analytic_distribution(wo.production_id.analytic_distribution, value, hours, wo.mo_analytic_account_line_ids, wo)
            if mo_analytic_line_vals:
                wo.mo_analytic_account_line_ids += self.env['account.analytic.line'].sudo().create(mo_analytic_line_vals)

            wc_analytic_line_vals = self.env['account.analytic.account']._perform_analytic_distribution(wo.workcenter_id.analytic_distribution, value, hours, wo.wc_analytic_account_line_ids, wo)
            if wc_analytic_line_vals:
                wo.wc_analytic_account_line_ids += self.env['account.analytic.line'].sudo().create(wc_analytic_line_vals)

    def unlink(self):
        (self.mo_analytic_account_line_ids | self.wc_analytic_account_line_ids).unlink()
        return super().unlink()

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.osv import expression
from odoo.tools import float_round, groupby


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

    def _get_product_accounts(self):
        accounts = super()._get_product_accounts()
        accounts.update({
            'production': self.categ_id.property_stock_account_production_cost_id,
        })
        return accounts

    def action_bom_cost(self):
        templates = self.filtered(lambda t: t.product_variant_count == 1 and t.bom_count > 0)
        if templates:
            return templates.mapped('product_variant_id').action_bom_cost()

    def button_bom_cost(self):
        templates = self.filtered(lambda t: t.product_variant_count == 1 and t.bom_count > 0)
        if templates:
            return templates.mapped('product_variant_id').button_bom_cost()


class ProductProduct(models.Model):
    _name = 'product.product'
    _inherit = 'product.product'

    def button_bom_cost(self):
        self.ensure_one()
        self._set_price_from_bom()

    def action_bom_cost(self):
        boms_to_recompute = self.env['mrp.bom'].search(['|', ('product_id', 'in', self.ids), '&', ('product_id', '=', False), ('product_tmpl_id', 'in', self.mapped('product_tmpl_id').ids)])
        for product in self:
            product._set_price_from_bom(boms_to_recompute)

    def _set_price_from_bom(self, boms_to_recompute=False):
        self.ensure_one()
        bom = self.env['mrp.bom']._bom_find(self)[self]
        if bom:
            self.standard_price = self._compute_bom_price(bom, boms_to_recompute=boms_to_recompute)
        else:
            bom = self.env['mrp.bom'].search([('byproduct_ids.product_id', '=', self.id)], order='sequence, product_id, id', limit=1)
            if bom:
                price = self._compute_bom_price(bom, boms_to_recompute=boms_to_recompute, byproduct_bom=True)
                if price:
                    self.standard_price = price

    def _compute_average_price(self, qty_invoiced, qty_to_invoice, stock_moves, is_returned=False):
        self.ensure_one()
        if stock_moves.product_id == self:
            return super()._compute_average_price(qty_invoiced, qty_to_invoice, stock_moves, is_returned=is_returned)
        bom = self.env['mrp.bom']._bom_find(self, company_id=stock_moves.company_id.id, bom_type='phantom')[self]
        if not bom:
            return super()._compute_average_price(qty_invoiced, qty_to_invoice, stock_moves, is_returned=is_returned)
        value = 0
        dummy, bom_lines = bom.explode(self, 1)
        bom_lines = {line: data for line, data in bom_lines}
        for bom_line, moves_list in groupby(stock_moves.filtered(lambda sm: sm.state != 'cancel'), lambda sm: sm.bom_line_id):
            if bom_line not in bom_lines:
                for move in moves_list:
                    component_quantity = next(
                        (bml.product_qty for bml in move.product_id.bom_line_ids if bml in bom_lines),
                        1
                    )
                    value += component_quantity * move.product_id._compute_average_price(qty_invoiced * move.product_qty, qty_to_invoice * move.product_qty, move, is_returned=is_returned)
                continue
            line_qty = bom_line.product_uom_id._compute_quantity(bom_lines[bom_line]['qty'], bom_line.product_id.uom_id)
            moves = self.env['stock.move'].concat(*moves_list)
            value += line_qty * bom_line.product_id._compute_average_price(qty_invoiced * line_qty, qty_to_invoice * line_qty, moves, is_returned=is_returned)
        return value

    def _compute_bom_price(self, bom, boms_to_recompute=False, byproduct_bom=False):
        self.ensure_one()
        if not bom:
            return 0
        if not boms_to_recompute:
            boms_to_recompute = []
        total = 0
        for opt in bom.operation_ids:
            if opt._skip_operation_line(self):
                continue

            duration_expected = (
                opt.workcenter_id._get_expected_duration(self) +
                opt.time_cycle * 100 / opt.workcenter_id.time_efficiency)
            total += (duration_expected / 60) * opt._total_cost_per_hour()

        for line in bom.bom_line_ids:
            if line._skip_bom_line(self):
                continue

            # Compute recursive if line has `child_line_ids`
            if line.child_bom_id and line.child_bom_id in boms_to_recompute:
                child_total = line.product_id._compute_bom_price(line.child_bom_id, boms_to_recompute=boms_to_recompute)
                total += line.product_id.uom_id._compute_price(child_total, line.product_uom_id) * line.product_qty
            else:
                total += line.product_id.uom_id._compute_price(line.product_id.standard_price, line.product_uom_id) * line.product_qty
        if byproduct_bom:
            byproduct_lines = bom.byproduct_ids.filtered(lambda b: b.product_id == self and b.cost_share != 0)
            product_uom_qty = 0
            for line in byproduct_lines:
                product_uom_qty += line.product_uom_id._compute_quantity(line.product_qty, self.uom_id, round=False)
            byproduct_cost_share = sum(byproduct_lines.mapped('cost_share'))
            if byproduct_cost_share and product_uom_qty:
                return total * byproduct_cost_share / 100 / product_uom_qty
        else:
            byproduct_cost_share = sum(bom.byproduct_ids.mapped('cost_share'))
            if byproduct_cost_share:
                total *= float_round(1 - byproduct_cost_share / 100, precision_rounding=0.0001)
            return bom.product_uom_id._compute_price(total / bom.product_qty, self.uom_id)


class ProductCategory(models.Model):
    _inherit = 'product.category'

    property_stock_account_production_cost_id = fields.Many2one(
        'account.account', 'Production Account', company_dependent=True,
        domain="[('deprecated', '=', False)]", check_company=True,
        help="""This account will be used as a valuation counterpart for both components and final products for manufacturing orders.
                If there are any workcenter/employee costs, this value will remain on the account once the production is completed.""")

    def write(self, vals):
        return super(ProductCategory, self.with_context(product_category_mrp_account_write=True)).write(vals)

    def _get_stock_account_property_field_names(self):
        mrp_value = ['property_stock_account_production_cost_id'] if self.property_stock_account_production_cost_id or not self.env.context.get('product_category_mrp_account_write') else []
        return super()._get_stock_account_property_field_names() + mrp_value

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import models


class StockMove(models.Model):
    _inherit = "stock.move"

    def _filter_anglo_saxon_moves(self, product):
        res = super(StockMove, self)._filter_anglo_saxon_moves(product)
        res += self.filtered(lambda m: m.bom_line_id.bom_id.product_tmpl_id.id == product.product_tmpl_id.id)
        return res

    def _get_analytic_distribution(self):
        distribution = self.raw_material_production_id.analytic_distribution
        if distribution:
            return distribution
        return super()._get_analytic_distribution()

    def _should_force_price_unit(self):
        self.ensure_one()
        return ((self.picking_type_id.code == 'mrp_operation' and self.production_id) or
                super()._should_force_price_unit()
        )

    def _ignore_automatic_valuation(self):
        return bool(self.raw_material_production_id)

    def _get_src_account(self, accounts_data):
        if self._is_production():
            return self.location_id.valuation_out_account_id.id or accounts_data['production'].id or accounts_data['stock_input'].id
        return super()._get_src_account(accounts_data)

    def _get_dest_account(self, accounts_data):
        if self._is_production_consumed():
            return self.location_dest_id.valuation_in_account_id.id or accounts_data['production'].id or accounts_data['stock_output'].id
        return super()._get_dest_account(accounts_data)

    def _is_production(self):
        self.ensure_one()
        return self.location_id.usage == 'production' and self.location_dest_id._should_be_valued()

    def _is_production_consumed(self):
        self.ensure_one()
        return self.location_dest_id.usage == 'production' and self.location_id._should_be_valued()

    def _create_out_svl(self, forced_quantity=None):
        product_unbuild_map = defaultdict(self.env['mrp.unbuild'].browse)
        for move in self:
            if move.unbuild_id:
                product_unbuild_map[move.product_id] |= move.unbuild_id
        return super(StockMove, self.with_context(product_unbuild_map=product_unbuild_map))._create_out_svl(forced_quantity)

    def _get_out_svl_vals(self, forced_quantity):
        unbuild_moves = self.filtered('unbuild_id')
        # 'real cost' of finished product moves @ build time
        price_unit_map = {
            move.id: (
                move.unbuild_id.mo_id.move_finished_ids.stock_valuation_layer_ids.filtered(
                    lambda svl: svl.product_id == move.product_id
                )[0].unit_cost,
                move.company_id.currency_id.round,
            )
            for move in unbuild_moves.sudo()
            if move.product_id.cost_method != 'standard' and
            move.unbuild_id.mo_id.move_finished_ids.stock_valuation_layer_ids
        }
        svl_vals_list = super()._get_out_svl_vals(forced_quantity)
        if price_unit_map:
            for svl_vals in svl_vals_list:
                if (move_id := svl_vals['stock_move_id']) in price_unit_map:
                    unit_cost = price_unit_map[move_id][0]
                    svl_vals.update({
                        'unit_cost': unit_cost,
                        'value': price_unit_map[move_id][1](unit_cost * svl_vals['quantity']),
                    })
        return svl_vals_list

    def _get_all_related_sm(self, product):
        moves = super()._get_all_related_sm(product)
        return moves | self.filtered(
            lambda m:
            m.bom_line_id.bom_id.type == 'phantom' and
            m.bom_line_id.bom_id == moves.bom_line_id.bom_id
        )

```

## File: models\stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _prepare_mo_vals(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom):
        res = super()._prepare_mo_vals(product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom)
        if not bom.analytic_distribution:
            if values.get('analytic_distribution'):
                res['analytic_distribution'] = values.get('analytic_distribution')
            elif values.get('analytic_account_id'):
                res['analytic_distribution'] = {values.get('analytic_account_id').id: 100}
        return res

```

## File: models\stock_valuation_layer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockValuationLayer(models.Model):
    _inherit = 'stock.valuation.layer'

    def _candidate_sort_key(self):
        self.ensure_one()
        res = super()._candidate_sort_key()
        if self.product_id in self.env.context.get('product_unbuild_map', ()):
            unbuild = self.env.context['product_unbuild_map'][self.product_id]
            # Give priority to the SVL that produced `self.product_id`
            res += (self.stock_move_id.id not in unbuild.mo_id.move_finished_ids.ids,)
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import analytic_account
from . import mrp_bom
from . import mrp_workcenter
from . import mrp_workorder
from . import mrp_production
from . import mrp_routing
from . import product
from . import stock_move
from . import stock_rule
from . import account_move
from . import stock_valuation_layer

```

## File: report\mrp_report_mo_overview.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import float_is_zero

class ReportMoOverview(models.AbstractModel):
    _inherit = 'report.mrp.report_mo_overview'

    def _get_unit_cost(self, move):
        valuation_layers = move.sudo().stock_valuation_layer_ids
        layers_quantity = sum(valuation_layers.mapped('quantity'))
        if valuation_layers and not float_is_zero(layers_quantity, precision_rounding=valuation_layers.uom_id.rounding):
            unit_price = sum(valuation_layers.mapped('value')) / layers_quantity
            return move.product_id.uom_id._compute_price(unit_price, move.product_uom)
        return super()._get_unit_cost(move)

```

## File: report\report_mrp_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="mrp_account.report_wip">
        <t t-call="web.html_container">
            <t t-call="web.external_layout">
                <div class="page">
                    <div class="oe_structure"/>
                    <div class="row mt8">
                        <div class="col-lg-12">
                            <h2>
                                <span>WIP Report for
                                    <t t-out="', '.join(docs.account_id.mapped('name'))">Acme Corp.</t>
                                </span>
                            </h2>
                        </div>
                    </div>
                    <div class="oe_structure"/>
                    <div class="row mt8">
                            <div class="col-12">
                                <table class="table table-sm">
                                    <thead>
                                        <tr>
                                            <th class="text-start align-middle"><span>Date</span></th>
                                            <th class="text-start align-middle"><span>Ref.</span></th>
                                            <th class="text-start align-middle"><span>Product</span></th>
                                            <th class="text-start align-middle"><span>Quantity</span></th>
                                            <th class="text-start align-middle"><span>Unit of Measure</span></th>
                                            <th class="text-end"><span>Amount</span></th>
                                        </tr>
                                   </thead>
                                   <tbody>
                                        <tr t-foreach="docs" t-as="line" t-att-style="'background-color: #F1F1F1;' if line_index % 2 == 0 else ''">
                                            <td class="align-middle">
                                               <span t-field="line.date">2023-08-15</span>
                                            </td>
                                            <td class="align-middle">
                                               <span t-field="line.ref">REF123</span>
                                            </td>
                                            <td class="align-middle">
                                                <span t-field="line.product_id">Laptop</span>
                                            </td>
                                            <td class="align-middle">
                                                <span t-field="line.unit_amount">5</span>
                                            </td>
                                            <td class="align-middle">
                                                <span t-field="line.product_uom_id">Units</span>
                                            </td>
                                            <td class="text-end align-middle">
                                                <span t-field="line.amount">$1000</span>
                                            </td>
                                        </tr>
                                        <tr>
                                            <td class="text-end" t-attf-colspan="6">
                                                <strong>
                                                    <span style="margin-right: 15px;">Total</span>
                                                    <span t-out="sum(docs.mapped('amount'))">$5000</span>
                                                </strong>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    <div class="oe_structure"/>
                </div>
            </t>
        </t>
    </template>

    <record id="wip_report" model="ir.actions.report">
        <field name="name">WIP report</field>
        <field name="model">account.analytic.line</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">mrp_account.report_wip</field>
        <field name="report_file">report_wip</field>
        <field name="binding_model_id" ref="model_account_analytic_line"/>
        <field name="binding_type">report</field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_report_mo_overview

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mrp_bom_invoice_bom_readonly,mrp.bom,mrp.model_mrp_bom,account.group_account_readonly,1,0,0,0
access_mrp_bom_invoice_bom,mrp.bom,mrp.model_mrp_bom,account.group_account_invoice,1,0,0,0
access_mrp_bom_line_invoice_bom_readonly,mrp.bom.line,mrp.model_mrp_bom_line,account.group_account_readonly,1,0,0,0
access_mrp_bom_line_invoice_bom,mrp.bom.line,mrp.model_mrp_bom_line,account.group_account_invoice,1,0,0,0

```

## File: views\analytic_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_analytic_account_view_form_mrp" model="ir.ui.view">
        <field name="name">account.analytic.account.form.mrp</field>
        <field name="model">account.analytic.account</field>
        <field name="inherit_id" ref="analytic.view_account_analytic_account_form"/>
        <field eval="14" name="priority"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" type="object" name="action_view_mrp_production"
                    groups="mrp.group_mrp_user"
                    icon="fa-wrench" invisible="production_count == 0">
                    <field string="Manufacturing Orders" name="production_count" widget="statinfo"/>
                </button>
                <button class="oe_stat_button" type="object" name="action_view_mrp_bom"
                    groups="mrp.group_mrp_user"
                    icon="fa-flask" invisible="bom_count == 0">
                    <field string="Bills of Materials" name="bom_count" widget="statinfo"/>
                </button>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\mrp_bom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_bom_form_view_inherited" model="ir.ui.view">
        <field name="name">mrp.bom.form.inherited</field>
        <field name="model">mrp.bom</field>
        <field name="inherit_id" ref="mrp.mrp_bom_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='picking_type_id']" position="after">
                <field name="analytic_distribution" widget="analytic_distribution"
                       groups="analytic.group_analytic_accounting"
                       options="{'product_field': 'product_id'}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mrp_production_views.xml

```xml
<odoo>
    <record id="view_production_tree_view_inherit" model="ir.ui.view">
        <field name="name">mrp.production.tree.view.inherit</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp.mrp_production_tree_view"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="analytic_distribution" widget="analytic_distribution"
                       groups="analytic.group_analytic_accounting"
                       options="{'product_field': 'product_id', 'business_domain': 'manufacturing_order'}"
                       optional="hide" />
            </field>
        </field>
    </record>

    <record id="mrp_production_form_view_inherited" model="ir.ui.view">
        <field name="name">mrp.production.view.inherited</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="before">
                <field name="show_valuation" invisible="1" groups="stock.group_stock_manager"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <t groups="stock.group_stock_manager">
                    <button type="object"
                        name="action_view_stock_valuation_layers"
                        class="oe_stat_button" icon="fa-dollar" groups="base.group_no_one"
                        invisible="not show_valuation" >
                        <div class="o_stat_info">
                            <span class="o_stat_text">Valuation</span>
                        </div>
                    </button>
                    <button class="oe_stat_button" type="object"
                        name="action_view_analytic_accounts"
                        icon="fa-bar-chart-o"
                        invisible="not analytic_distribution or state in ['draft', 'cancel']"
                        groups="analytic.group_analytic_accounting">
                        <div class="o_stat_info">
                            <span class="o_stat_text">Analytic Account</span>
                        </div>
                    </button>
                </t>
            </xpath>
            <xpath expr="//page[@name='miscellaneous']//field[@name='date_deadline']" position="after">
                <t groups="stock.group_stock_manager">
                    <field name="analytic_distribution" widget="analytic_distribution"
                       groups="analytic.group_analytic_accounting"
                       options="{'product_field': 'product_id', 'business_domain': 'manufacturing_order'}"/>
                </t>
            </xpath>
        </field>
    </record>

    <record id="view_production_graph_inherit_mrp_account" model="ir.ui.view">
        <field name="name">mrp.production.graph.inherited.mrp.account</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp.view_production_graph"/>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="inside">
                <field name="extra_cost" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="product_product_ext_form_view2" model="ir.ui.view">
            <field name="name">product_extended.product.form.view</field>
            <field name="model">product.template</field>
            <field name="priority">3</field>
            <field name="inherit_id" ref="product.product_template_only_form_view" />
            <field name="arch" type="xml">
                <xpath expr="//div[@name='standard_price_uom']" position="inside">
                    <t groups="mrp.group_mrp_manager">
                        <field name="bom_count" invisible="1"/>
                        <field name="cost_method" invisible="1"/>
                        <field name="valuation" invisible="1"/>
                        <button name="button_bom_cost"
                            string="Compute Price from BoM" type="object"
                            invisible="bom_count == 0 or valuation == 'real_time' and cost_method == 'fifo'"
                            help="Compute the price of the product using products and operations of related bill of materials, for manufactured products only."
                            class="oe_link pt-0"/>
                        </t>
                </xpath>
            </field>
        </record>

        <record id="product_product_view_form_normal_inherit_extended" model="ir.ui.view">
            <field name="name">product.product.view.form.normal.inherit.extended</field>
            <field name="model">product.product</field>
            <field name="priority">4</field>
            <field name="inherit_id" ref="product.product_normal_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='standard_price_uom']" position="inside">
                    <t groups="mrp.group_mrp_manager">
                        <field name="bom_count" invisible="1"/>
                        <field name="cost_method" invisible="1"/>
                        <field name="valuation" invisible="1"/>
                        <button name="button_bom_cost"
                            string="Compute Price from BoM" type="object"
                            invisible="bom_count == 0 or valuation == 'real_time' and cost_method == 'fifo'"
                            help="Compute the price of the product using products and operations of related bill of materials, for manufactured products only."
                            class="oe_link pt-0"
                            colspan="2"/>
                    </t>
                </xpath>
            </field>
        </record>

        <record id="product_variant_easy_edit_view_bom_inherit" model="ir.ui.view">
            <field name="name">product.product.product.view.form.easy.bom.inherit</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_variant_easy_edit_view"/>
            <field name="arch" type="xml">
                <data>
                <xpath expr="//field[@name='standard_price']" position="after">
                    <t groups="mrp.group_mrp_manager">
                        <field name="bom_count" invisible="1"/>
                        <field name="cost_method" invisible="1"/>
                        <field name="valuation" invisible="1"/>
                        <button name="button_bom_cost"
                            string="Compute Price from BoM"
                            type="object"
                            invisible="bom_count == 0 or valuation == 'real_time' and cost_method == 'fifo'"
                            help="Compute the price of the product using products and operations of related bill of materials, for manufactured products only."
                            class="oe_link pt-0"/>
                    </t>
                </xpath>
                </data>
            </field>
        </record>

        <record id="view_category_property_form" model="ir.ui.view">
            <field name="name">product.category.stock.property.form.inherit</field>
            <field name="model">product.category</field>
            <field name="inherit_id" ref="stock_account.view_category_property_form"/>
            <field name="arch" type="xml">
                <field name="property_stock_account_output_categ_id" position="after">
                    <field name="property_stock_account_production_cost_id" options="{'no_create': True}"/>
                </field>
            </field>
        </record>

        <record id="action_compute_price_bom_template" model="ir.actions.server">
            <field name="name">Compute Price from BoM</field>
            <field name="model_id" ref="product.model_product_template"/>
            <field name="binding_model_id" ref="product.model_product_template"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
            if records:
                records.action_bom_cost()
            </field>
        </record>

        <record id="action_compute_price_bom_product" model="ir.actions.server">
            <field name="name">Compute Price from BoM</field>
            <field name="model_id" ref="product.model_product_product"/>
            <field name="binding_model_id" ref="product.model_product_product"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
            if records:
                records.action_bom_cost()
            </field>
        </record>
</odoo>

```

