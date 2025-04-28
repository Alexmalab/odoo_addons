# Odoo Module: l10n_se_ocr

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name' : 'Sweden - Structured Communication OCR',
    'version' : '1.0',
    'author': 'XCLUDE',
    'website': 'https://www.xclude.se',
    'category': 'Accounting/Localizations',
    'description': """
Add Structured Communication to Customer Invoices and Vendor Bill.
------------------------------------------------------------------

Using OCR structured communication simplifies the reconciliation between invoices and payments.

For Customer Invoicing support for OCR level 1 to 4. The OCR number can be based on partner or
the invoice.

For Vendor Bill support, Default Vendor Specific OCR and validation for OCR are added.
    """,
    'depends': ['l10n_se'],
    'data': [
        'views/partner_view.xml',
        'views/account_journal_view.xml'
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[('se_ocr2', 'Sweden OCR Level 1 & 2'), ('se_ocr3', 'Sweden OCR Level 3'), ('se_ocr4', 'Sweden OCR Level 4')], ondelete={'se_ocr2': 'set default', 'se_ocr3': 'set default', 'se_ocr4': 'set default'})
    l10n_se_invoice_ocr_length = fields.Integer(string='OCR Number Length', help="Total length of OCR Reference Number including checksum.", default=6)

    @api.constrains('l10n_se_invoice_ocr_length')
    def _check_l10n_se_invoice_ocr_length(self):
        if self.l10n_se_invoice_ocr_length < 6:
            return ValidationError(_('OCR Reference Number length need to be greater than 5. Please correct settings under invoice journal settings.'))

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from stdnum import luhn


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_invoice_reference_se_ocr2(self, reference):
        self.ensure_one()
        return reference + luhn.calc_check_digit(reference)

    def _get_invoice_reference_se_ocr3(self, reference):
        self.ensure_one()
        reference = reference + str(len(reference) + 2)[:1]
        return reference + luhn.calc_check_digit(reference)

    def _get_invoice_reference_se_ocr4(self, reference):
        self.ensure_one()

        ocr_length = self.journal_id.l10n_se_invoice_ocr_length

        if len(reference) + 1 > ocr_length:
            raise UserError(_("OCR Reference Number length is greater than allowed. Allowed length in invoice journal setting is %s.") % str(ocr_length))

        reference = reference.rjust(ocr_length - 1, '0')
        return reference + luhn.calc_check_digit(reference)


    def _get_invoice_reference_se_ocr2_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr2(str(self.id))

    def _get_invoice_reference_se_ocr3_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr3(str(self.id))

    def _get_invoice_reference_se_ocr4_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr4(str(self.id))

    def _get_invoice_reference_se_ocr2_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr2(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    def _get_invoice_reference_se_ocr3_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr3(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    def _get_invoice_reference_se_ocr4_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr4(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        """ If Vendor Bill and Vendor OCR is set, add it. """
        if self.partner_id and self.move_type == 'in_invoice' and self.partner_id.l10n_se_default_vendor_payment_ref:
            self.payment_reference = self.partner_id.l10n_se_default_vendor_payment_ref
        return super(AccountMove, self)._onchange_partner_id()

    @api.onchange('payment_reference')
    def _onchange_payment_reference(self):
        """ If Vendor Bill and Payment Reference is changed check validation. """
        if self.partner_id and self.move_type == 'in_invoice' and self.partner_id.l10n_se_check_vendor_ocr:
            reference = self.payment_reference
            try:
                luhn.validate(reference)
            except: 
                return {'warning': {'title': _('Warning'), 'message': _('Vendor require OCR Number as payment reference. Payment reference isn\'t a valid OCR Number.')}}
        return super(AccountMove, self)._onchange_payment_reference()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from stdnum import luhn


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_se_check_vendor_ocr = fields.Boolean(string='Check Vendor OCR', help='This Vendor uses OCR Number on their Vendor Bills.')
    l10n_se_default_vendor_payment_ref = fields.Char(string='Default Vendor Payment Ref', help='If set, the vendor uses the same Default Payment Reference or OCR Number on all their Vendor Bills.')

    @api.onchange('l10n_se_default_vendor_payment_ref')
    def onchange_l10n_se_default_vendor_payment_ref(self):
        if not self.l10n_se_default_vendor_payment_ref == "" and self.l10n_se_check_vendor_ocr:
            reference = self.l10n_se_default_vendor_payment_ref
            try:
                luhn.validate(reference)
            except: 
                return {'warning': {'title': _('Warning'), 'message': _('Default vendor OCR number isn\'t a valid OCR number.')}}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import account_journal
from . import res_partner
```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_se_ocr_form" model="ir.ui.view">
            <field name="name">account.journal.se.ocr.form</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.view_account_journal_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='invoice_reference_model']" position="after">
                    <field name="l10n_se_invoice_ocr_length" attrs="{'invisible': [('invoice_reference_model', '!=', 'se_ocr4')]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_partner_ocr_form" model="ir.ui.view">
            <field name="name">res.partner.ocr.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <group name="accounting_entries" position="after">
                    <group string="Payment Options Sweden" name="payment_options">
                        <field name="l10n_se_check_vendor_ocr"/>
                        <field name="l10n_se_default_vendor_payment_ref"/>
                    </group>
                </group>
            </field>
        </record>
    </data>
</odoo>

```

