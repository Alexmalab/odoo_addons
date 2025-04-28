# Odoo Module: payment_alipay

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
    'name': 'Alipay Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 345,
    'summary': 'Payment Acquirer: Alipay Implementation',
    'description': """Alipay Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/alipay_views.xml',
        'views/payment_alipay_templates.xml',
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
import requests
import werkzeug

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class AlipayController(http.Controller):
    _notify_url = '/payment/alipay/notify'
    _return_url = '/payment/alipay/return'

    def _alipay_validate_data(self, **post):
        resp = post.get('trade_status')
        if resp:
            if resp in ['TRADE_FINISHED', 'TRADE_SUCCESS']:
                _logger.info('Alipay: validated data')
            elif resp == 'TRADE_CLOSED':
                _logger.warning('Alipay: payment refunded to user and closed the transaction')
            else:
                _logger.warning('Alipay: unrecognized alipay answer, received %s instead of TRADE_FINISHED/TRADE_SUCCESS and TRADE_CLOSED' % (post['trade_status']))
        if post.get('out_trade_no') and post.get('trade_no'):
            post['reference'] = request.env['payment.transaction'].sudo().search([('reference', '=', post['out_trade_no'])]).reference
            return request.env['payment.transaction'].sudo().form_feedback(post, 'alipay')
        return False

    def _alipay_validate_notification(self, **post):
        if post.get('out_trade_no'):
            alipay = request.env['payment.transaction'].sudo().search([('reference', '=', post.get('out_trade_no'))]).acquirer_id
        else:
            alipay = request.env['payment.acquirer'].sudo().search([('provider', '=', 'alipay')])
        val = {
            'service': 'notify_verify',
            'partner': alipay.alipay_merchant_partner_id,
            'notify_id': post['notify_id']
        }
        response = requests.post(alipay.alipay_get_form_action_url(), val)
        response.raise_for_status()
        _logger.info('Validate alipay Notification %s' % response.text)
        # After program is executed, the page must print “success” (without quote). If not, Alipay server would keep re-sending notification, until over 24 hour 22 minutes Generally, there are 8 notifications within 25 hours (Frequency: 2m,10m,15m,1h,2h,6h,15h)
        if response.text == 'true':
            self._alipay_validate_data(**post)
            return 'success'
        return ""

    @http.route('/payment/alipay/return', type='http', auth="public", methods=['GET', 'POST'], save_session=False)
    def alipay_return(self, **post):
        """ Alipay return

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.
        """
        _logger.info('Beginning Alipay form_feedback with post data %s', pprint.pformat(post))
        self._alipay_validate_data(**post)
        return werkzeug.utils.redirect('/payment/process')

    @http.route('/payment/alipay/notify', type='http', auth='public', methods=['POST'], csrf=False)
    def alipay_notify(self, **post):
        """ Alipay Notify """
        _logger.info('Beginning Alipay notification form_feedback with post data %s', pprint.pformat(post))
        return self._alipay_validate_notification(**post)

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
<odoo noupdate="1">
    <record id="payment.payment_acquirer_alipay" model="payment.acquirer">
        <field name="name">Alipay</field>
        <field name="provider">alipay</field>
        <field name="company_id" ref="base.main_company"/>
        <field name="view_template_id" ref="alipay_form"/>
    </record>
</odoo>

```

## File: models\payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from hashlib import md5
from werkzeug import urls

from odoo import api, fields, models, _
from odoo.tools.float_utils import float_compare
from odoo.addons.payment_alipay.controllers.main import AlipayController
from odoo.addons.payment.models.payment_acquirer import ValidationError

_logger = logging.getLogger(__name__)


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[
        ('alipay', 'Alipay')
    ], ondelete={'alipay': 'set default'})
    alipay_payment_method = fields.Selection([
        ('express_checkout', 'Express Checkout (only for Chinese Merchant)'),
        ('standard_checkout', 'Cross-border'),
    ], string='Account', default='express_checkout',
        help="  * Cross-border: For the Overseas seller \n  * Express Checkout: For the Chinese Seller")
    alipay_merchant_partner_id = fields.Char(
        string='Merchant Partner ID', required_if_provider='alipay', groups='base.group_user',
        help='The Merchant Partner ID is used to ensure communications coming from Alipay are valid and secured.')
    alipay_md5_signature_key = fields.Char(
        string='MD5 Signature Key', required_if_provider='alipay', groups='base.group_user',
        help="The MD5 private key is the 32-byte string which is composed of English letters and numbers.")
    alipay_seller_email = fields.Char(string='Alipay Seller Email', groups='base.group_user')

    def _get_feature_support(self):
        res = super(PaymentAcquirer, self)._get_feature_support()
        res['fees'].append('alipay')
        return res

    @api.model
    def _get_alipay_urls(self, environment):
        """ Alipay URLS """
        if environment == 'prod':
            return 'https://mapi.alipay.com/gateway.do'
        return 'https://openapi.alipaydev.com/gateway.do'

    def alipay_compute_fees(self, amount, currency_id, country_id):
        """ Compute alipay fees.

            :param float amount: the amount to pay
            :param integer country_id: an ID of a res.country, or None. This is
                                       the customer's country, to be compared to
                                       the acquirer company country.
            :return float fees: computed fees
        """
        fees = 0.0
        if self.fees_active:
            country = self.env['res.country'].browse(country_id)
            if country and self.company_id.sudo().country_id.id == country.id:
                percentage = self.fees_dom_var
                fixed = self.fees_dom_fixed
            else:
                percentage = self.fees_int_var
                fixed = self.fees_int_fixed
            fees = (percentage / 100.0 * amount + fixed) / (1 - percentage / 100.0)
        return fees

    def _build_sign(self, val):
        # Rearrange parameters in the data set alphabetically
        data_to_sign = sorted(val.items())
        # Exclude parameters that should not be signed
        data_to_sign = ["{}={}".format(k, v) for k, v in data_to_sign if k not in ['sign', 'sign_type', 'reference']]
        # And connect rearranged parameters with &
        data_string = '&'.join(data_to_sign)
        data_string += self.alipay_md5_signature_key
        return md5(data_string.encode('utf-8')).hexdigest()

    def _get_alipay_tx_values(self, values):
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')

        alipay_tx_values = ({
            '_input_charset': 'utf-8',
            'notify_url': urls.url_join(base_url, AlipayController._notify_url),
            'out_trade_no': values.get('reference'),
            'partner': self.alipay_merchant_partner_id,
            'return_url': urls.url_join(base_url, AlipayController._return_url),
            'subject': values.get('reference'),
            'total_fee': '%.2f' % (values.get('amount') + values.get('fees')),
        })
        if self.alipay_payment_method == 'standard_checkout':
            alipay_tx_values.update({
                'service': 'create_forex_trade',
                'product_code': 'NEW_OVERSEAS_SELLER',
                'currency': values.get('currency').name,
            })
        else:
            alipay_tx_values.update({
                'service': 'create_direct_pay_by_user',
                'payment_type': 1,
                'seller_email': self.alipay_seller_email,
            })
        sign = self._build_sign(alipay_tx_values)
        alipay_tx_values.update({
            'sign_type': 'MD5',
            'sign': sign,
        })
        return alipay_tx_values

    def alipay_form_generate_values(self, values):
        values.update(self._get_alipay_tx_values(values))
        return values

    def alipay_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_alipay_urls(environment)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _check_alipay_configuration(self, vals):
        acquirer_id = int(vals.get('acquirer_id'))
        acquirer = self.env['payment.acquirer'].sudo().browse(acquirer_id)
        if acquirer and acquirer.provider == 'alipay' and acquirer.alipay_payment_method == 'express_checkout':
            currency_id = int(vals.get('currency_id'))
            if currency_id:
                currency = self.env['res.currency'].sudo().browse(currency_id)
                if currency and currency.name != 'CNY':
                    _logger.info("Only CNY currency is allowed for Alipay Express Checkout")
                    raise ValidationError(_("""
                        Only transactions in Chinese Yuan (CNY) are allowed for Alipay Express Checkout.\n
                        If you wish to use another currency than CNY for your transactions, switch your
                        configuration to a Cross-border account on the Alipay payment acquirer in Odoo.
                    """))
        return True

    def write(self, vals):
        if vals.get('currency_id') or vals.get('acquirer_id'):
            for payment in self:
                check_vals = {
                    'acquirer_id': vals.get('acquirer_id', payment.acquirer_id.id),
                    'currency_id': vals.get('currency_id', payment.currency_id.id)
                }
                payment._check_alipay_configuration(check_vals)
        return super(PaymentTransaction, self).write(vals)

    @api.model
    def create(self, vals):
        self._check_alipay_configuration(vals)
        return super(PaymentTransaction, self).create(vals)

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def _alipay_form_get_tx_from_data(self, data):
        reference, txn_id, sign = data.get('reference'), data.get('trade_no'), data.get('sign')
        if not reference or not txn_id:
            _logger.info('Alipay: received data with missing reference (%s) or txn_id (%s)' % (reference, txn_id))
            raise ValidationError(_('Alipay: received data with missing reference (%s) or txn_id (%s)') % (reference, txn_id))

        txs = self.env['payment.transaction'].search([('reference', '=', reference)])
        if not txs or len(txs) > 1:
            error_msg = _('Alipay: received data for reference %s') % (reference)
            logger_msg = 'Alipay: received data for reference %s' % (reference)
            if not txs:
                error_msg += _('; no order found')
                logger_msg += '; no order found'
            else:
                error_msg += _('; multiple order found')
                logger_msg += '; multiple order found'
            _logger.info(logger_msg)
            raise ValidationError(error_msg)

        # verify sign
        sign_check = txs.acquirer_id._build_sign(data)
        if sign != sign_check:
            _logger.info('Alipay: invalid sign, received %s, computed %s, for data %s' % (sign, sign_check, data))
            raise ValidationError(_('Alipay: invalid sign, received %s, computed %s, for data %s') % (sign, sign_check, data))

        return txs

    def _alipay_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        if float_compare(float(data.get('total_fee', '0.0')), (self.amount + self.fees), 2) != 0:
            invalid_parameters.append(('total_fee', data.get('total_fee'), '%.2f' % (self.amount + self.fees)))  # mc_gross is amount + fees
        if self.acquirer_id.alipay_payment_method == 'standard_checkout':
            if data.get('currency') != self.currency_id.name:
                invalid_parameters.append(('currency', data.get('currency'), self.currency_id.name))
        else:
            if data.get('seller_email') != self.acquirer_id.alipay_seller_email:
                invalid_parameters.append(('seller_email', data.get('seller_email'), self.acquirer_id.alipay_seller_email))
        return invalid_parameters

    def _alipay_form_validate(self, data):
        if self.state in ['done']:
            _logger.info('Alipay: trying to validate an already validated tx (ref %s)', self.reference)
            return True

        status = data.get('trade_status')
        res = {
            'acquirer_reference': data.get('trade_no'),
        }
        if status in ['TRADE_FINISHED', 'TRADE_SUCCESS']:
            _logger.info('Validated Alipay payment for tx %s: set as done' % (self.reference))
            date_validate = fields.Datetime.now()
            res.update(date=date_validate)
            self._set_transaction_done()
            self.write(res)
            self.execute_callback()
            return True
        elif status == 'TRADE_CLOSED':
            _logger.info('Received notification for Alipay payment %s: set as Canceled' % (self.reference))
            res.update(state_message=data.get('close_reason', ''))
            self._set_transaction_cancel()
            return self.write(res)
        else:
            error = 'Received unrecognized status for Alipay payment %s: %s, set as error' % (self.reference, status)
            _logger.info(error)
            res.update(state_message=error)
            self._set_transaction_error()
            return self.write(res)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment

```

## File: views\alipay_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_acquirer_view_form_inherit_payment_alipay" model="ir.ui.view">
        <field name="name">payment.acquirer.view.form.inherit.payment.alipay</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'alipay')]}">
                    <field name="alipay_payment_method" widget="radio"/>
                    <field name="alipay_seller_email" attrs="{'invisible': [('alipay_payment_method', '=', 'standard_checkout')], 'required': [('provider', '=', 'alipay'), ('alipay_payment_method', '=', 'express_checkout')]}"/>
                    <field name="alipay_merchant_partner_id" password="True"/>
                    <field name="alipay_md5_signature_key" password="True"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\payment_alipay_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="alipay_form">
        <div>
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <input type="hidden" name="_input_charset" t-att-value="_input_charset"/>
            <input t-if="service == 'create_forex_trade'" type="hidden" name="currency" t-att-value="currency"/>
            <input type="hidden" name="notify_url" t-att-value="notify_url"/>
            <input type="hidden" name="out_trade_no" t-att-value="out_trade_no"/>
            <input type="hidden" name="partner" t-att-value="partner"/>
            <input t-if="product_code" type="hidden" name="product_code" t-att-value="product_code"/>
            <input type="hidden" name="return_url" t-att-value="return_url"/>
            <input type="hidden" name="service" t-att-value="service"/>
            <input type="hidden" name="sign" t-att-value="sign"/>
            <input type="hidden" name="subject" t-att-value="subject"/>
            <input type="hidden" name="total_fee" t-att-value="total_fee"/>
            <input type="hidden" name="sign_type" t-att-value="sign_type"/>
            <input t-if="payment_type" type="hidden" name="payment_type" t-att-value="payment_type"/>
            <input t-if="seller_email" type="hidden" name="seller_email" t-att-value="seller_email"/>
        </div>
    </template>
</odoo>

```

