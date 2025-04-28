# Odoo Module: account_payment

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizards


def post_init_hook(env):
    """ Create `account.payment.method` records for the installed payment providers. """
    PaymentProvider = env['payment.provider']
    installed_providers = PaymentProvider.search([('module_id.state', '=', 'installed')])
    for code in set(installed_providers.mapped('code')):
        PaymentProvider._setup_payment_method(code)


def uninstall_hook(env):
    """ Delete `account.payment.method` records created for the installed payment providers. """
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
        'data/ir_config_parameter.xml',
        'data/onboarding_data.xml',

        'security/ir.model.access.csv',
        'security/ir_rules.xml',

        'views/account_payment_menus.xml',
        'views/account_portal_templates.xml',
        'views/account_move_views.xml',
        'views/account_journal_views.xml',
        'views/account_payment_views.xml',
        'views/payment_form_templates.xml',
        'views/payment_provider_views.xml',
        'views/payment_transaction_views.xml',

        'wizards/account_payment_register_views.xml',
        'wizards/payment_link_wizard_views.xml',
        'wizards/payment_refund_wizard_views.xml',
        'wizards/res_config_settings_views.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'account_payment/static/src/js/payment_form.js',
            'account_payment/static/src/js/portal_invoice_page_payment.js',
            'account_payment/static/src/js/portal_my_invoices_payment.js',
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

        logged_in = not request.env.user._is_public()
        partner_sudo = request.env.user.partner_id if logged_in else invoice_sudo.partner_id
        self._validate_transaction_kwargs(kwargs, additional_allowed_keys={'name_next_installment'})
        return self._process_transaction(partner_sudo.id, invoice_sudo.currency_id.id, [invoice_id], False, **kwargs)

    @route('/invoice/transaction/overdue', type='json', auth='public')
    def overdue_invoices_transaction(self, payment_reference, **kwargs):
        """ Create a draft transaction for overdue invoices and return its processing values.

        :param str payment_reference: The reference to the current payment
        :param dict kwargs: Locally unused data passed to `_create_transaction`
        :return: The mandatory values for the processing of the transaction
        :rtype: dict
        :raise: ValidationError if the user is not logged in, or all the overdue invoices don't share the same currency.
        """
        logged_in = not request.env.user._is_public()
        if not logged_in:
            raise ValidationError(_("Please log in to pay your overdue invoices"))
        partner = request.env.user.partner_id
        overdue_invoices = request.env['account.move'].search(self._get_overdue_invoices_domain())
        currencies = overdue_invoices.mapped('currency_id')
        if not all(currency == currencies[0] for currency in currencies):
            raise ValidationError(_("Impossible to pay all the overdue invoices if they don't share the same currency."))
        self._validate_transaction_kwargs(kwargs)
        return self._process_transaction(partner.id, currencies[0].id, overdue_invoices.ids, payment_reference, **kwargs)

    def _process_transaction(self, partner_id, currency_id, invoice_ids, payment_reference, **kwargs):
        kwargs.update({
            'currency_id': currency_id,
            'partner_id': partner_id,
            'reference_prefix': payment_reference,
        })  # Inject the create values taken from the invoice into the kwargs.
        tx_sudo = self._create_transaction(
            custom_create_values={
                'invoice_ids': [Command.set(invoice_ids)],
            },
            **kwargs,
        )

        return tx_sudo._get_processing_values()

    # Payment overrides

    @route()
    def payment_pay(self, *args, amount=None, invoice_id=None, access_token=None, **kwargs):
        """ Override of `payment` to replace the missing transaction values by that of the invoice.

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
                # To display on the payment form; will be later overwritten when creating the tx.
                'reference': invoice_sudo.name,
                # To fix the currency if incorrect and avoid mismatches when creating the tx.
                'currency_id': invoice_sudo.currency_id.id,
                # To fix the partner if incorrect and avoid mismatches when creating the tx.
                'partner_id': invoice_sudo.partner_id.id,
                'company_id': invoice_sudo.company_id.id,
                'invoice_id': invoice_id,
            })
        return super().payment_pay(*args, amount=amount, access_token=access_token, **kwargs)

    def _get_extra_payment_form_values(self, invoice_id=None, access_token=None, **kwargs):
        """ Override of `payment` to reroute the payment flow to the portal view of the invoice.

        :param str invoice_id: The invoice for which a payment id made, as an `account.move` id.
        :param str access_token: The portal or payment access token, respectively if we are in a
                                 portal or payment link flow.
        :return: The extended rendering context values.
        :rtype: dict
        """
        form_values = super()._get_extra_payment_form_values(
            invoice_id=invoice_id, access_token=access_token, **kwargs
        )
        if invoice_id:
            invoice_id = self._cast_as_int(invoice_id)

            try:  # Check document access against what could be a portal access token.
                invoice_sudo = self._document_check_access('account.move', invoice_id, access_token)
            except AccessError:  # It is a payment access token computed on the payment context.
                if not payment_utils.check_access_token(
                    access_token,
                    kwargs.get('partner_id'),
                    kwargs.get('amount'),
                    kwargs.get('currency_id'),
                ):
                    raise
                invoice_sudo = request.env['account.move'].sudo().browse(invoice_id)

            # Interrupt the payment flow if the invoice has been canceled.
            if invoice_sudo.state == 'cancel':
                form_values['amount'] = 0.0

            # Reroute the next steps of the payment flow to the portal view of the invoice.
            form_values.update({
                'transaction_route': f'/invoice/transaction/{invoice_id}',
                'landing_route': f'{invoice_sudo.access_url}'
                                 f'?access_token={invoice_sudo._portal_ensure_token()}',
                'access_token': invoice_sudo.access_token,
            })
        return form_values

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, http
from odoo.exceptions import AccessError, MissingError, ValidationError
from odoo.http import request

from odoo.addons.account.controllers import portal
from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.controllers.portal import PaymentPortal


class PortalAccount(portal.PortalAccount, PaymentPortal):

    def _invoice_get_page_view_values(self, invoice, access_token, payment=False, **kwargs):
        # EXTENDS account

        values = super()._invoice_get_page_view_values(invoice, access_token, **kwargs)

        if not invoice._has_to_be_paid():
            # Do not compute payment-related stuff if given invoice doesn't have to be paid.
            return {
                **values,
                'payment': payment,  # We want to show the dialog even when everything has been paid (with a custom message)
            }

        epd = values.get('epd_discount_amount_currency', 0.0)
        discounted_amount = invoice.amount_residual - epd

        common_view_values = self._get_common_page_view_values(
            invoices_data={
                'partner': invoice.partner_id,
                'company': invoice.company_id,
                'total_amount': invoice.amount_total,
                'currency': invoice.currency_id,
                'amount_residual': discounted_amount,
                'landing_route': invoice.get_portal_url(),
                'transaction_route': f'/invoice/transaction/{invoice.id}',
            },
            access_token=access_token,
            **kwargs)

        amount_custom = float(kwargs['amount']) if kwargs.get('amount') else 0.0
        values |= {
            **common_view_values,
            'amount_custom': amount_custom,
            'payment': payment,
            'invoice_id': invoice.id,
            'invoice_name': invoice.name,
            'show_epd': epd,
        }
        return values

    @http.route(['/my/invoices/overdue'], type='http', auth='public', methods=['GET'], website=True, sitemap=False)
    def portal_my_overdue_invoices(self, access_token=None, **kw):
        try:
            request.env['account.move'].check_access('read')
        except (AccessError, MissingError):
            return request.redirect('/my')

        overdue_invoices = request.env['account.move'].search(self._get_overdue_invoices_domain())

        values = self._overdue_invoices_get_page_view_values(overdue_invoices, **kw)
        return request.render("account_payment.portal_overdue_invoices_page", values) if 'payment' in values else request.redirect('/my/invoices')

    def _overdue_invoices_get_page_view_values(self, overdue_invoices, **kwargs):
        values = {'page_name': 'overdue_invoices'}

        if len(overdue_invoices) == 0:
            return values

        first_invoice = overdue_invoices[0]
        partner = first_invoice.partner_id
        company = first_invoice.company_id
        currency = first_invoice.currency_id

        if any(invoice.partner_id != partner for invoice in overdue_invoices):
            raise ValidationError(_("Overdue invoices should share the same partner."))
        if any(invoice.company_id != company for invoice in overdue_invoices):
            raise ValidationError(_("Overdue invoices should share the same company."))
        if any(invoice.currency_id != currency for invoice in overdue_invoices):
            raise ValidationError(_("Overdue invoices should share the same currency."))

        total_amount = sum(overdue_invoices.mapped('amount_total'))
        amount_residual = sum(overdue_invoices.mapped('amount_residual'))
        batch_name = company.get_next_batch_payment_communication() if len(overdue_invoices) > 1 else first_invoice.name
        values.update({
            'payment': {
                'date': fields.Date.today(),
                'reference': batch_name,
                'amount': total_amount,
                'currency': currency,
            },
            'amount': total_amount,
        })

        common_view_values = self._get_common_page_view_values(
            invoices_data={
                'partner': partner,
                'company': company,
                'total_amount': total_amount,
                'currency': currency,
                'amount_residual': amount_residual,
                'payment_reference': batch_name,
                'landing_route': '/my/invoices/',
                'transaction_route': '/invoice/transaction/overdue',
            },
            **kwargs)
        values |= common_view_values
        return values

    def _get_common_page_view_values(self, invoices_data, access_token=None, **kwargs):
        logged_in = not request.env.user._is_public()
        # We set partner_id to the partner id of the current user if logged in, otherwise we set it
        # to the invoice partner id. We do this to ensure that payment tokens are assigned to the
        # correct partner and to avoid linking tokens to the public user.
        partner_sudo = request.env.user.partner_id if logged_in else invoices_data['partner']
        invoice_company = invoices_data['company'] or request.env.company

        availability_report = {}
        # Select all the payment methods and tokens that match the payment context.
        providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            invoice_company.id,
            partner_sudo.id,
            invoices_data['total_amount'],
            currency_id=invoices_data['currency'].id,
            report=availability_report,
        )  # In sudo mode to read the fields of providers and partner (if logged out).
        payment_methods_sudo = request.env['payment.method'].sudo()._get_compatible_payment_methods(
            providers_sudo.ids,
            partner_sudo.id,
            currency_id=invoices_data['currency'].id,
            report=availability_report,
        )  # In sudo mode to read the fields of providers.
        tokens_sudo = request.env['payment.token'].sudo()._get_available_tokens(
            providers_sudo.ids, partner_sudo.id
        )  # In sudo mode to read the partner's tokens (if logged out) and provider fields.

        # Make sure that the partner's company matches the invoice's company.
        company_mismatch = not PaymentPortal._can_partner_pay_in_company(
            partner_sudo, invoice_company
        )

        portal_page_values = {
            'company_mismatch': company_mismatch,
            'expected_company': invoice_company,
        }
        payment_form_values = {
            'show_tokenize_input_mapping': PaymentPortal._compute_show_tokenize_input_mapping(
                providers_sudo
            ),
        }
        payment_context = {
            'currency': invoices_data['currency'],
            'partner_id': partner_sudo.id,
            'providers_sudo': providers_sudo,
            'payment_methods_sudo': payment_methods_sudo,
            'tokens_sudo': tokens_sudo,
            'availability_report': availability_report,
            'transaction_route': invoices_data['transaction_route'],
            'landing_route': invoices_data['landing_route'],
            'access_token': access_token,
            'payment_reference': invoices_data.get('payment_reference', False),
        }
        # Merge the dictionaries while allowing the redefinition of keys.
        values = portal_page_values | payment_form_values | payment_context | self._get_extra_payment_form_values(**kwargs)
        return values

    @http.route()
    def portal_my_invoice_detail(self, invoice_id, payment_token=None, amount=None, **kw):
        # EXTENDS account

        # If we have a custom payment amount, make sure it hasn't been tampered with
        if amount and not payment_utils.check_access_token(payment_token, invoice_id, amount):
            return request.redirect('/my')
        return super().portal_my_invoice_detail(invoice_id, amount=amount, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal
from . import payment
```

## File: data\ir_config_parameter.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <record id="enable_portal_payment" model="ir.config_parameter" forcecreate="0">
        <field name="key">account_payment.enable_portal_payment</field>
        <field name="value">True</field>
    </record>

</odoo>

```

## File: data\onboarding_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="onboarding_onboarding_step_payment_provider" model="onboarding.onboarding.step">
            <field name="title">Online Payments</field>
            <field name="description">Enable credit &amp; debit card payments supported by Stripe.</field>
            <field name="button_text">Activate Stripe</field>
            <field name="panel_step_open_action_name">action_open_step_payment_provider</field>
            <field name="step_image" type="base64" file="base/static/img/onboarding_bank-account.png"></field>
            <field name="step_image_filename">onboarding_bank-account.png</field>
            <field name="step_image_alt">Onboarding Online Payments</field>
        </record>
    </data>
</odoo>

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
from odoo.tools import format_date, str2bool
from odoo.tools.translate import _

from odoo.addons.payment import utils as payment_utils


class AccountMove(models.Model):
    _inherit = 'account.move'

    transaction_ids = fields.Many2many(
        string="Transactions", comodel_name='payment.transaction',
        relation='account_invoice_transaction_rel', column1='invoice_id', column2='transaction_id',
        readonly=True, copy=False)
    authorized_transaction_ids = fields.Many2many(
        string="Authorized Transactions", comodel_name='payment.transaction',
        compute='_compute_authorized_transaction_ids', readonly=True, copy=False,
        compute_sudo=True)
    transaction_count = fields.Integer(
        string="Transaction Count", compute='_compute_transaction_count'
    )
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
    def _compute_transaction_count(self):
        for invoice in self:
            invoice.transaction_count = len(invoice.transaction_ids)

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
        pending_transactions = transactions.filtered(
            lambda tx: tx.state in {'pending', 'authorized'}
                       and tx.provider_code not in {'none', 'custom'})
        enabled_feature = str2bool(
            self.env['ir.config_parameter'].sudo().get_param(
                'account_payment.enable_portal_payment'
            )
        )
        return enabled_feature and bool(
            (self.amount_residual or not transactions)
            and self.state == 'posted'
            and self.payment_state in ('not_paid', 'in_payment', 'partial')
            and not self.currency_id.is_zero(self.amount_residual)
            and self.amount_total
            and self.move_type == 'out_invoice'
            and not pending_transactions
        )

    def _get_online_payment_error(self):
        """
        Returns the appropriate error message to be displayed if _has_to_be_paid() method returns False.
        """
        self.ensure_one()
        transactions = self.transaction_ids.filtered(lambda tx: tx.state in ('pending', 'authorized', 'done'))
        pending_transactions = transactions.filtered(
            lambda tx: tx.state in {'pending', 'authorized'}
                       and tx.provider_code not in {'none', 'custom'})
        enabled_feature = str2bool(
            self.env['ir.config_parameter'].sudo().get_param(
                'account_payment.enable_portal_payment'
            )
        )
        errors = []
        if not enabled_feature:
            errors.append(_("This invoice cannot be paid online."))
        if transactions or self.currency_id.is_zero(self.amount_residual):
            errors.append(_("There is no amount to be paid."))
        if self.state != 'posted':
            errors.append(_("This invoice isn't posted."))
        if self.currency_id.is_zero(self.amount_residual):
            errors.append(_("This invoice has already been paid."))
        if self.move_type != 'out_invoice':
            errors.append(_("This is not an outgoing invoice."))
        if pending_transactions:
            errors.append(_("There are pending transactions for this invoice."))
        return '\n'.join(errors)

    def get_portal_last_transaction(self):
        self.ensure_one()
        return self.with_context(active_test=False).transaction_ids.sudo()._get_last()

    def payment_action_capture(self):
        """ Capture all transactions linked to this invoice. """
        self.ensure_one()
        payment_utils.check_rights_on_recordset(self)

        # In sudo mode to bypass the checks on the rights on the transactions.
        return self.transaction_ids.sudo().action_capture()

    def payment_action_void(self):
        """ Void all transactions linked to this invoice. """
        payment_utils.check_rights_on_recordset(self)

        # In sudo mode to bypass the checks on the rights on the transactions.
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
        next_payment_values = self._get_invoice_next_payment_values()
        amount_max = next_payment_values.get('amount_due')
        additional_info = {}
        open_installments = []
        installment_state = next_payment_values.get('installment_state')
        next_amount_to_pay = next_payment_values.get('next_amount_to_pay')
        if installment_state in ('next', 'overdue'):
            open_installments = []
            for installment in next_payment_values.get('not_reconciled_installments'):
                data = {
                    'type': installment['type'],
                    'number': installment['number'],
                    'amount': installment['amount_residual_currency_unsigned'],
                    'date_maturity': format_date(self.env, installment['date_maturity']),
                }
                open_installments.append(data)

        elif installment_state == 'epd':
            amount_max = next_amount_to_pay  # with epd, next_amount_to_pay is the invoice amount residual
            additional_info.update({
                'has_eligible_epd': True,
                'discount_date': next_payment_values.get('discount_date')
            })

        return {
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'open_installments': open_installments,
            'amount': next_amount_to_pay,
            'amount_max': amount_max,
            **additional_info
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
            payment_method = (
                tx_sudo.payment_method_id.primary_payment_method_id
                or tx_sudo.payment_method_id
            )
            if (
                tx_sudo  # The payment was created by a transaction.
                and tx_sudo.provider_id.support_refund != 'none'
                and payment_method.support_refund != 'none'
                and tx_sudo.operation != 'refund'
            ):
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
            if payment.use_electronic_payment_method:
                payment.suitable_payment_token_ids = self.env['payment.token'].sudo().search([
                    *self.env['payment.token']._check_company_domain(payment.company_id),
                    ('provider_id.capture_manually', '=', False),
                    ('partner_id', '=', payment.partner_id.id),
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
            groupby=['source_payment_id'],
            aggregates=['__count']
        )
        data = {source_payment.id: count for source_payment, count in rg_data}
        for payment in self:
            payment.refunds_count = data.get(payment.id, 0)

    #=== ONCHANGE METHODS ===#

    @api.onchange('partner_id', 'payment_method_line_id', 'journal_id')
    def _onchange_set_payment_token_id(self):
        codes = [key for key in dict(self.env['payment.provider']._fields['code']._description_selection(self.env))]
        if not (self.payment_method_code in codes and self.partner_id and self.journal_id):
            self.payment_token_id = False
            return

        self.payment_token_id = self.env['payment.token'].sudo().search([
            *self.env['payment.token']._check_company_domain(self.company_id),
            ('partner_id', '=', self.partner_id.id),
            ('provider_id.capture_manually', '=', False),
            ('provider_id', '=', self.payment_method_line_id.payment_provider_id.id),
         ], limit=1)  # In sudo mode to read the provider fields.

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
        transactions._post_process()
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
            action['view_mode'] = 'list,form'
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
            'payment_method_id': self.payment_token_id.payment_method_id.id,
            'reference': self.env['payment.transaction']._compute_reference(
                self.payment_token_id.provider_id.code, prefix=self.memo
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
                'type': ('bank',),
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
        domain="[('code', '=', code)]",
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

## File: models\onboarding_onboarding_step.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class OnboardingStep(models.Model):
    _inherit = 'onboarding.onboarding.step'

    @api.model
    def action_open_step_payment_provider(self):
        self.env.company.payment_onboarding_payment_method = 'stripe'
        menu = self.env.ref('account_payment.payment_provider_menu', raise_if_not_found=False)
        menu_id = menu.id if menu else None
        return self.env.company._run_payment_onboarding_step(menu_id)

    @api.model
    def action_validate_step_payment_provider(self):
        validation_response = super().action_validate_step_payment_provider()
        self.action_validate_step("account_payment.onboarding_onboarding_step_payment_provider")
        return validation_response

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    journal_id = fields.Many2one(
        string="Payment Journal",
        help="The journal in which the successful transactions are posted.",
        comodel_name='account.journal',
        compute='_compute_journal_id',
        inverse='_inverse_journal_id',
        check_company=True,
        domain='[("type", "=", "bank")]',
        copy=False,
    )

    #=== COMPUTE METHODS ===#

    def _ensure_payment_method_line(self, allow_create=True):
        self.ensure_one()
        if not self.id:
            return

        default_payment_method = self._get_provider_payment_method(self._get_code())
        if not default_payment_method:
            return

        pay_method_line = self.env['account.payment.method.line'].search([
            ('payment_provider_id', '=', self.id),
            ('journal_id', '!=', False),
        ], limit=1)

        if not self.journal_id:
            if pay_method_line:
                pay_method_line.unlink()
                return

        if not pay_method_line:
            pay_method_line = self.env['account.payment.method.line'].search(
                [
                    *self.env['account.payment.method.line']._check_company_domain(self.company_id),
                    ('code', '=', self._get_code()),
                    ('payment_provider_id', '=', False),
                    ('journal_id', '!=', False),
                ],
                limit=1,
            )
        if pay_method_line:
            pay_method_line.payment_provider_id = self
            pay_method_line.journal_id = self.journal_id
            pay_method_line.name = self.name
        elif allow_create:
            create_values = {
                'name': self.name,
                'payment_method_id': default_payment_method.id,
                'journal_id': self.journal_id.id,
                'payment_provider_id': self.id,
            }
            pay_method_line_same_code = self.env['account.payment.method.line'].search(
                [
                    *self.env['account.payment.method.line']._check_company_domain(self.company_id),
                    ('code', '=', self._get_code()),
                ],
                limit=1,
            )
            if pay_method_line_same_code:
                create_values['payment_account_id'] = pay_method_line_same_code.payment_account_id.id
            self.env['account.payment.method.line'].create(create_values)

    @api.depends('code', 'state', 'company_id')
    def _compute_journal_id(self):
        for provider in self:
            pay_method_line = self.env['account.payment.method.line'].search([
                ('payment_provider_id', '=', provider._origin.id),
                ('journal_id', '!=', False),
            ], limit=1)

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

    def _check_existing_payment(self, payment_method):
        existing_payment_count = self.env['account.payment'].search_count([('payment_method_id', '=', payment_method.id)], limit=1)
        return bool(existing_payment_count)

    @api.model
    def _remove_provider(self, code, **kwargs):
        """ Override of `payment` to delete the payment method of the provider. """
        payment_method = self._get_provider_payment_method(code)
        # If the payment method is used by any payments, we block the uninstallation of the module.
        if self._check_existing_payment(payment_method):
            raise UserError(_("You cannot uninstall this module as payments using this payment method already exist."))
        super()._remove_provider(code, **kwargs)
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
            action['view_mode'] = 'list,form'
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
                prefix = separator.join(invoices.filtered(lambda inv: inv.name).mapped('name'))
                if name := values.get('name_next_installment'):
                    prefix = name
                return prefix
        return super()._compute_reference_prefix(provider_code, separator, **values)

    #=== BUSINESS METHODS - POST-PROCESSING ===#

    def _post_process(self):
        """ Override of `payment` to add account-specific logic to the post-processing.

        In particular, for confirmed transactions we write a message in the chatter with the payment
        and transaction references, post relevant fiscal documents, and create missing payments. For
        cancelled transactions, we cancel the payment.
        """
        super()._post_process()
        for tx in self.filtered(lambda t: t.state == 'done'):
            # Validate invoices automatically once the transaction is confirmed.
            self.invoice_ids.filtered(lambda inv: inv.state == 'draft').action_post()

            # Create and post missing payments.
            # As there is nothing to reconcile for validation transactions, no payment is created
            # for them. This is also true for validations with or without a validity check (transfer
            # of a small amount with immediate refund) because validation amounts are not included
            # in payouts. As the reconciliation is done in the child transactions for partial voids
            # and captures, no payment is created for their source transactions either.
            if (
                tx.operation != 'validation'
                and not tx.payment_id
                and not any(child.state in ['done', 'cancel'] for child in tx.child_transaction_ids)
            ):
                tx.with_company(tx.company_id)._create_payment()

            if tx.payment_id:
                message = _(
                    "The payment related to the transaction with reference %(ref)s has been"
                    " posted: %(link)s",
                    ref=tx.reference,
                    link=tx.payment_id._get_html_link(),
                )
                tx._log_message_on_linked_documents(message)
        for tx in self.filtered(lambda t: t.state == 'cancel'):
            tx.payment_id.action_cancel()

    def _create_payment(self, **extra_create_values):
        """Create an `account.payment` record for the current transaction.

        If the transaction is linked to some invoices, their reconciliation is done automatically.

        Note: self.ensure_one()

        :param dict extra_create_values: Optional extra create values
        :return: The created payment
        :rtype: recordset of `account.payment`
        """
        self.ensure_one()

        reference = (f'{self.reference} - '
                     f'{self.partner_id.display_name or ""} - '
                     f'{self.provider_reference or ""}'
                    )

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
            'memo': reference,
            'write_off_line_vals': [],
            'invoice_ids': self.invoice_ids,
            **extra_create_values,
        }

        for invoice in self.invoice_ids:
            if invoice.state != 'posted':
                continue
            next_payment_values = invoice._get_invoice_next_payment_values()
            if next_payment_values['installment_state'] == 'epd' and self.amount == next_payment_values['amount_due']:
                aml = next_payment_values['epd_line']
                epd_aml_values_list = [({
                    'aml': aml,
                    'amount_currency': -aml.amount_residual_currency,
                    'balance': -aml.balance,
                })]
                open_balance = next_payment_values['epd_discount_amount']
                early_payment_values = self.env['account.move']._get_invoice_counterpart_amls_for_early_payment_discount(epd_aml_values_list, open_balance)
                for aml_values_list in early_payment_values.values():
                    if (aml_values_list):
                        aml_vl = aml_values_list[0]
                        aml_vl['partner_id'] = invoice.partner_id.id
                        payment_values['write_off_line_vals'] += [aml_vl]
                break

        payment = self.env['account.payment'].create(payment_values)
        payment.action_post()

        # Track the payment to make a one2one.
        self.payment_id = payment

        # Reconcile the payment with the source transaction's invoices in case of a partial capture.
        if self.operation == self.source_transaction_id.operation:
            invoices = self.source_transaction_id.invoice_ids
        else:
            invoices = self.invoice_ids
        if invoices:
            invoices.filtered(lambda inv: inv.state == 'draft').action_post()

            (payment.move_id.line_ids + invoices.line_ids).filtered(
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
        author = self.env.user.partner_id if self.env.uid == SUPERUSER_ID else self.partner_id
        if self.source_transaction_id:
            for invoice in self.source_transaction_id.invoice_ids:
                invoice.message_post(body=message, author_id=author.id)
            payment_id = self.source_transaction_id.payment_id
            if payment_id:
                payment_id.message_post(body=message, author_id=author.id)
        for invoice in self._get_invoices_to_notify():
            invoice.message_post(body=message, author_id=author.id)

    def _get_invoices_to_notify(self):
        """ Return the invoices on which to log payment-related messages. """
        return self.invoice_ids

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import account_move
from . import account_payment
from . import account_payment_method
from . import account_payment_method_line
from . import onboarding_onboarding_step
from . import payment_provider
from . import payment_transaction

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
payment_link_wizard,payment.link.wizard,payment.model_payment_link_wizard,account.group_account_invoice,1,1,1,0
payment_refund_wizard,payment.refund.wizard,model_payment_refund_wizard,account.group_account_invoice,1,1,1,0
payment_transaction_user,payment.transaction.user,model_payment_transaction,account.group_account_invoice,1,1,1,0

```

## File: security\ir_rules.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

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
/** @odoo-module **/

import PaymentForm from "@payment/js/payment_form";

PaymentForm.include({
    /**
     * Set whether we are paying an installment before submitting.
     *
     * @override method from payment.payment_form
     * @private
     * @param {Event} ev
     * @returns {void}
     */
    async _submitForm(ev) {
        ev.stopPropagation();
        ev.preventDefault();

        const paymentDialog = this.el.closest("#pay_with");
        const chosenPaymentDetails = paymentDialog
            ? paymentDialog.querySelector(".o_btn_payment_tab.active")
            : null;
        if (chosenPaymentDetails){
            if (chosenPaymentDetails.id === "o_payment_installments_tab") {
                this.paymentContext.amount = parseFloat(this.paymentContext.invoiceNextAmountToPay);
            } else {
                this.paymentContext.amount = parseFloat(this.paymentContext.invoiceAmountDue);
            }
        }
        await this._super(...arguments);
    },

        /**
     * Prepare the params for the RPC to the transaction route.
     *
     * @override method from payment.payment_form
     * @private
     * @return {object} The transaction route params.
     */
        _prepareTransactionRouteParams() {
            const transactionRouteParams =  this._super(...arguments);
            transactionRouteParams.payment_reference = this.paymentContext.paymentReference;
            return transactionRouteParams;
        },
});

```

## File: static\src\js\portal_invoice_page_payment.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.PortalInvoicePagePayment = publicWidget.Widget.extend({
    selector: "#portal_pay",

    /**
     * Show the payment dialog when the context parameter is set.
     *
     * @returns {void}
     */
    start() {
        if (this.el.dataset.payment) {
            const paymentDialog = new Modal("#pay_with");
            paymentDialog.show();
        }
        return this._super(...arguments);
    },
});

```

## File: static\src\js\portal_my_invoices_payment.js

```javascript
/** @odoo-module **/

import {_t} from "@web/core/l10n/translation";
import {deserializeDateTime} from "@web/core/l10n/dates";
import publicWidget from "@web/legacy/js/public/public_widget";

const {DateTime} = luxon;

publicWidget.registry.PortalMyInvoicesPaymentList = publicWidget.Widget.extend({
    selector: ".o_portal_my_doc_table",

    start() {
        this._setDueDateLabel();
        return this._super(...arguments);
    },

    _setDueDateLabel() {
        const dueDateLabels = this.el.querySelectorAll(".o_portal_invoice_due_date");
        const today = DateTime.now().startOf("day");
        dueDateLabels.forEach((label) => {
            const dateTime = deserializeDateTime(label.getAttribute("datetime")).startOf('day');
            const diff = dateTime.diff(today).as("days");

            let dueDateLabel = "";

            if (diff === 0) {
                dueDateLabel = _t("due today");
            } else if (diff > 0) {
                dueDateLabel = _t("due in %s day(s)", Math.abs(diff).toFixed());
            } else {
                dueDateLabel = _t("%s day(s) overdue", Math.abs(diff).toFixed());
            }
            // We use `.createTextNode()` to escape possible HTML in translations (XSS)
            label.replaceChildren(document.createTextNode(dueDateLabel));
        });
    },
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
                <field name="code" column_invisible="True"/>
                <field name="payment_provider_id" options="{'no_open': True, 'no_create': True}" optional="hide"/>
                <field name="payment_provider_state" column_invisible="True"/>
                <button name="action_open_provider_form"
                        type="object"
                        string="SETUP"
                        class="float-end btn-secondary"
                        invisible="not payment_provider_id"
                        groups="base.group_system"/>
            </xpath>
            <xpath expr="//field[@name='inbound_payment_method_line_ids']/list" position="attributes">
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
                <attribute name="invisible" separator="or" add="authorized_transaction_ids"/>
            </xpath>
            <xpath expr="//button[@id='account_invoice_payment_secondary_btn']" position="attributes">
                <attribute name="invisible" separator="or" add="authorized_transaction_ids"/>
            </xpath>
            <xpath expr="//button[@id='account_invoice_payment_btn']" position="after">
                <field name="authorized_transaction_ids" invisible="1"/>
                <button name="payment_action_capture" type="object"
                        groups="account.group_account_invoice"
                        string="Capture Transaction" class="oe_highlight" data-hotkey="shift+g"
                        invisible="move_type not in ('out_invoice', 'out_refund', 'in_invoice', 'in_refund') or state != 'posted' or not authorized_transaction_ids"/>
                <button name="payment_action_void" type="object"
                        groups="account.group_account_invoice"
                        string="Void Transaction" data-hotkey="shift+v"
                        confirm="Are you sure you want to void the authorized transaction? This action can't be undone."
                        invisible="move_type not in ('out_invoice', 'out_refund', 'in_invoice', 'in_refund') or state != 'posted' or not authorized_transaction_ids"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_view_payment_transactions" type="object"
                        groups="account.group_account_invoice"
                        class="oe_stat_button" icon="fa-money"
                        invisible="transaction_count == 0">
                        <field name="transaction_count" widget="statinfo" string="Payment Transaction"/>
                </button>
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
    <menuitem action="payment.action_payment_method"
        id="payment_method_menu"
        parent="account.root_payment_menu"
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
                    invisible="amount_available_for_refund &lt;= 0"
                    class="btn-secondary"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_view_refunds"
                    type="object"
                    class="oe_stat_button"
                    icon="fa-money"
                    invisible="refunds_count == 0">
                    <field name="refunds_count" widget="statinfo" string="Refunds"/>
                </button>
            </xpath>
            <xpath expr='//group[2]' position="inside">
                <field name="source_payment_id" invisible="not source_payment_id"/>
                <field name="payment_transaction_id" groups="base.group_no_one" invisible="not use_electronic_payment_method"/>
            </xpath>
            <field name="payment_method_line_id" position="after">
                <field name="payment_method_code" invisible="1"/>
                <field name="suitable_payment_token_ids" invisible="1"/>
                <field name="use_electronic_payment_method" invisible="1"/>
                <field name="payment_token_id" options="{'no_create': True}"
                    invisible="not use_electronic_payment_method"
                    readonly="state != 'draft'"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\account_portal_templates.xml

```xml
<odoo>
    <template id="portal_my_home_overdue_invoice" name="Portal layout: overdue invoices" inherit_id="portal.portal_breadcrumbs" priority="30">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <t t-if="page_name == 'overdue_invoices'">
                <li t-attf-class="breadcrumb-item active">
                    <a t-attf-href="/my/invoices?{{ keep_query() }}">Invoices &amp; Bills</a>
                </li>
                <li class="breadcrumb-item active">
                    <t t-out="payment['reference']"/>
                </li>
            </t>
        </xpath>

        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="after">
            <a t-if="page_name == 'invoice' and not invoice and filterby != 'bills' and overdue_invoice_count > 0"
                class='btn btn-primary'
                href='/my/invoices/overdue'>
                Pay overdue
            </a>
        </xpath>
    </template>

    <template id="account_payment.portal_docs_entry" inherit_id="portal.portal_docs_entry">
        <xpath expr="//a" position="inside">
            <button
                t-if="show_pay_overdue_invoices_button"
                onclick="event.preventDefault(); window.location.href='/my/invoices/overdue'"
                class="btn btn-secondary ms-auto">
                Pay Now
            </button>
        </xpath>
    </template>

    <template id="portal_my_home_account_payment" name="Overdue invoices" customize_show="True" inherit_id="portal.portal_my_home" priority="20">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
            <t t-set="portal_alert_category_enable" t-value="True"/>
        </xpath>
        <div id="portal_alert_category" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="bg_color" t-value="'alert alert-primary align-items-center'"/>
                <t t-set="placeholder_count" t-value="'overdue_invoice_count'"/>
                <t t-set="title">Invoices to pay</t>
                <t t-set="url" t-value="'/my/invoices?filterby=overdue_invoices'"/>
                <t t-set="show_count" t-value="True"/>
                <t t-set="show_pay_overdue_invoices_button" t-value="True"/>
            </t>
        </div>
    </template>

    <template id="portal_overdue_invoices_page" name="Overdue payments">
        <t t-call="portal.portal_layout">
            <div class="row justify-content-center my-3">
                <div class="col-lg-7">
                    <div class="text-bg-light row mx-0 rounded">
                        <t t-call="payment.summary_item">
                            <t t-set="name" t-value="'amount'"/>
                            <t t-set="label">Amount</t>
                            <t t-set="value" t-value="abs(payment['amount'])"/>
                            <t t-set="options"
                                t-value="{'widget': 'monetary', 'display_currency': currency}"/>
                        </t>
                        <t t-call="payment.summary_item">
                            <t t-set="name" t-value="'reference'"/>
                            <t t-set="label">Reference</t>
                            <t t-set="value" t-value="payment['reference']"/>
                            <t t-set="include_separator" t-value="True"/>
                        </t>
                    </div>
                    <div class="mt-4">
                        <t t-call="payment.form"/>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="portal_my_invoices_payment" name="Payment on My Invoices" inherit_id="account.portal_my_invoices">
        <xpath expr="//t[@t-call='portal.portal_table']/thead/tr/th[last()]" position="after">
            <th class="d-none d-lg-table-cell"/>
            <th/>
        </xpath>
        <xpath expr="//t[@t-foreach='invoices']/tr/td[last()]" position="after">
            <td class="d-none d-lg-table-cell text-center">
                <t t-if="invoice_data.get('installment_state', False) in ('next', 'overdue')">
                    <span
                        t-attf-class="{{'text-danger' if invoice_data['installment_state'] == 'overdue' else ''}}"
                        t-out="invoice_data['next_amount_to_pay']"
                        t-options="{'widget': 'monetary', 'display_currency': invoice.currency_id}"
                    />
                    <small
                        t-if="invoice_data['installment_state'] == 'next'"
                        class="o_portal_invoice_due_date"
                        t-att-datetime="invoice_data['next_due_date']"
                    />
                    <small t-if="invoice_data['installment_state'] == 'overdue'" class="text-danger">
                        overdue
                    </small>
                </t>
            </td>
            <td class="text-center">
                <a t-if="invoice._has_to_be_paid()"
                    t-att-href="invoice.get_portal_url(anchor='portal_pay', query_string='&amp;payment=True')"
                    title="Pay Now"
                    aria-label="Pay now"
                    class="btn btn-sm btn-primary"
                    role="button"
                >
                    <i class="fa fa-arrow-circle-right"/><span class='d-none d-md-inline'> Pay Now</span>
                </a>
            </td>
        </xpath>
        <xpath expr="//t[@name='invoice_status_posted']/span[1]" position="before">
            <t t-set="tx_sudo" t-value="invoice.get_portal_last_transaction()"/>
        </xpath>
        <xpath expr="//span[@name='invoice_status_waiting_for_payment']" position="before">
            <span t-elif="invoice.payment_state in ('not_paid', 'partial') and tx_sudo.state == 'authorized'"
                    class="badge rounded-pill text-bg-primary">
                <i class="fa fa-fw fa-check"/>
                <span class="d-none d-md-inline"> Authorized</span>
            </span>
            <span t-elif="invoice.payment_state in ('not_paid', 'partial') and tx_sudo.state == 'pending' and tx_sudo.provider_code not in ('none', 'custom')"
                    class="badge rounded-pill text-bg-warning">
                <span class="d-none d-md-inline"> Pending</span>
            </span>
            <span t-elif="invoice.payment_state in ('paid', 'in_payment') and tx_sudo.state == 'done'" class="badge rounded-pill text-bg-success">
                <i class="fa fa-fw fa-check"></i>
                <span class="d-none d-md-inline"> Paid</span>
            </span>
        </xpath>
    </template>

    <template id="portal_invoice_payment" name="Invoice Payment">
        <div class="row">
            <div class="modal fade" id="pay_with" role="dialog">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header pb-0">
                            <h3 class="modal-title">Pay</h3>
                            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                        </div>
                        <div class="modal-body">
                            <div t-if="company_mismatch">
                                <t t-call="payment.company_mismatch_warning"/>
                            </div>
                            <div t-elif="installment_state in ('next', 'overdue') and amount_due != next_amount_to_pay" id="o_payment_installments_modal">
                                <div class="btn-group w-100" id="o_payment_tabs" role="tablist">
                                    <button class="btn btn-outline-primary o_btn_payment_tab active"
                                        id="o_payment_installments_tab"
                                        data-bs-toggle="pill"
                                        data-bs-target="#o_payment_installments"
                                        type="button"
                                        role="tab"
                                        aria-controls="o_payment_installments"
                                        aria-selected="true"
                                    >
                                        <strong>Installment</strong>
                                        <br/>
                                        <t
                                            t-out="next_amount_to_pay"
                                            t-options="{'widget': 'monetary', 'display_currency': currency}"
                                        />
                                    </button>
                                    <button class="btn btn-outline-primary o_btn_payment_tab"
                                        id="o_payment_full_tab"
                                        data-bs-toggle="pill"
                                        data-bs-target="#o_payment_full"
                                        type="button"
                                        role="tab"
                                        aria-controls="o_payment_full"
                                        aria-selected="false"
                                    >
                                        <strong>Full Amount</strong><br/>
                                        <t t-out="amount_due" t-options="{'widget': 'monetary', 'display_currency': currency}"/>
                                    </button>
                                </div>
                                <div class="tab-content" id="o_payment_tabcontent">
                                    <div t-if="payment_state == 'in_payment'" class="alert alert-danger mt-2">
                                        A payment has already been made on this invoice, please make sure to not pay twice.
                                    </div>
                                    <div
                                        class="tab-pane my-3 show active"
                                        id="o_payment_installments"
                                        role="tabpanel"
                                        aria-labelledby="o_payment_installments_tab"
                                        tabindex="0"
                                    >
                                        <div class="text-bg-light row row-cols-1 row-cols-md-2 mx-0 py-2 rounded">
                                            <t t-call="payment.summary_item">
                                                <t t-set="name" t-value="'amount_installment'"/>
                                                <t t-set="label">Amount</t>
                                                <t t-set="value" t-value="next_amount_to_pay"/>
                                                <t t-set="options"
                                                    t-value="{'widget': 'monetary', 'display_currency': currency}"
                                                />
                                            </t>
                                            <t t-call="payment.summary_item">
                                                <t t-set="name" t-value="'reference_installment'"/>
                                                <t t-set="label">Reference</t>
                                                <t t-set="value" t-value="next_payment_reference"/>
                                                <t t-set="include_separator" t-value="True"/>
                                            </t>
                                        </div>
                                    </div>
                                    <div class="tab-pane my-3"
                                        id="o_payment_full"
                                        role="tabpanel"
                                        aria-labelledby="o_payment_full_tab"
                                        tabindex="0"
                                    >
                                        <div t-if="show_epd" class="alert alert-success">
                                            Early Payment Discount of <span t-out="epd_discount_amount_currency" t-options="{'widget': 'monetary', 'display_currency': currency}"/> has been applied.
                                        </div>
                                        <div class="text-bg-light row row-cols-1 row-cols-md-2 mx-0 py-2 rounded">
                                            <t t-call="payment.summary_item">
                                                <t t-set="name" t-value="'amount_full'"/>
                                                <t t-set="label">Amount</t>
                                                <t t-set="value" t-value="amount_due"/>
                                                <t t-set="options"
                                                    t-value="{'widget': 'monetary', 'display_currency': currency}"
                                                />
                                            </t>
                                            <t t-call="payment.summary_item">
                                                <t t-set="name" t-value="'reference_full'"/>
                                                <t t-set="label">Reference</t>
                                                <t t-set="value" t-value="invoice_name"/>
                                                <t t-set="include_separator" t-value="True"/>
                                            </t>
                                        </div>
                                    </div>
                                    <!-- set default amount as the max. amount due, the 'amount'
                                         will be updated/set js-side depending the active tab -->
                                    <t t-call="account_payment.form">
                                        <t t-set="amount" t-value="amount_due"/>
                                    </t>
                                </div>
                            </div>
                            <t t-else="">
                                <div t-if="show_epd" class="alert alert-success">
                                    Early Payment Discount of <span t-out="epd_discount_amount_currency" t-options="{'widget': 'monetary', 'display_currency': currency}"/> has been applied.
                                </div>
                                <div t-if="payment_state == 'in_payment'" class="alert alert-danger">
                                    A payment has already been made on this invoice, please make sure to not pay twice.
                                </div>
                                <div class="text-bg-light row row-cols-1 row-cols-md-2 mx-0 mb-3 py-2 rounded">
                                    <t t-call="payment.summary_item">
                                        <t t-set="name" t-value="'amount_full'"/>
                                        <t t-set="label">Amount</t>
                                        <t t-set="value" t-value="amount_due"/>
                                        <t t-set="options"
                                            t-value="{'widget': 'monetary', 'display_currency': currency}"
                                        />
                                    </t>
                                    <t t-call="payment.summary_item">
                                        <t t-set="name" t-value="'reference_full'"/>
                                        <t t-set="label">Reference</t>
                                        <t t-set="value" t-value="next_payment_reference"/>
                                        <t t-set="include_separator" t-value="True"/>
                                    </t>
                                </div>
                                <t t-call="account_payment.form">
                                    <t t-set="amount" t-value="amount_due"/>
                                </t>
                            </t>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="portal_invoice_payment_paid" name="Invoice Paid">
        <div class="modal fade" id="pay_with" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h3 class="modal-title">Pay Invoice</h3>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <div class="alert alert-warning text-center" role="alert" t-out="invoice._get_online_payment_error()"/>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="portal_invoice_page_inherit_payment" name="Payment on My Invoices" inherit_id="account.portal_invoice_page">
        <xpath expr="//t[@t-call='portal.portal_record_sidebar']//h2[1]" position="after">
            <t t-set="pending_txs" t-value="invoice.transaction_ids.filtered(lambda tx: tx.state in ('pending', 'authorized') and tx.provider_code not in ('none', 'custom'))"/>
            <div t-if="invoice.currency_id.is_zero(invoice.amount_residual)" class="rounded text-bg-success fw-normal badge">
                <i class="fa fa-fw fa-check-circle"/> Paid
            </div>
            <div t-if="invoice.payment_state == 'in_payment' and not invoice.currency_id.is_zero(invoice.amount_residual)" class="rounded text-bg-info fw-normal badge">
                <i class="fa fa-fw fa-check-circle"/> Processing Payment
            </div>
            <div t-elif="pending_txs" class="rounded text-bg-info fw-normal badge">
                <i class="fa fa-fw fa-check-circle"/> Pending
            </div>
        </xpath>
        <xpath expr="//t[@t-call='portal.portal_record_sidebar']//div[hasclass('o_download_pdf')]" position="before">
            <a t-if="invoice._has_to_be_paid()"
                href="#"
                class="btn btn-primary"
                data-bs-toggle="modal"
                data-bs-target="#pay_with">
                <i class="fa fa-fw fa-arrow-circle-right"/> Pay Now
            </a>
        </xpath>
        <xpath expr="//div[@id='invoice_content']//div[hasclass('o_portal_html_view')]" position="before">
            <t t-set="tx" t-value="invoice.get_portal_last_transaction()"/>
            <div t-if="invoice.get_portal_last_transaction() and invoice.amount_total and not success and not error and (invoice.payment_state != 'not_paid' or tx.state in ('pending', 'authorized'))"
                 class="o_account_payment_tx_status"
                 t-att-data-invoice-id="invoice.id">
                <t t-call="payment.state_header"/>
            </div>
            <div t-if="invoice._has_to_be_paid()" id="portal_pay" t-att-data-payment="payment">
                <t t-call="account_payment.portal_invoice_payment"/>
            </div>
            <div t-else="" id="portal_pay" t-att-data-payment="payment">
                <t t-call="account_payment.portal_invoice_payment_paid"/>
            </div>
        </xpath>
    </template>

    <template id="portal_invoice_error" name="Invoice error display: payment errors"
            inherit_id="account.portal_invoice_error">
        <xpath expr="//t[@name='generic']" position="after">
            <t t-if="error == 'pay_invoice_invalid_doc'">
                There was an error processing your payment: invalid invoice.
            </t>
            <t t-elif="error == 'pay_invoice_invalid_token'">
                There was en error processing your payment: invalid credit card ID.
            </t>
            <t t-elif="error == 'pay_invoice_tx_fail'">
                There was an error processing your payment: transaction failed.<br />
                <t t-set="tx_sudo" t-value="invoice.get_portal_last_transaction()"/>
                <t t-if="tx_sudo and tx_sudo.state_message">
                    <t t-out="tx_sudo.state_message"/>
                </t>
            </t>
            <t t-elif="error == 'pay_invoice_tx_token'">
                There was an error processing your payment: issue with credit card ID validation.
            </t>
        </xpath>
    </template>

    <template id="portal_invoice_success" name="Invoice success display: payment success"
            inherit_id="account.portal_invoice_success">
        <xpath expr="//a[hasclass('close')]" position="after">
            <t t-if="success == 'pay_invoice'">
                <t t-set="tx_sudo" t-value="invoice.get_portal_last_transaction()"/>
                <span t-if="tx_sudo.provider_id.sudo().done_msg" t-out="tx_sudo.provider_id.sudo().done_msg"/>
                <div t-if="tx_sudo.provider_id.sudo().pending_msg and tx_sudo.provider_code == 'custom' and invoice.ref">
                    <b>Communication: </b><span t-out='invoice.ref'/>
                </div>
            </t>
            <t t-if="success == 'pay_invoice' and invoice.payment_state in ('paid', 'in_payment')">
                Done, your online payment has been successfully processed. Thank you for your order.
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\payment_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="account_payment.form" inherit_id="payment.form" name="Invoice Payment Form">
        <xpath expr="//form[@id='o_payment_form']" position="attributes">
            <attribute name="t-att-data-invoice-next-amount-to-pay">next_amount_to_pay</attribute>
            <attribute name="t-att-data-invoice-amount-due">amount_due</attribute>
            <attribute name="t-att-data-payment-reference">payment_reference</attribute>
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
                    required="state != 'disabled' and code not in ['none', 'custom']"/>
                <field name="support_refund" groups="base.group_no_one"/>
            </group>
        </field>
    </record>

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
                        invisible="invoices_count == 0">
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
                wizard.suitable_payment_token_ids = self.env['payment.token'].sudo().search([
                    *self.env['payment.token']._check_company_domain(wizard.company_id),
                    ('provider_id.capture_manually', '=', False),
                    ('partner_id', '=', wizard.partner_id.id),
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
            if wizard.can_edit_wizard \
                    and wizard.payment_method_line_id.code in codes \
                    and wizard.journal_id \
                    and wizard.partner_id:

                wizard.payment_token_id = self.env['payment.token'].sudo().search([
                    *self.env['payment.token']._check_company_domain(wizard.company_id),
                    ('partner_id', '=', wizard.partner_id.id),
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
                       invisible="not use_electronic_payment_method or not can_edit_wizard or (can_group_payments and not group_payment)"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: wizards\payment_link_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.tools import format_date, formatLang, str2bool

from odoo.addons.payment import utils as payment_utils


class PaymentLinkWizard(models.TransientModel):
    _inherit = 'payment.link.wizard'

    invoice_amount_due = fields.Monetary(
        string="Amount Due",
        compute='_compute_invoice_amount_due',
        currency_field='currency_id'
    )
    open_installments = fields.Json(export_string_translation=False)
    open_installments_preview = fields.Html(
        export_string_translation=False, compute='_compute_open_installments_preview'
    )
    display_open_installments = fields.Boolean(compute='_compute_display_open_installments')
    has_eligible_epd = fields.Boolean()
    discount_date = fields.Date()
    epd_info = fields.Char(
        string="Early Payment Discount Information",
        compute='_compute_epd_info',
    )

    def _compute_warning_message(self):
        super()._compute_warning_message()
        for wizard in self:
            if not wizard.warning_message and not str2bool(self.env['ir.config_parameter'].sudo().get_param('account_payment.enable_portal_payment')):
                wizard.warning_message = _("Online payment option is not enabled in Configuration.")

    @api.depends('amount_max')
    def _compute_invoice_amount_due(self):
        for wizard in self:
            wizard.invoice_amount_due = wizard.amount_max

    @api.depends('open_installments')
    def _compute_open_installments_preview(self):
        for wizard in self:
            preview = ""
            if wizard.display_open_installments:
                for installment in wizard.open_installments or []:
                    preview += "<div>"
                    preview += _(
                        '#%(number)s - Installment of <strong>%(amount)s</strong> due on <strong class="text-primary">%(date)s</strong>',
                        number=installment['number'],
                        amount=formatLang(
                            self.env,
                            installment['amount'],
                            currency_obj=wizard.currency_id,
                        ),
                        date=installment['date_maturity'],
                    )
                    preview += "</div>"
            wizard.open_installments_preview = preview

    @api.depends('amount')
    def _compute_epd_info(self):
        for wizard in self:
            wizard.epd_info = ''
            if wizard.has_eligible_epd and wizard.amount == wizard.invoice_amount_due:
                msg = _("A discount will be applied if the customer pays before %s included.", format_date(wizard.env, wizard.discount_date))
                wizard.epd_info = msg

    @api.depends('open_installments')
    def _compute_display_open_installments(self):
        # hides the installments section if only one installment
        for wizard in self:
            installments = wizard.open_installments or []
            wizard.display_open_installments = len(installments) > 1

    def _prepare_url(self, base_url, related_document):
        """ Override of `payment` to use the portal page URL. """
        res = super()._prepare_url(base_url, related_document)
        if self.res_model != 'account.move':
            return res

        return f'{base_url}/{related_document.get_portal_url()}'

    def _prepare_query_params(self, related_document):
        """ Override of `payment` to define custom query params for invoice payment. """
        res = super()._prepare_query_params(related_document)
        if self.res_model != 'account.move':
            return res

        return {
            'move_id': related_document.id,
            'amount': self.amount,
            'payment_token': self._prepare_access_token(),
            'payment': True,
        }

    def _prepare_access_token(self):
        """ Override of `payment` to generate the access token only based on the amount. """
        res = super()._prepare_access_token()
        if self.res_model != 'account.move':
            return res

        return payment_utils.generate_access_token(self.res_id, self.amount)

    def _prepare_anchor(self):
        """ Override of `payment` to set the 'portal_pay' anchor. """
        res = super()._prepare_anchor()
        if self.res_model != 'account.move':
            return res

        return '#portal_pay'

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

    <record id="payment_link_wizard__form_inherit_account_payment" model="ir.ui.view">
        <field name="name">payment.link.wizard.form.inherit.account_payment</field>
        <field name="model">payment.link.wizard</field>
        <field name="inherit_id" ref="payment.payment_link_wizard_view_form"/>
        <field name="arch" type="xml">

            <div name="no_partner_email" position="before">
                <field name="epd_info"
                       class="alert alert-info fw-bold w-100 mb-3"
                       role="alert"
                       invisible="not epd_info"/>
            </div>

            <field name="amount" position="after">
                <field name="invoice_amount_due" invisible="res_model != 'account.move'"/>
            </field>

            <field name="currency_id" position="after">
                <field name="open_installments" invisible="1"/>
            </field>

            <group name="payment_info" position="after">
                <group name="next_installments"
                    string="Next Installments"
                    invisible="not display_open_installments"
                    class="mt-n4"
                >
                    <field name="open_installments_preview" nolabel="1"/>
                </group>
            </group>
        </field>
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
    support_refund = fields.Selection(
        string="Refund",
        selection=[('none', "Unsupported"), ('full_only', "Full Only"), ('partial', "Partial")],
        compute='_compute_support_refund',
    )
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

    @api.depends('transaction_id.provider_id', 'transaction_id.payment_method_id')
    def _compute_support_refund(self):
        for wizard in self:
            tx_sudo = wizard.transaction_id.sudo()  # needed for users without access to the provider
            p_support_refund = tx_sudo.provider_id.support_refund
            pm_sudo = tx_sudo.payment_method_id
            pm_support_refund = (pm_sudo.primary_payment_method_id or pm_sudo).support_refund
            if p_support_refund == 'none' or pm_support_refund == 'none':
                wizard.support_refund = 'none'
            elif p_support_refund == 'full_only' or pm_support_refund == 'full_only':
                wizard.support_refund = 'full_only'
            else:  # Both support partial refunds.
                wizard.support_refund = 'partial'

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
                     invisible="not has_pending_refund">
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
                               invisible="refunded_amount &lt;= 0"/>
                        <field name="amount_available_for_refund"/>
                        <field name="amount_to_refund"
                               readonly="support_refund == 'full_only'"/>
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

## File: wizards\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pay_invoices_online = fields.Boolean(config_parameter='account_payment.enable_portal_payment')

```

## File: wizards\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.account</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <field name="module_account_payment" position="replace">
                <field name="pay_invoices_online" string="Invoice Online Payment"/>
            </field>
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
from . import res_config_settings

```

