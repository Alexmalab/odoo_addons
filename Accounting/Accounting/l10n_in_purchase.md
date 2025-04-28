# Odoo Module: l10n_in_purchase

Category: Accounting/Accounting

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
    'version': '1.0',
    'description': """GST Purchase Report""",
    'category': 'Accounting/Accounting',
    'depends': [
        'l10n_in',
        'purchase',
    ],
    'data': [
        'views/report_purchase_order.xml',
        'views/purchase_order_views.xml',
    ],
    'installable': True,
    'application': False,
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
        if self.purchase_vendor_bill_id.purchase_order_id or self.purchase_id:
            journal_id = self.purchase_vendor_bill_id.purchase_order_id.l10n_in_journal_id or self.purchase_id.l10n_in_journal_id
            if journal_id:
                self.journal_id = journal_id
        return super()._onchange_purchase_auto_complete()

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PurchaseOrder(models.Model):
    _inherit = "purchase.order"

    l10n_in_journal_id = fields.Many2one('account.journal', string="Journal", \
        states={'posted': [('readonly', True)]}, domain="[('type','=', 'purchase')]")

    @api.onchange('company_id')
    def l10n_in_onchange_company_id(self):
        domain = [('company_id', '=', self.company_id.id), ('type', '=', 'purchase')]

        journal = self.env['account.journal'].search(domain, limit=1)
        if journal:
            self.l10n_in_journal_id = journal.id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_order
from . import account_move

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
            <xpath expr="//group[@name='other_info']//field[@name='user_id']" position="after">
                <field name="l10n_in_journal_id" domain="[('company_id', '=', company_id), ('type','=','purchase')]" options="{'no_create': True}"/>
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
                <t t-if="line.product_id.l10n_in_hsn_code and o.company_id.country_id.code == 'IN'">
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
                <t t-if="order_line.product_id.l10n_in_hsn_code and o.company_id.country_id.code == 'IN'">
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

