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
    'website': 'https://www.odoo.com/page/manufacturing',
    'depends': ['mrp', 'stock_account'],
    "init_xml" : [],
    "demo_xml" : [],
    "data": [
        'security/ir.model.access.csv',
        "views/product_views.xml",
        "views/mrp_production_views.xml",
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

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
        for product, qty in res.items():
            bom_kit = self.env['mrp.bom']._bom_find(product=product, company_id=self.company_id[:1].id, bom_type='phantom')
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

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, fields, models
from odoo.tools import float_is_zero


class MrpProductionWorkcenterLineTime(models.Model):
    _inherit = 'mrp.workcenter.productivity'

    cost_already_recorded = fields.Boolean('Cost Recorded', help="Technical field automatically checked when a ongoing production posts journal entries for its costs. This way, we can record one production's cost multiple times and only consider new entries in the work centers time lines.")


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    extra_cost = fields.Float(copy=False, help='Extra cost per produced unit')
    show_valuation = fields.Boolean(compute='_compute_show_valuation')

    def _compute_show_valuation(self):
        for order in self:
            order.show_valuation = any(m.state == 'done' for m in order.move_finished_ids)

    def _cal_price(self, consumed_moves):
        """Set a price unit on the finished move according to `consumed_moves`.
        """
        super(MrpProduction, self)._cal_price(consumed_moves)
        work_center_cost = 0
        finished_move = self.move_finished_ids.filtered(lambda x: x.product_id == self.product_id and x.state not in ('done', 'cancel') and x.quantity_done > 0)
        if finished_move:
            finished_move.ensure_one()
            for work_order in self.workorder_ids:
                time_lines = work_order.time_ids.filtered(lambda x: x.date_end and not x.cost_already_recorded)
                duration = sum(time_lines.mapped('duration'))
                time_lines.write({'cost_already_recorded': True})
                work_center_cost += (duration / 60.0) * work_order.workcenter_id.costs_hour
            if finished_move.product_id.cost_method in ('fifo', 'average'):
                qty_done = finished_move.product_uom._compute_quantity(finished_move.quantity_done, finished_move.product_id.uom_id)
                extra_cost = self.extra_cost * qty_done
                finished_move.price_unit = (sum([-m.stock_valuation_layer_ids.value for m in consumed_moves.sudo()]) + work_center_cost + extra_cost) / qty_done
        return True

    def _prepare_wc_analytic_line(self, wc_line):
        wc = wc_line.workcenter_id
        hours = wc_line.duration / 60.0
        value = hours * wc.costs_hour
        account = wc.costs_hour_account_id.id
        return {
            'name': wc_line.name + ' (H)',
            'amount': -value,
            'account_id': account,
            'ref': wc.code,
            'unit_amount': hours,
            'company_id': self.company_id.id,
        }

    def _costs_generate(self):
        """ Calculates total costs at the end of the production.
        """
        self.ensure_one()
        AccountAnalyticLine = self.env['account.analytic.line'].sudo()
        for wc_line in self.workorder_ids.filtered('workcenter_id.costs_hour_account_id'):
            vals = self._prepare_wc_analytic_line(wc_line)
            precision_rounding = (wc_line.workcenter_id.costs_hour_account_id.currency_id or self.company_id.currency_id).rounding
            if not float_is_zero(vals.get('amount', 0.0), precision_rounding=precision_rounding):
                # we use SUPERUSER_ID as we do not guarantee an mrp user
                # has access to account analytic lines but still should be
                # able to produce orders
                AccountAnalyticLine.create(vals)

    def button_mark_done(self):
        self.ensure_one()
        res = super(MrpProduction, self).button_mark_done()
        self._costs_generate()
        return res

    def action_view_stock_valuation_layers(self):
        self.ensure_one()
        domain = [('id', 'in', (self.move_raw_ids + self.move_finished_ids + self.scrap_ids.move_id).stock_valuation_layer_ids.ids)]
        action = self.env.ref('stock_account.stock_valuation_layer_action').read()[0]
        context = literal_eval(action['context'])
        context.update(self.env.context)
        context['no_at_date'] = True
        return dict(action, domain=domain, context=context)

```

## File: models\mrp_workcenter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MrpWorkcenter(models.Model):
    _inherit = 'mrp.workcenter'

    costs_hour_account_id = fields.Many2one('account.analytic.account', string='Analytic Account',
                                            help="Fill this only if you want automatic analytic accounting entries on production orders.")

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _
from odoo.exceptions import UserError


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
        price = self._get_price_from_bom()
        action_rec = self.env.ref('stock_account.action_view_change_standard_price')
        action = action_rec.read([])[0]
        action['context'] = {'default_new_price': price}
        return action

    def action_bom_cost(self):
        real_time_products = self.filtered(lambda p: p.valuation == 'real_time')
        if real_time_products:
            raise UserError(_('The inventory valuation of some products %s is automated. You can only update their cost from the product form.') % (real_time_products.mapped('display_name')))

        boms_to_recompute = self.env['mrp.bom'].search(['|', ('product_id', 'in', self.ids), '&', ('product_id', '=', False), ('product_tmpl_id', 'in', self.mapped('product_tmpl_id').ids)])
        for product in self:
            product.standard_price = product._get_price_from_bom(boms_to_recompute)

    def _get_price_from_bom(self, boms_to_recompute=False):
        self.ensure_one()
        bom = self.env['mrp.bom']._bom_find(product=self)
        return self._compute_bom_price(bom, boms_to_recompute=boms_to_recompute)

    def _compute_bom_price(self, bom, boms_to_recompute=False):
        self.ensure_one()
        if not bom:
            return self.standard_price
        if not boms_to_recompute:
            boms_to_recompute = []
        total = 0
        for opt in bom.routing_id.operation_ids:
            duration_expected = (
                opt.workcenter_id.time_start +
                opt.workcenter_id.time_stop +
                opt.time_cycle)
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
        return bom.product_uom_id._compute_price(total / bom.product_qty, self.uom_id)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import mrp_workcenter
from . import mrp_production
from . import product
from . import account_move

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mrp_bom_invoice_bom,mrp.bom,mrp.model_mrp_bom,account.group_account_invoice,1,0,0,0
access_mrp_bom_line_invoice_bom,mrp.bom.line,mrp.model_mrp_bom_line,account.group_account_invoice,1,0,0,0

```

## File: views\mrp_production_views.xml

```xml
<odoo>
    <record id="mrp_production_form_view_inherited" model="ir.ui.view">
        <field name="name">mrp.production.view.inherited</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='finished_move_line_ids']" position="before">
                <field name="show_valuation" invisible="1"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button string="Valuation" type="object"
                    name="action_view_stock_valuation_layers"
                    class="oe_stat_button" icon="fa-dollar" groups="stock.group_stock_manager"
                    attrs="{'invisible': [('show_valuation', '=', False)]}" />
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
                <xpath expr="//span[@name='update_cost_price']" position="inside">
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
                <xpath expr="//span[@name='update_cost_price']" position="inside">
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
                <xpath expr="//div[@name='update_cost_price']" position="inside">
                    <field name="bom_count" invisible="1"/>
                    <button name="button_bom_cost"
                        string="Compute Price from BoM"
                        type="object"
                        attrs="{'invisible': ['|', ('bom_count', '=', 0), '&amp;', ('valuation', '=', 'real_time'), ('cost_method', '=', 'fifo')]}"
                        help="Compute the price of the product using products and operations of related bill of materials, for manufactured products only."
                        class="oe_link oe_read_only pt-0"/>
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

