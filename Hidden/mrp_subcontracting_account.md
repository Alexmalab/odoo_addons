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
        subcontracted_productions = self._get_subcontracted_productions()
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
from . import stock_move

```

