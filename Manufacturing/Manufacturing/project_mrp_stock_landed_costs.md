# Odoo Module: project_mrp_stock_landed_costs

Category: Manufacturing/Manufacturing

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
    'name': 'Project MRP Landed Costs',
    'version': '1.0',
    'summary': 'Technical Bridge',
    'category': 'Manufacturing/Manufacturing',
    'depends': ['project_mrp_account', 'mrp_landed_costs'],
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
        if self.cost_id.target_model == 'manufacturing':
            res['analytic_distribution'] = self.move_id.production_id.project_id._get_analytic_distribution()
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_landed_costs

```

