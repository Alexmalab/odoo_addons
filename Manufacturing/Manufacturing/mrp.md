# Odoo Module: mrp

Category: Manufacturing/Manufacturing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard
from . import report

from odoo import api, SUPERUSER_ID


def _pre_init_mrp(cr):
    """ Allow installing MRP in databases with large stock.move / stock.move.line tables (>1M records)
        - Creating the computed+stored field stock_move.is_done is terribly slow with the ORM and
          leads to "Out of Memory" crashes
        - stock.move.line.done_move is a stored+related on the former... 
        - Also set the default value for unit_factor in the same UPDATE query to save some SQL constraint checks"""
    cr.execute("""ALTER TABLE "stock_move" ADD COLUMN "unit_factor" float;""")
    cr.execute("""ALTER TABLE "stock_move" ADD COLUMN "is_done" bool;""")
    cr.execute("""ALTER TABLE "stock_move_line" ADD COLUMN "done_move" bool;""")
    cr.execute("""UPDATE stock_move
                     SET is_done=COALESCE(state in ('done', 'cancel'), FALSE),
                         unit_factor=1.0;""")
    cr.execute("""UPDATE stock_move_line
                     SET done_move=sm.is_done
                    FROM stock_move sm
                   WHERE move_id=sm.id;""")

def _create_warehouse_data(cr, registry):
    """ This hook is used to add a default manufacture_pull_id, manufacture
    picking_type on every warehouse. It is necessary if the mrp module is
    installed after some warehouses were already created.
    """
    env = api.Environment(cr, SUPERUSER_ID, {})
    warehouse_ids = env['stock.warehouse'].search([('manufacture_pull_id', '=', False)])
    warehouse_ids.write({'manufacture_to_resupply': True})

def uninstall_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    warehouses = env["stock.warehouse"].search([])
    pbm_routes = warehouses.mapped("pbm_route_id")
    warehouses.write({"pbm_route_id": False})
    # Fail unlink means that the route is used somewhere (e.g. route_id on stock.rule). In this case
    # we don't try to do anything.
    try:
        with env.cr.savepoint():
            pbm_routes.unlink()
    except:
        pass


```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Manufacturing',
    'version': '2.0',
    'website': 'https://www.odoo.com/page/manufacturing',
    'category': 'Manufacturing/Manufacturing',
    'sequence': 16,
    'summary': 'Manufacturing Orders & BOMs',
    'depends': ['product', 'stock', 'resource'],
    'description': "",
    'data': [
        'security/mrp_security.xml',
        'security/ir.model.access.csv',
        'data/mrp_data.xml',
        'wizard/mrp_product_produce_views.xml',
        'wizard/change_production_qty_views.xml',
        'wizard/mrp_workcenter_block_view.xml',
        'wizard/stock_warn_insufficient_qty_views.xml',
        'views/mrp_views_menus.xml',
        'views/stock_move_views.xml',
        'views/mrp_workorder_views.xml',
        'views/mrp_workcenter_views.xml',
        'views/mrp_production_views.xml',
        'views/mrp_routing_views.xml',
        'views/mrp_bom_views.xml',
        'views/product_views.xml',
        'views/stock_warehouse_views.xml',
        'views/stock_picking_views.xml',
        'views/mrp_unbuild_views.xml',
        'views/ir_attachment_view.xml',
        'views/res_config_settings_views.xml',
        'views/mrp_templates.xml',
        'views/stock_scrap_views.xml',
        'report/mrp_report_views_main.xml',
        'report/mrp_report_bom_structure.xml',
        'report/mrp_production_templates.xml',
        'report/report_stock_rule.xml',
        'report/mrp_zebra_production_templates.xml',
    ],
    'qweb': ['static/src/xml/mrp.xml'],
    'demo': [
        'data/mrp_demo.xml',
    ],
    'test': [],
    'application': True,
    'pre_init_hook': '_pre_init_mrp',
    'post_init_hook': '_create_warehouse_data',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: data\mrp_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="sequence_mrp_route" model="ir.sequence">
            <field name="name">Routing</field>
            <field name="code">mrp.routing</field>
            <field name="prefix">RO/</field>
            <field name="padding">5</field>
            <field name="number_next">1</field>
            <field name="number_increment">1</field>
            <field name="company_id" eval="False"/>
        </record>
        <function model="res.company" name="create_missing_unbuild_sequences" />
        <!--
             Stock rules and routes
        -->

        <record id="route_warehouse0_manufacture" model='stock.location.route'>
            <field name="name">Manufacture</field>
            <field name="company_id"></field>
            <field name="sequence">5</field>
        </record>

        <!-- Enable the manufacturing in warehouse0 -->
        <record id='stock.warehouse0' model='stock.warehouse'>
            <field name='manufacture_to_resupply' eval='True'/>
        </record>

        <!--  Category for Blocking Reasons Workcenter -->
        <record id="category_availability" model="mrp.workcenter.productivity.loss.type">
            <field name="loss_type">availability</field>
        </record>
        <record id="category_performance" model="mrp.workcenter.productivity.loss.type">
            <field name="loss_type">performance</field>
        </record>
        <record id="category_quality" model="mrp.workcenter.productivity.loss.type">
            <field name="loss_type">quality</field>
        </record>
        <record id="category_productive" model="mrp.workcenter.productivity.loss.type">
            <field name="loss_type">productive</field>
        </record>

        <!-- Reasons To Block Workcenter -->
        <record id="block_reason0" model="mrp.workcenter.productivity.loss">
            <field name="name">Material Availability</field>
            <field name="loss_id" ref="mrp.category_availability"></field>
            <field name="sequence">1</field>
        </record>
        <record id="block_reason1" model="mrp.workcenter.productivity.loss">
            <field name="name">Equipment Failure</field>
            <field name="loss_id" ref="mrp.category_availability"></field>
            <field name="sequence">2</field>
        </record>
        <record id="block_reason2" model="mrp.workcenter.productivity.loss">
            <field name="name">Setup and Adjustments</field>
            <field name="loss_id" ref="mrp.category_availability"></field>
            <field name="sequence">3</field>
        </record>
        <record id="block_reason4" model="mrp.workcenter.productivity.loss">
            <field name="name">Reduced Speed</field>
            <field name="loss_id" ref="mrp.category_performance"></field>
            <field name="manual" eval="False"/>
            <field name="sequence">5</field>
        </record>
        <record id="block_reason5" model="mrp.workcenter.productivity.loss">
            <field name="name">Process Defect</field>
            <field name="loss_id" ref="mrp.category_quality"></field>
            <field name="sequence">6</field>
        </record>
        <record id="block_reason6" model="mrp.workcenter.productivity.loss">
            <field name="name">Reduced Yield</field>
            <field name="loss_id" ref="mrp.category_quality"></field>
            <field name="sequence">7</field>
        </record>
        <record id="block_reason7" model="mrp.workcenter.productivity.loss">
            <field name="name">Fully Productive Time</field>
            <field name="loss_id" ref="mrp.category_productive"></field>
            <field name="manual" eval="False"/>
            <field name="sequence">0</field>
        </record>

    </data>

</odoo>

```

## File: data\mrp_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="base.user_demo" model="res.users">
            <field eval="[(4, ref('group_mrp_user'))]" name="groups_id"/>
        </record>

        <!-- Resource: res.company -->
        <record id="stock.res_company_1" model="res.company">
            <field eval="1.0" name="manufacturing_lead"/>
        </record>

        <!-- Resource: mrp.workcenter -->

        <record id="mrp_workcenter_3" model="mrp.workcenter">
            <field name="name">Assembly Line 1</field>
            <field name="resource_calendar_id" ref="resource.resource_calendar_std"/>
        </record>

        <record id="mrp_workcenter_1" model="mrp.workcenter">
            <field name="name">Drill Station 1</field>
            <field name="resource_calendar_id" ref="resource.resource_calendar_std"/>
        </record>

        <record id="mrp_workcenter_2" model="mrp.workcenter">
            <field name="name">Assembly Line 2</field>
            <field name="resource_calendar_id" ref="resource.resource_calendar_std"/>
        </record>



        <!-- Resource: mrp.routing -->

        <record id="mrp_routing_0" model="mrp.routing">
            <field name="name">Primary Assembly</field>
        </record>

        <record id="mrp_routing_1" model="mrp.routing">
            <field name="name">Secondary Assembly</field>
        </record>

        <record id="mrp_routing_2" model="mrp.routing">
            <field name="name">Manual Component's Assembly</field>
        </record>

        <record id="mrp_routing_3" model="mrp.routing">
            <field name="name">Assemble Furniture</field>
        </record>


        <!-- Resource: mrp.routing.workcenter -->

        <record id="mrp_routing_workcenter_0" model="mrp.routing.workcenter">
            <field name="routing_id" ref="mrp_routing_0"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Manual Assembly</field>
            <field name="time_cycle">60</field>
            <field name="sequence">5</field>
            <field name="worksheet" type="base64" file="mrp/static/img/assebly-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_1" model="mrp.routing.workcenter">
            <field name="routing_id" ref="mrp_routing_1"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Long time assembly</field>
            <field name="time_cycle">180</field>
            <field name="sequence">15</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_3" model="mrp.routing.workcenter">
            <field name="routing_id" ref="mrp_routing_1"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Testing</field>
            <field name="time_cycle">60</field>
            <field name="sequence">10</field>
            <field name="worksheet" type="base64" file="mrp/static/img/assebly-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_4" model="mrp.routing.workcenter">
            <field name="routing_id" ref="mrp_routing_1"/>
            <field name="workcenter_id" ref="mrp_workcenter_1"/>
            <field name="name">Packing</field>
            <field name="time_cycle">30</field>
            <field name="sequence">5</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_2" model="mrp.routing.workcenter">
            <field name="routing_id" ref="mrp_routing_2"/>
            <field name="workcenter_id" ref="mrp_workcenter_2"/>
            <field name="time_cycle">120</field>
            <field name="sequence">5</field>
            <field name="name">Manual Assembly</field>
            <field name="worksheet" type="base64" file="mrp/static/img/assebly-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_5" model="mrp.routing.workcenter">
            <field name="routing_id" ref="mrp_routing_3"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="time_cycle">120</field>
            <field name="sequence">10</field>
            <field name="name">Assembly Line 1</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
        </record>

        <!-- Resource: mrp.bom -->

        <record id="product.product_product_3_product_template" model="product.template">
            <field name="route_ids" eval="[(6, 0, [ref('stock.route_warehouse0_mto'), ref('mrp.route_warehouse0_manufacture')])]"/>
        </record>
        <record id="mrp_bom_manufacture" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_product_3_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="routing_id" ref="mrp_routing_0"/>
        </record>

        <record id="mrp_bom_manufacture_line_1" model="mrp.bom.line">
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">5</field>
            <field name="bom_id" ref="mrp_bom_manufacture"/>
        </record>

        <record id="mrp_bom_manufacture_line_2" model="mrp.bom.line">
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">5</field>
            <field name="bom_id" ref="mrp_bom_manufacture"/>
        </record>

        <record id="mrp_bom_manufacture_line_3" model="mrp.bom.line">
            <field name="product_id" ref="product.product_product_16"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">5</field>
            <field name="bom_id" ref="mrp_bom_manufacture"/>
        </record>

        <record id="mrp_production_1" model="mrp.production">
            <field name="product_id" ref="product.product_product_3"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">3</field>
            <field name="bom_id" ref="mrp_bom_manufacture"/>
        </record>

        <function model="stock.move" name="create">
            <value model="stock.move" eval="obj().env.ref('mrp.mrp_production_1')._get_moves_raw_values()"/>
        </function>

        <!-- Table -->

        <record id="product_product_computer_desk" model="product.product">
            <field name="name">Table (MTO)</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">290</field>
            <field name="list_price">520</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Solid wood table.</field>
            <field name="default_code">FURN_9666</field>
            <field name="tracking">serial</field>
            <field name="image_1920" type="base64" file="mrp/static/img/table.png"/>
        </record>
        <record id="product_product_computer_desk_head" model="product.product">
            <field name="name">Table Top</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">240</field>
            <field name="list_price">380</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Solid wood is a durable natural material.</field>
            <field name="default_code">FURN_8522</field>
            <field name="tracking">serial</field>
            <field name="image_1920" type="base64" file="mrp/static/img/table_top.png"/>
        </record>
        <record id="product_product_computer_desk_leg" model="product.product">
            <field name="name">Table Leg</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">10</field>
            <field name="list_price">50</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">18″ x 2½″ Square Leg</field>
            <field name="default_code">FURN_2333</field>
            <field name="tracking">lot</field>
            <field name="image_1920" type="base64" file="mrp/static/img/table_leg.png"/>
        </record>
        <record id="product_product_computer_desk_bolt" model="product.product">
            <field name="name">Bolt</field>
            <field name="categ_id" ref="product.product_category_consumable"/>
            <field name="standard_price">0.5</field>
            <field name="list_price">0.5</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Stainless steel screw full (dia - 5mm, Length - 10mm)</field>
            <field name="default_code">CONS_89957</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_computer_desk_bolt.png"/>
        </record>
        <record id="product_product_computer_desk_screw" model="product.product">
            <field name="name">Screw</field>
            <field name="categ_id" ref="product.product_category_consumable"/>
            <field name="standard_price">0.1</field>
            <field name="list_price">0.2</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Stainless steel screw</field>
            <field name="default_code">CONS_25630</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_computer_desk_screw.png"/>
        </record>

        <record id="product_product_wood_ply" model="product.product">
            <field name="name">Ply Layer</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">10</field>
            <field name="list_price">10</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Layers that are stick together to assemble wood panels.</field>
            <field name="default_code">FURN_7111</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_wood_ply.png"/>
        </record>
        <record id="product_product_wood_wear" model="product.product">
            <field name="name">Wear Layer</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">10</field>
            <field name="list_price">10</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Top layer of a wood panel.</field>
            <field name="default_code">FURN_8111</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_wood_wear.png"/>
        </record>
        <record id="product_product_ply_veneer" model="product.product">
            <field name="name">Ply Veneer</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">10</field>
            <field name="list_price">10</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_9111</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_ply_veneer.png"/>
        </record>

        <record id="product_product_wood_panel" model="product.product">
            <field name="name">Wood Panel</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">80</field>
            <field name="list_price">100</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_7023</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_wood_panel.png"/>
        </record>
        <record id="product_product_plastic_laminate" model="product.product">
            <field name="name">Plastic Laminate</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">3000</field>
            <field name="list_price">1000</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_8621</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_plastic_laminate.png"/>
        </record>

        <record id="product_product_computer_desk_product_template" model="product.template">
            <field name="route_ids" eval="[(6, 0, [ref('stock.route_warehouse0_mto'), ref('mrp.route_warehouse0_manufacture')])]"/>
        </record>

        <record id="mrp_bom_desk" model="mrp.bom">
            <field name="product_tmpl_id" ref="product_product_computer_desk_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">3</field>
            <field name="consumption">flexible</field>
            <field name="routing_id" ref="mrp_routing_3"/>
        </record>

        <record id="mrp_bom_desk_line_1" model="mrp.bom.line">
            <field name="product_id" ref="product_product_computer_desk_head"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_desk"/>
            <field name="operation_id" ref="mrp.mrp_routing_workcenter_5"/>
        </record>

        <record id="mrp_bom_desk_line_2" model="mrp.bom.line">
            <field name="product_id" ref="product_product_computer_desk_leg"/>
            <field name="product_qty">4</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="bom_id" ref="mrp_bom_desk"/>
            <field name="operation_id" ref="mrp.mrp_routing_workcenter_5"/>
        </record>

        <record id="mrp_bom_desk_line_3" model="mrp.bom.line">
            <field name="product_id" ref="product_product_computer_desk_bolt"/>
            <field name="product_qty">4</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">3</field>
            <field name="bom_id" ref="mrp_bom_desk"/>
            <field name="operation_id" ref="mrp.mrp_routing_workcenter_5"/>
        </record>

        <record id="mrp_bom_desk_line_4" model="mrp.bom.line">
            <field name="product_id" ref="product_product_computer_desk_screw"/>
            <field name="product_qty">10</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">4</field>
            <field name="bom_id" ref="mrp_bom_desk"/>
            <field name="operation_id" ref="mrp.mrp_routing_workcenter_5"/>
        </record>

        <record id="mrp_production_3" model="mrp.production">
            <field name="product_id" ref="product_product_computer_desk"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">1</field>
            <field name="bom_id" ref="mrp_bom_desk"/>
        </record>

        <function model="stock.move" name="create">
            <value model="stock.move" eval="obj().env.ref('mrp.mrp_production_3')._get_moves_raw_values()"/>
        </function>

        <record id="mrp_bom_table_top" model="mrp.bom">
            <field name="product_tmpl_id" ref="product_product_computer_desk_head_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="routing_id" ref="mrp_routing_0"/>
        </record>
        <record id="mrp_bom_line_wood_panel" model="mrp.bom.line">
            <field name="product_id" ref="product_product_wood_panel"/>
            <field name="product_qty">2</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_table_top"/>
        </record>
        <record id="mrp_bom_line_plastic_laminate" model="mrp.bom.line">
            <field name="product_id" ref="product_product_plastic_laminate"/>
            <field name="product_qty">4</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="bom_id" ref="mrp_bom_table_top"/>
        </record>

        <record id="mrp_bom_plastic_laminate" model="mrp.bom">
            <field name="product_tmpl_id" ref="product_product_plastic_laminate_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="routing_id" ref="mrp_routing_1"/>
        </record>
        <record id="mrp_bom_line_plastic_laminate" model="mrp.bom.line">
            <field name="product_id" ref="product_product_ply_veneer"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_plastic_laminate"/>
        </record>

        <record id="mrp_bom_wood_panel" model="mrp.bom">
            <field name="product_tmpl_id" ref="product_product_wood_panel_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
        </record>
        <record id="mrp_bom_line_wood_panel_ply" model="mrp.bom.line">
            <field name="product_id" ref="product_product_wood_ply"/>
            <field name="product_qty">3</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_wood_panel"/>
        </record>
        <record id="mrp_bom_line_wood_panel_wear" model="mrp.bom.line">
            <field name="product_id" ref="product_product_wood_wear"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_wood_panel"/>
        </record>

        <!-- Table Kit -->

        <record id="product_product_table_kit" model="product.product">
            <field name="name">Table Kit</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">600.0</field>
            <field name="list_price">147.0</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Table kit</field>
            <field name="default_code">FURN_78236</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_table_kit.png"/>
        </record>

         <record id="product_product_table_kit_product_template" model="product.template">
            <field name="route_ids" eval="[(6, 0, [ref('mrp.route_warehouse0_manufacture')])]"/>
        </record>

        <record id="mrp_bom_kit" model="mrp.bom">
            <field name="product_tmpl_id" ref="product_product_table_kit_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="type">phantom</field>
        </record>

        <record id="mrp_bom_kit_line_1" model="mrp.bom.line">
            <field name="product_id" ref="product_product_wood_panel"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="bom_id" ref="mrp_bom_kit"/>
        </record>

        <record id="mrp_bom_kit_line_2" model="mrp.bom.line">
            <field name="product_id" ref="product_product_computer_desk_bolt"/>
            <field name="product_qty">4</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="bom_id" ref="mrp_bom_kit"/>
        </record>


        <!-- Manufacturing Order Demo With Lots-->

        <record id="product_product_drawer_drawer" model="product.product">
            <field name="name">Drawer Black</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="tracking">lot</field>
            <field name="standard_price">2000.0</field>
            <field name="list_price">2250.0</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Drawer on casters for great usability.</field>
            <field name="default_code">FURN_2100</field>
            <field name="barcode">601647855646</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_drawer_black.png"/>
        </record>

        <record id="product_product_drawer_case" model="product.product">
            <field name="name">Drawer Case Black</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="tracking">lot</field>
            <field name="standard_price">800</field>
            <field name="list_price">850</field>
            <field name="type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_5623</field>
            <field name="barcode">601647855647</field>
            <field name="image_1920" type="base64" file="mrp/static/img/product_product_drawer_case_black.png"/>
        </record>

        <record id="product.product_product_27" model="product.product">
            <field name="tracking">lot</field>
        </record>

        <record id="lot_product_27_0" model="stock.production.lot">
            <field name="name">0000000000030</field>
            <field name="product_id" ref="product.product_product_27"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record id="lot_product_product_drawer_drawer_0" model="stock.production.lot">
            <field name="name">0000000010001</field>
            <field name="product_id" ref="product_product_drawer_drawer"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record id="lot_product_product_drawer_case_0" model="stock.production.lot">
            <field name="name">0000000020045</field>
            <field name="product_id" ref="product_product_drawer_case"/>
            <field name="company_id" ref="base.main_company"/>
        </record>


        <!-- Initital Inventory -->

        <record id="stock_inventory_drawer" model="stock.inventory">
            <field name="name">Inventory: Drawer + Kit</field>
        </record>

        <record id="stock_inventory_drawer_lot0" model="stock.inventory.line">
            <field name="product_id" ref="product.product_product_27"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">50</field>
            <field name="prod_lot_id" ref="lot_product_27_0"/>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>
        <record id="stock_inventory_drawer_lot1" model="stock.inventory.line">
            <field name="product_id" ref="product.product_product_27"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">40</field>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>
        <record id="stock_inventory_line_product_drawer_drawer" model="stock.inventory.line">
            <field name="product_id" ref="product_product_drawer_drawer"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">50</field>
            <field name="prod_lot_id" ref="lot_product_product_drawer_drawer_0"/>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>
        <record id="stock_inventory_line_product_drawer_case" model="stock.inventory.line">
            <field name="product_id" ref="product_product_drawer_case"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">50</field>
            <field name="prod_lot_id" ref="lot_product_product_drawer_case_0"/>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>

        <record id="stock_inventory_line_product_wood_panel" model="stock.inventory.line">
            <field name="product_id" ref="product_product_wood_panel"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">50</field>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>
        <record id="stock_inventory_line_product_ply" model="stock.inventory.line">
            <field name="product_id" ref="product_product_wood_ply"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">20</field>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>
        <record id="stock_inventory_line_product_wear" model="stock.inventory.line">
            <field name="product_id" ref="product_product_wood_wear"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">30</field>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>
        <record id="stock_inventory_line_product_table_kit" model="stock.inventory.line">
            <field name="product_id" ref="product_product_table_kit"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_drawer"/>
            <field name="product_qty">30</field>
            <field name="location_id" ref="stock.stock_location_14"/>
        </record>

        <function model="stock.inventory" name="_action_done">
            <function eval="[[('state','=','draft'), ('id', '=', ref('stock_inventory_drawer'))]]" model="stock.inventory" name="search"/>
        </function>

        <!-- BoM -->

        <record id="mrp_bom_laptop_cust" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_product_27_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="code">PRIM-ASSEM</field>
        </record>
        <record id="mrp_bom_laptop_cust_line_1" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_drawer"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_laptop_cust"/>
        </record>
        <record id="mrp_bom_laptop_cust_line_2" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_case"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="bom_id" ref="mrp_bom_laptop_cust"/>
        </record>

        <record id="mrp_bom_laptop_cust_rout" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_product_27_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="routing_id" ref="mrp_routing_1"/>
            <field name="code">SEC-ASSEM</field>
        </record>
        <record id="mrp_bom_laptop_cust_rout_line_1" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_drawer"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_laptop_cust_rout"/>
        </record>
        <record id="mrp_bom_laptop_cust_rout_line_2" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_case"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="bom_id" ref="mrp_bom_laptop_cust_rout"/>
        </record>

        <record id="product.product_product_27" model="product.product">
            <field name="type">product</field>
        </record>
        <record id="mrp_production_laptop_cust" model="mrp.production">
            <field name="product_id" ref="product.product_product_27"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">5</field>
            <field name="location_src_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_stock"/>
            <field name="bom_id" ref="mrp_bom_laptop_cust"/>
        </record>

        <function model="stock.move" name="create">
            <value model="stock.move" eval="obj().env.ref('mrp.mrp_production_laptop_cust')._get_moves_raw_values()"/>
        </function>

        <!-- Run Scheduler -->
        <function model="procurement.group" name="run_scheduler"/>


        <!-- OEE -->

        <record id="mrp_workcenter_efficiency_0" model="mrp.workcenter.productivity">
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="loss_id" ref="block_reason7"/>
            <field name="date_start" eval="(datetime.now() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_end" eval="(datetime.now() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="mrp_workcenter_efficiency_1" model="mrp.workcenter.productivity">
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="loss_id" ref="block_reason0"/>
            <field name="date_start" eval="(datetime.now() - timedelta(hours=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_end" eval="(datetime.now() - timedelta(hours=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="mrp_workcenter_efficiency_2" model="mrp.workcenter.productivity">
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="loss_id" ref="block_reason1"/>
            <field name="date_start" eval="(datetime.now() - timedelta(days=5, hours=4)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_end" eval="(datetime.now() - timedelta(days=5, hours=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="mrp_workcenter_efficiency_3" model="mrp.workcenter.productivity">
            <field name="workcenter_id" ref="mrp_workcenter_1"/>
            <field name="loss_id" ref="block_reason7"/>
            <field name="date_start" eval="(datetime.now() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_end" eval="(datetime.now() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="mrp_workcenter_efficiency_4" model="mrp.workcenter.productivity">
            <field name="workcenter_id" ref="mrp_workcenter_1"/>
            <field name="loss_id" ref="block_reason0"/>
            <field name="date_start" eval="(datetime.now() - timedelta(days=5,hours=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_end" eval="(datetime.now() - timedelta(days=5,hours=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="mrp_workcenter_efficiency_5" model="mrp.workcenter.productivity">
            <field name="workcenter_id" ref="mrp_workcenter_1"/>
            <field name="loss_id" ref="block_reason1"/>
            <field name="date_start" eval="(datetime.now() - timedelta(hours=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <function model="mrp.production" name="action_confirm">
            <value eval="[ref('mrp.mrp_production_laptop_cust')]"/>
        </function>

        <function model="mrp.production" name="action_assign">
            <value eval="[ref('mrp.mrp_production_laptop_cust')]"/>
        </function>

        <function model="mrp.product.produce" name="create">
            <value model="mrp.product.produce" eval="dict(
                obj().with_context(active_id=ref('mrp.mrp_production_laptop_cust')).default_get(list(obj().fields_get())),
                **{
                    'qty_producing': obj().env.ref('mrp.mrp_production_laptop_cust').product_qty,
                    'finished_lot_id': ref('mrp.lot_product_27_0'),
                }
            )"/>
        </function>

        <function model="mrp.product.produce.line" name="create">
            <value model="mrp.product.produce.line" eval="[dict(item,
                 **{
                     'raw_product_produce_id': obj().env['mrp.product.produce'].search([('production_id', '=', obj().env.ref('mrp.mrp_production_laptop_cust').id)]).id
                 }) for item in  obj().env['mrp.product.produce'].search([('production_id', '=', obj().env.ref('mrp.mrp_production_laptop_cust').id)])._update_workorder_lines()['to_create']]"/>
        </function>

        <function model="mrp.product.produce" name="do_produce">
            <value model="mrp.product.produce" search="[('production_id', '=', obj().env.ref('mrp.mrp_production_laptop_cust').id)]"/>
        </function>

        <function model="mrp.production" name="post_inventory">
            <value eval="[ref('mrp.mrp_production_laptop_cust')]"/>
        </function>

        <function model="mrp.production" name="button_mark_done">
            <value eval="[ref('mrp.mrp_production_laptop_cust')]"/>
        </function>

        <!-- set 'create component' as True for the demo manufacturing picking type
        while leaving the default value to False for the others -->
        <function model="stock.warehouse" name="write">
            <value model="stock.warehouse" eval="obj().env['stock.warehouse'].search([]).ids"/>
            <value eval="{'manufacture_to_resupply': True}"/>
        </function>

        <function model="stock.picking.type" name="write">
            <value model="stock.picking.type" eval="obj().env['stock.picking.type'].search([('code', '=', 'mrp_operation')]).ids"/>
            <value eval="{'use_create_components_lots': True}"/>
        </function>


    </data>
</odoo>

```

## File: doc\mrp.rst

```rst
MRP 

In this chapter, we are going to explain briefly how to use mrp in its simplest form (without the mrp operations) and the impact on stock.  We suppose you know the terms used in the warehouse documentation.  

Order flow
**********

An order can be created manually or through the creation of a procurement.  

Important for stock management are the raw materials location and finished products location.  Creating a manufacturing order manually, this is by default equal to the location of the standard warehouse.  In case of a manufacturing order from a procurement, it will have the source location of the applied procurement rule as both the raw materials and finished products location.  

For a draft manufacturing order, you can change the products to consume, but not anything else.  The consumed products tab and the finished products tab will give the progress of the raw materials and finished products consumed/produced.  A left table described what needs to be produced and the right table what has been produced/consumed.  Behind the scenes, these are stock moves from the raw materials location to the production location (virtual) or from the production location (virtual) to the finished products location. 

First, the system will calculate the scheduled products from the bill of material and the quantity necessary.  These will be added to the scheduled products (third tab) and to the products to consume upon confirmation of the manufacturing order.  

The manufacturing order becomes ready to produce when the different moves are assigned (ready to transfer).  It is also possible to force the reservation (not recommended).  

Then you can also Mark as Started. 

When you click Produce, a wizard will be opened, where you can enter the quantity produced.  Based upon the quantity of finished goods produced, OpenERP will give you the theoretical amount that should still be produced.  The theoretical amount is actualy the ('percentage of finished products after this production' * scheduled amount of raw materials) - raw materials consumed.  

The advantage to the wizard  (instead of the arrows in v7) is in case of traceability.  Actually, if the raw materials and the finished goods require traceability, you should be able to specify for which finished good the raw material has been used. (requires lot for finished product and raw material)  That way, it is possible to do the full traceability in production.  

It is possible also to cancel a production order.  The products consumed/produced until then, will stay, but every move left, will be cancelled.  













```

## File: models\mrp_abstract_workorder.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_round, float_is_zero


class MrpAbstractWorkorder(models.AbstractModel):
    _name = "mrp.abstract.workorder"
    _description = "Common code between produce wizards and workorders."
    _check_company_auto = True

    production_id = fields.Many2one('mrp.production', 'Manufacturing Order', required=True, check_company=True)
    product_id = fields.Many2one(related='production_id.product_id', readonly=True, store=True, check_company=True)
    qty_producing = fields.Float(string='Currently Produced Quantity', digits='Product Unit of Measure')
    product_uom_id = fields.Many2one('uom.uom', 'Unit of Measure', required=True, readonly=True)
    finished_lot_id = fields.Many2one(
        'stock.production.lot', string='Lot/Serial Number',
        domain="[('product_id', '=', product_id), ('company_id', '=', company_id)]", check_company=True)
    product_tracking = fields.Selection(related="product_id.tracking")
    consumption = fields.Selection([
        ('strict', 'Strict'),
        ('flexible', 'Flexible')],
        required=True,
    )
    use_create_components_lots = fields.Boolean(related="production_id.picking_type_id.use_create_components_lots")
    company_id = fields.Many2one(related='production_id.company_id')

    @api.model
    def _prepare_component_quantity(self, move, qty_producing):
        """ helper that computes quantity to consume (or to create in case of byproduct)
        depending on the quantity producing and the move's unit factor"""
        if move.product_id.tracking == 'serial':
            uom = move.product_id.uom_id
        else:
            uom = move.product_uom
        return move.product_uom._compute_quantity(
            qty_producing * move.unit_factor,
            uom,
            round=False
        )

    def _workorder_line_ids(self):
        return self.raw_workorder_line_ids | self.finished_workorder_line_ids

    @api.onchange('qty_producing')
    def _onchange_qty_producing(self):
        """ Modify the qty currently producing will modify the existing
        workorder line in order to match the new quantity to consume for each
        component and their reserved quantity.
        """
        if self.qty_producing <= 0:
            raise UserError(_('You have to produce at least one %s.') % self.product_uom_id.name)
        line_values = self._update_workorder_lines()
        for values in line_values['to_create']:
            self.env[self._workorder_line_ids()._name].new(values)
        for line in line_values['to_delete']:
            if line in self.raw_workorder_line_ids:
                self.raw_workorder_line_ids -= line
            else:
                self.finished_workorder_line_ids -= line
        for line, vals in line_values['to_update'].items():
            line.update(vals)

    def _update_workorder_lines(self):
        """ Update workorder lines, according to the new qty currently
        produced. It returns a dict with line to create, update or delete.
        It do not directly write or unlink the line because this function is
        used in onchange and request that write on db (e.g. workorder creation).
        """
        line_values = {'to_create': [], 'to_delete': [], 'to_update': {}}
        # moves are actual records
        move_finished_ids = self.move_finished_ids._origin.filtered(lambda move: move.product_id != self.product_id and move.state not in ('done', 'cancel'))
        move_raw_ids = self.move_raw_ids._origin.filtered(lambda move: move.state not in ('done', 'cancel'))
        for move in move_raw_ids | move_finished_ids:
            move_workorder_lines = self._workorder_line_ids().filtered(lambda w: w.move_id == move)

            # Compute the new quantity for the current component
            rounding = move.product_uom.rounding
            new_qty = self._prepare_component_quantity(move, self.qty_producing)

            # In case the production uom is different than the workorder uom
            # it means the product is serial and production uom is not the reference
            new_qty = self.product_uom_id._compute_quantity(
                new_qty,
                self.production_id.product_uom_id,
                round=False
            )
            qty_todo = float_round(new_qty - sum(move_workorder_lines.mapped('qty_to_consume')), precision_rounding=rounding)

            # Remove or lower quantity on exisiting workorder lines
            if float_compare(qty_todo, 0.0, precision_rounding=rounding) < 0:
                qty_todo = abs(qty_todo)
                # Try to decrease or remove lines that are not reserved and
                # partialy reserved first. A different decrease strategy could
                # be define in _unreserve_order method.
                for workorder_line in move_workorder_lines.sorted(key=lambda wl: wl._unreserve_order()):
                    if float_compare(qty_todo, 0, precision_rounding=rounding) <= 0:
                        break
                    # If the quantity to consume on the line is lower than the
                    # quantity to remove, the line could be remove.
                    if float_compare(workorder_line.qty_to_consume, qty_todo, precision_rounding=rounding) <= 0:
                        qty_todo = float_round(qty_todo - workorder_line.qty_to_consume, precision_rounding=rounding)
                        if line_values['to_delete']:
                            line_values['to_delete'] |= workorder_line
                        else:
                            line_values['to_delete'] = workorder_line
                    # decrease the quantity on the line
                    else:
                        new_val = workorder_line.qty_to_consume - qty_todo
                        # avoid to write a negative reserved quantity
                        new_reserved = max(0, workorder_line.qty_reserved - qty_todo)
                        line_values['to_update'][workorder_line] = {
                            'qty_to_consume': new_val,
                            'qty_done': new_val,
                            'qty_reserved': new_reserved,
                        }
                        qty_todo = 0
            else:
                # Search among wo lines which one could be updated
                qty_reserved_wl = defaultdict(float)
                # Try to update the line with the greater reservation first in
                # order to promote bigger batch.
                for workorder_line in move_workorder_lines.sorted(key=lambda wl: wl.qty_reserved, reverse=True):
                    rounding = workorder_line.product_uom_id.rounding
                    if float_compare(qty_todo, 0, precision_rounding=rounding) <= 0:
                        break
                    move_lines = workorder_line._get_move_lines()
                    qty_reserved_wl[workorder_line.lot_id] += workorder_line.qty_reserved
                    # The reserved quantity according to exisiting move line
                    # already produced (with qty_done set) and other production
                    # lines with the same lot that are currently on production.
                    qty_reserved_remaining = sum(move_lines.mapped('product_uom_qty')) - sum(move_lines.mapped('qty_done')) - qty_reserved_wl[workorder_line.lot_id]
                    if float_compare(qty_reserved_remaining, 0, precision_rounding=rounding) > 0:
                        qty_to_add = min(qty_reserved_remaining, qty_todo)
                        line_values['to_update'][workorder_line] = {
                            'qty_done': workorder_line.qty_to_consume + qty_to_add,
                            'qty_to_consume': workorder_line.qty_to_consume + qty_to_add,
                            'qty_reserved': workorder_line.qty_reserved + qty_to_add,
                        }
                        qty_todo -= qty_to_add
                        qty_reserved_wl[workorder_line.lot_id] += qty_to_add

                    # If a line exists without reservation and without lot. It
                    # means that previous operations could not find any reserved
                    # quantity and created a line without lot prefilled. In this
                    # case, the system will not find an existing move line with
                    # available reservation anymore and will increase this line
                    # instead of creating a new line without lot and reserved
                    # quantities.
                    if not workorder_line.qty_reserved and not workorder_line.lot_id and workorder_line.product_tracking != 'serial':
                        line_values['to_update'][workorder_line] = {
                            'qty_done': workorder_line.qty_to_consume + qty_todo,
                            'qty_to_consume': workorder_line.qty_to_consume + qty_todo,
                        }
                        qty_todo = 0

                # if there are still qty_todo, create new wo lines
                if float_compare(qty_todo, 0.0, precision_rounding=rounding) > 0:
                    for values in self._generate_lines_values(move, qty_todo):
                        line_values['to_create'].append(values)
        return line_values

    @api.model
    def _generate_lines_values(self, move, qty_to_consume):
        """ Create workorder line. First generate line based on the reservation,
        in order to prefill reserved quantity, lot and serial number.
        If the quantity to consume is greater than the reservation quantity then
        create line with the correct quantity to consume but without lot or
        serial number.
        """
        lines = []
        is_tracked = move.product_id.tracking == 'serial'
        if move in self.move_raw_ids._origin:
            # Get the inverse_name (many2one on line) of raw_workorder_line_ids
            initial_line_values = {self.raw_workorder_line_ids._get_raw_workorder_inverse_name(): self.id}
        else:
            # Get the inverse_name (many2one on line) of finished_workorder_line_ids
            initial_line_values = {self.finished_workorder_line_ids._get_finished_workoder_inverse_name(): self.id}
        for move_line in move.move_line_ids:
            line = dict(initial_line_values)
            if float_compare(qty_to_consume, 0.0, precision_rounding=move.product_uom.rounding) <= 0:
                break
            # move line already 'used' in workorder (from its lot for instance)
            if move_line.lot_produced_ids or float_compare(move_line.product_uom_qty, move_line.qty_done, precision_rounding=move.product_uom.rounding) <= 0:
                continue
            # search wo line on which the lot is not fully consumed or other reserved lot
            linked_wo_line = self._workorder_line_ids().filtered(
                lambda line: line.move_id == move and
                line.lot_id == move_line.lot_id
            )
            if linked_wo_line:
                if float_compare(sum(linked_wo_line.mapped('qty_to_consume')), move_line.product_uom_qty - move_line.qty_done, precision_rounding=move.product_uom.rounding) < 0:
                    to_consume_in_line = min(qty_to_consume, move_line.product_uom_qty - move_line.qty_done - sum(linked_wo_line.mapped('qty_to_consume')))
                else:
                    continue
            else:
                to_consume_in_line = min(qty_to_consume, move_line.product_uom_qty - move_line.qty_done)
            line.update({
                'move_id': move.id,
                'product_id': move.product_id.id,
                'product_uom_id': is_tracked and move.product_id.uom_id.id or move.product_uom.id,
                'qty_to_consume': to_consume_in_line,
                'qty_reserved': to_consume_in_line,
                'lot_id': move_line.lot_id.id,
                'qty_done': to_consume_in_line,
            })
            lines.append(line)
            qty_to_consume -= to_consume_in_line
        # The move has not reserved the whole quantity so we create new wo lines
        if float_compare(qty_to_consume, 0.0, precision_rounding=move.product_uom.rounding) > 0:
            line = dict(initial_line_values)
            if move.product_id.tracking == 'serial':
                while float_compare(qty_to_consume, 0.0, precision_rounding=move.product_uom.rounding) > 0:
                    line.update({
                        'move_id': move.id,
                        'product_id': move.product_id.id,
                        'product_uom_id': move.product_id.uom_id.id,
                        'qty_to_consume': 1,
                        'qty_done': 1,
                    })
                    lines.append(line)
                    qty_to_consume -= 1
            else:
                line.update({
                    'move_id': move.id,
                    'product_id': move.product_id.id,
                    'product_uom_id': move.product_uom.id,
                    'qty_to_consume': qty_to_consume,
                    'qty_done': qty_to_consume,
                })
                lines.append(line)
        return lines

    def _update_finished_move(self):
        """ Update the finished move & move lines in order to set the finished
        product lot on it as well as the produced quantity. This method get the
        information either from the last workorder or from the Produce wizard."""
        move_line_vals = []
        for abstract_wo in self:
            production_move = abstract_wo.production_id.move_finished_ids.filtered(
                lambda move: move.product_id == abstract_wo.product_id and
                move.state not in ('done', 'cancel')
            )
            if not production_move:
                continue
            if production_move.product_id.tracking != 'none':
                if not abstract_wo.finished_lot_id:
                    raise UserError(_('You need to provide a lot for the finished product.'))
                move_line = production_move.move_line_ids.filtered(
                    lambda line: line.lot_id.id == abstract_wo.finished_lot_id.id
                )
                if move_line:
                    if abstract_wo.product_id.tracking == 'serial':
                        raise UserError(_('You cannot produce the same serial number twice.'))
                    move_line.product_uom_qty += abstract_wo.qty_producing
                    move_line.qty_done += abstract_wo.qty_producing
                else:
                    location_dest_id = production_move.location_dest_id._get_putaway_strategy(abstract_wo.product_id).id or production_move.location_dest_id.id
                    move_line_vals.append({
                        'move_id': production_move.id,
                        'product_id': production_move.product_id.id,
                        'lot_id': abstract_wo.finished_lot_id.id,
                        'product_uom_qty': abstract_wo.qty_producing,
                        'product_uom_id': abstract_wo.product_uom_id.id,
                        'qty_done': abstract_wo.qty_producing,
                        'location_id': production_move.location_id.id,
                        'location_dest_id': location_dest_id,
                    })
            else:
                production_move._set_quantity_done(
                    abstract_wo.product_uom_id._compute_quantity(abstract_wo.qty_producing, production_move.product_uom, rounding_method='HALF-UP')
                )
        self.env['stock.move.line'].create(move_line_vals)

    def _update_moves(self):
        """ Once the production is done. Modify the workorder lines into
        stock move line with the registered lot and quantity done.
        """
        # Before writting produce quantities, we ensure they respect the bom strictness
        vals_list = []
        line_to_unlink = self._workorder_line_ids().browse()
        self._strict_consumption_check()
        for abstract_wo in self:
            workorder_lines_to_process = abstract_wo._workorder_line_ids().filtered(lambda line: line.product_id != abstract_wo.product_id and line.qty_done > 0)
            for line in workorder_lines_to_process:
                line._update_move_lines()
                if float_compare(line.qty_done, 0, precision_rounding=line.product_uom_id.rounding) > 0:
                    vals_list += line._create_extra_move_lines()

            line_to_unlink |= abstract_wo._workorder_line_ids().filtered(lambda line: line.product_id != abstract_wo.product_id)
        line_to_unlink.unlink()
        self.env['stock.move.line'].create(vals_list)

    def _strict_consumption_check(self):
        for abstract_wo in self:
            if abstract_wo.consumption == 'strict':
                for move in abstract_wo.move_raw_ids:
                    lines = abstract_wo._workorder_line_ids().filtered(lambda l: l.move_id == move)
                    qty_done = 0.0
                    qty_to_consume = 0.0
                    for line in lines:
                        qty_done += line.product_uom_id._compute_quantity(line.qty_done, line.product_id.uom_id)
                        qty_to_consume += line.product_uom_id._compute_quantity(line.qty_to_consume, line.product_id.uom_id)
                    rounding = abstract_wo.product_uom_id.rounding
                    if float_compare(qty_done, qty_to_consume, precision_rounding=rounding) != 0:
                        raise UserError(_('You should consume the quantity of %s defined in the BoM. If you want to consume more or less components, change the consumption setting on the BoM.') % lines[0].product_id.name)


class MrpAbstractWorkorderLine(models.AbstractModel):
    _name = "mrp.abstract.workorder.line"
    _description = "Abstract model to implement product_produce_line as well as\
    workorder_line"
    _check_company_auto = True

    move_id = fields.Many2one('stock.move', check_company=True)
    product_id = fields.Many2one('product.product', 'Product', required=True, check_company=True)
    product_tracking = fields.Selection(related="product_id.tracking")
    lot_id = fields.Many2one(
        'stock.production.lot', 'Lot/Serial Number',
        check_company=True,
        domain="[('product_id', '=', product_id), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]")
    qty_to_consume = fields.Float('To Consume', digits='Product Unit of Measure')
    product_uom_id = fields.Many2one('uom.uom', string='Unit of Measure')
    qty_done = fields.Float('Consumed', digits='Product Unit of Measure')
    qty_reserved = fields.Float('Reserved', digits='Product Unit of Measure')
    company_id = fields.Many2one('res.company', compute='_compute_company_id')

    @api.onchange('product_id')
    def _onchange_product_id(self):
        if self.product_id and not self.move_id:
            self.product_uom_id = self.product_id.uom_id

    @api.onchange('qty_done')
    def _onchange_qty_done(self):
        """ When the user is encoding a produce line for a tracked product, we apply some logic to
        help him. This onchange will warn him if he set `qty_done` to a non-supported value.
        """
        res = {}
        if self.product_id.tracking == 'serial' and not float_is_zero(self.qty_done, self.product_uom_id.rounding):
            if float_compare(self.qty_done, 1.0, precision_rounding=self.product_uom_id.rounding) != 0:
                message = _('You can only process 1.0 %s of products with unique serial number.') % self.product_id.uom_id.name
                res['warning'] = {'title': _('Warning'), 'message': message}
        return res

    def _compute_company_id(self):
        for line in self:
            line.company_id = line._get_production().company_id

    def _update_move_lines(self):
        """ update a move line to save the workorder line data"""
        self.ensure_one()
        if self.lot_id:
            move_lines = self.move_id.move_line_ids.filtered(lambda ml: ml.lot_id == self.lot_id and not ml.lot_produced_ids)
        else:
            move_lines = self.move_id.move_line_ids.filtered(lambda ml: not ml.lot_id and not ml.lot_produced_ids)

        # Sanity check: if the product is a serial number and `lot` is already present in the other
        # consumed move lines, raise.
        if self.product_id.tracking != 'none' and not self.lot_id:
            raise UserError(_('Please enter a lot or serial number for %s !' % self.product_id.display_name))

        if self.lot_id and self.product_id.tracking == 'serial' and self.lot_id in self.move_id.move_line_ids.filtered(lambda ml: ml.qty_done).mapped('lot_id'):
            raise UserError(_('You cannot consume the same serial number twice. Please correct the serial numbers encoded.'))

        # Update reservation and quantity done
        for ml in move_lines:
            rounding = ml.product_uom_id.rounding
            if float_compare(self.qty_done, 0, precision_rounding=rounding) <= 0:
                break
            quantity_to_process = min(self.qty_done, ml.product_uom_qty - ml.qty_done)
            self.qty_done -= quantity_to_process

            new_quantity_done = (ml.qty_done + quantity_to_process)
            # if we produce less than the reserved quantity to produce the finished products
            # in different lots,
            # we create different component_move_lines to record which one was used
            # on which lot of finished product
            if float_compare(ml.product_uom_id._compute_quantity(new_quantity_done, ml.product_id.uom_id), ml.product_qty, precision_rounding=rounding) >= 0:
                ml.write({
                    'qty_done': new_quantity_done,
                    'lot_produced_ids': self._get_produced_lots(),
                })
            else:
                new_qty_reserved = ml.product_uom_qty - new_quantity_done
                default = {
                    'product_uom_qty': new_quantity_done,
                    'qty_done': new_quantity_done,
                    'lot_produced_ids': self._get_produced_lots(),
                }
                ml.copy(default=default)
                ml.with_context(bypass_reservation_update=True).write({
                    'product_uom_qty': new_qty_reserved,
                    'qty_done': 0
                })

    def _create_extra_move_lines(self):
        """Create new sml if quantity produced is bigger than the reserved one"""
        vals_list = []
        # apply putaway
        location_dest_id = self.move_id.location_dest_id._get_putaway_strategy(self.product_id) or self.move_id.location_dest_id
        quants = self.env['stock.quant']._gather(self.product_id, self.move_id.location_id, lot_id=self.lot_id, strict=False)
        # Search for a sub-locations where the product is available.
        # Loop on the quants to get the locations. If there is not enough
        # quantity into stock, we take the move location. Anyway, no
        # reservation is made, so it is still possible to change it afterwards.
        for quant in quants:
            quantity = quant.quantity - quant.reserved_quantity
            quantity = self.product_id.uom_id._compute_quantity(quantity, self.product_uom_id, rounding_method='HALF-UP')
            rounding = quant.product_uom_id.rounding
            if (float_compare(quant.quantity, 0, precision_rounding=rounding) <= 0 or
                    float_compare(quantity, 0, precision_rounding=self.product_uom_id.rounding) <= 0):
                continue
            vals = {
                'move_id': self.move_id.id,
                'product_id': self.product_id.id,
                'location_id': quant.location_id.id,
                'location_dest_id': location_dest_id.id,
                'product_uom_qty': 0,
                'product_uom_id': self.product_uom_id.id,
                'qty_done': min(quantity, self.qty_done),
                'lot_produced_ids': self._get_produced_lots(),
            }
            if self.lot_id:
                vals.update({'lot_id': self.lot_id.id})

            vals_list.append(vals)
            self.qty_done -= vals['qty_done']
            # If all the qty_done is distributed, we can close the loop
            if float_compare(self.qty_done, 0, precision_rounding=self.product_id.uom_id.rounding) <= 0:
                break

        if float_compare(self.qty_done, 0, precision_rounding=self.product_id.uom_id.rounding) > 0:
            vals = {
                'move_id': self.move_id.id,
                'product_id': self.product_id.id,
                'location_id': self.move_id.location_id.id,
                'location_dest_id': location_dest_id.id,
                'product_uom_qty': 0,
                'product_uom_id': self.product_uom_id.id,
                'qty_done': self.qty_done,
                'lot_produced_ids': self._get_produced_lots(),
            }
            if self.lot_id:
                vals.update({'lot_id': self.lot_id.id})

            vals_list.append(vals)

        return vals_list

    def _unreserve_order(self):
        """ Unreserve line with lower reserved quantity first """
        self.ensure_one()
        return (self.qty_reserved,)

    def _get_move_lines(self):
        return self.move_id.move_line_ids.filtered(lambda ml:
        ml.lot_id == self.lot_id and ml.product_id == self.product_id)

    def _get_produced_lots(self):
        return self.move_id in self._get_production().move_raw_ids and self._get_final_lots() and [(4, lot.id) for lot in self._get_final_lots()]

    @api.model
    def _get_raw_workorder_inverse_name(self):
        raise NotImplementedError('Method _get_raw_workorder_inverse_name() undefined on %s' % self)

    @api.model
    def _get_finished_workoder_inverse_name(self):
        raise NotImplementedError('Method _get_finished_workoder_inverse_name() undefined on %s' % self)

    # To be implemented in specific model
    def _get_final_lots(self):
        raise NotImplementedError('Method _get_final_lots() undefined on %s' % self)

    def _get_production(self):
        raise NotImplementedError('Method _get_production() undefined on %s' % self)

```

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_round, float_compare

from itertools import groupby
from collections import defaultdict


class MrpBom(models.Model):
    """ Defines bills of material for a product or a product template """
    _name = 'mrp.bom'
    _description = 'Bill of Material'
    _inherit = ['mail.thread']
    _rec_name = 'product_tmpl_id'
    _order = "sequence"
    _check_company_auto = True

    def _get_default_product_uom_id(self):
        return self.env['uom.uom'].search([], limit=1, order='id').id

    code = fields.Char('Reference')
    active = fields.Boolean(
        'Active', default=True,
        help="If the active field is set to False, it will allow you to hide the bills of material without removing it.")
    type = fields.Selection([
        ('normal', 'Manufacture this product'),
        ('phantom', 'Kit')], 'BoM Type',
        default='normal', required=True)
    product_tmpl_id = fields.Many2one(
        'product.template', 'Product',
        check_company=True,
        domain="[('type', 'in', ['product', 'consu']), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", required=True)
    product_id = fields.Many2one(
        'product.product', 'Product Variant',
        check_company=True,
        domain="['&', ('product_tmpl_id', '=', product_tmpl_id), ('type', 'in', ['product', 'consu']),  '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="If a product variant is defined the BOM is available only for this product.")
    bom_line_ids = fields.One2many('mrp.bom.line', 'bom_id', 'BoM Lines', copy=True)
    byproduct_ids = fields.One2many('mrp.bom.byproduct', 'bom_id', 'By-products', copy=True)
    product_qty = fields.Float(
        'Quantity', default=1.0,
        digits='Unit of Measure', required=True)
    product_uom_id = fields.Many2one(
        'uom.uom', 'Unit of Measure',
        default=_get_default_product_uom_id, required=True,
        help="Unit of Measure (Unit of Measure) is the unit of measurement for the inventory control", domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_tmpl_id.uom_id.category_id')
    sequence = fields.Integer('Sequence', help="Gives the sequence order when displaying a list of bills of material.")
    routing_id = fields.Many2one(
        'mrp.routing', 'Routing', check_company=True,
        tracking=True,
        help="The operations for producing this BoM.  When a routing is specified, the production orders will "
             " be executed through work orders, otherwise everything is processed in the production order itself. ")
    ready_to_produce = fields.Selection([
        ('all_available', ' When all components are available'),
        ('asap', 'When components for 1st operation are available')], string='Manufacturing Readiness',
        default='asap', help="Defines when a Manufacturing Order is considered as ready to be started", required=True)
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type', domain="[('code', '=', 'mrp_operation'), ('company_id', '=', company_id)]",
        check_company=True,
        help=u"When a procurement has a ‘produce’ route with a operation type set, it will try to create "
             "a Manufacturing Order for that product using a BoM of the same operation type. That allows "
             "to define stock rules which trigger different manufacturing orders with different BoMs.")
    company_id = fields.Many2one(
        'res.company', 'Company', index=True,
        default=lambda self: self.env.company)
    consumption = fields.Selection([
        ('strict', 'Strict'),
        ('flexible', 'Flexible')],
        help="Defines if you can consume more or less components than the quantity defined on the BoM.",
        default='strict',
        string='Consumption'
    )

    @api.onchange('product_id')
    def onchange_product_id(self):
        if self.product_id:
            for line in self.bom_line_ids:
                line.bom_product_template_attribute_value_ids = False

    @api.constrains('product_id', 'product_tmpl_id', 'bom_line_ids')
    def _check_bom_lines(self):
        for bom in self:
            for bom_line in bom.bom_line_ids:
                if bom.product_id:
                    same_product = bom.product_id == bom_line.product_id
                else:
                    same_product = bom.product_tmpl_id == bom_line.product_id.product_tmpl_id
                if same_product:
                    raise ValidationError(_("BoM line product %s should not be the same as BoM product.") % bom.display_name)
                if bom.product_id and bom_line.bom_product_template_attribute_value_ids:
                    raise ValidationError(_("BoM cannot concern product %s and have a line with attributes (%s) at the same time.")
                        % (bom.product_id.display_name, ", ".join([ptav.display_name for ptav in bom_line.bom_product_template_attribute_value_ids])))
                for ptav in bom_line.bom_product_template_attribute_value_ids:
                    if ptav.product_tmpl_id != bom.product_tmpl_id:
                        raise ValidationError(
                            _("The attribute value %s set on product %s does not match the BoM product %s.") %
                            (ptav.display_name, ptav.product_tmpl_id.display_name, bom_line.parent_product_tmpl_id.display_name)
                        )

    @api.onchange('bom_line_ids', 'product_qty')
    def onchange_bom_structure(self):
        if self.type == 'phantom' and self._origin and self.env['stock.move'].search([('bom_line_id', 'in', self._origin.bom_line_ids.ids)], limit=1):
            return {
                'warning': {
                    'title': _('Warning'),
                    'message': _(
                        'The product has already been used at least once, editing its structure may lead to undesirable behaviours. '
                        'You should rather archive the product and create a new one with a new bill of materials.'),
                }
            }

    @api.onchange('product_uom_id')
    def onchange_product_uom_id(self):
        res = {}
        if not self.product_uom_id or not self.product_tmpl_id:
            return
        if self.product_uom_id.category_id.id != self.product_tmpl_id.uom_id.category_id.id:
            self.product_uom_id = self.product_tmpl_id.uom_id.id
            res['warning'] = {'title': _('Warning'), 'message': _('The Product Unit of Measure you chose has a different category than in the product form.')}
        return res

    @api.onchange('product_tmpl_id')
    def onchange_product_tmpl_id(self):
        if self.product_tmpl_id:
            self.product_uom_id = self.product_tmpl_id.uom_id.id
            if self.product_id.product_tmpl_id != self.product_tmpl_id:
                self.product_id = False
            for line in self.bom_line_ids:
                line.bom_product_template_attribute_value_ids = False

    @api.onchange('routing_id')
    def onchange_routing_id(self):
        for line in self.bom_line_ids:
            line.operation_id = False

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        for bom in res:
            if float_compare(bom.product_qty, 0, precision_rounding=bom.product_uom_id.rounding) <= 0:
                raise UserError(_('The quantity to produce must be positive!'))
        return res

    def write(self, values):
        res = super().write(values)
        for bom in self:
            if float_compare(bom.product_qty, 0, precision_rounding=bom.product_uom_id.rounding) <= 0:
                raise UserError(_('The quantity to produce must be positive!'))
        return res

    @api.model
    def name_create(self, name):
        # prevent to use string as product_tmpl_id
        if isinstance(name, str):
            raise UserError(_("You cannot create a new Bill of Material from here."))
        return super(MrpBom, self).name_create(name)

    def name_get(self):
        return [(bom.id, '%s%s' % (bom.code and '%s: ' % bom.code or '', bom.product_tmpl_id.display_name)) for bom in self]

    @api.constrains('product_tmpl_id', 'product_id', 'type')
    def check_kit_has_not_orderpoint(self):
        product_ids = [pid for bom in self.filtered(lambda bom: bom.type == "phantom")
                           for pid in (bom.product_id.ids or bom.product_tmpl_id.product_variant_ids.ids)]
        if self.env['stock.warehouse.orderpoint'].search([('product_id', 'in', product_ids)], count=True):
            raise ValidationError(_("You can not create a kit-type bill of materials for products that have at least one reordering rule."))

    def unlink(self):
        if self.env['mrp.production'].search([('bom_id', 'in', self.ids), ('state', 'not in', ['done', 'cancel'])], limit=1):
            raise UserError(_('You can not delete a Bill of Material with running manufacturing orders.\nPlease close or cancel it first.'))
        return super(MrpBom, self).unlink()

    @api.model
    def _bom_find_domain(self, product_tmpl=None, product=None, picking_type=None, company_id=False, bom_type=False):
        if product:
            if not product_tmpl:
                product_tmpl = product.product_tmpl_id
            domain = ['|', ('product_id', '=', product.id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product_tmpl.id)]
        elif product_tmpl:
            domain = [('product_tmpl_id', '=', product_tmpl.id)]
        else:
            # neither product nor template, makes no sense to search
            raise UserError(_('You should provide either a product or a product template to search a BoM'))
        if picking_type:
            domain += ['|', ('picking_type_id', '=', picking_type.id), ('picking_type_id', '=', False)]
        if company_id or self.env.context.get('company_id'):
            domain = domain + ['|', ('company_id', '=', False), ('company_id', '=', company_id or self.env.context.get('company_id'))]
        if bom_type:
            domain += [('type', '=', bom_type)]
        # order to prioritize bom with product_id over the one without
        return domain

    @api.model
    def _bom_find(self, product_tmpl=None, product=None, picking_type=None, company_id=False, bom_type=False):
        """ Finds BoM for particular product, picking and company """
        if product and product.type == 'service' or product_tmpl and product_tmpl.type == 'service':
            return False
        domain = self._bom_find_domain(product_tmpl=product_tmpl, product=product, picking_type=picking_type, company_id=company_id, bom_type=bom_type)
        if domain is False:
            return domain
        return self.search(domain, order='sequence, product_id', limit=1)

    @api.model
    def _get_product2bom(self, products, bom_type=False, picking_type=False, company_id=False):
        """Optimized variant of _bom_find to work with recordset"""

        bom_by_product = defaultdict(lambda: self.env['mrp.bom'])
        products = products.filtered(lambda p: p.type != 'service')
        if not products:
            return bom_by_product
        product_templates = products.mapped('product_tmpl_id')
        domain = ['|', ('product_id', 'in', products.ids), '&', ('product_id', '=', False), ('product_tmpl_id', 'in', product_templates.ids), ('active', '=', True)]
        if picking_type:
            domain += ['|', ('picking_type_id', '=', picking_type.id), ('picking_type_id', '=', False)]
        if company_id or self.env.context.get('company_id'):
            domain = domain + ['|', ('company_id', '=', False), ('company_id', '=', company_id or self.env.context.get('company_id'))]
        if bom_type:
            domain += [('type', '=', bom_type)]

        if len(products) == 1:
            bom = self.search(domain, order='sequence, product_id, id', limit=1)
            if bom:
                bom_by_product[products] = bom
            return bom_by_product

        boms = self.search(domain, order='sequence, product_id, id')

        products_ids = set(products.ids)
        for bom in boms:
            products_implies = bom.product_id or bom.product_tmpl_id.product_variant_ids
            for product in products_implies:
                if product.id in products_ids and product not in bom_by_product:
                    bom_by_product[product] = bom
        return bom_by_product

    def explode(self, product, quantity, picking_type=False):
        """
            Explodes the BoM and creates two lists with all the information you need: bom_done and line_done
            Quantity describes the number of times you need the BoM: so the quantity divided by the number created by the BoM
            and converted into its UoM
        """
        from collections import defaultdict

        graph = defaultdict(list)
        V = set()

        def check_cycle(v, visited, recStack, graph):
            visited[v] = True
            recStack[v] = True
            for neighbour in graph[v]:
                if visited[neighbour] == False:
                    if check_cycle(neighbour, visited, recStack, graph) == True:
                        return True
                elif recStack[neighbour] == True:
                    return True
            recStack[v] = False
            return False

        product_ids = set()
        product_boms = {}
        def update_product_boms():
            products = self.env['product.product'].browse(product_ids)
            product_boms.update(self._get_product2bom(products, bom_type='phantom',
                picking_type=picking_type or self.picking_type_id, company_id=self.company_id.id))
            # Set missing keys to default value
            for product in products:
                product_boms.setdefault(product, self.env['mrp.bom'])

        boms_done = [(self, {'qty': quantity, 'product': product, 'original_qty': quantity, 'parent_line': False})]
        lines_done = []
        V |= set([product.product_tmpl_id.id])

        bom_lines = []
        for bom_line in self.bom_line_ids:
            product_id = bom_line.product_id
            V |= set([product_id.product_tmpl_id.id])
            graph[product.product_tmpl_id.id].append(product_id.product_tmpl_id.id)
            bom_lines.append((bom_line, product, quantity, False))
            product_ids.add(product_id.id)
        update_product_boms()
        product_ids.clear()
        while bom_lines:
            current_line, current_product, current_qty, parent_line = bom_lines[0]
            bom_lines = bom_lines[1:]

            if current_line._skip_bom_line(current_product):
                continue

            line_quantity = current_qty * current_line.product_qty
            if not current_line.product_id in product_boms:
                update_product_boms()
                product_ids.clear()
            bom = product_boms.get(current_line.product_id)
            if bom:
                converted_line_quantity = current_line.product_uom_id._compute_quantity(line_quantity / bom.product_qty, bom.product_uom_id)
                bom_lines += [(line, current_line.product_id, converted_line_quantity, current_line) for line in bom.bom_line_ids]
                for bom_line in bom.bom_line_ids:
                    graph[current_line.product_id.product_tmpl_id.id].append(bom_line.product_id.product_tmpl_id.id)
                    if bom_line.product_id.product_tmpl_id.id in V and check_cycle(bom_line.product_id.product_tmpl_id.id, {key: False for  key in V}, {key: False for  key in V}, graph):
                        raise UserError(_('Recursion error!  A product with a Bill of Material should not have itself in its BoM or child BoMs!'))
                    V |= set([bom_line.product_id.product_tmpl_id.id])
                    if not bom_line.product_id in product_boms:
                        product_ids.add(bom_line.product_id.id)
                boms_done.append((bom, {'qty': converted_line_quantity, 'product': current_product, 'original_qty': quantity, 'parent_line': current_line}))
            else:
                # We round up here because the user expects that if he has to consume a little more, the whole UOM unit
                # should be consumed.
                rounding = current_line.product_uom_id.rounding
                line_quantity = float_round(line_quantity, precision_rounding=rounding, rounding_method='UP')
                lines_done.append((current_line, {'qty': line_quantity, 'product': current_product, 'original_qty': quantity, 'parent_line': parent_line}))

        return boms_done, lines_done

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Bills of Materials'),
            'template': '/mrp/static/xls/mrp_bom.xls'
        }]


class MrpBomLine(models.Model):
    _name = 'mrp.bom.line'
    _order = "sequence, id"
    _rec_name = "product_id"
    _description = 'Bill of Material Line'
    _check_company_auto = True

    def _get_default_product_uom_id(self):
        return self.env['uom.uom'].search([], limit=1, order='id').id

    product_id = fields.Many2one('product.product', 'Component', required=True, check_company=True)
    product_tmpl_id = fields.Many2one('product.template', 'Product Template', related='product_id.product_tmpl_id')
    company_id = fields.Many2one(
        related='bom_id.company_id', store=True, index=True, readonly=True)
    product_qty = fields.Float(
        'Quantity', default=1.0,
        digits='Product Unit of Measure', required=True)
    product_uom_id = fields.Many2one(
        'uom.uom', 'Product Unit of Measure',
        default=_get_default_product_uom_id,
        required=True,
        help="Unit of Measure (Unit of Measure) is the unit of measurement for the inventory control", domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    sequence = fields.Integer(
        'Sequence', default=1,
        help="Gives the sequence order when displaying.")
    routing_id = fields.Many2one(
        'mrp.routing', 'Routing',
        related='bom_id.routing_id', store=True, readonly=False,
        help="The list of operations to produce the finished product. The routing is mainly used to "
             "compute work center costs during operations and to plan future loads on work centers "
             "based on production planning.")
    bom_id = fields.Many2one(
        'mrp.bom', 'Parent BoM',
        index=True, ondelete='cascade', required=True)
    parent_product_tmpl_id = fields.Many2one('product.template', 'Parent Product Template', related='bom_id.product_tmpl_id')
    possible_bom_product_template_attribute_value_ids = fields.Many2many('product.template.attribute.value', compute='_compute_possible_bom_product_template_attribute_value_ids')
    bom_product_template_attribute_value_ids = fields.Many2many(
        'product.template.attribute.value', string="Apply on Variants", ondelete='restrict',
        domain="[('id', 'in', possible_bom_product_template_attribute_value_ids)]",
        help="BOM Product Variants needed to apply this line.")
    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Consumed in Operation', check_company=True,
        domain="[('routing_id', '=', routing_id), '|', ('company_id', '=', company_id), ('company_id', '=', False)]",
        help="The operation where the components are consumed, or the finished products created.")
    child_bom_id = fields.Many2one(
        'mrp.bom', 'Sub BoM', compute='_compute_child_bom_id')
    child_line_ids = fields.One2many(
        'mrp.bom.line', string="BOM lines of the referred bom",
        compute='_compute_child_line_ids')
    attachments_count = fields.Integer('Attachments Count', compute='_compute_attachments_count')

    _sql_constraints = [
        ('bom_qty_zero', 'CHECK (product_qty>=0)', 'All product quantities must be greater or equal to 0.\n'
            'Lines with 0 quantities can be used as optional lines. \n'
            'You should install the mrp_byproduct module if you want to manage extra products on BoMs !'),
    ]

    @api.depends(
        'parent_product_tmpl_id.attribute_line_ids.value_ids',
        'parent_product_tmpl_id.attribute_line_ids.attribute_id.create_variant',
        'parent_product_tmpl_id.attribute_line_ids.product_template_value_ids.ptav_active',
    )
    def _compute_possible_bom_product_template_attribute_value_ids(self):
        for line in self:
            line.possible_bom_product_template_attribute_value_ids = line.parent_product_tmpl_id.valid_product_template_attribute_line_ids._without_no_variant_attributes().product_template_value_ids._only_active()

    @api.depends('product_id', 'bom_id')
    def _compute_child_bom_id(self):
        for line in self:
            if not line.product_id:
                line.child_bom_id = False
            else:
                line.child_bom_id = self.env['mrp.bom']._bom_find(
                    product_tmpl=line.product_id.product_tmpl_id,
                    product=line.product_id)

    @api.depends('product_id')
    def _compute_attachments_count(self):
        for line in self:
            nbr_attach = self.env['mrp.document'].search_count([
                '|',
                '&', ('res_model', '=', 'product.product'), ('res_id', '=', line.product_id.id),
                '&', ('res_model', '=', 'product.template'), ('res_id', '=', line.product_id.product_tmpl_id.id)])
            line.attachments_count = nbr_attach

    @api.depends('child_bom_id')
    def _compute_child_line_ids(self):
        """ If the BOM line refers to a BOM, return the ids of the child BOM lines """
        for line in self:
            line.child_line_ids = line.child_bom_id.bom_line_ids.ids or False

    @api.onchange('product_uom_id')
    def onchange_product_uom_id(self):
        res = {}
        if not self.product_uom_id or not self.product_id:
            return res
        if self.product_uom_id.category_id != self.product_id.uom_id.category_id:
            self.product_uom_id = self.product_id.uom_id.id
            res['warning'] = {'title': _('Warning'), 'message': _('The Product Unit of Measure you chose has a different category than in the product form.')}
        return res

    @api.onchange('product_id')
    def onchange_product_id(self):
        if self.product_id:
            self.product_uom_id = self.product_id.uom_id.id

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if 'product_id' in values and 'product_uom_id' not in values:
                values['product_uom_id'] = self.env['product.product'].browse(values['product_id']).uom_id.id
        return super(MrpBomLine, self).create(vals_list)

    def _skip_bom_line(self, product):
        """ Control if a BoM line should be produced, can be inherited to add
        custom control. It currently checks that all variant values are in the
        product.

        If multiple values are encoded for the same attribute line, only one of
        them has to be found on the variant.
        """
        self.ensure_one()
        if product._name == 'product.template':
            return False
        if self.bom_product_template_attribute_value_ids:
            for ptal, iter_ptav in groupby(self.bom_product_template_attribute_value_ids.sorted('attribute_line_id'), lambda ptav: ptav.attribute_line_id):
                if not any([ptav in product.product_template_attribute_value_ids for ptav in iter_ptav]):
                    return True
        return False

    def action_see_attachments(self):
        domain = [
            '|',
            '&', ('res_model', '=', 'product.product'), ('res_id', '=', self.product_id.id),
            '&', ('res_model', '=', 'product.template'), ('res_id', '=', self.product_id.product_tmpl_id.id)]
        attachment_view = self.env.ref('mrp.view_document_file_kanban_mrp')
        return {
            'name': _('Attachments'),
            'domain': domain,
            'res_model': 'mrp.document',
            'type': 'ir.actions.act_window',
            'view_id': attachment_view.id,
            'views': [(attachment_view.id, 'kanban'), (False, 'form')],
            'view_mode': 'kanban,tree,form',
            'help': _('''<p class="o_view_nocontent_smiling_face">
                        Upload files to your product
                    </p><p>
                        Use this feature to store any files, like drawings or specifications.
                    </p>'''),
            'limit': 80,
            'context': "{'default_res_model': '%s','default_res_id': %d, 'default_company_id': %s}" % ('product.product', self.product_id.id, self.company_id.id)
        }


class MrpByProduct(models.Model):
    _name = 'mrp.bom.byproduct'
    _description = 'Byproduct'
    _rec_name = "product_id"
    _check_company_auto = True

    product_id = fields.Many2one('product.product', 'By-product', required=True, check_company=True)
    company_id = fields.Many2one(related='bom_id.company_id', store=True, index=True, readonly=True)
    product_qty = fields.Float(
        'Quantity',
        default=1.0, digits='Product Unit of Measure', required=True)
    product_uom_id = fields.Many2one('uom.uom', 'Unit of Measure', required=True)
    bom_id = fields.Many2one('mrp.bom', 'BoM', ondelete='cascade')
    routing_id = fields.Many2one(
        'mrp.routing', 'Routing', store=True, related='bom_id.routing_id')
    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Produced in Operation', check_company=True,
        domain="[('routing_id', '=', routing_id), '|', ('company_id', '=', company_id), ('company_id', '=', False)]")

    @api.onchange('product_id')
    def onchange_product_id(self):
        """ Changes UoM if product_id changes. """
        if self.product_id:
            self.product_uom_id = self.product_id.uom_id.id

    @api.onchange('product_uom_id')
    def onchange_uom(self):
        res = {}
        if self.product_uom_id and self.product_id and self.product_uom_id.category_id != self.product_id.uom_id.category_id:
            res['warning'] = {
                'title': _('Warning'),
                'message': _('The unit of measure you choose is in a different category than the product unit of measure.')
            }
            self.product_uom_id = self.product_id.uom_id.id
        return res


```

## File: models\mrp_document.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MrpDocument(models.Model):
    """ Extension of ir.attachment only used in MRP to handle archivage
    and basic versioning.
    """
    _name = 'mrp.document'
    _description = "Production Document"
    _inherits = {
        'ir.attachment': 'ir_attachment_id',
    }
    _order = "priority desc, id desc"

    def copy(self, default=None):
        ir_default = default
        if ir_default:
            ir_fields = list(self.env['ir.attachment']._fields)
            ir_default = {field : default[field] for field in default.keys() if field in ir_fields}
        new_attach = self.ir_attachment_id.with_context(no_document=True).copy(ir_default)
        return super().copy(dict(default, ir_attachment_id=new_attach.id))

    ir_attachment_id = fields.Many2one('ir.attachment', string='Related attachment', required=True, ondelete='cascade')
    active = fields.Boolean('Active', default=True)
    priority = fields.Selection([
        ('0', 'Normal'),
        ('1', 'Low'),
        ('2', 'High'),
        ('3', 'Very High')], string="Priority", help='Gives the sequence order when displaying a list of MRP documents.')

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
from collections import defaultdict
from itertools import groupby

from odoo import api, fields, models, _
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools import date_utils, float_compare, float_round, float_is_zero


class MrpProduction(models.Model):
    """ Manufacturing Orders """
    _name = 'mrp.production'
    _description = 'Production Order'
    _date_name = 'date_planned_start'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'date_planned_start asc,id'

    @api.model
    def _get_default_picking_type(self):
        company_id = self.env.context.get('default_company_id', self.env.company.id)
        return self.env['stock.picking.type'].search([
            ('code', '=', 'mrp_operation'),
            ('warehouse_id.company_id', '=', company_id),
        ], limit=1).id

    @api.model
    def _get_default_location_src_id(self):
        location = False
        company_id = self.env.context.get('default_company_id', self.env.company.id)
        if self.env.context.get('default_picking_type_id'):
            location = self.env['stock.picking.type'].browse(self.env.context['default_picking_type_id']).default_location_src_id
        if not location:
            location = self.env['stock.warehouse'].search([('company_id', '=', company_id)], limit=1).lot_stock_id
        return location and location.id or False

    @api.model
    def _get_default_location_dest_id(self):
        location = False
        company_id = self.env.context.get('default_company_id', self.env.company.id)
        if self._context.get('default_picking_type_id'):
            location = self.env['stock.picking.type'].browse(self.env.context['default_picking_type_id']).default_location_dest_id
        if not location:
            location = self.env['stock.warehouse'].search([('company_id', '=', company_id)], limit=1).lot_stock_id
        return location and location.id or False

    @api.model
    def _get_default_date_planned_finished(self):
        if self.env.context.get('default_date_planned_start'):
            return fields.Datetime.to_datetime(self.env.context.get('default_date_planned_start')) + datetime.timedelta(hours=1)
        return datetime.datetime.now() + datetime.timedelta(hours=1)

    name = fields.Char(
        'Reference', copy=False, readonly=True, default=lambda x: _('New'))
    origin = fields.Char(
        'Source', copy=False,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help="Reference of the document that generated this production order request.")

    product_id = fields.Many2one(
        'product.product', 'Product',
        domain="[('bom_ids', '!=', False), ('bom_ids.active', '=', True), ('bom_ids.type', '=', 'normal'), ('type', 'in', ['product', 'consu']), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        readonly=True, required=True, check_company=True,
        states={'draft': [('readonly', False)]})
    product_tmpl_id = fields.Many2one('product.template', 'Product Template', related='product_id.product_tmpl_id')
    product_qty = fields.Float(
        'Quantity To Produce',
        default=1.0, digits='Product Unit of Measure',
        readonly=True, required=True, tracking=True,
        states={'draft': [('readonly', False)]})
    product_uom_id = fields.Many2one(
        'uom.uom', 'Product Unit of Measure',
        readonly=True, required=True,
        states={'draft': [('readonly', False)]})
    product_uom_qty = fields.Float(string='Total Quantity', compute='_compute_product_uom_qty', store=True)
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type',
        domain="[('code', '=', 'mrp_operation'), ('company_id', '=', company_id)]",
        default=_get_default_picking_type, required=True, check_company=True,
        readonly=True, states={'draft': [('readonly', False)]})
    location_src_id = fields.Many2one(
        'stock.location', 'Components Location',
        default=_get_default_location_src_id,
        readonly=True, required=True,
        domain="[('usage','=','internal'), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        states={'draft': [('readonly', False)]}, check_company=True,
        help="Location where the system will look for components.")
    location_dest_id = fields.Many2one(
        'stock.location', 'Finished Products Location',
        default=_get_default_location_dest_id,
        readonly=True, required=True,
        domain="[('usage','=','internal'), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        states={'draft': [('readonly', False)]}, check_company=True,
        help="Location where the system will stock the finished products.")
    date_planned_start = fields.Datetime(
        'Planned Date', copy=False, default=fields.Datetime.now,
        help="Date at which you plan to start the production.",
        index=True, required=True, store=True)
    date_planned_finished = fields.Datetime(
        'Planned End Date',
        default=_get_default_date_planned_finished,
        help="Date at which you plan to finish the production.",
        copy=False, store=True)
    date_deadline = fields.Datetime(
        'Deadline', copy=False, index=True,
        help="Informative date allowing to define when the manufacturing order should be processed at the latest to fulfill delivery on time.")
    date_start = fields.Datetime('Start Date', copy=False, index=True, readonly=True)
    date_finished = fields.Datetime('End Date', copy=False, index=True, readonly=True)
    date_start_wo = fields.Datetime(
        'Plan From', copy=False, readonly=True,
        help="Work orders will be planned based on the availability of the work centers\
              starting from this date. If empty, the work orders will be planned as soon as possible.",
    )
    bom_id = fields.Many2one(
        'mrp.bom', 'Bill of Material',
        readonly=True, states={'draft': [('readonly', False)]},
        domain="""[
        '&',
            '|',
                ('company_id', '=', False),
                ('company_id', '=', company_id),
            '&',
                '|',
                    ('product_id','=',product_id),
                    '&',
                        ('product_tmpl_id.product_variant_ids','=',product_id),
                        ('product_id','=',False),
        ('type', '=', 'normal')]""",
        check_company=True,
        help="Bill of Materials allow you to define the list of required components to make a finished product.")
    routing_id = fields.Many2one(
        'mrp.routing', 'Routing',
        readonly=True, compute='_compute_routing', store=True,
        help="The list of operations (list of work centers) to produce the finished product. The routing "
             "is mainly used to compute work center costs during operations and to plan future loads on "
             "work centers based on production planning.")

    state = fields.Selection([
        ('draft', 'Draft'),
        ('confirmed', 'Confirmed'),
        ('planned', 'Planned'),
        ('progress', 'In Progress'),
        ('to_close', 'To Close'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')], string='State',
        compute='_compute_state', copy=False, index=True, readonly=True,
        store=True, tracking=True,
        help=" * Draft: The MO is not confirmed yet.\n"
             " * Confirmed: The MO is confirmed, the stock rules and the reordering of the components are trigerred.\n"
             " * Planned: The WO are planned.\n"
             " * In Progress: The production has started (on the MO or on the WO).\n"
             " * To Close: The production is done, the MO has to be closed.\n"
             " * Done: The MO is closed, the stock moves are posted. \n"
             " * Cancelled: The MO has been cancelled, can't be confirmed anymore.")
    reservation_state = fields.Selection([
        ('confirmed', 'Waiting'),
        ('assigned', 'Ready'),
        ('waiting', 'Waiting Another Operation')],
        string='Material Availability',
        compute='_compute_state', copy=False, index=True, readonly=True,
        store=True, tracking=True,
        help=" * Ready: The material is available to start the production.\n\
            * Waiting: The material is not available to start the production.\n\
            The material availability is impacted by the manufacturing readiness\
            defined on the BoM.")

    move_raw_ids = fields.One2many(
        'stock.move', 'raw_material_production_id', 'Components',
        copy=True, states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        domain=[('scrapped', '=', False)])
    move_finished_ids = fields.One2many(
        'stock.move', 'production_id', 'Finished Products',
        copy=False, states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        domain=[('scrapped', '=', False)])
    finished_move_line_ids = fields.One2many(
        'stock.move.line', compute='_compute_lines', inverse='_inverse_lines', string="Finished Product"
        )
    workorder_ids = fields.One2many(
        'mrp.workorder', 'production_id', 'Work Orders',
        copy=False, readonly=True)
    workorder_count = fields.Integer('# Work Orders', compute='_compute_workorder_count')
    workorder_done_count = fields.Integer('# Done Work Orders', compute='_compute_workorder_done_count')
    move_dest_ids = fields.One2many('stock.move', 'created_production_id',
        string="Stock Movements of Produced Goods")

    unreserve_visible = fields.Boolean(
        'Allowed to Unreserve Inventory', compute='_compute_unreserve_visible',
        help='Technical field to check when we can unreserve')
    post_visible = fields.Boolean(
        'Allowed to Post Inventory', compute='_compute_post_visible',
        help='Technical field to check when we can post')
    user_id = fields.Many2one(
        'res.users', 'Responsible', default=lambda self: self.env.user,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        domain=lambda self: [('groups_id', 'in', self.env.ref('mrp.group_mrp_user').id)])
    company_id = fields.Many2one(
        'res.company', 'Company', default=lambda self: self.env.company,
        index=True, required=True)

    qty_produced = fields.Float(compute="_get_produced_qty", string="Quantity Produced")
    procurement_group_id = fields.Many2one(
        'procurement.group', 'Procurement Group',
        copy=False)
    orderpoint_id = fields.Many2one('stock.warehouse.orderpoint', 'Orderpoint')
    propagate_cancel = fields.Boolean(
        'Propagate cancel and split',
        help='If checked, when the previous move of the move (which was generated by a next procurement) is cancelled or split, the move generated by this move will too')
    propagate_date = fields.Boolean(string="Propagate Rescheduling",
        help='The rescheduling is propagated to the next move.')
    propagate_date_minimum_delta = fields.Integer(string='Reschedule if Higher Than',
        help='The change must be higher than this value to be propagated')
    scrap_ids = fields.One2many('stock.scrap', 'production_id', 'Scraps')
    scrap_count = fields.Integer(compute='_compute_scrap_move_count', string='Scrap Move')
    priority = fields.Selection([('0', 'Not urgent'), ('1', 'Normal'), ('2', 'Urgent'), ('3', 'Very Urgent')], 'Priority',
                                readonly=True, states={'draft': [('readonly', False)]}, default='1')
    is_locked = fields.Boolean('Is Locked', default=True, copy=False)
    show_final_lots = fields.Boolean('Show Final Lots', compute='_compute_show_lots')
    production_location_id = fields.Many2one('stock.location', "Production Location", related='product_id.property_stock_production', readonly=False, related_sudo=False)
    picking_ids = fields.Many2many('stock.picking', compute='_compute_picking_ids', string='Picking associated to this manufacturing order')
    delivery_count = fields.Integer(string='Delivery Orders', compute='_compute_picking_ids')
    confirm_cancel = fields.Boolean(compute='_compute_confirm_cancel')

    @api.depends('move_raw_ids.state', 'move_finished_ids.state')
    def _compute_confirm_cancel(self):
        """ If the manufacturing order contains some done move (via an intermediate
        post inventory), the user has to confirm the cancellation.
        """
        domain = [
            ('state', '=', 'done'),
            '|',
                ('production_id', 'in', self.ids),
                ('raw_material_production_id', 'in', self.ids)
        ]
        res = self.env['stock.move'].read_group(domain, ['state', 'production_id', 'raw_material_production_id'], ['production_id', 'raw_material_production_id'], lazy=False)
        productions_with_done_move = {}
        for rec in res:
            production_record = rec['production_id'] or rec['raw_material_production_id']
            if production_record:
                productions_with_done_move[production_record[0]] = True
        for production in self:
            production.confirm_cancel = productions_with_done_move.get(production.id, False)

    @api.depends('procurement_group_id')
    def _compute_picking_ids(self):
        for order in self:
            order.picking_ids = self.env['stock.picking'].search([
                ('group_id', '=', order.procurement_group_id.id), ('group_id', '!=', False),
            ])
            order.delivery_count = len(order.picking_ids)

    def action_view_mo_delivery(self):
        """ This function returns an action that display picking related to
        manufacturing order orders. It can either be a in a list or in a form
        view, if there is only one picking to show.
        """
        self.ensure_one()
        action = self.env.ref('stock.action_picking_tree_all').read()[0]
        pickings = self.mapped('picking_ids')
        if len(pickings) > 1:
            action['domain'] = [('id', 'in', pickings.ids)]
        elif pickings:
            form_view = [(self.env.ref('stock.view_picking_form').id, 'form')]
            if 'views' in action:
                action['views'] = form_view + [(state,view) for state,view in action['views'] if view != 'form']
            else:
                action['views'] = form_view
            action['res_id'] = pickings.id
        action['context'] = dict(self._context, default_origin=self.name, create=False)
        return action

    @api.depends('product_uom_id', 'product_qty', 'product_id.uom_id')
    def _compute_product_uom_qty(self):
        for production in self:
            if production.product_id.uom_id != production.product_uom_id:
                production.product_uom_qty = production.product_uom_id._compute_quantity(production.product_qty, production.product_id.uom_id)
            else:
                production.product_uom_qty = production.product_qty

    @api.depends('product_id.tracking')
    def _compute_show_lots(self):
        for production in self:
            production.show_final_lots = production.product_id.tracking != 'none'

    def _inverse_lines(self):
        """ Little hack to make sure that when you change something on these objects, it gets saved"""
        pass

    @api.depends('move_finished_ids.move_line_ids')
    def _compute_lines(self):
        for production in self:
            production.finished_move_line_ids = production.move_finished_ids.mapped('move_line_ids')

    @api.depends('bom_id.routing_id', 'bom_id.routing_id.operation_ids')
    def _compute_routing(self):
        for production in self:
            if production.bom_id.routing_id.operation_ids:
                production.routing_id = production.bom_id.routing_id.id
            else:
                production.routing_id = False

    @api.depends('workorder_ids')
    def _compute_workorder_count(self):
        data = self.env['mrp.workorder'].read_group([('production_id', 'in', self.ids)], ['production_id'], ['production_id'])
        count_data = dict((item['production_id'][0], item['production_id_count']) for item in data)
        for production in self:
            production.workorder_count = count_data.get(production.id, 0)

    @api.depends('workorder_ids.state')
    def _compute_workorder_done_count(self):
        data = self.env['mrp.workorder'].read_group([
            ('production_id', 'in', self.ids),
            ('state', '=', 'done')], ['production_id'], ['production_id'])
        count_data = dict((item['production_id'][0], item['production_id_count']) for item in data)
        for production in self:
            production.workorder_done_count = count_data.get(production.id, 0)

    @api.depends('move_raw_ids.state', 'move_finished_ids.state', 'workorder_ids', 'workorder_ids.state', 'qty_produced', 'move_raw_ids.quantity_done', 'product_qty')
    def _compute_state(self):
        """ Compute the production state. It use the same process than stock
        picking. It exists 3 extra steps for production:
        - planned: Workorder has been launched (workorders only)
        - progress: At least one item is produced.
        - to_close: The quantity produced is greater than the quantity to
        produce and all work orders has been finished.
        """

        # Manually track "state" and "reservation_state" since tracking doesn't work with computed
        # fields.
        tracking = not self._context.get("mail_notrack") and not self._context.get("tracking_disable")
        initial_values = {}
        if tracking:
            initial_values = dict(
                (production.id, {"state": production.state, "reservation_state": production.reservation_state})
                for production in self
            )

        # TODO: duplicated code with stock_picking.py
        for production in self:
            if not production.move_raw_ids:
                production.state = 'draft'
            elif all(move.state == 'draft' for move in production.move_raw_ids):
                production.state = 'draft'
            elif all(move.state == 'cancel' for move in production.move_raw_ids):
                production.state = 'cancel'
            elif all(move.state in ['cancel', 'done'] for move in production.move_raw_ids):
                if (
                    production.bom_id.consumption == 'flexible'
                    and float_compare(production.qty_produced, production.product_qty, precision_rounding=production.product_uom_id.rounding) == -1
                    and production.state != 'done'
                ):
                    production.state = 'progress'
                else:
                    production.state = 'done'
            elif production.move_finished_ids.filtered(lambda m: m.state not in ('cancel', 'done') and m.product_id.id == production.product_id.id)\
                 and (float_compare(production.qty_produced, production.product_qty, precision_rounding=production.product_uom_id.rounding) >= 0)\
                 and (not production.routing_id or all(wo_state in ('cancel', 'done') for wo_state in production.workorder_ids.mapped('state'))):
                production.state = 'to_close'
            elif production.workorder_ids and any(wo_state in ('progress') for wo_state in production.workorder_ids.mapped('state'))\
                 or float_compare(production.qty_produced, 0, precision_rounding=production.product_uom_id.rounding) > 0 \
                    and float_compare(production.qty_produced, production.product_qty, precision_rounding=production.product_uom_id.rounding) < 0:
                production.state = 'progress'
            elif production.workorder_ids:
                production.state = 'planned'
            else:
                production.state = 'confirmed'

            # Compute reservation state
            # State where the reservation does not matter.
            production.reservation_state = False
            # Compute reservation state according to its component's moves.
            if production.state not in ('draft', 'done', 'cancel'):
                relevant_move_state = production.move_raw_ids._get_relevant_state_among_moves()
                if relevant_move_state == 'partially_available':
                    if production.routing_id and production.routing_id.operation_ids and production.bom_id.ready_to_produce == 'asap':
                        production.reservation_state = production._get_ready_to_produce_state()
                    else:
                        production.reservation_state = 'confirmed'
                elif relevant_move_state != 'draft':
                    production.reservation_state = relevant_move_state

        if tracking and initial_values:
            self.message_track(self.fields_get(["state", "reservation_state"]), initial_values)

    @api.depends('move_raw_ids', 'is_locked', 'state', 'move_raw_ids.quantity_done')
    def _compute_unreserve_visible(self):
        for order in self:
            already_reserved = order.is_locked and order.state not in ('done', 'cancel') and order.mapped('move_raw_ids.move_line_ids')
            any_quantity_done = any([m.quantity_done > 0 for m in order.move_raw_ids])
            order.unreserve_visible = not any_quantity_done and already_reserved

    @api.depends('move_finished_ids.quantity_done', 'move_finished_ids.state', 'is_locked')
    def _compute_post_visible(self):
        for order in self:
            order.post_visible = order.is_locked and any((x.quantity_done > 0 and x.state not in ['done', 'cancel']) for x in order.move_finished_ids)

    @api.depends('workorder_ids.state', 'move_finished_ids', 'move_finished_ids.quantity_done', 'is_locked')
    def _get_produced_qty(self):
        for production in self:
            done_moves = production.move_finished_ids.filtered(lambda x: x.state != 'cancel' and x.product_id.id == production.product_id.id)
            qty_produced = sum(done_moves.mapped('quantity_done'))
            production.qty_produced = qty_produced
        return True

    def _compute_scrap_move_count(self):
        data = self.env['stock.scrap'].read_group([('production_id', 'in', self.ids)], ['production_id'], ['production_id'])
        count_data = dict((item['production_id'][0], item['production_id_count']) for item in data)
        for production in self:
            production.scrap_count = count_data.get(production.id, 0)

    _sql_constraints = [
        ('name_uniq', 'unique(name, company_id)', 'Reference must be unique per Company!'),
        ('qty_positive', 'check (product_qty > 0)', 'The quantity to produce must be positive!'),
    ]

    @api.onchange('company_id')
    def onchange_company_id(self):
        if self.company_id:
            if self.move_raw_ids:
                self.move_raw_ids.update({'company_id': self.company_id})
            if self.picking_type_id and self.picking_type_id.company_id != self.company_id:
                self.picking_type_id = self.env['stock.picking.type'].search([
                    ('code', '=', 'mrp_operation'),
                    ('warehouse_id.company_id', '=', self.company_id.id),
                ], limit=1).id

    @api.onchange('product_id', 'picking_type_id', 'company_id')
    def onchange_product_id(self):
        """ Finds UoM of changed product. """
        if not self.product_id:
            self.bom_id = False
        else:
            bom = self.env['mrp.bom']._bom_find(product=self.product_id, picking_type=self.picking_type_id, company_id=self.company_id.id, bom_type='normal')
            if bom:
                self.bom_id = bom.id
                self.product_qty = self.bom_id.product_qty
                self.product_uom_id = self.bom_id.product_uom_id.id
            else:
                self.bom_id = False
                self.product_uom_id = self.product_id.uom_id.id
            return {'domain': {'product_uom_id': [('category_id', '=', self.product_id.uom_id.category_id.id)]}}

    @api.onchange('bom_id')
    def _onchange_bom_id(self):
        self.product_qty = self.bom_id.product_qty
        self.product_uom_id = self.bom_id.product_uom_id.id
        self.move_raw_ids = [(2, move.id) for move in self.move_raw_ids.filtered(lambda m: m.bom_line_id)]
        self.picking_type_id = self.bom_id.picking_type_id or self.picking_type_id

    @api.onchange('date_planned_start')
    def _onchange_date_planned_start(self):
        self.move_raw_ids.update({
            'date': self.date_planned_start,
            'date_expected': self.date_planned_start,
        })
        if self.date_planned_start and not self.routing_id:
            self.date_planned_finished = self.date_planned_start + datetime.timedelta(hours=1)

    @api.onchange('bom_id', 'product_id', 'product_qty', 'product_uom_id')
    def _onchange_move_raw(self):
        # Clear move raws if we are changing the product. In case of creation (self._origin is empty),
        # we need to avoid keeping incorrect lines, so clearing is necessary too.
        if self.product_id != self._origin.product_id:
            self.move_raw_ids = [(5,)]
        if self.bom_id and self.product_qty > 0:
            # keep manual entries
            list_move_raw = [(4, move.id) for move in self.move_raw_ids.filtered(lambda m: not m.bom_line_id)]
            moves_raw_values = self._get_moves_raw_values()
            move_raw_dict = {move.bom_line_id.id: move for move in self.move_raw_ids.filtered(lambda m: m.bom_line_id)}
            for move_raw_values in moves_raw_values:
                if move_raw_values['bom_line_id'] in move_raw_dict:
                    # update existing entries
                    list_move_raw += [(1, move_raw_dict[move_raw_values['bom_line_id']].id, move_raw_values)]
                else:
                    # add new entries
                    list_move_raw += [(0, 0, move_raw_values)]
            self.move_raw_ids = list_move_raw
        else:
            self.move_raw_ids = [(2, move.id) for move in self.move_raw_ids.filtered(lambda m: m.bom_line_id)]

    @api.onchange('location_src_id', 'move_raw_ids', 'routing_id')
    def _onchange_location(self):
        source_location = self.location_src_id
        self.move_raw_ids.update({
            'warehouse_id': source_location.get_warehouse().id,
            'location_id': source_location.id,
        })

    @api.onchange('picking_type_id')
    def onchange_picking_type(self):
        location = self.env.ref('stock.stock_location_stock')
        try:
            location.check_access_rule('read')
        except (AttributeError, AccessError):
            location = self.env['stock.warehouse'].search([('company_id', '=', self.env.company.id)], limit=1).lot_stock_id
        self.move_raw_ids.update({'picking_type_id': self.picking_type_id})
        self.location_src_id = self.picking_type_id.default_location_src_id.id or location.id
        self.location_dest_id = self.picking_type_id.default_location_dest_id.id or location.id

    @api.constrains('product_id', 'move_raw_ids')
    def _check_production_lines(self):
        for production in self:
            for move in production.move_raw_ids:
                if production.product_id == move.product_id:
                    raise ValidationError(_("The component %s should not be the same as the product to produce.") % production.product_id.display_name)

    def write(self, vals):
        res = super(MrpProduction, self).write(vals)
        if 'date_planned_start' in vals:
            moves = (self.mapped('move_raw_ids') + self.mapped('move_finished_ids')).filtered(
                lambda r: r.state not in ['done', 'cancel'])
            moves.write({
                'date_expected': fields.Datetime.to_datetime(vals['date_planned_start']),
            })
        for production in self:
            if 'date_planned_start' in vals:
                if production.state in ['done', 'cancel']:
                    raise UserError(_('You cannot move a manufacturing order once it is cancelled or done.'))
                if production.workorder_ids and not self.env.context.get('force_date', False):
                    raise UserError(_('You cannot move a planned manufacturing order.'))
            if 'move_raw_ids' in vals and production.state != 'draft':
                production.move_raw_ids.filtered(lambda m: m.state == 'draft')._action_confirm()
        return res

    @api.model
    def create(self, values):
        if not values.get('name', False) or values['name'] == _('New'):
            picking_type_id = values.get('picking_type_id') or self._get_default_picking_type()
            picking_type_id = self.env['stock.picking.type'].browse(picking_type_id)
            if picking_type_id:
                values['name'] = picking_type_id.sequence_id.next_by_id()
            else:
                values['name'] = self.env['ir.sequence'].next_by_code('mrp.production') or _('New')
        if not values.get('procurement_group_id'):
            procurement_group_vals = self._prepare_procurement_group_vals(values)
            values['procurement_group_id'] = self.env["procurement.group"].create(procurement_group_vals).id
        production = super(MrpProduction, self).create(values)
        production.move_raw_ids.write({
            'group_id': production.procurement_group_id.id,
        })
        # Trigger move_raw creation when importing a file
        if 'import_file' in self.env.context:
            production._onchange_move_raw()
        return production

    def copy(self, default=None):
        """
        When there is a kit in the child bom of that production, we should not copy the workorder
        """
        res = super().copy(default)
        if any(bom.type == 'phantom' for bom in self.bom_id.bom_line_ids.child_bom_id):
            res.move_raw_ids.workorder_id = False
        return res

    def unlink(self):
        if any(production.state == 'done' for production in self):
            raise UserError(_('Cannot delete a manufacturing order in done state.'))
        self.action_cancel()
        not_cancel = self.filtered(lambda m: m.state != 'cancel')
        if not_cancel:
            productions_name = ', '.join([prod.display_name for prod in not_cancel])
            raise UserError(_('%s cannot be deleted. Try to cancel them before.') % productions_name)

        workorders_to_delete = self.workorder_ids.filtered(lambda wo: wo.state != 'done')
        if workorders_to_delete:
            workorders_to_delete.unlink()
        return super(MrpProduction, self).unlink()

    def action_toggle_is_locked(self):
        self.ensure_one()
        self.is_locked = not self.is_locked
        return True

    def _get_finished_move_value(self, product_id, product_uom_qty, product_uom, operation_id=False, byproduct_id=False):
        return {
            'product_id': product_id,
            'product_uom_qty': product_uom_qty,
            'product_uom': product_uom,
            'operation_id': operation_id,
            'byproduct_id': byproduct_id,
            'unit_factor': product_uom_qty / self.product_qty,
            'name': self.name,
            'date': self.date_planned_start,
            'date_expected': self.date_planned_finished,
            'picking_type_id': self.picking_type_id.id,
            'location_id': self.product_id.with_context(force_company=self.company_id.id).property_stock_production.id,
            'location_dest_id': self.location_dest_id.id,
            'company_id': self.company_id.id,
            'production_id': self.id,
            'warehouse_id': self.location_dest_id.get_warehouse().id,
            'origin': self.name,
            'group_id': self.procurement_group_id.id,
            'propagate_cancel': self.propagate_cancel,
            'propagate_date': self.propagate_date,
            'propagate_date_minimum_delta': self.propagate_date_minimum_delta,
            'move_dest_ids': [(4, x.id) for x in self.move_dest_ids if not byproduct_id],
        }

    def _generate_finished_moves(self):
        if self.product_id in self.bom_id.byproduct_ids.mapped('product_id'):
            raise UserError(_("You cannot have %s  as the finished product and in the Byproducts") % self.product_id.name)
        moves_values = [self._get_finished_move_value(self.product_id.id, self.product_qty, self.product_uom_id.id)]
        for byproduct in self.bom_id.byproduct_ids:
            product_uom_factor = self.product_uom_id._compute_quantity(self.product_qty, self.bom_id.product_uom_id)
            qty = byproduct.product_qty * (product_uom_factor / self.bom_id.product_qty)
            move_values = self._get_finished_move_value(byproduct.product_id.id,
                qty, byproduct.product_uom_id.id, byproduct.operation_id.id,
                byproduct.id)
            moves_values.append(move_values)
        moves = self.env['stock.move'].create(moves_values)
        return moves

    def _get_moves_raw_values(self):
        moves = []
        for production in self:
            factor = production.product_uom_id._compute_quantity(production.product_qty, production.bom_id.product_uom_id) / production.bom_id.product_qty
            boms, lines = production.bom_id.explode(production.product_id, factor, picking_type=production.bom_id.picking_type_id)
            for bom_line, line_data in lines:
                if bom_line.child_bom_id and bom_line.child_bom_id.type == 'phantom' or\
                        bom_line.product_id.type not in ['product', 'consu']:
                    continue
                moves.append(production._get_move_raw_values(bom_line, line_data))
        return moves

    def _get_move_raw_values(self, bom_line, line_data):
        quantity = line_data['qty']
        # alt_op needed for the case when you explode phantom bom and all the lines will be consumed in the operation given by the parent bom line
        alt_op = line_data['parent_line'] and line_data['parent_line'].operation_id.id or False
        source_location = self.location_src_id
        data = {
            'sequence': bom_line.sequence,
            'name': self.name,
            'reference': self.name,
            'date': self.date_planned_start,
            'date_expected': self.date_planned_start,
            'bom_line_id': bom_line.id,
            'picking_type_id': self.picking_type_id.id,
            'product_id': bom_line.product_id.id,
            'product_uom_qty': quantity,
            'product_uom': bom_line.product_uom_id.id,
            'location_id': source_location.id,
            'location_dest_id': self.product_id.with_context(force_company=self.company_id.id).property_stock_production.id,
            'raw_material_production_id': self.id,
            'company_id': self.company_id.id,
            'operation_id': bom_line.operation_id.id or alt_op,
            'price_unit': bom_line.product_id.standard_price,
            'procure_method': 'make_to_stock',
            'origin': self.name,
            'state': 'draft',
            'warehouse_id': source_location.get_warehouse().id,
            'group_id': self.procurement_group_id.id,
            'propagate_cancel': self.propagate_cancel,
        }
        return data

    def _update_raw_move(self, bom_line, line_data):
        """ :returns update_move, old_quantity, new_quantity """
        quantity = line_data['qty']
        self.ensure_one()
        move = self.move_raw_ids.filtered(lambda x: x.bom_line_id.id == bom_line.id and x.state not in ('done', 'cancel'))
        if move:
            old_qty = move[0].product_uom_qty
            remaining_qty = move[0].raw_material_production_id.product_qty - move[0].raw_material_production_id.qty_produced
            if quantity > 0:
                move[0].write({'product_uom_qty': quantity})
                move[0]._recompute_state()
                move[0]._action_assign()
                move[0].unit_factor = remaining_qty and (quantity - move[0].quantity_done) / remaining_qty or 1.0
                return move[0], old_qty, quantity
            else:
                if move[0].quantity_done > 0:
                    raise UserError(_('Lines need to be deleted, but can not as you still have some quantities to consume in them. '))
                move[0]._action_cancel()
                move[0].unlink()
                return self.env['stock.move'], old_qty, quantity
        else:
            move_values = self._get_move_raw_values(bom_line, line_data)
            move_values['state'] = 'confirmed'
            move = self.env['stock.move'].create(move_values)
            return move, 0, quantity

    def _get_ready_to_produce_state(self):
        """ returns 'assigned' if enough components are reserved in order to complete
        the first operation in the routing. If not returns 'waiting'
        """
        self.ensure_one()
        first_operation = self.routing_id.operation_ids[0]
        # Get BoM line related to first opeation in rounting. If there is only
        # one opeation in the routing then it will need all BoM lines.
        bom_line_ids = self.env['mrp.bom.line']
        if len(self.routing_id.operation_ids) == 1:
            bom_line_ids = self.bom_id.bom_line_ids
        else:
            bom_line_ids = self.bom_id.bom_line_ids.filtered(lambda bl: bl.operation_id == first_operation)
        bom_line_ids = bom_line_ids.filtered(lambda bl: not bl._skip_bom_line(self.product_id))

        moves_in_first_operation = self.move_raw_ids.filtered(lambda m: m.bom_line_id in bom_line_ids)
        if all(move.state == 'assigned' for move in moves_in_first_operation):
            return 'assigned'
        return 'confirmed'

    def action_confirm(self):
        self._check_company()
        for production in self:
            # Avoid confirming it twice
            if production.state != 'draft':
                continue
            if not production.move_raw_ids:
                raise UserError(_("Add some materials to consume before marking this MO as to do."))
            for move_raw in production.move_raw_ids:
                move_raw.write({
                    'unit_factor': move_raw.product_uom_qty / production.product_qty,
                })
            production._generate_finished_moves()
            production.move_raw_ids._adjust_procure_method()
            (production.move_raw_ids | production.move_finished_ids)._action_confirm()
        return True

    def action_assign(self):
        for production in self:
            production.move_raw_ids._action_assign()
            production.workorder_ids._refresh_wo_lines()
        return True

    def open_produce_product(self):
        self.ensure_one()
        if self.bom_id.type == 'phantom':
            raise UserError(_('You cannot produce a MO with a bom kit product.'))
        action = self.env.ref('mrp.act_mrp_product_produce').read()[0]
        return action

    def button_plan(self):
        """ Create work orders. And probably do stuff, like things. """
        orders_to_plan = self.filtered(lambda order: order.routing_id and order.state == 'confirmed')
        for order in orders_to_plan:
            order.move_raw_ids.filtered(lambda m: m.state == 'draft')._action_confirm()
            quantity = order.product_uom_id._compute_quantity(order.product_qty, order.bom_id.product_uom_id) / order.bom_id.product_qty
            boms, lines = order.bom_id.explode(order.product_id, quantity, picking_type=order.bom_id.picking_type_id)
            order._generate_workorders(boms)
            order._plan_workorders()
        return True

    def _get_start_date(self):
        return self.date_start_wo or datetime.datetime.now()

    def _plan_workorders(self):
        """ Plan all the production's workorders depending on the workcenters
        work schedule"""
        self.ensure_one()

        # Schedule all work orders (new ones and those already created)
        qty_to_produce = max(self.product_qty - self.qty_produced, 0)
        qty_to_produce = self.product_uom_id._compute_quantity(qty_to_produce, self.product_id.uom_id)
        start_date = self._get_start_date()
        for workorder in self.workorder_ids:
            workcenters = workorder.workcenter_id | workorder.workcenter_id.alternative_workcenter_ids

            best_finished_date = datetime.datetime.max
            vals = {}
            for workcenter in workcenters:
                # compute theoretical duration
                time_cycle = workorder.operation_id.time_cycle
                cycle_number = float_round(qty_to_produce / workcenter.capacity, precision_digits=0, rounding_method='UP')
                duration_expected = workcenter.time_start + workcenter.time_stop + cycle_number * time_cycle * 100.0 / workcenter.time_efficiency

                # get first free slot
                # planning 0 hours gives the start of the next attendance
                from_date = workcenter.resource_calendar_id.plan_hours(0, start_date, compute_leaves=True, resource=workcenter.resource_id, domain=[('time_type', 'in', ['leave', 'other'])])
                # If the workcenter is unavailable, try planning on the next one
                if from_date is False:
                    continue
                to_date = workcenter.resource_calendar_id.plan_hours(duration_expected / 60.0, from_date, compute_leaves=True, resource=workcenter.resource_id, domain=[('time_type', 'in', ['leave', 'other'])])

                # Check if this workcenter is better than the previous ones
                if to_date and to_date < best_finished_date:
                    best_start_date = from_date
                    best_finished_date = to_date
                    best_workcenter = workcenter
                    vals = {
                        'workcenter_id': workcenter.id,
                        'capacity': workcenter.capacity,
                        'duration_expected': duration_expected,
                    }

            # If none of the workcenter are available, raise
            if best_finished_date == datetime.datetime.max:
                raise UserError(_('Impossible to plan the workorder. Please check the workcenter availabilities.'))

            # Instantiate start_date for the next workorder planning
            if workorder.next_work_order_id:
                if workorder.operation_id.batch == 'no' or workorder.operation_id.batch_size >= qty_to_produce:
                    start_date = best_finished_date
                else:
                    cycle_number = float_round(workorder.operation_id.batch_size / best_workcenter.capacity, precision_digits=0, rounding_method='UP')
                    duration = best_workcenter.time_start + cycle_number * workorder.operation_id.time_cycle * 100.0 / best_workcenter.time_efficiency
                    start_date = best_workcenter.resource_calendar_id.plan_hours(duration / 60.0, best_start_date, compute_leaves=True, resource=best_workcenter.resource_id, domain=[('time_type', 'in', ['leave', 'other'])])

            # Create leave on chosen workcenter calendar
            leave = self.env['resource.calendar.leaves'].create({
                'name': self.name + ' - ' + workorder.name,
                'calendar_id': best_workcenter.resource_calendar_id.id,
                'date_from': best_start_date,
                'date_to': best_finished_date,
                'resource_id': best_workcenter.resource_id.id,
                'time_type': 'other'
            })
            vals['leave_id'] = leave.id
            workorder.write(vals)
        self.with_context(force_date=True).write({
            'date_planned_start': self.workorder_ids[0].date_planned_start,
            'date_planned_finished': self.workorder_ids[-1].date_planned_finished
        })

    def button_unplan(self):
        if any(wo.state == 'done' for wo in self.workorder_ids):
            raise UserError(_("Some work orders are already done, you cannot unplan this manufacturing order."))
        elif any(wo.state == 'progress' for wo in self.workorder_ids):
            raise UserError(_("Some work orders have already started, you cannot unplan this manufacturing order."))
        self.workorder_ids.unlink()

    def _generate_workorders(self, exploded_boms):
        workorders = self.env['mrp.workorder']
        original_one = False
        for bom, bom_data in exploded_boms:
            # If the routing of the parent BoM and phantom BoM are the same, don't recreate work orders, but use one master routing
            if bom.routing_id.id and (not bom_data['parent_line'] or bom_data['parent_line'].bom_id.routing_id.id != bom.routing_id.id):
                temp_workorders = self._workorders_create(bom, bom_data)
                workorders += temp_workorders
                if temp_workorders: # In order to avoid two "ending work orders"
                    if original_one:
                        temp_workorders[-1].next_work_order_id = original_one
                    original_one = temp_workorders[0]
        return workorders

    def _workorders_create(self, bom, bom_data):
        """
        :param bom: in case of recursive boms: we could create work orders for child
                    BoMs
        """
        workorders = self.env['mrp.workorder']

        # Initial qty producing
        quantity = max(self.product_qty - sum(self.move_finished_ids.filtered(lambda move: move.product_id == self.product_id).mapped('quantity_done')), 0)
        if self.product_id.tracking == 'serial':
            quantity = 1.0

        for operation in bom.routing_id.operation_ids:
            workorder_vals = self._prepare_workorder_vals(
                operation, workorders, quantity)
            workorder = workorders.create(workorder_vals)
            if workorders:
                workorders[-1].next_work_order_id = workorder.id
                workorders[-1]._start_nextworkorder()
            workorders += workorder

            moves_raw = self.move_raw_ids.filtered(lambda move: move.operation_id == operation and move.bom_line_id.bom_id.routing_id == bom.routing_id)
            moves_finished = self.move_finished_ids.filtered(lambda move: move.operation_id == operation)

            # - Raw moves from a BoM where a routing was set but no operation was precised should
            #   be consumed at the last workorder of the linked routing.
            # - Raw moves from a BoM where no rounting was set should be consumed at the last
            #   workorder of the main routing.
            if len(workorders) == len(bom.routing_id.operation_ids):
                moves_raw |= self.move_raw_ids.filtered(lambda move: not move.operation_id and move.bom_line_id.bom_id.routing_id == bom.routing_id)
                moves_raw |= self.move_raw_ids.filtered(lambda move: not move.workorder_id and not move.bom_line_id.bom_id.routing_id)

                moves_finished |= self.move_finished_ids.filtered(lambda move: move.product_id != self.product_id and not move.operation_id)

            moves_raw.mapped('move_line_ids').write({'workorder_id': workorder.id})
            (moves_finished | moves_raw).write({'workorder_id': workorder.id})

            workorder._generate_wo_lines()
        return workorders

    def _check_lots(self):
        # Check that the components were consumed for lots that we have produced.
        if self.product_id.tracking != 'none':
            finished_lots = self.finished_move_line_ids.mapped('lot_id')
            raw_finished_lots = self.move_raw_ids.mapped('move_line_ids.lot_produced_ids')
            if (raw_finished_lots - finished_lots):
                lots_short = raw_finished_lots - finished_lots
                error_msg = _(
                    'Some components have been consumed for a lot/serial number that has not been produced. '
                    'Unlock the MO and click on the components lines to correct it.\n'
                    'List of the components:\n'
                )
                move_lines = self.move_raw_ids.mapped('move_line_ids').filtered(lambda ml: lots_short & ml.lot_produced_ids)
                for ml in move_lines:
                    error_msg += ml.product_id.display_name + ' (' + ', '.join((lots_short & ml.lot_produced_ids).mapped('name')) + ')\n'
                raise UserError(error_msg)

    def action_cancel(self):
        """ Cancels production order, unfinished stock moves and set procurement
        orders in exception """
        if not self.move_raw_ids:
            self.state = 'cancel'
            return True
        self._action_cancel()
        return True

    def _action_cancel(self):
        documents_by_production = {}
        for production in self:
            documents = defaultdict(list)
            for move_raw_id in self.move_raw_ids.filtered(lambda m: m.state not in ('done', 'cancel')):
                iterate_key = self._get_document_iterate_key(move_raw_id)
                if iterate_key:
                    document = self.env['stock.picking']._log_activity_get_documents({move_raw_id: (move_raw_id.product_uom_qty, 0)}, iterate_key, 'UP')
                    for key, value in document.items():
                        documents[key] += [value]
            if documents:
                documents_by_production[production] = documents

        self.workorder_ids.filtered(lambda x: x.state not in ['done', 'cancel']).action_cancel()
        finish_moves = self.move_finished_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
        raw_moves = self.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
        (finish_moves | raw_moves)._action_cancel()
        picking_ids = self.picking_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
        picking_ids.action_cancel()

        for production, documents in documents_by_production.items():
            filtered_documents = {}
            for (parent, responsible), rendering_context in documents.items():
                if not parent or parent._name == 'stock.picking' and parent.state == 'cancel' or parent == production:
                    continue
                filtered_documents[(parent, responsible)] = rendering_context
            production._log_manufacture_exception(filtered_documents, cancel=True)

        # In case of a flexible BOM, we don't know from the state of the moves if the MO should
        # remain in progress or done. Indeed, if all moves are done/cancel but the quantity produced
        # is lower than expected, it might mean:
        # - we have used all components but we still want to produce the quantity expected
        # - we have used all components and we won't be able to produce the last units
        #
        # However, if the user clicks on 'Cancel', it is expected that the MO is either done or
        # canceled. If the MO is still in progress at this point, it means that the move raws
        # are either all done or a mix of done / canceled => the MO should be done.
        self.filtered(lambda p: p.state not in ['done', 'cancel'] and p.bom_id.consumption == 'flexible').write({'state': 'done'})

        return True

    def _get_document_iterate_key(self, move_raw_id):
        return move_raw_id.move_orig_ids and 'move_orig_ids' or False

    def _cal_price(self, consumed_moves):
        self.ensure_one()
        return True

    def post_inventory(self):
        for order in self:
            # In case the routing allows multiple WO running at the same time, it is possible that
            # the quantity produced in one of the workorders is lower than the quantity produced in
            # the MO.
            if order.product_id.tracking != "none" and any(
                wo.state not in ["done", "cancel"]
                and float_compare(wo.qty_produced, order.qty_produced, precision_rounding=order.product_uom_id.rounding) == -1
                for wo in order.workorder_ids
            ):
                raise UserError(
                    _(
                        "At least one work order has a quantity produced lower than the quantity produced in the manufacturing order. "
                        + "You must complete the work orders before posting the inventory."
                    )
                )

            if not any(order.move_raw_ids.mapped('quantity_done')):
                raise UserError(_("You must indicate a non-zero amount consumed for at least one of your components"))

            moves_not_to_do = order.move_raw_ids.filtered(lambda x: x.state == 'done')
            moves_to_do = order.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
            for move in moves_to_do.filtered(lambda m: m.product_qty == 0.0 and m.quantity_done > 0):
                move.product_uom_qty = move.quantity_done
            # MRP do not merge move, catch the result of _action_done in order
            # to get extra moves.
            moves_to_do = moves_to_do._action_done()
            moves_to_do = order.move_raw_ids.filtered(lambda x: x.state == 'done') - moves_not_to_do
            order._cal_price(moves_to_do)
            moves_to_finish = order.move_finished_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
            moves_to_finish = moves_to_finish._action_done()
            order.workorder_ids.mapped('raw_workorder_line_ids').unlink()
            order.workorder_ids.mapped('finished_workorder_line_ids').unlink()
            order.action_assign()
            consume_move_lines = moves_to_do.mapped('move_line_ids')
            for moveline in moves_to_finish.mapped('move_line_ids'):
                if moveline.move_id.has_tracking != 'none' and moveline.product_id == order.product_id or moveline.lot_id in consume_move_lines.mapped('lot_produced_ids'):
                    if any([not ml.lot_produced_ids for ml in consume_move_lines]):
                        raise UserError(_('You can not consume without telling for which lot you consumed it'))
                    # Link all movelines in the consumed with same lot_produced_ids false or the correct lot_produced_ids
                    filtered_lines = consume_move_lines.filtered(lambda ml: moveline.lot_id in ml.lot_produced_ids)
                    moveline.write({'consume_line_ids': [(6, 0, [x for x in filtered_lines.ids])]})
                else:
                    # Link with everything
                    moveline.write({'consume_line_ids': [(6, 0, [x for x in consume_move_lines.ids])]})
        return True

    def button_mark_done(self):
        self.ensure_one()
        self._check_company()
        for wo in self.workorder_ids:
            if wo.time_ids.filtered(lambda x: (not x.date_end) and (x.loss_type in ('productive', 'performance'))):
                raise UserError(_('Work order %s is still running') % wo.name)
        self._check_lots()

        self.post_inventory()
        # Moves without quantity done are not posted => set them as done instead of canceling. In
        # case the user edits the MO later on and sets some consumed quantity on those, we do not
        # want the move lines to be canceled.
        (self.move_raw_ids | self.move_finished_ids).filtered(lambda x: x.state not in ('done', 'cancel')).write({
            'state': 'done',
            'product_uom_qty': 0.0,
        })
        return self.write({'date_finished': fields.Datetime.now()})

    def do_unreserve(self):
        for production in self:
            production.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel'))._do_unreserve()
        return True

    def button_unreserve(self):
        self.ensure_one()
        self.do_unreserve()
        return True

    def button_scrap(self):
        self.ensure_one()
        return {
            'name': _('Scrap'),
            'view_mode': 'form',
            'res_model': 'stock.scrap',
            'view_id': self.env.ref('stock.stock_scrap_form_view2').id,
            'type': 'ir.actions.act_window',
            'context': {'default_production_id': self.id,
                        'product_ids': (self.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel')) | self.move_finished_ids.filtered(lambda x: x.state == 'done')).mapped('product_id').ids,
                        'default_company_id': self.company_id.id
                        },
            'target': 'new',
        }

    def action_see_move_scrap(self):
        self.ensure_one()
        action = self.env.ref('stock.action_stock_scrap').read()[0]
        action['domain'] = [('production_id', '=', self.id)]
        action['context'] = dict(self._context, default_origin=self.name)
        return action

    @api.model
    def get_empty_list_help(self, help):
        self = self.with_context(
            empty_list_help_document_name=_("manufacturing order"),
        )
        return super(MrpProduction, self).get_empty_list_help(help)

    def _log_downside_manufactured_quantity(self, moves_modification):

        def _keys_in_sorted(move):
            """ sort by picking and the responsible for the product the
            move.
            """
            return (move.picking_id.id, move.product_id.responsible_id.id)

        def _keys_in_groupby(move):
            """ group by picking and the responsible for the product the
            move.
            """
            return (move.picking_id, move.product_id.responsible_id)

        def _render_note_exception_quantity_mo(rendering_context):
            values = {
                'production_order': self,
                'order_exceptions': dict((key, d[key]) for d in rendering_context for key in d),
                'impacted_pickings': False,
                'cancel': False
            }
            return self.env.ref('mrp.exception_on_mo').render(values=values)

        documents = {}
        for move, (old_qty, new_qty) in moves_modification.items():
            document = self.env['stock.picking']._log_activity_get_documents(
                {move: (old_qty, new_qty)}, 'move_dest_ids', 'DOWN', _keys_in_sorted, _keys_in_groupby)
            for key, value in document.items():
                if documents.get(key):
                    documents[key] += [value]
                else:
                    documents[key] = [value]
        self.env['stock.picking']._log_activity(_render_note_exception_quantity_mo, documents)

    def _log_manufacture_exception(self, documents, cancel=False):

        def _render_note_exception_quantity_mo(rendering_context):
            visited_objects = []
            order_exceptions = {}
            for exception in rendering_context:
                order_exception, visited = exception
                order_exceptions.update(order_exception)
                visited_objects += visited
            visited_objects = [sm for sm in visited_objects if sm._name == 'stock.move']
            impacted_object = []
            if visited_objects:
                visited_objects = self.env[visited_objects[0]._name].concat(*visited_objects)
                visited_objects |= visited_objects.mapped('move_orig_ids')
                impacted_object = visited_objects.filtered(lambda m: m.state not in ('done', 'cancel')).mapped('picking_id')
            values = {
                'production_order': self,
                'order_exceptions': order_exceptions,
                'impacted_object': impacted_object,
                'cancel': cancel
            }
            return self.env.ref('mrp.exception_on_mo').render(values=values)

        self.env['stock.picking']._log_activity(_render_note_exception_quantity_mo, documents)

    def _prepare_workorder_vals(self, operation, workorders, quantity):
        self.ensure_one()
        todo_uom = self.product_uom_id.id
        if self.product_id.tracking == 'serial' and self.product_uom_id.uom_type != 'reference':
            todo_uom = self.env['uom.uom'].search([('category_id', '=', self.product_uom_id.category_id.id), ('uom_type', '=', 'reference')]).id
        return {
            'name': operation.name,
            'production_id': self.id,
            'workcenter_id': operation.workcenter_id.id,
            'product_uom_id': todo_uom,
            'operation_id': operation.id,
            'state': len(workorders) == 0 and 'ready' or 'pending',
            'qty_producing': quantity,
            'consumption': self.bom_id.consumption,
        }

    @api.model
    def _prepare_procurement_group_vals(self, values):
        return {'name': values['name']}

```

## File: models\mrp_routing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class MrpRouting(models.Model):
    """ Specifies routings of work centers """
    _name = 'mrp.routing'
    _description = 'Routings'

    name = fields.Char('Routing', required=True)
    active = fields.Boolean(
        'Active', default=True,
        help="If the active field is set to False, it will allow you to hide the routing without removing it.")
    code = fields.Char(
        'Reference',
        copy=False, default=lambda self: _('New'), readonly=True)
    note = fields.Text('Description')
    operation_ids = fields.One2many(
        'mrp.routing.workcenter', 'routing_id', 'Operations',
        copy=True)
    company_id = fields.Many2one(
        'res.company', 'Company', default=lambda self: self.env.company)

    @api.model
    def create(self, vals):
        if 'code' not in vals or vals['code'] == _('New'):
            vals['code'] = self.env['ir.sequence'].next_by_code('mrp.routing') or _('New')
        return super(MrpRouting, self).create(vals)


class MrpRoutingWorkcenter(models.Model):
    _name = 'mrp.routing.workcenter'
    _description = 'Work Center Usage'
    _order = 'sequence, id'
    _check_company_auto = True

    name = fields.Char('Operation', required=True)
    workcenter_id = fields.Many2one('mrp.workcenter', 'Work Center', required=True, check_company=True)
    sequence = fields.Integer(
        'Sequence', default=100,
        help="Gives the sequence order when displaying a list of routing Work Centers.")
    routing_id = fields.Many2one(
        'mrp.routing', 'Parent Routing',
        index=True, ondelete='cascade', required=True,
        help="The routing contains all the Work Centers used and for how long. This will create work orders afterwards "
        "which alters the execution of the manufacturing order.")
    note = fields.Text('Description')
    company_id = fields.Many2one(
        'res.company', 'Company',
        readonly=True, related='routing_id.company_id', store=True)
    worksheet = fields.Binary('PDF', help="Upload your PDF file.")
    worksheet_type = fields.Selection([
        ('pdf', 'PDF'), ('google_slide', 'Google Slide')],
        string="Work Sheet", default="pdf",
        help="Defines if you want to use a PDF or a Google Slide as work sheet."
    )
    worksheet_google_slide = fields.Char('Google Slide', help="Paste the url of your Google Slide. Make sure the access to the document is public.")
    time_mode = fields.Selection([
        ('auto', 'Compute based on real time'),
        ('manual', 'Set duration manually')], string='Duration Computation',
        default='manual')
    time_mode_batch = fields.Integer('Based on', default=10)
    time_cycle_manual = fields.Float(
        'Manual Duration', default=60,
        help="Time in minutes. Is the time used in manual mode, or the first time supposed in real time when there are not any work orders yet.")
    time_cycle = fields.Float('Duration', compute="_compute_time_cycle")
    workorder_count = fields.Integer("# Work Orders", compute="_compute_workorder_count")
    batch = fields.Selection([
        ('no',  'Once all products are processed'),
        ('yes', 'Once some products are processed')], string='Start Next Operation',
        default='no', required=True)
    batch_size = fields.Float('Quantity to Process', default=1.0)
    workorder_ids = fields.One2many('mrp.workorder', 'operation_id', string="Work Orders")

    @api.depends('time_cycle_manual', 'time_mode', 'workorder_ids')
    def _compute_time_cycle(self):
        manual_ops = self.filtered(lambda operation: operation.time_mode == 'manual')
        for operation in manual_ops:
            operation.time_cycle = operation.time_cycle_manual
        for operation in self - manual_ops:
            data = self.env['mrp.workorder'].read_group([
                ('operation_id', '=', operation.id),
                ('state', '=', 'done')], ['operation_id', 'duration', 'qty_produced'], ['operation_id'],
                limit=operation.time_mode_batch)
            count_data = dict((item['operation_id'][0], (item['duration'], item['qty_produced'])) for item in data)
            if count_data.get(operation.id) and count_data[operation.id][1]:
                operation.time_cycle = (count_data[operation.id][0] / count_data[operation.id][1]) * (operation.workcenter_id.capacity or 1.0)
            else:
                operation.time_cycle = operation.time_cycle_manual

    def _compute_workorder_count(self):
        data = self.env['mrp.workorder'].read_group([
            ('operation_id', 'in', self.ids),
            ('state', '=', 'done')], ['operation_id'], ['operation_id'])
        count_data = dict((item['operation_id'][0], item['operation_id_count']) for item in data)
        for operation in self:
            operation.workorder_count = count_data.get(operation.id, 0)


```

## File: models\mrp_unbuild.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import AccessError, UserError
from odoo.tools import float_compare, float_round, float_is_zero


class MrpUnbuild(models.Model):
    _name = "mrp.unbuild"
    _description = "Unbuild Order"
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'id desc'

    name = fields.Char('Reference', copy=False, readonly=True, default=lambda x: _('New'))
    product_id = fields.Many2one(
        'product.product', 'Product', check_company=True,
        domain="[('bom_ids', '!=', False), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        required=True, states={'done': [('readonly', True)]})
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda s: s.env.company,
        required=True, index=True, states={'done': [('readonly', True)]})
    product_qty = fields.Float(
        'Quantity', default=1.0,
        required=True, states={'done': [('readonly', True)]})
    product_uom_id = fields.Many2one(
        'uom.uom', 'Unit of Measure',
        required=True, states={'done': [('readonly', True)]})
    bom_id = fields.Many2one(
        'mrp.bom', 'Bill of Material',
        domain="""[
        '|',
            ('product_id', '=', product_id),
            '&',
                ('product_tmpl_id.product_variant_ids', '=', product_id),
                ('product_id','=',False),
        ('type', '=', 'normal'),
        '|',
            ('company_id', '=', company_id),
            ('company_id', '=', False)
        ]
""",
        required=True, states={'done': [('readonly', True)]}, check_company=True)
    mo_id = fields.Many2one(
        'mrp.production', 'Manufacturing Order',
        domain="[('state', 'in', ['done', 'cancel']), ('company_id', '=', company_id)]",
        states={'done': [('readonly', True)]}, check_company=True)
    lot_id = fields.Many2one(
        'stock.production.lot', 'Lot/Serial Number',
        domain="[('product_id', '=', product_id), ('company_id', '=', company_id)]", check_company=True,
        states={'done': [('readonly', True)]}, help="Lot/Serial Number of the product to unbuild.")
    has_tracking=fields.Selection(related='product_id.tracking', readonly=True)
    location_id = fields.Many2one(
        'stock.location', 'Source Location',
        domain="[('usage','=','internal'), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        check_company=True,
        required=True, states={'done': [('readonly', True)]}, help="Location where the product you want to unbuild is.")
    location_dest_id = fields.Many2one(
        'stock.location', 'Destination Location',
        domain="[('usage','=','internal'), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        check_company=True,
        required=True, states={'done': [('readonly', True)]}, help="Location where you want to send the components resulting from the unbuild order.")
    consume_line_ids = fields.One2many(
        'stock.move', 'consume_unbuild_id', readonly=True,
        string='Consumed Disassembly Lines')
    produce_line_ids = fields.One2many(
        'stock.move', 'unbuild_id', readonly=True,
        string='Processed Disassembly Lines')
    state = fields.Selection([
        ('draft', 'Draft'),
        ('done', 'Done')], string='Status', default='draft', index=True)

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            warehouse = self.env['stock.warehouse'].search([('company_id', '=', self.company_id.id)], limit=1)
            self.location_id = warehouse.lot_stock_id
            self.location_dest_id = warehouse.lot_stock_id
        else:
            self.location_id = False
            self.location_dest_id = False

    @api.onchange('mo_id')
    def _onchange_mo_id(self):
        if self.mo_id:
            self.product_id = self.mo_id.product_id.id
            self.product_qty = self.mo_id.product_qty
            self.product_uom_id = self.mo_id.product_uom_id
            self.bom_id = self.mo_id.bom_id
            if self.lot_id and self.lot_id not in self.mo_id.move_finished_ids.move_line_ids.lot_id:
                return {'warning': {
                    'title': _("Warning"),
                    'message': _("The selected serial number does not correspond to the one used in the manufacturing order, please select another one.")
                }}

    @api.onchange('lot_id')
    def _onchange_lot_id(self):
        if self.mo_id and self.lot_id and self.lot_id not in self.mo_id.move_finished_ids.move_line_ids.lot_id:
            return {'warning': {
                'title': _("Warning"),
                'message': _("The selected serial number does not correspond to the one used in the manufacturing order, please select another one.")
            }}

    @api.onchange('product_id')
    def _onchange_product_id(self):
        if self.product_id:
            self.bom_id = self.env['mrp.bom']._bom_find(product=self.product_id, company_id=self.company_id.id)
            self.product_uom_id = self.mo_id.product_id == self.product_id and self.mo_id.product_uom_id.id or self.product_id.uom_id.id
            if self.company_id:
                return {'domain': {'mo_id': [('state', '=', 'done'), ('product_id', '=', self.product_id.id), ('company_id', '=', self.company_id.id)]}}

    @api.constrains('product_qty')
    def _check_qty(self):
        if self.product_qty <= 0:
            raise ValueError(_('Unbuild Order product quantity has to be strictly positive.'))

    @api.model
    def create(self, vals):
        if not vals.get('name') or vals['name'] == _('New'):
            vals['name'] = self.env['ir.sequence'].next_by_code('mrp.unbuild') or _('New')
        return super(MrpUnbuild, self).create(vals)

    def unlink(self):
        if 'done' in self.mapped('state'):
            raise UserError(_("You cannot delete an unbuild order if the state is 'Done'."))
        return super(MrpUnbuild, self).unlink()

    def action_unbuild(self):
        self.ensure_one()
        self._check_company()
        if self.product_id.tracking != 'none' and not self.lot_id.id:
            raise UserError(_('You should provide a lot number for the final product.'))

        if self.mo_id:
            if self.mo_id.state != 'done':
                raise UserError(_('You cannot unbuild a undone manufacturing order.'))

        consume_moves = self._generate_consume_moves()
        consume_moves._action_confirm()
        produce_moves = self._generate_produce_moves()
        produce_moves._action_confirm()

        finished_moves = consume_moves.filtered(lambda m: m.product_id == self.product_id)
        consume_moves -= finished_moves

        if any(produce_move.has_tracking != 'none' and not self.mo_id for produce_move in produce_moves):
            raise UserError(_('Some of your components are tracked, you have to specify a manufacturing order in order to retrieve the correct components.'))

        if any(consume_move.has_tracking != 'none' and not self.mo_id for consume_move in consume_moves):
            raise UserError(_('Some of your byproducts are tracked, you have to specify a manufacturing order in order to retrieve the correct byproducts.'))

        for finished_move in finished_moves:
            if finished_move.has_tracking != 'none':
                self.env['stock.move.line'].create({
                    'move_id': finished_move.id,
                    'lot_id': self.lot_id.id,
                    'qty_done': finished_move.product_uom_qty,
                    'product_id': finished_move.product_id.id,
                    'product_uom_id': finished_move.product_uom.id,
                    'location_id': finished_move.location_id.id,
                    'location_dest_id': finished_move.location_dest_id.id,
                })
            else:
                finished_move.quantity_done = finished_move.product_uom_qty

        # TODO: Will fail if user do more than one unbuild with lot on the same MO. Need to check what other unbuild has aready took
        for move in produce_moves | consume_moves:
            if move.has_tracking != 'none':
                original_move = move in produce_moves and self.mo_id.move_raw_ids or self.mo_id.move_finished_ids
                original_move = original_move.filtered(lambda m: m.product_id == move.product_id)
                needed_quantity = move.product_uom_qty
                moves_lines = original_move.mapped('move_line_ids')
                if move in produce_moves and self.lot_id:
                    moves_lines = moves_lines.filtered(lambda ml: self.lot_id in ml.lot_produced_ids)
                for move_line in moves_lines:
                    # Iterate over all move_lines until we unbuilded the correct quantity.
                    taken_quantity = min(needed_quantity, move_line.qty_done)
                    if taken_quantity:
                        self.env['stock.move.line'].create({
                            'move_id': move.id,
                            'lot_id': move_line.lot_id.id,
                            'qty_done': taken_quantity,
                            'product_id': move.product_id.id,
                            'product_uom_id': move_line.product_uom_id.id,
                            'location_id': move.location_id.id,
                            'location_dest_id': move.location_dest_id.id,
                        })
                        needed_quantity -= taken_quantity
            else:
                move.quantity_done = float_round(move.product_uom_qty, precision_rounding=move.product_uom.rounding)

        finished_moves._action_done()
        consume_moves._action_done()
        produce_moves._action_done()
        produced_move_line_ids = produce_moves.mapped('move_line_ids').filtered(lambda ml: ml.qty_done > 0)
        consume_moves.mapped('move_line_ids').write({'produce_line_ids': [(6, 0, produced_move_line_ids.ids)]})

        return self.write({'state': 'done'})

    def _generate_consume_moves(self):
        moves = self.env['stock.move']
        for unbuild in self:
            if unbuild.mo_id:
                finished_moves = unbuild.mo_id.move_finished_ids.filtered(lambda move: move.state == 'done')
                qty_produced = 0
                for line in finished_moves.move_line_ids:
                    if line.product_id != unbuild.mo_id.product_id:
                        continue
                    qty_produced += line.product_uom_id._compute_quantity(line.qty_done, unbuild.product_uom_id)
                if float_is_zero(qty_produced, precision_rounding=unbuild.product_uom_id.rounding):
                    return moves
                factor = unbuild.product_qty / qty_produced
                for product in finished_moves.product_id:
                    product_moves = finished_moves.filtered(lambda m: m.product_id == product)
                    qty = sum(product_moves.mapped('product_uom_qty'))
                    moves += unbuild._generate_move_from_existing_move_with_qty(product_moves[0], factor * qty, self.location_id, product_moves[0].location_id)
            else:
                factor = unbuild.product_uom_id._compute_quantity(unbuild.product_qty, unbuild.bom_id.product_uom_id) / unbuild.bom_id.product_qty
                moves += unbuild._generate_move_from_bom_line(self.product_id, self.product_uom_id, unbuild.product_qty)
                for byproduct in unbuild.bom_id.byproduct_ids:
                    quantity = byproduct.product_qty * factor
                    moves += unbuild._generate_move_from_bom_line(byproduct.product_id, byproduct.product_uom_id, quantity, byproduct_id=byproduct.id)
        return moves

    def _generate_produce_moves(self):
        moves = self.env['stock.move']
        for unbuild in self:
            if unbuild.mo_id:
                raw_moves = unbuild.mo_id.move_raw_ids.filtered(lambda move: move.state == 'done')
                qty_produced = 0
                for line in unbuild.mo_id.move_finished_ids.move_line_ids:
                    if line.product_id != unbuild.mo_id.product_id or line.state != 'done':
                        continue
                    qty_produced += line.product_uom_id._compute_quantity(line.qty_done, unbuild.product_uom_id)
                if float_is_zero(qty_produced, precision_rounding=unbuild.product_uom_id.rounding):
                    return moves
                factor = unbuild.product_qty / qty_produced
                for product in raw_moves.product_id:
                    product_moves = raw_moves.filtered(lambda m: m.product_id == product)
                    qty = sum(product_moves.mapped('product_uom_qty'))
                    moves += unbuild._generate_move_from_existing_move_with_qty(product_moves[0], factor * qty, product_moves[0].location_dest_id, self.location_dest_id)
            else:
                factor = unbuild.product_uom_id._compute_quantity(unbuild.product_qty, unbuild.bom_id.product_uom_id) / unbuild.bom_id.product_qty
                boms, lines = unbuild.bom_id.explode(unbuild.product_id, factor, picking_type=unbuild.bom_id.picking_type_id)
                for line, line_data in lines:
                    moves += unbuild._generate_move_from_bom_line(line.product_id, line.product_uom_id, line_data['qty'], bom_line_id=line.id)
        return moves

    def _generate_move_from_existing_move(self, move, factor, location_id, location_dest_id):
        return self._generate_move_from_existing_move_with_qty(move, move.product_uom_qty * factor, location_id, location_dest_id)

    def _generate_move_from_existing_move_with_qty(self, move, qty, location_id, location_dest_id):
        return self.env['stock.move'].create({
            'name': self.name,
            'date': self.create_date,
            'product_id': move.product_id.id,
            'product_uom_qty': qty,
            'product_uom': move.product_uom.id,
            'procure_method': 'make_to_stock',
            'location_dest_id': location_dest_id.id,
            'location_id': location_id.id,
            'warehouse_id': location_dest_id.get_warehouse().id,
            'unbuild_id': self.id,
            'company_id': move.company_id.id,
        })

    def _generate_move_from_bom_line(self, product, product_uom, quantity, bom_line_id=False, byproduct_id=False):
        location_id = bom_line_id and product.property_stock_production or self.location_id
        location_dest_id = bom_line_id and self.location_dest_id or product.with_context(force_company=self.company_id.id).property_stock_production
        warehouse = location_dest_id.get_warehouse()
        return self.env['stock.move'].create({
            'name': self.name,
            'date': self.create_date,
            'bom_line_id': bom_line_id,
            'byproduct_id': byproduct_id,
            'product_id': product.id,
            'product_uom_qty': quantity,
            'product_uom': product_uom.id,
            'procure_method': 'make_to_stock',
            'location_dest_id': location_dest_id.id,
            'location_id': location_id.id,
            'warehouse_id': warehouse.id,
            'unbuild_id': self.id,
            'company_id': self.company_id.id,
        })

    def action_validate(self):
        self.ensure_one()
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        available_qty = self.env['stock.quant']._get_available_quantity(self.product_id, self.location_id, self.lot_id, strict=True)
        if float_compare(available_qty, self.product_qty, precision_digits=precision) >= 0:
            return self.action_unbuild()
        else:
            return {
                'name': _('Insufficient Quantity'),
                'view_mode': 'form',
                'res_model': 'stock.warn.insufficient.qty.unbuild',
                'view_id': self.env.ref('mrp.stock_warn_insufficient_qty_unbuild_form_view').id,
                'type': 'ir.actions.act_window',
                'context': {
                    'default_product_id': self.product_id.id,
                    'default_location_id': self.location_id.id,
                    'default_unbuild_id': self.id
                },
                'target': 'new'
            }


```

## File: models\mrp_workcenter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil import relativedelta
import datetime

from odoo import api, exceptions, fields, models, _
from odoo.exceptions import ValidationError


class MrpWorkcenter(models.Model):
    _name = 'mrp.workcenter'
    _description = 'Work Center'
    _order = "sequence, id"
    _inherit = ['resource.mixin']
    _check_company_auto = True

    # resource
    name = fields.Char('Work Center', related='resource_id.name', store=True, readonly=False)
    time_efficiency = fields.Float('Time Efficiency', related='resource_id.time_efficiency', default=100, store=True, readonly=False)
    active = fields.Boolean('Active', related='resource_id.active', default=True, store=True, readonly=False)

    code = fields.Char('Code', copy=False)
    note = fields.Text(
        'Description',
        help="Description of the Work Center.")
    capacity = fields.Float(
        'Capacity', default=1.0,
        help="Number of pieces that can be produced in parallel. In case the work center has a capacity of 5 and you have to produce 10 units on your work order, the usual operation time will be multiplied by 2.")
    sequence = fields.Integer(
        'Sequence', default=1, required=True,
        help="Gives the sequence order when displaying a list of work centers.")
    color = fields.Integer('Color')
    costs_hour = fields.Float(string='Cost per hour', help='Specify cost of work center per hour.', default=0.0)
    time_start = fields.Float('Time before prod.', help="Time in minutes for the setup.")
    time_stop = fields.Float('Time after prod.', help="Time in minutes for the cleaning.")
    routing_line_ids = fields.One2many('mrp.routing.workcenter', 'workcenter_id', "Routing Lines")
    order_ids = fields.One2many('mrp.workorder', 'workcenter_id', "Orders")
    workorder_count = fields.Integer('# Work Orders', compute='_compute_workorder_count')
    workorder_ready_count = fields.Integer('# Read Work Orders', compute='_compute_workorder_count')
    workorder_progress_count = fields.Integer('Total Running Orders', compute='_compute_workorder_count')
    workorder_pending_count = fields.Integer('Total Pending Orders', compute='_compute_workorder_count')
    workorder_late_count = fields.Integer('Total Late Orders', compute='_compute_workorder_count')

    time_ids = fields.One2many('mrp.workcenter.productivity', 'workcenter_id', 'Time Logs')
    working_state = fields.Selection([
        ('normal', 'Normal'),
        ('blocked', 'Blocked'),
        ('done', 'In Progress')], 'Workcenter Status', compute="_compute_working_state", store=True)
    blocked_time = fields.Float(
        'Blocked Time', compute='_compute_blocked_time',
        help='Blocked hours over the last month', digits=(16, 2))
    productive_time = fields.Float(
        'Productive Time', compute='_compute_productive_time',
        help='Productive hours over the last month', digits=(16, 2))
    oee = fields.Float(compute='_compute_oee', help='Overall Equipment Effectiveness, based on the last month')
    oee_target = fields.Float(string='OEE Target', help="Overall Effective Efficiency Target in percentage", default=90)
    performance = fields.Integer('Performance', compute='_compute_performance', help='Performance over the last month')
    workcenter_load = fields.Float('Work Center Load', compute='_compute_workorder_count')
    alternative_workcenter_ids = fields.Many2many(
        'mrp.workcenter',
        'mrp_workcenter_alternative_rel',
        'workcenter_id',
        'alternative_workcenter_id',
        domain="[('id', '!=', id), '|', ('company_id', '=', company_id), ('company_id', '=', False)]",
        string="Alternative Workcenters", check_company=True,
        help="Alternative workcenters that can be substituted to this one in order to dispatch production"
    )

    @api.constrains('alternative_workcenter_ids')
    def _check_alternative_workcenter(self):
        if self in self.alternative_workcenter_ids:
            raise ValidationError(_("A workcenter cannot be an alternative of itself"))

    @api.depends('order_ids.duration_expected', 'order_ids.workcenter_id', 'order_ids.state', 'order_ids.date_planned_start')
    def _compute_workorder_count(self):
        MrpWorkorder = self.env['mrp.workorder']
        result = {wid: {} for wid in self._ids}
        result_duration_expected = {wid: 0 for wid in self._ids}
        #Count Late Workorder
        data = MrpWorkorder.read_group([('workcenter_id', 'in', self.ids), ('state', 'in', ('pending', 'ready')), ('date_planned_start', '<', datetime.datetime.now().strftime('%Y-%m-%d'))], ['workcenter_id'], ['workcenter_id'])
        count_data = dict((item['workcenter_id'][0], item['workcenter_id_count']) for item in data)
        #Count All, Pending, Ready, Progress Workorder
        res = MrpWorkorder.read_group(
            [('workcenter_id', 'in', self.ids)],
            ['workcenter_id', 'state', 'duration_expected'], ['workcenter_id', 'state'],
            lazy=False)
        for res_group in res:
            result[res_group['workcenter_id'][0]][res_group['state']] = res_group['__count']
            if res_group['state'] in ('pending', 'ready', 'progress'):
                result_duration_expected[res_group['workcenter_id'][0]] += res_group['duration_expected']
        for workcenter in self:
            workcenter.workorder_count = sum(count for state, count in result[workcenter.id].items() if state not in ('done', 'cancel'))
            workcenter.workorder_pending_count = result[workcenter.id].get('pending', 0)
            workcenter.workcenter_load = result_duration_expected[workcenter.id]
            workcenter.workorder_ready_count = result[workcenter.id].get('ready', 0)
            workcenter.workorder_progress_count = result[workcenter.id].get('progress', 0)
            workcenter.workorder_late_count = count_data.get(workcenter.id, 0)

    @api.depends('time_ids', 'time_ids.date_end', 'time_ids.loss_type')
    def _compute_working_state(self):
        for workcenter in self:
            # We search for a productivity line associated to this workcenter having no `date_end`.
            # If we do not find one, the workcenter is not currently being used. If we find one, according
            # to its `type_loss`, the workcenter is either being used or blocked.
            time_log = self.env['mrp.workcenter.productivity'].search([
                ('workcenter_id', '=', workcenter.id),
                ('date_end', '=', False)
            ], limit=1)
            if not time_log:
                # the workcenter is not being used
                workcenter.working_state = 'normal'
            elif time_log.loss_type in ('productive', 'performance'):
                # the productivity line has a `loss_type` that means the workcenter is being used
                workcenter.working_state = 'done'
            else:
                # the workcenter is blocked
                workcenter.working_state = 'blocked'

    def _compute_blocked_time(self):
        # TDE FIXME: productivity loss type should be only losses, probably count other time logs differently ??
        data = self.env['mrp.workcenter.productivity'].read_group([
            ('date_start', '>=', fields.Datetime.to_string(datetime.datetime.now() - relativedelta.relativedelta(months=1))),
            ('workcenter_id', 'in', self.ids),
            ('date_end', '!=', False),
            ('loss_type', '!=', 'productive')],
            ['duration', 'workcenter_id'], ['workcenter_id'], lazy=False)
        count_data = dict((item['workcenter_id'][0], item['duration']) for item in data)
        for workcenter in self:
            workcenter.blocked_time = count_data.get(workcenter.id, 0.0) / 60.0

    def _compute_productive_time(self):
        # TDE FIXME: productivity loss type should be only losses, probably count other time logs differently
        data = self.env['mrp.workcenter.productivity'].read_group([
            ('date_start', '>=', fields.Datetime.to_string(datetime.datetime.now() - relativedelta.relativedelta(months=1))),
            ('workcenter_id', 'in', self.ids),
            ('date_end', '!=', False),
            ('loss_type', '=', 'productive')],
            ['duration', 'workcenter_id'], ['workcenter_id'], lazy=False)
        count_data = dict((item['workcenter_id'][0], item['duration']) for item in data)
        for workcenter in self:
            workcenter.productive_time = count_data.get(workcenter.id, 0.0) / 60.0

    @api.depends('blocked_time', 'productive_time')
    def _compute_oee(self):
        for order in self:
            if order.productive_time:
                order.oee = round(order.productive_time * 100.0 / (order.productive_time + order.blocked_time), 2)
            else:
                order.oee = 0.0

    def _compute_performance(self):
        wo_data = self.env['mrp.workorder'].read_group([
            ('date_start', '>=', fields.Datetime.to_string(datetime.datetime.now() - relativedelta.relativedelta(months=1))),
            ('workcenter_id', 'in', self.ids),
            ('state', '=', 'done')], ['duration_expected', 'workcenter_id', 'duration'], ['workcenter_id'], lazy=False)
        duration_expected = dict((data['workcenter_id'][0], data['duration_expected']) for data in wo_data)
        duration = dict((data['workcenter_id'][0], data['duration']) for data in wo_data)
        for workcenter in self:
            if duration.get(workcenter.id):
                workcenter.performance = 100 * duration_expected.get(workcenter.id, 0.0) / duration[workcenter.id]
            else:
                workcenter.performance = 0.0

    @api.constrains('capacity')
    def _check_capacity(self):
        if any(workcenter.capacity <= 0.0 for workcenter in self):
            raise exceptions.UserError(_('The capacity must be strictly positive.'))

    def unblock(self):
        self.ensure_one()
        if self.working_state != 'blocked':
            raise exceptions.UserError(_("It has already been unblocked."))
        times = self.env['mrp.workcenter.productivity'].search([('workcenter_id', '=', self.id), ('date_end', '=', False)])
        times.write({'date_end': fields.Datetime.now()})
        return {'type': 'ir.actions.client', 'tag': 'reload'}

    @api.model_create_multi
    def create(self, vals_list):
        # resource_type is 'human' by default. As we are not living in
        # /r/latestagecapitalism, workcenters are 'material'
        records = super(MrpWorkcenter, self.with_context(default_resource_type='material')).create(vals_list)
        return records

    def write(self, vals):
        if 'company_id' in vals:
            self.resource_id.company_id = vals['company_id']
        return super(MrpWorkcenter, self).write(vals)

    def action_work_order(self):
        action = self.env.ref('mrp.action_work_orders').read()[0]
        return action


class MrpWorkcenterProductivityLossType(models.Model):
    _name = "mrp.workcenter.productivity.loss.type"
    _description = 'MRP Workorder productivity losses'
    _rec_name = 'loss_type'

    @api.depends('loss_type')
    def name_get(self):
        """ As 'category' field in form view is a Many2one, its value will be in
        lower case. In order to display its value capitalized 'name_get' is
        overrided.
        """
        result = []
        for rec in self:
            result.append((rec.id, rec.loss_type.title()))
        return result

    loss_type = fields.Selection([
            ('availability', 'Availability'),
            ('performance', 'Performance'),
            ('quality', 'Quality'),
            ('productive', 'Productive')], string='Category', default='availability', required=True)


class MrpWorkcenterProductivityLoss(models.Model):
    _name = "mrp.workcenter.productivity.loss"
    _description = "Workcenter Productivity Losses"
    _order = "sequence, id"

    name = fields.Char('Blocking Reason', required=True)
    sequence = fields.Integer('Sequence', default=1)
    manual = fields.Boolean('Is a Blocking Reason', default=True)
    loss_id = fields.Many2one('mrp.workcenter.productivity.loss.type', domain=([('loss_type', 'in', ['quality', 'availability'])]), string='Category')
    loss_type = fields.Selection(string='Effectiveness Category', related='loss_id.loss_type', store=True, readonly=False)


class MrpWorkcenterProductivity(models.Model):
    _name = "mrp.workcenter.productivity"
    _description = "Workcenter Productivity Log"
    _order = "id desc"
    _rec_name = "loss_id"
    _check_company_auto = True

    def _get_default_company_id(self):
        company_id = False
        if self.env.context.get('default_company_id'):
            company_id = self.env.context['default_company_id']
        if not company_id and self.env.context.get('default_workorder_id'):
            workorder = self.env['mrp.workorder'].browse(self.env.context['default_workorder_id'])
            company_id = workorder.company_id
        if not company_id and self.env.context.get('default_workcenter_id'):
            workcenter = self.env['mrp.workcenter'].browse(self.env.context['default_workcenter_id'])
            company_id = workcenter.company_id
        if not company_id:
            company_id = self.env.company
        return company_id

    production_id = fields.Many2one('mrp.production', string='Manufacturing Order', related='workorder_id.production_id', readonly='True')
    workcenter_id = fields.Many2one('mrp.workcenter', "Work Center", required=True, check_company=True)
    company_id = fields.Many2one(
        'res.company', required=True, index=True,
        default=lambda self: self._get_default_company_id())
    workorder_id = fields.Many2one('mrp.workorder', 'Work Order', check_company=True)
    user_id = fields.Many2one(
        'res.users', "User",
        default=lambda self: self.env.uid)
    loss_id = fields.Many2one(
        'mrp.workcenter.productivity.loss', "Loss Reason",
        ondelete='restrict', required=True)
    loss_type = fields.Selection(
        "Effectiveness", related='loss_id.loss_type', store=True, readonly=False)
    description = fields.Text('Description')
    date_start = fields.Datetime('Start Date', default=fields.Datetime.now, required=True)
    date_end = fields.Datetime('End Date')
    duration = fields.Float('Duration', compute='_compute_duration', store=True)

    @api.depends('date_end', 'date_start')
    def _compute_duration(self):
        for blocktime in self:
            if blocktime.date_start and blocktime.date_end:
                d1 = fields.Datetime.from_string(blocktime.date_start)
                d2 = fields.Datetime.from_string(blocktime.date_end)
                diff = d2 - d1
                if (blocktime.loss_type not in ('productive', 'performance')) and blocktime.workcenter_id.resource_calendar_id:
                    r = blocktime.workcenter_id._get_work_days_data_batch(d1, d2)[blocktime.workcenter_id.id]['hours']
                    blocktime.duration = round(r * 60, 2)
                else:
                    blocktime.duration = round(diff.total_seconds() / 60.0, 2)
            else:
                blocktime.duration = 0.0

    def button_block(self):
        self.ensure_one()
        self.workcenter_id.order_ids.end_all()


```

## File: models\mrp_workorder.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta
from collections import defaultdict
from math import floor

from odoo import api, fields, models, _, SUPERUSER_ID
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_round


class MrpWorkorder(models.Model):
    _name = 'mrp.workorder'
    _description = 'Work Order'
    _inherit = ['mail.thread', 'mail.activity.mixin', 'mrp.abstract.workorder']

    def _read_group_workcenter_id(self, workcenters, domain, order):
        workcenter_ids = self.env.context.get('default_workcenter_id')
        if not workcenter_ids:
            workcenter_ids = workcenters._search([], order=order, access_rights_uid=SUPERUSER_ID)
        return workcenters.browse(workcenter_ids)

    name = fields.Char(
        'Work Order', required=True,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]})
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda self: self.env.company,
        required=True, index=True, readonly=True)
    workcenter_id = fields.Many2one(
        'mrp.workcenter', 'Work Center', required=True,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        group_expand='_read_group_workcenter_id', check_company=True)
    working_state = fields.Selection(
        'Workcenter Status', related='workcenter_id.working_state', readonly=False,
        help='Technical: used in views only')
    production_availability = fields.Selection(
        'Stock Availability', readonly=True,
        related='production_id.reservation_state', store=True,
        help='Technical: used in views and domains only.')
    production_state = fields.Selection(
        'Production State', readonly=True,
        related='production_id.state',
        help='Technical: used in views only.')
    qty_production = fields.Float('Original Production Quantity', readonly=True, related='production_id.product_qty')
    qty_remaining = fields.Float('Quantity To Be Produced', compute='_compute_qty_remaining', digits='Product Unit of Measure')
    qty_produced = fields.Float(
        'Quantity', default=0.0,
        readonly=True,
        digits='Product Unit of Measure',
        help="The number of products already handled by this work order")
    is_produced = fields.Boolean(string="Has Been Produced",
        compute='_compute_is_produced')
    state = fields.Selection([
        ('pending', 'Waiting for another WO'),
        ('ready', 'Ready'),
        ('progress', 'In Progress'),
        ('done', 'Finished'),
        ('cancel', 'Cancelled')], string='Status',
        default='pending')
    leave_id = fields.Many2one(
        'resource.calendar.leaves',
        help='Slot into workcenter calendar once planned',
        check_company=True)
    date_planned_start = fields.Datetime(
        'Scheduled Date Start',
        compute='_compute_dates_planned',
        inverse='_set_dates_planned',
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        store=True,
        tracking=True)
    date_planned_finished = fields.Datetime(
        'Scheduled Date Finished',
        compute='_compute_dates_planned',
        inverse='_set_dates_planned',
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        store=True,
        tracking=True)
    date_start = fields.Datetime(
        'Effective Start Date',
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]})
    date_finished = fields.Datetime(
        'Effective End Date',
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]})

    duration_expected = fields.Float(
        'Expected Duration', digits=(16, 2),
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help="Expected duration (in minutes)")
    duration = fields.Float(
        'Real Duration', compute='_compute_duration',
        readonly=True, store=True)
    duration_unit = fields.Float(
        'Duration Per Unit', compute='_compute_duration',
        group_operator="avg", readonly=True, store=True)
    duration_percent = fields.Integer(
        'Duration Deviation (%)', compute='_compute_duration',
        group_operator="avg", readonly=True, store=True)
    progress = fields.Float('Progress Done (%)', digits=(16, 2), compute='_compute_progress')

    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Operation',
        check_company=True)
        # Should be used differently as BoM can change in the meantime
    worksheet = fields.Binary(
        'Worksheet', related='operation_id.worksheet', readonly=True)
    worksheet_type = fields.Selection(
        'Worksheet Type', related='operation_id.worksheet_type', readonly=True)
    worksheet_google_slide = fields.Char(
        'Worksheet URL', related='operation_id.worksheet_google_slide', readonly=True)
    move_raw_ids = fields.One2many(
        'stock.move', 'workorder_id', 'Raw Moves',
        domain=[('raw_material_production_id', '!=', False), ('production_id', '=', False)])
    move_finished_ids = fields.One2many(
        'stock.move', 'workorder_id', 'Finished Moves',
        domain=[('raw_material_production_id', '=', False), ('production_id', '!=', False)])
    move_line_ids = fields.One2many(
        'stock.move.line', 'workorder_id', 'Moves to Track',
        help="Inventory moves for which you must scan a lot number at this work order")
    finished_lot_id = fields.Many2one(
        'stock.production.lot', 'Lot/Serial Number', domain="[('id', 'in', allowed_lots_domain)]",
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]}, check_company=True)
    time_ids = fields.One2many(
        'mrp.workcenter.productivity', 'workorder_id')
    is_user_working = fields.Boolean(
        'Is the Current User Working', compute='_compute_working_users',
        help="Technical field indicating whether the current user is working. ")
    working_user_ids = fields.One2many('res.users', string='Working user on this work order.', compute='_compute_working_users')
    last_working_user_id = fields.One2many('res.users', string='Last user that worked on this work order.', compute='_compute_working_users')

    next_work_order_id = fields.Many2one('mrp.workorder', "Next Work Order", check_company=True)
    scrap_ids = fields.One2many('stock.scrap', 'workorder_id')
    scrap_count = fields.Integer(compute='_compute_scrap_move_count', string='Scrap Move')
    production_date = fields.Datetime('Production Date', related='production_id.date_planned_start', store=True, readonly=False)
    color = fields.Integer('Color', compute='_compute_color')
    capacity = fields.Float(
        'Capacity', default=1.0,
        help="Number of pieces that can be produced in parallel.")
    raw_workorder_line_ids = fields.One2many('mrp.workorder.line',
        'raw_workorder_id', string='Components')
    finished_workorder_line_ids = fields.One2many('mrp.workorder.line',
        'finished_workorder_id', string='By-products')
    allowed_lots_domain = fields.One2many(comodel_name='stock.production.lot', compute="_compute_allowed_lots_domain")

    # Both `date_planned_start` and `date_planned_finished` are related fields on `leave_id`. Let's say
    # we slide a workorder on a gantt view, a single call to write is made with both
    # fields Changes. As the ORM doesn't batch the write on related fields and instead
    # makes multiple call, the constraint check_dates() is raised.
    # That's why the compute and set methods are needed. to ensure the dates are updated
    # in the same time.
    @api.depends('leave_id')
    def _compute_dates_planned(self):
        for workorder in self:
            workorder.date_planned_start = workorder.leave_id.date_from
            workorder.date_planned_finished = workorder.leave_id.date_to

    def _set_dates_planned(self):
        if not self[0].date_planned_start or not self[0].date_planned_finished:
            raise UserError(_("It is not possible to unplan one single Work Order. "
                              "You should unplan the Manufacturing Order instead in order to unplan all the linked operations."))
        date_from = self[0].date_planned_start
        date_to = self[0].date_planned_finished
        self.mapped('leave_id').sudo().write({
            'date_from': date_from,
            'date_to': date_to,
        })

    @api.onchange('finished_lot_id')
    def _onchange_finished_lot_id(self):
        """When the user changes the lot being currently produced, suggest
        a quantity to produce consistent with the previous workorders. """
        previous_wo = self.env['mrp.workorder'].search([
            ('next_work_order_id', '=', self.id)
        ])
        if previous_wo:
            line = previous_wo.finished_workorder_line_ids.filtered(lambda line: line.product_id == self.product_id and line.lot_id == self.finished_lot_id)
            if line:
                self.qty_producing = line.qty_done

    @api.depends('production_id.workorder_ids.finished_workorder_line_ids',
    'production_id.workorder_ids.finished_workorder_line_ids.qty_done',
    'production_id.workorder_ids.finished_workorder_line_ids.lot_id')
    def _compute_allowed_lots_domain(self):
        """ Check if all the finished products has been assigned to a serial
        number or a lot in other workorders. If yes, restrict the selectable lot
        to the lot/sn used in other workorders.
        """
        productions = self.mapped('production_id')
        treated = self.browse()
        for production in productions:
            if production.product_id.tracking == 'none':
                continue

            rounding = production.product_uom_id.rounding
            finished_workorder_lines = production.workorder_ids.mapped('finished_workorder_line_ids').filtered(lambda wl: wl.product_id == production.product_id)
            qties_done_per_lot = defaultdict(list)
            for finished_workorder_line in finished_workorder_lines:
                # It is possible to have finished workorder lines without a lot (eg using the dummy
                # test type). Ignore them when computing the allowed lots.
                if finished_workorder_line.lot_id:
                    qties_done_per_lot[finished_workorder_line.lot_id.id].append(finished_workorder_line.qty_done)

            qty_to_produce = production.product_qty
            if production.product_id.tracking == 'serial':
                qty_to_produce = production.product_uom_id._compute_quantity(
                    production.product_qty,
                    production.product_id.uom_id,
                    round=False
                )
            allowed_lot_ids = self.env['stock.production.lot']
            qty_produced = sum([max(qty_dones) for qty_dones in qties_done_per_lot.values()])
            if float_compare(qty_produced, qty_to_produce, precision_rounding=rounding) < 0:
                # If we haven't produced enough, all lots are available
                allowed_lot_ids = self.env['stock.production.lot'].search([
                    ('product_id', '=', production.product_id.id),
                    ('company_id', '=', production.company_id.id),
                ])
            else:
                # If we produced enough, only the already produced lots are available
                allowed_lot_ids = self.env['stock.production.lot'].browse(qties_done_per_lot.keys())
            workorders = production.workorder_ids.filtered(lambda wo: wo.state not in ('done', 'cancel'))
            for workorder in workorders:
                if workorder.product_tracking == 'serial':
                    workorder.allowed_lots_domain = allowed_lot_ids - workorder.finished_workorder_line_ids.filtered(lambda wl: wl.product_id == production.product_id).mapped('lot_id')
                else:
                    workorder.allowed_lots_domain = allowed_lot_ids
                treated |= workorder
        (self - treated).allowed_lots_domain = False

    def name_get(self):
        return [(wo.id, "%s - %s - %s" % (wo.production_id.name, wo.product_id.name, wo.name)) for wo in self]

    def unlink(self):
        # Removes references to workorder to avoid Validation Error
        (self.mapped('move_raw_ids') | self.mapped('move_finished_ids')).write({'workorder_id': False})
        self.mapped('leave_id').unlink()
        previous_wos = self.env['mrp.workorder'].search([
            ('next_work_order_id', 'in', self.ids),
            ('id', 'not in', self.ids)
        ])
        for pw in previous_wos:
            while pw.next_work_order_id and pw.next_work_order_id in self:
                pw.next_work_order_id = pw.next_work_order_id.next_work_order_id
        return super(MrpWorkorder, self).unlink()

    def _get_real_uom_qty(self, qty, to_production_uom=False):
        if self.product_id.tracking == 'serial' and self.production_id.product_uom_id.uom_type != 'reference':
            if to_production_uom:
                uom_from = self.product_uom_id
                uom_to = self.production_id.product_uom_id
            else:
                uom_from = self.production_id.product_uom_id
                uom_to = self.product_uom_id
            return uom_from._compute_quantity(qty, uom_to, round=False)
        return qty

    @api.depends('production_id.product_qty', 'qty_produced')
    def _compute_is_produced(self):
        self.is_produced = False
        for order in self.filtered(lambda p: p.production_id):
            rounding = order.production_id.product_uom_id.rounding
            production_qty = order._get_real_uom_qty(order.production_id.product_qty)
            order.is_produced = float_compare(order.qty_produced, production_qty, precision_rounding=rounding) >= 0

    @api.depends('time_ids.duration', 'qty_produced')
    def _compute_duration(self):
        for order in self:
            order.duration = sum(order.time_ids.mapped('duration'))
            order.duration_unit = round(order.duration / max(order.qty_produced, 1), 2)  # rounding 2 because it is a time
            if order.duration_expected:
                order.duration_percent = 100 * (order.duration_expected - order.duration) / order.duration_expected
            else:
                order.duration_percent = 0

    @api.depends('duration', 'duration_expected', 'state')
    def _compute_progress(self):
        for order in self:
            if order.state == 'done':
                order.progress = 100
            elif order.duration_expected:
                order.progress = order.duration * 100 / order.duration_expected
            else:
                order.progress = 0

    def _compute_working_users(self):
        """ Checks whether the current user is working, all the users currently working and the last user that worked. """
        for order in self:
            order.working_user_ids = [(4, order.id) for order in order.time_ids.filtered(lambda time: not time.date_end).sorted('date_start').mapped('user_id')]
            if order.working_user_ids:
                order.last_working_user_id = order.working_user_ids[-1]
            elif order.time_ids:
                order.last_working_user_id = order.time_ids.filtered('date_end').sorted('date_end')[-1].user_id if order.time_ids.filtered('date_end') else order.time_ids[-1].user_id
            else:
                order.last_working_user_id = False
            if order.time_ids.filtered(lambda x: (x.user_id.id == self.env.user.id) and (not x.date_end) and (x.loss_type in ('productive', 'performance'))):
                order.is_user_working = True
            else:
                order.is_user_working = False

    def _compute_scrap_move_count(self):
        data = self.env['stock.scrap'].read_group([('workorder_id', 'in', self.ids)], ['workorder_id'], ['workorder_id'])
        count_data = dict((item['workorder_id'][0], item['workorder_id_count']) for item in data)
        for workorder in self:
            workorder.scrap_count = count_data.get(workorder.id, 0)

    @api.depends('date_planned_finished', 'production_id.date_planned_finished')
    def _compute_color(self):
        late_orders = self.filtered(lambda x: x.production_id.date_planned_finished
                                              and x.date_planned_finished
                                              and x.date_planned_finished > x.production_id.date_planned_finished)
        for order in late_orders:
            order.color = 4
        for order in (self - late_orders):
            order.color = 2

    @api.onchange('date_planned_start', 'duration_expected', 'workcenter_id')
    def _onchange_date_planned_start(self):
        if self.date_planned_start and self.duration_expected and self.workcenter_id:
            self.date_planned_finished = self._calculate_date_planned_finished()

    def _calculate_date_planned_finished(self, date_planned_start=False):
        return self.workcenter_id.resource_calendar_id.plan_hours(
            self.duration_expected / 60.0, date_planned_start or self.date_planned_start,
            compute_leaves=True, domain=[('time_type', 'in', ['leave', 'other'])]
        )

    @api.onchange('date_planned_finished')
    def _onchange_date_planned_finished(self):
        if self.date_planned_start and self.date_planned_finished:
            self.duration_expected = self._calculate_duration_expected()

    def _calculate_duration_expected(self, date_planned_start=False, date_planned_finished=False):
        interval = self.workcenter_id.resource_calendar_id.get_work_duration_data(
            date_planned_start or self.date_planned_start, date_planned_finished or self.date_planned_finished,
            domain=[('time_type', 'in', ['leave', 'other'])]
        )
        return interval['hours'] * 60

    def write(self, values):
        if 'production_id' in values:
            raise UserError(_('You cannot link this work order to another manufacturing order.'))
        if 'workcenter_id' in values:
            for workorder in self:
                if workorder.workcenter_id.id != values['workcenter_id']:
                    if workorder.state in ('progress', 'done', 'cancel'):
                        raise UserError(_('You cannot change the workcenter of a work order that is in progress or done.'))
                    workorder.leave_id.resource_id = self.env['mrp.workcenter'].browse(values['workcenter_id']).resource_id
        if list(values.keys()) != ['time_ids'] and any(workorder.state == 'done' for workorder in self):
            raise UserError(_('You can not change the finished work order.'))
        if 'date_planned_start' in values or 'date_planned_finished' in values:
            for workorder in self:
                start_date = fields.Datetime.to_datetime(values.get('date_planned_start')) or workorder.date_planned_start
                end_date = fields.Datetime.to_datetime(values.get('date_planned_finished')) or workorder.date_planned_finished
                if start_date and end_date and start_date > end_date:
                    raise UserError(_('The planned end date of the work order cannot be prior to the planned start date, please correct this to save the work order.'))
                if 'duration_expected' not in values:
                    if 'date_planned_start' in values and 'date_planned_finished' in values:
                        computed_finished_time = workorder._calculate_date_planned_finished(start_date)
                        values['date_planned_finished'] = computed_finished_time
                    elif 'date_planned_start' in values:
                        computed_duration = workorder._calculate_duration_expected(date_planned_start=start_date)
                        values['duration_expected'] = computed_duration
                    elif 'date_planned_finished' in values:
                        computed_duration = workorder._calculate_duration_expected(date_planned_finished=end_date)
                        values['duration_expected'] = computed_duration
                # Update MO dates if the start date of the first WO or the
                # finished date of the last WO is update.
                if workorder == workorder.production_id.workorder_ids[0] and 'date_planned_start' in values:
                    workorder.production_id.with_context(force_date=True).write({
                        'date_planned_start': fields.Datetime.to_datetime(values['date_planned_start'])
                    })
                if workorder == workorder.production_id.workorder_ids[-1] and 'date_planned_finished' in values:
                    workorder.production_id.with_context(force_date=True).write({
                        'date_planned_finished': fields.Datetime.to_datetime(values['date_planned_finished'])
                    })
        return super(MrpWorkorder, self).write(values)

    def _generate_wo_lines(self):
        """ Generate workorder line """
        self.ensure_one()
        moves = (self.move_raw_ids | self.move_finished_ids).filtered(
            lambda move: move.state not in ('done', 'cancel')
        )
        for move in moves:
            qty_to_consume = self._prepare_component_quantity(move, self.qty_producing)
            qty_to_consume = self._get_real_uom_qty(qty_to_consume, True)
            line_values = self._generate_lines_values(move, qty_to_consume)
            self.env['mrp.workorder.line'].create(line_values)

    def _apply_update_workorder_lines(self):
        """ update existing line on the workorder. It could be trigger manually
        after a modification of qty_producing.
        """
        self.ensure_one()
        line_values = self._update_workorder_lines()
        self.env['mrp.workorder.line'].create(line_values['to_create'])
        if line_values['to_delete']:
            line_values['to_delete'].unlink()
        for line, vals in line_values['to_update'].items():
            line.write(vals)

    def _refresh_wo_lines(self):
        """ Modify exisiting workorder line in order to match the reservation on
        stock move line. The strategy is to remove the line that were not
        processed yet then call _generate_lines_values that recreate workorder
        line depending the reservation.
        """
        for workorder in self:
            raw_moves = workorder.move_raw_ids.filtered(
                lambda move: move.state not in ('done', 'cancel')
            )
            wl_to_unlink = self.env['mrp.workorder.line']
            for move in raw_moves:
                rounding = move.product_uom.rounding
                qty_already_consumed = 0.0
                workorder_lines = workorder.raw_workorder_line_ids.filtered(lambda w: w.move_id == move)
                for wl in workorder_lines:
                    if not wl.qty_done:
                        wl_to_unlink |= wl
                        continue

                    qty_already_consumed += wl.qty_done
                qty_to_consume = self._prepare_component_quantity(move, workorder.qty_producing)
                qty_to_consume = self._get_real_uom_qty(qty_to_consume, True)
                wl_to_unlink.unlink()
                if float_compare(qty_to_consume, qty_already_consumed, precision_rounding=rounding) > 0:
                    line_values = workorder._generate_lines_values(move, qty_to_consume - qty_already_consumed)
                    self.env['mrp.workorder.line'].create(line_values)

    def _defaults_from_finished_workorder_line(self, reference_lot_lines):
        for r_line in reference_lot_lines:
            # see which lot we could suggest and its related qty_producing
            if not r_line.lot_id:
                continue
            candidates = self.finished_workorder_line_ids.filtered(lambda line: line.lot_id == r_line.lot_id)
            rounding = self.product_uom_id.rounding
            if not candidates:
                self.write({
                    'finished_lot_id': r_line.lot_id.id,
                    'qty_producing': r_line.qty_done,
                })
                return True
            elif float_compare(candidates.qty_done, r_line.qty_done, precision_rounding=rounding) < 0:
                self.write({
                    'finished_lot_id': r_line.lot_id.id,
                    'qty_producing': r_line.qty_done - candidates.qty_done,
                })
                return True
        return False

    def record_production(self):
        if not self:
            return True

        self.ensure_one()
        self._check_company()
        if float_compare(self.qty_producing, 0, precision_rounding=self.product_uom_id.rounding) <= 0:
            raise UserError(_('Please set the quantity you are currently producing. It should be different from zero.'))

        # Ensure serial numbers are used once
        if self.product_id.tracking == 'serial' and self.finished_lot_id:
            line = self.finished_workorder_line_ids.filtered(
                lambda line: line.lot_id.id == self.finished_lot_id.id
            )
            if line:
                raise UserError(_('You cannot produce the same serial number twice.'))

        # If last work order, then post lots used
        if not self.next_work_order_id:
            self._update_finished_move()

        # Transfer quantities from temporary to final move line or make them final
        self._update_moves()

        # Transfer lot (if present) and quantity produced to a finished workorder line
        if self.product_tracking != 'none':
            self._create_or_update_finished_line()

        # Update workorder quantity produced
        self.qty_produced += self.qty_producing

        # Suggest a finished lot on the next workorder
        if self.next_work_order_id and self.product_tracking != 'none' and (not self.next_work_order_id.finished_lot_id or self.next_work_order_id.finished_lot_id == self.finished_lot_id):
            self.next_work_order_id._defaults_from_finished_workorder_line(self.finished_workorder_line_ids)
            # As we may have changed the quantity to produce on the next workorder,
            # make sure to update its wokorder lines
            self.next_work_order_id._apply_update_workorder_lines()

        # One a piece is produced, you can launch the next work order
        self._start_nextworkorder()

        # Test if the production is done
        rounding = self.production_id.product_uom_id.rounding
        production_qty = self._get_real_uom_qty(self.qty_production)
        if float_compare(self.qty_produced, production_qty, precision_rounding=rounding) < 0:
            previous_wo = self.env['mrp.workorder']
            if self.product_tracking != 'none':
                previous_wo = self.env['mrp.workorder'].search([
                    ('next_work_order_id', '=', self.id)
                ])
            candidate_found_in_previous_wo = False
            if previous_wo:
                candidate_found_in_previous_wo = self._defaults_from_finished_workorder_line(previous_wo.finished_workorder_line_ids)
            if not candidate_found_in_previous_wo:
                # self is the first workorder
                self.qty_producing = self.qty_remaining
                self.finished_lot_id = False
                if self.product_tracking == 'serial':
                    self.qty_producing = 1

            self._apply_update_workorder_lines()
        else:
            self.qty_producing = 0
            self.button_finish()
        return True

    def _get_byproduct_move_to_update(self):
        return self.production_id.move_finished_ids.filtered(lambda x: (x.product_id.id != self.production_id.product_id.id) and (x.state not in ('done', 'cancel')))

    def _create_or_update_finished_line(self):
        """
        1. Check that the final lot and the quantity producing is valid regarding
            other workorders of this production
        2. Save final lot and quantity producing to suggest on next workorder
        """
        self.ensure_one()
        final_lot_quantity = self._get_real_uom_qty(self.qty_production)
        rounding = self.product_uom_id.rounding
        # Get the max quantity possible for current lot in other workorders, avoid
        # considering already completed steps up to the current one
        for workorder in (self.production_id.workorder_ids.filtered(lambda wo: not (wo.id < self.id and wo.state in ('done','cancel'))) - self):
            # We add the remaining quantity to the produced quantity for the
            # current lot. For 5 finished products: if in the first wo it
            # creates 4 lot A and 1 lot B and in the second it create 3 lot A
            # and it remains 2 units to product, it could produce 5 lot A.
            # In this case we select 4 since it would conflict with the first
            # workorder otherwise.
            line = workorder.finished_workorder_line_ids.filtered(lambda line: line.lot_id == self.finished_lot_id)
            line_without_lot = workorder.finished_workorder_line_ids.filtered(lambda line: line.product_id == workorder.product_id and not line.lot_id)
            quantity_remaining = workorder.qty_remaining + line_without_lot.qty_done
            quantity = line.qty_done + quantity_remaining
            if line and float_compare(quantity, final_lot_quantity, precision_rounding=rounding) <= 0:
                final_lot_quantity = quantity
            elif float_compare(quantity_remaining, final_lot_quantity, precision_rounding=rounding) < 0:
                final_lot_quantity = quantity_remaining

        # final lot line for this lot on this workorder.
        current_lot_lines = self.finished_workorder_line_ids.filtered(lambda line: line.lot_id == self.finished_lot_id)

        # this lot has already been produced
        if float_compare(final_lot_quantity, current_lot_lines.qty_done + self.qty_producing, precision_rounding=rounding) < 0:
            raise UserError(_('You have produced %s %s of lot %s in the previous workorder. You are trying to produce %s in this one') %
                (final_lot_quantity, self.product_id.uom_id.name, self.finished_lot_id.name, current_lot_lines.qty_done + self.qty_producing))

        # Update workorder line that regiter final lot created
        if not current_lot_lines:
            current_lot_lines = self.env['mrp.workorder.line'].create({
                'finished_workorder_id': self.id,
                'product_id': self.product_id.id,
                'lot_id': self.finished_lot_id.id,
                'qty_done': self.qty_producing,
            })
        else:
            current_lot_lines.qty_done += self.qty_producing

    def _start_nextworkorder(self):
        rounding = self.product_id.uom_id.rounding
        production_qty = self._get_real_uom_qty(self.qty_production)
        if self.next_work_order_id.state == 'pending' and (
                (self.operation_id.batch == 'no' and
                 float_compare(production_qty, self.qty_produced, precision_rounding=rounding) <= 0) or
                (self.operation_id.batch == 'yes' and
                 float_compare(self.operation_id.batch_size, self.qty_produced, precision_rounding=rounding) <= 0)):
            self.next_work_order_id.state = 'ready'

    def button_start(self):
        self.ensure_one()
        # As button_start is automatically called in the new view
        if self.state in ('done', 'cancel'):
            return True

        # Need a loss in case of the real time exceeding the expected
        timeline = self.env['mrp.workcenter.productivity']
        if self.duration < self.duration_expected:
            loss_id = self.env['mrp.workcenter.productivity.loss'].search([('loss_type','=','productive')], limit=1)
            if not len(loss_id):
                raise UserError(_("You need to define at least one productivity loss in the category 'Productivity'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses."))
        else:
            loss_id = self.env['mrp.workcenter.productivity.loss'].search([('loss_type','=','performance')], limit=1)
            if not len(loss_id):
                raise UserError(_("You need to define at least one productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses."))
        if self.production_id.state != 'progress':
            self.production_id.write({
                'date_start': datetime.now(),
            })
        timeline.create({
            'workorder_id': self.id,
            'workcenter_id': self.workcenter_id.id,
            'description': _('Time Tracking: ')+self.env.user.name,
            'loss_id': loss_id[0].id,
            'date_start': datetime.now(),
            'user_id': self.env.user.id,  # FIXME sle: can be inconsistent with company_id
            'company_id': self.company_id.id,
        })
        if self.state == 'progress':
            return True
        else:
            start_date = datetime.now()
            vals = {
                'state': 'progress',
                'date_start': start_date,
                'date_planned_start': start_date,
            }
            if self.date_planned_finished and self.date_planned_finished < start_date:
                vals['date_planned_finished'] = start_date
            return self.write(vals)

    def button_finish(self):
        self.ensure_one()
        self.end_all()
        end_date = datetime.now()
        return self.write({
            'state': 'done',
            'date_finished': end_date,
            'date_planned_finished': end_date
        })

    def end_previous(self, doall=False):
        """
        @param: doall:  This will close all open time lines on the open work orders when doall = True, otherwise
        only the one of the current user
        """
        # TDE CLEANME
        timeline_obj = self.env['mrp.workcenter.productivity']
        domain = [('workorder_id', 'in', self.ids), ('date_end', '=', False)]
        if not doall:
            domain.append(('user_id', '=', self.env.user.id))
        not_productive_timelines = timeline_obj.browse()
        for timeline in timeline_obj.search(domain, limit=None if doall else 1):
            wo = timeline.workorder_id
            if wo.duration_expected <= wo.duration:
                if timeline.loss_type == 'productive':
                    not_productive_timelines += timeline
                timeline.write({'date_end': fields.Datetime.now()})
            else:
                maxdate = fields.Datetime.from_string(timeline.date_start) + relativedelta(minutes=wo.duration_expected - wo.duration)
                enddate = datetime.now()
                if maxdate > enddate:
                    timeline.write({'date_end': enddate})
                else:
                    timeline.write({'date_end': maxdate})
                    not_productive_timelines += timeline.copy({'date_start': maxdate, 'date_end': enddate})
        if not_productive_timelines:
            loss_id = self.env['mrp.workcenter.productivity.loss'].search([('loss_type', '=', 'performance')], limit=1)
            if not len(loss_id):
                raise UserError(_("You need to define at least one unactive productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses."))
            not_productive_timelines.write({'loss_id': loss_id.id})
        return True

    def end_all(self):
        return self.end_previous(doall=True)

    def button_pending(self):
        self.end_previous()
        return True

    def button_unblock(self):
        for order in self:
            order.workcenter_id.unblock()
        return True

    def action_cancel(self):
        self.leave_id.unlink()
        return self.write({'state': 'cancel'})

    def button_done(self):
        if any([x.state in ('done', 'cancel') for x in self]):
            raise UserError(_('A Manufacturing Order is already done or cancelled.'))
        self.end_all()
        end_date = datetime.now()
        return self.write({
            'state': 'done',
            'date_finished': end_date,
            'date_planned_finished': end_date,
        })

    def button_scrap(self):
        self.ensure_one()
        return {
            'name': _('Scrap'),
            'view_mode': 'form',
            'res_model': 'stock.scrap',
            'view_id': self.env.ref('stock.stock_scrap_form_view2').id,
            'type': 'ir.actions.act_window',
            'context': {'default_company_id': self.production_id.company_id.id,
                        'default_workorder_id': self.id,
                        'default_production_id': self.production_id.id,
                        'product_ids': (self.production_id.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel')) | self.production_id.move_finished_ids.filtered(lambda x: x.state == 'done')).mapped('product_id').ids},
            'target': 'new',
        }

    def action_see_move_scrap(self):
        self.ensure_one()
        action = self.env.ref('stock.action_stock_scrap').read()[0]
        action['domain'] = [('workorder_id', '=', self.id)]
        return action

    @api.depends('qty_production', 'qty_produced')
    def _compute_qty_remaining(self):
        for wo in self:
            production_qty = wo._get_real_uom_qty(wo.qty_production)
            wo.qty_remaining = float_round(production_qty - wo.qty_produced, precision_rounding=wo.production_id.product_uom_id.rounding)

    def _update_workorder_lines(self):
        # OVERRIDE
        line_values = super(MrpWorkorder, self)._update_workorder_lines()
        # wo lines without move_id should also be deleted
        for wo_line in self._workorder_line_ids().filtered(lambda w: not w.move_id and (not w.finished_workorder_id or w.product_id != w.finished_workorder_id.product_id)):
            if not line_values['to_delete']:
                line_values['to_delete'] = wo_line
            elif wo_line not in line_values['to_delete']:
                line_values['to_delete'] |= wo_line
        return line_values


class MrpWorkorderLine(models.Model):
    _name = 'mrp.workorder.line'
    _inherit = ["mrp.abstract.workorder.line"]
    _description = "Workorder move line"

    raw_workorder_id = fields.Many2one('mrp.workorder', 'Component for Workorder',
        ondelete='cascade')
    finished_workorder_id = fields.Many2one('mrp.workorder', 'Finished Product for Workorder',
        ondelete='cascade')

    @api.model
    def _get_raw_workorder_inverse_name(self):
        return 'raw_workorder_id'

    @api.model
    def _get_finished_workoder_inverse_name(self):
        return 'finished_workorder_id'

    def _get_final_lots(self):
        return (self.raw_workorder_id or self.finished_workorder_id).finished_lot_id

    def _get_production(self):
        return (self.raw_workorder_id or self.finished_workorder_id).production_id

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import collections
from datetime import timedelta
import operator as py_operator
from odoo import api, fields, models
from odoo.tools.float_utils import float_round, float_is_zero


OPERATORS = {
    '<': py_operator.lt,
    '>': py_operator.gt,
    '<=': py_operator.le,
    '>=': py_operator.ge,
    '=': py_operator.eq,
    '!=': py_operator.ne
}

class ProductTemplate(models.Model):
    _inherit = "product.template"

    bom_line_ids = fields.One2many('mrp.bom.line', 'product_tmpl_id', 'BoM Components')
    bom_ids = fields.One2many('mrp.bom', 'product_tmpl_id', 'Bill of Materials')
    bom_count = fields.Integer('# Bill of Material',
        compute='_compute_bom_count', compute_sudo=False)
    used_in_bom_count = fields.Integer('# of BoM Where is Used',
        compute='_compute_used_in_bom_count', compute_sudo=False)
    mrp_product_qty = fields.Float('Manufactured',
        compute='_compute_mrp_product_qty', compute_sudo=False)
    produce_delay = fields.Float(
        'Manufacturing Lead Time', default=0.0,
        help="Average lead time in days to manufacture this product. In the case of multi-level BOM, the manufacturing lead times of the components will be added.")

    def _compute_bom_count(self):
        for product in self:
            product.bom_count = self.env['mrp.bom'].search_count([('product_tmpl_id', '=', product.id)])

    def _compute_used_in_bom_count(self):
        for template in self:
            template.used_in_bom_count = self.env['mrp.bom'].search_count(
                [('bom_line_ids.product_id', 'in', template.product_variant_ids.ids)])

    def action_used_in_bom(self):
        self.ensure_one()
        action = self.env.ref('mrp.mrp_bom_form_action').read()[0]
        action['domain'] = [('bom_line_ids.product_id', 'in', self.product_variant_ids.ids)]
        return action

    def _compute_mrp_product_qty(self):
        for template in self:
            template.mrp_product_qty = float_round(sum(template.mapped('product_variant_ids').mapped('mrp_product_qty')), precision_rounding=template.uom_id.rounding)

    def action_view_mos(self):
        action = self.env.ref('mrp.mrp_production_report').read()[0]
        action['domain'] = [('state', '=', 'done'), ('product_tmpl_id', 'in', self.ids)]
        action['context'] = {
            'graph_measure': 'product_uom_qty',
            'time_ranges': {'field': 'date_planned_start', 'range': 'last_365_days'}
        }
        return action

class ProductProduct(models.Model):
    _inherit = "product.product"

    variant_bom_ids = fields.One2many('mrp.bom', 'product_id', 'BOM Product Variants')
    bom_line_ids = fields.One2many('mrp.bom.line', 'product_id', 'BoM Components')
    bom_count = fields.Integer('# Bill of Material',
        compute='_compute_bom_count', compute_sudo=False)
    used_in_bom_count = fields.Integer('# BoM Where Used',
        compute='_compute_used_in_bom_count', compute_sudo=False)
    mrp_product_qty = fields.Float('Manufactured',
        compute='_compute_mrp_product_qty', compute_sudo=False)

    def _compute_bom_count(self):
        for product in self:
            product.bom_count = self.env['mrp.bom'].search_count(['|', ('product_id', '=', product.id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product.product_tmpl_id.id)])

    def _compute_used_in_bom_count(self):
        for product in self:
            product.used_in_bom_count = self.env['mrp.bom'].search_count([('bom_line_ids.product_id', '=', product.id)])

    def get_components(self):
        """ Return the components list ids in case of kit product.
        Return the product itself otherwise"""
        self.ensure_one()
        bom_kit = self.env['mrp.bom']._bom_find(product=self, bom_type='phantom')
        if bom_kit:
            boms, bom_sub_lines = bom_kit.explode(self, 1)
            return [bom_line.product_id.id for bom_line, data in bom_sub_lines if bom_line.product_id.type == 'product']
        else:
            return super(ProductProduct, self).get_components()

    def action_used_in_bom(self):
        self.ensure_one()
        action = self.env.ref('mrp.mrp_bom_form_action').read()[0]
        action['domain'] = [('bom_line_ids.product_id', '=', self.id)]
        return action

    def _compute_mrp_product_qty(self):
        date_from = fields.Datetime.to_string(fields.datetime.now() - timedelta(days=365))
        #TODO: state = done?
        domain = [('state', '=', 'done'), ('product_id', 'in', self.ids), ('date_planned_start', '>', date_from)]
        read_group_res = self.env['mrp.production'].read_group(domain, ['product_id', 'product_uom_qty'], ['product_id'])
        mapped_data = dict([(data['product_id'][0], data['product_uom_qty']) for data in read_group_res])
        for product in self:
            if not product.id:
                product.mrp_product_qty = 0.0
                continue
            product.mrp_product_qty = float_round(mapped_data.get(product.id, 0), precision_rounding=product.uom_id.rounding)

    def _compute_quantities_dict(self, lot_id, owner_id, package_id, from_date=False, to_date=False):
        """ When the product is a kit, this override computes the fields :
         - 'virtual_available'
         - 'qty_available'
         - 'incoming_qty'
         - 'outgoing_qty'
         - 'free_qty'

        This override is used to get the correct quantities of products
        with 'phantom' as BoM type.
        """
        bom_kits = self.env['mrp.bom']._get_product2bom(self, bom_type='phantom')
        kits = self.filtered(lambda p: bom_kits.get(p))
        res = super(ProductProduct, self - kits)._compute_quantities_dict(lot_id, owner_id, package_id, from_date=from_date, to_date=to_date)
        qties = self.env.context.get("mrp_compute_quantities", {})
        qties.update(res)
        for product in bom_kits:
            bom_sub_lines = bom_kits[product].explode(product, 1)[1]
            # group lines by component
            bom_sub_lines_grouped = collections.defaultdict(list)
            for info in bom_sub_lines:
                bom_sub_lines_grouped[info[0].product_id].append(info)
            ratios_virtual_available = []
            ratios_qty_available = []
            ratios_incoming_qty = []
            ratios_outgoing_qty = []
            ratios_free_qty = []

            for component, bom_sub_lines in bom_sub_lines_grouped.items():
                component = component.with_context(mrp_compute_quantities=qties)
                qty_per_kit = 0
                for bom_line, bom_line_data in bom_sub_lines:
                    if component.type != 'product' or float_is_zero(bom_line_data['qty'], precision_rounding=bom_line.product_uom_id.rounding):
                        # As BoMs allow components with 0 qty, a.k.a. optionnal components, we simply skip those
                        # to avoid a division by zero. The same logic is applied to non-storable products as those
                        # products have 0 qty available.
                        continue
                    uom_qty_per_kit = bom_line_data['qty'] / bom_line_data['original_qty']
                    qty_per_kit += bom_line.product_uom_id._compute_quantity(uom_qty_per_kit, bom_line.product_id.uom_id, round=False, raise_if_failure=False)
                if not qty_per_kit:
                    continue
                rounding = component.uom_id.rounding
                component_res = (
                    qties.get(component.id)
                    if component.id in qties
                    else {
                        "virtual_available": float_round(component.virtual_available, precision_rounding=rounding),
                        "qty_available": float_round(component.qty_available, precision_rounding=rounding),
                        "incoming_qty": float_round(component.incoming_qty, precision_rounding=rounding),
                        "outgoing_qty": float_round(component.outgoing_qty, precision_rounding=rounding),
                        "free_qty": float_round(component.free_qty, precision_rounding=rounding),
                    }
                )
                ratios_virtual_available.append(component_res["virtual_available"] / qty_per_kit)
                ratios_qty_available.append(component_res["qty_available"] / qty_per_kit)
                ratios_incoming_qty.append(component_res["incoming_qty"] / qty_per_kit)
                ratios_outgoing_qty.append(component_res["outgoing_qty"] / qty_per_kit)
                ratios_free_qty.append(component_res["free_qty"] / qty_per_kit)
            if bom_sub_lines and ratios_virtual_available:  # Guard against all cnsumable bom: at least one ratio should be present.
                res[product.id] = {
                    'virtual_available': min(ratios_virtual_available) * bom_kits[product].product_qty // 1,
                    'qty_available': min(ratios_qty_available) * bom_kits[product].product_qty // 1,
                    'incoming_qty': min(ratios_incoming_qty) * bom_kits[product].product_qty // 1,
                    'outgoing_qty': min(ratios_outgoing_qty) * bom_kits[product].product_qty // 1,
                    'free_qty': min(ratios_free_qty) * bom_kits[product].product_qty // 1,
                }
            else:
                res[product.id] = {
                    'virtual_available': 0,
                    'qty_available': 0,
                    'incoming_qty': 0,
                    'outgoing_qty': 0,
                    'free_qty': 0,
                }

        return res

    def action_view_bom(self):
        action = self.env.ref('mrp.product_open_bom').read()[0]
        template_ids = self.mapped('product_tmpl_id').ids
        # bom specific to this variant or global to template
        action['context'] = {
            'default_product_tmpl_id': template_ids[0],
            'default_product_id': self.ids[0],
        }
        action['domain'] = ['|', ('product_id', 'in', self.ids), '&', ('product_id', '=', False), ('product_tmpl_id', 'in', template_ids)]
        return action

    def action_view_mos(self):
        action = self.product_tmpl_id.action_view_mos()
        action['domain'] = [('state', '=', 'done'), ('product_id', 'in', self.ids)]
        return action

    def action_open_quants(self):
        bom_kits = {}
        for product in self:
            bom = self.env['mrp.bom']._bom_find(product=product, bom_type='phantom')
            if bom:
                bom_kits[product] = bom
        components = self - self.env['product.product'].concat(*list(bom_kits.keys()))
        for product in bom_kits:
            boms, bom_sub_lines = bom_kits[product].explode(product, 1)
            components |= self.env['product.product'].concat(*[l[0].product_id for l in bom_sub_lines])
        res = super(ProductProduct, components).action_open_quants()
        if bom_kits:
            res['context']['single_product'] = False
            res['context'].pop('default_product_tmpl_id', None)
        return res

    def _search_qty_available_new(self, operator, value, lot_id=False, owner_id=False, package_id=False):
        '''extending the method in stock.product to take into account kits'''
        product_ids = super(ProductProduct, self)._search_qty_available_new(operator, value, lot_id, owner_id, package_id)
        kit_boms = self.env['mrp.bom'].search([('type', "=", 'phantom')])
        kit_products = self.env['product.product']
        for kit in kit_boms:
            if kit.product_id:
                kit_products |= kit.product_id
            else:
                kit_products |= kit.product_tmpl_id.product_variant_ids
        for product in kit_products:
            if OPERATORS[operator](product.qty_available, value):
                product_ids.append(product.id)
        return list(set(product_ids))

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Company(models.Model):
    _inherit = 'res.company'

    manufacturing_lead = fields.Float(
        'Manufacturing Lead Time', default=0.0, required=True,
        help="Security days for each manufacturing operation.")

    def _create_unbuild_sequence(self):
        unbuild_vals = []
        for company in self:
            unbuild_vals.append({
                'name': 'Unbuild',
                'code': 'mrp.unbuild',
                'company_id': company.id,
                'prefix': 'UB/',
                'padding': 5,
                'number_next': 1,
                'number_increment': 1
            })
        if unbuild_vals:
            self.env['ir.sequence'].create(unbuild_vals)

    @api.model
    def create_missing_unbuild_sequences(self):
        company_ids  = self.env['res.company'].search([])
        company_has_unbuild_seq = self.env['ir.sequence'].search([('code', '=', 'mrp.unbuild')]).mapped('company_id')
        company_todo_sequence = company_ids - company_has_unbuild_seq
        company_todo_sequence._create_unbuild_sequence()

    def _create_per_company_sequences(self):
        super(Company, self)._create_per_company_sequences()
        self._create_unbuild_sequence()


```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    manufacturing_lead = fields.Float(related='company_id.manufacturing_lead', string="Manufacturing Lead Time", readonly=False)
    use_manufacturing_lead = fields.Boolean(string="Default Manufacturing Lead Time", config_parameter='mrp.use_manufacturing_lead')
    group_mrp_byproducts = fields.Boolean("By-Products",
        implied_group='mrp.group_mrp_byproducts')
    module_mrp_mps = fields.Boolean("Master Production Schedule")
    module_mrp_plm = fields.Boolean("Product Lifecycle Management (PLM)")
    module_mrp_workorder = fields.Boolean("Work Orders")
    module_quality_control = fields.Boolean("Quality")
    module_mrp_subcontracting = fields.Boolean("Subcontracting")
    group_mrp_routings = fields.Boolean("MRP Work Orders",
        implied_group='mrp.group_mrp_routings')

    @api.onchange('use_manufacturing_lead')
    def _onchange_use_manufacturing_lead(self):
        if not self.use_manufacturing_lead:
            self.manufacturing_lead = 0.0

    @api.onchange('group_mrp_routings')
    def _onchange_group_mrp_routings(self):
        # If we activate 'MRP Work Orders', it means that we need to install 'mrp_workorder'.
        # The opposite is not always true: other modules (such as 'quality_mrp_workorder') may
        # depend on 'mrp_workorder', so we should not automatically uninstall the module if 'MRP
        # Work Orders' is deactivated.
        # Long story short: if 'mrp_workorder' is already installed, we don't uninstall it based on
        # group_mrp_routings
        if self.group_mrp_routings:
            self.module_mrp_workorder = True
        elif not self.env['ir.module.module'].search([('name', '=', 'mrp_workorder'), ('state', '=', 'installed')]):
            self.module_mrp_workorder = False

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, exceptions, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_round, float_is_zero


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    workorder_id = fields.Many2one('mrp.workorder', 'Work Order', check_company=True)
    production_id = fields.Many2one('mrp.production', 'Production Order', check_company=True)
    lot_produced_ids = fields.Many2many('stock.production.lot', string='Finished Lot/Serial Number', check_company=True)
    lot_produced_qty = fields.Float(
        'Quantity Finished Product', digits='Product Unit of Measure',
        help="Informative, not used in matching")
    done_move = fields.Boolean('Move Done', related='move_id.is_done', readonly=False, store=True)  # TDE FIXME: naming

    def _get_similar_move_lines(self):
        lines = super(StockMoveLine, self)._get_similar_move_lines()
        if self.move_id.production_id:
            finished_moves = self.move_id.production_id.move_finished_ids
            finished_move_lines = finished_moves.mapped('move_line_ids')
            lines |= finished_move_lines.filtered(lambda ml: ml.product_id == self.product_id and (ml.lot_id or ml.lot_name))
        if self.move_id.raw_material_production_id:
            raw_moves = self.move_id.raw_material_production_id.move_raw_ids
            raw_moves_lines = raw_moves.mapped('move_line_ids')
            lines |= raw_moves_lines.filtered(lambda ml: ml.product_id == self.product_id and (ml.lot_id or ml.lot_name))
        return lines

    def _reservation_is_updatable(self, quantity, reserved_quant):
        self.ensure_one()
        if self.lot_produced_ids:
            ml_remaining_qty = self.qty_done - self.product_uom_qty
            ml_remaining_qty = self.product_uom_id._compute_quantity(ml_remaining_qty, self.product_id.uom_id, rounding_method="HALF-UP")
            if float_compare(ml_remaining_qty, quantity, precision_rounding=self.product_id.uom_id.rounding) < 0:
                return False
        return super(StockMoveLine, self)._reservation_is_updatable(quantity, reserved_quant)

    def write(self, vals):
        for move_line in self:
            if move_line.move_id.production_id and 'lot_id' in vals:
                move_line.production_id.move_raw_ids.mapped('move_line_ids')\
                    .filtered(lambda r: not r.done_move and move_line.lot_id in r.lot_produced_ids)\
                    .write({'lot_produced_ids': [(4, vals['lot_id'])]})
            production = move_line.move_id.production_id or move_line.move_id.raw_material_production_id
            if production and move_line.state == 'done' and any(field in vals for field in ('lot_id', 'location_id', 'qty_done')):
                move_line._log_message(production, move_line, 'mrp.track_production_move_template', vals)
        return super(StockMoveLine, self).write(vals)


class StockMove(models.Model):
    _inherit = 'stock.move'

    created_production_id = fields.Many2one('mrp.production', 'Created Production Order', check_company=True)
    production_id = fields.Many2one(
        'mrp.production', 'Production Order for finished products', check_company=True)
    raw_material_production_id = fields.Many2one(
        'mrp.production', 'Production Order for components', check_company=True)
    unbuild_id = fields.Many2one(
        'mrp.unbuild', 'Disassembly Order', check_company=True)
    consume_unbuild_id = fields.Many2one(
        'mrp.unbuild', 'Consumed Disassembly Order', check_company=True)
    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Operation To Consume', check_company=True)  # TDE FIXME: naming
    workorder_id = fields.Many2one(
        'mrp.workorder', 'Work Order To Consume', check_company=True)
    # Quantities to process, in normalized UoMs
    bom_line_id = fields.Many2one('mrp.bom.line', 'BoM Line', check_company=True)
    byproduct_id = fields.Many2one(
        'mrp.bom.byproduct', 'By-products', check_company=True,
        help="By-product line that generated the move in a manufacturing order")
    unit_factor = fields.Float('Unit Factor', default=1)
    is_done = fields.Boolean(
        'Done', compute='_compute_is_done',
        store=True,
        help='Technical Field to order moves')
    needs_lots = fields.Boolean('Tracking', compute='_compute_needs_lots')
    order_finished_lot_ids = fields.Many2many('stock.production.lot', compute='_compute_order_finished_lot_ids')
    finished_lots_exist = fields.Boolean('Finished Lots Exist', compute='_compute_order_finished_lot_ids')

    def _unreserve_initial_demand(self, new_move):
        # If you were already putting stock.move.lots on the next one in the work order, transfer those to the new move
        self.filtered(lambda m: m.production_id or m.raw_material_production_id)\
        .mapped('move_line_ids')\
        .filtered(lambda ml: ml.qty_done == 0.0)\
        .write({'move_id': new_move, 'product_uom_qty': 0})

    @api.model
    def _prepare_merge_moves_distinct_fields(self):
        distinct_fields = super()._prepare_merge_moves_distinct_fields()
        distinct_fields.append('bom_line_id')
        return distinct_fields

    @api.depends('raw_material_production_id.move_finished_ids.move_line_ids.lot_id')
    def _compute_order_finished_lot_ids(self):
        for move in self:
            if move.raw_material_production_id.move_finished_ids:
                finished_lots_ids = move.raw_material_production_id.move_finished_ids.mapped('move_line_ids.lot_id').ids
                if finished_lots_ids:
                    move.order_finished_lot_ids = finished_lots_ids
                    move.finished_lots_exist = True
                else:
                    move.order_finished_lot_ids = False
                    move.finished_lots_exist = False
            else:
                move.order_finished_lot_ids = False
                move.finished_lots_exist = False

    @api.depends('product_id.tracking')
    def _compute_needs_lots(self):
        for move in self:
            move.needs_lots = move.product_id.tracking != 'none'

    @api.depends('raw_material_production_id.is_locked', 'picking_id.is_locked')
    def _compute_is_locked(self):
        super(StockMove, self)._compute_is_locked()
        for move in self:
            if move.raw_material_production_id:
                move.is_locked = move.raw_material_production_id.is_locked

    @api.depends('state')
    def _compute_is_done(self):
        for move in self:
            move.is_done = (move.state in ('done', 'cancel'))

    @api.depends('raw_material_production_id.name')
    def _compute_reference(self):
        not_prod_move = self.env['stock.move']
        for move in self:
            if not move.raw_material_production_id:
                not_prod_move |= move
                continue
            move.write({
                'name': move.raw_material_production_id.name,
                'reference': move.raw_material_production_id.name,
            })
        super(StockMove, not_prod_move)._compute_reference()

    @api.model
    def default_get(self, fields_list):
        defaults = super(StockMove, self).default_get(fields_list)
        if self.env.context.get('default_raw_material_production_id'):
            production_id = self.env['mrp.production'].browse(self.env.context['default_raw_material_production_id'])
            if production_id.state == 'done':
                defaults['state'] = 'done'
                defaults['product_uom_qty'] = 0.0
                defaults['additional'] = True
            elif production_id.state == 'draft':
                defaults['group_id'] = production_id.procurement_group_id.id
                defaults['reference'] = production_id.name
        return defaults

    def _action_assign(self):
        res = super(StockMove, self)._action_assign()
        for move in self.filtered(lambda x: x.production_id or x.raw_material_production_id):
            if move.move_line_ids:
                move.move_line_ids.write({'production_id': move.raw_material_production_id.id,
                                               'workorder_id': move.workorder_id.id,})
        return res

    def _action_confirm(self, merge=True, merge_into=False):
        moves = self.action_explode()
        # we go further with the list of ids potentially changed by action_explode
        return super(StockMove, moves)._action_confirm(merge=merge, merge_into=merge_into)

    def action_explode(self):
        """ Explodes pickings """
        # in order to explode a move, we must have a picking_type_id on that move because otherwise the move
        # won't be assigned to a picking and it would be weird to explode a move into several if they aren't
        # all grouped in the same picking.
        moves_to_return = self.env['stock.move']
        moves_to_unlink = self.env['stock.move']
        phantom_moves_vals_list = []
        for move in self:
            if not move.picking_type_id or (move.production_id and move.production_id.product_id == move.product_id):
                moves_to_return |= move
                continue
            bom = self.env['mrp.bom'].sudo()._bom_find(product=move.product_id, company_id=move.company_id.id, bom_type='phantom')
            if not bom:
                moves_to_return |= move
                continue
            if move.picking_id.immediate_transfer:
                factor = move.product_uom._compute_quantity(move.quantity_done, bom.product_uom_id) / bom.product_qty
            else:
                factor = move.product_uom._compute_quantity(move.product_uom_qty, bom.product_uom_id) / bom.product_qty
            boms, lines = bom.sudo().explode(move.product_id, factor, picking_type=bom.picking_type_id)
            for bom_line, line_data in lines:
                if move.picking_id.immediate_transfer:
                    phantom_moves_vals_list += move._generate_move_phantom(bom_line, 0, line_data['qty'])
                else:
                    phantom_moves_vals_list += move._generate_move_phantom(bom_line, line_data['qty'], 0)
            # delete the move with original product which is not relevant anymore
            moves_to_unlink |= move

        moves_to_unlink.sudo().unlink()
        if phantom_moves_vals_list:
            phantom_moves = self.env['stock.move'].create(phantom_moves_vals_list)
            phantom_moves._adjust_procure_method()
            moves_to_return |= phantom_moves.action_explode()
        return moves_to_return

    def _action_cancel(self):
        res = super(StockMove, self)._action_cancel()
        for production in self.mapped('raw_material_production_id'):
            if production.state != 'cancel':
                continue
            production._action_cancel()
        return res

    def _decrease_reserved_quanity(self, quantity):
        """ Decrease the reservation on move lines but keeps the
        all other data.
        """
        move_line_to_unlink = self.env['stock.move.line']
        for move in self:
            reserved_quantity = quantity
            for move_line in move.move_line_ids:
                if move_line.product_uom_qty > reserved_quantity:
                    move_line.product_uom_qty = reserved_quantity
                else:
                    move_line.product_uom_qty = 0
                    reserved_quantity -= move_line.product_uom_qty
                if not move_line.product_uom_qty and not move_line.qty_done:
                    move_line_to_unlink |= move_line
        move_line_to_unlink.unlink()
        return True

    def _do_unreserve(self):
        production_moves = self.filtered(lambda m: m.raw_material_production_id or m.production_id)
        production_moves._decrease_reserved_quanity(0.0)
        return super(StockMove, self - production_moves)._do_unreserve()

    def _prepare_phantom_move_values(self, bom_line, product_qty, quantity_done):
        return {
            'picking_id': self.picking_id.id if self.picking_id else False,
            'product_id': bom_line.product_id.id,
            'product_uom': bom_line.product_uom_id.id,
            'product_uom_qty': product_qty,
            'quantity_done': quantity_done,
            'state': 'draft',  # will be confirmed below
            'name': self.name,
            'bom_line_id': bom_line.id,
        }

    def _generate_move_phantom(self, bom_line, product_qty, quantity_done):
        vals = []
        if bom_line.product_id.type in ['product', 'consu']:
            vals = self.copy_data(default=self._prepare_phantom_move_values(bom_line, product_qty, quantity_done))
            if self.state == 'assigned':
                vals['state'] = 'assigned'
        return vals

    def _get_upstream_documents_and_responsibles(self, visited):
            if self.production_id and self.production_id.state not in ('done', 'cancel'):
                return [(self.production_id, self.production_id.user_id, visited)]
            else:
                return super(StockMove, self)._get_upstream_documents_and_responsibles(visited)

    def _delay_alert_get_documents(self):
        res = super(StockMove, self)._delay_alert_get_documents()
        productions = self.mapped('raw_material_production_id')
        return res + list(productions)

    def _should_be_assigned(self):
        res = super(StockMove, self)._should_be_assigned()
        return bool(res and not (self.production_id or self.raw_material_production_id))

    def _key_assign_picking(self):
        keys = super(StockMove, self)._key_assign_picking()
        return keys + (self.created_production_id,)


    def _compute_kit_quantities(self, product_id, kit_qty, kit_bom, filters):
        """ Computes the quantity delivered or received when a kit is sold or purchased.
        A ratio 'qty_processed/qty_needed' is computed for each component, and the lowest one is kept
        to define the kit's quantity delivered or received.
        :param product_id: The kit itself a.k.a. the finished product
        :param kit_qty: The quantity from the order line
        :param kit_bom: The kit's BoM
        :param filters: Dict of lambda expression to define the moves to consider and the ones to ignore
        :return: The quantity delivered or received
        """
        qty_ratios = []
        boms, bom_sub_lines = kit_bom.explode(product_id, kit_qty)
        for bom_line, bom_line_data in bom_sub_lines:
            bom_line_moves = self.filtered(lambda m: m.bom_line_id == bom_line)
            if bom_line_moves:
                if float_is_zero(bom_line_data['qty'], precision_rounding=bom_line.product_uom_id.rounding):
                    # As BoMs allow components with 0 qty, a.k.a. optionnal components, we simply skip those
                    # to avoid a division by zero.
                    continue
                # We compute the quantities needed of each components to make one kit.
                # Then, we collect every relevant moves related to a specific component
                # to know how many are considered delivered.
                uom_qty_per_kit = bom_line_data['qty'] / bom_line_data['original_qty']
                qty_per_kit = bom_line.product_uom_id._compute_quantity(uom_qty_per_kit, bom_line.product_id.uom_id, round=False)
                if not qty_per_kit:
                    continue
                incoming_moves = bom_line_moves.filtered(filters['incoming_moves'])
                outgoing_moves = bom_line_moves.filtered(filters['outgoing_moves'])
                qty_processed = sum(incoming_moves.mapped('product_qty')) - sum(outgoing_moves.mapped('product_qty'))
                # We compute a ratio to know how many kits we can produce with this quantity of that specific component
                qty_ratios.append(qty_processed / qty_per_kit)
            else:
                return 0.0
        if qty_ratios:
            # Now that we have every ratio by components, we keep the lowest one to know how many kits we can produce
            # with the quantities delivered of each component. We use the floor division here because a 'partial kit'
            # doesn't make sense.
            return min(qty_ratios) // 1
        else:
            return 0.0

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockPickingType(models.Model):
    _inherit = 'stock.picking.type'

    code = fields.Selection(selection_add=[('mrp_operation', 'Manufacturing')])
    count_mo_todo = fields.Integer(string="Number of Manufacturing Orders to Process",
        compute='_get_mo_count')
    count_mo_waiting = fields.Integer(string="Number of Manufacturing Orders Waiting",
        compute='_get_mo_count')
    count_mo_late = fields.Integer(string="Number of Manufacturing Orders Late",
        compute='_get_mo_count')
    use_create_components_lots = fields.Boolean(
        string="Create New Lots/Serial Numbers for Components",
        help="Allow to create new lot/serial numbers for the components",
        default=False,
    )

    def _get_mo_count(self):
        mrp_picking_types = self.filtered(lambda picking: picking.code == 'mrp_operation')
        if not mrp_picking_types:
            self.count_mo_waiting = False
            self.count_mo_todo = False
            self.count_mo_late = False
            return
        domains = {
            'count_mo_waiting': [('reservation_state', '=', 'waiting')],
            'count_mo_todo': ['|', ('state', 'in', ('confirmed', 'draft', 'planned', 'progress', 'to_close'))],
            'count_mo_late': [('date_planned_start', '<', fields.Date.today()), ('state', '=', 'confirmed')],
        }
        for field in domains:
            data = self.env['mrp.production'].read_group(domains[field] +
                [('state', 'not in', ('done', 'cancel')), ('picking_type_id', 'in', self.ids)],
                ['picking_type_id'], ['picking_type_id'])
            count = {x['picking_type_id'] and x['picking_type_id'][0]: x['picking_type_id_count'] for x in data}
            for record in mrp_picking_types:
                record[field] = count.get(record.id, 0)
        remaining = (self - mrp_picking_types)
        if remaining:
            remaining.count_mo_waiting = False
            remaining.count_mo_todo = False
            remaining.count_mo_late = False

    def get_mrp_stock_picking_action_picking_type(self):
        action = self.env.ref('mrp.mrp_production_action_picking_deshboard').read()[0]
        if self:
            action['display_name'] = self.display_name
        return action

class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def _less_quantities_than_expected_add_documents(self, moves, documents):
        documents = super(StockPicking, self)._less_quantities_than_expected_add_documents(moves, documents)

        def _keys_in_sorted(move):
            """ sort by picking and the responsible for the product the
            move.
            """
            return (move.raw_material_production_id.id, move.product_id.responsible_id.id)

        def _keys_in_groupby(move):
            """ group by picking and the responsible for the product the
            move.
            """
            return (move.raw_material_production_id, move.product_id.responsible_id)

        production_documents = self._log_activity_get_documents(moves, 'move_dest_ids', 'DOWN', _keys_in_sorted, _keys_in_groupby)
        return {**documents, **production_documents}

```

## File: models\stock_production_lot.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.exceptions import UserError


class StockProductionLot(models.Model):
    _inherit = 'stock.production.lot'

    def _check_create(self):
        active_mo_id = self.env.context.get('active_mo_id')
        if active_mo_id:
            active_mo = self.env['mrp.production'].browse(active_mo_id)
            if not active_mo.picking_type_id.use_create_components_lots:
                raise UserError(_('You are not allowed to create or edit a lot or serial number for the components with the operation type "Manufacturing". To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers for Components".'))
        return super(StockProductionLot, self)._check_create()

```

## File: models\stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


class StockRule(models.Model):
    _inherit = 'stock.rule'
    action = fields.Selection(selection_add=[('manufacture', 'Manufacture')])

    def _get_message_dict(self):
        message_dict = super(StockRule, self)._get_message_dict()
        source, destination, operation = self._get_message_values()
        manufacture_message = _('When products are needed in <b>%s</b>, <br/> a manufacturing order is created to fulfill the need.') % (destination)
        if self.location_src_id:
            manufacture_message += _(' <br/><br/> The components will be taken from <b>%s</b>.') % (source)
        message_dict.update({
            'manufacture': manufacture_message
        })
        return message_dict

    @api.onchange('action')
    def _onchange_action_operation(self):
        domain = {'picking_type_id': []}
        if self.action == 'manufacture':
            domain = {'picking_type_id': [('code', '=', 'mrp_operation')]}
        return {'domain': domain}

    @api.model
    def _run_manufacture(self, procurements):
        productions_values_by_company = defaultdict(list)
        for procurement, rule in procurements:
            bom = rule._get_matching_bom(procurement.product_id, procurement.company_id, procurement.values)
            if not bom:
                msg = _('There is no Bill of Material of type manufacture or kit found for the product %s. Please define a Bill of Material for this product.') % (procurement.product_id.display_name,)
                raise UserError(msg)

            productions_values_by_company[procurement.company_id.id].append(rule._prepare_mo_vals(*procurement, bom))

        for company_id, productions_values in productions_values_by_company.items():
            # create the MO as SUPERUSER because the current user may not have the rights to do it (mto product launched by a sale for example)
            productions = self.env['mrp.production'].sudo().with_context(force_company=company_id).create(productions_values)
            self.env['stock.move'].sudo().create(productions._get_moves_raw_values())
            productions.action_confirm()

            for production in productions:
                origin_production = production.move_dest_ids and production.move_dest_ids[0].raw_material_production_id or False
                orderpoint = production.orderpoint_id
                if orderpoint:
                    production.message_post_with_view('mail.message_origin_link',
                                                      values={'self': production, 'origin': orderpoint},
                                                      subtype_id=self.env.ref('mail.mt_note').id)
                if origin_production:
                    production.message_post_with_view('mail.message_origin_link',
                                                      values={'self': production, 'origin': origin_production},
                                                      subtype_id=self.env.ref('mail.mt_note').id)
        return True

    @api.model
    def _run_pull(self, procurements):
        # Override to correctly assign the move generated from the pull
        # in its production order (pbm_sam only)
        for procurement, rule in procurements:
            warehouse_id = rule.warehouse_id
            if not warehouse_id:
                warehouse_id = rule.location_id.get_warehouse()
            if rule.picking_type_id == warehouse_id.sam_type_id:
                manu_type_id = warehouse_id.manu_type_id
                if manu_type_id:
                    name = manu_type_id.sequence_id.next_by_id()
                else:
                    name = self.env['ir.sequence'].next_by_code('mrp.production') or _('New')
                # Create now the procurement group that will be assigned to the new MO
                # This ensure that the outgoing move PostProduction -> Stock is linked to its MO
                # rather than the original record (MO or SO)
                procurement.values['group_id'] = self.env["procurement.group"].create({'name': name})
        return super()._run_pull(procurements)

    def _get_custom_move_fields(self):
        fields = super(StockRule, self)._get_custom_move_fields()
        fields += ['bom_line_id']
        return fields

    def _get_matching_bom(self, product_id, company_id, values):
        if values.get('bom_id', False):
            return values['bom_id']
        return self.env['mrp.bom'].with_context(
            company_id=company_id.id, force_company=company_id.id
        )._bom_find(product=product_id, picking_type=self.picking_type_id, bom_type='normal')  # TDE FIXME: context bullshit

    def _prepare_mo_vals(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom):
        date_deadline = fields.Datetime.to_string(self._get_date_planned(product_id, company_id, values))
        mo_values = {
            'origin': origin,
            'product_id': product_id.id,
            'product_qty': product_qty,
            'product_uom_id': product_uom.id,
            'location_src_id': self.location_src_id.id or self.picking_type_id.default_location_src_id.id or location_id.id,
            'location_dest_id': location_id.id,
            'bom_id': bom.id,
            'date_deadline': date_deadline,
            'date_planned_finished': fields.Datetime.from_string(values['date_planned']),
            'date_planned_start': date_deadline,
            'procurement_group_id': False,
            'propagate_cancel': self.propagate_cancel,
            'propagate_date': self.propagate_date,
            'propagate_date_minimum_delta': self.propagate_date_minimum_delta,
            'orderpoint_id': values.get('orderpoint_id', False) and values.get('orderpoint_id').id,
            'picking_type_id': self.picking_type_id.id or values['warehouse_id'].manu_type_id.id,
            'company_id': company_id.id,
            'move_dest_ids': values.get('move_dest_ids') and [(4, x.id) for x in values['move_dest_ids']] or False,
            'user_id': False,
        }
        # Use the procurement group created in _run_pull mrp override
        # Preserve the origin from the original stock move, if available
        if location_id.get_warehouse().manufacture_steps == 'pbm_sam':
            if values.get('move_dest_ids') and values.get('group_id') and values['move_dest_ids'][0].origin != values['group_id'].name:
                origin = values['move_dest_ids'][0].origin
                mo_values.update({
                    'name': values['group_id'].name,
                    'procurement_group_id': values['group_id'].id,
                    'origin': origin,
                })
        return mo_values

    def _get_date_planned(self, product_id, company_id, values):
        format_date_planned = fields.Datetime.from_string(values['date_planned'])
        if product_id.produce_delay:
            date_planned = format_date_planned - relativedelta(days=product_id.produce_delay)
        else:
            date_planned = format_date_planned - relativedelta(hours=1)
        date_planned = date_planned - relativedelta(days=company_id.manufacturing_lead)
        return date_planned

    def _push_prepare_move_copy_values(self, move_to_copy, new_date):
        new_move_vals = super(StockRule, self)._push_prepare_move_copy_values(move_to_copy, new_date)
        new_move_vals['production_id'] = False
        return new_move_vals

class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    @api.model
    def run(self, procurements):
        """ If 'run' is called on a kit, this override is made in order to call
        the original 'run' method with the values of the components of that kit.
        """
        procurements_without_kit = []
        for procurement in procurements:
            bom_kit = self.env['mrp.bom']._bom_find(
                product=procurement.product_id,
                company_id=procurement.company_id.id,
                bom_type='phantom',
            )
            if bom_kit:
                order_qty = procurement.product_uom._compute_quantity(procurement.product_qty, bom_kit.product_uom_id, round=False)
                qty_to_produce = (order_qty / bom_kit.product_qty)
                boms, bom_sub_lines = bom_kit.explode(procurement.product_id, qty_to_produce)
                for bom_line, bom_line_data in bom_sub_lines:
                    bom_line_uom = bom_line.product_uom_id
                    quant_uom = bom_line.product_id.uom_id
                    # recreate dict of values since each child has its own bom_line_id
                    values = dict(procurement.values, bom_line_id=bom_line.id)
                    component_qty, procurement_uom = bom_line_uom._adjust_uom_quantities(bom_line_data['qty'], quant_uom)
                    procurements_without_kit.append(self.env['procurement.group'].Procurement(
                        bom_line.product_id, component_qty, procurement_uom,
                        procurement.location_id, procurement.name,
                        procurement.origin, procurement.company_id, values))
            else:
                procurements_without_kit.append(procurement)
        return super(ProcurementGroup, self).run(procurements_without_kit)

    def _get_moves_to_assign_domain(self, company_id):
        domain = super(ProcurementGroup, self)._get_moves_to_assign_domain(company_id)
        domain = expression.AND([domain, [('production_id', '=', False)]])
        return domain

```

## File: models\stock_scrap.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockScrap(models.Model):
    _inherit = 'stock.scrap'

    production_id = fields.Many2one(
        'mrp.production', 'Manufacturing Order',
        states={'done': [('readonly', True)]}, check_company=True)
    workorder_id = fields.Many2one(
        'mrp.workorder', 'Work Order',
        states={'done': [('readonly', True)]},
        help='Not to restrict or prefer quants, but informative.', check_company=True)

    @api.onchange('workorder_id')
    def _onchange_workorder_id(self):
        if self.workorder_id:
            self.location_id = self.workorder_id.production_id.location_src_id.id

    @api.onchange('production_id')
    def _onchange_production_id(self):
        if self.production_id:
            self.location_id = self.production_id.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel')) and self.production_id.location_src_id.id or self.production_id.location_dest_id.id

    def _prepare_move_values(self):
        vals = super(StockScrap, self)._prepare_move_values()
        if self.production_id:
            vals['origin'] = vals['origin'] or self.production_id.name
            if self.product_id in self.production_id.move_finished_ids.mapped('product_id'):
                vals.update({'production_id': self.production_id.id})
            else:
                vals.update({'raw_material_production_id': self.production_id.id})
        return vals

    def _get_origin_moves(self):
        return super(StockScrap, self)._get_origin_moves() or self.production_id and self.production_id.move_raw_ids.filtered(lambda x: x.product_id == self.product_id)

```

## File: models\stock_traceability.py

```python
from odoo import models, api

class MrpStockReport(models.TransientModel):
    _inherit = 'stock.traceability.report'

    @api.model
    def _get_reference(self, move_line):
        res_model, res_id, ref = super(MrpStockReport, self)._get_reference(move_line)
        if move_line.move_id.production_id and not move_line.move_id.scrapped:
            res_model = 'mrp.production'
            res_id = move_line.move_id.production_id.id
            ref = move_line.move_id.production_id.name
        if move_line.move_id.raw_material_production_id and not move_line.move_id.scrapped:
            res_model = 'mrp.production'
            res_id = move_line.move_id.raw_material_production_id.id
            ref = move_line.move_id.raw_material_production_id.name
        if move_line.move_id.unbuild_id:
            res_model = 'mrp.unbuild'
            res_id = move_line.move_id.unbuild_id.id
            ref = move_line.move_id.unbuild_id.name
        if move_line.move_id.consume_unbuild_id:
            res_model = 'mrp.unbuild'
            res_id = move_line.move_id.consume_unbuild_id.id
            ref = move_line.move_id.consume_unbuild_id.name
        return res_model, res_id, ref

    @api.model
    def _get_linked_move_lines(self, move_line):
        move_lines, is_used = super(MrpStockReport, self)._get_linked_move_lines(move_line)
        if not move_lines:
            move_lines = (move_line.move_id.consume_unbuild_id and move_line.produce_line_ids) or (move_line.move_id.production_id and move_line.consume_line_ids)
        if not is_used:
            is_used = (move_line.move_id.unbuild_id and move_line.consume_line_ids) or (move_line.move_id.raw_material_production_id and move_line.produce_line_ids)
        return move_lines, is_used

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, UserError


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    manufacture_to_resupply = fields.Boolean(
        'Manufacture to Resupply', default=True,
        help="When products are manufactured, they can be manufactured in this warehouse.")
    manufacture_pull_id = fields.Many2one(
        'stock.rule', 'Manufacture Rule')
    manufacture_mto_pull_id = fields.Many2one(
        'stock.rule', 'Manufacture MTO Rule')
    pbm_mto_pull_id = fields.Many2one(
        'stock.rule', 'Picking Before Manufacturing MTO Rule')
    sam_rule_id = fields.Many2one(
        'stock.rule', 'Stock After Manufacturing Rule')
    manu_type_id = fields.Many2one(
        'stock.picking.type', 'Manufacturing Operation Type',
        domain="[('code', '=', 'mrp_operation'), ('company_id', '=', company_id)]", check_company=True)

    pbm_type_id = fields.Many2one('stock.picking.type', 'Picking Before Manufacturing Operation Type', check_company=True)
    sam_type_id = fields.Many2one('stock.picking.type', 'Stock After Manufacturing Operation Type', check_company=True)

    manufacture_steps = fields.Selection([
        ('mrp_one_step', 'Manufacture (1 step)'),
        ('pbm', 'Pick components and then manufacture (2 steps)'),
        ('pbm_sam', 'Pick components, manufacture and then store products (3 steps)')],
        'Manufacture', default='mrp_one_step', required=True,
        help="Produce : Move the components to the production location\
        directly and start the manufacturing process.\nPick / Produce : Unload\
        the components from the Stock to Input location first, and then\
        transfer it to the Production location.")

    pbm_route_id = fields.Many2one('stock.location.route', 'Picking Before Manufacturing Route', ondelete='restrict')

    pbm_loc_id = fields.Many2one('stock.location', 'Picking before Manufacturing Location', check_company=True)
    sam_loc_id = fields.Many2one('stock.location', 'Stock after Manufacturing Location', check_company=True)

    def get_rules_dict(self):
        result = super(StockWarehouse, self).get_rules_dict()
        production_location_id = self._get_production_location()
        for warehouse in self:
            result[warehouse.id].update({
                'mrp_one_step': [],
                'pbm': [
                    self.Routing(warehouse.lot_stock_id, warehouse.pbm_loc_id, warehouse.pbm_type_id, 'pull'),
                    self.Routing(warehouse.pbm_loc_id, production_location_id, warehouse.manu_type_id, 'pull'),
                ],
                'pbm_sam': [
                    self.Routing(warehouse.lot_stock_id, warehouse.pbm_loc_id, warehouse.pbm_type_id, 'pull'),
                    self.Routing(warehouse.pbm_loc_id, production_location_id, warehouse.manu_type_id, 'pull'),
                    self.Routing(warehouse.sam_loc_id, warehouse.lot_stock_id, warehouse.sam_type_id, 'push'),
                ],
            })
        return result

    @api.model
    def _get_production_location(self):
        location = self.env['stock.location'].with_context(force_company=self.company_id.id).search([('usage', '=', 'production'), ('company_id', '=', self.company_id.id)], limit=1)
        if not location:
            raise UserError(_('Can\'t find any production location.'))
        return location

    def _get_routes_values(self):
        routes = super(StockWarehouse, self)._get_routes_values()
        routes.update({
            'pbm_route_id': {
                'routing_key': self.manufacture_steps,
                'depends': ['manufacture_steps', 'manufacture_to_resupply'],
                'route_update_values': {
                    'name': self._format_routename(route_type=self.manufacture_steps),
                    'active': self.manufacture_steps != 'mrp_one_step',
                },
                'route_create_values': {
                    'product_categ_selectable': True,
                    'warehouse_selectable': True,
                    'product_selectable': False,
                    'company_id': self.company_id.id,
                    'sequence': 10,
                },
                'rules_values': {
                    'active': True,
                }
            }
        })
        return routes

    def _get_route_name(self, route_type):
        names = {
            'mrp_one_step': _('Manufacture (1 step)'),
            'pbm': _('Pick components and then manufacture'),
            'pbm_sam': _('Pick components, manufacture and then store products (3 steps)'),
        }
        if route_type in names:
            return names[route_type]
        else:
            return super(StockWarehouse, self)._get_route_name(route_type)

    def _get_global_route_rules_values(self):
        rules = super(StockWarehouse, self)._get_global_route_rules_values()
        location_src = self.manufacture_steps == 'mrp_one_step' and self.lot_stock_id or self.pbm_loc_id
        production_location = self._get_production_location()
        location_id = self.manufacture_steps == 'pbm_sam' and self.sam_loc_id or self.lot_stock_id
        rules.update({
            'manufacture_pull_id': {
                'depends': ['manufacture_steps', 'manufacture_to_resupply'],
                'create_values': {
                    'action': 'manufacture',
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'picking_type_id': self.manu_type_id.id,
                    'route_id': self._find_global_route('mrp.route_warehouse0_manufacture', _('Manufacture')).id
                },
                'update_values': {
                    'active': self.manufacture_to_resupply,
                    'name': self._format_rulename(location_id, False, 'Production'),
                    'location_id': location_id.id,
                    'propagate_cancel': self.manufacture_steps == 'pbm_sam'
                },
            },
            'manufacture_mto_pull_id': {
                'depends': ['manufacture_steps', 'manufacture_to_resupply'],
                'create_values': {
                    'procure_method': 'mts_else_mto',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_global_route('stock.route_warehouse0_mto', _('Make To Order')).id,
                    'location_id': production_location.id,
                    'location_src_id': location_src.id,
                    'picking_type_id': self.manu_type_id.id
                },
                'update_values': {
                    'name': self._format_rulename(location_src, production_location, 'MTO'),
                    'active': self.manufacture_to_resupply,
                },
            },
            'pbm_mto_pull_id': {
                'depends': ['manufacture_steps', 'manufacture_to_resupply'],
                'create_values': {
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_global_route('stock.route_warehouse0_mto', _('Make To Order')).id,
                    'name': self._format_rulename(self.lot_stock_id, self.pbm_loc_id, 'MTO'),
                    'location_id': self.pbm_loc_id.id,
                    'location_src_id': self.lot_stock_id.id,
                    'picking_type_id': self.pbm_type_id.id
                },
                'update_values': {
                    'active': self.manufacture_steps != 'mrp_one_step' and self.manufacture_to_resupply,
                }
            },
            # The purpose to move sam rule in the manufacture route instead of
            # pbm_route_id is to avoid conflict with receipt in multiple
            # step. For example if the product is manufacture and receipt in two
            # step it would conflict in WH/Stock since product could come from
            # WH/post-prod or WH/input. We do not have this conflict with
            # manufacture route since it is set on the product.
            'sam_rule_id': {
                'depends': ['manufacture_steps', 'manufacture_to_resupply'],
                'create_values': {
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_global_route('mrp.route_warehouse0_manufacture', _('Manufacture')).id,
                    'name': self._format_rulename(self.sam_loc_id, self.lot_stock_id, False),
                    'location_id': self.lot_stock_id.id,
                    'location_src_id': self.sam_loc_id.id,
                    'picking_type_id': self.sam_type_id.id
                },
                'update_values': {
                    'active': self.manufacture_steps == 'pbm_sam' and self.manufacture_to_resupply,
                }
            }

        })
        return rules

    def _get_locations_values(self, vals, code=False):
        values = super(StockWarehouse, self)._get_locations_values(vals, code=code)
        def_values = self.default_get(['manufacture_steps'])
        manufacture_steps = vals.get('manufacture_steps', def_values['manufacture_steps'])
        code = vals.get('code') or code or ''
        code = code.replace(' ', '').upper()
        company_id = vals.get('company_id', self.company_id.id)
        values.update({
            'pbm_loc_id': {
                'name': _('Pre-Production'),
                'active': manufacture_steps in ('pbm', 'pbm_sam'),
                'usage': 'internal',
                'barcode': self._valid_barcode(code + '-PREPRODUCTION', company_id)
            },
            'sam_loc_id': {
                'name': _('Post-Production'),
                'active': manufacture_steps == 'pbm_sam',
                'usage': 'internal',
                'barcode': self._valid_barcode(code + '-POSTPRODUCTION', company_id)
            },
        })
        return values

    def _get_sequence_values(self):
        values = super(StockWarehouse, self)._get_sequence_values()
        values.update({
            'pbm_type_id': {'name': self.name + ' ' + _('Sequence picking before manufacturing'), 'prefix': self.code + '/PC/', 'padding': 5, 'company_id': self.company_id.id},
            'sam_type_id': {'name': self.name + ' ' + _('Sequence stock after manufacturing'), 'prefix': self.code + '/SFP/', 'padding': 5, 'company_id': self.company_id.id},
            'manu_type_id': {'name': self.name + ' ' + _('Sequence production'), 'prefix': self.code + '/MO/', 'padding': 5, 'company_id': self.company_id.id},
        })
        return values

    def _get_picking_type_create_values(self, max_sequence):
        data, next_sequence = super(StockWarehouse, self)._get_picking_type_create_values(max_sequence)
        data.update({
            'pbm_type_id': {
                'name': _('Pick Components'),
                'code': 'internal',
                'use_create_lots': True,
                'use_existing_lots': True,
                'default_location_src_id': self.lot_stock_id.id,
                'default_location_dest_id': self.pbm_loc_id.id,
                'sequence': next_sequence + 1,
                'sequence_code': 'PC',
                'company_id': self.company_id.id,
            },
            'sam_type_id': {
                'name': _('Store Finished Product'),
                'code': 'internal',
                'use_create_lots': True,
                'use_existing_lots': True,
                'default_location_src_id': self.sam_loc_id.id,
                'default_location_dest_id': self.lot_stock_id.id,
                'sequence': next_sequence + 3,
                'sequence_code': 'SFP',
                'company_id': self.company_id.id,
            },
            'manu_type_id': {
                'name': _('Manufacturing'),
                'code': 'mrp_operation',
                'use_create_lots': True,
                'use_existing_lots': True,
                'sequence': next_sequence + 2,
                'sequence_code': 'MO',
                'company_id': self.company_id.id,
            },
        })
        return data, max_sequence + 4

    def _get_picking_type_update_values(self):
        data = super(StockWarehouse, self)._get_picking_type_update_values()
        data.update({
            'pbm_type_id': {'active': self.manufacture_to_resupply and self.manufacture_steps in ('pbm', 'pbm_sam') and self.active},
            'sam_type_id': {'active': self.manufacture_to_resupply and self.manufacture_steps == 'pbm_sam' and self.active},
            'manu_type_id': {
                'active': self.manufacture_to_resupply and self.active,
                'default_location_src_id': self.manufacture_steps in ('pbm', 'pbm_sam') and self.pbm_loc_id.id or self.lot_stock_id.id,
                'default_location_dest_id': self.manufacture_steps == 'pbm_sam' and self.sam_loc_id.id or self.lot_stock_id.id,
            },
        })
        return data

    def write(self, vals):
        if any(field in vals for field in ('manufacture_steps', 'manufacture_to_resupply')):
            for warehouse in self:
                warehouse._update_location_manufacture(vals.get('manufacture_steps', warehouse.manufacture_steps))
        return super(StockWarehouse, self).write(vals)

    def _get_all_routes(self):
        routes = super(StockWarehouse, self)._get_all_routes()
        routes |= self.filtered(lambda self: self.manufacture_to_resupply and self.manufacture_pull_id and self.manufacture_pull_id.route_id).mapped('manufacture_pull_id').mapped('route_id')
        return routes

    def _update_location_manufacture(self, new_manufacture_step):
        self.mapped('pbm_loc_id').write({'active': new_manufacture_step != 'mrp_one_step'})
        self.mapped('sam_loc_id').write({'active': new_manufacture_step == 'pbm_sam'})

    def _update_name_and_code(self, name=False, code=False):
        res = super(StockWarehouse, self)._update_name_and_code(name, code)
        # change the manufacture stock rule name
        for warehouse in self:
            if warehouse.manufacture_pull_id and name:
                warehouse.manufacture_pull_id.write({'name': warehouse.manufacture_pull_id.name.replace(warehouse.name, name, 1)})
        return res

class Orderpoint(models.Model):
    _inherit = "stock.warehouse.orderpoint"

    @api.constrains('product_id')
    def check_product_is_not_kit(self):
        if self.env['mrp.bom'].search(['|', ('product_id', 'in', self.product_id.ids),
                                            '&', ('product_id', '=', False), ('product_tmpl_id', 'in', self.product_id.product_tmpl_id.ids),
                                       ('type', '=', 'phantom')], count=True):
            raise ValidationError(_("A product with a kit-type bill of materials can not have a reordering rule."))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_document
from . import mrp_abstract_workorder
from . import res_config_settings
from . import mrp_bom
from . import mrp_routing
from . import mrp_workcenter
from . import mrp_production
from . import stock_traceability
from . import mrp_unbuild
from . import mrp_workorder
from . import product
from . import res_company
from . import stock_move
from . import stock_picking
from . import stock_production_lot
from . import stock_rule
from . import stock_scrap
from . import stock_warehouse

```

## File: report\mrp_production_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_mrporder">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="web.internal_layout">
                <div class="page">
                    <div class="oe_structure"/>
                    <div class="row">
                        <div class="col-7">
                            <h2><span t-field="o.name"/></h2>
                        </div>
                        <div class="col-5">
                            <span class="text-right">
                                <img t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', o.name, 600, 100)" style="width:350px;height:60px"/>
                            </span>
                        </div>
                    </div>
                    <div class="row mt32 mb32">
                        <div class="col-3" t-if="o.origin">
                            <strong>Source Document:</strong><br/>
                            <span t-field="o.origin"/>
                        </div>
                        <div class="col-3">
                            <strong>Responsible:</strong><br/>
                            <span t-field="o.user_id"/>
                        </div>
                    </div>

                    <div class="row mt32 mb32">
                        <div class="col-3">
                            <strong>Finished Product:</strong><br/>
                            <span t-field="o.product_id"/>
                        </div>
                        <div class="col-3">
                            <strong>Quantity to Produce:</strong><br/>
                            <span t-field="o.product_qty"/>
                            <span t-field="o.product_uom_id.name" groups="uom.group_uom"/>
                        </div>
                    </div>

                    <div t-if="o.workorder_ids">
                        <h3>
                            <t t-if="o.state == 'done'">Operations Done</t>
                            <t t-else="">Operations Planned</t>
                        </h3>
                        <table class="table table-sm">
                            <tr>
                                <th><strong>Operation</strong></th>
                                <th><strong>WorkCenter</strong></th>
                                <th><strong>No. Of Minutes</strong></th>
                            </tr>
                            <tr t-foreach="o.workorder_ids" t-as="line2">
                                <td><span t-field="line2.name"/></td>
                                <td><span t-field="line2.workcenter_id.name"/></td>
                                <td>
                                    <span t-if="o.state != 'done'" t-field="line2.duration_expected"/>
                                    <span t-if="o.state == 'done'" t-field="line2.duration"/>
                                </td>
                            </tr>
                        </table>
                    </div>

                    <h3 t-if="o.move_raw_ids">
                        <t t-if="o.state == 'done'">
                            Consumed Products
                        </t>
                        <t t-else="">
                            Products to Consume
                        </t>
                    </h3>

                    <table class="table table-sm" t-if="o.move_raw_ids">
                        <t t-set="has_product_barcode" t-value="any(o.move_raw_ids.filtered(lambda x: x.product_id.barcode))"/>
                        <thead>
                            <tr>
                                <th>Product</th>
                                <th t-attf-class="{{ 'text-right' if not has_product_barcode else '' }}">Quantity</th>
                                <th t-if="has_product_barcode" width="15%" class="text-center">Barcode</th>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-if="o.move_raw_ids">
                                <tr t-foreach="o.move_raw_ids" t-as="raw_line">
                                    <td>
                                        <span t-field="raw_line.product_id"/>
                                    </td>
                                    <td t-attf-class="{{ 'text-right' if not has_product_barcode else '' }}">
                                        <span t-field="raw_line.product_uom_qty"/>
                                        <span t-field="raw_line.product_uom" groups="uom.group_uom"/>
                                    </td>
                                    <td t-if="has_product_barcode" width="15%" class="text-center">
                                        <t t-if="raw_line.product_id.barcode">
                                            <img t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', raw_line.product_id.barcode, 600, 100)" style="width:100%;height:35px" alt="Barcode"/>
                                        </t>
                                    </td>
                                </tr>
                            </t>
                        </tbody>
                    </table>
                    <div class="oe_structure"/>
                </div>
            </t>
        </t>
    </t>
</template>

<template id="label_production_view_pdf">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-foreach="docs" t-as="production">
                <t t-foreach="production.move_finished_ids" t-as="move">
                    <t t-if="production.state == 'done'">
                        <t t-set="move_lines" t-value="move.move_line_ids.filtered(lambda x: x.state == 'done' and x.qty_done)"/>
                    </t>
                    <t t-else="">
                        <t t-set="move_lines" t-value="move.move_line_ids.filtered(lambda x: x.state != 'done' and x.product_qty)"/>
                    </t>
                    <t t-foreach="move_lines" t-as="move_line">
                        <t t-if="move_line.product_uom_id.category_id.measure_type == 'unit'">
                            <t t-set="qty" t-value="int(move_line.qty_done)"/>
                        </t>
                        <t t-else="">
                            <t t-set="qty" t-value="1"/>
                        </t>
                        <t t-foreach="range(qty)" t-as="item">
                            <t t-translation="off">
                                <div style="display: inline-table; height: 10rem; width: 32%;">
                                    <table class="table table-bordered" style="border: 2px solid black;" t-if="production.move_finished_ids">
                                        <tr>
                                            <th class="table-active text-left" style="height:4rem;">
                                                <span t-esc="move.product_id.display_name"/>
                                                <br/>
                                                Quantity:
                                                <t t-if="move_line.product_uom_id.category_id.measure_type == 'unit'">
                                                    <span>1.0</span>
                                                    <span t-field="move_line.product_uom_id" groups="uom.group_uom"/>
                                                </t>
                                                <t t-else="">
                                                    <span t-esc="move_line.product_uom_qty" t-if="move_line.state !='done'"/>
                                                    <span t-esc="move_line.qty_done"  t-if="move_line.state =='done'"/>
                                                    <span t-field="move_line.product_uom_id" groups="uom.group_uom"/>
                                                </t>
                                            </th>
                                        </tr>
                                        <t t-if="move_line.product_id.tracking != 'none'">
                                            <tr>
                                                <td class="text-center align-middle">
                                                    <t t-if="move_line.lot_name or move_line.lot_id">
                                                        <img t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', move_line.lot_name or move_line.lot_id.name, 600, 150)" style="width:100%;height:4rem" alt="Barcode"/>
                                                        <span t-esc="move_line.lot_name or move_line.lot_id.name"/>
                                                    </t>
                                                    <t t-else="">
                                                        <span class="text-muted">No barcode available</span>
                                                    </t>
                                                </td>
                                            </tr>
                                        </t>
                                        <t t-if="move_line.product_id.tracking == 'none'">
                                            <tr>
                                                <td class="text-center align-middle" style="height: 6rem;">
                                                    <t t-if="move_line.product_id.barcode">
                                                            <img t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', move_line.product_id.barcode, 600, 150)" style="width:100%;height:4rem" alt="Barcode"/>
                                                            <span t-esc="move_line.product_id.barcode"/>
                                                    </t>
                                                    <t t-else="">
                                                        <span class="text-muted">No barcode available</span>
                                                    </t>
                                                </td>
                                            </tr>
                                        </t>
                                    </table>
                                </div>
                            </t>
                        </t>
                    </t>
                </t>
            </t>
        </div>
    </t>
</template>

<template id="production_message">
    <t t-if="move.move_id.raw_material_production_id">
        <t t-set="message">Consumed</t>
    </t>
    <t t-if="move.move_id.production_id">
        <t t-set="message">Produced</t>
    </t>
    <strong><t t-esc="message"/> quantity has been updated.</strong>
</template>

<template id="track_production_move_template">
    <div>
        <t t-call="mrp.production_message"/>
        <t t-call="stock.message_body"/>
    </div>
</template>
</odoo>

```

## File: report\mrp_report_bom_structure.py

```python
# -*- coding: utf-8 -*-

import json

from odoo import api, models, _
from odoo.tools import float_round

class ReportBomStructure(models.AbstractModel):
    _name = 'report.mrp.report_bom_structure'
    _description = 'BOM Structure Report'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = []
        for bom_id in docids:
            bom = self.env['mrp.bom'].browse(bom_id)
            candidates = bom.product_id or bom.product_tmpl_id.product_variant_ids
            quantity = float(data.get('quantity', bom.product_qty))
            for product_variant_id in candidates.ids:
                if data and data.get('childs'):
                    doc = self._get_pdf_line(bom_id, product_id=product_variant_id, qty=quantity, child_bom_ids=json.loads(data.get('childs')))
                else:
                    doc = self._get_pdf_line(bom_id, product_id=product_variant_id, qty=quantity, unfolded=True)
                doc['report_type'] = 'pdf'
                doc['report_structure'] = data and data.get('report_type') or 'all'
                docs.append(doc)
            if not candidates:
                if data and data.get('childs'):
                    doc = self._get_pdf_line(bom_id, qty=quantity, child_bom_ids=json.loads(data.get('childs')))
                else:
                    doc = self._get_pdf_line(bom_id, qty=quantity, unfolded=True)
                doc['report_type'] = 'pdf'
                doc['report_structure'] = data and data.get('report_type') or 'all'
                docs.append(doc)
        return {
            'doc_ids': docids,
            'doc_model': 'mrp.bom',
            'docs': docs,
        }

    @api.model
    def get_html(self, bom_id=False, searchQty=1, searchVariant=False):
        res = self._get_report_data(bom_id=bom_id, searchQty=searchQty, searchVariant=searchVariant)
        res['lines']['report_type'] = 'html'
        res['lines']['report_structure'] = 'all'
        res['lines']['has_attachments'] = res['lines']['attachments'] or any(component['attachments'] for component in res['lines']['components'])
        res['lines'] = self.env.ref('mrp.report_mrp_bom').render({'data': res['lines']})
        return res

    @api.model
    def get_bom(self, bom_id=False, product_id=False, line_qty=False, line_id=False, level=False):
        lines = self._get_bom(bom_id=bom_id, product_id=product_id, line_qty=line_qty, line_id=line_id, level=level)
        return self.env.ref('mrp.report_mrp_bom_line').render({'data': lines})

    @api.model
    def get_operations(self, bom_id=False, qty=0, level=0):
        bom = self.env['mrp.bom'].browse(bom_id)
        lines = self._get_operation_line(bom.routing_id, float_round(qty / bom.product_qty, precision_rounding=1, rounding_method='UP'), level)
        values = {
            'bom_id': bom_id,
            'currency': self.env.company.currency_id,
            'operations': lines,
        }
        return self.env.ref('mrp.report_mrp_operation_line').render({'data': values})

    @api.model
    def _get_report_data(self, bom_id, searchQty=0, searchVariant=False):
        lines = {}
        bom = self.env['mrp.bom'].browse(bom_id)
        bom_quantity = searchQty or bom.product_qty or 1
        bom_product_variants = {}
        bom_uom_name = ''

        if bom:
            bom_uom_name = bom.product_uom_id.name

            # Get variants used for search
            if not bom.product_id:
                for variant in bom.product_tmpl_id.product_variant_ids:
                    bom_product_variants[variant.id] = variant.display_name

        lines = self._get_bom(bom_id, product_id=searchVariant, line_qty=bom_quantity, level=1)
        return {
            'lines': lines,
            'variants': bom_product_variants,
            'bom_uom_name': bom_uom_name,
            'bom_qty': bom_quantity,
            'is_variant_applied': self.env.user.user_has_groups('product.group_product_variant') and len(bom_product_variants) > 1,
            'is_uom_applied': self.env.user.user_has_groups('uom.group_uom')
        }

    def _get_bom(self, bom_id=False, product_id=False, line_qty=False, line_id=False, level=False):
        bom = self.env['mrp.bom'].browse(bom_id)
        company = bom.company_id or self.env.company
        bom_quantity = line_qty
        if line_id:
            current_line = self.env['mrp.bom.line'].browse(int(line_id))
            bom_quantity = current_line.product_uom_id._compute_quantity(line_qty, bom.product_uom_id) or 0
        # Display bom components for current selected product variant
        if product_id:
            product = self.env['product.product'].browse(int(product_id))
        else:
            product = bom.product_id or bom.product_tmpl_id.product_variant_id
        if product:
            price = product.uom_id._compute_price(product.with_context(force_company=company.id).standard_price, bom.product_uom_id) * bom_quantity
            attachments = self.env['mrp.document'].search(['|', '&', ('res_model', '=', 'product.product'),
            ('res_id', '=', product.id), '&', ('res_model', '=', 'product.template'), ('res_id', '=', product.product_tmpl_id.id)])
        else:
            # Use the product template instead of the variant
            price = bom.product_tmpl_id.uom_id._compute_price(bom.product_tmpl_id.with_context(force_company=company.id).standard_price, bom.product_uom_id) * bom_quantity
            attachments = self.env['mrp.document'].search([('res_model', '=', 'product.template'), ('res_id', '=', bom.product_tmpl_id.id)])
        operations = []
        if bom.product_qty > 0:
            operations = self._get_operation_line(bom.routing_id, float_round(bom_quantity / bom.product_qty, precision_rounding=1, rounding_method='UP'), 0)
        lines = {
            'bom': bom,
            'bom_qty': bom_quantity,
            'bom_prod_name': product.display_name,
            'currency': company.currency_id,
            'product': product,
            'code': bom and bom.display_name or '',
            'price': price,
            'total': sum([op['total'] for op in operations]),
            'level': level or 0,
            'operations': operations,
            'operations_cost': sum([op['total'] for op in operations]),
            'attachments': attachments,
            'operations_time': sum([op['duration_expected'] for op in operations])
        }
        components, total = self._get_bom_lines(bom, bom_quantity, product, line_id, level)
        lines['components'] = components
        lines['total'] += total
        return lines

    def _get_bom_lines(self, bom, bom_quantity, product, line_id, level):
        components = []
        total = 0
        for line in bom.bom_line_ids:
            line_quantity = (bom_quantity / (bom.product_qty or 1.0)) * line.product_qty
            if line._skip_bom_line(product):
                continue
            company = bom.company_id or self.env.company
            price = line.product_id.uom_id._compute_price(line.product_id.with_context(force_company=company.id).standard_price, line.product_uom_id) * line_quantity
            if line.child_bom_id:
                factor = line.product_uom_id._compute_quantity(line_quantity, line.child_bom_id.product_uom_id) / line.child_bom_id.product_qty
                sub_total = self._get_price(line.child_bom_id, factor, line.product_id)
            else:
                sub_total = price
            sub_total = self.env.company.currency_id.round(sub_total)
            components.append({
                'prod_id': line.product_id.id,
                'prod_name': line.product_id.display_name,
                'code': line.child_bom_id and line.child_bom_id.display_name or '',
                'prod_qty': line_quantity,
                'prod_uom': line.product_uom_id.name,
                'prod_cost': company.currency_id.round(price),
                'parent_id': bom.id,
                'line_id': line.id,
                'level': level or 0,
                'total': sub_total,
                'child_bom': line.child_bom_id.id,
                'phantom_bom': line.child_bom_id and line.child_bom_id.type == 'phantom' or False,
                'attachments': self.env['mrp.document'].search(['|', '&',
                    ('res_model', '=', 'product.product'), ('res_id', '=', line.product_id.id), '&', ('res_model', '=', 'product.template'), ('res_id', '=', line.product_id.product_tmpl_id.id)]),

            })
            total += sub_total
        return components, total

    def _get_operation_line(self, routing, qty, level):
        operations = []
        total = 0.0
        for operation in routing.operation_ids:
            operation_cycle = float_round(qty / operation.workcenter_id.capacity, precision_rounding=1, rounding_method='UP')
            duration_expected = operation_cycle * operation.time_cycle + operation.workcenter_id.time_stop + operation.workcenter_id.time_start
            total = ((duration_expected / 60.0) * operation.workcenter_id.costs_hour)
            operations.append({
                'level': level or 0,
                'operation': operation,
                'name': operation.name + ' - ' + operation.workcenter_id.name,
                'duration_expected': duration_expected,
                'total': self.env.company.currency_id.round(total),
            })
        return operations

    def _get_price(self, bom, factor, product):
        price = 0
        if bom.routing_id:
            # routing are defined on a BoM and don't have a concept of quantity.
            # It means that the operation time are defined for the quantity on
            # the BoM (the user produces a batch of products). E.g the user
            # product a batch of 10 units with a 5 minutes operation, the time
            # will be the 5 for a quantity between 1-10, then doubled for
            # 11-20,...
            operation_cycle = float_round(factor, precision_rounding=1, rounding_method='UP')
            operations = self._get_operation_line(bom.routing_id, operation_cycle, 0)
            price += sum([op['total'] for op in operations])

        for line in bom.bom_line_ids:
            if line._skip_bom_line(product):
                continue
            if line.child_bom_id:
                qty = line.product_uom_id._compute_quantity(line.product_qty * factor, line.child_bom_id.product_uom_id) / line.child_bom_id.product_qty
                sub_price = self._get_price(line.child_bom_id, qty, line.product_id)
                price += sub_price
            else:
                prod_qty = line.product_qty * factor
                company = bom.company_id or self.env.company
                not_rounded_price = line.product_id.uom_id._compute_price(line.product_id.with_context(force_company=company.id).standard_price, line.product_uom_id) * prod_qty
                price += company.currency_id.round(not_rounded_price)
        return price

    def _get_pdf_line(self, bom_id, product_id=False, qty=1, child_bom_ids=[], unfolded=False):

        def get_sub_lines(bom, product_id, line_qty, line_id, level):
            data = self._get_bom(bom_id=bom.id, product_id=product_id, line_qty=line_qty, line_id=line_id, level=level)
            bom_lines = data['components']
            lines = []
            for bom_line in bom_lines:
                lines.append({
                    'name': bom_line['prod_name'],
                    'type': 'bom',
                    'quantity': bom_line['prod_qty'],
                    'uom': bom_line['prod_uom'],
                    'prod_cost': bom_line['prod_cost'],
                    'bom_cost': bom_line['total'],
                    'level': bom_line['level'],
                    'code': bom_line['code'],
                    'child_bom': bom_line['child_bom'],
                    'prod_id': bom_line['prod_id']
                })
                if bom_line['child_bom'] and (unfolded or bom_line['child_bom'] in child_bom_ids):
                    line = self.env['mrp.bom.line'].browse(bom_line['line_id'])
                    lines += (get_sub_lines(line.child_bom_id, line.product_id.id, bom_line['prod_qty'], line, level + 1))
            if data['operations']:
                lines.append({
                    'name': _('Operations'),
                    'type': 'operation',
                    'quantity': data['operations_time'],
                    'uom': _('minutes'),
                    'bom_cost': data['operations_cost'],
                    'level': level,
                })
                for operation in data['operations']:
                    if unfolded or 'operation-' + str(bom.id) in child_bom_ids:
                        lines.append({
                            'name': operation['name'],
                            'type': 'operation',
                            'quantity': operation['duration_expected'],
                            'uom': _('minutes'),
                            'bom_cost': operation['total'],
                            'level': level + 1,
                        })
            return lines

        bom = self.env['mrp.bom'].browse(bom_id)
        product_id = product_id or bom.product_id.id or bom.product_tmpl_id.product_variant_id.id
        data = self._get_bom(bom_id=bom_id, product_id=product_id, line_qty=qty)
        pdf_lines = get_sub_lines(bom, product_id, qty, False, 1)
        data['components'] = []
        data['lines'] = pdf_lines
        return data

```

## File: report\mrp_report_bom_structure.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="report_mrp_bom">
        <div class="container o_mrp_bom_report_page">
            <t t-if="data.get('components') or data.get('lines')">
                <div class="row">
                    <div class="col-lg-12">
                        <h1>BoM Structure &amp; Cost</h1>
                        <h3>
                            <a href="#" t-if="data['report_type'] == 'html'" t-att-data-res-id="data['product'].id" t-att-data-model="data['product']._name" class="o_mrp_bom_action">
                                <t t-esc="data['bom_prod_name']"/>
                            </a>
                            <t t-else="" t-esc="data['bom_prod_name']"/>
                        </h3>
                        <h6 t-if="data['bom'].code">Reference: <t t-esc="data['bom'].code"/></h6>
                    </div>
                </div>
                <t t-set="currency" t-value="data['currency']"/>
                <div class="row">
                    <div class="col-lg-12">
                        <div class="mt16">
                            <table width="100%" class="o_mrp_bom_expandable">
                                <thead>
                                    <tr>
                                        <th>Product</th>
                                        <th name="th_mrp_bom_h">BoM</th>
                                        <th class="text-right">Quantity</th>
                                        <th class="text-left" groups="uom.group_uom">Unit of Measure</th>
                                        <th t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_prod_cost text-right" title="This is the cost defined on the product.">Product Cost</th>
                                        <th t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_bom_cost text-right" title="This is the cost based on the BoM of the product. It is computed by summing the costs of the components and operations needed to build the product.">BoM Cost</th>
                                        <th t-if="data['report_type'] == 'html' and data['has_attachments']" class="o_mrp_has_attachments" title="Files attached to the product">Attachments</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr>
                                        <td>
                                            <span><a href="#" t-if="data['report_type'] == 'html'" t-att-data-res-id="data['product'].id" t-att-data-model="'product.product'" class="o_mrp_bom_action"><t t-esc="data['bom_prod_name']"/></a><t t-else="" t-esc="data['bom_prod_name']"/></span>
                                        </td>
                                        <td name="td_mrp_bom">
                                            <div><a href="#" t-if="data['report_type'] == 'html'" t-att-data-res-id="data['bom'].id" t-att-data-model="'mrp.bom'" class="o_mrp_bom_action"><t t-esc="data['code']"/></a><t t-else="" t-esc="data['code']"/></div>
                                        </td>
                                        <td class="text-right"><span><t t-esc="data['bom_qty']" t-options='{"widget": "float", "decimal_precision": "Product Unit of Measure"}'/></span></td>
                                        <td groups="uom.group_uom"><span><t t-esc="data['bom'].product_uom_id.name"/></span></td>
                                        <td t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_prod_cost text-right">
                                            <span><t t-esc="data['price']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                        </td>
                                        <td t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_bom_cost text-right">
                                            <span><t t-esc="data['total']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                        </td>
                                        <td t-if="data['report_type'] == 'html'">
                                            <span>
                                                <t t-if="data['attachments']">
                                                    <a href="#" role="button" t-att-data-res-id="data['attachments'].ids" t-att-data-model="'mrp.document'" class="o_mrp_show_attachment_action fa fa-fw o_button_icon fa-files-o"/>
                                                </t>
                                            </span>
                                        </td>
                                    </tr>
                                    <t t-if="data['report_type'] == 'html'" t-call="mrp.report_mrp_bom_line"/>
                                    <t t-if="data['report_type'] == 'pdf'" t-call="mrp.report_mrp_bom_pdf_line"/>
                                </tbody>
                                <tfoot>
                                    <tr>
                                        <td></td>
                                        <td name="td_mrp_bom_f"></td>
                                        <td t-if="data['report_structure'] != 'bom_structure'" class="text-right o_mrp_prod_cost"><span><strong>Unit Cost</strong></span></td>
                                        <td groups="uom.group_uom"></td>
                                        <td t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_prod_cost text-right">
                                            <span><t t-esc="data['price']/data['bom_qty']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                        </td>
                                        <td t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_bom_cost text-right">
                                            <span><t t-esc="data['total']/data['bom_qty']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                        </td>
                                    </tr>
                                </tfoot>
                            </table>
                        </div>
                    </div>
                </div>
            </t>
            <t t-else="">
                <h1 class="text-center">No data available.</h1>
            </t>
        </div>
    </template>

    <template id="report_mrp_bom_line">
        <t t-set="currency" t-value="data['currency']"/>
        <t t-foreach="data['components']" t-as="l">
            <t t-set="space_td" t-value="'margin-left: '+ str(l['level'] * 20) + 'px;'"/>
            <tr class="o_mrp_bom_report_line" t-att-data-id="l['child_bom']" t-att-parent_id="l['parent_id']" t-att-data-line="l['line_id']" t-att-data-product_id="l['prod_id']" t-att-data-qty="l['prod_qty']" t-att-data-level="l['level']">
                <td>
                    <div t-att-style="space_td">
                        <t t-if="l['child_bom']">
                            <div t-att-data-function="'get_bom'" class="o_mrp_bom_unfoldable fa fa-fw fa-caret-right" style="display:inline-block;" role="img" aria-label="Unfold" title="Unfold"/>
                        </t>
                        <div t-att-class="None if l['child_bom'] else 'o_mrp_bom_no_fold'" style="display:inline-block;">
                            <a href="#" t-att-data-res-id="l['prod_id']" t-att-data-model="'product.product'" class="o_mrp_bom_action"><t t-esc="l['prod_name']"/></a>
                        </div>
                        <t t-if="l['phantom_bom']">
                            <div class="fa fa-dropbox" title="This is a BoM of type Kit!" role="img" aria-label="This is a BoM of type Kit!"/>
                        </t>
                    </div>
                </td>
                <td name="td_mrp_bom">
                    <div>
                        <a href="#" t-att-data-res-id="l['child_bom']" t-att-data-model="'mrp.bom'" class="o_mrp_bom_action"><t t-esc="l['code']"/></a>
                  </div>
                </td>
                <td class="text-right"><span><t t-esc="l['prod_qty']" t-options='{"widget": "float", "decimal_precision": "Product Unit of Measure"}'/></span></td>
                <td groups="uom.group_uom"><span><t t-esc="l['prod_uom']"/></span></td>
                <td class="o_mrp_prod_cost text-right">
                    <span t-esc="l['prod_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                </td>
                <td class="o_mrp_bom_cost text-right">
                    <span t-esc="l['total']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                </td>
                <td>
                    <span>
                        <t t-if="l['attachments']">
                            <a href="#" role="button" t-att-data-res-id="l['attachments'].ids" t-att-data-model="'mrp.document'" class="o_mrp_show_attachment_action fa fa-fw o_button_icon fa-files-o"/>
                        </t>
                    </span>
                </td>
            </tr>
        </t>
        <t t-if="data['operations']">
            <t t-set="space_td" t-value="'margin-left: '+ str(data['level'] * 20) + 'px;'"/>
            <tr class="o_mrp_bom_report_line o_mrp_bom_cost" t-att-data-id="'operation-' + str(data['bom'].id)" t-att-data-bom-id="data['bom'].id" t-att-parent_id="data['bom'].id" t-att-data-qty="data['bom_qty']" t-att-data-level="data['level']">
                <td name="td_opr">
                    <span t-att-style="space_td"/>
                    <span class="o_mrp_bom_unfoldable fa fa-fw fa-caret-right" t-att-data-function="'get_operations'" role="img" aria-label="Unfold" title="Unfold"/>
                    Operations
                </td>
                <td/>
                <td class="text-right">
                    <span t-esc="data['operations_time']" t-options='{"widget": "float_time"}'/>
                </td>
                <td groups="uom.group_uom"><span>Minutes</span></td>
                <td class="o_mrp_prod_cost">
                </td>
                <td class="o_mrp_bom_cost text-right">
                    <span t-esc="data['operations_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                </td>
                <td/>
            </tr>
        </t>
    </template>

    <template id="report_mrp_operation_line">
      <t t-set="currency" t-value="data['currency']"/>
      <t t-foreach="data['operations']" t-as="op">
          <t t-set="space_td" t-value="'margin-left: '+ str(op['level'] * 20) + 'px;'"/>
          <tr class="o_mrp_bom_report_line o_mrp_bom_cost" t-att-parent_id="'operation-' + str(data['bom_id'])">
              <td name="td_opr_line">
                  <span t-att-style="space_td"/>
                  <a href="#" t-att-data-res-id="op['operation'].id" t-att-data-model="'mrp.routing.workcenter'" class="o_mrp_bom_action"><t t-esc="op['name']"/></a>
              </td>
              <td/>
              <td class="text-right">
                  <span t-esc="op['duration_expected']" t-options='{"widget": "float_time"}'/>
              </td>
              <td groups="uom.group_uom"><span>Minutes</span></td>
              <td class="o_mrp_prod_cost"></td>
              <td class="o_mrp_bom_cost text-right">
                  <span t-esc="op['total']" t-options='{"widget": "monetary", "display_currency": currency}'/>
              </td>
              <td/>
          </tr>
      </t>
    </template>

    <template id="report_mrp_bom_pdf">
        <t t-call="web.html_container">
            <t t-call="mrp.report_mrp_bom"/>
        </t>
    </template>

    <template id="report_mrp_bom_pdf_line">
      <t t-set="currency" t-value="data['currency']"/>
      <t t-foreach="data['lines']" t-as="l">
          <t t-set="space_td" t-value="'margin-left: '+ str(l['level'] * 20) + 'px;'"/>
          <tr t-if="data['report_structure'] != 'bom_structure' or l['type'] != 'operation'">
              <td>
                  <div t-att-style="space_td">
                    <div><t t-esc="l['name']"/></div>
                  </div>
              </td>
              <td name="td_mrp_code">
                  <div t-if="l.get('code')" t-esc="l['code']" />
              </td>
              <td class="text-right">
                  <span>
                      <t t-if="l['type'] == 'operation'" t-esc="l['quantity']" t-options='{"widget": "float_time"}'/>
                      <t t-if="l['type'] == 'bom'" t-esc="l['quantity']" t-options='{"widget": "float", "decimal_precision": "Product Unit of Measure"}'/>
                  </span>
              </td>
              <td groups="uom.group_uom"><span><t t-esc="l['uom']"/></span></td>
              <td t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_prod_cost text-right">
                  <span t-if="'prod_cost' in l" t-esc="l['prod_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
              </td>
              <td t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_bom_cost text-right">
                  <span t-esc="l['bom_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
              </td>
          </tr>
      </t>
    </template>

    <template id="report_bom_structure">
        <t t-set="data_report_landscape" t-value="True"/>
        <t t-call="web.basic_layout">
            <t t-call-assets="mrp.assets_common" t-js="False"/>
            <t t-foreach="docs" t-as="data">
                <div class="page">
                    <t t-call="mrp.report_mrp_bom"/>
                </div>
                <p style="page-break-before:always;"> </p>
            </t>
        </t>
    </template>
</odoo>

```

## File: report\mrp_report_views_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <report 
            string="Production Order" 
            id="action_report_production_order" 
            model="mrp.production" 
            name="mrp.report_mrporder"
            file="mrp.report.mrp_production_templates" 
            report_type="qweb-pdf"
            print_report_name="'Production Order - %s' % object.name"
        />
        <report 
            string="BoM Structure" 
            id="action_report_bom_structure" 
            model="mrp.bom" 
            name="mrp.report_bom_structure"
            file="mrp.report_bom_structure"
            report_type="qweb-pdf"
            print_report_name="'Bom Structure - %s' % object.display_name"
        />
        <report id="label_manufacture_template"
            model="mrp.production"
            string="Finished Product Label (ZPL)"
            name="mrp.label_production_view"
            file="mrp.label_production_view"
            report_type="qweb-text"
        />
        <report
            string="Finished Product Label (PDF)"
            id="action_report_finished_product"
            model="mrp.production"
            name="mrp.label_production_view_pdf"
            file="mrp.label_production_view_pdf"
            report_type="qweb-pdf"
            print_report_name="'Finished products - %s' % object.name"
        />
    </data>
</odoo>

```

## File: report\mrp_zebra_production_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <template id="label_production_view">
            <t t-foreach="docs" t-as="production">
                <t t-foreach="production.move_finished_ids" t-as="move">
                    <t t-foreach="move.move_line_ids" t-as="move_line">
                        <t t-if="move_line.product_uom_id.category_id.measure_type == 'unit'">
                            <t t-set="qty" t-value="int(move_line.qty_done)"/>
                        </t>
                        <t t-else="">
                            <t t-set="qty" t-value="1"/>
                        </t>
                        <t t-foreach="range(qty)" t-as="item">
                            <t t-translation="off">
^XA
^FO100,50
^A0N,44,33^FD<t t-esc="move_line.product_id.display_name"/>^FS
<t t-if="move_line.product_id.tracking != 'none' and move_line.lot_id">
^FO100,100
^A0N,44,33^FDLN/SN: <t t-esc="move_line.lot_id.name"/>^FS
^FO100,150^BY3
^BCN,100,Y,N,N
^FD<t t-esc="move_line.lot_id.name"/>^FS
^XZ
</t>
<t t-if="move_line.product_id.tracking == 'none' and move_line.product_id.barcode">
^FO100,100^BY3
^BCN,100,Y,N,N
^FD<t t-esc="move_line.product_id.barcode"/>^FS
^XZ
</t>
                            </t>
                        </t>
                    </t>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

## File: report\report_stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ReportStockRule(models.AbstractModel):
    _inherit = 'report.stock.report_stock_rule'

    @api.model
    def _get_rule_loc(self, rule, product_id):
        """ We override this method to handle manufacture rule which do not have a location_src_id.
        """
        res = super(ReportStockRule, self)._get_rule_loc(rule, product_id)
        if rule.action == 'manufacture':
            res['source'] = product_id.property_stock_production
        return res

```

## File: report\report_stock_rule.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <template id="mrp_report_stock_rule" inherit_id="stock.report_stock_rule">
        <xpath expr="//div[hasclass('o_report_stock_rule_rule')]/t" position="before">
            <t t-if="rule[0].action == 'manufacture'">
                <t t-if="rule[1] == 'origin'">
                    <t t-call="stock.report_stock_rule_left_arrow"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_rule')]/t[last()]" position="after">
            <t t-if="rule[0].action == 'manufacture'">
                <t t-if="rule[1] == 'destination'">
                    <t t-call="stock.report_stock_rule_right_arrow"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_rule_name')]/span" position="before">
            <t t-if="rule[0].action == 'manufacture'">
                <i class="fa fa-wrench fa-fw" t-attf-style="color: #{color};"/>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_legend')]" position="inside">
            <div class="o_report_stock_rule_legend_line">
                <div class="o_report_stock_rule_legend_label">Manufacture</div>
                <div class="o_report_stock_rule_legend_symbol">
                    <div class="fa fa-wrench fa-fw" t-attf-style="color: #{color};"/>
                </div>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_report_bom_structure
from . import report_stock_rule

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mrp_workcenter_productivity_loss_manager,mrp.workcenter.productivity.loss,model_mrp_workcenter_productivity_loss,mrp.group_mrp_manager,1,1,1,1
access_mrp_workcenter_productivity_loss,mrp.workcenter.productivity.loss,model_mrp_workcenter_productivity_loss,mrp.group_mrp_user,1,0,0,0
access_mrp_workcenter_productivity_loss_type,mrp.workcenter.productivity.loss.type,model_mrp_workcenter_productivity_loss_type,mrp.group_mrp_user,1,0,0,0
access_mrp_workcenter_productivity,mrp.workcenter.productivity,model_mrp_workcenter_productivity,mrp.group_mrp_user,1,1,1,1
access_mrp_workcenter,mrp.workcenter,model_mrp_workcenter,mrp.group_mrp_user,1,0,0,0
access_mrp_routing,mrp.routing,model_mrp_routing,mrp.group_mrp_user,1,0,0,0
access_mrp_routing_workcenter,mrp.routing.workcenter,model_mrp_routing_workcenter,mrp.group_mrp_user,1,0,0,0
access_mrp_bom,mrp.bom,model_mrp_bom,group_mrp_user,1,0,0,0
access_mrp_bom_line,mrp.bom.line,model_mrp_bom_line,group_mrp_user,1,0,0,0
access_mrp_bom_byproduct_user,mrp.bom.byproduct,model_mrp_bom_byproduct,mrp.group_mrp_user,1,0,0,0
access_mrp_production,mrp.production user,model_mrp_production,mrp.group_mrp_user,1,1,1,1
access_mrp_workcenter_manager,mrp.workcenter.manager,model_mrp_workcenter,mrp.group_mrp_manager,1,1,1,1
access_mrp_resource_manager,access_mrp_resource manager,resource.model_resource_resource,mrp.group_mrp_manager,1,1,1,1
access_mrp_routing_manager,mrp.routing.manager,model_mrp_routing,mrp.group_mrp_manager,1,1,1,1
access_mrp_routing_workcenter_manager,mrp.routing.workcenter.manager,model_mrp_routing_workcenter,mrp.group_mrp_manager,1,1,1,1
access_mrp_bom_manager,mrp.bom.manager,model_mrp_bom,mrp.group_mrp_manager,1,1,1,1
access_mrp_bom_line_manager,mrp.bom.line.manager,model_mrp_bom_line,mrp.group_mrp_manager,1,1,1,1
access_mrp_bom_byproduct_manager,mrp.bom.byproduct manager,model_mrp_bom_byproduct,mrp.group_mrp_manager,1,1,1,1
access_stock_location_mrp_worker,stock.location mrp_worker,stock.model_stock_location,mrp.group_mrp_user,1,0,0,0
access_stock_move_mrp_worker,stock.move mrp_worker,stock.model_stock_move,mrp.group_mrp_user,1,1,1,0
access_stock_picking_mrp_worker,stock.picking mrp_worker,stock.model_stock_picking,mrp.group_mrp_user,1,1,1,1
access_stock_warehouse,stock.warehouse mrp_worker,stock.model_stock_warehouse,mrp.group_mrp_user,1,0,0,0
access_mrp_production_stock_worker,mrp.production stock_worker,model_mrp_production,stock.group_stock_user,1,0,0,0
access_product_product_user,product.product user,product.model_product_product,mrp.group_mrp_user,1,0,0,0
access_product_template_user,product.template user,product.model_product_template,mrp.group_mrp_user,1,0,0,0
access_uom_uom_user,uom.uom user,uom.model_uom_uom,mrp.group_mrp_user,1,0,0,0
access_product_supplierinfo_user,product.supplierinfo user,product.model_product_supplierinfo,mrp.group_mrp_user,1,1,1,1
access_res_partner,res.partner,base.model_res_partner,mrp.group_mrp_user,1,0,0,0
access_mrp_workorder_mrp_user,mrp.workorder.user,model_mrp_workorder,mrp.group_mrp_user,1,1,1,1
access_mrp_workorder_mrp_manager,mrp.workorder,model_mrp_workorder,mrp.group_mrp_manager,1,1,1,1
access_mrp_workorder_line_mrp_user,mrp.workorder.user,model_mrp_workorder_line,mrp.group_mrp_user,1,1,1,1
access_mrp_workorder_line_mrp_manager,mrp.workorder,model_mrp_workorder_line,mrp.group_mrp_manager,1,1,1,1
access_resource_calendar_leaves_user,mrp.resource.calendar.leaves.user,resource.model_resource_calendar_leaves,mrp.group_mrp_user,1,1,1,1
access_resource_calendar_leaves_manager,mrp.resource.calendar.leaves.manager,resource.model_resource_calendar_leaves,mrp.group_mrp_manager,1,0,0,0
access_resource_calendar_attendance_mrp_user,mrp.resource.calendar.attendance.mrp.user,resource.model_resource_calendar_attendance,mrp.group_mrp_user,1,1,1,1
access_resource_calendar_attendance_manager,mrp.resource.calendar.attendance.manager,resource.model_resource_calendar_attendance,mrp.group_mrp_manager,1,1,1,1
access_uom_category,uom.category,uom.model_uom_category,mrp.group_mrp_user,1,0,0,0
access_resource_resource,resource.resource,resource.model_resource_resource,mrp.group_mrp_user,1,0,0,0
access_resource_resource_manager,resource.resource.manager,resource.model_resource_resource,mrp.group_mrp_manager,1,1,1,1
access_product_supplierinfo_manager,product.supplierinfo user,product.model_product_supplierinfo,mrp.group_mrp_manager,1,0,0,0
access_mrp_production_manager,mrp.production manager,model_mrp_production,mrp.group_mrp_manager,1,0,0,0
access_stock_move_mrp_manager,stock.move mrp_manager,stock.model_stock_move,mrp.group_mrp_manager,1,0,0,0
access_stock_production_lot_user,stock.production.lot,stock.model_stock_production_lot,mrp.group_mrp_user,1,1,1,1
access_stock_warehouse_orderpoint_user,stock.warehouse.orderpoint,stock.model_stock_warehouse_orderpoint,mrp.group_mrp_user,1,0,0,0
access_stock_picking_mrp_manager,stock.picking mrp_manager,stock.model_stock_picking,mrp.group_mrp_manager,1,0,0,0
access_mrp_bom_stockuser,mrp.bom,model_mrp_bom,stock.group_stock_user,1,0,0,0
access_mrp_bom_line_stockuser,mrp.bom.line,model_mrp_bom_line,stock.group_stock_user,1,0,0,0
access_uom_category_mrp_manager,uom.category mrp_manager,uom.model_uom_category,mrp.group_mrp_manager,1,1,1,1
access_uom_uom_mrp_manager,uom.uom mrp_manager,uom.model_uom_uom,mrp.group_mrp_manager,1,1,1,1
access_product_category_mrp_manager,product.category mrp_manager,product.model_product_category,mrp.group_mrp_manager,1,1,1,1
access_product_template_mrp_manager,product.template mrp_manager,product.model_product_template,mrp.group_mrp_manager,1,1,1,1
access_product_product_mrp_manager,product.product mrp_manager,product.model_product_product,mrp.group_mrp_manager,1,1,1,1
access_product_packaging_mrp_manager,product.packaging mrp_manager,product.model_product_packaging,mrp.group_mrp_manager,1,1,1,1
access_product_pricelist_mrp_manager,product.pricelist mrp_manager,product.model_product_pricelist,mrp.group_mrp_manager,1,1,1,1
access_product_group_res_partner_mrp_manager,res_partner group_mrp_manager,base.model_res_partner,mrp.group_mrp_manager,1,1,1,0
access_product_pricelist_item_mrp_manager,product.pricelist.item mrp_manager,product.model_product_pricelist_item,mrp.group_mrp_manager,1,1,1,1
access_resource_calendar_manufacturinguser,resource.calendar manufacturing.user,resource.model_resource_calendar,mrp.group_mrp_user,1,0,0,0
access_mrp_unbuild,mrp.unbuild,model_mrp_unbuild,group_mrp_user,1,1,1,1
access_mrp_unbuild_manager,mrp.unbuild manager,model_mrp_unbuild,group_mrp_manager,1,1,1,1
access_mrp_document_mrp_manager,mrp.document group_user,model_mrp_document,group_mrp_manager,1,1,1,1
access_mrp_document_mrp_user,mrp.document group_user,model_mrp_document,group_mrp_user,1,1,1,1

```

## File: security\mrp_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">

    <record id="base.module_category_manufacturing_manufacturing" model="ir.module.category">
        <field name="description">Helps you manage your manufacturing processes and generate reports on those processes.</field>
        <field name="sequence">5</field>
    </record>

    <record id="group_mrp_user" model="res.groups">
        <field name="name">User</field>
        <field name="implied_ids" eval="[(4, ref('stock.group_stock_user'))]"/>
        <field name="category_id" ref="base.module_category_manufacturing_manufacturing"/>
    </record>
    <record id="group_mrp_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_manufacturing_manufacturing"/>
        <field name="implied_ids" eval="[(4, ref('group_mrp_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>


    <record id="group_mrp_routings" model="res.groups">
        <field name="name">Manage Work Order Operations</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_mrp_byproducts" model="res.groups">
        <field name="name">Produce residual products</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

</data>
<data noupdate="1">
    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('mrp.group_mrp_manager'))]"/>
    </record>
<!-- Multi -->
    <record model="ir.rule" id="mrp_production_rule">
        <field name="name">mrp_production multi-company</field>
        <field name="model_id" search="[('model','=','mrp.production')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="mrp_unbuild_rule">
        <field name="name">mrp_unbuild multi-company</field>
        <field name="model_id" search="[('model','=','mrp.unbuild')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="mrp_workcenter_rule">
        <field name="name">mrp_workcenter multi-company</field>
        <field name="model_id" search="[('model','=','mrp.workcenter')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id', 'in', company_ids),('company_id','=',False)]</field>
    </record>

    <record model="ir.rule" id="mrp_workorder_rule">
        <field name="name">mrp_workorder multi-company</field>
        <field name="model_id" search="[('model','=','mrp.workorder')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="mrp_bom_rule">
        <field name="name">mrp_bom multi-company</field>
        <field name="model_id" search="[('model','=','mrp.bom')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id', 'in', company_ids),('company_id','=',False)]</field>
    </record>

    <record model="ir.rule" id="mrp_bom_line_rule">
        <field name="name">mrp_bom_line multi-company</field>
        <field name="model_id" search="[('model','=','mrp.bom.line')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id', 'in', company_ids),('company_id','=',False)]</field>
    </record>

    <record model="ir.rule" id="mrp_bom_byproduct_rule">
        <field name="name">mrp_bom_byproduct multi-company</field>
        <field name="model_id" search="[('model','=','mrp.bom.byproduct')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id', 'in', company_ids),('company_id','=',False)]</field>
    </record>

    <record model="ir.rule" id="mrp_routing_rule">
        <field name="name">mrp_routing multi-company</field>
        <field name="model_id" search="[('model','=','mrp.routing')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id', 'in', company_ids),('company_id','=',False)]</field>
    </record>

    <record model="ir.rule" id="mrp_routing_workcenter_rule">
        <field name="name">mrp_routing_workcenter multi-company</field>
        <field name="model_id" search="[('model','=','mrp.routing.workcenter')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id', 'in', company_ids),('company_id','=',False)]</field>
    </record>

    <record model="ir.rule" id="mrp_workcenter_productivity">
        <field name="name">mrp_workcenter_productivity multi-company</field>
        <field name="model_id" search="[('model','=','mrp.workcenter.productivity')]" model="ir.model"/>
        <field name="global" eval="True"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M53.373 32.454c.742 0 1.206.784.824 1.4-2.009 3.235-5.668 5.4-9.848 5.4-6.318 0-11.445-4.944-11.484-11.056C32.825 22.058 38.012 17 44.349 17c4.176 0 7.831 2.16 9.842 5.389.385.62-.07 1.41-.818 1.41h-8.386l-3.19 4.328 3.19 4.327h8.386zM39.684 39.64l-15.97 15.473c-1.994 1.931-5.226 1.931-7.22 0a4.837 4.837 0 0 1 0-6.994l15.971-15.472c1.289 3.183 3.932 5.744 7.219 6.993zm-16.39 10.74c0-1.024-.857-1.854-1.914-1.854s-1.914.83-1.914 1.854.857 1.854 1.914 1.854 1.914-.83 1.914-1.854z"/><path id="e" d="M53.373 30.454c.742 0 1.206.784.824 1.4-2.009 3.235-5.668 5.4-9.848 5.4-6.318 0-11.445-4.944-11.484-11.056C32.825 20.058 38.012 15 44.349 15c4.176 0 7.831 2.16 9.842 5.389.385.62-.07 1.41-.818 1.41h-8.386l-3.19 4.328 3.19 4.327h8.386zM39.684 37.64l-15.97 15.473c-1.994 1.931-5.226 1.931-7.22 0a4.837 4.837 0 0 1 0-6.994l15.971-15.472c1.289 3.183 3.932 5.744 7.219 6.993zm-16.39 10.74c0-1.024-.857-1.854-1.914-1.854s-1.914.83-1.914 1.854.857 1.854 1.914 1.854 1.914-.83 1.914-1.854z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V36.075l15.053-15.2 18.452.218 2.734 6.93v4.524L53.62 51.98 42.667 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" transform="matrix(-1 0 0 1 69.334 0)" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" transform="matrix(-1 0 0 1 69.334 0)" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\mrp.js

```javascript
odoo.define('mrp.mrp_state', function (require) {
"use strict";

var AbstractField = require('web.AbstractField');
var core = require('web.core');
var fields = require('web.basic_fields');
var field_registry = require('web.field_registry');
var time = require('web.time');

var _t = core._t;

/**
 * This widget is used to display the availability on a workorder.
 */
var SetBulletStatus = AbstractField.extend({
    // as this widget is based on hardcoded values, use it in another context
    // probably won't work
    // supportedFieldTypes: ['selection'],
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.classes = this.nodeOptions && this.nodeOptions.classes || {};
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
    _renderReadonly: function () {
        this._super.apply(this, arguments);
        var bullet_class = this.classes[this.value] || 'default';
        if (this.value) {
            var title = this.value === 'waiting' ? _t('Waiting Materials') : '';
            this.$el.attr({'title': title, 'style': 'display:inline'});
            this.$el.removeClass('text-success text-danger text-default');
            this.$el.html($('<span>' + title + '</span>').addClass('badge badge-' + bullet_class));
        }
    }
});

var TimeCounter = AbstractField.extend({
    supportedFieldTypes: [],
    /**
     * @override
     */
    willStart: function () {
        var self = this;
        var def = this._rpc({
            model: 'mrp.workcenter.productivity',
            method: 'search_read',
            domain: [
                ['workorder_id', '=', this.record.data.id],
            ],
        }).then(function (result) {
            if (self.mode === 'readonly') {
                var currentDate = new Date();
                self.duration = 0;
                _.each(result, function (data) {
                    self.duration += data.date_end ?
                        self._getDateDifference(data.date_start, data.date_end) :
                        self._getDateDifference(time.auto_str_to_date(data.date_start), currentDate);
                });
            }
        });
        return Promise.all([this._super.apply(this, arguments), def]);
    },

    destroy: function () {
        this._super.apply(this, arguments);
        clearTimeout(this.timer);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    isSet: function () {
        return true;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Compute the difference between two dates.
     *
     * @private
     * @param {string} dateStart
     * @param {string} dateEnd
     * @returns {integer} the difference in millisecond
     */
    _getDateDifference: function (dateStart, dateEnd) {
        return moment(dateEnd).diff(moment(dateStart));
    },
    /**
     * @override
     */
    _render: function () {
        this._startTimeCounter();
    },
    /**
     * @private
     */
    _startTimeCounter: function () {
        var self = this;
        clearTimeout(this.timer);
        if (this.record.data.is_user_working) {
            this.timer = setTimeout(function () {
                self.duration += 1000;
                self._startTimeCounter();
            }, 1000);
        } else {
            clearTimeout(this.timer);
        }

        const duration = moment.duration(this.duration, 'milliseconds');
        const hours = Math.floor(duration.asHours());
        const minutes = duration.minutes();
        const seconds = duration.seconds();
        //display the variables in two-character format, e.g: 09 intead of 9
        const text = _.str.sprintf("%02d:%02d:%02d", hours, minutes, seconds);
        const $mrpTimeText = $('<span>', { text });
        this.$el.empty().append($mrpTimeText);
    },
});

var FieldEmbedURLViewer = fields.FieldChar.extend({

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.page = 1;
        this.srcDirty = false;
    },

    /**
     * force to set 'src' for embed iframe viewer when its value has changed
     *
     * @override
     *
     */
    reset: function () {
        this._super.apply(this, arguments);
        this._updateIframePreview();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Initializes and returns an iframe for the viewer
     *
     * @private
     * @returns {jQueryElement}
     */
    _prepareIframe: function () {
        return $('<iframe>', {
            class: 'o_embed_iframe d-none',
            allowfullscreen: true,
        });
    },

    /**
     * @override
     * @private
     */
    _renderEdit: function () {
        if (!this.$('iframe.o_embed_iframe').length) {
            this.$input = this.$el;
            this.setElement(this.$el.wrap('<div class="o_embed_url_viewer o_field_widget"/>').parent());
            this.$el.append(this._prepareIframe());
        }
        this._prepareInput(this.$input);

        // Do not set iframe src if widget is invisible
        if (!this.record.evalModifiers(this.attrs.modifiers).invisible) {
            this._updateIframePreview();
        } else {
            this.srcDirty = true;
        }
    },
    /**
     * @override
     * @private
     */
    _renderReadonly: function () {
        if (!this.$('iframe.o_embed_iframe').length) {
            this.$el.addClass('o_embed_url_viewer');
            this.$el.append(this._prepareIframe());
        }
        this._updateIframePreview();
    },
    /**
     * Set the associated src for embed iframe viewer
     *
     * @private
     * @returns {string} source of the google slide
     */
    _getEmbedSrc: function () {
        var src = false;
        if (this.value) {
            // check given google slide url is valid or not
            var googleRegExp = /(^https:\/\/docs.google.com).*(\/d\/e\/|\/d\/)([A-Za-z0-9-_]+)/;
            var google = this.value.match(googleRegExp);
            if (google && google[3]) {
                src = 'https://docs.google.com/presentation' + google[2] + google[3] + '/preview?slide=' + this.page;
            }
        }
        return src || this.value;
    },
    /**
     * update iframe attrs
     *
     * @private
     */
    _updateIframePreview: function () {
        var $iframe = this.$('iframe.o_embed_iframe');
        var src = this._getEmbedSrc();
        $iframe.toggleClass('d-none', !src);
        if (src) {
            $iframe.attr('src', src);
        } else {
            $iframe.removeAttr('src');
        }
    },
    /**
     * Listen to modifiers updates to and only render iframe when it is necessary
     *
     * @override
     */
    updateModifiersValue: function () {
        this._super.apply(this, arguments);
        if (!this.attrs.modifiersValue.invisible && this.srcDirty) {
            this._updateIframePreview();
            this.srcDirty = false;
        }
    },
});


field_registry
    .add('bullet_state', SetBulletStatus)
    .add('mrp_time_counter', TimeCounter)
    .add('embed_viewer', FieldEmbedURLViewer);

return FieldEmbedURLViewer;
});

```

## File: static\src\js\mrp_bom_report.js

```javascript
odoo.define('mrp.mrp_bom_report', function (require) {
'use strict';

var core = require('web.core');
var framework = require('web.framework');
var stock_report_generic = require('stock.stock_report_generic');

var QWeb = core.qweb;
var _t = core._t;

var MrpBomReport = stock_report_generic.extend({
    events: {
        'click .o_mrp_bom_unfoldable': '_onClickUnfold',
        'click .o_mrp_bom_foldable': '_onClickFold',
        'click .o_mrp_bom_action': '_onClickAction',
        'click .o_mrp_show_attachment_action': '_onClickShowAttachment',
    },
    get_html: function() {
        var self = this;
        var args = [
            this.given_context.active_id,
            this.given_context.searchQty || false,
            this.given_context.searchVariant,
        ];
        return this._rpc({
                model: 'report.mrp.report_bom_structure',
                method: 'get_html',
                args: args,
                context: this.given_context,
            })
            .then(function (result) {
                self.data = result;
            });
    },
    set_html: function() {
        var self = this;
        return this._super().then(function () {
            self.$('.o_content').html(self.data.lines);
            self.renderSearch();
            self.update_cp();
        });
    },
    render_html: function(event, $el, result){
        if (result.indexOf('mrp.document') > 0) {
            if (this.$('.o_mrp_has_attachments').length === 0) {
                var column = $('<th/>', {
                    class: 'o_mrp_has_attachments',
                    title: 'Files attached to the product Attachments',
                    text: 'Attachments',
                });
                this.$('table thead th:last-child').after(column);
            }
        }
        $el.after(result);
        $(event.currentTarget).toggleClass('o_mrp_bom_foldable o_mrp_bom_unfoldable fa-caret-right fa-caret-down');
        this._reload_report_type();
    },
    get_bom: function(event) {
      var self = this;
      var $parent = $(event.currentTarget).closest('tr');
      var activeID = $parent.data('id');
      var productID = $parent.data('product_id');
      var lineID = $parent.data('line');
      var qty = $parent.data('qty');
      var level = $parent.data('level') || 0;
      return this._rpc({
              model: 'report.mrp.report_bom_structure',
              method: 'get_bom',
              args: [
                  activeID,
                  productID,
                  parseFloat(qty),
                  lineID,
                  level + 1,
              ]
          })
          .then(function (result) {
              self.render_html(event, $parent, result);
          });
    },
    get_operations: function(event) {
      var self = this;
      var $parent = $(event.currentTarget).closest('tr');
      var activeID = $parent.data('bom-id');
      var qty = $parent.data('qty');
      var level = $parent.data('level') || 0;
      return this._rpc({
              model: 'report.mrp.report_bom_structure',
              method: 'get_operations',
              args: [
                  activeID,
                  parseFloat(qty),
                  level + 1
              ]
          })
          .then(function (result) {
              self.render_html(event, $parent, result);
          });
    },
    update_cp: function () {
        var status = {
            cp_content: {
                $buttons: this.$buttonPrint,
                $searchview_buttons: this.$searchView
            },
        };
        return this.updateControlPanel(status);
    },
    renderSearch: function () {
        this.$buttonPrint = $(QWeb.render('mrp.button'));
        this.$buttonPrint.find('.o_mrp_bom_print').on('click', this._onClickPrint.bind(this));
        this.$buttonPrint.find('.o_mrp_bom_print_unfolded').on('click', this._onClickPrint.bind(this));
        this.$searchView = $(QWeb.render('mrp.report_bom_search', _.omit(this.data, 'lines')));
        this.$searchView.find('.o_mrp_bom_report_qty').on('change', this._onChangeQty.bind(this)).change();
        this.$searchView.find('.o_mrp_bom_report_variants').on('change', this._onChangeVariants.bind(this)).change();
        this.$searchView.find('.o_mrp_bom_report_type').on('change', this._onChangeType.bind(this));
    },
    _onClickPrint: function (ev) {
        var childBomIDs = _.map(this.$el.find('.o_mrp_bom_foldable').closest('tr'), function (el) {
            return $(el).data('id');
        });
        framework.blockUI();
        var reportname = 'mrp.report_bom_structure?docids=' + this.given_context.active_id +
                         '&report_type=' + this.given_context.report_type +
                         '&quantity=' + (this.given_context.searchQty || 1);
        if (! $(ev.currentTarget).hasClass('o_mrp_bom_print_unfolded')) {
            reportname += '&childs=' + JSON.stringify(childBomIDs);
        }
        if (this.given_context.searchVariant) {
            reportname += '&variant=' + this.given_context.searchVariant;
        }
        var action = {
            'type': 'ir.actions.report',
            'report_type': 'qweb-pdf',
            'report_name': reportname,                                
            'report_file': 'mrp.report_bom_structure',
        };
        return this.do_action(action).then(function (){
            framework.unblockUI();
        });
    },
    _onChangeQty: function (ev) {
        var qty = $(ev.currentTarget).val().trim();
        if (qty) {
            this.given_context.searchQty = parseFloat(qty);
            this._reload();
        }
    },
    _onChangeType: function (ev) {
        var report_type = $("option:selected", $(ev.currentTarget)).data('type');
        this.given_context.report_type = report_type;
        this._reload_report_type();
    },
    _onChangeVariants: function (ev) {
        this.given_context.searchVariant = $(ev.currentTarget).val();
        this._reload();
    },
    _onClickUnfold: function (ev) {
        var redirect_function = $(ev.currentTarget).data('function');
        this[redirect_function](ev);
    },
    _onClickFold: function (ev) {
        this._removeLines($(ev.currentTarget).closest('tr'));
        $(ev.currentTarget).toggleClass('o_mrp_bom_foldable o_mrp_bom_unfoldable fa-caret-right fa-caret-down');
    },
    _onClickAction: function (ev) {
        ev.preventDefault();
        return this.do_action({
            type: 'ir.actions.act_window',
            res_model: $(ev.currentTarget).data('model'),
            res_id: $(ev.currentTarget).data('res-id'),
            context: {
                'active_id': $(ev.currentTarget).data('res-id')
            },
            views: [[false, 'form']],
            target: 'current'
        });
    },
    _onClickShowAttachment: function (ev) {
        ev.preventDefault();
        var ids = $(ev.currentTarget).data('res-id');
        return this.do_action({
            name: _t('Attachments'),
            type: 'ir.actions.act_window',
            res_model: $(ev.currentTarget).data('model'),
            domain: [['id', 'in', ids]],
            views: [[false, 'kanban'], [false, 'list'], [false, 'form']],
            view_mode: 'kanban,list,form',
            target: 'current',
        });
    },
    _reload: function () {
        var self = this;
        return this.get_html().then(function () {
            self.$('.o_content').html(self.data.lines);
            self._reload_report_type();
        });
    },
    _reload_report_type: function () {
        this.$('.o_mrp_bom_cost.o_hidden, .o_mrp_prod_cost.o_hidden').toggleClass('o_hidden');
        if (this.given_context.report_type === 'bom_structure') {
           this.$('.o_mrp_bom_cost, .o_mrp_prod_cost').toggleClass('o_hidden');
        }
    },
    _removeLines: function ($el) {
        var self = this;
        var activeID = $el.data('id');
        _.each(this.$('tr[parent_id='+ activeID +']'), function (parent) {
            var $parent = self.$(parent);
            var $el = self.$('tr[parent_id='+ $parent.data('id') +']');
            if ($el.length) {
                self._removeLines($parent);
            }
            $parent.remove();
        });
    },
});

core.action_registry.add('mrp_bom_report', MrpBomReport);
return MrpBomReport;

});

```

## File: static\src\xml\mrp.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="mrp.button">
        <div class="o_list_buttons o_mrp_bom_report_buttons">
            <button type="button" class="btn btn-primary o_mrp_bom_print">Print</button>
            <button type="button" class="btn btn-primary o_mrp_bom_print_unfolded">Print Unfolded</button>
        </div>
    </t>

    <form class="form-inline" t-name="mrp.report_bom_search">
        <div class="form-group col-lg-4">
            <label>Quantity:</label>
            <div class="row">
                <div class="col-lg-6">
                    <input type="number" step="any" t-att-value="bom_qty" min="1" class="o_input o_mrp_bom_report_qty"/>
                </div>
                <div class="col-lg-6">
                    <t t-if="is_uom_applied" t-esc="bom_uom_name"/>
                </div>
            </div>
        </div>
        <div t-if="is_variant_applied" class="form-group col-lg-4">
            <label>Variant:</label>
            <select class="o_input o_mrp_bom_report_variants">
                <option t-foreach="variants" t-as="variant" t-att-value="variant">
                    <t t-esc="variants[variant]"/>
                </option>
            </select>
        </div>
        <div t-attf-class="form-group #{is_variant_applied ? 'col-lg-4' : 'col-lg-8'}">
            <label>Report:</label>
            <select class="o_input o_mrp_bom_report_type">
                <option t-att-data-type="'all'">BoM Structure &amp; Cost</option>
                <option t-att-data-type="'bom_structure'">BoM Structure</option>
            </select>
        </div>
    </form>
</templates>

```

## File: views\ir_attachment_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Files -->
    <record model="ir.ui.view" id="view_document_file_kanban_mrp">
        <field name="name">mrp.document kanban.mrp</field>
        <field name="model">mrp.document</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="ir_attachment_id"/>
                <field name="mimetype"/>
                <field name="type"/>
                <field name="name"/>
                <field name="priority"/>
                <field name="create_uid"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_area o_kanban_attachment oe_kanban_global_click">
                            <div class="o_kanban_image">
                                <div class="o_kanban_image_wrapper">
                                    <t t-set="webimage" t-value="new RegExp('image.*(gif|jpeg|jpg|png)').test(record.mimetype.value)"/>
                                    <div t-if="record.type.raw_value == 'url'" class="o_url_image fa fa-link fa-3x text-muted"/>
                                    <img t-elif="webimage" t-attf-src="/web/image/#{record.ir_attachment_id.raw_value}" width="100" height="100" alt="Document" class="o_attachment_image"/>
                                    <div t-else="!webimage" class="o_image o_image_thumbnail" t-att-data-mimetype="record.mimetype.value"/>
                                </div>
                            </div>
                            <div class="o_kanban_details">
                                <div class="o_kanban_details_wrapper">
                                    <div class="o_kanban_record_title">
                                        <field name="name" class="o_text_overflow"/>
                                    </div>
                                    <div class="o_kanban_record_body">
                                      <field name="name" attrs="{'invisible':[('type','=','url')]}"/>
                                      <field name="url" widget="url" attrs="{'invisible':[('type','=','binary')]}"/>
                                    </div>
                                    <div class="o_kanban_record_bottom">
                                        <span class="oe_kanban_bottom_left">
                                            <field name="priority" widget="priority"/>
                                        </span>
                                        <div class="oe_kanban_bottom_right">
                                            <img t-att-src="kanban_image('res.users', 'image_128', record.create_uid.raw_value)" t-att-data-member_id="record.create_uid.raw_value" t-att-alt="record.create_uid.raw_value" class="oe_kanban_avatar"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="view_document_form" model="ir.ui.view">
        <field name="name">mrp.document.form</field>
        <field name="model">mrp.document</field>
        <field name="inherit_id" ref="base.view_attachment_form"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
          <xpath expr="//field[@name='mimetype']" position="after">
              <field name="priority" widget="priority"/>
              <field name="company_id" groups="base.group_multi_company"/>
          </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mrp_bom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Bill of Materials -->
        <record id="action_report_mrp_bom" model="ir.actions.client">
            <field name="name">BoM Structure &amp; Cost</field>
            <field name="tag">mrp_bom_report</field>
            <field name="context" eval="{'model': 'report.mrp.report_bom_structure'}" />
        </record>

        <record id="mrp_bom_byproduct_form_view" model="ir.ui.view">
            <field name="name">mrp.bom.byproduct.form</field>
            <field name="model">mrp.bom.byproduct</field>
            <field name="arch" type="xml">
                <form string="Byproduct">
                    <group>
                        <field name="routing_id" invisible="1"/>
                        <field name="company_id"/>
                        <field name="product_id"/>
                        <label for="product_qty"/>
                        <div class="o_row">
                            <field name="product_qty"/>
                            <field name="product_uom_id" groups="uom.group_uom"/>
                        </div>
                        <field name="operation_id" groups="mrp.group_mrp_routings" options="{'no_quick_create':True,'no_create_edit':True}"/>
                    </group>
                </form>
            </field>
        </record>

        <record id="mrp_bom_form_view" model="ir.ui.view">
            <field name="name">mrp.bom.form</field>
            <field name="model">mrp.bom</field>
            <field name="priority">100</field>
            <field name="arch" type="xml">
                <form string="Bill of Material">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="%(action_report_mrp_bom)d" type="action"
                                class="oe_stat_button" icon="fa-bars" string="Structure &amp; Cost"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="product_tmpl_id" context="{'default_type': 'product'}"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <field name="product_id" groups="product.group_product_variant" context="{'default_type': 'product'}"/>
                            <label for="product_qty" string="Quantity"/>
                            <div class="o_row">
                                <field name="product_qty"/>
                                <field name="product_uom_id" options="{'no_open':True,'no_create':True}" groups="uom.group_uom"/>
                            </div>
                            <field name="routing_id" attrs="{'invisible': [('type','not in',('normal','phantom'))]}" groups="mrp.group_mrp_routings" context="{'default_company_id': company_id}"/>
                        </group>
                        <group>
                            <field name="code"/>
                            <field name="type" widget="radio"/>
                            <p colspan="2" class="oe_grey oe_edit_only" attrs="{'invisible': [('type','!=','phantom')]}">
                            <ul>
                                A BoM of type kit is used to split the product into its components.
                                <li>
                                    At the creation of a Manufacturing Order.
                                </li>
                                <li>
                                    At the creation of a Stock Transfer.
                                </li>
                            </ul>
                            </p>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Components">
                            <field name="bom_line_ids" widget="one2many" context="{'default_parent_product_tmpl_id': product_tmpl_id, 'default_product_id': False, 'default_company_id': company_id, 'default_routing_id': routing_id}">
                                <tree string="Components" editable="bottom">
                                    <field name="company_id" invisible="1"/>
                                    <field name="routing_id" invisible="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="product_id" context="{'default_type': 'product'}"/>
                                    <field name="product_tmpl_id" invisible="1"/>
                                    <button name="action_see_attachments" type="object" icon="fa-files-o" aria-label="Product Attachments" title="Product Attachments" class="float-right oe_read_only"/>
                                    <field name="attachments_count" class="text-left oe_read_only"
                                    string=" "/>
                                    <field name="product_qty"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="parent_product_tmpl_id" invisible="1" />
                                    <field name="possible_bom_product_template_attribute_value_ids" invisible="1"/>
                                    <field name="product_uom_id" options="{'no_open':True,'no_create':True}" groups="uom.group_uom"/>
                                    <field name="bom_product_template_attribute_value_ids" widget="many2many_tags" options="{'no_create': True}" attrs="{'column_invisible': [('parent.product_id', '!=', False)]}" groups="product.group_product_variant"/>
                                    <field name="operation_id" groups="mrp.group_mrp_routings" attrs="{'column_invisible': [('parent.type','not in', ('normal', 'phantom'))]}" options="{'no_quick_create':True,'no_create_edit':True}"/>
                                </tree>
                            </field>
                        </page>
                        <page string="By-products" attrs="{'invisible': [('type','!=','normal')]}" groups="mrp.group_mrp_byproducts">
                            <field name="byproduct_ids"  context="{'form_view_ref' : 'mrp.mrp_bom_byproduct_form_view', 'default_company_id': company_id, 'default_routing_id': routing_id}">
                                <tree string="By-products"  editable="top">
                                    <field name="company_id" invisible="1"/>
                                    <field name="routing_id" invisible="1"/>
                                    <field name="product_id" context="{'default_type': 'product'}"/>
                                    <field name="product_qty"/>
                                    <field name="product_uom_id" groups="uom.group_uom"/>
                                    <field name="operation_id" groups="mrp.group_mrp_routings" options="{'no_quick_create':True,'no_create_edit':True}"/>
                                </tree>
                           </field>
                       </page>
                        <page string="Miscellaneous">
                            <group>
                                <group>
                                    <field name="ready_to_produce" attrs="{'invisible': [('type','!=','normal')]}" string="Manufacturing Readiness" groups="mrp.group_mrp_routings"/>
                                    <field name="consumption" attrs="{'invisible': [('type','=','phantom')], 'required': [('type','!=','phantom')]}"/>
                                </group>
                                <group>
                                    <field name="picking_type_id" attrs="{'invisible': [('type','=','phantom')]}" string="Operation" groups="stock.group_adv_location"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                    </sheet>
                    <div class="oe_chatter">
                         <field name="message_follower_ids" widget="mail_followers"/>
                         <field name="message_ids" colspan="4" widget="mail_thread" nolabel="1"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="mrp_bom_tree_view" model="ir.ui.view">
            <field name="name">mrp.bom.tree</field>
            <field name="model">mrp.bom</field>
            <field name="arch" type="xml">
                <tree string="Bill of Materials">
                    <field name="active" invisible="1"/>
                    <field name="sequence" widget="handle"/>
                    <field name="product_tmpl_id"/>
                    <field name="code" optional="show"/>
                    <field name="type"/>
                    <field name="product_id" groups="product.group_product_variant" optional="show"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show"/>
                    <field name="product_qty" optional="show"/>
                    <field name="product_uom_id" groups="uom.group_uom" optional="show" string="Unit of Measure"/>
                    <field name="routing_id" groups="mrp.group_mrp_routings" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="mrp_bom_kanban_view" model="ir.ui.view">
            <field name="name">mrp.bom.kanban</field>
            <field name="model">mrp.bom</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="product_tmpl_id"/>
                    <field name="product_qty"/>
                    <field name="product_uom_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings mt4">
                                        <strong class="o_kanban_record_title"><span clatt="mt4"><field name="product_tmpl_id"/></span></strong>
                                    </div>
                                    <span class="float-right badge badge-pill"><t t-esc="record.product_qty.value"/> <small><t t-esc="record.product_uom_id.value"/></small></span>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_mrp_bom_filter" model="ir.ui.view">
            <field name="name">mrp.bom.select</field>
            <field name="model">mrp.bom</field>
            <field name="arch" type="xml">
                <search string="Search Bill Of Material">
                    <field name="code" string="Bill of Materials" filter_domain="['|', ('code', 'ilike', self), ('product_tmpl_id', 'ilike', self)]"/>
                    <field name="product_tmpl_id" string="Product"/>
                    <field name="bom_line_ids" string="Component"/>
                    <filter string="Manufacturing" name="normal" domain="[('type', '=', 'normal')]"/>
                    <filter string="Kit" name="phantom" domain="[('type', '=', 'phantom')]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By...">
                        <filter string="Product" name="product" domain="[]" context="{'group_by': 'product_tmpl_id'}"/>
                        <filter string='BoM Type' name="group_by_type" domain="[]" context="{'group_by' : 'type'}"/>
                        <filter string='Unit of Measure' name="default_unit_of_measure" domain="[]" context="{'group_by' : 'product_uom_id'}"/>
                        <filter string="Routing" name="routings" domain="[]" context="{'group_by': 'routing_id'}"/>
                   </group>
                </search>
            </field>
        </record>

        <record id="mrp_bom_form_action" model="ir.actions.act_window">
            <field name="name">Bills of Materials</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.bom</field>
            <field name="domain">[]</field> <!-- force empty -->
            <field name="view_mode">tree,kanban,form</field>
            <field name="search_view_id" ref="view_mrp_bom_filter"/>
            <field name="context">{'search_default_group_by_type': True, 'default_company_id': allowed_company_ids[0]}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a bill of materials
              </p><p>
                Bills of materials allow you to define the list of required raw
                materials used to make a finished product; through a manufacturing
                order or a pack of products.
              </p>
            </field>
        </record>

        <menuitem id="menu_mrp_bom_form_action"
            action="mrp_bom_form_action"
            parent="menu_mrp_bom"
            sequence="13"/>

        <!-- BOM Line -->
        <record id="mrp_bom_line_view_form" model="ir.ui.view">
            <field name="name">mrp.bom.line.view.form</field>
            <field name="model">mrp.bom.line</field>
            <field name="arch" type="xml">
                <form string="Bill of Material line">
                    <group>
                        <group string="Product">
                            <field name="product_id"/>
                            <field name="parent_product_tmpl_id" invisible="1"/>
                            <label for="product_qty" string="Quantity"/>
                            <div class="o_row">
                                <field name="product_qty"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <field name="product_uom_id" options="{'no_open':True,'no_create':True}" groups="uom.group_uom"/>
                            </div>
                            <field name="possible_bom_product_template_attribute_value_ids" invisible="1"/>
                            <field name="bom_product_template_attribute_value_ids" widget="many2many_tags" options="{'no_create': True}" groups="product.group_product_variant"/>
                        </group>
                        <group string="BoM details">
                            <field name="company_id" invisible="1"/>
                            <field name="routing_id" invisible="1"/>
                            <field name="sequence"/>
                            <field name="operation_id" groups="mrp.group_mrp_routings"/>
                        </group>
                    </group>
                </form>
            </field>
        </record>

        <record id="template_open_bom" model="ir.actions.act_window">
            <field name="context">{'default_product_tmpl_id': active_id, 'search_default_product_tmpl_id': active_id}</field>
            <field name="name">Bill of Materials</field>
            <field name="res_model">mrp.bom</field>
        </record>

        <record id="product_open_bom" model="ir.actions.act_window">
            <field name="context">{'default_product_id': active_id, 'search_default_product_id': active_id}</field>
            <field name="name">Bill of Materials</field>
            <field name="res_model">mrp.bom</field>
            <field name="domain">[]</field> <!-- Force empty -->
        </record>
    </data>
</odoo>

```

## File: views\mrp_production_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Manufacturing Order -->
        <record id="mrp_production_tree_view" model="ir.ui.view">
            <field name="name">mrp.production.tree</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <tree string="Manufacturing Orders" default_order="date_planned_start desc"
                      decoration-bf="message_needaction==True" decoration-info="state=='confirmed'"
                      decoration-danger="date_planned_start&lt;current_date and state not in ('done','cancel')"
                      decoration-muted="state in ('done','cancel')" multi_edit="1">
                    <field name="message_needaction" invisible="1"/>
                    <field name="name"/>
                    <field name="date_planned_start" readonly="1" optional="show"
                        attrs="{'readonly': [('state', 'in', ['done', 'cancel'])]}"/>
                    <field name="product_id" readonly="1" optional="show"/>
                    <field name="product_uom_id" string="Unit of Measure" options="{'no_open':True,'no_create':True}" groups="uom.group_uom" optional="show"/>
                    <field name="bom_id" readonly="1" optional="hide"/>
                    <field name="origin" optional="show"/>
                    <field name="user_id" optional="hide"/>
                    <field name="routing_id" groups="mrp.group_mrp_routings" optional="show"/>
                    <field name="reservation_state" optional="show"/>
                    <field name="product_qty" sum="Total Qty" string="Quantity" readonly="1" optional="show"/>
                    <field name="state" optional="show"/>
                    <field name="company_id" readonly="1" groups="base.group_multi_company" optional="show"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                </tree>
            </field>
        </record>

        <record id="production_order_server_action" model="ir.actions.server">
            <field name="name">Mrp: Plan Production Orders</field>
            <field name="model_id" ref="mrp.model_mrp_production"/>
            <field name="binding_model_id" ref="mrp.model_mrp_production"/>
            <field name="binding_view_types">list</field>
            <field name="groups_id" eval="[(4, ref('mrp.group_mrp_routings'))]"/>
            <field name="state">code</field>
            <field name="code">records.button_plan()</field>
        </record>

        <record id="mrp_production_form_view" model="ir.ui.view">
            <field name="name">mrp.production.form</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <form string="Manufacturing Orders">
                <header>
                    <field name="confirm_cancel" invisible="1"/>
                    <button name="button_mark_done" attrs="{'invisible': [('state', '!=', 'to_close')]}" string="Mark as Done" type="object" class="oe_highlight"/>
                    <button name="action_confirm" attrs="{'invisible': ['|', ('state', '!=', 'draft'), ('is_locked', '=', False)]}" string="Mark as Todo" type="object" class="oe_highlight"/>
                    <button name="action_assign" attrs="{'invisible': ['|', '|', ('is_locked', '=', False), ('state', 'in', ('draft', 'done', 'cancel')), ('reservation_state', '=', 'assigned')]}" string="Check availability" type="object" class="oe_highlight"/>
                    <button name="button_plan" attrs="{'invisible': ['|', ('state', '!=', 'confirmed'), ('routing_id', '=', False)]}" type="object" string="Plan" class="oe_highlight"/>
                    <button name="button_unplan" type="object" string="Unplan" attrs="{'invisible': ['|', '|', ('state', '!=', 'planned'), ('date_planned_start', '=', False), ('date_planned_finished', '=', False)]}"/>
                    <button name="open_produce_product" attrs="{'invisible': ['|', '|', '|', ('state', '=', 'to_close'), ('is_locked', '=', False), ('reservation_state', '!=', 'assigned'), ('routing_id', '!=', False)]}" string="Produce" type="object" class="oe_highlight"/>
                    <button name="open_produce_product" attrs="{'invisible': ['|', '|', '|', ('state', '=', 'to_close'), ('is_locked', '=', False), ('reservation_state', 'not in', ('confirmed', 'waiting')), ('routing_id', '!=', False)]}" string="Produce" type="object"/>
                    <button name="post_inventory" string="Post Inventory" type="object" attrs="{'invisible': [('post_visible', '=', False)]}" groups="base.group_no_one"/>
                    <button name="button_scrap" type="object" string="Scrap" attrs="{'invisible': ['|', ('state', 'in', ('cancel', 'draft')), ('is_locked', '=', False)]}"/>
                    <button name="button_unreserve" type="object" string="Unreserve" attrs="{'invisible': [('unreserve_visible', '=', False)]}"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,confirmed,assigned,done"/>
                    <button name="action_toggle_is_locked" attrs="{'invisible': ['|', '|', ('state', 'in', ('cancel', 'draft')), ('id', '=', False), ('is_locked', '=', False)]}" string="Unlock" groups="mrp.group_mrp_manager" type="object" help="Unlock the manufacturing order to correct what has been consumed or produced."/>
                    <button name="action_toggle_is_locked" attrs="{'invisible': [('is_locked', '=', True)]}" string="Lock" class="oe_highlight" groups="mrp.group_mrp_manager" type="object"/>
                    <button name="action_cancel" type="object" string="Cancel"
                            attrs="{'invisible': ['|', '|', '|', ('id', '=', False), ('is_locked', '=', False), ('state', 'in', ('done','cancel')), ('confirm_cancel', '=', True)]}"/>
                    <button name="action_cancel" type="object" string="Cancel"
                            attrs="{'invisible': ['|', '|', '|', ('id', '=', False), ('is_locked', '=', False), ('state', 'in', ('done','cancel')), ('confirm_cancel', '=', False)]}"
                            confirm="Some product moves have already been confirmed, this manufacturing order can't be completely cancelled. Are you still sure you want to process ?"/>
                </header>
                <sheet>
                    <field name="reservation_state" invisible="1"/>
                    <field name="is_locked" invisible="1"/>
                    <field name="post_visible" invisible="1"/>
                    <field name="unreserve_visible" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button name="%(stock.action_stock_report)d" icon="fa-arrow-up" class="oe_stat_button" string="Traceability" type="action" states="done" groups="stock.group_production_lot"/>
                        <button name="%(action_mrp_workorder_production_specific)d" type="action" attrs="{'invisible': [('workorder_count', '=', 0)]}" class="oe_stat_button" icon="fa-play-circle-o">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="workorder_done_count" widget="statinfo" nolabel="1"/> / <field name="workorder_count" widget="statinfo" nolabel="1"/></span>
                                <span class="o_stat_text">Work Orders</span>
                            </div>
                        </button>
                        <button name="%(action_mrp_production_moves)d" type="action" string="Product Moves" class="oe_stat_button" icon="fa-exchange" attrs="{'invisible': [('state', 'not in', ('progress', 'done'))]}"/>
                        <button type="object" name="action_view_mo_delivery" class="oe_stat_button" icon="fa-truck"  groups="base.group_user" attrs="{'invisible': [('delivery_count', '=', 0)]}">
                            <field name="delivery_count" widget="statinfo" string="Transfers"/>
                        </button>
                        <button class="oe_stat_button" name="action_see_move_scrap" type="object" icon="fa-arrows-v" attrs="{'invisible': [('scrap_count', '=', 0)]}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="scrap_count"/></span>
                                <span class="o_stat_text">Scraps</span>
                            </div>
                        </button>
                        <field name="workorder_ids" invisible="1"/>
                    </div>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Manufacturing Reference" nolabel="1"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="id" invisible="1"/>
                            <field name="product_id" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                            <field name="product_tmpl_id" invisible="1"/>
                            <label for="product_qty"/>
                            <div class="o_row no-gutters d-flex">
                                <div class="col">
                                    <field name="product_qty" class="mr-1" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                    <field name="product_uom_id" options="{'no_open':True,'no_create':True}" force_save="1" groups="uom.group_uom" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                    <button type="action"
                                        name="%(mrp.action_change_production_qty)d"
                                        context="{'default_mo_id': id}"
                                        string="Update" class="oe_link" attrs="{'invisible': ['|', ('state', 'in', ('draft', 'done','cancel')), ('id', '=', False)]}"/>
                                </div>
                            </div>
                            <field name="bom_id"
                                context="{'default_product_tmpl_id': product_tmpl_id}" required="1" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                            <field name="routing_id" groups="mrp.group_mrp_routings"/>
                        </group>
                        <group>
                            <field name="date_deadline" attrs="{'readonly': [('state', 'in', ['done', 'cancel'])]}"/>
                            <field name="date_start_wo"
                                attrs="{'invisible': [
                                '|',
                                    '&amp;',
                                        ('routing_id', '!=', False),
                                        ('state', 'not in', ['draft', 'confirmed']),
                                    ('routing_id', '=', False),
                                ], 'readonly': [
                                    ('routing_id', '!=', False),
                                    ('state', 'not in', ['draft', 'confirmed']),
                                ]}"/>
                            <div class="o_row o_td_label">
                                <label for="date_planned_start" string="Planned Date"
                                       attrs="{'invisible': [
                                                ('routing_id', '!=', False),
                                                ('state', 'in', ['draft', 'confirmed'])
                                        ]}"/>
                            </div>
                            <div class="o_row">
                                <field name="date_planned_start"
                                       attrs="{'invisible': [
                                            ('routing_id', '!=', False),
                                            ('state', 'in', ['draft', 'confirmed']),
                                        ], 'readonly': [
                                       '|',
                                        '&amp;',
                                            ('routing_id', '=', False),
                                            ('state', 'in', ['done', 'cancel']),
                                        '&amp;',
                                            ('routing_id', '!=', False),
                                            ('state', 'not in', ['draft', 'confirmed'])]}"/>
                                <label for="date_planned_finished" string="to" attrs="{'invisible': [
                                    '|',
                                        ('id', '=', False),
                                        '&amp;',
                                            ('routing_id', '!=', False),
                                            ('state', 'in', ['draft', 'confirmed'])
                                ]}"/>
                                <field name="date_planned_finished" required="1"
                                attrs="{'readonly': [
                                    '|',
                                        '&amp;',
                                            ('routing_id', '=', False),
                                            ('state', 'in', ['done', 'cancel']),
                                        ('routing_id', '!=', False)
                                ], 'invisible': [
                                    ('routing_id', '!=', False),
                                    ('state', 'in', ['draft', 'confirmed'])
                                ]}"/>
                            </div>
                            <field name="user_id"/>
                            <field name="origin"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" attrs="{'readonly': [('state', '!=', 'draft')]}" force_save="1"/>
                            <field name="show_final_lots" invisible="1"/>
                            <field name="production_location_id" invisible="1" readonly="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Components">
                            <field name="move_raw_ids" context="{'final_lots': show_final_lots, 'tree_view_ref': 'mrp.view_stock_move_raw_tree', 'form_view_ref': 'mrp.view_stock_move_lots', 'default_location_id': location_src_id, 'default_location_dest_id': production_location_id, 'default_state': 'draft', 'default_raw_material_production_id': id, 'default_picking_type_id': picking_type_id}" attrs="{'readonly': ['&amp;', ('state', '!=', 'draft'), ('is_locked', '=', True)]}"/>
                        </page>
                        <page string="Finished Products">
                            <field name="finished_move_line_ids" context="{'form_view_ref': 'mrp.view_finisehd_move_line'}" attrs="{'readonly': [('is_locked', '=', True)], 'invisible': [('finished_move_line_ids', '=', [])]}">
                                 <tree default_order="done_move" editable="bottom" create="0" delete="0" decoration-muted="state in ('done', 'cancel')">
                                    <field name="product_id" readonly="1"/>
                                    <field name="company_id" invisible="1"/>
                                    <field name="lot_id" groups="stock.group_production_lot" context="{'default_product_id': product_id, 'default_company_id': company_id}" attrs="{'invisible': [('lots_visible', '=', False)]}"/>
                                    <field name="product_uom_id" groups="uom.group_uom"/>
                                    <field name="qty_done" string="Produced"/>
                                    <field name="lots_visible" invisible="1"/>
                                    <field name="done_move" invisible="1"/>
                                    <field name="state" invisible="1"/>
                                </tree>
                            </field>
                            <p attrs="{'invisible': [('finished_move_line_ids', '!=', [])]}">
                                Use the Produce button or process the work orders to create some finished products.
                            </p>
                        </page>
                        <page string="Miscellaneous" groups="stock.group_stock_multi_locations">
                            <group>
                                <group>
                                    <field name="picking_type_id" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                    <field name="location_src_id" options="{'no_create': True}" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                    <field name="location_dest_id" options="{'no_create': True}" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers"/>
                    <field name="activity_ids" widget="mail_activity"/>
                    <field name="message_ids" widget="mail_thread"/>
                </div>
                </form>
            </field>
        </record>

        <record id="mrp_production_kanban_view" model="ir.ui.view">
            <field name="name">mrp.production.kanban</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="name"/>
                    <field name="product_id"/>
                    <field name="product_qty"/>
                    <field name="product_uom_id" options="{'no_open':True,'no_create':True}"/>
                    <field name="date_deadline"/>
                    <field name="state"/>
                    <field name="activity_state"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings mt4">
                                        <strong class="o_kanban_record_title"><span><t t-esc="record.product_id.value"/></span></strong>
                                    </div>
                                    <span class="float-right text-right"><t t-esc="record.product_qty.value"/> <small><t t-esc="record.product_uom_id.value"/></small></span>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left text-muted">
                                        <span><t t-esc="record.name.value"/> <t t-esc="record.date_deadline.value and record.date_deadline.value.split(' ')[0] or False"/></span>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <span t-attf-class="badge #{['draft', 'cancel'].indexOf(record.state.raw_value) > -1 ? 'badge-secondary' : ['none'].indexOf(record.state.raw_value) > -1 ? 'badge-danger' : ['confirmed'].indexOf(record.state.raw_value) > -1 ? 'badge-warning' : ['done'].indexOf(record.state.raw_value) > -1 ? 'badge-success' : 'badge-primary'}"><t t-esc="record.state.value"/></span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_production_calendar" model="ir.ui.view">
            <field name="name">mrp.production.calendar</field>
            <field name="model">mrp.production</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <calendar date_start="date_planned_start" date_stop="date_planned_finished"
                          string="Manufacturing Orders" color="routing_id" event_limit="5">
                    <field name="routing_id"/>
                    <field name="user_id" avatar_field="image_128"/>
                    <field name="product_id"/>
                    <field name="product_qty"/>
                </calendar>
            </field>
        </record>

        <record id="view_production_gantt" model="ir.ui.view">
            <field name="name">mrp.production.gantt</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <gantt date_stop="date_finished" date_start="date_start" string="Productions" default_group_by="routing_id" create="0">
                </gantt>
            </field>
        </record>

        <record id="view_production_pivot" model="ir.ui.view">
            <field name="name">mrp.production.pivot</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <pivot string="Manufacturing Orders">
                    <field name="date_deadline" type="row"/>
                </pivot>
            </field>
        </record>

        <record id="view_production_graph" model="ir.ui.view">
            <field name="name">mrp.production.graph</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <graph string="Manufacturing Orders" type="line">
                    <field name="date_planned_finished"/>
                </graph>
            </field>
        </record>

        <record id="view_mrp_production_filter" model="ir.ui.view">
            <field name="name">mrp.production.select</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <search string="Search Production">
                    <field name="name" string="Manufacturing Order" filter_domain="['|', ('name', 'ilike', self), ('origin', 'ilike', self)]"/>
                    <field name="product_id"/>
                    <field name="move_raw_ids" string="Component" filter_domain="[('move_raw_ids.product_id', 'ilike', self)]"/>
                    <field name="name" string="Work Center" filter_domain="[('routing_id.operation_ids.workcenter_id', 'ilike', self)]"/>
                    <field name="routing_id" groups="mrp.group_mrp_routings"/>
                    <field name="origin"/>
                    <filter string="To Do" name="todo" domain="[('state', 'in', ('draft', 'confirmed', 'planned','progress', 'to_close'))]"
                        help="Manufacturing Orders which are in confirmed state."/>
                    <separator/>
                    <filter string="Draft" name="filter_draft" domain="[('state', '=', 'draft')]"/>
                    <filter string="Confirmed" name="filter_confirmed" domain="[('state', '=', 'confirmed')]"/>
                    <filter string="Planned" name="filter_planned" domain="[('state', '=', 'planned')]" groups="mrp.group_mrp_routings"/>
                    <filter string="In Progress" name="filter_in_progress" domain="[('state', '=', 'progress')]"/>
                    <filter string="To Close" name="filter_to_close" domain="[('state', '=', 'to_close')]"/>
                    <filter string="Done" name="filter_done" domain="[('state', '=', 'done')]"/>
                    <filter string="Cancelled" name="filter_cancel" domain="[('state', '=', 'cancel')]"/>
                    <separator/>
                    <filter string="Waiting for Component" name="waiting" domain="[('reservation_state', 'in', ('waiting', 'confirmed'))]"/>
                    <filter string="Component Available" name="filter_ready" domain="[('reservation_state', '=', 'assigned')]"/>
                    <separator/>
                    <filter string="Late" name="late" help="Production started late"
                        domain="['&amp;', ('date_deadline', '&lt;', current_date), ('state', '=', 'confirmed')]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Warnings" name="activities_exception"
                        domain="[('activity_exception_decoration', '!=', False)]"/>
                    <group expand="0" string="Group By...">
                        <filter string="Product" name="product" domain="[]" context="{'group_by': 'product_id'}"/>
                        <filter string="Routing" name="routing" domain="[]" context="{'group_by': 'routing_id'}" groups="mrp.group_mrp_routings"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by': 'state'}"/>
                        <filter string="Material Availability" name="groupby_reservation_state" domain="[]" context="{'group_by': 'reservation_state'}"/>
                        <filter string="Scheduled Date" name="scheduled_date" domain="[]" context="{'group_by': 'date_planned_start'}" help="Scheduled Date by Month"/>
                    </group>
               </search>
            </field>
        </record>

        <record id="mrp_production_action" model="ir.actions.act_window">
            <field name="name">Manufacturing Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.production</field>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph</field>
            <field name="view_id" eval="False"/>
            <field name="search_view_id" ref="view_mrp_production_filter"/>
            <field name="context">{'search_default_todo': True, 'default_company_id': allowed_company_ids[0]}</field>
            <field name="domain">[('picking_type_id.active', '=', True)]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new manufacturing order <br/>
              </p><p>
                Consume components and build finished products using bills of materials
              </p>
            </field>
        </record>

        <record id="mrp_production_action_picking_deshboard" model="ir.actions.act_window">
            <field name="name">Manufacturing Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.production</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" eval="False"/>
            <field name="search_view_id" ref="view_mrp_production_filter"/>
            <field name="domain">[('picking_type_id', '=', active_id)]</field>
            <field name="context">{}</field>
        </record>


        <menuitem action="mrp_production_action"
            id="menu_mrp_production_action"
            parent="menu_mrp_manufacturing"
            sequence="1"/>

        <record id="mrp_production_report" model="ir.actions.act_window">
            <field name="name">Manufacturing Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.production</field>
            <field name="view_mode">graph,pivot,form</field>
            <field name="target">current</field>
        </record>

        <menuitem id="menu_mrp_workorder_todo"
                name="Work Orders"
                action="mrp_workorder_todo"
                parent="menu_mrp_manufacturing"
                groups="group_mrp_routings"/>

        <menuitem id="menu_mrp_production_report"
            name="Manufacturing Orders"
            parent="menu_mrp_reporting"
            action="mrp_production_report"
            sequence="11"/>

        <menuitem id="menu_mrp_work_order_report"
            name="Work Orders"
            parent="menu_mrp_reporting"
            action="mrp_workorder_report"
            groups="group_mrp_routings"
            sequence="11"/>

        <record id="act_product_mrp_production" model="ir.actions.act_window">
            <field name="context">{'search_default_product_id': [active_id]}</field>
            <field name="name">Manufacturing Orders</field>
            <field name="res_model">mrp.production</field>
            <field name="view_id" ref="mrp_production_tree_view"/>
        </record>

        <record id="action_mrp_production_form" model="ir.actions.act_window">
            <field name="name">Manufacturing Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.production</field>
            <field name="view_mode">form</field>
            <field name="context">{'default_picking_type_id': active_id}</field>
        </record>
    </data>
</odoo>

```

## File: views\mrp_routing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Routings Workcenter -->
        <record id="mrp_routing_workcenter_tree_view" model="ir.ui.view">
            <field name="name">mrp.routing.workcenter.tree</field>
            <field name="model">mrp.routing.workcenter</field>
            <field name="arch" type="xml">
                <tree string="Routing Work Centers">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="workcenter_id"/>
                    <field name="time_cycle" widget="float_time" string="Duration (minutes)" sum="Total Duration"/>
                </tree>
            </field>
        </record>

        <record id="mrp_routing_workcenter_form_view" model="ir.ui.view">
            <field name="name">mrp.routing.workcenter.form</field>
            <field name="model">mrp.routing.workcenter</field>
            <field name="arch" type="xml">
                <form string="Routing Work Centers">
                    <sheet>
                        <field name="routing_id" invisible="1" required="0"/>
                        <group>
                            <group name="description">
                                <field name="name"/>
                                <field name="workcenter_id" context="{'default_company_id': company_id}"/>
                                <field name="sequence" groups="base.group_no_one"/>
                                <field name="company_id" groups="base.group_multi_company" />
                            </group><group name="workorder">
                                <field name="workorder_count" invisible="1"/>
                                <field name="time_mode" widget="radio"/>
                                <label for="time_mode_batch" attrs="{'invisible': [('time_mode', '=', 'manual')]}"/>
                                <div attrs="{'invisible': [('time_mode', '=', 'manual')]}">
                                    last
                                    <field name="time_mode_batch" class="oe_inline"/>
                                    work orders
                                </div>
                                <label for="time_cycle_manual" attrs="{'invisible': [('time_mode', '=', 'auto'), ('workorder_count', '!=' , 0)]}" string="Default Duration"/>
                                <div attrs="{'invisible':  [('time_mode', '=', 'auto'), ('workorder_count', '!=' , 0)]}">
                                    <field name="time_cycle_manual" widget="float_time" class="oe_inline"/> minutes
                                </div>
                                <field name="time_cycle" invisible="1"/>
                            </group>
                            <group>
                                <field name="batch" widget="radio"/>
                                <field name="batch_size" attrs="{'invisible': [('batch', '=', 'no')]}"/>
                            </group>
                        </group>
                        <notebook>
                            <page string="Description">
                                <field name="note"/>
                            </page>
                            <page string="Work Sheet">
                                <group>
                                    <field name="worksheet_type" widget="radio"/>
                                    <field name="worksheet" widget="pdf_viewer" attrs="{'invisible':  [('worksheet_type', '=', 'google_slide')]}"/>
                                    <field name="worksheet_google_slide" placeholder="Google Slide Link" widget="embed_viewer" attrs="{'invisible':  [('worksheet_type', '=', 'pdf')]}"/>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <!-- Routings -->
        <record id="mrp_routing_form_view" model="ir.ui.view">
            <field name="name">mrp.routing.form</field>
            <field name="model">mrp.routing</field>
            <field name="arch" type="xml">
                <form string="Routing">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="%(action_mrp_routing_time)d" type="action" class="oe_stat_button" icon="fa-clock-o">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_text">Time<br/> Analysis</span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title">
                            <h1>
                                <field name="code"/>
                            </h1>
                        </div>
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="active" invisible="1"/>
                            </group>
                            <group>
                                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                           </group>
                        </group>
                        <notebook>
                            <page string="Work Center Operations">
                                <field name="operation_ids" context="{'default_routing_id': id}"/>
                            </page>
                            <page string="Notes">
                                <field name="note"/>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="mrp_routing_tree_view" model="ir.ui.view">
            <field name="name">mrp.routing.tree</field>
            <field name="model">mrp.routing</field>
            <field name="arch" type="xml">
                <tree string="Routing">
                    <field name="code"/>
                    <field name="name"/>
                    <field name="active" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="mrp_routing_kanban_view" model="ir.ui.view">
            <field name="name">mrp.routing.kanban</field>
            <field name="model">mrp.routing</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="code"/>
                    <field name="name"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings mt4">
                                        <strong class="o_kanban_record_title"><span><t t-esc="record.name.value"/></span></strong>
                                    </div>
                                    <span class="badge badge-pill"><field name="code"/></span>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="mrp_routing_search_view" model="ir.ui.view">
            <field name="name">mrp.routing.search</field>
            <field name="model">mrp.routing</field>
            <field name="arch" type="xml">
                <search string="Routing">
                    <field name="name" string="Routing" filter_domain="['|', ('name', 'ilike', self), ('code', 'ilike', self)]"/>
                    <filter name="inactive" string="Archived" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="mrp_routing_action" model="ir.actions.act_window">
            <field name="name">Routings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.routing</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_id" ref="mrp_routing_tree_view"/>
            <field name="search_view_id" ref="mrp_routing_search_view"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new routing
              </p><p>
                Routings define the successive operations that need to be
                done to realize a Manufacturing Order. Each operation from
                a Routing is done at a specific Work Center and has a specific duration.
              </p>
            </field>
        </record>

        <menuitem id="menu_mrp_routing_action"
          action="mrp_routing_action"
          parent="menu_mrp_bom"
          groups="group_mrp_routings"
          sequence="50"/>
    </data>
</odoo>

```

## File: views\mrp_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="mrp assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/mrp/static/src/scss/mrp_workorder_kanban.scss" />
            <script type="text/javascript" src="/mrp/static/src/js/mrp.js"></script>
            <script type="text/javascript" src="/mrp/static/src/js/mrp_bom_report.js"></script>
        </xpath>
   </template>

    <template id="assets_common" name="mrp bom common assets" inherit_id="web.assets_common">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/mrp/static/src/scss/mrp_bom_report.scss" />
            <link rel="stylesheet" type="text/scss" href="/mrp/static/src/scss/mrp_fields.scss" />
            <link rel="stylesheet" type="text/scss" href="/mrp/static/src/scss/mrp_gantt.scss" />
        </xpath>
    </template>

    <template id="qunit_suite" inherit_id="web.qunit_suite">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/mrp/static/tests/mrp_tests.js"/>
        </xpath>
    </template>

    <template id="exception_on_mo">
        <div class="alert alert-warning" role="alert"> 
            Exception(s) occurred on the manufacturing order(s):
            <a href="#" data-oe-model="mrp.production" t-att-data-oe-id="production_order.id"><t t-esc="production_order.name"/></a>.
            Manual actions may be needed.
            <div class="mt16">
                <p>Exception(s):</p>
                <ul t-foreach="order_exceptions.items()" t-as="order_exception">
                    <li>
                        <t t-set="move_raw_id" t-value="order_exception[0]"/>
                        <t t-set="exception" t-value="order_exception[1]"/>
                        <t t-set="order" t-value="exception[0]"/>
                        <t t-set="new_qty" t-value="exception[1][0]"/>
                        <t t-set="old_qty" t-value="exception[1][1]"/>
                            <a href="#" data-oe-model="mrp.production" t-att-data-oe-id="production_order.id"><t t-esc="production_order.name"/></a>:
                            <t t-esc="new_qty"/> <t t-esc="move_raw_id.product_uom.name"/> of <t t-esc="move_raw_id.product_id.name"/>
                            <t t-if="cancel">
                                cancelled
                            </t>
                            <t t-if="not cancel">
                                ordered instead of <t t-esc="old_qty"/> <t t-esc="move_raw_id.product_uom.name"/>
                            </t>
                      </li>
                </ul>
            </div>
            <div class="mt16" t-if="not cancel and impacted_pickings">
                <p>Impacted Transfer(s):</p>
                <ul t-foreach="impacted_pickings" t-as="picking">
                    <li><a href="#" data-oe-model="stock.picking" t-att-data-oe-id="picking.id"><t t-esc="picking.name"/></a></li>
                </ul>
            </div>
        </div>
    </template>

</odoo>

```

## File: views\mrp_unbuild_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--  Unbuild and scrap menu -->

   <record model="ir.actions.act_window" id="action_mrp_unbuild_moves">
        <field name="name">Stock Moves</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">stock.move.line</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('move_id.unbuild_id', '=', active_id), ('move_id.consume_unbuild_id', '=', active_id)]</field>
    </record>

        <record id="mrp_unbuild_search_view" model="ir.ui.view">
            <field name="name">mrp.unbuild.search</field>
            <field name="model">mrp.unbuild</field>
            <field name="arch" type="xml">
                <search string="Search">
                    <field name="product_id"/>
                    <field name="mo_id"/>
                    <group expand="0" string="Filters">
                        <filter name="draft" string="Draft" domain="[('state', '=', 'draft')]"/>
                        <filter name="done" string="Done" domain="[('state', '=', 'done')]"/>
                        <filter invisible="1" string="Late Activities" name="activities_overdue"
                            domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                            help="Show all records which has next action date is before today"/>
                        <filter invisible="1" string="Today Activities" name="activities_today"
                            domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                        <filter invisible="1" string="Future Activitie" name="activities_upcoming_all"
                            domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    </group>
                    <group expand='0' string='Group by...'>
                        <filter string='Product' name="productgroup" context="{'group_by': 'product_id'}"/>
                        <filter string="Manufacturing Order" name="mogroup" context="{'group_by': 'mo_id'}"/>
                    </group>
               </search>
            </field>
        </record>

       <record model="ir.actions.act_window" id="action_mrp_unbuild_move_line">
            <field name="name">Product Moves</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">stock.move.line</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">['|', ('move_id.unbuild_id', '=', active_id), ('move_id.consume_unbuild_id', '=', active_id)]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_empty_folder">
                    There's no product move yet
                </p><p>
                    This menu gives you the full traceability of inventory operations on a specific product.
                    You can filter on the product to see all the past movements for the product.
                </p>
            </field>
        </record>

        <record id="mrp_unbuild_kanban_view" model="ir.ui.view">
            <field name="name">mrp.unbuild.kanban</field>
            <field name="model">mrp.unbuild</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="name"/>
                    <field name="product_id"/>
                    <field name="product_qty"/>
                    <field name="product_uom_id"/>
                    <field name="state"/>
                    <field name="location_id"/>
                    <field name="activity_state"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings mt4">
                                        <strong class="o_kanban_record_title"><span><field name="name"/></span></strong>
                                    </div>
                                    <strong><t t-esc="record.product_qty.value"/> <small><t t-esc="record.product_uom_id.value"/></small></strong>
                                </div>
                                <div class="row">
                                    <div class="col-8 text-muted">
                                        <span><t t-esc="record.product_id.value"/></span>
                                    </div>
                                    <div class="col-4">
                                        <span class="float-right text-right">
                                            <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'done': 'success'}}"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="mrp_unbuild_form_view" model="ir.ui.view">
            <field name="name">mrp.unbuild.form</field>
            <field name="model">mrp.unbuild</field>
            <field name="arch" type="xml">
                <form string="Unbuild Orders">
                    <header>
                        <button name="action_validate" string="Unbuild" type="object" states="draft" class="oe_highlight"/>
                        <field name="state" widget="statusbar" statusbar_visible="draft,done"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" name="%(action_mrp_unbuild_moves)d"
                                    string="Product Moves" type="action" icon="fa-exchange" states="done"/>
                        </div>
                        <div class="oe_title">
                            <h1><field name="name" placeholder="Unbuild Order" nolabel="1"/></h1>
                        </div>
                        <group>
                            <group>
                                <field name="product_id" attrs="{'readonly':[('mo_id','!=',False)]}" force_save="1"/>
                                <field name="bom_id" attrs="{'readonly':[('mo_id','!=',False)]}" force_save="1"/>
                                <label for="product_qty"/>
                                <div class="o_row">
                                    <field name="product_qty"/>
                                    <field name="product_uom_id" options="{'no_open':True,'no_create':True}" groups="uom.group_uom"/>
                                </div>
                            </group>
                            <group>
                                <field name="mo_id"/>
                                <field name="location_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                <field name="location_dest_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                <field name="has_tracking" invisible="1"/>
                                <field name="lot_id" attrs="{'invisible': [('has_tracking', '=', 'none')], 'required': [('has_tracking', '!=', 'none')]}" groups="stock.group_production_lot"/>
                                <field name="company_id" group="base.group_multi_company"/>
                            </group>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" widget="mail_followers"/>
                        <field name="activity_ids" widget="mail_activity"/>
                        <field name="message_ids" widget="mail_thread"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="mrp_unbuild_tree_view" model="ir.ui.view">
            <field name="name">mrp.unbuild.tree</field>
            <field name="model">mrp.unbuild</field>
            <field name="arch" type="xml">
                <tree>
                    <field name="name"/>
                    <field name="product_id"/>
                    <field name="bom_id"/>
                    <field name="mo_id"/>
                    <field name="lot_id" groups="stock.group_production_lot"/>
                    <field name="product_qty"/>
                    <field name="product_uom_id" groups="uom.group_uom"/>
                    <field name="state"/>
                    <field name="location_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

    <record model="ir.actions.act_window" id="mrp_unbuild">
        <field name="name">Unbuild Orders</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.unbuild</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create an unbuild order
          </p><p>
            An unbuild order is used to break down a finished product into its components.
          </p>
        </field>
    </record>

    <menuitem id="menu_mrp_unbuild"
          name="Unbuild Orders"
          parent="menu_mrp_manufacturing"
          action="mrp_unbuild"
          sequence="20"/>

</odoo>

```

## File: views\mrp_views_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Top menu item -->
        <menuitem id="menu_mrp_root"
            name="Manufacturing"
            groups="group_mrp_user,group_mrp_manager"
            icon="fa-wrench"
            web_icon="mrp,static/description/icon.png"
            sequence="35"/>

        <menuitem id="menu_mrp_manufacturing"
            name="Operations"
            parent="menu_mrp_root"
            sequence="10"/>

        <menuitem id="mrp_planning_menu_root"
            name="Planning"
            parent="menu_mrp_root"
            sequence="15"/>

        <menuitem id="menu_mrp_bom"
            name="Master Data"
            parent="menu_mrp_root"
            sequence="20"/>

        <menuitem id="menu_mrp_reporting"
              name="Reporting"
              parent="menu_mrp_root"
              sequence="25"/>

        <menuitem id="menu_mrp_configuration"
            name="Configuration"
            parent="menu_mrp_root"
            groups="group_mrp_manager"
            sequence="100"/>

    </data>
</odoo>

```

## File: views\mrp_workcenter_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="act_product_mrp_production_workcenter" model="ir.actions.act_window">
            <field name="context">{'search_default_confirmed': 1}</field>
            <field name="name">Manufacturing Orders</field>
            <field name="res_model">mrp.production</field>
            <field name="view_mode">tree,form,gantt</field>
            <field name="domain">[('routing_id', '!=', False),('routing_id.operation_ids.workcenter_id','=', active_id)]</field>
        </record>

        <record id="action_work_orders" model="ir.actions.act_window">
            <field name="name">Work Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workorder</field>
            <field name="view_mode">tree,form,gantt,pivot,graph,calendar</field>
            <field name="search_view_id" ref="view_mrp_production_work_order_search"/>
            <field name="domain">[('state', 'not in', ('done', 'cancel'))]</field>
            <field name="context">{'search_default_workcenter_id': active_id}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a work center
              </p><p>
                Work Centers allow you to create and manage manufacturing
                units. They consist of workers and/or machines, which are
                considered as units for task assignation as well as capacity
                and planning forecast.
              </p>
            </field>
        </record>

        <!-- Work Centers -->
        <record id="mrp_workcenter_tree_view" model="ir.ui.view">
            <field name="name">mrp.workcenter.tree</field>
            <field name="model">mrp.workcenter</field>
            <field name="arch" type="xml">
                <tree string="Work Center">
                    <field name="sequence" widget="handle"/>
                    <field name="code"/>
                    <field name="name"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="active" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="mrp_workcenter_view_kanban" model="ir.ui.view">
            <field name="name">mrp.workcenter.kanban</field>
            <field name="model">mrp.workcenter</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_content oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-12">
                                        <strong><field name="name"/></strong>
                                    </div>
                                    <div class="col-12">
                                        <span>Code <field name="code"/></span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="oee_pie_view" model="ir.ui.view">
            <field name="name">mrp.workcenter.productivity.graph</field>
            <field name="model">mrp.workcenter.productivity</field>
            <field name="priority">20</field>
            <field name="arch" type="xml">
                <graph string="Workcenter Productivity" type="pie">
                    <field name="loss_id"/>
                    <field name="duration" type="measure"/>
                </graph>
            </field>
        </record>
        <record model="ir.actions.act_window" id="mrp_workcenter_productivity_report_oee">
            <field name="name">Overall Equipment Effectiveness</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workcenter.productivity</field>
            <field name="view_id" eval="oee_pie_view"/>
            <field name="view_mode">graph,pivot,tree,form</field>
            <field name="domain">[('workcenter_id','=',active_id')]</field>
            <field name="context">{'search_default_thismonth':True}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                Overall Equipment Effectiveness: no working or blocked time
              </p>
            </field>
        </record>
        <record model="ir.actions.act_window" id="mrp_workcenter_productivity_report_blocked">
            <field name="name">Productivity Losses</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workcenter.productivity</field>
            <field name="view_mode">tree,form,graph,pivot</field>
            <field name="context">{'search_default_availability': '1',
                                   'search_default_performance': '1',
                                   'search_default_quality': '1',
                                   'default_workcenter_id': active_id,
                                   'search_default_workcenter_id': [active_id]}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                No productivity loss for this equipment
              </p>
            </field>
        </record>

        <record model="ir.actions.act_window" id="mrp_workorder_workcenter_report">
            <field name="name">Work Orders Performance</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workorder</field>
            <field name="domain">[('workcenter_id','=', active_id),('state','=','done')]</field>
            <field name="view_mode">graph,pivot,tree,form,gantt</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new work orders performance
              </p>
            </field>
        </record>

        <record model="ir.actions.act_window" id="mrp_workorder_report">
            <field name="name">Work Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workorder</field>
            <field name="domain">[]</field>
            <field name="context">{'search_default_done': True}</field>
            <field name="view_mode">graph,pivot,tree,form,gantt</field>
            <field name="search_view_id" ref="view_mrp_production_work_order_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new work orders performance
              </p>
            </field>
        </record>

        <!-- Workcenter Kanban view-->
        <record model="ir.ui.view" id="mrp_workcenter_kanban">
            <field name="name">mrp.workcenter.kanban</field>
            <field name="model">mrp.workcenter</field>
            <field name="arch" type="xml">
                <kanban class="oe_background_grey o_kanban_dashboard o_workcenter_kanban" create="0">
                    <field name="name"/>
                    <field name="color"/>
                    <field name="workorder_count"/>
                    <field name="working_state"/>
                    <field name="oee_target"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                                <div t-attf-class="o_kanban_card_header o_kanban_record_top">
                                    <div class="o_kanban_record_headings o_kanban_card_header_title">
                                        <span class="o_primary ml8" style="display: inline-block">
                                            <field name="name"/>
                                        </span>
                                    </div>
                                    <div class="o_kanban_manage_button_section">
                                        <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
                                    </div>
                                </div>
                                <div class="container o_kanban_card_content">
                                    <div class="row mb16">
                                        <div class="col-6 o_kanban_primary_left">
                                            <div class="btn-group" name="o_wo">
                                            <t t-if="record.workorder_count.raw_value &gt; 0">
                                                <button class="btn btn-primary" name="action_work_order" type="object" context="{'search_default_ready': 1, 'search_default_progress': 1}">
                                                    <span>WORK ORDERS</span>
                                                </button>
                                            </t>
                                            <t  t-if="record.workorder_count.raw_value &lt;= 0">
                                                <button class="btn btn-warning" name="%(act_product_mrp_production_workcenter)d" type="action">
                                                    <span>PLAN ORDERS</span>
                                                </button>
                                            </t>
                                            </div>
                                        </div>
                                        <div class="col-6 o_kanban_primary_right">
                                            <div class="row" t-if="record.workorder_ready_count.raw_value &gt; 0">
                                                <div class="col-8">
                                                    <a name="action_work_order" type="object" context="{'search_default_ready': 1}">
                                                        To Launch
                                                    </a>
                                                </div>
                                                <div class="col-4 text-right">
                                                    <field name="workorder_ready_count"/>
                                                </div>
                                            </div>
                                            <div class="row" t-if="record.workorder_progress_count.raw_value &gt; 0">
                                                <div class="col-8">
                                                    <a name="action_work_order" type="object" context="{'search_default_progress': 1}">
                                                        In Progress
                                                    </a>
                                                </div>
                                                <div class="col-4 text-right">
                                                    <field name="workorder_progress_count"/>
                                                </div>
                                            </div>
                                            <div class="row" t-if="record.workorder_late_count.raw_value &gt; 0">
                                                <div class="col-8">
                                                    <a name="action_work_order" type="object" context="{'search_default_late': 1}">
                                                        Late
                                                    </a>
                                                </div>
                                                <div class="col-4 text-right">
                                                    <field name="workorder_late_count"/>
                                                </div>
                                            </div>
                                            <div class="row" t-if="record.oee.raw_value &gt; 0">
                                                <div class="col-6">
                                                    <a name="%(mrp_workcenter_productivity_report_oee)d" type="action">
                                                        OEE
                                                    </a>
                                                </div>
                                                <div class="col-6 text-right">
                                                    <span t-att-class="record.oee_target.raw_value and (record.oee.raw_value &lt; record.oee_target.raw_value) and 'text-danger' or (record.oee.raw_value &gt; record.oee_target.raw_value) and 'text-success' or 'text-warning'">
                                                        <strong>
                                                            <field name="oee"/>%
                                                        </strong>
                                                    </span>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="row">
                                        <div class="col-12 o_kanban_primary_left" style="position:absolute; bottom:10px;" name="wc_stages">
                                            <a name="%(act_mrp_block_workcenter)d" type="action" class="o_status float-right"
                                                title="No workorder currently in progress. Click to mark work center as blocked."
                                                aria-label="No workorder currently in progress. Click to mark work center as blocked."
                                                attrs="{'invisible': [('working_state','in',('blocked','done'))]}"/>
                                            <a name="unblock" type="object" class=" o_status o_status_red float-right"
                                                title="Workcenter blocked, click to unblock."
                                                aria-label="Workcenter blocked, click to unblock."
                                                attrs="{'invisible': [('working_state','in',('normal','done'))]}"/>
                                            <a name="%(act_mrp_block_workcenter)d" type="action" class="o_status o_status_green float-right"
                                                title="Work orders in progress. Click to block work center."
                                                aria-label="Work orders in progress. Click to block work center."
                                                attrs="{'invisible': [('working_state','in',('normal','blocked'))]}"/>
                                        </div>
                                    </div>
                                </div><div class="container o_kanban_card_manage_pane dropdown-menu" role="menu">
                                    <div class="row">
                                        <div class="col-6 o_kanban_card_manage_section o_kanban_manage_view">
                                            <div role="menuitem" class="o_kanban_card_manage_title">
                                                <span>Actions</span>
                                            </div>
                                            <div role="menuitem" name="plan_order">
                                                <a name="action_work_order" type="object">Plan Orders</a>
                                            </div>
                                        </div>
                                        <div class="col-6 o_kanban_card_manage_section o_kanban_manage_new">
                                            <div role="menuitem" class="o_kanban_card_manage_title">
                                                <span>Reporting</span>
                                            </div>
                                            <div role="menuitem">
                                                <a name="%(mrp_workcenter_productivity_report_oee)d" type="action">OEE</a>
                                            </div>
                                            <div role="menuitem">
                                                <a name="%(mrp_workorder_workcenter_report)d" type="action" context="{'search_default_thisyear':True}">
                                                    Performance
                                                </a>
                                            </div>
                                            <div role="menuitem">
                                                <a name="action_work_order" type="object" context="{'search_default_waiting': 1}">Waiting Availability</a>
                                            </div>
                                        </div>
                                    </div>

                                    <div t-if="widget.editable" class="o_kanban_card_manage_settings row">
                                        <div role="menuitem" aria-haspopup="true" class="col-8">
                                            <ul role="menu" class="oe_kanban_colorpicker" data-field="color"/>
                                        </div>
                                        <div role="menuitem" class="col-4 text-right">
                                            <a type="edit">Settings</a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="mrp_workcenter_view" model="ir.ui.view">
            <field name="name">mrp.workcenter.form</field>
            <field name="model">mrp.workcenter</field>
            <field name="arch" type="xml">
                <form string="Work Center">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="%(mrp_workcenter_productivity_report_oee)d" type="action" class="oe_stat_button" icon="fa-pie-chart">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="oee" widget="statinfo" nolabel="1"/>%</span>
                                    <span class="o_stat_text">OEE</span>
                                </div>
                            </button>
                            <button name="%(mrp_workcenter_productivity_report_blocked)d" type="action" class="oe_stat_button" icon="fa-bar-chart">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="blocked_time" widget="statinfo" nolabel="1"/> Hours</span>
                                    <span class="o_stat_text">Lost</span>
                                </div>
                            </button>
                            <button name="%(action_mrp_workcenter_load_report_graph)d" type="action" class="oe_stat_button" icon="fa-bar-chart" context="{'search_default_workcenter_id': id}">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="workcenter_load" widget="statinfo" nolabel="1"/> Minutes</span>
                                    <span class="o_stat_text">Load</span>
                                </div>
                            </button>
                            <button name="%(mrp_workorder_report)d" type="action" class="oe_stat_button" icon="fa-bar-chart" context="{'search_default_workcenter_id': id, 'search_default_thisyear': True}">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="performance" widget="statinfo" nolabel="1"/>%</span>
                                    <span class="o_stat_text">Performance</span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <group>
                            <group>
                                <field name="active" invisible="1"/>
                                <field name="name" string="Work Center Name" required="True"/>
                                <field
                                    name="alternative_workcenter_ids"
                                    widget="many2many_tags"
                                    context="{'default_company_id': company_id}"
                                />
                            </group>
                            <group>
                                <field name="code"/>
                                <field name="resource_calendar_id" required="1"/>
                                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                            </group>
                        </group>
                        <notebook>
                            <page string="General Information">
                                <group>
                                    <group string="Production Information" name="capacity">
                                        <label for="time_efficiency"/>
                                        <div class="o_row">
                                            <field name="time_efficiency"/> %
                                        </div>
                                        <field name="capacity"/>

                                        <label for="oee_target"/>
                                        <div class="o_row">
                                            <field name="oee_target"/> %
                                        </div>
                                    </group>
                                    <group string="Costing Information" name="costing">
                                        <field name="costs_hour"/>
                                    </group>
                                    <group>
                                        <label for="time_start"/>
                                        <div>
                                            <field name="time_start" widget="float_time" class="oe_inline"/> minutes
                                        </div>
                                        <label for="time_stop"/>
                                        <div>
                                            <field name="time_stop" widget="float_time" class="oe_inline"/> minutes
                                        </div>
                                    </group>
                                </group>
                                <separator string="Description"/>
                                <field name="note" nolabel="1" placeholder="Description of the work center..."/>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_mrp_workcenter_search" model="ir.ui.view">
            <field name="name">mrp.workcenter.search</field>
            <field name="model">mrp.workcenter</field>
            <field name="arch" type="xml">
                <search string="Search for mrp workcenter">
                    <field name="name" string="Work Center" filter_domain="['|', ('name', 'ilike', self), ('code', 'ilike', self)]"/>
                    <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By...">
                        <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="mrp_workcenter_action" model="ir.actions.act_window">
            <field name="name">Work Centers</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workcenter</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_id" ref="mrp_workcenter_tree_view"/>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('mrp_workcenter_tree_view')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('mrp_workcenter_view_kanban')}),
                (0, 0, {'view_mode': 'form', 'view_id': ref('mrp_workcenter_view')})]"/>
            <field name="search_view_id" ref="view_mrp_workcenter_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new work center
              </p><p>
                Manufacturing operations are processed at Work Centers. A Work Center can be composed
                of workers and/or machines, they are used for costing, scheduling, capacity planning, etc.
                The Work Centers are defined on the Routing's operations.
              </p>
            </field>
        </record>

        <record id="mrp_workcenter_kanban_action" model="ir.actions.act_window">
            <field name="name">Work Centers Overview</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workcenter</field>
            <field name="view_mode">kanban,form</field>
            <field name="view_id" ref="mrp_workcenter_kanban"/>
            <field name="search_view_id" ref="view_mrp_workcenter_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new work center
              </p><p>
                Manufacturing operations are processed at Work Centers. A Work Center can be composed of
                workers and/or machines, they are used for costing, scheduling, capacity planning, etc.
                The Work Centers are defined on the Routing's operations.
              </p>
            </field>
        </record>

        <menuitem id="menu_view_resource_search_mrp"
            action="mrp_workcenter_action"
            groups="group_mrp_routings"
            parent="menu_mrp_bom"
            sequence="90"/>

        <menuitem id="menu_mrp_dashboard"
            name="Overview"
            action="mrp_workcenter_kanban_action"
            groups="group_mrp_routings"
            parent="menu_mrp_root"
            sequence="5"/>


    <record id="oee_loss_form_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.loss.form</field>
        <field name="model">mrp.workcenter.productivity.loss</field>
        <field name="arch" type="xml">
            <form string="Workcenter Productivity Loss">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="loss_id" widget="selection"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="oee_loss_tree_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.loss.tree</field>
        <field name="model">mrp.workcenter.productivity.loss</field>
        <field name="arch" type="xml">
            <tree string="Workcenter Productivity Loss" editable='bottom'>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="loss_type" string="Category"/>
            </tree>
        </field>
    </record>

    <record id="view_mrp_workcenter_productivity_loss_kanban" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.loss.kanban</field>
        <field name="model">mrp.workcenter.productivity.loss</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="name"/>
                <field name="manual"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <strong>Reason: </strong><field name="name"/>
                            </div>
                            <div>
                                <strong>Effectiveness Category: </strong><field name="loss_type"/>
                            </div>
                            <div>
                                <strong>Is a Blocking Reason? </strong>
                                <span class="float-right" title="Is a Blocking Reason?">
                                    <field name="manual" widget="boolean"/>
                                </span>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="oee_loss_search_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.loss.search</field>
        <field name="model">mrp.workcenter.productivity.loss</field>
        <field name="arch" type="xml">
            <search string="Operations">
                <field name="name"/>
            </search>
        </field>
    </record>


    <record model="ir.actions.act_window" id="mrp_workcenter_productivity_loss_action">
        <field name="name">Productivity Losses</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workcenter.productivity.loss</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="domain">[('manual', '=', 'True')]</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            No productivity loss defined
          </p>
        </field>
    </record>

    <record id="oee_search_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.search</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="arch" type="xml">
            <search string="Operations">
                <field name="workcenter_id"/>
                <field name="workcenter_id"/>
                <field name="loss_id"/>
                <separator/>
                <filter name="availability" string="Availability Losses" domain="[('loss_type','=','availability')]"/>
                <filter name="performance" string="Performance Losses" domain="[('loss_type','=','performance')]"/>
                <filter name="quality" string="Quality Losses" domain="[('loss_type','=','quality')]"/>
                <filter name="productive" string="Fully Productive" domain="[('loss_type','=','productive')]"/>
                <separator/>
                <group expand='0' string='Group by...'>
                    <filter string="User" name="user" context="{'group_by': 'create_uid'}"/>
                    <filter string='Workcenter' name="workcenter_group" context="{'group_by': 'workcenter_id'}"/>
                    <filter string="Loss Reason" name="loss_group" context="{'group_by': 'loss_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="oee_form_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.form</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="priority">5</field>
        <field name="arch" type="xml">
            <form string="Workcenter Productivity">
                <group>
                    <group>
                        <field name="production_id"/>
                        <field  name="workorder_id"/>
                        <field name="workcenter_id"/>
                        <field name="loss_id"/>
                        <field name="company_id" invisible="1"/>
                    </group><group>
                        <field name="date_start"/>
                        <field name="date_end"/>
                        <field name="duration"/>
                        <field name="company_id"/>
                    </group>
                    <field name="description"/>
                </group>
            </form>
        </field>
    </record>

    <record id="oee_tree_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.tree</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="arch" type="xml">
            <tree string="Workcenter Productivity">
                <field name="date_start"/>
                <field name="date_end"/>
                <field name="workcenter_id"/>
                <field name="user_id"/>
                <field name="loss_id"/>
                <field name="duration" string="Duration (minutes)" sum="Duration"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="oee_graph_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.graph</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="arch" type="xml">
            <graph string="Workcenter Productivity">
                <field name="workcenter_id"/>
                <field name="loss_id"/>
                <field name="duration" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="oee_pivot_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.pivot</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="arch" type="xml">
            <pivot string="Workcenter Productivity">
                <field name="date_start" type="row" interval="day"/>
                <field name="loss_type" type="col"/>
                <field name="duration" type="measure"/>
            </pivot>
        </field>
    </record>

    <record model="ir.actions.act_window" id="mrp_workcenter_productivity_report">
        <field name="name">Overall Equipment Effectiveness</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workcenter.productivity</field>
        <field name="view_mode">graph,pivot,tree,form</field>
        <field name="domain">[]</field>
        <field name="context">{'search_default_workcenter_group': 1, 'search_default_thismonth':True, 'create':False,'edit':False}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            Overall Equipment Effectiveness: no working or blocked time
          </p>
        </field>
    </record>

    <menuitem id="menu_mrp_workcenter_productivity_report"
          parent="menu_mrp_reporting"
          action="mrp_workcenter_productivity_report"
          groups="group_mrp_routings"
          sequence="12"/>
    </data>
</odoo>

```

## File: views\mrp_workorder_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_mrp_production_work_order_search" model="ir.ui.view">
        <field name="name">mrp.production.work.order.search</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <search>
                <field name="production_id"/>
                <field name="workcenter_id"/>
                <filter string="Ready" name="ready" domain="[('state','=','ready')]"/>
                <filter string="Pending" name="pending" domain="[('state','=','pending')]"/>
                <filter string="In Progress" name="progress" domain="[('state','=','progress')]"/>
                <filter string="Done" name="done" domain="[('state','=', 'done')]"/>
                <filter string="Late" name="late" domain="[('date_planned_start','&lt;=',time.strftime('%%Y-%%m-%%d'))]"/>
                <separator/>
                <filter string="Start Date" name="date_start_filter" date="date_start"/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
            </search>
        </field>
    </record>

    <record id="mrp_workorder_delta_report" model="ir.actions.act_window">
        <field name="name">Work Orders Deviation</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">graph,pivot,tree,form,gantt,calendar</field>
        <field name="context">{'search_default_workcenter_id': active_id}</field>
        <field name="domain">[]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
              No data to display
            </p><p>
              You will get here statistics about the
              work orders duration related to this routing.
            </p>
        </field>
    </record>

    <record id="action_mrp_routing_time" model="ir.actions.act_window">
        <field name="name">Work Orders</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">graph,pivot,tree,form,gantt,calendar</field>
        <field name="context">{'search_default_done': True}</field>
        <field name="search_view_id" ref="view_mrp_production_work_order_search"/>
        <field name="domain">[('production_id.routing_id', '=', active_id), ('state', '=', 'done')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
              No data to display
            </p><p>
              You will get here statistics about the
              work orders duration related to this routing.
            </p>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_mrp_workorder_production_specific">
        <field name="name">Work Orders</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">tree,form,gantt,calendar,pivot,graph</field>
        <field name="domain">[('production_id', '=', active_id)]</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Start a new work order
          </p><p>
            Work Orders are operations to be processed at a Work Center to realize a
            Manufacturing Order. Work Orders are trigerred by Manufacturing Orders,
            they are based on the Routing defined on these ones
            </p>
        </field>
    </record>

    <record model="ir.ui.view" id="mrp_production_workorder_tree_view_inherit">
        <field name="name">mrp.production.work.order.tree</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <tree string="Work Orders" delete="0" create="0" decoration-success="date_planned_start&gt;=current_date and state == 'ready'" decoration-muted="state in ('done','cancel')" decoration-danger="date_planned_start&lt;current_date and state in ('ready')">
                <field name="name"/>
                <field name="date_planned_start"/>
                <field name="workcenter_id" widget="selection"/>
                <field name="production_id"/>
                <field name="product_id"/>
                <field name="qty_production"/>
                <field name="product_uom_id"/>
                <field name="state"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="mrp_production_workorder_form_view_inherit">
        <field name="name">mrp.production.work.order.form</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <form string="Production Workcenter" delete="0" create="0">
            <field name="is_user_working" invisible="1"/>
            <field name="working_state" invisible="1"/>
            <field name="production_state" invisible="1"/>
            <field name="allowed_lots_domain" invisible="1"/>
            <header>
                <button name="button_finish" type="object" string="Finish Order" attrs="{'invisible': ['|', ('state', '!=', 'progress'), ('is_produced', '=', False)]}" class="btn-info"/>
                <button name="button_start" type="object" string="Start Working" attrs="{'invisible': ['|', ('working_state', '=', 'blocked'), ('state', '!=', 'pending')]}"/>
                <button name="button_start" type="object" string="Start Working" attrs="{'invisible': ['|', ('working_state', '=', 'blocked'), ('state', '!=', 'ready')]}" class="btn-success"/>
                <button name="record_production" type="object" string="Done" class="btn-success" attrs="{'invisible': ['|', '|', '|', ('is_produced', '=', True), ('working_state', '=', 'blocked'), ('state', '!=', 'progress'), ('is_user_working', '=', False)]}"/>
                <button name="button_pending" type="object" string="Pause" class="btn-warning" attrs="{'invisible': ['|', '|', ('working_state', '=', 'blocked'), ('state', 'in', ('done', 'pending', 'ready', 'cancel')), ('is_user_working', '=', False)]}"/>
                <button name="%(act_mrp_block_workcenter_wo)d" type="action" context="{'default_workcenter_id': workcenter_id}" string="Block" class="btn-danger" attrs="{'invisible': ['|', '|', ('working_state', '=', 'blocked'), ('state', 'in', ('done', 'pending', 'ready', 'cancel')), ('is_user_working', '=', False)]}"/>
                <button name="button_unblock" type="object" string="Unblock" class="btn-danger" attrs="{'invisible': [('working_state', '!=', 'blocked')]}"/>
                <button name="button_start" type="object" string="Continue Production" attrs="{'invisible': ['|', '|', '|', ('production_state', '=', 'done'), ('working_state', '=', 'blocked'), ('is_user_working', '=', True), ('state', '!=', 'progress')]}"/>
                <button name="button_scrap" type="object" string="Scrap" attrs="{'invisible': [('state', 'in', ('done', 'cancel'))]}"/>
                <field name="state" widget="statusbar" statusbar_visible="pending,ready,progress,done"/>
            </header>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button class="oe_stat_button" name="action_see_move_scrap" type="object" icon="fa-arrows-v" attrs="{'invisible': [('scrap_count', '=', 0)]}">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value"><field name="scrap_count"/></span>
                            <span class="o_stat_text">Scraps</span>
                        </div>
                    </button>
                </div>
                <group>
                    <group>
                        <field name="product_id" string="To Produce"/>
                        <label for="qty_produced" string="Quantity Produced"/>
                        <div class="o_row">
                            <field name="qty_produced"/> /
                            <field name="qty_production"/>
                            <field name="product_uom_id"/>
                            <field name="production_availability" nolabel="1" widget="bullet_state" options="{'classes': {'assigned': 'success', 'waiting': 'danger'}}" attrs="{'invisible': [('state', '=', 'done')]}"/>
                        </div>
                        <field name="is_produced" invisible="1"/>
                    </group>
                </group>
                <notebook>
                <page string="Work Instruction" attrs="{'invisible': [('worksheet', '=', False),('worksheet_google_slide', '=', False)]}">
                    <field name="worksheet_type" invisible="1"/>
                    <field name="worksheet_google_slide" widget="embed_viewer" attrs="{'invisible': [('worksheet_type', '=', 'pdf')]}"/>
                    <field name="worksheet" widget="pdf_viewer" attrs="{'invisible': [('worksheet_type', '=', 'google_slide')]}"/>
                </page>
                <page string="Current Production">
                    <group>
                        <group>
                            <field name="qty_producing" string="Quantity in Production" attrs="{'readonly': ['|', ('product_tracking', '=', 'serial'), ('state', 'in', ('done', 'cancel'))]}"/>
                            <field name="company_id" invisible="1"/>
                            <field name="finished_lot_id" context="{'default_product_id': product_id, 'default_company_id': company_id}" attrs="{'invisible': [('product_tracking', '=', 'none')]}" groups="stock.group_production_lot"/>
                            <field name="product_tracking" invisible="1"/>
                        </group>
                    </group>
                    <h4 attrs="{'invisible': [('raw_workorder_line_ids', '=', [])]}">Components</h4>
                    <field name="raw_workorder_line_ids" attrs="{'invisible': [('raw_workorder_line_ids', '=', [])], 'readonly': [('state', 'in', 'cancel', 'done')]}">
                        <tree editable="bottom" create="0" delete="0">
                            <field name="product_id"/>
                            <field name="product_tracking" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                            <field name="lot_id" attrs="{'readonly': [('product_tracking', '=', 'none')]}" context="{'default_company_id': company_id, 'default_product_id': product_id, 'active_mo_id': parent.production_id}"/>
                            <field name="qty_to_consume" readonly="1" force_save="1"/>
                            <field name="qty_reserved" readonly="1"/>
                            <field name="qty_done" attrs="{'column_invisible': [('parent.state', 'not in', ('progress', 'done'))]}"/>
                            <field name="product_uom_id" invisible="1"/>
                            <field name="move_id" invisible="1"/>
                        </tree>
                    </field>
                    <h4 attrs="{'invisible': [('finished_workorder_line_ids', '=', [])]}">Finished Products</h4>
                    <field name="finished_workorder_line_ids" attrs="{'invisible': [('finished_workorder_line_ids', '=', [])], 'readonly': [('state', 'in', 'cancel', 'done')]}">
                        <tree editable="bottom" create="0" delete="0">
                            <field name="product_id"/>
                            <field name="product_tracking" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                            <field name="lot_id" attrs="{'readonly': [('product_tracking', '=', 'none')]}" context="{'default_company_id': company_id, 'default_product_id': product_id}"/>
                            <field name="qty_to_consume" readonly="1" string="To Produce" force_save="1"/>
                            <field name="qty_done" string="Produced"/>
                            <field name="product_uom_id" invisible="1"/>
                            <field name="move_id" invisible="1"/>
                        </tree>
                    </field>
                </page>
                <page string="Time Tracking" groups="mrp.group_mrp_manager">
                    <group>
                        <group>
                            <label for="date_planned_start" string="Planned Date"/>
                            <div class="o_row">
                                <field name="date_planned_start" class="mr8"/>
                                <div attrs="{'invisible': [('date_planned_start', '=', False)]}" class="o_row">
                                    <strong attrs="{'invisible': [('date_planned_finished', '=', False)]}" class="mr8">to</strong>
                                    <strong class="oe_edit_only mr8" attrs="{'invisible': [('date_planned_finished', '!=', False)]}">to</strong>
                                    <field name="date_planned_finished"/>
                                </div>
                            </div>
                            <label for="date_start" string="Effective Date"/>
                            <div class="o_row">
                                <field name="date_start" readonly="1"/>
                                <div  attrs="{'invisible': [('date_finished', '=', False)]}">
                                    <strong class="mr8">to</strong>
                                    <field name="date_finished" readonly="1"/>
                                </div>
                            </div>
                        </group>
                        <group>
                            <label for="duration_expected"/>
                            <div>
                                <field name="duration_expected" widget="float_time" class="oe_inline"/>
                                minutes
                            </div>
                            <label for="duration"/>
                            <div>
                                <button style="pointer-events: none;" class="oe_inline badge badge-secondary">
                                    <field name="duration" widget="mrp_time_counter" help="Time the currently logged user spent on this workorder."/>
                                </button>
                            </div>
                        </group>
                    </group>
                    <group>
                        <field name="time_ids" nolabel="1" context="{'default_workcenter_id': workcenter_id, 'default_workorder_id': id}">
                            <tree>
                                <field name="date_start"/>
                                <field name="date_end"/>
                                <field name="duration" widget="float_time" sum="Total duration"/>
                                <field name="user_id"/>
                                <field name="workcenter_id" invisible="1"/>
                                <field name="company_id" invisible="1"/>
                                <field name="loss_id" string="Productivity"/>
                            </tree>
                            <form>
                                <group>
                                    <group>
                                        <field name="date_start"/>
                                        <field name="date_end"/>
                                        <field name="duration" widget="float_time"/>
                                        <field name="company_id" invisible="1"/>
                                    </group>
                                    <group>
                                        <field name="user_id"/>
                                        <field name="workcenter_id"/>
                                        <field name="company_id" invisible="1"/>
                                        <field name="loss_id"/>
                                    </group>
                                </group>
                            </form>
                        </field>
                    </group>
                </page>
                <page string="Miscellaneous" name="workorder_page_misc" groups="mrp.group_mrp_manager">
                    <group>
                        <group>
                            <field name="workcenter_id"/>
                            <field name="production_id" readonly="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                </page>
                </notebook>
            </sheet>
            <div class="oe_chatter" groups="mrp.group_mrp_manager">
                <field name="message_follower_ids" widget="mail_followers"/>
                <field name="activity_ids" widget="mail_activity"/>
                <field name="message_ids" widget="mail_thread"/>
            </div>
            </form>
        </field>
    </record>

    <record id="view_mrp_production_workorder_form_view_filter" model="ir.ui.view">
        <field name="name">mrp.production.work.order.select</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <search string="Search Work Orders">
                <field name="name" string="Work Order"/>
                <field name="workcenter_id"/>
                <field name="production_id"/>
                <filter string="In Progress" name="progress" domain="[('state', '=', 'progress')]"/>
                <filter string="Ready" name="ready" domain="[('state', '=', 'ready')]"/>
                <filter string="Pending" name="pending" domain="[('state', '=', 'pending')]"/>
                <filter string="Finished" name="finish" domain="[('state', '=', 'done')]"/>
                <filter string="Available" name="available" domain="[('production_availability', '=', 'assigned')]"/>
                <separator/>
                <filter string="Late" name="late" domain="['&amp;', ('date_planned_start', '&lt;', current_date), ('state', '=', 'ready')]"
                    help="Production started late"/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Work Center" name="work_center" domain="[]" context="{'group_by': 'workcenter_id'}"/>
                    <filter string="Manufacturing Order" name="production" domain="[]" context="{'group_by': 'production_id'}"/>
                    <filter string="Status" name="status" domain="[]" context="{'group_by': 'state'}"/>
                    <filter string="Scheduled Date" name="scheduled_month" domain="[]" context="{'group_by': 'date_planned_start'}"/>
                </group>
             </search>
        </field>
    </record>

    <record id="workcenter_line_calendar" model="ir.ui.view">
        <field name="name">mrp.production.work.order.calendar</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <calendar date_stop="date_planned_finished" date_start="date_planned_start" string="Operations" color="workcenter_id" event_limit="5">
                <field name="workcenter_id"/>
                <field name="production_id"/>
                <field name="state"/>
            </calendar>
        </field>
    </record>

    <record id="workcenter_line_graph" model="ir.ui.view">
        <field name="name">mrp.production.work.order.graph</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <graph string="Operations">
                <field name="production_id"/>
                <field name="duration" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="workcenter_line_pivot" model="ir.ui.view">
        <field name="name">mrp.production.work.order.pivot</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <pivot string="Operations">
                <field name="date_start"/>
                <field name="operation_id"/>
                <field name="duration" type="measure"/>
                <field name="duration_expected" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="workcenter_line_gantt_production" model="ir.ui.view">
        <field name="name">mrp.production.work.order.gantt.production</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <gantt class="o_mrp_workorder_gantt" date_stop="date_planned_finished" date_start="date_planned_start" string="Operations" default_group_by="production_id" create="0"
                progress="progress" plan="0"
                decoration-danger="date_planned_start &lt; current_date and state in ['pending', 'ready']"
                decoration-success="state == 'done'"
                decoration-warning="state == 'cancel'"
                color="workcenter_id">
                <field name="date_planned_start"/>
                <field name="state"/>
                <field name="workcenter_id"/>

                <templates>
                    <div t-name="gantt-popover" class="container-fluid">
                        <div class="row no-gutters">
                            <div class="col">
                                <ul class="pl-1 mb-0 list-unstyled">
                                    <li><strong>Start Date: </strong> <t t-esc="userTimezoneStartDate.format('L LTS')"/></li>
                                    <li><strong>Stop Date: </strong> <t t-esc="userTimezoneStopDate.format('L LTS')"/></li>
                                    <li><strong>Workcenter: </strong> <t t-esc="workcenter_id[1]"/></li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </templates>
            </gantt>
        </field>
    </record>

    <record id="mrp_workorder_view_gantt" model="ir.ui.view">
        <field name="name">mrp.workorder.view.gantt</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <gantt class="o_mrp_workorder_gantt" date_stop="date_planned_finished" date_start="date_planned_start" string="Operations" default_group_by="workcenter_id" create="0"
                progress="progress" plan="0"
                decoration-danger="date_planned_start &lt; current_date and state in ['pending', 'ready']"
                decoration-success="state == 'done'"
                decoration-warning="state == 'cancel'"
                color="production_id">

                <field name="date_planned_start"/>
                <field name="state"/>
                <field name="workcenter_id"/>

                <templates>
                    <div t-name="gantt-popover" class="container-fluid">
                        <div class="row no-gutters">
                            <div class="col">
                                <ul class="pl-1 mb-0 list-unstyled">
                                    <li><strong>Start Date: </strong> <t t-esc="userTimezoneStartDate.format('L LTS')"/></li>
                                    <li><strong>Stop Date: </strong> <t t-esc="userTimezoneStopDate.format('L LTS')"/></li>
                                    <li><strong>Workcenter: </strong> <t t-esc="workcenter_id[1]"/></li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </templates>
            </gantt>
        </field>
    </record>

    <record model="ir.ui.view" id="workcenter_line_kanban">
        <field name="name">mrp.production.work.order.kanban</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <kanban class="oe_background_grey o_kanban_dashboard o_mrp_workorder_kanban" create="0">
                <field name="name"/>
                <field name="production_id"/>
                <field name="state"/>
                <field name="is_user_working"/>
                <field name="working_user_ids"/>
                <field name="last_working_user_id"/>
                <field name="working_state"/>
                <field name="workcenter_id"/>
                <field name="product_id"/>
                <field name="qty_production"/>
                <field name="product_uom_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click">
                            <div class="o_kanban_card_header o_kanban_record_top">
                                <div class="o_kanban_record_headings o_kanban_card_header_title">
                                    <strong class="o_primary ml8">
                                        <span><t t-esc="record.production_id.value"/></span> - <span><t t-esc="record.name.value"/></span>
                                    </strong>
                                </div>
                                <div class="o_kanban_manage_button_section">
                                    <h2>
                                        <span t-attf-class="badge #{['pending'].indexOf(record.state.raw_value) > -1 ? 'badge-warning' :['progress'].indexOf(record.state.raw_value) > -1 ? 'badge-secondary' : ['ready'].indexOf(record.state.raw_value) > -1 ? 'badge-primary' : ['done'].indexOf(record.state.raw_value) > -1 ? 'badge-success' : 'badge-danger'}">
                                            <t t-esc="record.state.value"/>
                                        </span>
                                    </h2>
                                </div>
                            </div>
                            <div class="container o_kanban_record_bottom">
                                <h5 class="oe_kanban_bottom_left">
                                    <span><t t-esc="record.product_id.value"/>, </span> <span><t t-esc="record.qty_production.value"/> <t t-esc="record.product_uom_id.value"/></span>
                                </h5>
                                <div class="oe_kanban_bottom_right" t-if="record.state.raw_value == 'progress'">
                                    <span t-if="record.working_state.raw_value != 'blocked' and record.working_user_ids.raw_value.length > 0"><i class="fa fa-play" role="img" aria-label="Run" title="Run"/></span>
                                    <span t-if="record.working_state.raw_value != 'blocked' and record.working_user_ids.raw_value.length == 0 and record.last_working_user_id.raw_value"><i class="fa fa-pause" role="img" aria-label="Pause" title="Pause"/></span>
                                    <span t-if="record.working_state.raw_value == 'blocked' and (record.working_user_ids.raw_value.length == 0 or record.last_working_user_id.raw_value)"><i class="fa fa-stop" role="img" aria-label="Stop" title="Stop"/></span>
                                    <t t-if="record.last_working_user_id.raw_value">
                                        <img t-att-src="kanban_image('res.users', 'image_128', record.last_working_user_id.raw_value)" class="oe_kanban_avatar" alt="Avatar"/>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_mrp_workorder_workcenter">
        <field name="name">Work Orders Planning</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">gantt,tree,form,calendar,pivot,graph</field>
        <field name="search_view_id" ref="view_mrp_production_workorder_form_view_filter"/>
        <field name="view_id" ref="mrp_workorder_view_gantt"/>
        <field name="context">{'search_default_work_center': True, 'search_default_ready': True, 'search_default_progress': True, 'search_default_pending': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Start a new work order
          </p><p>
            Work Orders are operations to be processed at a Work Center to realize a
            Manufacturing Order. Work Orders are trigerred by Manufacturing Orders,
            they are based on the Routing defined on these ones
            </p>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_mrp_workorder_production">
        <field name="name">Work Orders Planning</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="domain">[('production_state','not in',('done','cancel'))]</field>
        <field name="view_mode">gantt,tree,form,calendar,pivot,graph</field>
        <field name="search_view_id" ref="view_mrp_production_workorder_form_view_filter"/>
        <field name="view_id" ref="workcenter_line_gantt_production"/>
        <field name="context">{'search_default_production': True, 'search_default_ready': True, 'search_default_progress': True, 'search_default_pending': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Start a new work order
          </p><p>
            To manufacture or assemble products, and use components and
            finished products you must also handle manufacturing operations.
            Manufacturing operations are often called Work Orders. The various
            operations will have different impacts on the costs of
            manufacturing and planning depending on the available workload.
          </p>
        </field>
    </record>

    <record model="ir.actions.act_window" id="mrp_workorder_todo">
        <field name="name">Work Orders</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">tree,kanban,form,calendar,pivot,graph,gantt</field>
        <field name="search_view_id" ref="view_mrp_production_workorder_form_view_filter"/>
        <field name="context">{'search_default_ready': True, 'search_default_progress': True, 'search_default_pending': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Start a new work order
          </p><p>
            Work Orders are operations to be processed at a Work Center to realize a
            Manufacturing Order. Work Orders are trigerred by Manufacturing Orders,
            they are based on the Routing defined on these ones
            </p>
        </field>
    </record>

    <record id="view_workcenter_load_pivot" model="ir.ui.view">
        <field name="name">report.workcenter.load.pivot</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <pivot string="Work Center Loads">
                <field name="duration_expected" type="measure"/>
                <field name="workcenter_id" type="row"/>
                <field name="production_date" type="row" interval="day"/>
            </pivot>
        </field>
    </record>

    <record id="view_work_center_load_graph" model="ir.ui.view">
        <field name="name">report.workcenter.load.graph</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <graph string="Work Center load" type="bar">
                <field name="production_date" interval="day"/>
                <field name="workcenter_id"/>
                <field name="duration_expected" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="action_mrp_workcenter_load_report_graph" model="ir.actions.act_window">
        <field name="name">Work Center Loads</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">graph,pivot</field>
        <field name="view_id" ref="view_workcenter_load_pivot"/>
    </record>

    <record id="action_mrp_workcenter_load_report_pivot" model="ir.actions.act_window.view">
        <field name="view_mode">graph</field>
        <field name="view_id" ref="view_work_center_load_graph"/>
        <field name="act_window_id" ref="action_mrp_workcenter_load_report_graph"/>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Product Template -->
        <record id="view_mrp_product_template_form_inherited" model="ir.ui.view">
            <field name="name">product.form.mrp.inherited</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="stock.view_template_property_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='route_ids']" position="after">
                    <label for="produce_delay" attrs="{'invisible':[('type','=','service')]}"/>
                    <div attrs="{'invisible':[('type','=','service')]}">
                        <field name="produce_delay" class="oe_inline"/> days
                    </div>
                </xpath>
            </field>
        </record>

        <record id="mrp_product_template_search_view" model="ir.ui.view">
            <field name="name">mrp.product.template.search</field>
            <field name="model">product.template</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="product.product_template_search_view"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='consumable']" position="after">
                    <separator/>
                    <filter string="Manufactured Products" name="manufactured_products" domain="[('bom_ids', '!=', False)]"/>
                    <filter string="BoM Components" name="components" domain="[('bom_line_ids', '!=', False)]"/>
                </xpath>
            </field>
        </record>

        <record id="mrp_product_product_search_view" model="ir.ui.view">
            <field name="name">mrp.product.product.search</field>
            <field name="model">product.product</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="product.product_search_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='consumable']" position="after">
                    <separator/>
                    <filter string="Manufactured Products" name="manufactured_products" domain="[('bom_ids', '!=', False)]"/>
                    <filter string="BoM Components" name="components" domain="[('bom_line_ids', '!=', False)]"/>
                </xpath>
            </field>
        </record>

        <record id="product_template_action" model="ir.actions.act_window">
            <field name="name">Products</field>
            <field name="res_model">product.template</field>
            <field name="search_view_id" ref="mrp_product_template_search_view"/>
            <field name="view_mode">kanban,tree,form</field>
            <field name="context">{"search_default_consumable": 1, 'default_type': 'product'}</field>
        </record>

        <menuitem id="menu_mrp_product_form"
            name="Products"
            action="product_template_action"
            parent="menu_mrp_bom" sequence="1"/>

        <record id="mrp_product_variant_action" model="ir.actions.act_window">
            <field name="name">Product Variants</field>
            <field name="res_model">product.product</field>
            <field name="search_view_id" ref="mrp_product_product_search_view"/>
            <field name="view_mode">kanban,tree,form</field>
        </record>

        <menuitem id="product_variant_mrp" name="Product Variants"
            action="mrp_product_variant_action"
            parent="menu_mrp_bom" groups="product.group_product_variant" sequence="2"/>


        <record id="product_template_form_view_bom_button" model="ir.ui.view">
            <field name="name">product.template.procurement</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="stock.product_template_form_view_procurement_button"/>
            <field name="groups_id" eval="[(4, ref('mrp.group_mrp_user'))]"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_open_product_lot']" position="after">
                    <button class="oe_stat_button" name="%(template_open_bom)d" type="action"
                        attrs="{'invisible':[('type', 'not in', ['product', 'consu'])]}" icon="fa-flask">
                        <field string="Bill of Materials" name="bom_count" widget="statinfo" />
                    </button>
                    <button class="oe_stat_button" name="action_used_in_bom" type="object"
                        attrs="{'invisible':['|',('type', 'not in', ['product', 'consu']), ('used_in_bom_count', '=', 0)]}" icon="fa-level-up">
                        <field string="Used In" name="used_in_bom_count" widget="statinfo" />
                    </button>
                    <button class="oe_stat_button" name="action_view_mos" type="object"
                        attrs="{'invisible': ['|', '|', ('type', 'not in', ['product', 'consu']), ('bom_count', '=', 0), ('mrp_product_qty', '=', 0)]}" icon="fa-list-alt" help="Manufactured in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="mrp_product_qty" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Manufactured</span>
                        </div>
                    </button>
                </xpath>
            </field>
        </record>

        <record id="product_product_form_view_bom_button" model="ir.ui.view">
            <field name="name">product.product.procurement</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="stock.product_form_view_procurement_button"/>
            <field name="groups_id" eval="[(4, ref('mrp.group_mrp_user'))]"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_open_product_lot']" position="after">
                    <button class="oe_stat_button" name="action_view_bom" type="object"
                        attrs="{'invisible':[('type', 'not in', ['product', 'consu'])]}" icon="fa-flask">
                        <field string="Bill of Materials" name="bom_count" widget="statinfo" />
                    </button>
                    <button class="oe_stat_button" name="action_used_in_bom" type="object"
                        attrs="{'invisible':['|',('type', 'not in', ['product', 'consu']), ('used_in_bom_count', '=', 0)]}" icon="fa-level-up">
                        <field string="Used In" name="used_in_bom_count" widget="statinfo" />
                    </button>
                    <button class="oe_stat_button" name="action_view_mos" type="object"
                        attrs="{'invisible': ['|', '|', ('type', 'not in', ['product', 'consu']), ('bom_count', '=', 0), ('mrp_product_qty', '=', 0)]}" icon="fa-list-alt" help="Manufactured in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="mrp_product_qty" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Manufactured</span>
                        </div>
                    </button>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.mrp</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="35"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form" />
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="Manufacturing" string="Manufacturing" data-key="mrp" groups="mrp.group_mrp_manager">
                        <h2>Operations</h2>
                        <div class="row mt16 o_settings_container">
                            <div class="col-lg-6 col-12 o_setting_box" id="work_order" title="Work Order Operations allow you to create and manage the manufacturing operations that should be followed within your work centers in order to produce a product. They are attached to bills of materials that will define the required components.">
                                <div class="o_setting_left_pane">
                                    <field name="group_mrp_routings"/>
                                    <field name="module_mrp_workorder" invisible="1"/>
                                </div>
                                <div class="o_setting_right_pane" id="workorder_settings">
                                    <label for="group_mrp_routings" string="Work Orders"/>
                                    <div class="text-muted">
                                        Process operations at specific work centers based on the routing
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('group_mrp_routings','=',False)]}">
                                        <div class="mt8">
                                            <div>
                                                <button name="%(mrp.mrp_routing_action)d" icon="fa-arrow-right" type="action" string="Routings" class="btn-link"/>
                                            </div>
                                            <div>
                                                <button name="%(mrp.mrp_workcenter_action)d" icon="fa-arrow-right" type="action" string="Work Centers" class="btn-link"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-lg-6 col-12 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="module_mrp_subcontracting"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_mrp_subcontracting"/>
                                    <div class="text-muted">
                                        Subcontract the production of some products
                                    </div>
                                </div>
                            </div>
                            <div class="col-lg-6 col-12 o_setting_box" id="quality_control">
                                <div class="o_setting_left_pane">
                                    <field name="module_quality_control" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_quality_control"/>
                                    <div class="text-muted">
                                        Add quality checks to your work orders
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="row mt16 o_settings_container">
                            <div class="col-lg-6 col-12 o_setting_box" id="mrp_byproduct" title="Add by-products to bills of materials. This can be used to get several finished products as well. Without this option you only do: A + B = C. With the option: A + B = C + D.">
                                <div class="o_setting_left_pane">
                                    <field name="group_mrp_byproducts"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_mrp_byproducts"/>
                                    <div class="text-muted">
                                        Produce residual products (A + B -> C + D)
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Planning</h2>
                        <div class="row mt16 o_settings_container">
                            <div class="col-lg-6 col-12 o_setting_box" id="mrp_mps" title="Using a MPS report to schedule your reordering and manufacturing operations is useful if you have long lead time and if you produce based on sales forecasts.">
                                <div class="o_setting_left_pane">
                                    <field name="module_mrp_mps" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_mrp_mps"/>
                                    <div class="text-muted">
                                        Plan manufacturing or purchase orders based on forecasts
                                    </div>
                                    <div class="content-group" id="content_mrp_mps"/>
                                </div>
                            </div>
                            <div class="col-lg-6 col-12 o_setting_box" id="security_lead_time">
                                <div class="o_setting_left_pane">
                                    <field name="use_manufacturing_lead"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Security Lead Time" for="use_manufacturing_lead"/>
                                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                                    <div class="text-muted">
                                        Schedule manufacturing orders earlier to avoid delays
                                     </div>
                                     <div class="content-group" attrs="{'invisible': [('use_manufacturing_lead','=',False)]}">
                                        <div class="mt16" >
                                            <field name="manufacturing_lead" class="oe_inline"/> days
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="action_mrp_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'mrp', 'bin_size': False}</field>
        </record>

        <menuitem id="menu_mrp_config" name="Settings" parent="menu_mrp_configuration"
            sequence="0" action="action_mrp_configuration" groups="base.group_system"/>
    </data>
</odoo>

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record model="ir.actions.act_window" id="action_mrp_production_moves">
            <field name="name">Inventory Moves</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">stock.move.line</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">['|', ('move_id.raw_material_production_id', '=', active_id), ('move_id.production_id', '=', active_id)]</field>
        </record>

        <record id="view_stock_move_lots" model="ir.ui.view">
            <field name="name">stock.move.lots.form</field>
            <field name="model">stock.move</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <form string="Lots">
                    <field name="state" invisible="1" force_save="1"/>
                    <group>
                        <group>
                            <field name="product_id" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <label for="product_uom_qty"/>
                            <div class="o_row">
                                <span><field name="product_uom_qty" attrs="{'readonly': [('parent.state', '!=', 'draft')]}" nolabel="1"/></span>
                                <span><field name="product_uom" readonly="1" force_save="1" nolabel="1"/></span>
                            </div>
                            <label for="quantity_done" attrs="{'invisible': [('parent.state', '=', 'draft')]}"/>
                            <div class="o_row" attrs="{'invisible': [('parent.state', '=', 'draft')]}">
                                <span><field name="quantity_done" attrs="{'readonly': ['|', '|', ('move_line_ids', '!=', False), ('is_locked', '=', True), '|', ('finished_lots_exist', '=', True), ('has_tracking', '!=', 'none')]}" nolabel="1"/></span>
                                <span> / </span>
                                <span><field name="reserved_availability" nolabel="1"/></span>
                            </div>
                            <field name="company_id" invisible="1"/>
                            <field name="is_done" invisible="1"/>
                            <field name="workorder_id" invisible="1"/>
                            <field name="bom_line_id" invisible="1"/>
                            <field name="location_id" invisible="1"/>
                            <field name="location_dest_id" invisible="1"/>
                            <field name="picking_type_id" invisible="1"/>
                            <field name="operation_id" invisible="1"/>
                            <field name="warehouse_id" invisible="1"/>
                            <field name="production_id" invisible="1"/>
                            <field name="date" invisible="1"/>
                            <field name="date_expected" invisible="1"/>
                            <field name="raw_material_production_id" invisible="1"/>
                            <field name="is_locked" invisible="1"/>
                            <field name="name" invisible="1"/>
                            <field name="has_tracking" invisible="1"/>
                            <field name="order_finished_lot_ids" invisible="1"/>
                            <field name="finished_lots_exist" invisible="1"/>
                        </group>
                    </group>
                    <field name="move_line_ids" attrs="{'invisible': [('parent.state', '=', 'draft')], 'readonly': ['|', ('is_locked', '=', True), ('state', '=', 'cancel')]}" context="{'default_workorder_id': workorder_id, 'default_product_uom_id': product_uom, 'default_product_id': product_id,  'default_location_id': location_id, 'default_location_dest_id': location_dest_id, 'default_production_id': production_id or raw_material_production_id, 'default_company_id': company_id}">
                        <tree editable="bottom" decoration-success="product_uom_qty==qty_done" decoration-danger="(product_uom_qty &gt; 0) and (qty_done&gt;product_uom_qty)">
                            <field name="lot_id" attrs="{'column_invisible': [('parent.has_tracking', '=', 'none')]}" context="{'default_product_id': parent.product_id}"/>
                            <field name="lot_produced_ids" widget="many2many_tags" options="{'no_open': True, 'no_create': True}" domain="[('id', 'in', parent.order_finished_lot_ids)]" invisible="not context.get('final_lots')"/>
                            <field name="owner_id" groups="stock.group_tracking_owner" options="{'no_open': True, 'no_create': True}" optional="hide"/>
                            <field name="package_id" groups="stock.group_tracking_lot" options="{'no_open': True, 'no_create': True}" optional="hide"/>
                            <field name="product_uom_qty" string="Reserved" readonly="1" optional="show"/>
                            <field name="qty_done"/>
                            <field name="workorder_id" invisible="1"/>
                            <field name="product_id" invisible="1"/>
                            <field name="product_uom_id" invisible="1"/>
                            <field name="location_id" invisible="1"/>
                            <field name="location_dest_id" invisible="1"/>
                            <field name="production_id" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                        </tree>
                    </field>
                </form>
            </field>
        </record>

        <record id="view_stock_move_raw_tree" model="ir.ui.view">
            <field name="name">stock.move.raw.tree</field>
            <field name="model">stock.move</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <tree delete="0" default_order="is_done,sequence" decoration-muted="is_done" decoration-warning="quantity_done - product_uom_qty &gt; 0.0001" decoration-success="not is_done and quantity_done - product_uom_qty &lt; 0.0001" decoration-danger="not is_done and reserved_availability &lt; product_uom_qty and product_uom_qty - reserved_availability &gt; 0.0001">
                    <field name="product_id" required="1"/>
                    <field name="company_id" invisible="1"/>
                    <field name="product_uom_category_id" invisible="1"/>
                    <field name="name" invisible="1"/>
                    <field name="unit_factor" invisible="1"/>
                    <field name="product_uom" groups="uom.group_uom"/>
                    <field name="date" invisible="1"/>
                    <field name="date_expected" invisible="1"/>
                    <field name="picking_type_id" invisible="1"/>
                    <field name="has_tracking" invisible="1"/>
                    <field name="operation_id" invisible="1"/>
                    <field name="needs_lots" readonly="1" groups="stock.group_production_lot"/>
                    <field name="is_done" invisible="1"/>
                    <field name="bom_line_id" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                    <field name="location_id" invisible="1"/>
                    <field name="warehouse_id" invisible="1"/>
                    <field name="location_dest_id" domain="[('id', 'child_of', parent.location_dest_id)]" invisible="1"/>
                    <field name="state" invisible="1" force_save="1"/>
                    <field name="product_uom_qty" string="To Consume"/>
                    <field name="reserved_availability" attrs="{'invisible': [('is_done', '=', True)], 'column_invisible': [('parent.state', 'in', ('draft', 'done'))]}" string="Reserved"/>
                    <field name="quantity_done" string="Consumed" attrs="{'column_invisible': [('parent.state', '=', 'draft')]}" readonly="1"/>
                    <field name="group_id" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="view_move_kanban_inherit_mrp" model="ir.ui.view">
            <field name="name">stock.move.kanban.inherit.mrp</field>
            <field name="model">stock.move</field>
            <field name="inherit_id" ref="stock.view_move_kandan"/>
            <field name="arch" type="xml">
                <xpath expr="//templates" position="before">
                    <field name="bom_line_id"/>
                </xpath>
            </field>
        </record>

        <record id="view_finisehd_move_line" model="ir.ui.view">
            <field name="name">mrp.finished.move.line.form</field>
            <field name="priority">1000</field>
            <field name="model">stock.move.line</field>
            <field name="arch" type="xml">
                <form string="Finished Product">
                    <group>
                        <group>
                            <field name="company_id" invisible="1"/>
                            <field name="product_id" readonly="1"/>
                            <label for="qty_done" string ="Quantity"/>
                            <div class="o_row">
                                <span><field name="qty_done" readonly="1" nolabel="1"/></span>
                                <span>/</span>
                                <span><field name="product_uom_qty" readonly="1" nolabel="1"/></span>
                                <span><field name="product_uom_id" attrs="{'readonly': [('id', '!=', False)]}" nolabel="1"/></span>
                            </div>
                            <field name="lot_id" string="Lot/Serial number" groups="stock.group_production_lot" readonly="1" attrs="{'invisible': [('lots_visible', '=', False)]}"/>
                            <field name="lots_visible" invisible="1"/>
                        </group>
                    </group>
                </form>
            </field>
        </record>

        <record id="stock_move_line_view_search" model="ir.ui.view">
            <field name="name">stock.move.line.search</field>
            <field name="model">stock.move.line</field>
            <field name="inherit_id" ref="stock.stock_move_line_view_search" />
            <field name="arch" type="xml">
                <filter name="manufacturing" position="attributes">
                    <attribute name="invisible">0</attribute>
                    <attribute name="domain">[('move_id.production_id', '!=', False)]</attribute>
                </filter>
            </field>
        </record>

    <menuitem id="menu_mrp_traceability"
          name="Lots/Serial Numbers"
          parent="menu_mrp_bom"
          action="stock.action_production_lot_form"
          groups="stock.group_production_lot"
          sequence="15"/>

    <menuitem id="menu_mrp_scrap"
            name="Scrap Orders"
            parent="menu_mrp_manufacturing"
            action="stock.action_stock_scrap"
            sequence="25"/>

</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_production_type_kanban" model="ir.ui.view">
        <field name="name">stock.picking.type.kanban</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.stock_picking_type_kanban"/>
        <field name="arch" type="xml">
            <field name="code" position="after">
                <field name="count_mo_todo"/>
                <field name="count_mo_waiting"/>
                <field name="count_mo_late"/>
            </field>

            <xpath expr='//div[@name="stock_picking"]' position="after">
                <div t-if="record.code.raw_value == 'mrp_operation'" t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                    <div>
                        <div t-attf-class="o_kanban_card_header">
                            <div class="o_kanban_card_header_title">
                                <a type="object" name="get_mrp_stock_picking_action_picking_type" class="o_primary" t-if="!selection_mode">
                                    <field name="name"/>
                                </a>
                                <span class="o_primary" t-if="selection_mode"><field name="name"/></span>
                                <div class="o_secondary"><field class="o_secondary"  name="warehouse_id" readonly="1"/></div>
                            </div>
                            <div class="o_kanban_manage_button_section" t-if="!selection_mode">
                                <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
                            </div>
                        </div>
                        <div class="container o_kanban_card_content" t-if="!selection_mode">
                            <div class="row">
                                <div class="col-6 o_kanban_primary_left">
                                    <button class="btn btn-primary" name="%(mrp_production_action_picking_deshboard)d" type="action" context="{'search_default_todo': 1, 'default_picking_type_id': active_id}">
                                        <span t-if="record.code.raw_value =='mrp_operation'"><t t-esc="record.count_mo_todo.value"/> To Process</span>
                                    </button>
                                </div>
                                <div class="col-6 o_kanban_primary_right">
                                    <div t-if="record.count_mo_waiting.raw_value > 0" class="row">
                                        <div class="col-9">
                                            <a name="%(mrp_production_action_picking_deshboard)d" type="action" context="{'search_default_waiting': 1}">
                                                Waiting
                                            </a>
                                        </div>
                                        <div class="col-3">
                                            <field name="count_mo_waiting"/>
                                        </div>
                                    </div>
                                    <div t-if="record.count_mo_late.raw_value > 0" class="row">
                                        <div class="col-9">
                                            <a class="oe_kanban_stock_picking_type_list" name="%(mrp_production_action_picking_deshboard)d" type="action" context="{'search_default_late': 1, 'default_picking_type_id': active_id}">
                                                Late
                                            </a>
                                        </div>
                                        <div class="col-3">
                                            <field name="count_mo_late"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div><div class="container o_kanban_card_manage_pane dropdown-menu" role="menu">
                            <div class="row">
                                <div class="col-6 o_kanban_card_manage_section o_kanban_manage_view" name="picking_left_manage_pane">
                                    <div role="menuitem" class="o_kanban_card_manage_title">
                                        <span>View</span>
                                    </div>
                                    <div role="menuitem">
                                        <a name="%(mrp_production_action_picking_deshboard)d" type="action">All</a>
                                    </div>
                                    <div role="menuitem">
                                        <a name="%(mrp_production_action_picking_deshboard)d" type="action" context="{'search_default_inprogress': 1}">In Progress</a>
                                    </div>
                                    <div role="menuitem">
                                        <a name="%(mrp_production_action_picking_deshboard)d" type="action" context="{'search_default_planned': 1}">Planned</a>
                                    </div>
                                </div>
                                <div class="col-6 o_kanban_card_manage_section o_kanban_manage_new">
                                    <div role="menuitem" class="o_kanban_card_manage_title">
                                        <span>New</span>
                                    </div>
                                    <div role="menuitem">
                                        <a name="%(action_mrp_production_form)d" type="action">Production Order</a>
                                    </div>
                                </div>
                            </div>

                            <div t-if="widget.editable" class="o_kanban_card_manage_settings row">
                                <div role="menuitem" aria-haspopup="true" class="col-8">
                                    <ul role="menu" class="oe_kanban_colorpicker" data-field="color"/>
                                </div>
                                <div role="menuitem" class="col-4 text-right">
                                    <a type="edit">Settings</a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
    <record id="view_picking_type_form_inherit_mrp" model="ir.ui.view">
        <field name="name">Operation Types</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.view_picking_type_form"/>
        <field name="arch" type="xml">
            <field name="show_operations" position="attributes">
                <attribute name="attrs">{"invisible": [("code", "=", "mrp_operation")]}</attribute>
            </field>
            <xpath expr="//group[@name='stock_picking_type_lot']" position="after">
                <group attrs='{"invisible": [("code", "!=", "mrp_operation")]}' string="Traceability" groups="stock.group_production_lot">
                    <field name="use_create_components_lots"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_scrap_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_scrap_view_form2_mrp_inherit_mrp" model="ir.ui.view">
        <field name="name">stock.scrap.view.form2.inherit.mrp</field>
        <field name="model">stock.scrap</field>
        <field name="inherit_id" ref="stock.stock_scrap_form_view2"/>
        <field name="arch" type="xml">
            <field name="owner_id" position="after">
                <field name="workorder_id" invisible="1"/>
                <field name="production_id" invisible="1"/>
            </field>
        </field>
    </record>
    <record id="stock_scrap_view_form_mrp_inherit_mrp" model="ir.ui.view">
        <field name="name">stock.scrap.view.form.inherit.mrp</field>
        <field name="model">stock.scrap</field>
        <field name="inherit_id" ref="stock.stock_scrap_form_view"/>
        <field name="arch" type="xml">
            <field name="owner_id" position="after">
                <field name="workorder_id" domain="[('production_id', '=', product_id)]" attrs="{'invisible': [('workorder_id', '=', False)]}"/>
                <field name="production_id" domain="[('company_id', '=', company_id)]" attrs="{'invisible': [('production_id', '=', False)]}"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\stock_warehouse_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Warehouse -->
        <record id="view_warehouse_inherit_mrp" model="ir.ui.view">
            <field name="name">Stock Warehouse Inherit MRP</field>
            <field name="model">stock.warehouse</field>
            <field name="inherit_id" ref="stock.view_warehouse"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='resupply_wh_ids']" position="before">
                    <field name="manufacture_to_resupply" />
                    <field name="manufacture_steps" attrs="{'invisible': [('manufacture_to_resupply', '=', False)]}" widget="radio"/>
                </xpath>
                <xpath expr="//field[@name='out_type_id']" position="after">
                    <field name="manu_type_id" readonly="True"/>
                </xpath>
                <xpath expr="//group[@name='group_resupply']" position="attributes">
                    <attribute name="attrs">{}</attribute>
                </xpath>
                <xpath expr="//field[@name='wh_output_stock_loc_id']" position="after">
                    <field name="sam_loc_id"/>
                    <field name="pbm_loc_id"/>
                </xpath>
                <xpath expr="//field[@name='out_type_id']" position="after">
                    <field name="sam_type_id"/>
                    <field name="pbm_type_id"/>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: wizard\change_production_qty.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_is_zero, float_round, float_compare


class ChangeProductionQty(models.TransientModel):
    _name = 'change.production.qty'
    _description = 'Change Production Qty'

    mo_id = fields.Many2one('mrp.production', 'Manufacturing Order',
        required=True, ondelete='cascade')
    product_qty = fields.Float(
        'Quantity To Produce',
        digits='Product Unit of Measure', required=True)

    @api.model
    def default_get(self, fields):
        res = super(ChangeProductionQty, self).default_get(fields)
        if 'mo_id' in fields and not res.get('mo_id') and self._context.get('active_model') == 'mrp.production' and self._context.get('active_id'):
            res['mo_id'] = self._context['active_id']
        if 'product_qty' in fields and not res.get('product_qty') and res.get('mo_id'):
            res['product_qty'] = self.env['mrp.production'].browse(res['mo_id']).product_qty
        return res

    @api.model
    def _update_finished_moves(self, production, new_qty, old_qty):
        """ Update finished product and its byproducts. This method only update
        the finished moves not done or cancel and just increase or decrease
        their quantity according the unit_ratio. It does not use the BoM, BoM
        modification during production would not be taken into consideration.
        """
        modification = {}
        for move in production.move_finished_ids:
            if move.state in ('done', 'cancel'):
                continue
            done_qty = sum(production.move_finished_ids.filtered(
                lambda r:
                    r.product_id == move.product_id and
                    r.state == 'done'
                ).mapped('product_uom_qty')
            )
            qty = (new_qty - old_qty) * move.unit_factor + done_qty
            modification[move] = (move.product_uom_qty + qty, move.product_uom_qty)
            if (move.product_uom_qty + qty) > 0:
                move.write({'product_uom_qty': move.product_uom_qty + qty})
            else:
                move._action_cancel()

        return modification

    def change_prod_qty(self):
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        for wizard in self:
            production = wizard.mo_id
            produced = sum(production.move_finished_ids.filtered(lambda m: m.product_id == production.product_id).mapped('quantity_done'))
            if float_compare(wizard.product_qty, produced, precision_digits=precision) < 0:
                format_qty = '%.{precision}f'.format(precision=precision)
                raise UserError(_("You have already processed %s. Please input a quantity higher than %s ") % (format_qty % produced, format_qty % produced))
            old_production_qty = production.product_qty
            production.write({'product_qty': wizard.product_qty})
            done_moves = production.move_finished_ids.filtered(lambda x: x.state == 'done' and x.product_id == production.product_id)
            qty_produced = production.product_id.uom_id._compute_quantity(sum(done_moves.mapped('product_qty')), production.product_uom_id)
            factor = production.product_uom_id._compute_quantity(production.product_qty - qty_produced, production.bom_id.product_uom_id) / production.bom_id.product_qty
            boms, lines = production.bom_id.explode(production.product_id, factor, picking_type=production.bom_id.picking_type_id)
            documents = {}
            for line, line_data in lines:
                if line.child_bom_id and line.child_bom_id.type == 'phantom' or\
                        line.product_id.type not in ['product', 'consu']:
                    continue
                move = production.move_raw_ids.filtered(lambda x: x.bom_line_id.id == line.id and x.state not in ('done', 'cancel'))
                if move:
                    move = move[0]
                    old_qty = move.product_uom_qty
                else:
                    old_qty = 0
                iterate_key = production._get_document_iterate_key(move)
                if iterate_key:
                    document = self.env['stock.picking']._log_activity_get_documents({move: (line_data['qty'], old_qty)}, iterate_key, 'UP')
                    for key, value in document.items():
                        if documents.get(key):
                            documents[key] += [value]
                        else:
                            documents[key] = [value]

                production._update_raw_move(line, line_data)

            production._log_manufacture_exception(documents)
            operation_bom_qty = {}
            for bom, bom_data in boms:
                for operation in bom.routing_id.operation_ids:
                    operation_bom_qty[operation.id] = bom.product_qty * bom_data['qty']
            finished_moves_modification = self._update_finished_moves(production, production.product_qty - qty_produced, old_production_qty)
            production._log_downside_manufactured_quantity(finished_moves_modification)
            moves = production.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
            moves._action_assign()
            for wo in production.workorder_ids:
                operation = wo.operation_id
                if operation_bom_qty.get(operation.id):
                    cycle_number = float_round(operation_bom_qty[operation.id] / operation.workcenter_id.capacity, precision_digits=0, rounding_method='UP')
                    wo.duration_expected = (operation.workcenter_id.time_start +
                                 operation.workcenter_id.time_stop +
                                 cycle_number * operation.time_cycle * 100.0 / operation.workcenter_id.time_efficiency)
                production_qty = wo._get_real_uom_qty(wo.qty_production)
                quantity = production_qty - wo.qty_produced
                if production.product_id.tracking == 'serial':
                    quantity = 1.0 if not float_is_zero(quantity, precision_digits=precision) else 0.0
                else:
                    quantity = quantity if float_compare(quantity, 0, precision_digits=precision) > 0 else 0
                if float_is_zero(quantity, precision_digits=precision):
                    wo.finished_lot_id = False
                    wo._workorder_line_ids().unlink()
                wo.qty_producing = quantity
                if float_compare(wo.qty_produced, production_qty, precision_digits=precision) < 0 and wo.state == 'done':
                    wo.state = 'progress'
                if float_compare(wo.qty_produced, production_qty, precision_digits=precision) == 0 and wo.state == 'progress':
                    wo.state = 'done'
                    if wo.next_work_order_id.state == 'pending':
                        wo.next_work_order_id.state = 'ready'
                # assign moves; last operation receive all unassigned moves
                # TODO: following could be put in a function as it is similar as code in _workorders_create
                # TODO: only needed when creating new moves
                moves_raw = production.move_raw_ids.filtered(lambda move: move.operation_id == operation and move.state not in ('done', 'cancel'))
                if wo == production.workorder_ids[-1]:
                    moves_raw |= production.move_raw_ids.filtered(lambda move: not move.operation_id)
                moves_finished = production.move_finished_ids.filtered(lambda move: move.operation_id == operation) #TODO: code does nothing, unless maybe by_products?
                moves_raw.mapped('move_line_ids').write({'workorder_id': wo.id})
                (moves_finished + moves_raw).write({'workorder_id': wo.id})
                if wo.state not in ('done', 'cancel'):
                    line_values = wo._update_workorder_lines()
                    wo._workorder_line_ids().create(line_values['to_create'])
                    if line_values['to_delete']:
                        line_values['to_delete'].unlink()
                    for line, vals in line_values['to_update'].items():
                        line.write(vals)
        return {}

```

## File: wizard\change_production_qty_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        
        <!--  Change Product Quantity -->
        <record id="view_change_production_qty_wizard" model="ir.ui.view">
            <field name="name">Change Quantity To Produce</field>
            <field name="model">change.production.qty</field>
            <field name="arch" type="xml">
                <form string="Change Product Qty">
                    <group>
                        <field name="product_qty"/>
                        <field name="mo_id" invisible="1"/>
                    </group>
                    <footer>
                        <button name="change_prod_qty" string="Approve"
                            colspan="1" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" />
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_change_production_qty" model="ir.actions.act_window">
            <field name="name">Change Quantity To Produce</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">change.production.qty</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>
       
    </data>
</odoo>    

```

## File: wizard\mrp_product_produce.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare


class MrpProductProduce(models.TransientModel):
    _name = "mrp.product.produce"
    _description = "Record Production"
    _inherit = ["mrp.abstract.workorder"]

    @api.model
    def default_get(self, fields):
        res = super(MrpProductProduce, self).default_get(fields)
        production = self.env['mrp.production']
        production_id = self.env.context.get('default_production_id') or self.env.context.get('active_id')
        if production_id:
            production = self.env['mrp.production'].browse(production_id)
        if production.exists():
            serial_finished = (production.product_id.tracking == 'serial')
            todo_uom = production.product_uom_id.id
            todo_quantity = self._get_todo(production)
            if serial_finished:
                todo_quantity = 1.0
                if production.product_uom_id.uom_type != 'reference':
                    todo_uom = self.env['uom.uom'].search([('category_id', '=', production.product_uom_id.category_id.id), ('uom_type', '=', 'reference')]).id
            if 'production_id' in fields:
                res['production_id'] = production.id
            if 'product_id' in fields:
                res['product_id'] = production.product_id.id
            if 'product_uom_id' in fields:
                res['product_uom_id'] = todo_uom
            if 'serial' in fields:
                res['serial'] = bool(serial_finished)
            if 'qty_producing' in fields:
                res['qty_producing'] = todo_quantity
            if 'consumption' in fields:
                res['consumption'] = production.bom_id.consumption
        return res

    serial = fields.Boolean('Requires Serial')
    product_tracking = fields.Selection(related="product_id.tracking")
    is_pending_production = fields.Boolean(compute='_compute_pending_production')

    move_raw_ids = fields.One2many(related='production_id.move_raw_ids', string="PO Components")
    move_finished_ids = fields.One2many(related='production_id.move_finished_ids')

    raw_workorder_line_ids = fields.One2many('mrp.product.produce.line',
        'raw_product_produce_id', string='Components')
    finished_workorder_line_ids = fields.One2many('mrp.product.produce.line',
        'finished_product_produce_id', string='By-products')
    production_id = fields.Many2one('mrp.production', 'Manufacturing Order',
        required=True, ondelete='cascade')

    @api.depends('qty_producing')
    def _compute_pending_production(self):
        """ Compute if it exits remaining quantity once the quantity on the
        current wizard will be processed. The purpose is to display or not
        button 'continue'.
        """
        for product_produce in self:
            remaining_qty = product_produce._get_todo(product_produce.production_id)
            product_produce.is_pending_production = remaining_qty - product_produce.qty_producing > 0.0

    def continue_production(self):
        """ Save current wizard and directly opens a new. """
        self.ensure_one()
        self._record_production()
        action = self.production_id.open_produce_product()
        action['context'] = {'default_production_id': self.production_id.id}
        return action

    def action_generate_serial(self):
        self.ensure_one()
        product_produce_wiz = self.env.ref('mrp.view_mrp_product_produce_wizard', False)
        self.finished_lot_id = self.env['stock.production.lot'].create({
            'product_id': self.product_id.id,
            'company_id': self.production_id.company_id.id
        })
        return {
            'name': _('Produce'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mrp.product.produce',
            'res_id': self.id,
            'view_id': product_produce_wiz.id,
            'target': 'new',
        }

    def do_produce(self):
        """ Save the current wizard and go back to the MO. """
        self.ensure_one()
        self._record_production()
        self._check_company()
        return {'type': 'ir.actions.act_window_close'}

    def _get_todo(self, production):
        """ This method will return remaining todo quantity of production. """
        main_product_moves = production.move_finished_ids.filtered(lambda x: x.product_id.id == production.product_id.id)
        todo_quantity = production.product_qty - sum(main_product_moves.mapped('quantity_done'))
        todo_quantity = todo_quantity if (todo_quantity > 0) else 0
        return todo_quantity

    def _record_production(self):
        # Check all the product_produce line have a move id (the user can add product
        # to consume directly in the wizard)
        for line in self._workorder_line_ids():
            if not line.move_id:
                # Find move_id that would match
                if line.raw_product_produce_id:
                    moves = line.raw_product_produce_id.move_raw_ids
                else:
                    moves = line.finished_product_produce_id.move_finished_ids
                move_id = moves.filtered(lambda m: m.product_id == line.product_id and m.state not in ('done', 'cancel'))
                if not move_id:
                    # create a move to assign it to the line
                    production = line._get_production()
                    if line.raw_product_produce_id:
                        values = {
                            'name': production.name,
                            'reference': production.name,
                            'product_id': line.product_id.id,
                            'product_uom': line.product_uom_id.id,
                            'location_id': production.location_src_id.id,
                            'location_dest_id': self.product_id.property_stock_production.id,
                            'raw_material_production_id': production.id,
                            'group_id': production.procurement_group_id.id,
                            'origin': production.name,
                            'state': 'confirmed',
                            'company_id': production.company_id.id,
                        }
                    else:
                        values = production._get_finished_move_value(line.product_id.id, 0, line.product_uom_id.id)
                    move_id = self.env['stock.move'].create(values)
                line.move_id = move_id.id

        # because of an ORM limitation (fields on transient models are not
        # recomputed by updates in non-transient models), the related fields on
        # this model are not recomputed by the creations above
        self.invalidate_cache(['move_raw_ids', 'move_finished_ids'])

        # Save product produce lines data into stock moves/move lines
        for wizard in self:
            quantity = wizard.qty_producing
            if float_compare(quantity, 0, precision_rounding=self.product_uom_id.rounding) <= 0:
                raise UserError(_("The production order for '%s' has no quantity specified.") % self.product_id.display_name)
            quantity = wizard.product_uom_id._compute_quantity(quantity, wizard.production_id.product_uom_id)
            if float_compare(quantity, wizard.production_id.product_qty, precision_rounding=self.product_uom_id.rounding) > 0:
                wizard.production_id.write({'product_qty': quantity})
        self._update_finished_move()
        self._update_moves()
        self.production_id.filtered(lambda mo: mo.state == 'confirmed').write({
            'date_start': datetime.now(),
        })


class MrpProductProduceLine(models.TransientModel):
    _name = 'mrp.product.produce.line'
    _inherit = ["mrp.abstract.workorder.line"]
    _description = "Record production line"

    raw_product_produce_id = fields.Many2one('mrp.product.produce', 'Component in Produce wizard')
    finished_product_produce_id = fields.Many2one('mrp.product.produce', 'Finished Product in Produce wizard')

    @api.model
    def _get_raw_workorder_inverse_name(self):
        return 'raw_product_produce_id'

    @api.model
    def _get_finished_workoder_inverse_name(self):
        return 'finished_product_produce_id'

    def _get_final_lots(self):
        product_produce_id = self.raw_product_produce_id or self.finished_product_produce_id
        return product_produce_id.finished_lot_id | product_produce_id.finished_workorder_line_ids.mapped('lot_id')

    def _get_production(self):
        product_produce_id = self.raw_product_produce_id or self.finished_product_produce_id
        return product_produce_id.production_id

    @api.onchange('lot_id')
    def _onchange_lot_id(self):
        """ When the user is encoding a produce line for a tracked product, we apply some logic to
        help him. This onchange will automatically switch `qty_done` to 1.0.
        """
        if self.product_id.tracking == 'serial':
            if self.lot_id:
                self.qty_done = 1
            else:
                self.qty_done = 0

```

## File: wizard\mrp_product_produce_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="view_mrp_product_produce_wizard" model="ir.ui.view">
            <field name="name">MRP Product Produce</field>
            <field name="model">mrp.product.produce</field>
            <field name="arch" type="xml">
                <form string="Produce">
                    <group>
                        <group>
                            <field name="serial" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                            <field name="production_id" invisible="1"/>
                            <field name="product_id" readonly="1"/>
                            <field name="is_pending_production" invisible="1"/>
                            <label for="qty_producing" string="Quantity"/>
                            <div class="o_row">
                                <field name="qty_producing" attrs="{'readonly': [('serial', '=', True)]}"/>
                                <field name="product_uom_id" readonly="1" groups="uom.group_uom"/>
                            </div>
                            <field name="product_tracking" invisible="1"/>
                            <label for="finished_lot_id" attrs="{'invisible': [('product_tracking', '=', 'none')]}"/>
                            <div class="o_row">
                                <field name="finished_lot_id" attrs="{'invisible': [('product_tracking', '=', 'none')], 'required': [('product_tracking', '!=', 'none'), ('finished_lot_id', '!=', False)]}" context="{'default_product_id': product_id, 'default_company_id': company_id}"/>
                                <button name="action_generate_serial" type="object" class="btn btn-primary fa fa-plus-square-o" aria-label="Creates a new serial/lot number" title="Creates a new serial/lot number" role="img" attrs="{'invisible': ['|', ('product_tracking', '=', 'none'), ('finished_lot_id', '!=', False)]}"/>
                            </div>
                        </group>
                    </group>
                    <h4 attrs="{'invisible': [('raw_workorder_line_ids', '=', [])]}">Components</h4>
                    <group>
                        <field name="raw_workorder_line_ids" attrs="{'invisible': [('raw_workorder_line_ids', '=', [])]}" nolabel="1" context="{'w_production': True, 'active_id': production_id, 'default_finished_lot_id': finished_lot_id}">
                            <tree editable="bottom" delete="0" decoration-danger="(qty_to_consume &lt; qty_done)">
                                <field name="company_id" invisible="1"/>
                                <field name="product_id" attrs="{'readonly': [('move_id', '!=', False)]}" required="1" domain="[('id', '!=', parent.product_id), ('type', 'in', ['product', 'consu']), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" force_save="1"/>
                                <field name="product_tracking" invisible="1"/>
                                <field name="lot_id" attrs="{'readonly': [('product_tracking', '=', 'none')]}" context="{'default_product_id': product_id, 'active_mo_id': parent.production_id, 'default_company_id': company_id}" groups="stock.group_production_lot"/>
                                <field name="qty_to_consume" readonly="1" force_save="1"/>
                                <field name="qty_reserved" readonly="1" force_save="1" optional="show"/>
                                <field name="qty_done"/>
                                <field name="product_uom_id" readonly="1" force_save="1" groups="uom.group_uom"/>
                                <field name="move_id" invisible="1"/>
                            </tree>
                        </field>
                    </group>
                    <h4 attrs="{'invisible': [('finished_workorder_line_ids', '=', [])]}">By-products</h4>
                    <group>
                        <field name="finished_workorder_line_ids" attrs="{'invisible': [('finished_workorder_line_ids', '=', [])]}" nolabel="1" context="{'w_production': True, 'active_id': production_id, 'default_finished_lot_id': finished_lot_id, 'default_company_id': company_id}">
                            <tree editable="bottom" delete="0" decoration-danger="(qty_to_consume &lt; qty_done)">
                                <field name="company_id" invisible="1"/>
                                <field name="product_id" attrs="{'readonly': [('move_id', '!=', False)]}" required="1" domain="[('id', '!=', parent.product_id), ('type', 'in', ['product', 'consu']), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" force_save="1"/>
                                <field name="product_tracking" invisible="1"/>
                                <field name="lot_id" attrs="{'readonly': [('product_tracking', '=', 'none')]}" context="{'default_product_id': product_id, 'default_company_id': company_id}" groups="stock.group_production_lot"/>
                                <field name="qty_to_consume" string="To Produce" readonly="1" force_save="1"/>
                                <field name="qty_done" string="Produced"/>
                                <field name="product_uom_id" readonly="1" force_save="1" groups="uom.group_uom"/>
                                <field name="move_id" invisible="1"/>
                            </tree>
                        </field>
                    </group>
                    <footer>
                        <button name="continue_production" attrs="{'invisible':[('is_pending_production', '=', False)]}" type="object" string="Continue" class="btn-primary"/>
                        <button name="do_produce" type="object" string="Save" class="btn-default btn-secondary" attrs="{'invisible':[('is_pending_production', '=', False)]}"/>
                        <button name="do_produce" type="object" string="Save" class="btn-primary" attrs="{'invisible':[('is_pending_production', '=', True)]}"/>
                        <button string="Discard" class="btn-default btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="mrp_product_produce_line_form" model="ir.ui.view">
            <field name="name">MRP Product Produce Line</field>
            <field name="model">mrp.product.produce.line</field>
            <field name="priority">100</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <group>
                            <field name="company_id" invisible="1"/>
                            <field name="product_id"/>
                            <field name="product_tracking" invisible="1"/>
                            <field name="lot_id" attrs="{'readonly': [('product_tracking', '=', 'none')]}" context="{'default_product_id': product_id}" groups="stock.group_production_lot"/>
                            <field name="qty_to_consume" readonly="1" force_save="1"/>
                            <field name="qty_reserved" readonly="1" force_save="1" optional="show"/>
                            <field name="qty_done"/>
                            <field name="product_uom_id" readonly="1" force_save="1" groups="uom.group_uom"/>
                            <field name="move_id" invisible="1"/>
                        </group>
                    </group>
                </form>
            </field>
        </record>

        <record id="mrp_product_produce_line_kanban" model="ir.ui.view">
            <field name="name">MRP Product Produce Line</field>
            <field name="model">mrp.product.produce.line</field>
            <field name="priority">100</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="product_id"/>
                    <field name="lot_id"/>
                    <field name="qty_to_consume"/>
                    <field name="qty_reserved"/>
                    <field name="qty_done"/>
                    <field name="product_uom_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title"><span><field name="product_id"/></span></strong>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body">
                                    <span>Lot <field name="lot_id"/></span><br/>
                                    <span>Quantity To Consume <field name="qty_to_consume"/></span><br/>
                                    <span>Quantity Reserved <field name="qty_reserved"/></span><br/>
                                    <span>Quantity Done <field name="qty_done"/></span><br/>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="act_mrp_product_produce" model="ir.actions.act_window">
            <field name="name">Produce</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.product.produce</field>
            <field name="view_mode">form</field>
            <field name="context">{}</field>
            <field name="target">new</field>
        </record>
</odoo>

```

## File: wizard\mrp_workcenter_block_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Workcenter Block Dialog -->
    <record id="mrp_workcenter_block_wizard_form" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.form</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="arch" type="xml">
            <form string="Block Workcenter">
                <group>
                    <field name="loss_id" class="oe_inline" domain="[('manual','=',True)]"/>
                    <field name="description" placeholder="Add a description..."/>
                    <field name="workcenter_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                </group>
                <footer>
                    <button name="button_block" string="Block" type="object" class="btn-danger text-uppercase"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>
    
    <record id="act_mrp_block_workcenter" model="ir.actions.act_window">
        <field name="name">Block Workcenter</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workcenter.productivity</field>
        <field name="view_mode">form</field>
        <field name="context">{'default_workcenter_id': active_id}</field>
        <field name="view_id" eval="mrp_workcenter_block_wizard_form"/>
        <field name="target">new</field>
    </record>
    
    <record id="act_mrp_block_workcenter_wo" model="ir.actions.act_window">
        <field name="name">Block Workcenter</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workcenter.productivity</field>
        <field name="view_mode">form</field>
        <field name="view_id" eval="mrp_workcenter_block_wizard_form"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\stock_warn_insufficient_qty.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockWarnInsufficientQtyUnbuild(models.TransientModel):
    _name = 'stock.warn.insufficient.qty.unbuild'
    _inherit = 'stock.warn.insufficient.qty'
    _description = 'Warn Insufficient Unbuild Quantity'

    unbuild_id = fields.Many2one('mrp.unbuild', 'Unbuild')

    def _get_reference_document_company_id(self):
        return self.unbuild_id.company_id

    def action_done(self):
        self.ensure_one()
        return self.unbuild_id.action_unbuild()

```

## File: wizard\stock_warn_insufficient_qty_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_warn_insufficient_qty_unbuild_form_view" model="ir.ui.view">
        <field name="name">stock.warn.insufficient.qty.unbuild</field>
        <field name="model">stock.warn.insufficient.qty.unbuild</field>
        <field name="inherit_id" ref="stock.stock_warn_insufficient_qty_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='location_id']" position="after">
                <field name="unbuild_id" invisible="True"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_product_produce
from . import change_production_qty
from . import stock_warn_insufficient_qty

```

