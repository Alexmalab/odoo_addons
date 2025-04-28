# Odoo Module: l10n_din5008_sale

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'DIN 5008 - Sale',
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_din5008',
        'sale',
    ],
    'data': [
        'report/din5008_sale_order_layout.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\sale.py

```python
from odoo import models, fields, _
from odoo.tools import format_date


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    l10n_din5008_template_data = fields.Binary(compute='_compute_l10n_din5008_template_data')
    l10n_din5008_document_title = fields.Char(compute='_compute_l10n_din5008_document_title')
    l10n_din5008_addresses = fields.Binary(compute='_compute_l10n_din5008_addresses', exportable=False)

    def _compute_l10n_din5008_template_data(self):
        for record in self:
            record.l10n_din5008_template_data = data = []
            if record.state in ('draft', 'sent'):
                if record.name:
                    data.append((_("Quotation No."), record.name))
                if record.date_order:
                    data.append((_("Quotation Date"), format_date(self.env, record.date_order)))
                if record.validity_date:
                    data.append((_("Expiration"), format_date(self.env, record.validity_date)))
            else:
                if record.name:
                    data.append((_("Order No."), record.name))
                if record.date_order:
                    data.append((_("Order Date"), format_date(self.env, record.date_order)))
            if record.client_order_ref:
                data.append((_('Customer Reference'), record.client_order_ref))
            if record.user_id:
                data.append((_("Salesperson"), record.user_id.name))
            if 'incoterm' in record._fields and record.incoterm:
                data.append((_("Incoterm"), record.incoterm.code))

    def _compute_l10n_din5008_document_title(self):
        for record in self:
            if self._context.get('proforma'):
                record.l10n_din5008_document_title = _('Pro Forma Invoice')
            elif record.state in ('draft', 'sent'):
                record.l10n_din5008_document_title = _('Quotation')
            else:
                record.l10n_din5008_document_title = _('Sales Order')

    def _compute_l10n_din5008_addresses(self):
        for record in self:
            record.l10n_din5008_addresses = data = []
            commercial_partner = record.partner_id.commercial_partner_id
            delivery_partner = record.partner_shipping_id
            invoice_partner = record.partner_invoice_id

            different_partner_count = len((commercial_partner | delivery_partner | invoice_partner).ids)
            # To avoid repetition in the address block.
            if different_partner_count <= 1:
                continue

            if self._context.get('proforma'):
                if delivery_partner and delivery_partner != commercial_partner:
                    data.append((_("Shipping Address:"), delivery_partner))
                if invoice_partner and invoice_partner != commercial_partner:
                    # if the proforma invoice has an invoice address different from the company address,
                    # the company address will have its separate address block.
                    # we will not display the VAT number in the main address block, but in the beneficiary block
                    data.append((_("Beneficiary:"), commercial_partner, {'show_tax_id': True}))
            else:
                if invoice_partner != delivery_partner and delivery_partner != commercial_partner:
                    data.append((_("Shipping Address:"), delivery_partner))
                if invoice_partner != delivery_partner and invoice_partner != commercial_partner:
                    data.append((_("Invoicing Address:"), invoice_partner))
                if invoice_partner == delivery_partner and invoice_partner != commercial_partner:
                    data.append((_("Invoicing and Shipping Address:"), invoice_partner))

    def check_field_access_rights(self, operation, field_names):
        field_names = super().check_field_access_rights(operation, field_names)
        return [field_name for field_name in field_names if field_name not in {
            'l10n_din5008_addresses',
        }]

```

## File: models\__init__.py

```python
from . import sale

```

## File: report\din5008_sale_order_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="external_layout_din5008_saleorder" inherit_id="l10n_din5008.external_layout_din5008">
            <xpath expr="//t[@t-set='address']" position="before">
                <t t-if="doc and doc._name == 'sale.order' and doc.partner_id">
                    <t t-set="address">
                        <t t-if="doc.env.context.get('proforma')">
                            <address class="mb-0" t-field="doc.partner_invoice_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                            <t t-set="main_addr_id" t-value="doc.partner_invoice_id"/>
                            <div t-if="doc.partner_id.commercial_partner_id == doc.partner_invoice_id and main_addr_id.vat" id="partner_vat_address_same_as_shipping">
                                <t t-if="doc.company_id.account_fiscal_country_id.vat_label" t-out="doc.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>
                                <t t-else="">Tax ID</t>: <span t-field="main_addr_id.vat"/>
                            </div>
                        </t>
                        <t t-else="">
                            <address class="mb-0" t-field="doc.partner_id.commercial_partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                            <t t-set="main_addr_id" t-value="doc.partner_id.commercial_partner_id"/>
                            <div t-if="main_addr_id.vat" id="partner_vat_address_same_as_shipping">
                                <t t-if="doc.company_id.account_fiscal_country_id.vat_label" t-out="doc.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>
                                <t t-else="">Tax ID</t>: <span t-field="main_addr_id.vat"/>
                            </div>
                        </t>
                    </t>
                </t>
            </xpath>
        </template>
    </data>
</odoo>

```

