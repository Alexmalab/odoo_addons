# Odoo Module: l10n_din5008_stock

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
    'name': 'DIN 5008 - Stock',
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_din5008',
        'stock',
    ],
    'data': [
        'report/din5008_stock_picking_layout.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock.py

```python
from odoo import models, fields, _
from odoo.tools import format_date


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    l10n_din5008_addresses = fields.Binary(compute='_compute_l10n_din5008_addresses', exportable=False)

    def _compute_l10n_din5008_addresses(self):
        for record in self:
            record.l10n_din5008_addresses = data = []
            if not record.partner_id:
                continue
            if record.picking_type_id.code == 'incoming':
                data.append((_("Vendor Address:"), record.partner_id))
            elif record.picking_type_id.code == 'internal':
                data.append((_("Warehouse Address:"), record.partner_id))
            elif record.picking_type_id.code == 'outgoing':
                main_address_box = record.move_ids[0].partner_id if record.should_print_delivery_address() else record.partner_id
                if main_address_box.id != record.partner_id.commercial_partner_id.id:
                    data.append((
                        _("Beneficiary:"),
                        record.partner_id.commercial_partner_id,
                        # If the main delivery address is not the company address,
                        # the company address will have a separate beneficiary address block.
                        # The VAT number will not be displayed in the main address block,
                        # but in the beneficiary address block
                        {'show_tax_id': True},
                    ))
                if main_address_box.id != record.partner_id.id:
                    data.append((_("Customer Address:"), record.partner_id))

    def check_field_access_rights(self, operation, field_names):
        field_names = super().check_field_access_rights(operation, field_names)
        return [field_name for field_name in field_names if field_name not in {
            'l10n_din5008_addresses',
        }]

```

## File: models\__init__.py

```python
from . import stock

```

## File: report\din5008_stock_picking_layout.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <data>
        <template id="external_layout_din5008_deliveryslip" inherit_id="l10n_din5008.external_layout_din5008">
            <xpath expr="//t[@t-set='address']" position="before">
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

