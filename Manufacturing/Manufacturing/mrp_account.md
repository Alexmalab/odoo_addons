# Odoo Module: mrp_account

Category: Manufacturing/Manufacturing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

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
    ],
    'demo': [
        'data/mrp_account_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
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
        <field name="date_planned_start" eval="(datetime.now() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>

    <record id="mrp_production_drawer_3" model="mrp.production">
        <field name="product_id" ref="product.product_product_27"/>
        <field name="product_uom_id" ref="uom.product_uom_unit"/>
        <field name="product_qty">3</field>
        <field name="location_src_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_stock"/>
        <field name="bom_id" ref="mrp.mrp_bom_drawer"/>
        <field name="create_date" eval="(datetime.now() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="date_planned_start" eval="(datetime.now() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>

    <function model="stock.move" name="create">
        <value model="stock.move" eval="
            obj().env.ref('mrp_account.mrp_production_drawer_2')._get_moves_raw_values() +
            obj().env.ref('mrp_account.mrp_production_drawer_2')._get_moves_finished_values() +
            obj().env.ref('mrp_account.mrp_production_drawer_3')._get_moves_raw_values() +
            obj().env.ref('mrp_account.mrp_production_drawer_3')._get_moves_finished_values()"/>
    </function>

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
        <value eval="{'quantity_done': 5}"/>
    </function>
    <function model="stock.move" name="write">
        <value model="stock.move" eval="obj().env['stock.move'].search([('raw_material_production_id', '=', obj().env.ref('mrp_account.mrp_production_drawer_3').id)]).ids"/>
        <value eval="{'quantity_done': 4}"/>
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

    production_ids = fields.One2many('mrp.production', 'analytic_account_id', string='Manufacturing Orders')
    production_count = fields.Integer("Manufacturing Orders Count", compute='_compute_production_count')
    bom_ids = fields.One2many('mrp.bom', 'analytic_account_id', string='Bills of Materials')
    bom_count = fields.Integer("BoM Count", compute='_compute_bom_count')
    workcenter_ids = fields.One2many('mrp.workcenter', 'costs_hour_account_id', string='Workcenters')
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

```

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    analytic_account_id = fields.Many2one('account.analytic.account', 'Analytic Account', company_dependent=True, copy=True,
        help="Analytic account in which cost and revenue entries will take place for financial management of the manufacturing order.")

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, fields, models, _
from odoo.tools import float_is_zero, float_round


class MrpProductionWorkcenterLineTime(models.Model):
    _inherit = 'mrp.workcenter.productivity'

    cost_already_recorded = fields.Boolean('Cost Recorded', help="Technical field automatically checked when a ongoing production posts journal entries for its costs. This way, we can record one production's cost multiple times and only consider new entries in the work centers time lines.")


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    extra_cost = fields.Float(copy=False, help='Extra cost per produced unit')
    show_valuation = fields.Boolean(compute='_compute_show_valuation')
    analytic_account_id = fields.Many2one(
        'account.analytic.account', 'Analytic Account', copy=True,
        help="Analytic account in which cost and revenue entries will take\
        place for financial management of the manufacturing order.",
        compute='_compute_analytic_account_id', store=True, readonly=False)

    def _compute_show_valuation(self):
        for order in self:
            order.show_valuation = any(m.state == 'done' for m in order.move_finished_ids)

    @api.depends('bom_id')
    def _compute_analytic_account_id(self):
        for order in self:
            if order.bom_id.analytic_account_id:
                order.analytic_account_id = order.bom_id.analytic_account_id

    def write(self, vals):
        origin_analytic_account = {production: production.analytic_account_id for production in self}
        res = super().write(vals)
        for production in self:
            if vals.get('name'):
                production.move_raw_ids.analytic_account_line_id.ref = production.display_name
                for workorder in production.workorder_ids:
                    workorder.mo_analytic_account_line_id.ref = production.display_name
                    workorder.mo_analytic_account_line_id.name = _("[WC] %s", workorder.display_name)
            if 'analytic_account_id' in vals and production.state != 'draft':
                if vals['analytic_account_id'] and origin_analytic_account[production]:
                    # Link the account analytic lines to the new AA
                    production.move_raw_ids.analytic_account_line_id.write({'account_id': vals['analytic_account_id']})
                    production.workorder_ids.mo_analytic_account_line_id.write({'account_id': vals['analytic_account_id']})
                elif vals['analytic_account_id'] and not origin_analytic_account[production]:
                    # Create the account analytic lines if no AA is set in the MO
                    production.move_raw_ids._account_analytic_entry_move()
                    production.workorder_ids._create_or_update_analytic_entry()
                else:
                    production.move_raw_ids.analytic_account_line_id.unlink()
                    production.workorder_ids.mo_analytic_account_line_id.unlink()
        return res

    def action_view_stock_valuation_layers(self):
        self.ensure_one()
        domain = [('id', 'in', (self.move_raw_ids + self.move_finished_ids + self.scrap_ids.move_id).stock_valuation_layer_ids.ids)]
        action = self.env["ir.actions.actions"]._for_xml_id("stock_account.stock_valuation_layer_action")
        context = literal_eval(action['context'])
        context.update(self.env.context)
        context['no_at_date'] = True
        context['search_default_group_by_product_id'] = False
        return dict(action, domain=domain, context=context)

    def action_view_analytic_account(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "res_model": "account.analytic.account",
            'res_id': self.analytic_account_id.id,
            "context": {"create": False},
            "name": _("Analytic Account"),
            'view_mode': 'form',
        }

    def _cal_price(self, consumed_moves):
        """Set a price unit on the finished move according to `consumed_moves`.
        """
        super(MrpProduction, self)._cal_price(consumed_moves)
        work_center_cost = 0
        finished_move = self.move_finished_ids.filtered(
            lambda x: x.product_id == self.product_id and x.state not in ('done', 'cancel') and x.quantity_done > 0)
        if finished_move:
            finished_move.ensure_one()
            for work_order in self.workorder_ids:
                time_lines = work_order.time_ids.filtered(
                    lambda x: x.date_end and not x.cost_already_recorded)
                duration = sum(time_lines.mapped('duration'))
                time_lines.write({'cost_already_recorded': True})
                work_center_cost += (duration / 60.0) * \
                    work_order.workcenter_id.costs_hour
            qty_done = finished_move.product_uom._compute_quantity(
                finished_move.quantity_done, finished_move.product_id.uom_id)
            extra_cost = self.extra_cost * qty_done
            total_cost = - sum(consumed_moves.sudo().stock_valuation_layer_ids.mapped('value')) + work_center_cost + extra_cost
            byproduct_moves = self.move_byproduct_ids.filtered(lambda m: m.state not in ('done', 'cancel') and m.quantity_done > 0)
            byproduct_cost_share = 0
            for byproduct in byproduct_moves:
                if byproduct.cost_share == 0:
                    continue
                byproduct_cost_share += byproduct.cost_share
                if byproduct.product_id.cost_method in ('fifo', 'average'):
                    byproduct.price_unit = total_cost * byproduct.cost_share / 100 / byproduct.product_uom._compute_quantity(byproduct.quantity_done, byproduct.product_id.uom_id)
            if finished_move.product_id.cost_method in ('fifo', 'average'):
                finished_move.price_unit = total_cost * float_round(1 - byproduct_cost_share / 100, precision_rounding=0.0001) / qty_done
        return True

    def _get_backorder_mo_vals(self):
        res = super()._get_backorder_mo_vals()
        res['extra_cost'] = self.extra_cost
        return res

```

## File: models\mrp_workcenter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MrpWorkcenter(models.Model):
    _inherit = 'mrp.workcenter'

    costs_hour_account_id = fields.Many2one(
        'account.analytic.account', string='Analytic Account',
        help="Fill this only if you want automatic analytic accounting entries on production orders.")

```

## File: models\mrp_workorder.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.tools import float_is_zero


class MrpWorkorder(models.Model):
    _inherit = 'mrp.workorder'

    mo_analytic_account_line_id = fields.Many2one('account.analytic.line', copy=False)
    wc_analytic_account_line_id = fields.Many2one('account.analytic.line', copy=False)

    def _compute_duration(self):
        res = super()._compute_duration()
        self._create_or_update_analytic_entry()
        return res

    def _set_duration(self):
        res = super()._set_duration()
        self._create_or_update_analytic_entry()
        return res

    def action_cancel(self):
        (self.mo_analytic_account_line_id | self.wc_analytic_account_line_id).unlink()
        return super().action_cancel()

    def _prepare_analytic_line_values(self, account, unit_amount, amount):
        self.ensure_one()
        return {
            'name': _("[WC] %s", self.display_name),
            'amount': amount,
            'account_id': account.id,
            'unit_amount': unit_amount,
            'product_id': self.product_id.id,
            'product_uom_id': self.env.ref('uom.product_uom_hour').id,
            'company_id': self.company_id.id,
            'ref': self.production_id.name,
            'category': 'manufacturing_order',
        }

    def _create_or_update_analytic_entry(self):
        wo_to_link_mo_analytic_line = self.env['mrp.workorder']
        wo_to_link_wc_analytic_line = self.env['mrp.workorder']
        mo_analytic_line_vals_list = []
        wc_analytic_line_vals_list = []
        for wo in self.filtered(lambda wo: wo.production_id.analytic_account_id or wo.workcenter_id.costs_hour_account_id):
            hours = wo.duration / 60.0
            value = -hours * wo.workcenter_id.costs_hour
            mo_account = wo.production_id.analytic_account_id
            wc_account = wo.workcenter_id.costs_hour_account_id
            if mo_account:
                mo_currency = mo_account.currency_id or wo.company_id.currency_id
                is_zero = float_is_zero(value, precision_rounding=mo_currency.rounding)
                if wo.mo_analytic_account_line_id:
                    wo.mo_analytic_account_line_id.write({
                        'unit_amount': hours,
                        'amount': value if not is_zero else 0,
                    })
                elif not is_zero:
                    wo_to_link_mo_analytic_line += wo
                    mo_analytic_line_vals_list.append(wo._prepare_analytic_line_values(mo_account, hours, value))
            if wc_account and wc_account != mo_account:
                wc_currency = wc_account.currency_id or wo.company_id.currency_id
                is_zero = float_is_zero(value, precision_rounding=wc_currency.rounding)
                if wo.wc_analytic_account_line_id:
                    wo.wc_analytic_account_line_id.write({
                        'unit_amount': hours,
                        'amount': value if not is_zero else 0,
                    })
                elif not is_zero:
                    wo_to_link_wc_analytic_line += wo
                    wc_analytic_line_vals_list.append(wo._prepare_analytic_line_values(wc_account, hours, value))
        analytic_lines = self.env['account.analytic.line'].sudo().create(mo_analytic_line_vals_list + wc_analytic_line_vals_list)
        mo_analytic_lines, wc_analytic_lines = analytic_lines[:len(wo_to_link_mo_analytic_line)], analytic_lines[len(wo_to_link_mo_analytic_line):]
        for wo, analytic_line in zip(wo_to_link_mo_analytic_line, mo_analytic_lines):
            wo.mo_analytic_account_line_id = analytic_line
        for wo, analytic_line in zip(wo_to_link_wc_analytic_line, wc_analytic_lines):
            wo.wc_analytic_account_line_id = analytic_line

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import float_round, groupby


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

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
    _description = 'Product'

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

    def _compute_average_price(self, qty_invoiced, qty_to_invoice, stock_moves):
        self.ensure_one()
        if stock_moves.product_id == self:
            return super()._compute_average_price(qty_invoiced, qty_to_invoice, stock_moves)
        bom = self.env['mrp.bom']._bom_find(self, company_id=stock_moves.company_id.id, bom_type='phantom')[self]
        if not bom:
            return super()._compute_average_price(qty_invoiced, qty_to_invoice, stock_moves)
        value = 0
        dummy, bom_lines = bom.explode(self, 1)
        bom_lines = {line: data for line, data in bom_lines}
        for bom_line, moves_list in groupby(stock_moves.filtered(lambda sm: sm.state != 'cancel'), lambda sm: sm.bom_line_id):
            if bom_line not in bom_lines:
                for move in moves_list:
                    value += move.product_qty * move.product_id._compute_average_price(qty_invoiced * move.product_qty, qty_to_invoice * move.product_qty, move)
                continue
            line_qty = bom_line.product_uom_id._compute_quantity(bom_lines[bom_line]['qty'], bom_line.product_id.uom_id)
            moves = self.env['stock.move'].concat(*moves_list)
            value += line_qty * bom_line.product_id._compute_average_price(qty_invoiced * line_qty, qty_to_invoice * line_qty, moves)
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
                opt.workcenter_id.time_start +
                opt.workcenter_id.time_stop +
                opt.time_cycle * 100 / opt.workcenter_id.time_efficiency)
            total += (duration_expected / 60) * opt.workcenter_id.costs_hour

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

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class StockMove(models.Model):
    _inherit = "stock.move"

    def _filter_anglo_saxon_moves(self, product):
        res = super(StockMove, self)._filter_anglo_saxon_moves(product)
        res += self.filtered(lambda m: m.bom_line_id.bom_id.product_tmpl_id.id == product.product_tmpl_id.id)
        return res

    def _generate_analytic_lines_data(self, unit_amount, amount):
        vals = super()._generate_analytic_lines_data(unit_amount, amount)
        if self.raw_material_production_id.analytic_account_id:
            vals['name'] = _('[Raw] %s', self.product_id.display_name)
            vals['ref'] = self.raw_material_production_id.display_name
            vals['category'] = 'manufacturing_order'
        return vals

    def _get_analytic_account(self):
        account = self.raw_material_production_id.analytic_account_id
        if account:
            return account
        return super()._get_analytic_account()

    def _get_src_account(self, accounts_data):
        if not self.unbuild_id:
            return super()._get_src_account(accounts_data)
        else:
            return self.location_dest_id.valuation_out_account_id.id or accounts_data['stock_input'].id

    def _get_dest_account(self, accounts_data):
        if not self.unbuild_id:
            return super()._get_dest_account(accounts_data)
        else:
            return self.location_id.valuation_in_account_id.id or accounts_data['stock_output'].id

    def _is_returned(self, valued_type):
        if self.unbuild_id:
            return True
        return super()._is_returned(valued_type)

    def _should_force_price_unit(self):
        self.ensure_one()
        return self.picking_type_id.code == 'mrp_operation' or super()._should_force_price_unit()

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
        if not bom.analytic_account_id and values.get('analytic_account_id'):
            res['analytic_account_id'] = values.get('analytic_account_id').id
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
from . import product
from . import stock_move
from . import stock_rule
from . import account_move

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
        <field name="groups_id" eval="[(4, ref('mrp.group_mrp_user'))]"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" type="object" name="action_view_mrp_production"
                    icon="fa-wrench" attrs="{'invisible': [('production_count', '=', 0)]}">
                    <field string="Manufacturing Orders" name="production_count" widget="statinfo"/>
                </button>
                <button class="oe_stat_button" type="object" name="action_view_mrp_bom"
                    icon="fa-flask" attrs="{'invisible': [('bom_count', '=', 0)]}">
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
                <field name="analytic_account_id" groups="analytic.group_analytic_accounting"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mrp_production_views.xml

```xml
<odoo>
    <record id="mrp_production_form_view_inherited" model="ir.ui.view">
        <field name="name">mrp.production.view.inherited</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view" />
        <field name="groups_id" eval="[(4, ref('stock.group_stock_manager'))]"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="before">
                <field name="show_valuation" invisible="1"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button string="Valuation" type="object"
                    name="action_view_stock_valuation_layers"
                    class="oe_stat_button" icon="fa-dollar" groups="base.group_no_one"
                    attrs="{'invisible': [('show_valuation', '=', False)]}" />
                <button class="oe_stat_button" type="object"
                    name="action_view_analytic_account"
                    string="Analytic Account" icon="fa-bar-chart-o"
                    attrs="{'invisible': ['|', ('analytic_account_id', '=', False), ('state', 'in', ['draft', 'cancel'])]}"
                    groups="analytic.group_analytic_accounting"/>
            </xpath>
            <xpath expr="//page[@name='miscellaneous']//field[@name='date_deadline']" position="after">
                <field name="analytic_account_id" groups="analytic.group_analytic_accounting"/>
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
            <field name="groups_id" eval="[(4, ref('mrp.group_mrp_manager'))]"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='standard_price_uom']" position="inside">
                    <field name="cost_method" invisible="1"/>
                    <field name="valuation" invisible="1"/>
                    <button name="button_bom_cost"
                        string="Compute Price from BoM" type="object"
                        attrs="{'invisible': ['|', ('bom_count', '=', 0), '&amp;', ('valuation', '=', 'real_time'), ('cost_method', '=', 'fifo')]}"
                        help="Compute the price of the product using products and operations of related bill of materials, for manufactured products only."
                        class="oe_link oe_read_only pt-0"/>
                </xpath>
            </field>
        </record>

        <record id="product_product_view_form_normal_inherit_extended" model="ir.ui.view">
            <field name="name">product.product.view.form.normal.inherit.extended</field>
            <field name="model">product.product</field>
            <field name="priority">4</field>
            <field name="inherit_id" ref="product.product_normal_form_view"/>
            <field name="groups_id" eval="[(4, ref('mrp.group_mrp_manager'))]"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='standard_price_uom']" position="inside">
                    <field name="cost_method" invisible="1"/>
                    <field name="valuation" invisible="1"/>
                    <button name="button_bom_cost"
                        string="Compute Price from BoM" type="object"
                        attrs="{'invisible': ['|', ('bom_count', '=', 0), '&amp;', ('valuation', '=', 'real_time'), ('cost_method', '=', 'fifo')]}"
                        help="Compute the price of the product using products and operations of related bill of materials, for manufactured products only."
                        class="oe_link oe_read_only pt-0"
                        colspan="2"/>
                </xpath>
            </field>
        </record>

        <record id="product_variant_easy_edit_view_bom_inherit" model="ir.ui.view">
            <field name="name">product.product.product.view.form.easy.bom.inherit</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_variant_easy_edit_view"/>
            <field name="groups_id" eval="[(4, ref('mrp.group_mrp_manager'))]"/>
            <field name="arch" type="xml">
                <data>
                <xpath expr="//field[@name='standard_price']" position="replace">
                    <field name="bom_count" invisible="1"/>
                    <field name="cost_method" invisible="1"/>
                    <field name="valuation" invisible="1"/>
                    <label for="standard_price"/>
                    <div>
                        <field name="standard_price" widget='monetary' options="{'currency_field': 'cost_currency_id'}"/>
                        <button name="button_bom_cost"
                            string="Compute Price from BoM"
                            type="object"
                            attrs="{'invisible': ['|', ('bom_count', '=', 0), '&amp;', ('valuation', '=', 'real_time'), ('cost_method', '=', 'fifo')]}"
                            help="Compute the price of the product using products and operations of related bill of materials, for manufactured products only."
                            class="oe_link oe_read_only pt-0"/>
                    </div>
                </xpath>
                </data>
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

