# Odoo Module: purchase_price_diff

Category: Hidden

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
    'name': 'WMS Accounting',
    'version': '1.1',
    'summary': 'Inventory, Logistic, Valuation, Accounting',
    'description': """
WMS Accounting module
======================
This module adds the price difference account. Used in standard perpetual valuation.
    """,
    'depends': ['purchase_stock'],
    'data': [
        'views/product_views.xml',
    ],
    'category': 'Hidden',
    'sequence': 16,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-

from odoo import models

class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _get_price_diff_account(self):
        self.ensure_one()
        if self.product_id.cost_method == 'standard':
            debit_pdiff_account = self.product_id.property_account_creditor_price_difference \
                                    or self.product_id.categ_id.property_account_creditor_price_difference_categ
            debit_pdiff_account = self.move_id.fiscal_position_id.map_account(debit_pdiff_account)
            return debit_pdiff_account
        return super()._get_price_diff_account()

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProductCategory(models.Model):
    _inherit = "product.category"

    property_account_creditor_price_difference_categ = fields.Many2one(
        'account.account', string="Price Difference Account",
        company_dependent=True,
        help="This account will be used to value price difference between purchase price and accounting cost.")


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

    property_account_creditor_price_difference = fields.Many2one(
        'account.account', string="Price Difference Account", company_dependent=True,
        help="This account is used in automated inventory valuation to "\
             "record the price difference between a purchase order and its related vendor bill when validating this vendor bill.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_line
from . import product

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_category_property_form" model="ir.ui.view">
        <field name="name">product.category.property.form.inherit.stock</field>
        <field name="model">product.category</field>
        <field name="inherit_id" ref="account.view_category_property_form"/>
        <field name="arch" type="xml">
            <field name="property_account_income_categ_id" position="before">
                <field name="property_account_creditor_price_difference_categ"
                    domain="[('deprecated','=',False)]" groups="account.group_account_readonly"
                    attrs="{'invisible':[('property_valuation', '=', 'manual_periodic')]}"/>
            </field>
        </field>
    </record>

    <record id="product_template_form_view" model="ir.ui.view">
        <field name="name">product.normal.form.inherit.stock</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="account.product_template_form_view"/>
        <field name="arch" type="xml">
            <field name="property_account_expense_id" position="after">
                <field name="property_account_creditor_price_difference" domain="[('deprecated','=',False)]" attrs="{'readonly':[('purchase_ok', '=', 0)]}" groups="account.group_account_readonly"/>
            </field>
        </field>
    </record>
</odoo>
```

