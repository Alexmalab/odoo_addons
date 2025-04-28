# Odoo Module: l10n_sa_invoice

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'K.S.A - Invoice',
    'version': '1.0.0',
    'author': 'Odoo S.A.',
    'category': 'Accounting/Localizations',
    'license': 'LGPL-3',
    'description': """
    Invoices for the Kingdom of Saudi Arabia
""",
    'depends': ['l10n_sa', 'l10n_gcc_invoice'],
    'data': [
        'views/view_move_form.xml',
        'views/report_invoice.xml',
    ],
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_repr


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_sa_delivery_date = fields.Date(string='Delivery Date', default=fields.Date.context_today, copy=False,
                                        readonly=True, states={'draft': [('readonly', False)]},
                                        help="In case of multiple deliveries, you should take the date of the latest one. ")
    l10n_sa_show_delivery_date = fields.Boolean(compute='_compute_show_delivery_date')
    l10n_sa_qr_code_str = fields.Char(string='Zatka QR Code', compute='_compute_qr_code_str')
    l10n_sa_confirmation_datetime = fields.Datetime(string='Confirmation Date', readonly=True, copy=False)

    @api.depends('country_code', 'move_type')
    def _compute_show_delivery_date(self):
        for move in self:
            move.l10n_sa_show_delivery_date = move.country_code == 'SA' and move.move_type in ('out_invoice', 'out_refund')

    @api.depends('amount_total_signed', 'amount_tax_signed', 'l10n_sa_confirmation_datetime', 'company_id', 'company_id.vat')
    def _compute_qr_code_str(self):
        """ Generate the qr code for Saudi e-invoicing. Specs are available at the following link at page 23
        https://zatca.gov.sa/ar/E-Invoicing/SystemsDevelopers/Documents/20210528_ZATCA_Electronic_Invoice_Security_Features_Implementation_Standards_vShared.pdf
        """
        def get_qr_encoding(tag, field):
            company_name_byte_array = field.encode()
            company_name_tag_encoding = tag.to_bytes(length=1, byteorder='big')
            company_name_length_encoding = len(company_name_byte_array).to_bytes(length=1, byteorder='big')
            return company_name_tag_encoding + company_name_length_encoding + company_name_byte_array

        for record in self:
            qr_code_str = ''
            if record.l10n_sa_confirmation_datetime and record.company_id.vat:
                seller_name_enc = get_qr_encoding(1, record.company_id.display_name)
                company_vat_enc = get_qr_encoding(2, record.company_id.vat)
                time_sa = fields.Datetime.context_timestamp(self.with_context(tz='Asia/Riyadh'), record.l10n_sa_confirmation_datetime)
                timestamp_enc = get_qr_encoding(3, time_sa.isoformat())
                invoice_total_enc = get_qr_encoding(4, float_repr(abs(record.amount_total_signed), 2))
                total_vat_enc = get_qr_encoding(5, float_repr(abs(record.amount_tax_signed), 2))

                str_to_encode = seller_name_enc + company_vat_enc + timestamp_enc + invoice_total_enc + total_vat_enc
                qr_code_str = base64.b64encode(str_to_encode).decode()
            record.l10n_sa_qr_code_str = qr_code_str

    def _post(self, soft=True):
        res = super()._post(soft)
        for record in self:
            if record.country_code == 'SA' and record.move_type in ('out_invoice', 'out_refund'):
                if not record.l10n_sa_show_delivery_date:
                    raise UserError(_('Delivery Date cannot be empty'))
                self.write({
                    'l10n_sa_confirmation_datetime': fields.Datetime.now()
                })
        return res

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="arabic_english_invoice" inherit_id="l10n_gcc_invoice.arabic_english_invoice">
        <xpath expr="//div[@name='due_date']" position="after">
            <div class="row" t-if="o.l10n_sa_delivery_date" name="delivery_date">
                <div class="col-2">
                    <strong style="white-space:nowrap">Delivery Date:
                    </strong>
                </div>
                <div class="col-2">
                    <span t-field="o.l10n_sa_delivery_date"/>
                </div>
                <div class="col-2 text-right">
                    <strong style="white-space:nowrap">:
                        تاريخ التوصيل
                    </strong>
                </div>
            </div>
        </xpath>
    </template>

    <template id="external_layout_standard" inherit_id="l10n_gcc_invoice.external_layout_standard">
        <xpath expr="//div[@name='qr_code']" position="inside">
            <img t-if="o.l10n_sa_qr_code_str"
                 style="display:block;margin:10% auto 0 auto;"
                 t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s'%('QR', o.l10n_sa_qr_code_str, 150, 150)"/>
        </xpath>
    </template>

</odoo>

```

## File: views\view_move_form.xml

```xml
<odoo>
    <data>
        <record id="view_move_form" model="ir.ui.view">
            <field name="name">account.move.deliver_date</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <field name="invoice_date" position="after">
                    <field name="l10n_sa_show_delivery_date" invisible="1"/>
                    <field name="l10n_sa_delivery_date" attrs="{'invisible': [('l10n_sa_show_delivery_date', '=', False)], 'required': [('l10n_sa_show_delivery_date', '=', True)]}"/>
                </field>
            </field>
        </record>
    </data>
</odoo>
```

