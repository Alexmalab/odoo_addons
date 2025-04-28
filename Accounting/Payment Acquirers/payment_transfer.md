# Odoo Module: payment_transfer

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'transfer')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Wire Transfer Payment Acquirer',
    'version': '2.0',
    'category': 'Accounting/Payment Acquirers',
    'summary': 'Payment Acquirer: Wire Transfer Implementation',
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_transfer_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'auto_install': True,
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class TransferController(http.Controller):
    _accept_url = '/payment/transfer/feedback'

    @http.route(_accept_url, type='http', auth='public', methods=['POST'], csrf=False)
    def transfer_form_feedback(self, **post):
        _logger.info("beginning _handle_feedback_data with post data %s", pprint.pformat(post))
        request.env['payment.transaction'].sudo()._handle_feedback_data('transfer', post)
        return request.redirect('/payment/status')

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

    <record id="payment.payment_acquirer_transfer" model="payment.acquirer">
        <field name="provider">transfer</field>
        <field name="state">enabled</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">False</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">False</field>
        <!-- Clear the default value to trigger the computation of the message -->
        <field name="pending_msg" eval="False"/>
    </record>

</odoo>

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
        selection_add=[('transfer', "Wire Transfer")], default='transfer',
        ondelete={'transfer': 'set default'})
    qr_code = fields.Boolean(
        string="Enable QR Codes", help="Enable the use of QR-codes when paying by wire transfer.")

    @api.depends('provider')
    def _compute_view_configuration_fields(self):
        """ Override of payment to hide the credentials page.

        :return: None
        """
        super()._compute_view_configuration_fields()
        self.filtered(lambda acq: acq.provider == 'transfer').write({
            'show_credentials_page': False,
            'show_payment_icon_ids': False,
            'show_pre_msg': False,
            'show_done_msg': False,
            'show_cancel_msg': False,
        })

    @api.model_create_multi
    def create(self, values_list):
        """ Make sure to have a pending_msg set. """
        # This is done here and not in a default to have access to all required values.
        acquirers = super().create(values_list)
        acquirers._transfer_ensure_pending_msg_is_set()
        return acquirers

    def write(self, values):
        """ Make sure to have a pending_msg set. """
        # This is done here and not in a default to have access to all required values.
        res = super().write(values)
        self._transfer_ensure_pending_msg_is_set()
        return res

    def _transfer_ensure_pending_msg_is_set(self):
        for acquirer in self.filtered(lambda a: a.provider == 'transfer' and not a.pending_msg):
            company_id = acquirer.company_id.id
            # filter only bank accounts marked as visible
            accounts = self.env['account.journal'].search([
                ('type', '=', 'bank'), ('company_id', '=', company_id)
            ]).bank_account_id
            acquirer.pending_msg = f'<div>' \
                f'<h3>{_("Please use the following transfer details")}</h3>' \
                f'<h4>{_("Bank Account") if len(accounts) == 1 else _("Bank Accounts")}</h4>' \
                f'<ul>{"".join(f"<li>{account.display_name}</li>" for account in accounts)}</ul>' \
                f'<h4>{_("Communication")}</h4>' \
                f'<p>{_("Please use the order name as communication reference.")}</p>' \
                f'</div>'

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_transfer.controllers.main import TransferController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Transfer-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider != 'transfer':
            return res

        return {
            'api_url': TransferController._accept_url,
            'reference': self.reference,
        }

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on transfer data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The transfer feedback data
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'transfer':
            return tx

        reference = data.get('reference')
        tx = self.search([('reference', '=', reference), ('provider', '=', 'transfer')])
        if not tx:
            raise ValidationError(
                "Wire Transfer: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on transfer data.

        Note: self.ensure_one()

        :param dict data: The transfer feedback data
        :return: None
        """
        super()._process_feedback_data(data)
        if self.provider != 'transfer':
            return

        _logger.info(
            "validated transfer payment for tx with reference %s: set as pending", self.reference
        )
        self._set_pending()

    def _log_received_message(self):
        """ Override of payment to remove transfer acquirer from the recordset.

        :return: None
        """
        other_provider_txs = self.filtered(lambda t: t.provider != 'transfer')
        super(PaymentTransaction, other_provider_txs)._log_received_message()

    def _get_sent_message(self):
        """ Override of payment to return a different message.

        :return: The 'transaction sent' message
        :rtype: str
        """
        message = super()._get_sent_message()
        if self.provider == 'transfer':
            message = _(
                "The customer has selected %(acq_name)s to make the payment.",
                acq_name=self.acquirer_id.name
            )
        return message

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_acquirer
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M19.25 33.173c1.733.551 3.65.827 5.75.827 4 0 7-1 9-3h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.514-1.416.5-2.742h13.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V33.173zm29.447 14.382a.342.342 0 0 0 .339-.343V37.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zm-9.242-18.427a.938.938 0 0 1 0 1.42L24.8 30.772c-.6.52-1.55.1-1.55-.71v-3.434c-6.058.087-8.67 1.591-6.898 7.256.197.629-.563 1.116-1.097.728C13.546 33.369 12 30.99 12 28.59c0-5.946 4.975-7.204 11.25-7.276v-3.127c0-.807.948-1.23 1.55-.71l6.875 5.937z"/><path id="e" d="M19.25 31.173c1.733.551 3.65.827 5.75.827 4 0 7-1 9-3h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.514-1.416.5-2.742h13.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V31.173zm29.447 14.382a.342.342 0 0 0 .339-.343V35.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zm-9.242-18.427a.938.938 0 0 1 0 1.42L24.8 28.772c-.6.52-1.55.1-1.55-.71v-3.434c-6.058.087-8.67 1.591-6.898 7.256.197.629-.563 1.116-1.097.728C13.546 31.369 12 28.99 12 26.59c0-5.946 4.975-7.204 11.25-7.276v-3.127c0-.807.948-1.23 1.55-.71l6.875 5.937z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L13 23l4.914-2.142 5.764-5.35L32 22l3.561.74L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_transfer_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="reference" t-att-value="reference"/>
        </form>
    </template>

    <template id="transfer_transaction_status" inherit_id="payment.transaction_status">
        <xpath expr="//span[@t-out='tx.acquirer_id.sudo().pending_msg']" position="after">
            <t t-if="tx.acquirer_id.sudo().provider == 'transfer'">
                <div t-if="tx.reference">
                    <strong>Communication: </strong><span t-esc="tx.reference"/>
                </div>
                <div t-if="tx.acquirer_id.sudo().qr_code">
                    <t t-set="qr_code" t-value="tx.company_id.sudo().partner_id.bank_ids[:1].build_qr_code_base64(tx.amount, tx.reference, None, tx.currency_id, tx.partner_id)"/>
                    <div t-if="qr_code" class="mt-2">
                        <h3>Or scan me with your banking app.</h3>
                        <img class="border border-dark rounded" t-att-src="qr_code"/>
                    </div>
                </div>
            </t>
        </xpath>
    </template>

</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">Wire Transfer Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <field name="capture_manually" position="after">
                <field name="qr_code" attrs="{'invisible': [('provider', '!=', 'transfer')]}" />
            </field>
            <xpath expr="//group[@name='payment_followup']" position="attributes">
                <attribute name="attrs">
                    {'invisible': [('provider', '=', 'transfer')]}
                </attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

