# Odoo Module: project_mrp_sale

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
    'name': "MRP Project Sale",
    'version': '1.0',
    'summary': "Technical Bridge",
    'category': 'Services/Project',
    'depends': ['project_mrp', 'sale_mrp', 'sale_project'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _prepare_procurement_values(self):
        res = super()._prepare_procurement_values()
        project = self.sale_line_id.order_id.project_id
        if project:
            res['project_id'] = project.id
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_move

```

