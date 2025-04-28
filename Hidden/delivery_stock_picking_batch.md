# Odoo Module: delivery_stock_picking_batch

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
    'name': 'Delivery Stock Picking Batch',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Batch Transfer, Carrier',
    'description': """
This module makes the link between the batch pickings and carrier applications.

Allows to prepare batches depending on their carrier
""",
    'depends': ['stock_delivery', 'stock_picking_batch'],
    'data': [
        'views/stock_picking_type_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class StockPickingType(models.Model):
    _inherit = "stock.picking.type"

    def _get_default_weight_uom(self):
        return self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    batch_group_by_carrier = fields.Boolean('Carrier', help="Automatically group batches by carriers")
    batch_max_weight = fields.Integer("Maximum weight",
                                      help="A transfer will not be automatically added to batches that will exceed this weight if the transfer is added to it.\n"
                                           "Leave this value as '0' if no weight limit.")
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name', readonly=True, default=_get_default_weight_uom)

    def _compute_weight_uom_name(self):
        for picking_type in self:
            picking_type.weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    @api.model
    def _get_batch_group_by_keys(self):
        return super()._get_batch_group_by_keys() + ['batch_group_by_carrier']


class StockPicking(models.Model):
    _inherit = "stock.picking"

    def _get_possible_pickings_domain(self):
        domain = super()._get_possible_pickings_domain()
        if self.picking_type_id.batch_group_by_carrier:
            domain = expression.AND([domain, [('carrier_id', '=', self.carrier_id.id if self.carrier_id else False)]])

        return domain

    def _get_possible_batches_domain(self):
        domain = super()._get_possible_batches_domain()
        if self.picking_type_id.batch_group_by_carrier:
            domain = expression.AND([domain, [('picking_ids.carrier_id', '=', self.carrier_id.id if self.carrier_id else False)]])

        return domain

    def _get_auto_batch_description(self):
        description = super()._get_auto_batch_description()
        if self.picking_type_id.batch_group_by_carrier and self.carrier_id:
            description = f"{description}, {self.carrier_id.name}" if description else self.carrier_id.name
        return description

    def _is_auto_batchable(self, picking=None):
        """ Verifies if a picking can be put in a batch with another picking without violating auto_batch constrains.
        """
        res = super()._is_auto_batchable(picking)
        if not picking:
            picking = self.env['stock.picking']
        if self.picking_type_id.batch_max_weight:
            res = res and (self.weight + picking.weight <= self.picking_type_id.batch_max_weight)
        return res

```

## File: models\stock_picking_batch.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class StockPickingBatch(models.Model):
    _inherit = "stock.picking.batch"

    def _is_picking_auto_mergeable(self, picking):
        """ Verifies if a picking can be safely inserted into the batch without violating auto_batch_constrains.
        """
        res = super()._is_picking_auto_mergeable(picking)
        if self.picking_type_id.batch_max_weight:
            batch_weight = sum(self.picking_ids.mapped('weight'))
            res = res and (batch_weight + picking.weight <= self.picking_type_id.batch_max_weight)
        return res

    def _is_line_auto_mergeable(self, num_of_moves=False, num_of_pickings=False, weight=False):
        res = super()._is_line_auto_mergeable(num_of_moves, num_of_pickings, weight)
        if self.picking_type_id.batch_max_weight:
            wave_weight = sum(self.move_ids.mapped('weight'))
            res = res and (wave_weight + weight <= self.picking_type_id.batch_max_weight)
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking
from . import stock_picking_batch

```

## File: views\stock_picking_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_type_form_inherit" model="ir.ui.view">
        <field name="name">stock.picking.type.form.inherit</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock_picking_batch.view_picking_type_form_inherit"/>
        <field name="arch" type="xml">
            <div name="batch_contact" position="after">
                <span invisible="not auto_batch"/>
                <div name="batch_carrier" class="o_row" invisible="not auto_batch">
                    <field name="batch_group_by_carrier"/>
                    <label for="batch_group_by_carrier"/>
                </div>
            </div>
            <field name="batch_auto_confirm" position="before">
                <span invisible="not auto_batch"/>
                <div class="o_row" name="batch_max_weight" invisible="not auto_batch">
                    <field name="batch_max_weight" class="oe_inline"/>
                    <span><field name="weight_uom_name"/></span>
                </div>
            </field>
        </field>
    </record>
</odoo>

```

