# Odoo Module: l10n_br_sales

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Brazil - Sale',
    'version': '1.0',
    'description': 'Sale modifications for Brazil',
    'category': 'Localization',
    'depends': [
        'l10n_br',
        'sale',
    ],
    'data': [
        'views/sale_portal_templates.xml',
        'report/sale_order_templates.xml',
        'report/report_invoice_templates.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\sale_order.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models

class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _get_name_portal_content_view(self):
        self.ensure_one()
        return 'l10n_br_sales.sale_order_portal_content_brazil' if self.company_id.country_code == 'BR' else super()._get_name_portal_content_view()

    def _get_name_tax_totals_view(self):
        self.ensure_one()
        return 'l10n_br_sales.document_tax_totals_brazil' if self.company_id.country_code == 'BR' else super()._get_name_tax_totals_view()

```

## File: models\__init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import sale_order

```

## File: report\report_invoice_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="document_tax_totals_brazil" inherit_id="sale.document_tax_totals" primary="True">
        <t t-as="subtotal" position="replace"/>
    </template>
</odoo>

```

## File: report\sale_order_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_saleorder_document" inherit_id="sale.report_saleorder_raw">
        <xpath expr='//t[@t-call="sale.report_saleorder_document"]' position="attributes">
            <attribute name="t-if" add="doc.company_id.country_code != 'BR'" separator=" and "/>
        </xpath>
        <xpath expr='//t[@t-call="sale.report_saleorder_document"]' position="after">
            <t t-elif="doc.company_id.country_code == 'BR'"
                t-call="l10n_br_sales.report_saleorder_document_brazil" t-lang="doc.partner_id.lang"/>
        </xpath>
    </template>

    <template id="report_saleorder_document_brazil" inherit_id="sale.report_saleorder_document" primary="True">
        <th name="th_taxes" position="replace"/>
        <td name="td_taxes" position="replace"/>
        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="attributes">
            <attribute name="t-value">current_subtotal + line.price_total</attribute>
        </t>
        <span t-field="line.price_subtotal" position="attributes">
            <attribute name="t-field">line.price_total</attribute>
        </span>
        <xpath expr="//t[@t-call='sale.document_tax_totals']" position="replace">
            <t t-call="l10n_br_sales.document_tax_totals_brazil"/>
        </xpath>
    </template>
</odoo>

```

## File: views\sale_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="sale_order_portal_content_brazil" inherit_id="sale.sale_order_portal_content" primary="True">
        <!-- hide the taxes th -->
        <th id="taxes_header" position="replace"/>
        <!-- hide the taxes td -->
        <td id="taxes" position="replace"/>
        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="attributes">
            <attribute name="t-value">current_subtotal + line.price_total</attribute>
        </t>
        <span t-field="line.price_subtotal" position="attributes">
            <attribute name="t-field">line.price_total</attribute>
        </span>
        <xpath expr="//t[@t-call='sale.sale_order_portal_content_totals_table']" position="replace">
            <table class="table table-sm">
                <t t-set="tax_totals" t-value="sale_order.tax_totals"/>
                <t t-call="l10n_br_sales.document_tax_totals_brazil"/>
            </table>
        </xpath>
    </template>
</odoo>

```

