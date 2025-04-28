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
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template(model='account.journal')
    def _get_latam_document_account_journal(self, template_code):
        """ We add use_documents or not depending on the context"""
        if self.env.company._localization_use_documents():
            return {
                'sale': {'l10n_latam_use_documents': True},
                'purchase': {'l10n_latam_use_documents': True},
            }

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

    def _compute_has_sequence_holes(self):
        use_documents_journals = self.filtered(lambda j: j.l10n_latam_use_documents)
        use_documents_journals.has_sequence_holes = False
        if other_journals := self - use_documents_journals:
            super(AccountJournal, other_journals)._compute_has_sequence_holes()

    @api.constrains('l10n_latam_use_documents')
    def check_use_document(self):
        for rec in self:
            if rec.env['account.move'].search_count([('journal_id', '=', rec.id), ('posted_before', '=', True)], limit=1):
                raise ValidationError(_(
                    'You can not modify the field "Use Documents?" if there are validated invoices in this journal!'))

    @api.depends('type', 'l10n_latam_use_documents')
    def _compute_debit_sequence(self):
        super()._compute_debit_sequence()
        for journal in self:
            if journal.l10n_latam_use_documents:
                journal.debit_sequence = False

    @api.depends('type', 'l10n_latam_use_documents')
    def _compute_refund_sequence(self):
        super()._compute_refund_sequence()
        for journal in self:
            if journal.l10n_latam_use_documents:
                journal.refund_sequence = False

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import models, fields, api, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools.sql import column_exists, create_column, drop_index, index_exists


class AccountMove(models.Model):

    _inherit = "account.move"

    _sql_constraints = [(
        'unique_name', "", "Another entry with the same name already exists.",
    ), (
        'unique_name_latam', "", "Another entry with the same name already exists.",
    )]

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

        if not index_exists(self.env.cr, "account_move_unique_name_latam"):
            drop_index(self.env.cr, "account_move_unique_name", self._table)
            self.env.cr.execute("""
                CREATE UNIQUE INDEX account_move_unique_name
                                 ON account_move(name, journal_id)
                              WHERE (state = 'posted' AND name != '/'
                                AND (l10n_latam_document_type_id IS NULL OR move_type NOT IN ('in_invoice', 'in_refund', 'in_receipt')));
                CREATE UNIQUE INDEX account_move_unique_name_latam
                                 ON account_move(name, commercial_partner_id, l10n_latam_document_type_id, company_id)
                              WHERE (state = 'posted' AND name != '/'
                                AND (l10n_latam_document_type_id IS NOT NULL AND move_type IN ('in_invoice', 'in_refund', 'in_receipt')));
            """)
        return super()._auto_init()

    l10n_latam_available_document_type_ids = fields.Many2many('l10n_latam.document.type', compute='_compute_l10n_latam_available_document_types')
    l10n_latam_document_type_id = fields.Many2one(
        'l10n_latam.document.type', string='Document Type', readonly=False, auto_join=True, index='btree_not_null', compute='_compute_l10n_latam_document_type', store=True)
    l10n_latam_document_number = fields.Char(
        compute='_compute_l10n_latam_document_number', inverse='_inverse_l10n_latam_document_number',
        string='Document Number', readonly=False)
    l10n_latam_use_documents = fields.Boolean(related='journal_id.l10n_latam_use_documents')
    l10n_latam_manual_document_number = fields.Boolean(compute='_compute_l10n_latam_manual_document_number', string='Manual Number')
    l10n_latam_document_type_id_code = fields.Char(related='l10n_latam_document_type_id.code', string='Doc Type')

    @api.depends('l10n_latam_document_type_id')
    def _compute_name(self):
        """ Change the way that the use_document moves name is computed:

        * If move use document but does not have document type selected then name = 'False' to do not show the name.
        * If move use document and are numbered manually do not compute name at all (will be set manually)
        * If move use document and is in draft state and has not been posted before we restart name to False (this is
           when we change the document type) """
        without_doc_type = self.filtered(lambda x: x.journal_id.l10n_latam_use_documents and not x.l10n_latam_document_type_id)
        manual_documents = self.filtered(lambda x: x.journal_id.l10n_latam_use_documents and x.l10n_latam_manual_document_number)
        (without_doc_type + manual_documents.filtered(lambda x: not x.name)).name = False
        # we need to group moves by document type as _compute_name will apply the same name prefix of the first record to the others
        group_by_document_type = defaultdict(self.env['account.move'].browse)
        for move in (self - without_doc_type - manual_documents):
            group_by_document_type[move.l10n_latam_document_type_id.id] += move
        for group in group_by_document_type.values():
            super(AccountMove, group)._compute_name()

    def _compute_name_placeholder(self):
        use_documents_moves = self.filtered(lambda m: m.journal_id.l10n_latam_use_documents)
        use_documents_moves.name_placeholder = False
        if other_moves := self - use_documents_moves:
            super(AccountMove, other_moves)._compute_name_placeholder()

    @api.depends('l10n_latam_document_type_id', 'journal_id')
    def _compute_l10n_latam_manual_document_number(self):
        """ Indicates if this document type uses a sequence or if the numbering is made manually """
        recs_with_journal_id = self.filtered(lambda x: x.journal_id and x.journal_id.l10n_latam_use_documents)
        for rec in recs_with_journal_id:
            rec.l10n_latam_manual_document_number = rec._is_manual_document_number()
        remaining = self - recs_with_journal_id
        remaining.l10n_latam_manual_document_number = False

    def _is_manual_document_number(self):
        return self.journal_id.type == 'purchase'

    @api.depends('name')
    def _compute_l10n_latam_document_number(self):
        recs_with_name = self.filtered(lambda x: x.name and x.name != "/")
        for rec in recs_with_name:
            name = rec.name
            doc_code_prefix = rec.l10n_latam_document_type_id.doc_code_prefix
            if doc_code_prefix and name:
                name = name.split(" ", 1)[-1]
            rec.l10n_latam_document_number = name
        remaining = self - recs_with_name
        remaining.l10n_latam_document_number = False

    @api.onchange('l10n_latam_document_type_id', 'l10n_latam_document_number', 'partner_id')
    def _inverse_l10n_latam_document_number(self):
        for rec in self.filtered(lambda x: x.l10n_latam_document_type_id):
            if not rec.l10n_latam_document_number:
                rec.name = False
            else:
                l10n_latam_document_number = rec.l10n_latam_document_number
                if not rec._skip_format_document_number():
                    l10n_latam_document_number = rec.l10n_latam_document_type_id._format_document_number(rec.l10n_latam_document_number)
                if rec.l10n_latam_document_number != l10n_latam_document_number:
                    rec.l10n_latam_document_number = l10n_latam_document_number
                rec.name = "%s %s" % (rec.l10n_latam_document_type_id.doc_code_prefix, l10n_latam_document_number)

    @api.onchange('l10n_latam_document_type_id')
    def _onchange_l10n_latam_document_type_id(self):
        # if we change document or journal and we are in draft and not posted, we clean number so that is recomputed
        if (self.journal_id.l10n_latam_use_documents and self.l10n_latam_document_type_id
              and not self.l10n_latam_manual_document_number and self.state == 'draft' and not self.posted_before):
            self.name = False
            self._compute_name()

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

    def _skip_format_document_number(self):
        """Hook to be overridden in localisation"""
        self.ensure_one()
        return False

    def _get_starting_sequence(self):
        if self.journal_id.l10n_latam_use_documents:
            if self.l10n_latam_document_type_id:
                return "%s 00000000" % (self.l10n_latam_document_type_id.doc_code_prefix)
            # There was no pattern found, propose one
            return ""

        return super(AccountMove, self)._get_starting_sequence()

    def _post(self, soft=True):
        for rec in self.filtered(lambda x: x.l10n_latam_use_documents and (not x.name or x.name == '/')):
            if rec.move_type in ('in_receipt', 'out_receipt'):
                raise UserError(_('We do not accept the usage of document types on receipts yet. '))
        return super()._post(soft)

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
        internal_types = []
        invoice_type = self.move_type
        if invoice_type in ['out_refund', 'in_refund']:
            internal_types = ['credit_note']
        elif invoice_type in ['out_invoice', 'in_invoice']:
            internal_types = ['invoice', 'debit_note']
        if self.debit_origin_id:
            internal_types = ['debit_note']
        internal_types += ['all']
        return [('internal_type', 'in', internal_types), ('country_id', '=', self.company_id.account_fiscal_country_id.id)]

    @api.depends('journal_id', 'partner_id', 'company_id', 'move_type', 'debit_origin_id')
    def _compute_l10n_latam_available_document_types(self):
        self.l10n_latam_available_document_type_ids = False
        for rec in self.filtered(lambda x: x.journal_id and x.l10n_latam_use_documents and x.partner_id):
            rec.l10n_latam_available_document_type_ids = self.env['l10n_latam.document.type'].search(rec._get_l10n_latam_documents_domain())

    @api.depends('l10n_latam_available_document_type_ids')
    def _compute_l10n_latam_document_type(self):
        for rec in self.filtered(lambda x: x.state == 'draft' and (not x.posted_before if x.move_type in ['out_invoice', 'out_refund'] else True)):
            document_types = rec.l10n_latam_available_document_type_ids._origin
            rec.l10n_latam_document_type_id = document_types and document_types[0].id

    def _compute_made_sequence_gap(self):
        use_documents_moves = self.filtered(lambda m: m.journal_id.l10n_latam_use_documents)
        use_documents_moves.made_sequence_gap = False
        if other_moves := self - use_documents_moves:
            super(AccountMove, other_moves)._compute_made_sequence_gap()

    def _set_next_made_sequence_gap(self, made_gap: bool):
        if other_moves := self.filtered(lambda m: not m.journal_id.l10n_latam_use_documents):
            super(AccountMove, other_moves)._set_next_made_sequence_gap(made_gap)

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields
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
        related='move_id.l10n_latam_document_type_id', auto_join=True, store=True, index='btree_not_null')

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
    _rec_names_search = ['name', 'code']

    active = fields.Boolean(default=True)
    sequence = fields.Integer(
        default=10, required=True, help='To set in which order show the documents type taking into account the most'
        ' commonly used first')
    country_id = fields.Many2one(
        'res.country', required=True, index=True, help='Country in which this type of document is valid')
    name = fields.Char(required=True, help='The document name', translate=True)
    doc_code_prefix = fields.Char(
        'Document Code Prefix', help="Prefix for Documents Codes on Invoices and Account Moves. For eg. 'FA ' will"
        " build 'FA 0001-0000001' Document Number")
    code = fields.Char(help='Code used by different localizations')
    report_name = fields.Char('Name on Reports', help='Name that will be printed in reports, for example "CREDIT NOTE"', translate=True)
    internal_type = fields.Selection(
        [('invoice', 'Invoices'), ('debit_note', 'Debit Notes'), ('credit_note', 'Credit Notes'), ('all', 'All Documents')],
        help='Analog to odoo account.move.move_type but with more options allowing to identify the kind of document we are'
        ' working with. (not only related to account.move, could be for documents of other models like stock.picking)')

    def _format_document_number(self, document_number):
        """ Method to be inherited by different localizations. The purpose of this method is to allow:
        * making validations on the document_number. If it is wrong it should raise an exception
        * format the document_number against a pattern and return it
        """
        self.ensure_one()
        return document_number

    @api.depends('code')
    def _compute_display_name(self):
        for rec in self:
            name = rec.name
            if rec.code:
                name = f'({rec.code}) {name}'
            rec.display_name = name

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

```

## File: report\invoice_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields
from odoo.tools import SQL


class AccountInvoiceReport(models.Model):

    _inherit = 'account.invoice.report'

    l10n_latam_document_type_id = fields.Many2one('l10n_latam.document.type', 'Document Type', index=True)
    _depends = {'account.move': ['l10n_latam_document_type_id'],}

    def _select(self) -> SQL:
        return SQL("%s, move.l10n_latam_document_type_id as l10n_latam_document_type_id",
                   super()._select())

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
access_l10n_latam_document_type,access_l10n_latam_document_type,model_l10n_latam_document_type,base.group_user,1,0,0,0
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
                <field name="l10n_latam_use_documents" invisible="not l10n_latam_company_use_documents or type not in ['purchase', 'sale']"/>
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

            <xpath expr="//div[@name='journal_div']" position="after">
                <field name="l10n_latam_document_type_id"
                    invisible="not l10n_latam_use_documents"
                    readonly="posted_before"
                    required="partner_id and l10n_latam_use_documents"
                    domain="[('id', 'in', l10n_latam_available_document_type_ids)]" options="{'no_open': True, 'no_create': True}"/>
                <field name="l10n_latam_document_number"
                    invisible="(not l10n_latam_use_documents or not l10n_latam_manual_document_number) and (not l10n_latam_use_documents or highest_name or state != 'draft')"
                    readonly="posted_before and state != 'draft'"
                    required="partner_id and l10n_latam_use_documents and l10n_latam_manual_document_number"/>
            </xpath>

            <!-- on latam_documents we use document_number to set name -->
            <field name="name" position="attributes">
                <attribute name="readonly">state != 'draft' or l10n_latam_use_documents</attribute>
                <attribute name="force_save">1</attribute>
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
        <field name="name">l10n_latam.document.type.list</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="arch" type="xml">
            <list string="Document Type" decoration-muted="(not active)" create="0" edit="0">
                <field name="code"/>
                <field name="name"/>
                <field name="doc_code_prefix"/>
                <field name='report_name'/>
                <field name='internal_type'/>
                <field name='country_id'/>
                <field name="active" widget="boolean_toggle"/>
            </list>
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
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>
        <xpath expr="//img/../.." position="after">
            <div t-attf-class="header o_company_#{company.id}_layout"
                 t-if="custom_header and company._localization_use_documents()">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content d-flex border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>

        <div class="o_footer_content d-flex border-top pt-2" position="after">
            <div class="o_footer_content border-top pt-2"
                 t-if="custom_footer and company._localization_use_documents()">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_bold" inherit_id="web.external_layout_bold" >
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>
        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company._localization_use_documents()">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content row border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content row border-top pt-2" position="after">
            <div class="o_footer_content row border-top pt-2"
                 t-if="custom_footer and company._localization_use_documents()">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_boxed" inherit_id="web.external_layout_boxed" >
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>
        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company._localization_use_documents()">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content row border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content row border-top pt-2" position="after">
            <div class="o_footer_content row border-top pt-2"
                 t-if="custom_footer and company._localization_use_documents()">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_striped" inherit_id="web.external_layout_striped" >
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>
        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company._localization_use_documents()">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content border-top pt-2 text-center" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content border-top pt-2 text-center" position="after">
            <div class="o_footer_content border-top pt-2 text-center"
                 t-if="custom_footer and company._localization_use_documents()">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_bubble" inherit_id="web.external_layout_bubble">
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>
        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company._localization_use_documents()">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-4 text-center" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-4 text-center" position="after">
            <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-4 text-center"
                 t-if="custom_footer and company._localization_use_documents()">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_wave" inherit_id="web.external_layout_wave">
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>
        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company._localization_use_documents()">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-5 text-center" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-5 text-center" position="after">
            <div t-attf-class="o_footer_content {{report_type != 'pdf' and 'position-absolute end-0 start-0 bottom-0 mx-5'}} pt-5 text-center"
                 t-if="custom_footer and company._localization_use_documents()">
                <t t-out="custom_footer"/>
            </div>
        </div>
    </template>

    <template id="external_layout_folder" inherit_id="web.external_layout_folder">
        <!-- support for custom header -->
        <xpath expr="//img/../.." position="attributes">
            <attribute name="t-if">not custom_header</attribute>
        </xpath>
        <xpath expr="//img/../.." position="after">
            <div t-if="custom_header and company._localization_use_documents()">
                <t t-call="#{custom_header}"/>
            </div>
        </xpath>

        <!-- support for custom footer -->
        <div class="o_footer_content d-flex border-top pt-2" position="attributes">
            <attribute name="t-if">not custom_footer</attribute>
        </div>
        <div class="o_footer_content d-flex border-top pt-2" position="after">
            <div class="o_footer_content border-top pt-2"
                 t-if="custom_footer and company._localization_use_documents()">
                <t t-out="custom_footer"/>
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
            new_move._onchange_l10n_latam_document_type_id()
        return res

```

## File: wizards\account_move_reversal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import UserError


class AccountMoveReversal(models.TransientModel):
    _inherit = "account.move.reversal"

    l10n_latam_use_documents = fields.Boolean(compute='_compute_documents_info')
    l10n_latam_document_type_id = fields.Many2one('l10n_latam.document.type', 'Document Type', ondelete='cascade', domain="[('id', 'in', l10n_latam_available_document_type_ids)]", compute='_compute_document_type', readonly=False, store=True)
    l10n_latam_available_document_type_ids = fields.Many2many('l10n_latam.document.type', compute='_compute_documents_info')
    l10n_latam_document_number = fields.Char(string='Document Number')
    l10n_latam_manual_document_number = fields.Boolean(compute='_compute_l10n_latam_manual_document_number', string='Manual Number')

    @api.depends('l10n_latam_document_type_id', 'journal_id')
    def _compute_l10n_latam_manual_document_number(self):
        self.l10n_latam_manual_document_number = False
        for wiz in self.filtered(lambda x: x.journal_id and x.journal_id.l10n_latam_use_documents):
            wiz.l10n_latam_manual_document_number = self.env['account.move'].new({
                'move_type': wiz._reverse_type_map(wiz.move_ids[0].move_type),
                'journal_id': wiz.journal_id.id,
                'partner_id': wiz.move_ids[0].partner_id.id,
                'company_id': wiz.move_ids[0].company_id.id,
                'reversed_entry_id': wiz.move_ids[0].id,
            })._is_manual_document_number()

    @api.model
    def _reverse_type_map(self, move_type):
        self.ensure_one()
        match = {
            'entry': 'entry',
            'out_invoice': 'out_refund',
            'in_invoice': 'in_refund',
            'in_refund': 'in_invoice',
            'out_receipt': 'in_receipt',
            'in_receipt': 'out_receipt'}
        return match.get(move_type)

    @api.depends('l10n_latam_available_document_type_ids', 'journal_id')
    def _compute_document_type(self):
        for record in self.filtered(
                lambda x: not x.l10n_latam_document_type_id or
                x.l10n_latam_document_type_id not in x.l10n_latam_available_document_type_ids):
            document_types = record.l10n_latam_available_document_type_ids._origin
            record.l10n_latam_document_type_id = document_types[0] if document_types else False

    @api.depends('move_ids', 'journal_id')
    def _compute_documents_info(self):
        self.l10n_latam_available_document_type_ids = False
        self.l10n_latam_use_documents = False
        for record in self:
            if len(record.move_ids) > 1:
                move_ids_use_document = record.move_ids._origin.filtered(lambda move: move.l10n_latam_use_documents)
                if move_ids_use_document:
                    raise UserError(_('You can only reverse documents with legal invoicing documents from Latin America one at a time.\nProblematic documents: %s', ", ".join(move_ids_use_document.mapped('name'))))
            else:
                record.l10n_latam_use_documents = record.journal_id.l10n_latam_use_documents

            if record.l10n_latam_use_documents:
                refund = record.env['account.move'].new({
                    'move_type': record._reverse_type_map(record.move_ids.move_type),
                    'journal_id': record.journal_id.id,
                    'partner_id': record.move_ids.partner_id.id,
                    'company_id': record.move_ids.company_id.id,
                    'reversed_entry_id': record.move_ids.id,
                })
                record.l10n_latam_available_document_type_ids = refund.l10n_latam_available_document_type_ids

    def _prepare_default_reversal(self, move):
        """ Set the default document type and number in the new revsersal move taking into account the ones selected in
        the wizard """
        res = super()._prepare_default_reversal(move)
        res.update({
            'l10n_latam_document_type_id': self.l10n_latam_document_type_id.id,
            'l10n_latam_document_number': self.l10n_latam_document_number,
        })
        return res

    @api.onchange('l10n_latam_document_number', 'l10n_latam_document_type_id')
    def _onchange_l10n_latam_document_number(self):
        if self.l10n_latam_document_type_id:
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
                <field name="l10n_latam_document_type_id" invisible="not l10n_latam_use_documents" required="l10n_latam_use_documents" options="{'no_open': True, 'no_create': True}"/>
                <field name="l10n_latam_document_number" invisible="not l10n_latam_use_documents or not l10n_latam_manual_document_number" required="l10n_latam_manual_document_number and l10n_latam_use_documents"/>
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

