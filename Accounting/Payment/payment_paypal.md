# Odoo Module: payment_paypal

Category: Accounting/Payment

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from odoo.addons.payment.models.payment_acquirer import create_missing_journal_for_acquirers
from odoo.addons.payment import reset_payment_provider

def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'paypal')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Paypal Payment Acquirer',
    'category': 'Accounting/Payment',
    'summary': 'Payment Acquirer: Paypal Implementation',
    'version': '1.0',
    'description': """Paypal Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_paypal_templates.xml',
        'data/payment_acquirer_data.xml',
        'data/payment_paypal_email_data.xml',
    ],
    'installable': True,
    'post_init_hook': 'create_missing_journal_for_acquirers',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-

import json
import logging
import pprint

import requests
import werkzeug
from werkzeug import urls

from odoo import http
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class PaypalController(http.Controller):
    _notify_url = '/payment/paypal/ipn/'
    _return_url = '/payment/paypal/dpn/'
    _cancel_url = '/payment/paypal/cancel/'

    def _parse_pdt_response(self, response):
        """ Parse a text response for a PDT verification.

            :param str response: text response, structured in the following way:
                STATUS\nkey1=value1\nkey2=value2...\n
             or STATUS\nError message...\n
            :rtype tuple(str, dict)
            :return: tuple containing the STATUS str and the key/value pairs
                     parsed as a dict
        """
        lines = [line for line in response.split('\n') if line]
        status = lines.pop(0)

        pdt_post = {}
        for line in lines:
            split = line.split('=', 1)
            if len(split) == 2:
                pdt_post[split[0]] = urls.url_unquote_plus(split[1])
            else:
                _logger.warning('Paypal: error processing pdt response: %s', line)

        return status, pdt_post

    def paypal_validate_data(self, **post):
        """ Paypal IPN: three steps validation to ensure data correctness

         - step 1: return an empty HTTP 200 response -> will be done at the end
           by returning ''
         - step 2: POST the complete, unaltered message back to Paypal (preceded
           by cmd=_notify-validate or _notify-synch for PDT), with same encoding
         - step 3: paypal send either VERIFIED or INVALID (single word) for IPN
                   or SUCCESS or FAIL (+ data) for PDT

        Once data is validated, process it. """
        res = False
        post['cmd'] = '_notify-validate'
        reference = post.get('item_number')
        tx = None
        if reference:
            tx = request.env['payment.transaction'].sudo().search([('reference', '=', reference)])
        if not tx:
            # we have seemingly received a notification for a payment that did not come from
            # odoo, acknowledge it otherwise paypal will keep trying
            _logger.warning('received notification for unknown payment reference')
            return False
        paypal_url = tx.acquirer_id.paypal_get_form_action_url()
        pdt_request = bool(post.get('amt'))  # check for specific pdt param
        if pdt_request:
            # this means we are in PDT instead of DPN like before
            # fetch the PDT token
            post['at'] = tx and tx.acquirer_id.paypal_pdt_token or ''
            post['cmd'] = '_notify-synch'  # command is different in PDT than IPN/DPN
        urequest = requests.post(paypal_url, post)
        urequest.raise_for_status()
        resp = urequest.text
        if pdt_request:
            resp, post = self._parse_pdt_response(resp)
        if resp in ['VERIFIED', 'SUCCESS']:
            _logger.info('Paypal: validated data')
            res = request.env['payment.transaction'].sudo().form_feedback(post, 'paypal')
            if not res and tx:
                tx._set_transaction_error('Validation error occured. Please contact your administrator.')
        elif resp in ['INVALID', 'FAIL']:
            _logger.warning('Paypal: answered INVALID/FAIL on data verification')
            if tx:
                tx._set_transaction_error('Invalid response from Paypal. Please contact your administrator.')
        else:
            _logger.warning('Paypal: unrecognized paypal answer, received %s instead of VERIFIED/SUCCESS or INVALID/FAIL (validation: %s)' % (resp, 'PDT' if pdt_request else 'IPN/DPN'))
            if tx:
                tx._set_transaction_error('Unrecognized error from Paypal. Please contact your administrator.')
        return res

    @http.route('/payment/paypal/ipn/', type='http', auth='public', methods=['POST'], csrf=False)
    def paypal_ipn(self, **post):
        """ Paypal IPN. """
        _logger.info('Beginning Paypal IPN form_feedback with post data %s', pprint.pformat(post))  # debug
        try:
            self.paypal_validate_data(**post)
        except ValidationError:
            _logger.exception('Unable to validate the Paypal payment')
        return ''

    @http.route('/payment/paypal/dpn', type='http', auth="public", methods=['POST', 'GET'], csrf=False)
    def paypal_dpn(self, **post):
        """ Paypal DPN """
        _logger.info('Beginning Paypal DPN form_feedback with post data %s', pprint.pformat(post))  # debug
        try:
            res = self.paypal_validate_data(**post)
        except ValidationError:
            _logger.exception('Unable to validate the Paypal payment')
        return werkzeug.utils.redirect('/payment/process')

    @http.route('/payment/paypal/cancel', type='http', auth="public", csrf=False)
    def paypal_cancel(self, **post):
        """ When the user cancels its Paypal payment: GET on this route """
        _logger.info('Beginning Paypal cancel with post data %s', pprint.pformat(post))  # debug
        return werkzeug.utils.redirect('/payment/process')

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="payment.payment_acquirer_paypal" model="payment.acquirer">
            <field name="name">Paypal</field>
            <field name="image_128" type="base64" file="payment_paypal/static/src/img/paypal_icon.png"/>
            <field name="provider">paypal</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="paypal_form"/>
        </record>

    </data>
</odoo>

```

## File: data\payment_paypal_email_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <template id="mail_template_paypal_invite_user_to_configure">
        <div>
            <p>
                Hello,
                <br/><br/>
                You have received a payment through PayPal.<br/>
                Kindly follow the instructions given by PayPal to create your account.<br/>
                Then, help us complete your Paypal credentials in Odoo.<br/><br/>
            </p>
            <a t-attf-href="/web#id=#{acquirer.id}&amp;model=payment.acquirer&amp;view_type=form" style="background-color: #875A7B; padding: 10px; text-decoration: none; color: #fff; border-radius: 5px; font-size: 12px;">Set Paypal credentials</a>
            <p>
                <br/><br/>
                Thanks,<br/>
                <b>The Odoo Team</b>
            </p>
        </div>
    </template>
</odoo>

```

## File: models\payment.py

```python
# coding: utf-8

import json
import logging

import dateutil.parser
import pytz
from werkzeug import urls

from odoo import api, fields, models, _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.addons.payment_paypal.controllers.main import PaypalController
from odoo.tools.float_utils import float_compare


_logger = logging.getLogger(__name__)


class AcquirerPaypal(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('paypal', 'Paypal')])
    paypal_email_account = fields.Char('Email', required_if_provider='paypal', groups='base.group_user')
    paypal_seller_account = fields.Char(
        'Merchant Account ID', groups='base.group_user',
        help='The Merchant ID is used to ensure communications coming from Paypal are valid and secured.')
    paypal_use_ipn = fields.Boolean('Use IPN', default=True, help='Paypal Instant Payment Notification', groups='base.group_user')
    paypal_pdt_token = fields.Char(string='PDT Identity Token', help='Payment Data Transfer allows you to receive notification of successful payments as they are made.', groups='base.group_user')
    # Default paypal fees
    fees_dom_fixed = fields.Float(default=0.35)
    fees_dom_var = fields.Float(default=3.4)
    fees_int_fixed = fields.Float(default=0.35)
    fees_int_var = fields.Float(default=3.9)

    def _get_feature_support(self):
        """Get advanced feature support by provider.

        Each provider should add its technical in the corresponding
        key for the following features:
            * fees: support payment fees computations
            * authorize: support authorizing payment (separates
                         authorization and capture)
            * tokenize: support saving payment data in a payment.tokenize
                        object
        """
        res = super(AcquirerPaypal, self)._get_feature_support()
        res['fees'].append('paypal')
        return res

    @api.model
    def _get_paypal_urls(self, environment):
        """ Paypal URLS """
        if environment == 'prod':
            return {
                'paypal_form_url': 'https://www.paypal.com/cgi-bin/webscr',
                'paypal_rest_url': 'https://api.paypal.com/v1/oauth2/token',
            }
        else:
            return {
                'paypal_form_url': 'https://www.sandbox.paypal.com/cgi-bin/webscr',
                'paypal_rest_url': 'https://api.sandbox.paypal.com/v1/oauth2/token',
            }

    def paypal_compute_fees(self, amount, currency_id, country_id):
        """ Compute paypal fees.

            :param float amount: the amount to pay
            :param integer country_id: an ID of a res.country, or None. This is
                                       the customer's country, to be compared to
                                       the acquirer company country.
            :return float fees: computed fees
        """
        if not self.fees_active:
            return 0.0
        country = self.env['res.country'].browse(country_id)
        if country and self.company_id.sudo().country_id.id == country.id:
            percentage = self.fees_dom_var
            fixed = self.fees_dom_fixed
        else:
            percentage = self.fees_int_var
            fixed = self.fees_int_fixed
        fees = (percentage / 100.0 * amount + fixed) / (1 - percentage / 100.0)
        return fees

    def paypal_form_generate_values(self, values):
        base_url = self.get_base_url()

        paypal_tx_values = dict(values)
        paypal_tx_values.update({
            'cmd': '_xclick',
            'business': self.paypal_email_account,
            'item_name': '%s: %s' % (self.company_id.name, values['reference']),
            'item_number': values['reference'],
            'amount': values['amount'],
            'currency_code': values['currency'] and values['currency'].name or '',
            'address1': values.get('partner_address'),
            'city': values.get('partner_city'),
            'country': values.get('partner_country') and values.get('partner_country').code or '',
            'state': values.get('partner_state') and (values.get('partner_state').code or values.get('partner_state').name) or '',
            'email': values.get('partner_email'),
            'zip_code': values.get('partner_zip'),
            'first_name': values.get('partner_first_name'),
            'last_name': values.get('partner_last_name'),
            'paypal_return': urls.url_join(base_url, PaypalController._return_url),
            'notify_url': urls.url_join(base_url, PaypalController._notify_url),
            'cancel_return': urls.url_join(base_url, PaypalController._cancel_url),
            'handling': '%.2f' % paypal_tx_values.pop('fees', 0.0) if self.fees_active else False,
            'custom': json.dumps({'return_url': '%s' % paypal_tx_values.pop('return_url')}) if paypal_tx_values.get('return_url') else False,
        })
        return paypal_tx_values

    def paypal_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_paypal_urls(environment)['paypal_form_url']


class TxPaypal(models.Model):
    _inherit = 'payment.transaction'

    paypal_txn_type = fields.Char('Transaction type')

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def _paypal_form_get_tx_from_data(self, data):
        reference, txn_id = data.get('item_number'), data.get('txn_id')
        if not reference or not txn_id:
            error_msg = _('Paypal: received data with missing reference (%s) or txn_id (%s)') % (reference, txn_id)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        # find tx -> @TDENOTE use txn_id ?
        txs = self.env['payment.transaction'].search([('reference', '=', reference)])
        if not txs or len(txs) > 1:
            error_msg = 'Paypal: received data for reference %s' % (reference)
            if not txs:
                error_msg += '; no order found'
            else:
                error_msg += '; multiple order found'
            _logger.info(error_msg)
            raise ValidationError(error_msg)
        return txs[0]

    def _paypal_form_get_invalid_parameters(self, data):
        invalid_parameters = []
        _logger.info('Received a notification from Paypal with IPN version %s', data.get('notify_version'))
        if data.get('test_ipn'):
            _logger.warning(
                'Received a notification from Paypal using sandbox'
            ),

        # TODO: txn_id: shoudl be false at draft, set afterwards, and verified with txn details
        if self.acquirer_reference and data.get('txn_id') != self.acquirer_reference:
            invalid_parameters.append(('txn_id', data.get('txn_id'), self.acquirer_reference))
        # check what is buyed
        if float_compare(float(data.get('mc_gross', '0.0')), (self.amount + self.fees), 2) != 0:
            invalid_parameters.append(('mc_gross', data.get('mc_gross'), '%.2f' % (self.amount + self.fees)))  # mc_gross is amount + fees
        if data.get('mc_currency') != self.currency_id.name:
            invalid_parameters.append(('mc_currency', data.get('mc_currency'), self.currency_id.name))
        if 'handling_amount' in data and float_compare(float(data.get('handling_amount')), self.fees, 2) != 0:
            invalid_parameters.append(('handling_amount', data.get('handling_amount'), self.fees))
        # check buyer
        if self.payment_token_id and data.get('payer_id') != self.payment_token_id.acquirer_ref:
            invalid_parameters.append(('payer_id', data.get('payer_id'), self.payment_token_id.acquirer_ref))
        # check seller
        if data.get('receiver_id') and self.acquirer_id.paypal_seller_account and data['receiver_id'] != self.acquirer_id.paypal_seller_account:
            invalid_parameters.append(('receiver_id', data.get('receiver_id'), self.acquirer_id.paypal_seller_account))
        if not data.get('receiver_id') or not self.acquirer_id.paypal_seller_account:
            # Check receiver_email only if receiver_id was not checked.
            # In Paypal, this is possible to configure as receiver_email a different email than the business email (the login email)
            # In Odoo, there is only one field for the Paypal email: the business email. This isn't possible to set a receiver_email
            # different than the business email. Therefore, if you want such a configuration in your Paypal, you are then obliged to fill
            # the Merchant ID in the Paypal payment acquirer in Odoo, so the check is performed on this variable instead of the receiver_email.
            # At least one of the two checks must be done, to avoid fraudsters.
            if data.get('receiver_email') and data.get('receiver_email') != self.acquirer_id.paypal_email_account:
                invalid_parameters.append(('receiver_email', data.get('receiver_email'), self.acquirer_id.paypal_email_account))
            if data.get('business') and data.get('business') != self.acquirer_id.paypal_email_account:
                invalid_parameters.append(('business', data.get('business'), self.acquirer_id.paypal_email_account))

        return invalid_parameters

    def _paypal_form_validate(self, data):
        status = data.get('payment_status')
        former_tx_state = self.state
        res = {
            'acquirer_reference': data.get('txn_id'),
            'paypal_txn_type': data.get('payment_type'),
        }
        if not self.acquirer_id.paypal_pdt_token and not self.acquirer_id.paypal_seller_account and status in ['Completed', 'Processed', 'Pending']:
            template = self.env.ref('payment_paypal.mail_template_paypal_invite_user_to_configure', False)
            if template:
                render_template = template.render({
                    'acquirer': self.acquirer_id,
                }, engine='ir.qweb')
                mail_body = self.env['mail.thread']._replace_local_links(render_template)
                mail_values = {
                    'body_html': mail_body,
                    'subject': _('Add your Paypal account to Odoo'),
                    'email_to': self.acquirer_id.paypal_email_account,
                    'email_from': self.acquirer_id.create_uid.email
                }
                self.env['mail.mail'].sudo().create(mail_values).send()

        if status in ['Completed', 'Processed']:
            try:
                # dateutil and pytz don't recognize abbreviations PDT/PST
                tzinfos = {
                    'PST': -8 * 3600,
                    'PDT': -7 * 3600,
                }
                date = dateutil.parser.parse(data.get('payment_date'), tzinfos=tzinfos).astimezone(pytz.utc).replace(tzinfo=None)
            except:
                date = fields.Datetime.now()
            res.update(date=date)
            self._set_transaction_done()
            if self.state == 'done' and self.state != former_tx_state:
                _logger.info('Validated Paypal payment for tx %s: set as done' % (self.reference))
                return self.write(res)
            return True
        elif status in ['Pending', 'Expired']:
            res.update(state_message=data.get('pending_reason', ''))
            self._set_transaction_pending()
            if self.state == 'pending' and self.state != former_tx_state:
                _logger.info('Received notification for Paypal payment %s: set as pending' % (self.reference))
                return self.write(res)
            return True
        else:
            error = 'Received unrecognized status for Paypal payment %s: %s, set as error' % (self.reference, status)
            res.update(state_message=error)
            self._set_transaction_cancel()
            if self.state == 'cancel' and self.state != former_tx_state:
                _logger.info(error)
                return self.write(res)
            return True

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M35.363 37.5c1.129-2.152 1.841-4.318 2.137-6.5h11.536v-3.212a.342.342 0 0 0-.339-.343H37.5c-.109-.965-.34-1.879-.697-2.742h12.233c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V37.5H35.363zM15.46 55.642h-1.036V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm4.765-36h4.547c2.803 0 6.062 2.167 5.09 6.933-.858 4.21-4.06 6.686-7.892 6.686h-2.496c-.549 0-.913.478-.999 1.047L19.334 41h-4.326c-.548 0-1.085-.479-.999-1.048l2.997-19.904c.086-.57.45-1.048 1-1.048h4.182zm11.656 10.987c-.76 3.915-3.604 6.218-7.006 6.218l-.81-.003-.488.003c-.488.003-.727.442-.803.97l-.053.363-.774 5.388c-.081.56-.416 1.036-.92 1.072L19.896 44c-.487 0-.963-.445-.887-.974l.015-.103h1.985l1.043-7.26.029.004.176-1.36h1.589c4.727 0 8.518-3.272 9.606-8.307.512 1.003.717 2.325.393 3.987z"/><path id="e" d="M35.363 35.5c1.129-2.152 1.841-4.318 2.137-6.5h11.536v-3.212a.342.342 0 0 0-.339-.343H37.5c-.109-.965-.34-1.879-.697-2.742h12.233c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V35.5H35.363zM15.46 53.642h-1.036V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm4.765-36h4.547c2.803 0 6.062 2.167 5.09 6.933-.858 4.21-4.06 6.686-7.892 6.686h-2.496c-.549 0-.913.478-.999 1.047L19.334 39h-4.326c-.548 0-1.085-.479-.999-1.048l2.997-19.904c.086-.57.45-1.048 1-1.048h4.182zm11.656 10.987c-.76 3.915-3.604 6.218-7.006 6.218l-.81-.003-.488.003c-.488.003-.727.442-.803.97l-.053.363-.774 5.388c-.081.56-.416 1.036-.92 1.072L19.896 42c-.487 0-.963-.445-.887-.974l.015-.103h1.985l1.043-7.26.029.004.176-1.36h1.589c4.727 0 8.518-3.272 9.606-8.307.512 1.003.717 2.325.393 3.987z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L17.423 17.25H28l3.391 5.453h5.859L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_paypal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="paypal_form">
            <div>
                <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
                <input type="hidden" name="cmd" t-att-value="cmd"/>
                <input type="hidden" name="business" t-att-value="business"/>
                <input type="hidden" name="bn" value="OdooInc_SP" />
                <input type="hidden" name="item_name" t-att-value="item_name"/>
                <input type="hidden" name="item_number" t-att-value="item_number"/>
                <input type="hidden" name="amount" t-att-value="amount"/>
                <input t-if="handling" type="hidden" name="handling"
                    t-att-value="handling"/>
                <input type="hidden" name="currency_code" t-att-value="currency_code"/>
                <!-- partner / address data -->
                <input type="hidden" name="address1" t-att-value="address1"/>
                <input type="hidden" name="city" t-att-value="city"/>
                <input type="hidden" name="country" t-att-value="country"/>
                <input type="hidden" name="email" t-att-value="email"/>
                <input type="hidden" name="first_name" t-att-value="first_name"/>
                <input type="hidden" name="last_name" t-att-value="last_name"/>
                <input type="hidden" name="zip" t-att-value="zip_code"/>
                <input type="hidden" name="rm" value="2"/>
                <input t-if='state' type="hidden" name="state"
                    t-att-value='state'/>
                <!-- after payment parameters -->
                <input t-if='custom' type="hidden" name="custom"
                    t-att-value='custom'/>
                <!-- URLs -->
                <input t-if="paypal_return" type="hidden" name='return'
                    t-att-value="paypal_return"/>
                <input t-if="acquirer.paypal_use_ipn" type="hidden" name='notify_url'
                    t-att-value="notify_url"/>
                <input t-if="cancel_return" type="hidden" name="cancel_return"
                    t-att-value="cancel_return"/>
            </div>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="acquirer_form_paypal" model="ir.ui.view">
            <field name="name">acquirer.form.paypal</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//group[@name="acquirer"]' position='inside'>
                    <group attrs="{'invisible': [('provider', '!=', 'paypal')]}">
                        <field name="paypal_email_account" attrs="{'required':[ ('provider', '=', 'paypal'), ('state', '!=', 'disabled')]}"/>
                        <field name="paypal_seller_account"/>
                        <field name="paypal_pdt_token"/>
                        <field name="paypal_use_ipn" attrs="{'required':[ ('provider', '=', 'paypal'), ('state', '!=', 'disabled')]}"/>
                        <a colspan="2" href="https://www.odoo.com/documentation/13.0/applications/general/payment_acquirers/paypal.html" target="_blank">How to configure your paypal account?</a>
                    </group>
                </xpath>
            </field>
        </record>

        <record id="transaction_form_paypal" model="ir.ui.view">
            <field name="name">acquirer.transaction.form.paypal</field>
            <field name="model">payment.transaction</field>
            <field name="inherit_id" ref="payment.transaction_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='acquirer_reference']" position="after">
                    <field name="paypal_txn_type" readonly="1" attrs="{'invisible': [('provider', '!=', 'paypal')]}"/>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

