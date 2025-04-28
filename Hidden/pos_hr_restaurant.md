# Odoo Module: pos_hr_restaurant

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'POS HR Restaurant',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Link module between pos_hr and pos_restaurant',
    'description': """
This module adapts the behavior of the PoS when the pos_hr and pos_restaurant are installed.
""",
    'depends': ['pos_hr', 'pos_restaurant'],
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_hr_restaurant/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: static\src\overrides\components\navbar\navbar.js

```javascript
import { Navbar } from "@point_of_sale/app/navbar/navbar";
import { patch } from "@web/core/utils/patch";

patch(Navbar.prototype, {
    get showEditPlanButton() {
        if (
            this.pos.config.module_pos_restaurant &&
            (!this.pos.config.module_pos_hr || this.pos.employeeIsAdmin)
        ) {
            return super.showEditPlanButton;
        } else {
            return false;
        }
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import "@pos_restaurant/overrides/models/pos_store";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    shouldResetIdleTimer() {
        return this.mainScreen?.name !== "LoginScreen" && super.shouldResetIdleTimer(...arguments);
    },
});

```

