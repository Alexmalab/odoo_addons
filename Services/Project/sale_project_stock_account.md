# Odoo Module: sale_project_stock_account

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
    'name': 'Sale Project Stock Account',
    'version': '1.0',
    'summary': 'Technical Bridge',
    'category': 'Services/Project',
    'depends': ['sale_project', 'project_stock_account'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv.expression import AND


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _get_valid_moves_domain(self):
        domain = super()._get_valid_moves_domain()
        # If anglo-saxon accounting enabled: we do not generate AALs for the reinvoiced products
        if self.env.user.company_id.anglo_saxon_accounting:
            domain = AND([domain, [('product_id.expense_policy', 'not in', ('sales_price', 'cost'))]])
        return domain

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_move

```

