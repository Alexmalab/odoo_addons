# Odoo Module: account_payment

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizards

from odoo import api, SUPERUSER_ID


def post_init_hook(cr, registry):
    """ Create `account.payment.method` records for the installed payment providers. """
    env = api.Environment(cr, SUPERUSER_ID, {})
    PaymentProvider = env['payment.provider']
    installed_providers = PaymentProvider.search([('module_id.state', '=', 'installed')])
    for code in set(installed_providers.mapped('code')):
        PaymentProvider._setup_payment_method(code)


def uninstall_hook(cr, registry):
    """ Delete `account.payment.method` records created for the installed payment providers. """
    env = api.Environment(cr, SUPERUSER_ID, {})
    installed_providers = env['payment.provider'].search([('module_id.state', '=', 'installed')])
    env['account.payment.method'].search([
        ('code', 'in', installed_providers.mapped('code')),
        ('payment_type', '=', 'inbound'),
    ]).unlink()

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment - Account",
    'category': 'Accounting/Accounting',
    'summary': "Enable customers to pay invoices on the portal and post payments when transactions are processed.",
    'version': '2.0',
    'depends': ['account', 'payment'],
    'auto_install': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'security/ir_rules.xml',

        'views/account_payment_menus.xml',
        'views/account_portal_templates.xml',
        'views/payment_templates.xml',
        'views/account_move_views.xml',
        'views/account_journal_views.xml',
        'views/account_payment_views.xml',
        'views/payment_provider_views.xml',
        'views/payment_transaction_views.xml',

        'wizards/account_payment_register_views.xml',
        'wizards/payment_link_wizard_views.xml',
        'wizards/payment_refund_wizard_views.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'account_payment/static/src/js/payment_form.js',
        ],
    },
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.exceptions import AccessError, MissingError, ValidationError
from odoo.fields import Command
from odoo.http import request, route

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.controllers import portal as payment_portal


class PaymentPortal(payment_portal.PaymentPortal):

    @route('/invoice/transaction/<int:invoice_id>', type='json', auth='public')
    def invoice_transaction(self, invoice_id, access_token, **kwargs):
        """ Create a draft transaction and return its processing values.

        :param int invoice_id: The invoice to pay, as an `account.move` id
        :param str access_token: The access token used to authenticate the request
        :param dict kwargs: Locally unused data passed to `_create_transaction`
        :return: The mandatory values for the processing of the transaction
        :rtype: dict
        :raise: ValidationError if the invoice id or the access token is invalid
        """
        # Check the invoice id and the access token
        try:
            invoice_sudo = self._document_check_access('account.move', invoice_id, access_token)
        except MissingError as error:
            raise error
        except AccessError:
            raise ValidationError(_("The access token is invalid."))

        kwargs['reference_prefix'] = None  # Allow the reference to be computed based on the invoice
        logged_in = not request.env.user._is_public()
        partner = request.env.user.partner_id if logged_in else invoice_sudo.partner_id
        kwargs['partner_id'] = partner.id
        kwargs.pop('custom_create_values', None)  # Don't allow passing arbitrary create values
        tx_sudo = self._create_transaction(
            custom_create_values={'invoice_ids': [Command.set([invoice_id])]}, **kwargs,
        )

        return tx_sudo._get_processing_values()

    # Payment overrides

    @route()
    def payment_pay(self, *args, amount=None, invoice_id=None, access_token=None, **kwargs):
        """ Override of `payment` to replace the missing transaction values by that of the invoice.

        This is necessary for the reconciliation as all transaction values, excepted the amount,
        need to match exactly that of the invoice.

        :param str amount: The (possibly partial) amount to pay used to check the access token.
        :param str invoice_id: The invoice for which a payment id made, as an `account.move` id.
        :param str access_token: The access token used to authenticate the partner.
        :return: The result of the parent method.
        :rtype: str
        :raise ValidationError: If the invoice id is invalid.
        """
        # Cast numeric parameters as int or float and void them if their str value is malformed.
        amount = self._cast_as_float(amount)
        invoice_id = self._cast_as_int(invoice_id)
        if invoice_id:
            invoice_sudo = request.env['account.move'].sudo().browse(invoice_id).exists()
            if not invoice_sudo:
                raise ValidationError(_("The provided parameters are invalid."))

            # Check the access token against the invoice values. Done after fetching the invoice
            # as we need the invoice fields to check the access token.
            if not payment_utils.check_access_token(
                access_token, invoice_sudo.partner_id.id, amount, invoice_sudo.currency_id.id
            ):
                raise ValidationError(_("The provided parameters are invalid."))

            kwargs.update({
                'currency_id': invoice_sudo.currency_id.id,
                'partner_id': invoice_sudo.partner_id.id,
                'company_id': invoice_sudo.company_id.id,
                'invoice_id': invoice_id,
            })
        return super().payment_pay(*args, amount=amount, access_token=access_token, **kwargs)

    def _get_custom_rendering_context_values(self, invoice_id=None, **kwargs):
        """ Override of `payment` to add the invoice id in the custom rendering context values.

        :param int invoice_id: The invoice for which a payment id made, as an `account.move` id.
        :param dict kwargs: Optional data. This parameter is not used here.
        :return: The extended rendering context values.
        :rtype: dict
        """
        rendering_context_values = super()._get_custom_rendering_context_values(
            invoice_id=invoice_id, **kwargs
        )
        if invoice_id:
            rendering_context_values['invoice_id'] = invoice_id

            # Interrupt the payment flow if the invoice has been canceled.
            invoice_sudo = request.env['account.move'].sudo().browse(invoice_id)
            if invoice_sudo.state == 'cancel':
                rendering_context_values['amount'] = 0.0

        return rendering_context_values

    def _create_transaction(self, *args, invoice_id=None, custom_create_values=None, **kwargs):
        """ Override of `payment` to add the invoice id in the custom create values.

        :param int invoice_id: The invoice for which a payment id made, as an `account.move` id.
        :param dict custom_create_values: Additional create values overwriting the default ones.
        :param dict kwargs: Optional data. This parameter is not used here.
        :return: The result of the parent method.
        :rtype: recordset of `payment.transaction`
        """
        if invoice_id:
            if custom_create_values is None:
                custom_create_values = {}
            custom_create_values['invoice_ids'] = [Command.set([int(invoice_id)])]

        return super()._create_transaction(
            *args, invoice_id=invoice_id, custom_create_values=custom_create_values, **kwargs
        )

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request

from odoo.addons.account.controllers import portal
from odoo.addons.payment.controllers.portal import PaymentPortal
from odoo.addons.portal.controllers.portal import _build_url_w_params


class PortalAccount(portal.PortalAccount):

    def _invoice_get_page_view_values(self, invoice, access_token, **kwargs):
        values = super()._invoice_get_page_view_values(invoice, access_token, **kwargs)

        if not invoice._has_to_be_paid():
            # Do not compute payment-related stuff if given invoice doesn't have to be paid.
            return values

        logged_in = not request.env.user._is_public()
        # We set partner_id to the partner id of the current user if logged in, otherwise we set it
        # to the invoice partner id. We do this to ensure that payment tokens are assigned to the
        # correct partner and to avoid linking tokens to the public user.
        partner_sudo = request.env.user.partner_id if logged_in else invoice.partner_id
        invoice_company = invoice.company_id or request.env.company

        providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            invoice_company.id,
            partner_sudo.id,
            invoice.amount_total,
            currency_id=invoice.currency_id.id
        )  # In sudo mode to read the fields of providers and partner (if not logged in)
        tokens = request.env['payment.token'].search(
            [('provider_id', 'in', providers_sudo.ids), ('partner_id', '=', partner_sudo.id)]
        )  # Tokens are cleared at the end if the user is not logged in

        # Make sure that the partner's company matches the invoice's company.
        if not PaymentPortal._can_partner_pay_in_company(partner_sudo, invoice_company):
            providers_sudo = request.env['payment.provider'].sudo()
            tokens = request.env['payment.token']

        fees_by_provider = {
            pro_sudo: pro_sudo._compute_fees(
                invoice.amount_residual, invoice.currency_id, invoice.partner_id.country_id
            ) for pro_sudo in providers_sudo.filtered('fees_active')
        }
        values.update({
            'providers': providers_sudo,
            'tokens': tokens,
            'fees_by_provider': fees_by_provider,
            'show_tokenize_input': PaymentPortal._compute_show_tokenize_input_mapping(
                providers_sudo, logged_in=logged_in
            ),
            'amount': invoice.amount_residual,
            'currency': invoice.currency_id,
            'partner_id': partner_sudo.id,
            'access_token': access_token,
            'transaction_route': f'/invoice/transaction/{invoice.id}/',
            'landing_route': _build_url_w_params(invoice.access_url, {'access_token': access_token})
        })
        if not logged_in:
            # Don't display payment tokens of the invoice partner if the user is not logged in, but
            # inform that logging in will make them available.
            values.update({
                'existing_token': bool(tokens),
                'tokens': request.env['payment.token'],
            })
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal
from . import payment
```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import UserError


class AccountJournal(models.Model):
    _inherit = "account.journal"

    def _get_available_payment_method_lines(self, payment_type):
        lines = super()._get_available_payment_method_lines(payment_type)

        return lines.filtered(lambda l: l.payment_provider_state != 'disabled')

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_to_payment_provider(self):
        linked_providers = self.env['payment.provider'].sudo().search([]).filtered(
            lambda p: p.journal_id.id in self.ids and p.state != 'disabled'
        )
        if linked_providers:
            raise UserError(_(
                "You must first deactivate a payment provider before deleting its journal.\n"
                "Linked providers: %s", ', '.join(p.display_name for p in linked_providers)
            ))

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

from odoo.addons.payment import utils as payment_utils


class AccountMove(models.Model):
    _inherit = 'account.move'

    transaction_ids = fields.Many2many(
        string="Transactions", comodel_name='payment.transaction',
        relation='account_invoice_transaction_rel', column1='invoice_id', column2='transaction_id',
        readonly=True, copy=False)
    authorized_transaction_ids = fields.Many2many(
        string="Authorized Transactions", comodel_name='payment.transaction',
        compute='_compute_authorized_transaction_ids', readonly=True, copy=False)
    amount_paid = fields.Monetary(
        string="Amount paid",
        compute='_compute_amount_paid'
    )

    @api.depends('transaction_ids')
    def _compute_authorized_transaction_ids(self):
        for invoice in self:
            invoice.authorized_transaction_ids = invoice.transaction_ids.filtered(
                lambda tx: tx.state == 'authorized'
            )

    @api.depends('transaction_ids')
    def _compute_amount_paid(self):
        """ Sum all the transaction amount for which state is in 'authorized' or 'done'
        """
        for invoice in self:
            invoice.amount_paid = sum(
                invoice.transaction_ids.filtered(
                    lambda tx: tx.state in ('authorized', 'done')
                ).mapped('amount')
            )

    def _has_to_be_paid(self):
        self.ensure_one()
        transactions = self.transaction_ids.filtered(lambda tx: tx.state in ('pending', 'authorized', 'done'))
        pending_manual_txs = transactions.filtered(lambda tx: tx.state == 'pending' and tx.provider_code in ('none', 'custom'))
        return bool(
            (
                self.amount_residual
                or not transactions
            )
            and self.state == 'posted'
            and self.payment_state in ('not_paid', 'partial')
            and self.amount_total
            and self.move_type == 'out_invoice'
            and (pending_manual_txs or not transactions or self.amount_paid < self.amount_total)
        )

    def get_portal_last_transaction(self):
        self.ensure_one()
        return self.with_context(active_test=False).transaction_ids._get_last()

    def payment_action_capture(self):
        """ Capture all transactions linked to this invoice. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on provider fields.
        self.authorized_transaction_ids.sudo().action_capture()

    def payment_action_void(self):
        """ Void all transactions linked to this invoice. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on provider fields.
        self.authorized_transaction_ids.sudo().action_void()

    def action_view_payment_transactions(self):
        action = self.env['ir.actions.act_window']._for_xml_id('payment.action_payment_transaction')

        if len(self.transaction_ids) == 1:
            action['view_mode'] = 'form'
            action['res_id'] = self.transaction_ids.id
            action['views'] = []
        else:
            action['domain'] = [('id', 'in', self.transaction_ids.ids)]

        return action

    def _get_default_payment_link_values(self):
        self.ensure_one()
        return {
            'description': self.payment_reference,
            'amount': self.amount_residual,
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'amount_max': self.amount_residual,
        }

```

## File: models\account_payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, Command, fields, models
from odoo.exceptions import ValidationError


class AccountPayment(models.Model):
    _inherit = 'account.payment'

    # == Business fields ==
    payment_transaction_id = fields.Many2one(
        string="Payment Transaction",
        comodel_name='payment.transaction',
        readonly=True,
        auto_join=True,  # No access rule bypass since access to payments means access to txs too
    )
    payment_token_id = fields.Many2one(
        string="Saved Payment Token", comodel_name='payment.token', domain="""[
            ('id', 'in', suitable_payment_token_ids),
        ]""",
        help="Note that only tokens from providers allowing to capture the amount are available.")
    amount_available_for_refund = fields.Monetary(compute='_compute_amount_available_for_refund')

    # == Display purpose fields ==
    suitable_payment_token_ids = fields.Many2many(
        comodel_name='payment.token',
        compute='_compute_suitable_payment_token_ids',
        compute_sudo=True,
    )
    # Technical field used to hide or show the payment_token_id if needed
    use_electronic_payment_method = fields.Boolean(
        compute='_compute_use_electronic_payment_method',
    )

    # == Fields used for traceability ==
    source_payment_id = fields.Many2one(
        string="Source Payment",
        comodel_name='account.payment',
        help="The source payment of related refund payments",
        related='payment_transaction_id.source_transaction_id.payment_id',
        readonly=True,
        store=True,  # Stored for the group by in `_compute_refunds_count`
        index='btree_not_null',
    )
    refunds_count = fields.Integer(string="Refunds Count", compute='_compute_refunds_count')

    #=== COMPUTE METHODS ===#

    def _compute_amount_available_for_refund(self):
        for payment in self:
            tx_sudo = payment.payment_transaction_id.sudo()
            if tx_sudo.provider_id.support_refund and tx_sudo.operation != 'refund':
                # Only consider refund transactions that are confirmed by summing the amounts of
                # payments linked to such refund transactions. Indeed, should a refund transaction
                # be stuck forever in a transient state (due to webhook failure, for example), the
                # user would never be allowed to refund the source transaction again.
                refund_payments = self.search([('source_payment_id', '=', payment.id)])
                refunded_amount = abs(sum(refund_payments.mapped('amount')))
                payment.amount_available_for_refund = payment.amount - refunded_amount
            else:
                payment.amount_available_for_refund = 0

    @api.depends('payment_method_line_id')
    def _compute_suitable_payment_token_ids(self):
        for payment in self:
            related_partner_ids = (
                    payment.partner_id
                    | payment.partner_id.commercial_partner_id
                    | payment.partner_id.commercial_partner_id.child_ids
            )._origin

            if payment.use_electronic_payment_method:
                payment.suitable_payment_token_ids = self.env['payment.token'].sudo().search([
                    ('company_id', '=', payment.company_id.id),
                    ('provider_id.capture_manually', '=', False),
                    ('partner_id', 'in', related_partner_ids.ids),
                    ('provider_id', '=', payment.payment_method_line_id.payment_provider_id.id),
                ])
            else:
                payment.suitable_payment_token_ids = [Command.clear()]

    @api.depends('payment_method_line_id')
    def _compute_use_electronic_payment_method(self):
        for payment in self:
            # Get a list of all electronic payment method codes.
            # These codes are comprised of 'electronic' and the providers of each payment provider.
            codes = [key for key in dict(self.env['payment.provider']._fields['code']._description_selection(self.env))]
            payment.use_electronic_payment_method = payment.payment_method_code in codes

    def _compute_refunds_count(self):
        rg_data = self.env['account.payment']._read_group(
            domain=[
                ('source_payment_id', 'in', self.ids),
                ('payment_transaction_id.operation', '=', 'refund')
            ],
            fields=['source_payment_id'],
            groupby=['source_payment_id']
        )
        data = {x['source_payment_id'][0]: x['source_payment_id_count'] for x in rg_data}
        for payment in self:
            payment.refunds_count = data.get(payment.id, 0)

    #=== ONCHANGE METHODS ===#

    @api.onchange('partner_id', 'payment_method_line_id', 'journal_id')
    def _onchange_set_payment_token_id(self):
        codes = [key for key in dict(self.env['payment.provider']._fields['code']._description_selection(self.env))]
        if not (self.payment_method_code in codes and self.partner_id and self.journal_id):
            self.payment_token_id = False
            return

        related_partner_ids = (
                self.partner_id
                | self.partner_id.commercial_partner_id
                | self.partner_id.commercial_partner_id.child_ids
        )._origin

        self.payment_token_id = self.env['payment.token'].sudo().search([
            ('company_id', '=', self.company_id.id),
            ('partner_id', 'in', related_partner_ids.ids),
            ('provider_id.capture_manually', '=', False),
            ('provider_id', '=', self.payment_method_line_id.payment_provider_id.id),
         ], limit=1)

    #=== ACTION METHODS ===#

    def action_post(self):
        # Post the payments "normally" if no transactions are needed.
        # If not, let the provider update the state.

        payments_need_tx = self.filtered(
            lambda p: p.payment_token_id and not p.payment_transaction_id
        )
        # creating the transaction require to access data on payment providers, not always accessible to users
        # able to create payments
        transactions = payments_need_tx.sudo()._create_payment_transaction()

        res = super(AccountPayment, self - payments_need_tx).action_post()

        for tx in transactions:  # Process the transactions with a payment by token
            tx._send_payment_request()

        # Post payments for issued transactions
        transactions._finalize_post_processing()
        payments_tx_done = payments_need_tx.filtered(
            lambda p: p.payment_transaction_id.state == 'done'
        )
        super(AccountPayment, payments_tx_done).action_post()
        payments_tx_not_done = payments_need_tx.filtered(
            lambda p: p.payment_transaction_id.state != 'done'
        )
        payments_tx_not_done.action_cancel()

        return res

    def action_refund_wizard(self):
        self.ensure_one()
        return {
            'name': _("Refund"),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'payment.refund.wizard',
            'target': 'new',
        }

    def action_view_refunds(self):
        self.ensure_one()
        action = {
            'name': _("Refund"),
            'res_model': 'account.payment',
            'type': 'ir.actions.act_window',
        }
        if self.refunds_count == 1:
            refund_tx = self.env['account.payment'].search([
                ('source_payment_id', '=', self.id)
            ], limit=1)
            action['res_id'] = refund_tx.id
            action['view_mode'] = 'form'
        else:
            action['view_mode'] = 'tree,form'
            action['domain'] = [('source_payment_id', '=', self.id)]
        return action

    #=== BUSINESS METHODS - PAYMENT FLOW ===#

    def _create_payment_transaction(self, **extra_create_values):
        for payment in self:
            if payment.payment_transaction_id:
                raise ValidationError(_(
                    "A payment transaction with reference %s already exists.",
                    payment.payment_transaction_id.reference
                ))
            elif not payment.payment_token_id:
                raise ValidationError(_("A token is required to create a new payment transaction."))

        transactions = self.env['payment.transaction']
        for payment in self:
            transaction_vals = payment._prepare_payment_transaction_vals(**extra_create_values)
            transaction = self.env['payment.transaction'].create(transaction_vals)
            transactions += transaction
            payment.payment_transaction_id = transaction  # Link the transaction to the payment
        return transactions

    def _prepare_payment_transaction_vals(self, **extra_create_values):
        self.ensure_one()
        return {
            'provider_id': self.payment_token_id.provider_id.id,
            'reference': self.env['payment.transaction']._compute_reference(
                self.payment_token_id.provider_id.code, prefix=self.ref
            ),
            'amount': self.amount,
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'token_id': self.payment_token_id.id,
            'operation': 'offline',
            'payment_id': self.id,
            **({'invoice_ids': [Command.set(self._context.get('active_ids', []))]}
                if self._context.get('active_model') == 'account.move'
                else {}),
            **extra_create_values,
        }

    def _get_payment_refund_wizard_values(self):
        self.ensure_one()
        return {
            'transaction_id': self.payment_transaction_id.id,
            'payment_amount': self.amount,
            'amount_available_for_refund': self.amount_available_for_refund,
        }

```

## File: models\account_payment_method.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountPaymentMethod(models.Model):
    _inherit = 'account.payment.method'

    @api.model
    def _get_payment_method_information(self):
        res = super()._get_payment_method_information()
        for code, _desc in self.env['payment.provider']._fields['code'].selection:
            if code in ('none', 'custom'):
                continue
            res[code] = {
                'mode': 'electronic',
                'domain': [('type', '=', 'bank')],
            }
        return res

```

## File: models\account_payment_method_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


class AccountPaymentMethodLine(models.Model):
    _inherit = "account.payment.method.line"

    payment_provider_id = fields.Many2one(
        comodel_name='payment.provider',
        compute='_compute_payment_provider_id',
        store=True,
        readonly=False,
    )
    payment_provider_state = fields.Selection(
        related='payment_provider_id.state'
    )

    @api.depends('payment_provider_id.name')
    def _compute_name(self):
        super()._compute_name()
        for line in self:
            if line.payment_provider_id and not line.name:
                line.name = line.payment_provider_id.name

    @api.depends('payment_method_id')
    def _compute_payment_provider_id(self):
        results = self.journal_id._get_journals_payment_method_information()
        manage_providers = results['manage_providers']
        method_information_mapping = results['method_information_mapping']
        providers_per_code = results['providers_per_code']

        for line in self:
            journal = line.journal_id
            company = journal.company_id
            if (
                company
                and line.payment_method_id
                and not line.payment_provider_id
                and manage_providers
                and method_information_mapping.get(line.payment_method_id.id, {}).get('mode') == 'electronic'
            ):
                provider_ids = providers_per_code.get(company.id, {}).get(line.code, set())

                # Exclude the 'unique' / 'electronic' values that are already set on the journal.
                protected_provider_ids = set()
                for payment_type in ('inbound', 'outbound'):
                    lines = journal[f'{payment_type}_payment_method_line_ids']
                    for journal_line in lines:
                        if journal_line.payment_method_id:
                            if (
                                manage_providers
                                and method_information_mapping.get(journal_line.payment_method_id.id, {}).get('mode') == 'electronic'
                            ):
                                protected_provider_ids.add(journal_line.payment_provider_id.id)

                candidates_provider_ids = provider_ids - protected_provider_ids
                if candidates_provider_ids:
                    line.payment_provider_id = next(iter(candidates_provider_ids))

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_provider(self):
        """ Ensure we don't remove an account.payment.method.line that is linked to a provider
        in the test or enabled state.
        """
        active_provider = self.payment_provider_id.filtered(lambda provider: provider.state in ['enabled', 'test'])
        if active_provider:
            raise UserError(_(
                "You can't delete a payment method that is linked to a provider in the enabled "
                "or test state.\n""Linked providers(s): %s",
                ', '.join(a.display_name for a in active_provider),
            ))

    def action_open_provider_form(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Provider'),
            'view_mode': 'form',
            'res_model': 'payment.provider',
            'target': 'current',
            'res_id': self.payment_provider_id.id
        }

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError

class Paymentprovider(models.Model):
    _inherit = 'payment.provider'

    journal_id = fields.Many2one(
        string="Payment Journal",
        help="The journal in which the successful transactions are posted.",
        comodel_name='account.journal',
        compute='_compute_journal_id',
        inverse='_inverse_journal_id',
        domain='[("type", "=", "bank"), ("company_id", "=", company_id)]',
        copy=False,
    )

    #=== COMPUTE METHODS ===#

    def _ensure_payment_method_line(self, allow_create=True):
        self.ensure_one()
        if not self.id:
            return

        pay_method_line = self.env['account.payment.method.line'].search(
            [('code', '=', self.code), ('payment_provider_id', '=', self.id)],
            limit=1,
        )

        if not self.journal_id:
            if pay_method_line:
                pay_method_line.unlink()
                return

        if not pay_method_line:
            pay_method_line = self.env['account.payment.method.line'].search(
                [
                    ('company_id', '=', self.company_id.id),
                    ('code', '=', self.code),
                    ('payment_provider_id', '=', False),
                ],
                limit=1,
            )
        if pay_method_line:
            pay_method_line.payment_provider_id = self
            pay_method_line.journal_id = self.journal_id
            pay_method_line.name = self.name
        elif allow_create:
            default_payment_method_id = self._get_default_payment_method_id(self.code)
            if not default_payment_method_id:
                return

            create_values = {
                'name': self.name,
                'payment_method_id': default_payment_method_id,
                'journal_id': self.journal_id.id,
                'payment_provider_id': self.id,
            }
            pay_method_line_same_code = self.env['account.payment.method.line'].search(
                [
                    ('company_id', '=', self.company_id.id),
                    ('code', '=', self.code),
                ],
                limit=1,
            )
            if pay_method_line_same_code:
                create_values['payment_account_id'] = pay_method_line_same_code.payment_account_id.id
            self.env['account.payment.method.line'].create(create_values)

    @api.depends('code', 'state', 'company_id')
    def _compute_journal_id(self):
        for provider in self:
            pay_method_line = self.env['account.payment.method.line'].search(
                [('code', '=', provider.code), ('payment_provider_id', '=', provider._origin.id)],
                limit=1,
            )

            if pay_method_line:
                provider.journal_id = pay_method_line.journal_id
            elif provider.state in ('enabled', 'test'):
                provider.journal_id = self.env['account.journal'].search(
                    [
                        ('company_id', '=', provider.company_id.id),
                        ('type', '=', 'bank'),
                    ],
                    limit=1,
                )
                if provider.id:
                    provider._ensure_payment_method_line()

    def _inverse_journal_id(self):
        for provider in self:
            provider._ensure_payment_method_line()

    @api.model
    def _get_default_payment_method_id(self, code):
        provider_payment_method = self._get_provider_payment_method(code)
        if provider_payment_method:
            return provider_payment_method.id
        return None

    @api.model
    def _get_provider_payment_method(self, code):
        return self.env['account.payment.method'].search([('code', '=', code)], limit=1)

    #=== BUSINESS METHODS ===#

    @api.model
    def _setup_provider(self, code):
        """ Override of `payment` to create the payment method of the provider. """
        super()._setup_provider(code)
        self._setup_payment_method(code)

    @api.model
    def _setup_payment_method(self, code):
        if code not in ('none', 'custom') and not self._get_provider_payment_method(code):
            providers_description = dict(self._fields['code']._description_selection(self.env))
            self.env['account.payment.method'].sudo().create({
                'name': providers_description[code],
                'code': code,
                'payment_type': 'inbound',
            })

    def _check_existing_payment_method_lines(self, payment_method):
        existing_payment_method_lines_count =  \
            self.env['account.payment.method.line'].search_count([('payment_method_id', '=', \
                payment_method.id)], limit=1)
        return bool(existing_payment_method_lines_count)

    @api.model
    def _remove_provider(self, code):
        """ Override of `payment` to delete the payment method of the provider. """
        payment_method = self._get_provider_payment_method(code)
        if self._check_existing_payment_method_lines(payment_method):
            raise UserError(_("To uninstall this module, please remove first the corresponding payment method line in the incoming payments tab defined on the bank journal."))
        super()._remove_provider(code)
        payment_method.unlink()

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, SUPERUSER_ID, _


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    payment_id = fields.Many2one(
        string="Payment", comodel_name='account.payment', readonly=True)

    invoice_ids = fields.Many2many(
        string="Invoices", comodel_name='account.move', relation='account_invoice_transaction_rel',
        column1='transaction_id', column2='invoice_id', readonly=True, copy=False,
        domain=[('move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund'))])
    invoices_count = fields.Integer(string="Invoices Count", compute='_compute_invoices_count')

    #=== COMPUTE METHODS ===#

    @api.depends('invoice_ids')
    def _compute_invoices_count(self):
        tx_data = {}
        if self.ids:
            self.env.cr.execute(
                '''
                SELECT transaction_id, count(invoice_id)
                FROM account_invoice_transaction_rel
                WHERE transaction_id IN %s
                GROUP BY transaction_id
                ''',
                [tuple(self.ids)]
            )
            tx_data = dict(self.env.cr.fetchall())  # {id: count}
        for tx in self:
            tx.invoices_count = tx_data.get(tx.id, 0)

    #=== ACTION METHODS ===#

    def action_view_invoices(self):
        """ Return the action for the views of the invoices linked to the transaction.

        Note: self.ensure_one()

        :return: The action
        :rtype: dict
        """
        self.ensure_one()

        action = {
            'name': _("Invoices"),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'target': 'current',
        }
        invoice_ids = self.invoice_ids.ids
        if len(invoice_ids) == 1:
            invoice = invoice_ids[0]
            action['res_id'] = invoice
            action['view_mode'] = 'form'
            action['views'] = [(self.env.ref('account.view_move_form').id, 'form')]
        else:
            action['view_mode'] = 'tree,form'
            action['domain'] = [('id', 'in', invoice_ids)]
        return action

    #=== BUSINESS METHODS - PAYMENT FLOW ===#

    @api.model
    def _compute_reference_prefix(self, provider_code, separator, **values):
        """ Compute the reference prefix from the transaction values.

        If the `values` parameter has an entry with 'invoice_ids' as key and a list of (4, id, O) or
        (6, 0, ids) X2M command as value, the prefix is computed based on the invoice name(s).
        Otherwise, an empty string is returned.

        Note: This method should be called in sudo mode to give access to documents (INV, SO, ...).

        :param str provider_code: The code of the provider handling the transaction
        :param str separator: The custom separator used to separate data references
        :param dict values: The transaction values used to compute the reference prefix. It should
                            have the structure {'invoice_ids': [(X2M command), ...], ...}.
        :return: The computed reference prefix if invoice ids are found, an empty string otherwise
        :rtype: str
        """
        command_list = values.get('invoice_ids')
        if command_list:
            # Extract invoice id(s) from the X2M commands
            invoice_ids = self._fields['invoice_ids'].convert_to_cache(command_list, self)
            invoices = self.env['account.move'].browse(invoice_ids).exists()
            if len(invoices) == len(invoice_ids):  # All ids are valid
                return separator.join(invoices.mapped('name'))
        return super()._compute_reference_prefix(provider_code, separator, **values)

    def _set_canceled(self, state_message=None):
        """ Update the transactions' state to 'cancel'.

        :param str state_message: The reason for which the transaction is set in 'cancel' state
        :return: updated transactions
        :rtype: `payment.transaction` recordset
        """
        processed_txs = super()._set_canceled(state_message)
        # Cancel the existing payments
        processed_txs.payment_id.action_cancel()
        return processed_txs

    #=== BUSINESS METHODS - POST-PROCESSING ===#

    def _reconcile_after_done(self):
        """ Post relevant fiscal documents and create missing payments.

        As there is nothing to reconcile for validation transactions, no payment is created for
        them. This is also true for validations with a validity check (transfer of a small amount
        with immediate refund) because validation amounts are not included in payouts.

        :return: None
        """
        super()._reconcile_after_done()

        # Validate invoices automatically once the transaction is confirmed
        self.invoice_ids.filtered(lambda inv: inv.state == 'draft').action_post()

        # Create and post missing payments for transactions requiring reconciliation
        for tx in self.filtered(lambda t: t.operation != 'validation' and not t.payment_id):
            tx._create_payment()

    def _create_payment(self, **extra_create_values):
        """Create an `account.payment` record for the current transaction.

        If the transaction is linked to some invoices, their reconciliation is done automatically.

        Note: self.ensure_one()

        :param dict extra_create_values: Optional extra create values
        :return: The created payment
        :rtype: recordset of `account.payment`
        """
        self.ensure_one()

        payment_method_line = self.provider_id.journal_id.inbound_payment_method_line_ids\
            .filtered(lambda l: l.payment_provider_id == self.provider_id)
        payment_values = {
            'amount': abs(self.amount),  # A tx may have a negative amount, but a payment must >= 0
            'payment_type': 'inbound' if self.amount > 0 else 'outbound',
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.commercial_partner_id.id,
            'partner_type': 'customer',
            'journal_id': self.provider_id.journal_id.id,
            'company_id': self.provider_id.company_id.id,
            'payment_method_line_id': payment_method_line.id,
            'payment_token_id': self.token_id.id,
            'payment_transaction_id': self.id,
            'ref': f'{self.reference} - {self.partner_id.name} - {self.provider_reference or ""}',
            **extra_create_values,
        }
        payment = self.env['account.payment'].create(payment_values)
        payment.action_post()

        # Track the payment to make a one2one.
        self.payment_id = payment

        if self.invoice_ids:
            self.invoice_ids.filtered(lambda inv: inv.state == 'draft').action_post()

            (payment.line_ids + self.invoice_ids.line_ids).filtered(
                lambda line: line.account_id == payment.destination_account_id
                and not line.reconciled
            ).reconcile()

        return payment

    #=== BUSINESS METHODS - LOGGING ===#

    def _log_message_on_linked_documents(self, message):
        """ Log a message on the payment and the invoices linked to the transaction.

        For a module to implement payments and link documents to a transaction, it must override
        this method and call super, then log the message on documents linked to the transaction.

        Note: self.ensure_one()

        :param str message: The message to be logged
        :return: None
        """
        self.ensure_one()
        self = self.with_user(SUPERUSER_ID)  # Log messages as 'OdooBot'
        if self.source_transaction_id.payment_id:
            self.source_transaction_id.payment_id.message_post(body=message)
            for invoice in self.source_transaction_id.invoice_ids:
                invoice.message_post(body=message)
        for invoice in self.invoice_ids:
            invoice.message_post(body=message)

    #=== BUSINESS METHODS - POST-PROCESSING ===#

    def _finalize_post_processing(self):
        """ Override of `payment` to write a message in the chatter with the payment and transaction
        references.

        :return: None
        """
        super()._finalize_post_processing()
        for tx in self.filtered('payment_id'):
            message = _(
                "The payment related to the transaction with reference %(ref)s has been posted: "
                "%(link)s", ref=tx.reference, link=tx.payment_id._get_html_link()
            )
            tx._log_message_on_linked_documents(message)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import account_move
from . import account_payment
from . import account_payment_method
from . import account_payment_method_line
from . import payment_provider
from . import payment_transaction

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
payment_link_wizard,payment.link.wizard,payment.model_payment_link_wizard,account.group_account_invoice,1,1,1,0
payment_refund_wizard,payment.refund.wizard,model_payment_refund_wizard,account.group_account_invoice,1,1,1,0

```

## File: security\ir_rules.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Transactions -->

    <record id="payment_transaction_billing_rule" model="ir.rule">
        <field name="name">Access every transaction</field>
        <field name="model_id" ref="payment.model_payment_transaction"/>
        <!-- Reset the domain defined by payment.transaction_user_rule -->
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('account.group_account_invoice'))]"/>
    </record>

    <!-- Tokens -->

    <record id="payment_token_billing_rule" model="ir.rule">
        <field name="name">Access every token</field>
        <field name="model_id" ref="payment.model_payment_token"/>
        <!-- Reset the domain defined by payment.token_user_rule -->
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('account.group_account_invoice'))]"/>
    </record>

</odoo>

```

## File: static\src\js\payment_form.js

```javascript
odoo.define('account_payment.payment_form', require => {
    'use strict';

    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const PaymentMixin = {

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Add `invoice_id` to the transaction route params if it is provided.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} code - The provider code of the selected payment option.
         * @param {number} paymentOptionId - The id of the selected payment option.
         * @param {string} flow - The online payment flow of the selected payment option.
         * @return {object} The extended transaction route params.
         */
        _prepareTransactionRouteParams: function (code, paymentOptionId, flow) {
            const transactionRouteParams = this._super(...arguments);
            return {
                ...transactionRouteParams,
                'invoice_id': this.txContext.invoiceId ? parseInt(this.txContext.invoiceId) : null,
            };
        },

    };

    checkoutForm.include(PaymentMixin);
    manageForm.include(PaymentMixin);

});

```

## File: views\account_journal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_journal_form" model="ir.ui.view">
        <field name="name">account.journal.form.inherit.payment</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.view_account_journal_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='inbound_payment_method_line_ids']//field[@name='payment_account_id']" position="after">
                <field name="payment_provider_state" invisible="1"/>
                <field name="code" invisible="1"/>
                <field name="payment_provider_id" options="{'no_open': True, 'no_create': True}" optional="hide" domain="[('code', '=', code)]"/>
                <button name="action_open_provider_form"
                        type="object"
                        string="SETUP"
                        class="float-end btn-secondary"
                        attrs="{'invisible': [('payment_provider_id', '=', False)]}"
                        groups="base.group_system"/>
            </xpath>
            <xpath expr="//field[@name='inbound_payment_method_line_ids']/tree" position="attributes">
                <attribute name="decoration-muted">payment_provider_state == 'disabled'</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="account_invoice_view_form_inherit_payment" model="ir.ui.view">
        <field name="name">account.move.view.form.inherit.payment</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <!--
            The user must capture/void the authorized transactions before registering a new payment.
            -->
            <xpath expr="//button[@id='account_invoice_payment_btn']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', '|', '|', ('state', '!=', 'posted'), ('payment_state', 'not in', ('partial', 'not_paid')), ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt')), ('authorized_transaction_ids', '!=', [])]}</attribute>
            </xpath>
            <xpath expr="//button[@id='account_invoice_payment_btn']" position="after">
                <field name="authorized_transaction_ids" invisible="1"/>
                <button name="payment_action_capture" type="object"
                        groups="account.group_account_invoice"
                        string="Capture Transaction" class="oe_highlight" data-hotkey="shift+g"
                        attrs="{'invisible': ['|', '|', ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('state', '!=', 'posted'), ('authorized_transaction_ids', '=', [])]}"/>
                <button name="payment_action_void" type="object"
                        groups="account.group_account_invoice"
                        string="Void Transaction" data-hotkey="shift+v"
                        confirm="Are you sure you want to void the authorized transaction? This action can't be undone."
                        attrs="{'invisible': ['|', '|', ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('state', '!=', 'posted'), ('authorized_transaction_ids', '=', [])]}"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="transaction_ids" invisible="1" />
                <button name="action_view_payment_transactions" type="object"
                        class="oe_stat_button" icon="fa-money"
                        string="Payment Transaction"
                        attrs="{'invisible': [('transaction_ids', '=', [])]}" />
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\account_payment_menus.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <menuitem action="payment.action_payment_provider"
        id="payment_provider_menu"
        parent="account.root_payment_menu"
        sequence="10"/>
    <menuitem action="payment.action_payment_icon"
        id="payment_icon_menu"
        parent="account.root_payment_menu"
        groups="base.group_no_one"
        sequence="15"/>
    <menuitem action="payment.action_payment_token"
        id="payment_token_menu"
        parent="account.root_payment_menu"
        groups="base.group_no_one"
        sequence="20"/>
    <menuitem action="payment.action_payment_transaction"
        id="payment_transaction_menu"
        parent="account.root_payment_menu"
        groups="base.group_no_one"
        sequence="25"/>

</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="view_account_payment_form_inherit_payment" model="ir.ui.view">
        <field name="name">view.account.payment.form.inherit.payment</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="account.view_account_payment_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header/button[@name='action_draft']" position="after">
                <field name="amount_available_for_refund" invisible="1"/>
                <button name="action_refund_wizard"
                    type="object"
                    string="Refund"
                    groups="account.group_account_invoice"
                    attrs="{'invisible': [('amount_available_for_refund', '&lt;=', 0)]}"
                    class="btn-secondary"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_view_refunds"
                    type="object"
                    class="oe_stat_button"
                    icon="fa-money"
                    attrs="{'invisible': [('refunds_count', '=', 0)]}">
                    <field name="refunds_count" widget="statinfo" string="Refunds"/>
                </button>
            </xpath>
            <xpath expr='//group[2]' position="inside">
                <field name="source_payment_id" attrs="{'invisible': [('source_payment_id', '=', False)]}"/>
                <field name="payment_transaction_id" groups="base.group_no_one" attrs="{'invisible': [('use_electronic_payment_method', '!=', True)]}"/>
            </xpath>
            <field name="payment_method_line_id" position="after">
                <field name="payment_method_code" invisible="1"/>
                <field name="suitable_payment_token_ids" invisible="1"/>
                <field name="use_electronic_payment_method" invisible="1"/>
                <field name="payment_token_id" options="{'no_create': True}"
                    attrs="{'invisible': [('use_electronic_payment_method', '!=', True)], 'readonly': [('state', '!=', 'draft')]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\account_portal_templates.xml

```xml
<odoo>
    <template id="portal_my_invoices_payment" name="Payment on My Invoices" inherit_id="account.portal_my_invoices">
        <xpath expr="//t[@t-call='portal.portal_table']/thead/tr/th[last()]" position="before">
            <th></th>
        </xpath>
        <xpath expr="//t[@t-foreach='invoices']/tr/td[last()]" position="before">
            <td class="text-center">
                <t t-set="tx_ids" t-value="invoice.transaction_ids.filtered(lambda tx: tx.state in ('pending', 'authorized'))"/>
                <t t-set="pending_manual_txs" t-value="tx_ids.filtered(lambda tx: tx.state == 'pending' and tx.provider_code in ('none', 'custom'))"/>
                <a t-if="invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and invoice.amount_total and invoice.move_type == 'out_invoice' and (pending_manual_txs or not tx_ids or invoice.amount_paid &lt; invoice.amount_total)"
                    t-att-href="invoice.get_portal_url(anchor='portal_pay')" title="Pay Now" aria-label="Pay now" class="btn btn-sm btn-primary" role="button">
                    <i class="fa fa-arrow-circle-right"/><span class='d-none d-md-inline'> Pay Now</span>
                </a>
            </td>
        </xpath>
        <xpath expr="//t[@t-foreach='invoices']/tr/td[hasclass('tx_status')]" position="replace">
            <t t-set="last_tx" t-value="invoice.get_portal_last_transaction()"/>
            <td class="tx_status text-center">
                <t t-if="invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and (last_tx.state not in ['pending', 'authorized'] or (last_tx.state == 'pending' and last_tx.provider_code in ('none', 'custom')))">
                    <span class="badge rounded-pill text-bg-info"><i class="fa fa-fw fa-clock-o"></i><span class="d-none d-md-inline"> Waiting for Payment</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and last_tx.state == 'authorized'">
                    <span class="badge rounded-pill text-bg-primary"><i class="fa fa-fw fa-check"/><span class="d-none d-md-inline"> Authorized</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and last_tx.state == 'pending' and last_tx.provider_code not in ('none', 'custom')">
                  <span class="badge rounded-pill text-bg-warning"><span class="d-none d-md-inline"> Pending</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and invoice.payment_state in ('paid', 'in_payment')">
                    <span class="badge rounded-pill text-bg-success"><i class="fa fa-fw fa-check"></i><span class="d-none d-md-inline"> Paid</span></span>
                </t>
                <t t-if="invoice.state == 'posted' and invoice.payment_state == 'reversed'">
                    <span class="badge rounded-pill text-bg-success"><i class="fa fa-fw fa-check"></i><span class="d-none d-md-inline"> Reversed</span></span>
                </t>
                <t t-if="invoice.state == 'cancel'">
                    <span class="badge rounded-pill text-bg-danger"><i class="fa fa-fw fa-remove"></i><span class="d-none d-md-inline"> Cancelled</span></span>
                </t>
            </td>
        </xpath>
    </template>

    <template id="portal_invoice_payment" name="Invoice Payment">
        <div class="row">
            <div class="modal fade" id="pay_with" role="dialog">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h3 class="modal-title">Pay with</h3>
                            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                        </div>
                        <div class="modal-body">
                            <div t-if="providers or tokens" id="payment_method" class="text-start col-md-13">
                                <t t-call="payment.checkout"/>
                            </div>
                            <div t-else="" class="alert alert-warning">
                                <strong>No suitable payment option could be found.</strong><br/>
                                If you believe that it is an error, please contact the website administrator.
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="portal_invoice_page_inherit_payment" name="Payment on My Invoices" inherit_id="account.portal_invoice_page">
        <xpath expr="//t[@t-call='portal.portal_record_sidebar']//div[hasclass('o_download_pdf')]" position="before">
            <t t-set="tx_ids" t-value="invoice.transaction_ids.filtered(lambda tx: tx.state in ('pending', 'authorized'))"/>
            <t t-set="pending_manual_txs" t-value="tx_ids.filtered(lambda tx: tx.state == 'pending' and tx.provider_code in ('none', 'custom'))"/>
            <div class="d-grid">
                <a href="#" t-if="invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and invoice.amount_total and invoice.move_type == 'out_invoice' and (pending_manual_txs or not tx_ids or invoice.amount_paid &lt; invoice.amount_total)"

                    class="btn btn-primary mb-2" data-bs-toggle="modal" data-bs-target="#pay_with">
                    <i class="fa fa-fw fa-arrow-circle-right"/> Pay Now
                </a>
                <div t-if="tx_ids and not pending_manual_txs and not invoice.amount_residual and invoice.payment_state not in ('paid', 'in_payment')" class="alert alert-info py-1 mb-2" >
                    <i class="fa fa-fw fa-check-circle"/> Pending
                </div>
                <div t-if="invoice.payment_state in ('paid', 'in_payment')" class="alert alert-success py-1 mb-2" >
                    <i class="fa fa-fw fa-check-circle"/> Paid
                </div>
            </div>
        </xpath>
        <xpath expr="//div[@id='invoice_content']//div[hasclass('o_portal_html_view')]" position="before">
            <t t-set="tx" t-value="invoice.get_portal_last_transaction()"/>
            <div t-if="invoice.transaction_ids and invoice.amount_total and not success and not error and (invoice.payment_state != 'not_paid' or tx.state in ('pending', 'authorized'))"
                 class="o_account_payment_tx_status"
                 t-att-data-invoice-id="invoice.id">
                <t t-call="payment.transaction_status"/>
            </div>
            <t t-set="tx_ids" t-value="invoice.transaction_ids.filtered(lambda tx: tx.state in ('authorized', 'done'))"/>
            <div t-if="(invoice.amount_residual or not tx_ids) and invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and invoice.amount_total" id="portal_pay">
                <t t-call="account_payment.portal_invoice_payment"/>
            </div>
            <div class="panel-body" t-if="existing_token">
                <div class="offset-lg-3 col-lg-6">
                    <i class="fa fa-info"></i> You have credits card registered, you can log-in to be able to use them.
                </div>
            </div>
        </xpath>
    </template>

    <template id="portal_invoice_error" name="Invoice error display: payment errors"
            inherit_id="account.portal_invoice_error">
        <xpath expr="//t[@name='generic']" position="after">
            <t t-if="error == 'pay_invoice_invalid_doc'">
                There was an error processing your payment: invalid invoice.
            </t>
            <t t-if="error == 'pay_invoice_invalid_token'">
                There was en error processing your payment: invalid credit card ID.
            </t>
            <t t-if="error == 'pay_invoice_tx_fail'">
                There was an error processing your payment: transaction failed.<br />
                <t t-set="tx_id" t-value="invoice.get_portal_last_transaction()"/>
                <t t-if="tx_id and tx_id.state_message">
                    <t t-esc="tx_id.state_message"/>
                </t>
            </t>
            <t t-if="error == 'pay_invoice_tx_token'">
                There was an error processing your payment: issue with credit card ID validation.
            </t>
        </xpath>
    </template>

    <template id="portal_invoice_success" name="Invoice success display: payment success"
            inherit_id="account.portal_invoice_success">
        <xpath expr="//a[hasclass('close')]" position="after">
            <t t-if="success == 'pay_invoice'">
                <t t-set="payment_tx_id" t-value="invoice.get_portal_last_transaction()"/>
                <span t-if='payment_tx_id.provider_id.sudo().done_msg' t-out="payment_tx_id.provider_id.sudo().done_msg"/>
                <div t-if="payment_tx_id.provider_id.sudo().pending_msg and payment_tx_id.provider_code == 'custom' and invoice.ref">
                    <b>Communication: </b><span t-esc='invoice.ref'/>
                </div>
            </t>
            <t t-if="success == 'pay_invoice' and invoice.payment_state in ('paid', 'in_payment')">
                Done, your online payment has been successfully processed. Thank you for your order.
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">payment.provider.form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="payment_followup" position="attributes">
                <attribute name="invisible">
                    0
                </attribute>
            </group>
            <group name="payment_followup" position="inside">
                <field name="journal_id"
                    context="{'default_type': 'bank'}"
                    attrs="{'required': [('state', '!=', 'disabled'), ('code', 'not in', ['none', 'custom'])]}"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <!-- Include account-related values in payment checkout form to pass them to the client -->
    <template id="payment_checkout_inherit" inherit_id="payment.checkout">
        <xpath expr="//form[@name='o_payment_checkout']" position="attributes">
            <attribute name="t-att-data-invoice-id">invoice_id</attribute>
        </xpath>
    </template>

    <!-- Include account-related values in payment manage form to pass them to the client -->
    <template id="payment_manage_inherit" inherit_id="payment.manage">
        <xpath expr="//form[@name='o_payment_manage']" position="attributes">
            <attribute name="t-att-data-invoice-id">invoice_id</attribute>
        </xpath>
    </template>

</odoo>

```

## File: views\payment_transaction_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_transaction_form" model="ir.ui.view">
        <field name="name">payment.transaction.form</field>
        <field name="model">payment.transaction</field>
        <field name="inherit_id" ref="payment.payment_transaction_form"/>
        <field name="arch" type="xml">
            <button name="action_view_refunds" position="before">
                <button name="action_view_invoices" type="object"
                        class="oe_stat_button" icon="fa-money"
                        attrs="{'invisible': [('invoices_count', '=', 0)]}">
                    <field name="invoices_count" widget="statinfo" string="Invoice(s)"/>
                </button>
            </button>
            <field name="reference" position="after">
                <field name="payment_id"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: wizards\account_payment_register.py

```python
from odoo import api, Command, fields, models


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    # == Business fields ==
    payment_token_id = fields.Many2one(
        comodel_name='payment.token',
        string="Saved payment token",
        store=True, readonly=False,
        compute='_compute_payment_token_id',
        domain='''[
            ('id', 'in', suitable_payment_token_ids),
        ]''',
        help="Note that tokens from providers set to only authorize transactions (instead of capturing the amount) are "
             "not available.")

    # == Display purpose fields ==
    suitable_payment_token_ids = fields.Many2many(
        comodel_name='payment.token',
        compute='_compute_suitable_payment_token_ids'
    )
    # Technical field used to hide or show the payment_token_id if needed
    use_electronic_payment_method = fields.Boolean(
        compute='_compute_use_electronic_payment_method',
    )
    payment_method_code = fields.Char(
        related='payment_method_line_id.code')

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('payment_method_line_id')
    def _compute_suitable_payment_token_ids(self):
        for wizard in self:
            if wizard.can_edit_wizard and wizard.use_electronic_payment_method:
                related_partner_ids = (
                        wizard.partner_id
                        | wizard.partner_id.commercial_partner_id
                        | wizard.partner_id.commercial_partner_id.child_ids
                )._origin

                wizard.suitable_payment_token_ids = self.env['payment.token'].sudo().search([
                    ('company_id', '=', wizard.company_id.id),
                    ('provider_id.capture_manually', '=', False),
                    ('partner_id', 'in', related_partner_ids.ids),
                    ('provider_id', '=', wizard.payment_method_line_id.payment_provider_id.id),
                ])
            else:
                wizard.suitable_payment_token_ids = [Command.clear()]

    @api.depends('payment_method_line_id')
    def _compute_use_electronic_payment_method(self):
        for wizard in self:
            # Get a list of all electronic payment method codes.
            # These codes are comprised of the codes of each payment provider.
            codes = [key for key in dict(self.env['payment.provider']._fields['code']._description_selection(self.env))]
            wizard.use_electronic_payment_method = wizard.payment_method_code in codes

    @api.onchange('can_edit_wizard', 'payment_method_line_id', 'journal_id')
    def _compute_payment_token_id(self):
        codes = [key for key in dict(self.env['payment.provider']._fields['code']._description_selection(self.env))]
        for wizard in self:
            related_partner_ids = (
                    wizard.partner_id
                    | wizard.partner_id.commercial_partner_id
                    | wizard.partner_id.commercial_partner_id.child_ids
            )._origin
            if wizard.can_edit_wizard \
                    and wizard.payment_method_line_id.code in codes \
                    and wizard.journal_id \
                    and related_partner_ids:

                wizard.payment_token_id = self.env['payment.token'].sudo().search([
                    ('company_id', '=', wizard.company_id.id),
                    ('partner_id', 'in', related_partner_ids.ids),
                    ('provider_id.capture_manually', '=', False),
                    ('provider_id', '=', wizard.payment_method_line_id.payment_provider_id.id),
                 ], limit=1)
            else:
                wizard.payment_token_id = False

    # -------------------------------------------------------------------------
    # BUSINESS METHODS
    # -------------------------------------------------------------------------

    def _create_payment_vals_from_wizard(self, batch_result):
        # OVERRIDE
        payment_vals = super()._create_payment_vals_from_wizard(batch_result)
        payment_vals['payment_token_id'] = self.payment_token_id.id
        return payment_vals

```

## File: wizards\account_payment_register_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_payment_register_form_inherit_payment" model="ir.ui.view">
        <field name="name">account.payment.register.form.inherit.payment</field>
        <field name="model">account.payment.register</field>
        <field name="inherit_id" ref="account.view_account_payment_register_form"/>
        <field name="arch" type="xml">
            <field name="payment_method_line_id" position="after">
                <field name="payment_method_code" invisible="1"/>
                <field name="suitable_payment_token_ids" invisible="1"/>
                <field name="use_electronic_payment_method" invisible="1"/>
                <field name="payment_token_id"
                       options="{'no_create': True}"
                       attrs="{'invisible': ['|', ('use_electronic_payment_method', '!=', True), '|', ('can_edit_wizard', '=', False), '&amp;', ('can_group_payments', '=', True), ('group_payment', '=', False)]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: wizards\payment_link_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PaymentLinkWizard(models.TransientModel):
    _inherit = 'payment.link.wizard'

    def _get_additional_link_values(self):
        """ Override of `payment` to add `invoice_id` to the payment link values.

        The other values related to the invoice are directly read from the invoice.

        Note: self.ensure_one()

        :return: The additional payment link values.
        :rtype: dict
        """
        res = super()._get_additional_link_values()
        if self.res_model != 'account.move':
            return res

        # Invoice-related fields are retrieved in the controller.
        return {
            'invoice_id': self.res_id,
        }

```

## File: wizards\payment_link_wizard_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="action_invoice_order_generate_link" model="ir.actions.act_window">
        <field name="name">Generate a Payment Link</field>
        <field name="res_model">payment.link.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="payment.payment_link_wizard_view_form"/>
        <field name="target">new</field>
        <field name="binding_model_id" ref="model_account_move"/>
        <field name="binding_view_types">form</field>
    </record>

</odoo>

```

## File: wizards\payment_refund_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class PaymentRefundWizard(models.TransientModel):
    _name = 'payment.refund.wizard'
    _description = "Payment Refund Wizard"

    payment_id = fields.Many2one(
        string="Payment",
        comodel_name='account.payment',
        readonly=True,
        default=lambda self: self.env.context.get('active_id'),
    )
    transaction_id = fields.Many2one(
        string="Payment Transaction", related='payment_id.payment_transaction_id'
    )
    payment_amount = fields.Monetary(string="Payment Amount", related='payment_id.amount')
    refunded_amount = fields.Monetary(string="Refunded Amount", compute='_compute_refunded_amount')
    amount_available_for_refund = fields.Monetary(
        string="Maximum Refund Allowed", related='payment_id.amount_available_for_refund'
    )
    amount_to_refund = fields.Monetary(
        string="Refund Amount", compute='_compute_amount_to_refund', store=True, readonly=False
    )
    currency_id = fields.Many2one(string="Currency", related='transaction_id.currency_id')
    support_refund = fields.Selection(related='transaction_id.provider_id.support_refund')
    has_pending_refund = fields.Boolean(
        string="Has a pending refund", compute='_compute_has_pending_refund'
    )

    @api.constrains('amount_to_refund')
    def _check_amount_to_refund_within_boundaries(self):
        for wizard in self:
            if not 0 < wizard.amount_to_refund <= wizard.amount_available_for_refund:
                raise ValidationError(_(
                    "The amount to be refunded must be positive and cannot be superior to %s.",
                    wizard.amount_available_for_refund
                ))

    @api.depends('amount_available_for_refund')
    def _compute_refunded_amount(self):
        for wizard in self:
            wizard.refunded_amount = wizard.payment_amount - wizard.amount_available_for_refund

    @api.depends('amount_available_for_refund')
    def _compute_amount_to_refund(self):
        """ Set the default amount to refund to the amount available for refund. """
        for wizard in self:
            wizard.amount_to_refund = wizard.amount_available_for_refund

    @api.depends('payment_id')  # To always trigger the compute
    def _compute_has_pending_refund(self):
        for wizard in self:
            pending_refunds_count = self.env['payment.transaction'].search_count([
                ('source_transaction_id', '=', wizard.payment_id.payment_transaction_id.id),
                ('operation', '=', 'refund'),
                ('state', 'in', ['draft', 'pending', 'authorized']),
            ])
            wizard.has_pending_refund = pending_refunds_count > 0

    def action_refund(self):
        for wizard in self:
            wizard.transaction_id.action_refund(amount_to_refund=wizard.amount_to_refund)

```

## File: wizards\payment_refund_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_refund_wizard_view_form" model="ir.ui.view">
        <field name="name">payment.refund.wizard.form</field>
        <field name="model">payment.refund.wizard</field>
        <field name="arch" type="xml">
            <form string="Refund">
                <field name="has_pending_refund" invisible="1"/>
                <div class="alert alert-warning"
                     id="alert_draft_refund_tx"
                     role="alert"
                     attrs="{'invisible': [('has_pending_refund', '=', False)]}">
                    <p>
                        <strong>Warning!</strong> There is a refund pending for this payment.
                        Wait a moment for it to be processed. If the refund is still pending in a
                        few minutes, please check your payment provider configuration.
                    </p>
                </div>
                <group>
                    <group>
                        <field name="payment_id" invisible="1"/>
                        <field name="transaction_id" invisible="1"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="support_refund" invisible="1"/>
                        <field name="payment_amount"/>
                        <field name="refunded_amount"
                               attrs="{'invisible': [('refunded_amount', '&lt;=', 0)]}"/>
                        <field name="amount_available_for_refund"/>
                        <field name="amount_to_refund"
                               attrs="{'readonly': [('support_refund', '=', 'full_only')]}"/>
                    </group>
                </group>
                <footer>
                    <button string="Refund" type="object" name="action_refund" class="btn-primary"/>
                    <button string="Close" special="cancel" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import payment_link_wizard
from . import payment_refund_wizard

```

