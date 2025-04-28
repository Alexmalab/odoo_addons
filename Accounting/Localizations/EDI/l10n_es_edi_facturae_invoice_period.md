# Odoo Module: l10n_es_edi_facturae_invoice_period

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Spain - Facturae EDI - Invoice Period',
    'version': '1.0',
    'description': """
    Patch module to add the missing Invoice Period in the Facturae EDI.
    """,
    'license': 'LGPL-3',
    'category': 'Accounting/Localizations/EDI',
    'depends': [
        'l10n_es_edi_facturae',
    ],
    'data': [
        'views/account_move_views.xml',
    ],
    'auto_install': True,
}

```

## File: models\account_move.py

```python
from odoo import fields, models

class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_es_invoicing_period_start_date = fields.Date(string="Invoice Period Start Date")
    l10n_es_invoicing_period_end_date = fields.Date(string="Invoice Period End Date")

    def _l10n_es_edi_facturae_export_facturae(self):
        # EXTENDS l10n_es_edi_facturae
        template_values, signature_values = super()._l10n_es_edi_facturae_export_facturae()

        invoicing_period = {
            'StartDate': self.l10n_es_invoicing_period_start_date,
            'EndDate': self.l10n_es_invoicing_period_end_date,
        } if self.l10n_es_invoicing_period_start_date and self.l10n_es_invoicing_period_end_date else None

        template_values['Invoices'][0]['InvoiceIssueData']['InvoicingPeriod'] = invoicing_period
        return template_values, signature_values

```

## File: models\__init__.py

```python
from . import account_move

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_move_form" model="ir.ui.view">
            <field name="name">account.move.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='other_info']" position="inside">
                        <group string="Factura-e"
                               name="l10n_es_facturae_invoicing_period">
                            <field name="l10n_es_invoicing_period_start_date"/>
                            <field name="l10n_es_invoicing_period_end_date"/>
                        </group>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

