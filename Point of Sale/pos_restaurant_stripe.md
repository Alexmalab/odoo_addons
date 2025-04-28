# Odoo Module: pos_restaurant_stripe

Category: Point of Sale

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
    'name': 'POS Restaurant Stripe',
    'version': '1.0',
    'category': 'Point of Sale',
    'sequence': 6,
    'summary': 'Adds American style tipping to Stripe',
    'depends': ['pos_stripe', 'pos_restaurant', 'payment_stripe'],
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_restaurant_stripe/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_order.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class PosOrder(models.Model):
    _inherit = 'pos.order'

    def set_no_tip(self):
        """Capture the payment when no tip is set."""
        res = super(PosOrder, self).set_no_tip()

        for payment in self.payment_ids:
            if payment.payment_method_id.use_payment_terminal == 'stripe':
                payment.payment_method_id.stripe_capture_payment(payment.transaction_id)

        return res

```

## File: models\pos_payment.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class PosPayment(models.Model):
    _inherit = 'pos.payment'

    def _update_payment_line_for_tip(self, tip_amount):
        """Capture the payment when a tip is set."""
        res = super(PosPayment, self)._update_payment_line_for_tip(tip_amount)

        if self.payment_method_id.use_payment_terminal == 'stripe':
            self.payment_method_id.stripe_capture_payment(self.transaction_id, amount=self.amount)

        return res

```

## File: models\__init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_payment
from . import pos_order

```

## File: static\src\js\payment_stripe.js

```javascript
odoo.define('pos_restaurant_stripe.payment', function (require) {
    "use strict";

    var PaymentStripe = require('pos_stripe.payment');

    PaymentStripe.include({
        captureAfterPayment: async function (processPayment, line) {
            // Don't capture if the customer can tip, in that case we
            // will capture later.
            if (! this.canBeAdjusted(line.cid)) {
                return this._super(...arguments);
            }
        },

        canBeAdjusted: function (cid) {
            var order = this.pos.get_order();
            var line = order.get_paymentline(cid);
            return this.pos.config.set_tip_after_payment &&
                line.payment_method.use_payment_terminal === "stripe" &&
                line.card_type !== 'interac' &&
                ! line.card_type.includes('eftpos');
        }
    });
});

```

