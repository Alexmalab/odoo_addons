# Odoo Module: mrp_subonctracting_landed_costs

Category: Manufacturing/Manufacturing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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
                <attribute name="context">{'search_view_ref': 'mrp_subcontracting.mrp_production_subcontracting_filter', 'tree_view_ref': 'mrp_subcontracting.mrp_production_subcontracting_tree_view'}</attribute>
            </field>
        </field>
    </record>
</odoo>

```

