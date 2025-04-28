# Odoo Module: pos_self_order_adyen

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    "name": "POS Self Order Adyen",
    "summary": "Addon for the Self Order App that allows customers to pay by Adyen.",
    "category": "Sales/Point Of Sale",
    "depends": ["pos_adyen", "pos_self_order"],
    "auto_install": True,
    "license": "LGPL-3",
}

```

## File: controllers\main.py

```python
import logging
from odoo.addons.pos_adyen.controllers.main import PosAdyenController
from odoo import fields
from odoo.http import request

_logger = logging.getLogger(__name__)


class PosSelfAdyenController(PosAdyenController):

    def _process_payment_response(self, data, adyen_pm_sudo):
        self_order_id = None
        try:
            self_order_id = PosAdyenController._get_additional_data_from_unparsed(data['SaleToPOIResponse']['PaymentResponse']['Response']['AdditionalResponse'], 'metadata.self_order_id')
        except KeyError:
            self_order_id = None

        if not self_order_id:
            return super()._process_payment_response(data, adyen_pm_sudo)

        order_sudo = request.env['pos.order'].sudo().search([('id', '=', self_order_id)], limit=1)
        if not order_sudo:
            _logger.warning('Received an Adyen event notification for the self order #%d that does not exist (anymore)', self_order_id)
            return request.make_json_response('[accepted]') # https://docs.adyen.com/point-of-sale/design-your-integration/choose-your-architecture/cloud/#guarantee

        order = order_sudo.sudo(False).with_user(order_sudo.session_id.config_id.self_ordering_default_user_id).with_company(order_sudo.session_id.config_id.company_id)

        payment_result = data['SaleToPOIResponse']['PaymentResponse']['Response']['Result']

        if payment_result == 'Success' and order.config_id.self_ordering_mode == 'kiosk':
            payment_amount = data['SaleToPOIResponse']['PaymentResponse']['PaymentResult']['AmountsResp']['AuthorizedAmount']
            card_type = data['SaleToPOIResponse']['PaymentResponse']['PaymentResult']['PaymentInstrumentData']['CardData']['PaymentBrand']
            transaction_id = data['SaleToPOIResponse']['PaymentResponse']['SaleData']['SaleTransactionID']['TransactionID']
            order.add_payment({
                'amount': payment_amount,
                'payment_date': fields.Datetime.now(),
                'payment_method_id': adyen_pm_sudo.id,
                'card_type': card_type,
                'cardholder_name': '',
                'transaction_id': transaction_id,
                'payment_status': payment_result,
                'ticket': '',
                'pos_order_id': order.id
            })
            order.action_pos_order_paid()
            order._send_order()

        if order.config_id.self_ordering_mode == 'kiosk':
            order.env['bus.bus']._sendone(f'pos_config-{order.config_id.access_token}', 'PAYMENT_STATUS', {
                'payment_result': payment_result,
                'order': order._export_for_self_order(),
            })
        return request.make_json_response('[accepted]') # https://docs.adyen.com/point-of-sale/design-your-integration/choose-your-architecture/cloud/#guarantee

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: models\pos_payment_method.py

```python
from datetime import datetime, timezone
import random
from odoo import models, api
from odoo.addons.pos_adyen.models.pos_payment_method import UNPREDICTABLE_ADYEN_DATA


class PosPaymentMethod(models.Model):
    _inherit = "pos.payment.method"

    @api.model
    def _get_valid_acquirer_data(self):
        res = super()._get_valid_acquirer_data()
        res['metadata.self_order_id'] = UNPREDICTABLE_ADYEN_DATA
        return res

    def _payment_request_from_kiosk(self, order):
        if self.use_payment_terminal != 'adyen':
            return super()._payment_request_from_kiosk(order)
        else:
            pos_config = order.session_id.config_id
            random_number = random.randrange(10**9, 10**10 - 1)

            # https://docs.adyen.com/point-of-sale/basic-tapi-integration/make-a-payment/#make-a-payment
            data = {
                'SaleToPOIRequest': {
                    'MessageHeader': {
                        'ProtocolVersion': "3.0",
                        'MessageClass': "Service",
                        'MessageType': "Request",
                        'MessageCategory': "Payment",
                        'SaleID': f'{pos_config.display_name} (ID:{pos_config.id})', #	Your unique ID for the POS system component to send this request from.
                        'ServiceID': str(random_number), # Your unique ID for this request, consisting of 1-10 alphanumeric characters.
                        'POIID': self.adyen_terminal_identifier, #	The unique ID of the terminal to send this request to.
                    },
                    'PaymentRequest': {
                        'SaleData': {
                            'SaleTransactionID': {
                                'TransactionID': order.pos_reference, # your reference to identify a payment.
                                'TimeStamp': datetime.now(tz=timezone.utc).isoformat(timespec='seconds'), # date and time of the request in UTC format.
                            },
                            'SaleToAcquirerData': 'metadata.self_order_id=' + str(order.id),
                        },
                        'PaymentTransaction': {
                            'AmountsReq': {
                                'Currency': order.currency_id.name, # the transaction currency.
                                'RequestedAmount': order.amount_total, # The final transaction amount.
                            },
                        },
                    },
                },
            }

            req = self.proxy_adyen_request(data)

            return req and (isinstance(req, bool) or not req.get('error'))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_payment_method

```

