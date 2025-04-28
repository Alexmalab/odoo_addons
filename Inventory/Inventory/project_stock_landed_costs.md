# Odoo Module: project_stock_landed_costs

Category: Inventory/Inventory

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
    'name': 'Project Stock Landed Costs',
    'version': '1.0',
    'summary': 'Technical Bridge',
    'category': 'Inventory/Inventory',
    'depends': ['project_stock_account', 'stock_landed_costs'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock_landed_costs.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AdjustmentLines(models.Model):
    _inherit = 'stock.valuation.adjustment.lines'

    def _prepare_account_move_line_values(self):
        res = super()._prepare_account_move_line_values()
        if self.cost_id.target_model == 'picking':
            res['analytic_distribution'] = self.move_id.picking_id.project_id._get_analytic_distribution()
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_landed_costs

```

