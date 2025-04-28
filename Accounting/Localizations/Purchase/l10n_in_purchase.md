# Odoo Module: l10n_in_purchase

Category: Accounting/Localizations/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indian - Purchase Report(GST)',
    'countries': ['in'],
    'version': '1.0',
    'description': """GST Purchase Report""",
    'category': 'Accounting/Localizations/Purchase',
    'depends': [
        'l10n_in',
        'purchase',
    ],
    'data': [
        'views/report_purchase_order.xml',
        'views/purchase_order_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    @api.onchange('purchase_vendor_bill_id', 'purchase_id')
    def _onchange_purchase_auto_complete(self):
        purchase_order_id = self.purchase_vendor_bill_id.purchase_order_id or self.purchase_id
        if purchase_order_id and purchase_order_id.country_code == 'IN':
            self.l10n_in_gst_treatment = purchase_order_id.l10n_in_gst_treatment
        return super()._onchange_purchase_auto_complete()

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PurchaseOrder(models.Model):
    _inherit = "purchase.order"

    l10n_in_gst_treatment = fields.Selection([
            ('regular', 'Registered Business - Regular'),
            ('composition', 'Registered Business - Composition'),
            ('unregistered', 'Unregistered Business'),
            ('consumer', 'Consumer'),
            ('overseas', 'Overseas'),
            ('special_economic_zone', 'Special Economic Zone'),
            ('deemed_export', 'Deemed Export'),
            ('uin_holders', 'UIN Holders'),
        ], string="GST Treatment", compute="_compute_l10n_in_gst_treatment", store=True)

    @api.depends('partner_id')
    def _compute_l10n_in_gst_treatment(self):
        for order in self:
            # set default value as False so CacheMiss error never occurs for this field.
            order.l10n_in_gst_treatment = False
            if order.country_code == 'IN':
                l10n_in_gst_treatment = order.partner_id.l10n_in_gst_treatment
                if not l10n_in_gst_treatment and order.partner_id.country_id and order.partner_id.country_id.code != 'IN':
                    l10n_in_gst_treatment = 'overseas'
                if not l10n_in_gst_treatment:
                    l10n_in_gst_treatment = order.partner_id.vat and 'regular' or 'consumer'
                order.l10n_in_gst_treatment = l10n_in_gst_treatment

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_order

```

## File: views\purchase_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_purchase_order_form_inherit_l10n_in_purchase" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit.l10n.in.purchase</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="country_code" invisible="1"/>
                <field name="l10n_in_gst_treatment" invisible="country_code != 'IN'" required="country_code == 'IN'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\report_purchase_order.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="gst_report_purchaseorder_document_inherit" inherit_id="purchase.report_purchaseorder_document">
        <xpath expr="//t[@t-foreach='o.order_line']//td[@id='product']" position="replace">
            <td>
                <span t-field="line.name"/>
                <t t-if="line.product_id.l10n_in_hsn_code and o.company_id.account_fiscal_country_id.code == 'IN'">
                    <h6>
                        <strong class="ml16">HSN/SAC Code:</strong>
                        <span t-field="line.product_id.l10n_in_hsn_code"/>
                    </h6>
                </t>
            </td>
        </xpath>
    </template>

    <template id="gst_report_purchasequotation_document_inherit" inherit_id="purchase.report_purchasequotation_document">
        <xpath expr="//t[@t-foreach='o.order_line']//td[@id='product']" position="replace">
            <td>
                <span t-field="order_line.name"/>
                <t t-if="order_line.product_id.l10n_in_hsn_code and o.company_id.account_fiscal_country_id.code == 'IN'">
                    <h6>
                        <strong class="ml16">HSN/SAC Code:</strong>
                        <span t-field="order_line.product_id.l10n_in_hsn_code"/>
                    </h6>
                </t>
            </td>
        </xpath>
    </template>

</odoo>

```

