# Odoo Module: project_purchase_stock

Category: Hidden

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
    'name': 'Project - Purchase - Stock',
    'version': '1.0',
    'description': 'Add a project link between POs and their generated stock pickings.',
    'license': 'LGPL-3',
    'category': 'Hidden',
    'depends': ['project_purchase', 'project_stock'],
    'auto_install': True,
}

```

## File: models\purchase_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    def _prepare_picking(self):
        res = super()._prepare_picking()
        if not self.project_id:
            return res
        return {
            **res,
            'project_id': self.project_id.id,
        }

```

## File: models\stock_rule.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _prepare_purchase_order(self, company_id, origins, values):
        res = super()._prepare_purchase_order(company_id, origins, values)
        if values[0].get('project_id'):
            res['project_id'] = values[0].get('project_id')
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_order
from . import stock_rule

```

