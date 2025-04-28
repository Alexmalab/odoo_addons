# Odoo Module: stock_landed_costs_company

Category: Inventory/Inventory

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

def _stock_landed_costs_company_post_init(env):
    env.cr.execute("""
        UPDATE stock_landed_cost cost
        SET company_id = journal.company_id
        FROM account_journal journal
        WHERE cost.account_journal_id = journal.id
    """)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Landed Costs for company\'s branches',
    'version': '1.0',
    'description': """
    This module is a patch that stores the company_id field on the landed cost model.
    That way, it is possible to create/use landed costs from a branch.
    """,
    'depends': ['stock_landed_costs'],
    'category': 'Inventory/Inventory',
    'sequence': 16,
    'installable': True,
    'auto_install': True,
    'post_init_hook': '_stock_landed_costs_company_post_init',
    'license': 'LGPL-3',
}

```

## File: models\stock_landed_cost.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockLandedCost(models.Model):
    _inherit = 'stock.landed.cost'

    company_id = fields.Many2one('res.company', 'Company', required=True, related=False, default=lambda self: self.env.company)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_landed_cost

```

