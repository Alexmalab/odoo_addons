# Odoo Module: pos_sale_margin

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'POS - Sale Margin',
    'version': '1.1',
    'category': 'Hidden',
    'summary': 'Link module between Point of Sale and Sales Margin',
    'description': """

This module adds enable you to view the margin of your Point of Sale orders in the Sales Margin report.
""",
    'depends': ['pos_sale', 'sale_margin'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleReport(models.Model):
    _inherit = "sale.report"

    def _fill_pos_fields(self, additional_fields):
        values = super()._fill_pos_fields(additional_fields)
        values['margin'] = 'SUM(l.price_subtotal - COALESCE(l.total_cost,0) / CASE COALESCE(pos.currency_rate, 0) WHEN 0 THEN 1.0 ELSE pos.currency_rate END)'
        return values

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_report

```

