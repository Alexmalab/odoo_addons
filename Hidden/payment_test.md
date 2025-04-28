# Odoo Module: payment_test

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.payment.models.payment_acquirer import create_missing_journal_for_acquirers

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Test Payment Acquirer',
    'version': '1.0',
    'category': 'Hidden',
    'description': """
This module implements a simple test payment acquirer flow to allow
the testing of successful payments behaviors on e-commerce. It includes
a protection to avoid using it in production environment. However, do
not use it in production environment.
""",
    'depends': ['payment'],
    'data': [
        'views/payment_test_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'installable': True,
    'post_init_hook': 'create_missing_journal_for_acquirers',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.exceptions import UserError


class PaymentTestController(http.Controller):

    @http.route(['/payment/test/s2s/create_json_3ds'], type='json', auth='public', csrf=False)
    def payment_test_s2s_create_json_3ds(self, verify_validity=False, **kwargs):
        if not kwargs.get('partner_id'):
            kwargs = dict(kwargs, partner_id=request.env.user.partner_id.id)
        acquirer = request.env['payment.acquirer'].browse(int(kwargs.get('acquirer_id')))
        if acquirer.state != 'test':
            raise UserError(_("Please do not use this acquirer for a production environment!"))
        token = acquirer.s2s_process(kwargs)

        return {
            'result': True,
            'id': token.id,
            'short_name': 'short_name',
            '3d_secure': False,
            'verified': True,
        }

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="payment_acquirer_test" model="payment.acquirer">
        <field name="name">Test</field>
        <field name="display_as">Test (do not use in production)</field>
        <field name="sequence">99</field>
        <field name="provider">test</field>
        <field name="image_128" type="base64" file="payment_test/static/src/img/test_logo.jpg"/>
        <field name="description" type="html">
<p>Test Payment Acquirer</p>
<ul class="list-inline">
    <li class="list-inline-item"><i class="fa fa-check"/>S2S Online Payment</li>
    <li class="list-inline-item"><i class="fa fa-check"/>Payment status tracking</li>
    <li class="list-inline-item"><i class="fa fa-check"/>Never loose money while testing</li>
</ul>
        </field>
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_ideal"),
                                                        ref("payment.payment_icon_cc_bancontact"),
                                                        ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_visa")])]'/>

        <field name="module_id" ref="base.module_payment_test"/>
        <field name="company_id" ref="base.main_company"/>
        <field name="registration_view_template_id" ref="test_s2s_form"/>
        <field name="state">test</field>
        <field name="payment_flow">s2s</field>
    </record>
</data></odoo>

```

## File: models\payment_acquirer.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime
from uuid import uuid4

from odoo import api, exceptions, fields, models, _


class PaymentAcquirerTest(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('test', 'Test')])

    @api.model
    def create(self, values):
        if values.get('provider') == 'test' and 'state' in values and values.get('state') not in ('test', 'disabled'):
            raise exceptions.UserError(_('This acquirer should not be used for other purposes than testing.'))
        return super(PaymentAcquirerTest, self).create(values)

    def write(self, values):
        if any(rec.provider == 'test' for rec in self) and 'state' in values and values.get('state') not in ('test', 'disabled'):
            raise exceptions.UserError(_('This acquirer should not be used for other purposes than testing.'))
        return super(PaymentAcquirerTest, self).write(values)

    @api.model
    def test_s2s_form_process(self, data):
        """ Return a minimal token to allow proceeding to transaction creation. """
        payment_token = self.env['payment.token'].sudo().create({
            'acquirer_ref': uuid4(),
            'acquirer_id': int(data['acquirer_id']),
            'partner_id': int(data['partner_id']),
            'name': 'XXXXXXXXXXXX%s - %s' % (data['cc_number'][-4:], data['cc_holder_name'])
        })
        return payment_token


class PaymentTransactionTest(models.Model):
    _inherit = 'payment.transaction'

    def test_create(self, values):
        """Automatically set the transaction as successful upon creation. """
        return {'date': datetime.now(), 'state': 'done'}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_acquirer

```

## File: views\payment_test_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="test_s2s_form">
        <input type="hidden" name="data_set" data-create-route="/payment/test/s2s/create_json_3ds"/>
        <input type="hidden" name="acquirer_id" t-att-value="id"/>
        <input t-if="return_url" type="hidden" name="return_url" t-att-value="return_url"/>
        <input t-if="partner_id" type="hidden" name="partner_id" t-att-value="partner_id"/>
        <div t-attf-class="row mt8 #{'' if bootstrap_formatting else 'o_card_brand_detail'}">
            <div t-att-class="'form-group col-lg-12' if bootstrap_formatting else 'form-group'">
                <input type="tel" name="cc_number" id="cc_number" class="form-control" placeholder="Card number" data-is-required="true"/>
                <div class="card_placeholder"></div>
                <div class="visa"></div>
                <input type="hidden" name="cc_brand" value=""/>
            </div>
            <div t-att-class="'form-group col-lg-5' if bootstrap_formatting else 'form-group'">
                <input type="text" name="cc_holder_name" id="cc_holder_name" class="form-control" placeholder="Cardholder name" data-is-required="true"/>
            </div>
            <div t-att-class="'form-group col-lg-3' if bootstrap_formatting else 'form-group'">
                <input type="text" name="cc_expiry" id="cc_expiry" class="form-control" maxlength="7" placeholder="Expires (MM / YY)" data-is-required="true"/>
            </div>
            <div t-att-class="'form-group col-lg-4' if bootstrap_formatting else 'form-group'">
                <input type="text" name="cvc" id="cvc" class="form-control" maxlength="4" placeholder="CVC" data-is-required="true"/>
            </div>
        </div>
    </template>
</odoo>

```

