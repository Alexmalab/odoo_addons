# Odoo Module: l10n_in_stock

Category: Accounting/Localizations

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
    'name': 'Indian - Stock Report(GST)',
    'version': '1.0',
    'description': """GST Stock Report""",
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_in',
        'stock',
    ],
    'data': [
        'views/report_stockpicking_operations.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\product_demo.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="stock.product_cable_management_box" model="product.product">
            <field name="l10n_in_hsn_code">48196000</field>
        </record>
    </data>
</odoo>

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = "stock.move"

    def _l10n_in_get_product_price_unit(self):
        self.ensure_one()
        return self.product_id.uom_id._compute_price(
            self.product_id.with_company(self.company_id).standard_price, self.product_uom
        )

    def _l10n_in_get_product_tax(self):
        self.ensure_one()
        return {
            'is_from_order': False,
            'taxes': (
                self.picking_code == "incoming" and
                self.product_id.supplier_taxes_id or self.product_id.taxes_id
            ),
        }

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def _should_generate_commercial_invoice(self):
        super(StockPicking, self)._should_generate_commercial_invoice()
        return True

    def _get_l10n_in_dropship_dest_partner(self):
        """
        To be overriden by `l10n_in_purchase_stock` will be ideal to use it for `l10n_in_ewaybill_stock`
        returns destination partner from purchase_id
        """
        pass

    def _l10n_in_get_invoice_partner(self):
        """
        To be overriden by `l10n_in_sale_stock` will be ideal to use it for `l10n_in_ewaybill_stock`
        returns invoice partner from sale_id
        """
        pass

    def _l10n_in_get_fiscal_position(self):
        """
        To be inherited by `l10n_in_*_stock` will be ideal to use it for `l10n_in_ewaybill_stock`
        returns fiscal position from order
        """
        pass

```

## File: models\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking
from . import stock_move

```

## File: views\report_stockpicking_operations.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="gst_report_picking_inherit" inherit_id="stock.report_picking">
        <xpath expr="//span[@t-field='ml.product_id.description_picking']" position="after">
            <t t-if="ml.product_id and ml.product_id.l10n_in_hsn_code and o.company_id.account_fiscal_country_id.code == 'IN'"><h6><strong class="ml16">HSN/SAC Code:</strong> <span t-field="ml.product_id.l10n_in_hsn_code"/></h6></t>
        </xpath>
    </template>

</odoo>

```

