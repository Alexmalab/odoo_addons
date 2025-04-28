# Odoo Module: sale_expense_margin

Category: Sales/Sales

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
    'name': 'Sales Expense Margin',
    'version': '1.0',
    'category': 'Sales/Sales',
    'description': 'When re-invoicing the expense on the SO, set the cost to the total untaxed amount of the expense.',
    'depends': ['sale_expense', 'sale_margin'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _sale_prepare_sale_line_values(self, order, price):
        res = super()._sale_prepare_sale_line_values(order, price)
        if self.expense_id:
            res['expense_id'] = self.expense_id.id
        return res

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields

class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    expense_id = fields.Many2one('hr.expense', string='Expense')

    @api.depends('is_expense')
    def _compute_purchase_price(self):
        expense_lines = self.filtered('expense_id')
        for line in expense_lines:
            expense = line.expense_id
            product_cost = expense.untaxed_amount_currency / (expense.quantity or 1.0)
            line.purchase_price = line._convert_to_sol_currency(product_cost, expense.currency_id)

        return super(SaleOrderLine, self - expense_lines)._compute_purchase_price()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_line
from . import sale_order_line

```

