# Odoo Module: payment_test

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'test')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Test Payment Acquirer',
    'version': '2.0',
    'category': 'Hidden',
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_templates.xml',
        'views/payment_test_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_test/static/src/js/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class PaymentTestController(http.Controller):

    @http.route('/payment/test/simulate_payment', type='json', auth='public')
    def test_simulate_payment(self, reference, customer_input):
        """ Simulate the response of a payment request.

        :param str reference: The reference of the transaction
        :param str customer_input: The payment method details
        :return: None
        """
        fake_api_response = {
            'reference': reference,
            'cc_summary': customer_input[-4:],
        }
        request.env['payment.transaction'].sudo()._handle_feedback_data('test', fake_api_response)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_acquirer_test" model="payment.acquirer">
        <field name="provider">test</field>
        <field name="inline_form_view_id" ref="inline_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">False</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">True</field>
        <field name="allow_tokenization">True</field>
    </record>

    <record id="payment_method_test" model="account.payment.method">
        <field name="name">Test</field>
        <field name="code">test</field>
        <field name="payment_type">inbound</field>
    </record>

</odoo>

```

## File: models\account_payment_method.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountPaymentMethod(models.Model):
    _inherit = 'account.payment.method'

    @api.model
    def _get_payment_method_information(self):
        res = super()._get_payment_method_information()
        res['test'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('test', 'Test')], ondelete={'test': 'set default'})

    @api.depends('provider')
    def _compute_view_configuration_fields(self):
        """ Override of payment to hide the credentials page.

        :return: None
        """
        super()._compute_view_configuration_fields()
        self.filtered(lambda acq: acq.provider == 'test').show_credentials_page = False

    @api.constrains('state', 'provider')
    def _check_acquirer_state(self):
        if self.filtered(lambda a: a.provider == 'test' and a.state not in ('test', 'disabled')):
            raise UserError(_("Test acquirers should never be enabled."))

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'test':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_test.payment_method_test').id

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _send_payment_request(self):
        """ Override of payment to simulate a payment request.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_payment_request()
        if self.provider != 'test':
            return

        # The payment request response would normally transit through the controller but in the end,
        # all that interests us is the reference. To avoid making a localhost request, we bypass the
        # controller and handle the fake feedback data directly.
        self._handle_feedback_data('test', {'reference': self.reference})

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on dummy data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The dummy feedback data
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'test':
            return tx

        reference = data.get('reference')
        tx = self.search([('reference', '=', reference), ('provider', '=', 'test')])
        if not tx:
            raise ValidationError(
                "Test: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on dummy data.

        Note: self.ensure_one()

        :param dict data: The dummy feedback data
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_feedback_data(data)
        if self.provider != "test":
            return

        self._set_done()  # Dummy transactions are always successful
        if self.tokenize:
            token = self.env['payment.token'].create({
                'acquirer_id': self.acquirer_id.id,
                'name': payment_utils.build_token_name(payment_details_short=data['cc_summary']),
                'partner_id': self.partner_id.id,
                'acquirer_ref': 'fake acquirer reference',
                'verified': True,
            })
            self.token_id = token.id

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_method
from . import payment_acquirer
from . import payment_transaction

```

## File: static\src\js\payment_form.js

```javascript
odoo.define('payment_test.payment_form', require => {
    'use strict';

    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const paymentTestMixin = {

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Simulate a feedback from a payment provider and redirect the customer to the status page.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} provider - The provider of the acquirer
         * @param {number} acquirerId - The id of the acquirer handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {Promise}
         */
        _processDirectPayment: function (provider, acquirerId, processingValues) {
            if (provider !== 'test') {
                return this._super(...arguments);
            }

            const customerInput = document.getElementById('customer_input').value;
            return this._rpc({
                route: '/payment/test/simulate_payment',
                params: {
                    'reference': processingValues.reference,
                    'customer_input': customerInput,
                },
            }).then(() => {
                window.location = '/payment/status';
            });
        },

        /**
         * Prepare the inline form of Test for direct payment.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} provider - The provider of the selected payment option's acquirer
         * @param {integer} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {Promise}
         */
        _prepareInlineForm: function (provider, paymentOptionId, flow) {
            if (provider !== 'test') {
                return this._super(...arguments);
            } else if (flow === 'token') {
                return Promise.resolve();
            }
            this._setPaymentFlow('direct');
            return Promise.resolve()
        },
    };
    checkoutForm.include(paymentTestMixin);
    manageForm.include(paymentTestMixin);
});

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="verified_token_checkmark" inherit_id="payment.verified_token_checkmark">
        <xpath expr="//t[@name='payment_test_hook']" position="replace">
            <t t-if="token.provider=='test'">
                <span class="badge badge-warning">Test Token</span>
            </t>
        </xpath>
    </template>

</odoo>

```

## File: views\payment_test_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="inline_form">
        <div t-attf-id="test-container-{{acquirer_id}}">
            <div class="row mt-8">
                <div class="form-group col-lg-12">
                    <input name="acquirer_id" type="hidden" id="acquirer_id" t-att-value="id"/>
                    <input name="partner_id" type="hidden" t-att-value="partner_id"/>
                </div>
                <div class="form-group col-lg-12">
                    <input type="text" name="customer_input" id="customer_input" class="form-control"
                           placeholder="Test Card Information"/>
                </div>
            </div>
        </div>
    </template>

</odoo>

```

