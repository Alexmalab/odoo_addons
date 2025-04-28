# Odoo Module: l10n_pl_sale_stock

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2009 - now Grzegorz Grzelak grzegorz.grzelak@openglobe.pl

{
    'name': 'Poland - Sales and Stock',
    'version': '2.0',
    'author': 'Odoo S.A.',
    'category': 'Accounting/Localizations',
    'description': """
        Bridge module to compute the Sale Date for the invoice when sale_stock is installed
    """,
    'depends': [
        'l10n_pl', 'sale_stock',
    ],
    'data': [
    ],
    'demo': [
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_pl_delivery_date = fields.Date(
        compute='_compute_l10n_pl_delivery_date', store=True,
    )

    @api.depends('line_ids.sale_line_ids.order_id.effective_date')
    def _compute_l10n_pl_delivery_date(self):
        for move in self:
            sale_order_effective_date = list(filter(None, move.line_ids.sale_line_ids.order_id.mapped('effective_date')))
            effective_date_res = max(sale_order_effective_date) if sale_order_effective_date else False
            # if multiple sale order we take the bigger effective_date
            if effective_date_res:
                move.l10n_pl_delivery_date = effective_date_res

```

## File: models\__init__.py

```python

from . import account_move

```

