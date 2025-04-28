# Odoo Module: pos_hr_restaurant

Category: Hidden

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
    'name': 'PoS HR Restaurant',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Link module between pos_hr and pos_restaurant',
    'description': """
This module adapts the behavior of the PoS when the pos_hr and pos_restaurant are installed.
""",
    'depends': ['pos_hr', 'pos_restaurant'],
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_hr_restaurant/static/src/js/**/*.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class PosOrder(models.Model):
    _inherit = "pos.order"

    def _get_fields_for_draft_order(self):
        fields = super()._get_fields_for_draft_order()
        fields.append('employee_id')
        return fields

    @api.model
    def get_table_draft_orders(self, table_ids):
        table_orders = super().get_table_draft_orders(table_ids)
        for order in table_orders:
            if order['employee_id']:
                order['employee_id'] = order['employee_id'][0]

        return table_orders

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order

```

## File: static\src\js\Chrome.js

```javascript
/* @odoo-module alias=pos_restaurant_hr.chrome */

import Chrome from 'point_of_sale.Chrome';
import Registries from 'point_of_sale.Registries';


export const PosHrRestaurantChrome = (Chrome) => class extends Chrome {
    //@override
    _shouldResetIdleTimer() {
        return super._shouldResetIdleTimer() && this.tempScreen.name !== 'LoginScreen';
    }
}

Registries.Component.extend(Chrome, PosHrRestaurantChrome);

```

