# Odoo Module: l10n_latam_check

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import wizards

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Third Party and Deferred/Electronic Checks Management',
    'version': "1.0.0",
    'category': 'Accounting/Localizations',
    'summary': 'Checks Management',
    'description': """
Own Checks Management
---------------------

Extends 'Check Printing Base' module to manage own checks with more features:

* allow using own checks that are not printed but filled manually by the user
* allow to use deferred or electronic checks
  * printing is disabled
  * check number is set manually by the user
* add an optional "Check Cash-In Date" for post-dated checks (deferred payments)
* add a menu to track own checks

Third Party Checks Management
-----------------------------

Add new "Third party check Management" feature.

There are 2 main Payment Methods additions:

* New Third Party Checks:

  * Payments of this payment method represent the check you get from a customer when getting paid (from an invoice or a manual payment)

* Existing Third Party check.

  * Payments of this payment method are to track moves of the check, for eg:

    * Use a check to pay a vendor
    * Deposit the check on the bank
    * Get the check back from the bank (rejection)
    * Get the check back from the vendor (a rejection or return)
    * Transfer the check from one third party check journal to the other (one shop to another)

  * Those operations can be done with multiple checks at once
""",
    'author': 'ADHOC SA',
    'license': 'LGPL-3',
    'depends': [
        'account',
        'base_vat',
    ],
    'data': [
        'data/account_payment_method_data.xml',
        'wizards/l10n_latam_payment_mass_transfer_views.xml',
        'security/ir.model.access.csv',
        'security/security.xml',
        'views/account_payment_view.xml',
        'views/l10n_latam_check_view.xml',
        'wizards/account_payment_register_views.xml',
    ],
    'installable': True,
}

```

## File: data\account_payment_method_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="account_payment_method_own_checks" model="account.payment.method">
        <field name="name">Own Checks</field>
        <field name="code">own_checks</field>
        <field name="payment_type">outbound</field>
    </record>

    <!-- third party checks -->
    <record id="account_payment_method_new_third_party_checks" model="account.payment.method">
        <field name="name">New Third Party Checks</field>
        <field name="code">new_third_party_checks</field>
        <field name="payment_type">inbound</field>
    </record>

    <record id="account_payment_method_in_third_party_checks" model="account.payment.method">
        <field name="name">Existing Third Party Checks</field>
        <field name="code">in_third_party_checks</field>
        <field name="payment_type">inbound</field>
    </record>

    <record id="account_payment_method_out_third_party_checks" model="account.payment.method">
        <field name="name">Existing Third Party Checks</field>
        <field name="code">out_third_party_checks</field>
        <field name="payment_type">outbound</field>
    </record>

    <record id="account_payment_method_return_third_party_checks" model="account.payment.method">
        <field name="name">Return Third Party Checks</field>
        <field name="code">return_third_party_checks</field>
        <field name="payment_type">outbound</field>
    </record>

</odoo>

```

## File: models\account_chart_template.py

```python
from odoo import models, Command, api, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @api.model
    def _get_third_party_checks_country_codes(self):
        """ Return the list of country codes for the countries where third party checks journals should be created
        when installing the COA"""
        return ["AR"]

    @template(model='account.journal')
    def _get_latam_check_account_journal(self, template_code):
        if self.env.company.country_id.code in self._get_third_party_checks_country_codes():
            return {
                "third_party_check": {
                    'name': _('Third Party Checks'),
                    'type': 'cash',
                    'outbound_payment_method_line_ids': [
                        Command.create({
                            'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_out_third_party_checks').id,
                            'payment_account_id': 'base_outstanding_payments',
                        }),
                    ],
                    'inbound_payment_method_line_ids': [
                        Command.create({
                            'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_new_third_party_checks').id,
                            'payment_account_id': 'base_outstanding_receipts',
                        }),
                        Command.create({
                            'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_in_third_party_checks').id,
                            'payment_account_id': 'base_outstanding_receipts',
                        }),
                    ],
                },
                "rejected_third_party_check": {
                    'name': _('Rejected Third Party Checks'),
                    'type': 'cash',
                    'outbound_payment_method_line_ids': [
                        Command.create({
                            'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_out_third_party_checks').id,
                            'payment_account_id': 'base_outstanding_payments',
                        }),
                    ],
                    'inbound_payment_method_line_ids': [
                        Command.create({
                            'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_new_third_party_checks').id,
                            'payment_account_id': 'base_outstanding_receipts',
                        }),
                        Command.create({
                            'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_in_third_party_checks').id,
                            'payment_account_id': 'base_outstanding_receipts',
                        }),
                    ],
                },
            }

    @template(model='account.account')
    def _get_latam_check_outstanding_account_account(self, template_code):
        if self.env.company.country_id.code in self._get_third_party_checks_country_codes():
            return {
                'base_outstanding_receipts': {
                    'name': _("Outstanding Receipts"),
                    'code': '1.1.1.02.003',
                    'reconcile': True,
                    'account_type': 'asset_current',
                },
                'base_outstanding_payments': {
                    'name': _("Outstanding Payments"),
                    'code': '1.1.1.02.004',
                    'reconcile': True,
                    'account_type': 'asset_current',
                },
            }

```

## File: models\account_journal.py

```python
from odoo import models, api


class AccountJournal(models.Model):
    _inherit = "account.journal"

    def _default_outbound_payment_methods(self):
        res = super()._default_outbound_payment_methods()
        if self.company_id.country_id.code != "AR":
            return res
        if self._is_payment_method_available('own_checks'):
            res |= self.env.ref('l10n_latam_check.account_payment_method_own_checks')
        if self._is_payment_method_available('return_third_party_checks'):
            res |= self.env.ref('l10n_latam_check.account_payment_method_return_third_party_checks')
        return res

    @api.model
    def _get_reusable_payment_methods(self):
        """ We are able to have multiple times Checks payment method in a journal """
        res = super()._get_reusable_payment_methods()
        res.add("own_checks")
        return res

    def create(self, vals_list):
        journals = super().create(vals_list)
        inbound_payment_accounts = self.env['account.account'].search([
            ('code', '=', '1.1.1.02.003'),
            ('company_ids', 'in', journals.company_id.ids)
        ]).grouped('company_ids')

        outbound_payment_accounts = self.env['account.account'].search([
            ('code', '=', '1.1.1.02.004'),
            ('company_ids', 'in', journals.company_id.ids)
        ]).grouped('company_ids')

        for journal in journals:
            if journal.country_code != 'AR' or journal.type not in ('bank', 'cash'):
                continue

            for payment_method_line in journal.inbound_payment_method_line_ids:
                if payment_method_line.payment_account_id:
                    continue
                payment_method_line.payment_account_id = inbound_payment_accounts.get(journal.company_id)

            for payment_method_line in journal.outbound_payment_method_line_ids:
                if payment_method_line.payment_account_id:
                    continue
                payment_method_line.payment_account_id = outbound_payment_accounts.get(journal.company_id)

        return journals

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMove(models.Model):

    _inherit = 'account.move'

    def button_draft(self):
        super().button_draft()
        for move in self.filtered(lambda x: x.origin_payment_id.payment_method_code == 'own_checks'):
            move.origin_payment_id._l10n_latam_check_unlink_split_move()

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountMoveLine(models.Model):

    _inherit = 'account.move.line'

    l10n_latam_check_ids = fields.One2many('l10n_latam.check', 'outstanding_line_id', string='Checks')

```

## File: models\account_payment.py

```python
from odoo import fields, models, api, Command, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools.misc import format_date


class AccountPayment(models.Model):
    _inherit = 'account.payment'

    l10n_latam_new_check_ids = fields.One2many('l10n_latam.check', 'payment_id', string='Checks')
    l10n_latam_move_check_ids = fields.Many2many(
        comodel_name='l10n_latam.check',
        relation='l10n_latam_check_account_payment_rel',
        column1="payment_id",
        column2="check_id",
        required=True,
        copy=False,
        string="Checks Operations"
    )
    # Warning message in case of unlogical third party check operations
    l10n_latam_check_warning_msg = fields.Text(compute='_compute_l10n_latam_check_warning_msg')
    amount = fields.Monetary(compute="_compute_amount", readonly=False, store=True)

    @api.constrains('state', 'move_id')
    def _check_move_id(self):
        for payment in self:
            if (
                not payment.move_id and
                payment.payment_method_code in ('own_checks', 'new_third_party_checks', 'in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks') and
                not payment.outstanding_account_id
            ):
                raise ValidationError(_("A payment with any Third Party Check or Own Check payment methods needs an outstanding account"))

    @api.depends('l10n_latam_move_check_ids.amount', 'l10n_latam_new_check_ids.amount', 'payment_method_code')
    def _compute_amount(self):
        for rec in self:
            checks = rec.l10n_latam_new_check_ids if rec._is_latam_check_payment(check_subtype='new_check') else rec.l10n_latam_move_check_ids
            if checks:
                rec.amount = sum(checks.mapped('amount'))

    def _is_latam_check_payment(self, check_subtype=False):
        if check_subtype == 'move_check':
            codes = ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks']
        elif check_subtype == 'new_check':
            codes = ['new_third_party_checks', 'own_checks']
        else:
            codes = ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks', 'new_third_party_checks', 'own_checks']
        return self.payment_method_code in codes

    def action_post(self):
        # unlink checks if payment method code is not for checks. We do it on post and not when changing payment
        # method so that the user don't loose checks data in case of changing payment method and coming back again
        # also, changing partner recompute payment method so all checks would be cleaned
        for payment in self.filtered(lambda x: x.l10n_latam_new_check_ids and not x._is_latam_check_payment(check_subtype='new_check')):
            payment.l10n_latam_new_check_ids.unlink()
        if not self.env.context.get('l10n_ar_skip_remove_check'):
            for payment in self.filtered(lambda x: x.l10n_latam_move_check_ids and not x._is_latam_check_payment(check_subtype='move_check')):
                payment.l10n_latam_move_check_ids = False
        msgs = self._get_blocking_l10n_latam_warning_msg()
        if msgs:
            raise ValidationError('* %s' % '\n* '.join(msgs))
        super().action_post()
        self._l10n_latam_check_split_move()

    def _get_latam_checks(self):
        self.ensure_one()
        if self._is_latam_check_payment(check_subtype='new_check'):
            return self.l10n_latam_new_check_ids
        elif self._is_latam_check_payment(check_subtype='move_check'):
            return self.l10n_latam_move_check_ids
        else:
            return self.env['l10n_latam.check']

    def _get_blocking_l10n_latam_warning_msg(self):
        msgs = []
        for rec in self.filtered(lambda x: x.state == 'draft' and x._is_latam_check_payment()):
            if any(rec.currency_id != check.currency_id for check in rec._get_latam_checks()):
                msgs.append(_('The currency of the payment and the currency of the check must be the same.'))
            if not rec.currency_id.is_zero(sum(rec._get_latam_checks().mapped('amount')) - rec.amount):
                msgs.append(
                    _('The amount of the payment  does not match the amount of the selected check. '
                      'Please try to deselect and select the check again.')
                )
            # checks being moved
            if rec._is_latam_check_payment(check_subtype='move_check'):
                if any(check.payment_id.state == 'draft' for check in rec.l10n_latam_move_check_ids):
                    msgs.append(
                        _('Selected checks "%s" are not posted', rec.l10n_latam_move_check_ids.filtered(lambda x: x.payment_id.state == 'draft').mapped('display_name'))
                    )
                elif rec.payment_type == 'outbound' and any(check.current_journal_id != rec.journal_id for check in rec.l10n_latam_move_check_ids):
                    # check outbound payment and transfer or inbound transfer
                    msgs.append(_(
                        'Some checks are not anymore in journal, it seems it has been moved by another payment.')
                    )
                elif rec.payment_type == 'inbound' and not rec._is_latam_check_transfer() and any(rec.l10n_latam_move_check_ids.mapped('current_journal_id')):
                    msgs.append(
                        _("Some checks are already in hand and can't be received again. Checks: %s",
                          ', '.join(rec.l10n_latam_move_check_ids.mapped('display_name')))
                    )

                for check in rec.l10n_latam_move_check_ids:
                    date = rec.date or fields.Datetime.now()

                    last_operation = check._get_last_operation()
                    if last_operation and last_operation[0].date > date:
                        msgs.append(
                            _(
                              "It seems you're trying to move a check with a date (%(date)s) prior to last "
                              "operation done with the check (%(last_operation)s). This may be wrong, please "
                              "double check it. By continue, the last operation on "
                              "the check will remain being %(last_operation)s",
                              date=format_date(self.env, date), last_operation=last_operation.display_name
                            )
                        )
        return msgs

    def _get_reconciled_checks_error(self):
        checks_reconciled = self.l10n_latam_new_check_ids.filtered(lambda x: x.issue_state in ['debited', 'voided'])
        if checks_reconciled:
            raise UserError(
                _("You can't cancel or re-open a payment with checks if some check has been debited or been voided. "
                  "Checks:\n%s", ('\n'.join(['* %s (%s)' % (x.name, x.issue_state) for x in checks_reconciled])))
            )

    def action_cancel(self):
        self._get_reconciled_checks_error()
        super().action_cancel()

    def action_draft(self):
        self._get_reconciled_checks_error()
        super().action_draft()

    def _l10n_latam_check_split_move(self):
        for payment in self.filtered(lambda x: x.payment_method_code == 'own_checks' and x.payment_type == 'outbound'):
            if len(payment.l10n_latam_new_check_ids) == 1:
                liquidity_line = payment._seek_for_lines()[0]
                payment.l10n_latam_new_check_ids.outstanding_line_id = liquidity_line.id
                continue

            vals = {
                'journal_id': payment.journal_id.id,
                'move_type': 'entry',
                'line_ids': [],
            }
            payment_liquidity_line = payment._seek_for_lines()[0]

            # One line per check
            checks_total = sum(payment.l10n_latam_new_check_ids.mapped('amount'))
            liquidity_balance_total = 0.0
            liquidity_balance = 0.0
            for check in payment.l10n_latam_new_check_ids:
                liquidity_amount_currency = -check.amount

                if check == payment.l10n_latam_new_check_ids[-1]:
                    liquidity_balance = payment.currency_id.round(payment_liquidity_line.balance - liquidity_balance)
                else:
                    liquidity_balance = payment.currency_id.round(payment_liquidity_line.balance * check.amount / checks_total)
                    liquidity_balance_total += liquidity_balance

                vals['line_ids'].append(
                    Command.create({
                        'name': _(
                            'Check %(check_number)s - %(suffix)s',
                            check_number=check.name,
                            suffix=''.join([item[1] for item in payment._get_aml_default_display_name_list()])),
                        'date_maturity': check.payment_date,
                        'amount_currency': liquidity_amount_currency,
                        'currency_id': check.currency_id.id,
                        'debit': max(0.0, liquidity_balance),
                        'credit': -min(liquidity_balance, 0.0),
                        'partner_id': payment_liquidity_line.partner_id.id,
                        'account_id': payment_liquidity_line.account_id.id,
                        'l10n_latam_check_ids': [Command.link(check.id)]
                    }),
                )

            # Cancel payment line
            vals['line_ids'].append(
                Command.create({
                    'name': payment_liquidity_line.name,
                    'date_maturity': payment_liquidity_line.date_maturity,
                    'amount_currency': -payment_liquidity_line.amount_currency,
                    'currency_id': payment_liquidity_line.currency_id.id,
                    'debit': -payment_liquidity_line.debit,
                    'credit': -payment_liquidity_line.credit,
                    'partner_id': payment_liquidity_line.partner_id.id,
                    'account_id': payment_liquidity_line.account_id.id,
                }),
            )
            move_id = self.env['account.move'].create(vals)
            move_id.action_post()
            split_move_counterpart_line = move_id.line_ids.filtered(lambda x: x.amount_currency == -payment_liquidity_line.amount_currency)
            (split_move_counterpart_line + payment_liquidity_line).reconcile()

    def _l10n_latam_check_unlink_split_move(self):
        self.ensure_one()
        for check in self.l10n_latam_new_check_ids:
            if self.move_id == check.outstanding_line_id.move_id:
                check.outstanding_line_id = False
                continue
            check.outstanding_line_id.move_id.button_draft()
            check.outstanding_line_id.move_id.unlink()

    @api.depends(
        'payment_method_line_id', 'state', 'date', 'amount', 'currency_id', 'company_id',
        'l10n_latam_move_check_ids.issuer_vat', 'l10n_latam_move_check_ids.bank_id', 'l10n_latam_move_check_ids.payment_id.date',
        'l10n_latam_new_check_ids.amount', 'l10n_latam_new_check_ids.name',
    )
    def _compute_l10n_latam_check_warning_msg(self):
        """
        Compute warning message for latam checks checks
        We use l10n_latam_check_number as de dependency because on the interface this is the field the user is using.
        Another approach could be to add an onchange on _inverse_l10n_latam_check_number method
        """
        self.l10n_latam_check_warning_msg = False
        for rec in self.filtered(lambda x: x._is_latam_check_payment()):
            msgs = rec._get_blocking_l10n_latam_warning_msg()
            # new third party check uniqueness warning (on own checks it's done by a sql constraint)
            if rec.payment_method_code == 'new_third_party_checks':
                same_checks = self.env['l10n_latam.check']
                for check in rec.l10n_latam_new_check_ids.filtered(
                        lambda x: x.name and x.payment_method_line_id.code == 'new_third_party_checks' and
                        x.bank_id and x.issuer_vat):
                    same_checks += same_checks.search([
                        ('company_id', '=', rec.company_id.id),
                        ('bank_id', '=', check.bank_id.id),
                        ('issuer_vat', '=', check.issuer_vat),
                        ('name', '=', check.name),
                        ('payment_id.state', '!=', 'draft'),
                        ('id', '!=', check._origin.id)], limit=1)
                if same_checks:
                    msgs.append(
                        _("Other checks were found with same number, issuer and bank. Please double check you are not "
                          "encoding the same check more than once. List of other payments/checks: %s",
                          ", ".join(same_checks.mapped('display_name')))
                    )
            rec.l10n_latam_check_warning_msg = msgs and '* %s' % '\n* '.join(msgs) or False

    @api.model
    def _get_trigger_fields_to_synchronize(self):
        res = super()._get_trigger_fields_to_synchronize()
        return res + ('l10n_latam_new_check_ids',)

    def _prepare_move_line_default_vals(self, write_off_line_vals=None, force_balance=None):
        """ Add check name and operation on liquidity line """
        res = super()._prepare_move_line_default_vals(write_off_line_vals=write_off_line_vals, force_balance=force_balance)

        # if only one check we don't create the split line, we add same data on liquidity line
        if self.payment_method_code == 'own_checks' and self.payment_type == 'outbound' and len(self.l10n_latam_new_check_ids) == 1:
            res[0].update({
                'name': _(
                    'Check %(check_number)s - %(suffix)s',
                    check_number=self.l10n_latam_new_check_ids.name,
                    suffix=''.join([item[1] for item in self._get_aml_default_display_name_list()])),
                'date_maturity': self.l10n_latam_new_check_ids.payment_date,
            })
        # we dont check the payment method code because when deposited on bank/cash journals pay method is manual but we still change the label
        # we dont want this names on the own checks because it doesn't add value, already each split/check line will have it name
        elif (self.l10n_latam_new_check_ids or self.l10n_latam_move_check_ids) and self.payment_method_code != 'own_checks':
            check_name = [check_name for check_name in (self.l10n_latam_new_check_ids | self.l10n_latam_move_check_ids).mapped('name') if check_name]
            document_name = (
                _('Checks %s received') if self.payment_type == 'inbound' else _('Checks %s delivered')) % (
                ', '.join(check_name)
            )
            res[0].update({
                'name': document_name + ' - ' + ''.join([item[1] for item in self._get_aml_default_display_name_list()]),
            })
        return res

    @api.depends('l10n_latam_move_check_ids')
    def _compute_destination_account_id(self):
        # EXTENDS 'account'
        super()._compute_destination_account_id()
        for payment in self:
            if payment.l10n_latam_move_check_ids and (not payment.partner_id or payment.partner_id == payment.company_id.partner_id):
                payment.destination_account_id = payment.company_id.transfer_account_id.id

    def _is_latam_check_transfer(self):
        self.ensure_one()
        return not self.partner_id and self.destination_account_id == self.company_id.transfer_account_id

```

## File: models\account_payment_method.py

```python
from odoo import models, api


class AccountPaymentMethod(models.Model):
    _inherit = 'account.payment.method'

    @api.model
    def _get_payment_method_information(self):
        res = super()._get_payment_method_information()
        res['new_third_party_checks'] = {'mode': 'multi', 'type': ('cash',)}
        res['in_third_party_checks'] = {'mode': 'multi', 'type': ('cash',)}
        res['out_third_party_checks'] = {'mode': 'multi', 'type': ('cash',)}
        res['return_third_party_checks'] = {'mode': 'multi', 'type': ('bank',)}
        res['own_checks'] = {'mode': 'multi', 'type': ('bank',)}
        return res

```

## File: models\l10n_latam_check.py

```python
# pylint: disable=protected-access
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import stdnum

from odoo import models, fields, api, Command, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import index_exists


_logger = logging.getLogger(__name__)


class l10nLatamAccountPaymentCheck(models.Model):
    _name = 'l10n_latam.check'
    _description = 'Account payment check'
    _check_company_auto = True
    _inherit = ['mail.thread', 'mail.activity.mixin']

    payment_id = fields.Many2one(
        'account.payment',
        required=True,
        ondelete='cascade',
    )
    operation_ids = fields.Many2many(
        comodel_name='account.payment',
        relation='l10n_latam_check_account_payment_rel',
        column1="check_id",
        column2="payment_id",
        readonly=True,
        check_company=True,
    )
    current_journal_id = fields.Many2one(
        comodel_name='account.journal',
        compute='_compute_current_journal', store=True,
    )
    name = fields.Char(string='Number')
    bank_id = fields.Many2one(
        comodel_name='res.bank',
        compute='_compute_bank_id', store=True, readonly=False,
    )
    issuer_vat = fields.Char(
        compute='_compute_issuer_vat', store=True, readonly=False,
    )
    payment_date = fields.Date(readonly=False, required=True)
    amount = fields.Monetary()
    outstanding_line_id = fields.Many2one('account.move.line', readonly=True, check_company=True)
    issue_state = fields.Selection(
        selection=[('handed', 'Handed'), ('debited', 'Debited'), ('voided', 'Voided')],
        compute='_compute_issue_state',
        store=True
    )
    # fields from payment
    payment_method_code = fields.Char(related='payment_id.payment_method_code')
    partner_id = fields.Many2one(related='payment_id.partner_id')
    original_journal_id = fields.Many2one(related='payment_id.journal_id')
    company_id = fields.Many2one(related='payment_id.company_id', store=True)
    currency_id = fields.Many2one(related='payment_id.currency_id')
    payment_method_line_id = fields.Many2one(
        related='payment_id.payment_method_line_id',
        store=True,
    )

    def _auto_init(self):
        super()._auto_init()
        if not index_exists(self.env.cr, 'l10n_latam_check_unique'):
            # issue_state is used to know that is an own check and also that is posted
            self.env.cr.execute("""
                CREATE UNIQUE INDEX l10n_latam_check_unique
                    ON l10n_latam_check(name, payment_method_line_id)
                WHERE outstanding_line_id IS NOT NULL
            """)

    @api.onchange('name')
    def _onchange_name(self):
        if self.name:
            self.name = self.name.zfill(8)

    def _prepare_void_move_vals(self):
        return {
            'ref': 'Void check',
            'journal_id': self.outstanding_line_id.move_id.journal_id.id,
            'line_ids': [
                Command.create({
                    'name': "Void check %s" % self.outstanding_line_id.name,
                    'date_maturity': self.outstanding_line_id.date_maturity,
                    'amount_currency': self.outstanding_line_id.amount_currency,
                    'currency_id': self.outstanding_line_id.currency_id.id,
                    'debit': self.outstanding_line_id.debit,
                    'credit': self.outstanding_line_id.credit,
                    'partner_id': self.outstanding_line_id.partner_id.id,
                    'account_id': self.payment_id.destination_account_id.id,
                }),
                Command.create({
                    'name': "Void check %s" % self.outstanding_line_id.name,
                    'date_maturity': self.outstanding_line_id.date_maturity,
                    'amount_currency': -self.outstanding_line_id.amount_currency,
                    'currency_id': self.outstanding_line_id.currency_id.id,
                    'debit': -self.outstanding_line_id.debit,
                    'credit': -self.outstanding_line_id.credit,
                    'partner_id': self.outstanding_line_id.partner_id.id,
                    'account_id': self.outstanding_line_id.account_id.id,
                }),
            ],
        }

    @api.depends('outstanding_line_id.amount_residual')
    def _compute_issue_state(self):
        for rec in self:
            if not rec.outstanding_line_id:
                rec.issue_state = False
            elif rec.amount and not rec.outstanding_line_id.amount_residual:
                if any(
                    line.account_id.account_type in ['liability_payable', 'asset_receivable']
                    for line in rec.outstanding_line_id.matched_debit_ids.debit_move_id.move_id.line_ids
                ):
                    rec.issue_state = 'voided'
                else:
                    rec.issue_state = 'debited'
            else:
                rec.issue_state = 'handed'

    def action_void(self):
        for rec in self.filtered('outstanding_line_id'):
            void_move = rec.env['account.move'].create(rec._prepare_void_move_vals())
            void_move.action_post()
            (void_move.line_ids[1] + rec.outstanding_line_id).reconcile()

    def _get_last_operation(self):
        self.ensure_one()
        return (self.payment_id + self.operation_ids).filtered(
                lambda x: x.state != 'draft').sorted(key=lambda payment: (payment.date, payment._origin.id))[-1:]

    @api.depends('payment_id.state', 'operation_ids.state')
    def _compute_current_journal(self):
        for rec in self:
            last_operation = rec._get_last_operation()
            if not last_operation:
                rec.current_journal_id = False
                continue
            if last_operation.payment_type == 'inbound':
                rec.current_journal_id = last_operation.journal_id
            else:
                rec.current_journal_id = False

    def button_open_payment(self):
        self.ensure_one()
        return self.payment_id._get_records_action()

    def button_open_check_operations(self):
        ''' Redirect the user to the invoice(s) paid by this payment.
        :return:    An action on account.move.
        '''
        self.ensure_one()
        operations = ((self.operation_ids + self.payment_id).filtered(lambda x: x.state != 'draft'))
        action = {
            'name': _("Check Operations"),
            'type': 'ir.actions.act_window',
            'res_model': 'account.payment',
            'views': [
                (self.env.ref('l10n_latam_check.view_account_third_party_check_operations_tree').id, 'list'),
                (False, 'form')
            ],
            'context': {'create': False},
            'domain': [('id', 'in', operations.ids)],
        }
        return action

    def action_show_reconciled_move(self):
        self.ensure_one()
        move = self._get_reconciled_move()
        return move._get_records_action()

    def action_show_journal_entry(self):
        self.ensure_one()
        return self.outstanding_line_id.move_id._get_records_action()

    def _get_reconciled_move(self):
        reconciled_line = self.outstanding_line_id.full_reconcile_id.reconciled_line_ids - self.outstanding_line_id
        return (reconciled_line.move_id.line_ids - reconciled_line).mapped('move_id')

    @api.constrains('amount')
    def _constrains_min_amount(self):
        min_amount_error = self.filtered(lambda x: x.amount <= 0)
        if min_amount_error:
            raise ValidationError(_('The amount of the check must be greater than 0'))

    @api.depends('payment_method_line_id.code', 'payment_id.partner_id')
    def _compute_bank_id(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.bank_id = rec.partner_id.bank_ids[:1].bank_id
        (self - new_third_party_checks).bank_id = False

    @api.depends('payment_method_line_id.code', 'payment_id.partner_id')
    def _compute_issuer_vat(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.issuer_vat = rec.payment_id.partner_id.vat
        (self - new_third_party_checks).issuer_vat = False

    @api.onchange('issuer_vat')
    def _clean_issuer_vat(self):
        for rec in self.filtered(lambda x: x.issuer_vat and x.company_id.country_id.code):
            stdnum_vat = stdnum.util.get_cc_module(rec.company_id.country_id.code, 'vat')
            if hasattr(stdnum_vat, 'compact'):
                rec.issuer_vat = stdnum_vat.compact(rec.issuer_vat)

    @api.constrains('issuer_vat')
    def _check_issuer_vat(self):
        for rec in self.filtered(lambda x: x.issuer_vat and x.company_id.country_id):
            if not self.env['res.partner']._run_vat_test(rec.issuer_vat, rec.company_id.country_id):
                error_message = self.env['res.partner']._build_vat_error_message(
                    rec.company_id.country_id.code.lower(), rec.issuer_vat, 'Check Issuer VAT'
                )
                raise ValidationError(error_message)

    @api.ondelete(at_uninstall=False)
    def _unlink_if_payment_is_draft(self):
        if any(check.payment_id.state != 'draft' for check in self):
            raise UserError("Can't delete a check if payment is In Process!")

```

## File: models\__init__.py

```python
from . import account_payment
from . import account_payment_method
from . import account_chart_template
from . import l10n_latam_check
from . import account_move
from . import account_move_line
from . import account_journal

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_latam_payment_mass_transfer,access_l10n_latam_payment_mass_transfer,model_l10n_latam_payment_mass_transfer,account.group_account_invoice,1,1,1,0
access_l10n_latam_account_payment_register_check,access.account.payment.register.check,model_l10n_latam_payment_register_check,account.group_account_invoice,1,1,1,1
access_l10n_latam_check_readonly,l10n_latam.check,model_l10n_latam_check,account.group_account_readonly,1,0,0,0
access_l10n_latam_check,l10n_latam.check,model_l10n_latam_check,account.group_account_invoice,1,1,1,1

```

## File: security\security.xml

```xml
<odoo noupdate="1">

    <record model="ir.rule" id="l10n_latam_check_comp_rule">
        <field name="name">Latam Check company rule</field>
        <field name="model_id" ref="model_l10n_latam_check"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: views\account_payment_view.xml

```xml
<odoo>

    <record id="view_account_payment_form_inherited" model="ir.ui.view">
        <field name="name">account.payment.form.inherited</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="account.view_account_payment_form"/>
        <field name="arch" type="xml">
            <group>
                <notebook>
                    <page name="latam_checks_page" string="Checks" invisible="payment_method_code not in ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks', 'new_third_party_checks', 'own_checks']">
                        <group name="latam_checks" colspan="2">
                            <field name="l10n_latam_new_check_ids" invisible="payment_method_code not in ['new_third_party_checks', 'own_checks']" nolabel="1" colspan="2" readonly="state != 'draft'">
                                <list name="new_checks" editable="bottom">
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="name" />
                                    <field name="bank_id" column_invisible="parent.payment_method_code == 'own_checks'"/>
                                    <field name="issuer_vat" column_invisible="parent.payment_method_code == 'own_checks'"/>
                                    <field name="payment_date"/>
                                    <field name="amount" />
                                    <button type="object" name="get_formview_action" icon="fa-pencil-square-o" title="open" help="Open" column_invisible="parent.state == 'draft'"/>
                                </list>
                            </field>
                            <field name="l10n_latam_move_check_ids" invisible="payment_method_code not in ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks']"
                                domain="
                                    [('payment_method_code', '=', 'new_third_party_checks'), ('current_journal_id', '=', journal_id), ('company_id', '=', company_id)]
                                        if payment_type == 'outbound' else
                                    [('payment_method_code', '=', 'new_third_party_checks'), ('current_journal_id', '=', False), ('company_id', '=', company_id)]" options="{'no_create': True}"
                                    nolabel="1" colspan="2" readonly="state != 'draft'">
                                <list name="existing_checks">
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="name" />
                                    <field name="bank_id" optional="hide"/>
                                    <field name="issuer_vat" optional="hide"/>
                                    <field name="payment_date" optional="hide"/>
                                    <field name="amount"/>
                                    <button type="object" name="get_formview_action" icon="fa-pencil-square-o" title="open" help="Open"/>
                                </list>
                            </field>
                        </group>
                    </page>
                </notebook>
            </group>
            <sheet position="before">
                <div class="alert alert-danger mb-0" role="alert" invisible="not l10n_latam_check_warning_msg">
                    <field name="l10n_latam_check_warning_msg" nolabel="1"/>
                </div>
            </sheet>
        </field>
    </record>

</odoo>

```

## File: views\l10n_latam_check_view.xml

```xml
<odoo>

    <!-- Own checks search view -->
    <record model="ir.ui.view" id="view_account_payment_search">
        <field name="name">account.check.search</field>
        <field name="model">l10n_latam.check</field>
        <field name="inherit_id" eval="False"/>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="partner_id"/>
                <field name="original_journal_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <separator/>
                <filter string="Payment Date" name="payment_date" date="payment_date"/>
                <separator/>
                <filter string="Handed" name="checks_on_hand" domain="[('issue_state', '=', 'handed')]"/>
                <filter string="Voided" name="checks_voided" domain="[('issue_state', '=', 'voided')]"/>
                <filter string="Debited" name="checks_debited" domain="[('issue_state', '=', 'debited')]"/>
                <separator/>
                <filter string="Partner" name="groupby_partner" domain="[]" context="{'group_by': 'partner_id'}"/>
                <filter string="Payment Date" name="groupby_date" domain="[]" context="{'group_by': 'payment_date'}"/>
                <filter string="State" name="groupby_issue_state" domain="[]" context="{'group_by': 'issue_state'}"/>
                <filter string="Company" name="groupby_company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue" domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]" help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today" domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all" domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
            </search>
        </field>
    </record>

    <!-- Third party checks search view -->
    <record model="ir.ui.view" id="view_account_payment_third_party_checks_search">
        <field name="name">account.check.search</field>
        <field name="model">l10n_latam.check</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="view_account_payment_search"/>
        <field name="arch" type="xml">
            <filter name="checks_on_hand" position="replace"/>
            <filter name="checks_voided" position="replace"/>
            <filter name="checks_debited" position="replace">
                <filter string="On hand" name="checks_on_hand"
                    domain="[('current_journal_id.inbound_payment_method_line_ids.payment_method_id.code', '=', 'in_third_party_checks')]"/>
            </filter>
            <field name="original_journal_id" position="before">
                <field name="issuer_vat"/>
                <field name="bank_id"/>
                <field name="current_journal_id"/>
            </field>
            <filter name="groupby_issue_state" position="replace">
                <filter name="groupby_current_journal"
                    string="Current Journal"
                    context="{'group_by': 'current_journal_id'}"/>
            </filter>
        </field>
    </record>

    <record model="ir.ui.view" id="view_account_third_party_check_operations_tree">
        <field name="name">account.check.operations.list</field>
        <field name="model">account.payment</field>
        <field name="priority" eval="99"/>
        <field name="arch" type="xml">
            <list default_order="date desc, id desc, name desc" create="false" delete="false" duplicate="false" >
                <field name="date" readonly="state in ['cancel', 'posted']"/>
                <field name="name"/>
                <field name="payment_type"/>
                <field name="journal_id"/>
                <field name="partner_id" string="Customer"/>
                <field name="state" column_invisible="True"/>
            </list>
        </field>
    </record>

    <record model="ir.ui.view" id="view_account_check_calendar">
        <field name="name">account.check.calendar</field>
        <field name="model">l10n_latam.check</field>
        <field name="arch" type="xml">
            <calendar
                    mode="month"
                    date_start="payment_date"
                    color="original_journal_id">
                <field name="amount"/>
            </calendar>
        </field>
    </record>

    <record model="ir.ui.view" id="view_account_check_pivot">
        <field name="name">account.check.calendar</field>
        <field name="model">l10n_latam.check</field>
        <field name="arch" type="xml">
            <pivot>
                <field name="payment_date" type="row" interval="month"/>
                <field name="payment_date" type="row" interval="week"/>
                <field name="amount" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="l10n_latam_check_view_form" model="ir.ui.view">
        <field name="name">l10n_latam_check.view.form</field>
        <field name="model">l10n_latam.check</field>
        <field name="arch" type="xml">
            <form create="false" edit="false" delete="false">
                <field name="outstanding_line_id" invisible="True"/>
                <header>
                    <button name="action_void" string="Void Check" invisible="issue_state != 'handed'" type="object" class="oe_highlight" confirm="Marking a check as void will cancel the check and generate a new entry that will re-open the debt."  data-hotkey="v"/>
                    <field name="issue_state" statusbar_visible="issue_state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button  icon="fa-bars" type="object" name="button_open_check_operations" invisible="payment_method_code != 'new_third_party_checks'">
                            <span>Operations</span>
                        </button>
                        <button  icon="fa-bars" type="object" name="button_open_payment" invisible="payment_method_code != 'own_checks'">
                            <span>Payment</span>
                        </button>
                        <button  icon="fa-bars" type="object" invisible="not outstanding_line_id" name="action_show_journal_entry" groups="account.group_account_user,account.group_account_readonly">
                            <span>Journal Entry</span>
                        </button>
                        <button  icon="fa-bars" type="object" invisible="not issue_state or issue_state == 'handed'" name="action_show_reconciled_move" groups="account.group_account_user,account.group_account_readonly">
                            <span>Reconciled move</span>
                        </button>
                    </div>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="payment_date"/>
                            <field name="original_journal_id"/>
                            <field name="current_journal_id" invisible="issue_state"/>
                        </group>
                        <group>
                            <field name="amount"/>
                            <field name="bank_id"  invisible="issue_state"/>
                            <field name="issuer_vat"  invisible="issue_state"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                </sheet>
                <chatter/>
            </form>
        </field>
    </record>

 <!-- Own Check Views and menus -->

    <record model="ir.ui.view" id="view_account_own_check_tree">
        <field name="name">account.check.list</field>
        <field name="model">l10n_latam.check</field>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <list edit="false" create="false" delete="false" duplicate="false" sample="1" decoration-info="issue_state == 'handed'" decoration-muted="issue_state in ('voided','debited')">
                    <header>
                    </header>
                    <field name="payment_date" optional="show"/>
                    <field name="name"/>
                    <field name="original_journal_id"/>
                    <field name="company_id" optional="hide" groups="base.group_multi_company"/>
                    <field name="payment_method_line_id" column_invisible="True"/>
                    <field name="partner_id" string="Customer"/>
                    <field name="amount"  optional="show"/>
                    <field name="currency_id" string="Payment Currency" optional="hide"/>
                    <field name="issue_state" widget="badge" decoration-info="issue_state == 'handed'"  decoration-muted="issue_state in ('voided','debited')"/>
            </list>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_own_check">
        <field name="name">Own Checks</field>
        <field name="res_model">l10n_latam.check</field>
        <field name="view_mode">list,form,calendar,graph,pivot</field>
        <field name="view_id" ref="view_account_own_check_tree"/>
        <field name="search_view_id" ref="view_account_payment_search"/>
        <field name="domain">[('outstanding_line_id', '!=', False)]</field>
        <field name="context">{'search_default_checks_on_hand': True}</field>
    </record>

    <menuitem
        action="action_own_check"
        id="menu_own_check"
        sequence="50"
        parent="account.menu_finance_payables"/>

<!-- Third party check Views and menus -->
    <record model="ir.ui.view" id="view_account_third_party_check_tree">
        <field name="name">account.check.list</field>
        <field name="model">l10n_latam.check</field>
        <field name="priority">110</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="view_account_own_check_tree"/>
        <field name="arch" type="xml">
            <field name="issue_state" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </field>
            <field name="original_journal_id" position="replace">
                <field name="current_journal_id" string="Current Journal"/>
            </field>

            <list position="inside">
                <header>
                    <button name="%(action_view_l10n_latam_payment_mass_transfer)d" type="action" string="Check Transfer"/>
                </header>
            </list>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_third_party_check">
        <field name="name">Third Party Checks</field>
        <field name="res_model">l10n_latam.check</field>
        <field name="view_mode">list,form,calendar,graph,pivot</field>
        <field name="view_id" ref="view_account_third_party_check_tree"/>
        <field name="search_view_id" ref="l10n_latam_check.view_account_payment_third_party_checks_search"/>
        <field name="domain">[('payment_method_code', '=', 'new_third_party_checks'), ('payment_id.state', '!=', 'draft')]</field>
        <field name="context">{'search_default_checks_on_hand': 1}</field>
    </record>

    <menuitem
        action="action_third_party_check"
        id="menu_third_party_check"
        sequence="40"
        parent="account.menu_finance_receivables"/>

</odoo>

```

## File: wizards\account_payment_register.py

```python
from odoo import models, fields, api, Command


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    l10n_latam_new_check_ids = fields.One2many('l10n_latam.payment.register.check', 'payment_register_id', string="New Checks")
    l10n_latam_move_check_ids = fields.Many2many(
        comodel_name='l10n_latam.check',
        string='Checks',
    )

    @api.depends('l10n_latam_move_check_ids.amount', 'l10n_latam_new_check_ids.amount', 'payment_method_code')
    def _compute_amount(self):
        super()._compute_amount()
        for wizard in self.filtered(lambda x: x._is_latam_check_payment(check_subtype='new_check')):
            wizard.amount = sum(wizard.l10n_latam_new_check_ids.mapped('amount'))
        for wizard in self.filtered(lambda x: x._is_latam_check_payment(check_subtype='move_check')):
            wizard.amount = sum(wizard.l10n_latam_move_check_ids.mapped('amount'))

    def _is_latam_check_payment(self, check_subtype=False):
        if check_subtype == 'move_check':
            codes = ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks']
        elif check_subtype == 'new_check':
            codes = ['new_third_party_checks', 'own_checks']
        else:
            codes = ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks', 'new_third_party_checks', 'own_checks']
        return self.payment_method_code in codes

    def _create_payment_vals_from_wizard(self, batch_result):
        vals = super()._create_payment_vals_from_wizard(batch_result)
        if self.l10n_latam_new_check_ids:
            vals.update({'l10n_latam_new_check_ids': [Command.create({
                'name': x.name,
                'bank_id': x.bank_id.id,
                'issuer_vat': x.issuer_vat,
                'payment_date': x.payment_date,
                'amount': x.amount}) for x in self.l10n_latam_new_check_ids
            ]})
        if self.l10n_latam_move_check_ids:
            vals.update({
                'l10n_latam_move_check_ids': [Command.link(x.id) for x in self.l10n_latam_move_check_ids]
            })
        return vals

```

## File: wizards\account_payment_register_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_payment_register_form" model="ir.ui.view">
        <field name="name">account.payment.register.form</field>
        <field name="model">account.payment.register</field>
        <field name="inherit_id" ref="account.view_account_payment_register_form"/>
        <field name="arch" type="xml">
            <group>
                <notebook invisible="not can_edit_wizard or (can_group_payments and not group_payment)">
                    <page name="latam_checks_page" string="Checks" invisible="payment_method_code not in ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks', 'new_third_party_checks', 'own_checks']">
                        <group name="latam_checks" colspan="2">
                            <field name="l10n_latam_new_check_ids" invisible="payment_method_code not in ['new_third_party_checks', 'own_checks']" nolabel="1" colspan="2" >
                                <list editable="bottom">
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="name" />
                                    <field name="bank_id" column_invisible="parent.payment_method_code == 'own_checks'"/>
                                    <field name="issuer_vat" column_invisible="parent.payment_method_code == 'own_checks'"/>
                                    <field name="payment_date"/>
                                    <field name="amount" />
                                </list>
                            </field>
                            <field name="l10n_latam_move_check_ids" invisible="payment_method_code not in ['in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks']"
                                domain="
                                    [('payment_method_code', '=', 'new_third_party_checks'), ('current_journal_id', '=', journal_id), ('company_id', '=', company_id)]
                                        if payment_type == 'outbound' else
                                    [('payment_method_code', '=', 'new_third_party_checks'), ('current_journal_id', '=', False), ('company_id', '=', company_id)]" options="{'no_create': True}"
                                    nolabel="1" colspan="2">
                                <list name="existing_checks">
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="name" />
                                    <field name="bank_id" optional="hide"/>
                                    <field name="issuer_vat" optional="hide"/>
                                    <field name="payment_date" optional="show"/>
                                    <field name="amount"/>
                                </list>
                            </field>
                        </group>
                    </page>
                </notebook>
            </group>
            <div role="alert" position="after">
                <div role="alert" class="alert alert-info"
                        invisible="payment_method_code not in ['new_third_party_checks', 'in_third_party_checks', 'out_third_party_checks', 'return_third_party_checks', 'own_checks'] or can_edit_wizard and (not can_group_payments or can_group_payments and group_payment)">
                    <p>You can't use checks when paying invoices of different partners or same partner without grouping</p>
                </div>
            </div>
        </field>
    </record>

</odoo>

```

## File: wizards\l10n_latam_payment_mass_transfer.py

```python
# -*- coding: utf-8 -*-

from odoo import models, api, fields, _, Command
from odoo.exceptions import UserError


class L10nLatamPaymentMassTransfer(models.TransientModel):
    _name = 'l10n_latam.payment.mass.transfer'
    _description = 'Checks Mass Transfers'
    _check_company_auto = True

    payment_date = fields.Date(
        string="Payment Date",
        required=True,
        default=fields.Date.context_today,
    )
    destination_journal_id = fields.Many2one(
        comodel_name='account.journal',
        string='Destination Journal',
        check_company=True,
        domain="[('type', 'in', ('bank', 'cash')), ('id', '!=', journal_id)]",
    )
    communication = fields.Char(string="Memo")
    journal_id = fields.Many2one(
        'account.journal',
        check_company=True,
        compute='_compute_journal_company'
    )
    company_id = fields.Many2one(
        'res.company',
        compute="_compute_journal_company"
    )
    check_ids = fields.Many2many(
        'l10n_latam.check', 'latam_tranfer_check_rel'
        'transfer_id', 'check_id', check_company=True,
    )

    @api.depends('check_ids')
    def _compute_journal_company(self):
        # use ._origin because if not a NewId for the checks is used and the returned
        # value for current_journal_id is wrong
        journal = self.check_ids._origin.mapped("current_journal_id")
        if len(journal) != 1:
            raise UserError(_("All selected checks must be on the same journal and on hand"))
        self.journal_id = journal
        self.company_id = journal.company_id.id

    @api.model
    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        if 'check_ids' in fields_list and 'check_ids' not in res:
            if self._context.get('active_model') != 'l10n_latam.check':
                raise UserError(_("The register payment wizard should only be called on account.payment records."))
            checks = self.env['l10n_latam.check'].browse(self._context.get('active_ids', []))
            if checks.filtered(lambda x: x.payment_method_line_id.code != 'new_third_party_checks'):
                raise 'You have select some payments that are not checks. Please call this action from the Third Party Checks menu'
            elif not all(check.payment_id.state != 'draft' for check in checks):
                raise UserError(_("All the selected checks must be posted"))
            currency_ids = checks.mapped('currency_id')
            if any(x != currency_ids[0] for x in currency_ids):
                raise UserError(_("All the selected checks must use the same currency"))
            res['check_ids'] = checks.ids
        return res

    def _create_payments(self):
        """ This is nedeed because we would like to create a payment of type internal transfer for each check with the
        counterpart journal and then, when posting a second payment will be created automatically """
        self.ensure_one()
        checks = self.check_ids.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks' and x.currency_id == self.check_ids[0].currency_id)
        currency_id = self.check_ids[0].currency_id

        pay_method_line = self.journal_id._get_available_payment_method_lines('outbound').filtered(
            lambda x: x.code in ('out_third_party_checks', 'return_third_party_checks')
        )[:1]

        outbound_payment = self.env['account.payment'].create({
            'date': self.payment_date,
            'amount': sum(checks.mapped('amount')),
            'partner_id': self.env.company.partner_id.id,
            'payment_type': 'outbound',
            'memo': self.communication,
            'journal_id': self.journal_id.id,
            'currency_id': currency_id.id,
            'payment_method_line_id': pay_method_line.id if pay_method_line else False,
            'l10n_latam_move_check_ids': [Command.link(x.id) for x in checks],
        })
        outbound_payment.action_post()

        inbound_payment = self.env['account.payment'].create({
            'date': self.payment_date,
            'amount': sum(checks.mapped('amount')),
            'partner_id': self.env.company.partner_id.id,
            'payment_type': 'inbound',
            'memo': self.communication,
            'journal_id': self.destination_journal_id.id,
            'currency_id': currency_id.id,
            'l10n_latam_move_check_ids': [Command.link(x.id) for x in checks],
        })

        dest_payment_method = self.destination_journal_id.inbound_payment_method_line_ids.filtered(
            lambda x: x.code == 'in_third_party_checks'
        )
        if dest_payment_method:
            inbound_payment.payment_method_line_id = dest_payment_method
            inbound_payment.action_post()
        else:
            # In case the journal is not part of the third party check, when posting the move we remove the checks
            # when the payment method line is not for checks, but in this case, we don't want to remove it so that
            # the operation_ids is filled with the two payments
            inbound_payment.with_context(l10n_ar_skip_remove_check=True).action_post()

        body_inbound = _("This payment has been created from: ") + outbound_payment._get_html_link()
        inbound_payment.message_post(body=body_inbound)
        body_outbound = _("A second payment has been created: ") + inbound_payment._get_html_link()
        outbound_payment.message_post(body=body_outbound)

        (outbound_payment.move_id.line_ids + inbound_payment.move_id.line_ids).filtered(
            lambda l:
            l.account_id == outbound_payment.destination_account_id and not l.reconciled
        ).reconcile()

        return outbound_payment

    def action_create_payments(self):
        payments = self._create_payments()

        action = {
            'name': _('Payments'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.payment',
            'context': {'create': False},
        }
        if len(payments) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': payments.id,
            })
        else:
            action.update({
                'view_mode': 'list,form',
                'domain': [('id', 'in', payments.ids)],
            })
        return action

```

## File: wizards\l10n_latam_payment_mass_transfer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_l10n_latam_payment_mass_transfer_form" model="ir.ui.view">
        <field name="name">l10n_latam.payment.mass.transfer.form</field>
        <field name="model">l10n_latam.payment.mass.transfer</field>
        <field name="arch" type="xml">
            <form>
                <field name="check_ids" invisible="1"/>
                <field name="journal_id" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <group>
                    <group name="destination_journal_group">
                        <field name="destination_journal_id" options="{'no_open': True, 'no_create': True}" required="1"/>
                    </group>
                    <group name="other_check_info">
                        <field name="payment_date"/>
                        <field name="communication"/>
                    </group>
                </group>
                <footer>
                    <button string="Create Transfers"
                        name="action_create_payments"
                        type="object"
                        class="oe_highlight"
                        data-hotkey="q"/>
                    <button string="Cancel"
                        class="btn btn-secondary"
                        special="cancel"
                        data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_view_l10n_latam_payment_mass_transfer" model="ir.actions.act_window">
        <field name="name">Check Transfer</field>
        <field name="res_model">l10n_latam.payment.mass.transfer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizards\l10n_latam_payment_register_check.py

```python
# pylint: disable=protected-access
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import models, fields, api
import stdnum

_logger = logging.getLogger(__name__)


class l10nLatamCheckPaymentRegisterCheck(models.TransientModel):
    _name = 'l10n_latam.payment.register.check'
    _description = 'Payment register check'
    _check_company_auto = True

    payment_register_id = fields.Many2one('account.payment.register', required=True, ondelete='cascade')
    company_id = fields.Many2one(related='payment_register_id.company_id')
    currency_id = fields.Many2one(related='payment_register_id.currency_id')
    name = fields.Char(string='Number')
    bank_id = fields.Many2one(
        comodel_name='res.bank',
        compute='_compute_bank_id', store=True, readonly=False,
    )
    issuer_vat = fields.Char(
        compute='_compute_issuer_vat', store=True, readonly=False,
    )
    payment_date = fields.Date(readonly=False, required=True)
    amount = fields.Monetary()

    @api.onchange('name')
    def _onchange_name(self):
        if self.name:
            self.name = self.name.zfill(8)

    @api.depends('payment_register_id.payment_method_line_id.code', 'payment_register_id.partner_id')
    def _compute_bank_id(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_register_id.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.bank_id = rec.payment_register_id.partner_id.bank_ids[:1].bank_id
        (self - new_third_party_checks).bank_id = False

    @api.depends('payment_register_id.payment_method_line_id.code', 'payment_register_id.partner_id')
    def _compute_issuer_vat(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_register_id.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.issuer_vat = rec.payment_register_id.partner_id.vat
        (self - new_third_party_checks).issuer_vat = False

    @api.onchange('issuer_vat')
    def _clean_issuer_vat(self):
        for rec in self.filtered(lambda x: x.issuer_vat and x.company_id.country_id.code):
            stdnum_vat = stdnum.util.get_cc_module(rec.company_id.country_id.code, 'vat')
            if hasattr(stdnum_vat, 'compact'):
                rec.issuer_vat = stdnum_vat.compact(rec.issuer_vat)

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import l10n_latam_payment_mass_transfer
from . import l10n_latam_payment_register_check

```

