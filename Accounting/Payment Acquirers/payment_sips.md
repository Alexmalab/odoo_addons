# Odoo Module: payment_sips

Category: Accounting/Payment Acquirers

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
    'version': '1.1',
    'author': 'Eezee-It',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 385,
    'description': """
Worldline SIPS Payment Acquirer for online payments

Implements the Worldline SIPS API for payment acquirers.
Other SIPS providers may be compatible, though this is
not guaranteed.""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_sips_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'installable': True,
    'application': True,
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

    @http.route('/payment/sips/ipn/', type='http', auth='public', methods=['POST'], csrf=False)
    def sips_ipn(self, **post):
        """ Sips IPN. """
        _logger.info('Beginning Sips IPN form_feedback with post data %s', pprint.pformat(post))  # debug
        if not post:
            # SIPS sometimes sends empty notifications, the reason why is
            # unclear but they tend to pollute logs and do not provide any
            # meaningful information; log as a warning instead of a traceback
            _logger.warning('Sips: received empty notification; skip.')
        else:
            self.sips_validate_data(**post)
        return ''

    @http.route('/payment/sips/dpn', type='http', auth="public", methods=['POST'], csrf=False, save_session=False)
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

## File: models\const.py

```python

from collections import namedtuple

Currency = namedtuple('Currency', ['iso_id', 'decimal'])

# ISO 4217 Data for currencies supported by sips
# NOTE: these are listed on the Atos Wordline SIPS POST documentation page
# at https://documentation.sips.worldline.com/en/WLSIPS.001-GD-Data-dictionary.html#Sips.001_DD_en-Value-currencyCode
# Yet with the simu environment, some of these currencies are *not* working
# I have no way to know if this is caused by the SIMU environment, or if it's
# the doc of SIPS that lists currencies that don't work, but since this list is
# restrictive, I'm gonna assume they are supported when using the right flow
# and payment methods, which may not work in SIMU...
# Since SIPS advises to use 'in production', well...
SIPS_SUPPORTED_CURRENCIES = {
    'ARS': Currency('032', 2),
    'AUD': Currency('036', 2),
    'BHD': Currency('048', 3),
    'KHR': Currency('116', 2),
    'CAD': Currency('124', 2),
    'LKR': Currency('144', 2),
    'CNY': Currency('156', 2),
    'HRK': Currency('191', 2),
    'CZK': Currency('203', 2),
    'DKK': Currency('208', 2),
    'HKD': Currency('344', 2),
    'HUF': Currency('348', 2),
    'ISK': Currency('352', 0),
    'INR': Currency('356', 2),
    'ILS': Currency('376', 2),
    'JPY': Currency('392', 0),
    'KRW': Currency('410', 0),
    'KWD': Currency('414', 3),
    'MYR': Currency('458', 2),
    'MUR': Currency('480', 2),
    'MXN': Currency('484', 2),
    'NPR': Currency('524', 2),
    'NZD': Currency('554', 2),
    'NOK': Currency('578', 2),
    'QAR': Currency('634', 2),
    'RUB': Currency('643', 2),
    'SAR': Currency('682', 2),
    'SGD': Currency('702', 2),
    'ZAR': Currency('710', 2),
    'SEK': Currency('752', 2),
    'CHF': Currency('756', 2),
    'THB': Currency('764', 2),
    'AED': Currency('784', 2),
    'TND': Currency('788', 3),
    'GBP': Currency('826', 2),
    'USD': Currency('840', 2),
    'TWD': Currency('901', 2),
    'RSD': Currency('941', 2),
    'RON': Currency('946', 2),
    'TRY': Currency('949', 2),
    'XOF': Currency('952', 0),
    'XPF': Currency('953', 0),
    'BGN': Currency('975', 2),
    'EUR': Currency('978', 2),
    'UAH': Currency('980', 2),
    'PLN': Currency('985', 2),
    'BRL': Currency('986', 2),
}

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

from .const import SIPS_SUPPORTED_CURRENCIES

_logger = logging.getLogger(__name__)


class AcquirerSips(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('sips', 'Sips')], ondelete={'sips': 'set default'})
    sips_merchant_id = fields.Char('Merchant ID', required_if_provider='sips', groups='base.group_user')
    sips_secret = fields.Char('Secret Key', size=64, required_if_provider='sips', groups='base.group_user')
    sips_test_url = fields.Char("Test url", required_if_provider='sips', default='https://payment-webinit.simu.sips-services.com/paymentInit')
    sips_prod_url = fields.Char("Production url", required_if_provider='sips', default='https://payment-webinit.sips-services.com/paymentInit')
    sips_version = fields.Char("Interface Version", required_if_provider='sips', default='HP_2.31')
    sips_key_version = fields.Integer("Secret Key Version", required_if_provider='sips', default=2)

    def _sips_generate_shasign(self, values):
        """ Generate the shasign for incoming or outgoing communications.
        :param dict values: transaction values
        :return string: shasign
        """
        if self.provider != 'sips':
            raise ValidationError(_('Incorrect payment acquirer provider'))
        data = values['Data']
        key = self.sips_secret

        shasign = sha256((data + key).encode('utf-8'))
        return shasign.hexdigest()

    def sips_form_generate_values(self, values):
        self.ensure_one()
        base_url = self.get_base_url()
        currency = self.env['res.currency'].sudo().browse(values['currency_id'])
        sips_currency = SIPS_SUPPORTED_CURRENCIES.get(currency.name)
        if not sips_currency:
            raise ValidationError(_('Currency not supported by Wordline: %s') % currency.name)
        # rounded to its smallest unit, depends on the currency
        amount = round(values['amount'] * (10 ** sips_currency.decimal))

        sips_tx_values = dict(values)
        data = {
            'amount': amount,
            'currencyCode': sips_currency.iso_id,
            'merchantId': self.sips_merchant_id,
            'normalReturnUrl': urls.url_join(base_url, SipsController._return_url),
            'automaticResponseUrl': urls.url_join(base_url, SipsController._notify_url),
            'transactionReference': values['reference'],
            'statementReference': values['reference'],
            'keyVersion': self.sips_key_version,
        }
        sips_tx_values.update({
            'Data': '|'.join([f'{k}={v}' for k,v in data.items()]),
            'InterfaceVersion': self.sips_version,
        })

        return_context = {}
        if sips_tx_values.get('return_url'):
            return_context['return_url'] = urls.url_quote(sips_tx_values.get('return_url'))
        return_context['reference'] = sips_tx_values['reference']
        sips_tx_values['Data'] += '|returnContext=%s' % (json.dumps(return_context))

        shasign = self._sips_generate_shasign(sips_tx_values)
        sips_tx_values['Seal'] = shasign
        return sips_tx_values

    def sips_get_form_action_url(self):
        self.ensure_one()
        return self.sips_prod_url if self.state == 'enabled' else self.sips_test_url


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
        res = super()._compute_reference(values=values, prefix=prefix)
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
            (key, value) = element.split('=')
            res[key] = value
        return res

    @api.model
    def _sips_form_get_tx_from_data(self, data):
        """ Given a data dict coming from sips, verify it and find the related
        transaction record. """

        data = self._sips_data_to_object(data.get('Data'))
        reference = data.get('transactionReference')

        if not reference:
            return_context = json.loads(data.get('returnContext', '{}'))
            reference = return_context.get('reference')

        payment_tx = self.search([('reference', '=', reference)])
        if not payment_tx:
            error_msg = _('Sips: received data for reference %s; no order found') % reference
            _logger.error(error_msg)
            raise ValidationError(error_msg)
        return payment_tx

    def _sips_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        data = self._sips_data_to_object(data.get('Data'))

        # amounts should match
        # get currency decimals from const
        sips_currency = SIPS_SUPPORTED_CURRENCIES.get(self.currency_id.name)
        # convert from int to float using decimals from currency
        amount_converted = float(data.get('amount', '0.0')) / (10 ** sips_currency.decimal)
        if float_compare(amount_converted, self.amount, sips_currency.decimal) != 0:
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
                # fallback on now to avoid failing to register the payment
                # because a provider formats their dates badly or because
                # some library is not behaving
                date = fields.Datetime.now()
        data = {
            'acquirer_reference': data.get('transactionReference'),
            'date': date,
        }
        res = False
        if status in self._sips_valid_tx_status:
            msg = f'ref: {self.reference}, got valid response [{status}], set as done.'
            _logger.info(msg)
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_done()
            res = True
        elif status in self._sips_error_tx_status:
            msg = f'ref: {self.reference}, got response [{status}], set as cancel.'
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        elif status in self._sips_wait_tx_status:
            msg = f'ref: {self.reference}, got wait response [{status}], set as cancel.'
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        elif status in self._sips_refused_tx_status:
            msg = f'ref: {self.reference}, got refused response [{status}], set as cancel.'
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        elif status in self._sips_pending_tx_status:
            msg = f'ref: {self.reference}, got pending response [{status}], set as pending.'
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_pending()
        elif status in self._sips_cancel_tx_status:
            msg = f'ref: {self.reference}, got cancel response [{status}], set as cancel.'
            data.update(state_message=msg)
            self.write(data)
            self._set_transaction_cancel()
        else:
            msg = f'ref: {self.reference}, got unrecognized response [{status}], set as cancel.'
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
                        <field name="sips_version" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" />
                        <field name="sips_key_version" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" />
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

