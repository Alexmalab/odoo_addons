# Odoo Module: pos_sale_loyalty

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
    'name': 'pos_sale_loyalty',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Link module between pos_sale and pos_loyalty',
    'description': """
This module correct some behaviors when both module are installed.
""",
    'depends': ['pos_sale', 'pos_loyalty'],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_sale_loyalty/static/src/js/**/*.js',
        ],
        'web.assets_tests': [
            'pos_sale_loyalty/static/src/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _get_sale_order_fields(self):
        field_names = super()._get_sale_order_fields()
        field_names.append('reward_id')
        return field_names

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order

```

## File: static\src\js\models.js

```javascript
/** @odoo-module alias=pos_sale_loyalty.models **/


import { Orderline } from 'point_of_sale.models';
import Registries from 'point_of_sale.Registries';

export const PosSaleLoyaltyOrderline = (Orderline) => class PosSaleLoyaltyOrderline extends Orderline {
    //@override
    ignoreLoyaltyPoints(args) {
        if (this.sale_order_origin_id) {
            return true;
        }
        return super.ignoreLoyaltyPoints(args);
    }
    //@override
    setQuantityFromSOL(saleOrderLine) {
        // we need to consider reward product such as discount in a quotation
        if (saleOrderLine.reward_id) {
            this.set_quantity(saleOrderLine.product_uom_qty);
        } else {
            super.setQuantityFromSOL(...arguments);
        }
    }
};

Registries.Model.extend(Orderline, PosSaleLoyaltyOrderline);

```

## File: static\src\tours\PosSaleLoyaltyTour.js

```javascript
/** @odoo-module **/

import { PaymentScreen } from 'point_of_sale.tour.PaymentScreenTourMethods';
import { ProductScreen } from 'pos_sale.tour.ProductScreenTourMethods';
import { ReceiptScreen } from 'point_of_sale.tour.ReceiptScreenTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

// First tour should not get any automatic rewards
startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickQuotationButton();
ProductScreen.do.selectFirstOrder();
ProductScreen.do.clickDisplayedProduct('Desk Pad');
ProductScreen.do.clickPayButton();
PaymentScreen.do.clickPaymentMethod('Bank');
PaymentScreen.do.clickValidate();
ReceiptScreen.check.isShown();

Tour.register('PosSaleLoyaltyTour1', { test: true, url: '/pos/web' }, getSteps());

```

