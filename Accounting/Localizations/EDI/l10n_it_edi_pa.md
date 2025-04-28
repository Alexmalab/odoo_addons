# Odoo Module: l10n_it_edi_pa

Category: Accounting/Localizations/EDI

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
    'name': 'Italy - E-invoicing (PA)',
    'version': '0.1',
    'depends': [
        'l10n_it_edi'
    ],
    'auto_install': ['l10n_it_edi'],
    'author': 'Odoo',
    'description': """
Public Administration partners flow handling for the E-invoice implementation for Italy.

    Several more fields are required for invoicing Public Administration businesses.
    The Origin Document is to be exported in the XML when invoicing the Public Administration,
    It can be a Contract, an Agreement, a Purchase Order, a Linked Invoice or a Down Payment,
    it will need the CIG and CUP fields which are mandatory.
    They both serve the purpose to trace public funds being invested on purchases.
    CIG is the Tender Unique Identifier, CUP identifies the Public Project of Investment.
    """,
    'category': 'Accounting/Localizations/EDI',
    'website': 'https://www.odoo.com/documentation/16.0/applications/finance/accounting/fiscal_localizations/localizations/italy.html',
    'data': [
        'views/account_move_view.xml',
        'views/report_invoice.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\account_edi_format.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    def _l10n_it_edi_check_ordinary_invoice_configuration(self, invoice):
        errors = super()._l10n_it_edi_check_ordinary_invoice_configuration(invoice)
        if invoice._is_commercial_partner_pa():
            if not invoice.l10n_it_origin_document_type:
                errors.append(_("This invoice targets the Public Administration, please fill out"
                              " Origin Document Type field in the Electronic Invoicing tab."))
            if invoice.l10n_it_origin_document_date and invoice.l10n_it_origin_document_date > fields.Date.today():
                errors.append(_("The Origin Document Date cannot be in the future."))
        return errors

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_origin_document_type = fields.Selection(
        string="Origin Document Type",
        selection=[('purchase_order', 'Purchase Order'), ('contract', 'Contract'), ('agreement', 'Agreement')],
        readonly=True, states={'draft': [('readonly', False)]}, copy=False)
    l10n_it_origin_document_name = fields.Char(
        string="Origin Document Name",
        readonly=True, states={'draft': [('readonly', False)]}, copy=False)
    l10n_it_origin_document_date = fields.Date(
        string="Origin Document Date",
        readonly=True, states={'draft': [('readonly', False)]}, copy=False)
    l10n_it_cig = fields.Char(
        string="CIG",
        readonly=True, states={'draft': [('readonly', False)]}, copy=False,
        help="Tender Unique Identifier")
    l10n_it_cup = fields.Char(
        string="CUP",
        readonly=True, states={'draft': [('readonly', False)]}, copy=False,
        help="Public Investment Unique Identifier")
    # Technical field for showing the above fields or not
    l10n_it_partner_pa = fields.Boolean(compute='_compute_l10n_it_partner_pa')

    @api.depends('commercial_partner_id.l10n_it_pa_index', 'company_id')
    def _compute_l10n_it_partner_pa(self):
        for move in self:
            move.l10n_it_partner_pa = (move.country_code == 'IT' and move.commercial_partner_id.l10n_it_pa_index and
                                       len(move.commercial_partner_id.l10n_it_pa_index) == 6)

    def _prepare_fatturapa_export_values(self):
        """Add origin document features."""
        template_values = super()._prepare_fatturapa_export_values()
        template_values.update({
            'origin_document_type': self.l10n_it_origin_document_type,
            'origin_document_name': self.l10n_it_origin_document_name,
            'origin_document_date': self.l10n_it_origin_document_date,
            'cig': self.l10n_it_cig,
            'cup': self.l10n_it_cup,
        })
        return template_values

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import account_edi_format

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_invoice_form_l10n_it_pa" model="ir.ui.view">
        <field name="name">account.move.form.l10n.it.pa</field>
        <field name="model">account.move</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="l10n_it_edi.account_invoice_form_l10n_it"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//page[@name='electronic_invoicing']/group" position="inside">
                <field name="l10n_it_partner_pa" invisible="1"/>
                <group attrs="{'invisible': [('l10n_it_partner_pa', '=', False)]}">
                    <field name="l10n_it_origin_document_type"/>
                    <field name="l10n_it_origin_document_name"/>
                    <field name="l10n_it_origin_document_date"/>
                    <field name="l10n_it_cig"/>
                    <field name="l10n_it_cup"/>
                </group>
            </xpath>
        </data>
        </field>
    </record>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <div name="comment" position="before">
            <div t-if="o.l10n_it_origin_document_type" name="pa_fields">
                <b><span t-field="o.l10n_it_origin_document_type"/></b>: <span t-field="o.l10n_it_origin_document_name"/><br/>
                <t t-if="o.l10n_it_origin_document_date"><b>Document Date: </b><span t-field="o.l10n_it_origin_document_date"/><br/></t>
                <t t-if="o.l10n_it_cig"><b>CIG: </b><span t-field="o.l10n_it_cig"/><br/></t>
                <t t-if="o.l10n_it_cup"><b>CUP: </b><span t-field="o.l10n_it_cup"/><br/></t>
            </div>
        </div>
    </template>
</odoo>

```

