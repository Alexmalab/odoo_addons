# Odoo Module: l10n_in_purchase_stock

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
    'name': "India Purchase and Warehouse Management",
    'icon': '/l10n_in/static/description/icon.png',

    'summary': """
        Define default purchase journal on the warehouse""",

    'description': """
        Define default purchase journal on the warehouse,
        help you to choose correct purchase journal on the purchase order when
        you change the picking operation.
        useful when you setup the multiple GSTIN units.
    """,

    'author': "Odoo",
    'website': "https://www.odoo.com",
    'category': 'Accounting/Localizations/Purchase',
    'version': '1.0',

    'depends': ['l10n_in_purchase', 'l10n_in_stock'],

    'data': [
        'views/stock_warehouse_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMove(models.Model):
    _inherit = "account.move"

    def _l10n_in_get_warehouse_address(self):
        res = super()._l10n_in_get_warehouse_address()
        if self.invoice_line_ids.purchase_line_id:
            company_shipping_id = self.mapped(
                "invoice_line_ids.purchase_line_id.move_ids.warehouse_id.partner_id"
            )
            if len(company_shipping_id) == 1:
                return company_shipping_id
        return res

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class PurchaseOrder(models.Model):
    _inherit = "purchase.order"

    @api.onchange('company_id', 'picking_type_id')
    def l10n_in_onchange_company_id(self):
        if self.picking_type_id.warehouse_id and self.picking_type_id.warehouse_id.l10n_in_purchase_journal_id:
            self.l10n_in_journal_id = self.picking_type_id.warehouse_id.l10n_in_purchase_journal_id.id
        else:
            super().l10n_in_onchange_company_id()

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class Stock(models.Model):
    _inherit = 'stock.warehouse'

    l10n_in_purchase_journal_id = fields.Many2one('account.journal', string="Purchase Journal")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_order
from . import stock_warehouse
from . import account_move

```

## File: views\stock_warehouse_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_stock_warehouse_inherit_l10n_in_stock" model="ir.ui.view">
        <field name="name">stock.warehouse.form.inherit.l10n.in.stock</field>
        <field name="model">stock.warehouse</field>
        <field name="inherit_id" ref="stock.view_warehouse"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="l10n_in_purchase_journal_id" domain="[('company_id', '=', company_id), ('type','=','purchase')]" options="{'no_create': True}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

