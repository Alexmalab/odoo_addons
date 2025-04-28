# Odoo Module: mrp_subcontracting_landed_costs

Category: Manufacturing/Manufacturing

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
    'name': 'Landed Costs With Subcontracting order',
    'version': '1.0',
    'summary': 'Advanced views to manage landed cost for subcontracting orders',
    'description': """
This module allows users to more easily identify subcontracting orders when applying landed costs,
by also displaying the associated picking reference in the search view.
    """,
    'depends': ['stock_landed_costs', 'mrp_subcontracting'],
    'category': 'Manufacturing/Manufacturing',
    'data': [
        'views/stock_landed_cost_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock_move.py

```python
from odoo import models


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _get_stock_valuation_layer_ids(self):
        self.ensure_one()
        stock_valuation_layer_ids = super()._get_stock_valuation_layer_ids()
        subcontracted_productions = self._get_subcontract_production()
        if self.is_subcontract and subcontracted_productions:
            return subcontracted_productions.move_finished_ids.stock_valuation_layer_ids
        return stock_valuation_layer_ids

```

## File: models\__init__.py

```python
from . import stock_move

```

## File: views\stock_landed_cost_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id='view_mrp_landed_costs_form' model='ir.ui.view'>
        <field name="name">mrp.subcontracting.landed.cost.form</field>
        <field name="model">stock.landed.cost</field>
        <field name="inherit_id" ref="mrp_landed_costs.view_mrp_landed_costs_form"/>
        <field name="arch" type="xml">
            <field name="mrp_production_ids" position="attributes">
                <attribute name="context">{'search_view_ref': 'mrp_subcontracting.mrp_production_subcontracting_filter', 'list_view_ref': 'mrp_subcontracting.mrp_production_subcontracting_tree_view'}</attribute>
            </field>
            <field name="picking_ids" position="attributes">
                <attribute name="domain">[('company_id', '=', company_id), '|', ('move_ids.stock_valuation_layer_ids', '!=', False), '&amp;', ('move_ids.is_subcontract', '!=', False), ('move_ids.state', '=', 'done')]</attribute>
            </field>
        </field>
    </record>
</odoo>

```

