# Odoo Module: account_check_printing

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import wizard


def create_check_sequence_on_bank_journals(env):
    env['account.journal'].search([('type', '=', 'bank')])._create_check_sequence()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Check Printing Base',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'summary': 'Check printing basic features',
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
    'post_init_hook': 'create_check_sequence_on_bank_journals',
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
        </record>

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

    def _default_outbound_payment_methods(self):
        res = super()._default_outbound_payment_methods()
        if self._is_payment_method_available('check_printing'):
            res |= self.env.ref('account_check_printing.account_payment_method_check')
        return res

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

    @api.model_create_multi
    def create(self, vals_list):
        journals = super().create(vals_list)
        journals.filtered(lambda j: not j.check_sequence_id)._create_check_sequence()
        return journals

    def _create_check_sequence(self):
        """ Create a check sequence for the journal """
        for journal in self:
            journal.check_sequence_id = self.env['ir.sequence'].sudo().create({
                'name': journal.name + _(": Check Number Sequence"),
                'implementation': 'no_gap',
                'padding': 5,
                'number_increment': 1,
                'company_id': journal.company_id.id,
            })

    def _get_journal_dashboard_data_batched(self):
        dashboard_data = super()._get_journal_dashboard_data_batched()
        self._fill_dashboard_data_count(dashboard_data, 'account.payment', 'num_checks_to_print', [
            ('payment_method_line_id.code', '=', 'check_printing'),
            ('state', '=', 'posted'),
            ('is_move_sent','=', False),
        ])
        return dashboard_data

    def action_checks_to_print(self):
        payment_method_line_id = self.outbound_payment_method_line_ids.filtered(lambda l: l.code == 'check_printing')[:1].id
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
                default_payment_method_line_id=payment_method_line_id,
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
from odoo.tools.sql import column_exists, create_column

INV_LINES_PER_STUB = 9


class AccountPaymentRegister(models.TransientModel):
    _inherit = "account.payment.register"

    @api.depends('payment_type', 'journal_id', 'partner_id')
    def _compute_payment_method_line_id(self):
        super()._compute_payment_method_line_id()
        for record in self:
            preferred = record.partner_id.with_company(record.company_id).property_payment_method_id
            method_line = record.journal_id.outbound_payment_method_line_ids.filtered(
                lambda l: l.payment_method_id == preferred
            )
            if record.payment_type == 'outbound' and method_line:
                record.payment_method_line_id = method_line[0]


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
    payment_method_line_id = fields.Many2one(index=True)
    show_check_number = fields.Boolean(compute='_compute_show_check_number')

    @api.depends('payment_method_line_id.code', 'check_number')
    def _compute_show_check_number(self):
        for payment in self:
            payment.show_check_number = (
                payment.payment_method_line_id.code == 'check_printing'
                and payment.check_number
            )

    @api.constrains('check_number')
    def _constrains_check_number(self):
        for payment_check in self.filtered('check_number'):
            if not payment_check.check_number.isdecimal():
                raise ValidationError(_('Check numbers can only consist of digits'))

    def _auto_init(self):
        """
        Create compute stored field check_number
        here to avoid MemoryError on large databases.
        """
        if not column_exists(self.env.cr, 'account_payment', 'check_number'):
            create_column(self.env.cr, 'account_payment', 'check_number', 'varchar')

        return super()._auto_init()

    @api.constrains('check_number', 'journal_id')
    def _constrains_check_number_unique(self):
        payment_checks = self.filtered('check_number')
        if not payment_checks:
            return
        self.env.flush_all()
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

    @api.depends('payment_method_line_id', 'currency_id', 'amount')
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
    def _compute_payment_method_line_id(self):
        super()._compute_payment_method_line_id()
        for record in self:
            preferred = record.partner_id.with_company(record.company_id).property_payment_method_id
            method_line = record.journal_id.outbound_payment_method_line_ids\
                .filtered(lambda l: l.payment_method_id == preferred)
            if record.payment_type == 'outbound' and method_line:
                record.payment_method_line_id = method_line[0]

    def _get_aml_default_display_name_list(self):
        # Extends 'account'
        values = super()._get_aml_default_display_name_list()
        if self.check_number:
            date_index = [i for i, value in enumerate(values) if value[0] == 'date'][0]
            values.insert(date_index - 1, ('check_number', self.check_number))
            values.insert(date_index - 1, ('sep', ' - '))
        return values

    def action_post(self):
        payment_method_check = self.env.ref('account_check_printing.account_payment_method_check')
        for payment in self.filtered(lambda p: p.payment_method_id == payment_method_check and p.check_manual_sequencing):
            sequence = payment.journal_id.check_sequence_id
            payment.check_number = sequence.next_by_id()
        return super(AccountPayment, self).action_post()

    def print_checks(self):
        """ Check that the recordset is valid, set the payments state to sent and call print_checks() """
        # Since this method can be called via a client_action_multi, we need to make sure the received records are what we expect
        self = self.filtered(lambda r: r.payment_method_line_id.code == 'check_printing' and r.state != 'reconciled')

        if len(self) == 0:
            raise UserError(_("Payments to print as a checks must have 'Check' selected as payment method and "
                              "not have already been reconciled"))
        if any(payment.journal_id != self[0].journal_id for payment in self):
            raise UserError(_("In order to print multiple checks at once, they must belong to the same bank journal."))

        if not self[0].journal_id.check_manual_sequencing:
            # The wizard asks for the number printed on the first pre-printed check
            # so payments are attributed the number of the check the'll be printed on.
            self.env.cr.execute("""
                  SELECT payment.check_number
                    FROM account_payment payment
                    JOIN account_move move ON move.id = payment.move_id
                   WHERE move.journal_id = %(journal_id)s
                   AND payment.check_number IS NOT NULL
                ORDER BY payment.check_number::BIGINT DESC
                   LIMIT 1
            """, {
                'journal_id': self.journal_id.id,
            })
            last_check_number = (self.env.cr.fetchone() or (False,))[0]
            number_len = len(last_check_number or "")
            next_check_number = f'{int(last_check_number) + 1:0{number_len}}'

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
        """ Returns the data structure used by the template: a list of dicts containing what to print on pages.
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

            if invoice.is_outbound() or invoice.move_type == 'in_receipt':
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

        # Decode the reconciliation to keep only invoices.
        term_lines = self.line_ids.filtered(lambda line: line.account_id.account_type in ('asset_receivable', 'liability_payable'))
        invoices = (term_lines.matched_debit_ids.debit_move_id.move_id + term_lines.matched_credit_ids.credit_move_id.move_id)\
            .filtered(lambda x: x.is_outbound() or x.move_type == 'in_receipt')
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
                          if invoice.move_type in ('in_invoice', 'in_receipt')]

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
        res['check_printing'] = {'mode': 'multi', 'domain': [('type', '=', 'bank')]}
        return res

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
from . import account_payment_method
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M4 17a4 4 0 0 1 4-4h38v20a4 4 0 0 1-4 4H4V17Z" fill="#FC868B"/><path d="M8 8a4 4 0 0 1 4-4h26a4 4 0 0 1 4 4v5H8V8Z" fill="#985184"/><path d="M13 37h24v5a4 4 0 0 1-4 4H13v-9Z" fill="#FBB945"/><path d="M37 37H13v-8h24v8Z" fill="#953B24"/></svg>

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
                                    <t t-out="dashboard.num_checks_to_print"/>
                                    <span>&amp;nbsp;</span>
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
                <xpath expr="//page[@id='outbound_payment_settings']//group[@name='outgoing_payment']" position="inside">
                    <group string="Check Printing"
                           invisible="',check_printing,'.lower() not in (selected_payment_method_codes or '').lower() or type != 'bank'">
                        <field name="check_sequence_id" invisible="1"/>
                        <field name="check_manual_sequencing"
                        />
                        <field name="check_next_number"
                               invisible="not check_manual_sequencing"/>
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
                    <button name="print_checks" class="oe_highlight" invisible="payment_method_code != 'check_printing' or state != 'posted' or is_move_sent" string="Print Check" type="object" data-hotkey="g" />
                    <button name="action_unmark_sent" invisible="payment_method_code != 'check_printing' or not is_move_sent" string="Unmark Sent" type="object" data-hotkey="l" />
                    <button name="action_void_check" invisible="payment_method_code != 'check_printing' or state != 'posted' or not is_move_sent" string="Void Check" type="object" data-hotkey="o" />
                </xpath>
                <xpath expr="//div[@name='amount_div']" position="after">
                    <field name="check_amount_in_words" invisible="payment_method_code != 'check_printing'" groups="base.group_no_one"/>
                </xpath>
                <xpath expr="//field[@name='payment_method_line_id']" position="after">
                    <field name="check_manual_sequencing" invisible="1"/>
                    <field name="show_check_number" invisible="1"/>
                    <field name="check_number" invisible="not show_check_number"/>
                </xpath>
                <xpath expr="//field[@name='name']" position='before'>
                    <widget name="web_ribbon" title="Sent" invisible="not is_move_sent"/>
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
                    <filter name="checks_to_send" string="Checks to Print" domain="[('payment_method_line_id.code', '=', 'check_printing'), ('state', '=', 'posted'), ('is_move_sent', '=', False)]"/>
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
            <setting id="print_checks" position="inside">
                <div class="content-group" invisible="not module_account_check_printing">
                    <div class="row mt16">
                        <label for="account_check_printing_layout" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_layout" required="True"/>
                    </div>
                    <div class="row">
                        <label for="account_check_printing_multi_stub" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_multi_stub" class="w-50"/>
                    </div>
                    <div class="row">
                        <label for="account_check_printing_margin_top" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_margin_top"/>
                    </div>
                    <div class="row">
                        <label for="account_check_printing_margin_left" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_margin_left"/>
                    </div>
                    <div class="row" invisible="country_code != 'CA'">
                        <label for="account_check_printing_margin_right" class="col-lg-4 o_light_label"/>
                        <field name="account_check_printing_margin_right"/>
                    </div>
                    <div class="row" invisible="country_code != 'CA'">
                      <label for="account_check_printing_date_label" class="col-lg-4 o_light_label"/>
                      <field name="account_check_printing_date_label"/>
                    </div>
                </div>
            </setting>
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
                <field name="property_payment_method_id" options="{'no_open': True, 'no_create': True}" groups="account.group_account_invoice,account.group_account_readonly"/>
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
        checks_action = payments.do_print_checks()
        checks_action.update({'close_on_report_download': True})
        return checks_action

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
                        <button name="print_checks" string="Print" type="object" class="oe_highlight" data-hotkey="q"/>
                        <button string="Cancel" class="btn btn-secondary" special="cancel" data-hotkey="x"/>
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

