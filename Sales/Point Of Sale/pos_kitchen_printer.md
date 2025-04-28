# Odoo Module: pos_kitchen_printer

Category: Sales/Point Of Sale

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
    'name': 'Pos Kitchen Printer',
    'version': '1.0',
    'category': 'Sales/Point Of Sale',
    'summary': 'Restaurant Kitchen Printer extensions for the Point of Sale ',
    'description': """

- Kitchen Order Printing: allows you to print orders updates to kitchen or bar printers

""",
    'depends': ['pos_restaurant'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosOrder(models.Model):
    _inherit = 'pos.order'

    multiprint_resume = fields.Char()

    @api.model
    def _order_fields(self, ui_order):
        order_fields = super(PosOrder, self)._order_fields(ui_order)
        order_fields['multiprint_resume'] = ui_order.get('multiprint_resume')
        return order_fields

    def _get_fields_for_draft_order(self):
        fields = super(PosOrder, self)._get_fields_for_draft_order()
        fields.append('multiprint_resume')
        return fields

    def _get_fields_for_order_line(self):
        fields = super(PosOrder, self)._get_fields_for_order_line()
        fields.append('mp_dirty')
        return fields


class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    mp_dirty = fields.Boolean()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order

```

