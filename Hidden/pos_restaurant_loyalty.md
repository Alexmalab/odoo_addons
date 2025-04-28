# Odoo Module: pos_restaurant_loyalty

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'POS - Restaurant Loyality',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Link module between pos_restaurant and pos_loyalty',
    'description': """
This module correct some behaviors when both module are installed.
""",
    'depends': ['pos_restaurant', 'pos_loyalty'],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_restaurant_loyalty/static/src/**/*',
        ],
        'web.assets_tests': [
            'pos_restaurant_loyalty/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: static\src\overrides\models\pos_store.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    async setTable(table, orderUid = null) {
        await super.setTable(...arguments)
        this.selectedOrder._updateRewards()
    }
})

```

