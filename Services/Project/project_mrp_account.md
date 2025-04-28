# Odoo Module: project_mrp_account

Category: Services/Project

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "MRP Account Project",
    'version': '1.0',
    'summary': "Monitor MRP account using project",
    'category': 'Services/Project',
    'depends': ['mrp_account', 'project_mrp'],
    'data': ['views/mrp_production_views.xml'],
    'demo': [
        'data/project_mrp_account_demo.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\project_mrp_account_demo.xml

```xml
<odoo>
    <data>
        <record id="product_product_dinning_table" model="product.product">
            <field name="name">Dining Table</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="standard_price">100.0</field>
            <field name="list_price">110.50</field>
            <field name="is_storable" eval="True"/>
            <field name="weight">5.00</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
        </record>

        <record id="mrp_production_drawer_1" model="mrp.production">
            <field name="product_id" ref="product.product_product_27"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">5</field>
            <field name="location_src_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_stock"/>
            <field name="bom_id" ref="mrp.mrp_bom_drawer"/>
        </record>
        <record id="mrp_production_dinning_table_2" model="mrp.production">
            <field name="product_id" ref="project_mrp_account.product_product_dinning_table"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">5</field>
            <field name="location_src_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_stock"/>
            <field name="bom_id" ref="mrp.mrp_bom_wood_panel"/>
        </record>

        <!-- Function confirm MO -->
        <function model="mrp.production" name="action_confirm" eval="[[
            ref('project_mrp_account.mrp_production_drawer_1'), ref('project_mrp_account.mrp_production_dinning_table_2'),
        ]]"/>
        <function model="mrp.production" name="button_mark_done" eval="[[
            ref('project_mrp_account.mrp_production_dinning_table_2'),
        ]]"/>
    </data>
</odoo>

```

## File: models\mrp_production.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    has_analytic_account = fields.Boolean(compute='_compute_has_analytic_account')

    @api.depends('project_id')
    def _compute_has_analytic_account(self):
        has_analytic_account_per_project_id = {p.id: bool(p._get_analytic_accounts()) for p in self.project_id}
        for production in self:
            production.has_analytic_account = has_analytic_account_per_project_id.get(production.project_id.id, False)

    def action_view_analytic_accounts(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'account.analytic.account',
            'domain': [('id', 'in', self.project_id._get_analytic_accounts().ids)],
            'name': _('Analytic Accounts'),
            'view_mode': 'list,form',
        }

    def write(self, vals):
        res = super().write(vals)
        for production in self:
            if 'project_id' in vals and production.state != 'draft':
                production.move_raw_ids._account_analytic_entry_move()
                production.workorder_ids._create_or_update_analytic_entry()
        return res

```

## File: models\mrp_workorder.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MrpWorkorder(models.Model):
    _inherit = 'mrp.workorder'

    def _create_or_update_analytic_entry_for_record(self, value, hours):
        super()._create_or_update_analytic_entry_for_record(value, hours)
        project = self.production_id.project_id
        mo_analytic_line_vals = self.env['account.analytic.account']._perform_analytic_distribution(project._get_analytic_distribution(), value, hours, self.mo_analytic_account_line_ids, self)
        if mo_analytic_line_vals:
            self.mo_analytic_account_line_ids += self.env['account.analytic.line'].sudo().create(mo_analytic_line_vals)

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv import expression


class Project(models.Model):
    _inherit = "project.project"

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_profitability_labels(self):
        labels = super()._get_profitability_labels()
        labels['manufacturing_order'] = self.env._('Manufacturing Orders')
        return labels

    def _get_profitability_sequence_per_invoice_type(self):
        sequence_per_invoice_type = super()._get_profitability_sequence_per_invoice_type()
        sequence_per_invoice_type['manufacturing_order'] = 12
        return sequence_per_invoice_type

    def _get_profitability_aal_domain(self):
        return expression.AND([
            super()._get_profitability_aal_domain(),
            [('category', '!=', 'manufacturing_order')],
        ])

    def _get_profitability_items(self, with_action=True):
        profitability_items = super()._get_profitability_items(with_action)
        mrp_category = 'manufacturing_order'
        mrp_aal_read_group = self.env['account.analytic.line'].sudo()._read_group(
            [('auto_account_id', 'in', self.account_id.ids), ('category', '=', mrp_category)],
            ['currency_id'],
            ['amount:sum'],
        )
        if mrp_aal_read_group:
            can_see_manufactoring_order = with_action and len(self) == 1 and self.env.user.has_group('mrp.group_mrp_user')
            total_amount = 0
            for currency, amount_summed in mrp_aal_read_group:
                total_amount += currency._convert(amount_summed, self.currency_id, self.company_id)

            mrp_costs = {
                'id': mrp_category,
                'sequence': self._get_profitability_sequence_per_invoice_type()[mrp_category],
                'billed': total_amount,
                'to_bill': 0.0,
            }
            if can_see_manufactoring_order:
                mrp_costs['action'] = {'name': 'action_view_mrp_production', 'type': 'object'}
            costs = profitability_items['costs']
            costs['data'].append(mrp_costs)
            costs['total']['billed'] += mrp_costs['billed']
        return profitability_items

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import ValidationError
from odoo.tools import format_list


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _get_analytic_distribution(self):
        distribution = self.raw_material_production_id.project_id._get_analytic_distribution()
        return distribution or super()._get_analytic_distribution()

    def _prepare_analytic_line_values(self, account_field_values, amount, unit_amount):
        res = super()._prepare_analytic_line_values(account_field_values, amount, unit_amount)
        if self.raw_material_production_id:
            res['category'] = 'manufacturing_order'
        return res

    def _prepare_analytic_lines(self):
        res = super()._prepare_analytic_lines()
        if res and self.raw_material_production_id:
            # Check that all mandatory plans are set on the project linked to the MO of the stock move before generating the AALs
            project = self.raw_material_production_id.project_id
            mandatory_plans = project._get_mandatory_plans(self.company_id, business_domain='manufacturing_order')
            missing_plan_names = [plan['name'] for plan in mandatory_plans if not project[plan['column_name']]]
            if missing_plan_names:
                raise ValidationError(_(
                    "'%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the manufacturing order.",
                    missing_plan_names=format_list(self.env, missing_plan_names),
                    project_name=project.name,
                ))
        return res

```

## File: models\stock_rule.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _prepare_mo_vals(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom):
        res = super()._prepare_mo_vals(product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom)
        if values.get('project_id'):
            res['project_id'] = values['project_id']
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_production
from . import mrp_workorder
from . import project_project
from . import stock_move
from . import stock_rule

```

## File: views\mrp_production_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_production_form_view_inherit_project_mrp_account" model="ir.ui.view">
        <field name="name">mrp.production.view.inherit.project_mrp_account</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp_account.mrp_production_form_view_inherited"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_stock_valuation_layers']" position="after">
                <button class="oe_stat_button" type="object"
                    name="action_view_analytic_accounts"
                    icon="fa-bar-chart-o"
                    invisible="not has_analytic_account or state in ['draft', 'cancel']"
                    groups="analytic.group_analytic_accounting">
                    <div class="o_stat_info">
                        <span class="o_stat_text">Analytic Account</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

