# Odoo Module: purchase_requisition_sale

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase Requisition Sale',
    'description': "Bridge module for Purchase requisition and Sales. Used to properly create purchase requisitions for subcontracted services",
    'version': '1.0',
    'category': 'Inventory/Purchase',
    'sequence': 70,
    'depends': ['purchase_requisition', 'sale_purchase'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: wizard\purchase_requisition_create_alternative.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class PurchaseRequisitionCreateAlternative(models.TransientModel):
    _inherit = 'purchase.requisition.create.alternative'

    @api.model
    def _get_alternative_line_value(self, order_line):
        res_line = super()._get_alternative_line_value(order_line)
        if order_line.sale_line_id:
            res_line['sale_line_id'] = order_line.sale_line_id.id

        return res_line

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_requisition_create_alternative

```

