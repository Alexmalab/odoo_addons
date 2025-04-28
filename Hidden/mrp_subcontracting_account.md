# Odoo Module: mrp_subcontracting_account

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
    'name': 'Subcontracting Management with Stock Valuation',
    'version': '0.1',
    'category': 'Hidden',
    'description': """
        This bridge module allows to manage subcontracting with valuation.
    """,
    'depends': ['mrp_subcontracting', 'mrp_account'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
    'data': [
        'security/mrp_subcontracting_account_security.xml',
        'security/ir.model.access.csv',
    ],
}

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    def _cal_price(self, consumed_moves):
        finished_move = self.move_finished_ids.filtered(lambda x: x.product_id == self.product_id and x.state not in ('done', 'cancel') and x.quantity_done > 0)
        # Take the price unit of the reception move
        last_done_receipt = finished_move.move_dest_ids.filtered(lambda m: m.state == 'done')[-1:]
        if last_done_receipt.is_subcontract:
            self.extra_cost = last_done_receipt._get_price_unit()
        return super()._cal_price(consumed_moves=consumed_moves)

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields

class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _compute_bom_price(self, bom, boms_to_recompute=False, byproduct_bom=False):
        """ Add the price of the subcontracting supplier if it exists with the bom configuration.
        """
        price = super()._compute_bom_price(bom, boms_to_recompute, byproduct_bom)
        if bom and bom.type == 'subcontract':
            seller = self._select_seller(quantity=bom.product_qty, uom_id=bom.product_uom_id, params={'subcontractor_ids': bom.subcontractor_ids})
            if seller:
                seller_price = seller.currency_id._convert(seller.price, self.env.company.currency_id, (bom.company_id or self.env.company), fields.Date.today())
                price += seller.product_uom._compute_price(seller_price, self.uom_id)
        return price

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _should_force_price_unit(self):
        self.ensure_one()
        return self.is_subcontract or super()._should_force_price_unit()

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv.expression import OR


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def action_view_stock_valuation_layers(self):
        action = super(StockPicking, self).action_view_stock_valuation_layers()
        subcontracted_productions = self._get_subcontract_production()
        if not subcontracted_productions:
            return action
        domain = action['domain']
        domain_subcontracting = [('id', 'in', (subcontracted_productions.move_raw_ids | subcontracted_productions.move_finished_ids).stock_valuation_layer_ids.ids)]
        domain = OR([domain, domain_subcontracting])
        return dict(action, domain=domain)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_production
from . import stock_picking
from . import product_product
from . import stock_move

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_subcontracting_portal_analytic_account,subcontracting.portal.analytic_account,mrp_account.model_account_analytic_account,base.group_portal,1,0,0,0
access_subcontracting_portal_analytic_line,subcontracting.portal.analytic_line,mrp_account.model_account_analytic_line,base.group_portal,1,1,0,0

```

## File: security\mrp_subcontracting_account_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.rule" id="analytic_accout_subcontractor_rule">
        <field name="name">Analytic Account Subcontractor</field>
        <field name="model_id" ref="mrp_account.model_account_analytic_account"/>
        <field name="domain_force">[('bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="analytic_accout_line_subcontractor_rule">
        <field name="name">Analytic Account Line Subcontractor</field>
        <field name="model_id" ref="mrp_account.model_account_analytic_line"/>
        <field name="domain_force">[('account_id.bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

</odoo>

```

