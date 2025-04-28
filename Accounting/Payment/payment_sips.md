# Odoo Module: payment_sips

Category: Accounting/Payment

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import controllers
from odoo.addons.payment.models.payment_acquirer import create_missing_journal_for_acquirers
from odoo.addons.payment import reset_payment_provider

def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'sips')

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright 2015 Eezee-It

{
    'name': 'Worldline SIPS',
    'version': '1.0',
    'author': 'Eezee-It',
    'category': 'Accounting/Payment',
    'description': """
Worldline SIPS Payment Acquirer for online payments

Works with Worldline keys version 2.0, contains implementation of
payments acquirer using Worldline SIPS.""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_sips_templates.xml',
        'data/payment_acquirer_data.xml',
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

# Copyright 2015 Eezee-It

import json
import logging
import pprint
import werkzeug

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class SipsController(http.Controller):
    _notify_url = '/payment/sips/ipn/'
    _return_url = '/payment/sips/dpn/'

    def sips_validate_data(self, **post):
        sips = request.env['payment.acquirer'].search([('provider', '=', 'sips')], limit=1)
        security = sips.sudo()._sips_generate_shasign(post)
        if security == post['Seal']:
            _logger.debug('Sips: validated data')
            return request.env['payment.transaction'].sudo().form_feedback(post, 'sips')
        _logger.warning('Sips: data are corrupted')
        return False

    @http.route([
        '/payment/sips/ipn/'],
        type='http', auth='public', methods=['POST'], csrf=False)
    def sips_ipn(self, **post):
        """ Sips IPN. """
        _logger.info('Beginning Sips IPN form_feedback with post data %s', pprint.pformat(post))  # debug
        if not post:
            # SIPS sometimes send empty notification, the reason why is
            # unclear but they tend to pollute logs and do not provide any
            # meaningful information; log as a warning instead of a traceback
            _logger.warning('Sips: received empty notification; skip.')
        else:
            self.sips_validate_data(**post)
        return ''

    @http.route([
        '/payment/sips/dpn'], type='http', auth="public", methods=['POST'], csrf=False, save_session=False)
    def sips_dpn(self, **post):
        """ Sips DPN
        The session cookie created by Odoo has not the attribute SameSite. Most of browsers will force this attribute
        with the value 'Lax'. After the payment, Sips will perform a POST request on this route. For all these reasons,
        the cookie won't be added to the request. As a result, if we want to save the session, the server will create
        a new session cookie. Therefore, the previous session and all related information will be lost, so it will lead
        to undesirable behaviors. This is the reason why `save_session=False` is needed.
        """
        try:
            _logger.info('Beginning Sips DPN form_feedback with post data %s', pprint.pformat(post))  # debug
            self.sips_validate_data(**post)
        except:
            pass
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
    <data noupdate="0">

        <record id="payment.payment_acquirer_sips" model="payment.acquirer">
            <field name="name">Sips</field>
            <field name="image_128" type="base64" file="payment_sips/static/src/img/sips_icon.png"/>
            <field name="provider">sips</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="payment_sips.sips_form"/>
        </record>

    </data>
</odoo>

```

## File: models\payment.py

```python
# coding: utf-8

# Copyright 2015 Eezee-It

import datetime
from dateutil import parser
import json
import logging
import pytz
import re
import time
from hashlib import sha256

from werkzeug import urls

from odoo import models, fields, api
from odoo.tools.float_utils import float_compare
from odoo.tools.translate import _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.addons.payment_sips.controllers.main import SipsController

_logger = logging.getLogger(__name__)


CURRENCY_CODES = {
    'EUR': '978',
    'USD': '840',
    'CHF': '756',
    'GBP': '826',
    'CAD': '124',
    'JPY': '392',
    'MXN': '484',
    'TRY': '949',
    'AUD': '036',
    'NZD': '554',
    'NOK': '578',
    'BRL': '986',
    'ARS': '032',
    'KHR': '116',
    'TWD': '901',
}


class AcquirerSips(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('sips', 'Sips')])
    sips_merchant_id = fields.Char('Merchant ID', help="Used for production only", required_if_provider='sips', groups='base.group_user')
    sips_secret = fields.Char('Secret Key', size=64, required_if_provider='sips', groups='base.group_user')
    sips_test_url = fields.Char("Test url", required_if_provider='sips', default='https://payment-webinit.simu.sips-services.com/paymentInit')
    sips_prod_url = fields.Char("Production url", required_if_provider='sips', default='https://payment-webinit.sips-services.com/paymentInit')
    sips_version = fields.Char("Interface Version", required_if_provider='sips', default='HP_2.3')

    def _sips_generate_shasign(self, values):
        """ Generate the shasign for incoming or outgoing communications.
        :param dict values: transaction values
        :return string: shasign
        """
        if self.provider != 'sips':
            raise ValidationError(_('Incorrect payment acquirer provider'))
        data = values['Data']

        # Test key provided by Worldine
        key = u'002001000000001_KEY1'

        if self.state == 'enabled':
            key = getattr(self, 'sips_secret')

        shasign = sha256((data + key).encode('utf-8'))
        return shasign.hexdigest()

    def sips_form_generate_values(self, values):
        self.ensure_one()
        base_url = self.get_base_url()
        currency = self.env['res.currency'].sudo().browse(values['currency_id'])
        currency_code = CURRENCY_CODES.get(currency.name, False)
        if not currency_code:
            raise ValidationError(_('Currency not supported by Wordline'))
        amount = round(values['amount'] * 100)
        if self.state == 'enabled':
            # For production environment, key version 2 is required
            merchant_id = getattr(self, 'sips_merchant_id')
            key_version = self.env['ir.config_parameter'].sudo().get_param('sips.key_version', '2')
        else:
            # Test key provided by Atos Wordline works only with version 1
            merchant_id = '002001000000001'
            key_version = '1'

        sips_tx_values = dict(values)
        sips_tx_values.update({
            'Data': u'amount=%s|' % amount +
                    u'currencyCode=%s|' % currency_code +
                    u'merchantId=%s|' % merchant_id +
                    u'normalReturnUrl=%s|' % urls.url_join(base_url, SipsController._return_url) +
                    u'automaticResponseUrl=%s|' % urls.url_join(base_url, SipsController._notify_url) +
                    u'transactionReference=%s|' % values['reference'] +
                    u'statementReference=%s|' % values['reference'] +
                    u'keyVersion=%s' % key_version,
            'InterfaceVersion': self.sips_version,
        })

        return_context = {}
        if sips_tx_values.get('return_url'):
            return_context[u'return_url'] = u'%s' % urls.url_quote(sips_tx_values.pop('return_url'))
        return_context[u'reference'] = u'%s' % sips_tx_values['reference']
        sips_tx_values['Data'] += u'|returnContext=%s' % (json.dumps(return_context))

        shasign = self._sips_generate_shasign(sips_tx_values)
        sips_tx_values['Seal'] = shasign
        return sips_tx_values

    def sips_get_form_action_url(self):
        self.ensure_one()
        return self.state == 'enabled' and self.sips_prod_url or self.sips_test_url


class TxSips(models.Model):
    _inherit = 'payment.transaction'

    _sips_valid_tx_status = ['00']
    _sips_wait_tx_status = ['90', '99']
    _sips_refused_tx_status = ['05', '14', '34', '54', '75', '97']
    _sips_error_tx_status = ['03', '12', '24', '25', '30', '40', '51', '63', '94']
    _sips_pending_tx_status = ['60']
    _sips_cancel_tx_status = ['17']

    @api.model
    def _compute_reference(self, values=None, prefix=None):
        res = super(TxSips, self)._compute_reference(values=values, prefix=prefix)
        acquirer = self.env['payment.acquirer'].browse(values.get('acquirer_id'))
        if acquirer and acquirer.provider == 'sips':
            return re.sub(r'[^0-9a-zA-Z]+', 'x', res) + 'x' + str(int(time.time()))
        return res

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    def _sips_data_to_object(self, data):
        res = {}
        for element in data.split('|'):
            element_split = element.split('=')
            res[element_split[0]] = element_split[1]
        return res

    @api.model
    def _sips_form_get_tx_from_data(self, data):
        """ Given a data dict coming from sips, verify it and find the related
        transaction record. """

        data = self._sips_data_to_object(data.get('Data'))
        reference = data.get('transactionReference')

        if not reference:
            custom = json.loads(data.pop('returnContext', False) or '{}')
            reference = custom.get('reference')

        payment_tx = self.search([('reference', '=', reference)])
        if not payment_tx or len(payment_tx) > 1:
            error_msg = _('Sips: received data for reference %s') % reference
            if not payment_tx:
                error_msg += _('; no order found')
            else:
                error_msg += _('; multiple order found')
            _logger.error(error_msg)
            raise ValidationError(error_msg)
        return payment_tx

    def _sips_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        data = self._sips_data_to_object(data.get('Data'))

        # TODO: txn_id: should be false at draft, set afterwards, and verified with txn details
        if self.acquirer_reference and data.get('transactionReference') != self.acquirer_reference:
            invalid_parameters.append(('transactionReference', data.get('transactionReference'), self.acquirer_reference))
        # check what is bought
        if float_compare(float(data.get('amount', '0.0')) / 100, self.amount, 2) != 0:
            invalid_parameters.append(('amount', data.get('amount'), '%.2f' % self.amount))

        return invalid_parameters

    def _sips_form_validate(self, data):
        data = self._sips_data_to_object(data.get('Data'))
        status = data.get('responseCode')
        date = data.get('transactionDateTime')
        if date:
            try:
                # dateutil.parser 2.5.3 and up should handle dates formatted as
                # '2020-04-08T05:54:18+02:00', which strptime does not
                # (+02:00 does not work as %z expects +0200 before Python 3.7)
                # See odoo/odoo#49160
                date = parser.parse(date).astimezone(pytz.utc).replace(tzinfo=None)
            except:
                # will fallback on now in the write to avoid failing to
                # register the payment because a provider formats their 
                # dates badly or because some local library is not behaving
                date = False
        data = {
            'acquirer_reference': data.get('transactionReference'),
            'date': date or fields.Datetime.now(),
        }
        res = False
        if status in self._sips_valid_tx_status:
            msg = 'Payment for tx ref: %s, got response [%s], set as done.' % \
                  (self.reference, status)
            _logger.info(msg)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_done()
            res = True
        elif status in self._sips_error_tx_status:
            msg = 'Payment for tx ref: %s, got response [%s], set as ' \
                  'error.' % (self.reference, status)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        elif status in self._sips_wait_tx_status:
            msg = 'Received wait status for payment ref: %s, got response ' \
                  '[%s], set as error.' % (self.reference, status)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        elif status in self._sips_refused_tx_status:
            msg = 'Received refused status for payment ref: %s, got response' \
                  ' [%s], set as error.' % (self.reference, status)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        elif status in self._sips_pending_tx_status:
            msg = 'Payment ref: %s, got response [%s] set as pending.' \
                  % (self.reference, status)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_pending()
        elif status in self._sips_cancel_tx_status:
            msg = 'Received notification for payment ref: %s, got response ' \
                  '[%s], set as cancel.' % (self.reference, status)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        else:
            msg = 'Received unrecognized status for payment ref: %s, got ' \
                  'response [%s], set as error.' % (self.reference, status)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()

        _logger.info(msg)
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 43h2.714v4.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H32.732c1.623-2.333 2.212-4.5 1.768-6.5h14.536v-3.212a.342.342 0 0 0-.339-.343h-14.79a15.663 15.663 0 0 0-1.215-2.742h16.344c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V43zm-3.79 12.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm-1.394-16.606C15 37.5 14.5 37.025 13.733 35.448c-.556-1.144-.908-2.315-1.026-3.6-.035-.377-.04-1.232-.01-1.572.127-1.427.479-2.634 1.124-3.861a9.868 9.868 0 0 1 4.994-4.507c.24-.096.63-.236.637-.229a.859.859 0 0 1-.071.132c-.633 1.046-1.176 3.203-1.275 5.059l-.018.353-.115.162a6.324 6.324 0 0 0-1.094 2.865 9.018 9.018 0 0 0 0 1.521 6.257 6.257 0 0 0 1.208 3.02c.836 1.112 2.034 1.88 3.332 2.14.426.086.53.095 1.082.094.445 0 .547-.005.751-.038a5.349 5.349 0 0 0 2.003-.723c.116-.07.23-.14.252-.158l.041-.031-.092-.039a15.055 15.055 0 0 1-1.08-.556 9.864 9.864 0 0 1-3.215-3.084 9.796 9.796 0 0 1-.613-9.633c.095-.196.236-.466.313-.6.152-.265.535-.852.586-.896.078-.07 1.26-.095 1.865-.041a9.844 9.844 0 0 1 8.65 7.201c.254.942.352 1.744.334 2.764a8.847 8.847 0 0 1-.301 2.261c-.715 2.785-2.654 5.138-5.278 6.405a9.795 9.795 0 0 1-3.547.946 16.36 16.36 0 0 1-1.405-.002 9.878 9.878 0 0 1-5.736-2.407zM27.92 32.69c.022-.079.065-.272.097-.43a6.412 6.412 0 0 0-.299-3.548c-.64-1.645-1.91-2.891-3.508-3.446a1.81 1.81 0 0 0-.196-.06c-.044 0-.192.682-.25 1.148-.029.237-.029 1.131 0 1.37.155 1.251.585 2.3 1.327 3.232.16.201.553.607.747.772.553.47 1.12.796 1.808 1.04.111.04.21.071.219.069a.487.487 0 0 0 .055-.147z"/><path id="e" d="M19.25 42h2.714v3.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H32.732c1.623-2.333 2.212-4.5 1.768-6.5h14.536v-3.212a.342.342 0 0 0-.339-.343h-14.79a15.663 15.663 0 0 0-1.215-2.742h16.344c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V42zm-3.79 11.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm-1.394-16.606C15 35.5 14.5 35.025 13.733 33.448c-.556-1.144-.908-2.315-1.026-3.6-.035-.377-.04-1.232-.01-1.572.127-1.427.479-2.634 1.124-3.861a9.868 9.868 0 0 1 4.994-4.507c.24-.096.63-.236.637-.229a.859.859 0 0 1-.071.132c-.633 1.046-1.176 3.203-1.275 5.059l-.018.353-.115.162a6.324 6.324 0 0 0-1.094 2.865 9.018 9.018 0 0 0 0 1.521 6.257 6.257 0 0 0 1.208 3.02c.836 1.112 2.034 1.88 3.332 2.14.426.086.53.095 1.082.094.445 0 .547-.005.751-.038a5.349 5.349 0 0 0 2.003-.723c.116-.07.23-.14.252-.158l.041-.031-.092-.039a15.055 15.055 0 0 1-1.08-.556 9.864 9.864 0 0 1-3.215-3.084 9.796 9.796 0 0 1-.613-9.633c.095-.196.236-.466.313-.6.152-.265.535-.852.586-.896.078-.07 1.26-.095 1.865-.041a9.844 9.844 0 0 1 8.65 7.201c.254.942.352 1.744.334 2.764a8.847 8.847 0 0 1-.301 2.261c-.715 2.785-2.654 5.138-5.278 6.405a9.795 9.795 0 0 1-3.547.946 16.36 16.36 0 0 1-1.405-.002 9.878 9.878 0 0 1-5.736-2.407zM27.92 30.69c.022-.079.065-.272.097-.43a6.412 6.412 0 0 0-.299-3.548c-.64-1.645-1.91-2.891-3.508-3.446a1.81 1.81 0 0 0-.196-.06c-.044 0-.192.682-.25 1.148-.029.237-.029 1.131 0 1.37.155 1.251.585 2.3 1.327 3.232.16.201.553.607.747.772.553.47 1.12.796 1.808 1.04.111.04.21.071.219.069a.487.487 0 0 0 .055-.147z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L16.034 21.5 18 21.197l3.6-1.697 7.4 3.203h8.25L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_sips_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <template id="sips_form">
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <input type="hidden" name="Data" t-att-value="Data"/>
            <input type="hidden" name="InterfaceVersion" t-att-value="InterfaceVersion"/>
            <input type="hidden" name="Seal" t-att-value="Seal"/>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="acquirer_form_sips" model="ir.ui.view">
            <field name="name">acquirer.form.sips</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//group[@name="acquirer"]' position='inside'>
                    <group attrs="{'invisible': [('provider', '!=', 'sips')]}">
                        <field name="sips_merchant_id" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}"/>
                        <field name="sips_secret" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}"/>
                        <field name="sips_test_url" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" groups='base.group_no_one'/>
                        <field name="sips_prod_url" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" groups='base.group_no_one'/>
                        <field name="sips_version" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" groups='base.group_no_one'/>
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

