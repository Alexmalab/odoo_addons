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
    'images': [
    ],
    'depends': [
        'account_check_printing',
        'base_vat',
    ],
    'data': [
        'data/account_payment_method_data.xml',
        'wizards/l10n_latam_payment_mass_transfer_views.xml',
        'security/ir.model.access.csv',
        'views/account_payment_view.xml',
        'views/account_journal_view.xml',
        'wizards/account_payment_register_views.xml',
    ],
    'installable': True,
    'auto_install': False,
    'application': False,
}

```

## File: data\account_payment_method_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

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

</odoo>

```

## File: models\account_chart_template.py

```python
from odoo import models, Command, api, _


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    @api.model
    def _get_third_party_checks_country_codes(self):
        """ Return the list of country codes for the countries where third party checks journals should be created
        when installing the COA"""
        return ["AR"]

    def _create_bank_journals(self, company, acc_template_ref):
        res = super()._create_bank_journals(company, acc_template_ref)

        if company.country_id.code in self._get_third_party_checks_country_codes():
            self.env['account.journal'].create({
                'name': _('Third Party Checks'),
                'type': 'cash',
                'company_id': company.id,
                'outbound_payment_method_line_ids': [
                    Command.create({'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_out_third_party_checks').id}),
                ],
                'inbound_payment_method_line_ids': [
                    Command.create({'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_new_third_party_checks').id}),
                    Command.create({'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_in_third_party_checks').id}),
                ]})
            self.env['account.journal'].create({
                'name': _('Rejected Third Party Checks'),
                'type': 'cash',
                'company_id': company.id,
                'outbound_payment_method_line_ids': [
                    Command.create({'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_out_third_party_checks').id}),
                ],
                'inbound_payment_method_line_ids': [
                    Command.create({'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_new_third_party_checks').id}),
                    Command.create({'payment_method_id': self.env.ref('l10n_latam_check.account_payment_method_in_third_party_checks').id}),
                ]})
        return res

```

## File: models\account_journal.py

```python
from odoo import models, fields, api, _
from odoo.exceptions import UserError


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    l10n_latam_manual_checks = fields.Boolean(
        string='Use electronic and deferred checks',
        help="* Allows putting numbers manually\n"
             "* Enables Check Cash-In Date feature\n"
             "* Disables printing"
    )

    @api.constrains('l10n_latam_manual_checks', 'check_manual_sequencing')
    def _check_l10n_latam_manual_checks(self):
        """ Protect from setting check_manual_sequencing (Manual Numbering) + Use electronic/deferred checks for these reasons
        * Printing checks for manual checks (electronic/deferred) is not implemented and using a "check printing" option together with the manual
          checks is confusing
        * The next check number field shown when choosing "Manual Numbering" don't have any meaning when using manual checks (electronic/deferred)
        * Some methods of account_check_printing module behave differently if "Manual Numbering" is configured
        """
        recs = self.filtered(
            lambda x: x.check_manual_sequencing and x.l10n_latam_manual_checks)
        if recs:
            raise UserError(_(
                "Manual checks (electronic/deferred) can't be used together with check manual sequencing (check printing functionality), "
                "please choose one or the other. Journals: %s", ",".join(recs.mapped("name"))))

```

## File: models\account_payment.py

```python
import stdnum

from odoo import fields, models, _, api
from odoo.exceptions import UserError, ValidationError
from odoo.tools.misc import format_date


class AccountPayment(models.Model):
    _inherit = 'account.payment'
    _rec_names_search = ['name', 'check_number']

    # Third party check operation links
    l10n_latam_check_id = fields.Many2one(
        comodel_name='account.payment',
        string='Check',
        readonly=True, states={'draft': [('readonly', False)]},
        copy=False,
        check_company=True,
    )
    l10n_latam_check_operation_ids = fields.One2many(
        comodel_name='account.payment',
        inverse_name='l10n_latam_check_id',
        string='Check Operations',
        readonly=True,
    )
    l10n_latam_check_current_journal_id = fields.Many2one(
        comodel_name='account.journal',
        string="Check Current Journal",
        compute='_compute_l10n_latam_check_current_journal', store=True,
    )
    # Warning message in case of unlogical third party check operations
    l10n_latam_check_warning_msg = fields.Text(
        compute='_compute_l10n_latam_check_warning_msg',
    )
    l10n_latam_check_number = fields.Char(
        compute='_compute_l10n_latam_check_number', inverse='_inverse_l10n_latam_check_number',
    )
    # New third party check info
    l10n_latam_check_bank_id = fields.Many2one(
        comodel_name='res.bank',
        string='Check Bank',
        compute='_compute_l10n_latam_check_bank_id', store=True, readonly=False,
        states={'posted': [('readonly', True)], 'cancel': [('readonly', True)]},
    )
    l10n_latam_check_issuer_vat = fields.Char(
        string='Check Issuer VAT',
        compute='_compute_l10n_latam_check_issuer_vat', store=True, readonly=False,
        states={'posted': [('readonly', True)], 'cancel': [('readonly', True)]},
    )
    l10n_latam_check_payment_date = fields.Date(
        string='Check Cash-In Date',
        help="Date from when you can cash in the check, turn the check into cash",
        readonly=True, states={'draft': [('readonly', False)]},
    )

    # This is a technical field for the view only
    l10n_latam_manual_checks = fields.Boolean(
        related='journal_id.l10n_latam_manual_checks',
    )

    @api.depends('check_number')
    def _compute_l10n_latam_check_number(self):
        """ This dummy computed field is added for two reasons:
        1. add a new field so that we don't need to modify attrs on the views for the original check_number field
        (not nice on terms of inheritance)
        2. if we set it as related (with readonly=False) it didn't work properly for our use case: If the user changes
        the proposed number the value was not saved. This computed with inverse does the trick"""
        for rec in self:
            rec.l10n_latam_check_number = rec.check_number

    def _inverse_l10n_latam_check_number(self):
        for rec in self:
            rec.check_number = rec.l10n_latam_check_number

    def _compute_check_number(self):
        """ Override from account_check_printing.
        For electronic/deferred own checks or third party checks, don't call super so that number is not cleaned """
        latam_checks = self.filtered(
            lambda x: x.payment_method_line_id.code == 'new_third_party_checks' or
            (x.payment_method_line_id.code == 'check_printing' and x.l10n_latam_manual_checks))
        return super(AccountPayment, self - latam_checks)._compute_check_number()

    def _inverse_check_number(self):
        """ On third party checks or electronic/deferred own checks, avoid calling super because is not needed to write
        the sequence for these use case. """
        avoid_inverse = self.filtered(
            lambda x: x.l10n_latam_manual_checks or x.payment_method_line_id.code == 'new_third_party_checks')
        return super(AccountPayment, self - avoid_inverse)._inverse_check_number()

    @api.depends('payment_method_line_id.code', 'partner_id')
    def _compute_l10n_latam_check_bank_id(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.l10n_latam_check_bank_id = rec.partner_id.bank_ids[:1].bank_id
        (self - new_third_party_checks).l10n_latam_check_bank_id = False

    @api.depends('payment_method_line_id.code', 'partner_id')
    def _compute_l10n_latam_check_issuer_vat(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.l10n_latam_check_issuer_vat = rec.partner_id.vat
        (self - new_third_party_checks).l10n_latam_check_issuer_vat = False

    @api.onchange('l10n_latam_check_issuer_vat')
    def _clean_l10n_latam_check_issuer_vat(self):
        for rec in self.filtered(lambda x: x.l10n_latam_check_issuer_vat and x.company_id.country_id.code):
            stdnum_vat = stdnum.util.get_cc_module(rec.company_id.country_id.code, 'vat')
            if hasattr(stdnum_vat, 'compact'):
                rec.l10n_latam_check_issuer_vat = stdnum_vat.compact(rec.l10n_latam_check_issuer_vat)

    @api.constrains('l10n_latam_check_issuer_vat', 'company_id')
    def _check_l10n_latam_check_issuer_vat(self):
        for rec in self.filtered(lambda x: x.l10n_latam_check_issuer_vat and x.company_id.country_id):
            if not self.env['res.partner']._run_vat_test(rec.l10n_latam_check_issuer_vat, rec.company_id.country_id):
                error_message = self.env['res.partner']._build_vat_error_message(
                    rec.company_id.country_id.code.lower(), rec.l10n_latam_check_issuer_vat, 'Check Issuer VAT')
                raise ValidationError(error_message)

    @api.depends('payment_method_line_id', 'l10n_latam_check_issuer_vat', 'l10n_latam_check_bank_id', 'company_id',
                 'l10n_latam_check_number', 'l10n_latam_check_id', 'state', 'date', 'is_internal_transfer', 'amount', 'currency_id')
    def _compute_l10n_latam_check_warning_msg(self):
        """
        Compute warning message for latam checks checks
        We use l10n_latam_check_number as de dependency because on the interface this is the field the user is using.
        Another approach could be to add an onchange on _inverse_l10n_latam_check_number method
        """
        self.l10n_latam_check_warning_msg = False
        latam_draft_checks = self.filtered(
            lambda x: x.state == 'draft' and (x.l10n_latam_manual_checks or x.payment_method_line_id.code in [
                'in_third_party_checks', 'out_third_party_checks', 'new_third_party_checks']))
        for rec in latam_draft_checks:
            msgs = rec._get_blocking_l10n_latam_warning_msg()
            # new third party check
            if rec.l10n_latam_check_number and rec.payment_method_line_id.code == 'new_third_party_checks' and \
                    rec.l10n_latam_check_bank_id and rec.l10n_latam_check_issuer_vat:
                same_checks = self.search([
                    ('company_id', '=', rec.company_id.id),
                    ('l10n_latam_check_bank_id', '=', rec.l10n_latam_check_bank_id.id),
                    ('l10n_latam_check_issuer_vat', '=', rec.l10n_latam_check_issuer_vat),
                    ('check_number', '=', rec.l10n_latam_check_number),
                    ('id', '!=', rec._origin.id)])
                if same_checks:
                    msgs.append(_(
                        "Other checks were found with same number, issuer and bank. Please double check you are not "
                        "encoding the same check more than once. List of other payments/checks: %s",
                        ", ".join(same_checks.mapped('display_name'))))
            rec.l10n_latam_check_warning_msg = msgs and '* %s' % '\n* '.join(msgs) or False

    def _get_blocking_l10n_latam_warning_msg(self):
        msgs = []
        for rec in self.filtered('l10n_latam_check_id'):
            if rec.currency_id != rec.l10n_latam_check_id.currency_id:
                msgs.append(_(
                    'The currency of the payment (%s) and the currency of the check (%s) must be the same.') % (
                        rec.currency_id.name, rec.l10n_latam_check_id.currency_id.name))
            if not rec.currency_id.is_zero(rec.l10n_latam_check_id.amount - rec.amount):
                msgs.append(_(
                    'The amount of the payment (%s) does not match the amount of the selected check (%s). '
                    'Please try to deselect and select the check again.', rec.amount, rec.l10n_latam_check_id.amount))
            if rec.payment_method_line_id.code in ['in_third_party_checks', 'out_third_party_checks']:
                if rec.l10n_latam_check_id.state != 'posted':
                    msgs.append(_('Selected check "%s" is not posted', rec.l10n_latam_check_id.display_name))
                elif (rec.payment_type == 'outbound' and
                        rec.l10n_latam_check_id.l10n_latam_check_current_journal_id != rec.journal_id) or (
                        rec.payment_type == 'inbound' and rec.is_internal_transfer and
                        rec.l10n_latam_check_id.l10n_latam_check_current_journal_id != rec.destination_journal_id):
                    # check outbound payment and transfer or inbound transfer
                    msgs.append(_(
                        'Check "%s" is not anymore in journal "%s", it seems it has been moved by another payment.',
                        rec.l10n_latam_check_id.display_name, rec.journal_id.name
                        if rec.payment_type == 'outbound' else rec.destination_journal_id.name))
                elif rec.payment_type == 'inbound' and not rec.is_internal_transfer and \
                        rec.l10n_latam_check_id.l10n_latam_check_current_journal_id:
                    msgs.append(_("Check '%s' is on journal '%s', it can't be received it again",
                                rec.l10n_latam_check_id.display_name, rec.journal_id.name))
            # moved third party check
            if rec.l10n_latam_check_id:
                date = rec.date or fields.Datetime.now()
                last_operation = rec.env['account.payment'].search([
                    ('state', '=', 'posted'),
                    '|', ('l10n_latam_check_id', '=', rec.l10n_latam_check_id.id),
                    ('id', '=', rec.l10n_latam_check_id.id),
                ], order="date desc, id desc", limit=1)
                if last_operation and last_operation[0].date > date:
                    msgs.append(_(
                        "It seems you're trying to move a check with a date (%s) prior to last operation done with "
                        "the check (%s). This may be wrong, please double check it. By continue, the last operation on "
                        "the check will remain being %s",
                        format_date(self.env, date), last_operation.display_name, last_operation.display_name))
        return msgs

    @api.depends('is_internal_transfer')
    def _compute_payment_method_line_fields(self):
        """ Add is_internal_transfer as a trigger to re-compute """
        return super()._compute_payment_method_line_fields()

    @api.depends('l10n_latam_check_operation_ids.state', 'payment_method_line_id.code')
    def _compute_l10n_latam_check_current_journal(self):
        new_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        payments = self.env['account.payment'].search(
            [('l10n_latam_check_id', 'in', new_checks.ids), ('state', '=', 'posted')], order="date desc, id desc")

        # we store on a dict the first payment (last operation) for each check
        checks_mapping = {}
        for payment in payments:
            if payment.l10n_latam_check_id not in checks_mapping:
                checks_mapping[payment.l10n_latam_check_id] = payment

        for rec in new_checks:
            last_operation = checks_mapping.get(rec)
            if not last_operation:
                rec.l10n_latam_check_current_journal_id = rec.journal_id
                continue
            if last_operation.is_internal_transfer and last_operation.payment_type == 'outbound':
                rec.l10n_latam_check_current_journal_id = last_operation.paired_internal_transfer_payment_id.journal_id
            elif last_operation.payment_type == 'inbound':
                rec.l10n_latam_check_current_journal_id = last_operation.journal_id
            else:
                rec.l10n_latam_check_current_journal_id = False

    @api.depends('l10n_latam_manual_checks')
    def _compute_show_check_number(self):
        latam_checks = self.filtered(
            lambda x: x.payment_method_line_id.code == 'new_third_party_checks' or
            (x.payment_method_line_id.code == 'check_printing' and x.l10n_latam_manual_checks))
        latam_checks.show_check_number = False
        super(AccountPayment, self - latam_checks)._compute_show_check_number()

    @api.constrains('check_number', 'journal_id')
    def _constrains_check_number_unique(self):
        """ Don't enforce uniqueness for third party checks"""
        third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        return super(AccountPayment, self - third_party_checks)._constrains_check_number_unique()

    @api.onchange('l10n_latam_check_id')
    def _onchange_check(self):
        for rec in self.filtered('l10n_latam_check_id'):
            rec.amount = rec.l10n_latam_check_id.amount

    @api.onchange('payment_method_line_id', 'is_internal_transfer', 'journal_id', 'destination_journal_id')
    def _onchange_to_reset_check_ids(self):
        # If any of these fields change, the domain of the selectable checks could change
        self.l10n_latam_check_id = False

    @api.onchange('l10n_latam_check_number')
    def _onchange_check_number(self):
        for rec in self.filtered(
            lambda x: x.journal_id.company_id.country_id.code == "AR" and
            x.l10n_latam_check_number and x.l10n_latam_check_number.isdecimal()):
            rec.l10n_latam_check_number = '%08d' % int(rec.l10n_latam_check_number)

    def _get_payment_method_codes_to_exclude(self):
        res = super()._get_payment_method_codes_to_exclude()
        if self.is_internal_transfer:
            res.append('new_third_party_checks')
        return res

    def action_unmark_sent(self):
        """ Unmarking as sent for electronic/deferred check would give the option to print and re-number check but
        it's not implemented yet for this kind of checks"""
        if self.filtered(lambda x: x.payment_method_line_id.code == 'check_printing' and x.l10n_latam_manual_checks):
            raise UserError(_('Unmark sent is not implemented for electronic or deferred checks'))
        return super().action_unmark_sent()

    def action_post(self):
        msgs = self._get_blocking_l10n_latam_warning_msg()
        if msgs:
            raise ValidationError('* %s' % '\n* '.join(msgs))

        res = super().action_post()

        # mark own checks that are not printed as sent
        self.filtered(lambda x: x.payment_method_line_id.code == 'check_printing' and x.l10n_latam_manual_checks).write({'is_move_sent': True})
        return res

    def action_draft(self):
        if self.l10n_latam_check_operation_ids.filtered(lambda x: x.state == "posted"):
            raise ValidationError(_(
               "This third party check is already used to make one or more payments. Please reset them to draft first.\n"
                "Payments made with this check: %s",
                "".join(f'\n    - {payment.name}' for payment in self.l10n_latam_check_operation_ids)))
        return super().action_draft()

    @api.model
    def _get_trigger_fields_to_synchronize(self):
        res = super()._get_trigger_fields_to_synchronize()
        return res + ('l10n_latam_check_number',)

    def _prepare_move_line_default_vals(self, write_off_line_vals=None):
        """ Add check name and operation on liquidity line """
        res = super()._prepare_move_line_default_vals(write_off_line_vals=write_off_line_vals)
        check = self if (self.payment_method_line_id.code == 'new_third_party_checks' or (self.payment_method_line_id.code == 'check_printing' and self.l10n_latam_manual_checks)) \
            else self.l10n_latam_check_id
        if check:
            document_name = (_('Check %s received') if self.payment_type == 'inbound' else _('Check %s delivered')) % (
                check.check_number)
            res[0].update({
                'name': document_name + ' - ' + ''.join([item[1] for item in self._get_aml_default_display_name_list()]),
            })
        return res

    def name_get(self):
        """ Add check number to display_name on check_id m2o field """
        res_names = super().name_get()
        for i, (res_name, rec) in enumerate(zip(res_names, self)):
            if rec.check_number and rec.payment_method_line_id.code == 'new_third_party_checks':
                res_names[i] = (res_name[0], "%s %s" % (res_name[1], _("(Check %s)", rec.check_number)))
        return res_names

    def button_open_check_operations(self):
        ''' Redirect the user to the invoice(s) paid by this payment.
        :return:    An action on account.move.
        '''
        self.ensure_one()

        operations = (self.l10n_latam_check_operation_ids.filtered(lambda x: x.state == 'posted') + self)
        action = {
            'name': _("Check Operations"),
            'type': 'ir.actions.act_window',
            'res_model': 'account.payment',
            'views': [
                (self.env.ref('l10n_latam_check.view_account_third_party_check_operations_tree').id, 'tree'),
                (False, 'form')],
            'context': {'create': False},
            'domain': [('id', 'in', operations.ids)],
        }
        return action

    def _create_paired_internal_transfer_payment(self):
        """
        Two modifications when only when transferring from a third party checks journal:
        1. When a paired transfer is created, the default odoo behavior is to use on the paired transfer the first
        available payment method. If we are transferring to another third party checks journal, then set as payment
        method on the paired transfer 'in_third_party_checks' or 'out_third_party_checks'
        2. On the paired transfer set the l10n_latam_check_id field, this field is needed for the
        l10n_latam_check_operation_ids and also for some warnings and constrains.
        """
        third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code in [
            'in_third_party_checks',
            'out_third_party_checks'
        ])
        for rec in third_party_checks:
            dest_payment_method_code = 'in_third_party_checks' if rec.payment_type == 'outbound' else 'out_third_party_checks'
            dest_payment_method = rec.destination_journal_id.inbound_payment_method_line_ids.filtered(
                lambda x: x.code == dest_payment_method_code)
            if dest_payment_method:
                super(AccountPayment, rec.with_context(
                    default_payment_method_line_id=dest_payment_method.id,
                    default_l10n_latam_check_id=rec.l10n_latam_check_id,
                ))._create_paired_internal_transfer_payment()
            else:
                super(AccountPayment, rec.with_context(
                    default_l10n_latam_check_id=rec.l10n_latam_check_id,
                ))._create_paired_internal_transfer_payment()
        super(AccountPayment, self - third_party_checks)._create_paired_internal_transfer_payment()

    @api.constrains('l10n_latam_check_id')
    def _check_l10n_latam_check_id(self):
        if self.filtered(lambda x: x.payment_method_line_id.code == 'out_third_party_checks'):
            payments = self.env['account.payment'].search_count([
                ('l10n_latam_check_id', 'in', self.l10n_latam_check_id.ids),
                ('payment_type', '=', 'outbound'),
                ('journal_id', 'in', self.journal_id.ids),
                ('id', 'not in', self.ids)],
                limit=1)
            if payments:
                raise ValidationError(_(
                    "The check(s) '%s' is already used on another payment. Please select another check or "
                    "deselect the check on this payment.", self.l10n_latam_check_id.mapped('display_name')))

```

## File: models\account_payment_method.py

```python
from odoo import models, api


class AccountPaymentMethod(models.Model):
    _inherit = 'account.payment.method'

    @api.model
    def _get_payment_method_information(self):
        res = super()._get_payment_method_information()
        res['new_third_party_checks'] = {'mode': 'multi', 'domain': [('type', '=', 'cash')]}
        res['in_third_party_checks'] = {'mode': 'multi', 'domain': [('type', '=', 'cash')]}
        res['out_third_party_checks'] = {'mode': 'multi', 'domain': [('type', '=', 'cash')]}
        return res

```

## File: models\__init__.py

```python
from . import account_journal
from . import account_payment
from . import account_payment_method
from . import account_chart_template

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_latam_payment_mass_transfer,access_l10n_latam_payment_mass_transfer,model_l10n_latam_payment_mass_transfer,account.group_account_invoice,1,1,1,0

```

## File: views\account_journal_view.xml

```xml
<odoo>

    <record id="view_account_journal_tree" model="ir.ui.view">
        <field name="name">account.journal.tree</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account_check_printing.view_account_journal_form_inherited"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='check_sequence_id']/.." position="after">
                <group string="Checks Management" name="check_management"
                    attrs="{'invisible': ['|', '!', ('selected_payment_method_codes', 'ilike', ',check_printing,'), ('type', '!=', 'bank')]}">
                    <field name="l10n_latam_manual_checks"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\account_payment_view.xml

```xml
<odoo>

    <!-- Own checks search view -->
    <record model="ir.ui.view" id="view_account_payment_search">
        <field name="name">account.check.search</field>
        <field name="model">account.payment</field>
        <field name="priority">20</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="account.view_account_payment_search"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="check_number"/>
            </field>
            <filter name="date" position="after">
                <separator/>
            </filter>
            <filter name="groupby_date" position="after">
                <filter string="Check Cash-In Date"
                    name="groupby_l10n_latam_check_payment_date"
                    context="{'group_by': 'l10n_latam_check_payment_date'}"/>
            </filter>
        </field>
    </record>

    <!-- Third party checks search view -->
    <record model="ir.ui.view" id="view_account_payment_third_party_checks_search">
        <field name="name">account.check.search</field>
        <field name="model">account.payment</field>
        <field name="priority">22</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="account.view_account_payment_search"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="check_number"/>
            </field>
            <field name="journal_id" position="after">
                <field name="l10n_latam_check_current_journal_id"/>
            </field>
            <filter name="date" position="after">
                <separator/>
                <filter string="Checks on hand" name="checks_on_hand"
                    domain="[('state', '=', 'posted'),
                                ('l10n_latam_check_current_journal_id.inbound_payment_method_line_ids.payment_method_id.code', 'in', ['new_third_party_checks', 'in_third_party_checks'])]"/>
            </filter>
            <filter name="journal" position="after">
                <filter name="groupby_third_party_check_current_journal"
                    string="Current Journal"
                    context="{'group_by': 'l10n_latam_check_current_journal_id'}"/>
            </filter>
            <filter name="unmatched" position="attributes">
                <attribute name="invisible">1</attribute>
            </filter>
            <filter name="groupby_date" position="after">
                <filter string="Check Cash-In Date"
                    name="groupby_l10n_latam_check_payment_date"
                    context="{'group_by': 'l10n_latam_check_payment_date'}"/>
            </filter>
        </field>
    </record>

    <record id="view_account_payment_form_inherited" model="ir.ui.view">
        <field name="name">account.payment.form.inherited</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="account_check_printing.view_account_payment_form_inherited" />
        <field name="arch" type="xml">
            <sheet position="before">
                <div class="alert alert-danger mb-0" role="alert"
                        attrs="{'invisible': [('l10n_latam_check_warning_msg', '=', False)]}">
                    <field name="l10n_latam_check_warning_msg" nolabel="1"/>
                </div>
            </sheet>

            <field name="destination_journal_id" position="after">
                <!-- Move Third party checks -->
                <field name="l10n_latam_check_id"
                    attrs="{
                        'invisible': [('payment_method_code', 'not in', ['in_third_party_checks', 'out_third_party_checks']), ('l10n_latam_check_id', '=', False)],
                        'required': [('payment_method_code', 'in', ['in_third_party_checks', 'out_third_party_checks'])]}"
                    domain="
                        [('payment_method_code', '=', 'new_third_party_checks'), ('l10n_latam_check_current_journal_id', '=', journal_id), ('state', '=', 'posted'), ('company_id', '=', company_id)]
                            if payment_type == 'outbound' else
                        [('payment_method_code', '=', 'new_third_party_checks'), ('l10n_latam_check_current_journal_id', '=', destination_journal_id), ('state', '=', 'posted'), ('company_id', '=', company_id)]
                            if is_internal_transfer else
                        [('payment_method_code', '=', 'new_third_party_checks'), ('l10n_latam_check_current_journal_id', '=', False), ('state', '=', 'posted'), ('company_id', '=', company_id)]"
                    context="{'search_view_ref': 'l10n_latam_check.view_account_payment_third_party_checks_search'}"
                    options="{'no_create': True}"
                />
            </field>
            <field name="payment_method_line_id" position="after">
                <field name="l10n_latam_manual_checks" invisible="1"/>
                <field name="l10n_latam_check_number"
                    string='Check Number'
                    attrs="{
                        'invisible': [('payment_method_code', '!=', 'new_third_party_checks'), '|', ('payment_method_code', '!=', 'check_printing'), ('l10n_latam_manual_checks', '=', False)],
                        'required': ['|', ('payment_method_code', '=', 'new_third_party_checks'), '&amp;', ('payment_method_code', '=', 'check_printing'), ('l10n_latam_manual_checks', '=', True)],
                        'readonly': [('state', '!=', 'draft')]}"/>
                <field name="l10n_latam_check_payment_date" attrs="{
                        'invisible': [('payment_method_code', '!=', 'new_third_party_checks'), '|', ('payment_method_code', '!=', 'check_printing'), ('l10n_latam_manual_checks', '=', False)]}"/>
                <field name="l10n_latam_check_bank_id" string="Check Bank"
                    attrs="{'invisible': [('payment_method_code', '!=', 'new_third_party_checks')]}"/>
                <field name="l10n_latam_check_issuer_vat" string="Check Issuer Vat"
                    attrs="{'invisible': [('payment_method_code', '!=', 'new_third_party_checks')]}"/>
                <label for="l10n_latam_check_current_journal_id" string="Check Current Journal"
                    attrs="{'invisible': ['|', ('state', '!=', 'posted'), ('payment_method_code', '!=', 'new_third_party_checks')]}"/>
                <div class="oe_inline"
                        attrs="{'invisible': ['|', ('state', '!=', 'posted'), ('payment_method_code', '!=', 'new_third_party_checks')]}">
                    <field name="l10n_latam_check_current_journal_id"/>
                    <span attrs="{'invisible': [('l10n_latam_check_current_journal_id', '!=', False)]}">Not in Wallet</span>
                    <button name="button_open_check_operations" type="object" string="⇒ Check Operations" class="oe_link"/>
                </div>
            </field>
        </field>
    </record>

    <record model="ir.ui.view" id="view_account_third_party_check_operations_tree">
        <field name="name">account.check.operations.tree</field>
        <field name="model">account.payment</field>
        <field name="priority" eval="99"/>
        <field name="arch" type="xml">
            <tree default_order="date desc, id desc, name desc">
                <field name="date"/>
                <field name="name"/>
                <field name="payment_type"/>
                <field name="journal_id"/>
                <field name="partner_id" string="Customer"/>
                <field name="state" invisible="1"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="view_account_check_calendar">
        <field name="name">account.check.calendar</field>
        <field name="model">account.payment</field>
        <field name="arch" type="xml">
            <calendar
                    mode="month"
                    date_start="l10n_latam_check_payment_date"
                    color="journal_id">
                <field name="amount"/>
            </calendar>
        </field>
    </record>

    <record model="ir.ui.view" id="view_account_check_pivot">
        <field name="name">account.check.calendar</field>
        <field name="model">account.payment</field>
        <field name="arch" type="xml">
            <pivot>
                <field name="l10n_latam_check_payment_date" type="row" interval="month"/>
                <field name="l10n_latam_check_payment_date" type="row" interval="week"/>
            <field name="amount" type="measure"/>
            </pivot>
        </field>
    </record>

 <!-- Own Check Views and menus -->

    <record model="ir.ui.view" id="view_account_own_check_tree">
        <field name="name">account.check.tree</field>
        <field name="model">account.payment</field>
        <field name="priority">100</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="account.view_account_payment_tree"/>
        <field name="arch" type="xml">
            <field name="name" position="attributes">
                <attribute name="optional">hide</attribute>
            </field>
            <field name="name" position="after">
                <field name="check_number"/>
            </field>
            <field name="payment_method_line_id" position="attributes">
                <attribute name="invisible">1</attribute>
            </field>
            <field name="date" position="after">
                <field name="l10n_latam_check_payment_date" optional="show"/>
            </field>
            <tree>
                <field name="is_matched"/>
            </tree>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_own_check">
        <field name="name">Own Checks</field>
        <field name="res_model">account.payment</field>
        <field name="view_mode">tree,form,calendar,graph,pivot</field>
        <field name="view_id" ref="view_account_own_check_tree"/>
        <field name="search_view_id" ref="view_account_payment_search"/>
        <field name="domain">[('payment_method_code', '=', 'check_printing')]</field>
        <field name="context">{'search_default_unmatched': True}</field>
    </record>

    <menuitem
        action="action_own_check"
        id="menu_own_check"
        sequence="50"
        parent="account.menu_finance_payables"/>

<!-- Third party check Views and menus -->

    <record model="ir.ui.view" id="view_account_third_party_check_tree">
        <field name="name">account.check.tree</field>
        <field name="model">account.payment</field>
        <field name="priority">110</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="view_account_own_check_tree"/>
        <field name="arch" type="xml">
            <field name="is_matched" position="replace"/>
            <field name="journal_id" position="replace">
                <field name="l10n_latam_check_current_journal_id" string="Current Journal"/>
            </field>
            <tree position="attributes">
                <attribute name="create">false</attribute>
            </tree>
            <tree position="inside">
                <header>
                    <button name="%(action_view_l10n_latam_payment_mass_transfer)d" type="action" string="Check Transfer" groups="account.group_account_user"/>
                </header>
            </tree>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_third_party_check">
        <field name="name">Third Party Checks</field>
        <field name="res_model">account.payment</field>
        <field name="view_mode">tree,form,calendar,graph,pivot</field>
        <field name="view_id" ref="view_account_third_party_check_tree"/>
        <field name="search_view_id" ref="l10n_latam_check.view_account_payment_third_party_checks_search"/>
        <field name="domain">[('payment_method_code', '=', 'new_third_party_checks')]</field>
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
from odoo import models, fields, api


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    l10n_latam_check_id = fields.Many2one(
        comodel_name='account.payment',
        string='Check',
    )
    l10n_latam_check_bank_id = fields.Many2one(
        comodel_name='res.bank',
        string='Check Bank',
        compute='_compute_l10n_latam_check_bank_id', store=True, readonly=False,
    )
    l10n_latam_check_issuer_vat = fields.Char(
        string='Check Issuer VAT',
        compute='_compute_l10n_latam_check_issuer_vat', store=True, readonly=False,
    )
    l10n_latam_check_number = fields.Char(
        string="Check Number",
    )
    l10n_latam_manual_checks = fields.Boolean(
        related='journal_id.l10n_latam_manual_checks',
    )
    l10n_latam_check_payment_date = fields.Date(
        string='Check Cash-In Date', help="Date from when you can cash in the check, turn the check into cash",
    )

    @api.depends('payment_method_line_id.code', 'partner_id')
    def _compute_l10n_latam_check_bank_id(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.l10n_latam_check_bank_id = rec.partner_id.bank_ids[:1].bank_id
        (self - new_third_party_checks).l10n_latam_check_bank_id = False

    @api.depends('payment_method_line_id.code', 'partner_id')
    def _compute_l10n_latam_check_issuer_vat(self):
        new_third_party_checks = self.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        for rec in new_third_party_checks:
            rec.l10n_latam_check_issuer_vat = rec.partner_id.vat
        (self - new_third_party_checks).l10n_latam_check_issuer_vat = False

    @api.depends('l10n_latam_check_id')
    def _compute_amount(self):
        super()._compute_amount()
        for wizard in self.filtered('l10n_latam_check_id'):
            wizard.amount = wizard.l10n_latam_check_id.amount

    @api.depends('l10n_latam_check_id')
    def _compute_currency_id(self):
        super()._compute_currency_id()
        for wizard in self.filtered('l10n_latam_check_id'):
            wizard.currency_id = wizard.l10n_latam_check_id.currency_id

    @api.onchange('l10n_latam_check_number')
    def _onchange_l10n_latam_check_number(self):
        for rec in self.filtered(lambda x: x.journal_id.company_id.country_id.code == "AR" and x.l10n_latam_check_number
                                 and x.l10n_latam_check_number.isdecimal()):
            rec.l10n_latam_check_number = '%08d' % int(rec.l10n_latam_check_number)

    @api.onchange('payment_method_line_id', 'journal_id')
    def _onchange_to_reset_check_ids(self):
        # If any of these fields change, the domain of the selectable checks could change
        self.l10n_latam_check_id = False

    def _create_payment_vals_from_wizard(self, batch_result):
        vals = super()._create_payment_vals_from_wizard(batch_result)
        vals.update({
            'l10n_latam_check_id': self.l10n_latam_check_id.id,
            'l10n_latam_check_bank_id': self.l10n_latam_check_bank_id.id,
            'l10n_latam_check_issuer_vat': self.l10n_latam_check_issuer_vat,
            'check_number': self.l10n_latam_check_number,
            'l10n_latam_check_payment_date': self.l10n_latam_check_payment_date,
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
            <field name="payment_method_line_id" position="after">
                <field name="l10n_latam_manual_checks" invisible="1"/>
                <field name="payment_method_code" invisible="1"/>
                <field name="l10n_latam_check_number"
                    attrs="{
                        'invisible': [('payment_method_code', '!=', 'new_third_party_checks'), '|', ('l10n_latam_manual_checks', '=', False), ('payment_method_code', '!=', 'check_printing')],
                        'required': [
                            '|', '&amp;', ('payment_method_code', '=', 'check_printing'), ('l10n_latam_manual_checks', '!=', False), ('payment_method_code', '=', 'new_third_party_checks'),
                        ]}"/>

                <div class="o_row"
                        attrs="{'invisible': ['|', ('can_edit_wizard', '=', False), '&amp;', ('can_group_payments', '=', True), ('group_payment', '=', False)]}" colspan="2">
                    <group>
                        <field name="l10n_latam_check_id"
                            attrs="{
                                'invisible': [('payment_method_code', 'not in', ['in_third_party_checks', 'out_third_party_checks'])],
                                'required': [('payment_method_code', 'in', ['in_third_party_checks', 'out_third_party_checks'])]}"
                            domain="
                                [('payment_method_code', '=', 'new_third_party_checks'), ('l10n_latam_check_current_journal_id', '=', journal_id), ('state', '=', 'posted')]
                                    if payment_type == 'outbound' else
                                [('payment_method_code', '=', 'new_third_party_checks'), ('l10n_latam_check_current_journal_id', '=', False), ('state', '=', 'posted')]"
                            options="{'no_create': True}"/>
                        <field name="l10n_latam_check_payment_date"
                            attrs="{'invisible': [('payment_method_code', '!=', 'new_third_party_checks'), '|', ('l10n_latam_manual_checks', '=', False), ('payment_method_code', '!=', 'check_printing')]}"/>
                        <field name="l10n_latam_check_bank_id" string="Check Bank"
                            attrs="{'invisible': [('payment_method_code', '!=', 'new_third_party_checks')]}"/>
                        <field name="l10n_latam_check_issuer_vat" string="Check Issuer Vat"
                            attrs="{'invisible': [('payment_method_code', '!=', 'new_third_party_checks')]}"/>
                    </group>
                </div>
                <div colspan="2" class="o_row"
                        attrs="{'invisible': ['|', ('payment_method_code', 'not in', ['new_third_party_checks', 'in_third_party_checks', 'out_third_party_checks']), '&amp;', ('can_edit_wizard', '=', True), '|', ('can_group_payments', '=', False), '&amp;', ('can_group_payments', '=', True), ('group_payment', '=', True)]}">
                    <p class="alert alert-warning" role="alert">You can't use checks when paying invoices of different partners or same partner without grouping</p>
                </div>
            </field>
        </field>
    </record>

</odoo>

```

## File: wizards\l10n_latam_payment_mass_transfer.py

```python
# -*- coding: utf-8 -*-

from odoo import models, api, fields, _
from odoo.exceptions import UserError


class L10nLatamPaymentMassTransfer(models.TransientModel):
    _name = 'l10n_latam.payment.mass.transfer'
    _description = 'Checks Mass Transfers'

    payment_date = fields.Date(
        string="Payment Date",
        required=True,
        default=fields.Date.context_today,
    )
    destination_journal_id = fields.Many2one(
        comodel_name='account.journal',
        string='Destination Journal',
        domain="[('type', 'in', ('bank', 'cash')), ('company_id', '=', company_id), ('id', '!=', journal_id)]",
    )
    communication = fields.Char(
        string="Memo",
    )
    journal_id = fields.Many2one(
        'account.journal',
        compute='_compute_journal_company'
    )
    company_id = fields.Many2one(
        'res.company',
        compute="_compute_journal_company"
    )
    check_ids = fields.Many2many(
        'account.payment',
    )

    @api.depends('check_ids')
    def _compute_journal_company(self):
        # use ._origin because if not a NewId for the checks is used and the returned
        # value for l10n_latam_check_current_journal_id is wrong
        journal = self.check_ids._origin.mapped("l10n_latam_check_current_journal_id")
        if len(journal) != 1 or not journal.inbound_payment_method_line_ids.filtered(
           lambda x: x.code == 'in_third_party_checks'):
            raise UserError(_("All selected checks must be on the same journal and on hand"))
        self.journal_id = journal
        self.company_id = journal.company_id.id

    @api.model
    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        if 'check_ids' in fields_list and 'check_ids' not in res:
            if self._context.get('active_model') != 'account.payment':
                raise UserError(_("The register payment wizard should only be called on account.payment records."))
            checks = self.env['account.payment'].browse(self._context.get('active_ids', []))
            if checks.filtered(lambda x: x.payment_method_line_id.code != 'new_third_party_checks'):
                raise 'You have select some payments that are not checks. Please call this action from the Third Party Checks menu'
            elif not all(check.state == 'posted' for check in checks):
                raise UserError(_("All the selected checks must be posted"))
            res['check_ids'] = checks.ids
        return res

    def _create_payments(self):
        """ This is nedeed because we would like to create a payment of type internal transfer for each check with the
        counterpart journal and then, when posting a second payment will be created automatically """
        self.ensure_one()
        checks = self.check_ids.filtered(lambda x: x.payment_method_line_id.code == 'new_third_party_checks')
        payment_vals_list = []

        pay_method_line = self.journal_id._get_available_payment_method_lines('outbound').filtered(
            lambda x: x.code == 'out_third_party_checks')

        for check in checks:
            payment_vals_list.append({
                'date': self.payment_date,
                'l10n_latam_check_id': check.id,
                'amount': check.amount,
                'payment_type': 'outbound',
                'ref': self.communication,
                'journal_id': self.journal_id.id,
                'currency_id': check.currency_id.id,
                'is_internal_transfer': True,
                'payment_method_line_id': pay_method_line.id,
                'destination_journal_id': self.destination_journal_id.id,
            })
        payments = self.env['account.payment'].create(payment_vals_list)
        payments.action_post()
        return payments

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
                'view_mode': 'tree,form',
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
                        data-hotkey="z"/>
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

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import l10n_latam_payment_mass_transfer

```

