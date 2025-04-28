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

## File: models\account_move.py

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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields

class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    expense_id = fields.Many2one('hr.expense', string='Expense')

    @api.depends('is_expense')
    def _compute_purchase_price(self):
        date_today = fields.Date.context_today(self)
        expense_lines = self.filtered('expense_id')
        for line in expense_lines:
            if line.expense_id.product_has_cost:
                product_cost = line.expense_id.untaxed_amount / line.expense_id.quantity
            else:
                product_cost = line.expense_id.untaxed_amount

            from_currency = line.expense_id.currency_id
            to_currency = line.currency_id or line.order_id.currency_id

            if to_currency and product_cost and from_currency != to_currency:
                line.purchase_price = from_currency._convert(
                    from_amount=product_cost,
                    to_currency=to_currency,
                    company=line.company_id or self.env.company,
                    date=line.order_id.date_order or date_today,
                    round=False)
            else:
                line.purchase_price = product_cost
        return super(SaleOrderLine, self - expense_lines)._compute_purchase_price()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import sale_order_line

```

