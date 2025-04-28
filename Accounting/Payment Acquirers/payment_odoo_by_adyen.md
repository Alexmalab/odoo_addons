# Odoo Module: payment_odoo_by_adyen

Category: Accounting/Payment Acquirers

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
    'name': 'Odoo Payments by Adyen Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 330,
    'summary': 'Payment Acquirer: Odoo Payments by Adyen',
    'version': '1.0',
    'description': """Odoo Payments by Adyen""",
    'depends': ['payment', 'adyen_platforms'],
    'data': [
        'views/payment_views.xml',
        'views/payment_odoo_by_adyen_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'installable': True,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import pprint

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class OdooByAdyenController(http.Controller):
    _notification_url = '/payment/odoo_adyen/notification'

    @http.route('/payment/odoo_adyen/notification', type='json', auth='public', csrf=False)
    def odoo_adyen_notification(self):
        data = json.loads(request.httprequest.data)
        _logger.info('Beginning Odoo by Adyen form_feedback with data %s', pprint.pformat(data)) 
        if data.get('authResult') not in ['CANCELLED']:
            request.env['payment.transaction'].sudo().form_feedback(data, 'odoo_adyen')

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

        <record id="payment.payment_acquirer_odoo_by_adyen" model="payment.acquirer">
            <field name="name">Odoo Payments by Adyen</field>
            <field name="image_128" type="base64" file="payment_odoo_by_adyen/static/src/img/odoo_icon.png"/>
            <field name="provider">odoo_adyen</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="odoo_by_adyen_form"/>
        </record>

    </data>
</odoo>

```

## File: models\payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac
import json
import logging
from werkzeug import urls

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.addons.payment_odoo_by_adyen.controllers.main import OdooByAdyenController

_logger = logging.getLogger(__name__)


class AcquirerOdooByAdyen(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[
       ('odoo_adyen', 'Odoo Payments by Adyen')
    ], ondelete={'odoo_adyen': 'set default'})
    odoo_adyen_account_id = fields.Many2one('adyen.account', required_if_provider='odoo_adyen', related='company_id.adyen_account_id')
    odoo_adyen_payout_id = fields.Many2one('adyen.payout', required_if_provider='odoo_adyen', string='Adyen Payout', domain="[('adyen_account_id', '=', odoo_adyen_account_id)]")

    @api.constrains('provider', 'state')
    def _check_odoo_adyen_test(self):
        for payment_acquirer in self:
            if payment_acquirer.provider == 'odoo_adyen' and payment_acquirer.state == 'test':
                raise ValidationError(_('Odoo Payments by Adyen is not available in test mode.'))

    def _get_feature_support(self):
        res = super(AcquirerOdooByAdyen, self)._get_feature_support()
        res['tokenize'].append('odoo_adyen')
        return res

    @api.model
    def _odoo_adyen_format_amount(self, amount, currency_id):
        return {
            'value': int(amount * (10 ** currency_id.decimal_places)),
            'currency': currency_id.name,
        }

    @api.model
    def _odoo_adyen_compute_signature(self, amount, currency_id, reference):
        secret = self.env['ir.config_parameter'].sudo().get_param('database.secret')
        token_str = '%s%s%s' % (
            int(amount * (10 ** currency_id.decimal_places)),
            currency_id.name,
            reference
        )
        return hmac.new(secret.encode('utf-8'), token_str.encode('utf-8'), hashlib.sha256).hexdigest()

    def odoo_adyen_form_generate_values(self, values):
        # Don't use the value returned by `self.get_base_url` for the notification_url as
        # `request.httprequest.url_root` could be forged to retrieve the signature and
        # fake a payment update
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        data = {
            'adyen_uuid': self.odoo_adyen_account_id.adyen_uuid,
            'payout': self.odoo_adyen_payout_id.code,
            'amount': self._odoo_adyen_format_amount(values['amount'], values['currency']),
            'reference': values['reference'],
            'shopperLocale': values.get('partner_lang'),
            'metadata': {
                'merchant_signature': self._odoo_adyen_compute_signature(values['amount'],values['currency'],values['reference']),
                'notification_url': urls.url_join(base_url, OdooByAdyenController._notification_url),
            },
            'returnUrl': urls.url_join(self.get_base_url(), '/payment/process'),
        }

        if self.save_token in ['ask', 'always']:
            data.update({
                'shopperReference': '%s_%s' % (self.odoo_adyen_account_id.adyen_uuid, values['partner_id']),
                'storePaymentMethod': True,
                'recurringProcessingModel': 'CardOnFile',
            })

        values.update({
            'data': json.dumps(data),
        })
        return values

    def odoo_adyen_get_form_action_url(self):
        self.ensure_one()
        proxy_url = self.env['ir.config_parameter'].sudo().get_param('adyen_platforms.proxy_url')
        return urls.url_join(proxy_url, 'pay_by_link')

    def odoo_adyen_create_account(self):
        return self.env['adyen.account'].action_create_redirect()

class TxOdooByAdyen(models.Model):
    _inherit = 'payment.transaction'

    def odoo_adyen_s2s_do_transaction(self, **kwargs):
        self.ensure_one()
        # Don't use the value returned by `self.get_base_url` for the notification_url as
        # `request.httprequest.url_root` could be forged to retrieve the signature and
        # fake a payment update
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        data = {
            'payout': self.acquirer_id.odoo_adyen_payout_id.code,
            'amount': self.acquirer_id._odoo_adyen_format_amount(self.amount, self.currency_id),
            'reference': self.reference,
            'paymentMethod': {
                'type': self.payment_token_id.odoo_adyen_payment_method_type,
                'storedPaymentMethodId': self.payment_token_id.acquirer_ref,
            },
            'shopperReference': '%s_%s' % (self.acquirer_id.odoo_adyen_account_id.adyen_uuid, self.partner_id.id),
            'shopperInteraction': 'ContAuth',
            'metadata': {
                'merchant_signature': self.acquirer_id._odoo_adyen_compute_signature(self.amount, self.currency_id, self.reference),
                'notification_url': urls.url_join(base_url, OdooByAdyenController._notification_url),
            },
            'returnUrl': urls.url_join(self.get_base_url(), '/payment/process'),
        }
        self.acquirer_id.odoo_adyen_account_id._adyen_rpc('payments', data)

    @api.model
    def _odoo_adyen_form_get_tx_from_data(self, data):
        reference = data.get('merchantReference')
        if not reference:
            error_msg = _('Odoo Payments by Adyen: received data with missing reference (%s)', reference)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        tx = self.env['payment.transaction'].search([('reference', '=', reference)])
        if not tx or len(tx) > 1:
            error_msg = _('Odoo Payments by Adyen: received data for reference %s') % (reference)
            if not tx:
                error_msg += _('; no order found')
            else:
                error_msg += _('; multiple order found')
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        return tx

    def _odoo_adyen_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        if self.acquirer_reference and data.get('pspReference') != self.acquirer_reference:
            invalid_parameters.append(('pspReference', data.get('pspReference'), self.acquirer_reference))

        return invalid_parameters

    def _odoo_adyen_form_validate(self, data):
        merchant_signature = self.acquirer_id._odoo_adyen_compute_signature(self.amount, self.currency_id, self.reference)
        if merchant_signature != data['additionalData']['metadata.merchant_signature']:
            return False

        # Save token
        if self.partner_id and not self.payment_token_id and \
               (self.type == 'form_save' or self.acquirer_id.save_token == 'always') \
               and 'recurring.shopperReference' in data['additionalData']:
            res = self.acquirer_id.odoo_adyen_account_id._adyen_rpc('payment_methods', {
                'shopperReference': data['additionalData']['recurring.shopperReference']
            })
            stored_payment_methods = res['storedPaymentMethods']
            pm_id = data['additionalData']['recurring.recurringDetailReference']
            token_id = self.env['payment.token'].create({
                'name': _("Card No XXXXXXXXXXXX%s", data['additionalData']['cardSummary']),
                'acquirer_ref': pm_id,
                'acquirer_id': self.acquirer_id.id,
                'partner_id': self.partner_id.id,
                'odoo_adyen_payment_method_type': next(pm['type'] for pm in stored_payment_methods if pm['id'] == pm_id)
            })
            self.payment_token_id = token_id

        # Update status
        if data['success']:
            self.write({'acquirer_reference': data.get('pspReference')})
            self._set_transaction_done()
            return True
        else:
            error = _('Odoo Payment by Adyen: feedback error')
            _logger.info(error)
            self.write({'state_message': error})
            self._set_transaction_cancel()
            return False

class PaymentToken(models.Model):
    _inherit = 'payment.token'

    odoo_adyen_payment_method_type = fields.Char(string='PaymentMethod Type')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment

```

## File: views\payment_odoo_by_adyen_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="odoo_by_adyen_form">
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <input type="hidden" name="data" t-att-value="data"/>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="acquirer_form_odoo_by_adyen" model="ir.ui.view">
            <field name="name">acquirer.form.odoo.by.adyen</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//group[@name="acquirer"]' position='inside'>
                    <group attrs="{'invisible': [('provider', '!=', 'odoo_adyen')]}">
                        <button name="odoo_adyen_create_account" icon="fa-arrow-right" type="object" class="btn-link" string="Create an account in 1 minute"
                            attrs="{'invisible': [('odoo_adyen_account_id', '!=', False)]}"/>
                        <field name="odoo_adyen_account_id" readonly="1" attrs="{'invisible': [('odoo_adyen_account_id', '=', False)]}"/>
                        <field name="odoo_adyen_payout_id" attrs="{'invisible': [('odoo_adyen_account_id', '=', False)], 'required':[('provider', '=', 'odoo_adyen'), ('state', '!=', 'disabled')]}"/>
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

