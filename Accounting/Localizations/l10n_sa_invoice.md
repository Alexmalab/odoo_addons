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
    'name': 'Saudi Arabia - Invoice',
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

        <xpath expr="//t[@t-set='address']" position="after">
            <t t-set="information_block">
                <p>
                    <img t-if="o.l10n_sa_qr_code_str"
                         style="display:block;"
                         t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s'%('QR', quote_plus(o.l10n_sa_qr_code_str), 200, 200)"/>
                </p>
            </t>
        </xpath>
        <xpath expr="//th[@name='th_total']//span[2]" position="attributes">
            <span>
                 <attribute name="class">d-none</attribute>
            </span>
        </xpath>
        <xpath expr="//th[@name='th_total']//span[2]" position="after">
            <span>
                Subtotal<br/>(inclusive of VAT)
            </span>
        </xpath>
        <xpath expr="//th[@name='th_total']//span" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//th[@name='th_total']//span" position="after">
            <span>
                المجموع شامل ضريبة القيمة المضافة
            </span>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span[2]" position="attributes">
            <span>
                <attribute name="class">d-none</attribute>
            </span>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span[2]" position="after">
            <span>
                Subtotal<br/>(exclusive of VAT)
            </span>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span" position="attributes">
            <span>
                <attribute name="class">d-none</attribute>
            </span>
        </xpath>
        <xpath expr="//th[@name='th_subtotal']//span" position="after">
            <span>
                المجموع الفرعي بدون الضريبة
            </span>
        </xpath>
        <xpath expr="//th[@name='th_taxes']//span" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//th[@name='th_taxes']//span" position="after">
            <span>
                نسبة الضريبة
            </span>
        </xpath>
        <xpath expr="//tr" position="attributes">
            <attribute name="style">font-size: 14px;</attribute>
        </xpath>
        <xpath expr="//span[@t-field='line.l10n_gcc_invoice_tax_amount']" position="attributes">
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </xpath>
        <xpath expr="//span[@t-field='line.price_unit']" position="attributes">
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]//strong" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//div[hasclass('clearfix')]//strong" position="after">
            <strong>
                Invoice Taxable Amount
                /<br/>
                المبلغ الخاضع للضريبة غير شامل ضريبة القيمة المضافة
            </strong>
        </xpath>
        <xpath expr="//t[@t-call='account.tax_groups_totals']" position="attributes">
            <attribute name="t-if">0</attribute>
        </xpath>
        <xpath expr="//t[@t-call='account.tax_groups_totals']" position="after">
            <t t-foreach="tax_totals['groups_by_subtotal'][subtotal_to_show]" t-as="amount_by_group">
                <t t-set="arabic_tax_group_name" t-value="json.loads(o_sec.tax_totals_json)['groups_by_subtotal'][json.loads(o_sec.tax_totals_json)['subtotals'][subtotal_index]['name']][amount_by_group_index]['tax_group_name']"/>
                <tr>
                    <t t-if="len(tax_totals['groups_by_subtotal'][subtotal_to_show]) > 1 or (tax_totals['amount_untaxed'] != amount_by_group['tax_group_base_amount'])">
                        <td>
                            <span t-esc="amount_by_group['tax_group_name']"/>
                            <span t-if="arabic_tax_group_name != amount_by_group['tax_group_name']" class="text-nowrap">/
                                <t t-esc="arabic_tax_group_name"/>
                            </span>
                            <span class="text-nowrap"> on
                                <t t-esc="amount_by_group['formatted_tax_group_base_amount']"/>
                            </span>
                        </td>
                        <td class="text-right o_price_total">
                            <span class="text-nowrap" t-esc="amount_by_group['formatted_tax_group_amount']"/>
                        </td>
                    </t>
                    <t t-else="">
                        <td>
                            <span class="text-nowrap" t-esc="amount_by_group['tax_group_name']"/>
                            <span t-if="arabic_tax_group_name != amount_by_group['tax_group_name']" class="text-nowrap">/
                                <t t-esc="arabic_tax_group_name"/>
                            </span>
                        </td>
                        <td class="text-right o_price_total">
                            <span class="text-nowrap" t-esc="amount_by_group['formatted_tax_group_amount']" />
                        </td>
                    </t>
                </tr>
            </t>
        </xpath>
        <xpath expr="//tr[hasclass('o_total')]//strong" position="attributes">
            <attribute name="class">d-none</attribute>
        </xpath>
        <xpath expr="//tr[hasclass('o_total')]//strong" position="after">
            <strong>
                Invoice Total (inclusive of VAT)
                /
                إجمالي قيمة الفاتورة شامل ضريبة القيمة المضافة
            </strong>
        </xpath>
        <xpath expr="//div[@name='invoice_date']//span" position="before">
            <span t-if="o.l10n_sa_confirmation_datetime" t-field="o.l10n_sa_confirmation_datetime"/>
        </xpath>
        <xpath expr="//div[@name='invoice_date']//span[@t-field='o.invoice_date']" position="attributes">
            <attribute name="t-if">not o.l10n_sa_confirmation_datetime</attribute>
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

