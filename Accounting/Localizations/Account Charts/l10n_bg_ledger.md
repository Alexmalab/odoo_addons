# Odoo Module: l10n_bg_ledger

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Bulgaria - Report ledger',
    'icon': '/account/static/description/l10n.png',
    'countries': ['bg'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Report ledger for Bulgaria
    """,
    'depends': [
        'l10n_bg'
    ],
    'data': [
        'views/account_journal_views.xml',
        'views/account_move_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_journal.py

```python
from odoo import fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    l10n_bg_customer_invoice = fields.Selection(string="Customer Invoices", selection='_l10n_bg_document_type_selection_values', default='01')
    l10n_bg_credit_notes = fields.Selection(string="Credit Notes", selection='_l10n_bg_document_type_selection_values', default='03')
    l10n_bg_debit_notes = fields.Selection(string="Debit Notes", selection='_l10n_bg_document_type_selection_values', default='02')

    def _l10n_bg_document_type_selection_values(self):
        return [
            ('01', '01 - Invoice'),
            ('02', '02 - Debit notice'),
            ('03', '03 - Credit notice'),
            ('07', '07 - Customs declaration'),
            ('09', '09 - Protocol or another document'),
            ('11', '11 - Invoice - cash account'),
            ('12', '12 - Debit notification - cash account'),
            ('13', '13 - Credit notification - cash account'),
            ('81', '81 - Report for the sales carried out'),
            ('82', '82 - Report for the sales carried out by a special levying procedure'),
            ('91', '91 - Protocol of due tax under Art. 151c, Para 3 of the Act'),
            ('93', '93 - Protocol of due tax under Art. 151c, Para 7 of the Act with a recipient being a person not applying the special regime'),
            ('94', '94 - Protocol of due tax under Art. 151c, Para 7 of the Act with a recipient being a person applying the special regime'),
        ]

```

## File: models\account_move.py

```python
from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_bg_document_type = fields.Selection(
        string="Document Type (BG)",
        selection='_l10n_bg_document_type_selection_values',
        compute='_compute_l10n_bg_document_type',
        readonly=False,
        store=True,
        copy=False,
    )
    l10n_bg_document_number = fields.Char(string="Document Number (BG)", compute='_compute_l10n_bg_document_number')
    l10n_bg_exemption_reason = fields.Selection(string="Exemption reason (BG)", selection=[
        ('01', '01 - A delivery under Part 1 of Appendix 2 of LVAT'),
        ('02', '02 - A delivery under Part 2 of Appendix 2 of LVAT'),
        ('03', '03 - Import under Appendix 3 of VAT act'),
    ])

    def _l10n_bg_document_type_selection_values(self):
        return [
            ('01', '01 - Invoice'),
            ('02', '02 - Debit notice'),
            ('03', '03 - Credit notice'),
            ('07', '07 - Customs declaration'),
            ('09', '09 - Protocol or another document'),
            ('11', '11 - Invoice - cash account'),
            ('12', '12 - Debit notification - cash account'),
            ('13', '13 - Credit notification - cash account'),
            ('81', '81 - Report for the sales carried out'),
            ('82', '82 - Report for the sales carried out by a special levying procedure'),
            ('91', '91 - Protocol of due tax under Art. 151c, Para 3 of the Act'),
            ('93', '93 - Protocol of due tax under Art. 151c, Para 7 of the Act with a recipient being a person not applying the special regime'),
            ('94', '94 - Protocol of due tax under Art. 151c, Para 7 of the Act with a recipient being a person applying the special regime'),
        ]

    @api.depends('journal_id', 'move_type')
    def _compute_l10n_bg_document_type(self):
        for move in self:
            if move.journal_id:
                if 'debit_origin_id' in self._fields and move.debit_origin_id:
                    move.l10n_bg_document_type = move.journal_id.l10n_bg_debit_notes
                elif move.move_type in ('out_invoice', 'in_invoice'):
                    move.l10n_bg_document_type = move.journal_id.l10n_bg_customer_invoice
                elif move.move_type in ('in_refund', 'out_refund'):
                    move.l10n_bg_document_type = move.journal_id.l10n_bg_credit_notes

    @api.depends('l10n_bg_document_type', 'move_type', 'state', 'ref', 'name')
    def _compute_l10n_bg_document_number(self):
        for move in self:
            if move.state == 'draft':
                move.l10n_bg_document_number = ""
            elif move.is_sale_document(include_receipts=True):
                move.l10n_bg_document_number = move.name
            else:
                move.l10n_bg_document_number = move.ref

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move
from . import account_journal

```

## File: views\account_journal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_bg_journal_view_form" model="ir.ui.view">
        <field name="name">l10n_bg.journal.view.form</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.view_account_journal_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='bank_account_number']" position="before">
                <group string="Bulgaria VAT Document Types" invisible="type not in ('sale', 'purchase')">
                    <field name="l10n_bg_customer_invoice"/>
                    <field name="l10n_bg_credit_notes"/>
                    <field name="l10n_bg_debit_notes"/>
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
    <record id="l10n_bg_move_view_form" model="ir.ui.view">
        <field name="name">l10n_bg.move.view.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='sale_info_group']" position="after">
                <group string="Bulgaria VAT Document" name="l10n_bg_documents">
                    <field string="Document Type" name="l10n_bg_document_type"/>
                    <field string="Document Number" name="l10n_bg_document_number"/>
                    <field string="Exemption reason" name="l10n_bg_exemption_reason"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="l10n_bg_move_view_tree" model="ir.ui.view">
        <field name="name">l10n_bg.move.view.tree</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_invoice_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field string="Document Number" name="l10n_bg_document_number" optional="show"/>
                <field string="Document Type" name="l10n_bg_document_type" optional="show"/>
            </xpath>
        </field>
    </record>

    <record id="l10n_bg_move_view_filter" model="ir.ui.view">
        <field name="name">l10n_bg.move.view.filter</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='group_by_company']" position="after">
                <filter string="Document Type" name="l10n_bg_document_type_groupby" context="{'group_by': 'l10n_bg_document_type'}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

