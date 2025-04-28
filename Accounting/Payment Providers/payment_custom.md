# Odoo Module: payment_custom

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'custom')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'custom')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Custom Payment Modes',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A payment provider for custom flows like wire transfers.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_custom_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'payment_custom/static/src/js/post_processing.js',
        ],
    },
    'application': False,
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo.http import Controller, request, route

_logger = logging.getLogger(__name__)


class CustomController(Controller):
    _process_url = '/payment/custom/process'

    @route(_process_url, type='http', auth='public', methods=['POST'], csrf=False)
    def custom_process_transaction(self, **post):
        _logger.info("Handling custom processing with data:\n%s", pprint.pformat(post))
        request.env['payment.transaction'].sudo()._handle_notification_data('custom', post)
        return request.redirect('/payment/status')

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_transfer" model="payment.provider">
        <field name="code">custom</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <!-- Clear the default value before recomputing the pending_msg -->
        <field name="pending_msg" eval="False"/>
        <field name="custom_mode">wire_transfer</field>
    </record>

    <function model="payment.provider"
              name="_transfer_ensure_pending_msg_is_set"
              eval="[[ref('payment.payment_provider_transfer')]]"/>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.tools import is_html_empty


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    _sql_constraints = [(
        'custom_providers_setup',
        "CHECK(custom_mode IS NULL OR (code = 'custom' AND custom_mode IS NOT NULL))",
        "Only custom providers should have a custom mode."
    )]

    code = fields.Selection(
        selection_add=[('custom', "Custom")], ondelete={'custom': 'set default'}
    )
    custom_mode = fields.Selection(
        string="Custom Mode",
        selection=[('wire_transfer', "Wire Transfer")],
        required_if_provider='custom',
    )
    qr_code = fields.Boolean(
        string="Enable QR Codes", help="Enable the use of QR-codes when paying by wire transfer.")

    @api.depends('code')
    def _compute_view_configuration_fields(self):
        """ Override of payment to hide the credentials page.

        :return: None
        """
        super()._compute_view_configuration_fields()
        self.filtered(lambda p: p.code == 'custom').update({
            'show_credentials_page': False,
            'show_payment_icon_ids': False,
            'show_pre_msg': False,
            'show_done_msg': False,
            'show_cancel_msg': False,
        })

    def _transfer_ensure_pending_msg_is_set(self):
        transfer_providers_without_msg = self.filtered(
            lambda p: p.code == 'custom'
            and p.custom_mode == 'wire_transfer'
            and is_html_empty(p.pending_msg)
        )

        if not transfer_providers_without_msg:
            return  # Don't bother translating the messages.

        account_payment_module = self.env['ir.module.module']._get('account_payment')
        if account_payment_module.state != 'installed':
            transfer_providers_without_msg.pending_msg = f'<div>' \
                f'<h3>{_("Please use the following transfer details")}</h3>' \
                f'<h4>{_("Bank Account")}</h4>' \
                f'<h4>{_("Communication")}</h4>' \
                f'<p>{_("Please use the order name as communication reference.")}</p>' \
                f'</div>'
            return

        for provider in transfer_providers_without_msg:
            company_id = provider.company_id.id
            accounts = self.env['account.journal'].search([
                ('type', '=', 'bank'), ('company_id', '=', company_id)
            ]).bank_account_id
            provider.pending_msg = f'<div>' \
                f'<h3>{_("Please use the following transfer details")}</h3>' \
                f'<h4>{_("Bank Account") if len(accounts) == 1 else _("Bank Accounts")}</h4>' \
                f'<ul>{"".join(f"<li>{account.display_name}</li>" for account in accounts)}</ul>' \
                f'<h4>{_("Communication")}</h4>' \
                f'<p>{_("Please use the order name as communication reference.")}</p>' \
                f'</div>'

    def _get_removal_values(self):
        """ Override of `payment` to nullify the `custom_mode` field. """
        res = super()._get_removal_values()
        res['custom_mode'] = None
        return res

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_custom.controllers.main import CustomController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return custom-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'custom':
            return res

        return {
            'api_url': CustomController._process_url,
            'reference': self.reference,
        }

    def _get_communication(self):
        """ Return the communication the user should use for their transaction.

        This communication might change according to the settings and the accounting localization.

        Note: self.ensure_one()

        :return: The selected communication.
        :rtype: str
        """
        self.ensure_one()
        communication = ""
        if hasattr(self, 'invoice_ids') and self.invoice_ids:
            communication = self.invoice_ids[0].payment_reference
        elif hasattr(self, 'sale_order_ids') and self.sale_order_ids:
            communication = self.sale_order_ids[0].reference
        return communication or self.reference

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on custom data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification feedback data
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'custom' or len(tx) == 1:
            return tx

        reference = notification_data.get('reference')
        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'custom')])
        if not tx:
            raise ValidationError(
                "Wire Transfer: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on custom data.

        Note: self.ensure_one()

        :param dict notification_data: The custom data
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'custom':
            return

        _logger.info(
            "validated custom payment for transaction with reference %s: set as pending",
            self.reference
        )
        self._set_pending()

    def _log_received_message(self):
        """ Override of `payment` to remove custom providers from the recordset.

        :return: None
        """
        other_provider_txs = self.filtered(lambda t: t.provider_code != 'custom')
        super(PaymentTransaction, other_provider_txs)._log_received_message()

    def _get_sent_message(self):
        """ Override of payment to return a different message.

        :return: The 'transaction sent' message
        :rtype: str
        """
        message = super()._get_sent_message()
        if self.provider_code == 'custom':
            message = _(
                "The customer has selected %(provider_name)s to make the payment.",
                provider_name=self.provider_id.name
            )
        return message

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M19.25 33.173c1.733.551 3.65.827 5.75.827 4 0 7-1 9-3h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.514-1.416.5-2.742h13.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V33.173zm29.447 14.382a.342.342 0 0 0 .339-.343V37.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zm-9.242-18.427a.938.938 0 0 1 0 1.42L24.8 30.772c-.6.52-1.55.1-1.55-.71v-3.434c-6.058.087-8.67 1.591-6.898 7.256.197.629-.563 1.116-1.097.728C13.546 33.369 12 30.99 12 28.59c0-5.946 4.975-7.204 11.25-7.276v-3.127c0-.807.948-1.23 1.55-.71l6.875 5.937z"/><path id="e" d="M19.25 31.173c1.733.551 3.65.827 5.75.827 4 0 7-1 9-3h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.514-1.416.5-2.742h13.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V31.173zm29.447 14.382a.342.342 0 0 0 .339-.343V35.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zm-9.242-18.427a.938.938 0 0 1 0 1.42L24.8 28.772c-.6.52-1.55.1-1.55-.71v-3.434c-6.058.087-8.67 1.591-6.898 7.256.197.629-.563 1.116-1.097.728C13.546 31.369 12 28.99 12 26.59c0-5.946 4.975-7.204 11.25-7.276v-3.127c0-.807.948-1.23 1.55-.71l6.875 5.937z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L13 23l4.914-2.142 5.764-5.35L32 22l3.561.74L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\post_processing.js

```javascript
odoo.define('payment_custom.post_processing', require => {
    'use strict';

    const paymentPostProcessing = require('payment.post_processing');

    paymentPostProcessing.include({
        /**
         * Don't wait for the transaction to be confirmed before redirecting customers to the
         * landing route because custom transactions remain in the state 'pending' forever.
         *
         * @override method from `payment.post_processing`
         * @param {Object} display_values_list - The post-processing values of the transactions
         */
        processPolledData: function (display_values_list) {
            // In almost every case, there will be a single transaction to display. If there are
            // more than one transaction, the last one will most likely be the one that counts. We
            // use that one to redirect the user to the landing page.
            if (display_values_list.length > 0 && display_values_list[0].provider_code === 'custom') {
                window.location = display_values_list[0].landing_route;
            } else {
                return this._super(...arguments);
            }
        }
    });
});

```

## File: views\payment_custom_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="reference" t-att-value="reference"/>
        </form>
    </template>

    <template id="custom_transaction_status" inherit_id="payment.transaction_status">
        <xpath expr="//div[@id='o_payment_status_alert']" position="inside">
            <t t-if="tx.provider_id.sudo().code == 'custom'">
                <div>
                    <strong>Communication: </strong><span t-esc="tx._get_communication()"/>
                </div>
                <div t-if="tx.provider_id.sudo().qr_code">
                    <t t-set="qr_code" t-value="tx.company_id.sudo().partner_id.bank_ids[:1].build_qr_code_base64(tx.amount, tx._get_communication(), None, tx.currency_id, tx.partner_id)"/>
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

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Custom Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <field name="capture_manually" position="after">
                <field name="qr_code" attrs="{'invisible': [('code', '!=', 'custom')]}" />
            </field>
            <group name="payment_followup" position="attributes">
                <attribute name="attrs">
                    {'invisible': [('code', '=', 'custom')]}
                </attribute>
            </group>
        </field>
    </record>

</odoo>

```

