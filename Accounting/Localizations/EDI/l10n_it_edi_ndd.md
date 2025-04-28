# Odoo Module: l10n_it_edi_ndd

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Italy - E-invoicing - Additional module to support the debit notes (nota di debito - NDD)',
    'countries': ['it'],
    'version': '1.0',
    'depends': [
        'l10n_it_edi',
    ],
    'auto_install': True,
    'description': """
Additional module to support the debit notes (nota di debito - NDD) by adding payment method and document types
    """,
    'category': 'Accounting/Localizations/EDI',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/italy.html',
    'data': [
        'views/account_move_views.xml',
        'views/account_payment_method.xml',
        'views/l10n_it_document_type.xml',
        'data/l10n_it.document.type.csv',
        'security/ir.model.access.csv',
    ],
    'license': 'LGPL-3',
}

```

## File: data\l10n_it.document.type.csv

```csv
"id","code","name","type"
"l10n_it_document_type_01","TD01","Invoice (Immediate or Accompanying if <DatiTrasporto> or <DatiDDT> are completed)","sale"
"l10n_it_document_type_02","TD02","Deposit/advance on invoice","sale"
"l10n_it_document_type_03","TD03","Deposit/advance on parcel","sale"
"l10n_it_document_type_04","TD04","Credit note","sale"
"l10n_it_document_type_05","TD05","Debit note","sale"
"l10n_it_document_type_06","TD06","Parcel","sale"
"l10n_it_document_type_07","TD07","Simplified invoice","sale"
"l10n_it_document_type_08","TD08","Simplified credit note","sale"
"l10n_it_document_type_09","TD09","Simplified debit note","sale"
"l10n_it_document_type_16","TD16","Internal reverse charge self-invoice (Article 17 of Presidential Decree no. 633/72 for invoices with Natura subcodes 'N6')","purchase"
"l10n_it_document_type_17","TD17","Self-invoice for purchases of foreign services (both within and outside the EU) - Alternative to esterometro","purchase"
"l10n_it_document_type_18","TD18","Self-invoice for the purchase of intra-community goods - Alternative to esterometro","purchase"
"l10n_it_document_type_19","TD19","Self-invoice for foreign goods (both intra and non-EU) already present in Italy (Article 17 paragraph 2 of Presidential Decree 633/72) - Alternative to esterometro","purchase"
"l10n_it_document_type_20","TD20","Self-invoice Report (ex art.6 c8 471/97 or art.46 c5 331/93), or for invoice not received or for a lower amount","purchase"
"l10n_it_document_type_21","TD21","Self-invoice for ceiling clearance","purchase"
"l10n_it_document_type_22","TD22","Self-invoice for extraction of goods from VAT warehouse without VAT liability","purchase"
"l10n_it_document_type_23","TD23","Self-invoice for extraction of goods from VAT warehouse with VAT liability","purchase"
"l10n_it_document_type_24","TD24","Deferred invoice (Article 21, paragraph 4, third period letter to Presidential Decree 633/72)","sale"
"l10n_it_document_type_25","TD25","Deferred invoice (Article 21, paragraph 4, third period, letter b (Dropshipping))","sale"
"l10n_it_document_type_26","TD26","Transfer of depreciable assets and for internal transfers (ex art.36 Presidential Decree 633/72)","sale"
"l10n_it_document_type_27","TD27","Self-invoice for self-consumption or for free transfers without recourse","sale"
"l10n_it_document_type_28","TD28","Self-invoice for Italian tax from foreign suppliers identified but not established in Italy and for purchases from San Marino with VAT (paper invoice)","purchase"

```

## File: models\account_move.py

```python
from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column
from odoo.addons.l10n_it_edi_ndd.models.account_payment_methode_line import L10N_IT_PAYMENT_METHOD_SELECTION
from odoo.addons.l10n_it_edi.models.account_move import get_text


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_payment_method = fields.Selection(
        selection=L10N_IT_PAYMENT_METHOD_SELECTION,
        compute='_compute_l10n_it_payment_method',
        store=True,
        readonly=False,
    )

    l10n_it_document_type = fields.Many2one(
        comodel_name='l10n_it.document.type',
        compute='_compute_l10n_it_document_type',
        store=True,
        readonly=False,
    )

    def _auto_init(self):
        # Create compute stored field l10n_it_document_type and l10n_it_payment_method
        # here to avoid timeout error on large databases.
        if not column_exists(self.env.cr, 'account_move', 'l10n_it_payment_method'):
            create_column(self.env.cr, 'account_move', 'l10n_it_payment_method', 'varchar')
        if not column_exists(self.env.cr, 'account_move', 'l10n_it_document_type'):
            create_column(self.env.cr, 'account_move', 'l10n_it_document_type', 'integer')
        return super()._auto_init()

    @api.depends('line_ids.matching_number', 'payment_state')
    def _compute_l10n_it_payment_method(self):
        if self.env.company.account_fiscal_country_id.code != 'IT':
            return

        move_lines_per_matching_number = self.env['account.move.line'].search([
            ('matching_number', 'in', self.line_ids.filtered('matching_number').mapped('matching_number')),
            ('company_id', '=', self.env.company.id),
        ]).grouped('matching_number')

        for move in self:
            matching_numbers = move.line_ids.filtered('matching_number').mapped('matching_number')
            if matching_numbers:
                # We use matching_numbers[0] directly, assuming there's a valid key in the dictionary.
                matching_lines = move_lines_per_matching_number.get(matching_numbers[0])
                if matching_lines and matching_lines.payment_id:
                    payment_method_line = matching_lines.payment_id.payment_method_line_id[0]
                    if payment_method_line:
                        move.l10n_it_payment_method = payment_method_line.l10n_it_payment_method
                        continue  # Skip to the next move

            # Default handling if no valid matching lines found or if conditions don't match
            move.l10n_it_payment_method = move.payment_id.payment_method_line_id.l10n_it_payment_method or move.l10n_it_payment_method or 'MP05'

    @api.depends('state')
    def _compute_l10n_it_document_type(self):
        document_type = self.env['l10n_it.document.type'].search([]).grouped('code')
        for move in self:
            if move.country_code != 'IT' or move.l10n_it_document_type or move.state != 'posted':
                continue

            move.l10n_it_document_type = document_type.get(move._l10n_it_edi_get_document_type())

    def _l10n_it_edi_get_values(self, pdf_values=None):
        # EXTENDS 'l10n_it_edi'
        res = super()._l10n_it_edi_get_values(pdf_values)
        res['document_type'] = self.l10n_it_document_type.code
        res['payment_method'] = self.l10n_it_payment_method

        return res

    def _reverse_moves(self, default_values_list=None, cancel=False):
        """
            This function is needed because the l10n_it_document_type is set only if no value are set when posting it
            But when reversing the move, the document type of the original move is copied and so it isn't recomputed.
        """
        # EXTENDS account
        reverse_moves = super()._reverse_moves(default_values_list, cancel)
        for move in reverse_moves:
            move.l10n_it_document_type = False
        return reverse_moves

    def _l10n_it_edi_import_invoice(self, invoice, data, is_new):
        res = super()._l10n_it_edi_import_invoice(invoice=invoice, data=data, is_new=is_new)
        if not res:
            return
        self = res

        #l10n_it_payment_method
        if payment_method := get_text(data['xml_tree'], '//DatiPagamento/DettaglioPagamento/ModalitaPagamento'):
            if payment_method in self.env['account.payment.method.line']._get_l10n_it_payment_method_selection_code():
                self.l10n_it_payment_method = payment_method

        return self

```

## File: models\account_payment_methode_line.py

```python
from odoo import fields, models

L10N_IT_PAYMENT_METHOD_SELECTION = [
    ('MP01', "MP01 - Cash"),
    ('MP02', "MP02 - Check"),
    ('MP03', "MP03 - Cashier's check"),
    ('MP04', "MP04 - Cash at the Treasury"),
    ('MP05', "MP05 - Wire transfer"),
    ('MP06', "MP06 - Promissory note"),
    ('MP07', "MP07 - Bank slip"),
    ('MP08', "MP08 - Payment card"),
    ('MP09', "MP09 - RID"),
    ('MP10', "MP10 - RID users"),
    ('MP11', "MP11 - Fast RID"),
    ('MP12', "MP12 - RIBA"),
    ('MP13', "MP13 - MAV"),
    ('MP14', "MP14 - Treasury receipt"),
    ('MP15', "MP15 - Transfer of special accounting accounts"),
    ('MP16', "MP16 - Bank direct debit"),
    ('MP17', "MP17 - Postal domiciliation"),
    ('MP18', "MP18 - Postal account slip"),
    ('MP19', "MP19 - SEPA Direct Debit"),
    ('MP20', "MP20 - SEPA Direct Debit CORE"),
    ('MP21', "MP21 - SEPA Direct Debit B2B"),
    ('MP22', "MP22 - Withholding from sums already collected"),
    ('MP23', "MP23 - PagoPA"),
]


class AccountPaymentMethodLine(models.Model):
    _inherit = "account.payment.method.line"

    l10n_it_payment_method = fields.Selection(
        selection=L10N_IT_PAYMENT_METHOD_SELECTION,
        string="Italian Payment Method",
        default='MP05',
    )

    def _get_l10n_it_payment_method_selection_code(self):
        return [payment_method[0] for payment_method in L10N_IT_PAYMENT_METHOD_SELECTION]

```

## File: models\l10n_it_document_type.py

```python
from odoo import fields, models


class L10nItDocumentType(models.Model):
    _name = 'l10n_it.document.type'
    _description = 'Italian Document Type'

    name = fields.Char(required=True, help='The document type name', translate=True)
    code = fields.Char(required=True)
    type = fields.Selection(
        selection=[
            ('sale', "Sale"),
            ('purchase', "Purchase"),
        ],
        required=True,
    )

    def _compute_display_name(self):
        for document_type in self:
            document_type.display_name = f"{document_type.code} - {document_type.name}"

```

## File: models\__init__.py

```python
from . import account_move
from . import account_payment_methode_line
from . import l10n_it_document_type

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_it_edi_ndd.access_l10n_it_document_type,access_l10n_it_document_type,l10n_it_edi_ndd.model_l10n_it_document_type,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_invoice_form_l10n_it" model="ir.ui.view">
        <field name="name">account.move.form.l10n.it</field>
        <field name="model">account.move</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="l10n_it_edi.account_invoice_form_l10n_it"/>
        <field name="arch" type="xml">
            <field name="l10n_it_ddt_id" position="after">
                <field name="l10n_it_document_type"
                       string="Document Type"
                       domain="[('type', '=', 'purchase' if move_type in ('in_invoice', 'in_receipt', 'in_refund') else 'sale')]"
                />
                <field name="l10n_it_payment_method" string="Payment Method"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\account_payment_method.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_form_l10n_it" model="ir.ui.view">
            <field name="name">account.journal.form</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.view_account_journal_form"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@id='inbound_payment_settings']/field[@name='inbound_payment_method_line_ids']/tree" position="inside">
                    <field name="l10n_it_payment_method" optional="hide" column_invisible="parent.country_code != 'IT'"/>
                </xpath>

                <xpath expr="//page[@id='outbound_payment_settings']/field[@name='outbound_payment_method_line_ids']/tree" position="inside">
                    <field name="l10n_it_payment_method" optional="hide" column_invisible="parent.country_code != 'IT'"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\l10n_it_document_type.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Useful for the search more of the field -->
    <record id="l10n_it_document_type_tree" model="ir.ui.view">
        <field name="name">Document Type Tree</field>
        <field name="model">l10n_it.document.type</field>
        <field name="arch" type="xml">
            <tree string="Document Type">
                <field name="code"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

</odoo>

```

