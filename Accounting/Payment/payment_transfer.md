# Odoo Module: payment_transfer

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
    reset_payment_provider(cr, registry, 'transfer')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Transfer Payment Acquirer',
    'category': 'Accounting/Payment',
    'summary': 'Payment Acquirer: Transfer Implementation',
    'version': '1.0',
    'description': """Transfer Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_transfer_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'installable': True,
    'auto_install': True,
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


class TransferController(http.Controller):
    _accept_url = '/payment/transfer/feedback'

    @http.route([
        '/payment/transfer/feedback',
    ], type='http', auth='public', csrf=False)
    def transfer_form_feedback(self, **post):
        _logger.info('Beginning form_feedback with post data %s', pprint.pformat(post))  # debug
        request.env['payment.transaction'].sudo().form_feedback(post, 'transfer')
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

        <record id="payment.payment_acquirer_transfer" model="payment.acquirer">
            <field name="name">Wire Transfer</field>
            <field name="image_128" type="base64" file="payment_transfer/static/src/img/transfer_icon.png"/>
            <field name="provider">transfer</field>
            <field name="state">enabled</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="transfer_form"/>
            <field name="pending_msg">
              <![CDATA[
              <h3>Please make a payment to: </h3>
              <ul>
                <li>Bank:&nbsp;</li>
                <li>Account Number:</li>
                <li>Account Holder: </li>
              </ul>]]>
            </field>
        </record>
    </data>
</odoo>

```

## File: models\payment.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.tools.float_utils import float_compare

import logging
import pprint

_logger = logging.getLogger(__name__)


class TransferPaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('transfer', 'Manual Payment')], default='transfer')

    @api.model
    def _create_missing_journal_for_acquirers(self, company=None):
        # By default, the wire transfer method uses the default Bank journal.
        company = company or self.env.company
        acquirers = self.env['payment.acquirer'].search(
            [('provider', '=', 'transfer'), ('journal_id', '=', False), ('company_id', '=', company.id)])

        bank_journal = self.env['account.journal'].search(
            [('type', '=', 'bank'), ('company_id', '=', company.id)], limit=1)
        if bank_journal:
            acquirers.write({'journal_id': bank_journal.id})
        return super(TransferPaymentAcquirer, self)._create_missing_journal_for_acquirers(company=company)

    def transfer_get_form_action_url(self):
        return '/payment/transfer/feedback'

    def _format_transfer_data(self):
        company_id = self.env.company.id
        # filter only bank accounts marked as visible
        journals = self.env['account.journal'].search([('type', '=', 'bank'), ('company_id', '=', company_id)])
        accounts = journals.mapped('bank_account_id').name_get()
        bank_title = _('Bank Accounts') if len(accounts) > 1 else _('Bank Account')
        bank_accounts = ''.join(['<ul>'] + ['<li>%s</li>' % name for id, name in accounts] + ['</ul>'])
        post_msg = _('''<div>
<h3>Please use the following transfer details</h3>
<h4>%(bank_title)s</h4>
%(bank_accounts)s
<h4>Communication</h4>
<p>Please use the order name as communication reference.</p>
</div>''') % {
            'bank_title': bank_title,
            'bank_accounts': bank_accounts,
        }
        return post_msg

    @api.model
    def create(self, values):
        """ Hook in create to create a default pending_msg. This is done in create
        to have access to the name and other creation values. If no pending_msg
        or a void pending_msg is given at creation, generate a default one. """
        if values.get('provider') == 'transfer' and not values.get('pending_msg'):
            values['pending_msg'] = self._format_transfer_data()
        return super(TransferPaymentAcquirer, self).create(values)

    def write(self, values):
        """ Hook in write to create a default pending_msg. See create(). """
        if not values.get('pending_msg', False) and all(not acquirer.pending_msg and acquirer.provider != 'transfer' for acquirer in self) and values.get('provider') == 'transfer':
            values['pending_msg'] = self._format_transfer_data()
        return super(TransferPaymentAcquirer, self).write(values)


class TransferPaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _transfer_form_get_tx_from_data(self, data):
        reference, amount, currency_name = data.get('reference'), data.get('amount'), data.get('currency_name')
        tx = self.search([('reference', '=', reference)])

        if not tx or len(tx) > 1:
            error_msg = _('received data for reference %s') % (pprint.pformat(reference))
            if not tx:
                error_msg += _('; no order found')
            else:
                error_msg += _('; multiple order found')
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        return tx

    def _transfer_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        if float_compare(float(data.get('amount') or '0.0'), self.amount, 2) != 0:
            invalid_parameters.append(('amount', data.get('amount'), '%.2f' % self.amount))
        if data.get('currency') != self.currency_id.name:
            invalid_parameters.append(('currency', data.get('currency'), self.currency_id.name))

        return invalid_parameters

    def _transfer_form_validate(self, data):
        _logger.info('Validated transfer payment for tx %s: set as pending' % (self.reference))
        self._set_transaction_pending()
        return True

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M19.25 33.173c1.733.551 3.65.827 5.75.827 4 0 7-1 9-3h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.514-1.416.5-2.742h13.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V33.173zm29.447 14.382a.342.342 0 0 0 .339-.343V37.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zm-9.242-18.427a.938.938 0 0 1 0 1.42L24.8 30.772c-.6.52-1.55.1-1.55-.71v-3.434c-6.058.087-8.67 1.591-6.898 7.256.197.629-.563 1.116-1.097.728C13.546 33.369 12 30.99 12 28.59c0-5.946 4.975-7.204 11.25-7.276v-3.127c0-.807.948-1.23 1.55-.71l6.875 5.937z"/><path id="e" d="M19.25 31.173c1.733.551 3.65.827 5.75.827 4 0 7-1 9-3h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.514-1.416.5-2.742h13.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V31.173zm29.447 14.382a.342.342 0 0 0 .339-.343V35.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zm-9.242-18.427a.938.938 0 0 1 0 1.42L24.8 28.772c-.6.52-1.55.1-1.55-.71v-3.434c-6.058.087-8.67 1.591-6.898 7.256.197.629-.563 1.116-1.097.728C13.546 31.369 12 28.99 12 26.59c0-5.946 4.975-7.204 11.25-7.276v-3.127c0-.807.948-1.23 1.55-.71l6.875 5.937z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L13 23l4.914-2.142 5.764-5.35L32 22l3.561.74L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_transfer_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="transfer_form">
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <t t-if="return_url">
                <input type="hidden" name='return_url' t-att-value='return_url'/>
            </t>
            <input type="hidden" name='reference' t-att-value='reference'/>
            <input type="hidden" name='amount' t-att-value='amount'/>
            <input type="hidden" name='currency' t-att-value='currency.name'/>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_acquirer_view_form_inherit_transfer" model="ir.ui.view">
        <field name="name">payment.acquirer.view.form.inherit.transfer</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.acquirer_form"/>
        <field name="arch" type="xml">
            <page name="acquirer_credentials" position="attributes">
                <attribute name="attrs">{'invisible': [('provider', 'in', ['manual', 'transfer'])]}</attribute>
            </page>
            <field name="pre_msg" position="attributes">
                <attribute name="attrs">{'invisible': [('provider', '=', 'transfer')]}</attribute>
            </field>
            <field name="done_msg" position="attributes">
                <attribute name="attrs">{'invisible': [('provider', '=', 'transfer')]}</attribute>
            </field>
            <field name="cancel_msg" position="attributes">
                <attribute name="attrs">{'invisible': [('provider', '=', 'transfer')]}</attribute>
            </field>
        </field>
    </record>
</odoo>

```

