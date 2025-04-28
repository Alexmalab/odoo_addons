# Odoo Module: l10n_es_edi_tbai_multi_refund

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    "name": "TicketBAI multi refund",
    "summary": "Link one refund with multiple invoices",
    "version": "1.0",
    "category": "Accounting/Localizations/EDI",
    "license": "LGPL-3",
    "auto_install": True,
    "depends": [
        "l10n_es_edi_tbai",
    ],
    "data": [
        "data/template_LROE_bizkaia.xml",
        "views/account_move_view.xml",
    ],
}

```

## File: data\template_LROE_bizkaia.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<data>
    <template id="template_LROE_240_inner_recibidas" inherit_id="l10n_es_edi_tbai.template_LROE_240_inner_recibidas">
        <xpath expr="//FacturasRectificadasSustituidas" position="attributes">
            <attribute name="t-if">credit_note_invoices</attribute>
        </xpath>

        <xpath expr="//IDFacturaRectificadaSustituida" position="attributes">
            <attribute name="t-foreach">credit_note_invoices</attribute>
            <attribute name="t-as">credit_note_invoice</attribute>
        </xpath>
    </template>

    <template id="template_invoice_factura" inherit_id="l10n_es_edi_tbai.template_invoice_factura">
        <xpath expr="//FacturasRectificadasSustituidas" position="attributes">
            <attribute name="t-if">credit_note_invoices</attribute>
        </xpath>

        <xpath expr="//IDFacturaRectificadaSustituida" position="attributes">
            <attribute name="t-foreach">credit_note_invoices</attribute>
            <attribute name="t-as">credit_note_invoice</attribute>
        </xpath>
    </template>
</data>
```

## File: models\account_edi_format.py

```python
from odoo import models


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    def _l10n_es_tbai_refunded_invoices(self, invoice):
        return super()._l10n_es_tbai_refunded_invoices(invoice) | invoice.l10n_es_tbai_reversed_ids

    def _l10n_es_tbai_get_in_invoice_values_batuz(self, invoice):
        values = super()._l10n_es_tbai_get_in_invoice_values_batuz(invoice)
        credit_notes = values.pop('credit_note_invoice', self.env['account.move']) | invoice.l10n_es_tbai_reversed_ids
        if credit_notes:
            values['credit_note_invoices'] = credit_notes
        return values

    def _l10n_es_tbai_get_invoice_values(self, invoice, cancel):
        values = super()._l10n_es_tbai_get_invoice_values(invoice, cancel)
        credit_notes = values.pop('credit_note_invoice', self.env['account.move']) | invoice.l10n_es_tbai_reversed_ids
        if credit_notes:
            values['credit_note_invoices'] = credit_notes
        return values
```

## File: models\account_move.py

```python
from odoo import fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_es_tbai_reversed_ids = fields.Many2many(
        'account.move', 'account_move_tbai_reversed_moves', 'refund_id', 'reversed_move_id',
        string="Refunded Invoices",
        domain="[('move_type', '=', 'in_invoice' if move_type == 'in_refund' else 'out_invoice'), ('commercial_partner_id', '=', commercial_partner_id)]",
        help="In the case where a refund has multiple original invoices, you can set them here. ",
    )
```

## File: models\__init__.py

```python
from . import account_edi_format
from . import account_move

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<data>
    <record id="view_move_form" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_es_edi_tbai_multi_refund</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <field name="l10n_es_tbai_refund_reason" position='after'>
                <field name="l10n_es_tbai_reversed_ids" invisible="move_type not in ('in_refund', 'out_refund')" widget="many2many_tags" options="{'no_create': True}" />
            </field>
        </field>
    </record>
</data>

```

