# Odoo Module: l10n_din5008_sale

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python

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
        'report/din5008_sale_templates.xml',
        'report/din5008_sale_order_layout.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: report\din5008_sale_order_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="external_layout_din5008_saleorder" inherit_id="l10n_din5008.external_layout_din5008">
            <xpath expr="//div[@id='din5008_report_main_address']" position="before">
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

## File: report\din5008_sale_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_saleorder_document" inherit_id="sale.report_saleorder_document">
        <xpath expr="//t[@t-set='forced_vat']" position="after">
            <t t-set="din5008_document_information">
                <div class="information_block" t-if="doc and doc._name=='sale.order'">
                    <table>
                        <t t-if="doc.state in {'draft', 'sent'}">
                            <tr t-if="doc.name">
                                <td>Quotation No.:</td>
                                <td><div t-field="doc.name"/></td>
                            </tr>
                            <tr t-if="doc.date_order">
                                <td>Quotation Date:</td>
                                <td><div t-field="doc.date_order" t-options="{'widget': 'date'}"/></td>
                            </tr>
                            <tr t-if="doc.validity_date">
                                <td>Expiration:</td>
                                <td><div t-field="doc.validity_date" t-options="{'widget': 'date'}"/></td>
                            </tr>
                        </t>
                        <t t-else="">
                            <tr t-if="doc.name">
                                <td>Order No.:</td>
                                <td><div t-field="doc.name"/></td>
                            </tr>
                            <tr t-if="doc.date_order">
                                <td>Order Date:</td>
                                <td><div t-field="doc.date_order" t-options="{'widget': 'date'}"/></td>
                            </tr>
                        </t>
                        <tr t-if="doc.client_order_ref">
                            <td>Customer Reference:</td>
                            <td><div t-field="doc.client_order_ref"/></td>
                        </tr>
                        <tr t-if="doc.user_id">
                            <td>Salesperson:</td>
                            <td><div t-field="doc.user_id.name"/></td>
                        </tr>
                        <tr t-if="'incoterm' in doc._fields and doc.incoterm">
                            <td>Incoterm:</td>
                            <td><div t-field="doc.incoterm.code"/></td>
                        </tr>
                    </table>
                </div>
            </t>

            <t t-set="din5008_address_block">
                <tr t-if="doc and doc._name=='sale.order'">
                    <t t-set="commercial_partner" t-value="doc.partner_id.commercial_partner_id"/>
                    <t t-set="invoice_partner" t-value="doc.partner_invoice_id"/>
                    <t t-set="delivery_partner" t-value="doc.partner_shipping_id"/>

                    <t t-if="doc.env.context.get('proforma')">
                        <td class="shipping_address" t-if="delivery_partner and delivery_partner != commercial_partner">
                            <span class="fw-bold">Shipping Address:</span>
                            <address t-esc="delivery_partner" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                        </td>
                        <td class="shipping_address" t-if="invoice_partner and invoice_partner != commercial_partner">
                            <span class="fw-bold">Beneficiary:</span>
                            <address class="mb-0" t-esc="commercial_partner" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                            <div t-if="commercial_partner.vat">
                                <t t-if="commercial_partner.company_id.account_fiscal_country_id.vat_label" t-out="commercial_partner.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>
                                <t t-else="">Tax ID</t>: <span t-field="commercial_partner.vat"/>
                            </div>
                        </td>
                    </t>
                    <t t-else="">
                        <t t-if="doc.partner_invoice_id == doc.partner_shipping_id">
                            <td class="shipping_address">
                                <span class="fw-bold">Invoicing and Shipping Address:</span>
                                <address t-esc="doc.partner_shipping_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                            </td>
                        </t>
                        <t t-else="">
                            <td class="shipping_address">
                                <span class="fw-bold">Shipping Address:</span>
                                <address t-esc="doc.partner_shipping_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                            </td>
                            <td class="shipping_address">
                                <span class="fw-bold">Invoicing Address:</span>
                                <address t-esc="doc.partner_invoice_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                            </td>
                        </t>
                    </t>
                </tr>
            </t>

            <t t-set="din5008_document_title">
                <span t-if="doc and doc._name == 'sale.order'">
                    <t t-if="doc.env.context.get('proforma')">Pro Forma Invoice</t>
                    <t t-elif="doc.state in {'draft', 'sent'}">Quotation</t>
                    <t t-else="">Sales Order</t>
                </span>
            </t>
        </xpath>
    </template>
</odoo>

```

