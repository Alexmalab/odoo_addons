# Odoo Module: payment_payulatam

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from odoo.addons.payment.models.payment_acquirer import create_missing_journal_for_acquirers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'PayuLatam Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 370,
    'summary': 'Payment Acquirer: PayuLatam Implementation',
    'description': """Payulatam payment acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_payulatam_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'application': True,
    'post_init_hook': 'create_missing_journal_for_acquirers',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
import werkzeug

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class PayuLatamController(http.Controller):

    @http.route('/payment/payulatam/response', type='http', auth='public', csrf=False)
    def payulatam_response(self, **post):
        """ PayUlatam."""
        _logger.info('PayU Latam: entering form_feedback with post response data %s', pprint.pformat(post))
        if post:
            request.env['payment.transaction'].sudo().form_feedback(post, 'payulatam')
        return werkzeug.utils.redirect('/payment/process')

    @http.route('/payment/payulatam/webhook', type='http', auth='public', methods=['POST'], csrf=False)
    def payulatam_webhook(self, **data):
        _logger.info("handling confirmation from PayU Latam with data:\n%s", pprint.pformat(data))
        state_pol = data.get('state_pol')
        if state_pol == '4':
            lapTransactionState = 'APPROVED'
        elif state_pol == '6':
            lapTransactionState = 'DECLINED'
        elif state_pol == '5':
            lapTransactionState = 'EXPIRED'
        else:
            lapTransactionState = f'INVALID state_pol {state_pol}'

        data = {
            'signature': data.get('sign'),
            'TX_VALUE': data.get('value'),
            'currency': data.get('currency'),
            'referenceCode': data.get('reference_sale'),
            'transactionId': data.get('transaction_id'),
            'transactionState': data.get('state_pol'),
            'message': data.get('response_message_pol'),
            'lapTransactionState': lapTransactionState,
            'merchantId': data.get('merchant_id'),
        }

        try:
            request.env['payment.transaction'].sudo().with_context(
                payulatam_is_confirmation_page=True
            ).form_feedback(data, 'payulatam')
        except ValidationError:
            _logger.exception(
                'An error occurred while handling the confirmation from PayU with data:\n%s',
                pprint.pformat(data))
        return http.Response(status=200)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="payment.payment_acquirer_payulatam" model="payment.acquirer">
            <field name="provider">payulatam</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="payulatam_form"/>
        </record>
    </data>
</odoo>

```

## File: models\payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import decimal
import logging
import uuid

from hashlib import md5
from werkzeug import urls

from odoo import api, fields, models, _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.tools.float_utils import float_compare, float_round, float_split


_logger = logging.getLogger(__name__)


class PaymentAcquirerPayulatam(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[
        ('payulatam', 'PayU Latam')
    ], ondelete={'payulatam': 'set default'})
    payulatam_merchant_id = fields.Char(string="PayU Latam Merchant ID", required_if_provider='payulatam', groups='base.group_user')
    payulatam_account_id = fields.Char(string="PayU Latam Account ID", required_if_provider='payulatam', groups='base.group_user')
    payulatam_api_key = fields.Char(string="PayU Latam API Key", required_if_provider='payulatam', groups='base.group_user')

    def _get_payulatam_urls(self, environment):
        """ PayUlatam URLs"""
        if environment == 'prod':
            return 'https://checkout.payulatam.com/ppp-web-gateway-payu/'
        return 'https://sandbox.checkout.payulatam.com/ppp-web-gateway-payu/'

    def _payulatam_generate_sign(self, inout, values):
        if inout not in ('in', 'out'):
            raise Exception("Type must be 'in' or 'out'")

        if inout == 'in':
            data_string = ('~').join((self.payulatam_api_key, self.payulatam_merchant_id, values['referenceCode'],
                                      str(values['amount']), values['currency']))
        else:
            # "Confirmation" and "Response" pages have a different way to calculate what they call the `new_value`
            if self.env.context.get('payulatam_is_confirmation_page'):
                # https://developers.payulatam.com/latam/en/docs/integrations/webcheckout-integration/confirmation-page.html#signature-validation
                # For confirmation page, PayU Latam round to the first digit if the second one is a zero
                # to generate their signature.
                # e.g:
                #  150.00 -> 150.0
                #  150.26 -> 150.26
                # This happens to be Python 3's default behavior when casting to `float`.
                new_value = "%d.%d" % float_split(float(values.get('TX_VALUE')), 2)
            else:
                # https://developers.payulatam.com/latam/en/docs/integrations/webcheckout-integration/response-page.html#signature-validation
                # PayU Latam use the "Round half to even" rounding method to generate their signature.
                new_value = decimal.Decimal(values.get('TX_VALUE')).quantize(
                    decimal.Decimal('0.1'), decimal.ROUND_HALF_EVEN)
            data_string = ('~').join((self.payulatam_api_key, self.payulatam_merchant_id, values['referenceCode'],
                                      str(new_value), values['currency'], values.get('transactionState')))
        return md5(data_string.encode('utf-8')).hexdigest()

    def payulatam_form_generate_values(self, values):
        tx = self.env['payment.transaction'].search([('reference', '=', values.get('reference'))])
        # payulatam will not allow any payment twise even if payment was failed last time.
        # so, replace reference code if payment is not done or pending.
        if tx.state not in ['done', 'pending']:
            tx.reference = str(uuid.uuid4())
        payulatam_values = dict(
            values,
            merchantId=self.payulatam_merchant_id,
            accountId=self.payulatam_account_id,
            description=values.get('reference'),
            referenceCode=tx.reference,
            amount=float_round(values['amount'], 2),
            tax='0',  # This is the transaction VAT. If VAT zero is sent the system, 19% will be applied automatically. It can contain two decimals. Eg 19000.00. In the where you do not charge VAT, it should should be set as 0.
            taxReturnBase='0',
            currency=values['currency'].name,
            buyerEmail=values['partner_email'],
            responseUrl=urls.url_join(self.get_base_url(), '/payment/payulatam/response'),
            confirmationUrl=urls.url_join(self.get_base_url(), '/payment/payulatam/webhook'),
        )
        payulatam_values['signature'] = self._payulatam_generate_sign("in", payulatam_values)
        return payulatam_values

    def payulatam_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_payulatam_urls(environment)


class PaymentTransactionPayulatam(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _payulatam_form_get_tx_from_data(self, data):
        """ Given a data dict coming from payulatam, verify it and find the related
        transaction record. """
        reference, txnid, sign = data.get('referenceCode'), data.get('transactionId'), data.get('signature')
        if not reference or not txnid or not sign:
            raise ValidationError(_('PayU Latam: received data with missing reference (%s) or transaction id (%s) or sign (%s)') % (reference, txnid, sign))

        transaction = self.search([('reference', '=', reference)])

        if not transaction:
            error_msg = (_('PayU Latam: received data for reference %s; no order found') % (reference))
            raise ValidationError(error_msg)
        elif len(transaction) > 1:
            error_msg = (_('PayU Latam: received data for reference %s; multiple orders found') % (reference))
            raise ValidationError(error_msg)

        # verify shasign
        sign_check = transaction.acquirer_id._payulatam_generate_sign('out', data)
        if sign_check.upper() != sign.upper():
            raise ValidationError(('PayU Latam: invalid sign, received %s, computed %s, for data %s') % (sign, sign_check, data))
        return transaction

    def _payulatam_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        if self.acquirer_reference and data.get('transactionId') != self.acquirer_reference:
            invalid_parameters.append(('Reference code', data.get('transactionId'), self.acquirer_reference))
        if float_compare(float(data.get('TX_VALUE', '0.0')), self.amount, 2) != 0:
            invalid_parameters.append(('Amount', data.get('TX_VALUE'), '%.2f' % self.amount))
        if data.get('merchantId') != self.acquirer_id.payulatam_merchant_id:
            invalid_parameters.append(('Merchant Id', data.get('merchantId'), self.acquirer_id.payulatam_merchant_id))
        return invalid_parameters

    def _payulatam_form_validate(self, data):
        self.ensure_one()

        status = data.get('lapTransactionState') or data.find('transactionResponse').find('state').text
        res = {
            'acquirer_reference': data.get('transactionId') or data.find('transactionResponse').find('transactionId').text,
            'state_message': data.get('message') or ""
        }

        if status == 'APPROVED':
            _logger.info('Validated PayU Latam payment for tx %s: set as done' % (self.reference))
            res.update(state='done', date=fields.Datetime.now())
            self._set_transaction_done()
            self.write(res)
            self.execute_callback()
            return True
        elif status == 'PENDING':
            _logger.info('Received notification for PayU Latam payment %s: set as pending' % (self.reference))
            res.update(state='pending')
            self._set_transaction_pending()
            return self.write(res)
        elif status in ['EXPIRED', 'DECLINED']:
            _logger.info('Received notification for PayU Latam payment %s: set as Cancel' % (self.reference))
            res.update(state='cancel')
            self._set_transaction_cancel()
            return self.write(res)
        else:
            error = 'Received unrecognized status for PayU Latam payment %s: %s, set as error' % (self.reference, status)
            _logger.info(error)
            res.update(state='cancel', state_message=error)
            self._set_transaction_cancel()
            return self.write(res)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment

```

## File: views\payment_payulatam_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="payulatam_form">
        <div>
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <input type="hidden" name="merchantId" t-att-value='merchantId'/>
            <input type="hidden" name="accountId" t-att-value='accountId'/>
            <input type="hidden" name="description" t-att-value='description'/>
            <input type="hidden" name="referenceCode" t-att-value='referenceCode'/>
            <input type="hidden" name="amount" t-att-value='amount'/>
            <input type="hidden" name="currency" t-att-value='currency'/>
            <input type="hidden" name="signature" t-att-value='signature'/>
            <input type="hidden" name="tax" t-att-value='tax'/>
            <input type="hidden" name="taxReturnBase" t-att-value="taxReturnBase"/>
            <input type="hidden" name="buyerEmail" t-att-value='buyerEmail'/>
            <input type="hidden" name="responseUrl" t-att-value='responseUrl'/>
            <input type="hidden" name="confirmationUrl" t-att-value='confirmationUrl'/>
            <input type="hidden" name="extra1" t-att-value="extra1"/>
        </div>
    </template>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_acquirer_form_inherit_payment_payulatam" model="ir.ui.view">
        <field name="name">payment.acquirer.form.inherit.payment.payulatam</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'payulatam')]}">
                    <field name="payulatam_account_id" options="{'no_create': True}"/>
                    <field name="payulatam_merchant_id"/>
                    <field name="payulatam_api_key"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

