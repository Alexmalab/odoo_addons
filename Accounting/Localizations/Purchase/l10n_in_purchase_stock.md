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
    'countries': ['in'],

    'summary': "Get warehouse address if the bill is created from Purchase Order",

    'description': """
Get the warehouse address if the bill is created from the Purchase Order

So this module is to get the warehouse address if the bill is created from Purchase Order
    """,

    'website': "https://www.odoo.com",
    'category': 'Accounting/Localizations/Purchase',
    'version': '1.0',

    'depends': [
        'l10n_in_purchase',
        'l10n_in_stock',
        'purchase_stock'
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

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = "stock.move"

    def _l10n_in_get_product_price_unit(self):
        self.ensure_one()
        if line_id := self.purchase_line_id:
            if qty := line_id.product_qty:
                company_id = line_id.company_id
                return line_id.currency_id._convert(
                    line_id.product_uom._compute_price(line_id.price_subtotal / qty, self.product_uom),
                    company_id.currency_id,
                    company_id,
                    self.date,
                    round=False
                )
            return 0.00
        return super()._l10n_in_get_product_price_unit()

    def _l10n_in_get_product_tax(self):
        self.ensure_one()
        if line_id := self.purchase_line_id:
            return {
                'is_from_order': True,
                'taxes': line_id.taxes_id
            }
        return super()._l10n_in_get_product_tax()

```

## File: models\stock_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockPicking(models.Model):
    _inherit = "stock.picking"

    def _get_l10n_in_dropship_dest_partner(self):
        self.ensure_one()
        if line_id := self.purchase_id:
            return line_id.dest_address_id
        return False

    def _l10n_in_get_fiscal_position(self):
        self.ensure_one()
        if purchase_order := self.purchase_id:
            purchase_order.fiscal_position_id
        return super()._l10n_in_get_fiscal_position()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import stock_picking
from . import stock_move

```

