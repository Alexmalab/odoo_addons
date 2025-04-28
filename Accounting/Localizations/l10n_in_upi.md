# Odoo Module: l10n_in_upi

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
    'name': 'Indian - UPI',
    'version': '1.0',
    'description': """
Invoice with UPI QR code
=========================
This module adds QR code in invoice report for UPI payment allowing to make payment via any UPI app.

To print UPI Qr code add UPI id in company and tick "QR Codes" in configuration
  """,
    'category': 'Accounting/Localizations',
    'depends': ['l10n_in'],
    'data': ['views/res_company_views.xml'],
    'license': 'LGPL-3',
    'auto_install': True,
}

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64

from odoo import models
from odoo.tools.image import image_data_uri


class AccountMove(models.Model):
    _inherit = "account.move"

    def _generate_qr_code(self, silent_errors=False):
        self.ensure_one()
        if self.company_id.country_code == 'IN' and self.is_sale_document(include_receipts=True):
            payment_url = 'upi://pay?pa=%s&pn=%s&am=%s&tr=%s&tn=%s' % (
                self.company_id.l10n_in_upi_id,
                self.company_id.name,
                self.amount_residual,
                self.payment_reference or self.name,
                ("Payment for %s" % self.name))
            barcode = self.env['ir.actions.report'].barcode(barcode_type="QR", value=payment_url, width=120, height=120)
            return image_data_uri(base64.b64encode(barcode))
        return super()._generate_qr_code(silent_errors)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Company(models.Model):
    _inherit = 'res.company'

    l10n_in_upi_id = fields.Char(string="UPI Id")

```

## File: models\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_invoice
from . import res_company

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_in_upi</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="l10n_in_upi_id" attrs="{'invisible': [('country_code', '!=', 'IN')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

