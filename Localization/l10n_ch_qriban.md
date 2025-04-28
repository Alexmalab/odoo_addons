# Odoo Module: l10n_ch_qriban

Category: Localization

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
    'name': "Switzerland - QR-IBAN",
    'description': """
Swiss localization
==================
Added a QR-IBAN field on bank account.
If this field is empty, but the bank account number itself is a valid QR-IBAN number, it will still be used as QR-IBAN as before.  
But if you fill in the new QR-IBAN field, that one will be used as the QR-IBAN.  This should help for reconciliation as 
on the bank statements, the old IBAN code is still used.  
    """,
    'version': '1.0',
    'author': 'Odoo S.A',
    'category': 'Localization',
    'depends': ['l10n_ch'],
    'data': [
        'views/res_bank_views.xml',
        'views/swissqr_report.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\res_bank.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.base_iban.models.res_partner_bank import normalize_iban, pretty_iban, validate_iban
from odoo.addons.base.models.res_bank import sanitize_account_number
from odoo.exceptions import ValidationError

class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    l10n_ch_qr_iban = fields.Char(string='QR-IBAN', help="Put the QR-IBAN here for your own bank accounts.  That way, you can "
                                                         "still use the main IBAN in the Account Number while you will see the "
                                                         "QR-IBAN for the barcode.  ")

    def _validate_qr_iban(self, qr_iban):
        # Check first if it's a valid IBAN.
        validate_iban(qr_iban)
        # We sanitize first so that _check_qr_iban_range() can extract correct IID from IBAN to validate it.
        sanitized_qr_iban = sanitize_account_number(qr_iban)
        # Now, check if it's valid QR-IBAN (based on its IID).
        if not self._check_qr_iban_range(sanitized_qr_iban):
            raise ValidationError(_("QR-IBAN '%s' is invalid.") % qr_iban)
        return True

    @api.model
    def create(self, vals):
        if vals.get('l10n_ch_qr_iban'):
            self._validate_qr_iban(vals['l10n_ch_qr_iban'])
            vals['l10n_ch_qr_iban'] = pretty_iban(normalize_iban(vals['l10n_ch_qr_iban']))
        return super().create(vals)

    def write(self, vals):
        if vals.get('l10n_ch_qr_iban'):
            self._validate_qr_iban(vals['l10n_ch_qr_iban'])
            vals['l10n_ch_qr_iban'] = pretty_iban(normalize_iban(vals['l10n_ch_qr_iban']))
        return super().write(vals)

    def _is_qr_iban(self):
        return super(ResPartnerBank, self)._is_qr_iban() or self.l10n_ch_qr_iban

    def _prepare_swiss_code_url_vals(self, amount, currency_name, debtor_partner, reference_type, reference, comment):
        qr_code_vals = super()._prepare_swiss_code_url_vals(amount, currency_name, debtor_partner, reference_type, reference, comment)
        # If there is a QR IBAN we use it for the barcode instead of the account number
        if self.l10n_ch_qr_iban:
            qr_code_vals[3] = sanitize_account_number(self.l10n_ch_qr_iban)
        return qr_code_vals

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_bank

```

## File: views\res_bank_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_bank_form" model="ir.ui.view">
        <field name="name">l10n_ch_qr.res.partner.bank.form</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="l10n_ch.isr_partner_bank_form"/>
        <field name="arch" type="xml">
            <xpath expr="//label[@for='l10n_ch_postal']" position="before">
                <field name="l10n_ch_qr_iban" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
            </xpath>
        </field>
    </record>

    <!-- Setup wizard view -->
    <record id="setup_bank_account_wizard_qr_inherit" model="ir.ui.view">
        <field name="name">account.setup.bank.manual.config.form.ch.qr.inherit</field>
        <field name="model">account.setup.bank.manual.config</field>
        <field name="inherit_id" ref="l10n_ch.setup_bank_account_wizard_inherit"/>
        <field name="arch" type="xml">
            <field name="l10n_ch_postal" position="after">
                <field name="l10n_ch_qr_iban" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\swissqr_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="l10n_ch_swissqr_qriban_template" inherit_id="l10n_ch.l10n_ch_swissqr_template">
        <xpath expr="//div[@id='receipt_indication_zone']/div[hasclass('swissqr_text')]/span[@t-field='o.invoice_partner_bank_id.acc_number']" position="replace">
            <span class="content" t-field="o.invoice_partner_bank_id.acc_number" t-if="not o.invoice_partner_bank_id.l10n_ch_qr_iban"/>
            <span class="content" t-field="o.invoice_partner_bank_id.l10n_ch_qr_iban" t-if="o.invoice_partner_bank_id.l10n_ch_qr_iban"/>
        </xpath>
        <xpath expr="//div[@id='indications_zone']/div[hasclass('swissqr_text')]/span[@t-field='o.invoice_partner_bank_id.acc_number']" position="replace">
            <span class="content" t-field="o.invoice_partner_bank_id.acc_number" t-if="not o.invoice_partner_bank_id.l10n_ch_qr_iban"/>
            <span class="content" t-field="o.invoice_partner_bank_id.l10n_ch_qr_iban" t-if="o.invoice_partner_bank_id.l10n_ch_qr_iban"/>
        </xpath>
    </template>
</odoo>

```

