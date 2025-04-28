# Odoo Module: pos_restaurant_adyen

Category: Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'POS Restaurant Adyen',
    'version': '1.0',
    'category': 'Point of Sale',
    'sequence': 6,
    'summary': 'Adds American style tipping to Adyen',
    'depends': ['pos_adyen', 'pos_restaurant', 'payment_adyen'],
    'data': [
        'views/pos_payment_method_views.xml',
        ],
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_restaurant_adyen/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import http
from odoo.http import request
from odoo.addons.payment_adyen.controllers.main import AdyenController

_logger = logging.getLogger(__name__)


class PosRestaurantAdyenController(AdyenController):

    @http.route()
    def adyen_webhook(self, **post):
        if post.get('eventCode') in ['CAPTURE', 'AUTHORISATION_ADJUSTMENT'] and post.get('success') != 'true':
                _logger.warning('%s for transaction_id %s failed', post.get('eventCode'), post.get('originalReference'))
        return super(PosRestaurantAdyenController, self).adyen_webhook(**post)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosOrder(models.Model):
    _inherit = 'pos.order'

    def action_pos_order_paid(self):
        res = super(PosOrder, self).action_pos_order_paid()
        if not self.config_id.set_tip_after_payment:
            payment_lines = self.payment_ids.filtered(lambda line: line.payment_method_id.use_payment_terminal == 'adyen')
            for payment_line in payment_lines:
                payment_line._adyen_capture()
        return res

```

## File: models\pos_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import requests

from odoo import models

TIMEOUT = 10


class PosPayment(models.Model):
    _inherit = 'pos.payment'

    def _update_payment_line_for_tip(self, tip_amount):
        """Capture the payment when a tip is set."""
        res = super(PosPayment, self)._update_payment_line_for_tip(tip_amount)
        if self.payment_method_id.use_payment_terminal == 'adyen':
            self._adyen_capture()
        return res

    def _adyen_capture(self):
        data = {
            'originalReference': self.transaction_id,
            'modificationAmount': {
                'value': int(self.amount * 10**self.currency_id.decimal_places),
                'currency': self.currency_id.name,
            },
            'merchantAccount': self.payment_method_id.adyen_merchant_account,
        }

        return self.payment_method_id.proxy_adyen_request(data, 'capture')

```

## File: models\pos_payment_method.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    adyen_merchant_account = fields.Char(help='The POS merchant account code used in Adyen')

    def _get_adyen_endpoints(self):
        return {
            **super(PosPaymentMethod, self)._get_adyen_endpoints(),
            'adjust': 'https://pal-%s.adyen.com/pal/servlet/Payment/v52/adjustAuthorisation',
            'capture': 'https://pal-%s.adyen.com/pal/servlet/Payment/v52/capture',
        }

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_pos_payment_method(self):
        result = super()._loader_params_pos_payment_method()
        result['search_params']['fields'].append('adyen_merchant_account')
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order
from . import pos_payment
from . import pos_payment_method
from . import pos_session

```

## File: static\src\js\payment_adyen.js

```javascript
odoo.define('pos_restaurant_adyen.payment', function (require) {
    "use strict";

    var PaymentAdyen = require('pos_adyen.payment');

    PaymentAdyen.include({
        _adyen_pay_data: function () {
            var data = this._super();

            if (data.SaleToPOIRequest.PaymentRequest.SaleData.SaleToAcquirerData) {
                data.SaleToPOIRequest.PaymentRequest.SaleData.SaleToAcquirerData += "&authorisationType=PreAuth";
            } else {
                data.SaleToPOIRequest.PaymentRequest.SaleData.SaleToAcquirerData = "authorisationType=PreAuth";
            }
    
            return data;
        },

        send_payment_adjust: function (cid) {
            var order = this.pos.get_order();
            var line = order.get_paymentline(cid);
            var data = {
                originalReference: line.transaction_id,
                modificationAmount: {
                    value: parseInt(line.amount * Math.pow(10, this.pos.currency.decimal_places)),
                    currency: this.pos.currency.name,
                },
                merchantAccount: this.payment_method.adyen_merchant_account,
                additionalData: {
                    industryUsage: 'DelayedCharge',
                },
            };

            return this._call_adyen(data, 'adjust');
        },

        canBeAdjusted: function (cid) {
            var order = this.pos.get_order();
            var line = order.get_paymentline(cid);
            return ['mc', 'visa', 'amex', 'discover'].includes(line.card_type);
        }
    });
});

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_restaurant_adyen" model="ir.ui.view">
      <field name="name">pos.payment.method.form.inherit.restaurant.adyen</field>
      <field name="model">pos.payment.method</field>
      <field name="inherit_id" ref="pos_adyen.pos_payment_method_view_form_inherit_pos_adyen"/>
      <field name="arch" type="xml">
          <xpath expr="//field[@name='adyen_terminal_identifier']" position="after">
                <field name="adyen_merchant_account" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
          </xpath>
      </field>
    </record>
</odoo>

```

