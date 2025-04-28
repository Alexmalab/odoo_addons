# Odoo Module: l10n_in_purchase_stock

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
    'name': "India Purchase and Warehouse Management",

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
    'category': 'Accounting/Accounting',
    'version': '1.0',

    'depends': ['l10n_in_purchase', 'l10n_in_stock'],

    'data': [
        'views/stock_warehouse_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

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

