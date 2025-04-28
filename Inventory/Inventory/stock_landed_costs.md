# Odoo Module: stock_landed_costs

Category: Inventory/Inventory

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
    'name': 'WMS Landed Costs',
    'version': '1.1',
    'summary': 'Landed Costs',
    'description': """
Landed Costs Management
=======================
This module allows you to easily add extra costs on pickings and decide the split of these costs among their stock moves in order to take them into account in your stock valuation.
    """,
    'depends': ['stock_account', 'purchase_stock'],
    'category': 'Inventory/Inventory',
    'sequence': 16,
    'demo': [
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/stock_landed_cost_security.xml',
        'data/stock_landed_cost_data.xml',
        'views/account_move_views.xml',
        'views/product_views.xml',
        'views/stock_landed_cost_views.xml',
        'views/stock_valuation_layer_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\stock_landed_cost_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Stock landed costs - related subtypes for messaging / Chatter -->
        <record id="mt_stock_landed_cost_open" model="mail.message.subtype">
            <field name="name">Done</field>
            <field name="res_model">stock.landed.cost</field>
            <field name="description">Landed cost validated</field>
        </record>

        <record id="seq_stock_landed_costs" model="ir.sequence">
            <field name="name">Stock Landed Costs</field>
            <field name="code">stock.landed.cost</field>
            <field name="prefix">LC/%(year)s/</field>
            <field name="padding">4</field>
            <field name="company_id" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    landed_costs_ids = fields.One2many('stock.landed.cost', 'vendor_bill_id', string='Landed Costs')
    landed_costs_visible = fields.Boolean(compute='_compute_landed_costs_visible')

    @api.depends('line_ids', 'line_ids.is_landed_costs_line')
    def _compute_landed_costs_visible(self):
        for account_move in self:
            if account_move.landed_costs_ids:
                account_move.landed_costs_visible = False
            else:
                account_move.landed_costs_visible = any(line.is_landed_costs_line for line in account_move.line_ids)

    def button_create_landed_costs(self):
        """Create a `stock.landed.cost` record associated to the account move of `self`, each
        `stock.landed.costs` lines mirroring the current `account.move.line` of self.
        """
        self.ensure_one()
        landed_costs_lines = self.line_ids.filtered(lambda line: line.is_landed_costs_line)

        landed_costs = self.env['stock.landed.cost'].create({
            'vendor_bill_id': self.id,
            'cost_lines': [(0, 0, {
                'product_id': l.product_id.id,
                'name': l.product_id.name,
                'account_id': l.product_id.product_tmpl_id.get_product_accounts()['stock_input'].id,
                'price_unit': l.currency_id._convert(l.price_subtotal, l.company_currency_id, l.company_id, l.move_id.date),
                'split_method': l.product_id.split_method_landed_cost or 'equal',
            }) for l in landed_costs_lines],
        })
        action = self.env["ir.actions.actions"]._for_xml_id("stock_landed_costs.action_stock_landed_cost")
        return dict(action, view_mode='form', res_id=landed_costs.id, views=[(False, 'form')])

    def action_view_landed_costs(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("stock_landed_costs.action_stock_landed_cost")
        domain = [('id', 'in', self.landed_costs_ids.ids)]
        context = dict(self.env.context, default_vendor_bill_id=self.id)
        views = [(self.env.ref('stock_landed_costs.view_stock_landed_cost_tree2').id, 'tree'), (False, 'form'), (False, 'kanban')]
        return dict(action, domain=domain, context=context, views=views)


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    product_type = fields.Selection(related='product_id.type', readonly=True)
    is_landed_costs_line = fields.Boolean()

    @api.onchange('is_landed_costs_line')
    def _onchange_is_landed_costs_line(self):
        """Mark an invoice line as a landed cost line and adapt `self.account_id`. The default
        value can be set according to `self.product_id.landed_cost_ok`."""
        if self.product_id:
            accounts = self.product_id.product_tmpl_id._get_product_accounts()
            aml_account = accounts['expense']
            if self.product_type != 'service':
                self.is_landed_costs_line = False
            elif self.is_landed_costs_line:
                aml_account = (self.move_id.company_id.anglo_saxon_accounting and accounts['stock_input']) or aml_account
            self.account_id = aml_account

    @api.onchange('product_id')
    def _onchange_is_landed_costs_line_product(self):
        if self.product_id.landed_cost_ok:
            self.is_landed_costs_line = True
        else:
            self.is_landed_costs_line = False

    def _can_use_stock_accounts(self):
        return super()._can_use_stock_accounts() or (self.product_id.type == 'service' and self.product_id.landed_cost_ok)

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.addons.stock_landed_costs.models.stock_landed_cost import SPLIT_METHOD
from odoo.exceptions import UserError
from odoo import _


class ProductTemplate(models.Model):
    _inherit = "product.template"

    landed_cost_ok = fields.Boolean('Is a Landed Cost', help='Indicates whether the product is a landed cost.')
    split_method_landed_cost = fields.Selection(SPLIT_METHOD, string="Default Split Method",
                                                help="Default Split Method when used for Landed Cost")

    def write(self, vals):
        for product in self:
            if (('type' in vals and vals['type'] != 'service') or ('landed_cost_ok' in vals and not vals['landed_cost_ok'])) and product.type == 'service' and product.landed_cost_ok:
                if self.env['account.move.line'].search_count([('product_id', 'in', product.product_variant_ids.ids), ('is_landed_costs_line', '=', True)]):
                    raise UserError(_("You cannot change the product type or disable landed cost option because the product is used in an account move line."))
                vals['landed_cost_ok'] = False

        return super().write(vals)

```

## File: models\purchase.py

```python
from odoo import api,models

class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _prepare_account_move_line(self, move=False):
        res = super()._prepare_account_move_line(move)
        res.update({'is_landed_costs_line': self.product_id.landed_cost_ok})
        return res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    lc_journal_id = fields.Many2one('account.journal')


```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    lc_journal_id = fields.Many2one('account.journal', string='Default Journal', related='company_id.lc_journal_id', readonly=False)


```

## File: models\stock_landed_cost.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError
from odoo.tools.float_utils import float_is_zero


SPLIT_METHOD = [
    ('equal', 'Equal'),
    ('by_quantity', 'By Quantity'),
    ('by_current_cost_price', 'By Current Cost'),
    ('by_weight', 'By Weight'),
    ('by_volume', 'By Volume'),
]


class StockLandedCost(models.Model):
    _name = 'stock.landed.cost'
    _description = 'Stock Landed Cost'
    _inherit = ['mail.thread', 'mail.activity.mixin']

    def _default_account_journal_id(self):
        """Take the journal configured in the company, else fallback on the stock journal."""
        lc_journal = self.env['account.journal']
        if self.env.company.lc_journal_id:
            lc_journal = self.env.company.lc_journal_id
        else:
            lc_journal = self.env['ir.property']._get("property_stock_journal", "product.category")
        return lc_journal

    name = fields.Char(
        'Name', default=lambda self: _('New'),
        copy=False, readonly=True, tracking=True)
    date = fields.Date(
        'Date', default=fields.Date.context_today,
        copy=False, required=True, states={'done': [('readonly', True)]}, tracking=True)
    target_model = fields.Selection(
        [('picking', 'Transfers')], string="Apply On",
        required=True, default='picking',
        copy=False, states={'done': [('readonly', True)]})
    picking_ids = fields.Many2many(
        'stock.picking', string='Transfers',
        copy=False, states={'done': [('readonly', True)]})
    allowed_picking_ids = fields.Many2many('stock.picking', compute='_compute_allowed_picking_ids')
    cost_lines = fields.One2many(
        'stock.landed.cost.lines', 'cost_id', 'Cost Lines',
        copy=True, states={'done': [('readonly', True)]})
    valuation_adjustment_lines = fields.One2many(
        'stock.valuation.adjustment.lines', 'cost_id', 'Valuation Adjustments',
        states={'done': [('readonly', True)]})
    description = fields.Text(
        'Item Description', states={'done': [('readonly', True)]})
    amount_total = fields.Monetary(
        'Total', compute='_compute_total_amount',
        store=True, tracking=True)
    state = fields.Selection([
        ('draft', 'Draft'),
        ('done', 'Posted'),
        ('cancel', 'Cancelled')], 'State', default='draft',
        copy=False, readonly=True, tracking=True)
    account_move_id = fields.Many2one(
        'account.move', 'Journal Entry',
        copy=False, readonly=True)
    account_journal_id = fields.Many2one(
        'account.journal', 'Account Journal',
        required=True, states={'done': [('readonly', True)]}, default=lambda self: self._default_account_journal_id())
    company_id = fields.Many2one('res.company', string="Company",
        related='account_journal_id.company_id')
    stock_valuation_layer_ids = fields.One2many('stock.valuation.layer', 'stock_landed_cost_id')
    vendor_bill_id = fields.Many2one(
        'account.move', 'Vendor Bill', copy=False, domain=[('move_type', '=', 'in_invoice')])
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')

    @api.depends('cost_lines.price_unit')
    def _compute_total_amount(self):
        for cost in self:
            cost.amount_total = sum(line.price_unit for line in cost.cost_lines)

    @api.depends('company_id')
    def _compute_allowed_picking_ids(self):
        # Backport of f329de26: allowed_picking_ids is useless, view_stock_landed_cost_form no longer uses it,
        # the field and its compute are kept since this is a stable version. Still, this compute has been made
        # more resilient to MemoryErrors.
        valued_picking_ids_per_company = defaultdict(list)
        if self.company_id:
            self.env.cr.execute("""SELECT sm.picking_id, sm.company_id
                                     FROM stock_move AS sm
                               INNER JOIN stock_valuation_layer AS svl ON svl.stock_move_id = sm.id
                                    WHERE sm.picking_id IS NOT NULL AND sm.company_id IN %s
                                 GROUP BY sm.picking_id, sm.company_id""", [tuple(self.company_id.ids)])
            for res in self.env.cr.fetchall():
                valued_picking_ids_per_company[res[1]].append(res[0])
        for cost in self:
            n = 5000
            cost.allowed_picking_ids = valued_picking_ids_per_company[cost.company_id.id][:n]
            for ids_chunk in tools.split_every(n, valued_picking_ids_per_company[cost.company_id.id][n:]):
                cost.allowed_picking_ids = [(4, id_) for id_ in ids_chunk]

    @api.onchange('target_model')
    def _onchange_target_model(self):
        if self.target_model != 'picking':
            self.picking_ids = False

    @api.model
    def create(self, vals):
        if vals.get('name', _('New')) == _('New'):
            vals['name'] = self.env['ir.sequence'].next_by_code('stock.landed.cost')
        return super().create(vals)

    def unlink(self):
        self.button_cancel()
        return super().unlink()

    def _track_subtype(self, init_values):
        if 'state' in init_values and self.state == 'done':
            return self.env.ref('stock_landed_costs.mt_stock_landed_cost_open')
        return super()._track_subtype(init_values)

    def button_cancel(self):
        if any(cost.state == 'done' for cost in self):
            raise UserError(
                _('Validated landed costs cannot be cancelled, but you could create negative landed costs to reverse them'))
        return self.write({'state': 'cancel'})

    def button_validate(self):
        self._check_can_validate()
        cost_without_adjusment_lines = self.filtered(lambda c: not c.valuation_adjustment_lines)
        if cost_without_adjusment_lines:
            cost_without_adjusment_lines.compute_landed_cost()
        if not self._check_sum():
            raise UserError(_('Cost and adjustments lines do not match. You should maybe recompute the landed costs.'))

        for cost in self:
            cost = cost.with_company(cost.company_id)
            move = self.env['account.move']
            move_vals = {
                'journal_id': cost.account_journal_id.id,
                'date': cost.date,
                'ref': cost.name,
                'line_ids': [],
                'move_type': 'entry',
            }
            valuation_layer_ids = []
            cost_to_add_byproduct = defaultdict(lambda: 0.0)
            for line in cost.valuation_adjustment_lines.filtered(lambda line: line.move_id):
                remaining_qty = sum(line.move_id.stock_valuation_layer_ids.mapped('remaining_qty'))
                linked_layer = line.move_id.stock_valuation_layer_ids[:1]

                # Prorate the value at what's still in stock
                cost_to_add = (remaining_qty / line.move_id.product_qty) * line.additional_landed_cost
                if not cost.company_id.currency_id.is_zero(cost_to_add):
                    valuation_layer = self.env['stock.valuation.layer'].create({
                        'value': cost_to_add,
                        'unit_cost': 0,
                        'quantity': 0,
                        'remaining_qty': 0,
                        'stock_valuation_layer_id': linked_layer.id,
                        'description': cost.name,
                        'stock_move_id': line.move_id.id,
                        'product_id': line.move_id.product_id.id,
                        'stock_landed_cost_id': cost.id,
                        'company_id': cost.company_id.id,
                    })
                    linked_layer.remaining_value += cost_to_add
                    valuation_layer_ids.append(valuation_layer.id)
                # Update the AVCO
                product = line.move_id.product_id
                if product.cost_method == 'average':
                    cost_to_add_byproduct[product] += cost_to_add
                # Products with manual inventory valuation are ignored because they do not need to create journal entries.
                if product.valuation != "real_time":
                    continue
                # `remaining_qty` is negative if the move is out and delivered proudcts that were not
                # in stock.
                qty_out = 0
                if line.move_id._is_in():
                    qty_out = line.move_id.product_qty - remaining_qty
                elif line.move_id._is_out():
                    qty_out = line.move_id.product_qty
                move_vals['line_ids'] += line._create_accounting_entries(move, qty_out)

            # batch standard price computation avoid recompute quantity_svl at each iteration
            products = self.env['product.product'].browse(p.id for p in cost_to_add_byproduct.keys())
            for product in products:  # iterate on recordset to prefetch efficiently quantity_svl
                if not float_is_zero(product.quantity_svl, precision_rounding=product.uom_id.rounding):
                    product.with_company(cost.company_id).sudo().with_context(disable_auto_svl=True).standard_price += cost_to_add_byproduct[product] / product.quantity_svl

            move_vals['stock_valuation_layer_ids'] = [(6, None, valuation_layer_ids)]
            # We will only create the accounting entry when there are defined lines (the lines will be those linked to products of real_time valuation category).
            cost_vals = {'state': 'done'}
            if move_vals.get("line_ids"):
                move = move.create(move_vals)
                cost_vals.update({'account_move_id': move.id})
            cost.write(cost_vals)
            if cost.account_move_id:
                move._post()

            if cost.vendor_bill_id and cost.vendor_bill_id.state == 'posted' and cost.company_id.anglo_saxon_accounting:
                all_amls = cost.vendor_bill_id.line_ids | cost.account_move_id.line_ids
                for product in cost.cost_lines.product_id:
                    accounts = product.product_tmpl_id.get_product_accounts()
                    input_account = accounts['stock_input']
                    all_amls.filtered(lambda aml: aml.account_id == input_account and not aml.reconciled).reconcile()

        return True

    def get_valuation_lines(self):
        self.ensure_one()
        lines = []

        for move in self._get_targeted_move_ids():
            # it doesn't make sense to make a landed cost for a product that isn't set as being valuated in real time at real cost
            if move.product_id.cost_method not in ('fifo', 'average') or move.state == 'cancel' or not move.product_qty:
                continue
            vals = {
                'product_id': move.product_id.id,
                'move_id': move.id,
                'quantity': move.product_qty,
                'former_cost': sum(move.stock_valuation_layer_ids.mapped('value')),
                'weight': move.product_id.weight * move.product_qty,
                'volume': move.product_id.volume * move.product_qty
            }
            lines.append(vals)

        if not lines:
            target_model_descriptions = dict(self._fields['target_model']._description_selection(self.env))
            raise UserError(_("You cannot apply landed costs on the chosen %s(s). Landed costs can only be applied for products with FIFO or average costing method.", target_model_descriptions[self.target_model]))
        return lines

    def compute_landed_cost(self):
        AdjustementLines = self.env['stock.valuation.adjustment.lines']
        AdjustementLines.search([('cost_id', 'in', self.ids)]).unlink()

        towrite_dict = {}
        for cost in self.filtered(lambda cost: cost._get_targeted_move_ids()):
            rounding = cost.currency_id.rounding
            total_qty = 0.0
            total_cost = 0.0
            total_weight = 0.0
            total_volume = 0.0
            total_line = 0.0
            all_val_line_values = cost.get_valuation_lines()
            for val_line_values in all_val_line_values:
                for cost_line in cost.cost_lines:
                    val_line_values.update({'cost_id': cost.id, 'cost_line_id': cost_line.id})
                    self.env['stock.valuation.adjustment.lines'].create(val_line_values)
                total_qty += val_line_values.get('quantity', 0.0)
                total_weight += val_line_values.get('weight', 0.0)
                total_volume += val_line_values.get('volume', 0.0)

                former_cost = val_line_values.get('former_cost', 0.0)
                # round this because former_cost on the valuation lines is also rounded
                total_cost += cost.currency_id.round(former_cost)

                total_line += 1

            for line in cost.cost_lines:
                value_split = 0.0
                for valuation in cost.valuation_adjustment_lines:
                    value = 0.0
                    if valuation.cost_line_id and valuation.cost_line_id.id == line.id:
                        if line.split_method == 'by_quantity' and total_qty:
                            per_unit = (line.price_unit / total_qty)
                            value = valuation.quantity * per_unit
                        elif line.split_method == 'by_weight' and total_weight:
                            per_unit = (line.price_unit / total_weight)
                            value = valuation.weight * per_unit
                        elif line.split_method == 'by_volume' and total_volume:
                            per_unit = (line.price_unit / total_volume)
                            value = valuation.volume * per_unit
                        elif line.split_method == 'equal':
                            value = (line.price_unit / total_line)
                        elif line.split_method == 'by_current_cost_price' and total_cost:
                            per_unit = (line.price_unit / total_cost)
                            value = valuation.former_cost * per_unit
                        else:
                            value = (line.price_unit / total_line)

                        if rounding:
                            value = tools.float_round(value, precision_rounding=rounding, rounding_method='UP')
                            fnc = min if line.price_unit > 0 else max
                            value = fnc(value, line.price_unit - value_split)
                            value_split += value

                        if valuation.id not in towrite_dict:
                            towrite_dict[valuation.id] = value
                        else:
                            towrite_dict[valuation.id] += value
        for key, value in towrite_dict.items():
            AdjustementLines.browse(key).write({'additional_landed_cost': value})
        return True

    def action_view_stock_valuation_layers(self):
        self.ensure_one()
        domain = [('id', 'in', self.stock_valuation_layer_ids.ids)]
        action = self.env["ir.actions.actions"]._for_xml_id("stock_account.stock_valuation_layer_action")
        return dict(action, domain=domain)

    def _get_targeted_move_ids(self):
        return self.picking_ids.move_lines

    def _check_can_validate(self):
        if any(cost.state != 'draft' for cost in self):
            raise UserError(_('Only draft landed costs can be validated'))
        for cost in self:
            if not cost._get_targeted_move_ids():
                target_model_descriptions = dict(self._fields['target_model']._description_selection(self.env))
                raise UserError(_('Please define %s on which those additional costs should apply.', target_model_descriptions[cost.target_model]))

    def _check_sum(self):
        """ Check if each cost line its valuation lines sum to the correct amount
        and if the overall total amount is correct also """
        prec_digits = self.env.company.currency_id.decimal_places
        for landed_cost in self:
            total_amount = sum(landed_cost.valuation_adjustment_lines.mapped('additional_landed_cost'))
            if not tools.float_is_zero(total_amount - landed_cost.amount_total, precision_digits=prec_digits):
                return False

            val_to_cost_lines = defaultdict(lambda: 0.0)
            for val_line in landed_cost.valuation_adjustment_lines:
                val_to_cost_lines[val_line.cost_line_id] += val_line.additional_landed_cost
            if any(not tools.float_is_zero(cost_line.price_unit - val_amount, precision_digits=prec_digits)
                   for cost_line, val_amount in val_to_cost_lines.items()):
                return False
        return True


class StockLandedCostLine(models.Model):
    _name = 'stock.landed.cost.lines'
    _description = 'Stock Landed Cost Line'

    name = fields.Char('Description')
    cost_id = fields.Many2one(
        'stock.landed.cost', 'Landed Cost',
        required=True, ondelete='cascade')
    product_id = fields.Many2one('product.product', 'Product', required=True)
    price_unit = fields.Monetary('Cost', required=True)
    split_method = fields.Selection(
        SPLIT_METHOD,
        string='Split Method',
        required=True,
        help="Equal : Cost will be equally divided.\n"
             "By Quantity : Cost will be divided according to product's quantity.\n"
             "By Current cost : Cost will be divided according to product's current cost.\n"
             "By Weight : Cost will be divided depending on its weight.\n"
             "By Volume : Cost will be divided depending on its volume.")
    account_id = fields.Many2one('account.account', 'Account', domain=[('deprecated', '=', False)])
    currency_id = fields.Many2one('res.currency', related='cost_id.currency_id')

    @api.onchange('product_id')
    def onchange_product_id(self):
        self.name = self.product_id.name or ''
        self.split_method = self.product_id.product_tmpl_id.split_method_landed_cost or self.split_method or 'equal'
        self.price_unit = self.product_id.standard_price or 0.0
        accounts_data = self.product_id.product_tmpl_id.get_product_accounts()
        self.account_id = accounts_data['stock_input']


class AdjustmentLines(models.Model):
    _name = 'stock.valuation.adjustment.lines'
    _description = 'Valuation Adjustment Lines'

    name = fields.Char(
        'Description', compute='_compute_name', store=True)
    cost_id = fields.Many2one(
        'stock.landed.cost', 'Landed Cost',
        ondelete='cascade', required=True)
    cost_line_id = fields.Many2one(
        'stock.landed.cost.lines', 'Cost Line', readonly=True)
    move_id = fields.Many2one('stock.move', 'Stock Move', readonly=True)
    product_id = fields.Many2one('product.product', 'Product', required=True)
    quantity = fields.Float(
        'Quantity', default=1.0,
        digits=0, required=True)
    weight = fields.Float(
        'Weight', default=1.0,
        digits='Stock Weight')
    volume = fields.Float(
        'Volume', default=1.0, digits='Volume')
    former_cost = fields.Monetary(
        'Original Value')
    additional_landed_cost = fields.Monetary(
        'Additional Landed Cost')
    final_cost = fields.Monetary(
        'New Value', compute='_compute_final_cost',
        store=True)
    currency_id = fields.Many2one('res.currency', related='cost_id.company_id.currency_id')

    @api.depends('cost_line_id.name', 'product_id.code', 'product_id.name')
    def _compute_name(self):
        for line in self:
            name = '%s - ' % (line.cost_line_id.name if line.cost_line_id else '')
            line.name = name + (line.product_id.code or line.product_id.name or '')

    @api.depends('former_cost', 'additional_landed_cost')
    def _compute_final_cost(self):
        for line in self:
            line.final_cost = line.former_cost + line.additional_landed_cost

    def _create_accounting_entries(self, move, qty_out):
        # TDE CLEANME: product chosen for computation ?
        cost_product = self.cost_line_id.product_id
        if not cost_product:
            return False
        accounts = self.product_id.product_tmpl_id.get_product_accounts()
        debit_account_id = accounts.get('stock_valuation') and accounts['stock_valuation'].id or False
        # If the stock move is dropshipped move we need to get the cost account instead the stock valuation account
        if self.move_id._is_dropshipped():
            debit_account_id = accounts.get('expense') and accounts['expense'].id or False
        already_out_account_id = accounts['stock_output'].id
        credit_account_id = self.cost_line_id.account_id.id or cost_product.categ_id.property_stock_account_input_categ_id.id

        if not credit_account_id:
            raise UserError(_('Please configure Stock Expense Account for product: %s.') % (cost_product.name))

        return self._create_account_move_line(move, credit_account_id, debit_account_id, qty_out, already_out_account_id)

    def _create_account_move_line(self, move, credit_account_id, debit_account_id, qty_out, already_out_account_id):
        """
        Generate the account.move.line values to track the landed cost.
        Afterwards, for the goods that are already out of stock, we should create the out moves
        """
        AccountMoveLine = []

        base_line = {
            'name': self.name,
            'product_id': self.product_id.id,
            'quantity': 0,
        }
        debit_line = dict(base_line, account_id=debit_account_id)
        credit_line = dict(base_line, account_id=credit_account_id)
        diff = self.additional_landed_cost
        if diff > 0:
            debit_line['debit'] = diff
            credit_line['credit'] = diff
        else:
            # negative cost, reverse the entry
            debit_line['credit'] = -diff
            credit_line['debit'] = -diff
        AccountMoveLine.append([0, 0, debit_line])
        AccountMoveLine.append([0, 0, credit_line])

        # Create account move lines for quants already out of stock
        if qty_out > 0:
            debit_line = dict(base_line,
                              name=(self.name + ": " + str(qty_out) + _(' already out')),
                              quantity=0,
                              account_id=already_out_account_id)
            credit_line = dict(base_line,
                               name=(self.name + ": " + str(qty_out) + _(' already out')),
                               quantity=0,
                               account_id=debit_account_id)
            diff = diff * qty_out / self.quantity
            if diff > 0:
                debit_line['debit'] = diff
                credit_line['credit'] = diff
            else:
                # negative cost, reverse the entry
                debit_line['credit'] = -diff
                credit_line['debit'] = -diff
            AccountMoveLine.append([0, 0, debit_line])
            AccountMoveLine.append([0, 0, credit_line])

            if self.env.company.anglo_saxon_accounting:
                expense_account_id = self.product_id.product_tmpl_id.get_product_accounts()['expense'].id
                debit_line = dict(base_line,
                                  name=(self.name + ": " + str(qty_out) + _(' already out')),
                                  quantity=0,
                                  account_id=expense_account_id)
                credit_line = dict(base_line,
                                   name=(self.name + ": " + str(qty_out) + _(' already out')),
                                   quantity=0,
                                   account_id=already_out_account_id)

                if diff > 0:
                    debit_line['debit'] = diff
                    credit_line['credit'] = diff
                else:
                    # negative cost, reverse the entry
                    debit_line['credit'] = -diff
                    credit_line['debit'] = -diff
                AccountMoveLine.append([0, 0, debit_line])
                AccountMoveLine.append([0, 0, credit_line])

        return AccountMoveLine

```

## File: models\stock_valuation_layer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockValuationLayer(models.Model):
    """Stock Valuation Layer"""

    _inherit = 'stock.valuation.layer'

    stock_landed_cost_id = fields.Many2one('stock.landed.cost', 'Landed Cost')


```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product
from . import purchase
from . import res_company
from . import res_config_settings
from . import stock_landed_cost
from . import account_move
from . import stock_valuation_layer


```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
"access_stock_landed_cost","stock.landed.cost","model_stock_landed_cost","stock.group_stock_manager",1,1,1,1
"access_stock_landed_cost_lines","stock.landed.cost.lines","model_stock_landed_cost_lines","stock.group_stock_manager",1,1,1,1
"access_stock_valuation_adjustment_lines","stock.valuation.adjustment.lines","model_stock_valuation_adjustment_lines","stock.group_stock_manager",1,1,1,1

```

## File: security\stock_landed_cost_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record model="ir.rule" id="stock_landed_cost_rule">
        <field name="name">stock_landed_cost multi-company</field>
        <field name="model_id" search="[('model','=','stock.landed.cost')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_view_move_form_inherited" model="ir.ui.view">
        <field name="name">account.view.move.form.inherited</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="groups_id" eval="[(4,ref('stock.group_stock_manager'))]"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="landed_costs_ids" invisible="1"/>
                <button string="Landed Costs" type="object"
                    name="action_view_landed_costs"
                    class="oe_stat_button" icon="fa-plus-square" groups="stock.group_stock_manager"
                    attrs="{'invisible': [('landed_costs_ids', '=', [])]}" />
            </xpath>

            <field name="state" position="before">
                <field name="landed_costs_visible" invisible="1"/>
                <button name="button_create_landed_costs" class="oe_highlight" string="Create Landed Costs" type="object" groups="account.group_account_invoice" attrs="{'invisible': [('landed_costs_visible', '!=', True)]}"/>
            </field>

            <xpath expr="//field[@name='invoice_line_ids']/tree/field[@name='name']" position="after">
                <field name="product_type" invisible="1"/>
                <field name="is_landed_costs_line" string="Landed Costs" attrs="{'readonly': [('product_type', '!=', 'service')], 'column_invisible': [('parent.move_type', '!=', 'in_invoice')]}" optional="show"/>
            </xpath>

            <xpath expr="//field[@name='line_ids']/tree/field[@name='name']" position="after">
                <field name="product_type" invisible="1"/>
                <field name="is_landed_costs_line" invisible="1"/>
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
        <record id="view_product_landed_cost_form" model="ir.ui.view">
            <field name="name">product.template.landed.cost.form</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="account.product_template_form_view"/>
            <field name="arch" type="xml">
                <group name="bill" position="after">
                    <div attrs="{'invisible':[('type', '!=', 'service')]}">
                        <group string="Landed Costs" name="landedcosts">
                            <field name="landed_cost_ok"/>
                            <field name="split_method_landed_cost" attrs="{'invisible': [('landed_cost_ok', '=', False)]}" class="col-4"/>
                        </group>
                    </div>
                </group>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.stock.landed.costs</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="stock.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div name="landed_cost_info" position="replace">
                <div class="content-group" attrs="{'invisible': [('module_stock_landed_costs', '=', False)]}">
                    <div class="mt16">
                        <label for="lc_journal_id" string="Default Journal"/>
                        <field name="lc_journal_id"/>
                    </div>
                </div>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\stock_landed_cost_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <!-- STOCK.LANDED.COST -->
        <record id='view_stock_landed_cost_form' model='ir.ui.view'>
            <field name="name">stock.landed.cost.form</field>
            <field name="model">stock.landed.cost</field>
            <field name="arch" type="xml">
                <form string="Landed Costs">
                    <field name="stock_valuation_layer_ids" invisible="1"/>
                    <header>
                        <button name="button_validate" string="Validate" states="draft" class="oe_highlight" type="object"/>
                        <button name="button_cancel" string="Cancel" states="draft" type="object"/>
                        <field name="state" widget="statusbar" statusbar_visible="draft,done"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button string="Valuation" type="object"
                                name="action_view_stock_valuation_layers"
                                class="oe_stat_button" icon="fa-dollar" groups="stock.group_stock_manager"
                                attrs="{'invisible': ['|' , ('state', 'not in', ['done']), ('stock_valuation_layer_ids', '=', [])]}"/>
                        </div>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only" string="Landed Cost"/>
                            <h1>
                                <field name="name" placeholder="Landed Cost Name"/>
                            </h1>
                        </div>
                        <group>
                            <group>
                                <field name="date"/>
                                <field name="target_model" widget="radio" invisible="1"/>
                                <field name="picking_ids" widget="many2many_tags"
                                    options="{'no_create_edit': True}" attrs="{'invisible': [('target_model', '!=', 'picking')]}"
                                    domain="[('company_id', '=', company_id), ('move_lines.stock_valuation_layer_ids', '!=', False)]"/>
                            </group>
                            <group>
                                <label for="account_journal_id" string="Journal"/>
                                <field name="account_journal_id" nolabel="1"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                                <field name="account_move_id" attrs="{'invisible': [('account_move_id', '=', False)]}"/>
                                <field name="vendor_bill_id"/>
                            </group>
                        </group>
                        <notebook>
                            <page string="Additional Costs" name="additional_costs">
                                <field name="cost_lines">
                                    <form string="Cost Lines">
                                        <group>
                                            <group>
                                                <field name="product_id"
                                                    domain="[('landed_cost_ok', '=', True)]"
                                                    context="{'default_landed_cost_ok': True}"/>
                                                <field name="price_unit"/>
                                                <field name="currency_id" invisible="1"/>
                                            </group>
                                            <group>
                                                <field name="split_method"/>
                                                <field name="account_id" options="{'no_create': True}"/>
                                            </group>
                                        </group>
                                        <label for="name"/>
                                        <field name="name"/>
                                    </form>
                                    <tree string="Cost Lines" editable="bottom">
                                        <field name="product_id"
                                            domain="[('landed_cost_ok', '=', True)]"
                                            context="{'default_landed_cost_ok': True, 'default_type': 'service'}"/>
                                        <field name="name"/>
                                        <field name="account_id" options="{'no_create': True}"/>
                                        <field name="split_method"/>
                                        <field name="price_unit"/>
                                        <field name="currency_id" invisible="1"/>
                                    </tree>
                                </field>
                                <group class="oe_subtotal_footer oe_right">
                                    <field name="currency_id" invisible="1"/>
                                    <field name="amount_total"/>
                                    <button colspan="2" name="compute_landed_cost" string="Compute" type="object" class="oe_right btn-secondary" states='draft'/>
                                </group>
                            </page>
                            <page string="Valuation Adjustments" name="valuation_adjustments">
                                <field name="valuation_adjustment_lines">
                                    <form string="Valuation Adjustments">
                                        <group>
                                            <group>
                                                <field name="product_id"/>
                                                <field name="quantity"/>
                                            </group>
                                            <group>
                                                <field name="currency_id" invisible="1"/>
                                                <field name="former_cost"/>
                                                <field name="additional_landed_cost"/>
                                            </group>
                                        </group>
                                    </form>
                                    <tree string="Valuation Adjustments" editable="bottom" create="0">
                                        <field name="cost_line_id" readonly="1"/>
                                        <field name="product_id" readonly="1"/>
                                        <field name="weight" readonly="1" optional="hide"/>
                                        <field name="volume" readonly="1" optional="hide"/>
                                        <field name="quantity" readonly="1"/>
                                        <field name="currency_id" invisible="1"/>
                                        <field name="former_cost" readonly="1"/>
                                        <field name="final_cost" readonly="1"/>
                                        <field name="additional_landed_cost"/>
                                    </tree>
                                </field>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids"/>
                        <field name="activity_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id='view_stock_landed_cost_tree' model='ir.ui.view'>
            <field name="name">stock.landed.cost.tree</field>
            <field name="model">stock.landed.cost</field>
            <field name="arch" type="xml">
                <tree string="Landed Costs">
                    <field name="name" decoration-bf="1"/>
                    <field name="date"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="state" widget="badge" decoration-success="state == 'done'" decoration-info="state == 'draft'"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                </tree>
            </field>
        </record>

        <record id='view_stock_landed_cost_tree2' model='ir.ui.view'>
            <field name="name">stock.landed.cost.tree</field>
            <field name="model">stock.landed.cost</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <tree string="Landed Costs">
                    <field name="name"/>
                    <field name="date"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="amount_total" widget="monetary"/>
                    <field name="state"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id='stock_landed_cost_view_kanban' model='ir.ui.view'>
            <field name="name">stock.landed.cost.kanban</field>
            <field name="model">stock.landed.cost</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="name"/>
                    <field name="date"/>
                    <field name="state"/>
                    <field name="account_journal_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div class="row mb4">
                                    <strong class="col-6">
                                        <span t-esc="record.name.value"/>
                                    </strong>
                                    <div class="col-6">
                                        <span class="float-right badge badge-secondary">
                                            <field name="state"/>
                                        </span>
                                    </div>
                                    <div class="col-6">
                                        <i class="fa fa-clock-o" title="Date" role="img" aria-label="Date"/><span t-esc="record.date.value"/>
                                    </div>
                                    <div class="col-6 text-right">
                                        <field name="account_journal_id"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_stock_landed_cost_search" model="ir.ui.view">
            <field name="name">stock.landed.cost.search</field>
            <field name="model">stock.landed.cost</field>
            <field name="arch" type="xml">
                <search string="Landed Costs">
                    <field name="name" string="Name"/>
                    <field name="picking_ids" string="Picking"/>
                    <separator/>
                    <filter string="Draft" name="draft" domain="[('state', '=', 'draft')]"/>
                    <filter string="Done" name="done" domain="[('state', '=', 'done')]"/>
                    <separator/>
                    <filter string="Date" name="date" date="date"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Status" name="status" context="{'group_by': 'state'}"/>
                        <filter string="Date" name="group_by_month" context="{'group_by': 'date'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id='action_stock_landed_cost' model='ir.actions.act_window'>
            <field name="name">Landed Costs</field>
            <field name="res_model">stock.landed.cost</field>
            <field name="view_mode">tree,form,kanban</field>
            <field name="context">{}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new landed cost
                </p>
            </field>
        </record>

        <menuitem action="action_stock_landed_cost" name="Landed Costs" parent="stock.menu_stock_warehouse_mgmt" id="menu_stock_landed_cost" sequence="115"/>
    </data>
</odoo>

```

## File: views\stock_valuation_layer_views.xml

```xml
<odoo>
    <record id="stock_valuation_layer_form_inherited" model="ir.ui.view">
        <field name="name">stock.valuation.layer.form.inherited</field>
        <field name="model">stock.valuation.layer</field>
        <field name="inherit_id" ref="stock_account.stock_valuation_layer_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='stock_move_id']" position="after">
                <field name="stock_landed_cost_id" attrs="{'invisible': [('stock_landed_cost_id', '=', False)]}" />
            </xpath>
        </field>
    </record>
</odoo>


```

