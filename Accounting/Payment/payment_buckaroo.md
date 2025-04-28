# Odoo Module: payment_buckaroo

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
    reset_payment_provider(cr, registry, 'buckaroo')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Buckaroo Payment Acquirer',
    'category': 'Accounting/Payment',
    'summary': 'Payment Acquirer: Buckaroo Implementation',
    'version': '1.0',
    'description': """Buckaroo Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_buckaroo_templates.xml',
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

import logging
import pprint
import werkzeug

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class BuckarooController(http.Controller):
    _return_url = '/payment/buckaroo/return'
    _cancel_url = '/payment/buckaroo/cancel'
    _exception_url = '/payment/buckaroo/error'
    _reject_url = '/payment/buckaroo/reject'

    @http.route([
        '/payment/buckaroo/return',
        '/payment/buckaroo/cancel',
        '/payment/buckaroo/error',
        '/payment/buckaroo/reject',
    ], type='http', auth='public', csrf=False)
    def buckaroo_return(self, **post):
        """ Buckaroo."""
        _logger.info('Buckaroo: entering form_feedback with post data %s', pprint.pformat(post))  # debug
        request.env['payment.transaction'].sudo().form_feedback(post, 'buckaroo')
        post = {key.upper(): value for key, value in post.items()}
        return_url = post.get('ADD_RETURNDATA') or '/'
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

        <record id="payment.payment_acquirer_buckaroo" model="payment.acquirer">
            <field name="name">Buckaroo</field>
            <field name="image_128" type="base64" file="payment_buckaroo/static/src/img/buckaroo_icon.png"/>
            <field name="provider">buckaroo</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="buckaroo_form"/>
        </record>

    </data>
</odoo>

```

## File: models\payment.py

```python
# coding: utf-8
from hashlib import sha1
import logging

from werkzeug import urls

from odoo import api, fields, models, _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.addons.payment_buckaroo.controllers.main import BuckarooController

from odoo.tools.float_utils import float_compare

_logger = logging.getLogger(__name__)


def normalize_keys_upper(data):
    """Set all keys of a dictionnary to uppercase

    Buckaroo parameters names are case insensitive
    convert everything to upper case to be able to easily detected the presence
    of a parameter by checking the uppercase key only
    """
    return {key.upper(): val for key, val in data.items()}


class AcquirerBuckaroo(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('buckaroo', 'Buckaroo')])
    brq_websitekey = fields.Char('WebsiteKey', required_if_provider='buckaroo', groups='base.group_user')
    brq_secretkey = fields.Char('SecretKey', required_if_provider='buckaroo', groups='base.group_user')

    def _get_buckaroo_urls(self, environment):
        """ Buckaroo URLs
        """
        if environment == 'prod':
            return {
                'buckaroo_form_url': 'https://checkout.buckaroo.nl/html/',
            }
        else:
            return {
                'buckaroo_form_url': 'https://testcheckout.buckaroo.nl/html/',
            }

    def _buckaroo_generate_digital_sign(self, inout, values):
        """ Generate the shasign for incoming or outgoing communications.

        :param browse acquirer: the payment.acquirer browse record. It should
                                have a shakey in shaky out
        :param string inout: 'in' (odoo contacting buckaroo) or 'out' (buckaroo
                             contacting odoo).
        :param dict values: transaction values

        :return string: shasign
        """
        assert inout in ('in', 'out')
        assert self.provider == 'buckaroo'

        keys = "add_returndata Brq_amount Brq_culture Brq_currency Brq_invoicenumber Brq_return Brq_returncancel Brq_returnerror Brq_returnreject brq_test Brq_websitekey".split()

        def get_value(key):
            if values.get(key):
                return values[key]
            return ''

        values = dict(values or {})

        if inout == 'out':
            for key in list(values):
                # case insensitive keys
                if key.upper() == 'BRQ_SIGNATURE':
                    del values[key]
                    break

            items = sorted(values.items(), key=lambda pair: pair[0].lower())
            sign = ''.join('%s=%s' % (k, urls.url_unquote_plus(v)) for k, v in items)
        else:
            sign = ''.join('%s=%s' % (k, get_value(k)) for k in keys)
        # Add the pre-shared secret key at the end of the signature
        sign = sign + self.brq_secretkey
        shasign = sha1(sign.encode('utf-8')).hexdigest()
        return shasign

    def buckaroo_form_generate_values(self, values):
        base_url = self.get_base_url()
        buckaroo_tx_values = dict(values)
        buckaroo_tx_values.update({
            'Brq_websitekey': self.brq_websitekey,
            'Brq_amount': values['amount'],
            'Brq_currency': values['currency'] and values['currency'].name or '',
            'Brq_invoicenumber': values['reference'],
            'brq_test': True if self.state == 'test' else False,
            'Brq_return': urls.url_join(base_url, BuckarooController._return_url),
            'Brq_returncancel': urls.url_join(base_url, BuckarooController._cancel_url),
            'Brq_returnerror': urls.url_join(base_url, BuckarooController._exception_url),
            'Brq_returnreject': urls.url_join(base_url, BuckarooController._reject_url),
            'Brq_culture': (values.get('partner_lang') or 'en_US').replace('_', '-'),
            'add_returndata': buckaroo_tx_values.pop('return_url', '') or '',
        })
        buckaroo_tx_values['Brq_signature'] = self._buckaroo_generate_digital_sign('in', buckaroo_tx_values)
        return buckaroo_tx_values

    def buckaroo_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_buckaroo_urls(environment)['buckaroo_form_url']


class TxBuckaroo(models.Model):
    _inherit = 'payment.transaction'

    # buckaroo status
    _buckaroo_valid_tx_status = [190]
    _buckaroo_pending_tx_status = [790, 791, 792, 793]
    _buckaroo_cancel_tx_status = [890, 891]
    _buckaroo_error_tx_status = [490, 491, 492]
    _buckaroo_reject_tx_status = [690]

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def _buckaroo_form_get_tx_from_data(self, data):
        """ Given a data dict coming from buckaroo, verify it and find the related
        transaction record. """
        origin_data = dict(data)
        data = normalize_keys_upper(data)
        reference, pay_id, shasign = data.get('BRQ_INVOICENUMBER'), data.get('BRQ_PAYMENT'), data.get('BRQ_SIGNATURE')
        if not reference or not pay_id or not shasign:
            error_msg = _('Buckaroo: received data with missing reference (%s) or pay_id (%s) or shasign (%s)') % (reference, pay_id, shasign)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        tx = self.search([('reference', '=', reference)])
        if not tx or len(tx) > 1:
            error_msg = _('Buckaroo: received data for reference %s') % (reference)
            if not tx:
                error_msg += _('; no order found')
            else:
                error_msg += _('; multiple order found')
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        # verify shasign
        shasign_check = tx.acquirer_id._buckaroo_generate_digital_sign('out', origin_data)
        if shasign_check.upper() != shasign.upper():
            error_msg = _('Buckaroo: invalid shasign, received %s, computed %s, for data %s') % (shasign, shasign_check, data)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        return tx

    def _buckaroo_form_get_invalid_parameters(self, data):
        invalid_parameters = []
        data = normalize_keys_upper(data)
        if self.acquirer_reference and data.get('BRQ_TRANSACTIONS') != self.acquirer_reference:
            invalid_parameters.append(('Transaction Id', data.get('BRQ_TRANSACTIONS'), self.acquirer_reference))
        # check what is buyed
        if float_compare(float(data.get('BRQ_AMOUNT', '0.0')), self.amount, 2) != 0:
            invalid_parameters.append(('Amount', data.get('BRQ_AMOUNT'), '%.2f' % self.amount))
        if data.get('BRQ_CURRENCY') != self.currency_id.name:
            invalid_parameters.append(('Currency', data.get('BRQ_CURRENCY'), self.currency_id.name))

        return invalid_parameters

    def _buckaroo_form_validate(self, data):
        data = normalize_keys_upper(data)
        status_code = int(data.get('BRQ_STATUSCODE', '0'))
        if status_code in self._buckaroo_valid_tx_status:
            self.write({'acquirer_reference': data.get('BRQ_TRANSACTIONS')})
            self._set_transaction_done()
            return True
        elif status_code in self._buckaroo_pending_tx_status:
            self.write({'acquirer_reference': data.get('BRQ_TRANSACTIONS')})
            self._set_transaction_pending()
            return True
        elif status_code in self._buckaroo_cancel_tx_status:
            self.write({'acquirer_reference': data.get('BRQ_TRANSACTIONS')})
            self._set_transaction_cancel()
            return True
        else:
            error = 'Buckaroo: feedback error'
            _logger.info(error)
            self.write({
                'state_message': error,
                'acquirer_reference': data.get('BRQ_TRANSACTIONS'),
            })
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CDC484"/><stop offset="100%" stop-color="#B5AA59"/></linearGradient><path id="d" d="M31.757 37.5L34 31h15.036v-3.212a.342.342 0 0 0-.339-.343H35l.92-2.742h13.116c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V37.5H31.757zM15.46 55.642h-1.036V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zM13 19h3.688l2.517 7.42h8.184L29.994 19H33l-8.76 26h-1.97L13 19zm7 11l2.964 9L26 30h-6z"/><path id="e" d="M31.757 35.5L34 29h15.036v-3.212a.342.342 0 0 0-.339-.343H35l.92-2.742h13.116c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V35.5H31.757zM15.46 53.642h-1.036V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zM13 17h3.688l2.517 7.42h8.184L29.994 17H33l-8.76 26h-1.97L13 17zm7 11l2.964 9L26 28h-6z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916l13.5-16.675L18.889 24l11.172-7 .967 5.703 4.533.036L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_buckaroo_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="buckaroo_form">
            <div>
                <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
                <input type="hidden" name="Brq_websitekey" t-att-value="Brq_websitekey"/>
                <input type="hidden" name="Brq_amount" t-att-value="Brq_amount"/>
                <input type="hidden" name="Brq_currency" t-att-value="Brq_currency"/>
                <input type="hidden" name="Brq_invoicenumber" t-att-value="Brq_invoicenumber"/>
                <input type="hidden" name="Brq_signature" t-att-value="Brq_signature"/>
                <input type="hidden" name="brq_test" t-att-value="brq_test"/>
                <input type="hidden" name="Brq_culture" t-att-value="Brq_culture"/>
                <!-- URLs -->
                <input t-if="Brq_return" type="hidden" name='Brq_return'
                    t-att-value="Brq_return"/>
                <input t-if="Brq_returncancel" type="hidden" name='Brq_returncancel'
                    t-att-value="Brq_returncancel"/>
                <input t-if="Brq_returnerror" type="hidden" name='Brq_returnerror'
                    t-att-value="Brq_returnerror"/>
                <input t-if="Brq_returnreject" type="hidden" name='Brq_returnreject'
                    t-att-value="Brq_returnreject"/>
                <input type="hidden" name='add_returndata' t-att-value="add_returndata"/>
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

        <record id="acquirer_form_buckaroo" model="ir.ui.view">
            <field name="name">acquirer.form.buckaroo</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//group[@name="acquirer"]' position='inside'>
                    <group attrs="{'invisible': [('provider', '!=', 'buckaroo')]}">
                        <field name="brq_websitekey" attrs="{'required':[ ('provider', '=', 'buckaroo'), ('state', '!=', 'disabled')]}"/>
                        <field name="brq_secretkey" attrs="{'required':[ ('provider', '=', 'buckaroo'), ('state', '!=', 'disabled')]}"/>
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

