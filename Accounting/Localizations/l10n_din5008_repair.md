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
        'report/din5008_repair_order_layout.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\repair.py

```python
from odoo import models, fields, _
from odoo.tools import format_date


class RepairOrder(models.Model):
    _inherit = 'repair.order'

    l10n_din5008_template_data = fields.Binary(compute='_compute_l10n_din5008_template_data')
    l10n_din5008_document_title = fields.Char(compute='_compute_l10n_din5008_document_title')

    def _compute_l10n_din5008_template_data(self):
        for record in self:
            record.l10n_din5008_template_data = data = []
            if record.product_id:
                data.append((_("Product to Repair"), record.product_id.name))
            if record.lot_id:
                data.append((_("Lot/Serial Number"), record.lot_id.name))
            data.append((_("Printing Date"), format_date(self.env, fields.Date.today())))

    def _compute_l10n_din5008_document_title(self):
        for record in self:
            if record.state == 'draft':
                record.l10n_din5008_document_title = _("Repair Quotation")
            else:
                record.l10n_din5008_document_title = _("Repair Order")

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
            <xpath expr="//t[@t-set='address']" position="before">
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

