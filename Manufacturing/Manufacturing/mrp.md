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
from . import controller
from . import populate

from odoo import api, SUPERUSER_ID


def _pre_init_mrp(cr):
    """ Allow installing MRP in databases with large stock.move table (>1M records)
        - Creating the computed+stored field stock_move.is_done and
          stock_move.unit_factor is terribly slow with the ORM and leads to "Out of
          Memory" crashes
    """
    cr.execute("""ALTER TABLE "stock_move" ADD COLUMN "is_done" bool;""")
    cr.execute("""UPDATE stock_move
                     SET is_done=COALESCE(state in ('done', 'cancel'), FALSE);""")
    cr.execute("""ALTER TABLE "stock_move" ADD COLUMN "unit_factor" double precision;""")
    cr.execute("""UPDATE stock_move
                     SET unit_factor=1;""")

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
    'website': 'https://www.odoo.com/app/manufacturing',
    'category': 'Manufacturing/Manufacturing',
    'sequence': 55,
    'summary': 'Manufacturing Orders & BOMs',
    'depends': ['product', 'stock', 'resource'],
    'description': "",
    'data': [
        'security/mrp_security.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'data/mail_templates.xml',
        'data/mrp_data.xml',
        'wizard/change_production_qty_views.xml',
        'wizard/mrp_workcenter_block_view.xml',
        'wizard/stock_warn_insufficient_qty_views.xml',
        'wizard/mrp_production_backorder.xml',
        'wizard/mrp_consumption_warning_views.xml',
        'wizard/mrp_immediate_production_views.xml',
        'wizard/stock_assign_serial_numbers.xml',
        'views/mrp_views_menus.xml',
        'views/stock_move_views.xml',
        'views/mrp_workorder_views.xml',
        'views/mrp_workcenter_views.xml',
        'views/mrp_bom_views.xml',
        'views/mrp_production_views.xml',
        'views/mrp_routing_views.xml',
        'views/product_views.xml',
        'views/stock_orderpoint_views.xml',
        'views/stock_warehouse_views.xml',
        'views/stock_picking_views.xml',
        'views/mrp_unbuild_views.xml',
        'views/ir_attachment_view.xml',
        'views/res_config_settings_views.xml',
        'views/stock_scrap_views.xml',
        'report/report_deliveryslip.xml',
        'report/mrp_report_views_main.xml',
        'report/mrp_report_bom_structure.xml',
        'report/mrp_production_templates.xml',
        'report/report_stock_forecasted.xml',
        'report/report_stock_reception.xml',
        'report/report_stock_rule.xml',
        'report/mrp_zebra_production_templates.xml',
    ],
    'demo': [
        'data/mrp_demo.xml',
    ],
    'test': [],
    'application': True,
    'pre_init_hook': '_pre_init_mrp',
    'post_init_hook': '_create_warehouse_data',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_backend': [
            'mrp/static/src/scss/mrp_workorder_kanban.scss',
            'mrp/static/src/js/mrp.js',
            'mrp/static/src/js/mrp_bom_report.js',
            'mrp/static/src/js/mrp_workorder_popover.js',
            'mrp/static/src/js/mrp_documents_controller_mixin.js',
            'mrp/static/src/js/mrp_documents_document_viewer.js',
            'mrp/static/src/js/mrp_documents_kanban_controller.js',
            'mrp/static/src/js/mrp_documents_kanban_record.js',
            'mrp/static/src/js/mrp_documents_kanban_renderer.js',
            'mrp/static/src/js/mrp_document_kanban_view.js',
            'mrp/static/src/js/mrp_should_consume.js',
            'mrp/static/src/js/mrp_field_one2many_with_copy.js',
        ],
        'web.assets_common': [
            'mrp/static/src/scss/mrp_bom_report.scss',
            'mrp/static/src/scss/mrp_fields.scss',
            'mrp/static/src/scss/mrp_gantt.scss',
            'mrp/static/src/scss/mrp_document_kanban_view.scss',
        ],
        'web.qunit_suite_tests': [
            'mrp/static/tests/**/*',
        ],
        'web.assets_qweb': [
            'mrp/static/src/xml/*.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controller\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import logging

from odoo import http
from odoo.http import request
from odoo.tools.translate import _

logger = logging.getLogger(__name__)


class MrpDocumentRoute(http.Controller):

    @http.route('/mrp/upload_attachment', type='http', methods=['POST'], auth="user")
    def upload_document(self, ufile, **kwargs):
        files = request.httprequest.files.getlist('ufile')
        result = {'success': _("All files uploaded")}
        for ufile in files:
            try:
                mimetype = ufile.content_type
                request.env['mrp.document'].create({
                    'name': ufile.filename,
                    'res_model': kwargs.get('res_model'),
                    'res_id': int(kwargs.get('res_id')),
                    'mimetype': mimetype,
                    'datas': base64.encodebytes(ufile.read()),
                })
            except Exception as e:
                logger.exception("Fail to upload document %s" % ufile.filename)
                result = {'error': str(e)}

        return json.dumps(result)

```

## File: controller\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="digest_tip_mrp_0" model="digest.tip">
            <field name="name">Tip: Use tablets in the shop to control manufacturing</field>
            <field name="sequence">600</field>
            <field name="group_id" ref="mrp.group_mrp_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Use tablets in the shop to control manufacturing</p>
    <p class="tip_content">With the Odoo work center control panel, your worker can start work orders in the shop and follow instructions of the worksheet. Quality tests are perfectly integrated into the process. Workers can trigger feedback loops, maintenance alerts, scrap products, etc.</p>
    <img src="/mrp/static/src/img/mrp-tablet.png" style="margin-top: 20px; max-width: 580px" width="100%" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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

## File: data\mrp_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <function model="res.company" name="create_missing_unbuild_sequences" />
        <!--
             Stock rules and routes
        -->

        <record id="route_warehouse0_manufacture" model='stock.location.route'>
            <field name="name">Manufacture</field>
            <field name="company_id"></field>
            <field name="sequence">10</field>
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

        <!-- Resource: mrp.bom -->

        <record id="product.product_product_3_product_template" model="product.template">
            <field name="route_ids" eval="[(6, 0, [ref('stock.route_warehouse0_mto'), ref('mrp.route_warehouse0_manufacture')])]"/>
        </record>
        <record id="mrp_bom_manufacture" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_product_3_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
        </record>
        <record id="mrp_routing_workcenter_0" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_manufacture"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Manual Assembly</field>
            <field name="time_cycle">60</field>
            <field name="sequence">5</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/assebly-worksheet.pdf"/>
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
            <value model="stock.move" eval="
                obj().env.ref('mrp.mrp_production_1')._get_moves_raw_values() +
                obj().env.ref('mrp.mrp_production_1')._get_moves_finished_values()"/>
        </function>

        <!-- Table -->

        <record id="product_product_computer_desk" model="product.product">
            <field name="name">Table</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">290</field>
            <field name="list_price">520</field>
            <field name="detailed_type">product</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Solid wood table.</field>
            <field name="default_code">FURN_9666</field>
            <field name="tracking">serial</field>
            <field name="image_1920" type="base64" file="mrp/static/img/table.png"/>
        </record>
        <record id="stock_warehouse_orderpoint_table" model="stock.warehouse.orderpoint">
            <field name="product_max_qty">0.0</field>
            <field name="product_min_qty">0.0</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="company_id" ref="base.main_company"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
            <field name="product_id" ref="product_product_computer_desk"/>
        </record>

        <record id="product_product_computer_desk_head" model="product.product">
            <field name="name">Table Top</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">240</field>
            <field name="list_price">380</field>
            <field name="detailed_type">product</field>
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
            <field name="detailed_type">product</field>
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
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">product</field>
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
            <field name="detailed_type">product</field>
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
            <field name="detailed_type">product</field>
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
            <field name="detailed_type">product</field>
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
            <field name="detailed_type">product</field>
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
        </record>
        <record id="mrp_routing_workcenter_5" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_desk"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="time_cycle">120</field>
            <field name="sequence">10</field>
            <field name="name">Assembly</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
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
        </record>

        <record id="mrp_bom_desk_line_4" model="mrp.bom.line">
            <field name="product_id" ref="product_product_computer_desk_screw"/>
            <field name="product_qty">10</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">4</field>
            <field name="bom_id" ref="mrp_bom_desk"/>
        </record>

        <!-- Table MO -->
        <record id="mrp_production_3" model="mrp.production">
            <field name="product_id" ref="product_product_computer_desk"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">1</field>
            <field name="date_planned_start" eval="(DateTime.today() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="bom_id" ref="mrp_bom_desk"/>
        </record>

        <function model="stock.move" name="create">
            <value model="stock.move" eval="
                obj().env.ref('mrp.mrp_production_3')._get_moves_raw_values() +
                obj().env.ref('mrp.mrp_production_3')._get_moves_finished_values()"/>
        </function>

        <record id="mrp_bom_table_top" model="mrp.bom">
            <field name="product_tmpl_id" ref="product_product_computer_desk_head_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
        </record>
        <record id="mrp_routing_workcenter_0" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_table_top"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Manual Assembly</field>
            <field name="time_cycle">60</field>
            <field name="sequence">5</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/assebly-worksheet.pdf"/>
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
        </record>
        <record id="mrp_routing_workcenter_1" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_plastic_laminate"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Long time assembly</field>
            <field name="time_cycle">180</field>
            <field name="sequence">15</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_3" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_plastic_laminate"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Testing</field>
            <field name="time_cycle">60</field>
            <field name="sequence">10</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/assebly-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_4" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_plastic_laminate"/>
            <field name="workcenter_id" ref="mrp_workcenter_1"/>
            <field name="name">Packing</field>
            <field name="time_cycle">30</field>
            <field name="sequence">5</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
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

        <!-- Table Top MO -->
        <record id="mrp_production_4" model="mrp.production">
            <field name="product_id" ref="product_product_computer_desk_head"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">2</field>
            <field name="location_src_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_stock"/>
            <field name="bom_id" ref="mrp_bom_table_top"/>
        </record>

        <!-- Generate Table & Table Top MO's moves -->
        <function model="stock.move" name="create">
            <value model="stock.move" eval="
                obj().env.ref('mrp.mrp_production_4')._get_moves_raw_values() +
                obj().env.ref('mrp.mrp_production_4')._get_moves_finished_values()"/>
        </function>

        <!-- Table Kit -->

        <record id="product_product_table_kit" model="product.product">
            <field name="name">Table Kit</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="standard_price">600.0</field>
            <field name="list_price">147.0</field>
            <field name="detailed_type">consu</field>
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
            <field name="standard_price">20.0</field>
            <field name="list_price">24.0</field>
            <field name="detailed_type">product</field>
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
            <field name="standard_price">10</field>
            <field name="list_price">20</field>
            <field name="detailed_type">product</field>
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
        <record id="lot_product_27_1" model="stock.production.lot">
            <field name="name">0000000000031</field>
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

        <record id="stock_inventory_drawer_lot0" model="stock.quant">
            <field name="product_id" ref="product.product_product_27"/>
            <field name="inventory_quantity">50.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
            <field name="lot_id" ref="lot_product_27_0"/>
        </record>
        <record id="stock_inventory_drawer_lot1" model="stock.quant">
            <field name="product_id" ref="product.product_product_27"/>
            <field name="inventory_quantity">40.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
            <field name="lot_id" ref="lot_product_27_1"/>
        </record>
        <record id="stock_inventory_product_drawer_drawer" model="stock.quant">
            <field name="product_id" ref="product_product_drawer_drawer"/>
            <field name="inventory_quantity">50.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
            <field name="lot_id" ref="lot_product_product_drawer_drawer_0"/>
        </record>
        <record id="stock_inventory_product_drawer_case" model="stock.quant">
            <field name="product_id" ref="product_product_drawer_case"/>
            <field name="inventory_quantity">50.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
            <field name="lot_id" ref="lot_product_product_drawer_case_0"/>
        </record>
        <record id="stock_inventory_product_wood_panel" model="stock.quant">
            <field name="product_id" ref="product_product_wood_panel"/>
            <field name="inventory_quantity">50.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
        </record>
        <record id="stock_inventory_product_ply" model="stock.quant">
            <field name="product_id" ref="product_product_wood_ply"/>
            <field name="inventory_quantity">20.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
        </record>
        <record id="stock_inventory_product_wear" model="stock.quant">
            <field name="product_id" ref="product_product_wood_wear"/>
            <field name="inventory_quantity">30.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
        </record>

        <function model="stock.quant" name="action_apply_inventory">
            <function eval="[[('id', 'in', (ref('stock_inventory_drawer_lot0'),
                                            ref('stock_inventory_drawer_lot1'),
                                            ref('stock_inventory_product_drawer_drawer'),
                                            ref('stock_inventory_product_drawer_case'),
                                            ref('stock_inventory_product_wood_panel'),
                                            ref('stock_inventory_product_ply'),
                                            ref('stock_inventory_product_wear'),
                                            ))]]" model="stock.quant" name="search"/>
        </function>

        <!-- BoM -->

        <record id="mrp_bom_drawer" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_product_27_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="code">PRIM-ASSEM</field>
        </record>
        <record id="mrp_bom_drawer_line_1" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_drawer"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_drawer"/>
        </record>
        <record id="mrp_bom_drawer_line_2" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_case"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="bom_id" ref="mrp_bom_drawer"/>
        </record>

        <record id="mrp_bom_drawer_rout" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_product_27_product_template"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="code">SEC-ASSEM</field>
        </record>
        <record id="mrp_routing_workcenter_1" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_drawer_rout"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Long time assembly</field>
            <field name="time_cycle">180</field>
            <field name="sequence">15</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_3" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_drawer_rout"/>
            <field name="workcenter_id" ref="mrp_workcenter_3"/>
            <field name="name">Testing</field>
            <field name="time_cycle">60</field>
            <field name="sequence">10</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/assebly-worksheet.pdf"/>
        </record>

        <record id="mrp_routing_workcenter_4" model="mrp.routing.workcenter">
            <field name="bom_id" ref="mrp_bom_drawer_rout"/>
            <field name="workcenter_id" ref="mrp_workcenter_1"/>
            <field name="name">Packing</field>
            <field name="time_cycle">30</field>
            <field name="sequence">5</field>
            <field name="worksheet_type">pdf</field>
            <field name="worksheet" type="base64" file="mrp/static/img/cutting-worksheet.pdf"/>
        </record>
        <record id="mrp_bom_drawer_rout_line_1" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_drawer"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">1</field>
            <field name="bom_id" ref="mrp_bom_drawer_rout"/>
        </record>
        <record id="mrp_bom_drawer_rout_line_2" model="mrp.bom.line">
            <field name="product_id" ref="product_product_drawer_case"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="sequence">2</field>
            <field name="bom_id" ref="mrp_bom_drawer_rout"/>
        </record>

        <record id="product.product_product_27" model="product.product">
            <field name="detailed_type">product</field>
        </record>
        <record id="mrp_production_drawer" model="mrp.production">
            <field name="product_id" ref="product.product_product_27"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">5</field>
            <field name="location_src_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_stock"/>
            <field name="bom_id" ref="mrp_bom_drawer"/>
        </record>

        <function model="stock.move" name="create">
            <value model="stock.move" eval="
                obj().env.ref('mrp.mrp_production_drawer')._get_moves_raw_values() +
                obj().env.ref('mrp.mrp_production_drawer')._get_moves_finished_values()"/>
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

        <function model="mrp.production" name="_create_workorder">
            <value eval="[ref('mrp.mrp_production_3')]"/>
        </function>

        <function model="mrp.production" name="action_confirm" eval="[[
            ref('mrp.mrp_production_3'),
            ref('mrp.mrp_production_4'),
            ref('mrp.mrp_production_drawer'),
        ]]"/>

        <function model="mrp.production" name="button_plan">
            <value eval="[ref('mrp.mrp_production_3')]"/>
        </function>

        <function model="mrp.production" name="write">
            <value eval="[ref('mrp.mrp_production_drawer')]"/>
            <value eval="{'qty_producing': 5, 'lot_producing_id': ref('mrp.lot_product_27_0')}"/>
        </function>

        <function model="mrp.production" name="action_assign">
            <value eval="[ref('mrp.mrp_production_drawer')]"/>
        </function>

        <function model="stock.move" name="write">
            <value model="stock.move" eval="obj().env['stock.move'].search([('raw_material_production_id', '=', obj().env.ref('mrp.mrp_production_drawer').id)]).ids"/>
            <value eval="{'quantity_done': 5}"/>
        </function>

        <function model="mrp.production" name="_post_inventory">
            <value eval="[ref('mrp.mrp_production_drawer')]"/>
        </function>

        <function model="mrp.production" name="button_mark_done">
            <value eval="[ref('mrp.mrp_production_drawer')]"/>
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

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.osv.expression import AND, NEGATIVE_TERM_OPERATORS, OR
from odoo.tools import float_round

from collections import defaultdict


class MrpBom(models.Model):
    """ Defines bills of material for a product or a product template """
    _name = 'mrp.bom'
    _description = 'Bill of Material'
    _inherit = ['mail.thread']
    _rec_name = 'product_tmpl_id'
    _order = "sequence, id"
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
        check_company=True, index=True,
        domain="[('type', 'in', ['product', 'consu']), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", required=True)
    product_id = fields.Many2one(
        'product.product', 'Product Variant',
        check_company=True, index=True,
        domain="['&', ('product_tmpl_id', '=', product_tmpl_id), ('type', 'in', ['product', 'consu']),  '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="If a product variant is defined the BOM is available only for this product.")
    bom_line_ids = fields.One2many('mrp.bom.line', 'bom_id', 'BoM Lines', copy=True)
    byproduct_ids = fields.One2many('mrp.bom.byproduct', 'bom_id', 'By-products', copy=True)
    product_qty = fields.Float(
        'Quantity', default=1.0,
        digits='Unit of Measure', required=True,
        help="This should be the smallest quantity that this product can be produced in. If the BOM contains operations, make sure the work center capacity is accurate.")
    product_uom_id = fields.Many2one(
        'uom.uom', 'Unit of Measure',
        default=_get_default_product_uom_id, required=True,
        help="Unit of Measure (Unit of Measure) is the unit of measurement for the inventory control", domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_tmpl_id.uom_id.category_id')
    sequence = fields.Integer('Sequence', help="Gives the sequence order when displaying a list of bills of material.")
    operation_ids = fields.One2many('mrp.routing.workcenter', 'bom_id', 'Operations', copy=True)
    ready_to_produce = fields.Selection([
        ('all_available', ' When all components are available'),
        ('asap', 'When components for 1st operation are available')], string='Manufacturing Readiness',
        default='all_available', help="Defines when a Manufacturing Order is considered as ready to be started", required=True)
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
        ('flexible', 'Allowed'),
        ('warning', 'Allowed with warning'),
        ('strict', 'Blocked')],
        help="Defines if you can consume more or less components than the quantity defined on the BoM:\n"
             "  * Allowed: allowed for all manufacturing users.\n"
             "  * Allowed with warning: allowed for all manufacturing users with summary of consumption differences when closing the manufacturing order.\n"
             "  * Blocked: only a manager can close a manufacturing order when the BoM consumption is not respected.",
        default='warning',
        string='Flexible Consumption',
        required=True
    )
    possible_product_template_attribute_value_ids = fields.Many2many(
        'product.template.attribute.value',
        compute='_compute_possible_product_template_attribute_value_ids')

    _sql_constraints = [
        ('qty_positive', 'check (product_qty > 0)', 'The quantity to produce must be positive!'),
    ]

    @api.depends(
        'product_tmpl_id.attribute_line_ids.value_ids',
        'product_tmpl_id.attribute_line_ids.attribute_id.create_variant',
        'product_tmpl_id.attribute_line_ids.product_template_value_ids.ptav_active',
    )
    def _compute_possible_product_template_attribute_value_ids(self):
        for bom in self:
            bom.possible_product_template_attribute_value_ids = bom.product_tmpl_id.valid_product_template_attribute_line_ids._without_no_variant_attributes().product_template_value_ids._only_active()

    @api.onchange('product_id')
    def _onchange_product_id(self):
        if self.product_id:
            self.bom_line_ids.bom_product_template_attribute_value_ids = False
            self.operation_ids.bom_product_template_attribute_value_ids = False
            self.byproduct_ids.bom_product_template_attribute_value_ids = False

    @api.constrains('active', 'product_id', 'product_tmpl_id', 'bom_line_ids')
    def _check_bom_cycle(self):
        boms_dict = dict()

        def _check_cycle(components, finished_products):
            """
            Check whether the components are part of the finished products (-> cycle). Then, if
            these components have a BoM, repeat the operation with the subcomponents (recursion).
            The method will return the list of product variants that creates the cycle
            """
            products_to_find = self.env['product.product']

            for component in components:
                if component in finished_products:
                    names = finished_products.mapped('display_name')
                    raise ValidationError(_("The current configuration is incorrect because it would create a cycle "
                                            "between these products: %s.") % ', '.join(names))
                if component not in boms_dict:
                    products_to_find |= component

            bom_find_result = self._bom_find(products_to_find)
            for component in components:
                if component not in boms_dict:
                    bom = bom_find_result[component]
                    boms_dict[component] = bom
                bom = boms_dict[component]
                subcomponents = bom.bom_line_ids.filtered(lambda l: not l._skip_bom_line(component)).product_id
                if subcomponents:
                    _check_cycle(subcomponents, finished_products | component)

        boms_to_check = self
        domain = []
        for product in self.bom_line_ids.product_id:
            domain = OR([domain, self._bom_find_domain(product)])
        if domain:
            boms_to_check |= self.env['mrp.bom'].search(domain)

        for bom in boms_to_check:
            if not bom.active:
                continue
            finished_products = bom.product_id or bom.product_tmpl_id.product_variant_ids
            if bom.bom_line_ids.bom_product_template_attribute_value_ids:
                grouped_by_components = defaultdict(lambda: self.env['product.product'])
                for finished in finished_products:
                    components = bom.bom_line_ids.filtered(lambda l: not l._skip_bom_line(finished)).product_id
                    grouped_by_components[components] |= finished
                for components, finished in grouped_by_components.items():
                    _check_cycle(components, finished)
            else:
                _check_cycle(bom.bom_line_ids.product_id, finished_products)

    def write(self, vals):
        res = super().write(vals)
        if 'sequence' in vals and self and self[-1].id == self._prefetch_ids[-1]:
            self.browse(self._prefetch_ids)._check_bom_cycle()
        return res

    @api.constrains('product_id', 'product_tmpl_id', 'bom_line_ids', 'byproduct_ids', 'operation_ids')
    def _check_bom_lines(self):
        for bom in self:
            apply_variants = bom.bom_line_ids.bom_product_template_attribute_value_ids | bom.operation_ids.bom_product_template_attribute_value_ids | bom.byproduct_ids.bom_product_template_attribute_value_ids
            if bom.product_id and apply_variants:
                raise ValidationError(_("You cannot use the 'Apply on Variant' functionality and simultaneously create a BoM for a specific variant."))
            for ptav in apply_variants:
                if ptav.product_tmpl_id != bom.product_tmpl_id:
                    raise ValidationError(_(
                        "The attribute value %(attribute)s set on product %(product)s does not match the BoM product %(bom_product)s.",
                        attribute=ptav.display_name,
                        product=ptav.product_tmpl_id.display_name,
                        bom_product=bom.product_tmpl_id.display_name
                    ))
            for byproduct in bom.byproduct_ids:
                if bom.product_id:
                    same_product = bom.product_id == byproduct.product_id
                else:
                    same_product = bom.product_tmpl_id == byproduct.product_id.product_tmpl_id
                if same_product:
                    raise ValidationError(_("By-product %s should not be the same as BoM product.") % bom.display_name)
                if byproduct.cost_share < 0:
                    raise ValidationError(_("By-products cost shares must be positive."))
            if sum(bom.byproduct_ids.mapped('cost_share')) > 100:
                raise ValidationError(_("The total cost share for a BoM's by-products cannot exceed 100."))

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
            self.bom_line_ids.bom_product_template_attribute_value_ids = False
            self.operation_ids.bom_product_template_attribute_value_ids = False
            self.byproduct_ids.bom_product_template_attribute_value_ids = False

            domain = [('product_tmpl_id', '=', self.product_tmpl_id.id)]
            if self.id.origin:
                domain.append(('id', '!=', self.id.origin))
            number_of_bom_of_this_product = self.env['mrp.bom'].search_count(domain)
            if number_of_bom_of_this_product:  # add a reference to the bom if there is already a bom for this product
                self.code = _("%s (new) %s", self.product_tmpl_id.name, number_of_bom_of_this_product)
            else:
                self.code = False

    def copy(self, default=None):
        res = super().copy(default)
        for bom_line in res.bom_line_ids:
            if bom_line.operation_id:
                operation = res.operation_ids.filtered(lambda op: op._get_comparison_values() == bom_line.operation_id._get_comparison_values())
                # Two operations could have the same values so we take the first one
                bom_line.operation_id = operation[:1]
        return res

    @api.model
    def name_create(self, name):
        # prevent to use string as product_tmpl_id
        if isinstance(name, str):
            raise UserError(_("You cannot create a new Bill of Material from here."))
        return super(MrpBom, self).name_create(name)

    def toggle_active(self):
        self.with_context({'active_test': False}).operation_ids.toggle_active()
        return super().toggle_active()

    def name_get(self):
        return [(bom.id, '%s%s' % (bom.code and '%s: ' % bom.code or '', bom.product_tmpl_id.display_name)) for bom in self]

    @api.constrains('product_tmpl_id', 'product_id', 'type')
    def check_kit_has_not_orderpoint(self):
        product_ids = [pid for bom in self.filtered(lambda bom: bom.type == "phantom")
                           for pid in (bom.product_id.ids or bom.product_tmpl_id.product_variant_ids.ids)]
        if self.env['stock.warehouse.orderpoint'].search([('product_id', 'in', product_ids)], count=True):
            raise ValidationError(_("You can not create a kit-type bill of materials for products that have at least one reordering rule."))

    @api.ondelete(at_uninstall=False)
    def _unlink_except_running_mo(self):
        if self.env['mrp.production'].search([('bom_id', 'in', self.ids), ('state', 'not in', ['done', 'cancel'])], limit=1):
            raise UserError(_('You can not delete a Bill of Material with running manufacturing orders.\nPlease close or cancel it first.'))

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        args = args or []
        domain = []
        if (name or '').strip():
            domain = ['|', (self._rec_name, operator, name), ('code', operator, name)]
            if operator in NEGATIVE_TERM_OPERATORS:
                domain = domain[1:]
        return self._search(AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)

    @api.model
    def _bom_find_domain(self, products, picking_type=None, company_id=False, bom_type=False):
        domain = ['&', '|', ('product_id', 'in', products.ids), '&', ('product_id', '=', False), ('product_tmpl_id', 'in', products.product_tmpl_id.ids), ('active', '=', True)]
        if company_id or self.env.context.get('company_id'):
            domain = AND([domain, ['|', ('company_id', '=', False), ('company_id', '=', company_id or self.env.context.get('company_id'))]])
        if picking_type:
            domain = AND([domain, ['|', ('picking_type_id', '=', picking_type.id), ('picking_type_id', '=', False)]])
        if bom_type:
            domain = AND([domain, [('type', '=', bom_type)]])
        return domain

    @api.model
    def _bom_find(self, products, picking_type=None, company_id=False, bom_type=False):
        """ Find the first BoM for each products

        :param products: `product.product` recordset
        :return: One bom (or empty recordset `mrp.bom` if none find) by product (`product.product` record)
        :rtype: defaultdict(`lambda: self.env['mrp.bom']`)
        """
        bom_by_product = defaultdict(lambda: self.env['mrp.bom'])
        products = products.filtered(lambda p: p.type != 'service')
        if not products:
            return bom_by_product
        domain = self._bom_find_domain(products, picking_type=picking_type, company_id=company_id, bom_type=bom_type)

        # Performance optimization, allow usage of limit and avoid the for loop `bom.product_tmpl_id.product_variant_ids`
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
            product_boms.update(self._bom_find(products, picking_type=picking_type or self.picking_type_id,
                company_id=self.company_id.id, bom_type='phantom'))
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
    product_tmpl_id = fields.Many2one('product.template', 'Product Template', related='product_id.product_tmpl_id', store=True, index=True)
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
    bom_id = fields.Many2one(
        'mrp.bom', 'Parent BoM',
        index=True, ondelete='cascade', required=True)
    parent_product_tmpl_id = fields.Many2one('product.template', 'Parent Product Template', related='bom_id.product_tmpl_id')
    possible_bom_product_template_attribute_value_ids = fields.Many2many(related='bom_id.possible_product_template_attribute_value_ids')
    bom_product_template_attribute_value_ids = fields.Many2many(
        'product.template.attribute.value', string="Apply on Variants", ondelete='restrict',
        domain="[('id', 'in', possible_bom_product_template_attribute_value_ids)]",
        help="BOM Product Variants needed to apply this line.")
    allowed_operation_ids = fields.One2many('mrp.routing.workcenter', related='bom_id.operation_ids')
    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Consumed in Operation', check_company=True,
        domain="[('id', 'in', allowed_operation_ids)]",
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

    @api.depends('product_id', 'bom_id')
    def _compute_child_bom_id(self):
        for line in self:
            if not line.product_id:
                line.child_bom_id = False
            else:
                line.child_bom_id = self.env['mrp.bom']._bom_find(line.product_id)[line.product_id]

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
        custom control.
        """
        self.ensure_one()
        if product._name == 'product.template':
            return False
        return not product._match_all_variant_values(self.bom_product_template_attribute_value_ids)

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
    _order = 'sequence, id'

    product_id = fields.Many2one('product.product', 'By-product', required=True, check_company=True)
    company_id = fields.Many2one(related='bom_id.company_id', store=True, index=True, readonly=True)
    product_qty = fields.Float(
        'Quantity',
        default=1.0, digits='Product Unit of Measure', required=True)
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    product_uom_id = fields.Many2one('uom.uom', 'Unit of Measure', required=True,
                                     domain="[('category_id', '=', product_uom_category_id)]")
    bom_id = fields.Many2one('mrp.bom', 'BoM', ondelete='cascade', index=True)
    allowed_operation_ids = fields.One2many('mrp.routing.workcenter', related='bom_id.operation_ids')
    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Produced in Operation', check_company=True,
        domain="[('id', 'in', allowed_operation_ids)]")
    possible_bom_product_template_attribute_value_ids = fields.Many2many(related='bom_id.possible_product_template_attribute_value_ids')
    bom_product_template_attribute_value_ids = fields.Many2many(
        'product.template.attribute.value', string="Apply on Variants", ondelete='restrict',
        domain="[('id', 'in', possible_bom_product_template_attribute_value_ids)]",
        help="BOM Product Variants needed to apply this line.")
    sequence = fields.Integer("Sequence")
    cost_share = fields.Float(
        "Cost Share (%)", digits=(5, 2),  # decimal = 2 is important for rounding calculations!!
        help="The percentage of the final production cost for this by-product line (divided between the quantity produced)."
             "The total of all by-products' cost share must be less than or equal to 100.")

    @api.onchange('product_id')
    def _onchange_product_id(self):
        """ Changes UoM if product_id changes. """
        if self.product_id:
            self.product_uom_id = self.product_id.uom_id.id

    def _skip_byproduct_line(self, product):
        """ Control if a byproduct line should be produced, can be inherited to add
        custom control.
        """
        self.ensure_one()
        if product._name == 'product.template':
            return False
        return not product._match_all_variant_values(self.bom_product_template_attribute_value_ids)

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

    def unlink(self):
        self.mapped('ir_attachment_id').unlink()
        return super(MrpDocument, self).unlink()

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import datetime
import math
import re
import warnings

from collections import defaultdict
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_compare, float_round, float_is_zero, format_datetime
from odoo.tools.misc import OrderedSet, format_date, groupby as tools_groupby

from odoo.addons.stock.models.stock_move import PROCUREMENT_PRIORITIES

SIZE_BACK_ORDER_NUMERING = 3

class MrpProduction(models.Model):
    """ Manufacturing Orders """
    _name = 'mrp.production'
    _description = 'Production Order'
    _date_name = 'date_planned_start'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'priority desc, date_planned_start asc,id'

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

    @api.model
    def _get_default_date_planned_start(self):
        if self.env.context.get('default_date_deadline'):
            return fields.Datetime.to_datetime(self.env.context.get('default_date_deadline'))
        return datetime.datetime.now()

    @api.model
    def _get_default_is_locked(self):
        return not self.user_has_groups('mrp.group_unlocked_by_default')

    name = fields.Char(
        'Reference', copy=False, readonly=True, default=lambda x: _('New'))
    priority = fields.Selection(
        PROCUREMENT_PRIORITIES, string='Priority', default='0',
        help="Components will be reserved first for the MO with the highest priorities.")
    backorder_sequence = fields.Integer("Backorder Sequence", default=0, copy=False, help="Backorder sequence, if equals to 0 means there is not related backorder")
    origin = fields.Char(
        'Source', copy=False,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help="Reference of the document that generated this production order request.")

    product_id = fields.Many2one(
        'product.product', 'Product',
        domain="""[
            ('type', 'in', ['product', 'consu']),
            '|',
                ('company_id', '=', False),
                ('company_id', '=', company_id)
        ]
        """,
        readonly=True, required=True, check_company=True,
        states={'draft': [('readonly', False)]})
    product_tracking = fields.Selection(related='product_id.tracking')
    product_tmpl_id = fields.Many2one('product.template', 'Product Template', related='product_id.product_tmpl_id')
    product_qty = fields.Float(
        'Quantity To Produce',
        default=1.0, digits='Product Unit of Measure',
        readonly=True, required=True, tracking=True,
        states={'draft': [('readonly', False)]})
    product_uom_id = fields.Many2one(
        'uom.uom', 'Product Unit of Measure',
        readonly=True, required=True,
        states={'draft': [('readonly', False)]}, domain="[('category_id', '=', product_uom_category_id)]")
    lot_producing_id = fields.Many2one(
        'stock.production.lot', string='Lot/Serial Number', copy=False,
        domain="[('product_id', '=', product_id), ('company_id', '=', company_id)]", check_company=True)
    qty_producing = fields.Float(string="Quantity Producing", digits='Product Unit of Measure', copy=False)
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    product_uom_qty = fields.Float(string='Total Quantity', compute='_compute_product_uom_qty', store=True)
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type',
        domain="[('code', '=', 'mrp_operation'), ('company_id', '=', company_id)]",
        default=_get_default_picking_type, required=True, check_company=True,
        readonly=True, states={'draft': [('readonly', False)]})
    use_create_components_lots = fields.Boolean(related='picking_type_id.use_create_components_lots')
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
        'Scheduled Date', copy=False, default=_get_default_date_planned_start,
        help="Date at which you plan to start the production.",
        index=True, required=True)
    date_planned_finished = fields.Datetime(
        'Scheduled End Date',
        default=_get_default_date_planned_finished,
        help="Date at which you plan to finish the production.",
        copy=False)
    date_deadline = fields.Datetime(
        'Deadline', copy=False, store=True, readonly=True, compute='_compute_date_deadline', inverse='_set_date_deadline',
        help="Informative date allowing to define when the manufacturing order should be processed at the latest to fulfill delivery on time.")
    date_start = fields.Datetime('Start Date', copy=False, readonly=True, help="Date of the WO")
    date_finished = fields.Datetime('End Date', copy=False, readonly=True, help="Date when the MO has been close")

    production_duration_expected = fields.Float("Expected Duration", help="Total expected duration (in minutes)", compute='_compute_production_duration_expected')
    production_real_duration = fields.Float("Real Duration", help="Total real duration (in minutes)", compute='_compute_production_real_duration')

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

    state = fields.Selection([
        ('draft', 'Draft'),
        ('confirmed', 'Confirmed'),
        ('progress', 'In Progress'),
        ('to_close', 'To Close'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')], string='State',
        compute='_compute_state', copy=False, index=True, readonly=True,
        store=True, tracking=True,
        help=" * Draft: The MO is not confirmed yet.\n"
             " * Confirmed: The MO is confirmed, the stock rules and the reordering of the components are trigerred.\n"
             " * In Progress: The production has started (on the MO or on the WO).\n"
             " * To Close: The production is done, the MO has to be closed.\n"
             " * Done: The MO is closed, the stock moves are posted. \n"
             " * Cancelled: The MO has been cancelled, can't be confirmed anymore.")
    reservation_state = fields.Selection([
        ('confirmed', 'Waiting'),
        ('assigned', 'Ready'),
        ('waiting', 'Waiting Another Operation')],
        string='MO Readiness',
        compute='_compute_reservation_state', copy=False, index=True, readonly=True,
        store=True, tracking=True,
        help="Manufacturing readiness for this MO, as per bill of material configuration:\n\
            * Ready: The material is available to start the production.\n\
            * Waiting: The material is not available to start the production.\n")

    move_raw_ids = fields.One2many(
        'stock.move', 'raw_material_production_id', 'Components',
        copy=False, states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        domain=[('scrapped', '=', False)])
    move_finished_ids = fields.One2many(
        'stock.move', 'production_id', 'Finished Products',
        copy=False, states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        domain=[('scrapped', '=', False)])
    move_byproduct_ids = fields.One2many('stock.move', compute='_compute_move_byproduct_ids', inverse='_set_move_byproduct_ids')
    finished_move_line_ids = fields.One2many(
        'stock.move.line', compute='_compute_lines', inverse='_inverse_lines', string="Finished Product"
        )
    workorder_ids = fields.One2many(
        'mrp.workorder', 'production_id', 'Work Orders', copy=True)
    move_dest_ids = fields.One2many('stock.move', 'created_production_id',
        string="Stock Movements of Produced Goods")

    unreserve_visible = fields.Boolean(
        'Allowed to Unreserve Production', compute='_compute_unreserve_visible',
        help='Technical field to check when we can unreserve')
    reserve_visible = fields.Boolean(
        'Allowed to Reserve Production', compute='_compute_unreserve_visible',
        help='Technical field to check when we can reserve quantities')
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
    product_description_variants = fields.Char('Custom Description')
    orderpoint_id = fields.Many2one('stock.warehouse.orderpoint', 'Orderpoint', index=True, copy=False)
    propagate_cancel = fields.Boolean(
        'Propagate cancel and split',
        help='If checked, when the previous move of the move (which was generated by a next procurement) is cancelled or split, the move generated by this move will too')
    delay_alert_date = fields.Datetime('Delay Alert Date', compute='_compute_delay_alert_date', search='_search_delay_alert_date')
    json_popover = fields.Char('JSON data for the popover widget', compute='_compute_json_popover')
    scrap_ids = fields.One2many('stock.scrap', 'production_id', 'Scraps')
    scrap_count = fields.Integer(compute='_compute_scrap_move_count', string='Scrap Move')
    is_locked = fields.Boolean('Is Locked', default=_get_default_is_locked, copy=False)
    is_planned = fields.Boolean('Its Operations are Planned', compute="_compute_is_planned", store=True)

    show_final_lots = fields.Boolean('Show Final Lots', compute='_compute_show_lots')
    production_location_id = fields.Many2one('stock.location', "Production Location", compute="_compute_production_location", store=True)
    picking_ids = fields.Many2many('stock.picking', compute='_compute_picking_ids', string='Picking associated to this manufacturing order')
    delivery_count = fields.Integer(string='Delivery Orders', compute='_compute_picking_ids')
    confirm_cancel = fields.Boolean(compute='_compute_confirm_cancel')
    consumption = fields.Selection([
        ('flexible', 'Allowed'),
        ('warning', 'Allowed with warning'),
        ('strict', 'Blocked')],
        required=True,
        readonly=True,
        default='flexible',
    )

    mrp_production_child_count = fields.Integer("Number of generated MO", compute='_compute_mrp_production_child_count')
    mrp_production_source_count = fields.Integer("Number of source MO", compute='_compute_mrp_production_source_count')
    mrp_production_backorder_count = fields.Integer("Count of linked backorder", compute='_compute_mrp_production_backorder')
    show_lock = fields.Boolean('Show Lock/unlock buttons', compute='_compute_show_lock')
    components_availability = fields.Char(
        string="Component Status", compute='_compute_components_availability',
        help="Latest component availability status for this MO. If green, then the MO's readiness status is ready, as per BOM configuration.")
    components_availability_state = fields.Selection([
        ('available', 'Available'),
        ('expected', 'Expected'),
        ('late', 'Late')], compute='_compute_components_availability')
    show_lot_ids = fields.Boolean('Display the serial number shortcut on the moves', compute='_compute_show_lot_ids')
    forecasted_issue = fields.Boolean(compute='_compute_forecasted_issue')
    show_serial_mass_produce = fields.Boolean('Display the serial mass product wizard action moves', compute='_compute_show_serial_mass_produce')

    @api.depends('procurement_group_id.stock_move_ids.created_production_id.procurement_group_id.mrp_production_ids',
                 'procurement_group_id.stock_move_ids.move_orig_ids.created_production_id.procurement_group_id.mrp_production_ids')
    def _compute_mrp_production_child_count(self):
        for production in self:
            production.mrp_production_child_count = len(production._get_children())

    @api.depends('procurement_group_id.mrp_production_ids.move_dest_ids.group_id.mrp_production_ids',
                 'procurement_group_id.stock_move_ids.move_dest_ids.group_id.mrp_production_ids')
    def _compute_mrp_production_source_count(self):
        for production in self:
            production.mrp_production_source_count = len(production._get_sources())

    @api.depends('procurement_group_id.mrp_production_ids')
    def _compute_mrp_production_backorder(self):
        for production in self:
            production.mrp_production_backorder_count = len(production.procurement_group_id.mrp_production_ids)

    @api.depends('state', 'reservation_state', 'date_planned_start', 'move_raw_ids', 'move_raw_ids.forecast_availability', 'move_raw_ids.forecast_expected_date')
    def _compute_components_availability(self):
        productions = self.filtered(lambda mo: mo.state not in ('cancel', 'done', 'draft'))
        productions.components_availability_state = 'available'
        productions.components_availability = _('Available')

        other_productions = self - productions
        other_productions.components_availability = False
        other_productions.components_availability_state = False

        all_raw_moves = productions.move_raw_ids
        # Force to prefetch more than 1000 by 1000
        all_raw_moves._fields['forecast_availability'].compute_value(all_raw_moves)
        for production in productions:
            if any(float_compare(move.forecast_availability, 0 if move.state == 'draft' else move.product_qty, precision_rounding=move.product_id.uom_id.rounding) == -1 for move in production.move_raw_ids):
                production.components_availability = _('Not Available')
                production.components_availability_state = 'late'
            else:
                forecast_date = max(production.move_raw_ids.filtered('forecast_expected_date').mapped('forecast_expected_date'), default=False)
                if forecast_date:
                    production.components_availability = _('Exp %s', format_date(self.env, forecast_date))
                    if production.date_planned_start:
                        production.components_availability_state = 'late' if forecast_date > production.date_planned_start else 'expected'

    @api.depends('move_finished_ids.date_deadline')
    def _compute_date_deadline(self):
        for production in self:
            production.date_deadline = min(production.move_finished_ids.filtered('date_deadline').mapped('date_deadline'), default=production.date_deadline or False)

    def _set_date_deadline(self):
        for production in self:
            production.move_finished_ids.date_deadline = production.date_deadline

    @api.depends('workorder_ids.duration_expected')
    def _compute_production_duration_expected(self):
        for production in self:
            production.production_duration_expected = sum(production.workorder_ids.mapped('duration_expected'))

    @api.depends('workorder_ids.duration')
    def _compute_production_real_duration(self):
        for production in self:
            production.production_real_duration = sum(production.workorder_ids.mapped('duration'))

    @api.depends("workorder_ids.date_planned_start", "workorder_ids.date_planned_finished")
    def _compute_is_planned(self):
        for production in self:
            if production.workorder_ids:
                production.is_planned = any(wo.date_planned_start and wo.date_planned_finished for wo in production.workorder_ids)
            else:
                production.is_planned = False

    @api.depends('move_raw_ids.delay_alert_date')
    def _compute_delay_alert_date(self):
        delay_alert_date_data = self.env['stock.move'].read_group([('id', 'in', self.move_raw_ids.ids), ('delay_alert_date', '!=', False)], ['delay_alert_date:max'], 'raw_material_production_id')
        delay_alert_date_data = {data['raw_material_production_id'][0]: data['delay_alert_date'] for data in delay_alert_date_data}
        for production in self:
            production.delay_alert_date = delay_alert_date_data.get(production.id, False)

    def _compute_json_popover(self):
        production_no_alert = self.filtered(lambda m: m.state in ('done', 'cancel') or not m.delay_alert_date)
        production_no_alert.json_popover = False
        for production in (self - production_no_alert):
            production.json_popover = json.dumps({
                'popoverTemplate': 'stock.PopoverStockRescheduling',
                'delay_alert_date': format_datetime(self.env, production.delay_alert_date, dt_format=False),
                'late_elements': [{
                        'id': late_document.id,
                        'name': late_document.display_name,
                        'model': late_document._name,
                    } for late_document in production.move_raw_ids.filtered(lambda m: m.delay_alert_date).move_orig_ids._delay_alert_get_documents()
                ]
            })

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
        action = self.env["ir.actions.actions"]._for_xml_id("stock.action_picking_tree_all")
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

    @api.depends('product_id', 'company_id')
    def _compute_production_location(self):
        if not self.company_id:
            return
        location_by_company = self.env['stock.location'].read_group([
            ('company_id', 'in', self.company_id.ids),
            ('usage', '=', 'production')
        ], ['company_id', 'ids:array_agg(id)'], ['company_id'])
        location_by_company = {lbc['company_id'][0]: lbc['ids'] for lbc in location_by_company}
        for production in self:
            if production.product_id:
                production.production_location_id = production.product_id.with_company(production.company_id).property_stock_production
            else:
                production.production_location_id = location_by_company.get(production.company_id.id)[0]

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

    @api.depends(
        'move_raw_ids.state', 'move_raw_ids.quantity_done', 'move_finished_ids.state',
        'workorder_ids.state', 'product_qty', 'qty_producing')
    def _compute_state(self):
        """ Compute the production state. This uses a similar process to stock
        picking, but has been adapted to support having no moves. This adaption
        includes some state changes outside of this compute.

        There exist 3 extra steps for production:
        - progress: At least one item is produced or consumed.
        - to_close: The quantity produced is greater than the quantity to
        produce and all work orders has been finished.
        """
        for production in self:
            if not production.state or not production.product_uom_id:
                production.state = 'draft'
            elif production.state == 'cancel' or (production.move_finished_ids and all(move.state == 'cancel' for move in production.move_finished_ids)):
                production.state = 'cancel'
            elif (
                production.state == 'done'
                or (production.move_raw_ids and all(move.state in ('cancel', 'done') for move in production.move_raw_ids))
                and all(move.state in ('cancel', 'done') for move in production.move_finished_ids)
            ):
                production.state = 'done'
            elif production.workorder_ids and all(wo_state in ('done', 'cancel') for wo_state in production.workorder_ids.mapped('state')):
                production.state = 'to_close'
            elif not production.workorder_ids and float_compare(production.qty_producing, production.product_qty, precision_rounding=production.product_uom_id.rounding) >= 0:
                production.state = 'to_close'
            elif any(wo_state in ('progress', 'done') for wo_state in production.workorder_ids.mapped('state')):
                production.state = 'progress'
            elif production.product_uom_id and not float_is_zero(production.qty_producing, precision_rounding=production.product_uom_id.rounding):
                production.state = 'progress'
            elif any(not float_is_zero(move.quantity_done, precision_rounding=move.product_uom.rounding or move.product_id.uom_id.rounding) for move in production.move_raw_ids if move.product_id):
                production.state = 'progress'

    @api.depends('state', 'move_raw_ids.state')
    def _compute_reservation_state(self):
        self.reservation_state = False
        for production in self:
            if production.state in ('draft', 'done', 'cancel'):
                continue
            relevant_move_state = production.move_raw_ids._get_relevant_state_among_moves()
            # Compute reservation state according to its component's moves.
            if relevant_move_state == 'partially_available':
                if production.workorder_ids.operation_id and production.bom_id.ready_to_produce == 'asap':
                    production.reservation_state = production._get_ready_to_produce_state()
                else:
                    production.reservation_state = 'confirmed'
            elif relevant_move_state != 'draft':
                production.reservation_state = relevant_move_state

    @api.depends('move_raw_ids', 'state', 'move_raw_ids.product_uom_qty')
    def _compute_unreserve_visible(self):
        for order in self:
            already_reserved = order.state not in ('done', 'cancel') and order.mapped('move_raw_ids.move_line_ids')
            any_quantity_done = any(m.quantity_done > 0 for m in order.move_raw_ids)

            order.unreserve_visible = not any_quantity_done and already_reserved
            order.reserve_visible = order.state in ('confirmed', 'progress', 'to_close') and any(move.product_uom_qty and move.state in ['confirmed', 'partially_available'] for move in order.move_raw_ids)

    @api.depends('workorder_ids.state', 'move_finished_ids', 'move_finished_ids.quantity_done')
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

    @api.depends('move_finished_ids')
    def _compute_move_byproduct_ids(self):
        for order in self:
            order.move_byproduct_ids = order.move_finished_ids.filtered(lambda m: m.product_id != order.product_id)

    def _set_move_byproduct_ids(self):
        move_finished_ids = self.move_finished_ids.filtered(lambda m: m.product_id == self.product_id)
        self.move_finished_ids = move_finished_ids | self.move_byproduct_ids

    @api.depends('state')
    def _compute_show_lock(self):
        for order in self:
            order.show_lock = order.state == 'done' or (
                not self.env.user.has_group('mrp.group_unlocked_by_default')
                and order.id is not False
                and order.state not in {'cancel', 'draft'}
            )

    @api.depends('state','move_raw_ids')
    def _compute_show_lot_ids(self):
        for order in self:
            order.show_lot_ids = order.state != 'draft' and any(m.product_id.tracking == 'serial' for m in order.move_raw_ids)

    @api.depends('state', 'move_raw_ids')
    def _compute_show_serial_mass_produce(self):
        self.show_serial_mass_produce = False
        for order in self:
            if order.state == 'confirmed' and order.product_id.tracking == 'serial' and \
                    float_compare(order.product_qty, 1, precision_rounding=order.product_uom_id.rounding) > 0 and \
                    float_compare(order.qty_producing, order.product_qty, precision_rounding=order.product_uom_id.rounding) < 0:
                have_serial_components, dummy, dummy, dummy = order._check_serial_mass_produce_components()
                if not have_serial_components:
                    order.show_serial_mass_produce = True

    _sql_constraints = [
        ('name_uniq', 'unique(name, company_id)', 'Reference must be unique per Company!'),
        ('qty_positive', 'check (product_qty > 0)', 'The quantity to produce must be positive!'),
    ]

    @api.depends('product_uom_qty', 'date_planned_start')
    def _compute_forecasted_issue(self):
        for order in self:
            warehouse = order.location_dest_id.warehouse_id
            order.forecasted_issue = False
            if order.product_id:
                virtual_available = order.product_id.with_context(warehouse=warehouse.id, to_date=order.date_planned_start).virtual_available
                if order.state == 'draft':
                    virtual_available += order.product_uom_qty
                if virtual_available < 0:
                    order.forecasted_issue = True

    @api.model
    def _search_delay_alert_date(self, operator, value):
        late_stock_moves = self.env['stock.move'].search([('delay_alert_date', operator, value)])
        return ['|', ('move_raw_ids', 'in', late_stock_moves.ids), ('move_finished_ids', 'in', late_stock_moves.ids)]

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            if self.move_raw_ids:
                self.move_raw_ids.update({'company_id': self.company_id})
            if self.picking_type_id and self.picking_type_id.company_id != self.company_id:
                self.picking_type_id = self.env['stock.picking.type'].search([
                    ('code', '=', 'mrp_operation'),
                    ('warehouse_id.company_id', '=', self.company_id.id),
                ], limit=1).id

    @api.onchange('product_id', 'picking_type_id', 'company_id')
    def _onchange_product_id(self):
        """ Finds UoM of changed product. """
        if not self.product_id:
            self.bom_id = False
        elif not self.bom_id or self.bom_id.product_tmpl_id != self.product_tmpl_id or (self.bom_id.product_id and self.bom_id.product_id != self.product_id):
            picking_type_id = self._context.get('default_picking_type_id')
            picking_type = picking_type_id and self.env['stock.picking.type'].browse(picking_type_id)
            bom = self.env['mrp.bom']._bom_find(self.product_id, picking_type=picking_type, company_id=self.company_id.id, bom_type='normal')[self.product_id]
            if bom:
                self.bom_id = bom.id
                self.product_qty = self.bom_id.product_qty
                self.product_uom_id = self.bom_id.product_uom_id.id
            else:
                self.bom_id = False
                self.product_uom_id = self.product_id.uom_id.id

    @api.onchange('product_qty', 'product_uom_id')
    def _onchange_product_qty(self):
        for workorder in self.workorder_ids:
            workorder.product_uom_id = self.product_uom_id
            if self._origin.product_qty:
                workorder.duration_expected = workorder._get_duration_expected(ratio=self.product_qty / self._origin.product_qty)
            else:
                workorder.duration_expected = workorder._get_duration_expected()
            if workorder.date_planned_start and workorder.duration_expected:
                workorder.date_planned_finished = workorder.date_planned_start + relativedelta(minutes=workorder.duration_expected)

    @api.onchange('bom_id')
    def _onchange_bom_id(self):
        if not self.product_id and self.bom_id:
            self.product_id = self.bom_id.product_id or self.bom_id.product_tmpl_id.product_variant_ids[:1]
        self.product_qty = self.bom_id.product_qty or 1.0
        self.product_uom_id = self.bom_id and self.bom_id.product_uom_id.id or self.product_id.uom_id.id
        self.move_raw_ids = [(2, move.id) for move in self.move_raw_ids.filtered(lambda m: m.bom_line_id)]
        self.move_finished_ids = [(2, move.id) for move in self.move_finished_ids]
        picking_type_id = self._context.get('default_picking_type_id')
        picking_type = picking_type_id and self.env['stock.picking.type'].browse(picking_type_id)
        self.picking_type_id = picking_type or self.bom_id.picking_type_id or self.picking_type_id

    @api.onchange('date_planned_start', 'product_id')
    def _onchange_date_planned_start(self):
        if self.date_planned_start and not self.is_planned:
            date_planned_finished = self.date_planned_start + relativedelta(days=self.product_id.produce_delay)
            date_planned_finished = date_planned_finished + relativedelta(days=self.company_id.manufacturing_lead)
            if date_planned_finished == self.date_planned_start:
                date_planned_finished = date_planned_finished + relativedelta(hours=1)
            self.date_planned_finished = date_planned_finished
            self.move_raw_ids = [(1, m.id, {'date': self.date_planned_start}) for m in self.move_raw_ids]
            self.move_finished_ids = [(1, m.id, {'date': date_planned_finished}) for m in self.move_finished_ids]

    @api.onchange('bom_id', 'product_id', 'product_qty', 'product_uom_id')
    def _onchange_move_raw(self):
        if not self.bom_id and not self._origin.product_id:
            return
        # Clear move raws if we are changing the product. In case of creation (self._origin is empty),
        # we need to avoid keeping incorrect lines, so clearing is necessary too.
        if self.product_id != self._origin.product_id:
            self.move_raw_ids = [(5,)]
        if self.bom_id and self.product_id and self.product_qty > 0:
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

    @api.onchange('product_id')
    def _onchange_move_finished_product(self):
        self.move_finished_ids = [(5,)]
        if self.product_id:
            self._create_update_move_finished()

    @api.onchange('bom_id', 'product_qty', 'product_uom_id')
    def _onchange_move_finished(self):
        if self.product_id and self.product_qty > 0:
            self._create_update_move_finished()
        else:
            self.move_finished_ids = [(2, move.id) for move in self.move_finished_ids.filtered(lambda m: m.bom_line_id)]

    @api.onchange('location_src_id', 'move_raw_ids', 'bom_id')
    def _onchange_location(self):
        source_location = self.location_src_id
        self.move_raw_ids.update({
            'warehouse_id': source_location.warehouse_id.id,
            'location_id': source_location.id,
        })

    @api.onchange('location_dest_id', 'move_finished_ids', 'bom_id')
    def _onchange_location_dest(self):
        destination_location = self.location_dest_id
        update_value_list = []
        for move in self.move_finished_ids:
            update_value_list += [(1, move.id, ({
                'warehouse_id': destination_location.warehouse_id.id,
                'location_dest_id': destination_location.id,
            }))]
        self.move_finished_ids = update_value_list

    @api.onchange('picking_type_id')
    def _onchange_picking_type(self):
        if not self.picking_type_id.default_location_src_id or not self.picking_type_id.default_location_dest_id.id:
            company_id = self.company_id.id if (self.company_id and self.company_id in self.env.companies) else self.env.company.id
            fallback_loc = self.env['stock.warehouse'].search([('company_id', '=', company_id)], limit=1).lot_stock_id
        self.location_src_id = self.picking_type_id.default_location_src_id.id or fallback_loc.id
        self.location_dest_id = self.picking_type_id.default_location_dest_id.id or fallback_loc.id

    @api.onchange('qty_producing', 'lot_producing_id')
    def _onchange_producing(self):
        self._set_qty_producing()

    @api.onchange('lot_producing_id')
    def _onchange_lot_producing(self):
        res = self._can_produce_serial_number()
        if res is not True:
            return res

    def _can_produce_serial_number(self, sn=None):
        self.ensure_one()
        sn = sn or self.lot_producing_id
        if self.product_id.tracking == 'serial' and sn:
            message, dummy = self.env['stock.quant']._check_serial_number(self.product_id, sn, self.company_id)
            if message:
                return {'warning': {'title': _('Warning'), 'message': message}}
        return True

    @api.onchange('bom_id', 'product_id')
    def _onchange_workorder_ids(self):
        if self.bom_id and self.product_id:
            self._create_workorder()
        else:
            self.workorder_ids = False

    @api.constrains('move_finished_ids')
    def _check_byproducts(self):
        for order in self:
            if any(move.cost_share < 0 for move in order.move_byproduct_ids):
                raise ValidationError(_("By-products cost shares must be positive."))
            if sum(order.move_byproduct_ids.mapped('cost_share')) > 100:
                raise ValidationError(_("The total cost share for a manufacturing order's by-products cannot exceed 100."))

    @api.constrains('product_id', 'move_raw_ids')
    def _check_production_lines(self):
        for production in self:
            for move in production.move_raw_ids:
                if production.product_id == move.product_id:
                    raise ValidationError(_("The component %s should not be the same as the product to produce.") % production.product_id.display_name)

    def write(self, vals):
        if 'move_byproduct_ids' in vals and 'move_finished_ids' not in vals:
            vals['move_finished_ids'] = vals['move_byproduct_ids']
            del vals['move_byproduct_ids']
        if 'workorder_ids' in self:
            production_to_replan = self.filtered(lambda p: p.is_planned)
        res = super(MrpProduction, self).write(vals)

        for production in self:
            if 'date_planned_start' in vals and not self.env.context.get('force_date', False):
                if production.state in ['done', 'cancel']:
                    raise UserError(_('You cannot move a manufacturing order once it is cancelled or done.'))
                if production.is_planned:
                    production.button_unplan()
            if vals.get('date_planned_start'):
                production.move_raw_ids.write({'date': production.date_planned_start, 'date_deadline': production.date_planned_start})
            if vals.get('date_planned_finished'):
                production.move_finished_ids.write({'date': production.date_planned_finished})
            if any(field in ['move_raw_ids', 'move_finished_ids', 'workorder_ids'] for field in vals) and production.state != 'draft':
                if production.state == 'done':
                    # for some reason moves added after state = 'done' won't save group_id, reference if added in
                    # "stock_move.default_get()"
                    production.move_raw_ids.filtered(lambda move: move.additional and move.date > production.date_planned_start).write({
                        'group_id': production.procurement_group_id.id,
                        'reference': production.name,
                        'date': production.date_planned_start,
                        'date_deadline': production.date_planned_start
                    })
                    production.move_finished_ids.filtered(lambda move: move.additional and move.date > production.date_planned_finished).write({
                        'reference': production.name,
                        'date': production.date_planned_finished,
                        'date_deadline': production.date_deadline
                    })
                production._autoconfirm_production()
                if production in production_to_replan:
                    production._plan_workorders(replan=True)
            if production.state == 'done' and ('lot_producing_id' in vals or 'qty_producing' in vals):
                finished_move_lines = production.move_finished_ids.filtered(
                    lambda move: move.product_id == production.product_id and move.state == 'done').mapped('move_line_ids')
                if 'lot_producing_id' in vals:
                    finished_move_lines.write({'lot_id': vals.get('lot_producing_id')})
                if 'qty_producing' in vals:
                    finished_move_lines.write({'qty_done': vals.get('qty_producing')})
            elif production.state not in ['draft', 'done', 'cancel'] and 'lot_producing_id' in vals:
                production.move_finished_ids.filtered(lambda m: m.product_id == production.product_id).move_line_ids.write({'lot_id': production.lot_producing_id.id})

            if not production.workorder_ids.operation_id and vals.get('date_planned_start') and not vals.get('date_planned_finished'):
                new_date_planned_start = fields.Datetime.to_datetime(vals.get('date_planned_start'))
                if not production.date_planned_finished or new_date_planned_start >= production.date_planned_finished:
                    production.date_planned_finished = new_date_planned_start + datetime.timedelta(hours=1)
        return res

    @api.model
    def create(self, values):
        # Remove from `move_finished_ids` the by-product moves and then move `move_byproduct_ids`
        # into `move_finished_ids` to avoid duplicate and inconsistency.
        if values.get('move_finished_ids', False) and values.get('move_byproduct_ids', False):
            values['move_finished_ids'] = list(filter(lambda move: move[2].get('byproduct_id', False) is False, values['move_finished_ids']))
        if values.get('move_byproduct_ids', False):
            values['move_finished_ids'] = values.get('move_finished_ids', []) + values['move_byproduct_ids']
            del values['move_byproduct_ids']
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
        (production.move_raw_ids | production.move_finished_ids).write({
            'group_id': production.procurement_group_id.id,
            'origin': production.name
        })
        production.move_raw_ids.write({'date': production.date_planned_start})
        production.move_finished_ids.write({'date': production.date_planned_finished})
        # Trigger SM & WO creation when importing a file
        if 'import_file' in self.env.context:
            production._onchange_move_raw()
            production._onchange_move_finished()
            production._onchange_workorder_ids()
        return production

    @api.ondelete(at_uninstall=False)
    def _unlink_except_done(self):
        if any(production.state == 'done' for production in self):
            raise UserError(_('Cannot delete a manufacturing order in done state.'))
        not_cancel = self.filtered(lambda m: m.state != 'cancel')
        if not_cancel:
            productions_name = ', '.join([prod.display_name for prod in not_cancel])
            raise UserError(_('%s cannot be deleted. Try to cancel them before.', productions_name))

    def unlink(self):
        self.action_cancel()
        workorders_to_delete = self.workorder_ids.filtered(lambda wo: wo.state != 'done')
        if workorders_to_delete:
            workorders_to_delete.unlink()
        return super(MrpProduction, self).unlink()

    def copy_data(self, default=None):
        default = dict(default or {})
        # covers at least 2 cases: backorders generation (follow default logic for moves copying)
        # and copying a done MO via the form (i.e. copy only the non-cancelled moves since no backorder = cancelled finished moves)
        if not default or 'move_finished_ids' not in default:
            move_finished_ids = self.move_finished_ids
            if self.state != 'cancel':
                move_finished_ids = self.move_finished_ids.filtered(lambda m: m.state != 'cancel' and m.product_qty != 0.0)
            default['move_finished_ids'] = [(0, 0, move.copy_data()[0]) for move in move_finished_ids]
        if not default or 'move_raw_ids' not in default:
            default['move_raw_ids'] = [(0, 0, move.copy_data()[0]) for move in self.move_raw_ids.filtered(lambda m: m.product_qty != 0.0)]
        return super(MrpProduction, self).copy_data(default=default)

    def action_toggle_is_locked(self):
        self.ensure_one()
        self.is_locked = not self.is_locked
        return True

    def action_product_forecast_report(self):
        self.ensure_one()
        action = self.product_id.action_product_forecast_report()
        action['context'] = {
            'active_id': self.product_id.id,
            'active_model': 'product.product',
            'move_to_match_ids': self.move_finished_ids.filtered(lambda m: m.product_id == self.product_id).ids
        }
        warehouse = self.picking_type_id.warehouse_id
        if warehouse:
            action['context']['warehouse'] = warehouse.id
        return action

    def _create_workorder(self):
        for production in self:
            if not production.bom_id or not production.product_id:
                continue
            workorders_values = []

            product_qty = production.product_uom_id._compute_quantity(production.product_qty, production.bom_id.product_uom_id)
            exploded_boms, dummy = production.bom_id.explode(production.product_id, product_qty / production.bom_id.product_qty, picking_type=production.bom_id.picking_type_id)

            for bom, bom_data in exploded_boms:
                # If the operations of the parent BoM and phantom BoM are the same, don't recreate work orders.
                if not (bom.operation_ids and (not bom_data['parent_line'] or bom_data['parent_line'].bom_id.operation_ids != bom.operation_ids)):
                    continue
                for operation in bom.operation_ids:
                    if operation._skip_operation_line(bom_data['product']):
                        continue
                    workorders_values += [{
                        'name': operation.name,
                        'production_id': production.id,
                        'workcenter_id': operation.workcenter_id.id,
                        'product_uom_id': production.product_uom_id.id,
                        'operation_id': operation.id,
                        'state': 'pending',
                    }]
            production.workorder_ids = [(5, 0)] + [(0, 0, value) for value in workorders_values]
            for workorder in production.workorder_ids:
                workorder.duration_expected = workorder._get_duration_expected()

    def _get_move_finished_values(self, product_id, product_uom_qty, product_uom, operation_id=False, byproduct_id=False, cost_share=0):
        group_orders = self.procurement_group_id.mrp_production_ids
        move_dest_ids = self.move_dest_ids
        if len(group_orders) > 1:
            move_dest_ids |= group_orders[0].move_finished_ids.filtered(lambda m: m.product_id == self.product_id).move_dest_ids
        date_planned_finished = self.date_planned_start + relativedelta(days=self.product_id.produce_delay)
        date_planned_finished = date_planned_finished + relativedelta(days=self.company_id.manufacturing_lead)
        if date_planned_finished == self.date_planned_start:
            date_planned_finished = date_planned_finished + relativedelta(hours=1)
        return {
            'product_id': product_id,
            'product_uom_qty': product_uom_qty,
            'product_uom': product_uom,
            'operation_id': operation_id,
            'byproduct_id': byproduct_id,
            'name': self.name,
            'date': date_planned_finished,
            'date_deadline': self.date_deadline,
            'picking_type_id': self.picking_type_id.id,
            'location_id': self.product_id.with_company(self.company_id).property_stock_production.id,
            'location_dest_id': self.location_dest_id.id,
            'company_id': self.company_id.id,
            'production_id': self.id,
            'warehouse_id': self.location_dest_id.warehouse_id.id,
            'origin': self.name,
            'group_id': self.procurement_group_id.id,
            'propagate_cancel': self.propagate_cancel,
            'move_dest_ids': [(4, x.id) for x in self.move_dest_ids if not byproduct_id],
            'cost_share': cost_share,
        }

    def _get_moves_finished_values(self):
        moves = []
        for production in self:
            if production.product_id in production.bom_id.byproduct_ids.mapped('product_id'):
                raise UserError(_("You cannot have %s  as the finished product and in the Byproducts", self.product_id.name))
            moves.append(production._get_move_finished_values(production.product_id.id, production.product_qty, production.product_uom_id.id))
            for byproduct in production.bom_id.byproduct_ids:
                if byproduct._skip_byproduct_line(production.product_id):
                    continue
                product_uom_factor = production.product_uom_id._compute_quantity(production.product_qty, production.bom_id.product_uom_id)
                qty = byproduct.product_qty * (product_uom_factor / production.bom_id.product_qty)
                moves.append(production._get_move_finished_values(
                    byproduct.product_id.id, qty, byproduct.product_uom_id.id,
                    byproduct.operation_id.id, byproduct.id, byproduct.cost_share))
        return moves

    def _create_update_move_finished(self):
        """ This is a helper function to support complexity of onchange logic for MOs.
        It is important that the special *2Many commands used here remain as long as function
        is used within onchanges.
        """
        # keep manual entries
        list_move_finished = [(4, move.id) for move in self.move_finished_ids.filtered(
            lambda m: not m.byproduct_id and m.product_id != self.product_id)]
        list_move_finished = []
        moves_finished_values = self._get_moves_finished_values()
        moves_byproduct_dict = {move.byproduct_id.id: move for move in self.move_finished_ids.filtered(lambda m: m.byproduct_id)}
        move_finished = self.move_finished_ids.filtered(lambda m: m.product_id == self.product_id)
        for move_finished_values in moves_finished_values:
            if move_finished_values.get('byproduct_id') in moves_byproduct_dict:
                # update existing entries
                list_move_finished += [(1, moves_byproduct_dict[move_finished_values['byproduct_id']].id, move_finished_values)]
            elif move_finished_values.get('product_id') == self.product_id.id and move_finished:
                list_move_finished += [(1, move_finished.id, move_finished_values)]
            else:
                # add new entries
                list_move_finished += [(0, 0, move_finished_values)]
        self.move_finished_ids = list_move_finished

    def _get_moves_raw_values(self):
        moves = []
        for production in self:
            if not production.bom_id:
                continue
            factor = production.product_uom_id._compute_quantity(production.product_qty, production.bom_id.product_uom_id) / production.bom_id.product_qty
            boms, lines = production.bom_id.explode(production.product_id, factor, picking_type=production.bom_id.picking_type_id)
            for bom_line, line_data in lines:
                if bom_line.child_bom_id and bom_line.child_bom_id.type == 'phantom' or\
                        bom_line.product_id.type not in ['product', 'consu']:
                    continue
                operation = bom_line.operation_id.id or line_data['parent_line'] and line_data['parent_line'].operation_id.id
                moves.append(production._get_move_raw_values(
                    bom_line.product_id,
                    line_data['qty'],
                    bom_line.product_uom_id,
                    operation,
                    bom_line
                ))
        return moves

    def _get_move_raw_values(self, product_id, product_uom_qty, product_uom, operation_id=False, bom_line=False):
        source_location = self.location_src_id
        origin = self.name
        if self.orderpoint_id and self.origin:
            origin = self.origin.replace(
                '%s - ' % (self.orderpoint_id.display_name), '')
            origin = '%s,%s' % (origin, self.name)
        data = {
            'sequence': bom_line.sequence if bom_line else 10,
            'name': self.name,
            'date': self.date_planned_start,
            'date_deadline': self.date_planned_start,
            'bom_line_id': bom_line.id if bom_line else False,
            'picking_type_id': self.picking_type_id.id,
            'product_id': product_id.id,
            'product_uom_qty': product_uom_qty,
            'product_uom': product_uom.id,
            'location_id': source_location.id,
            'location_dest_id': self.product_id.with_company(self.company_id).property_stock_production.id,
            'raw_material_production_id': self.id,
            'company_id': self.company_id.id,
            'operation_id': operation_id,
            'price_unit': product_id.standard_price,
            'procure_method': 'make_to_stock',
            'origin': origin,
            'state': 'draft',
            'warehouse_id': source_location.warehouse_id.id,
            'group_id': self.procurement_group_id.id,
            'propagate_cancel': self.propagate_cancel,
        }
        return data

    def _set_qty_producing(self):
        if self.product_id.tracking == 'serial':
            qty_producing_uom = self.product_uom_id._compute_quantity(self.qty_producing, self.product_id.uom_id, rounding_method='HALF-UP')
            if qty_producing_uom != 1:
                self.qty_producing = self.product_id.uom_id._compute_quantity(1, self.product_uom_id, rounding_method='HALF-UP')

        for move in (self.move_raw_ids | self.move_finished_ids.filtered(lambda m: m.product_id != self.product_id)):
            if move._should_bypass_set_qty_producing() or not move.product_uom:
                continue
            new_qty = float_round((self.qty_producing - self.qty_produced) * move.unit_factor, precision_rounding=move.product_uom.rounding)
            move.move_line_ids.filtered(lambda ml: ml.state not in ('done', 'cancel')).qty_done = 0
            move._set_quantity_done(new_qty)

    def _update_raw_moves(self, factor):
        self.ensure_one()
        update_info = []
        moves_to_assign = self.env['stock.move']
        for move in self.move_raw_ids.filtered(lambda m: m.state not in ('done', 'cancel')):
            old_qty = move.product_uom_qty
            new_qty = float_round(old_qty * factor, precision_rounding=move.product_uom.rounding, rounding_method='UP')
            if new_qty > 0:
                move.write({'product_uom_qty': new_qty})
                if move._should_bypass_reservation() \
                        or move.picking_type_id.reservation_method == 'at_confirm' \
                        or (move.reservation_date and move.reservation_date <= fields.Date.today()):
                    moves_to_assign |= move
                update_info.append((move, old_qty, new_qty))
        moves_to_assign._action_assign()
        return update_info

    def _get_ready_to_produce_state(self):
        """ returns 'assigned' if enough components are reserved in order to complete
        the first operation of the bom. If not returns 'waiting'
        """
        self.ensure_one()
        operations = self.workorder_ids.operation_id
        if len(operations) == 1:
            moves_in_first_operation = self.move_raw_ids
        else:
            first_operation = operations[0]
            moves_in_first_operation = self.move_raw_ids.filtered(lambda move: move.operation_id == first_operation)
        moves_in_first_operation = moves_in_first_operation.filtered(
            lambda move: move.bom_line_id and
            not move.bom_line_id._skip_bom_line(self.product_id)
        )

        if all(move.state == 'assigned' for move in moves_in_first_operation):
            return 'assigned'
        return 'confirmed'

    def _autoconfirm_production(self):
        """Automatically run `action_confirm` on `self`.

        If the production has one of its move was added after the initial call
        to `action_confirm`.
        """
        moves_to_confirm = self.env['stock.move']
        for production in self:
            if production.state in ('done', 'cancel'):
                continue
            additional_moves = production.move_raw_ids.filtered(
                lambda move: move.state == 'draft'
            )
            additional_moves.write({
                'group_id': production.procurement_group_id.id,
            })
            additional_moves._adjust_procure_method()
            moves_to_confirm |= additional_moves
            additional_byproducts = production.move_finished_ids.filtered(
                lambda move: move.state == 'draft'
            )
            moves_to_confirm |= additional_byproducts

        if moves_to_confirm:
            moves_to_confirm = moves_to_confirm._action_confirm()
            # run scheduler for moves forecasted to not have enough in stock
            moves_to_confirm._trigger_scheduler()

        self.workorder_ids.filtered(lambda w: w.state not in ['done', 'cancel'])._action_confirm()

    def _get_children(self):
        self.ensure_one()
        procurement_moves = self.procurement_group_id.stock_move_ids
        child_moves = procurement_moves.move_orig_ids
        return (procurement_moves | child_moves).created_production_id.procurement_group_id.mrp_production_ids.filtered(lambda p: p.origin != self.origin) - self

    def _get_sources(self):
        self.ensure_one()
        dest_moves = self.procurement_group_id.mrp_production_ids.move_dest_ids
        parent_moves = self.procurement_group_id.stock_move_ids.move_dest_ids
        return (dest_moves | parent_moves).group_id.mrp_production_ids.filtered(lambda p: p.origin != self.origin) - self

    def action_view_mrp_production_childs(self):
        self.ensure_one()
        mrp_production_ids = self._get_children().ids
        action = {
            'res_model': 'mrp.production',
            'type': 'ir.actions.act_window',
        }
        if len(mrp_production_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': mrp_production_ids[0],
            })
        else:
            action.update({
                'name': _("%s Child MO's") % self.name,
                'domain': [('id', 'in', mrp_production_ids)],
                'view_mode': 'tree,form',
            })
        return action

    def action_view_mrp_production_sources(self):
        self.ensure_one()
        mrp_production_ids = self._get_sources().ids
        action = {
            'res_model': 'mrp.production',
            'type': 'ir.actions.act_window',
        }
        if len(mrp_production_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': mrp_production_ids[0],
            })
        else:
            action.update({
                'name': _("MO Generated by %s") % self.name,
                'domain': [('id', 'in', mrp_production_ids)],
                'view_mode': 'tree,form',
            })
        return action

    def action_view_mrp_production_backorders(self):
        backorder_ids = self.procurement_group_id.mrp_production_ids.ids
        return {
            'res_model': 'mrp.production',
            'type': 'ir.actions.act_window',
            'name': _("Backorder MO's"),
            'domain': [('id', 'in', backorder_ids)],
            'view_mode': 'tree,form',
        }

    def action_generate_serial(self):
        self.ensure_one()
        if self.product_id.tracking == 'none':
            return
        name = self.env['stock.production.lot']._get_new_serial(self.company_id, self.product_id)
        self.lot_producing_id = self.env['stock.production.lot'].create({
            'product_id': self.product_id.id,
            'company_id': self.company_id.id,
            'name': name,
        })
        if self.product_id.tracking == 'serial':
            self._set_qty_producing()

    def _action_generate_immediate_wizard(self):
        view = self.env.ref('mrp.view_immediate_production')
        return {
            'name': _('Immediate Production?'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mrp.immediate.production',
            'views': [(view.id, 'form')],
            'view_id': view.id,
            'target': 'new',
            'context': dict(self.env.context, default_mo_ids=[(4, mo.id) for mo in self]),
        }

    def action_confirm(self):
        self._check_company()
        moves_ids_to_confirm = set()
        move_raws_ids_to_adjust = set()
        workorder_ids_to_confirm = set()
        for production in self:
            production_vals = {}
            if production.bom_id:
                production_vals.update({'consumption': production.bom_id.consumption})
            # In case of Serial number tracking, force the UoM to the UoM of product
            if production.product_tracking == 'serial' and production.product_uom_id != production.product_id.uom_id:
                production_vals.update({
                    'product_qty': production.product_uom_id._compute_quantity(production.product_qty, production.product_id.uom_id),
                    'product_uom_id': production.product_id.uom_id
                })
                for move_finish in production.move_finished_ids.filtered(lambda m: m.product_id == production.product_id):
                    move_finish.write({
                        'product_uom_qty': move_finish.product_uom._compute_quantity(move_finish.product_uom_qty, move_finish.product_id.uom_id),
                        'product_uom': move_finish.product_id.uom_id
                    })
            if production_vals:
                production.write(production_vals)
            move_raws_ids_to_adjust.update(production.move_raw_ids.ids)
            moves_ids_to_confirm.update((production.move_raw_ids | production.move_finished_ids).ids)
            workorder_ids_to_confirm.update(production.workorder_ids.ids)

        move_raws_to_adjust = self.env['stock.move'].browse(sorted(move_raws_ids_to_adjust))
        moves_to_confirm = self.env['stock.move'].browse(sorted(moves_ids_to_confirm))
        workorder_to_confirm = self.env['mrp.workorder'].browse(sorted(workorder_ids_to_confirm))

        move_raws_to_adjust._adjust_procure_method()
        moves_to_confirm._action_confirm(merge=False)
        workorder_to_confirm._action_confirm()
        # run scheduler for moves forecasted to not have enough in stock
        self.move_raw_ids._trigger_scheduler()
        self.picking_ids.filtered(
            lambda p: p.state not in ['cancel', 'done']).action_confirm()
        # Force confirm state only for draft production not for more advanced state like
        # 'progress' (in case of backorders with some qty_producing)
        self.filtered(lambda mo: mo.state == 'draft').state = 'confirmed'
        return True

    def action_assign(self):
        for production in self:
            production.move_raw_ids._action_assign()
        return True

    def button_plan(self):
        """ Create work orders. And probably do stuff, like things. """
        orders_to_plan = self.filtered(lambda order: not order.is_planned)
        orders_to_confirm = orders_to_plan.filtered(lambda mo: mo.state == 'draft')
        orders_to_confirm.action_confirm()
        for order in orders_to_plan:
            order._plan_workorders()
        return True

    def _plan_workorders(self, replan=False):
        """ Plan all the production's workorders depending on the workcenters
        work schedule.

        :param replan: If it is a replan, only ready and pending workorder will be taken into account
        :type replan: bool.
        """
        self.ensure_one()

        if not self.workorder_ids:
            return
        # Schedule all work orders (new ones and those already created)
        qty_to_produce = max(self.product_qty - self.qty_produced, 0)
        qty_to_produce = self.product_uom_id._compute_quantity(qty_to_produce, self.product_id.uom_id)
        start_date = max(self.date_planned_start, datetime.datetime.now())
        if replan:
            workorder_ids = self.workorder_ids.filtered(lambda wo: wo.state in ('pending', 'waiting', 'ready'))
            # We plan the manufacturing order according to its `date_planned_start`, but if
            # `date_planned_start` is in the past, we plan it as soon as possible.
            workorder_ids.leave_id.unlink()
        else:
            workorder_ids = self.workorder_ids.filtered(lambda wo: not wo.date_planned_start)
        for workorder in workorder_ids:
            workcenters = workorder.workcenter_id | workorder.workcenter_id.alternative_workcenter_ids

            best_finished_date = datetime.datetime.max
            vals = {}
            for workcenter in workcenters:
                # compute theoretical duration
                if workorder.workcenter_id == workcenter:
                    duration_expected = workorder.duration_expected
                else:
                    duration_expected = workorder._get_duration_expected(alternative_workcenter=workcenter)

                from_date, to_date = workcenter._get_first_available_slot(start_date, duration_expected)
                # If the workcenter is unavailable, try planning on the next one
                if not from_date:
                    continue
                # Check if this workcenter is better than the previous ones
                if to_date and to_date < best_finished_date:
                    best_start_date = from_date
                    best_finished_date = to_date
                    best_workcenter = workcenter
                    vals = {
                        'workcenter_id': workcenter.id,
                        'duration_expected': duration_expected,
                    }

            # If none of the workcenter are available, raise
            if best_finished_date == datetime.datetime.max:
                raise UserError(_('Impossible to plan the workorder. Please check the workcenter availabilities.'))

            # Instantiate start_date for the next workorder planning
            if workorder.next_work_order_id:
                start_date = best_finished_date

            # Create leave on chosen workcenter calendar
            leave = self.env['resource.calendar.leaves'].create({
                'name': workorder.display_name,
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

        self.workorder_ids.leave_id.unlink()
        self.workorder_ids.write({
            'date_planned_start': False,
            'date_planned_finished': False,
        })

    def _get_consumption_issues(self):
        """Compare the quantity consumed of the components, the expected quantity
        on the BoM and the consumption parameter on the order.

        :return: list of tuples (order_id, product_id, consumed_qty, expected_qty) where the
            consumption isn't honored. order_id and product_id are recordset of mrp.production
            and product.product respectively
        :rtype: list
        """
        issues = []
        if self.env.context.get('skip_consumption', False) or self.env.context.get('skip_immediate', False):
            return issues
        for order in self:
            if order.consumption == 'flexible' or not order.bom_id or not order.bom_id.bom_line_ids:
                continue
            expected_move_values = order._get_moves_raw_values()
            expected_qty_by_product = defaultdict(float)
            for move_values in expected_move_values:
                move_product = self.env['product.product'].browse(move_values['product_id'])
                move_uom = self.env['uom.uom'].browse(move_values['product_uom'])
                move_product_qty = move_uom._compute_quantity(move_values['product_uom_qty'], move_product.uom_id)
                expected_qty_by_product[move_product] += move_product_qty * order.qty_producing / order.product_qty

            done_qty_by_product = defaultdict(float)
            for move in order.move_raw_ids:
                qty_done = move.product_uom._compute_quantity(move.quantity_done, move.product_id.uom_id)
                rounding = move.product_id.uom_id.rounding
                if not (move.product_id in expected_qty_by_product or float_is_zero(qty_done, precision_rounding=rounding)):
                    issues.append((order, move.product_id, qty_done, 0.0))
                    continue
                done_qty_by_product[move.product_id] += qty_done

            for product, qty_to_consume in expected_qty_by_product.items():
                qty_done = done_qty_by_product.get(product, 0.0)
                if float_compare(qty_to_consume, qty_done, precision_rounding=product.uom_id.rounding) != 0:
                    issues.append((order, product, qty_done, qty_to_consume))

        return issues

    def _action_generate_consumption_wizard(self, consumption_issues):
        ctx = self.env.context.copy()
        lines = []
        for order, product_id, consumed_qty, expected_qty in consumption_issues:
            lines.append((0, 0, {
                'mrp_production_id': order.id,
                'product_id': product_id.id,
                'consumption': order.consumption,
                'product_uom_id': product_id.uom_id.id,
                'product_consumed_qty_uom': consumed_qty,
                'product_expected_qty_uom': expected_qty
            }))
        ctx.update({'default_mrp_production_ids': self.ids,
                    'default_mrp_consumption_warning_line_ids': lines,
                    'form_view_ref': False})
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.action_mrp_consumption_warning")
        action['context'] = ctx
        return action

    def _get_quantity_produced_issues(self):
        quantity_issues = []
        if self.env.context.get('skip_backorder', False):
            return quantity_issues
        for order in self:
            if not float_is_zero(order._get_quantity_to_backorder(), precision_rounding=order.product_uom_id.rounding):
                quantity_issues.append(order)
        return quantity_issues

    def _action_generate_backorder_wizard(self, quantity_issues):
        ctx = self.env.context.copy()
        lines = []
        for order in quantity_issues:
            lines.append((0, 0, {
                'mrp_production_id': order.id,
                'to_backorder': True
            }))
        ctx.update({'default_mrp_production_ids': self.ids, 'default_mrp_production_backorder_line_ids': lines})
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.action_mrp_production_backorder")
        action['context'] = ctx
        return action

    def action_cancel(self):
        """ Cancels production order, unfinished stock moves and set procurement
        orders in exception """
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
            # log an activity on Parent MO if child MO is cancelled.
            finish_moves = production.move_finished_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
            if finish_moves:
                production._log_downside_manufactured_quantity({finish_move: (production.product_uom_qty, 0.0) for finish_move in finish_moves}, cancel=True)

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

    def _post_inventory(self, cancel_backorder=False):
        moves_to_do, moves_not_to_do = set(), set()
        for move in self.move_raw_ids:
            if move.state == 'done':
                moves_not_to_do.add(move.id)
            elif move.state != 'cancel':
                moves_to_do.add(move.id)
                if move.product_qty == 0.0 and move.quantity_done > 0:
                    move.product_uom_qty = move.quantity_done
        self.env['stock.move'].browse(moves_to_do)._action_done(cancel_backorder=cancel_backorder)
        moves_to_do = self.move_raw_ids.filtered(lambda x: x.state == 'done') - self.env['stock.move'].browse(moves_not_to_do)
        # Create a dict to avoid calling filtered inside for loops.
        moves_to_do_by_order = defaultdict(lambda: self.env['stock.move'], [
            (key, self.env['stock.move'].concat(*values))
            for key, values in tools_groupby(moves_to_do, key=lambda m: m.raw_material_production_id.id)
        ])
        for order in self:
            finish_moves = order.move_finished_ids.filtered(lambda m: m.product_id == order.product_id and m.state not in ('done', 'cancel'))
            # the finish move can already be completed by the workorder.
            if finish_moves and not finish_moves.quantity_done:
                finish_moves._set_quantity_done(float_round(order.qty_producing - order.qty_produced, precision_rounding=order.product_uom_id.rounding, rounding_method='HALF-UP'))
                finish_moves.move_line_ids.lot_id = order.lot_producing_id
            # workorder duration need to be set to calculate the price of the product
            for workorder in order.workorder_ids:
                if workorder.state not in ('done', 'cancel'):
                    workorder.duration_expected = workorder._get_duration_expected()
                if workorder.duration == 0.0:
                    workorder.duration = workorder.duration_expected * order.qty_produced / order.product_qty
            order._cal_price(moves_to_do_by_order[order.id])
        moves_to_finish = self.move_finished_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
        moves_to_finish = moves_to_finish._action_done(cancel_backorder=cancel_backorder)
        self.action_assign()
        for order in self:
            consume_move_lines = moves_to_do_by_order[order.id].mapped('move_line_ids')
            order.move_finished_ids.move_line_ids.consume_line_ids = [(6, 0, consume_move_lines.ids)]
        return True

    @api.model
    def _get_name_backorder(self, name, sequence):
        if not sequence:
            return name
        seq_back = "-" + "0" * (SIZE_BACK_ORDER_NUMERING - 1 - int(math.log10(sequence))) + str(sequence)
        regex = re.compile(r"-\d+$")
        if regex.search(name) and sequence > 1:
            return regex.sub(seq_back, name)
        return name + seq_back

    def _get_backorder_mo_vals(self):
        self.ensure_one()
        if not self.procurement_group_id:
            # in the rare case that the procurement group has been removed somehow, create a new one
            self.procurement_group_id = self.env["procurement.group"].create({'name': self.name})
        next_seq = max(self.procurement_group_id.mrp_production_ids.mapped("backorder_sequence"), default=1)
        return {
            'name': self._get_name_backorder(self.name, next_seq + 1),
            'backorder_sequence': next_seq + 1,
            'procurement_group_id': self.procurement_group_id.id,
            'move_raw_ids': None,
            'move_finished_ids': None,
            'product_qty': self._get_quantity_to_backorder(),
            'lot_producing_id': False,
            'origin': self.origin
        }

    def _generate_backorder_productions(self, close_mo=True):
        backorders = self.env['mrp.production']
        bo_to_assign = self.env['mrp.production']
        for production in self:
            if production.backorder_sequence == 0:  # Activate backorder naming
                production.backorder_sequence = 1
            production.name = self._get_name_backorder(production.name, production.backorder_sequence)
            backorder_mo = production.copy(default=production._get_backorder_mo_vals())
            if close_mo:
                production.move_raw_ids.filtered(lambda m: m.state not in ('done', 'cancel')).write({
                    'raw_material_production_id': backorder_mo.id,
                })
                production.move_finished_ids.filtered(lambda m: m.state not in ('done', 'cancel')).write({
                    'production_id': backorder_mo.id,
                })
            else:
                new_moves_vals = []
                for move in production.move_raw_ids | production.move_finished_ids:
                    if not move.additional:
                        qty_to_split = move.product_uom_qty - move.unit_factor * production.qty_producing
                        qty_to_split = move.product_uom._compute_quantity(qty_to_split, move.product_id.uom_id, rounding_method='HALF-UP')
                        move_vals = move._split(qty_to_split)
                        if not move_vals:
                            continue
                        if move.raw_material_production_id:
                            move_vals[0]['raw_material_production_id'] = backorder_mo.id
                        else:
                            move_vals[0]['production_id'] = backorder_mo.id
                        new_moves_vals.append(move_vals[0])
                self.env['stock.move'].create(new_moves_vals)
            backorders |= backorder_mo
            if backorder_mo.picking_type_id.reservation_method == 'at_confirm':
                bo_to_assign |= backorder_mo

            # We need to adapt `duration_expected` on both the original workorders and their
            # backordered workorders. To do that, we use the original `duration_expected` and the
            # ratio of the quantity really produced and the quantity to produce.
            ratio = production.qty_producing / production.product_qty
            for workorder in production.workorder_ids:
                workorder.duration_expected = workorder.duration_expected * ratio
            for workorder in backorder_mo.workorder_ids:
                workorder.duration_expected = workorder.duration_expected * (1 - ratio)

        # As we have split the moves before validating them, we need to 'remove' the excess reservation
        if not close_mo:
            raw_moves = self.move_raw_ids.filtered(lambda m: not m.additional)
            raw_moves._do_unreserve()
            for sml in raw_moves.move_line_ids:
                try:
                    q = self.env['stock.quant']._update_reserved_quantity(sml.product_id, sml.location_id, sml.qty_done,
                                                                          lot_id=sml.lot_id, package_id=sml.package_id,
                                                                          owner_id=sml.owner_id, strict=True)
                    reserved_qty = sum([x[1] for x in q])
                    reserved_qty = sml.product_id.uom_id._compute_quantity(reserved_qty, sml.product_uom_id)
                except UserError:
                    reserved_qty = 0
                sml.with_context(bypass_reservation_update=True).product_uom_qty = reserved_qty
            raw_moves._recompute_state()
        backorders.action_confirm()
        bo_to_assign.action_assign()

        # Remove the serial move line without reserved quantity. Post inventory will assigned all the non done moves
        # So those move lines are duplicated.
        backorders.move_raw_ids.move_line_ids.filtered(lambda ml: ml.product_id.tracking == 'serial' and ml.product_qty == 0).unlink()

        wo_to_cancel = self.env['mrp.workorder']
        wo_to_update = self.env['mrp.workorder']
        for old_wo, wo in zip(self.workorder_ids, backorders.workorder_ids):
            if old_wo.qty_remaining == 0:
                wo_to_cancel += wo
                continue
            if not wo_to_update or wo_to_update[-1].production_id != wo.production_id:
                wo_to_update += wo
            wo.qty_produced = max(old_wo.qty_produced - old_wo.qty_producing, 0)
            if wo.product_tracking == 'serial':
                wo.qty_producing = 1
            else:
                wo.qty_producing = wo.qty_remaining
        wo_to_cancel.action_cancel()
        for wo in wo_to_update:
            wo.state = 'ready' if wo.next_work_order_id.production_availability == 'assigned' else 'waiting'

        return backorders

    def _split_productions(self, amounts=False, cancel_remaning_qty=False, set_consumed_qty=False):
        """ Splits productions into productions smaller quantities to produce, i.e. creates
        its backorders.
        :param dict amounts: a dict with a production as key and a list value containing
        the amounts each production split should produce including the original production,
        e.g. {mrp.production(1,): [3, 2]} will result in mrp.production(1,) having a product_qty=3
        and a new backorder with product_qty=2.
        :return: mrp.production records in order of [orig_prod_1, backorder_prod_1,
        backorder_prod_2, orig_prod_2, backorder_prod_2, etc.]
        """
        def _default_amounts(production):
            return [production.qty_producing, production._get_quantity_to_backorder()]

        if not amounts:
            amounts = {}
        for production in self:
            mo_amounts = amounts.get(production)
            if not mo_amounts:
                amounts[production] = _default_amounts(production)
                continue
            total_amount = sum(mo_amounts)
            diff = float_compare(production.product_qty, total_amount, precision_rounding=production.product_uom_id.rounding)
            if diff > 0 and not cancel_remaning_qty:
                amounts[production].append(production.product_qty - total_amount)
            elif diff < 0 or production.state in ['done', 'cancel']:
                raise UserError(_("Unable to split with more than the quantity to produce."))

        backorder_vals_list = []
        initial_qty_by_production = {}

        # Create the backorders.
        for production in self:
            initial_qty_by_production[production] = production.product_qty
            if production.backorder_sequence == 0:  # Activate backorder naming
                production.backorder_sequence = 1
            production.name = self._get_name_backorder(production.name, production.backorder_sequence)
            production.product_qty = amounts[production][0]
            backorder_vals = production.copy_data(default=production._get_backorder_mo_vals())[0]
            backorder_qtys = amounts[production][1:]

            next_seq = max(production.procurement_group_id.mrp_production_ids.mapped("backorder_sequence"), default=1)

            for qty_to_backorder in backorder_qtys:
                next_seq += 1
                backorder_vals_list.append(dict(
                    backorder_vals,
                    product_qty=qty_to_backorder,
                    name=production._get_name_backorder(production.name, next_seq),
                    backorder_sequence=next_seq,
                    state='confirmed'
                ))

        backorders = self.env['mrp.production'].with_context(skip_confirm=True).create(backorder_vals_list)

        index = 0
        production_to_backorders = {}
        production_ids = OrderedSet()
        for production in self:
            number_of_backorder_created = len(amounts.get(production, _default_amounts(production))) - 1
            production_backorders = backorders[index:index + number_of_backorder_created]
            production_to_backorders[production] = production_backorders
            production_ids.update(production.ids)
            production_ids.update(production_backorders.ids)
            index += number_of_backorder_created

        # Split the `stock.move` among new backorders.
        new_moves_vals = []
        moves = []
        for production in self:
            for move in production.move_raw_ids | production.move_finished_ids:
                if move.additional:
                    continue
                unit_factor = move.product_uom_qty / initial_qty_by_production[production]
                initial_move_vals = move.copy_data(move._get_backorder_move_vals())[0]
                move.with_context(do_not_unreserve=True).product_uom_qty = production.product_qty * unit_factor

                for backorder in production_to_backorders[production]:
                    move_vals = dict(
                        initial_move_vals,
                        product_uom_qty=backorder.product_qty * unit_factor
                    )
                    if move.raw_material_production_id:
                        move_vals['raw_material_production_id'] = backorder.id
                    else:
                        move_vals['production_id'] = backorder.id
                    new_moves_vals.append(move_vals)
                    moves.append(move)

        backorder_moves = self.env['stock.move'].create(new_moves_vals)
        # Split `stock.move.line`s. 2 options for this:
        # - do_unreserve -> action_assign
        # - Split the reserved amounts manually
        # The first option would be easier to maintain since it's less code
        # However it could be slower (due to `stock.quant` update) and could
        # create inconsistencies in mass production if a new lot higher in a
        # FIFO strategy arrives between the reservation and the backorder creation
        move_to_backorder_moves = defaultdict(lambda: self.env['stock.move'])
        for move, backorder_move in zip(moves, backorder_moves):
            move_to_backorder_moves[move] |= backorder_move

        move_lines_vals = []
        assigned_moves = set()
        partially_assigned_moves = set()
        move_lines_to_unlink = set()

        for initial_move, backorder_moves in move_to_backorder_moves.items():
            ml_by_move = []
            product_uom = initial_move.product_id.uom_id
            for move_line in initial_move.move_line_ids:
                available_qty = move_line.product_uom_id._compute_quantity(move_line.product_uom_qty, product_uom)
                if float_compare(available_qty, 0, precision_rounding=move_line.product_uom_id.rounding) <= 0:
                    continue
                ml_by_move.append((available_qty, move_line, move_line.copy_data()[0]))

            initial_move.move_line_ids.with_context(bypass_reservation_update=True).write({'product_uom_qty': 0})
            moves = list(initial_move | backorder_moves)

            move = moves and moves.pop(0)
            move_qty_to_reserve = move.product_qty
            for quantity, move_line, ml_vals in ml_by_move:
                while float_compare(quantity, 0, precision_rounding=product_uom.rounding) > 0 and move:
                    # Do not create `stock.move.line` if there is no initial demand on `stock.move`
                    taken_qty = min(move_qty_to_reserve, quantity)
                    taken_qty_uom = product_uom._compute_quantity(taken_qty, move_line.product_uom_id)
                    if move == initial_move:
                        move_line.with_context(bypass_reservation_update=True).product_uom_qty = taken_qty_uom
                        if set_consumed_qty:
                            move_line.qty_done = taken_qty_uom
                    elif not float_is_zero(taken_qty_uom, precision_rounding=move_line.product_uom_id.rounding):
                        new_ml_vals = dict(
                            ml_vals,
                            product_uom_qty=taken_qty_uom,
                            move_id=move.id
                        )
                        if set_consumed_qty:
                            new_ml_vals['qty_done'] = taken_qty_uom
                        move_lines_vals.append(new_ml_vals)
                    quantity -= taken_qty
                    move_qty_to_reserve -= taken_qty

                    if float_compare(move_qty_to_reserve, 0, precision_rounding=move.product_uom.rounding) <= 0:
                        assigned_moves.add(move.id)
                        move = moves and moves.pop(0)
                        move_qty_to_reserve = move and move.product_qty or 0

                # Unreserve the quantity removed from initial `stock.move.line` and
                # not assigned to a move anymore. In case of a split smaller than initial
                # quantity and fully reserved
                if quantity and move_line.product_id.type == 'product':
                    self.env['stock.quant']._update_reserved_quantity(
                        move_line.product_id, move_line.location_id, -quantity,
                        lot_id=move_line.lot_id, package_id=move_line.package_id,
                        owner_id=move_line.owner_id, strict=True)

            if move and move_qty_to_reserve != move.product_qty:
                partially_assigned_moves.add(move.id)

            move_lines_to_unlink.update(initial_move.move_line_ids.filtered(
                lambda ml: not ml.product_uom_qty and not ml.qty_done).ids)

        self.env['stock.move'].browse(assigned_moves).write({'state': 'assigned'})
        self.env['stock.move'].browse(partially_assigned_moves).write({'state': 'partially_available'})
        # Avoid triggering a useless _recompute_state
        self.env['stock.move.line'].browse(move_lines_to_unlink).write({'move_id': False})
        self.env['stock.move.line'].browse(move_lines_to_unlink).unlink()
        self.env['stock.move.line'].create(move_lines_vals)

        # We need to adapt `duration_expected` on both the original workorders and their
        # backordered workorders. To do that, we use the original `duration_expected` and the
        # ratio of the quantity produced and the quantity to produce.
        for production in self:
            initial_qty = initial_qty_by_production[production]
            initial_workorder_remaining_qty = []
            bo = production_to_backorders[production]

            # Adapt duration
            for workorder in (production | bo).workorder_ids:
                workorder.duration_expected = workorder.duration_expected * workorder.production_id.product_qty / initial_qty

            # Adapt quantities produced
            for workorder in production.workorder_ids:
                initial_workorder_remaining_qty.append(max(workorder.qty_produced - workorder.qty_production, 0))
                workorder.qty_produced = min(workorder.qty_produced, workorder.qty_production)
            workorders_len = len(bo.workorder_ids)
            for index, workorder in enumerate(bo.workorder_ids):
                remaining_qty = initial_workorder_remaining_qty[index // workorders_len]
                if remaining_qty:
                    workorder.qty_produced = max(workorder.qty_production, remaining_qty)
                    initial_workorder_remaining_qty[index % workorders_len] = max(remaining_qty - workorder.qty_produced, 0)
        backorders._action_confirm_mo_backorders()
        return self.env['mrp.production'].browse(production_ids)

    def _action_confirm_mo_backorders(self):
        self.workorder_ids._action_confirm()

    def button_mark_done(self):
        self._button_mark_done_sanity_checks()

        if not self.env.context.get('button_mark_done_production_ids'):
            self = self.with_context(button_mark_done_production_ids=self.ids)
        res = self._pre_button_mark_done()
        if res is not True:
            return res

        if self.env.context.get('mo_ids_to_backorder'):
            productions_to_backorder = self.browse(self.env.context['mo_ids_to_backorder'])
            productions_not_to_backorder = self - productions_to_backorder
            close_mo = False
        else:
            productions_not_to_backorder = self
            productions_to_backorder = self.env['mrp.production']
            close_mo = True

        self.workorder_ids.button_finish()

        backorders = productions_to_backorder._generate_backorder_productions(close_mo=close_mo)
        productions_not_to_backorder._post_inventory(cancel_backorder=True)
        productions_to_backorder._post_inventory(cancel_backorder=True)

        # if completed products make other confirmed/partially_available moves available, assign them
        done_move_finished_ids = (productions_to_backorder.move_finished_ids | productions_not_to_backorder.move_finished_ids).filtered(lambda m: m.state == 'done')
        done_move_finished_ids._trigger_assign()

        # Moves without quantity done are not posted => set them as done instead of canceling. In
        # case the user edits the MO later on and sets some consumed quantity on those, we do not
        # want the move lines to be canceled.
        (productions_not_to_backorder.move_raw_ids | productions_not_to_backorder.move_finished_ids).filtered(lambda x: x.state not in ('done', 'cancel')).write({
            'state': 'done',
            'product_uom_qty': 0.0,
        })

        for production in self:
            production.write({
                'date_finished': fields.Datetime.now(),
                'product_qty': production.qty_produced,
                'priority': '0',
                'is_locked': True,
                'state': 'done',
            })

        if not backorders:
            if self.env.context.get('from_workorder'):
                return {
                    'type': 'ir.actions.act_window',
                    'res_model': 'mrp.production',
                    'views': [[self.env.ref('mrp.mrp_production_form_view').id, 'form']],
                    'res_id': self.id,
                    'target': 'main',
                }
            return True
        context = self.env.context.copy()
        context = {k: v for k, v in context.items() if not k.startswith('default_')}
        for k, v in context.items():
            if k.startswith('skip_'):
                context[k] = False
        action = {
            'res_model': 'mrp.production',
            'type': 'ir.actions.act_window',
            'context': dict(context, mo_ids_to_backorder=None, button_mark_done_production_ids=None)
        }
        if len(backorders) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': backorders[0].id,
            })
        else:
            action.update({
                'name': _("Backorder MO"),
                'domain': [('id', 'in', backorders.ids)],
                'view_mode': 'tree,form',
            })
        return action

    def _pre_button_mark_done(self):
        productions_to_immediate = self._check_immediate()
        if productions_to_immediate:
            return productions_to_immediate._action_generate_immediate_wizard()

        for production in self:
            if float_is_zero(production.qty_producing, precision_rounding=production.product_uom_id.rounding):
                raise UserError(_('The quantity to produce must be positive!'))
            if production.move_raw_ids and not any(production.move_raw_ids.mapped('quantity_done')):
                raise UserError(_("You must indicate a non-zero amount consumed for at least one of your components"))

        consumption_issues = self._get_consumption_issues()
        if consumption_issues:
            return self._action_generate_consumption_wizard(consumption_issues)

        quantity_issues = self._get_quantity_produced_issues()
        if quantity_issues:
            return self._action_generate_backorder_wizard(quantity_issues)
        return True

    def _button_mark_done_sanity_checks(self):
        self._check_company()
        for order in self:
            order._check_sn_uniqueness()

    def do_unreserve(self):
        self.move_raw_ids.filtered(lambda x: x.state not in ('done', 'cancel'))._do_unreserve()

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
        action = self.env["ir.actions.actions"]._for_xml_id("stock.action_stock_scrap")
        action['domain'] = [('production_id', '=', self.id)]
        action['context'] = dict(self._context, default_origin=self.name)
        return action

    @api.model
    def get_empty_list_help(self, help):
        self = self.with_context(
            empty_list_help_document_name=_("manufacturing order"),
        )
        return super(MrpProduction, self).get_empty_list_help(help)

    def _log_downside_manufactured_quantity(self, moves_modification, cancel=False):

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
                'order_exceptions': rendering_context,
                'impacted_pickings': False,
                'cancel': cancel
            }
            return self.env.ref('mrp.exception_on_mo')._render(values=values)

        documents = self.env['stock.picking']._log_activity_get_documents(moves_modification, 'move_dest_ids', 'DOWN', _keys_in_sorted, _keys_in_groupby)
        documents = self.env['stock.picking']._less_quantities_than_expected_add_documents(moves_modification, documents)
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
            return self.env.ref('mrp.exception_on_mo')._render(values=values)

        self.env['stock.picking']._log_activity(_render_note_exception_quantity_mo, documents)

    def button_unbuild(self):
        self.ensure_one()
        return {
            'name': _('Unbuild: %s', self.product_id.display_name),
            'view_mode': 'form',
            'res_model': 'mrp.unbuild',
            'view_id': self.env.ref('mrp.mrp_unbuild_form_view_simplified').id,
            'type': 'ir.actions.act_window',
            'context': {'default_product_id': self.product_id.id,
                        'default_mo_id': self.id,
                        'default_company_id': self.company_id.id,
                        'default_location_id': self.location_dest_id.id,
                        'default_location_dest_id': self.location_src_id.id,
                        'create': False, 'edit': False},
            'target': 'new',
        }

    def action_serial_mass_produce_wizard(self):
        self.ensure_one()
        self._check_company()
        if self.state != 'confirmed':
            return
        if self.product_id.tracking != 'serial':
            return
        dummy, dummy, missing_components, multiple_lot_components = self._check_serial_mass_produce_components()
        message = ""
        if missing_components:
            message += _("Make sure enough quantities of these components are reserved to carry on production:\n")
            message += "\n".join(component.name for component in missing_components)
        if multiple_lot_components:
            if message:
                message += "\n"
            message += _("Component Lots must be unique for mass production. Please review reservation for:\n")
            message += "\n".join(component.name for component in multiple_lot_components)
        if message:
            raise UserError(message)
        next_serial = self.env['stock.production.lot']._get_next_serial(self.company_id, self.product_id)
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.act_assign_serial_numbers_production")
        action['context'] = {
            'default_production_id': self.id,
            'default_expected_qty': self.product_qty,
            'default_next_serial_number': next_serial,
            'default_next_serial_count': self.product_qty - self.qty_produced,
        }
        return action

    @api.model
    def _prepare_procurement_group_vals(self, values):
        return {'name': values['name']}

    def _get_quantity_to_backorder(self):
        self.ensure_one()
        return max(self.product_qty - self.qty_producing, 0)

    def _check_sn_uniqueness(self):
        """ Alert the user if the serial number as already been consumed/produced """
        if self.product_tracking == 'serial' and self.lot_producing_id:
            if self._is_finished_sn_already_produced(self.lot_producing_id):
                raise UserError(_('This serial number for product %s has already been produced', self.product_id.name))

        for move in self.move_finished_ids:
            if move.has_tracking != 'serial' or move.product_id == self.product_id:
                continue
            for move_line in move.move_line_ids:
                if self._is_finished_sn_already_produced(move_line.lot_id, excluded_sml=move_line):
                    raise UserError(_('The serial number %(number)s used for byproduct %(product_name)s has already been produced',
                                      number=move_line.lot_id.name, product_name=move_line.product_id.name))

        for move in self.move_raw_ids:
            if move.has_tracking != 'serial':
                continue
            for move_line in move.move_line_ids:
                if float_is_zero(move_line.qty_done, precision_rounding=move_line.product_uom_id.rounding):
                    continue
                message = _('The serial number %(number)s used for component %(component)s has already been consumed',
                    number=move_line.lot_id.name,
                    component=move_line.product_id.name)
                co_prod_move_lines = self.move_raw_ids.move_line_ids

                # Check presence of same sn in previous productions
                duplicates = self.env['stock.move.line'].search_count([
                    ('lot_id', '=', move_line.lot_id.id),
                    ('qty_done', '=', 1),
                    ('state', '=', 'done'),
                    ('location_dest_id.usage', '=', 'production'),
                    ('production_id', '!=', False),
                ])
                if duplicates:
                    # Maybe some move lines have been compensated by unbuild
                    duplicates_returned = move.product_id._count_returned_sn_products(move_line.lot_id)
                    removed = self.env['stock.move.line'].search_count([
                        ('lot_id', '=', move_line.lot_id.id),
                        ('state', '=', 'done'),
                        ('location_dest_id.scrap_location', '=', True)
                    ])
                    unremoved = self.env['stock.move.line'].search_count([
                        ('lot_id', '=', move_line.lot_id.id),
                        ('state', '=', 'done'),
                        ('location_id.scrap_location', '=', True),
                        ('location_dest_id.scrap_location', '=', False),
                    ])
                    # Either removed or unbuild
                    if not ((duplicates_returned or removed) and duplicates - duplicates_returned - removed + unremoved == 0):
                        raise UserError(message)
                # Check presence of same sn in current production
                duplicates = co_prod_move_lines.filtered(lambda ml: ml.qty_done and ml.lot_id == move_line.lot_id) - move_line
                if duplicates:
                    raise UserError(message)

    def _is_finished_sn_already_produced(self, lot, excluded_sml=None):
        excluded_sml = excluded_sml or self.env['stock.move.line']
        domain = [
            ('lot_id', '=', lot.id),
            ('qty_done', '=', 1),
            ('state', '=', 'done')
        ]
        co_prod_move_lines = self.move_finished_ids.move_line_ids - excluded_sml
        domain_unbuild = domain + [
            ('production_id', '=', False),
            ('location_dest_id.usage', '=', 'production')
        ]
        # Check presence of same sn in previous productions
        duplicates = self.env['stock.move.line'].search_count(domain + [
            ('location_id.usage', '=', 'production')
        ])
        if duplicates:
            # Maybe some move lines have been compensated by unbuild
            duplicates_unbuild = self.env['stock.move.line'].search_count(domain_unbuild + [
                ('move_id.unbuild_id', '!=', False)
            ])
            removed = self.env['stock.move.line'].search_count([
                ('lot_id', '=', lot.id),
                ('state', '=', 'done'),
                ('location_dest_id.scrap_location', '=', True)
            ])
            # Either removed or unbuild
            if not ((duplicates_unbuild or removed) and duplicates - duplicates_unbuild - removed == 0):
                return True
        # Check presence of same sn in current production
        duplicates = co_prod_move_lines.filtered(lambda ml: ml.qty_done and ml.lot_id == lot)
        return bool(duplicates)

    def _check_immediate(self):
        immediate_productions = self.browse()
        if self.env.context.get('skip_immediate'):
            return immediate_productions
        pd = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        for production in self:
            if all(float_is_zero(ml.qty_done, precision_digits=pd) for
                    ml in production.move_raw_ids.move_line_ids.filtered(lambda m: m.state not in ('done', 'cancel'))
                    ) and float_is_zero(production.qty_producing, precision_digits=pd):
                immediate_productions |= production
        return immediate_productions

    def _check_serial_mass_produce_components(self):
        have_serial_components = False
        have_lot_components = False
        missing_components = set()
        multiple_lot_components = set()
        for production in self:
            have_serial_components |= any(move.product_id.tracking == 'serial' for move in production.move_raw_ids)
            have_lot_components |= any(move.product_id.tracking == 'lot' for move in production.move_raw_ids)
            missing_components.update([move.product_id for move in production.move_raw_ids if float_compare(move.reserved_availability, move.product_uom_qty, precision_rounding=move.product_uom.rounding) < 0])
            components = {}
            for move in production.move_raw_ids:
                if move.product_id.tracking != 'lot':
                    continue
                lot_ids = move.mapped('move_line_ids.lot_id.id')
                if not lot_ids:
                    continue
                components.setdefault(move.product_id, set()).update(lot_ids)
            multiple_lot_components.update([p for p, l in components.items() if len(l) != 1])
        return (have_serial_components, have_lot_components, missing_components, multiple_lot_components)

    def _generate_backorder_productions_multi(self, serial_numbers, cancel_remaining_quantities=False):
        warnings.warn("Method '_generate_backorder_productions_multi()' is deprecated, use _split_productions() instead.", DeprecationWarning)
        self._split_productions({self: [1] * len(serial_numbers)}, cancel_remaining_quantities)

```

## File: models\mrp_routing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _, tools


class MrpRoutingWorkcenter(models.Model):
    _name = 'mrp.routing.workcenter'
    _description = 'Work Center Usage'
    _order = 'bom_id, sequence, id'
    _check_company_auto = True

    name = fields.Char('Operation', required=True)
    active = fields.Boolean(default=True)
    workcenter_id = fields.Many2one('mrp.workcenter', 'Work Center', required=True, check_company=True)
    sequence = fields.Integer(
        'Sequence', default=100,
        help="Gives the sequence order when displaying a list of routing Work Centers.")
    bom_id = fields.Many2one(
        'mrp.bom', 'Bill of Material',
        index=True, ondelete='cascade', required=True, check_company=True,
        help="The Bill of Material this operation is linked to")
    company_id = fields.Many2one('res.company', 'Company', related='bom_id.company_id')
    worksheet_type = fields.Selection([
        ('pdf', 'PDF'), ('google_slide', 'Google Slide'), ('text', 'Text')],
        string="Work Sheet", default="text",
        help="Defines if you want to use a PDF or a Google Slide as work sheet."
    )
    note = fields.Html('Description', help="Text worksheet description")
    worksheet = fields.Binary('PDF')
    worksheet_google_slide = fields.Char('Google Slide', help="Paste the url of your Google Slide. Make sure the access to the document is public.")
    time_mode = fields.Selection([
        ('auto', 'Compute based on tracked time'),
        ('manual', 'Set duration manually')], string='Duration Computation',
        default='manual')
    time_mode_batch = fields.Integer('Based on', default=10)
    time_computed_on = fields.Char('Computed on last', compute='_compute_time_computed_on')
    time_cycle_manual = fields.Float(
        'Manual Duration', default=60,
        help="Time in minutes:"
        "- In manual mode, time used"
        "- In automatic mode, supposed first time when there aren't any work orders yet")
    time_cycle = fields.Float('Duration', compute="_compute_time_cycle")
    workorder_count = fields.Integer("# Work Orders", compute="_compute_workorder_count")
    workorder_ids = fields.One2many('mrp.workorder', 'operation_id', string="Work Orders")
    possible_bom_product_template_attribute_value_ids = fields.Many2many(related='bom_id.possible_product_template_attribute_value_ids')
    bom_product_template_attribute_value_ids = fields.Many2many(
        'product.template.attribute.value', string="Apply on Variants", ondelete='restrict',
        domain="[('id', 'in', possible_bom_product_template_attribute_value_ids)]",
        help="BOM Product Variants needed to apply this line.")

    @api.depends('time_mode', 'time_mode_batch')
    def _compute_time_computed_on(self):
        for operation in self:
            operation.time_computed_on = _('%i work orders') % operation.time_mode_batch if operation.time_mode != 'manual' else False

    @api.depends('time_cycle_manual', 'time_mode', 'workorder_ids')
    def _compute_time_cycle(self):
        manual_ops = self.filtered(lambda operation: operation.time_mode == 'manual')
        for operation in manual_ops:
            operation.time_cycle = operation.time_cycle_manual
        for operation in self - manual_ops:
            data = self.env['mrp.workorder'].search([
                ('operation_id', '=', operation.id),
                ('qty_produced', '>', 0),
                ('state', '=', 'done')],
                limit=operation.time_mode_batch,
                order="date_finished desc, id desc")
            # To compute the time_cycle, we can take the total duration of previous operations
            # but for the quantity, we will take in consideration the qty_produced like if the capacity was 1.
            # So producing 50 in 00:10 with capacity 2, for the time_cycle, we assume it is 25 in 00:10
            # When recomputing the expected duration, the capacity is used again to divide the qty to produce
            # so that if we need 50 with capacity 2, it will compute the expected of 25 which is 00:10
            total_duration = 0  # Can be 0 since it's not an invalid duration for BoM
            cycle_number = 0  # Never 0 unless infinite item['workcenter_id'].capacity
            for item in data:
                total_duration += item['duration']
                cycle_number += tools.float_round((item['qty_produced'] / item['workcenter_id'].capacity or 1.0), precision_digits=0, rounding_method='UP')
            if cycle_number:
                operation.time_cycle = total_duration / cycle_number
            else:
                operation.time_cycle = operation.time_cycle_manual

    def _compute_workorder_count(self):
        data = self.env['mrp.workorder'].read_group([
            ('operation_id', 'in', self.ids),
            ('state', '=', 'done')], ['operation_id'], ['operation_id'])
        count_data = dict((item['operation_id'][0], item['operation_id_count']) for item in data)
        for operation in self:
            operation.workorder_count = count_data.get(operation.id, 0)

    def copy_to_bom(self):
        if 'bom_id' in self.env.context:
            bom_id = self.env.context.get('bom_id')
            for operation in self:
                operation.copy({'bom_id': bom_id})
            return {
                'view_mode': 'form',
                'res_model': 'mrp.bom',
                'views': [(False, 'form')],
                'type': 'ir.actions.act_window',
                'res_id': bom_id,
            }

    def _skip_operation_line(self, product):
        """ Control if a operation should be processed, can be inherited to add
        custom control.
        """
        self.ensure_one()
        if product._name == 'product.template':
            return False
        return not product._match_all_variant_values(self.bom_product_template_attribute_value_ids)

    def _get_comparison_values(self):
        if not self:
            return False
        self.ensure_one()
        return tuple(self[key] for key in  ('name', 'company_id', 'workcenter_id', 'time_mode', 'time_cycle_manual', 'bom_product_template_attribute_value_ids'))

    def write(self, values):
        if 'bom_id' in values:
            for op in self:
                op.bom_id.bom_line_ids.filtered(lambda line: line.operation_id == op).operation_id = False
                op.bom_id.byproduct_ids.filtered(lambda byproduct: byproduct.operation_id == op).operation_id = False
        return super().write(values)

```

## File: models\mrp_unbuild.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_compare, float_round
from odoo.osv import expression

from collections import defaultdict


class MrpUnbuild(models.Model):
    _name = "mrp.unbuild"
    _description = "Unbuild Order"
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'id desc'

    name = fields.Char('Reference', copy=False, readonly=True, default=lambda x: _('New'))
    product_id = fields.Many2one(
        'product.product', 'Product', check_company=True,
        domain="[('type', 'in', ['product', 'consu']), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
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
        states={'done': [('readonly', True)]}, check_company=True)
    mo_id = fields.Many2one(
        'mrp.production', 'Manufacturing Order',
        domain="[('state', '=', 'done'), ('company_id', '=', company_id), ('product_id', '=?', product_id), ('bom_id', '=?', bom_id)]",
        states={'done': [('readonly', True)]}, check_company=True)
    mo_bom_id = fields.Many2one('mrp.bom', 'Bill of Material used on the Production Order', related='mo_id.bom_id')
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
        ('done', 'Done')], string='Status', default='draft')

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            warehouse = self.env['stock.warehouse'].search([('company_id', '=', self.company_id.id)], limit=1)
            if self.location_id.company_id != self.company_id:
                self.location_id = warehouse.lot_stock_id
            if self.location_dest_id.company_id != self.company_id:
                self.location_dest_id = warehouse.lot_stock_id
        else:
            self.location_id = False
            self.location_dest_id = False

    @api.onchange('mo_id')
    def _onchange_mo_id(self):
        if self.mo_id:
            self.product_id = self.mo_id.product_id.id
            self.bom_id = self.mo_id.bom_id
            self.product_uom_id = self.mo_id.product_uom_id
            if self.has_tracking == 'serial':
                self.product_qty = 1
            else:
                self.product_qty = self.mo_id.qty_produced
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
            self.bom_id = self.env['mrp.bom']._bom_find(self.product_id, company_id=self.company_id.id)[self.product_id]
            self.product_uom_id = self.mo_id.product_id == self.product_id and self.mo_id.product_uom_id.id or self.product_id.uom_id.id

    @api.constrains('product_qty')
    def _check_qty(self):
        for unbuild in self:
            if unbuild.product_qty <= 0:
                raise ValidationError(_('Unbuild Order product quantity has to be strictly positive.'))

    @api.model
    def create(self, vals):
        if not vals.get('name') or vals['name'] == _('New'):
            vals['name'] = self.env['ir.sequence'].next_by_code('mrp.unbuild') or _('New')
        return super(MrpUnbuild, self).create(vals)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_done(self):
        if 'done' in self.mapped('state'):
            raise UserError(_("You cannot delete an unbuild order if the state is 'Done'."))

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
                    'qty_done': self.product_qty,
                    'product_id': finished_move.product_id.id,
                    'product_uom_id': finished_move.product_uom.id,
                    'location_id': finished_move.location_id.id,
                    'location_dest_id': finished_move.location_dest_id.id,
                })
            else:
                finished_move.quantity_done = self.product_qty

        # TODO: Will fail if user do more than one unbuild with lot on the same MO. Need to check what other unbuild has aready took
        qty_already_used = defaultdict(float)
        for move in produce_moves | consume_moves:
            if move.has_tracking != 'none':
                original_move = move in produce_moves and self.mo_id.move_raw_ids or self.mo_id.move_finished_ids
                original_move = original_move.filtered(lambda m: m.product_id == move.product_id)
                needed_quantity = move.product_uom_qty
                moves_lines = original_move.mapped('move_line_ids')
                if move in produce_moves and self.lot_id:
                    moves_lines = moves_lines.filtered(lambda ml: self.lot_id in ml.produce_line_ids.lot_id)  # FIXME sle: double check with arm
                for move_line in moves_lines:
                    # Iterate over all move_lines until we unbuilded the correct quantity.
                    taken_quantity = min(needed_quantity, move_line.qty_done - qty_already_used[move_line])
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
                        qty_already_used[move_line] += taken_quantity
            else:
                move.quantity_done = float_round(move.product_uom_qty, precision_rounding=move.product_uom.rounding)

        finished_moves._action_done()
        consume_moves._action_done()
        produce_moves._action_done()
        produced_move_line_ids = produce_moves.mapped('move_line_ids').filtered(lambda ml: ml.qty_done > 0)
        consume_moves.mapped('move_line_ids').write({'produce_line_ids': [(6, 0, produced_move_line_ids.ids)]})
        if self.mo_id:
            unbuild_msg = _(
                "%s %s unbuilt in", self.product_qty, self.product_uom_id.name) + " <a href=# data-oe-model=mrp.unbuild data-oe-id=%d>%s</a>" % (self.id, self.display_name)
            self.mo_id.message_post(
                body=unbuild_msg,
                subtype_id=self.env.ref('mail.mt_note').id)
        return self.write({'state': 'done'})

    def _generate_consume_moves(self):
        moves = self.env['stock.move']
        for unbuild in self:
            if unbuild.mo_id:
                finished_moves = unbuild.mo_id.move_finished_ids.filtered(lambda move: move.state == 'done')
                factor = unbuild.product_qty / unbuild.mo_id.product_uom_id._compute_quantity(unbuild.mo_id.product_qty, unbuild.product_uom_id)
                for finished_move in finished_moves:
                    moves += unbuild._generate_move_from_existing_move(finished_move, factor, unbuild.location_id, finished_move.location_id)
            else:
                factor = unbuild.product_uom_id._compute_quantity(unbuild.product_qty, unbuild.bom_id.product_uom_id) / unbuild.bom_id.product_qty
                moves += unbuild._generate_move_from_bom_line(self.product_id, self.product_uom_id, unbuild.product_qty)
                for byproduct in unbuild.bom_id.byproduct_ids:
                    if byproduct._skip_byproduct_line(unbuild.product_id):
                        continue
                    quantity = byproduct.product_qty * factor
                    moves += unbuild._generate_move_from_bom_line(byproduct.product_id, byproduct.product_uom_id, quantity, byproduct_id=byproduct.id)
        return moves

    def _generate_produce_moves(self):
        moves = self.env['stock.move']
        for unbuild in self:
            if unbuild.mo_id:
                raw_moves = unbuild.mo_id.move_raw_ids.filtered(lambda move: move.state == 'done')
                factor = unbuild.product_qty / unbuild.mo_id.product_uom_id._compute_quantity(unbuild.mo_id.qty_produced, unbuild.product_uom_id)
                for raw_move in raw_moves:
                    moves += unbuild._generate_move_from_existing_move(raw_move, factor, raw_move.location_dest_id, self.location_dest_id)
            else:
                factor = unbuild.product_uom_id._compute_quantity(unbuild.product_qty, unbuild.bom_id.product_uom_id) / unbuild.bom_id.product_qty
                boms, lines = unbuild.bom_id.explode(unbuild.product_id, factor, picking_type=unbuild.bom_id.picking_type_id)
                for line, line_data in lines:
                    moves += unbuild._generate_move_from_bom_line(line.product_id, line.product_uom_id, line_data['qty'], bom_line_id=line.id)
        return moves

    def _generate_move_from_existing_move(self, move, factor, location_id, location_dest_id):
        return self.env['stock.move'].create({
            'name': self.name,
            'date': self.create_date,
            'product_id': move.product_id.id,
            'product_uom_qty': move.product_uom_qty * factor,
            'product_uom': move.product_uom.id,
            'procure_method': 'make_to_stock',
            'location_dest_id': location_dest_id.id,
            'location_id': location_id.id,
            'warehouse_id': location_dest_id.warehouse_id.id,
            'unbuild_id': self.id,
            'company_id': move.company_id.id,
            'origin_returned_move_id': move.id,
        })

    def _generate_move_from_bom_line(self, product, product_uom, quantity, bom_line_id=False, byproduct_id=False):
        product_prod_location = product.with_company(self.company_id).property_stock_production
        location_id = bom_line_id and product_prod_location or self.location_id
        location_dest_id = bom_line_id and self.location_dest_id or product_prod_location
        warehouse = location_dest_id.warehouse_id
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
        unbuild_qty = self.product_uom_id._compute_quantity(self.product_qty, self.product_id.uom_id)
        if float_compare(available_qty, unbuild_qty, precision_digits=precision) >= 0:
            return self.action_unbuild()
        else:
            return {
                'name': self.product_id.display_name + _(': Insufficient Quantity To Unbuild'),
                'view_mode': 'form',
                'res_model': 'stock.warn.insufficient.qty.unbuild',
                'view_id': self.env.ref('mrp.stock_warn_insufficient_qty_unbuild_form_view').id,
                'type': 'ir.actions.act_window',
                'context': {
                    'default_product_id': self.product_id.id,
                    'default_location_id': self.location_id.id,
                    'default_unbuild_id': self.id,
                    'default_quantity': unbuild_qty,
                    'default_product_uom_name': self.product_id.uom_name
                },
                'target': 'new'
            }

```

## File: models\mrp_workcenter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil import relativedelta
from datetime import timedelta
from functools import partial
import datetime
from pytz import timezone
from random import randint

from odoo import api, exceptions, fields, models, _
from odoo.exceptions import ValidationError
from odoo.addons.resource.models.resource import make_aware, Intervals
from odoo.tools.float_utils import float_compare


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
    note = fields.Html(
        'Description',
        help="Description of the Work Center.")
    capacity = fields.Float(
        'Capacity', default=1.0,
        help="Number of pieces (in product UoM) that can be produced in parallel (at the same time) at this work center. For example: the capacity is 5 and you need to produce 10 units, then the operation time listed on the BOM will be multiplied by two. However, note that both time before and after production will only be counted once.")
    sequence = fields.Integer(
        'Sequence', default=1, required=True,
        help="Gives the sequence order when displaying a list of work centers.")
    color = fields.Integer('Color')
    costs_hour = fields.Float(string='Cost per hour', help='Specify cost of work center per hour.', default=0.0)
    time_start = fields.Float('Setup Time', help="Time in minutes for the setup.")
    time_stop = fields.Float('Cleanup Time', help="Time in minutes for the cleaning.")
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
    tag_ids = fields.Many2many('mrp.workcenter.tag')

    @api.constrains('alternative_workcenter_ids')
    def _check_alternative_workcenter(self):
        for workcenter in self:
            if workcenter in workcenter.alternative_workcenter_ids:
                raise ValidationError(_("Workcenter %s cannot be an alternative of itself.", workcenter.name))

    @api.depends('order_ids.duration_expected', 'order_ids.workcenter_id', 'order_ids.state', 'order_ids.date_planned_start')
    def _compute_workorder_count(self):
        MrpWorkorder = self.env['mrp.workorder']
        result = {wid: {} for wid in self._ids}
        result_duration_expected = {wid: 0 for wid in self._ids}
        # Count Late Workorder
        data = MrpWorkorder.read_group(
            [('workcenter_id', 'in', self.ids), ('state', 'in', ('pending', 'waiting', 'ready')), ('date_planned_start', '<', datetime.datetime.now().strftime('%Y-%m-%d'))],
            ['workcenter_id'], ['workcenter_id'])
        count_data = dict((item['workcenter_id'][0], item['workcenter_id_count']) for item in data)
        # Count All, Pending, Ready, Progress Workorder
        res = MrpWorkorder.read_group(
            [('workcenter_id', 'in', self.ids)],
            ['workcenter_id', 'state', 'duration_expected'], ['workcenter_id', 'state'],
            lazy=False)
        for res_group in res:
            result[res_group['workcenter_id'][0]][res_group['state']] = res_group['__count']
            if res_group['state'] in ('pending', 'waiting', 'ready', 'progress'):
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

    def action_show_operations(self):
        self.ensure_one()
        action = self.env['ir.actions.actions']._for_xml_id('mrp.mrp_routing_action')
        action['domain'] = [('workcenter_id', '=', self.id)]
        action['context'] = {
            'default_workcenter_id': self.id,
        }
        return action

    def action_work_order(self):
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.action_work_orders")
        return action

    def _get_unavailability_intervals(self, start_datetime, end_datetime):
        """Get the unavailabilities intervals for the workcenters in `self`.

        Return the list of unavailabilities (a tuple of datetimes) indexed
        by workcenter id.

        :param start_datetime: filter unavailability with only slots after this start_datetime
        :param end_datetime: filter unavailability with only slots before this end_datetime
        :rtype: dict
        """
        unavailability_ressources = self.resource_id._get_unavailable_intervals(start_datetime, end_datetime)
        return {wc.id: unavailability_ressources.get(wc.resource_id.id, []) for wc in self}

    def _get_first_available_slot(self, start_datetime, duration):
        """Get the first available interval for the workcenter in `self`.

        The available interval is disjoinct with all other workorders planned on this workcenter, but
        can overlap the time-off of the related calendar (inverse of the working hours).
        Return the first available interval (start datetime, end datetime) or,
        if there is none before 700 days, a tuple error (False, 'error message').

        :param start_datetime: begin the search at this datetime
        :param duration: minutes needed to make the workorder (float)
        :rtype: tuple
        """
        self.ensure_one()
        start_datetime, revert = make_aware(start_datetime)

        resource = self.resource_id
        get_available_intervals = partial(self.resource_calendar_id._work_intervals_batch, domain=[('time_type', 'in', ['other', 'leave'])], resources=resource, tz=timezone(self.resource_calendar_id.tz))
        get_workorder_intervals = partial(self.resource_calendar_id._leave_intervals_batch, domain=[('time_type', '=', 'other')], resources=resource, tz=timezone(self.resource_calendar_id.tz))

        remaining = duration
        start_interval = start_datetime
        delta = timedelta(days=14)

        for n in range(50):  # 50 * 14 = 700 days in advance (hardcoded)
            dt = start_datetime + delta * n
            available_intervals = get_available_intervals(dt, dt + delta)[resource.id]
            workorder_intervals = get_workorder_intervals(dt, dt + delta)[resource.id]
            for start, stop, dummy in available_intervals:
                # Shouldn't loop more than 2 times because the available_intervals contains the workorder_intervals
                # And remaining == duration can only occur at the first loop and at the interval intersection (cannot happen several time because available_intervals > workorder_intervals
                for i in range(2):
                    interval_minutes = (stop - start).total_seconds() / 60
                    # If the remaining minutes has never decrease update start_interval
                    if remaining == duration:
                        start_interval = start
                    # If there is a overlap between the possible available interval and a others WO
                    if Intervals([(start_interval, start + timedelta(minutes=min(remaining, interval_minutes)), dummy)]) & workorder_intervals:
                        remaining = duration
                    elif float_compare(interval_minutes, remaining, precision_digits=3) >= 0:
                        return revert(start_interval), revert(start + timedelta(minutes=remaining))
                    else:
                        # Decrease a part of the remaining duration
                        remaining -= interval_minutes
                        # Go to the next available interval because the possible current interval duration has been used
                        break
        return False, 'Not available slot 700 days after the planned start'

    def action_archive(self):
        res = super().action_archive()
        filtered_workcenters = ", ".join(workcenter.name for workcenter in self.filtered('routing_line_ids'))
        if filtered_workcenters:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                'title': _("Note that archived work center(s): '%s' is/are still linked to active Bill of Materials, which means that operations can still be planned on it/them. "
                           "To prevent this, deletion of the work center is recommended instead.", filtered_workcenters),
                'type': 'warning',
                'sticky': True,  #True/False will display for few seconds if false
                'next': {'type': 'ir.actions.act_window_close'},
                },
            }
        return res


class WorkcenterTag(models.Model):
    _name = 'mrp.workcenter.tag'
    _description = 'Add tag for the workcenter'
    _order = 'name'

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char("Tag Name", required=True)
    color = fields.Integer("Color Index", default=_get_default_color)

    _sql_constraints = [
        ('tag_name_unique', 'unique(name)',
         'The tag name must be unique.'),
    ]


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

    production_id = fields.Many2one('mrp.production', string='Manufacturing Order', related='workorder_id.production_id', readonly=True)
    workcenter_id = fields.Many2one('mrp.workcenter', "Work Center", required=True, check_company=True, index=True)
    company_id = fields.Many2one(
        'res.company', required=True, index=True,
        default=lambda self: self._get_default_company_id())
    workorder_id = fields.Many2one('mrp.workorder', 'Work Order', check_company=True, index=True)
    user_id = fields.Many2one(
        'res.users', "User",
        default=lambda self: self.env.uid)
    loss_id = fields.Many2one(
        'mrp.workcenter.productivity.loss', "Loss Reason",
        ondelete='restrict', required=True)
    loss_type = fields.Selection(
        string="Effectiveness", related='loss_id.loss_type', store=True, readonly=False)
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

    @api.constrains('workorder_id')
    def _check_open_time_ids(self):
        for workorder in self.workorder_id:
            open_time_ids_by_user = self.env["mrp.workcenter.productivity"].read_group([("id", "in", workorder.time_ids.ids), ("date_end", "=", False)], ["user_id", "open_time_ids_count:count(id)"], ["user_id"])
            if any(data["open_time_ids_count"] > 1 for data in open_time_ids_by_user):
                raise ValidationError(_('The Workorder (%s) cannot be started twice!', workorder.display_name))

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
import json

from odoo import api, fields, models, _, SUPERUSER_ID
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_round, format_datetime


class MrpWorkorder(models.Model):
    _name = 'mrp.workorder'
    _description = 'Work Order'

    def _read_group_workcenter_id(self, workcenters, domain, order):
        workcenter_ids = self.env.context.get('default_workcenter_id')
        if not workcenter_ids:
            workcenter_ids = workcenters._search([], order=order, access_rights_uid=SUPERUSER_ID)
        return workcenters.browse(workcenter_ids)

    name = fields.Char(
        'Work Order', required=True,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]})
    workcenter_id = fields.Many2one(
        'mrp.workcenter', 'Work Center', required=True,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)], 'progress': [('readonly', True)]},
        group_expand='_read_group_workcenter_id', check_company=True)
    working_state = fields.Selection(
        string='Workcenter Status', related='workcenter_id.working_state',
        help='Technical: used in views only')
    product_id = fields.Many2one(related='production_id.product_id', readonly=True, store=True, check_company=True)
    product_tracking = fields.Selection(related="product_id.tracking")
    product_uom_id = fields.Many2one('uom.uom', 'Unit of Measure', required=True, readonly=True)
    production_id = fields.Many2one('mrp.production', 'Manufacturing Order', required=True, check_company=True, readonly=True)
    production_availability = fields.Selection(
        string='Stock Availability', readonly=True,
        related='production_id.reservation_state', store=True,
        help='Technical: used in views and domains only.')
    production_state = fields.Selection(
        string='Production State', readonly=True,
        related='production_id.state',
        help='Technical: used in views only.')
    production_bom_id = fields.Many2one('mrp.bom', related='production_id.bom_id')
    qty_production = fields.Float('Original Production Quantity', readonly=True, related='production_id.product_qty')
    company_id = fields.Many2one(related='production_id.company_id')
    qty_producing = fields.Float(
        compute='_compute_qty_producing', inverse='_set_qty_producing',
        string='Currently Produced Quantity', digits='Product Unit of Measure')
    qty_remaining = fields.Float('Quantity To Be Produced', compute='_compute_qty_remaining', digits='Product Unit of Measure')
    qty_produced = fields.Float(
        'Quantity', default=0.0,
        readonly=True,
        digits='Product Unit of Measure',
        copy=False,
        help="The number of products already handled by this work order")
    is_produced = fields.Boolean(string="Has Been Produced",
        compute='_compute_is_produced')
    state = fields.Selection([
        ('pending', 'Waiting for another WO'),
        ('waiting', 'Waiting for components'),
        ('ready', 'Ready'),
        ('progress', 'In Progress'),
        ('done', 'Finished'),
        ('cancel', 'Cancelled')], string='Status',
        compute='_compute_state', store=True,
        default='pending', copy=False, readonly=True, index=True)
    leave_id = fields.Many2one(
        'resource.calendar.leaves',
        help='Slot into workcenter calendar once planned',
        check_company=True, copy=False)
    date_planned_start = fields.Datetime(
        'Scheduled Start Date',
        compute='_compute_dates_planned',
        inverse='_set_dates_planned',
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        store=True, copy=False)
    date_planned_finished = fields.Datetime(
        'Scheduled End Date',
        compute='_compute_dates_planned',
        inverse='_set_dates_planned',
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        store=True, copy=False)
    date_start = fields.Datetime(
        'Start Date', copy=False,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]})
    date_finished = fields.Datetime(
        'End Date', copy=False,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]})

    duration_expected = fields.Float(
        'Expected Duration', digits=(16, 2), default=60.0,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help="Expected duration (in minutes)")
    duration = fields.Float(
        'Real Duration', compute='_compute_duration', inverse='_set_duration',
        readonly=False, store=True, copy=False)
    duration_unit = fields.Float(
        'Duration Per Unit', compute='_compute_duration',
        group_operator="avg", readonly=True, store=True)
    duration_percent = fields.Integer(
        'Duration Deviation (%)', compute='_compute_duration',
        group_operator="avg", readonly=True, store=True)
    progress = fields.Float('Progress Done (%)', digits=(16, 2), compute='_compute_progress')

    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Operation', check_company=True)
        # Should be used differently as BoM can change in the meantime
    worksheet = fields.Binary(
        'Worksheet', related='operation_id.worksheet', readonly=True)
    worksheet_type = fields.Selection(
        string='Worksheet Type', related='operation_id.worksheet_type', readonly=True)
    worksheet_google_slide = fields.Char(
        'Worksheet URL', related='operation_id.worksheet_google_slide', readonly=True)
    operation_note = fields.Html("Description", related='operation_id.note', readonly=True)
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
        'stock.production.lot', string='Lot/Serial Number', compute='_compute_finished_lot_id',
        inverse='_set_finished_lot_id', domain="[('product_id', '=', product_id), ('company_id', '=', company_id)]",
        check_company=True, search='_search_finished_lot_id')
    time_ids = fields.One2many(
        'mrp.workcenter.productivity', 'workorder_id', copy=False)
    is_user_working = fields.Boolean(
        'Is the Current User Working', compute='_compute_working_users',
        help="Technical field indicating whether the current user is working. ")
    working_user_ids = fields.One2many('res.users', string='Working user on this work order.', compute='_compute_working_users')
    last_working_user_id = fields.One2many('res.users', string='Last user that worked on this work order.', compute='_compute_working_users')
    costs_hour = fields.Float(
        string='Cost per hour',
        help='Technical field to store the hourly cost of workcenter at time of work order completion (i.e. to keep a consistent cost).',
        default=0.0, group_operator="avg")

    next_work_order_id = fields.Many2one('mrp.workorder', "Next Work Order", check_company=True)
    scrap_ids = fields.One2many('stock.scrap', 'workorder_id')
    scrap_count = fields.Integer(compute='_compute_scrap_move_count', string='Scrap Move')
    production_date = fields.Datetime('Production Date', related='production_id.date_planned_start', store=True)
    json_popover = fields.Char('Popover Data JSON', compute='_compute_json_popover')
    show_json_popover = fields.Boolean('Show Popover?', compute='_compute_json_popover')
    consumption = fields.Selection(related='production_id.consumption')

    @api.depends('production_availability')
    def _compute_state(self):
        # Force the flush of the production_availability, the wo state is modify in the _compute_reservation_state
        # It is a trick to force that the state of workorder is computed as the end of the
        # cyclic depends with the mo.state, mo.reservation_state and wo.state
        for workorder in self:
            if workorder.state not in ('waiting', 'ready'):
                continue
            if workorder.production_id.reservation_state not in ('waiting', 'confirmed', 'assigned'):
                continue
            if workorder.production_id.reservation_state == 'assigned' and workorder.state == 'waiting':
                workorder.state = 'ready'
            elif workorder.production_id.reservation_state != 'assigned' and workorder.state == 'ready':
                workorder.state = 'waiting'

    @api.depends('production_state', 'date_planned_start', 'date_planned_finished')
    def _compute_json_popover(self):
        previous_wo_data = self.env['mrp.workorder'].read_group(
            [('next_work_order_id', 'in', self.ids)],
            ['ids:array_agg(id)', 'date_planned_start:max', 'date_planned_finished:max'],
            ['next_work_order_id'])
        previous_wo_dict = dict([(x['next_work_order_id'][0], {
            'id': x['ids'][0],
            'date_planned_start': x['date_planned_start'],
            'date_planned_finished': x['date_planned_finished']})
            for x in previous_wo_data])
        if self.ids:
            conflicted_dict = self._get_conflicted_workorder_ids()
        for wo in self:
            infos = []
            if not wo.date_planned_start or not wo.date_planned_finished or not wo.ids:
                wo.show_json_popover = False
                wo.json_popover = False
                continue
            if wo.state in ('pending', 'waiting', 'ready'):
                previous_wo = previous_wo_dict.get(wo.id)
                prev_start = previous_wo and previous_wo['date_planned_start'] or False
                prev_finished = previous_wo and previous_wo['date_planned_finished'] or False
                if wo.state == 'pending' and prev_start and not (prev_start > wo.date_planned_start):
                    infos.append({
                        'color': 'text-primary',
                        'msg': _("Waiting the previous work order, planned from %(start)s to %(end)s",
                            start=format_datetime(self.env, prev_start, dt_format=False),
                            end=format_datetime(self.env, prev_finished, dt_format=False))
                    })
                if wo.date_planned_finished < fields.Datetime.now():
                    infos.append({
                        'color': 'text-warning',
                        'msg': _("The work order should have already been processed.")
                    })
                if prev_start and prev_start > wo.date_planned_start:
                    infos.append({
                        'color': 'text-danger',
                        'msg': _("Scheduled before the previous work order, planned from %(start)s to %(end)s",
                            start=format_datetime(self.env, prev_start, dt_format=False),
                            end=format_datetime(self.env, prev_finished, dt_format=False))
                    })
                if conflicted_dict.get(wo.id):
                    infos.append({
                        'color': 'text-danger',
                        'msg': _("Planned at the same time as other workorder(s) at %s", wo.workcenter_id.display_name)
                    })
            color_icon = infos and infos[-1]['color'] or False
            wo.show_json_popover = bool(color_icon)
            wo.json_popover = json.dumps({
                'infos': infos,
                'color': color_icon,
                'icon': 'fa-exclamation-triangle' if color_icon in ['text-warning', 'text-danger'] else 'fa-info-circle',
                'replan': color_icon not in [False, 'text-primary']
            })

    @api.depends('production_id.lot_producing_id')
    def _compute_finished_lot_id(self):
        for workorder in self:
            workorder.finished_lot_id = workorder.production_id.lot_producing_id

    def _search_finished_lot_id(self, operator, value):
        return [('production_id.lot_producing_id', operator, value)]

    def _set_finished_lot_id(self):
        for workorder in self:
            workorder.production_id.lot_producing_id = workorder.finished_lot_id

    @api.depends('production_id.qty_producing')
    def _compute_qty_producing(self):
        for workorder in self:
            workorder.qty_producing = workorder.production_id.qty_producing

    def _set_qty_producing(self):
        for workorder in self:
            if workorder.qty_producing != 0 and workorder.production_id.qty_producing != workorder.qty_producing:
                workorder.production_id.qty_producing = workorder.qty_producing
                workorder.production_id._set_qty_producing()

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
            if not self.leave_id:
                return
            raise UserError(_("It is not possible to unplan one single Work Order. "
                              "You should unplan the Manufacturing Order instead in order to unplan all the linked operations."))
        date_from = self[0].date_planned_start
        date_to = self[0].date_planned_finished
        to_write = self.env['mrp.workorder']
        for wo in self.sudo():
            if wo.leave_id:
                to_write |= wo
            else:
                wo.leave_id = wo.env['resource.calendar.leaves'].create({
                    'name': wo.display_name,
                    'calendar_id': wo.workcenter_id.resource_calendar_id.id,
                    'date_from': date_from,
                    'date_to': date_to,
                    'resource_id': wo.workcenter_id.resource_id.id,
                    'time_type': 'other',
                })
        to_write.leave_id.write({
            'date_from': date_from,
            'date_to': date_to,
        })

    def name_get(self):
        res = []
        for wo in self:
            if len(wo.production_id.workorder_ids) == 1:
                res.append((wo.id, "%s - %s - %s" % (wo.production_id.name, wo.product_id.name, wo.name)))
            else:
                res.append((wo.id, "%s - %s - %s - %s" % (wo.production_id.workorder_ids.ids.index(wo._origin.id) + 1, wo.production_id.name, wo.product_id.name, wo.name)))
        return res

    def unlink(self):
        # Removes references to workorder to avoid Validation Error
        (self.mapped('move_raw_ids') | self.mapped('move_finished_ids')).write({'workorder_id': False})
        self.mapped('leave_id').unlink()
        mo_dirty = self.production_id.filtered(lambda mo: mo.state in ("confirmed", "progress", "to_close"))

        previous_wos = self.env['mrp.workorder'].search([
            ('next_work_order_id', 'in', self.ids),
            ('id', 'not in', self.ids)
        ])
        for pw in previous_wos:
            while pw.next_work_order_id and pw.next_work_order_id in self:
                pw.next_work_order_id = pw.next_work_order_id.next_work_order_id
        res = super().unlink()
        # We need to go through `_action_confirm` for all workorders of the current productions to
        # make sure the links between them are correct (`next_work_order_id` could be obsolete now).
        mo_dirty.workorder_ids._action_confirm()
        return res

    @api.depends('production_id.product_qty', 'qty_produced', 'production_id.product_uom_id')
    def _compute_is_produced(self):
        self.is_produced = False
        for order in self.filtered(lambda p: p.production_id and p.production_id.product_uom_id):
            rounding = order.production_id.product_uom_id.rounding
            order.is_produced = float_compare(order.qty_produced, order.production_id.product_qty, precision_rounding=rounding) >= 0

    @api.depends('time_ids.duration', 'qty_produced')
    def _compute_duration(self):
        for order in self:
            order.duration = sum(order.time_ids.mapped('duration'))
            order.duration_unit = round(order.duration / max(order.qty_produced, 1), 2)  # rounding 2 because it is a time
            if order.duration_expected:
                order.duration_percent = max(-2147483648, min(2147483647, 100 * (order.duration_expected - order.duration) / order.duration_expected))
            else:
                order.duration_percent = 0

    def _set_duration(self):

        def _float_duration_to_second(duration):
            minutes = duration // 1
            seconds = (duration % 1) * 60
            return minutes * 60 + seconds

        for order in self:
            old_order_duation = sum(order.time_ids.mapped('duration'))
            new_order_duration = order.duration
            if new_order_duration == old_order_duation:
                continue

            delta_duration = new_order_duration - old_order_duation

            if delta_duration > 0:
                date_start = datetime.now() - timedelta(seconds=_float_duration_to_second(delta_duration))
                self.env['mrp.workcenter.productivity'].create(
                    order._prepare_timeline_vals(delta_duration, date_start, datetime.now())
                )
            else:
                duration_to_remove = abs(delta_duration)
                timelines = order.time_ids.sorted(lambda t: t.date_start)
                timelines_to_unlink = self.env['mrp.workcenter.productivity']
                for timeline in timelines:
                    if duration_to_remove <= 0.0:
                        break
                    if timeline.duration <= duration_to_remove:
                        duration_to_remove -= timeline.duration
                        timelines_to_unlink |= timeline
                    else:
                        new_time_line_duration = timeline.duration - duration_to_remove
                        timeline.date_start = timeline.date_end - timedelta(seconds=_float_duration_to_second(new_time_line_duration))
                        break
                timelines_to_unlink.unlink()

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

    @api.onchange('operation_id')
    def _onchange_operation_id(self):
        if self.operation_id:
            self.name = self.operation_id.name
            self.workcenter_id = self.operation_id.workcenter_id.id

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
        if self.date_planned_start and self.date_planned_finished and self.workcenter_id:
            self.duration_expected = self._calculate_duration_expected()

    def _calculate_duration_expected(self, date_planned_start=False, date_planned_finished=False):
        interval = self.workcenter_id.resource_calendar_id.get_work_duration_data(
            date_planned_start or self.date_planned_start, date_planned_finished or self.date_planned_finished,
            domain=[('time_type', 'in', ['leave', 'other'])]
        )
        return interval['hours'] * 60

    @api.onchange('operation_id', 'workcenter_id', 'qty_production')
    def _onchange_expected_duration(self):
        self.duration_expected = self._get_duration_expected()

    @api.onchange('finished_lot_id')
    def _onchange_finished_lot_id(self):
        res = self.production_id._can_produce_serial_number(sn=self.finished_lot_id)
        if res is not True:
            return res

    def write(self, values):
        if 'production_id' in values:
            raise UserError(_('You cannot link this work order to another manufacturing order.'))
        if 'workcenter_id' in values:
            for workorder in self:
                if workorder.workcenter_id.id != values['workcenter_id']:
                    if workorder.state in ('progress', 'done', 'cancel'):
                        raise UserError(_('You cannot change the workcenter of a work order that is in progress or done.'))
                    workorder.leave_id.resource_id = self.env['mrp.workcenter'].browse(values['workcenter_id']).resource_id
        if 'date_planned_start' in values or 'date_planned_finished' in values:
            for workorder in self:
                start_date = fields.Datetime.to_datetime(values.get('date_planned_start', workorder.date_planned_start))
                end_date = fields.Datetime.to_datetime(values.get('date_planned_finished', workorder.date_planned_finished))
                if start_date and end_date and start_date > end_date:
                    raise UserError(_('The planned end date of the work order cannot be prior to the planned start date, please correct this to save the work order.'))
                if 'duration_expected' not in values and not self.env.context.get('bypass_duration_calculation'):
                    if values.get('date_planned_start') and values.get('date_planned_finished'):
                        computed_finished_time = workorder._calculate_date_planned_finished(start_date)
                        values['date_planned_finished'] = computed_finished_time
                    elif start_date and end_date:
                        computed_duration = workorder._calculate_duration_expected(date_planned_start=start_date, date_planned_finished=end_date)
                        values['duration_expected'] = computed_duration
                # Update MO dates if the start date of the first WO or the
                # finished date of the last WO is update.
                if workorder == workorder.production_id.workorder_ids[0] and 'date_planned_start' in values:
                    if values['date_planned_start']:
                        workorder.production_id.with_context(force_date=True).write({
                            'date_planned_start': fields.Datetime.to_datetime(values['date_planned_start'])
                        })
                if workorder == workorder.production_id.workorder_ids[-1] and 'date_planned_finished' in values:
                    if values['date_planned_finished']:
                        workorder.production_id.with_context(force_date=True).write({
                            'date_planned_finished': fields.Datetime.to_datetime(values['date_planned_finished'])
                        })
        return super(MrpWorkorder, self).write(values)

    @api.model_create_multi
    def create(self, values):
        res = super().create(values)
        # Auto-confirm manually added workorders.
        # We need to go through `_action_confirm` for all workorders of the current productions to
        # make sure the links between them are correct.
        if self.env.context.get('skip_confirm'):
            return res
        to_confirm = res.filtered(lambda wo: wo.production_id.state in ("confirmed", "progress", "to_close"))
        to_confirm = to_confirm.production_id.workorder_ids
        to_confirm._action_confirm()
        return res

    def _action_confirm(self):
        workorders_by_production = defaultdict(lambda: self.env['mrp.workorder'])
        for workorder in self:
            workorders_by_production[workorder.production_id] |= workorder

        for production, workorders in workorders_by_production.items():
            workorders_by_bom = defaultdict(lambda: self.env['mrp.workorder'])
            bom = self.env['mrp.bom']
            moves = production.move_raw_ids | production.move_finished_ids

            for workorder in workorders:
                bom = workorder.operation_id.bom_id or workorder.production_id.bom_id
                previous_workorder = workorders_by_bom[bom][-1:]
                previous_workorder.next_work_order_id = workorder.id
                workorders_by_bom[bom] |= workorder

                moves.filtered(lambda m: m.operation_id == workorder.operation_id).write({
                    'workorder_id': workorder.id
                })

            exploded_boms, dummy = production.bom_id.explode(production.product_id, 1, picking_type=production.bom_id.picking_type_id)
            exploded_boms = {b[0]: b[1] for b in exploded_boms}
            for move in moves:
                if move.workorder_id:
                    continue
                bom = move.bom_line_id.bom_id
                while bom and bom not in workorders_by_bom:
                    bom_data = exploded_boms.get(bom, {})
                    bom = bom_data.get('parent_line') and bom_data['parent_line'].bom_id or False
                if bom in workorders_by_bom:
                    move.write({
                        'workorder_id': workorders_by_bom[bom][-1:].id
                    })
                else:
                    move.write({
                        'workorder_id': workorders_by_bom[production.bom_id][-1:].id
                    })

            for workorders in workorders_by_bom.values():
                if not workorders:
                    continue
                if workorders[0].state == 'pending':
                    workorders[0].state = 'ready' if workorders[0].production_availability == 'assigned' else 'waiting'
                for workorder in workorders:
                    workorder._start_nextworkorder()

    def _get_byproduct_move_to_update(self):
        return self.production_id.move_finished_ids.filtered(lambda x: (x.product_id.id != self.production_id.product_id.id) and (x.state not in ('done', 'cancel')))

    def _start_nextworkorder(self):
        if self.state == 'done':
            next_order = self.next_work_order_id
            while next_order and next_order.state == 'cancel':
                next_order = next_order.next_work_order_id
            if next_order.state == 'pending':
                next_order.state = 'ready' if next_order.production_availability == 'assigned' else 'waiting'

    @api.model
    def gantt_unavailability(self, start_date, end_date, scale, group_bys=None, rows=None):
        """Get unavailabilities data to display in the Gantt view."""
        workcenter_ids = set()

        def traverse_inplace(func, row, **kargs):
            res = func(row, **kargs)
            if res:
                kargs.update(res)
            for row in row.get('rows'):
                traverse_inplace(func, row, **kargs)

        def search_workcenter_ids(row):
            if row.get('groupedBy') and row.get('groupedBy')[0] == 'workcenter_id' and row.get('resId'):
                workcenter_ids.add(row.get('resId'))

        for row in rows:
            traverse_inplace(search_workcenter_ids, row)
        start_datetime = fields.Datetime.to_datetime(start_date)
        end_datetime = fields.Datetime.to_datetime(end_date)
        workcenters = self.env['mrp.workcenter'].browse(workcenter_ids)
        unavailability_mapping = workcenters._get_unavailability_intervals(start_datetime, end_datetime)

        # Only notable interval (more than one case) is send to the front-end (avoid sending useless information)
        cell_dt = (scale in ['day', 'week'] and timedelta(hours=1)) or (scale == 'month' and timedelta(days=1)) or timedelta(days=28)

        def add_unavailability(row, workcenter_id=None):
            if row.get('groupedBy') and row.get('groupedBy')[0] == 'workcenter_id' and row.get('resId'):
                workcenter_id = row.get('resId')
            if workcenter_id:
                notable_intervals = filter(lambda interval: interval[1] - interval[0] >= cell_dt, unavailability_mapping[workcenter_id])
                row['unavailabilities'] = [{'start': interval[0], 'stop': interval[1]} for interval in notable_intervals]
                return {'workcenter_id': workcenter_id}

        for row in rows:
            traverse_inplace(add_unavailability, row)
        return rows

    def button_start(self):
        self.ensure_one()
        if any(not time.date_end for time in self.time_ids.filtered(lambda t: t.user_id.id == self.env.user.id)):
            return True
        # As button_start is automatically called in the new view
        if self.state in ('done', 'cancel'):
            return True

        if self.production_id.state != 'progress':
            self.production_id.write({
                'date_start': datetime.now(),
            })

        if self.product_tracking == 'serial':
            self.qty_producing = 1.0
        elif self.qty_producing == 0:
            self.qty_producing = self.qty_remaining

        self.env['mrp.workcenter.productivity'].create(
            self._prepare_timeline_vals(self.duration, datetime.now())
        )
        if self.state == 'progress':
            return True
        start_date = datetime.now()
        vals = {
            'state': 'progress',
            'date_start': start_date,
        }
        if not self.leave_id:
            leave = self.env['resource.calendar.leaves'].create({
                'name': self.display_name,
                'calendar_id': self.workcenter_id.resource_calendar_id.id,
                'date_from': start_date,
                'date_to': start_date + relativedelta(minutes=self.duration_expected),
                'resource_id': self.workcenter_id.resource_id.id,
                'time_type': 'other'
            })
            vals['leave_id'] = leave.id
            return self.write(vals)
        else:
            if not self.date_planned_start or self.date_planned_start > start_date:
                vals['date_planned_start'] = start_date
                vals['date_planned_finished'] = self._calculate_date_planned_finished(start_date)
            if self.date_planned_finished and self.date_planned_finished < start_date:
                vals['date_planned_finished'] = start_date
            return self.with_context(bypass_duration_calculation=True).write(vals)

    def button_finish(self):
        end_date = datetime.now()
        for workorder in self:
            if workorder.state in ('done', 'cancel'):
                continue
            workorder.end_all()
            vals = {
                'qty_produced': workorder.qty_produced or workorder.qty_producing or workorder.qty_production,
                'state': 'done',
                'date_finished': end_date,
                'date_planned_finished': end_date,
                'costs_hour': workorder.workcenter_id.costs_hour
            }
            if not workorder.date_start:
                vals['date_start'] = end_date
            if not workorder.date_planned_start or end_date < workorder.date_planned_start:
                vals['date_planned_start'] = end_date
            workorder.with_context(bypass_duration_calculation=True).write(vals)

            workorder._start_nextworkorder()
        return True

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
        self.end_all()
        return self.write({'state': 'cancel'})

    def action_replan(self):
        """Replan a work order.

        It actually replans every  "ready" or "pending"
        work orders of the linked manufacturing orders.
        """
        for production in self.production_id:
            production._plan_workorders(replan=True)
        return True

    def button_done(self):
        if any(x.state in ('done', 'cancel') for x in self):
            raise UserError(_('A Manufacturing Order is already done or cancelled.'))
        self.end_all()
        end_date = datetime.now()
        return self.write({
            'state': 'done',
            'date_finished': end_date,
            'date_planned_finished': end_date,
            'costs_hour': self.workcenter_id.costs_hour
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
        action = self.env["ir.actions.actions"]._for_xml_id("stock.action_stock_scrap")
        action['domain'] = [('workorder_id', '=', self.id)]
        return action

    def action_open_wizard(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.mrp_workorder_mrp_production_form")
        action['res_id'] = self.id
        return action

    @api.depends('qty_production', 'qty_produced')
    def _compute_qty_remaining(self):
        for wo in self:
            wo.qty_remaining = max(float_round(wo.qty_production - wo.qty_produced, precision_rounding=wo.production_id.product_uom_id.rounding), 0)

    def _get_duration_expected(self, alternative_workcenter=False, ratio=1):
        self.ensure_one()
        if not self.workcenter_id:
            return self.duration_expected
        if not self.operation_id:
            duration_expected_working = (self.duration_expected - self.workcenter_id.time_start - self.workcenter_id.time_stop) * self.workcenter_id.time_efficiency / 100.0
            if duration_expected_working < 0:
                duration_expected_working = 0
            return self.workcenter_id.time_start + self.workcenter_id.time_stop + duration_expected_working * ratio * 100.0 / self.workcenter_id.time_efficiency
        qty_production = self.production_id.product_uom_id._compute_quantity(self.qty_production, self.production_id.product_id.uom_id)
        cycle_number = float_round(qty_production / self.workcenter_id.capacity, precision_digits=0, rounding_method='UP')
        if alternative_workcenter:
            # TODO : find a better alternative : the settings of workcenter can change
            duration_expected_working = (self.duration_expected - self.workcenter_id.time_start - self.workcenter_id.time_stop) * self.workcenter_id.time_efficiency / (100.0 * cycle_number)
            if duration_expected_working < 0:
                duration_expected_working = 0
            alternative_wc_cycle_nb = float_round(qty_production / alternative_workcenter.capacity, precision_digits=0, rounding_method='UP')
            return alternative_workcenter.time_start + alternative_workcenter.time_stop + alternative_wc_cycle_nb * duration_expected_working * 100.0 / alternative_workcenter.time_efficiency
        time_cycle = self.operation_id.time_cycle
        return self.workcenter_id.time_start + self.workcenter_id.time_stop + cycle_number * time_cycle * 100.0 / self.workcenter_id.time_efficiency

    def _get_conflicted_workorder_ids(self):
        """Get conlicted workorder(s) with self.

        Conflict means having two workorders in the same time in the same workcenter.

        :return: defaultdict with key as workorder id of self and value as related conflicted workorder
        """
        self.flush(['state', 'date_planned_start', 'date_planned_finished', 'workcenter_id'])
        sql = """
            SELECT wo1.id, wo2.id
            FROM mrp_workorder wo1, mrp_workorder wo2
            WHERE
                wo1.id IN %s
                AND wo1.state IN ('pending', 'waiting', 'ready')
                AND wo2.state IN ('pending', 'waiting', 'ready')
                AND wo1.id != wo2.id
                AND wo1.workcenter_id = wo2.workcenter_id
                AND (DATE_TRUNC('second', wo2.date_planned_start), DATE_TRUNC('second', wo2.date_planned_finished))
                    OVERLAPS (DATE_TRUNC('second', wo1.date_planned_start), DATE_TRUNC('second', wo1.date_planned_finished))
        """
        self.env.cr.execute(sql, [tuple(self.ids)])
        res = defaultdict(list)
        for wo1, wo2 in self.env.cr.fetchall():
            res[wo1].append(wo2)
        return res

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

    def _prepare_timeline_vals(self, duration, date_start, date_end=False):
        # Need a loss in case of the real time exceeding the expected
        if not self.duration_expected or duration < self.duration_expected:
            loss_id = self.env['mrp.workcenter.productivity.loss'].search([('loss_type', '=', 'productive')], limit=1)
            if not len(loss_id):
                raise UserError(_("You need to define at least one productivity loss in the category 'Productivity'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses."))
        else:
            loss_id = self.env['mrp.workcenter.productivity.loss'].search([('loss_type', '=', 'performance')], limit=1)
            if not len(loss_id):
                raise UserError(_("You need to define at least one productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses."))
        return {
            'workorder_id': self.id,
            'workcenter_id': self.workcenter_id.id,
            'description': _('Time Tracking: %(user)s', user=self.env.user.name),
            'loss_id': loss_id[0].id,
            'date_start': date_start,
            'date_end': date_end,
            'user_id': self.env.user.id,  # FIXME sle: can be inconsistent with company_id
            'company_id': self.company_id.id,
        }

    def _update_finished_move(self):
        """ Update the finished move & move lines in order to set the finished
        product lot on it as well as the produced quantity. This method get the
        information either from the last workorder or from the Produce wizard."""
        production_move = self.production_id.move_finished_ids.filtered(
            lambda move: move.product_id == self.product_id and
            move.state not in ('done', 'cancel')
        )
        if not production_move:
            return
        if production_move.product_id.tracking != 'none':
            if not self.finished_lot_id:
                raise UserError(_('You need to provide a lot for the finished product.'))
            move_line = production_move.move_line_ids.filtered(
                lambda line: line.lot_id.id == self.finished_lot_id.id
            )
            if move_line:
                if self.product_id.tracking == 'serial':
                    raise UserError(_('You cannot produce the same serial number twice.'))
                move_line.product_uom_qty += self.qty_producing
                move_line.qty_done += self.qty_producing
            else:
                quantity = self.product_uom_id._compute_quantity(self.qty_producing, self.product_id.uom_id, rounding_method='HALF-UP')
                putaway_location = production_move.location_dest_id._get_putaway_strategy(self.product_id, quantity)
                move_line.create({
                    'move_id': production_move.id,
                    'product_id': production_move.product_id.id,
                    'lot_id': self.finished_lot_id.id,
                    'product_uom_qty': self.qty_producing,
                    'product_uom_id': self.product_uom_id.id,
                    'qty_done': self.qty_producing,
                    'location_id': production_move.location_id.id,
                    'location_dest_id': putaway_location.id,
                })
        else:
            rounding = production_move.product_uom.rounding
            production_move._set_quantity_done(
                float_round(self.qty_producing, precision_rounding=rounding)
            )

    def _check_sn_uniqueness(self):
        # todo master: remove
        pass

    def _update_qty_producing(self, quantity):
        self.ensure_one()
        if self.qty_producing:
            self.qty_producing = quantity

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import collections
from datetime import timedelta
from itertools import groupby
import operator as py_operator
from odoo import api, fields, models, _
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
    is_kits = fields.Boolean(compute='_compute_is_kits', compute_sudo=True)

    def _compute_bom_count(self):
        for product in self:
            product.bom_count = self.env['mrp.bom'].search_count(['|', ('product_tmpl_id', '=', product.id), ('byproduct_ids.product_id.product_tmpl_id', '=', product.id)])

    @api.depends_context('company')
    def _compute_is_kits(self):
        domain = [('product_tmpl_id', 'in', self.ids), ('type', '=', 'phantom'), '|', ('company_id', '=', False), ('company_id', '=', self.env.company.id)]
        bom_mapping = self.env['mrp.bom'].search_read(domain, ['product_tmpl_id'])
        kits_ids = set(b['product_tmpl_id'][0] for b in bom_mapping)
        for template in self:
            template.is_kits = (template.id in kits_ids)

    def _compute_show_qty_status_button(self):
        super()._compute_show_qty_status_button()
        for template in self:
            if template.is_kits:
                template.show_on_hand_qty_status_button = template.product_variant_count <= 1
                template.show_forecasted_qty_status_button = False

    def _compute_used_in_bom_count(self):
        for template in self:
            template.used_in_bom_count = self.env['mrp.bom'].search_count(
                [('bom_line_ids.product_tmpl_id', '=', template.id)])

    def write(self, values):
        if 'active' in values:
            self.filtered(lambda p: p.active != values['active']).with_context(active_test=False).bom_ids.write({
                'active': values['active']
            })
        return super().write(values)

    def action_used_in_bom(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.mrp_bom_form_action")
        action['domain'] = [('bom_line_ids.product_tmpl_id', '=', self.id)]
        return action

    def _compute_mrp_product_qty(self):
        for template in self:
            template.mrp_product_qty = float_round(sum(template.mapped('product_variant_ids').mapped('mrp_product_qty')), precision_rounding=template.uom_id.rounding)

    def action_view_mos(self):
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.mrp_production_report")
        action['domain'] = [('state', '=', 'done'), ('product_tmpl_id', 'in', self.ids)]
        action['context'] = {
            'graph_measure': 'product_uom_qty',
            'search_default_filter_plan_date': 1,
        }
        return action

    def action_archive(self):
        filtered_products = self.env['mrp.bom.line'].search([('product_id', 'in', self.product_variant_ids.ids)]).product_id.mapped('display_name')
        res = super().action_archive()
        if filtered_products:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                'title': _("Note that product(s): '%s' is/are still linked to active Bill of Materials, "
                            "which means that the product can still be used on it/them.", filtered_products),
                'type': 'warning',
                'sticky': True,  #True/False will display for few seconds if false
                'next': {'type': 'ir.actions.act_window_close'},
                },
            }
        return res


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
    is_kits = fields.Boolean(compute="_compute_is_kits", compute_sudo=True)

    def _compute_bom_count(self):
        for product in self:
            product.bom_count = self.env['mrp.bom'].search_count(['|', '|', ('byproduct_ids.product_id', '=', product.id), ('product_id', '=', product.id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product.product_tmpl_id.id)])

    @api.depends_context('company')
    def _compute_is_kits(self):
        domain = ['&', '&', ('type', '=', 'phantom'),
                       '|', ('company_id', '=', False),
                            ('company_id', '=', self.env.company.id),
                       '|', ('product_id', 'in', self.ids),
                            '&', ('product_id', '=', False),
                                 ('product_tmpl_id', 'in', self.product_tmpl_id.ids)]
        bom_mapping = self.env['mrp.bom'].search_read(domain, ['product_tmpl_id', 'product_id'])
        kits_template_ids = set([])
        kits_product_ids = set([])
        for bom_data in bom_mapping:
            if bom_data['product_id']:
                kits_product_ids.add(bom_data['product_id'][0])
            else:
                kits_template_ids.add(bom_data['product_tmpl_id'][0])
        for product in self:
            product.is_kits = (product.id in kits_product_ids or product.product_tmpl_id.id in kits_template_ids)

    def _compute_show_qty_status_button(self):
        super()._compute_show_qty_status_button()
        for product in self:
            if product.is_kits:
                product.show_on_hand_qty_status_button = True
                product.show_forecasted_qty_status_button = False

    def _compute_used_in_bom_count(self):
        for product in self:
            product.used_in_bom_count = self.env['mrp.bom'].search_count([('bom_line_ids.product_id', '=', product.id)])

    def write(self, values):
        if 'active' in values:
            self.filtered(lambda p: p.active != values['active']).with_context(active_test=False).variant_bom_ids.write({
                'active': values['active']
            })
        return super().write(values)

    def get_components(self):
        """ Return the components list ids in case of kit product.
        Return the product itself otherwise"""
        self.ensure_one()
        bom_kit = self.env['mrp.bom']._bom_find(self, bom_type='phantom')[self]
        if bom_kit:
            boms, bom_sub_lines = bom_kit.explode(self, 1)
            return [bom_line.product_id.id for bom_line, data in bom_sub_lines if bom_line.product_id.type == 'product']
        else:
            return super(ProductProduct, self).get_components()

    def action_used_in_bom(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.mrp_bom_form_action")
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
        bom_kits = self.env['mrp.bom']._bom_find(self, bom_type='phantom')
        kits = self.filtered(lambda p: bom_kits.get(p))
        regular_products = self - kits
        res = (
            super(ProductProduct, regular_products)._compute_quantities_dict(lot_id, owner_id, package_id, from_date=from_date, to_date=to_date)
            if regular_products
            else {}
        )
        qties = self.env.context.get("mrp_compute_quantities", {})
        qties.update(res)
        # pre-compute bom lines and identify missing kit components to prefetch
        bom_sub_lines_per_kit = {}
        prefetch_component_ids = set()
        for product in bom_kits:
            __, bom_sub_lines = bom_kits[product].explode(product, 1)
            bom_sub_lines_per_kit[product] = bom_sub_lines
            for bom_line, __ in bom_sub_lines:
                if bom_line.product_id.id not in qties:
                    prefetch_component_ids.add(bom_line.product_id.id)
        # compute kit quantities
        for product in bom_kits:
            bom_sub_lines = bom_sub_lines_per_kit[product]
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
                component = component.with_context(mrp_compute_quantities=qties).with_prefetch(prefetch_component_ids)
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
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.product_open_bom")
        template_ids = self.mapped('product_tmpl_id').ids
        # bom specific to this variant or global to template or that contains the product as a byproduct
        action['context'] = {
            'default_product_tmpl_id': template_ids[0],
            'default_product_id': self.env.user.has_group('product.group_product_variant') and self.ids[0] or False,
        }
        action['domain'] = ['|', '|', ('byproduct_ids.product_id', 'in', self.ids), ('product_id', 'in', self.ids), '&', ('product_id', '=', False), ('product_tmpl_id', 'in', template_ids)]
        return action

    def action_view_mos(self):
        action = self.product_tmpl_id.action_view_mos()
        action['domain'] = [('state', '=', 'done'), ('product_id', 'in', self.ids)]
        return action

    def action_open_quants(self):
        bom_kits = self.env['mrp.bom']._bom_find(self, bom_type='phantom')
        components = self - self.env['product.product'].concat(*list(bom_kits.keys()))
        for product in bom_kits:
            boms, bom_sub_lines = bom_kits[product].explode(product, 1)
            components |= self.env['product.product'].concat(*[l[0].product_id for l in bom_sub_lines])
        res = super(ProductProduct, components).action_open_quants()
        if bom_kits:
            res['context']['single_product'] = False
            res['context'].pop('default_product_tmpl_id', None)
        return res

    def _match_all_variant_values(self, product_template_attribute_value_ids):
        """ It currently checks that all variant values (`product_template_attribute_value_ids`)
        are in the product (`self`).

        If multiple values are encoded for the same attribute line, only one of
        them has to be found on the variant.
        """
        self.ensure_one()
        # The intersection of the values of the product and those of the line satisfy:
        # * the number of items equals the number of attributes (since a product cannot
        #   have multiple values for the same attribute),
        # * the attributes are a subset of the attributes of the line.
        return len(self.product_template_attribute_value_ids & product_template_attribute_value_ids) == len(product_template_attribute_value_ids.attribute_id)

    def _count_returned_sn_products(self, sn_lot):
        res = self.env['stock.move.line'].search_count([
            ('lot_id', '=', sn_lot.id),
            ('qty_done', '=', 1),
            ('state', '=', 'done'),
            ('production_id', '=', False),
            ('location_id.usage', '=', 'production'),
            ('move_id.unbuild_id', '!=', False),
        ])
        return super()._count_returned_sn_products(sn_lot) + res

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

    def action_archive(self):
        filtered_products = self.env['mrp.bom.line'].search([('product_id', 'in', self.ids)]).product_id.mapped('display_name')
        res = super().action_archive()
        if filtered_products:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                'title': _("Note that product(s): '%s' is/are still linked to active Bill of Materials, "
                            "which means that the product can still be used on it/them.", filtered_products),
                'type': 'warning',
                'sticky': True,  #True/False will display for few seconds if false
                'next': {'type': 'ir.actions.act_window_close'},
                },
            }
        return res

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
    module_quality_control_worksheet = fields.Boolean("Quality Worksheet")
    module_mrp_subcontracting = fields.Boolean("Subcontracting")
    group_mrp_routings = fields.Boolean("MRP Work Orders",
        implied_group='mrp.group_mrp_routings')
    group_unlocked_by_default = fields.Boolean("Unlock Manufacturing Orders", implied_group='mrp.group_unlocked_by_default')

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
        else:
            self.module_mrp_workorder = False

    @api.onchange('group_unlocked_by_default')
    def _onchange_group_unlocked_by_default(self):
        """ When changing this setting, we want existing MOs to automatically update to match setting. """
        if self.group_unlocked_by_default:
            self.env['mrp.production'].search([('state', 'not in', ('cancel', 'done')), ('is_locked', '=', True)]).is_locked = False
        else:
            self.env['mrp.production'].search([('state', 'not in', ('cancel', 'done')), ('is_locked', '=', False)]).is_locked = True

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, Command, fields, models, _
from odoo.osv import expression
from odoo.tools import float_compare, float_round, float_is_zero, OrderedSet


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    workorder_id = fields.Many2one('mrp.workorder', 'Work Order', check_company=True)
    production_id = fields.Many2one('mrp.production', 'Production Order', check_company=True)
    description_bom_line = fields.Char(related='move_id.description_bom_line')

    @api.depends('production_id')
    def _compute_picking_type_id(self):
        line_to_remove = self.env['stock.move.line']
        for line in self:
            if not line.production_id:
                continue
            line.picking_type_id = line.production_id.picking_type_id
            line_to_remove |= line
        return super(StockMoveLine, self - line_to_remove)._compute_picking_type_id()

    def _search_picking_type_id(self, operator, value):
        res = super()._search_picking_type_id(operator=operator, value=value)
        if operator in ['not in', '!=', 'not ilike']:
            if value is False:
                return expression.OR([[('production_id.picking_type_id', operator, value)], res])
            else:
                return expression.AND([[('production_id.picking_type_id', operator, value)], res])
        else:
            if value is False:
                return expression.AND([[('production_id.picking_type_id', operator, value)], res])
            else:
                return expression.OR([[('production_id.picking_type_id', operator, value)], res])

    @api.model_create_multi
    def create(self, values):
        res = super(StockMoveLine, self).create(values)
        for line in res:
            # If the line is added in a done production, we need to map it
            # manually to the produced move lines in order to see them in the
            # traceability report
            if line.move_id.raw_material_production_id and line.state == 'done':
                mo = line.move_id.raw_material_production_id
                finished_lots = mo.lot_producing_id
                finished_lots |= mo.move_finished_ids.filtered(lambda m: m.product_id != mo.product_id).move_line_ids.lot_id
                if finished_lots:
                    produced_move_lines = mo.move_finished_ids.move_line_ids.filtered(lambda sml: sml.lot_id in finished_lots)
                    line.produce_line_ids = [(6, 0, produced_move_lines.ids)]
                else:
                    produced_move_lines = mo.move_finished_ids.move_line_ids
                    line.produce_line_ids = [(6, 0, produced_move_lines.ids)]
        return res

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
        if self.produce_line_ids.lot_id:
            ml_remaining_qty = self.qty_done - self.product_uom_qty
            ml_remaining_qty = self.product_uom_id._compute_quantity(ml_remaining_qty, self.product_id.uom_id, rounding_method="HALF-UP")
            if float_compare(ml_remaining_qty, quantity, precision_rounding=self.product_id.uom_id.rounding) < 0:
                return False
        return super(StockMoveLine, self)._reservation_is_updatable(quantity, reserved_quant)

    def write(self, vals):
        for move_line in self:
            production = move_line.move_id.production_id or move_line.move_id.raw_material_production_id
            if production and move_line.state == 'done' and any(field in vals for field in ('lot_id', 'location_id', 'qty_done')):
                move_line._log_message(production, move_line, 'mrp.track_production_move_template', vals)
        return super(StockMoveLine, self).write(vals)

    def _get_aggregated_product_quantities(self, **kwargs):
        """Returns dictionary of products and corresponding values of interest grouped by optional kit_name

        Removes descriptions where description == kit_name. kit_name is expected to be passed as a
        kwargs value because this is not directly stored in move_line_ids. Unfortunately because we
        are working with aggregated data, we have to loop through the aggregation to do this removal.

        arguments: kit_name (optional): string value of a kit name passed as a kwarg
        returns: dictionary {same_key_as_super: {same_values_as_super, ...}
        """
        aggregated_move_lines = super()._get_aggregated_product_quantities(**kwargs)
        kit_name = kwargs.get('kit_name')
        if kit_name:
            for aggregated_move_line in aggregated_move_lines:
                if aggregated_move_lines[aggregated_move_line]['description'] == kit_name:
                    aggregated_move_lines[aggregated_move_line]['description'] = ""
        return aggregated_move_lines


class StockMove(models.Model):
    _inherit = 'stock.move'

    created_production_id = fields.Many2one('mrp.production', 'Created Production Order', check_company=True, index=True)
    production_id = fields.Many2one(
        'mrp.production', 'Production Order for finished products', check_company=True, index=True)
    raw_material_production_id = fields.Many2one(
        'mrp.production', 'Production Order for components', check_company=True, index=True)
    unbuild_id = fields.Many2one(
        'mrp.unbuild', 'Disassembly Order', check_company=True)
    consume_unbuild_id = fields.Many2one(
        'mrp.unbuild', 'Consumed Disassembly Order', check_company=True)
    allowed_operation_ids = fields.One2many(
        'mrp.routing.workcenter', related='raw_material_production_id.bom_id.operation_ids')
    operation_id = fields.Many2one(
        'mrp.routing.workcenter', 'Operation To Consume', check_company=True,
        domain="[('id', 'in', allowed_operation_ids)]")
    workorder_id = fields.Many2one(
        'mrp.workorder', 'Work Order To Consume', copy=False, check_company=True)
    # Quantities to process, in normalized UoMs
    bom_line_id = fields.Many2one('mrp.bom.line', 'BoM Line', check_company=True)
    byproduct_id = fields.Many2one(
        'mrp.bom.byproduct', 'By-products', check_company=True,
        help="By-product line that generated the move in a manufacturing order")
    unit_factor = fields.Float('Unit Factor', compute='_compute_unit_factor', store=True)
    is_done = fields.Boolean(
        'Done', compute='_compute_is_done',
        store=True,
        help='Technical Field to order moves')
    order_finished_lot_ids = fields.Many2many('stock.production.lot', string="Finished Lot/Serial Number", compute='_compute_order_finished_lot_ids')
    should_consume_qty = fields.Float('Quantity To Consume', compute='_compute_should_consume_qty', digits='Product Unit of Measure')
    cost_share = fields.Float(
        "Cost Share (%)", digits=(5, 2),  # decimal = 2 is important for rounding calculations!!
        help="The percentage of the final production cost for this by-product. The total of all by-products' cost share must be smaller or equal to 100.")
    product_qty_available = fields.Float('Product On Hand Quantity', related='product_id.qty_available', depends=['product_id'])
    product_virtual_available = fields.Float('Product Forecasted Quantity', related='product_id.virtual_available', depends=['product_id'])
    description_bom_line = fields.Char('Kit', compute='_compute_description_bom_line')

    @api.depends('bom_line_id')
    def _compute_description_bom_line(self):
        bom_line_description = {}
        for bom in self.bom_line_id.bom_id:
            if bom.type != 'phantom':
                continue
            line_ids = bom.bom_line_ids.ids
            total = len(line_ids)
            name = bom.display_name
            for i, line_id in enumerate(line_ids):
                bom_line_description[line_id] = '%s - %d/%d' % (name, i+1, total)

        for move in self:
            move.description_bom_line = bom_line_description.get(move.bom_line_id.id)

    @api.depends('raw_material_production_id.priority')
    def _compute_priority(self):
        super()._compute_priority()
        for move in self:
            move.priority = move.raw_material_production_id.priority or move.priority or '0'

    @api.depends('raw_material_production_id.picking_type_id', 'production_id.picking_type_id')
    def _compute_picking_type_id(self):
        super()._compute_picking_type_id()
        for move in self:
            if move.raw_material_production_id or move.production_id:
                move.picking_type_id = (move.raw_material_production_id or move.production_id).picking_type_id

    @api.depends('raw_material_production_id.lot_producing_id')
    def _compute_order_finished_lot_ids(self):
        for move in self:
            move.order_finished_lot_ids = move.raw_material_production_id.lot_producing_id

    @api.depends('raw_material_production_id', 'production_id')
    def _compute_is_locked(self):
        super(StockMove, self)._compute_is_locked()
        for move in self:
            if move.raw_material_production_id:
                move.is_locked = move.raw_material_production_id.is_locked
            if move.production_id:
                move.is_locked = move.production_id.is_locked

    @api.depends('state')
    def _compute_is_done(self):
        for move in self:
            move.is_done = (move.state in ('done', 'cancel'))

    @api.depends('product_uom_qty',
        'raw_material_production_id', 'raw_material_production_id.product_qty', 'raw_material_production_id.qty_produced',
        'production_id', 'production_id.product_qty', 'production_id.qty_produced')
    def _compute_unit_factor(self):
        for move in self:
            mo = move.raw_material_production_id or move.production_id
            if mo:
                move.unit_factor = move.product_uom_qty / ((mo.product_qty - mo.qty_produced) or 1)
            else:
                move.unit_factor = 1.0

    @api.depends('raw_material_production_id', 'raw_material_production_id.name', 'production_id', 'production_id.name')
    def _compute_reference(self):
        moves_with_reference = self.env['stock.move']
        for move in self:
            if move.raw_material_production_id and move.raw_material_production_id.name:
                move.reference = move.raw_material_production_id.name
                moves_with_reference |= move
            if move.production_id and move.production_id.name:
                move.reference = move.production_id.name
                moves_with_reference |= move
        super(StockMove, self - moves_with_reference)._compute_reference()

    @api.depends('raw_material_production_id.qty_producing', 'product_uom_qty', 'product_uom')
    def _compute_should_consume_qty(self):
        for move in self:
            mo = move.raw_material_production_id
            if not mo or not move.product_uom:
                move.should_consume_qty = 0
                continue
            move.should_consume_qty = float_round((mo.qty_producing - mo.qty_produced) * move.unit_factor, precision_rounding=move.product_uom.rounding)

    @api.onchange('product_uom_qty')
    def _onchange_product_uom_qty(self):
        if self.raw_material_production_id and self.has_tracking == 'none':
            mo = self.raw_material_production_id
            self._update_quantity_done(mo)

    @api.model
    def default_get(self, fields_list):
        defaults = super(StockMove, self).default_get(fields_list)
        if self.env.context.get('default_raw_material_production_id') or self.env.context.get('default_production_id'):
            production_id = self.env['mrp.production'].browse(self.env.context.get('default_raw_material_production_id') or self.env.context.get('default_production_id'))
            if production_id.state not in ('draft', 'cancel'):
                if production_id.state != 'done':
                    defaults['state'] = 'draft'
                else:
                    defaults['state'] = 'done'
                    defaults['additional'] = True
                defaults['product_uom_qty'] = 0.0
            elif production_id.state == 'draft':
                defaults['group_id'] = production_id.procurement_group_id.id
                defaults['reference'] = production_id.name
        return defaults

    def write(self, vals):
        if 'product_uom_qty' in vals:
            if 'move_line_ids' in vals:
                # first update lines then product_uom_qty as the later will unreserve
                # so possibly unlink lines
                move_line_vals = vals.pop('move_line_ids')
                super().write({'move_line_ids': move_line_vals})
            procurement_requests = []
            for move in self:
                if move.raw_material_production_id.state != 'confirmed' \
                        or not float_is_zero(move.product_uom_qty, precision_rounding=move.product_uom.rounding) \
                        or move.procure_method != 'make_to_order':
                    continue
                values = move._prepare_procurement_values()
                origin = move._prepare_procurement_origin()
                procurement_requests.append(self.env['procurement.group'].Procurement(
                    move.product_id, vals['product_uom_qty'], move.product_uom,
                    move.location_id, move.rule_id and move.rule_id.name or "/",
                    origin, move.company_id, values))
            self.env['procurement.group'].run(procurement_requests)
        return super().write(vals)

    def _action_assign(self):
        res = super(StockMove, self)._action_assign()
        for move in self.filtered(lambda x: x.production_id or x.raw_material_production_id):
            if move.move_line_ids:
                move.move_line_ids.write({'production_id': move.raw_material_production_id.id,
                                               'workorder_id': move.workorder_id.id,})
        return res

    def _action_confirm(self, merge=True, merge_into=False):
        moves = self.action_explode()
        merge_into = merge_into and merge_into.action_explode()
        # we go further with the list of ids potentially changed by action_explode
        return super(StockMove, moves)._action_confirm(merge=merge, merge_into=merge_into)

    def action_explode(self):
        """ Explodes pickings """
        # in order to explode a move, we must have a picking_type_id on that move because otherwise the move
        # won't be assigned to a picking and it would be weird to explode a move into several if they aren't
        # all grouped in the same picking.
        moves_ids_to_return = OrderedSet()
        moves_ids_to_unlink = OrderedSet()
        phantom_moves_vals_list = []
        for move in self:
            if not move.picking_type_id or (move.production_id and move.production_id.product_id == move.product_id):
                moves_ids_to_return.add(move.id)
                continue
            bom = self.env['mrp.bom'].sudo()._bom_find(move.product_id, company_id=move.company_id.id, bom_type='phantom')[move.product_id]
            if not bom:
                moves_ids_to_return.add(move.id)
                continue
            if move.picking_id.immediate_transfer or float_is_zero(move.product_uom_qty, precision_rounding=move.product_uom.rounding):
                factor = move.product_uom._compute_quantity(move.quantity_done, bom.product_uom_id) / bom.product_qty
            else:
                factor = move.product_uom._compute_quantity(move.product_uom_qty, bom.product_uom_id) / bom.product_qty
            boms, lines = bom.sudo().explode(move.product_id, factor, picking_type=bom.picking_type_id)
            for bom_line, line_data in lines:
                if move.picking_id.immediate_transfer or float_is_zero(move.product_uom_qty, precision_rounding=move.product_uom.rounding):
                    phantom_moves_vals_list += move._generate_move_phantom(bom_line, 0, line_data['qty'])
                else:
                    phantom_moves_vals_list += move._generate_move_phantom(bom_line, line_data['qty'], 0)
            # delete the move with original product which is not relevant anymore
            moves_ids_to_unlink.add(move.id)

        if phantom_moves_vals_list:
            phantom_moves = self.env['stock.move'].create(phantom_moves_vals_list)
            phantom_moves._adjust_procure_method()
            moves_ids_to_return |= phantom_moves.action_explode().ids
        move_to_unlink = self.env['stock.move'].browse(moves_ids_to_unlink).sudo()
        move_to_unlink.quantity_done = 0
        move_to_unlink._action_cancel()
        move_to_unlink.unlink()
        return self.env['stock.move'].browse(moves_ids_to_return)

    def action_show_details(self):
        self.ensure_one()
        action = super().action_show_details()
        if self.raw_material_production_id:
            action['views'] = [(self.env.ref('mrp.view_stock_move_operations_raw').id, 'form')]
            action['context']['show_destination_location'] = False
            action['context']['active_mo_id'] = self.raw_material_production_id.id
        elif self.production_id:
            action['views'] = [(self.env.ref('mrp.view_stock_move_operations_finished').id, 'form')]
            action['context']['show_source_location'] = False
            action['context']['show_reserved_quantity'] = False
        return action

    def _action_cancel(self):
        res = super(StockMove, self)._action_cancel()
        mo_to_cancel = self.mapped('raw_material_production_id').filtered(lambda p: all(m.state == 'cancel' for m in p.move_raw_ids))
        if mo_to_cancel:
            mo_to_cancel._action_cancel()
        return res

    def _prepare_move_split_vals(self, qty):
        defaults = super()._prepare_move_split_vals(qty)
        defaults['workorder_id'] = False
        return defaults

    def _prepare_procurement_origin(self):
        self.ensure_one()
        if self.raw_material_production_id and self.raw_material_production_id.orderpoint_id:
            return self.origin
        return super()._prepare_procurement_origin()

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
                for v in vals:
                    v['state'] = 'assigned'
        return vals

    @api.model
    def _consuming_picking_types(self):
        res = super()._consuming_picking_types()
        res.append('mrp_operation')
        return res

    def _filter_by_product(self, prod):
        res = super()._filter_by_product(prod)
        for sm in self:
            bom = sm.bom_line_id.bom_id
            if bom.type == 'phantom' and (bom.product_id == prod or (not bom.product_id and bom.product_tmpl_id == prod.product_tmpl_id)):
                res |= sm
        return res

    def _get_backorder_move_vals(self):
        self.ensure_one()
        return {
            'state': 'confirmed',
            'reservation_date': self.reservation_date,
            'move_orig_ids': [Command.link(m.id) for m in self.mapped('move_orig_ids')],
            'move_dest_ids': [Command.link(m.id) for m in self.mapped('move_dest_ids')]
        }

    def _get_source_document(self):
        res = super()._get_source_document()
        return res or self.production_id or self.raw_material_production_id

    def _get_upstream_documents_and_responsibles(self, visited):
        if self.production_id and self.production_id.state not in ('done', 'cancel'):
            return [(self.production_id, self.production_id.user_id, visited)]
        else:
            return super(StockMove, self)._get_upstream_documents_and_responsibles(visited)

    def _delay_alert_get_documents(self):
        res = super(StockMove, self)._delay_alert_get_documents()
        productions = self.raw_material_production_id | self.production_id
        return res + list(productions)

    def _should_be_assigned(self):
        res = super(StockMove, self)._should_be_assigned()
        return bool(res and not (self.production_id or self.raw_material_production_id))

    def _should_bypass_set_qty_producing(self):
        if self.state in ('done', 'cancel'):
            return True
        # Do not update extra product quantities
        if float_is_zero(self.product_uom_qty, precision_rounding=self.product_uom.rounding):
            return True
        if self.has_tracking != 'none' or self.state == 'done':
            return True
        return False

    def _should_bypass_reservation(self, forced_location=False):
        res = super(StockMove, self)._should_bypass_reservation(
            forced_location=forced_location)
        return bool(res and not self.production_id)

    def _key_assign_picking(self):
        keys = super(StockMove, self)._key_assign_picking()
        return keys + (self.created_production_id,)

    @api.model
    def _prepare_merge_moves_distinct_fields(self):
        res = super()._prepare_merge_moves_distinct_fields()
        res += ['created_production_id', 'cost_share']
        if self.bom_line_id and ("phantom" in self.bom_line_id.bom_id.mapped('type')):
            res.append('bom_line_id')
        return res

    @api.model
    def _prepare_merge_negative_moves_excluded_distinct_fields(self):
        return super()._prepare_merge_negative_moves_excluded_distinct_fields() + ['created_production_id']

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
            # skip service since we never deliver them
            if bom_line.product_id.type == 'service':
                continue
            if float_is_zero(bom_line_data['qty'], precision_rounding=bom_line.product_uom_id.rounding):
                # As BoMs allow components with 0 qty, a.k.a. optionnal components, we simply skip those
                # to avoid a division by zero.
                continue
            bom_line_moves = self.filtered(lambda m: m.bom_line_id == bom_line)
            if bom_line_moves:
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
                qty_ratios.append(float_round(qty_processed / qty_per_kit, precision_rounding=bom_line.product_id.uom_id.rounding))
            else:
                return 0.0
        if qty_ratios:
            # Now that we have every ratio by components, we keep the lowest one to know how many kits we can produce
            # with the quantities delivered of each component. We use the floor division here because a 'partial kit'
            # doesn't make sense.
            return min(qty_ratios) // 1
        else:
            return 0.0

    def _show_details_in_draft(self):
        self.ensure_one()
        production = self.raw_material_production_id or self.production_id
        if production and (self.state != 'draft' or production.state != 'draft'):
            return True
        elif production:
            return False
        else:
            return super()._show_details_in_draft()

    def _update_quantity_done(self, mo):
        self.ensure_one()
        new_qty = float_round((mo.qty_producing - mo.qty_produced) * self.unit_factor, precision_rounding=self.product_uom.rounding)
        if not self.is_quantity_done_editable:
            self.move_line_ids.filtered(lambda ml: ml.state not in ('done', 'cancel')).qty_done = 0
            self.move_line_ids = self._set_quantity_done_prepare_vals(new_qty)
        else:
            self.quantity_done = new_qty

    def _update_candidate_moves_list(self, candidate_moves_list):
        super()._update_candidate_moves_list(candidate_moves_list)
        for production in self.mapped('raw_material_production_id'):
            candidate_moves_list.append(production.move_raw_ids.filtered(lambda m: m.product_id in self.product_id))
        for production in self.mapped('production_id'):
            candidate_moves_list.append(production.move_finished_ids.filtered(lambda m: m.product_id in self.product_id))

    def _multi_line_quantity_done_set(self, quantity_done):
        if self.raw_material_production_id:
            self.move_line_ids.filtered(lambda ml: ml.state not in ('done', 'cancel')).qty_done = 0
            self.move_line_ids = self._set_quantity_done_prepare_vals(quantity_done)
        else:
            super()._multi_line_quantity_done_set(quantity_done)

    def _prepare_procurement_values(self):
        res = super()._prepare_procurement_values()
        res['bom_line_id'] = self.bom_line_id.id
        return res

```

## File: models\stock_orderpoint.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.tools.float_utils import float_is_zero
from odoo.osv.expression import AND


class StockWarehouseOrderpoint(models.Model):
    _inherit = 'stock.warehouse.orderpoint'

    show_bom = fields.Boolean('Show BoM column', compute='_compute_show_bom')
    bom_id = fields.Many2one(
        'mrp.bom', string='Bill of Materials', check_company=True,
        domain="[('type', '=', 'normal'), '&', '|', ('company_id', '=', company_id), ('company_id', '=', False), '|', ('product_id', '=', product_id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product_tmpl_id)]")

    def _get_replenishment_order_notification(self):
        self.ensure_one()
        domain = [('orderpoint_id', 'in', self.ids)]
        if self.env.context.get('written_after'):
            domain = AND([domain, [('write_date', '>', self.env.context.get('written_after'))]])
        production = self.env['mrp.production'].search(domain, limit=1)
        if production:
            action = self.env.ref('mrp.action_mrp_production_form')
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'title': _('The following replenishment order has been generated'),
                    'message': '%s',
                    'links': [{
                        'label': production.name,
                        'url': f'#action={action.id}&id={production.id}&model=mrp.production'
                    }],
                    'sticky': False,
                }
            }
        return super()._get_replenishment_order_notification()

    @api.depends('route_id')
    def _compute_show_bom(self):
        manufacture_route = []
        for res in self.env['stock.rule'].search_read([('action', '=', 'manufacture')], ['route_id']):
            manufacture_route.append(res['route_id'][0])
        for orderpoint in self:
            orderpoint.show_bom = orderpoint.route_id.id in manufacture_route

    def _quantity_in_progress(self):
        bom_kits = self.env['mrp.bom']._bom_find(self.product_id, bom_type='phantom')
        bom_kit_orderpoints = {
            orderpoint: bom_kits[orderpoint.product_id]
            for orderpoint in self
            if orderpoint.product_id in bom_kits
        }
        orderpoints_without_kit = self - self.env['stock.warehouse.orderpoint'].concat(*bom_kit_orderpoints.keys())
        res = super(StockWarehouseOrderpoint, orderpoints_without_kit)._quantity_in_progress()
        for orderpoint in bom_kit_orderpoints:
            dummy, bom_sub_lines = bom_kit_orderpoints[orderpoint].explode(orderpoint.product_id, 1)
            ratios_qty_available = []
            # total = qty_available + in_progress
            ratios_total = []
            for bom_line, bom_line_data in bom_sub_lines:
                component = bom_line.product_id
                if component.type != 'product' or float_is_zero(bom_line_data['qty'], precision_rounding=bom_line.product_uom_id.rounding):
                    continue
                uom_qty_per_kit = bom_line_data['qty'] / bom_line_data['original_qty']
                qty_per_kit = bom_line.product_uom_id._compute_quantity(uom_qty_per_kit, bom_line.product_id.uom_id, raise_if_failure=False)
                if not qty_per_kit:
                    continue
                qty_by_product_location, dummy = component._get_quantity_in_progress(orderpoint.location_id.ids)
                qty_in_progress = qty_by_product_location.get((component.id, orderpoint.location_id.id), 0.0)
                qty_available = component.qty_available / qty_per_kit
                ratios_qty_available.append(qty_available)
                ratios_total.append(qty_available + (qty_in_progress / qty_per_kit))
            # For a kit, the quantity in progress is :
            #  (the quantity if we have received all in-progress components) - (the quantity using only available components)
            product_qty = min(ratios_total or [0]) - min(ratios_qty_available or [0])
            res[orderpoint.id] = orderpoint.product_id.uom_id._compute_quantity(product_qty, orderpoint.product_uom, round=False)

        bom_manufacture = self.env['mrp.bom']._bom_find(orderpoints_without_kit.product_id, bom_type='normal')
        bom_manufacture = self.env['mrp.bom'].concat(*bom_manufacture.values())
        productions_group = self.env['mrp.production'].read_group(
            [('bom_id', 'in', bom_manufacture.ids), ('state', '=', 'draft'), ('orderpoint_id', 'in', orderpoints_without_kit.ids)],
            ['orderpoint_id', 'product_qty', 'product_uom_id'],
            ['orderpoint_id', 'product_uom_id'], lazy=False)
        for p in productions_group:
            uom = self.env['uom.uom'].browse(p['product_uom_id'][0])
            orderpoint = self.env['stock.warehouse.orderpoint'].browse(p['orderpoint_id'][0])
            res[orderpoint.id] += uom._compute_quantity(
                p['product_qty'], orderpoint.product_uom, round=False)
        return res

    def _get_qty_multiple_to_order(self):
        """ Calculates the minimum quantity that can be ordered according to the qty and UoM of the BoM
        """
        self.ensure_one()
        qty_multiple_to_order = super()._get_qty_multiple_to_order()
        if 'manufacture' in self.rule_ids.mapped('action'):
            bom = self.env['mrp.bom']._bom_find(self.product_id, bom_type='normal')[self.product_id]
            return bom.product_uom_id._compute_quantity(bom.product_qty, self.product_uom)
        return qty_multiple_to_order

    def _set_default_route_id(self):
        route_id = self.env['stock.rule'].search([
            ('action', '=', 'manufacture')
        ]).route_id
        orderpoint_wh_bom = self.filtered(lambda o: o.product_id.bom_ids)
        if route_id and orderpoint_wh_bom:
            orderpoint_wh_bom.route_id = route_id[0].id
        return super()._set_default_route_id()

    def _prepare_procurement_values(self, date=False, group=False):
        values = super()._prepare_procurement_values(date=date, group=group)
        values['bom_id'] = self.bom_id
        return values

    def _post_process_scheduler(self):
        """ Confirm the productions only after all the orderpoints have run their
        procurement to avoid the new procurement created from the production conflict
        with them. """
        self.env['mrp.production'].sudo().search([
            ('orderpoint_id', 'in', self.ids),
            ('move_raw_ids', '!=', False),
            ('state', '=', 'draft'),
        ]).action_confirm()
        return super()._post_process_scheduler()

    def _product_exclude_list(self):
        # don't create an order point for kit products
        boms = self.env['mrp.bom'].search([('type', '=', 'phantom')])
        variant_boms = boms.filtered(lambda x: x.product_id)
        return super()._product_exclude_list() + variant_boms.product_id.ids + (boms - variant_boms).product_tmpl_id.product_variant_ids.ids

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockPickingType(models.Model):
    _inherit = 'stock.picking.type'

    code = fields.Selection(selection_add=[
        ('mrp_operation', 'Manufacturing')
    ], ondelete={'mrp_operation': 'cascade'})
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
            'count_mo_todo': ['|', ('state', 'in', ('confirmed', 'draft', 'progress', 'to_close')), ('is_planned', '=', True)],
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
        action = self.env["ir.actions.actions"]._for_xml_id('mrp.mrp_production_action_picking_deshboard')
        if self:
            action['display_name'] = self.display_name
        return action

    @api.onchange('code')
    def _onchange_code(self):
        if self.code == 'mrp_operation':
            self.use_create_lots = True
            self.use_existing_lots = True

class StockPicking(models.Model):
    _inherit = 'stock.picking'

    has_kits = fields.Boolean(compute='_compute_has_kits')

    @api.depends('move_lines')
    def _compute_has_kits(self):
        for picking in self:
            picking.has_kits = any(picking.move_lines.mapped('bom_line_id'))

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

## File: models\stock_quant.py

```python
from odoo import models, _
from odoo.exceptions import RedirectWarning


class StockQuant(models.Model):
    _inherit = 'stock.quant'

    def action_apply_inventory(self):
        if self.sudo().product_id.filtered("is_kits"):
            raise RedirectWarning(
                _('You should update the components quantity instead of directly updating the quantity of the kit product.'),
                self.env.ref('stock.action_view_inventory_tree').id,
                _("Return to Inventory"),
            )
        return super().action_apply_inventory()

```

## File: models\stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.osv import expression
from odoo.addons.stock.models.stock_rule import ProcurementException
from odoo.tools import float_compare, OrderedSet


class StockRule(models.Model):
    _inherit = 'stock.rule'
    action = fields.Selection(selection_add=[
        ('manufacture', 'Manufacture')
    ], ondelete={'manufacture': 'cascade'})

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

    @api.depends('action')
    def _compute_picking_type_code_domain(self):
        remaining = self.browse()
        for rule in self:
            if rule.action == 'manufacture':
                rule.picking_type_code_domain = 'mrp_operation'
            else:
                remaining |= rule
        super(StockRule, remaining)._compute_picking_type_code_domain()

    def _should_auto_confirm_procurement_mo(self, p):
        return (not p.orderpoint_id and p.move_raw_ids) or (p.move_dest_ids.procure_method != 'make_to_order' and not p.move_raw_ids and not p.workorder_ids)

    @api.model
    def _run_manufacture(self, procurements):
        productions_values_by_company = defaultdict(list)
        errors = []
        for procurement, rule in procurements:
            if float_compare(procurement.product_qty, 0, precision_rounding=procurement.product_uom.rounding) <= 0:
                # If procurement contains negative quantity, don't create a MO that would be for a negative value.
                continue
            bom = rule._get_matching_bom(procurement.product_id, procurement.company_id, procurement.values)

            productions_values_by_company[procurement.company_id.id].append(rule._prepare_mo_vals(*procurement, bom))

        if errors:
            raise ProcurementException(errors)

        for company_id, productions_values in productions_values_by_company.items():
            # create the MO as SUPERUSER because the current user may not have the rights to do it (mto product launched by a sale for example)
            productions = self.env['mrp.production'].with_user(SUPERUSER_ID).sudo().with_company(company_id).create(productions_values)
            self.env['stock.move'].sudo().create(productions._get_moves_raw_values())
            self.env['stock.move'].sudo().create(productions._get_moves_finished_values())
            productions._create_workorder()
            productions.filtered(self._should_auto_confirm_procurement_mo).action_confirm()

            for production in productions:
                origin_production = production.move_dest_ids and production.move_dest_ids[0].raw_material_production_id or False
                orderpoint = production.orderpoint_id
                if orderpoint and orderpoint.create_uid.id == SUPERUSER_ID and orderpoint.trigger == 'manual':
                    production.message_post(
                        body=_('This production order has been created from Replenishment Report.'),
                        message_type='comment',
                        subtype_xmlid='mail.mt_note')
                elif orderpoint:
                    production.message_post_with_view(
                        'mail.message_origin_link',
                        values={'self': production, 'origin': orderpoint},
                        subtype_id=self.env.ref('mail.mt_note').id)
                elif origin_production:
                    production.message_post_with_view(
                        'mail.message_origin_link',
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
                warehouse_id = rule.location_id.warehouse_id
            manu_rule = rule.route_id.rule_ids.filtered(lambda r: r.action == 'manufacture' and r.warehouse_id == warehouse_id)
            if warehouse_id.manufacture_steps != 'pbm_sam' or not manu_rule:
                continue
            if rule.picking_type_id == warehouse_id.sam_type_id or (
                warehouse_id.sam_loc_id and warehouse_id.sam_loc_id.parent_path in rule.location_src_id.parent_path
            ):
                if float_compare(procurement.product_qty, 0, precision_rounding=procurement.product_uom.rounding) < 0:
                    procurement.values['group_id'] = procurement.values['group_id'].stock_move_ids.filtered(
                        lambda m: m.state not in ['done', 'cancel']).move_orig_ids.group_id[:1]
                    continue
                manu_type_id = manu_rule[0].picking_type_id
                if manu_type_id:
                    name = manu_type_id.sequence_id.next_by_id()
                else:
                    name = self.env['ir.sequence'].next_by_code('mrp.production') or _('New')
                # Create now the procurement group that will be assigned to the new MO
                # This ensure that the outgoing move PostProduction -> Stock is linked to its MO
                # rather than the original record (MO or SO)
                group = procurement.values.get('group_id')
                if group:
                    procurement.values['group_id'] = group.copy({'name': name})
                else:
                    procurement.values['group_id'] = self.env["procurement.group"].create({'name': name})
        return super()._run_pull(procurements)

    def _get_custom_move_fields(self):
        fields = super(StockRule, self)._get_custom_move_fields()
        fields += ['bom_line_id']
        return fields

    def _get_matching_bom(self, product_id, company_id, values):
        if values.get('bom_id', False):
            return values['bom_id']
        if values.get('orderpoint_id', False) and values['orderpoint_id'].bom_id:
            return values['orderpoint_id'].bom_id
        return self.env['mrp.bom']._bom_find(product_id, picking_type=self.picking_type_id, bom_type='normal', company_id=company_id.id)[product_id]

    def _prepare_mo_vals(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom):
        date_planned = self._get_date_planned(product_id, company_id, values)
        date_deadline = values.get('date_deadline') or date_planned + relativedelta(days=company_id.manufacturing_lead) + relativedelta(days=product_id.produce_delay)
        mo_values = {
            'origin': origin,
            'product_id': product_id.id,
            'product_description_variants': values.get('product_description_variants'),
            'product_qty': product_qty,
            'product_uom_id': product_uom.id,
            'location_src_id': self.location_src_id.id or self.picking_type_id.default_location_src_id.id or location_id.id,
            'location_dest_id': location_id.id,
            'bom_id': bom.id,
            'date_deadline': date_deadline,
            'date_planned_start': date_planned,
            'date_planned_finished': fields.Datetime.from_string(values['date_planned']),
            'procurement_group_id': False,
            'propagate_cancel': self.propagate_cancel,
            'orderpoint_id': values.get('orderpoint_id', False) and values.get('orderpoint_id').id,
            'picking_type_id': self.picking_type_id.id or values['warehouse_id'].manu_type_id.id,
            'company_id': company_id.id,
            'move_dest_ids': values.get('move_dest_ids') and [(4, x.id) for x in values['move_dest_ids']] or False,
            'user_id': False,
        }
        # Use the procurement group created in _run_pull mrp override
        # Preserve the origin from the original stock move, if available
        if location_id.warehouse_id.manufacture_steps == 'pbm_sam' and values.get('move_dest_ids') and values.get('group_id') and values['move_dest_ids'][0].origin != values['group_id'].name:
            origin = values['move_dest_ids'][0].origin
            mo_values.update({
                'name': values['group_id'].name,
                'procurement_group_id': values['group_id'].id,
                'origin': origin,
            })
        return mo_values

    def _get_date_planned(self, product_id, company_id, values):
        format_date_planned = fields.Datetime.from_string(values['date_planned'])
        date_planned = format_date_planned - relativedelta(days=product_id.produce_delay)
        date_planned = date_planned - relativedelta(days=company_id.manufacturing_lead)
        if date_planned == format_date_planned:
            date_planned = date_planned - relativedelta(hours=1)
        return date_planned

    def _get_lead_days(self, product, **values):
        """Add the product and company manufacture delay to the cumulative delay
        and cumulative description.
        """
        delay, delay_description = super()._get_lead_days(product, **values)
        bypass_delay_description = self.env.context.get('bypass_delay_description')
        manufacture_rule = self.filtered(lambda r: r.action == 'manufacture')
        if not manufacture_rule:
            return delay, delay_description
        manufacture_rule.ensure_one()
        manufacture_delay = product.produce_delay
        delay += manufacture_delay
        if not bypass_delay_description:
            delay_description.append((_('Manufacturing Lead Time'), _('+ %d day(s)', manufacture_delay)))
        security_delay = manufacture_rule.picking_type_id.company_id.manufacturing_lead
        delay += security_delay
        if not bypass_delay_description:
            delay_description.append((_('Manufacture Security Lead Time'), _('+ %d day(s)', security_delay)))
        return delay, delay_description

    def _push_prepare_move_copy_values(self, move_to_copy, new_date):
        new_move_vals = super(StockRule, self)._push_prepare_move_copy_values(move_to_copy, new_date)
        new_move_vals['production_id'] = False
        return new_move_vals


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    mrp_production_ids = fields.One2many('mrp.production', 'procurement_group_id')

    @api.model
    def run(self, procurements, raise_user_error=True):
        """ If 'run' is called on a kit, this override is made in order to call
        the original 'run' method with the values of the components of that kit.
        """
        procurements_without_kit = []
        product_by_company = defaultdict(OrderedSet)
        for procurement in procurements:
            product_by_company[procurement.company_id].add(procurement.product_id.id)
        kits_by_company = {
            company: self.env['mrp.bom']._bom_find(self.env['product.product'].browse(product_ids), company_id=company.id, bom_type='phantom')
            for company, product_ids in product_by_company.items()
        }
        for procurement in procurements:
            bom_kit = kits_by_company[procurement.company_id].get(procurement.product_id)
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
        return super(ProcurementGroup, self).run(procurements_without_kit, raise_user_error=raise_user_error)

    def _get_moves_to_assign_domain(self, company_id):
        domain = super(ProcurementGroup, self)._get_moves_to_assign_domain(company_id)
        domain = expression.AND([domain, [('production_id', '=', False)]])
        return domain

```

## File: models\stock_scrap.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


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

    @api.onchange('lot_id')
    def _onchange_serial_number(self):
        if self.product_id.tracking == 'serial' and self.lot_id:
            if self.production_id:
                message, recommended_location = self.env['stock.quant']._check_serial_number(self.product_id,
                                                                                             self.lot_id,
                                                                                             self.company_id,
                                                                                             self.location_id,
                                                                                             self.production_id.location_dest_id)
                if message:
                    if recommended_location:
                        self.location_id = recommended_location
                    return {'warning': {'title': _('Warning'), 'message': message}}
            else:
                return super()._onchange_serial_number()

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
            result[warehouse.id].update(warehouse._get_receive_rules_dict())
        return result

    @api.model
    def _get_production_location(self):
        location = self.env['stock.location'].search([('usage', '=', 'production'), ('company_id', '=', self.company_id.id)], limit=1)
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
        routes.update(self._get_receive_routes_values('manufacture_to_resupply'))
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
        def_values = self.default_get(['company_id', 'manufacture_steps'])
        manufacture_steps = vals.get('manufacture_steps', def_values['manufacture_steps'])
        code = vals.get('code') or code or ''
        code = code.replace(' ', '').upper()
        company_id = vals.get('company_id', def_values['company_id'])
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
            'pbm_type_id': {
                'active': self.manufacture_to_resupply and self.manufacture_steps in ('pbm', 'pbm_sam') and self.active,
                'barcode': self.code.replace(" ", "").upper() + "-PC",
            },
            'sam_type_id': {
                'active': self.manufacture_to_resupply and self.manufacture_steps == 'pbm_sam' and self.active,
                'barcode': self.code.replace(" ", "").upper() + "-SFP",
            },
            'manu_type_id': {
                'active': self.manufacture_to_resupply and self.active,
                'default_location_src_id': self.manufacture_steps in ('pbm', 'pbm_sam') and self.pbm_loc_id.id or self.lot_stock_id.id,
                'default_location_dest_id': self.manufacture_steps == 'pbm_sam' and self.sam_loc_id.id or self.lot_stock_id.id,
            },
        })
        return data

    def _create_missing_locations(self, vals):
        super()._create_missing_locations(vals)
        for company_id in self.company_id:
            location = self.env['stock.location'].search([('usage', '=', 'production'), ('company_id', '=', company_id.id)], limit=1)
            if not location:
                company_id._create_production_location()

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
from . import stock_orderpoint
from . import stock_picking
from . import stock_production_lot
from . import stock_rule
from . import stock_scrap
from . import stock_warehouse
from . import stock_quant

```

## File: populate\mrp.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from datetime import datetime, timedelta
from collections import defaultdict

from odoo import models
from odoo.tools import populate, OrderedSet
from odoo.addons.stock.populate.stock import COMPANY_NB_WITH_STOCK

_logger = logging.getLogger(__name__)


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _populate_factories(self):
        return super()._populate_factories() + [
            ('manufacturing_lead', populate.randint(0, 2)),
        ]


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _populate_factories(self):
        return super()._populate_factories() + [
            ('produce_delay', populate.randint(1, 4)),
        ]


class Warehouse(models.Model):
    _inherit = 'stock.warehouse'

    def _populate_factories(self):
        return super()._populate_factories() + [
            ('manufacture_steps', populate.iterate(['mrp_one_step', 'pbm', 'pbm_sam'], [0.6, 0.2, 0.2]))
        ]


# TODO : stock picking type manufacturing


class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    _populate_sizes = {'small': 100, 'medium': 2_000, 'large': 20_000}
    _populate_dependencies = ['product.product', 'stock.location']

    def _populate_factories(self):
        company_ids = self.env.registry.populated_models['res.company'][:COMPANY_NB_WITH_STOCK]

        product_tmpl_ids = self.env['product.product'].search([
            ('id', 'in', self.env.registry.populated_models['product.product']),
            ('type', 'in', ('product', 'consu'))
        ]).product_tmpl_id.ids
        # Use only a 80 % subset of the products - the 20 % remaining will leaves of the bom tree
        random = populate.Random('subset_product_bom')
        product_tmpl_ids = random.sample(product_tmpl_ids, int(len(product_tmpl_ids) * 0.8))

        def get_product_id(values=None, random=None, **kwargs):
            if random.random() > 0.5:  # 50 % change to target specific product.product
                return False
            return random.choice(self.env['product.template'].browse(values['product_tmpl_id']).product_variant_ids.ids)

        return [
            ('company_id', populate.randomize(
                [False] + company_ids,
                [0.9] + [0.1 / (len(company_ids) or 1.0)] * (len(company_ids))  # TODO: Inverse the weight, but need to make the bom tree by company (in bom line populate)
            )),
            ('product_tmpl_id', populate.randomize(product_tmpl_ids)),
            ('product_id', populate.compute(get_product_id)),
            ('product_qty', populate.randint(1, 5)),
            ('sequence', populate.randint(1, 1000)),
            ('code', populate.constant("R{counter}")),
            ('ready_to_produce', populate.randomize(['all_available', 'asap'])),
        ]


class MrpBomLine(models.Model):
    _inherit = 'mrp.bom.line'

    _populate_sizes = {'small': 500, 'medium': 10_000, 'large': 100_000}
    _populate_dependencies = ['mrp.bom']

    def _populate_factories(self):
        # TODO: tree of product by company to be more closer to the reality
        boms = self.env['mrp.bom'].search([('id', 'in', self.env.registry.populated_models['mrp.bom'])], order='sequence, product_id, id')

        product_manu_ids = OrderedSet()
        for bom in boms:
            if bom.product_id:
                product_manu_ids.add(bom.product_id.id)
            else:
                for product_id in bom.product_tmpl_id.product_variant_ids:
                    product_manu_ids.add(product_id.id)
        product_manu_ids = list(product_manu_ids)
        product_manu = self.env['product.product'].browse(product_manu_ids)
        # product_no_manu is products which don't have any bom (leaves in the BoM trees)
        product_no_manu = self.env['product.product'].browse(self.env.registry.populated_models['product.product']) - product_manu
        product_no_manu_ids = product_no_manu.ids

        def get_product_id(values, counter, random):
            bom = self.env['mrp.bom'].browse(values['bom_id'])
            last_product_bom = bom.product_id if bom.product_id else bom.product_tmpl_id.product_variant_ids[-1]
            # TODO: index in list is in O(n) can be avoid by a cache dict (if performance issue)
            index_prod = product_manu_ids.index(last_product_bom.id)
            # Always choose a product futher in the recordset `product_manu` to avoid any loops
            # Or used a product in the `product_no_manu`

            sparsity = 0.4  # Increase the sparsity will decrease the density of the BoM trees => smaller Tree

            len_remaining_manu = len(product_manu_ids) - index_prod - 1
            len_no_manu = len(product_no_manu_ids)
            threshold = len_remaining_manu / (len_remaining_manu + sparsity * len_no_manu)
            if random.random() <= threshold:
                # TODO: avoid copy the list (if performance issue)
                return random.choice(product_manu_ids[index_prod+1:])
            else:
                return random.choice(product_no_manu_ids)

        def get_product_uom_id(values, counter, random):
            return self.env['product.product'].browse(values['product_id']).uom_id.id

        return [
            ('bom_id', populate.iterate(boms.ids)),
            ('sequence', populate.randint(1, 1000)),
            ('product_id', populate.compute(get_product_id)),
            ('product_uom_id', populate.compute(get_product_uom_id)),
            ('product_qty', populate.randint(1, 10)),
        ]


class MrpWorkcenter(models.Model):
    _inherit = 'mrp.workcenter'

    _populate_sizes = {'small': 20, 'medium': 100, 'large': 1_000}

    def _populate(self, size):
        res = super()._populate(size)

        # Set alternative workcenters
        _logger.info("Set alternative workcenters")
        # Valid workcenters by company_id (the workcenter without company can be the alternative of all workcenter)
        workcenters_by_company = defaultdict(OrderedSet)
        for workcenter in res:
            workcenters_by_company[workcenter.company_id.id].add(workcenter.id)
        workcenters_by_company = {company_id: self.env['mrp.workcenter'].browse(workcenters) for company_id, workcenters in workcenters_by_company.items()}
        workcenters_by_company = {
            company_id: workcenters | workcenters_by_company.get(False, self.env['mrp.workcenter'])
            for company_id, workcenters in workcenters_by_company.items()}

        random = populate.Random('set_alternative_workcenter')
        for workcenter in res:
            nb_alternative = max(random.randint(0, 3), len(workcenters_by_company[workcenter.company_id.id]) - 1)
            if nb_alternative > 0:
                alternatives_workcenter_ids = random.sample((workcenters_by_company[workcenter.company_id.id] - workcenter).ids, nb_alternative)
                workcenter.alternative_workcenter_ids = [(6, 0, alternatives_workcenter_ids)]

        return res

    def _populate_factories(self):
        company_ids = self.env.registry.populated_models['res.company'][:COMPANY_NB_WITH_STOCK]
        resource_calendar_no_company = self.env.ref('resource.resource_calendar_std').copy({'company_id': False})

        def get_resource_calendar_id(values, counter, random):
            if not values['company_id']:
                return resource_calendar_no_company.id
            return self.env['res.company'].browse(values['company_id']).resource_calendar_id.id

        return [
            ('name', populate.constant("Workcenter - {counter}")),
            ('company_id', populate.iterate(company_ids + [False])),
            ('resource_calendar_id', populate.compute(get_resource_calendar_id)),
            ('active', populate.iterate([True, False], [0.9, 0.1])),
            ('code', populate.constant("W/{counter}")),
            ('capacity', populate.iterate([0.5, 1.0, 2.0, 5.0], [0.2, 0.4, 0.2, 0.2])),
            ('sequence', populate.randint(1, 1000)),
            ('color', populate.randint(1, 12)),
            ('costs_hour', populate.randint(5, 25)),
            ('time_start', populate.iterate([0.0, 2.0, 10.0], [0.6, 0.2, 0.2])),
            ('time_stop', populate.iterate([0.0, 2.0, 10.0], [0.6, 0.2, 0.2])),
            ('oee_target', populate.randint(80, 99)),
        ]


class MrpRoutingWorkcenter(models.Model):
    _inherit = 'mrp.routing.workcenter'

    _populate_sizes = {'small': 500, 'medium': 5_000, 'large': 50_000}
    _populate_dependencies = ['mrp.workcenter', 'mrp.bom']

    def _populate_factories(self):
        # Take a subset (70%) of bom to have some of then without any operation
        random = populate.Random('operation_subset_bom')
        boms_ids = self.env.registry.populated_models['mrp.bom']
        boms_ids = random.sample(boms_ids, int(len(boms_ids) * 0.7))

        # Valid workcenters by company_id (the workcenter without company can be used by any operation)
        workcenters_by_company = defaultdict(OrderedSet)
        for workcenter in self.env['mrp.workcenter'].browse(self.env.registry.populated_models['mrp.workcenter']):
            workcenters_by_company[workcenter.company_id.id].add(workcenter.id)
        workcenters_by_company = {company_id: self.env['mrp.workcenter'].browse(workcenters) for company_id, workcenters in workcenters_by_company.items()}
        workcenters_by_company = {
            company_id: workcenters | workcenters_by_company.get(False, self.env['mrp.workcenter'])
            for company_id, workcenters in workcenters_by_company.items()}

        def get_company_id(values, counter, random):
            bom = self.env['mrp.bom'].browse(values['bom_id'])
            return bom.company_id.id

        def get_workcenter_id(values, counter, random):
            return random.choice(workcenters_by_company[values['company_id']]).id

        return [
            ('bom_id', populate.iterate(boms_ids)),
            ('company_id', populate.compute(get_company_id)),
            ('workcenter_id', populate.compute(get_workcenter_id)),
            ('name', populate.constant("OP-{counter}")),
            ('sequence', populate.randint(1, 1000)),
            ('time_mode', populate.iterate(['auto', 'manual'])),
            ('time_mode_batch', populate.randint(1, 100)),
            ('time_cycle_manual', populate.randomize([1.0, 15.0, 60.0, 1440.0])),
        ]


class MrpBomByproduct(models.Model):
    _inherit = 'mrp.bom.byproduct'

    _populate_sizes = {'small': 50, 'medium': 1_000, 'large': 5_000}
    _populate_dependencies = ['mrp.bom.line', 'mrp.routing.workcenter']

    def _populate_factories(self):
        # Take a subset (50%) of bom to have some of then without any operation
        random = populate.Random('byproduct_subset_bom')
        boms_ids = self.env.registry.populated_models['mrp.bom']
        boms_ids = random.sample(boms_ids, int(len(boms_ids) * 0.5))

        boms = self.env['mrp.bom'].search([('id', 'in', self.env.registry.populated_models['mrp.bom'])], order='sequence, product_id, id')

        product_manu_ids = OrderedSet()
        for bom in boms:
            if bom.product_id:
                product_manu_ids.add(bom.product_id.id)
            else:
                for product_id in bom.product_tmpl_id.product_variant_ids:
                    product_manu_ids.add(product_id.id)
        product_manu = self.env['product.product'].browse(product_manu_ids)
        # product_no_manu is products which don't have any bom (leaves in the BoM trees)
        product_no_manu = self.env['product.product'].browse(self.env.registry.populated_models['product.product']) - product_manu
        product_no_manu_ids = product_no_manu.ids

        def get_product_uom_id(values, counter, random):
            return self.env['product.product'].browse(values['product_id']).uom_id.id

        return [
            ('bom_id', populate.iterate(boms_ids)),
            ('product_id', populate.randomize(product_no_manu_ids)),
            ('product_uom_id', populate.compute(get_product_uom_id)),
            ('product_qty', populate.randint(1, 10)),
        ]


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    _populate_sizes = {'small': 100, 'medium': 1_000, 'large': 10_000}
    _populate_dependencies = ['mrp.routing.workcenter', 'mrp.bom.line']

    def _populate(self, size):
        productions = super()._populate(size)

        def fill_mo_with_bom_info():
            productions_with_bom = productions.filtered('bom_id')
            _logger.info("Create Raw moves of MO(s) with bom (%d)" % len(productions_with_bom))
            self.env['stock.move'].create(productions_with_bom._get_moves_raw_values())
            _logger.info("Create Finished moves of MO(s) with bom (%d)" % len(productions_with_bom))
            self.env['stock.move'].create(productions_with_bom._get_moves_finished_values())
            _logger.info("Create Workorder moves of MO(s) with bom (%d)" % len(productions_with_bom))
            productions_with_bom._create_workorder()

        fill_mo_with_bom_info()

        def confirm_bom_mo(sample_ratio):
            # Confirm X % of prototype MO
            random = populate.Random('confirm_bom_mo')
            mo_ids = productions.filtered('bom_id').ids
            mo_to_confirm = self.env['mrp.production'].browse(random.sample(mo_ids, int(len(mo_ids) * 0.8)))
            _logger.info("Confirm %d MO with BoM" % len(mo_to_confirm))
            mo_to_confirm.action_confirm()

        # Uncomment this line to confirm a part of MO, can be useful to check performance
        # confirm_bom_mo(0.8)

        return productions

    def _populate_factories(self):
        now = datetime.now()
        company_ids = self.env.registry.populated_models['res.company'][:COMPANY_NB_WITH_STOCK]

        products = self.env['product.product'].browse(self.env.registry.populated_models['product.product'])
        product_ids = products.filtered(lambda product: product.type in ('product', 'consu')).ids

        boms = self.env['mrp.bom'].browse(self.env.registry.populated_models['mrp.bom'])
        boms_by_company = defaultdict(OrderedSet)
        for bom in boms:
            boms_by_company[bom.company_id.id].add(bom.id)
        boms_by_company = {company_id: self.env['mrp.bom'].browse(boms) for company_id, boms in boms_by_company.items()}
        boms_by_company = {
            company_id: boms | boms_by_company.get(False, self.env['mrp.bom'])
            for company_id, boms in boms_by_company.items()}

        def get_bom_id(values, counter, random):
            if random.random() > 0.7:  # 30 % of prototyping
                return False
            return random.choice(boms_by_company[values['company_id']]).id

        def get_consumption(values, counter, random):
            if not values['bom_id']:
                return 'flexible'
            return self.env['mrp.bom'].browse(values['bom_id']).consumption

        def get_product_id(values, counter, random):
            if not values['bom_id']:
                return random.choice(product_ids)
            bom = self.env['mrp.bom'].browse(values['bom_id'])
            return bom.product_id.id or random.choice(bom.product_tmpl_id.product_variant_ids.ids)

        def get_product_uom_id(values, counter, random):
            product = self.env['product.product'].browse(values['product_id'])
            return product.uom_id.id

        # Fetch all stock picking type and group then by company_id
        manu_picking_types = self.env['stock.picking.type'].search([('code', '=', 'mrp_operation')])
        manu_picking_types_by_company_id = defaultdict(OrderedSet)
        for picking_type in manu_picking_types:
            manu_picking_types_by_company_id[picking_type.company_id.id].add(picking_type.id)
        manu_picking_types_by_company_id = {company_id: list(picking_ids) for company_id, picking_ids in manu_picking_types_by_company_id.items()}

        def get_picking_type_id(values, counter, random):
            return random.choice(manu_picking_types_by_company_id[values['company_id']])

        def get_location_src_id(values, counter, random):
            # TODO : add some randomness
            picking_type = self.env['stock.picking.type'].browse(values['picking_type_id'])
            return picking_type.default_location_src_id.id

        def get_location_dest_id(values, counter, random):
            # TODO : add some randomness
            picking_type = self.env['stock.picking.type'].browse(values['picking_type_id'])
            return picking_type.default_location_dest_id.id

        def get_date_planned_start(values, counter, random):
            # 95.45 % of picking scheduled between (-10, 30) days and follow a gauss distribution (only +-15% picking is late)
            delta = random.gauss(10, 10)
            return now + timedelta(days=delta)

        return [
            ('company_id', populate.iterate(company_ids)),
            ('bom_id', populate.compute(get_bom_id)),
            ('consumption', populate.compute(get_consumption)),
            ('product_id', populate.compute(get_product_id)),
            ('product_uom_id', populate.compute(get_product_uom_id)),
            ('product_qty', populate.randint(1, 10)),
            ('picking_type_id', populate.compute(get_picking_type_id)),
            ('date_planned_start', populate.compute(get_date_planned_start)),
            ('location_src_id', populate.compute(get_location_src_id)),
            ('location_dest_id', populate.compute(get_location_dest_id)),
            ('priority', populate.iterate(['0', '1'], [0.95, 0.05])),
        ]


class StockMove(models.Model):
    _inherit = 'stock.move'

    _populate_dependencies = ['stock.picking', 'mrp.production']

    def _populate(self, size):
        moves = super()._populate(size)

        def confirm_prototype_mo(sample_ratio):
            # Confirm X % of prototype MO
            random = populate.Random('confirm_prototype_mo')
            mo_ids = moves.raw_material_production_id.ids
            mo_to_confirm = self.env['mrp.production'].browse(random.sample(mo_ids, int(len(mo_ids) * 0.8)))
            _logger.info("Confirm %d of prototype MO" % len(mo_to_confirm))
            mo_to_confirm.action_confirm()

        # (Un)comment this line to confirm a part of prototype MO, can be useful to check performance
        # confirm_prototype_mo(0.8)

        return moves.exists()  # Confirm Mo can unlink moves

    def _populate_attach_record_weight(self):
        fields, weight = super()._populate_attach_record_weight()
        return fields + ['raw_material_production_id'], weight + [1]

    def _populate_attach_record_generator(self):

        productions = self.env['mrp.production'].browse(self.env.registry.populated_models['mrp.production'])
        productions = productions.filtered(lambda prod: not prod.bom_id)

        def next_production_id():
            while productions:
                yield from productions.ids

        return {**super()._populate_attach_record_generator(), 'raw_material_production_id': next_production_id()}

    def _populate_factories(self):

        def _compute_production_values(iterator, field_name, model_name):
            for values in iterator:

                if values.get('raw_material_production_id'):
                    production = self.env['mrp.production'].browse(values['raw_material_production_id'])
                    values['location_id'] = production.location_src_id.id
                    values['location_dest_id'] = production.production_location_id.id
                    values['picking_type_id'] = production.picking_type_id.id
                    values['name'] = production.name
                    values['date'] = production.date_planned_start
                    values['company_id'] = production.company_id.id
                yield values

        return super()._populate_factories() + [
            ('_compute_production_values', _compute_production_values),
        ]

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp

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
                                <div t-field="o.name" t-options="{'widget': 'barcode', 'width': 600, 'height': 100, 'img_style': 'width:350px;height:60px'}"/>
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
                        <div class="col-3" t-if="o.product_description_variants">
                            <strong>Description:</strong><br/>
                            <span t-field="o.product_description_variants"/>
                        </div>
                        <div class="col-3">
                            <strong>Quantity to Produce:</strong><br/>
                            <span t-field="o.product_qty"/>
                            <span t-field="o.product_uom_id.name" groups="uom.group_uom"/>
                        </div>
                    </div>

                    <div t-if="o.workorder_ids" groups="mrp.group_mrp_routings">
                        <h3>
                            <span t-if="o.state == 'done'">Operations Done</span>
                            <span t-else="">Operations Planned</span>
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
                        <span t-if="o.state == 'done'">
                            Consumed Products
                        </span>
                        <span t-else="">
                            Products to Consume
                        </span>
                    </h3>

                    <table class="table table-sm" t-if="o.move_raw_ids">
                        <t t-set="has_product_barcode" t-value="any(m.product_id.barcode for m in o.move_raw_ids)"/>
                        <thead>
                            <tr>
                                <th>Product</th>
                                <th t-attf-class="{{ 'text-right' if not has_product_barcode else '' }}">Quantity</th>
                                <th t-if="has_product_barcode" width="15%" class="text-center">Barcode</th>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-if="o.move_raw_ids">
                                <tr t-foreach="o.move_raw_ids.filtered(lambda m: m.state != 'cancel')" t-as="raw_line">
                                    <td>
                                        <span t-field="raw_line.product_id"/>
                                    </td>
                                    <td t-attf-class="{{ 'text-right' if not has_product_barcode else '' }}">
                                        <t t-if="o.state == 'done'">
                                            <span t-field="raw_line.quantity_done"/>
                                        </t>
                                        <t t-else="">
                                            <span t-field="raw_line.product_uom_qty"/>
                                        </t>
                                        <span t-field="raw_line.product_uom" groups="uom.group_uom"/>
                                    </td>
                                    <td t-if="has_product_barcode" width="15%" class="text-center">
                                        <t t-if="raw_line.product_id.barcode">
                                            <div t-field="raw_line.product_id.barcode" t-options="{'widget': 'barcode', 'width': 600, 'height': 100, 'img_style': 'width:100%;height:35px'}"/>
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
            <t t-set="uom_categ_unit" t-value="env.ref('uom.product_uom_categ_unit')"/>
            <t t-foreach="docs" t-as="production">
                <t t-foreach="production.move_finished_ids" t-as="move">
                    <t t-if="production.state == 'done'">
                        <t t-set="move_lines" t-value="move.move_line_ids.filtered(lambda x: x.state == 'done' and x.qty_done)"/>
                    </t>
                    <t t-else="">
                        <t t-set="move_lines" t-value="move.move_line_ids.filtered(lambda x: x.state != 'done' and x.product_qty)"/>
                    </t>
                    <t t-foreach="move_lines" t-as="move_line">
                        <t t-if="move_line.product_uom_id.category_id == uom_categ_unit">
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
                                                <span>Quantity:</span>
                                                <t t-if="move_line.product_uom_id.category_id == uom_categ_unit">
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
                                                        <div t-field="move_line.lot_name or move_line.lot_id.name" t-options="{'widget': 'barcode', 'width': 600, 'height': 150, 'img_style': 'width:100%;height:4rem'}"/>
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
                                                        <div t-field="move_line.product_id.barcode" t-options="{'widget': 'barcode', 'width': 600, 'height': 150, 'img_style': 'width:100%;height:4rem'}"/>
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
            variant = data.get('variant')
            candidates = variant and self.env['product.product'].browse(int(variant)) or bom.product_id or bom.product_tmpl_id.product_variant_ids
            quantity = float(data.get('quantity', bom.product_qty))
            for product_variant_id in candidates.ids:
                if data and data.get('childs'):
                    doc = self._get_pdf_line(bom_id, product_id=product_variant_id, qty=quantity, child_bom_ids=set(json.loads(data.get('childs'))))
                else:
                    doc = self._get_pdf_line(bom_id, product_id=product_variant_id, qty=quantity, unfolded=True)
                doc['report_type'] = 'pdf'
                doc['report_structure'] = data and data.get('report_type') or 'all'
                docs.append(doc)
            if not candidates:
                if data and data.get('childs'):
                    doc = self._get_pdf_line(bom_id, qty=quantity, child_bom_ids=set(json.loads(data.get('childs'))))
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
        res['lines'] = self.env.ref('mrp.report_mrp_bom')._render({'data': res['lines']})
        return res

    @api.model
    def get_bom(self, bom_id=False, product_id=False, line_qty=False, line_id=False, level=False):
        lines = self._get_bom(bom_id=bom_id, product_id=product_id, line_qty=line_qty, line_id=line_id, level=level)
        return self.env.ref('mrp.report_mrp_bom_line')._render({'data': lines})

    @api.model
    def get_operations(self, product_id=False, bom_id=False, qty=0, level=0):
        bom = self.env['mrp.bom'].browse(bom_id)
        product = self.env['product.product'].browse(product_id)
        lines = self._get_operation_line(product, bom, float_round(qty / bom.product_qty, precision_rounding=1, rounding_method='UP'), level)
        values = {
            'bom_id': bom_id,
            'currency': self.env.company.currency_id,
            'operations': lines,
            'extra_column_count': self._get_extra_column_count()
        }
        return self.env.ref('mrp.report_mrp_operation_line')._render({'data': values})

    @api.model
    def get_byproducts(self, bom_id=False, qty=0, level=0, total=0):
        bom = self.env['mrp.bom'].browse(bom_id)
        lines, dummy = self._get_byproducts_lines(bom, qty, level, total)
        values = {
            'bom_id': bom_id,
            'currency': self.env.company.currency_id,
            'byproducts': lines,
            'extra_column_count': self._get_extra_column_count(),
        }
        return self.env.ref('mrp.report_mrp_byproduct_line')._render({'data': values})

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
            'is_uom_applied': self.env.user.user_has_groups('uom.group_uom'),
            'extra_column_count': self._get_extra_column_count()
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
            attachments = self.env['mrp.document'].search(['|', '&', ('res_model', '=', 'product.product'),
            ('res_id', '=', product.id), '&', ('res_model', '=', 'product.template'), ('res_id', '=', product.product_tmpl_id.id)])
        else:
            # Use the product template instead of the variant
            product = bom.product_tmpl_id
            attachments = self.env['mrp.document'].search([('res_model', '=', 'product.template'), ('res_id', '=', product.id)])
        operations = self._get_operation_line(product, bom, float_round(bom_quantity, precision_rounding=1, rounding_method='UP'), 0)
        lines = {
            'bom': bom,
            'bom_qty': bom_quantity,
            'bom_prod_name': product.display_name,
            'currency': company.currency_id,
            'product': product,
            'code': bom and bom.display_name or '',
            'price': product.uom_id._compute_price(product.with_company(company).standard_price, bom.product_uom_id) * bom_quantity,
            'total': sum([op['total'] for op in operations]),
            'level': level or 0,
            'operations': operations,
            'operations_cost': sum([op['total'] for op in operations]),
            'attachments': attachments,
            'operations_time': sum([op['duration_expected'] for op in operations])
        }
        components, total = self._get_bom_lines(bom, bom_quantity, product, line_id, level)
        lines['total'] += total
        lines['components'] = components
        byproducts, byproduct_cost_portion = self._get_byproducts_lines(bom, bom_quantity, level, lines['total'])
        lines['byproducts'] = byproducts
        lines['cost_share'] = float_round(1 - byproduct_cost_portion, precision_rounding=0.0001)
        lines['bom_cost'] = lines['total'] * lines['cost_share']
        lines['byproducts_cost'] = sum(byproduct['bom_cost'] for byproduct in byproducts)
        lines['byproducts_total'] = sum(byproduct['product_qty'] for byproduct in byproducts)
        lines['extra_column_count'] = self._get_extra_column_count()
        return lines

    def _get_bom_lines(self, bom, bom_quantity, product, line_id, level):
        components = []
        total = 0
        for line in bom.bom_line_ids:
            line_quantity = (bom_quantity / (bom.product_qty or 1.0)) * line.product_qty
            if line._skip_bom_line(product):
                continue
            company = bom.company_id or self.env.company
            price = line.product_id.uom_id._compute_price(line.product_id.with_company(company).standard_price, line.product_uom_id) * line_quantity
            if line.child_bom_id:
                factor = line.product_uom_id._compute_quantity(line_quantity, line.child_bom_id.product_uom_id)
                sub_total = self._get_price(line.child_bom_id, factor, line.product_id)
                byproduct_cost_share = sum(line.child_bom_id.byproduct_ids.mapped('cost_share'))
                if byproduct_cost_share:
                    sub_total *= float_round(1 - byproduct_cost_share / 100, precision_rounding=0.0001)
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

    def _get_byproducts_lines(self, bom, bom_quantity, level, total):
        byproducts = []
        byproduct_cost_portion = 0
        company = bom.company_id or self.env.company
        for byproduct in bom.byproduct_ids:
            line_quantity = (bom_quantity / (bom.product_qty or 1.0)) * byproduct.product_qty
            cost_share = byproduct.cost_share / 100
            byproduct_cost_portion += cost_share
            price = byproduct.product_id.uom_id._compute_price(byproduct.product_id.with_company(company).standard_price, byproduct.product_uom_id) * line_quantity
            byproducts.append({
                'product_id': byproduct.product_id,
                'product_name': byproduct.product_id.display_name,
                'product_qty': line_quantity,
                'product_uom': byproduct.product_uom_id.name,
                'product_cost': company.currency_id.round(price),
                'parent_id': bom.id,
                'level': level or 0,
                'bom_cost': company.currency_id.round(total * cost_share),
                'cost_share': cost_share,
            })
        return byproducts, byproduct_cost_portion

    def _get_operation_line(self, product, bom, qty, level):
        operations = []
        total = 0.0
        qty = bom.product_uom_id._compute_quantity(qty, bom.product_tmpl_id.uom_id)
        for operation in bom.operation_ids:
            if operation._skip_operation_line(product):
                continue
            operation_cycle = float_round(qty / operation.workcenter_id.capacity, precision_rounding=1, rounding_method='UP')
            duration_expected = (operation_cycle * operation.time_cycle * 100.0 / operation.workcenter_id.time_efficiency) + (operation.workcenter_id.time_stop + operation.workcenter_id.time_start)
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
        if bom.operation_ids:
            # routing are defined on a BoM and don't have a concept of quantity.
            # It means that the operation time are defined for the quantity on
            # the BoM (the user produces a batch of products). E.g the user
            # product a batch of 10 units with a 5 minutes operation, the time
            # will be the 5 for a quantity between 1-10, then doubled for
            # 11-20,...
            operation_cycle = float_round(factor, precision_rounding=1, rounding_method='UP')
            operations = self._get_operation_line(product, bom, operation_cycle, 0)
            price += sum([op['total'] for op in operations])

        for line in bom.bom_line_ids:
            if line._skip_bom_line(product):
                continue
            if line.child_bom_id:
                qty = line.product_uom_id._compute_quantity(line.product_qty * (factor / bom.product_qty), line.child_bom_id.product_uom_id)
                sub_price = self._get_price(line.child_bom_id, qty, line.product_id)
                byproduct_cost_share = sum(line.child_bom_id.byproduct_ids.mapped('cost_share'))
                if byproduct_cost_share:
                    sub_price *= float_round(1 - byproduct_cost_share / 100, precision_rounding=0.0001)
                price += sub_price
            else:
                prod_qty = line.product_qty * factor / bom.product_qty
                company = bom.company_id or self.env.company
                not_rounded_price = line.product_id.uom_id._compute_price(line.product_id.with_company(company).standard_price, line.product_uom_id) * prod_qty
                price += company.currency_id.round(not_rounded_price)
        return price

    def _get_sub_lines(self, bom, product_id, line_qty, line_id, level, child_bom_ids, unfolded):
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
                lines += (self._get_sub_lines(line.child_bom_id, line.product_id.id, bom_line['prod_qty'], line, level + 1, child_bom_ids, unfolded))
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
        if data['byproducts']:
                lines.append({
                    'name': _('Byproducts'),
                    'type': 'byproduct',
                    'uom': False,
                    'quantity': data['byproducts_total'],
                    'bom_cost': data['byproducts_cost'],
                    'level': level,
                })
                for byproduct in data['byproducts']:
                    if unfolded or 'byproduct-' + str(bom.id) in child_bom_ids:
                        lines.append({
                            'name': byproduct['product_name'],
                            'type': 'byproduct',
                            'quantity': byproduct['product_qty'],
                            'uom': byproduct['product_uom'],
                            'prod_cost': byproduct['product_cost'],
                            'bom_cost': byproduct['bom_cost'],
                            'level': level + 1,
                        })
        return lines

    def _get_pdf_line(self, bom_id, product_id=False, qty=1, child_bom_ids=None, unfolded=False):
        if child_bom_ids is None:
            child_bom_ids = set()

        bom = self.env['mrp.bom'].browse(bom_id)
        product_id = product_id or bom.product_id.id or bom.product_tmpl_id.product_variant_id.id
        data = self._get_bom(bom_id=bom_id, product_id=product_id, line_qty=qty, )
        pdf_lines = self._get_sub_lines(bom, product_id, qty, False, 1, child_bom_ids, unfolded)
        data['components'] = []
        data['lines'] = pdf_lines
        data['extra_column_count'] = self._get_extra_column_count()
        return data

    def _get_extra_column_count(self):
        return 0

```

## File: report\mrp_report_bom_structure.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="report_mrp_bom">
        <div class="container o_mrp_bom_report_page">
            <t t-if="data.get('components') or data.get('lines') or data.get('operations')">
                <div class="row">
                    <div class="col-lg-12">
                        <h1 style="display:inline;">BoM Structure </h1>
                        <h1 style="display:inline;" t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_prod_cost">&amp; Cost</h1>
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
                                            <span><a href="#" t-if="data['report_type'] == 'html'" t-att-data-res-id="data['product'].id" t-att-data-model="data['product']._name" class="o_mrp_bom_action"><t t-esc="data['bom_prod_name']"/></a><t t-else="" t-esc="data['bom_prod_name']"/></span>
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
                                            <span><t t-esc="data['bom_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
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
                                    <tr t-if="data['report_structure'] != 'bom_structure'" class="o_mrp_prod_cost">
                                        <td></td>
                                        <t t-foreach="range(data.get('extra_column_count', 0))" t-as="index">
                                            <td/>
                                        </t>
                                        <td name="td_mrp_bom_f" class="text-right">
                                            <span><t t-if="data['byproducts']" t-esc="data['bom_prod_name']"/></span>
                                        </td>
                                        <td class="text-right"><span><strong>Unit Cost</strong></span></td>
                                        <td groups="uom.group_uom"><span><t t-esc="data['bom'].product_uom_id.name"/></span></td>
                                        <td class="text-right">
                                            <span><t t-esc="data['price']/data['bom_qty']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                        </td>
                                        <td class="text-right">
                                            <span><t t-esc="data['cost_share'] * data['total'] / data['bom_qty']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                        </td>
                                    </tr>
                                    <t t-if="data['report_structure'] != 'bom_structure'" t-foreach="data['byproducts']" t-as="byproduct">
                                        <tr class="o_mrp_bom_cost">
                                            <td/>
                                            <t t-foreach="range(data.get('extra_column_count', 0))" t-as="index">
                                                <td/>
                                            </t>
                                            <td name="td_mrp_bom_byproducts_f" class="text-right">
                                                <span><t t-esc="byproduct['product_name']"/></span>
                                            </td>
                                            <td class="text-right"><span><strong>Unit Cost</strong></span></td>
                                            <td groups="uom.group_uom"><span><t t-esc="byproduct['product_uom']"/></span></td>
                                            <td class="text-right">
                                                <span><t t-esc="byproduct['product_cost'] / byproduct['product_qty']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                            </td>
                                            <td class="text-right">
                                                <span><t t-esc="byproduct['cost_share'] * data['total'] / byproduct['product_qty']" t-options='{"widget": "monetary", "display_currency": currency}'/></span>
                                            </td>
                                        </tr>
                                    </t>
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
        <t t-if="data['operations']" name="operations">
            <t t-set="space_td" t-value="'margin-left: '+ str(data['level'] * 20) + 'px;'"/>
            <tr class="o_mrp_bom_report_line o_mrp_bom_cost" t-att-data-product_id="data['product'].id" t-att-data-id="'operation-' + str(data['bom'].id)" t-att-data-bom-id="data['bom'].id" t-att-parent_id="data['bom'].id" t-att-data-qty="data['bom_qty']" t-att-data-level="data['level']">
                <td name="td_opr">
                    <span t-att-style="space_td"/>
                    <span class="o_mrp_bom_unfoldable fa fa-fw fa-caret-right" t-att-data-function="'get_operations'" role="img" aria-label="Unfold" title="Unfold"/>
                    Operations
                </td>
                <t t-foreach="range(data.get('extra_column_count', 0))" t-as="index">
                    <td/>
                </t>
                <td/>
                <td class="text-right">
                    <span t-esc="data['operations_time']" t-options='{"widget": "float_time"}'/>
                </td>
                <td groups="uom.group_uom"><span>Minutes</span></td>
                <td/>
                <td class="o_mrp_bom_cost text-right">
                    <span t-esc="data['operations_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                </td>
                <td/>
            </tr>
        </t>
        <t t-if="data['byproducts']">
            <t t-set="space_td" t-value="'margin-left: '+ str(data['level'] * 20) + 'px;'"/>
            <tr class="o_mrp_bom_report_line o_mrp_bom_cost" t-att-data-id="'byproduct-' + str(data['bom'].id)" t-att-data-bom-id="data['bom'].id" t-att-parent_id="data['bom'].id" t-att-data-qty="data['bom_qty']" t-att-data-level="data['level']" t-att-data-total="data['total']">
                <td name="td_byproducts">
                    <span t-att-style="space_td"/>
                    <span class="o_mrp_bom_unfoldable fa fa-fw fa-caret-right" t-att-data-function="'get_byproducts'" role="img" aria-label="Unfold" title="Unfold"/>
                    By-Products
                </td>
                <t t-foreach="range(data.get('extra_column_count', 0))" t-as="index">
                    <td/>
                </t>
                <td/>
                <td class="text-right">
                    <span t-esc="data['byproducts_total']"/>
                </td>
                <td groups="uom.group_uom"/>
                <td/>
                <td class="text-right">
                    <span t-esc="data['byproducts_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                </td>
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
              <t t-foreach="range(data.get('extra_column_count', 0))" t-as="index">
                  <td/>
              </t>
              <td/>
              <td class="text-right">
                  <span t-esc="op['duration_expected']" t-options='{"widget": "float_time"}'/>
              </td>
              <td groups="uom.group_uom"><span>Minutes</span></td>
              <td/>
              <td class="o_mrp_bom_cost text-right">
                  <span t-esc="op['total']" t-options='{"widget": "monetary", "display_currency": currency}'/>
              </td>
              <td/>
          </tr>
      </t>
    </template>

    <template id="report_mrp_byproduct_line">
        <t t-set="currency" t-value="data['currency']"/>
        <t t-foreach="data['byproducts']" t-as="byproduct">
            <t t-set="space_td" t-value="'margin-left: '+ str(byproduct['level'] * 20) + 'px;'"/>
            <tr class="o_mrp_bom_report_line o_mrp_bom_cost"  t-att-parent_id="'byproduct-' + str(data['bom_id'])">
                <td name="td_byproduct_line">
                    <span t-att-style="space_td"/>
                    <a href="#" t-att-data-res-id="byproduct['product_id'].id" t-att-data-model="byproduct['product_id']._name" class="o_mrp_bom_action"><t t-esc="byproduct['product_name']"/></a>
                </td>
                <t t-foreach="range(data.get('extra_column_count', 0))" t-as="index">
                    <td/>
                </t>
                <td/>
                <td class="text-right">
                    <span t-esc="byproduct['product_qty']"/>
                </td>
                <td groups="uom.group_uom"><span t-esc="byproduct['product_uom']"/></td>
                <td class="text-right">
                    <span t-esc="byproduct['product_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                </td>
                <td class="text-right">
                    <span t-esc="byproduct['bom_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
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
          <tr t-if="data['report_structure'] != 'bom_structure' or l['type'] not in ['operation', 'byproduct']">
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
                      <t t-if="l['type'] in ['bom', 'byproduct']" t-esc="l['quantity']" t-options='{"widget": "float", "decimal_precision": "Product Unit of Measure"}'/>
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
        <record id="action_report_production_order" model="ir.actions.report">
            <field name="name">Production Order</field>
            <field name="model">mrp.production</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">mrp.report_mrporder</field>
            <field name="report_file">mrp.report.mrp_production_templates</field>
            <field name="print_report_name">'Production Order - %s' % object.name</field>
            <field name="binding_model_id" ref="model_mrp_production"/>
            <field name="binding_type">report</field>
        </record>
        <record id="action_report_bom_structure" model="ir.actions.report">
            <field name="name">BoM Structure</field>
            <field name="model">mrp.bom</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">mrp.report_bom_structure</field>
            <field name="report_file">mrp.report_bom_structure</field>
            <field name="print_report_name">'Bom Structure - %s' % object.display_name</field>
            <field name="binding_model_id" ref="model_mrp_bom"/>
            <field name="binding_type">report</field>
        </record>
        <record id="label_manufacture_template" model="ir.actions.report">
            <field name="name">Finished Product Label (ZPL)</field>
            <field name="model">mrp.production</field>
            <field name="report_type">qweb-text</field>
            <field name="report_name">mrp.label_production_view</field>
            <field name="report_file">mrp.label_production_view</field>
            <field name="binding_model_id" ref="model_mrp_production"/>
            <field name="binding_type">report</field>
        </record>
        <record id="action_report_finished_product" model="ir.actions.report">
            <field name="name">Finished Product Label (PDF)</field>
            <field name="model">mrp.production</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">mrp.label_production_view_pdf</field>
            <field name="report_file">mrp.label_production_view_pdf</field>
            <field name="print_report_name">'Finished products - %s' % object.name</field>
            <field name="binding_model_id" ref="model_mrp_production"/>
            <field name="binding_type">report</field>
        </record>
        <!-- TODO: Delete this report -->
        <record id="label_production_order" model="ir.actions.report">
            <field name="name">Order Label</field>
            <field name="model">mrp.production</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">mrp.report_reception_report_label_mrp</field>
            <field name="report_file">mrp.report_reception_report_label_mrp</field>
            <field name="paperformat_id" ref="product.paperformat_label_sheet_dymo"/>
            <field name="binding_model_id" ref="model_mrp_production"/>
            <field name="binding_type">report</field>
        </record>
    </data>
</odoo>

```

## File: report\mrp_zebra_production_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <template id="label_production_view">
            <t t-set="uom_categ_unit" t-value="env.ref('uom.product_uom_categ_unit')"/>
            <t t-foreach="docs" t-as="production">
                <t t-foreach="production.move_finished_ids" t-as="move">
                    <t t-foreach="move.move_line_ids" t-as="move_line">
                        <t t-if="move_line.product_uom_id.category_id == uom_categ_unit">
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

## File: report\report_deliveryslip.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="stock_report_delivery_document_inherit_mrp" inherit_id="stock.report_delivery_document">
        <!-- needs to be set before so elif directly follows if later on -->
        <xpath expr="//t[@name='has_packages']" position="before">
            <!-- get only the top level kits' (i.e. no subkit) move lines for easier mapping later on + we ignore subkit groupings-->
            <!-- note that move.name uses top level kit's product.template.display_name value instead of product.template.name -->
            <t t-set="has_kits" t-value="o.move_line_ids.filtered(lambda l: l.move_id.bom_line_id and l.move_id.bom_line_id.bom_id.type == 'phantom')"/>
        </xpath>
        <xpath expr="//t[@name='no_package_section']" position="before">
            <t t-set="has_kits" t-value="o.move_line_ids.filtered(lambda l: l.move_id.bom_line_id and l.move_id.bom_line_id.bom_id.type == 'phantom')"/>
            <t t-if="has_kits">
                <!-- print the products not in a package or kit first -->
                <t t-set="move_lines" t-value="move_lines.filtered(lambda m: not m.move_id.bom_line_id)"/>
            </t>
        </xpath>
        <xpath expr="//t[@name='no_package_move_lines']" position="inside">
            <t t-call="mrp.stock_report_delivery_kit_sections"/>
        </xpath>
        <xpath expr="//t[@name='has_packages']" position="after">
            <!-- Additional use case: group by kits when no packages exist and then apply use case 1. (serial/lot numbers used/printed) -->
            <t t-elif="has_kits and not has_packages">
                <t t-call="mrp.stock_report_delivery_kit_sections"/>
                <t t-call="mrp.stock_report_delivery_no_kit_section"/>
            </t>
        </xpath>
    </template>

    <template id="stock_report_delivery_kit_sections">
        <!-- get all kits-related SML, including subkits and excluding the packaged SML -->
        <t t-set="all_kits_move_lines" t-value="o.move_line_ids.filtered(lambda l: l.move_id.bom_line_id.bom_id.type == 'phantom' and not l.result_package_id)"/>
        <!-- do another map to get unique top level kits -->
        <t t-set="boms" t-value="has_kits.mapped('move_id.bom_line_id.bom_id')"/>
        <t t-foreach="boms" t-as="bom">
            <!-- Separate product.product from template for variants-->
            <t t-if="bom.product_id">
                <t t-set="kit_product" t-value="bom.product_id"/>
            </t>
            <t t-else="">
                <t t-set="kit_product" t-value="bom.product_tmpl_id"/>
            </t>
            <tr t-att-class="'bg-200 font-weight-bold o_line_section'">
                <td colspan="99">
                    <span t-esc="kit_product.display_name"/>
                </td>
            </tr>
            <t t-set="kit_move_lines" t-value="all_kits_move_lines.filtered(lambda l: l.move_id.bom_line_id.bom_id == bom)"/>
            <t t-if="has_serial_number">
                <tr t-foreach="kit_move_lines" t-as="move_line">
                    <t t-set="description" t-value="move_line.move_id.description_picking"/>
                    <t t-if="description == kit_product.display_name">
                        <t t-set="description" t-value=""/>
                    </t>
                    <t t-call="stock.stock_report_delivery_has_serial_move_line"/>
                </tr>
            </t>
            <t t-else="">
                <t t-set="aggregated_lines" t-value="kit_move_lines._get_aggregated_product_quantities(kit_name=kit_product.display_name)"/>
                <t t-if="aggregated_lines">
                    <t t-call="stock.stock_report_delivery_aggregated_move_lines"/>
                </t>
            </t>
        </t>
    </template>

    <!-- No kit section is expected to only be called in no packages case -->
    <template id="stock_report_delivery_no_kit_section">
        <!-- Do another section for kit-less products if they exist -->
        <t t-set="no_kit_move_lines" t-value="o.move_line_ids.filtered(lambda l: l.move_id.bom_line_id.bom_id.type != 'phantom')"/>
        <t t-if="no_kit_move_lines">
            <tr t-att-class="'bg-200 font-weight-bold o_line_section'">
                <td colspan="99">
                    <span>Products not associated with a kit</span>
                </td>
            </tr>
            <t t-if="has_serial_number">
                <tr t-foreach="no_kit_move_lines" t-as="move_line">
                    <t t-call="stock.stock_report_delivery_has_serial_move_line"/>
                </tr>
            </t>
            <t t-else="">
                <t t-set="aggregated_lines" t-value="no_kit_move_lines._get_aggregated_product_quantities()"/>
                <t t-if="aggregated_lines">
                    <t t-call="stock.stock_report_delivery_aggregated_move_lines"/>
                </t>
            </t>
        </t>
    </template>
</odoo>

```

## File: report\report_stock_forecasted.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ReplenishmentReport(models.AbstractModel):
    _inherit = 'report.stock.report_product_product_replenishment'

    def _move_draft_domain(self, product_template_ids, product_variant_ids, wh_location_ids):
        in_domain, out_domain = super()._move_draft_domain(product_template_ids, product_variant_ids, wh_location_ids)
        in_domain += [('production_id', '=', False)]
        out_domain += [('raw_material_production_id', '=', False)]
        return in_domain, out_domain

    def _compute_draft_quantity_count(self, product_template_ids, product_variant_ids, wh_location_ids):
        res = super()._compute_draft_quantity_count(product_template_ids, product_variant_ids, wh_location_ids)
        res['draft_production_qty'] = {}
        domain = self._product_domain(product_template_ids, product_variant_ids)
        domain += [('state', '=', 'draft')]

        # Pending incoming quantity.
        mo_domain = domain + [('location_dest_id', 'in', wh_location_ids)]
        grouped_mo = self.env['mrp.production'].read_group(mo_domain, ['product_qty:sum'], 'product_id')
        res['draft_production_qty']['in'] = sum(mo['product_qty'] for mo in grouped_mo)

        # Pending outgoing quantity.
        move_domain = domain + [
            ('raw_material_production_id', '!=', False),
            ('location_id', 'in', wh_location_ids),
        ]
        grouped_moves = self.env['stock.move'].read_group(move_domain, ['product_qty:sum'], 'product_id')
        res['draft_production_qty']['out'] = sum(move['product_qty'] for move in grouped_moves)
        res['qty']['in'] += res['draft_production_qty']['in']
        res['qty']['out'] += res['draft_production_qty']['out']

        return res

```

## File: report\report_stock_forecasted.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="mrp_report_product_product_replenishment" inherit_id="stock.report_product_product_replenishment">
        <xpath expr="//tr[@name='draft_picking_in']" position="after">
            <tr t-if="docs['draft_production_qty']['in']" name="draft_mo_in">
                <td colspan="2">Production of Draft MO</td>
                <td t-esc="docs['draft_production_qty']['in']" t-options="{'widget': 'float', 'precision': precision}" class="text-right"/>
            </tr>
        </xpath>
        <xpath expr="//tr[@name='draft_picking_out']" position="after">
            <tr t-if="docs['draft_production_qty']['out']" name="draft_mo_out">
                <td colspan="2">Component of Draft MO</td>
                <td t-esc="-docs['draft_production_qty']['out']" t-options="{'widget': 'float', 'precision': precision}" class="text-right"/>
            </tr>
        </xpath>
         <xpath expr="//button[@name='unreserve_link']" position="after">
            <button t-if="line['move_out'] and line['move_out'].raw_material_production_id and line['move_out'].raw_material_production_id.unreserve_visible"
                class="btn btn-sm btn-primary o_report_replenish_unreserve"
                t-attf-model="mrp.production"
                t-att-model-id="line['move_out'].raw_material_production_id.id">
                Unreserve
            </button>
        </xpath>
        <xpath expr="//button[@name='reserve_link']" position="after">
            <button t-if="line['move_out'] and line['move_out'].raw_material_production_id and line['move_out'].raw_material_production_id.reserve_visible"
                class="btn btn-sm btn-primary o_report_replenish_reserve"
                t-attf-model="mrp.production"
                t-att-model-id="line['move_out'].raw_material_production_id.id">
                Reserve
            </button>
        </xpath>
         <xpath expr="//button[@name='change_priority_link']" position="after">
            <button t-if="line['move_out'] and line['move_out'].raw_material_production_id"
                t-attf-class="o_priority o_priority_star o_report_replenish_change_priority fa fa-star#{' one' if line['move_out'].raw_material_production_id.priority=='1' else '-o zero'}"
                t-attf-model="mrp.production"
                t-att-model-id="line['move_out'].raw_material_production_id.id"
            />

        </xpath>
   </template>
</odoo>

```

## File: report\report_stock_reception.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import format_date


class ReceptionReport(models.AbstractModel):
    _inherit = 'report.stock.report_reception'

    def _get_formatted_scheduled_date(self, source):
        if source._name == 'mrp.production':
            return format_date(self.env, source.date_planned_start)
        return super()._get_formatted_scheduled_date(source)

```

## File: report\report_stock_reception.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- TODO: Delete this file -->
    <!-- Reception Report Labels -->
    <template id="report_reception_report_label_mrp">
        <t t-if="quantity" t-set="qtys" t-value="[int(q) for q in quantity.split(',')]"/>
        <t t-else="" t-set="qtys" t-value="[1 for q in range(len(docs))]"/>
        <t t-call="web.basic_layout">
            <div class="page">
                <t t-foreach="range(len(docs))" t-as="index">
                    <t t-set="mo" t-value="docs[index]"/>
                    <t t-set="qty" t-value="qtys[index]"/>
                    <t t-foreach="range(qty)" t-as="j">
                        <div class="o_label_page o_label_dymo">
                            <div t-esc="mo.name"/>
                            <div class="font-weight-bold" t-esc="mo.product_id.display_name"/>
                        </div>
                    </t>
                </t>
            </div>
        </t>
    </template>
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
from . import report_stock_forecasted
from . import report_stock_reception
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
access_mrp_routing_workcenter,mrp.routing.workcenter,model_mrp_routing_workcenter,mrp.group_mrp_user,1,0,0,0
access_mrp_bom,mrp.bom,model_mrp_bom,group_mrp_user,1,0,0,0
access_mrp_bom_line,mrp.bom.line,model_mrp_bom_line,group_mrp_user,1,0,0,0
access_mrp_bom_byproduct_user,mrp.bom.byproduct,model_mrp_bom_byproduct,mrp.group_mrp_user,1,0,0,0
access_mrp_production,mrp.production user,model_mrp_production,mrp.group_mrp_user,1,1,1,1
access_mrp_workcenter_manager,mrp.workcenter.manager,model_mrp_workcenter,mrp.group_mrp_manager,1,1,1,1
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
access_stock_picking_mrp_manager,stock.picking mrp_manager,stock.model_stock_picking,mrp.group_mrp_manager,1,1,1,1
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
access_change_production_qty,access.change.production.qty,model_change_production_qty,mrp.group_mrp_user,1,1,1,0
access_stock_warn_insufficient_qty_unbuild,access.stock.warn.insufficient.qty.unbuild,model_stock_warn_insufficient_qty_unbuild,mrp.group_mrp_user,1,1,1,0
access_mrp_production_backorder,access.mrp.production.backorder,model_mrp_production_backorder,mrp.group_mrp_user,1,1,1,0
access_mrp_production_backorder_line,access.mrp.production.backorder.line,model_mrp_production_backorder_line,mrp.group_mrp_user,1,1,1,0
access_mrp_consumption_warning,access.mrp.consumption.warning,model_mrp_consumption_warning,mrp.group_mrp_user,1,1,1,0
access_mrp_consumption_warning_line,access.mrp.consumption.warning.line,model_mrp_consumption_warning_line,mrp.group_mrp_user,1,1,1,0
access_mrp_immediate_production,access.mrp.immediate.production,model_mrp_immediate_production,mrp.group_mrp_user,1,1,1,0
access_mrp_immediate_production_line,access.mrp.immediate.production.line,model_mrp_immediate_production_line,mrp.group_mrp_user,1,1,1,0
access_mrp_workcenter_tag_group_user,access.mrp.workcenter.tag,model_mrp_workcenter_tag,mrp.group_mrp_user,1,0,0,0
access_mrp_workcenter_tag_manager,access.mrp.workcenter.tag,model_mrp_workcenter_tag,mrp.group_mrp_manager,1,1,1,1

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

    <record id="group_unlocked_by_default" model="res.groups">
        <field name="name">Unlocked by default</field>
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
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="mrp_unbuild_rule">
        <field name="name">mrp_unbuild multi-company</field>
        <field name="model_id" search="[('model','=','mrp.unbuild')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="mrp_workcenter_rule">
        <field name="name">mrp_workcenter multi-company</field>
        <field name="model_id" search="[('model','=','mrp.workcenter')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="mrp_workorder_rule">
        <field name="name">mrp_workorder multi-company</field>
        <field name="model_id" search="[('model','=','mrp.workorder')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="mrp_bom_rule">
        <field name="name">mrp_bom multi-company</field>
        <field name="model_id" search="[('model','=','mrp.bom')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="mrp_bom_line_rule">
        <field name="name">mrp_bom_line multi-company</field>
        <field name="model_id" search="[('model','=','mrp.bom.line')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="mrp_bom_byproduct_rule">
        <field name="name">mrp_bom_byproduct multi-company</field>
        <field name="model_id" search="[('model','=','mrp.bom.byproduct')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="mrp_routing_workcenter_rule">
        <field name="name">mrp_routing_workcenter multi-company</field>
        <field name="model_id" search="[('model','=','mrp.routing.workcenter')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="mrp_workcenter_productivity">
        <field name="name">mrp_workcenter_productivity multi-company</field>
        <field name="model_id" search="[('model','=','mrp.workcenter.productivity')]" model="ir.model"/>
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
/** @odoo-module **/

import AbstractField from 'web.AbstractField';
import { _t } from 'web.core';
import fields from 'web.basic_fields';
import fieldUtils from 'web.field_utils';
import field_registry from 'web.field_registry';
import time from 'web.time';

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

var TimeCounter = fields.FieldFloatTime.extend({

    init: function () {
        this._super.apply(this, arguments);
        this.duration = this.record.data.duration;
    },

    willStart: function () {
        var self = this;
        var def = this._rpc({
            model: 'mrp.workcenter.productivity',
            method: 'search_read',
            domain: [
                ['workorder_id', '=', this.record.data.id],
                ['date_end', '=', false],
            ],
        }).then(function (result) {
            var currentDate = new Date();
            var duration = 0;
            if (result.length > 0) {
                duration += self._getDateDifference(time.auto_str_to_date(result[0].date_start), currentDate);
            }
            var minutes = duration / 60 >> 0;
            var seconds = duration % 60;
            self.duration += minutes + seconds / 60;
            if (self.mode === 'edit') {
                self.value = self.duration;
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
        return moment(dateEnd).diff(moment(dateStart), 'seconds');
    },
    /**
     * @override
     */
    _renderReadonly: function () {
        if (this.record.data.is_user_working) {
            this._startTimeCounter();
        } else {
            this._super.apply(this, arguments);
        }
    },
    /**
     * @private
     */
    _startTimeCounter: function () {
        var self = this;
        clearTimeout(this.timer);
        if (this.record.data.is_user_working) {
            this.timer = setTimeout(function () {
                self.duration += 1/60;
                self._startTimeCounter();
            }, 1000);
        } else {
            clearTimeout(this.timer);
        }
        this.$el.text(fieldUtils.format.float_time(this.duration));
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

fieldUtils.format.mrp_time_counter = fieldUtils.format.float_time;

export default FieldEmbedURLViewer;

```

## File: static\src\js\mrp_bom_report.js

```javascript
/** @odoo-module **/

import core from 'web.core';
import framework from 'web.framework';
import stock_report_generic from 'stock.stock_report_generic';

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
                if (! self.given_context.searchVariant) {
                    self.given_context.searchVariant = result.is_variant_applied && Object.keys(result.variants)[0];
                }
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
      var productId = $parent.data('product_id');
      var level = $parent.data('level') || 0;
      return this._rpc({
              model: 'report.mrp.report_bom_structure',
              method: 'get_operations',
              args: [
                  productId,
                  activeID,
                  parseFloat(qty),
                  level + 1
              ]
          })
          .then(function (result) {
              self.render_html(event, $parent, result);
          });
    },
    get_byproducts: function(event) {
        var self = this;
        var $parent = $(event.currentTarget).closest('tr');
        var activeID = $parent.data('bom-id');
        var qty = $parent.data('qty');
        var level = $parent.data('level') || 0;
        var total = $parent.data('total') || 0;
        return this._rpc({
                model: 'report.mrp.report_bom_structure',
                method: 'get_byproducts',
                args: [
                    activeID,
                    parseFloat(qty),
                    level + 1,
                    parseFloat(total)
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
                $searchview: this.$searchView
            },
        };
        return this.updateControlPanel(status);
    },
    renderSearch: function () {
        this.$buttonPrint = $(QWeb.render('mrp.button', {'is_variant_applied': this.data.is_variant_applied}));
        this.$buttonPrint.find('.o_mrp_bom_print').on('click', this._onClickPrint.bind(this));
        this.$buttonPrint.find('.o_mrp_bom_print_all_variants').on('click', this._onClickPrint.bind(this));
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
        this.searchModelConfig.env.services.ui.block();
        var reportname = 'mrp.report_bom_structure?docids=' + this.given_context.active_id +
                         '&report_type=' + this.given_context.report_type +
                         '&quantity=' + (this.given_context.searchQty || 1);
        if (! $(ev.currentTarget).hasClass('o_mrp_bom_print_unfolded')) {
            reportname += '&childs=' + JSON.stringify(childBomIDs);
        }
        if ($(ev.currentTarget).hasClass('o_mrp_bom_print_all_variants')) {
            reportname += '&all_variants=' + 1;
        } else if (this.given_context.searchVariant) {
            reportname += '&variant=' + this.given_context.searchVariant;
        }
        var action = {
            'type': 'ir.actions.report',
            'report_type': 'qweb-pdf',
            'report_name': reportname,
            'report_file': 'mrp.report_bom_structure',
        };
        return this.do_action(action).then(() => {
            this.searchModelConfig.env.services.ui.unblock();
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
export default MrpBomReport;

```

## File: static\src\js\mrp_documents_controller_mixin.js

```javascript
/** @odoo-module **/

import { _t, qweb } from 'web.core';
import fileUploadMixin from 'web.fileUploadMixin';
import DocumentViewer from '@mrp/js/mrp_documents_document_viewer';

const MrpDocumentsControllerMixin = Object.assign({}, fileUploadMixin, {
    events: {
        'click .o_mrp_documents_kanban_upload': '_onClickMrpDocumentsUpload',
    },
    custom_events: Object.assign({}, fileUploadMixin.custom_events, {
        kanban_image_clicked: '_onKanbanPreview',
        upload_file: '_onUploadFile',
    }),

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Called right after the reload of the view.
     */
    async reload() {
        await this._renderFileUploads();
    },

    /**
     * @override
     * @param {jQueryElement} $node
     */
    renderButtons($node) {
        if (this.is_action_enabled('edit')) {
            this.$buttons = $(qweb.render('MrpDocumentsKanbanView.buttons'));
            this.$buttons.appendTo($node);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _getFileUploadRoute() {
        return '/mrp/upload_attachment';
    },

    /**
     * @override
     * @param {integer} param0.recordId
     */
    _makeFileUploadFormDataKeys() {
        const context = this.model.get(this.handle, { raw: true }).getContext();
        return {
            res_id: context.default_res_id,
            res_model: context.default_res_model,
        };
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickMrpDocumentsUpload() {
        const $uploadInput = $('<input>', {
            type: 'file',
            name: 'files[]',
            multiple: 'multiple'
        });
        $uploadInput.on('change', async ev => {
            await this._uploadFiles(ev.target.files);
            $uploadInput.remove();
        });
        $uploadInput.click();
    },

    /**
     * Handles custom event to display the document viewer.
     *
     * @private
     * @param {OdooEvent} ev
     * @param {integer} ev.data.recordID
     * @param {Array<Object>} ev.data.recordList
     */
    _onKanbanPreview(ev) {
        ev.stopPropagation();
        const documents = ev.data.recordList;
        const documentID = ev.data.recordID;
        const documentViewer = new DocumentViewer(this, documents, documentID);
        documentViewer.appendTo(this.$('.o_mrp_documents_kanban_view'));
    },

    /**
     * Specially created to call `_uploadFiles` method from tests.
     *
     * @private
     * @param {OdooEvent} ev
     */
    async _onUploadFile(ev) {
        await this._uploadFiles(ev.data.files);
    },

    /**
     * @override
     * @param {Object} param0
     * @param {XMLHttpRequest} param0.xhr
     */
    _onUploadLoad({ xhr }) {
        const result = xhr.status === 200
            ? JSON.parse(xhr.response)
            : {
                error: _.str.sprintf(_t("status code: %s, message: %s"), xhr.status, xhr.response)
            };
        if (result.error) {
            this.displayNotification({ title: _t("Error"), message: result.error, sticky: true });
        }
        fileUploadMixin._onUploadLoad.apply(this, arguments);
    },
});

export default MrpDocumentsControllerMixin;

```

## File: static\src\js\mrp_documents_document_viewer.js

```javascript
/** @odoo-module **/

import DocumentViewer from '@mail/js/document_viewer';

/**
 * This file defines the DocumentViewer for the MRP Documents Kanban view.
 */
const MrpDocumentsDocumentViewer = DocumentViewer.extend({
    init(parent, attachments, activeAttachmentID) {
        this._super(...arguments);
        this.modelName = 'mrp.document';
    },
});

export default MrpDocumentsDocumentViewer;

```

## File: static\src\js\mrp_documents_kanban_controller.js

```javascript
/** @odoo-module **/

/**
 * This file defines the Controller for the MRP Documents Kanban view, which is an
 * override of the KanbanController.
 */

import MrpDocumentsControllerMixin from '@mrp/js/mrp_documents_controller_mixin';

import KanbanController from 'web.KanbanController';

const MrpDocumentsKanbanController = KanbanController.extend(MrpDocumentsControllerMixin, {
    events: Object.assign({}, KanbanController.prototype.events, MrpDocumentsControllerMixin.events),
    custom_events: Object.assign({}, KanbanController.prototype.custom_events, MrpDocumentsControllerMixin.custom_events),

    /**
     * @override
    */
    init() {
        this._super(...arguments);
        MrpDocumentsControllerMixin.init.apply(this, arguments);
    },
    /**
     * Override to update the records selection.
     *
     * @override
    */
    async reload() {
        await this._super(...arguments);
        await MrpDocumentsControllerMixin.reload.apply(this, arguments);
    },
});

export default MrpDocumentsKanbanController;

```

## File: static\src\js\mrp_documents_kanban_record.js

```javascript
/** @odoo-module **/

/**
 * This file defines the KanbanRecord for the MRP Documents Kanban view.
 */

import KanbanRecord from 'web.KanbanRecord';

const MrpDocumentsKanbanRecord = KanbanRecord.extend({
    events: Object.assign({}, KanbanRecord.prototype.events, {
        'click .o_mrp_download': '_onDownload',
        'click .o_kanban_previewer': '_onImageClicked',
    }),

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Handles the click on the download link to save the attachment locally.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onDownload(ev) {
        ev.preventDefault();
        window.location = `/web/content/${this.modelName}/${this.id}/datas?download=true`;
    },

    /**
     * Handles the click on the preview image. Triggers up `_onKanbanPreview` to
     * display `DocumentViewer`.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onImageClicked(ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this.trigger_up('kanban_image_clicked', {
            recordList: [this.recordData],
            recordID: this.recordData.id
        });
    },
});

export default MrpDocumentsKanbanRecord;

```

## File: static\src\js\mrp_documents_kanban_renderer.js

```javascript
/** @odoo-module **/

/**
 * This file defines the Renderer for the MRP Documents Kanban view, which is an
 * override of the KanbanRenderer.
 */

import KanbanRenderer from 'web.KanbanRenderer';
import MrpDocumentsKanbanRecord from '@mrp/js/mrp_documents_kanban_record';

const MrpDocumentsKanbanRenderer = KanbanRenderer.extend({
    config: Object.assign({}, KanbanRenderer.prototype.config, {
        KanbanRecord: MrpDocumentsKanbanRecord,
    }),
    /**
     * @override
     */
    async start() {
        this.$el.addClass('o_mrp_documents_kanban_view');
        await this._super(...arguments);
    },
});

export default MrpDocumentsKanbanRenderer;

```

## File: static\src\js\mrp_document_kanban_view.js

```javascript
/** @odoo-module **/

import KanbanView from 'web.KanbanView';
import MrpDocumentsKanbanController from '@mrp/js/mrp_documents_kanban_controller';
import MrpDocumentsKanbanRenderer from '@mrp/js/mrp_documents_kanban_renderer';
import viewRegistry from 'web.view_registry';

const MrpDocumentsKanbanView = KanbanView.extend({
    config: Object.assign({}, KanbanView.prototype.config, {
        Controller: MrpDocumentsKanbanController,
        Renderer: MrpDocumentsKanbanRenderer,
    }),
});

viewRegistry.add('mrp_documents_kanban', MrpDocumentsKanbanView);

export default MrpDocumentsKanbanView;

```

## File: static\src\js\mrp_field_one2many_with_copy.js

```javascript
/** @odoo-module **/

import { FieldOne2Many } from 'web.relational_fields';
import ListRenderer from 'web.ListRenderer';
import fieldRegistry from 'web.field_registry';
import {_t} from 'web.core';

//----------------------------------------------------

var MrpFieldOne2ManyWithCopyListRenderer = ListRenderer.extend({

    /**
     * @override
     */
    init: function (parent, state, params) {
        this._super.apply(this, arguments);
        this.creates.push({
            string: parent.nodeOptions.copy_text,
            context: '',
        });
    },
});

var MrpFieldOne2ManyWithCopy = FieldOne2Many.extend({

    /**
     * @override
     */
    init: function (parent, name, record, options) {
        this._super.apply(this, arguments);
        this.nodeOptions = _.defaults(this.nodeOptions, {
            copy_text: _t('Copy Existing Operations'),
        });
    },
    /**
     * @override
     */
    _getRenderer: function () {
        if (this.view.arch.tag === 'tree') {
            return MrpFieldOne2ManyWithCopyListRenderer;
        }
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    _openFormDialog: function (params) {
        if (params.context === undefined) {
            return this._super.apply(this, arguments);
        }
        const parent = this.getParent();
        const parentIsNew = parent.state.res_id === undefined;
        const parentHasChanged = parent.state.isDirty();
        if (parentIsNew || parentHasChanged) {
            this.displayNotification({ message: _t('Please click on the "save" button first'), type: 'danger' });
            return;
        }
        this.do_action({
            name: _t('Select Operations to Copy'),
            type: 'ir.actions.act_window',
            res_model: 'mrp.routing.workcenter',
            views: [[false, 'list'], [false, 'form']],
            domain: ['|', ['bom_id', '=', false], ['bom_id.active', '=', true]],
            context: {
                tree_view_ref: 'mrp.mrp_routing_workcenter_copy_to_bom_tree_view',
                bom_id: this.recordData.id,
            },
        });

    },
});

fieldRegistry.add('mrp_one2many_with_copy', MrpFieldOne2ManyWithCopy);

export default MrpFieldOne2ManyWithCopy;

```

## File: static\src\js\mrp_should_consume.js

```javascript
/** @odoo-module **/

import { FieldFloat } from 'web.basic_fields';
import fieldRegistry from 'web.field_registry';
import field_utils from 'web.field_utils';
/**
 * This widget is used to display alongside the total quantity to consume of a production order,
 * the exact quantity that the worker should consume depending on the BoM. Ex:
 * 2 components to make 1 finished product.
 * The production order is created to make 5 finished product and the quantity producing is set to 3.
 * The widget will be '3.000 / 5.000'.
 */
const MrpShouldConsume = FieldFloat.extend({
    /**
     * @override
     */
    init: function (parent, name, params) {
        this._super.apply(this, arguments);
        this.displayShouldConsume = !['done', 'draft', 'cancel'].includes(params.data.state);
        this.should_consume_qty = field_utils.format.float(params.data.should_consume_qty, params.fields.should_consume_qty, this.nodeOptions);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Prefix the classic float field (this.$el) by a static value.
     *
     * @private
     * @param {float} [value] quantity to display before the input `el`
     * @param {bool} [edit] whether the field will be editable or readonly
     */
    _addShouldConsume: function (value, edit=false) {
        const $to_consume_container = $('<span class="o_should_consume"/>');
        if (edit) {
            $to_consume_container.addClass('o_row');
        }
        $to_consume_container.text(value + ' / ');
        this.setElement(this.$el.wrap($to_consume_container).parent());
    },

    /**
     * @private
     * @override
     */
    _renderEdit: function () {
        if (this.displayShouldConsume) {
            if (!this.$el.text().includes('/')) {
                this.$input = this.$el;
                this._addShouldConsume(this.should_consume_qty, true);
            }
            this._prepareInput(this.$input);
        } else {
            this._super.apply(this);
        }
    },
    /**
     * Resets the content to the formated value in readonly mode.
     *
     * @override
     * @private
     */
    _renderReadonly: function () {
        this.$el.text(this._formatValue(this.value));
        if (this.displayShouldConsume) {
            this._addShouldConsume(this.should_consume_qty);
        }
    },
});

fieldRegistry.add('mrp_should_consume', MrpShouldConsume);

export default {
    MrpShouldConsume: MrpShouldConsume,
};

```

## File: static\src\js\mrp_workorder_popover.js

```javascript
/** @odoo-module **/

import PopoverWidget from 'stock.popover_widget';
import fieldRegistry from 'web.field_registry';
import { _t } from 'web.core';


/**
 * Link to a Char field representing a JSON:
 * {
 *  'replan': <REPLAN_BOOL>, // Show the replan btn
 *  'color': '<COLOR_CLASS>', // Color Class of the icon (d-none to hide)
 *  'infos': [
 *      {'msg' : '<MESSAGE>', 'color' : '<COLOR_CLASS>'},
 *      {'msg' : '<MESSAGE>', 'color' : '<COLOR_CLASS>'},
 *      ... ]
 * }
 */
var MrpWorkorderPopover = PopoverWidget.extend({
    popoverTemplate: 'mrp.workorderPopover',
    title: _t('Scheduling Information'),

    _render: function () {
        this._super.apply(this, arguments);
        if (! this.$popover) {
          return;
        }
        var self = this;
        this.$popover.find('.action_replan_button').click(function (e) {
            self._onReplanClick(e);
        });
    },

    _onReplanClick:function (e) {
        var self = this;
        this._rpc({
            model: 'mrp.workorder',
            method: 'action_replan',
            args: [[self.res_id]]
        }).then(function () {
            self.trigger_up('reload');
        });
    },
});

fieldRegistry.add('mrp_workorder_popover', MrpWorkorderPopover);

export default MrpWorkorderPopover;

```

## File: static\src\xml\mrp.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="mrp.button">
        <div class="o_list_buttons o_mrp_bom_report_buttons">
            <button type="button" class="btn btn-primary o_mrp_bom_print">Print</button>
            <t t-if="is_variant_applied">
                <button type="button" class="btn btn-primary o_mrp_bom_print_all_variants">Print All Variants</button>
            </t>
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

    <div t-name="mrp.workorderPopover">
        <t t-foreach="infos" t-as="info">
            <i t-attf-class="fa fa-arrow-right mr-2 #{ info.color }"></i><t t-esc="info.msg"/><br/>
        </t>
        <button t-if="replan" class="btn btn-primary action_replan_button">Replan</button>
    </div>

</templates>

```

## File: static\src\xml\mrp_document_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<div t-name="MrpDocumentsKanbanView.buttons">
    <button type="button" t-attf-class="btn btn-primary o_mrp_documents_kanban_upload">
        Upload
    </button>
</div>
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
            <kanban js_class="mrp_documents_kanban" create="false">
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
                                <t t-set="binaryPreviewable"
                                   t-value="new RegExp('(image|video|application/pdf|text)').test(record.mimetype.value) &amp;&amp; record.type.raw_value === 'binary'"/>
                                <div t-attf-class="o_kanban_image_wrapper #{(webimage or binaryPreviewable) ? 'o_kanban_previewer' : ''}">
                                    <t t-set="webimage" t-value="new RegExp('image.*(gif|jpeg|jpg|png)').test(record.mimetype.value)"/>
                                    <div t-if="record.type.raw_value == 'url'" class="o_url_image fa fa-link fa-3x text-muted" aria-label="Image is a link"/>
                                    <img t-elif="webimage" t-attf-src="/web/image/#{record.ir_attachment_id.raw_value}" width="100" height="100" alt="Document" class="o_attachment_image"/>
                                    <div t-else="" class="o_image o_image_thumbnail" t-att-data-mimetype="record.mimetype.value"/>
                                </div>
                            </div>
                            <div class="o_kanban_details">
                                <div class="o_kanban_details_wrapper">
                                    <div class="o_kanban_record_title">
                                        <field name="name" class="o_text_overflow"/>
                                    </div>
                                    <div class="o_kanban_record_body">
                                      <field name="url" widget="url" attrs="{'invisible':[('type','=','binary')]}"/>
                                    </div>
                                    <div class="o_kanban_record_bottom">
                                        <span class="oe_kanban_bottom_left">
                                            <field name="priority" widget="priority"/>
                                        </span>
                                        <div class="oe_kanban_bottom_right">
                                            <field name="create_uid" widget="many2one_avatar_user"/>
                                        </div>
                                    </div>
                                    <div class="o_dropdown_kanban dropdown" tabindex="-1">
                                        <a class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" href="#" role="button" aria-label="Dropdown menu" title="Dropdown menu">
                                            <span class="fa fa-ellipsis-v"/>
                                        </a>
                                        <div class="dropdown-menu" role="menu" aria-labelledby="dLabel">
                                            <a t-if="widget.editable" type="edit" class="dropdown-item">Edit</a>
                                            <a t-if="widget.deletable" type="delete" class="dropdown-item">Delete</a>
                                            <a class="dropdown-item o_mrp_download">Download</a>
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

    <record id="view_mrp_document_form" model="ir.ui.view">
        <field name="name">mrp.document.form</field>
        <field name="model">mrp.document</field>
        <field name="arch" type="xml">
            <form string="Attachments">
                <sheet>
                    <label for="name"/>
                    <h1>
                        <field name="name"/>
                    </h1>
                    <group>
                        <group>
                            <field name="type"/>
                            <field name="datas" filename="name" attrs="{'invisible':[('type','=','url')]}"/>
                            <field name="url" widget="url" attrs="{'invisible':[('type','=','binary')]}"/>
                        </group>
                        <group string="Attached To" groups="base.group_no_one">
                            <field name="res_name"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                        </group>
                        <group string="History" groups="base.group_no_one" attrs="{'invisible':[('create_date','=',False)]}">
                            <label for="create_uid" string="Creation"/>
                            <div name="creation_div">
                                <field name="create_uid" readonly="1" class="oe_inline"/> on
                                <field name="create_date" readonly="1" class="oe_inline"/>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
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
                        <field name="allowed_operation_ids" invisible="1"/>
                        <field name="company_id"/>
                        <field name="product_id"/>
                        <field name="product_uom_category_id" invisible="1"/>
                        <label for="product_qty"/>
                        <div class="o_row">
                            <field name="product_qty"/>
                            <field name="product_uom_id" groups="uom.group_uom"/>
                        </div>
                        <field name="operation_id" groups="mrp.group_mrp_routings" options="{'no_quick_create':True,'no_create_edit':True}"/>
                        <field name="possible_bom_product_template_attribute_value_ids" invisible="1"/>
                        <field name="bom_product_template_attribute_value_ids" widget="many2many_tags" options="{'no_create': True}" groups="product.group_product_variant"/>
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
                            <button name="%(action_mrp_routing_time)d" type="action" class="oe_stat_button" icon="fa-clock-o" groups="mrp.group_mrp_routings">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_text">Routing<br/>Performance</span>
                                </div>
                            </button>
                            <button name="%(action_report_mrp_bom)d" type="action"
                                class="oe_stat_button" icon="fa-bars" string="Structure &amp; Cost"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="product_tmpl_id" context="{'default_detailed_type': 'product'}"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <field name="product_id" groups="product.group_product_variant" context="{'default_detailed_type': 'product'}"/>
                            <label for="product_qty" string="Quantity"/>
                            <div class="o_row">
                                <field name="product_qty"/>
                                <field name="product_uom_id" options="{'no_open':True,'no_create':True}" groups="uom.group_uom"/>
                            </div>
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
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True, 'no_open': True}"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Components" name="components">
                            <field name="bom_line_ids" widget="one2many" context="{'default_parent_product_tmpl_id': product_tmpl_id, 'default_product_id': False, 'default_company_id': company_id, 'default_bom_id': id}">
                                <tree string="Components" editable="bottom">
                                    <field name="company_id" invisible="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="product_id" context="{'default_detailed_type': 'product'}"/>
                                    <field name="product_tmpl_id" invisible="1"/>
                                    <button name="action_see_attachments" type="object" icon="fa-files-o" aria-label="Product Attachments" title="Product Attachments" class="float-right oe_read_only"/>
                                    <field name="attachments_count" class="text-left oe_read_only"
                                    string=" "/>
                                    <field name="product_qty"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="parent_product_tmpl_id" invisible="1" />
                                    <field name="product_uom_id" options="{'no_open':True,'no_create':True}" groups="uom.group_uom"/>
                                    <field name="possible_bom_product_template_attribute_value_ids" invisible="1"/>
                                    <field name="bom_product_template_attribute_value_ids" optional="hide" widget="many2many_tags" options="{'no_create': True}" attrs="{'column_invisible': [('parent.product_id', '!=', False)]}" groups="product.group_product_variant"/>
                                    <field name="allowed_operation_ids" invisible="1"/>
                                    <field name="operation_id" groups="mrp.group_mrp_routings" optional="hidden" attrs="{'column_invisible': [('parent.type','not in', ('normal', 'phantom'))]}" options="{'no_quick_create':True,'no_create_edit':True}"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Operations"
                            name="operations"
                            attrs="{'invisible': [('type', 'not in',('normal','phantom'))]}"
                            groups="mrp.group_mrp_routings">
                                <field name="operation_ids"
                                    attrs="{'invisible': [('type','not in',('normal','phantom'))]}"
                                    groups="mrp.group_mrp_routings"
                                    context="{'bom_id_invisible': True, 'default_company_id': company_id, 'default_bom_id': id, 'tree_view_ref': 'mrp.mrp_routing_workcenter_bom_tree_view'}" widget="mrp_one2many_with_copy"/>
                        </page>
                        <page string="By-products"
                            name="by_products"
                            attrs="{'invisible': [('type','!=','normal')]}"
                            groups="mrp.group_mrp_byproducts">
                            <field name="byproduct_ids"  context="{'form_view_ref' : 'mrp.mrp_bom_byproduct_form_view', 'default_company_id': company_id, 'default_bom_id': id}">
                                <tree string="By-products"  editable="top">
                                    <field name="company_id" invisible="1"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field name="product_id" context="{'default_detailed_type': 'product'}"/>
                                    <field name="product_qty"/>
                                    <field name="product_uom_id" groups="uom.group_uom"/>
                                    <field name="cost_share" optional="hide"/>
                                    <field name="allowed_operation_ids" invisible="1"/>
                                    <field name="operation_id" groups="mrp.group_mrp_routings" options="{'no_quick_create':True,'no_create_edit':True}"/>
                                    <field name="possible_bom_product_template_attribute_value_ids" invisible="1"/>
                                    <field name="bom_product_template_attribute_value_ids" optional="hide" widget="many2many_tags" options="{'no_create': True}" attrs="{'column_invisible': [('parent.product_id', '!=', False)]}" groups="product.group_product_variant"/>
                                </tree>
                           </field>
                       </page>
                        <page string="Miscellaneous" name="miscellaneous">
                            <group>
                                <group>
                                    <field name="ready_to_produce" attrs="{'invisible': [('type','=','phantom')]}" string="Manufacturing Readiness" widget="radio" groups="mrp.group_mrp_routings"/>
                                    <field name="consumption" attrs="{'invisible': [('type','=','phantom')]}" widget="radio"/>
                                </group>
                                <group>
                                    <field name="picking_type_id" attrs="{'invisible': [('type','=','phantom')]}" string="Operation" groups="stock.group_adv_location"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                    </sheet>
                    <div class="oe_chatter">
                         <field name="message_follower_ids"/>
                         <field name="message_ids" colspan="4" nolabel="1"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="mrp_bom_tree_view" model="ir.ui.view">
            <field name="name">mrp.bom.tree</field>
            <field name="model">mrp.bom</field>
            <field name="arch" type="xml">
                <tree string="Bill of Materials" sample="1">
                    <field name="active" invisible="1"/>
                    <field name="sequence" widget="handle"/>
                    <field name="product_tmpl_id"/>
                    <field name="code" optional="show"/>
                    <field name="type"/>
                    <field name="product_id" groups="product.group_product_variant" optional="hide"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show"/>
                    <field name="product_qty" optional="hide"/>
                    <field name="product_uom_id" groups="uom.group_uom" optional="hide" string="Unit of Measure"/>
                </tree>
            </field>
        </record>

        <record id="mrp_bom_kanban_view" model="ir.ui.view">
            <field name="name">mrp.bom.kanban</field>
            <field name="model">mrp.bom</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1">
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
            <field name="context">{'default_company_id': allowed_company_ids[0]}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No bill of materials found. Let's create one!
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
                <form string="Bill of Material line" create="0" edit="0">
                    <sheet>
                        <group>
                            <group string="Component">
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
                            <group string="Operation">
                                <field name="company_id" invisible="1"/>
                                <field name="sequence" groups="base.group_no_one"/>
                                <field name="allowed_operation_ids" invisible="1"/>
                                <field name="operation_id" groups="mrp.group_mrp_routings"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="template_open_bom" model="ir.actions.act_window">
            <field name="context">{'default_product_tmpl_id': active_id}</field>
            <field name="name">Bill of Materials</field>
            <field name="res_model">mrp.bom</field>
            <field name="domain">['|', ('product_tmpl_id', '=', active_id), ('byproduct_ids.product_id.product_tmpl_id', '=', active_id)]</field>
        </record>

        <record id="product_open_bom" model="ir.actions.act_window">
            <field name="context">{'default_product_id': active_id}</field>
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
                <tree string="Manufacturing Orders" js_class="lazy_column_list" default_order="priority desc, date_planned_start desc" multi_edit="1" sample="1" decoration-info="state == 'draft'">
                    <header>
                        <button name="button_plan" type="object" string="Plan"/>
                        <button name="do_unreserve" type="object" string="Unreserve"/>
                    </header>
                    <field name="priority" optional="show" widget="priority" nolabel="1"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="name" decoration-bf="1"/>
                    <field name="date_planned_start" readonly="1" optional="show" widget="remaining_days"/>
                    <field name="date_deadline" widget="remaining_days" attrs="{'invisible': [('state', 'in', ['done', 'cancel'])]}" optional="hide"/>
                    <field name="product_id" readonly="1" optional="show"/>
                    <field name="lot_producing_id" optional="hide"/>
                    <field name="bom_id" readonly="1" optional="hide"/>
                    <field name="activity_ids" string="Next Activity" widget="list_activity" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="user_id" optional="hide" widget="many2one_avatar_user"/>
                    <field name="components_availability_state" invisible="1" options='{"lazy": true}'/>
                    <field name="components_availability" options='{"lazy": true}'
                        attrs="{'invisible': [('state', 'not in', ['confirmed', 'progress'])]}"
                        optional="show"
                        decoration-success="reservation_state == 'assigned' or components_availability_state == 'available'"
                        decoration-warning="reservation_state != 'assigned' and components_availability_state in ('expected', 'available')"
                        decoration-danger="reservation_state != 'assigned' and components_availability_state == 'late'"/>
                    <field name="reservation_state" optional="hide" decoration-danger="reservation_state == 'confirmed'" decoration-success="reservation_state == 'assigned'"/>
                    <field name="product_qty" sum="Total Qty" string="Quantity" readonly="1" optional="show"/>
                    <field name="product_uom_id" string="UoM" options="{'no_open':True,'no_create':True}" groups="uom.group_uom" optional="show"/>
                    <field name="production_duration_expected" attrs="{'invisible': [('production_duration_expected', '=', 0)]}" groups="mrp.group_mrp_routings" widget="float_time" sum="Total expected duration" optional="show"/>
                    <field name="production_real_duration" attrs="{'invisible': [('production_real_duration', '=', 0)]}" groups="mrp.group_mrp_routings" widget="float_time" sum="Total real duration" optional="show"/>
                    <field name="company_id" readonly="1" groups="base.group_multi_company" optional="show"/>
                    <field name="state" 
                           decoration-success="state in ('done', 'to_close')"
                           decoration-warning="state == 'progress'"
                           decoration-info="state in ('confirmed', 'draft')"
                           optional="show" widget="badge"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                    <field name="delay_alert_date" invisible="1"/>
                    <field nolabel="1" name="json_popover" widget="stock_rescheduling_popover" attrs="{'invisible': [('json_popover', '=', False)]}"/>
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

        <record id="action_production_order_mark_done" model="ir.actions.server">
            <field name="name">Mark as Done</field>
            <field name="model_id" ref="mrp.model_mrp_production"/>
            <field name="binding_model_id" ref="mrp.model_mrp_production"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
            if records:
                res = records.filtered(lambda mo: mo.state in {'confirmed', 'to_close', 'progress'}).button_mark_done()
                if res is not True:
                    action = res
            </field>
        </record>

        <record id="mrp_production_form_view" model="ir.ui.view">
            <field name="name">mrp.production.form</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <form string="Manufacturing Orders">
                <header>
                    <field name="confirm_cancel" invisible="1"/>
                    <field name="show_lock" invisible="1"/>
                    <button name="button_mark_done" attrs="{'invisible': ['|', '|', ('state', 'in', ('draft', 'cancel', 'done', 'to_close')), ('qty_producing', '=', 0), ('move_raw_ids', '!=', [])]}" string="Validate" type="object" class="oe_highlight"
                            confirm="There are no components to consume. Are you still sure you want to continue?" data-hotkey="g"/>
                    <button name="button_mark_done" attrs="{'invisible': ['|', '|', ('state', 'in', ('draft', 'cancel', 'done', 'to_close')), ('qty_producing', '=', 0), ('move_raw_ids', '=', [])]}" string="Validate" type="object" class="oe_highlight" data-hotkey="g"/>
                    <button name="button_mark_done" attrs="{'invisible': [
                        '|',
                        ('move_raw_ids', '=', []),
                        '&amp;',
                        '|',
                        ('state', 'not in', ('confirmed', 'progress')),
                        ('qty_producing', '!=', 0),
                        ('state', '!=', 'to_close')]}" string="Mark as Done" type="object" class="oe_highlight" data-hotkey="g"/>
                    <button name="button_mark_done" attrs="{'invisible': [
                        '|',
                        ('move_raw_ids', '!=', []),
                        '&amp;',
                        '|',
                        ('state', 'not in', ('confirmed', 'progress')),
                        ('qty_producing', '!=', 0),
                        ('state', '!=', 'to_close')]}" string="Mark as Done" type="object" class="oe_highlight" data-hotkey="g"
                            confirm="There are no components to consume. Are you still sure you want to continue?"/>
                    <button name="action_confirm" attrs="{'invisible': [('state', '!=', 'draft')]}" string="Confirm" type="object" class="oe_highlight" data-hotkey="v"/>
                    <button name="button_plan" attrs="{'invisible': ['|', '|', ('state', 'not in', ('confirmed', 'progress', 'to_close')), ('workorder_ids', '=', []), ('is_planned', '=', True)]}" type="object" string="Plan" class="oe_highlight" data-hotkey="x"/>
                    <button name="button_unplan" type="object" string="Unplan" attrs="{'invisible': ['|', ('is_planned', '=', False), ('state', '=', 'cancel')]}" data-hotkey="x"/>
                    <button name="action_assign" attrs="{'invisible': ['|', ('state', 'in', ('draft', 'done', 'cancel')), ('reserve_visible', '=', False)]}" string="Check availability" type="object" data-hotkey="q"/>
                    <button name="do_unreserve" type="object" string="Unreserve" attrs="{'invisible': [('unreserve_visible', '=', False)]}" data-hotkey="w"/>
                    <button name="button_scrap" type="object" string="Scrap" attrs="{'invisible': [('state', 'in', ('cancel', 'draft'))]}" data-hotkey="y"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,confirmed,progress,done"/>
                    <button name="action_toggle_is_locked" attrs="{'invisible': ['|', ('show_lock', '=', False), ('is_locked', '=', False)]}" string="Unlock" groups="mrp.group_mrp_manager" type="object" help="Unlock the manufacturing order to adjust what has been consumed or produced." data-hotkey="l"/>
                    <button name="action_toggle_is_locked" attrs="{'invisible': ['|', ('show_lock', '=', False), ('is_locked', '=', True)]}" string="Lock" groups="mrp.group_mrp_manager" type="object" help="Lock the manufacturing order to prevent changes to what has been consumed or produced." data-hotkey="l"/>
                    <field name="show_serial_mass_produce" invisible="1"/>
                    <button name="action_serial_mass_produce_wizard" attrs="{'invisible': [('show_serial_mass_produce', '=', False)]}" string="Mass Produce" type="object"/>
                    <button name="action_cancel" type="object" string="Cancel" data-hotkey="z"
                            attrs="{'invisible': ['|', '|', ('id', '=', False), ('state', 'in', ('done', 'cancel')), ('confirm_cancel', '=', True)]}"/>
                    <button name="action_cancel" type="object" string="Cancel" data-hotkey="z"
                            attrs="{'invisible': ['|', '|', ('id', '=', False), ('state', 'in', ('done', 'cancel')), ('confirm_cancel', '=', False)]}"
                            confirm="Some product moves have already been confirmed, this manufacturing order can't be completely cancelled. Are you still sure you want to process ?"/>
                    <button name="button_unbuild" type="object" string="Unbuild" attrs="{'invisible': [('state', '!=', 'done')]}" data-hotkey="shift+v"/>
                </header>
                <sheet>
                    <field name="reservation_state" invisible="1"/>
                    <field name="date_planned_finished" invisible="1"/>
                    <field name="is_locked" invisible="1"/>
                    <field name="qty_produced" invisible="1"/>
                    <field name="unreserve_visible" invisible="1"/>
                    <field name="reserve_visible" invisible="1"/>
                    <field name="consumption" invisible="1"/>
                    <field name="is_planned" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button" name="action_view_mrp_production_childs" type="object" icon="fa-wrench" attrs="{'invisible': [('mrp_production_child_count', '=', 0)]}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="mrp_production_child_count"/></span>
                                <span class="o_stat_text">Child MO</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="action_view_mrp_production_sources" type="object" icon="fa-wrench" attrs="{'invisible': [('mrp_production_source_count', '=', 0)]}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="mrp_production_source_count"/></span>
                                <span class="o_stat_text">Source MO</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="action_view_mrp_production_backorders" type="object" icon="fa-wrench" attrs="{'invisible': [('mrp_production_backorder_count', '&lt;', 2)]}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="mrp_production_backorder_count"/></span>
                                <span class="o_stat_text">Backorders</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="action_see_move_scrap" type="object" icon="fa-arrows-v" attrs="{'invisible': [('scrap_count', '=', 0)]}">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="scrap_count"/></span>
                                <span class="o_stat_text">Scraps</span>
                            </div>
                        </button>
                        <button type="object" name="action_view_mo_delivery" class="oe_stat_button" icon="fa-truck"  groups="base.group_user" attrs="{'invisible': [('delivery_count', '=', 0)]}">
                            <field name="delivery_count" widget="statinfo" string="Transfers"/>
                        </button>
                        <button name="%(stock.action_stock_report)d" icon="fa-arrow-up" class="oe_stat_button" string="Traceability" type="action" states="done" groups="stock.group_production_lot"/>
                        <button name="%(action_mrp_production_moves)d" type="action" string="Product Moves" class="oe_stat_button" icon="fa-exchange" attrs="{'invisible': [('state', 'not in', ('progress', 'done'))]}"/>
                    </div>
                    <div class="oe_title">
                        <h1>
                            <field name="priority" widget="priority" class="mr-3"/>
                            <field name="name" placeholder="Manufacturing Reference" nolabel="1"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="id" invisible="1"/>
                            <field name="use_create_components_lots" invisible="1"/>
                            <field name="show_lot_ids" invisible="1"/>
                            <field name="product_tracking" invisible="1"/>
                            <field name="product_id" context="{'default_detailed_type': 'product'}" attrs="{'readonly': [('state', '!=', 'draft')]}" default_focus="1"/>
                            <field name="product_tmpl_id" invisible="1"/>
                            <field name="forecasted_issue" invisible="1"/>
                            <field name="product_description_variants" attrs="{'invisible': [('product_description_variants', 'in', (False, ''))], 'readonly': [('state', '!=', 'draft')]}"/>
                            <label for="product_qty" string="Quantity"/>
                            <div class="o_row no-gutters d-flex">
                                <div attrs="{'invisible': [('state', '=', 'draft')]}" class="o_row">
                                    <field name="qty_producing" class="text-left" attrs="{'readonly': ['|', ('state', '=', 'cancel'), '&amp;', ('state', '=', 'done'), ('is_locked', '=', True)]}"/>
                                    /
                                </div>
                                <field name="product_qty" class="oe_inline text-left" attrs="{'readonly': [('state', '!=', 'draft')], 'invisible': [('state', 'not in', ('draft', 'done'))]}"/>
                                <button type="action" name="%(mrp.action_change_production_qty)d"
                                    context="{'default_mo_id': id}" class="oe_link oe_inline" style="margin: 0px; padding: 0px;" attrs="{'invisible': ['|', ('state', 'in', ('draft', 'done','cancel')), ('id', '=', False)]}">
                                    <field name="product_qty" class="oe_inline" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                </button>
                                <label for="product_uom_id" string="" class="oe_inline"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" force_save="1" groups="uom.group_uom" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                <span class='font-weight-bold'>To Produce</span>
                                <button type="object" name="action_product_forecast_report" icon="fa-area-chart" attrs="{'invisible': [('forecasted_issue', '=', True)]}"/>
                                <button type="object" name="action_product_forecast_report" icon="fa-area-chart" attrs="{'invisible': [('forecasted_issue', '=', False)]}" class="text-danger"/>
                            </div>
                            <label for="lot_producing_id" attrs="{'invisible': ['|', ('state', '=', 'draft'), ('product_tracking', 'in', ('none', False))]}"/>
                            <div class="o_row" attrs="{'invisible': ['|', ('state', '=', 'draft'), ('product_tracking', 'in', ('none', False))]}">
                                <field name="lot_producing_id"
                                    context="{'default_product_id': product_id, 'default_company_id': company_id}" attrs="{'invisible': [('product_tracking', 'in', ('none', False))]}"/>
                                <button name="action_generate_serial" type="object" class="btn btn-primary fa fa-plus-square-o" aria-label="Creates a new serial/lot number" title="Creates a new serial/lot number" role="img" attrs="{'invisible': ['|', ('product_tracking', 'in', ('none', False)), ('lot_producing_id', '!=', False)]}"/>
                            </div>
                            <field name="bom_id"
                                context="{'default_product_tmpl_id': product_tmpl_id}" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                        </group>
                        <group name="group_extra_info">
                            <label for="date_planned_start"/>
                            <div class="o_row">
                                <field name="date_planned_start"
                                    attrs="{'readonly': [('state', 'in', ['done', 'cancel'])]}"
                                    decoration-warning="state not in ('done', 'cancel') and date_planned_start &lt; now"
                                    decoration-danger="state not in ('done', 'cancel') and date_planned_start &lt; current_date"
                                    decoration-bf="state not in ('done', 'cancel') and (date_planned_start &lt; current_date or date_planned_start &lt; now)"/>
                                <field name="delay_alert_date" invisible="1"/>
                                <field nolabel="1" name="json_popover" widget="stock_rescheduling_popover" attrs="{'invisible': [('json_popover', '=', False)]}"/>
                            </div>
                            <field name="components_availability_state" invisible="1"/>
                            <field name="components_availability" attrs="{'invisible': [('state', 'not in', ['confirmed', 'progress'])]}"
                                decoration-success="reservation_state == 'assigned' or components_availability_state == 'available'"
                                decoration-warning="reservation_state != 'assigned' and components_availability_state in ('expected', 'available')"
                                decoration-danger="reservation_state != 'assigned' and components_availability_state == 'late'"/>
                            <field name="user_id"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" attrs="{'readonly': [('state', '!=', 'draft')]}" force_save="1"/>
                            <field name="show_final_lots" invisible="1"/>
                            <field name="production_location_id" invisible="1" readonly="1"/>
                            <field name="move_finished_ids" invisible="1" attrs="{'readonly': ['|', ('state', '=', 'cancel'), '&amp;', ('state', '=', 'done'), ('is_locked', '=', True)]}">
                                <tree editable="bottom">
                                    <field name="product_id"/>
                                    <field name="product_uom_qty"/>
                                    <field name="product_uom"/>
                                    <field name="operation_id"/>
                                    <field name="byproduct_id"/>
                                    <field name="name"/>
                                    <field name="date_deadline"/>
                                    <field name="picking_type_id"/>
                                    <field name="location_id"/>
                                    <field name="location_dest_id"/>
                                    <field name="company_id"/>
                                    <field name="warehouse_id"/>
                                    <field name="origin"/>
                                    <field name="group_id"/>
                                    <field name="propagate_cancel"/>
                                    <field name="move_dest_ids"/>
                                    <field name="state"/>
                                    <!-- Useless as the editable in tree declaration -> For Form Test-->
                                    <field name="product_uom_category_id"/>
                                    <field name="allowed_operation_ids"/>
                                </tree>
                            </field>
                        </group>
                    </group>
                    <notebook>
                        <page string="Components" name="components">
                            <field name="move_raw_ids"
                                context="{'default_date': date_planned_start, 'default_date_deadline': date_deadline, 'default_location_id': location_src_id, 'default_location_dest_id': production_location_id, 'default_state': 'draft', 'default_raw_material_production_id': id, 'default_picking_type_id': picking_type_id, 'default_company_id': company_id}"
                                attrs="{'readonly': ['|', ('state', '=', 'cancel'), '&amp;', ('state', '=', 'done'), ('is_locked', '=', True)]}" options="{'delete': [('state', '=', 'draft')]}">
                                <tree default_order="is_done,sequence" editable="bottom">
                                    <field name="product_id" force_save="1" required="1" context="{'default_detailed_type': 'product'}" attrs="{'readonly': ['|', '|', ('move_lines_count', '&gt;', 0), ('state', '=', 'cancel'), '&amp;', ('state', '!=', 'draft'), ('additional', '=', False) ]}"/>
                                    <field name="location_id" string="From" readonly="1" groups="stock.group_stock_multi_locations" optional="show"/>
                                    <field name="move_line_ids" invisible="1">
                                        <tree>
                                            <field name="lot_id" invisible="1"/>
                                            <field name="owner_id" invisible="1"/>
                                            <field name="package_id" invisible="1"/>
                                            <field name="result_package_id" invisible="1"/>
                                            <field name="location_id" invisible="1"/>
                                            <field name="location_dest_id" invisible="1"/>
                                            <field name="qty_done" invisible="1"/>
                                            <field name="product_id" invisible="1"/>
                                            <field name="product_uom_id" invisible="1"/>
                                            <field name="product_uom_qty" invisible="1"/>
                                            <field name="state" invisible="1"/>
                                            <field name="move_id" invisible="1"/>
                                            <field name="id" invisible="1"/>
                                        </tree>
                                    </field>
                                    
                                    <field name="propagate_cancel" invisible="1"/>
                                    <field name="price_unit" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="name" invisible="1"/>
                                    <field name="allowed_operation_ids" invisible="1"/>
                                    <field name="unit_factor" invisible="1"/>
                                    <field name="date_deadline" invisible="1" force_save="1"/>
                                    <field name="date" invisible="1"/>
                                    <field name="additional" invisible="1"/>
                                    <field name="picking_type_id" invisible="1"/>
                                    <field name="has_tracking" invisible="1"/>
                                    <field name="operation_id" invisible="1"/>
                                    <field name="is_done" invisible="1"/>
                                    <field name="bom_line_id" invisible="1"/>
                                    <field name="sequence" invisible="1"/>
                                    <field name="location_id" invisible="1"/>
                                    <field name="warehouse_id" invisible="1"/>
                                    <field name="is_locked" invisible="1"/>
                                    <field name="move_lines_count" invisible="1"/>
                                    <field name="location_dest_id" domain="[('id', 'child_of', parent.location_dest_id)]" invisible="1"/>
                                    <field name="state" invisible="1" force_save="1"/>
                                    <field name="should_consume_qty" invisible="1"/>
                                    <field name="product_uom_qty" widget="mrp_should_consume" force_save="1" string="To Consume" attrs="{'readonly': ['&amp;', ('parent.state', '!=', 'draft'), '|', '&amp;', ('parent.state', 'not in', ('confirmed', 'progress', 'to_close')), ('parent.is_planned', '!=', True), '&amp;', ('state', '!=', 'draft'), ('parent.is_locked', '=', True)]}" width="1"/>
                                    <field name="product_uom" attrs="{'readonly': [('state', '!=', 'draft'), ('id', '!=', False)]}" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"/>
                                    <field name="product_type" invisible="1"/>
                                    <field name="product_qty" invisible="1" readonly="1"/>
                                    <field name="reserved_availability" invisible="1"/>
                                    <field name="forecast_expected_date" invisible="1"/>
                                    <!-- Button are used in state draft to doesn't have the name of the column "Reserved"-->
                                    <button type="object" name="action_product_forecast_report" icon="fa-area-chart" attrs="{'column_invisible': [('parent.state', '!=', 'draft')], 'invisible': [('forecast_availability', '&lt;', 0)]}"/>
                                    <button type="object" name="action_product_forecast_report" icon="fa-area-chart text-danger" attrs="{'column_invisible': [('parent.state', '!=', 'draft')], 'invisible': [('forecast_availability', '&gt;=', 0)]}"/>
                                    <field name="forecast_availability" string="Reserved" attrs="{'column_invisible': [('parent.state', 'in', ('draft', 'done'))]}" widget="forecast_widget"/>
                                    <field name="is_quantity_done_editable" invisible="1"/>
                                    <field name="quantity_done" string="Consumed"
                                        decoration-success="not is_done and (quantity_done - should_consume_qty == 0)"
                                        decoration-warning="not is_done and (quantity_done - should_consume_qty &gt; 0.0001)"
                                        attrs="{'column_invisible': [('parent.state', '=', 'draft')], 'readonly': [('has_tracking', '!=','none')]}"/>
                                    <field name="show_details_visible" invisible="1"/>
                                    <field name="lot_ids" widget="many2many_tags"
                                        groups="stock.group_production_lot" optional="hide"
                                        attrs="{'invisible': ['|', '|', ('show_details_visible', '=', False), ('has_tracking', '!=', 'serial'), ('parent.state', '=', 'draft')],
                                                'readonly': ['|', '|', ('show_details_visible', '=', False), ('has_tracking', '!=', 'serial'), ('parent.state', '=', 'draft')],
                                                'column_invisible': [('parent.show_lot_ids', '=',  False)]}"
                                        options="{'create': [('parent.use_create_components_lots', '!=', False)]}"
                                        context="{'default_company_id': company_id, 'default_product_id': product_id}"
                                        domain="[('product_id','=',product_id)]"
                                    />
                                    <field name="group_id" invisible="1"/>
                                    <button name="action_show_details" type="object" icon="fa-list" context="{'default_product_uom_qty': 0}" attrs="{'invisible': ['|', ('show_details_visible', '=', False), ('has_tracking', '=','none')]}" options="{&quot;warn&quot;: true}"/>
                                    <button class="o_optional_button" name="action_show_details" type="object" icon="fa-list" context="{'default_product_uom_qty': 0}" attrs="{'invisible': ['|', ('has_tracking', '!=','none'), ('show_details_visible', '=', False)]}" options="{&quot;warn&quot;: true}"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Work Orders" name="operations" groups="mrp.group_mrp_routings">
                            <field name="workorder_ids" attrs="{'readonly': [('state', 'in', ['cancel', 'done'])]}" context="{'tree_view_ref': 'mrp.mrp_production_workorder_tree_editable_view', 'default_product_uom_id': product_uom_id, 'from_manufacturing_order': True}"/>
                        </page>
                        <page string="By-Products" name="finished_products" groups="mrp.group_mrp_byproducts">
                            <field name="move_byproduct_ids" context="{'default_date': date_planned_finished, 'default_date_deadline': date_deadline, 'default_location_id': production_location_id, 'default_location_dest_id': location_dest_id, 'default_state': 'draft', 'default_production_id': id, 'default_picking_type_id': picking_type_id, 'default_company_id': company_id}" attrs="{'readonly': ['|', ('state', '=', 'cancel'), '&amp;', ('state', '=', 'done'), ('is_locked', '=', True)]}" options="{'delete': [('state', '=', 'draft')]}">
                                <tree default_order="is_done,sequence" decoration-muted="is_done" editable="bottom">
                                    <field name="byproduct_id" invisible="1"/>
                                    <field name="product_id" context="{'default_detailed_type': 'product'}" domain="[('id', '!=', parent.product_id)]" required="1"/>
                                    <field name="location_dest_id" string="To" readonly="1" groups="stock.group_stock_multi_locations"/>

                                    <field name="move_line_ids" invisible="1">
                                        <tree>
                                            <field name="lot_id" invisible="1"/>
                                            <field name="owner_id" invisible="1"/>
                                            <field name="package_id" invisible="1"/>
                                            <field name="result_package_id" invisible="1"/>
                                            <field name="location_id" invisible="1"/>
                                            <field name="location_dest_id" invisible="1"/>
                                            <field name="qty_done" invisible="1"/>
                                            <field name="product_id" invisible="1"/>
                                            <field name="product_uom_id" invisible="1"/>
                                            <field name="product_uom_qty" invisible="1"/>
                                            <field name="state" invisible="1"/>
                                            <field name="move_id" invisible="1"/>
                                            <field name="id" invisible="1"/>
                                        </tree>
                                    </field>

                                    <field name="company_id" invisible="1"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="name" invisible="1"/>
                                    <field name="allowed_operation_ids" invisible="1"/>
                                    <field name="unit_factor" invisible="1"/>
                                    <field name="date" invisible="1"/>
                                    <field name="additional" invisible="1"/>
                                    <field name="picking_type_id" invisible="1"/>
                                    <field name="has_tracking" invisible="1"/>
                                    <field name="operation_id" invisible="1"/>
                                    <field name="is_done" invisible="1"/>
                                    <field name="bom_line_id" invisible="1"/>
                                    <field name="sequence" invisible="1"/>
                                    <field name="location_id" invisible="1"/>
                                    <field name="warehouse_id" invisible="1"/>
                                    <field name="is_locked" invisible="1"/>
                                    <field name="move_lines_count" invisible="1"/>
                                    <field name="location_dest_id" domain="[('id', 'child_of', parent.location_dest_id)]" invisible="1"/>
                                    <field name="state" invisible="1" force_save="1"/>
                                    <field name="product_uom_qty" string="To Produce" force_save="1" attrs="{'readonly': ['&amp;', ('parent.state', '!=', 'draft'), '|', '&amp;', ('parent.state', 'not in', ('confirmed', 'progress', 'to_close')), ('parent.is_planned', '!=', True), ('parent.is_locked', '=', True)]}"/>
                                    <field name="is_quantity_done_editable" invisible="1"/>
                                    <field name="quantity_done" string="Produced" attrs="{'column_invisible': [('parent.state', '=', 'draft')], 'readonly': [('is_quantity_done_editable', '=', False)]}"/>
                                    <field name="product_uom" groups="uom.group_uom"/>
                                    <field name="cost_share" optional="hide"/>
                                    <field name="show_details_visible" invisible="1"/>
                                    <field name="lot_ids" widget="many2many_tags"
                                        groups="stock.group_production_lot"
                                        attrs="{'invisible': ['|', '|', ('show_details_visible', '=', False), ('has_tracking', '!=', 'serial'), ('parent.state', '=', 'draft')]}"
                                        options="{'create': [('parent.use_create_components_lots', '!=', False)]}"
                                        context="{'default_company_id': company_id, 'default_product_id': product_id}"
                                        domain="[('product_id','=',product_id)]"
                                    />
                                    <button name="action_show_details" type="object" icon="fa-list" attrs="{'invisible': ['|', ('has_tracking', '=','none'), ('show_details_visible', '=', False)]}" options="{&quot;warn&quot;: true}"/>
                                    <button class="o_optional_button" name="action_show_details" type="object" icon="fa-list" attrs="{'invisible': ['|', ('has_tracking', '!=','none'), ('show_details_visible', '=', False)]}" options="{&quot;warn&quot;: true}"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Miscellaneous" name="miscellaneous">
                            <group>
                                <group>
                                    <field name="picking_type_id" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                    <field name="location_src_id" groups="stock.group_stock_multi_locations" options="{'no_create': True}" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                    <field name="location_dest_id" groups="stock.group_stock_multi_locations" options="{'no_create': True}" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                                </group>
                                <group>
                                    <field name="origin"/>
                                    <field name="date_deadline"
                                        attrs="{'invisible': ['|', ('state', 'in', ('done', 'cancel')), ('date_deadline', '=', False)]}"
                                        decoration-danger="date_deadline and date_deadline &lt; current_date"
                                        decoration-bf="date_deadline and date_deadline &lt; current_date"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
                </form>
            </field>
        </record>

        <record id="mrp_production_kanban_view" model="ir.ui.view">
            <field name="name">mrp.production.kanban</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1">
                    <field name="name"/>
                    <field name="product_id"/>
                    <field name="product_qty"/>
                    <field name="product_uom_id" options="{'no_open':True,'no_create':True}"/>
                    <field name="date_planned_start"/>
                    <field name="state"/>
                    <field name="activity_state"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="o_kanban_record_top">
                                    <field name="priority" widget="priority"/>
                                    <div class="o_kanban_record_headings mt4 ml-1">
                                        <strong class="o_kanban_record_title"><span><t t-esc="record.product_id.value"/></span></strong>
                                    </div>
                                    <span class="float-right text-right"><t t-esc="record.product_qty.value"/> <small><t t-esc="record.product_uom_id.value"/></small></span>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left text-muted">
                                        <span><t t-esc="record.name.value"/> <t t-esc="record.date_planned_start.value and record.date_planned_start.value.split(' ')[0] or False"/></span>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                        <field name="delay_alert_date" invisible="1"/>
                                        <field nolabel="1" name="json_popover" widget="stock_rescheduling_popover" attrs="{'invisible': [('json_popover', '=', False)]}"/>
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
                          string="Manufacturing Orders" event_limit="5" quick_add="False">
                    <field name="user_id" avatar_field="avatar_128"/>
                    <field name="product_id"/>
                    <field name="product_qty"/>
                </calendar>
            </field>
        </record>

        <record id="view_production_pivot" model="ir.ui.view">
            <field name="name">mrp.production.pivot</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <pivot string="Manufacturing Orders" sample="1">
                    <field name="date_planned_start" type="row"/>
                </pivot>
            </field>
        </record>

        <record id="view_production_graph" model="ir.ui.view">
            <field name="name">mrp.production.graph</field>
            <field name="model">mrp.production</field>
            <field name="arch" type="xml">
                <graph string="Manufacturing Orders" stacked="0" sample="1">
                    <field name="date_planned_finished"/>
                    <field name="product_uom_qty" type="measure"/>
                    <field name="backorder_sequence" invisible="1"/>
                    <field name="qty_producing" string="Quantity Produced"/>
                    <field name="product_uom_qty" string= "Product Quantity"/>
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
                    <field name="name" string="Work Center" filter_domain="[('bom_id.operation_ids.workcenter_id', 'ilike', self)]"/>
                    <field name="origin"/>
                    <filter string="To Do" name="todo" domain="[('state', 'in', ('draft', 'confirmed', 'progress', 'to_close'))]"
                        help="Manufacturing Orders which are in confirmed state."/>
                    <filter string="Starred" name="starred" domain="[('priority', '=', '1')]"/>
                    <separator/>
                    <filter string="Draft" name="filter_draft" domain="[('state', '=', 'draft')]"/>
                    <filter string="Confirmed" name="filter_confirmed" domain="[('state', '=', 'confirmed')]"/>
                    <filter string="Planned" name="filter_planned" domain="[('is_planned', '=', True)]" groups="mrp.group_mrp_routings"/>
                    <filter string="In Progress" name="filter_in_progress" domain="[('state', '=', 'progress')]"/>
                    <filter string="To Close" name="filter_to_close" domain="[('state', '=', 'to_close')]"/>
                    <filter string="Done" name="filter_done" domain="[('state', '=', 'done')]"/>
                    <filter string="Cancelled" name="filter_cancel" domain="[('state', '=', 'cancel')]"/>
                    <separator/>
                    <filter string="Waiting" name="waiting" domain="[('reservation_state', 'in', ('waiting', 'confirmed'))]"/>
                    <filter string="Ready" name="filter_ready" domain="[('reservation_state', '=', 'assigned')]"/>
                    <separator/>
                    <filter string="Planning Issues" name="planning_issues" help="Late MO or Late delivery of components"
                        domain="['|', ('delay_alert_date', '!=', False), '&amp;', ('date_deadline', '&lt;', current_date), ('state', '=', 'confirmed')]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter name="filter_date_planned_start" string="Scheduled Date" date="date_planned_start"/>
                    <filter name="filter_plan_date" invisible="1" string="Scheduled Date: Last 365 Days" domain="[('date_planned_start', '>', (datetime.datetime.now() + relativedelta(days=-365)).to_utc().strftime('%Y-%m-%d %H:%M:%S'))]"/>
                    <separator/>
                    <filter string="Warnings" name="activities_exception"
                        domain="[('activity_exception_decoration', '!=', False)]"/>
                    <group expand="0" string="Group By...">
                        <filter string="Product" name="product" domain="[]" context="{'group_by': 'product_id'}"/>
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
                No manufacturing order found. Let's create one.
              </p><p>
                Consume <a name="%(product.product_template_action)d" type='action' tabindex="-1">components</a> and build finished products using <a name="%(mrp_bom_form_action)d" type='action' tabindex="-1">bills of materials</a>
              </p>
            </field>
        </record>

        <record id="mrp_production_action_picking_deshboard" model="ir.actions.act_window">
            <field name="name">Manufacturing Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.production</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_id" eval="False"/>
            <field name="search_view_id" ref="view_mrp_production_filter"/>
            <field name="domain">[('picking_type_id', '=', active_id)]</field>
            <field name="context">{'default_picking_type_id': active_id}</field>
        </record>

        <record id="mrp_production_action_unreserve_tree" model="ir.actions.server">
            <field name="name">Unreserve</field>
            <field name="model_id" ref="mrp.model_mrp_production"/>
            <field name="binding_model_id" ref="mrp.model_mrp_production"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
            if records:
                records.do_unreserve()
            </field>
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
            <field name="context">{
                'search_default_product': 1,
                'search_default_scheduled_date': 2,
                'search_default_filter_confirmed': True,
                'search_default_filter_planned': True,
                'default_company_id': allowed_company_ids[0],
                'allowed_company_ids': allowed_company_ids
            }</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p><p>
                    Create a new manufacturing order
                </p>
            </field>
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
            sequence="10"/>

        <record id="action_mrp_production_form" model="ir.actions.act_window">
            <field name="name">Manufacturing Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.production</field>
            <field name="view_mode">form</field>
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
                <tree string="Routing Work Centers" multi_edit="1">
                    <field name="name"/>
                    <field name="bom_id"/>
                    <field name="workcenter_id"/>
                    <field name="time_mode" optional="show"/>
                    <field name="time_computed_on" optional="hide"/>
                    <field name="time_cycle" widget="float_time" string="Duration (minutes)" width="1.5"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company"/>
                    <field name="possible_bom_product_template_attribute_value_ids" invisible="1"/>
                    <field name="bom_product_template_attribute_value_ids" optional="hide" widget="many2many_tags" options="{'no_create': True}" groups="product.group_product_variant"/>
                </tree>
            </field>
        </record>

        <record id="mrp_routing_workcenter_bom_tree_view" model="ir.ui.view">
            <field name="name">mrp.routing.workcenter.bom.tree</field>
            <field name="model">mrp.routing.workcenter</field>
            <field name="inherit_id" ref="mrp_routing_workcenter_tree_view"/>
            <field name="priority">1000</field>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="delete">0</attribute>
                </xpath>
                <xpath expr="//field[@name='name']" position="before">
                    <field name="sequence" widget="handle"/>
                </xpath>
                <xpath expr="//field[@name='bom_id']" position="replace"/>
                <xpath expr="//field[@name='time_cycle']" position="attributes">
                    <attribute name="sum">Total Duration</attribute>
                </xpath>
                <xpath expr="//field[@name='bom_product_template_attribute_value_ids']" position="attributes">
                    <attribute name="attrs">{'column_invisible': [('parent.product_id', '!=', False)]}</attribute>
                </xpath>
                <xpath expr="//field[@name='bom_product_template_attribute_value_ids']" position="after">
                    <button name="action_archive" class="btn-link" type="object" icon="fa-times"/>
                </xpath>
            </field>
        </record>

        <record id="mrp_routing_workcenter_copy_to_bom_tree_view" model="ir.ui.view">
            <field name="name">mrp.routing.workcenter.copy_to_bom.tree</field>
            <field name="model">mrp.routing.workcenter</field>
            <field name="inherit_id" ref="mrp_routing_workcenter_tree_view"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="create">0</attribute>
                    <attribute name="delete">0</attribute>
                    <attribute name="export_xlsx">0</attribute>
                    <attribute name="multi_edit">0</attribute>
                </xpath>
                <xpath expr="//field[@name='name']" position="before">
                    <header>
                        <button name="copy_to_bom" type="object" string="Copy selected operations"/>
                    </header>
                </xpath>
            </field>
        </record>

        <record id="mrp_routing_workcenter_form_view" model="ir.ui.view">
            <field name="name">mrp.routing.workcenter.form</field>
            <field name="model">mrp.routing.workcenter</field>
            <field name="arch" type="xml">
                <form string="Routing Work Centers">
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <group>
                            <group name="description">
                                <field name="active" invisible="1"/>
                                <field name="name"/>
                                <field name="sequence" invisible="1"/>
                                <field name="bom_id" invisible="context.get('bom_id_invisible', False)" domain="[]"/>
                                <field name="workcenter_id" context="{'default_company_id': company_id}"/>
                                <field name="possible_bom_product_template_attribute_value_ids" invisible="1"/>
                                <field name="bom_product_template_attribute_value_ids" widget="many2many_tags" options="{'no_create': True}" groups="product.group_product_variant"/>
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
                                <field name="company_id" groups="base.group_multi_company" />
                            </group>
                        </group>
                        <notebook>
                            <page string="Work Sheet" name="worksheet">
                                <group>
                                    <field name="worksheet_type" widget="radio"/>
                                    <field name="worksheet" help="Upload your PDF file." widget="pdf_viewer" attrs="{'invisible':  [('worksheet_type', '!=', 'pdf')], 'required':  [('worksheet_type', '=', 'pdf')]}"/>
                                    <field name="worksheet_google_slide" placeholder="Google Slide Link" widget="embed_viewer" attrs="{'invisible':  [('worksheet_type', '!=', 'google_slide')], 'required': [('worksheet_type', '=', 'google_slide')]}"/>
                                    <field name="note" attrs="{'invisible':  [('worksheet_type', '!=', 'text')]}"/>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="mrp_routing_action" model="ir.actions.act_window">
            <field name="name">Operations</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.routing.workcenter</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="mrp_routing_workcenter_tree_view"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new operation
              </p><p>
                Operation define that need to be done to realize a Work Order.
                Each operation is done at a specific Work Center and has a specific duration.
              </p>
            </field>
            <field name="domain">['|', ('bom_id', '=', False), ('bom_id.active', '=', True)]</field>
        </record>

        <record id="mrp_routing_workcenter_filter" model="ir.ui.view">
            <field name="name">mrp.routing.workcenter.filter</field>
            <field name="model">mrp.routing.workcenter</field>
            <field name="arch" type="xml">
                <search string="Operations Search Filters">
                    <field name="name"/>
                    <field name="bom_id"/>
                    <field name="workcenter_id"/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group>
                        <filter string="Bill of Material" name="bom" context="{'group_by': 'bom_id'}"/>
                        <filter string="Workcenter" name="workcenter" context="{'group_by': 'workcenter_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <menuitem id="menu_mrp_routing_action"
          action="mrp_routing_action"
          parent="menu_mrp_configuration"
          groups="group_mrp_routings"
          sequence="100"/>

    </data>
</odoo>

```

## File: views\mrp_unbuild_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--  Unbuild and scrap menu -->

   <record id="action_mrp_unbuild_moves" model="ir.actions.act_window">
        <field name="name">Stock Moves</field>
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
                            domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                            help="Show all records which has next action date is before today"/>
                        <filter invisible="1" string="Today Activities" name="activities_today"
                            domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                        <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                            domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    </group>
                    <group expand='0' string='Group by...'>
                        <filter string='Product' name="productgroup" context="{'group_by': 'product_id'}"/>
                        <filter string="Manufacturing Order" name="mogroup" context="{'group_by': 'mo_id'}"/>
                    </group>
               </search>
            </field>
        </record>

        <record id="mrp_unbuild_kanban_view" model="ir.ui.view">
            <field name="name">mrp.unbuild.kanban</field>
            <field name="model">mrp.unbuild</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1">
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
                                            <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'done': 'success'}}" readonly="1"/>
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
                        <button name="action_validate" string="Unbuild" type="object" states="draft" class="oe_highlight" data-hotkey="v"/>
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
                                <field name="product_id" attrs="{'readonly':['|', ('mo_id','!=',False), ('state', '=', 'done')]}" force_save="1"/>
                                <field name="mo_bom_id" invisible="1"/>
                                <field name="bom_id" attrs="{'required': [('mo_id', '=', False)], 'readonly':['|', ('mo_id','!=',False), ('state', '=', 'done')], 'invisible': [('mo_id', '!=', False), ('mo_bom_id', '=', False)]}" force_save="1"/>
                                <label for="product_qty"/>
                                <div class="o_row">
                                    <field name="product_qty" attrs="{'readonly': ['|', ('has_tracking', '=', 'serial'), ('state', '=', 'done')]}"/>
                                    <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom" attrs="{'readonly': ['|', ('mo_id', '!=', False), ('state', '=', 'done')]}" force_save="1"/>
                                </div>
                            </group>
                            <group>
                                <field name="mo_id"/>
                                <field name="location_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                <field name="location_dest_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                <field name="has_tracking" invisible="1"/>
                                <field name="lot_id" attrs="{'invisible': [('has_tracking', '=', 'none')], 'required': [('has_tracking', '!=', 'none')]}" groups="stock.group_production_lot"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                            </group>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids"/>
                        <field name="activity_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <!-- simplified version of unbuild form for unbuild button via manufacturing order,
             expects required fields to be filled in via 'default_' values -->
        <record id="mrp_unbuild_form_view_simplified" model="ir.ui.view">
            <field name="name">mrp.unbuild.form.simplified</field>
            <field name="model">mrp.unbuild</field>
            <field name="arch" type="xml">
                <form string="Unbuild Order">
                    <sheet>
                        <group>
                            <group>
                                <field name="state" invisible="1"/>
                                <field name="product_id" invisible="1"/>
                                <field name="bom_id" invisible="1"/>
                                <label for="product_qty"/>
                                <div class="o_row">
                                    <field name="product_qty" attrs="{'readonly': [('has_tracking', '=', 'serial')]}"/>
                                    <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom" attrs="{'readonly': [('mo_id', '!=', False)]}" force_save="1"/>
                                </div>
                            </group>
                            <group>
                                <field name="mo_id" invisible="1"/>
                                <field name="location_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                <field name="location_dest_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                <field name="has_tracking" invisible="1"/>
                                <field name="lot_id" attrs="{'invisible': [('has_tracking', '=', 'none')], 'required': [('has_tracking', '!=', 'none')]}" groups="stock.group_production_lot"/>
                                <field name="company_id" groups="base.group_multi_company" readonly="1"/>
                            </group>
                        </group>
                    </sheet>
                    <footer class="oe_edit_only">
                        <button name="action_validate" string="Unbuild" type="object" states="draft" class="oe_highlight" data-hotkey="q"/>
                        <button string="Discard" special="cancel" data-hotkey="z"/>
                    </footer>
                </form>
            </field>
        </record>


        <record id="mrp_unbuild_tree_view" model="ir.ui.view">
            <field name="name">mrp.unbuild.tree</field>
            <field name="model">mrp.unbuild</field>
            <field name="arch" type="xml">
                <tree sample="1">
                    <field name="name" decoration-bf="1"/>
                    <field name="product_id"/>
                    <field name="bom_id"/>
                    <field name="mo_id"/>
                    <field name="lot_id" groups="stock.group_production_lot"/>
                    <field name="product_qty"/>
                    <field name="product_uom_id" groups="uom.group_uom"/>
                    <field name="location_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="state" widget='badge' decoration-success="state == 'done'" decoration-info="state == 'draft'"/>
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
            No unbuild order found
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
    <menuitem id="menu_mrp_root"
        name="Manufacturing"
        groups="group_mrp_user,group_mrp_manager"
        web_icon="mrp,static/description/icon.png"
        sequence="145">

        <menuitem id="menu_mrp_manufacturing"
            name="Operations"
            sequence="10"/>

        <menuitem id="mrp_planning_menu_root"
            name="Planning"
            sequence="15"/>

        <menuitem id="menu_mrp_bom"
            name="Products"
            sequence="20"/>

        <menuitem id="menu_mrp_reporting"
              name="Reporting"
              sequence="25"/>

        <menuitem id="menu_mrp_configuration"
            name="Configuration"
            groups="group_mrp_manager"
            sequence="100"/>

    </menuitem>
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
            <field name="view_mode">tree,kanban,form</field>
            <field name="domain">[('bom_id', '!=', False), ('bom_id.operation_ids.workcenter_id', '=', active_id)]</field>
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
                    No work orders to do!
                </p><p>
                    Work orders are operations to do as part of a manufacturing order.
                    Operations are defined in the bill of materials or added in the manufacturing order directly.
                </p>
            </field>
        </record>

        <!-- Work Centers -->
        <record id="mrp_workcenter_tree_view" model="ir.ui.view">
            <field name="name">mrp.workcenter.tree</field>
            <field name="model">mrp.workcenter</field>
            <field name="arch" type="xml">
                <tree string="Work Center" multi_edit="1">
                    <field name="sequence" widget="handle"/>
                    <field name="name" optional="show"/>
                    <field name="code" optional="show"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'no_create': True, 'color_field': 'color'}" optional="show"/>
                    <field name="alternative_workcenter_ids" widget="many2many_tags" optional="show"/>
                    <field name="productive_time" optional="hide"/>
                    <field name="costs_hour" optional="show"/>
                    <field name="capacity" optional="show"/>
                    <field name="time_efficiency" optional="show"/>
                    <field name="oee_target" optional="show"/>
                    <field name="time_start" optional="hide"/>
                    <field name="time_stop" optional="hide"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
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
                <graph string="Workcenter Productivity" type="pie" sample="1">
                    <field name="loss_id"/>
                    <field name="duration" type="measure" string="Duration (minutes)"/>
                </graph>
            </field>
        </record>
        <record model="ir.actions.act_window" id="mrp_workcenter_productivity_report_oee">
            <field name="name">Overall Equipment Effectiveness</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.workcenter.productivity</field>
            <field name="view_id" eval="oee_pie_view"/>
            <field name="view_mode">graph,pivot,tree,form</field>
            <field name="domain">[('workcenter_id','=',active_id)]</field>
            <field name="context">{'search_default_thismonth':True}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
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
            <field name="context">{'search_default_workcenter': 1,
                                   'search_default_ready': True,
                                   'search_default_waiting': True,
                                   'search_default_pending': True,
                                   'search_default_progress': True,}</field>
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
                <kanban class="oe_background_grey o_kanban_dashboard o_workcenter_kanban" create="0" sample="1">
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
                                                            <field name="oee" widget="integer"/>%
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
                                        <div role="menuitem" class="text-right">
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
                            <field name="routing_line_ids" invisible="1"/>
                            <button string="Operations" type="object"
                                name="action_show_operations"
                                attrs="{'invisible': [('routing_line_ids', '=', [])]}"
                                context="{'default_workcenter_id': active_id}"
                                class="oe_stat_button" icon="fa-cog"/>
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
                            <button name="%(action_mrp_workcenter_load_report_graph)d" type="action" class="oe_stat_button" icon="fa-bar-chart"
                                context="{'search_default_workcenter_id': id,
                                          'search_default_ready': True,
                                          'search_default_waiting': True,
                                          'search_default_pending': True,
                                          'search_default_progress': True}">
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
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
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
                            <page string="General Information" name="general_info">
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
                Manufacturing operations are processed at Work Centers. A Work Center can be composed of
                workers and/or machines, they are used for costing, scheduling, capacity planning, etc.
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
                They can be defined via the configuration menu.
              </p>
            </field>
        </record>

        <menuitem id="menu_view_resource_search_mrp"
            action="mrp_workcenter_action"
            groups="group_mrp_routings"
            parent="menu_mrp_configuration"
            sequence="90"/>

    <record id="oee_loss_form_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.loss.form</field>
        <field name="model">mrp.workcenter.productivity.loss</field>
        <field name="arch" type="xml">
            <form string="Workcenter Productivity Loss">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="loss_id" options="{'no_open': True, 'no_create': True}"/>
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

    <record id="oee_search_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.search</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="arch" type="xml">
            <search string="Operations">
                <field name="workcenter_id"/>
                <field name="loss_id"/>
                <separator/>
                <filter name="availability" string="Availability Losses" domain="[('loss_type','=','availability')]"/>
                <filter name="performance" string="Performance Losses" domain="[('loss_type','=','performance')]"/>
                <filter name="quality" string="Quality Losses" domain="[('loss_type','=','quality')]"/>
                <filter name="productive" string="Fully Productive" domain="[('loss_type','=','productive')]"/>
                <filter name="filter_date_start" string="Date" date="date_start"/>
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
            <graph string="Workcenter Productivity" sample="1">
                <field name="workcenter_id"/>
                <field name="loss_id"/>
                <field name="duration" type="measure" string="Duration (minutes)"/>
            </graph>
        </field>
    </record>

    <record id="oee_pivot_view" model="ir.ui.view">
        <field name="name">mrp.workcenter.productivity.pivot</field>
        <field name="model">mrp.workcenter.productivity</field>
        <field name="arch" type="xml">
            <pivot string="Workcenter Productivity" sample="1">
                <field name="date_start" type="row" interval="day"/>
                <field name="loss_type" type="col"/>
                <field name="duration" type="measure" string="Duration (minutes)"/>
            </pivot>
        </field>
    </record>

    <record model="ir.actions.act_window" id="mrp_workcenter_productivity_report">
        <field name="name">Overall Equipment Effectiveness</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workcenter.productivity</field>
        <field name="view_mode">graph,pivot,tree,form</field>
        <field name="domain">[]</field>
        <field name="context">{'search_default_workcenter_group': 1, 'search_default_loss_group': 2, 'create':False,'edit':False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
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
                <filter string="Waiting" name="waiting" domain="[('state','=','waiting')]"/>
                <filter string="Pending" name="pending" domain="[('state','=','pending')]"/>
                <filter string="In Progress" name="progress" domain="[('state','=','progress')]"/>
                <filter string="Done" name="done" domain="[('state','=', 'done')]"/>
                <filter string="Late" name="late" domain="[('date_planned_start','&lt;=',time.strftime('%%Y-%%m-%%d'))]"/>
                <separator/>
                <filter string="Start Date" name="date_start_filter" date="date_start"/>
                <group expand="0" string="Group By...">
                    <filter string="Work center" name="workcenter" domain="[]" context="{'group_by': 'workcenter_id'}"/>
                    <filter string="Product" name="product" domain="[]" context="{'group_by': 'product_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_mrp_routing_time" model="ir.actions.act_window">
        <field name="name">Work Orders</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">graph,pivot,tree,form,gantt,calendar</field>
        <field name="context">{'search_default_done': True}</field>
        <field name="search_view_id" ref="view_mrp_production_work_order_search"/>
        <field name="domain">[('operation_id.bom_id', '=', active_id), ('state', '=', 'done')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p><p>
                Get statistics about the work orders duration related to this routing.
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
            No work orders to do!
          </p><p>
            Work orders are operations to do as part of a manufacturing order.
            Operations are defined in the bill of materials or added in the manufacturing order directly.
          </p>
        </field>
    </record>

    <record model="ir.ui.view" id="mrp_production_workorder_tree_editable_view">
        <field name="name">mrp.production.work.order.tree.editable</field>
        <field name="model">mrp.workorder</field>
        <field name="priority" eval="100"/>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="consumption" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="is_produced" invisible="1"/>
                <field name="is_user_working" invisible="1"/>
                <field name="product_uom_id" invisible="1" readonly="0"/>
                <field name="production_state" invisible="1"/>
                <field name="production_bom_id" invisible="1"/>
                <field name="qty_producing" invisible="1"/>
                <field name="time_ids" invisible="1"/>
                <field name="working_state" invisible="1"/>
                <field name="operation_id" invisible="1" domain="['|', ('bom_id', '=', production_bom_id), ('bom_id', '=', False)]" context="{'default_workcenter_id': workcenter_id, 'default_company_id': company_id}"/>
                <field name="name" string="Operation"/>
                <field name="workcenter_id"/>
                <field name="product_id" optional="show"/>
                <field name="date_planned_start" optional="show"/>
                <field name="date_planned_finished" optional="hide"/>
                <field name="date_start" optional="hide" readonly="1"/>
                <field name="date_finished" optional="hide" readonly="1"/>
                <field name="duration_expected" widget="float_time" sum="expected duration"/>
                <field name="duration" widget="mrp_time_counter"
                  attrs="{'invisible': [('production_state','=', 'draft')], 'readonly': [('is_user_working', '=', True)]}" sum="real duration"/>
                <field name="state" widget="badge" decoration-warning="state == 'progress'" decoration-success="state == 'done'" decoration-info="state not in ('progress', 'done', 'cancel')"
                  attrs="{'invisible': [('production_state', '=', 'draft')], 'column_invisible': [('parent.state', '=', 'draft')]}"/>
                <button name="button_start" type="object" string="Start" class="btn-success"
                  attrs="{'invisible': ['|', '|', '|', ('production_state','in', ('draft', 'done', 'cancel')), ('working_state', '=', 'blocked'), ('state', '=', 'done'), ('is_user_working', '!=', False)]}"/>
                <button name="button_pending" type="object" string="Pause" class="btn-warning"
                  attrs="{'invisible': ['|', '|', ('production_state', 'in', ('draft', 'done', 'cancel')), ('working_state', '=', 'blocked'), ('is_user_working', '=', False)]}"/>
                <button name="button_finish" type="object" string="Done" class="btn-success"
                  attrs="{'invisible': ['|', '|', ('production_state', 'in', ('draft', 'done', 'cancel')), ('working_state', '=', 'blocked'), ('is_user_working', '=', False)]}"/>
                <button name="%(mrp.act_mrp_block_workcenter_wo)d" type="action" string="Block" context="{'default_workcenter_id': workcenter_id}" class="btn-danger"
                  attrs="{'invisible': ['|', ('production_state', 'in', ('draft', 'done', 'cancel')), ('working_state', '=', 'blocked')]}"/>
                <button name="button_unblock" type="object" string="Unblock" context="{'default_workcenter_id': workcenter_id}" class="btn-danger"
                  attrs="{'invisible': ['|', ('production_state', 'in', ('draft', 'done', 'cancel')), ('working_state', '!=', 'blocked')]}"/>
                <button name="action_open_wizard" type="object" icon="fa-external-link" class="oe_edit_only"
                    context="{'default_workcenter_id': workcenter_id}"/>
                <field name="show_json_popover" invisible="1"/>
                <field name="json_popover" widget="mrp_workorder_popover" string=" " width="0.1" attrs="{'invisible': [('show_json_popover', '=', False)]}"/>
            </tree>
        </field>
    </record>

    <record id="mrp_production_workorder_tree_view" model="ir.ui.view">
        <field name="name">mrp.production.work.order.tree</field>
        <field name="model">mrp.workorder</field>
        <field name="mode">primary</field>
        <field name="priority" eval="10"/>
        <field name="inherit_id" ref="mrp.mrp_production_workorder_tree_editable_view"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="create">0</attribute>
                <attribute name="sample">1</attribute>
            </xpath>
            <field name="workcenter_id" position="after">
                <field name="production_id"/>
            </field>
            <field name="state" position="attributes">
                <attribute name="attrs">{'invisible': [('production_state', '=', 'draft')]}</attribute>
            </field>
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
            <header>
                <field name="state" widget="statusbar" statusbar_visible="pending,waiting,ready,progress,done"/>
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
                <field name="workcenter_id" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="product_tracking" invisible="1"/>
                <field name="product_id" invisible="1"/>
                <field name="finished_lot_id" invisible="1"/>
                <field name="qty_producing" invisible="1"/>
                <group>
                    <group attrs="{'invisible': [('date_planned_start', '=', False)]}">
                        <label for="date_planned_start" string="Planned Date"/>
                        <div class="oe_inline">
                            <field name="date_planned_start" class="mr8 oe_inline" required="True"/>
                            <strong class="mr8 oe_inline">to</strong>
                            <field name="date_planned_finished" class="oe_inline" required="True"/>
                            <field name="show_json_popover" invisible="1"/>
                            <field name="json_popover" widget="mrp_workorder_popover" class="oe_inline mx-2" attrs="{'invisible': [('show_json_popover', '=', False)]}"/>
                        </div>
                        <label for="duration_expected"/>
                        <div class="o_row">
                            <field name="duration_expected" widget="float_time"/>
                            <span>minutes</span>
                        </div>
                    </group>
                    <group>
                        <field name="production_id"/>
                    </group>
                </group>
                <notebook>
                <page string="Components" name="components">
                    <field name="move_raw_ids" readonly="1">
                        <tree>
                            <field name="state" invisible="1"/>
                            <field name="product_type" invisible="1"/>
                            <field name="product_id"/>
                            <field name="product_qty" string="To Consume"/>
                            <field name="reserved_availability" string ="Reserved"/>
                            <field name="quantity_done" string="Consumed"/>
                            <field name="product_qty_available" string="On Hand" attrs="{'invisible': [('product_type', '!=', 'product')]}"/>
                            <field name="product_virtual_available" string="Forecasted" attrs="{'invisible': [('product_type', '!=', 'product')]}"/>
                        </tree>
                    </field>
                </page>
                <page string="Time Tracking" name="time_tracking" groups="mrp.group_mrp_manager">
                    <group>
                        <field name="time_ids" nolabel="1" context="{'default_workcenter_id': workcenter_id, 'default_workorder_id': id}">
                            <tree editable="bottom">
                                <field name="date_start"/>
                                <field name="date_end"/>
                                <field name="duration" widget="float_time" sum="Total duration"/>
                                <field name="user_id"/>
                                <field name="workcenter_id" invisible="1"/>
                                <field name="company_id" invisible="1"/>
                                <field name="loss_id" string="Productivity" optional="hide"/>
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
                <page string="Work Instruction" name="workorder_page_work_instruction" attrs="{'invisible': [('worksheet', '=', False), ('worksheet_google_slide', '=', False), ('operation_note', '=', False)]}">
                    <field name="worksheet_type" invisible="1"/>
                    <field name="worksheet" widget="pdf_viewer" attrs="{'invisible': [('worksheet_type', '!=', 'pdf')]}"/>
                    <field name="worksheet_google_slide" widget="embed_viewer" attrs="{'invisible': [('worksheet_type', '!=', 'google_slide')]}"/>
                    <field name="operation_note" attrs="{'invisible': [('worksheet_type', '!=', 'text')]}"/>
                </page>
                </notebook>
            </sheet>
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
                <filter string="Waiting" name="waiting" domain="[('state', '=', 'waiting')]"/>
                <filter string="Pending" name="pending" domain="[('state', '=', 'pending')]"/>
                <filter string="Finished" name="finish" domain="[('state', '=', 'done')]"/>
                <separator/>
                <filter string="Late" name="late" domain="['&amp;', ('date_planned_start', '&lt;', current_date), ('state', '=', 'ready')]"
                    help="Production started late"/>
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
            <calendar date_stop="date_planned_finished" date_start="date_planned_start" string="Operations" color="workcenter_id" event_limit="5" delete="0" create="0">
                <field name="workcenter_id" filters="1"/>
                <field name="production_id"/>
                <field name="state"/>
            </calendar>
        </field>
    </record>

    <record id="workcenter_line_graph" model="ir.ui.view">
        <field name="name">mrp.production.work.order.graph</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <graph string="Operations" stacked="0" sample="1">
                <field name="production_id"/>
                <field name="duration" type="measure" string="Duration (minutes)"/>
                <field name="duration_unit" type="measure"/>
                <field name="duration_expected" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="workcenter_line_pivot" model="ir.ui.view">
        <field name="name">mrp.production.work.order.pivot</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <pivot string="Operations" sample="1">
                <field name="date_start"/>
                <field name="operation_id"/>
                <field name="duration" type="measure" string="Duration (minutes)" widget="float_time"/>
                <field name="duration_unit" type="measure" widget="float_time"/>
                <field name="duration_expected" type="measure" widget="float_time"/>
            </pivot>
        </field>
    </record>

    <record id="workcenter_line_gantt_production" model="ir.ui.view">
        <field name="name">mrp.production.work.order.gantt.production</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <gantt class="o_mrp_workorder_gantt" date_stop="date_planned_finished" date_start="date_planned_start" string="Operations" default_group_by="production_id" create="0" delete="0"
                progress="progress" plan="0"
                decoration-danger="json_popover and 'text-danger' in json_popover"
                decoration-success="state == 'done'"
                decoration-warning="state == 'cancel' or (json_popover and 'text-warning' in json_popover)"
                color="workcenter_id"
                display_unavailability="1"
                sample="1"
                form_view_id="%(mrp_production_workorder_form_view_inherit)d">

                <field name="date_planned_start"/>
                <field name="state"/>
                <field name="workcenter_id"/>
                <field name="json_popover"/>

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
                delete="0" sample="1"
                progress="progress" plan="0"
                decoration-danger="json_popover and 'text-danger' in json_popover"
                decoration-success="state == 'done'"
                decoration-warning="state == 'cancel' or (json_popover and 'text-warning' in json_popover)"
                color="production_id"
                display_unavailability="1"
                form_view_id="mrp_production_workorder_form_view_inherit">

                <field name="date_planned_start"/>
                <field name="state"/>
                <field name="workcenter_id"/>
                <field name="json_popover"/>

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
            <kanban class="oe_background_grey o_kanban_dashboard o_mrp_workorder_kanban" create="0" sample="1">
                <field name="name"/>
                <field name="production_id"/>
                <field name="state" readonly="1"/>
                <field name="is_user_working"/>
                <field name="working_user_ids"/>
                <field name="last_working_user_id"/>
                <field name="working_state"/>
                <field name="workcenter_id"/>
                <field name="product_id"/>
                <field name="qty_production"/>
                <field name="product_uom_id" force_save="1"/>
                <field name="operation_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click">
                            <div class="o_kanban_card_header o_kanban_record_top">
                                <div class="o_kanban_record_headings o_kanban_card_header_title">
                                    <strong class="o_primary">
                                        <span><t t-esc="record.production_id.value"/></span> - <span><t t-esc="record.name.value"/></span>
                                    </strong>
                                </div>
                                <div class="o_kanban_manage_button_section">
                                    <h2 class="ml8">
                                        <span t-attf-class="badge #{['pending', 'waiting'].indexOf(record.state.raw_value) > -1 ? 'badge-warning' :['progress'].indexOf(record.state.raw_value) > -1 ? 'badge-secondary' : ['ready'].indexOf(record.state.raw_value) > -1 ? 'badge-primary' : ['done'].indexOf(record.state.raw_value) > -1 ? 'badge-success' : ['cancel'].indexOf(record.state.raw_value) > -1 ? 'badge-muted' : 'badge-danger'}">
                                            <t t-esc="record.state.value"/>
                                        </span>
                                    </h2>
                                </div>
                            </div>
                            <div class="o_kanban_record_bottom">
                                <h5 class="oe_kanban_bottom_left">
                                    <span><t t-esc="record.product_id.value"/>, </span> <span><t t-esc="record.qty_production.value"/> <t t-esc="record.product_uom_id.value"/></span>
                                </h5>
                                <div class="oe_kanban_bottom_right" t-if="record.state.raw_value == 'progress'">
                                    <span t-if="record.working_state.raw_value != 'blocked' and record.working_user_ids.raw_value.length > 0"><i class="fa fa-play" role="img" aria-label="Run" title="Run"/></span>
                                    <span t-if="record.working_state.raw_value != 'blocked' and record.working_user_ids.raw_value.length == 0 and record.last_working_user_id.raw_value"><i class="fa fa-pause" role="img" aria-label="Pause" title="Pause"/></span>
                                    <span t-if="record.working_state.raw_value == 'blocked' and (record.working_user_ids.raw_value.length == 0 or record.last_working_user_id.raw_value)"><i class="fa fa-stop" role="img" aria-label="Stop" title="Stop"/></span>
                                    <t t-if="record.last_working_user_id.raw_value">
                                        <img t-att-src="kanban_image('res.users', 'avatar_128', record.last_working_user_id.raw_value)" class="oe_kanban_avatar" alt="Avatar"/>
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
        <field name="context">{'search_default_work_center': True, 'search_default_ready': True, 'search_default_waiting': True, 'search_default_progress': True, 'search_default_pending': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            No work orders to do!
          </p><p>
            Work orders are operations to do as part of a manufacturing order.
            Operations are defined in the bill of materials or added in the manufacturing order directly.
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
        <field name="context">{'search_default_production': True, 'search_default_ready': True, 'search_default_waiting': True, 'search_default_progress': True, 'search_default_pending': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            No work orders to do!
          </p><p>
            Work orders are operations to do as part of a manufacturing order.
            Operations are defined in the bill of materials or added in the manufacturing order directly.
          </p>
        </field>
    </record>

    <record model="ir.actions.act_window" id="mrp_workorder_mrp_production_form">
        <field name="name">Work Orders</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="view_id" ref="mrp_production_workorder_form_view_inherit"/>
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
            No work orders to do!
          </p><p>
            Work orders are operations to do as part of a manufacturing order.
            Operations are defined in the bill of materials or added in the manufacturing order directly.
          </p>
        </field>
    </record>

    <record id="view_workcenter_load_pivot" model="ir.ui.view">
        <field name="name">report.workcenter.load.pivot</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <pivot string="Work Center Loads" sample="1">
                <field name="duration_expected" type="measure" string="Expected Duration (minutes)"/>
                <field name="workcenter_id" type="row"/>
                <field name="production_date" type="row" interval="day"/>
            </pivot>
        </field>
    </record>

    <record id="view_work_center_load_graph" model="ir.ui.view">
        <field name="name">report.workcenter.load.graph</field>
        <field name="model">mrp.workorder</field>
        <field name="arch" type="xml">
            <graph string="Work Center load" sample="1">
                <field name="production_date" interval="day"/>
                <field name="workcenter_id"/>
                <field name="duration_expected" type="measure" string="Expected Duration (minutes)"/>
            </graph>
        </field>
    </record>

    <record id="action_mrp_workcenter_load_report_graph" model="ir.actions.act_window">
        <field name="name">Work Center Loads</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">mrp.workorder</field>
        <field name="view_mode">graph,pivot</field>
        <field name="view_id" ref="view_workcenter_load_pivot"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
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
                <xpath expr="//label[@for='sale_delay']" position="before">
                    <label for="produce_delay" string="Manuf. Lead Time" attrs="{'invisible':[('type','=','service')]}"/>
                    <div attrs="{'invisible':[('type','=','service')]}">
                        <field name="produce_delay" class="oe_inline"/> days
                    </div>
                </xpath>
                <xpath expr="//field[@name='product_variant_count']" position="after">
                    <field name="is_kits" invisible="1"/>
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
            <field name="context">{"search_default_consumable": 1, 'default_detailed_type': 'product'}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No product found. Let's create one!
              </p><p>
                Define the components and finished products you wish to use in
                bill of materials and manufacturing orders.
              </p>
            </field>
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
                        <div class="row mt16 o_settings_container" name="process_operations_setting_container">
                            <div class="col-lg-6 col-12 o_setting_box" id="work_order" title="Work Order Operations allow you to create and manage the manufacturing operations that should be followed within your work centers in order to produce a product. They are attached to bills of materials that will define the required components.">
                                <div class="o_setting_left_pane">
                                    <field name="group_mrp_routings"/>
                                    <field name="module_mrp_workorder" invisible="1"/>
                                </div>
                                <div class="o_setting_right_pane" id="workorder_settings">
                                    <label for="group_mrp_routings" string="Work Orders"/>
                                    <a href="https://www.odoo.com/documentation/15.0/applications/inventory_and_mrp/manufacturing/management/bill_configuration.html#adding-a-routing" title="Documentation" class="o_doc_link" target="_blank"></a>
                                    <div class="text-muted">
                                        Process operations at specific work centers
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('group_mrp_routings','=',False)]}">
                                        <div class="mt8">
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
                                    <a href="https://www.odoo.com/documentation/15.0/applications/inventory_and_mrp/manufacturing/management/subcontracting.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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
                                    <div class="row mt-2" attrs="{'invisible': [('module_quality_control','=',False)]}">
                                        <field name="module_quality_control_worksheet" widget="upgrade_boolean" class="col-lg-1 ml16 mr0"/>
                                        <div class="col pl-0">
                                            <label for="module_quality_control_worksheet"/>
                                            <div class="text-muted">
                                                Create customizable worksheets for your quality checks
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-lg-6 col-12 o_setting_box" id="mrp_lock" title="Makes confirmed manufacturing orders locked rather than unlocked by default. This only applies to new manufacturing orders, not previously created ones.">
                                <div class="o_setting_left_pane">
                                    <field name="group_unlocked_by_default"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_unlocked_by_default"/>
                                    <div class="text-muted">
                                        Allow manufacturing users to modify quantities to consume, without the need for prior approval
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
                                    <a href="https://www.odoo.com/documentation/15.0/applications/inventory_and_mrp/manufacturing/management/use_mps.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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
                                    <a href="https://www.odoo.com/documentation/15.0/applications/inventory_and_mrp/inventory/management/planning/scheduled_dates.html" title="Documentation" class="mr-2 o_doc_link" target="_blank"></a>
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

        <record id="view_stock_move_operations_raw" model="ir.ui.view">
            <field name="name">stock.move.operations.raw.form</field>
            <field name="model">stock.move</field>
            <field name="priority">1</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="stock.view_stock_move_operations" />
            <field name="arch" type="xml">
                <xpath expr="//label[@for='product_uom_qty']" position="attributes">
		    <attribute name="string">Total To Consume</attribute>
                </xpath>
                <xpath expr="//label[@for='quantity_done']" position="attributes">
		    <attribute name="string">Consumed</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_stock_move_operations_finished" model="ir.ui.view">
            <field name="name">stock.move.operations.finished.form</field>
            <field name="model">stock.move</field>
            <field name="priority">1</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="stock.view_stock_move_operations" />
            <field name="arch" type="xml">
                <xpath expr="//label[@for='product_uom_qty']" position="attributes">
		    <attribute name="string">To Produce</attribute>
                </xpath>
                <xpath expr="//label[@for='quantity_done']" position="attributes">
		    <attribute name="string">Produced</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_stock_move_line_operation_tree_finished" model="ir.ui.view">
            <field name="name">stock.move.line.operation.tree.finished</field>
            <field name="model">stock.move.line</field>
            <field name="inherit_id" ref="stock.view_stock_move_line_operation_tree" />
            <field name="arch" type="xml">
                <xpath expr="//field[@name='lot_id']" position="attributes">
		            <attribute name="context">{
                        'active_mo_id': context.get('active_mo_id'),
                        'active_picking_id': picking_id,
                        'default_company_id': parent.company_id,
                        'default_product_id': parent.product_id,
                        }
                    </attribute>
                </xpath>
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
            name="Scrap"
            parent="menu_mrp_manufacturing"
            action="stock.action_stock_scrap"
            sequence="25"/>

    <menuitem id="menu_procurement_compute_mrp"
        action="stock.action_procurement_compute"
        parent="mrp_planning_menu_root"
        sequence="135"/>

</odoo>

```

## File: views\stock_orderpoint_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_warehouse_orderpoint_tree_editable_inherited_purchase" model="ir.ui.view">
        <field name="name">stock.warehouse.orderpoint.tree.editable.inherit.purchase</field>
        <field name="model">stock.warehouse.orderpoint</field>
        <field name="inherit_id" ref="stock.view_warehouse_orderpoint_tree_editable"/>
        <field name="arch" type="xml">
            <field name="route_id" position="after">
                <field name="show_bom" invisible="1"/>
                <field name="bom_id" optional="hide" attrs="{'invisible': [('show_bom', '=', False)]}" context="{'default_product_tmpl_id': product_tmpl_id}"/>
            </field>
        </field>
    </record>
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
                                <div class="o_primary" t-if="!selection_mode">
                                    <a type="object" name="get_mrp_stock_picking_action_picking_type">
                                        <field name="name"/>
                                    </a>
                                </div>
                                <span class="o_primary" t-if="selection_mode"><field name="name"/></span>
                                <div class="o_secondary"><field class="o_secondary"  name="warehouse_id" readonly="1" groups="stock.group_stock_multi_warehouses"/></div>
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
                                            <a class="oe_kanban_stock_picking_type_list" name="%(mrp_production_action_picking_deshboard)d" type="action" context="{'search_default_planning_issues': 1, 'default_picking_type_id': active_id}">
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
                                        <span>Orders</span>
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
                                        <a name="%(action_mrp_production_form)d" context="{'default_picking_type_id': active_id}" type="action">Manufacturing Order</a>
                                    </div>
                                </div>
                            </div>

                            <div t-if="widget.editable" class="o_kanban_card_manage_settings row">
                                <div role="menuitem" aria-haspopup="true" class="col-8">
                                    <ul role="menu" class="oe_kanban_colorpicker" data-field="color"/>
                                </div>
                                <div class="col-4">
                                    <a class="dropdown-item" role="menuitem" type="edit">Configuration</a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="view_picking_form_inherit_mrp" model="ir.ui.view">
        <field name="name">view.picking.form.inherit.mrp</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_create_lots']" position="after">
                <field name="has_kits" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='description_picking']" position="after">
                <field name="description_bom_line" optional="show" attrs="{'column_invisible': [('parent.has_kits', '=', False)]}"/>
            </xpath>
        </field>
    </record>

    <record id="view_stock_move_line_detailed_operation_tree_mrp" model="ir.ui.view">
        <field name="name">stock.move.line.operations.tree.mrp</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.view_stock_move_line_detailed_operation_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="after">
                <field name="description_bom_line" optional="show" attrs="{'column_invisible': [('parent.has_kits', '=', False)]}"/>
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

    <record id="stock_scrap_search_view_inherit_mrp" model="ir.ui.view">
        <field name="name">stock.scrap.search.inherit.mrp</field>
        <field name="model">stock.scrap</field>
        <field name="inherit_id" ref="stock.stock_scrap_search_view"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='transfer']" position="after">
                <filter string="Manufacturing Order" name="production_id" domain="[]" context="{'group_by':'production_id'}"/>
            </xpath>
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
                    <field name="manufacture_to_resupply"/>
                    <field name="manufacture_steps" attrs="{'invisible': [('manufacture_to_resupply', '=', False)]}" widget="radio" groups="stock.group_adv_location"/>
                </xpath>
                <xpath expr="//field[@name='out_type_id']" position="after">
                    <field name="manu_type_id" readonly="True"/>
                </xpath>
                <xpath expr="//group[@name='group_resupply']" position="attributes">
                    <attribute name="groups">stock.group_adv_location,stock.group_stock_multi_warehouses</attribute>
                </xpath>
                <xpath expr="//field[@name='wh_output_stock_loc_id']" position="after">
                    <field name="sam_loc_id" readonly="True"/>
                    <field name="pbm_loc_id" readonly="True"/>
                </xpath>
                <xpath expr="//field[@name='out_type_id']" position="after">
                    <field name="sam_type_id" readonly="True"/>
                    <field name="pbm_type_id" readonly="True"/>
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
from odoo.tools import float_is_zero, float_round


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
            if wizard.product_qty < produced:
                format_qty = '%.{precision}f'.format(precision=precision)
                raise UserError(_(
                    "You have already processed %(quantity)s. Please input a quantity higher than %(minimum)s ",
                    quantity=format_qty % produced,
                    minimum=format_qty % produced
                ))
            old_production_qty = production.product_qty
            new_production_qty = wizard.product_qty
            done_moves = production.move_finished_ids.filtered(lambda x: x.state == 'done' and x.product_id == production.product_id)
            qty_produced = production.product_id.uom_id._compute_quantity(sum(done_moves.mapped('product_qty')), production.product_uom_id)

            factor = (new_production_qty - qty_produced) / (old_production_qty - qty_produced)
            update_info = production._update_raw_moves(factor)
            documents = {}
            for move, old_qty, new_qty in update_info:
                iterate_key = production._get_document_iterate_key(move)
                if iterate_key:
                    document = self.env['stock.picking']._log_activity_get_documents({move: (new_qty, old_qty)}, iterate_key, 'UP')
                    for key, value in document.items():
                        if documents.get(key):
                            documents[key] += [value]
                        else:
                            documents[key] = [value]
            production._log_manufacture_exception(documents)
            finished_moves_modification = self._update_finished_moves(production, new_production_qty - qty_produced, old_production_qty - qty_produced)
            if finished_moves_modification:
                production._log_downside_manufactured_quantity(finished_moves_modification)
            production.write({'product_qty': new_production_qty})

            for wo in production.workorder_ids:
                operation = wo.operation_id
                wo.duration_expected = wo._get_duration_expected(ratio=new_production_qty / old_production_qty)
                quantity = wo.qty_production - wo.qty_produced
                if production.product_id.tracking == 'serial':
                    quantity = 1.0 if not float_is_zero(quantity, precision_digits=precision) else 0.0
                else:
                    quantity = quantity if (quantity > 0 and not float_is_zero(quantity, precision_digits=precision)) else 0
                wo._update_qty_producing(quantity)
                if wo.qty_produced < wo.qty_production and wo.state == 'done':
                    wo.state = 'progress'
                if wo.qty_produced == wo.qty_production and wo.state == 'progress':
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

        # run scheduler for moves forecasted to not have enough in stock
        self.mo_id.filtered(lambda mo: mo.state in ['confirmed', 'progress']).move_raw_ids._trigger_scheduler()

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
                        <button name="change_prod_qty" string="Approve" data-hotkey="q"
                            colspan="1" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
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

## File: wizard\mrp_consumption_warning.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class MrpConsumptionWarning(models.TransientModel):
    _name = 'mrp.consumption.warning'
    _description = "Wizard in case of consumption in warning/strict and more component has been used for a MO (related to the bom)"

    mrp_production_ids = fields.Many2many('mrp.production')
    mrp_production_count = fields.Integer(compute="_compute_mrp_production_count")

    consumption = fields.Selection([
        ('flexible', 'Allowed'),
        ('warning', 'Allowed with warning'),
        ('strict', 'Blocked')], compute="_compute_consumption")
    mrp_consumption_warning_line_ids = fields.One2many('mrp.consumption.warning.line', 'mrp_consumption_warning_id')

    @api.depends("mrp_production_ids")
    def _compute_mrp_production_count(self):
        for wizard in self:
            wizard.mrp_production_count = len(wizard.mrp_production_ids)

    @api.depends("mrp_consumption_warning_line_ids.consumption")
    def _compute_consumption(self):
        for wizard in self:
            consumption_map = set(wizard.mrp_consumption_warning_line_ids.mapped("consumption"))
            wizard.consumption = "strict" in consumption_map and "strict" or "warning" in consumption_map and "warning" or "flexible"

    def action_confirm(self):
        ctx = dict(self.env.context)
        ctx.pop('default_mrp_production_ids', None)
        action_from_do_finish = False
        if self.env.context.get('from_workorder'):
            if self.env.context.get('active_model') == 'mrp.workorder':
                action_from_do_finish = self.env['mrp.workorder'].browse(self.env.context.get('active_id')).do_finish()
        action_from_mark_done = self.mrp_production_ids.with_context(ctx, skip_consumption=True).button_mark_done()
        return action_from_do_finish or action_from_mark_done

    def action_cancel(self):
        if self.env.context.get('from_workorder') and len(self.mrp_production_ids) == 1:
            return {
                'type': 'ir.actions.act_window',
                'res_model': 'mrp.production',
                'views': [[self.env.ref('mrp.mrp_production_form_view').id, 'form']],
                'res_id': self.mrp_production_ids.id,
                'target': 'main',
            }

class MrpConsumptionWarningLine(models.TransientModel):
    _name = 'mrp.consumption.warning.line'
    _description = "Line of issue consumption"

    mrp_consumption_warning_id = fields.Many2one('mrp.consumption.warning', "Parent Wizard", readonly=True, required=True, ondelete="cascade")
    mrp_production_id = fields.Many2one('mrp.production', "Manufacturing Order", readonly=True, required=True, ondelete="cascade")
    consumption = fields.Selection(related="mrp_production_id.consumption")

    product_id = fields.Many2one('product.product', "Product", readonly=True, required=True)
    product_uom_id = fields.Many2one('uom.uom', "Unit of Measure", related="product_id.uom_id", readonly=True)
    product_consumed_qty_uom = fields.Float("Consumed", readonly=True)
    product_expected_qty_uom = fields.Float("To Consume", readonly=True)

```

## File: wizard\mrp_consumption_warning_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- MO Consumption Warning -->
        <record id="view_mrp_consumption_warning_form" model="ir.ui.view">
            <field name="name">Consumption Warning</field>
            <field name="model">mrp.consumption.warning</field>
            <field name="arch" type="xml">
                <form string="Consumption Warning">
                    <field name="mrp_production_ids" invisible="1"/>
                    <field name="consumption" invisible="1"/>
                    <field name="mrp_production_count" invisible="1"/>
                    <div class="m-2">
                        You consumed a different quantity than expected for the following products.
                        <b attrs="{'invisible': [('consumption', '=', 'strict')]}">
                            Please confirm it has been done on purpose.
                        </b>
                        <b attrs="{'invisible': [('consumption', '!=', 'strict')]}">
                            Please review your component consumption or ask a manager to validate 
                            <span attrs="{'invisible':[('mrp_production_count', '!=', 1)]}">this manufacturing order</span>
                            <span attrs="{'invisible':[('mrp_production_count', '=', 1)]}">these manufacturing orders</span>.
                        </b>
                    </div>
                    <field name="mrp_consumption_warning_line_ids" nolabel="1">
                        <tree create="0" delete="0" editable="top">
                            <field name="mrp_production_id" attrs="{'column_invisible':[('parent.mrp_production_count', '=', 1)]}" force_save="1"/>
                            <field name="consumption" invisible="1" force_save="1"/>
                            <field name="product_id" force_save="1"/>
                            <field name="product_uom_id" groups="uom.group_uom" force_save="1"/>
                            <field name="product_expected_qty_uom" force_save="1"/>
                            <field name="product_consumed_qty_uom" force_save="1"/>
                        </tree>
                    </field>
                    <footer>
                        <button name="action_confirm" string="Force" data-hotkey="q"
                            groups="mrp.group_mrp_manager" attrs="{'invisible': [('consumption', '!=', 'strict')]}"
                            colspan="1" type="object" class="btn-primary"/>
                        <button name="action_confirm" string="Confirm" attrs="{'invisible': [('consumption', '=', 'strict')]}" data-hotkey="q"
                            colspan="1" type="object" class="btn-primary"/>
                        <button name="action_cancel" string="Discard" data-hotkey="w"
                            colspan="1" type="object" class="btn-secondary"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_mrp_consumption_warning" model="ir.actions.act_window">
            <field name="name">Consumption Warning</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.consumption.warning</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>

    </data>
</odoo>    

```

## File: wizard\mrp_immediate_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.tools import float_compare


class MrpImmediateProductionLine(models.TransientModel):
    _name = 'mrp.immediate.production.line'
    _description = 'Immediate Production Line'

    immediate_production_id = fields.Many2one('mrp.immediate.production', 'Immediate Production', required=True)
    production_id = fields.Many2one('mrp.production', 'Production', required=True)
    to_immediate = fields.Boolean('To Process')


class MrpImmediateProduction(models.TransientModel):
    _name = 'mrp.immediate.production'
    _description = 'Immediate Production'

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if 'immediate_production_line_ids' in fields:
            if self.env.context.get('default_mo_ids'):
                res['mo_ids'] = self.env.context['default_mo_ids']
                res['immediate_production_line_ids'] = [(0, 0, {'to_immediate': True, 'production_id': mo_id[1]}) for mo_id in res['mo_ids']]
        return res

    mo_ids = fields.Many2many('mrp.production', 'mrp_production_production_rel')
    show_productions = fields.Boolean(compute='_compute_show_production')
    immediate_production_line_ids = fields.One2many(
        'mrp.immediate.production.line',
        'immediate_production_id',
        string="Immediate Production Lines")

    @api.depends('immediate_production_line_ids')
    def _compute_show_production(self):
        for wizard in self:
            wizard.show_productions = len(wizard.immediate_production_line_ids.production_id) > 1

    def process(self):
        productions_to_do = self.env['mrp.production']
        productions_not_to_do = self.env['mrp.production']
        for line in self.immediate_production_line_ids:
            if line.to_immediate is True:
                productions_to_do |= line.production_id
            else:
                productions_not_to_do |= line.production_id

        for production in productions_to_do:
            error_msg = ""
            if production.product_tracking in ('lot', 'serial') and not production.lot_producing_id:
                production.action_generate_serial()
            if production.product_tracking == 'serial' and float_compare(production.qty_producing, 1, precision_rounding=production.product_uom_id.rounding) == 1:
                production.qty_producing = 1
            else:
                production.qty_producing = production.product_qty - production.qty_produced
            production._set_qty_producing()
            for move in production.move_raw_ids.filtered(lambda m: m.state not in ['done', 'cancel']):
                rounding = move.product_uom.rounding
                for move_line in move.move_line_ids:
                    if move_line.product_uom_qty:
                        move_line.qty_done = min(move_line.product_uom_qty, move_line.move_id.should_consume_qty)
                    if float_compare(move.quantity_done, move.should_consume_qty, precision_rounding=rounding) >= 0:
                        break
                if float_compare(move.product_uom_qty, move.quantity_done, precision_rounding=move.product_uom.rounding) == 1:
                    if move.has_tracking in ('serial', 'lot'):
                        error_msg += "\n  - %s" % move.product_id.display_name

            if error_msg:
                error_msg = _('You need to supply Lot/Serial Number for products:') + error_msg
                raise UserError(error_msg)

        productions_to_validate = self.env.context.get('button_mark_done_production_ids')
        if productions_to_validate:
            productions_to_validate = self.env['mrp.production'].browse(productions_to_validate)
            productions_to_validate = productions_to_validate - productions_not_to_do
            return productions_to_validate.with_context(skip_immediate=True).button_mark_done()
        return True


```

## File: wizard\mrp_immediate_production_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_immediate_production" model="ir.ui.view">
        <field name="name">mrp.immediate.production.view.form</field>
        <field name="model">mrp.immediate.production</field>
        <field name="arch" type="xml">
            <form string="Immediate production?">
                <group>
                    <p>You have not recorded <i>produced</i> quantities yet, by clicking on <i>apply</i> Odoo will produce all the finished products and consume all components.</p>
                </group>

                <field name="show_productions" invisible="1"/>
                <field name="immediate_production_line_ids" nolabel="1" attrs="{'invisible': [('show_productions', '=', False)]}">
                    <tree create="0" delete="0" editable="top">
                        <field name="production_id"/>
                        <field name="to_immediate" widget="boolean_toggle"/>
                    </tree>
                </field>

                <footer>
                    <button name="process" string="Apply" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\mrp_production_backorder.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MrpProductionBackorderLine(models.TransientModel):
    _name = 'mrp.production.backorder.line'
    _description = "Backorder Confirmation Line"

    mrp_production_backorder_id = fields.Many2one('mrp.production.backorder', 'MO Backorder', required=True, ondelete="cascade")
    mrp_production_id = fields.Many2one('mrp.production', 'Manufacturing Order', required=True, ondelete="cascade", readonly=True)
    to_backorder = fields.Boolean('To Backorder')


class MrpProductionBackorder(models.TransientModel):
    _name = 'mrp.production.backorder'
    _description = "Wizard to mark as done or create back order"

    mrp_production_ids = fields.Many2many('mrp.production')

    mrp_production_backorder_line_ids = fields.One2many(
        'mrp.production.backorder.line',
        'mrp_production_backorder_id',
        string="Backorder Confirmation Lines")
    show_backorder_lines = fields.Boolean("Show backorder lines", compute="_compute_show_backorder_lines")

    @api.depends('mrp_production_backorder_line_ids')
    def _compute_show_backorder_lines(self):
        for wizard in self:
            wizard.show_backorder_lines = len(wizard.mrp_production_backorder_line_ids) > 1

    def action_close_mo(self):
        return self.mrp_production_ids.with_context(skip_backorder=True).button_mark_done()

    def action_backorder(self):
        ctx = dict(self.env.context)
        ctx.pop('default_mrp_production_ids', None)
        mo_ids_to_backorder = self.mrp_production_backorder_line_ids.filtered(lambda l: l.to_backorder).mrp_production_id.ids
        return self.mrp_production_ids.with_context(ctx, skip_backorder=True, mo_ids_to_backorder=mo_ids_to_backorder).button_mark_done()

```

## File: wizard\mrp_production_backorder.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        
        <!-- MO Backorder -->
        <record id="view_mrp_production_backorder_form" model="ir.ui.view">
            <field name="name">Create Backorder</field>
            <field name="model">mrp.production.backorder</field>
            <field name="arch" type="xml">
                <form string="Create a Backorder">
                    <group>
                        <p>
                            Create a backorder if you expect to process the remaining products later. Do not create a backorder if you will not process the remaining products.
                        </p>
                    </group>
                    <field name="show_backorder_lines" invisible="1"/>
                    <field name="mrp_production_backorder_line_ids" nolabel="1" attrs="{'invisible': [('show_backorder_lines', '=', False)]}">
                        <tree create="0" delete="0" editable="top">
                            <field name="mrp_production_id" force_save="1"/>
                            <field name="to_backorder" widget="boolean_toggle"/>
                        </tree>
                    </field>
                    <footer>
                        <button name="action_backorder" string="Create backorder" data-hotkey="q"
                            colspan="1" type="object" class="btn-primary" attrs="{'invisible': [('show_backorder_lines', '!=', False)]}"/>
                        <button name="action_backorder" string="Validate" data-hotkey="q"
                            colspan="1" type="object" class="btn-primary" attrs="{'invisible': [('show_backorder_lines', '=', False)]}"/>
                        <button name="action_close_mo" type="object" string="No Backorder" attrs="{'invisible': [('show_backorder_lines', '!=', False)]}" data-hotkey="x"/>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z" />
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_mrp_production_backorder" model="ir.actions.act_window">
            <field name="name">You produced less than initial demand</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">mrp.production.backorder</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>
       
    </data>
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
                    <button name="button_block" string="Block" type="object" class="btn-danger text-uppercase" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
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

## File: wizard\stock_assign_serial_numbers.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import Counter

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.tools.float_utils import float_compare


class StockAssignSerialNumbers(models.TransientModel):
    _inherit = 'stock.assign.serial'

    production_id = fields.Many2one('mrp.production', 'Production')
    expected_qty = fields.Float('Expected Quantity', digits='Product Unit of Measure')
    serial_numbers = fields.Text('Produced Serial Numbers')
    produced_qty = fields.Float('Produced Quantity', digits='Product Unit of Measure')
    show_apply = fields.Boolean(help="Technical field to show the Apply button")
    show_backorders = fields.Boolean(help="Technical field to show the Create Backorder and No Backorder buttons")

    def generate_serial_numbers_production(self):
        if self.next_serial_number and self.next_serial_count:
            generated_serial_numbers = "\n".join(self.env['stock.production.lot'].generate_lot_names(self.next_serial_number, self.next_serial_count))
            self.serial_numbers = "\n".join([self.serial_numbers, generated_serial_numbers]) if self.serial_numbers else generated_serial_numbers
            self._onchange_serial_numbers()
        action = self.env["ir.actions.actions"]._for_xml_id("mrp.act_assign_serial_numbers_production")
        action['res_id'] = self.id
        return action

    def _get_serial_numbers(self):
        if self.serial_numbers:
            return list(filter(lambda serial_number: len(serial_number.strip()) > 0, self.serial_numbers.split('\n')))
        return []

    @api.onchange('serial_numbers')
    def _onchange_serial_numbers(self):
        self.show_apply = False
        self.show_backorders = False
        serial_numbers = self._get_serial_numbers()
        duplicate_serial_numbers = [serial_number for serial_number, counter in Counter(serial_numbers).items() if counter > 1]
        if duplicate_serial_numbers:
            self.serial_numbers = ""
            self.produced_qty = 0
            raise UserError(_('Duplicate Serial Numbers (%s)') % ','.join(duplicate_serial_numbers))
        existing_serial_numbers = self.env['stock.production.lot'].search([
            ('company_id', '=', self.production_id.company_id.id),
            ('product_id', '=', self.production_id.product_id.id),
            ('name', 'in', serial_numbers),
        ])
        if existing_serial_numbers:
            self.serial_numbers = ""
            self.produced_qty = 0
            raise UserError(_('Existing Serial Numbers (%s)') % ','.join(existing_serial_numbers.mapped('display_name')))
        if len(serial_numbers) > self.expected_qty:
            self.serial_numbers = ""
            self.produced_qty = 0
            raise UserError(_('There are more Serial Numbers than the Quantity to Produce'))
        self.produced_qty = len(serial_numbers)
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        self.show_apply = float_compare(self.produced_qty, self.expected_qty, precision_digits=precision) == 0
        self.show_backorders = self.produced_qty > 0 and self.produced_qty < self.expected_qty

    def _assign_serial_numbers(self, cancel_remaining_quantity=False):
        serial_numbers = self._get_serial_numbers()
        productions = self.production_id._split_productions(
            {self.production_id: [1] * len(serial_numbers)}, cancel_remaining_quantity, set_consumed_qty=True)
        production_lots_vals = []
        for serial_name in serial_numbers:
            production_lots_vals.append({
                'product_id': self.production_id.product_id.id,
                'company_id': self.production_id.company_id.id,
                'name': serial_name,
            })
        production_lots = self.env['stock.production.lot'].create(production_lots_vals)
        for production, production_lot in zip(productions, production_lots):
            production.lot_producing_id = production_lot.id
            production.qty_producing = production.product_qty
            for workorder in production.workorder_ids:
                workorder.qty_produced = workorder.qty_producing

        if productions and len(production_lots) < len(productions):
            productions[-1].move_raw_ids.move_line_ids.write({'qty_done': 0})
            productions[-1].state = "confirmed"

    def apply(self):
        self._assign_serial_numbers()

    def create_backorder(self):
        self._assign_serial_numbers(False)

    def no_backorder(self):
        self._assign_serial_numbers(True)

```

## File: wizard\stock_assign_serial_numbers.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_assign_serial_numbers_production" model="ir.ui.view">
        <field name="name">mrp_assign_serial_numbers</field>
        <field name="model">stock.assign.serial</field>
        <field name="arch" type="xml">
            <form string="Serial Mass Produce">
                <group>
                    <field name="production_id" readonly="True"/>
                </group>
                <group>
                    <group>
                        <field name="next_serial_number"/>
                    </group>
                    <group>
                        <label for="next_serial_count"/>
                        <div class="o_row">
                            <span><field name="next_serial_count"/></span>
                            <button name="generate_serial_numbers_production" type="object" class="btn btn-secondary" title="Generate Serial Numbers">
                                <span>Generate</span>
                            </button>
                        </div>
                    </group>
                </group>
                <group>
                    <field name="serial_numbers" placeholder="copy paste a list and/or use Generate"/>
                </group>
                <field name="show_apply" invisible="1" />
                <field name="show_backorders" invisible="1" />
                <group>
                    <group>
                        <field name="produced_qty" readonly="True" force_save="True"/>
                    </group>
                    <group>
                        <field name="expected_qty" readonly="True"/>
                    </group>
                    <p class="o_form_label oe_inline" attrs="{'invisible': [('show_backorders', '=', False)]}">
                        You have entered less serial numbers than the quantity to produce.<br/>
                        Create a backorder if you expect to process the remaining quantities later.<br/>
                        Do not create a backorder if you will not process the remaining products.
                    </p>
                </group>
                <footer>
                    <button name="apply" string="Apply" type="object" class="btn-primary" attrs="{'invisible': [('show_apply', '=', False)]}"/>
                    <button name="create_backorder" string="Create Backorder" type="object" class="btn-primary" attrs="{'invisible': [('show_backorders', '=', False)]}"/>
                    <button name="no_backorder" string="No Backorder" type="object" class="btn-primary" attrs="{'invisible': [('show_backorders', '=', False)]}"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>

    <record id="act_assign_serial_numbers_production" model="ir.actions.act_window">
        <field name="name">Assign Serial Numbers</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">stock.assign.serial</field>
        <field name="view_id" ref="view_assign_serial_numbers_production"/>
        <field name="view_mode">form</field>
        <field name="context">{}</field>
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
            <xpath expr="//div[@name='description']" position="inside">
                Do you confirm you want to unbuild <strong><field name="quantity" readonly="True"/></strong><field name="product_uom_name" readonly="True" class="mx-1"/>from location <strong><field name="location_id" readonly="True"/></strong>? This may lead to inconsistencies in your inventory.
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import change_production_qty
from . import stock_warn_insufficient_qty
from . import mrp_production_backorder
from . import mrp_consumption_warning
from . import mrp_immediate_production
from . import stock_assign_serial_numbers

```

