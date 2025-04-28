# Odoo Module: project_sale_expense

Category: Hidden

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
    'name': 'Project - Sale - Expense',
    'version': '1.0',
    'description': 'Adds a full traceability of reinvoice expenses on the profitability report.',
    'license': 'LGPL-3',
    'category': 'Hidden',
    'depends': ['sale_project', 'sale_expense', 'project_hr_expense'],
    'auto_install': True,
}

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _sale_determine_order(self):
        """ For move lines created from expense, we override the normal behavior.
            Note: if no SO but an AA is given on the expense, we will determine anyway the SO from its project's AAs linked,
            using the same mecanism as in Vendor Bills.
        """
        mapping_from_project = self._get_so_mapping_from_project()
        mapping_from_expense = self._get_so_mapping_from_expense()
        mapping_from_project.update(mapping_from_expense)
        return mapping_from_project

```

## File: models\hr_expense.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Expense(models.Model):
    _inherit = "hr.expense"

    @api.depends('sale_order_id')
    def _compute_analytic_distribution(self):
        super()._compute_analytic_distribution()
        if not self.env.context.get('project_id'):
            for expense in self:
                if not self.sale_order_id:
                    continue
                expense.analytic_distribution = expense.sale_order_id.project_id._get_analytic_distribution()

```

## File: models\hr_expense_sheet.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class HrExpenseSheet(models.Model):
    _inherit = "hr.expense.sheet"

    def _do_create_moves(self):
        """ When creating the move of the expense, if the AA is given in the project of the SO, we take it as reference in the distribution.
            Otherwise, we create a AA for the project of the SO and set the distribution to it.
        """
        for expense in self.expense_line_ids:
            project = expense.sale_order_id.project_id
            if not project or expense.analytic_distribution:
                continue
            if not project.account_id:
                project._create_analytic_account()
            expense.analytic_distribution = project._get_analytic_distribution()
        return super()._do_create_moves()

```

## File: models\project_project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import models, fields
from collections import defaultdict


class Project(models.Model):
    _inherit = 'project.project'

    def _get_expenses_profitability_items(self, with_action=True):
        expenses_read_group = self.env['hr.expense']._read_group(
            [('sheet_id.state', 'in', ['post', 'done']), ('analytic_distribution', 'in', self.account_id.ids)],
            groupby=['sale_order_id', 'product_id', 'currency_id'],
            aggregates=['id:array_agg', 'untaxed_amount_currency:sum'],
        )
        if not expenses_read_group:
            return {}
        expenses_per_so_id = {}
        expense_ids = []
        dict_amount_per_currency = defaultdict(lambda: 0.0)
        can_see_expense = with_action and self.env.user.has_group('hr_expense.group_hr_expense_team_approver')
        for sale_order, product, currency, ids, untaxed_amount_currency_sum in expenses_read_group:
            expenses_per_so_id.setdefault(sale_order.id, {})[product.id] = ids
            if can_see_expense:
                expense_ids.extend(ids)
            dict_amount_per_currency[currency] += untaxed_amount_currency_sum

        amount_billed = 0.0
        for currency, untaxed_amount_currency_sum in dict_amount_per_currency.items():
            amount_billed += currency._convert(untaxed_amount_currency_sum, self.currency_id, self.company_id, round=False)

        sol_read_group = self.env['sale.order.line'].sudo()._read_group(
            [
                ('order_id', 'in', list(expenses_per_so_id.keys())),
                ('is_expense', '=', True),
                ('state', '=', 'sale'),
            ],
            ['order_id', 'product_id', 'currency_id'],
            ['untaxed_amount_to_invoice:sum', 'untaxed_amount_invoiced:sum'],
        )

        total_amount_expense_invoiced = total_amount_expense_to_invoice = 0.0
        reinvoice_expense_ids = []
        dict_invoices_amount_per_currency = defaultdict(lambda: {'to_invoice': 0.0, 'invoiced': 0.0})
        set_currency_ids = {self.currency_id.id}
        for order, product, currency, untaxed_amount_to_invoice_sum, untaxed_amount_invoiced_sum in sol_read_group:
            expense_data_per_product_id = expenses_per_so_id[order.id]
            set_currency_ids.add(currency.id)
            product_id = product.id
            if product_id in expense_data_per_product_id:
                dict_invoices_amount_per_currency[currency]['to_invoice'] += untaxed_amount_to_invoice_sum
                dict_invoices_amount_per_currency[currency]['invoiced'] += untaxed_amount_invoiced_sum
                reinvoice_expense_ids += expense_data_per_product_id[product_id]
        for currency, revenues in dict_invoices_amount_per_currency.items():
            total_amount_expense_to_invoice += currency._convert(revenues['to_invoice'], self.currency_id, self.company_id)
            total_amount_expense_invoiced += currency._convert(revenues['invoiced'], self.currency_id, self.company_id)

        section_id = 'expenses'
        sequence = self._get_profitability_sequence_per_invoice_type()[section_id]
        expense_data = {
            'costs': {
                'id': section_id,
                'sequence': sequence,
                'billed': -amount_billed,
                'to_bill': 0.0,
            },
        }
        if reinvoice_expense_ids:
            expense_data['revenues'] = {
                'id': section_id,
                'sequence': sequence,
                'invoiced': total_amount_expense_invoiced,
                'to_invoice': total_amount_expense_to_invoice,
            }
        if can_see_expense:
            def get_action(res_ids):
                args = [section_id, [('id', 'in', res_ids)]]
                if len(res_ids) == 1:
                    args.append(res_ids[0])
                return {'name': 'action_profitability_items', 'type': 'object', 'args': json.dumps(args)}

            if reinvoice_expense_ids:
                expense_data['revenues']['action'] = get_action(reinvoice_expense_ids)
            if expense_ids:
                expense_data['costs']['action'] = get_action(expense_ids)
        return expense_data

    def _get_already_included_profitability_invoice_line_ids(self):
        move_line_ids = super()._get_already_included_profitability_invoice_line_ids()
        expenses_read_group = self.env['hr.expense']._read_group(
            [('sheet_id.state', 'in', ['post', 'done']), ('analytic_distribution', 'in', self.account_id.ids)],
            groupby=['sale_order_id'],
            aggregates=['__count'],
        )
        if not expenses_read_group:
            return move_line_ids
        for sale_order, count in expenses_read_group:
            move_line_ids.extend(sale_order.invoice_ids.mapped('invoice_line_ids').ids)
        return move_line_ids

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_line
from . import hr_expense_sheet
from . import hr_expense
from . import project_project

```

