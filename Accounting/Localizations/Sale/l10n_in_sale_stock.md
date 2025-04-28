# Odoo Module: l10n_in_sale_stock

Category: Accounting/Localizations/Sale

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
    'name': "India Sales and Warehouse Management",

    'summary': """
        Define default sales journal on the warehouse""",

    'description': """
        Define default sales journal on the warehouse,
        help you to choose correct sales journal on the sales order when
        you change the warehouse.
        useful when you setup the multiple GSTIN units.
    """,

    'author': "Odoo",
    'website': "https://www.odoo.com",
    'category': 'Accounting/Localizations/Sale',
    'version': '0.1',

    'depends': ['l10n_in_sale', 'l10n_in_stock'],

    'data': [
        'views/stock_warehouse_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class SaleOrder(models.Model):
    _inherit = "sale.order"

    @api.depends('company_id','warehouse_id')
    def _compute_l10n_in_journal_id(self):
        super()._compute_l10n_in_journal_id()
        for order in self:
            if order.l10n_in_company_country_code == 'IN':
                if order.warehouse_id.l10n_in_sale_journal_id:
                    order.l10n_in_journal_id = order.warehouse_id.l10n_in_sale_journal_id.id

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class Stock(models.Model):
    _inherit = 'stock.warehouse'

    l10n_in_sale_journal_id = fields.Many2one('account.journal', string="Sale Journal")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order
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
                <field name="l10n_in_sale_journal_id" domain="[('company_id', '=', company_id), ('type','=','sale')]" options="{'no_create': True}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

