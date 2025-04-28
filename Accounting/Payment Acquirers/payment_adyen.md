# Odoo Module: payment_adyen

Category: Accounting/Payment Acquirers

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
    reset_payment_provider(cr, registry, 'adyen')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Adyen Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 340,
    'summary': 'Payment Acquirer: Adyen Implementation',
    'version': '1.0',
    'description': """Adyen Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_adyen_templates.xml',
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

import json
import logging
import pprint
import werkzeug

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class AdyenController(http.Controller):
    _return_url = '/payment/adyen/return/'

    @http.route([
        '/payment/adyen/return',
    ], type='http', auth='public', csrf=False)
    def adyen_return(self, **post):
        _logger.info('Beginning Adyen form_feedback with post data %s', pprint.pformat(post))  # debug
        if post.get('authResult') not in ['CANCELLED']:
            request.env['payment.transaction'].sudo().form_feedback(post, 'adyen')
        return werkzeug.utils.redirect('/payment/process')

    @http.route([
        '/payment/adyen/notification',
    ], type='http', auth='public', methods=['POST'], csrf=False)
    def adyen_notification(self, **post):
        tx = post.get('merchantReference') and request.env['payment.transaction'].sudo().search([('reference', 'in', [post.get('merchantReference')])], limit=1)
        if post.get('eventCode') in ['AUTHORISATION'] and tx:
            states = (post.get('merchantReference'), post.get('success'), tx.state)
            if (post.get('success') == 'true' and tx.state == 'done') or (post.get('success') == 'false' and tx.state in ['cancel', 'error']):
                _logger.info('Notification from Adyen for the reference %s: received %s, state is %s', states)
            else:
                _logger.warning('Notification from Adyen for the reference %s: received %s but state is %s', states)
        return '[accepted]'

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

        <record id="payment.payment_acquirer_adyen" model="payment.acquirer">
            <field name="name">Adyen</field>
            <field name="image_128" type="base64" file="payment_adyen/static/src/img/adyen_icon.png"/>
            <field name="provider">adyen</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="adyen_form"/>
        </record>

    </data>
</odoo>

```

## File: models\payment.py

```python
# coding: utf-8

import base64
import json
import binascii
from collections import OrderedDict
import hashlib
import hmac
import logging
from itertools import chain

from werkzeug import urls

from odoo import api, fields, models, tools, _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.addons.payment_adyen.controllers.main import AdyenController
from odoo.tools.pycompat import to_text

_logger = logging.getLogger(__name__)

# https://docs.adyen.com/developers/development-resources/currency-codes
CURRENCY_CODE_MAPS = {
    "BHD": 3,
    "CVE": 0,
    "DJF": 0,
    "GNF": 0,
    "IDR": 0,
    "JOD": 3,
    "JPY": 0,
    "KMF": 0,
    "KRW": 0,
    "KWD": 3,
    "LYD": 3,
    "OMR": 3,
    "PYG": 0,
    "RWF": 0,
    "TND": 3,
    "UGX": 0,
    "VND": 0,
    "VUV": 0,
    "XAF": 0,
    "XOF": 0,
    "XPF": 0,
}


class AcquirerAdyen(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[
        ('adyen', 'Adyen')
    ], ondelete={'adyen': 'set default'})
    adyen_merchant_account = fields.Char('Merchant Account', required_if_provider='adyen', groups='base.group_user')
    adyen_skin_code = fields.Char('Skin Code', required_if_provider='adyen', groups='base.group_user')
    adyen_skin_hmac_key = fields.Char('Skin HMAC Key', required_if_provider='adyen', groups='base.group_user')

    @api.model
    def _adyen_convert_amount(self, amount, currency):
        """
        Adyen requires the amount to be multiplied by 10^k,
        where k depends on the currency code.
        """
        k = CURRENCY_CODE_MAPS.get(currency.name, 2)
        paymentAmount = int(tools.float_round(amount, k) * (10**k))
        return paymentAmount

    @api.model
    def _get_adyen_urls(self, environment):
        """ Adyen URLs: yhpp: hosted payment page: pay.shtml for single, select.shtml for multiple """
        return {
            'adyen_form_url': 'https://%s.adyen.com/hpp/pay.shtml' % ('live' if environment == 'prod' else environment),
        }

    def _adyen_generate_merchant_sig_sha256(self, inout, values):
        """ Generate the shasign for incoming or outgoing communications., when using the SHA-256
        signature.

        :param string inout: 'in' (odoo contacting adyen) or 'out' (adyen
                             contacting odoo). In this last case only some
                             fields should be contained (see e-Commerce basic)
        :param dict values: transaction values
        :return string: shasign
        """
        def escapeVal(val):
            return val.replace('\\', '\\\\').replace(':', '\\:')

        def signParams(parms):
            signing_string = ':'.join(
                escapeVal(v)
                for v in chain(parms.keys(), parms.values())
            )
            hm = hmac.new(hmac_key, signing_string.encode('utf-8'), hashlib.sha256)
            return base64.b64encode(hm.digest())

        assert inout in ('in', 'out')
        assert self.provider == 'adyen'

        if inout == 'in':
            # All the fields sent to Adyen must be included in the signature. ALL the fucking
            # fields, despite what is claimed in the documentation. For example, in
            # https://docs.adyen.com/developers/hpp-manual, it is stated: "The resURL parameter does
            # not need to be included in the signature." It's a trap, it must be included as well!
            keys = [
                'merchantReference', 'paymentAmount', 'currencyCode', 'shipBeforeDate', 'skinCode',
                'merchantAccount', 'sessionValidity', 'merchantReturnData', 'shopperEmail',
                'shopperReference', 'allowedMethods', 'blockedMethods', 'offset',
                'shopperStatement', 'recurringContract', 'billingAddressType',
                'deliveryAddressType', 'brandCode', 'countryCode', 'shopperLocale', 'orderData',
                'offerEmail', 'resURL',
            ]
        else:
            keys = [
                'authResult', 'merchantReference', 'merchantReturnData', 'paymentMethod',
                'pspReference', 'shopperLocale', 'skinCode',
            ]

        hmac_key = binascii.a2b_hex(self.adyen_skin_hmac_key.encode('ascii'))
        raw_values = {k: values.get(k, '') for k in keys if k in values}
        raw_values_ordered = OrderedDict(sorted(raw_values.items(), key=lambda t: t[0]))

        return signParams(raw_values_ordered)

    def _adyen_generate_merchant_sig(self, inout, values):
        """ Generate the shasign for incoming or outgoing communications, when using the SHA-1
        signature (deprecated by Adyen).

        :param string inout: 'in' (odoo contacting adyen) or 'out' (adyen
                             contacting odoo). In this last case only some
                             fields should be contained (see e-Commerce basic)
        :param dict values: transaction values

        :return string: shasign
        """
        assert inout in ('in', 'out')
        assert self.provider == 'adyen'

        if inout == 'in':
            keys = "paymentAmount currencyCode shipBeforeDate merchantReference skinCode merchantAccount sessionValidity shopperEmail shopperReference recurringContract allowedMethods blockedMethods shopperStatement merchantReturnData billingAddressType deliveryAddressType offset".split()
        else:
            keys = "authResult pspReference merchantReference skinCode merchantReturnData".split()

        def get_value(key):
            if values.get(key):
                return values[key]
            return ''

        sign = ''.join('%s' % get_value(k) for k in keys).encode('ascii')
        key = self.adyen_skin_hmac_key.encode('ascii')
        return base64.b64encode(hmac.new(key, sign, hashlib.sha1).digest())

    def adyen_form_generate_values(self, values):
        base_url = self.get_base_url()
        # tmp
        import datetime
        from dateutil import relativedelta

        paymentAmount = self._adyen_convert_amount(values['amount'], values['currency'])
        if self.provider == 'adyen' and len(self.adyen_skin_hmac_key) == 64:
            tmp_date = datetime.datetime.today() + relativedelta.relativedelta(days=1)

            values.update({
                'merchantReference': values['reference'],
                'paymentAmount': '%d' % paymentAmount,
                'currencyCode': values['currency'] and values['currency'].name or '',
                'shipBeforeDate': tmp_date.strftime('%Y-%m-%d'),
                'skinCode': self.adyen_skin_code,
                'merchantAccount': self.adyen_merchant_account,
                'shopperLocale': values.get('partner_lang', ''),
                'sessionValidity': tmp_date.isoformat('T')[:19] + "Z",
                'resURL': urls.url_join(base_url, AdyenController._return_url),
                'merchantReturnData': json.dumps({'return_url': '%s' % values.pop('return_url')}) if values.get('return_url', '') else False,
                'shopperEmail': values.get('partner_email') or values.get('billing_partner_email') or '',
            })
            values['merchantSig'] = self._adyen_generate_merchant_sig_sha256('in', values)

        else:
            tmp_date = datetime.date.today() + relativedelta.relativedelta(days=1)

            values.update({
                'merchantReference': values['reference'],
                'paymentAmount': '%d' % paymentAmount,
                'currencyCode': values['currency'] and values['currency'].name or '',
                'shipBeforeDate': tmp_date,
                'skinCode': self.adyen_skin_code,
                'merchantAccount': self.adyen_merchant_account,
                'shopperLocale': values.get('partner_lang'),
                'sessionValidity': tmp_date,
                'resURL': urls.url_join(base_url, AdyenController._return_url),
                'merchantReturnData': json.dumps({'return_url': '%s' % values.pop('return_url')}) if values.get('return_url') else False,
            })
            values['merchantSig'] = self._adyen_generate_merchant_sig('in', values)

        return values

    def adyen_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_adyen_urls(environment)['adyen_form_url']


class TxAdyen(models.Model):
    _inherit = 'payment.transaction'

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def _adyen_form_get_tx_from_data(self, data):
        reference, pspReference = data.get('merchantReference'), data.get('pspReference')
        if not reference or not pspReference:
            error_msg = _('Adyen: received data with missing reference (%s) or missing pspReference (%s)') % (reference, pspReference)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        # find tx -> @TDENOTE use pspReference ?
        tx = self.env['payment.transaction'].search([('reference', '=', reference)])
        if not tx or len(tx) > 1:
            error_msg = _('Adyen: received data for reference %s') % (reference)
            if not tx:
                error_msg += _('; no order found')
            else:
                error_msg += _('; multiple order found')
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        # verify shasign
        if len(tx.acquirer_id.adyen_skin_hmac_key) == 64:
            shasign_check = tx.acquirer_id._adyen_generate_merchant_sig_sha256('out', data)
        else:
            shasign_check = tx.acquirer_id._adyen_generate_merchant_sig('out', data)
        if to_text(shasign_check) != to_text(data.get('merchantSig')):
            error_msg = _('Adyen: invalid merchantSig, received %s, computed %s') % (data.get('merchantSig'), shasign_check)
            _logger.warning(error_msg)
            raise ValidationError(error_msg)

        return tx

    def _adyen_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        # reference at acquirer: pspReference
        if self.acquirer_reference and data.get('pspReference') != self.acquirer_reference:
            invalid_parameters.append(('pspReference', data.get('pspReference'), self.acquirer_reference))
        # seller
        if data.get('skinCode') != self.acquirer_id.adyen_skin_code:
            invalid_parameters.append(('skinCode', data.get('skinCode'), self.acquirer_id.adyen_skin_code))
        # result
        if not data.get('authResult'):
            invalid_parameters.append(('authResult', data.get('authResult'), 'something'))

        return invalid_parameters

    def _adyen_form_validate(self, data):
        status = data.get('authResult', 'PENDING')
        if status == 'AUTHORISED':
            self.write({'acquirer_reference': data.get('pspReference')})
            self._set_transaction_done()
            return True
        elif status == 'PENDING':
            self.write({'acquirer_reference': data.get('pspReference')})
            self._set_transaction_pending()
            return True
        else:
            error = _('Adyen: feedback error')
            _logger.info(error)
            self.write({'state_message': error})
            self._set_transaction_cancel()
            return False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M19.25 45h2.714v2.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H38.5V31h10.536v-3.212a.342.342 0 0 0-.339-.343H38.5v-2.742h10.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V45zm-3.79 10.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm13.58-34.853a4.263 4.263 0 0 1 4.281 4.28v17.004H18.281A4.263 4.263 0 0 1 14 37.151v-4.757a4.263 4.263 0 0 1 4.28-4.28h4.638v6.777c0 .595.476 1.19 1.19 1.19h2.377v-9.394c0-.595-.475-1.19-1.189-1.19h-10.94v-5.35h16.648z"/><path id="e" d="M19.25 43h2.714v2.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H38.5V29h10.536v-3.212a.342.342 0 0 0-.339-.343H38.5v-2.742h10.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V43zm-3.79 10.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm13.58-34.853a4.263 4.263 0 0 1 4.281 4.28v17.004H18.281A4.263 4.263 0 0 1 14 35.151v-4.757a4.263 4.263 0 0 1 4.28-4.28h4.638v6.777c0 .595.476 1.19 1.19 1.19h2.377v-9.394c0-.595-.475-1.19-1.189-1.19h-10.94v-5.35h16.648z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L14.423 18.5 35 25.5l3.5-2.797L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_adyen_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="adyen_form">
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <input type="hidden" name="merchantReference" t-att-value="merchantReference"/>
            <input type="hidden" name="paymentAmount" t-att-value="paymentAmount"/>
            <input type="hidden" name="currencyCode" t-att-value="currencyCode"/>
            <input type="hidden" name="shipBeforeDate" t-att-value="shipBeforeDate"/>
            <input type="hidden" name="skinCode" t-att-value="skinCode"/>
            <input type="hidden" name="merchantAccount" t-att-value="merchantAccount"/>
            <input type="hidden" name="shopperLocale" t-att-value="shopperLocale"/>
            <input type="hidden" name="sessionValidity" t-att-value="sessionValidity"/>
            <input type="hidden" name="merchantSig" t-att-value="merchantSig"/>
            <!-- URLs -->
            <input type="hidden" name='resURL'
                t-att-value="resURL"/>
            <!-- custom -->
            <input t-if="merchantReturnData" type="hidden" name='merchantReturnData'
                t-att-value="merchantReturnData"/>
            <!-- shopperEmail is not included for SHA-1. To avoid breaking compatibility,
                    include only if filled in -->
            <input t-if="shopperEmail"
                type="hidden" name='shopperEmail' t-att-value="shopperEmail"/>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="acquirer_form_adyen" model="ir.ui.view">
            <field name="name">acquirer.form.adyen</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//group[@name="acquirer"]' position='inside'>
                    <group attrs="{'invisible': [('provider', '!=', 'adyen')]}">
                        <field name="adyen_merchant_account" attrs="{'required':[('provider', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                        <field name="adyen_skin_code" attrs="{'required':[('provider', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                        <field name="adyen_skin_hmac_key" attrs="{'required':[('provider', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                        <a colspan="2" href="https://www.adyen.com/home/payment-services/online-payments" target="_blank">How to configure your Adyen account?</a>
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

