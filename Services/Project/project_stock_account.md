# Odoo Module: project_stock_account

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
    'name': 'Project Stock Account',
    'version': '1.0',
    'summary': 'Handle analytics in Stock pickings with Project',
    'category': 'Services/Project',
    'depends': ['stock_account', 'project_stock'],
    'data': [
        'views/stock_picking_type_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_analytic_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    category = fields.Selection(selection_add=[('picking_entry', 'Inventory Transfer')])

```

## File: models\analytic_applicability.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountAnalyticApplicability(models.Model):
    _inherit = 'account.analytic.applicability'
    _description = "Analytic Plan's Applicabilities"

    business_domain = fields.Selection(
        selection_add=[
            ('stock_picking', 'Stock Picking'),
        ],
        ondelete={'stock_picking': 'cascade'},
    )

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.osv import expression

from odoo import models, _lt


class Project(models.Model):
    _inherit = 'project.project'

    def _get_profitability_labels(self):
        return {
            **super()._get_profitability_labels(),
            'other_costs': _lt('Materials'),
        }

    def _get_profitability_sequence_per_invoice_type(self):
        return {
            **super()._get_profitability_sequence_per_invoice_type(),
            'other_costs': 12,
        }

    def _get_profitability_items(self, with_action=True):
        profitability_items = super()._get_profitability_items(with_action)
        aal_from_picking = self._get_items_from_aal_picking(with_action)
        if aal_from_picking:
            profitability_items['costs']['data'] += aal_from_picking
            profitability_items['costs']['total']['billed'] += aal_from_picking[0]['billed']
        return profitability_items

    def _get_items_from_aal_picking(self, with_action=True):
        domain = self._get_domain_aal_with_no_move_line()
        domain = expression.AND([
            domain,
            [('category', '=', 'picking_entry')]
        ])
        aal_other_search = self.env['account.analytic.line'].sudo().search_read(domain, ['id', 'amount', 'currency_id'])
        if not aal_other_search:
            return False

        dict_amount_per_currency_id = {}
        set_currency_ids = {self.currency_id.id}
        cost_ids = []
        for aal in aal_other_search:
            set_currency_ids.add(aal['currency_id'][0])
            aal_amount = aal['amount']
            if not dict_amount_per_currency_id.get(aal['currency_id'][0]):
                dict_amount_per_currency_id[aal['currency_id'][0]] = aal_amount
            else:
                dict_amount_per_currency_id[aal['currency_id'][0]] += aal_amount
            cost_ids.append(aal['id'])

        total_costs = 0.0
        for currency_id, amounts in dict_amount_per_currency_id.items():
            currency = self.env['res.currency'].browse(currency_id).with_prefetch(dict_amount_per_currency_id)
            total_costs += currency._convert(amounts, self.currency_id, self.company_id)

        profitability_sequence_per_invoice_type = self._get_profitability_sequence_per_invoice_type()
        costs = [{'id': 'other_costs', 'sequence': profitability_sequence_per_invoice_type['other_costs_aal'], 'billed': total_costs, 'to_bill': 0.0}]

        if with_action and self.env.user.has_group('account.group_account_readonly'):
            costs[0]['action'] = self._get_action_for_profitability_section(cost_ids, 'other_costs_aal')

        return costs

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import ValidationError
from odoo.osv.expression import OR
from odoo.tools import format_list


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _get_analytic_distribution(self):
        if not self.picking_type_id.analytic_costs:
            return super()._get_analytic_distribution()
        distribution = self.picking_id.project_id._get_analytic_distribution()
        return distribution or super()._get_analytic_distribution()

    def _prepare_analytic_line_values(self, account_field_values, amount, unit_amount):
        res = super()._prepare_analytic_line_values(account_field_values, amount, unit_amount)
        if self.picking_id:
            res['name'] = self.picking_id.name
            res['category'] = 'picking_entry'
        return res

    def _get_valid_moves_domain(self):
        return ['&', ('picking_id.project_id', '!=', False), ('picking_type_id.analytic_costs', '!=', False)]

    def _account_analytic_entry_move(self):
        domain = self._get_valid_moves_domain()
        domain = OR([[('picking_id', '=', False)], domain])
        valid_moves = self.filtered_domain(domain)
        super(StockMove, valid_moves)._account_analytic_entry_move()

    def _prepare_analytic_lines(self):
        res = super()._prepare_analytic_lines()
        if res and self.picking_id:
            # Check that all mandatory plans are set on the project linked to the picking of the stock move before generating the AALs
            project = self.picking_id.project_id
            mandatory_plans = project._get_mandatory_plans(self.company_id, business_domain='stock_picking')
            missing_plan_names = [plan['name'] for plan in mandatory_plans if not project[plan['column_name']]]
            if missing_plan_names:
                raise ValidationError(_(
                    "'%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the stock picking.",
                    missing_plan_names=format_list(self.env, missing_plan_names),
                    project_name=project.name,
                ))
        return res

```

## File: models\stock_picking_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PickingType(models.Model):
    _inherit = 'stock.picking.type'

    analytic_costs = fields.Boolean(help="Validating stock pickings will generate analytic entries for the selected project. Products set for re-invoicing will also be billed to the customer.")

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import analytic_applicability
from . import stock_move
from . import stock_picking_type
from . import project_project
from . import account_analytic_line

```

## File: views\stock_picking_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_type_form_inherit_sale_project_stock" model="ir.ui.view">
        <field name="name">stock.picking.type.inherit.sale_project_stock</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.view_picking_type_form"/>
        <field name="arch" type="xml">
            <field name="create_backorder" position="after">
                <field name="analytic_costs" invisible="code not in ('incoming', 'outgoing')" groups="analytic.group_analytic_accounting"/>
            </field>
        </field>
    </record>
</odoo>

```

