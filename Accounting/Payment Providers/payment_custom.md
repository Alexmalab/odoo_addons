# Odoo Module: payment_custom

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The codes of the payment methods to activate when Wire Transfer is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    'wire_transfer',
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'custom')


def uninstall_hook(env):
    reset_payment_provider(env, 'custom')

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

        'data/payment_method_data.xml',
        'data/payment_provider_data.xml',  # Depends on `payment_method_wire_transfer`.
    ],
    'assets': {
        'web.assets_frontend': [
            'payment_custom/static/src/js/post_processing.js',
        ],
    },
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

## File: data\payment_method_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_method_wire_transfer" model="payment.method">
        <field name="name">Wire Transfer</field>
        <field name="code">wire_transfer</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment_custom/static/img/wire_transfer.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
    </record>

</odoo>

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
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment_custom.payment_method_wire_transfer'),
                     ])]"
        />
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
from odoo.osv.expression import AND

from odoo.addons.payment_custom import const


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

    @api.model_create_multi
    def create(self, values_list):
        providers = super().create(values_list)
        providers.filtered(lambda p: p.custom_mode == 'wire_transfer').pending_msg = None
        return providers

    def action_recompute_pending_msg(self):
        """ Recompute the pending message to include the existing bank accounts. """
        account_payment_module = self.env['ir.module.module']._get('account_payment')
        if account_payment_module.state == 'installed':
            for provider in self.filtered(lambda p: p.custom_mode == 'wire_transfer'):
                company_id = provider.company_id.id
                accounts = self.env['account.journal'].search([
                    *self.env['account.journal']._check_company_domain(company_id),
                    ('type', '=', 'bank'),
                ]).bank_account_id
                account_names = "".join(f"<li><pre>{account.display_name}</pre></li>" for account in accounts)
                provider.pending_msg = f'<div>' \
                    f'<h5>{_("Please use the following transfer details")}</h5>' \
                    f'<p><br></p>' \
                    f'<h6>{_("Bank Account") if len(accounts) == 1 else _("Bank Accounts")}</h6>' \
                    f'<ul>{account_names}</ul>'\
                    f'<p><br></p>' \
                    f'</div>'

    @api.model
    def _get_removal_domain(self, provider_code, custom_mode='', **kwargs):
        res = super()._get_removal_domain(provider_code, custom_mode=custom_mode, **kwargs)
        if provider_code == 'custom' and custom_mode:
            return AND([res, [('custom_mode', '=', custom_mode)]])
        return res

    @api.model
    def _get_removal_values(self):
        """ Override of `payment` to nullify the `custom_mode` field. """
        res = super()._get_removal_values()
        res['custom_mode'] = None
        return res

    def _transfer_ensure_pending_msg_is_set(self):
        transfer_providers_without_msg = self.filtered(
            lambda p: p.custom_mode == 'wire_transfer' and not p.pending_msg
        )
        if transfer_providers_without_msg:
            transfer_providers_without_msg.action_recompute_pending_msg()

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'custom' or self.custom_mode != 'wire_transfer':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M32 16h10v22a4 4 0 0 1-4 4h-2a4 4 0 0 1-4-4V16ZM8 16h10v22a4 4 0 0 1-4 4h-2a4 4 0 0 1-4-4V16Zm12 0h10v22a4 4 0 0 1-4 4h-2a4 4 0 0 1-4-4V16Z" fill="#2EBCFA"/><path d="M46 16a4 4 0 0 0-4-4H8a4 4 0 0 0 0 8h34a4 4 0 0 0 4-4Z" fill="#2EBCFA"/><path d="M42 12H8a4 4 0 0 1 4-4h26a4 4 0 0 1 4 4Z" fill="#088BF5"/><path d="M37 8H13a4 4 0 0 1 4-4h16a4 4 0 0 1 4 4ZM8 38a4 4 0 0 0 0 8h34a4 4 0 0 0 0-8H8Z" fill="#2EBCFA"/><path d="M12 42h2a4 4 0 0 0 4-4H8a4 4 0 0 0 4 4Zm26-26h-2a4 4 0 0 0-4 4h10a4 4 0 0 0-4-4ZM24 42h2a4 4 0 0 0 4-4H20a4 4 0 0 0 4 4Zm2-26h-2a4 4 0 0 0-4 4h10a4 4 0 0 0-4-4Zm10 26h2a4 4 0 0 0 4-4H32a4 4 0 0 0 4 4ZM14 16h-2a4 4 0 0 0-4 4h10a4 4 0 0 0-4-4Z" fill="#088BF5"/></svg>

```

## File: static\src\js\post_processing.js

```javascript
/** @odoo-module **/

import paymentPostProcessing from '@payment/js/post_processing';

paymentPostProcessing.include({
    /**
     * Don't wait for the transaction to be confirmed before redirecting customers to the
     * landing route because custom transactions remain in the state 'pending' forever.
     *
     * @override method from `@payment/js/post_processing`
     * @param {string} providerCode - The code of the provider handling the transaction.
     */
    _getFinalStates(providerCode) {
        const finalStates = this._super(...arguments);
        if (providerCode === 'custom') {
            finalStates.add('pending');
        }
        return finalStates;
    }
});

```

## File: views\payment_custom_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="payment_custom.token_form" inherit_id="payment.token_form">
        <xpath expr="//t[@t-set='hide_secured_by']" position="attributes">
            <attribute name="t-value" separator=" " add="or provider_sudo.code == 'custom'"/>
        </xpath>
    </template>

    <template id="payment_custom.payment_method_form" inherit_id="payment.method_form">
        <xpath expr="//t[@t-set='hide_secured_by']" position="attributes">
            <attribute name="t-value" separator=" " add="or provider_sudo.code == 'custom'"/>
        </xpath>
    </template>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="reference" t-att-value="reference"/>
        </form>
    </template>

    <template id="custom_state_header" inherit_id="payment.state_header">

        <xpath expr="//div[@name='o_payment_status_alert']" position="before">
            <h1 t-if=" tx.provider_id.sudo().code == 'custom' and tx.state == 'pending'"
                class="mb-3">
                Finalize your payment
            </h1>
        </xpath>

        <xpath expr="//div[@name='o_payment_status_alert']" position="inside">
            <t t-set="qr_code"
               t-value="tx.provider_id.sudo().qr_code and tx.company_id.sudo().partner_id.bank_ids[:1].build_qr_code_base64(tx.amount, tx._get_communication(), None, tx.currency_id, tx.partner_id)"
            />
            <t t-if="tx.provider_id.sudo().code == 'custom' and qr_code">
                <div class="position-relative order-2 d-flex flex-md-column justify-content-center align-items-center align-items-md-stretch w-100 w-md-auto">
                    <hr class="d-inline d-md-none w-100"/>
                    <div class="vr d-none d-md-block h-100 mx-auto"/>
                    <h6 class="my-1 my-md-3 px-3 px-md-0 text-muted">OR</h6>
                    <hr class="d-inline d-md-none w-100"/>
                    <div class="vr d-none d-md-block h-100 mx-auto"/>
                </div>
                <div class="o_qr_code_card card order-1 order-md-3 bg-info">
                    <div class="card-body d-flex flex-column align-items-center justify-content-center pb-3">
                        <img class="mb-2 border border-dark rounded" t-att-src="qr_code"/>
                        <small class="text-center text-wrap lh-sm">Scan me in your banking app</small>
                    </div>
                </div>
            </t>
        </xpath>

        <xpath expr="//t[@t-set='o_payment_status_alert_class']" position="replace">
            <t t-if="tx.provider_id.sudo().code == 'custom'">
               <t t-set="o_payment_status_alert_class"
                  t-value="'d-flex flex-column flex-md-row align-items-stretch gap-2 gap-md-3 mb32'"
               />
            </t>
            <t t-else="">$0</t>
        </xpath>

        <xpath expr="//div[@id='o_payment_status_icon']" position="attributes">
            <attribute name="t-if">tx.provider_id.sudo().code != 'custom'</attribute>
        </xpath>

        <xpath expr="//div[@id='o_payment_status_message']" position="replace">
            <div t-if="tx.provider_id.sudo().code == 'custom'"
                 id="o_payment_status_message"
                 class="order-3 order-md-1 flex-grow-1"
            >
                <div class="card flex-grow-1">
                    <div id="o_payment_status_message_details" class="card-body">
                        <t>$0</t>
                        <t t-if="tx._get_communication()">
                            <hr class="w-100"/>
                            <strong class="mt-auto">Communication: </strong>
                            <span t-out="tx._get_communication()"/>
                        </t>
                    </div>
                </div>
            </div>
            <t t-else="">$0</t>
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
        <!-- Load after account_payment for the invisible attr. of the payment_followup group. -->
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <field name="code" invisible="1" position="after">
                <field name="custom_mode" invisible="1"/>
            </field>
            <page name="credentials" position="attributes">
                <attribute name="invisible" separator="or" add="code == 'custom'"/>
            </page>
            <field name="payment_method_ids" position="attributes">
                <attribute name="invisible" separator="or" add="code == 'custom'"/>
            </field>
            <a name="action_view_payment_methods" position="attributes">
                <attribute name="invisible" separator="or" add="code == 'custom'"/>
            </a>
            <field name="capture_manually" position="after">
                <field name="qr_code" invisible="code != 'custom'" />
            </field>
            <group name="payment_followup" position="attributes">
                <attribute name="invisible">code == 'custom'</attribute>
            </group>
            <field name="pre_msg" position="attributes">
                <attribute name="invisible" separator="or" add="code == 'custom'"/>
            </field>
            <field name="pending_msg" position="after">
                <div class="o_row" colspan="2"
                     invisible="custom_mode != 'wire_transfer'">
                    <button string=" Reload Pending Message"
                            type="object"
                            name="action_recompute_pending_msg"
                            class="oe_link ms-0 ps-0"
                            icon="fa-refresh"
                            />
              </div>
            </field>
            <field name="done_msg" position="attributes">
                <attribute name="invisible" separator="or" add="code == 'custom'"/>
            </field>
            <field name="cancel_msg" position="attributes">
                <attribute name="invisible" separator="or" add="code == 'custom'"/>
            </field>
        </field>
    </record>

</odoo>

```

