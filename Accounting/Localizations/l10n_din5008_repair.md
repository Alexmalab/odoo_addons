# Odoo Module: l10n_din5008_repair

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
    'name': 'DIN 5008 - Repair',
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_din5008',
        'repair',
    ],
    'data': [
        'report/din5008_repair_templates.xml',
        'report/din5008_repair_order_layout.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\repair.py

```python
from odoo import models, fields


class RepairOrder(models.Model):
    _inherit = 'repair.order'

    l10n_din5008_printing_date = fields.Date(default=fields.Date.today, store=False)

```

## File: models\__init__.py

```python
from . import repair

```

## File: report\din5008_repair_order_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="external_layout_din5008_repairorder" inherit_id="l10n_din5008.external_layout_din5008">
            <xpath expr="//div[@id='din5008_report_main_address']" position="before">
                <t t-if="o and o._name == 'repair.order' and o.partner_id">
                    <t t-set="address">
                        <address class="mb-0" t-field="o.partner_id" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                        <div t-if="o.partner_id.vat">
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

## File: report\din5008_repair_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_repairorder" inherit_id="repair.report_repairorder">
        <xpath expr="//t[@t-set='o']" position="after">
            <t t-set="din5008_document_information">
                <div class="information_block" t-if="o and o._name=='repair.order'">
                    <table>
                        <tr t-if="o.product_id">
                            <td>Product to Repair:</td>
                            <td><t t-out="o.product_id.name"/></td>
                        </tr>
                        <tr t-if="o.lot_id">
                            <td>Lot/Serial Number:</td>
                            <td><t t-out="o.lot_id.name"/></td>
                        </tr>
                        <tr>
                            <td>Printing Date:</td>
                            <td><t t-out="o.l10n_din5008_printing_date" t-options="{'widget': 'date'}"/></td>
                        </tr>
                    </table>
                </div>
            </t>

            <t t-set="din5008_document_title">
                <span t-if="o and o._name == 'repair.order'">
                    <t t-if="o.state == 'draft'">Repair Quotation</t>
                    <t t-else="">Repair Order</t>
                </span>
            </t>
        </xpath>
    </template>
</odoo>

```

