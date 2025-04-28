# Odoo Module: pos_sale_gift_card

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
    'name': 'pos_sale_gift_card',
    'version': '1.1',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Link module between pos_sale and gift_card',
    'description': """
""",
    'depends': ['pos_sale', 'gift_card'],
    'assets': {
        'point_of_sale.assets': [
            'pos_sale_gift_card/static/src/**/*',
        ],
        'web.assets_tests': [
            'pos_sale_gift_card/static/tests/**/*',
        ]
    },
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\gift_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class GiftCard(models.Model):
    _inherit = "gift.card"

    def can_be_used_in_pos(self, sale_order_origin_id=False):
        can_be_used = super().can_be_used()
        return can_be_used or (sale_order_origin_id.id == self.sale_order_id.id)

    def _get_confirmed_redeem_pos_order_lines(self):
        confirmed = super()._get_confirmed_redeem_pos_order_lines()
        return confirmed.filtered(
                lambda l: not l.sale_order_line_id
            )

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def read_converted(self):
        result = super().read_converted()
        for sale_line in result:
            gift_card = self.env['sale.order.line'].browse(sale_line['id']).gift_card_id
            if gift_card:
                sale_line['gift_card_id'] = gift_card.id
                #If the order hasn't been confirmed the qty_to_invoice is 0, and the qty in the PoS will also be 0 so we set it to 1
                sale_line['qty_to_invoice'] = 1

        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order_line
from . import gift_card

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_sale_gift_card.models', function (require) {
    "use strict";

    var models = require('point_of_sale.models');
    var super_order_model = models.Order.prototype;

    models.Order = models.Order.extend({
        set_orderline_options: function (line, options) {
            super_order_model.set_orderline_options.apply(this, arguments);
            if (options.gift_card_id) {
                line.gift_card_id = options.gift_card_id;
            }
        },
    });

});

```

## File: static\src\js\PaymenScreen.js

```javascript
odoo.define('pos_sale_gift_card.PaymentScreen', function (require) {
    "use strict";
    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');
    const PosSaleGiftCardPaymentScreen = PosGiftCardPaymentScreen => class extends PosGiftCardPaymentScreen {
        async isGiftCardValid(line) {
            let is_valid = await this.rpc({
                model: "gift.card",
                method: 'can_be_used_in_pos',
                args: [line.gift_card_id, line.sale_order_origin_id],
              });
            return is_valid;
        }
    };

    Registries.Component.extend(PaymentScreen, PosSaleGiftCardPaymentScreen);

    return PosSaleGiftCardPaymentScreen;
});

```

