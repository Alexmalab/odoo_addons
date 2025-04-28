# Odoo Module: mrp_landed_costs

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
    'name': 'Landed Costs On MO',
    'version': '1.0',
    'summary': 'Landed Costs on Manufacturing Order',
    'description': """
This module allows you to easily add extra costs on manufacturing order 
and decide the split of these costs among their stock moves in order to 
take them into account in your stock valuation.
    """,
    'depends': ['stock_landed_costs', 'mrp'],
    'category': 'Manufacturing/Manufacturing',
    'data': [
        'views/stock_landed_cost_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock_landed_cost.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class StockLandedCost(models.Model):
    _inherit = 'stock.landed.cost'

    target_model = fields.Selection(selection_add=[
        ('manufacturing', "Manufacturing Orders")
    ], ondelete={'manufacturing': 'set default'})
    mrp_production_ids = fields.Many2many(
        'mrp.production', string='Manufacturing order',
        copy=False, states={'done': [('readonly', True)]}, groups='stock.group_stock_manager')

    @api.onchange('target_model')
    def _onchange_target_model(self):
        super()._onchange_target_model()
        if self.target_model != 'manufacturing':
            self.mrp_production_ids = False

    def _get_targeted_move_ids(self):
        return super()._get_targeted_move_ids() | self.mrp_production_ids.move_finished_ids

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_landed_cost

```

## File: views\stock_landed_cost_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id='view_mrp_landed_costs_form' model='ir.ui.view'>
        <field name="name">mrp.landed.cost.form</field>
        <field name="model">stock.landed.cost</field>
        <field name="inherit_id" ref="stock_landed_costs.view_stock_landed_cost_form"/>
        <field name="arch" type="xml">
            <field name="target_model" position="attributes">
                <attribute name="invisible">0</attribute>
                <attribute name="groups">stock.group_stock_manager</attribute>
            </field>
            <field name="picking_ids" position="after">
                <field name="mrp_production_ids" 
                    widget="many2many_tags" options="{'no_create_edit': True}"
                    attrs="{'invisible': [('target_model', '!=', 'manufacturing')]}"
                    domain="[('company_id', '=', company_id), ('move_finished_ids.stock_valuation_layer_ids', '!=', False)]"/>
            </field>
        </field>
    </record>
</odoo>

```

