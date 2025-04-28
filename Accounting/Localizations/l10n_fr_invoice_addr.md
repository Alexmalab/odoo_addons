# Odoo Module: l10n_fr_invoice_addr

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "France - Adding Mandatory Invoice Mentions (Decree no. 2022-1299)",
    'version': '1.0',
    'category': 'Accounting/Localizations',
    'description': """
Add new address fields necessary to respect the new 2024-07-01 French law
(https://www.legifrance.gouv.fr/jorf/id/JORFTEXT000046383394) to invoices.
""",
    'depends': [
        'l10n_fr',
    ],
    'auto_install': True,
    'data': [
        'views/report_invoice.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_fr_is_company_french = fields.Boolean(compute='_compute_l10n_fr_is_company_french')

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        company = self.env.company
        if view_type == 'form' and company.country_code in company._get_france_country_codes():
            shipping_field = arch.xpath("//field[@name='partner_shipping_id']")[0]
            shipping_field.attrib.pop("groups", None)
        return arch, view

    @api.depends('company_id.country_code')
    def _compute_l10n_fr_is_company_french(self):
        for record in self:
            record.l10n_fr_is_company_french = record.country_code in record.company_id._get_france_country_codes()

```

## File: models\__init__.py

```python
from . import account_move

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="(//address)[1]" position="after">
            <div class="mb-0" t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id.siret">
                SIRET: <t t-esc="o.partner_id.commercial_partner_id.siret"/>
            </div>
        </xpath>
        <xpath expr="(//address)[2]" position="after">
            <div class="mb-0" t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id.siret">
                SIRET: <t t-esc="o.partner_id.commercial_partner_id.siret"/>
            </div>
        </xpath>
        <xpath expr="(//address)[3]" position="after">
            <div class="mb-0" t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id.siret">
                SIRET: <t t-esc="o.partner_id.commercial_partner_id.siret"/>
            </div>
        </xpath>

        <xpath expr="//div[@id='informations']" position="inside">
            <t t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id != o.partner_id and o.move_type.startswith('out_')">
                <t t-set="partner" t-value="o.partner_id.commercial_partner_id"/>
                <div t-attf-class="#{'col-auto col-3 mw-100' if report_type != 'html' else 'col'} mb-2" name="customer_address">
                    <strong>Customer Address:</strong>
                    <br/>
                    <address t-field="partner.self" class="m-0" t-options="{'widget': 'contact', 'fields': ['address'], 'no_marker': True}"/>
                </div>
            </t>
        </xpath>

        <xpath expr="//div[@id='informations']" position="inside">
            <t t-if="o.l10n_fr_is_company_french and o.move_type.startswith('out_')">
                <t t-set="tax_scopes" t-value="o.invoice_line_ids.mapped('tax_ids.tax_scope')"/>
                <t t-set="has_service" t-value="'service' in tax_scopes"/>
                <t t-set="has_consu" t-value="'consu' in tax_scopes"/>

                <div t-if="has_service or has_consu" t-attf-class="#{'col-auto col-3 mw-100' if report_type != 'html' else 'col'} mb-2" name="operation_type">
                    <strong>Operation Type:</strong>
                    <br/>
                    <span t-if="has_service and has_consu">
                        Mixed Operation
                    </span>
                    <span t-elif="has_service and not has_consu">
                        Service Delivery
                    </span>
                    <span t-else="">
                        Goods Delivery
                    </span>
                </div>
            </t>
        </xpath>

        <xpath expr="//p[@name='payment_communication']" position="after">
            <p t-if="o.l10n_fr_is_company_french and o.move_type.startswith('out_') and 'on_invoice' in o.invoice_line_ids.mapped('tax_ids.tax_exigibility')">
                Option to pay tax on debits
            </p>
        </xpath>
    </template>
</odoo>

```

