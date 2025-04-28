# Odoo Module: l10n_latam_invoice_document

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import wizards
from . import report

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "LATAM Document",
    "version": "1.0",
    "author": "ADHOC SA",
    'category': 'Accounting/Localizations',
    "summary": "LATAM Document Types",
    'description': """
Functional
----------

In some Latinamerica countries, including Argentina and Chile, some accounting transactions like invoices and vendor bills are classified by a document types defined by the government fiscal authorities (In Argentina case AFIP, Chile case SII).

This module is intended to be extended by localizations in order to manage these document types and is an essential information that needs to be displayed in the printed reports and that needs to be easily identified, within the set of invoices as well of account moves.

Each document type have their own rules and sequence number, this last one is integrated with the invoice number and journal sequence in order to be easy for the localization user. In order to support or not this document types a Journal has a new option that lets to use document or not.

Technical
---------

If your localization needs this logic will then need to add this module as dependency and in your localization module extend:

* extend company's _localization_use_documents() method.
* create the data of the document types that exists for the specific country. The document type has a country field

""",
    "depends": [
        "account",
        "account_debit_note",
    ],
    "data": [
        'views/account_journal_view.xml',
        'views/account_move_line_view.xml',
        'views/account_move_view.xml',
        'views/l10n_latam_document_type_view.xml',
        'views/ir_sequence_view.xml',
        'views/report_templates.xml',
        'report/invoice_report_view.xml',
        'wizards/account_move_reversal_view.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, api, fields, _


class AccountChartTemplate(models.Model):

    _inherit = 'account.chart.template'

    @api.model
    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        """ We add use_documents or not depending on the context"""
        journal_data = super()._prepare_all_journals(acc_template_ref, company, journals_dict)

        # if chart has localization, then we use documents by default
        if company._localization_use_documents():
            for vals_journal in journal_data:
                if vals_journal['type'] in ['sale', 'purchase']:
                    vals_journal['l10n_latam_use_documents'] = True
        return journal_data

```

## File: models\account_journal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError

class AccountJournal(models.Model):

    _inherit = "account.journal"

    l10n_latam_use_documents = fields.Boolean(
        'Use Documents?', help="If active: will be using for legal invoicing (invoices, debit/credit notes)."
        " If not set means that will be used to register accounting entries not related to invoicing legal documents."
        " For Example: Receipts, Tax Payments, Register journal entries")
    l10n_latam_company_use_documents = fields.Boolean(compute='_compute_l10n_latam_company_use_documents')

    @api.depends('company_id')
    def _compute_l10n_latam_company_use_documents(self):
        for rec in self:
            rec.l10n_latam_company_use_documents = rec.company_id._localization_use_documents()

    @api.onchange('company_id', 'type')
    def _onchange_company(self):
        self.l10n_latam_use_documents = self.type in ['sale', 'purchase'] and \
            self.l10n_latam_company_use_documents

    @api.constrains('l10n_latam_use_documents')
    def check_use_document(self):
        for rec in self:
            if rec.env['account.move'].search([('journal_id', '=', rec.id), ('posted_before', '=', True)], limit=1):
                raise ValidationError(_(
                    'You can not modify the field "Use Documents?" if there are validated invoices in this journal!'))

    @api.onchange('type', 'l10n_latam_use_documents')
    def _onchange_type(self):
        res = super()._onchange_type()
        if self.l10n_latam_use_documents:
            self.refund_sequence = False
        return res

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import UserError, ValidationError
import re
from odoo.tools.misc import formatLang
from odoo.tools.sql import column_exists, create_column


class AccountMove(models.Model):

    _inherit = "account.move"

    def _auto_init(self):
        # Skip the computation of the field `l10n_latam_document_type_id` at the module installation
        # Without this, at the module installation,
        # it would call `_compute_l10n_latam_document_type` on all existing records
        # which can take quite a while if you already have a lot of moves. It can even fail with a MemoryError.
        # In addition, it sets `_compute_l10n_latam_document_type = False` on all records
        # because this field depends on the many2many `l10n_latam_available_document_type_ids`,
        # which relies on having records for the model `l10n_latam.document.type`
        # which only happens once the according localization module is loaded.
        # The localization module is loaded afterwards, because the localization module depends on this module,
        # (e.g. `l10n_cl` depends on `l10n_latam_invoice_document`, and therefore `l10n_cl` is loaded after)
        # and therefore there are no records for the model `l10n_latam.document.type` at the time this fields
        # gets computed on installation. Hence, all records' `_compute_l10n_latam_document_type` are set to `False`.
        # In addition, multiple localization module depends on this module (e.g. `l10n_cl`, `l10n_ar`)
        # So, imagine `l10n_cl` gets installed first, and then `l10n_ar` is installed next,
        # if `l10n_latam_document_type_id` needed to be computed on install,
        # the install of `l10n_cl` would call the compute method,
        # because `l10n_latam_invoice_document` would be installed at the same time,
        # but then `l10n_ar` would miss it, because `l10n_latam_invoice_document` would already be installed.
        # Besides, this field is computed only for drafts invoices, as stated in the compute method:
        # `for rec in self.filtered(lambda x: x.state == 'draft'):`
        # So, if we want this field to be computed on install, it must be done only on draft invoices, and only once
        # the localization modules are loaded.
        # It should be done in a dedicated post init hook,
        # filtering correctly the invoices for which it must be computed.
        # Though I don't think this is needed.
        # In practical, it's very rare to already have invoices (draft, in addition)
        # for a Chilian or Argentian company (`res.company`) before installing `l10n_cl` or `l10n_ar`.
        if not column_exists(self.env.cr, "account_move", "l10n_latam_document_type_id"):
            create_column(self.env.cr, "account_move", "l10n_latam_document_type_id", "int4")
        return super()._auto_init()

    l10n_latam_amount_untaxed = fields.Monetary(compute='_compute_l10n_latam_amount_and_taxes')
    l10n_latam_tax_ids = fields.One2many(compute="_compute_l10n_latam_amount_and_taxes", comodel_name='account.move.line')
    l10n_latam_available_document_type_ids = fields.Many2many('l10n_latam.document.type', compute='_compute_l10n_latam_available_document_types')
    l10n_latam_document_type_id = fields.Many2one(
        'l10n_latam.document.type', string='Document Type', readonly=False, auto_join=True, index=True,
        states={'posted': [('readonly', True)]}, compute='_compute_l10n_latam_document_type', store=True)
    l10n_latam_document_number = fields.Char(
        compute='_compute_l10n_latam_document_number', inverse='_inverse_l10n_latam_document_number',
        string='Document Number', readonly=True, states={'draft': [('readonly', False)]})
    l10n_latam_use_documents = fields.Boolean(related='journal_id.l10n_latam_use_documents')
    l10n_latam_manual_document_number = fields.Boolean(compute='_compute_l10n_latam_manual_document_number', string='Manual Number')

    @api.depends('l10n_latam_document_type_id')
    def _compute_name(self):
        """ Change the way that the use_document moves name is computed:

        * If move use document but does not have document type selected then name = '/' to do not show the name.
        * If move use document and are numbered manually do not compute name at all (will be set manually)
        * If move use document and is in draft state and has not been posted before we restart name to '/' (this is
           when we change the document type) """
        without_doc_type = self.filtered(lambda x: x.journal_id.l10n_latam_use_documents and not x.l10n_latam_document_type_id)
        manual_documents = self.filtered(lambda x: x.journal_id.l10n_latam_use_documents and x.l10n_latam_manual_document_number)
        (without_doc_type + manual_documents.filtered(lambda x: not x.name or x.name and x.state == 'draft' and not x.posted_before)).name = '/'
        # if we change document or journal and we are in draft and not posted, we clean number so that is recomputed in super
        self.filtered(
            lambda x: x.journal_id.l10n_latam_use_documents and x.l10n_latam_document_type_id
            and not x.l10n_latam_manual_document_number and x.state == 'draft' and not x.posted_before).name = '/'
        super(AccountMove, self - without_doc_type - manual_documents)._compute_name()

    @api.depends('l10n_latam_document_type_id', 'journal_id')
    def _compute_l10n_latam_manual_document_number(self):
        """ Indicates if this document type uses a sequence or if the numbering is made manually """
        recs_with_journal_id = self.filtered(lambda x: x.journal_id and x.journal_id.l10n_latam_use_documents)
        for rec in recs_with_journal_id:
            rec.l10n_latam_manual_document_number = self._is_manual_document_number(rec.journal_id)
        remaining = self - recs_with_journal_id
        remaining.l10n_latam_manual_document_number = False

    def _is_manual_document_number(self, journal):
        return True if journal.type == 'purchase' else False

    @api.depends('name')
    def _compute_l10n_latam_document_number(self):
        recs_with_name = self.filtered(lambda x: x.name != '/')
        for rec in recs_with_name:
            name = rec.name
            doc_code_prefix = rec.l10n_latam_document_type_id.doc_code_prefix
            if doc_code_prefix and name:
                name = name.split(" ", 1)[-1]
            rec.l10n_latam_document_number = name
        remaining = self - recs_with_name
        remaining.l10n_latam_document_number = False

    @api.onchange('l10n_latam_document_type_id', 'l10n_latam_document_number')
    def _inverse_l10n_latam_document_number(self):
        for rec in self.filtered(lambda x: x.l10n_latam_document_type_id):
            if not rec.l10n_latam_document_number:
                rec.name = '/'
            else:
                l10n_latam_document_number = rec.l10n_latam_document_type_id._format_document_number(rec.l10n_latam_document_number)
                if rec.l10n_latam_document_number != l10n_latam_document_number:
                    rec.l10n_latam_document_number = l10n_latam_document_number
                rec.name = "%s %s" % (rec.l10n_latam_document_type_id.doc_code_prefix, l10n_latam_document_number)

    @api.depends('journal_id', 'l10n_latam_document_type_id')
    def _compute_highest_name(self):
        manual_records = self.filtered('l10n_latam_manual_document_number')
        manual_records.highest_name = ''
        super(AccountMove, self - manual_records)._compute_highest_name()

    @api.model
    def _deduce_sequence_number_reset(self, name):
        if self.l10n_latam_use_documents:
            return 'never'
        return super(AccountMove, self)._deduce_sequence_number_reset(name)

    def _get_starting_sequence(self):
        if self.journal_id.l10n_latam_use_documents:
            if self.l10n_latam_document_type_id:
                return "%s 00000000" % (self.l10n_latam_document_type_id.doc_code_prefix)
            # There was no pattern found, propose one
            return ""

        return super(AccountMove, self)._get_starting_sequence()

    def _compute_l10n_latam_amount_and_taxes(self):
        recs_invoice = self.filtered(lambda x: x.is_invoice())
        for invoice in recs_invoice:
            tax_lines = invoice.line_ids.filtered('tax_line_id')
            currencies = invoice.line_ids.filtered(lambda x: x.currency_id == invoice.currency_id).mapped('currency_id')
            included_taxes = invoice.l10n_latam_document_type_id and \
                invoice.l10n_latam_document_type_id._filter_taxes_included(tax_lines.mapped('tax_line_id'))
            if not included_taxes:
                l10n_latam_amount_untaxed = invoice.amount_untaxed
                not_included_invoice_taxes = tax_lines
            else:
                included_invoice_taxes = tax_lines.filtered(lambda x: x.tax_line_id in included_taxes)
                not_included_invoice_taxes = tax_lines - included_invoice_taxes
                if invoice.is_inbound():
                    sign = -1
                else:
                    sign = 1
                amount = 'amount_currency' if len(currencies) == 1 else 'balance'
                l10n_latam_amount_untaxed = invoice.amount_untaxed + sign * sum(included_invoice_taxes.mapped(amount))
            invoice.l10n_latam_amount_untaxed = l10n_latam_amount_untaxed
            invoice.l10n_latam_tax_ids = not_included_invoice_taxes
        remaining = self - recs_invoice
        remaining.l10n_latam_amount_untaxed = False
        remaining.l10n_latam_tax_ids = [(5, 0)]

    def _post(self, soft=True):
        for rec in self.filtered(lambda x: x.l10n_latam_use_documents and (not x.name or x.name == '/')):
            if rec.move_type in ('in_receipt', 'out_receipt'):
                raise UserError(_('We do not accept the usage of document types on receipts yet. '))
        return super()._post(soft)

    @api.constrains('name', 'journal_id', 'state')
    def _check_unique_sequence_number(self):
        """ This uniqueness verification is only valid for customer invoices, and vendor bills that does not use
        documents. A new constraint method _check_unique_vendor_number has been created just for validate for this purpose """
        vendor = self.filtered(lambda x: x.is_purchase_document() and x.l10n_latam_use_documents)
        return super(AccountMove, self - vendor)._check_unique_sequence_number()

    @api.constrains('state', 'l10n_latam_document_type_id')
    def _check_l10n_latam_documents(self):
        """ This constraint checks that if a invoice is posted and does not have a document type configured will raise
        an error. This only applies to invoices related to journals that has the "Use Documents" set as True.
        And if the document type is set then check if the invoice number has been set, because a posted invoice
        without a document number is not valid in the case that the related journals has "Use Docuemnts" set as True """
        validated_invoices = self.filtered(lambda x: x.l10n_latam_use_documents and x.state == 'posted')
        without_doc_type = validated_invoices.filtered(lambda x: not x.l10n_latam_document_type_id)
        if without_doc_type:
            raise ValidationError(_(
                'The journal require a document type but not document type has been selected on invoices %s.',
                without_doc_type.ids
            ))
        without_number = validated_invoices.filtered(
            lambda x: not x.l10n_latam_document_number and x.l10n_latam_manual_document_number)
        if without_number:
            raise ValidationError(_(
                'Please set the document number on the following invoices %s.',
                without_number.ids
            ))

    @api.constrains('move_type', 'l10n_latam_document_type_id')
    def _check_invoice_type_document_type(self):
        for rec in self.filtered('l10n_latam_document_type_id.internal_type'):
            internal_type = rec.l10n_latam_document_type_id.internal_type
            invoice_type = rec.move_type
            if internal_type in ['debit_note', 'invoice'] and invoice_type in ['out_refund', 'in_refund']:
                raise ValidationError(_('You can not use a %s document type with a refund invoice', internal_type))
            elif internal_type == 'credit_note' and invoice_type in ['out_invoice', 'in_invoice']:
                raise ValidationError(_('You can not use a %s document type with a invoice', internal_type))

    def _get_l10n_latam_documents_domain(self):
        self.ensure_one()
        if self.move_type in ['out_refund', 'in_refund']:
            internal_types = ['credit_note']
        else:
            internal_types = ['invoice', 'debit_note']
        return [('internal_type', 'in', internal_types), ('country_id', '=', self.company_id.country_id.id)]

    @api.depends('journal_id', 'partner_id', 'company_id', 'move_type')
    def _compute_l10n_latam_available_document_types(self):
        self.l10n_latam_available_document_type_ids = False
        for rec in self.filtered(lambda x: x.journal_id and x.l10n_latam_use_documents and x.partner_id):
            rec.l10n_latam_available_document_type_ids = self.env['l10n_latam.document.type'].search(rec._get_l10n_latam_documents_domain())

    @api.depends('l10n_latam_available_document_type_ids', 'debit_origin_id')
    def _compute_l10n_latam_document_type(self):
        debit_note = self.debit_origin_id
        for rec in self.filtered(lambda x: x.state == 'draft'):
            document_types = rec.l10n_latam_available_document_type_ids._origin
            document_types = debit_note and document_types.filtered(lambda x: x.internal_type == 'debit_note') or document_types
            rec.l10n_latam_document_type_id = document_types and document_types[0].id

    def _compute_invoice_taxes_by_group(self):
        report_or_portal_view = 'commit_assetsbundle' in self.env.context or \
            not self.env.context.get('params', {}).get('view_type') == 'form'
        if not report_or_portal_view:
            return super()._compute_invoice_taxes_by_group()

        move_with_doc_type = self.filtered('l10n_latam_document_type_id')
        for move in move_with_doc_type:
            lang_env = move.with_context(lang=move.partner_id.lang).env
            tax_lines = move.l10n_latam_tax_ids
            tax_balance_multiplicator = -1 if move.is_inbound(True) else 1
            res = {}
            # There are as many tax line as there are repartition lines
            done_taxes = set()
            for line in tax_lines:
                res.setdefault(line.tax_line_id.tax_group_id, {'base': 0.0, 'amount': 0.0})
                res[line.tax_line_id.tax_group_id]['amount'] += tax_balance_multiplicator * (line.amount_currency if line.currency_id else line.balance)
                tax_key_add_base = tuple(move._get_tax_key_for_group_add_base(line))
                if tax_key_add_base not in done_taxes:
                    if line.currency_id and line.company_currency_id and line.currency_id != line.company_currency_id:
                        amount = line.company_currency_id._convert(line.tax_base_amount, line.currency_id, line.company_id, line.date or fields.Date.today())
                    else:
                        amount = line.tax_base_amount
                    res[line.tax_line_id.tax_group_id]['base'] += amount
                    # The base should be added ONCE
                    done_taxes.add(tax_key_add_base)

            # At this point we only want to keep the taxes with a zero amount since they do not
            # generate a tax line.
            zero_taxes = set()
            for line in move.line_ids:
                for tax in line.l10n_latam_tax_ids.flatten_taxes_hierarchy():
                    if tax.tax_group_id not in res or tax.id in zero_taxes:
                        res.setdefault(tax.tax_group_id, {'base': 0.0, 'amount': 0.0})
                        res[tax.tax_group_id]['base'] += tax_balance_multiplicator * (line.amount_currency if line.currency_id else line.balance)
                        zero_taxes.add(tax.id)

            res = sorted(res.items(), key=lambda l: l[0].sequence)
            move.amount_by_group = [(
                group.name, amounts['amount'],
                amounts['base'],
                formatLang(lang_env, amounts['amount'], currency_obj=move.currency_id),
                formatLang(lang_env, amounts['base'], currency_obj=move.currency_id),
                len(res),
                group.id
            ) for group, amounts in res]
        super(AccountMove, self - move_with_doc_type)._compute_invoice_taxes_by_group()

    @api.constrains('name', 'partner_id', 'company_id', 'posted_before')
    def _check_unique_vendor_number(self):
        """ The constraint _check_unique_sequence_number is valid for customer bills but not valid for us on vendor
        bills because the uniqueness must be per partner """
        for rec in self.filtered(
                lambda x: x.name and x.name != '/' and x.is_purchase_document() and x.l10n_latam_use_documents
                            and x.commercial_partner_id):
            domain = [
                ('move_type', '=', rec.move_type),
                # by validating name we validate l10n_latam_document_type_id
                ('name', '=', rec.name),
                ('company_id', '=', rec.company_id.id),
                ('id', '!=', rec.id),
                ('commercial_partner_id', '=', rec.commercial_partner_id.id),
                # allow to have to equal if they are cancelled
                ('state', '!=', 'cancel'),
            ]
            if rec.search(domain):
                raise ValidationError(_('Vendor bill number must be unique per vendor and company.'))

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, fields
from odoo.tools.sql import column_exists, create_column


class AccountMoveLine(models.Model):

    _inherit = 'account.move.line'

    def _auto_init(self):
        # Skip the computation of the field `l10n_latam_document_type_id` at the module installation
        # See `_auto_init` in `l10n_latam_invoice_document/models/account_move.py` for more information
        if not column_exists(self.env.cr, "account_move_line", "l10n_latam_document_type_id"):
            create_column(self.env.cr, "account_move_line", "l10n_latam_document_type_id", "int4")
        return super()._auto_init()

    l10n_latam_document_type_id = fields.Many2one(
        related='move_id.l10n_latam_document_type_id', auto_join=True, store=True, index=True)
    l10n_latam_price_unit = fields.Float(compute='compute_l10n_latam_prices_and_taxes', digits='Product Price')
    l10n_latam_price_subtotal = fields.Monetary(compute='compute_l10n_latam_prices_and_taxes')
    l10n_latam_price_net = fields.Float(compute='compute_l10n_latam_prices_and_taxes', digits='Product Price')
    l10n_latam_tax_ids = fields.One2many(compute="compute_l10n_latam_prices_and_taxes", comodel_name='account.tax')

    @api.depends('price_unit', 'price_subtotal', 'move_id.l10n_latam_document_type_id')
    def compute_l10n_latam_prices_and_taxes(self):
        for line in self:
            invoice = line.move_id
            included_taxes = \
                invoice.l10n_latam_document_type_id and invoice.l10n_latam_document_type_id._filter_taxes_included(
                    line.tax_ids)
            # For the unit price, we need the number rounded based on the product price precision.
            # The method compute_all uses the accuracy of the currency so, we multiply and divide for 10^(decimal accuracy of product price) to get the price correctly rounded.
            price_digits = 10**self.env['decimal.precision'].precision_get('Product Price')
            if not included_taxes:
                price_unit = line.tax_ids.with_context(round=False, force_sign=invoice._get_tax_force_sign()).compute_all(
                    line.price_unit * price_digits, invoice.currency_id, 1.0, line.product_id, invoice.partner_id)
                l10n_latam_price_unit = price_unit['total_excluded'] / price_digits
                l10n_latam_price_subtotal = line.price_subtotal
                not_included_taxes = line.tax_ids
                l10n_latam_price_net = l10n_latam_price_unit * (1 - (line.discount or 0.0) / 100.0)
            else:
                not_included_taxes = line.tax_ids - included_taxes
                l10n_latam_price_unit = included_taxes.with_context(force_sign=invoice._get_tax_force_sign()).compute_all(
                    line.price_unit * price_digits, invoice.currency_id, 1.0, line.product_id, invoice.partner_id)['total_included'] / price_digits
                l10n_latam_price_net = l10n_latam_price_unit * (1 - (line.discount or 0.0) / 100.0)
                price = line.price_unit * (1 - (line.discount or 0.0) / 100.0)
                l10n_latam_price_subtotal = included_taxes.with_context(force_sign=invoice._get_tax_force_sign()).compute_all(
                    price, invoice.currency_id, line.quantity, line.product_id,
                    invoice.partner_id)['total_included']

            line.l10n_latam_price_subtotal = l10n_latam_price_subtotal
            line.l10n_latam_price_unit = l10n_latam_price_unit
            line.l10n_latam_price_net = l10n_latam_price_net
            line.l10n_latam_tax_ids = not_included_taxes

```

## File: models\ir_sequence.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class IrSequence(models.Model):

    _inherit = 'ir.sequence'

    l10n_latam_document_type_id = fields.Many2one('l10n_latam.document.type', 'Document Type') #still needed for l10n_cl until next saas

```

## File: models\l10n_latam_document_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api
from odoo.osv import expression


class L10nLatamDocumentType(models.Model):

    _name = 'l10n_latam.document.type'
    _description = 'Latam Document Type'
    _order = 'sequence, id'

    active = fields.Boolean(default=True)
    sequence = fields.Integer(
        default=10, required=True, help='To set in which order show the documents type taking into account the most'
        ' commonly used first')
    country_id = fields.Many2one(
        'res.country', required=True, index=True, help='Country in which this type of document is valid')
    name = fields.Char(required=True, index=True, help='The document name')
    doc_code_prefix = fields.Char(
        'Document Code Prefix', help="Prefix for Documents Codes on Invoices and Account Moves. For eg. 'FA ' will"
        " build 'FA 0001-0000001' Document Number")
    code = fields.Char(help='Code used by different localizations')
    report_name = fields.Char('Name on Reports', help='Name that will be printed in reports, for example "CREDIT NOTE"')
    internal_type = fields.Selection(
        [('invoice', 'Invoices'), ('debit_note', 'Debit Notes'), ('credit_note', 'Credit Notes')], index=True,
        help='Analog to odoo account.move.move_type but with more options allowing to identify the kind of document we are'
        ' working with. (not only related to account.move, could be for documents of other models like stock.picking)')

    def _format_document_number(self, document_number):
        """ Method to be inherited by different localizations. The purpose of this method is to allow:
        * making validations on the document_number. If it is wrong it should raise an exception
        * format the document_number against a pattern and return it
        """
        self.ensure_one()
        return document_number

    def name_get(self):
        result = []
        for rec in self:
            name = rec.name
            if rec.code:
                name = '(%s) %s' % (rec.code, name)
            result.append((rec.id, name))
        return result

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        args = args or []
        if operator == 'ilike' and not (name or '').strip():
            domain = []
        else:
            domain = ['|', ('name', 'ilike', name), ('code', 'ilike', name)]
        return self._search(expression.AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)

    def _filter_taxes_included(self, taxes):
        """ This method is to be inherited by different localizations and must return filter the given taxes recordset
        returning the taxes to be included on reports of this document type. All taxes are going to be discriminated
        except the one returned by this method. """
        self.ensure_one()
        return self.env['account.tax']

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResCompany(models.Model):

    _inherit = "res.company"

    def _localization_use_documents(self):
        """ This method is to be inherited by localizations and return True if localization use documents """
        self.ensure_one()
        return False

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company
from . import l10n_latam_document_type
from . import account_journal
from . import account_move
from . import account_move_line
from . import account_chart_template
from . import ir_sequence

```

## File: report\invoice_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class AccountInvoiceReport(models.Model):

    _inherit = 'account.invoice.report'

    l10n_latam_document_type_id = fields.Many2one('l10n_latam.document.type', 'Document Type', index=True)
    _depends = {'account.move': ['l10n_latam_document_type_id'],}

    def _select(self):
        return super()._select() + ", move.l10n_latam_document_type_id as l10n_latam_document_type_id"

```

## File: report\invoice_report_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record model="ir.ui.view" id="view_account_invoice_report_search">
        <field name="name">account.invoice.report.search</field>
        <field name="model">account.invoice.report</field>
        <field name="inherit_id" ref="account.view_account_invoice_report_search"/>
        <field name="arch" type="xml">
            <search>
                <field name="l10n_latam_document_type_id"/>
            </search>
            <filter name="user" position="after">
                <filter domain="[]" string="Document Type" name="l10n_latam_document_type" context="{'group_by':'l10n_latam_document_type_id'}"/>
            </filter>
        </field>
    </record>

</odoo>

```

## File: report\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import invoice_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_latam_document_type_all,l10n_latam.document.type.all,model_l10n_latam_document_type,,1,0,0,0
access_l10n_latam_document_type_account_manager,l10n_latam.document.type.all,model_l10n_latam_document_type,account.group_account_manager,1,1,1,0

```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_journal_form" model="ir.ui.view">
        <field name="model">account.journal</field>
        <field name="name">account.journal.form</field>
        <field name="inherit_id" ref="account.view_account_journal_form"/>
        <field name="arch" type="xml">
            <form>
                <field name="country_code" invisible="1"/>
                <field name="l10n_latam_company_use_documents" invisible="1"/>
            </form>
            <field name="type" position="after">
                <field name="l10n_latam_use_documents" attrs="{'invisible': ['|', ('l10n_latam_company_use_documents', '=', False), ('type', 'not in', ['purchase','sale'])]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\account_move_line_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_move_line_filter" model="ir.ui.view">
        <field name="name">account.move.line.filter</field>
        <field name="model">account.move.line</field>
        <field name="inherit_id" ref="account.view_account_move_line_filter"/>
        <field name="arch" type="xml">
            <separator position="before">
                <field name="l10n_latam_document_type_id"/>
            </separator>
            <group>
                <filter string="Document Type" name="l10n_latam_document_type" domain="" context="{'group_by':'l10n_latam_document_type_id'}"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.move.select</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="l10n_latam_document_type_id"/>
            </field>
            <group>
                <filter string="Document Type" name="l10n_latam_document_type" domain="" context="{'group_by':'l10n_latam_document_type_id'}"/>
            </group>
        </field>
    </record>

    <record id="view_account_move_filter" model="ir.ui.view">
        <field name="name">account.move.filter</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_move_filter"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="l10n_latam_document_type_id"/>
            </field>
            <group>
                <filter string="Document Type" name="l10n_latam_document_type" domain="" context="{'group_by':'l10n_latam_document_type_id'}"/>
            </group>
        </field>
    </record>

    <record id="view_move_form" model="ir.ui.view">
        <field name="name">account.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <form>
                <field name="l10n_latam_available_document_type_ids" invisible="1"/>
                <field name="l10n_latam_use_documents" invisible="1"/>
                <field name="l10n_latam_manual_document_number" invisible="1"/>
            </form>

            <xpath expr="//field[@name='journal_id']/.." position="after">
                <field name="l10n_latam_document_type_id"
                    attrs="{'invisible': [('l10n_latam_use_documents', '=', False)], 'required': [('partner_id', '!=', False), ('l10n_latam_use_documents', '=', True)], 'readonly': [('posted_before', '=', True)]}"
                    domain="[('id', 'in', l10n_latam_available_document_type_ids)]" options="{'no_open': True, 'no_create': True}"/>
                <field name="l10n_latam_document_number"
                    attrs="{'invisible': ['|', ('l10n_latam_use_documents', '=', False), ('l10n_latam_manual_document_number', '=', False), '|', '|', ('l10n_latam_use_documents', '=', False), ('highest_name', '!=', False), ('state', '!=', 'draft')], 'required': [('partner_id', '!=', False), '|', ('l10n_latam_manual_document_number', '=', True), ('highest_name', '=', False)], 'readonly': [('posted_before', '=', True), ('state', '!=', 'draft')]}"/>
            </xpath>

            <!-- on latam_documents we use document_number to set name -->
            <field name="name" position="attributes">
                <attribute name="attrs">{'readonly': ['|', ('state', '!=', 'draft'), ('l10n_latam_use_documents', '=', True)]}</attribute>
            </field>

        </field>
    </record>

</odoo>

```

## File: views\ir_sequence_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="sequence_view" model="ir.ui.view">
        <field name="name">ir.sequence.form</field>
        <field name="model">ir.sequence</field>
        <field name="inherit_id" ref="base.sequence_view"/>
        <field name="arch" type="xml">
            <field name="implementation" position="after">
                <field name="l10n_latam_document_type_id" options="{'no_open': True, 'no_create': True}"/>
            </field>
        </field>
    </record>

    <record id="sequence_view_tree" model="ir.ui.view">
        <field name="name">ir.sequence.tree</field>
        <field name="model">ir.sequence</field>
        <field name="inherit_id" ref="base.sequence_view_tree"/>
        <field name="arch" type="xml">
            <field name="implementation" position="after">
                <field name="l10n_latam_document_type_id"/>
            </field>
        </field>
    </record>

    <record id="view_sequence_search" model="ir.ui.view">
        <field name="name">ir.sequence.search</field>
        <field name="model">ir.sequence</field>
        <field name="inherit_id" ref="base.view_sequence_search"/>
        <field name="arch" type="xml">
            <field name="code" position="before">
                <field name="l10n_latam_document_type_id"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\l10n_latam_document_type_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_document_type_form" model="ir.ui.view">
        <field name="name">l10n_latam.document.type.form</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="arch" type="xml">
            <form string="Document Type" create="0" edit="0">
                <group>
                    <field name='code'/>
                    <field name="name"/>
                    <field name='doc_code_prefix'/>
                    <field name='report_name'/>
                    <field name='internal_type'/>
                    <field name='country_id'/>
                </group>
            </form>
        </field>
    </record>

    <record id="view_document_type_tree" model="ir.ui.view">
        <field name="name">l10n_latam.document.type.tree</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="arch" type="xml">
            <tree string="Document Type" decoration-muted="(not active)" create="0" edit="0">
                <field name="code"/>
                <field name="name"/>
                <field name="doc_code_prefix"/>
                <field name='report_name'/>
                <field name='internal_type'/>
                <field name='country_id'/>
                <field name="active" widget="boolean_toggle"/>
            </tree>
        </field>
    </record>

    <record id="view_document_type_filter" model="ir.ui.view">
        <field name="name">l10n_latam.document.type.filter</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="arch" type="xml">
            <search string="Document Type">
                <field name="name"/>
                <field name="code"/>
                <field name='internal_type'/>
                <field name='country_id'/>
                <filter name="active" string="Active" domain="[('active','=',True)]" help="Show active document types"/>
                <filter name="inactive" string="Archived" domain="[('active','=',False)]" help="Show archived document types"/>
                <group expand="1" string="Group By...">
                    <filter string="Internal Type" name="group_by_internal_type" context="{'group_by':'internal_type'}"/>
                    <filter string="Localization" name="group_by_localization" context="{'group_by':'country_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_document_type">
        <field name="name">Document Types</field>
        <field name="res_model">l10n_latam.document.type</field>
        <field name="domain">['|', ('active', '=', True), ('active', '=', False)]</field>
        <field name="context">{"search_default_active":1}</field>
    </record>

    <menuitem action="action_document_type" id="menu_document_type" sequence="20" parent="account.account_account_menu"/>

</odoo>

```

## File: views\report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="external_layout_standard" inherit_id="web.external_layout_standard" >
        <!-- support for custom header -->
        <div t-attf-class="header o_company_#{company.id}_layout" position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </div>
        <div t-attf-class="header o_company_#{company.id}_layout" position="after">
            <div t-attf-class="header o_company_#{company.id}_layout" t-if="custom_header">
                <t t-if="custom_header" t-call="#{custom_header}"/>
            </div>
        </div>

        <!-- support for custom footer -->
        <div t-attf-class="footer o_standard_footer o_company_#{company.id}_layout" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>

        <div t-attf-class="footer o_standard_footer o_company_#{company.id}_layout" position="after">
            <div t-attf-class="footer o_standard_footer o_company_#{company.id}_layout" t-if="custom_footer">
                <t t-if="custom_footer" t-raw="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_clean" inherit_id="web.external_layout_clean" >
        <!-- support for custom header -->
        <div t-attf-class="header o_company_#{company.id}_layout" position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </div>
        <div t-attf-class="header o_company_#{company.id}_layout" position="after">
            <div t-attf-class="header o_company_#{company.id}_layout" t-if="custom_header">
                <div class="o_clean_header">
                    <t t-if="custom_header" t-call="#{custom_header}"/>
                </div>
            </div>
        </div>

        <!-- support for custom footer -->
        <div t-attf-class="footer o_clean_footer o_company_#{company.id}_layout" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div t-attf-class="footer o_clean_footer o_company_#{company.id}_layout" position="after">
            <div t-attf-class="footer o_clean_footer o_company_#{company.id}_layout" t-if="custom_footer">
                <t t-if="custom_footer" t-raw="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_boxed" inherit_id="web.external_layout_boxed" >
        <!-- support for custom header -->
        <div t-attf-class="header o_company_#{company.id}_layout" position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </div>
        <div t-attf-class="header o_company_#{company.id}_layout" position="after">
            <div t-attf-class="header o_company_#{company.id}_layout" t-if="custom_header">
                <div class="o_boxed_header">
                    <t t-if="custom_header" t-call="#{custom_header}"/>
                </div>
            </div>
        </div>

        <!-- support for custom footer -->
        <div t-attf-class="footer o_boxed_footer o_company_#{company.id}_layout" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div t-attf-class="footer o_boxed_footer o_company_#{company.id}_layout" position="after">
            <div t-attf-class="footer o_boxed_footer o_company_#{company.id}_layout" t-if="custom_footer">
                <t t-if="custom_footer" t-raw="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_background" inherit_id="web.external_layout_background" >
        <!-- support for custom header -->
        <div t-attf-class="o_company_#{company.id}_layout header" position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </div>
        <div t-attf-class="o_company_#{company.id}_layout header" position="after">
            <div t-attf-class="o_company_#{company.id}_layout header" t-if="custom_header">
                <div class="o_background_header">
                    <t t-if="custom_header" t-call="#{custom_header}"/>
                </div>
            </div>
        </div>

        <!-- support for custom footer -->
        <div t-attf-class="o_company_#{company.id}_layout footer o_background_footer" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div t-attf-class="o_company_#{company.id}_layout footer o_background_footer" position="after">
            <div t-attf-class="o_company_#{company.id}_layout footer o_background_footer" t-if="custom_footer">
                <t t-if="custom_footer" t-raw="custom_footer"/>
            </div>
        </div>
    </template>

</odoo>

```

## File: wizards\account_debit_note.py

```python
from odoo import models


class AccountDebitNote(models.TransientModel):

    _inherit = 'account.debit.note'

    def create_debit(self):
        """ Properly compute the latam document type of type debit note. """
        res = super().create_debit()
        new_move_id = res.get('res_id')
        if new_move_id:
            new_move = self.env['account.move'].browse(new_move_id)
            new_move._compute_l10n_latam_document_type()
        return res

```

## File: wizards\account_move_reversal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import UserError


class AccountMoveReversal(models.TransientModel):
    _inherit = "account.move.reversal"

    l10n_latam_use_documents = fields.Boolean(compute='_compute_document_type')
    l10n_latam_document_type_id = fields.Many2one('l10n_latam.document.type', 'Document Type', ondelete='cascade', domain="[('id', 'in', l10n_latam_available_document_type_ids)]", compute='_compute_document_type', readonly=False, inverse='_inverse_document_type')
    l10n_latam_available_document_type_ids = fields.Many2many('l10n_latam.document.type', compute='_compute_document_type')
    l10n_latam_document_number = fields.Char(string='Document Number')
    l10n_latam_manual_document_number = fields.Boolean(compute='_compute_l10n_latam_manual_document_number', string='Manual Number')

    def _inverse_document_type(self):
        self._clean_pipe()
        self.l10n_latam_document_number = '%s|%s' % (self.l10n_latam_document_type_id.id or '', self.l10n_latam_document_number or '')

    @api.depends('l10n_latam_document_type_id')
    def _compute_l10n_latam_manual_document_number(self):
        self.l10n_latam_manual_document_number = False
        for rec in self.filtered('move_ids'):
            move = rec.move_ids[0]
            if move.journal_id and move.journal_id.l10n_latam_use_documents:
                rec.l10n_latam_manual_document_number = self.env['account.move']._is_manual_document_number(move.journal_id)

    @api.model
    def _reverse_type_map(self, move_type):
        match = {
            'entry': 'entry',
            'out_invoice': 'out_refund',
            'in_invoice': 'in_refund',
            'in_refund': 'in_invoice',
            'out_receipt': 'in_receipt',
            'in_receipt': 'out_receipt'}
        return match.get(move_type)

    @api.depends('move_ids')
    def _compute_document_type(self):
        self.l10n_latam_available_document_type_ids = False
        self.l10n_latam_document_type_id = False
        self.l10n_latam_use_documents = False
        for record in self:
            if len(record.move_ids) > 1:
                move_ids_use_document = record.move_ids._origin.filtered(lambda move: move.l10n_latam_use_documents)
                if move_ids_use_document:
                    raise UserError(_('You can only reverse documents with legal invoicing documents from Latin America one at a time.\nProblematic documents: %s') % ", ".join(move_ids_use_document.mapped('name')))
            else:
                record.l10n_latam_use_documents = record.move_ids.journal_id.l10n_latam_use_documents

            if record.l10n_latam_use_documents:
                refund = record.env['account.move'].new({
                    'move_type': record._reverse_type_map(record.move_ids.move_type),
                    'journal_id': record.move_ids.journal_id.id,
                    'partner_id': record.move_ids.partner_id.id,
                    'company_id': record.move_ids.company_id.id,
                })
                record.l10n_latam_document_type_id = refund.l10n_latam_document_type_id
                record.l10n_latam_available_document_type_ids = refund.l10n_latam_available_document_type_ids

    def _prepare_default_reversal(self, move):
        """ Set the default document type and number in the new revsersal move taking into account the ones selected in
        the wizard """
        res = super()._prepare_default_reversal(move)
        # self.l10n_latam_document_number will have a ',' only if l10n_latam_document_type_id is changed and inverse methods is called
        if self.l10n_latam_document_number and '|' in self.l10n_latam_document_number:
            l10n_latam_document_type_id, l10n_latam_document_number = self.l10n_latam_document_number.split('|')
            res.update({
                'l10n_latam_document_type_id': int(l10n_latam_document_type_id) if l10n_latam_document_type_id else False,
                'l10n_latam_document_number': l10n_latam_document_number or False,
            })
        else:
            res.update({
                'l10n_latam_document_type_id': self.l10n_latam_document_type_id.id,
                'l10n_latam_document_number': self.l10n_latam_document_number,
            })
        return res

    def _clean_pipe(self):
        """ Clean pipe in case the user confirm but he gets a raise, the l10n_latam_document_number is stored now
        with the doc type id, we should remove to append new one or to format properly"""
        latam_document = self.l10n_latam_document_number or ''
        if '|' in latam_document:
            latam_document = latam_document[latam_document.index('|')+1:]
        self.l10n_latam_document_number = latam_document

    @api.onchange('l10n_latam_document_number', 'l10n_latam_document_type_id')
    def _onchange_l10n_latam_document_number(self):
        if self.l10n_latam_document_type_id:
            self._clean_pipe()
            l10n_latam_document_number = self.l10n_latam_document_type_id._format_document_number(
                self.l10n_latam_document_number)
            if self.l10n_latam_document_number != l10n_latam_document_number:
                self.l10n_latam_document_number = l10n_latam_document_number

```

## File: wizards\account_move_reversal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_move_reversal" model="ir.ui.view">
        <field name="name">account.move.reversal.form</field>
        <field name="model">account.move.reversal</field>
        <field name="inherit_id" ref="account.view_account_move_reversal"/>
        <field name="arch" type="xml">
            <form>
                <field name="l10n_latam_use_documents" invisible="1"/>
                <field name="l10n_latam_manual_document_number" invisible="1"/>
            </form>
            <field name="date" position="before">
                <field name="l10n_latam_available_document_type_ids" invisible="1"/>
                <field name="l10n_latam_document_type_id" attrs="{'invisible': ['|', ('l10n_latam_use_documents', '=', False), ('refund_method', '=', 'refund')], 'required': [('l10n_latam_use_documents', '=', True), ('refund_method', '!=', 'refund')]}" options="{'no_open': True, 'no_create': True}"/>
                <field name="l10n_latam_document_number" attrs="{'invisible': ['|', '|', ('l10n_latam_use_documents', '=', False), ('l10n_latam_manual_document_number', '=', False), ('refund_method', '=', 'refund')], 'required': [('l10n_latam_manual_document_number', '=', True), ('l10n_latam_use_documents', '=', True), ('refund_method', '!=', 'refund')]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_reversal
from . import account_debit_note

```

