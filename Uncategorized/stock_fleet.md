# Odoo Module: stock_fleet

Category: Uncategorized

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Stock Transport',
    'summary': 'Stock Transport: Dispatch Management System',
    'description': 'Transport Management: organize packs in your fleet, or carriers.',
    'version': '1.0',
    'depends': ['stock_picking_batch', 'fleet'],
    'demo': [
        'data/stock_fleet_demo.xml',
    ],
    'data': [
        'views/fleet_vehicle_model.xml',
        'views/stock_picking_batch.xml',
        'views/stock_picking_type.xml',
        'views/stock_picking_view.xml',
        'report/report_picking_batch.xml',
        'views/stock_location.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\stock_fleet_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="vehicle_tag_transport" model="fleet.vehicle.tag" >
        <field name="name">Transport</field>
        <field name="color" eval="5"/>
      </record>
    <!-- vehicle category -->
    <record id="model_category_truck" model="fleet.vehicle.model.category">
        <field name="name">Transport Truck</field>
        <field name="sequence">1</field>
        <field name="weight_capacity">44000</field>
        <field name="volume_capacity">32</field>
    </record>
    <record id="model_category_van" model="fleet.vehicle.model.category">
        <field name="name">Pickup Van</field>
        <field name="sequence">10</field>
        <field name="weight_capacity">7000</field>
        <field name="volume_capacity">15</field>
    </record>

    <!-- vehicle models -->
    <record id="model_volvo_fm" model="fleet.vehicle.model">
          <field name="name">FM</field>
          <field name="brand_id" ref="fleet.brand_volvo"/>
          <field name="vehicle_type">car</field>
          <field name="category_id" ref="model_category_truck"/>
    </record>
    <record id="model_ford_transit" model="fleet.vehicle.model">
          <field name="name">Transit</field>
          <field name="brand_id" ref="fleet.brand_ford"/>
          <field name="vehicle_type">car</field>
          <field name="category_id" ref="model_category_van"/>
    </record>

    <!-- fleet vehicles -->
    <record id="vehicle_truck_volvo" model="fleet.vehicle">
        <field name="license_plate">ODO-1347</field>
        <field name="vin_sn">1V6AD0ERXBC414305</field>
        <field name="model_id" ref="model_volvo_fm"/>
        <field name="category_id" ref="model_category_truck"/>
        <field name="color">Black</field>
        <field name="location">Millersburg Ohio (US) 44654</field>
        <field name="doors">2</field>
        <field name="driver_id" ref="base.user_admin" />
        <field name="acquisition_date" eval="(DateTime.now() - timedelta(days=336)).strftime('%Y-%m-%d')" />
        <field name="state_id" ref="fleet.fleet_vehicle_state_registered"/>
        <field name="odometer">18000</field>
        <field name="odometer_unit">kilometers</field>
        <field name="car_value">20000</field>
        <field name="model_year">2020</field>
        <field name="fuel_type">diesel</field>
        <field name="manager_id" ref="base.user_admin"/>
        <field eval="[(6,0,[ref('vehicle_tag_transport')])]" name="tag_ids"/>
    </record>

    <record id="dock_a" model="stock.location">
        <field name="name">Dock A</field>
        <field name="barcode">DOCKA</field>
        <field name="is_a_dock">True</field>
        <field name="location_id" ref="stock.stock_location_output"/>
    </record>
    <record id="dock_b" model="stock.location">
        <field name="name">Dock B</field>
        <field name="barcode">DOCKB</field>
        <field name="is_a_dock">True</field>
        <field name="location_id" ref="stock.stock_location_output"/>
    </record>
</odoo>

```

## File: models\fleet_vehicle_model.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.tools import format_list


class FleetVehicleModelCategory(models.Model):
    _inherit = 'fleet.vehicle.model.category'

    weight_capacity = fields.Float(string="Max Weight")
    weight_capacity_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_capacity_uom_name')
    volume_capacity = fields.Float(string="Max Volume")
    volume_capacity_uom_name = fields.Char(string='Volume unit of measure label', compute='_compute_volume_capacity_uom_name')

    def _compute_display_name(self):
        super()._compute_display_name()
        for record in self:
            additional_info = []
            if record.weight_capacity:
                additional_info.append(_("%(weight_capacity)s %(weight_uom)s", weight_capacity=record.weight_capacity, weight_uom=record.weight_capacity_uom_name))
            if record.volume_capacity:
                additional_info.append(_("%(volume_capacity)s %(volume_uom)s", volume_capacity=record.volume_capacity, volume_uom=record.volume_capacity_uom_name))
            if additional_info:
                additional_info = format_list(self.env, additional_info, "unit-short")
                record.display_name = _("%(display_name)s (%(load_capacity)s)", display_name=record.display_name, load_capacity=additional_info)

    def _compute_weight_capacity_uom_name(self):
        self.weight_capacity_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    def _compute_volume_capacity_uom_name(self):
        self.volume_capacity_uom_name = self.env['product.template']._get_volume_uom_name_from_ir_config_parameter()

```

## File: models\stock_location.py

```python
from odoo import fields, models


class StockPickingBatch(models.Model):
    _inherit = 'stock.location'

    is_a_dock = fields.Boolean("Is a Dock Location")

```

## File: models\stock_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockPicking(models.Model):
    _inherit = "stock.picking"

    zip = fields.Char(related='partner_id.zip', string='Zip', search="_search_zip")

    def _search_zip(self, operator, value):
        return [('partner_id.zip', operator, value)]

    def write(self, vals):
        res = super().write(vals)
        if 'batch_id' not in vals:
            return res
        batch = self.env['stock.picking.batch'].browse(vals.get('batch_id'))
        if batch and batch.dock_id:
            batch._set_moves_destination_to_dock()
        else:
            self._reset_location()
        return res

    def _reset_location(self):
        for picking in self:
            moves = picking.move_ids.filtered(lambda m: not m.location_dest_id._child_of(picking.location_dest_id))
            moves.write({'location_dest_id': picking.location_dest_id.id})

```

## File: models\stock_picking_batch.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class StockPickingBatch(models.Model):
    _inherit = 'stock.picking.batch'

    vehicle_id = fields.Many2one('fleet.vehicle', string="Vehicle")
    vehicle_category_id = fields.Many2one(
        'fleet.vehicle.model.category', string="Vehicle Category",
        compute='_compute_vehicle_category_id', store=True, readonly=False)
    dock_id = fields.Many2one('stock.location', string="Dock Location", domain="[('warehouse_id', '=', warehouse_id), ('is_a_dock', '=', True)]",
                              compute='_compute_dock_id', store=True, readonly=False)
    vehicle_weight_capacity = fields.Float(string="Vehcilce Payload Capacity",
                              related='vehicle_category_id.weight_capacity')
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name')
    vehicle_volume_capacity = fields.Float(string="Max Volume (m³)",
                              related='vehicle_category_id.volume_capacity')
    volume_uom_name = fields.Char(string='Volume unit of measure label', compute='_compute_volume_uom_name')
    driver_id = fields.Many2one(
        'res.partner', compute="_compute_driver_id", string="Driver", store=True, readonly=False)
    used_weight_percentage = fields.Float(
        string="Weight %", compute='_compute_capacity_percentage')
    used_volume_percentage = fields.Float(
        string="Volume %", compute='_compute_capacity_percentage')
    end_date = fields.Datetime('End Date', default=fields.Datetime.now)

    # Compute
    @api.depends('vehicle_id')
    def _compute_vehicle_category_id(self):
        for rec in self:
            rec.vehicle_category_id = rec.vehicle_id.category_id

    @api.depends('picking_ids', 'picking_ids.location_id', 'picking_ids.location_dest_id')
    def _compute_dock_id(self):
        for batch in self:
            if batch.picking_ids:
                if len(batch.picking_ids.location_id) == 1 and batch.picking_ids.location_id.is_a_dock:
                    batch.dock_id = batch.picking_ids.location_id

    def _compute_weight_uom_name(self):
        self.weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    def _compute_volume_uom_name(self):
        self.volume_uom_name = self.env['product.template']._get_volume_uom_name_from_ir_config_parameter()

    @api.depends('vehicle_id')
    def _compute_driver_id(self):
        for rec in self:
            rec.driver_id = rec.vehicle_id.driver_id

    @api.depends('estimated_shipping_weight', 'vehicle_category_id.weight_capacity',
                 'estimated_shipping_volume', 'vehicle_category_id.volume_capacity')
    def _compute_capacity_percentage(self):
        self.used_weight_percentage = False
        self.used_volume_percentage = False
        for batch in self:
            if batch.vehicle_weight_capacity:
                batch.used_weight_percentage = 100 * (batch.estimated_shipping_weight / batch.vehicle_weight_capacity)
            if batch.vehicle_volume_capacity:
                batch.used_volume_percentage = 100 * (batch.estimated_shipping_volume / batch.vehicle_volume_capacity)

    # CRUD
    def create(self, vals_list):
        batches = super().create(vals_list)
        batches.order_on_zip()
        batches.filtered(lambda b: b.dock_id)._set_moves_destination_to_dock()
        return batches

    def write(self, vals):
        res = super().write(vals)
        if 'dock_id' in vals:
            self._set_moves_destination_to_dock()
        return res

    # Public actions
    def order_on_zip(self):
        sorted_records = self.picking_ids.sorted(lambda p: p.zip or "")
        for idx, record in enumerate(sorted_records):
            record.batch_sequence = idx

    # Private buisness logic
    def _set_moves_destination_to_dock(self):
        for batch in self:
            if not batch.dock_id:
                batch.picking_ids._reset_location()
            elif batch.picking_type_id.code in ["internal", "incoming"]:
                batch.picking_ids.move_ids.write({'location_dest_id': batch.dock_id.id})
            else:
                batch.picking_ids.move_ids.write({'location_id': batch.dock_id.id})

```

## File: models\__init__.py

```python
from . import fleet_vehicle_model
from . import stock_picking_batch
from . import stock_picking
from . import stock_location

```

## File: report\report_picking_batch.xml

```xml
<odoo>
    <template id="report_picking_batch_inherit" inherit_id="stock_picking_batch.report_picking_batch">
        <xpath expr="//div[hasclass('page')]/div[2]" position="after">
            <div t-if="o.dock_id">
                <strong>Dock:</strong>
                <span t-field="o.dock_id"/>
            </div>
            <div t-if="o.vehicle_id">
                <strong>Vehicle:</strong>
                <span t-field="o.vehicle_id"/>
            </div>
            <div t-if="o.vehicle_category_id">
                <strong>Vehicle Category:</strong>
                <span t-field="o.vehicle_category_id"/>
            </div><br/>
        </xpath>
        <xpath expr="//table[hasclass('table')]//thead//tr//th" position="before">
            <th>Sequence</th>
        </xpath>
        <xpath expr="//table[hasclass('table')]//tbody//tr//td" position="before">
            <td>
                <span t-field="pick.batch_sequence"/>
            </td>
        </xpath>
    </template>
</odoo>

```

## File: views\fleet_vehicle_model.xml

```xml
<odoo>
  <data>
    <record id="fleet_vehicle_model_category_view_tree_stock_fleet" model="ir.ui.view">
        <field name="name">fleet.vehicle.model.category.view.list.inherit.stock.fleet</field>
        <field name="model">fleet.vehicle.model.category</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_model_category_view_tree"/>
        <field name="arch" type="xml">
            <data>
                <field name='name' position="after">
                    <field name="weight_capacity" optional="show"/>
                    <field name="volume_capacity" optional="show"/>
                </field>
            </data>
        </field>
    </record>

    <record id="fleet_vehicle_model_category_view_form_stock_fleet" model="ir.ui.view">
        <field name="name">fleet.vehicle.model.category.view.form.inherit.stock.fleet</field>
        <field name="model">fleet.vehicle.model.category</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_model_category_view_form"/>
        <field name="arch" type="xml">
            <data>
                <field name="name" position="after">
                    <label for="weight_capacity"/>
                    <div class='d-flex flex-row gap-1'>
                        <field name="weight_capacity"/>
                        <span><field name="weight_capacity_uom_name"/></span>
                    </div>
                    <label for="volume_capacity"/>
                    <div class='d-flex flex-row gap-1'>
                        <field name="volume_capacity"/>
                        <span><field name="volume_capacity_uom_name"/></span>
                    </div>
                </field>
            </data>
        </field>
    </record>
  </data>
</odoo>

```

## File: views\stock_location.xml

```xml
<odoo>
  <data>
    <record id="stock_location_form_stock_fleet" model="ir.ui.view">
        <field name="name">stock.location.form.inherit.stock.transport</field>
        <field name="model">stock.location</field>
        <field name="inherit_id" ref="stock.view_location_form"/>
        <field name="arch" type="xml">
            <field name="scrap_location" position="after">
                <field name="is_a_dock"/>
            </field>
        </field>
    </record>

    <record id="stock_location_tree_stock_fleet" model="ir.ui.view">
        <field name="name">stock.location.list.inherit.stock.transport</field>
        <field name="model">stock.location</field>
        <field name="inherit_id" ref="stock.view_location_tree2"/>
        <field name="arch" type="xml">
            <field name="is_empty" position="after">
                <field name="is_a_dock"/>
            </field>
        </field>
    </record>
  </data>
</odoo>

```

## File: views\stock_picking_batch.xml

```xml
<odoo>
    <record id="stock_picking_batch_pivot" model="ir.ui.view">
        <field name="name">stock.picking.batch.pivot</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <pivot string="Batch Transfer" class="oe_stock_picking_batch" sample="1">
                <field name="scheduled_date" type="row"/>
                <field name="vehicle_id" type="col"/>
            </pivot>
        </field>
    </record>

    <record id="stock_picking_batch_graph" model="ir.ui.view">
        <field name="name">stock.picking.batch.graph</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <graph string="Graph View" class="oe_stock_picking_batch" sample="1">
                <field name="scheduled_date" type="row" interval="day"/>
                <field name="vehicle_category_id" type="row"/>
            </graph>
        </field>
    </record>

    <record id="stock_picking_batch_form" model="ir.ui.view">
        <field name="name">stock.picking.batch.form.inherit.stock.fleet</field>
        <field name="model">stock.picking.batch</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@id='batch_delivery_data']" position="after">
                <group>
                    <field name="dock_id" groups="stock.group_stock_multi_locations"/>
                    <field name="vehicle_id" placeholder="Third Party Provider"/>
                    <field name="vehicle_category_id" placeholder="semi-truck"/>
                    <label for='used_weight_percentage' string="Weight" invisible="not vehicle_category_id or not vehicle_weight_capacity"/>
                        <div class='d-flex flex-row gap-4' invisible="not vehicle_category_id or not vehicle_weight_capacity">
                            <div class='d-flex flex-row gap-1'>
                                <field name='estimated_shipping_weight'/>
                                <span><field name='weight_uom_name'/></span>
                            </div>
                            <field name='used_weight_percentage' widget='progressbar'/>
                        </div>

                    <label for='used_volume_percentage' string="Volume" invisible="not vehicle_category_id or not vehicle_volume_capacity"/>
                        <div class='d-flex flex-row gap-4' invisible="not vehicle_category_id or not vehicle_volume_capacity">
                            <div class='d-flex flex-row gap-1'>
                                <field name='estimated_shipping_volume'/>
                                <span><field name='volume_uom_name'/></span>
                            </div>
                            <field name='used_volume_percentage' widget='progressbar'/>
                        </div>
                </group>
            </xpath>
        </field>
    </record>

    <record id="stock_picking_batch_tree" model="ir.ui.view">
        <field name="name">stock.picking.batch.list.inherit.stock.fleet</field>
        <field name="model">stock.picking.batch</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_tree"/>
        <field name="arch" type="xml">
            <data>
                <field name="user_id" position="attributes">
                    <attribute name="optional">show</attribute>
                </field>
                <field name="user_id" position="after">
                    <field name="vehicle_category_id" optional="hide"/>
                    <field name="vehicle_id" optional="hide"/>
                    <field name="dock_id" optional="hide" groups="stock.group_stock_multi_locations"/>
                    <field name="used_volume_percentage" optional="hide"/>
                    <field name="used_weight_percentage" optional="hide"/>
                </field>
            </data>
        </field>
    </record>

    <record id="stock_picking_batch_filter" model="ir.ui.view">
        <field name="name">stock.picking.batch.filter.inherit.stock.fleet</field>
        <field name="model">stock.picking.batch</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_filter"/>
        <field name="arch" type="xml">
            <field name="user_id" position="after">
                <field name="vehicle_id"/>
                <field name="dock_id"/>
                <field name="driver_id"/>
            </field>
            <xpath expr="//filter[@name='state']" position="after">
                <filter name="group_by_vehicle_id" string="Vehicle" context="{'group_by':'vehicle_id'}"/>
                <filter name="group_by_vehicle_category_id" string="Vehicle Category" context="{'group_by':'vehicle_category_id'}"/>
                <filter name="group_by_scheduled_date" string="Scheduled Date" context="{'group_by':'scheduled_date'}"/>
                <filter name="group_by_picking_type_id" string="Operation Type" context="{'group_by':'picking_type_id'}"/>
                <filter name="group_by_dock_id" string="Dock Location" context="{'group_by':'dock_id'}" groups="stock.group_stock_multi_locations"/>
            </xpath>
            <xpath expr="//filter[@name='done']" position="after">
                <filter name="vehicle_id" string="Own Fleet" domain="[('vehicle_id', '!=', False)]"/>
                <filter name="vehicle_id" string="Third Party Carrier" domain="[('vehicle_id', '=', False), ('vehicle_category_id', '!=', False)]"/>
            </xpath>
            <xpath expr="//filter[@name='my_transfers']" position="after">
                <filter name="scheduled_date" date="scheduled_date" string="Scheduled Date"/>
                <filter string="Today" name="filter_today" domain="[('scheduled_date', '&gt;=', datetime.datetime.combine(context_today(),
                    datetime.time(0,0,0))), ('scheduled_date', '&lt;=', datetime.datetime.combine(context_today(), datetime.time(23,59,59)))]"/>
                <filter string="Tomorrow" name="filter_tomorrow" domain="[('scheduled_date','&gt;=', (context_today() + relativedelta(days=1)).strftime('%Y-%m-%d'))]" />
                <filter string="Next 7 Days" name="filter_next_7_days" domain="[('scheduled_date','&gt;=', (context_today() + relativedelta(days=7)).strftime('%Y-%m-%d'))]"/>
            </xpath>
        </field>
    </record>

    <record id="stock_picking_batch_kanban" model="ir.ui.view">
        <field name="name">stock.picking.batch.kanban.inherit.stock.fleet</field>
        <field name="model">stock.picking.batch</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_kanban"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//footer" position="replace">
                    <footer class="pt-0">
                        <field name="dock_id"/>
                        <div>
                            <field name="state" widget="state_selection" class="float-start pt-1 me-1"/>
                            <field name="scheduled_date" readonly="state in ['cancel', 'done']"/>
                        </div>
                        <field name="user_id" widget="many2one_avatar_user" readonly="state not in ['draft', 'in_progress']" class="ms-auto"/>
                    </footer>
                </xpath>
            </data>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_type.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="stock_picking_type_kanban_inherit_stock_fleet" model="ir.ui.view">
            <field name="name">stock.picking.type.kanban.inherit.stock.transport</field>
            <field name="model">stock.picking.type</field>
            <field name="inherit_id" ref="stock.stock_picking_type_kanban"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//div[@name='kanban_menu_section']" position="after">
                        <t t-if="record.code.raw_value == 'incoming' or record.code.raw_value == 'outgoing'">
                            <div class="col-6">
                                <h5 role="menuitem" class="o_kanban_card_manage_title" style="white-space: nowrap;">
                                    <a>Transport Management</a>
                                </h5>
                                <div role="menuitem">
                                    <a name="action_batch" type="object" context="{'view_mode':'list,form'}">Manage Batches</a>
                                </div>
                                <div role="menuitem">
                                    <a name="action_batch" type="object" context="{'view_mode':'gantt'}">Dock Dispatching</a>
                                </div>
                                <div role="menuitem">
                                    <a name="action_batch" type="object" context="{'view_mode':'kanban'}">Batches by Route</a>
                                </div>
                                <div role="menuitem">
                                    <a name="action_batch" type="object" context="{'view_mode':'calendar'}">Calendar</a>
                                </div>
                                <div role="menuitem">
                                    <a name="action_batch" type="object" context="{'view_mode':'pivot'}">Statistics</a>
                                </div>
                            </div>
                        </t>
                    </xpath>
                </data>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\stock_picking_view.xml

```xml
<odoo>
    <record id="vpicktree" model="ir.ui.view">
        <field name="name">stock.picking.list.inherit.stock.fleet</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.vpicktree"/>
        <field name="arch" type="xml">
            <field name="picking_type_id" position="after">
                <field name="zip" optional="hide"/>
                <field name="shipping_weight" sum="Total Shipping Weight" string="Shipping Weight" optional="hide"/>
                <field name="shipping_volume" sum="Total Shipping Volume" string="Shipping Volume" optional="hide"/>
            </field>
        </field>
    </record>
    <record id="stock_picking_tree_inherit_stock_fleet" model="ir.ui.view">
        <field name="name">stock.picking.list.inherit.stock.transport</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_view_batch_tree_ref"/>
        <field name="arch" type="xml">
            <field name="zip" position="attributes">
                <attribute name="optional">show</attribute>
            </field>
        </field>
    </record>
</odoo>

```

