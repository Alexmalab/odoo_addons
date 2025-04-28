# Odoo Module: l10n_din5008_stock

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
    'name': 'DIN 5008 - Stock',
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_din5008',
        'stock',
    ],
    'data': [
        'report/din5008_stock_templates.xml',
        'report/din5008_stock_picking_layout.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: report\din5008_stock_picking_layout.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <data>
        <template id="external_layout_din5008_deliveryslip" inherit_id="l10n_din5008.external_layout_din5008">
            <xpath expr="//div[@id='din5008_report_main_address']" position="before">
                <t t-if="o and o._name == 'stock.picking' and (o.should_print_delivery_address() or o.partner_id)">
                    <t t-set="address">
                        <t t-set="main_address" t-value="o.move_ids[0].partner_id if o.should_print_delivery_address() else o.partner_id"/>
                        <t t-if="o.should_print_delivery_address()">
                            <address class="mb-0" t-field="o.move_ids[0].partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                        </t>
                        <t t-else="">
                            <address class="mb-0" t-field="o.partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                        </t>
                        <div t-if="not (o.picking_type_id.code == 'outgoing' and main_address.id != o.partner_id.commercial_partner_id.id)" id="partner_vat_address_same_as_shipping">
                            <t t-if="o.company_id.account_fiscal_country_id.vat_label" t-out="o.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>
                            <t t-else="">Tax ID</t>: <span t-field="o.partner_id.vat"/>
                        </div>
                    </t>
                </t>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: report\din5008_stock_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_delivery_document" inherit_id="stock.report_delivery_document">
        <xpath expr="//t[@t-set='address']" position="after">
            <t t-set="din5008_address_block">
                <tr t-if="o and o._name=='stock.picking' and o.partner_id">
                    <td class="shipping_address">
                        <t t-if="o.picking_type_id.code == 'incoming'">
                            <span class="fw-bold">Vendor Address:</span>
                            <address t-esc="o.partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                        </t>
                        <t t-if="o.picking_type_id.code == 'internal'">
                            <span class="fw-bold">Warehouse Address:</span>
                            <address t-esc="o.partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                        </t>
                        <t t-if="o.picking_type_id.code == 'outgoing'">
                            <t t-set="main_address_box" t-value="o.move_ids[0].partner_id if o.should_print_delivery_address() else o.partner_id"/>
                            <t t-if="main_address_box.id != o.partner_id.commercial_partner_id.id">
                                <span class="fw-bold">Beneficiary:</span>
                                <address class="mb-0" t-esc="o.partner_id.commercial_partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                                <div t-if="o.partner_id.vat">
                                    <t t-if="o.partner_id.company_id.account_fiscal_country_id.vat_label" t-out="o.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>
                                    <t t-else="">Tax ID</t>: <span t-field="o.partner_id.vat"/>
                                </div>
                            </t>
                            <t t-if="main_address_box.id != o.partner_id.id">
                                <span class="fw-bold">Customer Address:</span>
                                <address t-esc="o.partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                            </t>
                        </t>
                    </td>
                </tr>
            </t>
        </xpath>
    </template>
</odoo>

```

