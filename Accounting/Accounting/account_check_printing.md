# Odoo Module: account_check_printing

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Check Printing Base',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'summary': 'Check printing commons',
    'description': """
This module offers the basic functionalities to make payments by printing checks.
It must be used as a dependency for modules that provide country-specific check templates.
The check settings are located in the accounting journals configuration page.
    """,
    'depends': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'data/account_check_printing_data.xml',
        'views/account_journal_views.xml',
        'views/account_move_views.xml',
        'views/account_payment_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'wizard/print_prenumbered_checks_views.xml'
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\account_check_printing_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="account_payment_method_check" model="account.payment.method">
            <field name="name">Checks</field>
            <field name="code">check_printing</field>
            <field name="payment_type">outbound</field>
            <field name="sequence">20</field>
        </record>

        <function model="account.journal" name="_enable_check_printing_on_bank_journals"/>


        <record model="ir.actions.server" id="action_account_print_checks">
            <field name="name">Print Checks</field>
            <field name="model_id" ref="account.model_account_payment"/>
            <field name="binding_model_id" ref="account.model_account_payment" />
            <field name="binding_view_types">list</field>
            <field name="groups_id" eval="[(4, ref('account.group_account_user'))]"/>
            <field name="state">code</field>
            <field name="code">
if records:
    action = records.print_checks()
            </field>
        </record>

    </data>
</odoo>

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from odoo import models, fields, api, _
from odoo.exceptions import ValidationError

class AccountJournal(models.Model):
    _inherit = "account.journal"

    check_manual_sequencing = fields.Boolean(
        string='Manual Numbering',
        default=False,
        help="Check this option if your pre-printed checks are not numbered.",
    )
    check_sequence_id = fields.Many2one(
        comodel_name='ir.sequence',
        string='Check Sequence',
        readonly=True,
        copy=False,
        help="Checks numbering sequence.",
    )
    check_next_number = fields.Char(
        string='Next Check Number',
        compute='_compute_check_next_number',
        inverse='_inverse_check_next_number',
        help="Sequence number of the next printed check.",
    )
    check_printing_payment_method_selected = fields.Boolean(
        compute='_compute_check_printing_payment_method_selected',
        help="Technical feature used to know whether check printing was enabled as payment method.",
    )

    @api.depends('check_manual_sequencing')
    def _compute_check_next_number(self):
        for journal in self:
            sequence = journal.check_sequence_id
            if sequence:
                journal.check_next_number = sequence.get_next_char(sequence.number_next_actual)
            else:
                journal.check_next_number = 1

    def _inverse_check_next_number(self):
        for journal in self:
            if journal.check_next_number and not re.match(r'^[0-9]+$', journal.check_next_number):
                raise ValidationError(_('Next Check Number should only contains numbers.'))
            if int(journal.check_next_number) < journal.check_sequence_id.number_next_actual:
                raise ValidationError(_(
                    "The last check number was %s. In order to avoid a check being rejected "
                    "by the bank, you can only use a greater number.",
                    journal.check_sequence_id.number_next_actual
                ))
            if journal.check_sequence_id:
                journal.check_sequence_id.sudo().number_next_actual = int(journal.check_next_number)
                journal.check_sequence_id.sudo().padding = len(journal.check_next_number)

    @api.depends('type')
    def _compute_outbound_payment_method_ids(self):
        super()._compute_outbound_payment_method_ids()
        for journal in self:
            if journal.type == 'cash':
                check_method = self.env.ref('account_check_printing.account_payment_method_check')
                journal.outbound_payment_method_ids -= check_method

    @api.depends('outbound_payment_method_ids')
    def _compute_check_printing_payment_method_selected(self):
        for journal in self:
            journal.check_printing_payment_method_selected = any(
                pm.code == 'check_printing'
                for pm in journal.outbound_payment_method_ids
            )

    @api.model
    def create(self, vals):
        rec = super(AccountJournal, self).create(vals)
        if not rec.check_sequence_id:
            rec._create_check_sequence()
        return rec

    def _create_check_sequence(self):
        """ Create a check sequence for the journal """
        for journal in self:
            journal.check_sequence_id = self.env['ir.sequence'].sudo().create({
                'name': journal.name + _(" : Check Number Sequence"),
                'implementation': 'no_gap',
                'padding': 5,
                'number_increment': 1,
                'company_id': journal.company_id.id,
            })

    def _default_outbound_payment_methods(self):
        methods = super(AccountJournal, self)._default_outbound_payment_methods()
        return methods + self.env.ref('account_check_printing.account_payment_method_check')

    @api.model
    def _enable_check_printing_on_bank_journals(self):
        """ Enables check printing payment method and add a check sequence on bank journals.
            Called upon module installation via data file.
        """
        check_method = self.env.ref('account_check_printing.account_payment_method_check')
        for bank_journal in self.search([('type', '=', 'bank')]):
            bank_journal._create_check_sequence()
            bank_journal.outbound_payment_method_ids += check_method

    def get_journal_dashboard_datas(self):
        domain_checks_to_print = [
            ('journal_id', '=', self.id),
            ('payment_method_id.code', '=', 'check_printing'),
            ('state', '=', 'posted'),
            ('is_move_sent','=', False),
        ]
        return dict(
            super(AccountJournal, self).get_journal_dashboard_datas(),
            num_checks_to_print=self.env['account.payment'].search_count(domain_checks_to_print),
        )

    def action_checks_to_print(self):
        check_method = self.env.ref('account_check_printing.account_payment_method_check')
        return {
            'name': _('Checks to Print'),
            'type': 'ir.actions.act_window',
            'view_mode': 'list,form,graph',
            'res_model': 'account.payment',
            'context': dict(
                self.env.context,
                search_default_checks_to_send=1,
                journal_id=self.id,
                default_journal_id=self.id,
                default_payment_type='outbound',
                default_payment_method_id=check_method.id,
            ),
        }

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, fields, api
from odoo.tools.sql import column_exists, create_column


class AccountMove(models.Model):
    _inherit = 'account.move'

    preferred_payment_method_id = fields.Many2one(
        string="Preferred Payment Method",
        comodel_name='account.payment.method',
        compute='_compute_preferred_payment_method_idd',
        store=True,
    )

    def _auto_init(self):
        """ Create column for `preferred_payment_method_id` to avoid having it
        computed by the ORM on installation. Since `property_payment_method_id` is
        introduced in this module, there is no need for UPDATE
        """
        if not column_exists(self.env.cr, "account_move", "preferred_payment_method_id"):
            create_column(self.env.cr, "account_move", "preferred_payment_method_id", "int4")
        return super()._auto_init()

    @api.depends('partner_id')
    def _compute_preferred_payment_method_idd(self):
        for move in self:
            partner = move.partner_id
            # take the payment method corresponding to the move's company
            move.preferred_payment_method_id = partner.with_company(move.company_id).property_payment_method_id

```

## File: models\account_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import UserError, ValidationError, RedirectWarning
from odoo.tools.misc import formatLang, format_date

INV_LINES_PER_STUB = 9


class AccountPaymentRegister(models.TransientModel):
    _inherit = "account.payment.register"

    @api.depends('payment_type', 'journal_id', 'partner_id')
    def _compute_payment_method_id(self):
        super()._compute_payment_method_id()
        for record in self:
            preferred = record.partner_id.with_company(record.company_id).property_payment_method_id
            if record.payment_type == 'outbound' and preferred in record.journal_id.outbound_payment_method_ids:
                record.payment_method_id = preferred

class AccountPayment(models.Model):
    _inherit = "account.payment"

    check_amount_in_words = fields.Char(
        string="Amount in Words",
        store=True,
        compute='_compute_check_amount_in_words',
    )
    check_manual_sequencing = fields.Boolean(related='journal_id.check_manual_sequencing')
    check_number = fields.Char(
        string="Check Number",
        store=True,
        readonly=True,
        copy=False,
        compute='_compute_check_number',
        inverse='_inverse_check_number',
        help="The selected journal is configured to print check numbers. If your pre-printed check paper already has numbers "
             "or if the current numbering is wrong, you can change it in the journal configuration page.",
    )

    @api.constrains('check_number', 'journal_id')
    def _constrains_check_number(self):
        payment_checks = self.filtered('check_number')
        if not payment_checks:
            return
        for payment_check in payment_checks:
            if not payment_check.check_number.isdecimal():
                raise ValidationError(_('Check numbers can only consist of digits'))
        self.flush()
        self.env.cr.execute("""
            SELECT payment.check_number, move.journal_id
              FROM account_payment payment
              JOIN account_move move ON move.id = payment.move_id
              JOIN account_journal journal ON journal.id = move.journal_id,
                   account_payment other_payment
              JOIN account_move other_move ON other_move.id = other_payment.move_id
             WHERE payment.check_number::BIGINT = other_payment.check_number::BIGINT
               AND move.journal_id = other_move.journal_id
               AND payment.id != other_payment.id
               AND payment.id IN %(ids)s
               AND move.state = 'posted'
               AND other_move.state = 'posted'
               AND payment.check_number IS NOT NULL
               AND other_payment.check_number IS NOT NULL
        """, {
            'ids': tuple(payment_checks.ids),
        })
        res = self.env.cr.dictfetchall()
        if res:
            raise ValidationError(_(
                'The following numbers are already used:\n%s',
                '\n'.join(_(
                    '%(number)s in journal %(journal)s',
                    number=r['check_number'],
                    journal=self.env['account.journal'].browse(r['journal_id']).display_name,
                ) for r in res)
            ))

    @api.depends('payment_method_id', 'currency_id', 'amount')
    def _compute_check_amount_in_words(self):
        for pay in self:
            if pay.currency_id:
                pay.check_amount_in_words = pay.currency_id.amount_to_text(pay.amount)
            else:
                pay.check_amount_in_words = False

    @api.depends('journal_id', 'payment_method_code')
    def _compute_check_number(self):
        for pay in self:
            if pay.journal_id.check_manual_sequencing and pay.payment_method_code == 'check_printing':
                sequence = pay.journal_id.check_sequence_id
                pay.check_number = sequence.get_next_char(sequence.number_next_actual)
            else:
                pay.check_number = False

    def _inverse_check_number(self):
        for payment in self:
            if payment.check_number:
                sequence = payment.journal_id.check_sequence_id.sudo()
                sequence.padding = len(payment.check_number)

    @api.depends('payment_type', 'journal_id', 'partner_id')
    def _compute_payment_method_id(self):
        super()._compute_payment_method_id()
        for record in self:
            preferred = record.partner_id.with_company(record.company_id).property_payment_method_id
            if record.payment_type == 'outbound' and preferred in record.journal_id.outbound_payment_method_ids:
                record.payment_method_id = preferred

    def action_post(self):
        payment_method_check = self.env.ref('account_check_printing.account_payment_method_check')
        for payment in self.filtered(lambda p: p.payment_method_id == payment_method_check and p.check_manual_sequencing):
            sequence = payment.journal_id.check_sequence_id
            payment.check_number = sequence.next_by_id()
        return super(AccountPayment, self).action_post()

    def print_checks(self):
        """ Check that the recordset is valid, set the payments state to sent and call print_checks() """
        # Since this method can be called via a client_action_multi, we need to make sure the received records are what we expect
        self = self.filtered(lambda r: r.payment_method_id.code == 'check_printing' and r.state != 'reconciled')

        if len(self) == 0:
            raise UserError(_("Payments to print as a checks must have 'Check' selected as payment method and "
                              "not have already been reconciled"))
        if any(payment.journal_id != self[0].journal_id for payment in self):
            raise UserError(_("In order to print multiple checks at once, they must belong to the same bank journal."))

        if not self[0].journal_id.check_manual_sequencing:
            # The wizard asks for the number printed on the first pre-printed check
            # so payments are attributed the number of the check the'll be printed on.
            self.env.cr.execute("""
                  SELECT payment.id
                    FROM account_payment payment
                    JOIN account_move move ON movE.id = payment.move_id
                   WHERE journal_id = %(journal_id)s
                   AND payment.check_number IS NOT NULL
                ORDER BY payment.check_number::BIGINT DESC
                   LIMIT 1
            """, {
                'journal_id': self.journal_id.id,
            })
            last_printed_check = self.browse(self.env.cr.fetchone())
            number_len = len(last_printed_check.check_number or "")
            next_check_number = '%0{}d'.format(number_len) % (int(last_printed_check.check_number) + 1)

            return {
                'name': _('Print Pre-numbered Checks'),
                'type': 'ir.actions.act_window',
                'res_model': 'print.prenumbered.checks',
                'view_mode': 'form',
                'target': 'new',
                'context': {
                    'payment_ids': self.ids,
                    'default_next_check_number': next_check_number,
                }
            }
        else:
            self.filtered(lambda r: r.state == 'draft').action_post()
            return self.do_print_checks()

    def action_unmark_sent(self):
        self.write({'is_move_sent': False})

    def action_void_check(self):
        self.action_draft()
        self.action_cancel()

    def do_print_checks(self):
        check_layout = self.company_id.account_check_printing_layout
        redirect_action = self.env.ref('account.action_account_config')
        if not check_layout or check_layout == 'disabled':
            msg = _("You have to choose a check layout. For this, go in Invoicing/Accounting Settings, search for 'Checks layout' and set one.")
            raise RedirectWarning(msg, redirect_action.id, _('Go to the configuration panel'))
        report_action = self.env.ref(check_layout, False)
        if not report_action:
            msg = _("Something went wrong with Check Layout, please select another layout in Invoicing/Accounting Settings and try again.")
            raise RedirectWarning(msg, redirect_action.id, _('Go to the configuration panel'))
        self.write({'is_move_sent': True})
        return report_action.report_action(self)

    #######################
    #CHECK PRINTING METHODS
    #######################
    def _check_fill_line(self, amount_str):
        return amount_str and (amount_str + ' ').ljust(200, '*') or ''

    def _check_build_page_info(self, i, p):
        multi_stub = self.company_id.account_check_printing_multi_stub
        return {
            'sequence_number': self.check_number,
            'manual_sequencing': self.journal_id.check_manual_sequencing,
            'date': format_date(self.env, self.date),
            'partner_id': self.partner_id,
            'partner_name': self.partner_id.name,
            'currency': self.currency_id,
            'state': self.state,
            'amount': formatLang(self.env, self.amount, currency_obj=self.currency_id) if i == 0 else 'VOID',
            'amount_in_word': self._check_fill_line(self.check_amount_in_words) if i == 0 else 'VOID',
            'memo': self.ref,
            'stub_cropped': not multi_stub and len(self.move_id._get_reconciled_invoices()) > INV_LINES_PER_STUB,
            # If the payment does not reference an invoice, there is no stub line to display
            'stub_lines': p,
        }

    def _check_get_pages(self):
        """ Returns the data structure used by the template : a list of dicts containing what to print on pages.
        """
        stub_pages = self._check_make_stub_pages() or [False]
        pages = []
        for i, p in enumerate(stub_pages):
            pages.append(self._check_build_page_info(i, p))
        return pages

    def _check_make_stub_pages(self):
        """ The stub is the summary of paid invoices. It may spill on several pages, in which case only the check on
            first page is valid. This function returns a list of stub lines per page.
        """
        self.ensure_one()

        def prepare_vals(invoice, partials):
            number = ' - '.join([invoice.name, invoice.ref] if invoice.ref else [invoice.name])

            if invoice.is_outbound() or invoice.move_type == 'entry':
                invoice_sign = 1
                partial_field = 'debit_amount_currency'
            else:
                invoice_sign = -1
                partial_field = 'credit_amount_currency'

            if invoice.currency_id.is_zero(invoice.amount_residual):
                amount_residual_str = '-'
            else:
                amount_residual_str = formatLang(self.env, invoice_sign * invoice.amount_residual, currency_obj=invoice.currency_id)

            return {
                'due_date': format_date(self.env, invoice.invoice_date_due),
                'number': number,
                'amount_total': formatLang(self.env, invoice_sign * invoice.amount_total, currency_obj=invoice.currency_id),
                'amount_residual': amount_residual_str,
                'amount_paid': formatLang(self.env, invoice_sign * sum(partials.mapped(partial_field)), currency_obj=self.currency_id),
                'currency': invoice.currency_id,
            }

        # Decode the reconciliation to keep only bills.
        term_lines = self.line_ids.filtered(lambda line: line.account_id.internal_type in ('receivable', 'payable'))
        invoices = (term_lines.matched_debit_ids.debit_move_id.move_id + term_lines.matched_credit_ids.credit_move_id.move_id) \
            .filtered(lambda move: move.is_outbound() or move.move_type == 'entry')

        invoices = invoices.sorted(lambda x: x.invoice_date_due or x.date)

        # Group partials by invoices.
        invoice_map = {invoice: self.env['account.partial.reconcile'] for invoice in invoices}
        for partial in term_lines.matched_debit_ids:
            invoice = partial.debit_move_id.move_id
            if invoice in invoice_map:
                invoice_map[invoice] |= partial
        for partial in term_lines.matched_credit_ids:
            invoice = partial.credit_move_id.move_id
            if invoice in invoice_map:
                invoice_map[invoice] |= partial

        # Prepare stub_lines.
        if 'out_refund' in invoices.mapped('move_type'):
            stub_lines = [{'header': True, 'name': "Bills"}]
            stub_lines += [prepare_vals(invoice, partials)
                           for invoice, partials in invoice_map.items()
                           if invoice.move_type == 'in_invoice']
            stub_lines += [{'header': True, 'name': "Refunds"}]
            stub_lines += [prepare_vals(invoice, partials)
                           for invoice, partials in invoice_map.items()
                           if invoice.move_type == 'out_refund']
        else:
            stub_lines = [prepare_vals(invoice, partials)
                          for invoice, partials in invoice_map.items()
                          if invoice.move_type in ('in_invoice', 'entry')]

        # Crop the stub lines or split them on multiple pages
        if not self.company_id.account_check_printing_multi_stub:
            # If we need to crop the stub, leave place for an ellipsis line
            num_stub_lines = len(stub_lines) > INV_LINES_PER_STUB and INV_LINES_PER_STUB - 1 or INV_LINES_PER_STUB
            stub_pages = [stub_lines[:num_stub_lines]]
        else:
            stub_pages = []
            i = 0
            while i < len(stub_lines):
                # Make sure we don't start the credit section at the end of a page
                if len(stub_lines) >= i + INV_LINES_PER_STUB and stub_lines[i + INV_LINES_PER_STUB - 1].get('header'):
                    num_stub_lines = INV_LINES_PER_STUB - 1 or INV_LINES_PER_STUB
                else:
                    num_stub_lines = INV_LINES_PER_STUB
                stub_pages.append(stub_lines[i:i + num_stub_lines])
                i += num_stub_lines

        return stub_pages

    def _check_make_stub_line(self, invoice):
        """ Return the dict used to display an invoice/refund in the stub
        """
        # DEPRECATED: TO BE REMOVED IN MASTER
        # Find the account.partial.reconcile which are common to the invoice and the payment
        if invoice.move_type in ['in_invoice', 'out_refund']:
            invoice_sign = 1
            invoice_payment_reconcile = invoice.line_ids.mapped('matched_debit_ids').filtered(lambda r: r.debit_move_id in self.line_ids)
        else:
            invoice_sign = -1
            invoice_payment_reconcile = invoice.line_ids.mapped('matched_credit_ids').filtered(lambda r: r.credit_move_id in self.line_ids)

        if self.currency_id != self.journal_id.company_id.currency_id:
            amount_paid = abs(sum(invoice_payment_reconcile.mapped('amount_currency')))
        else:
            amount_paid = abs(sum(invoice_payment_reconcile.mapped('amount')))

        amount_residual = invoice_sign * invoice.amount_residual

        return {
            'due_date': format_date(self.env, invoice.invoice_date_due),
            'number': invoice.ref and invoice.name + ' - ' + invoice.ref or invoice.name,
            'amount_total': formatLang(self.env, invoice_sign * invoice.amount_total, currency_obj=invoice.currency_id),
            'amount_residual': formatLang(self.env, amount_residual, currency_obj=invoice.currency_id) if amount_residual * 10**4 != 0 else '-',
            'amount_paid': formatLang(self.env, invoice_sign * amount_paid, currency_obj=self.currency_id),
            'currency': invoice.currency_id,
        }

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields


class res_company(models.Model):
    _inherit = "res.company"

    # This field needs to be overridden with `selection_add` in the modules which intends to add report layouts.
    # The xmlID of all the report actions which are actually Check Layouts has to be kept as key of the selection.
    account_check_printing_layout = fields.Selection(
        string="Check Layout",
        selection=[
            ('disabled', 'None'),
        ],
        default='disabled',
        help="Select the format corresponding to the check paper you will be printing your checks on.\n"
             "In order to disable the printing feature, select 'None'.",
    )
    account_check_printing_date_label = fields.Boolean(
        string='Print Date Label',
        default=True,
        help="This option allows you to print the date label on the check as per CPA.\n"
             "Disable this if your pre-printed check includes the date label.",
    )
    account_check_printing_multi_stub = fields.Boolean(
        string='Multi-Pages Check Stub',
        help="This option allows you to print check details (stub) on multiple pages if they don't fit on a single page.",
    )
    account_check_printing_margin_top = fields.Float(
        string='Check Top Margin',
        default=0.25,
        help="Adjust the margins of generated checks to make it fit your printer's settings.",
    )
    account_check_printing_margin_left = fields.Float(
        string='Check Left Margin',
        default=0.25,
        help="Adjust the margins of generated checks to make it fit your printer's settings.",
    )
    account_check_printing_margin_right = fields.Float(
        string='Right Margin',
        default=0.25,
        help="Adjust the margins of generated checks to make it fit your printer's settings.",
    )

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    account_check_printing_layout = fields.Selection(
        related='company_id.account_check_printing_layout',
        string="Check Layout",
        readonly=False,
        help="Select the format corresponding to the check paper you will be printing your checks on.\n"
             "In order to disable the printing feature, select 'None'."
    )
    account_check_printing_date_label = fields.Boolean(
        related='company_id.account_check_printing_date_label',
        string="Print Date Label",
        readonly=False,
        help="This option allows you to print the date label on the check as per CPA.\n"
             "Disable this if your pre-printed check includes the date label."
    )
    account_check_printing_multi_stub = fields.Boolean(
        related='company_id.account_check_printing_multi_stub',
        string='Multi-Pages Check Stub',
        readonly=False,
        help="This option allows you to print check details (stub) on multiple pages if they don't fit on a single page."
    )
    account_check_printing_margin_top = fields.Float(
        related='company_id.account_check_printing_margin_top',
        string='Check Top Margin',
        readonly=False,
        help="Adjust the margins of generated checks to make it fit your printer's settings."
    )
    account_check_printing_margin_left = fields.Float(
        related='company_id.account_check_printing_margin_left',
        string='Check Left Margin',
        readonly=False,
        help="Adjust the margins of generated checks to make it fit your printer's settings."
    )
    account_check_printing_margin_right = fields.Float(
        related='company_id.account_check_printing_margin_right',
        string='Check Right Margin',
        readonly=False,
        help="Adjust the margins of generated checks to make it fit your printer's settings."
    )

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, fields


class ResPartner(models.Model):
    _inherit = 'res.partner'

    property_payment_method_id = fields.Many2one(
        comodel_name='account.payment.method',
        string='Payment Method',
        company_dependent=True,
        domain="[('payment_type', '=', 'outbound')]",
        help="Preferred payment method when paying this vendor. This is used to filter vendor bills"
             " by preferred payment method to register payments in mass. Use cases: create bank"
             " files for batch wires, check runs.",
    )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import account_move
from . import account_payment
from . import res_company
from . import res_config_settings
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_print_prenumbered_checks","access.print.prenumbered.checks","model_print_prenumbered_checks","account.group_account_user",1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient><path id="d" d="M50.25 52.79l-3.61-3.11v-8.95H24.347v14.439H42.08l2.882 3.243h-21.43c-.985 0-1.782-.785-1.782-1.754v-13.59h-3.86a.884.884 0 0 1-.89-.877v-9.648c0-1.938 1.595-3.509 3.563-3.509h1.187v-12.28c0-.969.797-1.754 1.781-1.754h20.637c.473 0 .926.185 1.26.514l4.3 4.235c.334.329.522.775.522 1.24v8.045h1.188c1.967 0 3.562 1.571 3.562 3.509v9.648a.884.884 0 0 1-.89.877h-3.86v9.723zm-21.926-7.05c0 1.41 4.636 1.052 4.636 3.734 0 1.284-1.015 2.381-2.664 2.61v1.063c0 .148-.135.268-.301.268h-1.004c-.166 0-.3-.12-.3-.268v-1.08c-.992-.151-1.875-.56-2.478-1.075a.248.248 0 0 1-.033-.355l.762-.911a.326.326 0 0 1 .431-.048c.625.448 1.433.804 2.204.804.899 0 1.308-.478 1.308-.922 0-1.313-4.636-1.028-4.636-3.796 0-1.18.97-2.136 2.441-2.433v-1.117c0-.148.135-.268.301-.268h1.004c.166 0 .301.12.301.268v1.059c.807.083 1.681.365 2.29.847.105.082.132.219.065.328l-.59.976c-.087.143-.294.185-.442.09-.56-.36-1.248-.635-1.918-.635-.836 0-1.377.338-1.377.86zm-3.977-14.367H46.64v-8.14H43.6c-.676-.022-1.013-.354-1.013-.998v-2.923l-18.24-.07v12.131zm24.715 5.263c.984 0 1.782-.785 1.782-1.754 0-.97-.798-1.755-1.782-1.755-.983 0-1.78.786-1.78 1.755 0 .969.797 1.754 1.78 1.754zm-7.031 12.698c1.61.343 2.64.782 3.089 1.317l5.773 6.88-2.328 1.954-5.774-6.88c-.34-.404-.592-1.495-.76-3.27zm7.072 10.736l2.33-1.954 1.282 1.529c.428.51.383.982-.135 1.416l-.776.651c-.517.434-.99.397-1.418-.113l-1.283-1.529zm.494-5.272l.01-.009a.5.5 0 0 1 .704.062l2.334 2.781a.5.5 0 0 1-.061.705l-.01.008a.5.5 0 0 1-.705-.061l-2.334-2.782a.5.5 0 0 1 .062-.704zM35 44h9v.998h-9V44zm0 3h9v.998h-9V47z"/><path id="e" d="M50.25 50.79l-3.61-3.11v-8.95H24.347v14.439H42.08l2.882 3.243h-21.43c-.985 0-1.782-.785-1.782-1.754v-13.59h-3.86a.884.884 0 0 1-.89-.877v-9.648c0-1.938 1.595-3.509 3.563-3.509h1.187v-12.28c0-.969.797-1.754 1.781-1.754h20.637c.473 0 .926.185 1.26.514l4.3 4.235c.334.329.522.775.522 1.24v8.045h1.188c1.967 0 3.562 1.571 3.562 3.509v9.648a.884.884 0 0 1-.89.877h-3.86v9.723zm-21.926-7.05c0 1.41 4.636 1.052 4.636 3.734 0 1.284-1.015 2.381-2.664 2.61v1.063c0 .148-.135.268-.301.268h-1.004c-.166 0-.3-.12-.3-.268v-1.08c-.992-.151-1.875-.56-2.478-1.075a.248.248 0 0 1-.033-.355l.762-.911a.326.326 0 0 1 .431-.048c.625.448 1.433.804 2.204.804.899 0 1.308-.478 1.308-.922 0-1.313-4.636-1.028-4.636-3.796 0-1.18.97-2.136 2.441-2.433v-1.117c0-.148.135-.268.301-.268h1.004c.166 0 .301.12.301.268v1.059c.807.083 1.681.365 2.29.847.105.082.132.219.065.328l-.59.976c-.087.143-.294.185-.442.09-.56-.36-1.248-.635-1.918-.635-.836 0-1.377.338-1.377.86zm-3.977-14.367H46.64v-8.14H43.6c-.676-.022-1.013-.354-1.013-.998v-2.923l-18.24-.07v12.131zm24.715 5.263c.984 0 1.782-.785 1.782-1.754 0-.97-.798-1.755-1.782-1.755-.983 0-1.78.786-1.78 1.755 0 .969.797 1.754 1.78 1.754zm-7.031 12.698c1.61.343 2.64.782 3.089 1.317l5.773 6.88-2.328 1.954-5.774-6.88c-.34-.404-.592-1.495-.76-3.27zm7.072 10.736l2.33-1.954 1.282 1.529c.428.51.383.982-.135 1.416l-.776.651c-.517.434-.99.397-1.418-.113l-1.283-1.529zm.494-5.272l.01-.009a.5.5 0 0 1 .704.062l2.334 2.781a.5.5 0 0 1-.061.705l-.01.008a.5.5 0 0 1-.705-.061l-2.334-2.782a.5.5 0 0 1 .062-.704zM35 42h9v.998h-9V42zm0 3h9v.998h-9V45z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M42.865 69H4c-2 0-4-1-4-4V38.062l22.272-24.547L24.037 13H44.02L49 17v13h5.814v10.707l-4.599 5.878v4.163l-1.966 2.081.555.876.84-.876 2.964 3.459-.858.636L53 58.5 42.865 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\account_journal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="account_journal_dashboard_kanban_view_inherited" model="ir.ui.view">
            <field name="name">account.journal.dashboard.kanban.inherited</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.account_journal_dashboard_kanban_view" />
            <field name="arch" type="xml">
                <xpath expr="//t[@t-name='JournalBodyBankCash']//div[hasclass('o_kanban_primary_right')]" position="inside">
                    <div t-if="journal_type == 'bank' and dashboard.num_checks_to_print != 0">
                        <div class="row">
                            <div class="col-12">
                                <a type="object" name="action_checks_to_print">
                                    <t t-esc="dashboard.num_checks_to_print"/>
                                    <t t-if="dashboard.num_checks_to_print == 1">Check to print</t>
                                    <t t-if="dashboard.num_checks_to_print != 1">Checks to print</t>
                                </a>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="view_account_journal_form_inherited" model="ir.ui.view">
            <field name="name">account.journal.form.inherited</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.view_account_journal_form" />
            <field name="arch" type="xml">
                <xpath expr="//page[@name='journal_entries']//group[@name='outgoing_payment']" position="after">
                    <group string="Check Printing"
                           attrs="{'invisible': ['|', ('type', '!=', 'bank'), ('check_printing_payment_method_selected', '=', False)]}">
                        <field name="check_printing_payment_method_selected" invisible="1"/>
                        <field name="check_sequence_id" invisible="1"/>
                        <field name="check_manual_sequencing"
                        />
                        <field name="check_next_number"
                               attrs="{'invisible': [('check_manual_sequencing', '=', False)]}"/>
                    </group>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.invoice.select.account_check_printing</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//search/group/filter[@name='status']" position="after">
                <filter name="preferred_payment_method" context="{'group_by': 'preferred_payment_method_id'}" groups="account.group_account_invoice,account.group_account_readonly"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_account_payment_form_inherited" model="ir.ui.view">
            <field name="name">account.payment.form.inherited</field>
            <field name="model">account.payment</field>
            <field name="inherit_id" ref="account.view_account_payment_form" />
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_post']" position="before">
                    <button name="print_checks" class="oe_highlight" attrs="{'invisible': ['|', '|', ('payment_method_code', '!=', 'check_printing'), ('state', '!=', 'posted'), ('is_move_sent', '=', True)]}" string="Print Check" type="object"/>
                    <button name="action_unmark_sent" attrs="{'invisible': ['|', ('payment_method_code', '!=', 'check_printing'), ('is_move_sent', '=', False)]}" string="Unmark Sent" type="object"/>
                    <button name="action_void_check" attrs="{'invisible': ['|', '|', ('payment_method_code', '!=', 'check_printing'), ('state', '!=', 'posted'), ('is_move_sent', '=', False)]}" string="Void Check" type="object"/>
                </xpath>
                <xpath expr="//div[@name='amount_div']" position="after">
                    <field name="check_amount_in_words" attrs="{'invisible': [('payment_method_code', '!=', 'check_printing')]}" groups="base.group_no_one"/>
                </xpath>
                <xpath expr="//field[@name='ref']" position="after">
                    <field name="check_manual_sequencing" invisible="1"/>
                    <field name="check_number" attrs="{'invisible': ['|', ('payment_method_code', '!=', 'check_printing'), ('check_number', '=', False)]}"/>
                </xpath>
                <xpath expr="//field[@name='name']" position='before'>
                    <widget name="web_ribbon" title="Sent" attrs="{'invisible': [('is_move_sent', '!=', True)]}"/>
                </xpath>
            </field>
        </record>

        <record id="view_payment_check_printing_search" model="ir.ui.view">
            <field name="name">account.payment.check.printing.search</field>
            <field name="model">account.payment</field>
            <field name="inherit_id" ref="account.view_account_payment_search"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='activities_overdue']" position="before">
                    <separator/>
                    <filter name="checks_to_send" string="Checks to Print" domain="[('payment_method_id.code', '=', 'check_printing'), ('state', '=', 'posted'), ('is_move_sent', '=', False)]"/>
                    <separator/>
                </xpath>
            </field>
        </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.account.check.printing</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="print_bills_payment" position="after">
                <div class="content-group">
                    <div class="row mt16">
                        <label for="account_check_printing_layout" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_layout" required="True"/>
                    </div>
                    <div class="row">
                        <label for="account_check_printing_multi_stub" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_multi_stub"/>
                    </div>
                    <div class="row">
                        <label for="account_check_printing_margin_top" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_margin_top"/>
                    </div>
                    <div class="row">
                        <label for="account_check_printing_margin_left" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_margin_left"/>
                    </div>
                    <div class="row" attrs="{'invisible': [('country_code', '!=', 'CA')]}">
                        <label for="account_check_printing_margin_right" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_margin_right"/>
                    </div>
                    <div class="row" attrs="{'invisible': [('country_code', '!=', 'CA')]}">
                      <label for="account_check_printing_date_label" class="col-lg-4 o_light_label"/>
                      <field name="account_check_printing_date_label"/>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_property_form" model="ir.ui.view">
        <field name="name">res.partner.property.form.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <field name="property_supplier_payment_term_id" position="after">
                <field name="property_payment_method_id" widget="selection" groups="account.group_account_invoice,account.group_account_readonly"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: wizard\print_prenumbered_checks.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class PrintPreNumberedChecks(models.TransientModel):
    _name = 'print.prenumbered.checks'
    _description = 'Print Pre-numbered Checks'

    next_check_number = fields.Char('Next Check Number', required=True)

    @api.constrains('next_check_number')
    def _check_next_check_number(self):
        for check in self:
            if check.next_check_number and not re.match(r'^[0-9]+$', check.next_check_number):
                raise ValidationError(_('Next Check Number should only contains numbers.'))

    def print_checks(self):
        check_number = int(self.next_check_number)
        number_len = len(self.next_check_number or "")
        payments = self.env['account.payment'].browse(self.env.context['payment_ids'])
        payments.filtered(lambda r: r.state == 'draft').action_post()
        payments.filtered(lambda r: r.state == 'posted' and not r.is_move_sent).write({'is_move_sent': True})
        for payment in payments:
            payment.check_number = '%0{}d'.format(number_len) % check_number
            check_number += 1
        return payments.do_print_checks()

```

## File: wizard\print_prenumbered_checks_views.xml

```xml
<?xml version="1.0" ?>
<odoo>

        <record id="print_pre_numbered_checks_view" model="ir.ui.view">
            <field name="name">Print Pre-numbered Checks</field>
            <field name="model">print.prenumbered.checks</field>
            <field name="arch" type="xml">
                <form string="Print Pre-numbered Checks">
                    <p>Please enter the number of the first pre-printed check that you are about to print on.</p>
                    <p>This will allow to save on payments the number of the corresponding check.</p>
                    <group>
                        <field name="next_check_number"/>
                    </group>
                    <footer>
                        <button name="print_checks" string="Print" type="object" class="oe_highlight"/>
                        <button string="Cancel" class="btn btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import print_prenumbered_checks

```

