# Odoo Module: pos_restaurant

Category: Sales/Point Of Sale

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
    'name': 'Restaurant',
    'version': '1.0',
    'category': 'Sales/Point Of Sale',
    'sequence': 6,
    'summary': 'Restaurant extensions for the Point of Sale ',
    'description': """

This module adds several features to the Point of Sale that are specific to restaurant management:
- Bill Printing: Allows you to print a receipt before the order is paid
- Bill Splitting: Allows you to split an order into different orders
- Kitchen Order Printing: allows you to print orders updates to kitchen or bar printers

""",
    'depends': ['point_of_sale'],
    'website': 'https://www.odoo.com/page/point-of-sale-restaurant',
    'data': [
        'security/ir.model.access.csv',
        'views/pos_order_views.xml',
        'views/pos_restaurant_views.xml',
        'views/pos_config_views.xml',
        'views/pos_restaurant_templates.xml',
    ],
    'qweb': [
        'static/src/xml/multiprint.xml',
        'static/src/xml/splitbill.xml',
        'static/src/xml/printbill.xml',
        'static/src/xml/notes.xml',
        'static/src/xml/floors.xml',
    ],
    'demo': [
        'data/pos_restaurant_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\pos_restaurant_demo.xml

```xml
<?xml version="1.0"?>
<odoo>

        <!-- ******  Basic Restaurant Setup ***** -->

        <!-- Kitchen Printer -->

        <record id="kitchen_printer" model="restaurant.printer">
            <field name="name">Kitchen Printer</field>
            <field name="proxy_ip">localhost</field>
            <field name="product_categories_ids" eval="[(6, 0, [ref('point_of_sale.pos_category_miscellaneous')])]" />
        </record>

        <record id="drinks" model="pos.category">
              <field name="name">Drinks</field>
        </record>

        <record id="product_category_pos_food" model="product.category">
            <field name="parent_id" ref="point_of_sale.product_category_pos"/>
            <field name="name">Food</field>
        </record>

        <record id="food" model="pos.category">
              <field name="name">Food</field>
        </record>

        <!-- Food -->
        <record id="pos_food_margherita" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Margherita</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza.png"/>
        </record>
        <record id="pos_food_funghi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Funghi</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza.png"/>
        </record>
        <record id="pos_food_vege" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Vegetarian</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza.png"/>
        </record>
        <record id="pos_food_bolo" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.5</field>
            <field name="name">Pasta Bolognese</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta.jpg"/>
        </record>
        <record id="pos_food_4formaggi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">5.5</field>
            <field name="name">Pasta 4 formaggi </field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta-4f.jpg"/>
        </record>
        <record id="pos_food_bacon" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.5</field>
            <field name="name">Bacon Burger</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-burger.jpg"/>
        </record>
        <record id="pos_food_cheeseburger" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Cheese Burger</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-cheeseburger.jpg"/>
        </record>
        <record id="pos_food_chicken" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.0</field>
            <field name="name">Chicken Curry Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-sandwich.jpg"/>
        </record>
        <record id="pos_food_tuna" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.0</field>
            <field name="name">Spicy Tuna Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-tuna.jpg"/>
        </record>
        <record id="pos_food_mozza" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.9</field>
            <field name="name">Mozzarella Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-mozza.jpg"/>
        </record>
        <record id="pos_food_club" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.4</field>
            <field name="name">Club Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-club.jpg"/>
        </record>
        <record id="pos_food_maki" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">12.0</field>
            <field name="name">Lunch Maki 18pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-maki.jpg"/>
        </record>
        <record id="pos_food_salmon" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">13.80</field>
            <field name="name">Lunch Salmon 20pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon.jpg"/>
        </record>
        <record id="pos_food_temaki" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">14.0</field>
            <field name="name">Lunch Temaki mix 3pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-temaki.jpg"/>
        </record>
        <record id="pos_food_chirashi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">9.25</field>
            <field name="name">Salmon and Avocado</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="food"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon-avocado.jpg"/>
        </record>

        <!-- Drinks -->
        <record id="coke" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Coca-Cola</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="drinks"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-coke.jpg"/>
        </record>

        <record id="water" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Water</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="drinks"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-water.jpg"/>
        </record>

        <record id="minute_maid" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Minute Maid</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_id" ref="drinks"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-minute_maid.jpg"/>
        </record>

        <!-- Pos Config -->
        <record model="pos.config" id="pos_config_restaurant">
            <field name="name">Bar</field>
            <field name="barcode_nomenclature_id" ref="barcodes.default_barcode_nomenclature"/>
            <field name="module_pos_restaurant">True</field>
            <field name="is_table_management">True</field>
            <field name="iface_splitbill">True</field>
            <field name="iface_printbill">True</field>
            <field name="iface_orderline_notes">True</field>
            <field name="printer_ids" eval="[(6, 0, [ref('pos_restaurant.kitchen_printer')])]" />
            <field name="iface_start_categ_id" ref="drinks"/>
            <field name="start_category">True</field>
        </record>

        <!-- Floors: Main Floor -->

        <record id="floor_main" model="restaurant.floor">
            <field name="name">Main Floor</field>
            <field name="background_color">rgb(136,137,242)</field>
            <field name="pos_config_id" eval="ref('pos_restaurant.pos_config_restaurant')" />
        </record>

        <record id="table_01" model="restaurant.table">
            <field name="name">T1</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">50</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_02" model="restaurant.table">
            <field name="name">T2</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">212</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_03" model="restaurant.table">
            <field name="name">T3</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">374</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_04" model="restaurant.table">
            <field name="name">T4</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">536</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_05" model="restaurant.table">
            <field name="name">T5</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">698</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_06" model="restaurant.table">
            <field name="name">T6</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">860</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_07" model="restaurant.table">
            <field name="name">T7</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">50</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_08" model="restaurant.table">
            <field name="name">T8</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">212</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_09" model="restaurant.table">
            <field name="name">T9</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">698</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_10" model="restaurant.table">
            <field name="name">T10</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">860</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_11" model="restaurant.table">
            <field name="name">T11</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_main')" />
            <field name="seats">4</field>
            <field name="color">rgb(78,210,190)</field>
            <field name="shape">round</field>
            <field name="width">210</field>
            <field name="height">210</field>
            <field name="position_h">400</field>
            <field name="position_v">230</field>
        </record>

        <!-- Restaurant Floor: Patio -->

        <record id="floor_patio" model="restaurant.floor">
            <field name="name">Patio</field>
            <field name="background_color">rgb(130, 233, 171)</field>
            <field name="pos_config_id" eval="ref('pos_restaurant.pos_config_restaurant')" />
        </record>

        <!-- Patio: Left table row -->

        <record id="table_21" model="restaurant.table">
            <field name="name">T1</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">100</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_22" model="restaurant.table">
            <field name="name">T2</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">100</field>
            <field name="position_v">166</field>
        </record>

        <record id="table_23" model="restaurant.table">
            <field name="name">T3</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">100</field>
            <field name="position_v">283</field>
        </record>

        <record id="table_24" model="restaurant.table">
            <field name="name">T4</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">100</field>
            <field name="position_v">400</field>
        </record>

        <!-- Patio: Right table row -->

        <record id="table_25" model="restaurant.table">
            <field name="name">T5</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">800</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_26" model="restaurant.table">
            <field name="name">T6</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">800</field>
            <field name="position_v">166</field>
        </record>

        <record id="table_27" model="restaurant.table">
            <field name="name">T7</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">800</field>
            <field name="position_v">283</field>
        </record>

        <record id="table_28" model="restaurant.table">
            <field name="name">T8</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">75</field>
            <field name="position_h">800</field>
            <field name="position_v">400</field>
        </record>

        <!-- Patio: Center table block -->

        <record id="table_29" model="restaurant.table">
            <field name="name">T9</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">330</field>
            <field name="position_v">100</field>
        </record>

        <record id="table_29" model="restaurant.table">
            <field name="name">T9</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">330</field>
            <field name="position_v">100</field>
        </record>

        <record id="table_30" model="restaurant.table">
            <field name="name">T10</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">560</field>
            <field name="position_v">100</field>
        </record>

        <record id="table_31" model="restaurant.table">
            <field name="name">T11</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">330</field>
            <field name="position_v">315</field>
        </record>

        <record id="table_32" model="restaurant.table">
            <field name="name">T12</field>
            <field name="floor_id" eval="ref('pos_restaurant.floor_patio')" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">560</field>
            <field name="position_v">315</field>
        </record>

</odoo>

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosConfig(models.Model):
    _inherit = 'pos.config'

    iface_splitbill = fields.Boolean(string='Bill Splitting', help='Enables Bill Splitting in the Point of Sale.')
    iface_printbill = fields.Boolean(string='Bill Printing', help='Allows to print the Bill before payment.')
    iface_orderline_notes = fields.Boolean(string='Orderline Notes', help='Allow custom notes on Orderlines.')
    floor_ids = fields.One2many('restaurant.floor', 'pos_config_id', string='Restaurant Floors', help='The restaurant floors served by this point of sale.')
    printer_ids = fields.Many2many('restaurant.printer', 'pos_config_printer_rel', 'config_id', 'printer_id', string='Order Printers')
    is_table_management = fields.Boolean('Table Management')
    is_order_printer = fields.Boolean('Order Printer')
    module_pos_restaurant = fields.Boolean(default=True)

    @api.onchange('module_pos_restaurant')
    def _onchange_module_pos_restaurant(self):
        if not self.module_pos_restaurant:
            self.update({'iface_printbill': False,
            'iface_splitbill': False,
            'is_order_printer': False,
            'is_table_management': False,
            'iface_orderline_notes': False})

    @api.onchange('is_table_management')
    def _onchange_is_table_management(self):
        if not self.is_table_management:
            self.floor_ids = [(5, 0, 0)]

    @api.onchange('is_order_printer')
    def _onchange_is_order_printer(self):
        if not self.is_order_printer:
            self.printer_ids = [(5, 0, 0)]

    def get_tables_order_count(self):
        """         """
        self.ensure_one()
        tables = self.env['restaurant.table'].search([('floor_id.pos_config_id', 'in', self.ids)])
        domain = [('state', '=', 'draft'), ('table_id', 'in', tables.ids)]

        order_stats = self.env['pos.order'].read_group(domain, ['table_id'], 'table_id')
        orders_map = dict((s['table_id'][0], s['table_id_count']) for s in order_stats)

        result = []
        for table in tables:
            result.append({'id': table.id, 'orders': orders_map.get(table.id, 0)})
        return result

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from itertools import groupby
from re import search

from odoo import api, fields, models


class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    note = fields.Char('Note added by the waiter.')
    mp_skip = fields.Boolean('Skip line when sending ticket to kitchen printers.')


class PosOrder(models.Model):
    _inherit = 'pos.order'

    table_id = fields.Many2one('restaurant.table', string='Table', help='The table where this order was served', index=True)
    customer_count = fields.Integer(string='Guests', help='The amount of customers that have been served by this order.')

    def _get_pack_lot_lines(self, order_lines):
        """Add pack_lot_lines to the order_lines.

        The function doesn't return anything but adds the results directly to the order_lines.

        :param order_lines: order_lines for which the pack_lot_lines are to be requested.
        :type order_lines: pos.order.line.
        """
        pack_lots = self.env['pos.pack.operation.lot'].search_read(
                domain = [('pos_order_line_id', 'in', [order_line['id'] for order_line in order_lines])],
                fields = [
                    'id',
                    'lot_name',
                    'pos_order_line_id'
                    ])
        for pack_lot in pack_lots:
            pack_lot['order_line'] = pack_lot['pos_order_line_id'][0]
            pack_lot['server_id'] = pack_lot['id']

            del pack_lot['pos_order_line_id']
            del pack_lot['id']

        for order_line_id, pack_lot_ids in groupby(pack_lots, key=lambda x:x['order_line']):
            next(order_line for order_line in order_lines if order_line['id'] == order_line_id)['pack_lot_ids'] = list(pack_lots)

    def _get_fields_for_order_line(self):
        return [
            'id',
            'discount',
            'product_id',
            'price_unit',
            'order_id',
            'qty',
            'note',
            'mp_skip',
        ]

    def _get_order_lines(self, orders):
        """Add pos_order_lines to the orders.

        The function doesn't return anything but adds the results directly to the orders.

        :param orders: orders for which the order_lines are to be requested.
        :type orders: pos.order.
        """
        order_lines = self.env['pos.order.line'].search_read(
                domain = [('order_id', 'in', [to['id'] for to in orders])],
                fields = self._get_fields_for_order_line())

        if order_lines != []:
            self._get_pack_lot_lines(order_lines)

        extended_order_lines = []
        for order_line in order_lines:
            order_line['product_id'] = order_line['product_id'][0]
            order_line['server_id'] = order_line['id']

            del order_line['id']
            if not 'pack_lot_ids' in order_line:
                order_line['pack_lot_ids'] = []
            extended_order_lines.append([0, 0, order_line])

        for order_id, order_lines in groupby(extended_order_lines, key=lambda x:x[2]['order_id']):
            next(order for order in orders if order['id'] == order_id[0])['lines'] = list(order_lines)

    def _get_payment_lines(self, orders):
        """Add account_bank_statement_lines to the orders.

        The function doesn't return anything but adds the results directly to the orders.

        :param orders: orders for which the payment_lines are to be requested.
        :type orders: pos.order.
        """
        payment_lines = self.env['pos.payment'].search_read(
                domain = [('pos_order_id', 'in', [po['id'] for po in orders])],
                fields = [
                    'id',
                    'amount',
                    'pos_order_id',
                    'payment_method_id',
                    ])

        extended_payment_lines = []
        for payment_line in payment_lines:
            payment_line['server_id'] = payment_line['id']
            payment_line['payment_method_id'] = payment_line['payment_method_id'][0]

            del payment_line['id']
            extended_payment_lines.append([0, 0, payment_line])
        for order_id, payment_lines in groupby(extended_payment_lines, key=lambda x:x[2]['pos_order_id']):
            next(order for order in orders if order['id'] == order_id[0])['statement_ids'] = list(payment_lines)

    def _get_fields_for_draft_order(self):
        return [
                    'id',
                    'pricelist_id',
                    'partner_id',
                    'sequence_number',
                    'session_id',
                    'pos_reference',
                    'create_uid',
                    'create_date',
                    'customer_count',
                    'fiscal_position_id',
                    'table_id',
                    'to_invoice',
                    ]

    @api.model
    def get_table_draft_orders(self, table_id):
        """Generate an object of all draft orders for the given table.

        Generate and return an JSON object with all draft orders for the given table, to send to the 
        front end application.

        :param table_id: Id of the selected table.
        :type table_id: int.
        :returns: list -- list of dict representing the table orders
        """
        table_orders = self.search_read(
                domain = [('state', '=', 'draft'), ('table_id', '=', table_id)],
                fields = self._get_fields_for_draft_order())

        self._get_order_lines(table_orders)
        self._get_payment_lines(table_orders)

        for order in table_orders:
            order['pos_session_id'] = order['session_id'][0]
            order['uid'] = search(r"\d{5,}-\d{3,}-\d{4,}", order['pos_reference']).group(0)
            order['name'] = order['pos_reference']
            order['creation_date'] = order['create_date']
            order['server_id'] = order['id']
            if order['fiscal_position_id']:
                order['fiscal_position_id'] = order['fiscal_position_id'][0]
            if order['pricelist_id']:
                order['pricelist_id'] = order['pricelist_id'][0]
            if order['partner_id']:
                order['partner_id'] = order['partner_id'][0]
            if order['table_id']:
                order['table_id'] = order['table_id'][0]

            if not 'lines' in order:
                order['lines'] = []
            if not 'statement_ids' in order:
                order['statement_ids'] = []

            del order['id']
            del order['session_id']
            del order['pos_reference']
            del order['create_date']

        return table_orders

    @api.model
    def _order_fields(self, ui_order):
        order_fields = super(PosOrder, self)._order_fields(ui_order)
        order_fields['table_id'] = ui_order.get('table_id', False)
        order_fields['customer_count'] = ui_order.get('customer_count', 0)
        return order_fields

```

## File: models\pos_restaurant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class RestaurantFloor(models.Model):

    _name = 'restaurant.floor'
    _description = 'Restaurant Floor'

    name = fields.Char('Floor Name', required=True, help='An internal identification of the restaurant floor')
    pos_config_id = fields.Many2one('pos.config', string='Point of Sale')
    background_image = fields.Binary('Background Image', help='A background image used to display a floor layout in the point of sale interface')
    background_color = fields.Char('Background Color', help='The background color of the floor layout, (must be specified in a html-compatible format)', default='rgb(210, 210, 210)')
    table_ids = fields.One2many('restaurant.table', 'floor_id', string='Tables', help='The list of tables in this floor')
    sequence = fields.Integer('Sequence', help='Used to sort Floors', default=1)

    def unlink(self):
        confs = self.mapped('pos_config_id').filtered(lambda c: c.is_table_management == True)
        opened_session = self.env['pos.session'].search([('config_id', 'in', confs.ids), ('state', '!=', 'closed')])
        if opened_session:
            error_msg = _("You cannot remove a floor that is used in a PoS session, close the session(s) first: \n")
            for floor in self:
                for session in opened_session:
                    if floor in session.config_id.floor_ids:
                        error_msg += _("Floor: %s - PoS Config: %s \n") % (floor.name, session.config_id.name)
            if confs:
                raise UserError(error_msg)
        return super(RestaurantFloor, self).unlink()


class RestaurantTable(models.Model):

    _name = 'restaurant.table'
    _description = 'Restaurant Table'

    name = fields.Char('Table Name', required=True, help='An internal identification of a table')
    floor_id = fields.Many2one('restaurant.floor', string='Floor')
    shape = fields.Selection([('square', 'Square'), ('round', 'Round')], string='Shape', required=True, default='square')
    position_h = fields.Float('Horizontal Position', default=10,
        help="The table's horizontal position from the left side to the table's center, in pixels")
    position_v = fields.Float('Vertical Position', default=10,
        help="The table's vertical position from the top to the table's center, in pixels")
    width = fields.Float('Width', default=50, help="The table's width in pixels")
    height = fields.Float('Height', default=50, help="The table's height in pixels")
    seats = fields.Integer('Seats', default=1, help="The default number of customer served at this table.")
    color = fields.Char('Color', help="The table's color, expressed as a valid 'background' CSS property value")
    active = fields.Boolean('Active', default=True, help='If false, the table is deactivated and will not be available in the point of sale')

    @api.model
    def create_from_ui(self, table):
        """ create or modify a table from the point of sale UI.
            table contains the table's fields. If it contains an
            id, it will modify the existing table. It then
            returns the id of the table.
        """
        if table.get('floor_id'):
            table['floor_id'] = table['floor_id'][0]

        table_id = table.pop('id', False)
        if table_id:
            self.browse(table_id).write(table)
        else:
            table_id = self.create(table).id
        return table_id


class RestaurantPrinter(models.Model):

    _name = 'restaurant.printer'
    _description = 'Restaurant Printer'

    name = fields.Char('Printer Name', required=True, default='Printer', help='An internal identification of the printer')
    printer_type = fields.Selection(string='Printer Type', default='iot',
        selection=[('iot', ' Use a printer connected to the IoT Box')])
    proxy_ip = fields.Char('Proxy IP Address', help="The IP Address or hostname of the Printer's hardware proxy")
    product_categories_ids = fields.Many2many('pos.category', 'printer_category_rel', 'printer_id', 'category_id', string='Printed Product Categories')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_order
from . import pos_restaurant

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_restaurant_printer,restaurant.printer.user,model_restaurant_printer,point_of_sale.group_pos_user,1,0,0,0
access_restaurant_printer_manager,restaurant.printer.manager,model_restaurant_printer,point_of_sale.group_pos_manager,1,1,1,1
access_restaurant_floor,restaurant.floor.user,model_restaurant_floor,point_of_sale.group_pos_user,1,0,0,0
access_restaurant_floor_manager,restaurant.floor.manager,model_restaurant_floor,point_of_sale.group_pos_manager,1,1,1,1
access_restaurant_table,restaurant.table.user,model_restaurant_table,point_of_sale.group_pos_user,1,0,0,0
access_restaurant_table_manager,restaurant.table.manager,model_restaurant_table,point_of_sale.group_pos_manager,1,1,1,1

```

## File: static\lib\js\jquery.ui.touch-punch.js

```javascript
/*!
 * jQuery UI Touch Punch 0.2.3
 *
 * Copyright 2011–2014, Dave Furfero
 * Dual licensed under the MIT or GPL Version 2 licenses.
 *
 * Depends:
 *  jquery.ui.widget.js
 *  jquery.ui.mouse.js
 */
(function ($) {

    // Detect touch support
    $.support.touch = (
      'ontouchend' in document || // Default check
      // Odoo fix for Chrome
      // See: https://github.com/furf/jquery-ui-touch-punch/issues/309
      'ontouchstart' in document ||
      'ontouchstart' in window ||
      navigator.maxTouchPoints > 0 ||
      navigator.msMaxTouchPoints > 0
    );
  
    // Ignore browsers without touch support
    if (!$.support.touch) {
      return;
    }
  
    var mouseProto = $.ui.mouse.prototype,
        _mouseInit = mouseProto._mouseInit,
        _mouseDestroy = mouseProto._mouseDestroy,
        touchHandled;
  
    /**
     * Simulate a mouse event based on a corresponding touch event
     * @param {Object} event A touch event
     * @param {String} simulatedType The corresponding mouse event
     */
    function simulateMouseEvent (event, simulatedType) {
  
      // Ignore multi-touch events
      if (event.originalEvent.touches.length > 1) {
        return;
      }
  
      event.preventDefault();
  
      var touch = event.originalEvent.changedTouches[0],
          simulatedEvent = document.createEvent('MouseEvents');
      
      // Initialize the simulated mouse event using the touch event's coordinates
      simulatedEvent.initMouseEvent(
        simulatedType,    // type
        true,             // bubbles                    
        true,             // cancelable                 
        window,           // view                       
        1,                // detail                     
        touch.screenX,    // screenX                    
        touch.screenY,    // screenY                    
        touch.clientX,    // clientX                    
        touch.clientY,    // clientY                    
        false,            // ctrlKey                    
        false,            // altKey                     
        false,            // shiftKey                   
        false,            // metaKey                    
        0,                // button                     
        null              // relatedTarget              
      );
  
      // Dispatch the simulated event to the target element
      event.target.dispatchEvent(simulatedEvent);
    }
  
    /**
     * Handle the jQuery UI widget's touchstart events
     * @param {Object} event The widget element's touchstart event
     */
    mouseProto._touchStart = function (event) {
  
      var self = this;
  
      // Ignore the event if another widget is already being handled
      if (touchHandled || !self._mouseCapture(event.originalEvent.changedTouches[0])) {
        return;
      }
  
      // Set the flag to prevent other widgets from inheriting the touch event
      touchHandled = true;
  
      // Track movement to determine if interaction was a click
      self._touchMoved = false;
  
      // Simulate the mouseover event
      simulateMouseEvent(event, 'mouseover');
  
      // Simulate the mousemove event
      simulateMouseEvent(event, 'mousemove');
  
      // Simulate the mousedown event
      simulateMouseEvent(event, 'mousedown');
    };
  
    /**
     * Handle the jQuery UI widget's touchmove events
     * @param {Object} event The document's touchmove event
     */
    mouseProto._touchMove = function (event) {
  
      // Ignore event if not handled
      if (!touchHandled) {
        return;
      }
  
      // Interaction was not a click
      this._touchMoved = true;
  
      // Simulate the mousemove event
      simulateMouseEvent(event, 'mousemove');
    };
  
    /**
     * Handle the jQuery UI widget's touchend events
     * @param {Object} event The document's touchend event
     */
    mouseProto._touchEnd = function (event) {
  
      // Ignore event if not handled
      if (!touchHandled) {
        return;
      }
  
      // Simulate the mouseup event
      simulateMouseEvent(event, 'mouseup');
  
      // Simulate the mouseout event
      simulateMouseEvent(event, 'mouseout');
  
      // If the touch interaction did not move, it should trigger a click
      if (!this._touchMoved) {
  
        // Simulate the click event
        simulateMouseEvent(event, 'click');
      }
  
      // Unset the flag to allow other widgets to inherit the touch event
      touchHandled = false;
    };
  
    /**
     * A duck punch of the $.ui.mouse _mouseInit method to support touch events.
     * This method extends the widget with bound touch event handlers that
     * translate touch events to mouse events and pass them to the widget's
     * original mouse event handling methods.
     */
    mouseProto._mouseInit = function () {
      
      var self = this;
  
      // Delegate the touch handlers to the widget's element
      self.element.bind({
        touchstart: $.proxy(self, '_touchStart'),
        touchmove: $.proxy(self, '_touchMove'),
        touchend: $.proxy(self, '_touchEnd')
      });
  
      // Call the original $.ui.mouse init method
      _mouseInit.call(self);
    };
  
    /**
     * Remove the touch event handlers
     */
    mouseProto._mouseDestroy = function () {
      
      var self = this;
  
      // Delegate the touch handlers to the widget's element
      self.element.unbind({
        touchstart: $.proxy(self, '_touchStart'),
        touchmove: $.proxy(self, '_touchMove'),
        touchend: $.proxy(self, '_touchEnd')
      });
  
      // Call the original $.ui.mouse destroy method
      _mouseDestroy.call(self);
    };
  
  })(jQuery);
```

## File: static\src\js\floors.js

```javascript
odoo.define('pos_restaurant.floors', function (require) {
"use strict";

var PosBaseWidget = require('point_of_sale.BaseWidget');
var chrome = require('point_of_sale.chrome');
var gui = require('point_of_sale.gui');
var models = require('point_of_sale.models');
var screens = require('point_of_sale.screens');
var core = require('web.core');
var rpc = require('web.rpc');
var session = require('web.session');

var QWeb = core.qweb;
var _t = core._t;

// At POS Startup, load the floors, and add them to the pos model
models.load_models({
    model: 'restaurant.floor',
    fields: ['name','background_color','table_ids','sequence'],
    domain: function(self){ return [['pos_config_id','=',self.config.id]]; },
    loaded: function(self,floors){
        self.floors = floors;
        self.floors_by_id = {};
        for (var i = 0; i < floors.length; i++) {
            floors[i].tables = [];
            self.floors_by_id[floors[i].id] = floors[i];
        }

        // Make sure they display in the correct order
        self.floors = self.floors.sort(function(a,b){ return a.sequence - b.sequence; });

        // Ignore floorplan features if no floor specified.
        self.config.iface_floorplan = !!self.floors.length;
    },
});

// At POS Startup, after the floors are loaded, load the tables, and associate
// them with their floor.
models.load_models({
    model: 'restaurant.table',
    fields: ['name','width','height','position_h','position_v','shape','floor_id','color','seats'],
    loaded: function(self,tables){
        self.tables_by_id = {};
        for (var i = 0; i < tables.length; i++) {
            self.tables_by_id[tables[i].id] = tables[i];
            var floor = self.floors_by_id[tables[i].floor_id[0]];
            if (floor) {
                floor.tables.push(tables[i]);
                tables[i].floor = floor;
            }
        }
    },
});

// The Table GUI element, should always be a child of the FloorScreenWidget
var TableWidget = PosBaseWidget.extend({
    template: 'TableWidget',
    init: function(parent, options){
        this._super(parent, options);
        this.table    = options.table;
        this.selected = false;
        this.moved    = false;
        this.dragpos  = {x:0, y:0};
    },
    // computes the absolute position of a DOM mouse event, used
    // when resizing tables
    event_position: function(event){
        if(event.touches && event.touches[0]){
            return {x: event.touches[0].screenX, y: event.touches[0].screenY};
        }else{
            return {x: event.screenX, y: event.screenY};
        }
    },
    // when a table is clicked, go to the table's orders
    // but if we're editing, we select/deselect it.
    click_handler: function(){
        var self = this;
        var floorplan = this.getParent();
        if (floorplan.editing) {
            setTimeout(function(){  // in a setTimeout to debounce with drag&drop start
                if (!self.dragging) {
                    if (self.moved) {
                        self.moved = false;
                    } else if (!self.selected) {
                        self.getParent().select_table(self);
                    } else {
                        self.getParent().deselect_tables();
                    }
                }
            },50);
        } else {
            floorplan.pos.set_table(this.table);
        }
    },
    // drag and drop for moving the table, at drag start
    dragstart_handler:   function(event, ui){
        this.dragging = true;
    },
    // drag and drop for moving the table, at drag end
    dragend_handler:   function(event, ui){
        this.dragging = false;
        this.moved = true;
        this.table.position_h = ui.position.left - this.table.width/2;
        this.table.position_v = ui.position.top - this.table.height/2;
        this.$el.css(this.table_style());
    },
    // drag and dropping the resizing handles
    handle_dragmove_handler: function(event, ui) {
        this.moved = true;
        this.table.width  = ui.size.width;
        this.table.height = ui.size.height;
        this.table.position_h = ui.position.left - ui.originalSize.width/2;
        this.table.position_v = ui.position.top - ui.originalSize.height/2;
        this.$el.css(this.table_style());
    },
    set_table_color: function(color){
        this.table.color = _.escape(color);
        this.$el.css({'background': this.table.color});
    },
    set_table_name: function(name){
        if (name) {
            this.table.name = name;
            this.renderElement();
        }
    },
    set_table_seats: function(seats){
        if (seats) {
            this.table.seats = Number(seats);
            this.renderElement();
        }
    },
    // The table's positioning is handled via css absolute positioning,
    // which is handled here.
    table_style: function(){
        var table = this.table;
        function unit(val){ return '' + val + 'px'; }
        var style = {
            'width':        unit(table.width),
            'height':       unit(table.height),
            'line-height':  unit(table.height),
            'margin-left':  unit(-table.width/2),
            'margin-top':   unit(-table.height/2),
            'top':          unit(table.position_v + table.height/2),
            'left':         unit(table.position_h + table.width/2),
            'border-radius': table.shape === 'round' ?
                    unit(Math.max(table.width,table.height)/2) : '3px',
        };
        if (table.color) {
            style.background = table.color;
        }
        if (table.height >= 150 && table.width >= 150) {
            style['font-size'] = '32px';
        }

        return style;
    },
    // convert the style dictionary to a ; separated string for inclusion in templates
    table_style_str: function(){
        var style = this.table_style();
        var str = "";
        var s;
        for (s in style) {
            str += s + ":" + style[s] + "; ";
        }
        return str;
    },
    // select the table (should be called via the floorplan)
    select: function() {
        this.selected = true;
        this.renderElement();

        this.$el.resizable({
            handles: 'all',
            resize: this.handle_dragmove_handler.bind(this),
        });

        this.$el.draggable({
            stop: this.dragend_handler.bind(this),
        });
    },
    // deselect the table (should be called via the floorplan)
    deselect: function() {
        this.selected = false;
        this.renderElement();
        this.save_changes();
    },
    // sends the table's modification to the server
    save_changes: function(){
        var self   = this;
        var fields = _.find(this.pos.models,function(model){ return model.model === 'restaurant.table'; }).fields;

        // we need a serializable copy of the table, containing only the fields defined on the server
        var serializable_table = {};
        for (var i = 0; i < fields.length; i++) {
            if (typeof this.table[fields[i]] !== 'undefined') {
                serializable_table[fields[i]] = this.table[fields[i]];
            }
        }
        // and the id ...
        serializable_table.id = this.table.id;

        rpc.query({
                model: 'restaurant.table',
                method: 'create_from_ui',
                args: [serializable_table],
            })
            .then(function (table_id){
                rpc.query({
                        model: 'restaurant.table',
                        method: 'search_read',
                        args: [[['id', '=', table_id]], fields],
                        limit: 1,
                    })
                    .then(function (result){
                        var table = result[0];
                        for (var field in table) {
                            self.table[field] = table[field];
                        }
                        self.pos.tables_by_id[table.id] = self.table;
                        // If selected, render with drag and resize event handlers.
                        if (!self.selected) {
                            self.renderElement();
                        } else {
                            self.select();
                        }
                    });
            }, function(type,err) {
                    self.gui.show_sync_error_popup();
            });
    },
    // destroy the table.  We do not really destroy it, we set it
    // to inactive so that it doesn't show up anymore, but it still
    // available on the database for the orders that depend on it.
    trash: function(){
        var self  = this;
        rpc.query({
                model: 'restaurant.table',
                method: 'create_from_ui',
                args: [{'active':false,'id':this.table.id}],
            })
            .then(function (table_id){
                // Removing all references from the table and the table_widget in in the UI ...
                for (var i = 0; i < self.pos.floors.length; i++) {
                    var floor = self.pos.floors[i];
                    for (var j = 0; j < floor.tables.length; j++) {
                        if (floor.tables[j].id === table_id) {
                            floor.tables.splice(j,1);
                            break;
                        }
                    }
                }
                var floorplan = self.getParent();
                for (var i = 0; i < floorplan.table_widgets.length; i++) {
                    if (floorplan.table_widgets[i] === self) {
                        floorplan.table_widgets.splice(i,1);
                    }
                }
                if (floorplan.selected_table === self) {
                    floorplan.selected_table = null;
                }
                floorplan.update_toolbar();
                self.destroy();
            }, function(type, err) {
                self.gui.show_sync_error_popup();
            });
    },
    get_notifications: function(){  //FIXME : Make this faster
        var orders = this.pos.get_table_orders(this.table);
        var notifications = {};
        for (var i = 0; i < orders.length; i++) {
            if (orders[i].hasChangesToPrint()) {
                notifications.printing = true;
                break;
            } else if (orders[i].hasSkippedChanges()) {
                notifications.skipped  = true;
            }
        }
        return notifications;
    },
    update_click_handlers: function(editing){
        var self = this;
        this.$el.off('mouseup touchend touchcancel click dragend');

        if (editing) {
            this.$el.on('mouseup touchend touchcancel', function(event){ self.click_handler(event,$(this)); });
        } else {
            this.$el.on('click dragend', function(event){ self.click_handler(event,$(this)); });
        }
    },
    renderElement: function(){
        var self = this;
        this.order_count    = this.table.order_count !== undefined ?
            this.table.order_count :
            this.pos.get_table_orders(this.table)
                .filter(o => o.orderlines.length !== 0 || o.paymentlines.length !==0).length;
        this.customer_count = this.pos.get_customer_count(this.table);
        this.fill           = Math.min(1,Math.max(0,this.customer_count / this.table.seats));
        this.notifications  = this.get_notifications();
        this._super();

        this.update_click_handlers();
    },
});

// The screen that allows you to select the floor, see and select the table,
// as well as edit them.
var FloorScreenWidget = screens.ScreenWidget.extend({
    template: 'FloorScreenWidget',

    // Ignore products, discounts, and client barcodes
    barcode_product_action: function(code){},
    barcode_discount_action: function(code){},
    barcode_client_action: function(code){},

    init: function(parent, options) {
        this._super(parent, options);
        this.floor = this.pos.floors[0];
        this.table_widgets = [];
        this.selected_table = null;
        this.editing = false;
    },
    _table_longpolling: function(){
        if (this.editing) {
            return;
        }
        var self = this;
        rpc.query({
            model: 'pos.config',
            method: 'get_tables_order_count',
            args: [this.pos.config.id],
        })
        .then(function (result){
            result.forEach(function(table){
                var table_obj = self.pos.tables_by_id[table.id];
                var unsynced_orders = self.pos.get_table_orders(table_obj)
                    .filter(o => o.server_id === undefined &&
                            (o.orderlines.length !== 0 || o.paymentlines.length !== 0)).length
                table_obj.order_count = table.orders + unsynced_orders;
            });
            self.table_widgets.forEach(
                    function(tw){
                        tw.renderElement();
                    });
        }, function(type,err) {
            self.table_widgets.forEach(
                    function(tw){
                        tw.table.order_count = self.pos.get_table_orders(tw.table)
                            .filter(o => o.orderlines.length !== 0 || o.paymentlines.length !== 0).length
                        tw.renderElement();
                    });
        });
    },
    hide: function(){
        this._super();
        if (this.editing) {
            this.toggle_editing();
        }
        this.chrome.widget.order_selector.show();
        clearInterval(this.table_longpolling);
    },
    show: function(){
        this._super();
        this.chrome.widget.order_selector.hide();
        for (var i = 0; i < this.table_widgets.length; i++) {
            this.table_widgets[i].renderElement();
        }
        this.check_empty_floor();

        this._table_longpolling();
        this.table_longpolling = setInterval(this._table_longpolling.bind(this), 5000);
    },
    click_floor_button: function(event,$el){
        var floor = this.pos.floors_by_id[$el.data('id')];
        if (floor !== this.floor) {
            if (this.editing) {
                this.toggle_editing();
            }
            this.floor = floor;
            this.selected_table = null;
            this.renderElement();
            this.check_empty_floor();
        }
    },
    background_image_url: function(floor) {
        return '/web/image?model=restaurant.floor&id='+floor.id+'&field=background_image';
    },
    get_floor_style: function() {
        var style = "";
        if (this.floor.background_image) {
            style += "background-image: url(" + this.background_image_url(this.floor) + "); ";
        }
        if (this.floor.background_color) {
            style += "background-color: " + _.escape(this.floor.background_color) + ";";
        }
        return style;
    },
    set_background_color: function(background) {
        var self = this;
        this.floor.background_color = background;
        rpc.query({
                model: 'restaurant.floor',
                method: 'write',
                args: [[this.floor.id], {'background_color': background}],
            })
            .guardedCatch(function (){
                self.gui.show_sync_error_popup();
            });
        this.$('.floor-map').css({"background-color": _.escape(background)});
    },
    deselect_tables: function(){
        for (var i = 0; i < this.table_widgets.length; i++) {
            var table = this.table_widgets[i];
            if (table.selected) {
                table.deselect();
            }
        }
        this.selected_table = null;
        this.update_toolbar();
    },
    select_table: function(table_widget){
        if (!table_widget.selected) {
            this.deselect_tables();
            table_widget.select();
            this.selected_table = table_widget;
            this.update_toolbar();
        }
    },
    tool_shape_action: function(){
        if (this.selected_table) {
            var table = this.selected_table.table;
            if (table.shape === 'square') {
                table.shape = 'round';
            } else {
                table.shape = 'square';
            }
            this.selected_table.renderElement();
            this.update_toolbar();
        }
    },
    tool_colorpicker_open: function(){
        this.$('.color-picker').addClass('oe_hidden');
        if (this.selected_table) {
            this.$('.color-picker.fg-picker').removeClass('oe_hidden');
        } else {
            this.$('.color-picker.bg-picker').removeClass('oe_hidden');
        }
    },
    tool_colorpicker_pick: function(event,$el){
        if (this.selected_table) {
            this.selected_table.set_table_color($el[0].style['background-color']);
        } else {
            this.set_background_color($el[0].style['background-color']);
        }
    },
    tool_colorpicker_close: function(){
        this.$('.color-picker').addClass('oe_hidden');
    },
    tool_rename_table: function(){
        var self = this;
        if (this.selected_table) {
            this.gui.show_popup('textinput',{
                'title':_t('Table Name ?'),
                'value': this.selected_table.table.name,
                'confirm': function(value) {
                    self.selected_table.set_table_name(value);
                },
            });
        }
    },
    tool_change_seats: function(){
        var self = this;
        if (this.selected_table) {
            this.gui.show_popup('number',{
                'title':_t('Number of Seats ?'),
                'cheap': true,
                'value': this.selected_table.table.seats,
                'confirm': function(value) {
                    self.selected_table.set_table_seats(value);
                },
            });
        }
    },
    tool_duplicate_table: function(){
        if (this.selected_table) {
            var tw = this.create_table(this.selected_table.table);
            tw.table.position_h += 10;
            tw.table.position_v += 10;
            tw.save_changes();
            this.select_table(tw);
        }
    },
    tool_new_table: function(){
        var tw = this.create_table({
            'position_v': 100,
            'position_h': 100,
            'width': 75,
            'height': 75,
            'shape': 'square',
            'seats': 1,
        });
        tw.save_changes();
        this.select_table(tw);
        this.check_empty_floor();
    },
    new_table_name: function(name){
        if (name) {
            var num = Number((name.match(/\d+/g) || [])[0] || 0);
            var str = (name.replace(/\d+/g,''));
            var n   = {num: num, str:str};
                n.num += 1;
            this.last_name = n;
        } else if (this.last_name) {
            this.last_name.num += 1;
        } else {
            this.last_name = {num: 1, str:'T'};
        }
        return '' + this.last_name.str + this.last_name.num;
    },
    create_table: function(params) {
        var table = {};
        for (var p in params) {
            table[p] = params[p];
        }

        table.name = this.new_table_name(params.name);

        delete table.id;
        table.floor_id = [this.floor.id,''];
        table.floor = this.floor;

        this.floor.tables.push(table);
        var tw = new TableWidget(this,{table: table});
            tw.appendTo('.floor-map .tables');
        this.table_widgets.push(tw);
        return tw;
    },
    tool_trash_table: function(){
        var self = this;
        if (this.selected_table) {
            this.gui.show_popup('confirm',{
                'title':  _t('Are you sure ?'),
                'comment':_t('Removing a table cannot be undone'),
                'confirm': function(){
                    self.selected_table.trash();
                },
            });
        }
    },
    toggle_editing: function(){
        this.editing = !this.editing;
        this.update_toolbar();
            this.update_table_click_handlers();

        if (!this.editing) {
            this.deselect_tables();
            }
        },
        update_table_click_handlers: function(){
            for (var i = 0; i < this.table_widgets.length; ++i) {
                if (this.editing) {
                    this.table_widgets[i].update_click_handlers("editing");
                } else {
                    this.table_widgets[i].update_click_handlers();
                }
        }
    },
    check_empty_floor: function(){
        if (!this.floor.tables.length) {
            if (!this.editing) {
                this.toggle_editing();
            }
            this.$('.empty-floor').removeClass('oe_hidden');
        } else {
            this.$('.empty-floor').addClass('oe_hidden');
        }
    },
    update_toolbar: function(){

        if (this.editing) {
            this.$('.edit-bar').removeClass('oe_hidden');
            this.$('.edit-button.editing').addClass('active');
        } else {
            this.$('.edit-bar').addClass('oe_hidden');
            this.$('.edit-button.editing').removeClass('active');
        }

        if (this.selected_table) {
            this.$('.needs-selection').removeClass('disabled');
            var table = this.selected_table.table;
            if (table.shape === 'square') {
                this.$('.button-option.square').addClass('oe_hidden');
                this.$('.button-option.round').removeClass('oe_hidden');
            } else {
                this.$('.button-option.square').removeClass('oe_hidden');
                this.$('.button-option.round').addClass('oe_hidden');
            }
        } else {
            this.$('.needs-selection').addClass('disabled');
        }
        this.tool_colorpicker_close();
    },
    renderElement: function(){
        var self = this;

        // cleanup table widgets from previous renders
        for (var i = 0; i < this.table_widgets.length; i++) {
            this.table_widgets[i].destroy();
        }

        this.table_widgets = [];

        this._super();

        for (var i = 0; i < this.floor.tables.length; i++) {
            var tw = new TableWidget(this,{
                table: this.floor.tables[i],
            });
            tw.appendTo(this.$('.floor-map .tables'));
            this.table_widgets.push(tw);
        }

        $('body').on('keyup', function (event) {
            if (event.which === $.ui.keyCode.ESCAPE) {
                if(self.editing) {
                    self.toggle_editing();
                }
            }
        });

        this.$('.floor-selector .button').click(function(event){
            self.click_floor_button(event,$(this));
        });

        this.$('.edit-button.shape').click(function(){
            self.tool_shape_action();
        });

        this.$('.edit-button.color').click(function(){
            self.tool_colorpicker_open();
        });

        this.$('.edit-button.dup-table').click(function(){
            self.tool_duplicate_table();
        });

        this.$('.edit-button.new-table').click(function(){
            self.tool_new_table();
        });

        this.$('.edit-button.rename').click(function(){
            self.tool_rename_table();
        });

        this.$('.edit-button.seats').click(function(){
            self.tool_change_seats();
        });

        this.$('.edit-button.trash').click(function(){
            self.tool_trash_table();
        });

        this.$('.color-picker .close-picker').click(function(event){
            self.tool_colorpicker_close();
            event.stopPropagation();
        });

        this.$('.color-picker .color').click(function(event){
            self.tool_colorpicker_pick(event,$(this));
            event.stopPropagation();
        });

        this.$('.edit-button.editing').click(function(){
            self.toggle_editing();
        });

        this.$('.floor-map,.floor-map .tables').click(function(event){
            if (event.target === self.$('.floor-map')[0] ||
                event.target === self.$('.floor-map .tables')[0]) {
                self.deselect_tables();
            }
        });

        this.$('.color-picker .close-picker').click(function(event){
            self.tool_colorpicker_close();
            event.stopPropagation();
        });

        this.update_toolbar();

    },
});

screens.ProductScreenWidget.include({
    show: function () {
        var self = this;
        this._super();
        if (this.pos.config.iface_floorplan) {
            this.$el.bind("mousemove mousedown touchstart click scroll keypress", function () {
                self.set_idle_timer();
            });
            this.set_idle_timer();
        }
    },
    hide: function () {
        this._super();
        if (this.pos.config.iface_floorplan) {
            this.$el.unbind("mousemove mousedown touchstart click scroll keypress");
            clearTimeout(this.idle_timer);
        }
    },
    /**
     * Set a timeout to go back to the floorplan and clear the previous timeout
     *
     * @param {number} timeout, optional timeout in miliseconds, default one minute.
     */
    set_idle_timer: function(timeout=60000) {
        var self = this;
        clearTimeout(this.idle_timer);
        this.idle_timer = setTimeout(function () {
            self.pos.set_table(null);
        }, timeout);
    },
});

gui.define_screen({
    'name': 'floors',
    'widget': FloorScreenWidget,
    'condition': function(){
        return this.pos.config.iface_floorplan;
    },
});

gui.Gui.include({
    show_sync_error_popup: function() {
        if (this.show_sync_errors) {
            this.show_popup('error-sync',{
                'title':_t('Changes could not be saved'),
                'body': _t('You must be connected to the internet to save your changes.\n\n' +
                        'Changes made to previously synced orders will get lost at the next sync.\n' +
                        'Orders that where not synced before will be synced next time you open and close the same table.'),
            });
        }
    },
});

// Add the FloorScreen to the GUI, and set it as the default screen
chrome.Chrome.include({
    build_widgets: function(){
        this._super();
        if (this.pos.config.iface_floorplan) {
            this.gui.set_startup_screen('floors');
        }
    },
});

// New orders are now associated with the current table, if any.
var _super_order = models.Order.prototype;
models.Order = models.Order.extend({
    initialize: function() {
        _super_order.initialize.apply(this,arguments);
        if (!this.table) {
            this.table = this.pos.table;
        }
        this.customer_count = this.customer_count || 1;
        this.save_to_db();
    },
    export_as_JSON: function() {
        var json = _super_order.export_as_JSON.apply(this,arguments);
        json.table     = this.table ? this.table.name : undefined;
        json.table_id  = this.table ? this.table.id : false;
        json.floor     = this.table ? this.table.floor.name : false;
        json.floor_id  = this.table ? this.table.floor.id : false;
        json.customer_count = this.customer_count;
        return json;
    },
    init_from_JSON: function(json) {
        _super_order.init_from_JSON.apply(this,arguments);
        this.table = this.pos.tables_by_id[json.table_id];
        this.floor = this.table ? this.pos.floors_by_id[json.floor_id] : undefined;
        this.customer_count = json.customer_count || 1;
    },
    export_for_printing: function() {
        var json = _super_order.export_for_printing.apply(this,arguments);
        json.table = this.table ? this.table.name : undefined;
        json.floor = this.table ? this.table.floor.name : undefined;
        json.customer_count = this.get_customer_count();
        return json;
    },
    get_customer_count: function(){
        return this.customer_count;
    },
    set_customer_count: function(count) {
        this.customer_count = Math.max(count,0);
        this.trigger('change');
    },
});

// We need to modify the OrderSelector to hide itself when we're on
// the floor plan
chrome.OrderSelectorWidget.include({
    floor_button_click_handler: function(){
        this.pos.set_table(null);
    },
    hide: function(){
        this.$el.addClass('oe_invisible');
    },
    show: function(){
        this.$el.removeClass('oe_invisible');
    },
    renderElement: function(){
        var self = this;
        this._super();
        if (this.pos.config.iface_floorplan) {
            if (this.pos.get_order()) {
                if (this.pos.table && this.pos.table.floor) {
                    this.$('.orders').prepend(QWeb.render('BackToFloorButton',{table: this.pos.table, floor:this.pos.table.floor}));
                    this.$('.floor-button').click(function(){
                        self.floor_button_click_handler();
                    });
                }
                this.$el.removeClass('oe_invisible');
            } else {
                this.$el.addClass('oe_invisible');
            }
        }
    },
});

// We need to change the way the regular UI sees the orders, it
// needs to only see the orders associated with the current table,
// and when an order is validated, it needs to go back to the floor map.
//
// And when we change the table, we must create an order for that table
// if there is none.
var _super_posmodel = models.PosModel.prototype;
models.PosModel = models.PosModel.extend({
    after_load_server_data: function() {
        var res = _super_posmodel.after_load_server_data.call(this);
        if (this.config.iface_floorplan) {
            this.table = null;
        }
        return res;
    },

    transfer_order_to_different_table: function () {
        this.order_to_transfer_to_different_table = this.get_order();

        // go to 'floors' screen, this will set the order to null and
        // eventually this will cause the gui to go to its
        // default_screen, which is 'floors'
        this.set_table(null);
    },

    remove_from_server_and_set_sync_state: function(ids_to_remove){
        var self = this;
        this.set_synch('connecting', ids_to_remove.length);
        self._remove_from_server(ids_to_remove)
            .then(function(server_ids) {
                self.set_synch('connected');
            }).catch(function(reason){
                self.set_synch('error');
            });
    },

    /**
     * Request the orders of the table with given id.
     * @param {number} table_id.
     * @param {dict} options.
     * @param {number} options.timeout optional timeout parameter for the rpc call.
     * @return {Promise}
     */
    _get_from_server: function (table_id, options) {
        options = options || {};
        var self = this;
        var timeout = typeof options.timeout === 'number' ? options.timeout : 7500;
        return new Promise( function(resolve, reject) {
            rpc.query({
                model: 'pos.order',
                method: 'get_table_draft_orders',
                args: [table_id],
                kwargs: {context: session.user_context},
            }, {
                timeout: timeout,
                shadow: false,
            }).then(function(orders){
                orders.forEach(function(order) {
                    order.multiprint_resume = JSON.parse(order.multiprint_resume? order.multiprint_resume: false);
                });
                resolve(orders);
            }).catch(function(err) {
                reject(err);
            });
        });
    },

    transfer_order_to_table: function(table) {
        this.order_to_transfer_to_different_table.table = table;
        this.order_to_transfer_to_different_table.save_to_db();
    },

    push_order_for_transfer: function(order_ids, table_orders) {
        order_ids.push(this.order_to_transfer_to_different_table.uid);
        table_orders.push(this.order_to_transfer_to_different_table);
    },

    clean_table_transfer: function(table) {
        if (this.order_to_transfer_to_different_table && table) {
            this.order_to_transfer_to_different_table = null;
            this.set_table(table);
        }
    },

    sync_from_server: function(table, table_orders, order_ids) {
        var self = this;
        var ids_to_remove = this.db.get_ids_to_remove_from_server();
        var orders_to_sync = this.db.get_unpaid_orders_to_sync(order_ids);
        if (orders_to_sync.length) {
            this.set_synch('connecting', orders_to_sync.length);
            this._save_to_server(orders_to_sync, {'draft': true}).then(function (server_ids) {
                server_ids.forEach(server_id => self.update_table_order(server_id, table_orders));
                if (!ids_to_remove.length) {
                    self.set_synch('connected');
                } else {
                    self.remove_from_server_and_set_sync_state(ids_to_remove);
                }
            }).catch(function(reason){
                self.set_synch('error');
            }).finally(function(){
                self.clean_table_transfer(table);
            });
        } else {
            if (ids_to_remove.length) {
                self.remove_from_server_and_set_sync_state(ids_to_remove);
            }
            self.clean_table_transfer(table);
        }
    },

    update_table_order: function(server_id, table_orders) {
        const order = table_orders.find(o => o.name === server_id.pos_reference);
        if (order) {
            order.server_id = server_id.id;
            order.save_to_db();
        }
        return order;
    },

    set_order_on_table: function() {
        var orders = this.get_order_list();
        if (orders.length) {
            this.set_order(orders[0]); // and go to the first one ...
        } else {
            this.add_new_order();  // or create a new order with the current table
        }
    },

    sync_to_server: function(table) {
        var self = this;
        var ids_to_remove = this.db.get_ids_to_remove_from_server();

        clearInterval(this.table_longpolling);

        this.set_synch('connecting', 1);
        this._get_from_server(table.id).then(function (server_orders) {
            var orders = self.get_order_list();
            orders.forEach(function(order){
                if (order.server_id){
                    self.get("orders").remove(order);
                    order.destroy();
                }
            });
            server_orders.forEach(function(server_order){
                if (server_order.lines.length){
                    var new_order = new models.Order({},{pos: self, json: server_order});
                    self.get("orders").add(new_order);
                    new_order.save_to_db();
                }
            })
            if (!ids_to_remove.length) {
                self.set_synch('connected');
            } else {
                self.remove_from_server_and_set_sync_state(ids_to_remove);
            }
        }).catch(function(reason){
            self.set_synch('error');
        }).finally(function(){
            self.set_order_on_table();
        });
    },

    get_order_with_uid: function() {
        var order_ids = [];
        this.get_order_list().forEach(function(o){
            order_ids.push(o.uid);
        });

        return order_ids;
    },

    /**
     * Changes the current table.
     *
     * Switch table and make sure all nececery syncing tasks are done.
     * @param {object} table.
     */
    set_table: function(table) {
        if(!table){
            this.sync_from_server(table, this.get_order_list(), this.get_order_with_uid());
            this.set_order(null);
        } else if (this.order_to_transfer_to_different_table) {
            var order_ids = this.get_order_with_uid();

            this.transfer_order_to_table(table);
            this.push_order_for_transfer(order_ids, this.get_order_list());

            this.sync_from_server(table, this.get_order_list(), order_ids);
            this.set_order(null);
        } else {
            this.table = table;
            this.sync_to_server(table);
        }
    },

    // if we have tables, we do not load a default order, as the default order will be
    // set when the user selects a table.
    set_start_order: function() {
        if (!this.config.iface_floorplan) {
            _super_posmodel.set_start_order.apply(this,arguments);
        }
    },

    // we need to prevent the creation of orders when there is no
    // table selected.
    add_new_order: function() {
        if (this.config.iface_floorplan) {
            if (this.table) {
                return _super_posmodel.add_new_order.call(this);
            } else {
                console.warn("WARNING: orders cannot be created when there is no active table in restaurant mode");
                return undefined;
            }
        } else {
            return _super_posmodel.add_new_order.apply(this,arguments);
        }
    },


    // get the list of unpaid orders (associated to the current table)
    get_order_list: function() {
        var orders = _super_posmodel.get_order_list.call(this);
        if (!this.config.iface_floorplan) {
            return orders;
        } else if (!this.table) {
            return [];
        } else {
            var t_orders = [];
            for (var i = 0; i < orders.length; i++) {
                if ( orders[i].table === this.table) {
                    t_orders.push(orders[i]);
                }
            }
            return t_orders;
        }
    },

    // get the list of orders associated to a table. FIXME: should be O(1)
    get_table_orders: function(table) {
        var orders   = _super_posmodel.get_order_list.call(this);
        var t_orders = [];
        for (var i = 0; i < orders.length; i++) {
            if (orders[i].table === table) {
                t_orders.push(orders[i]);
            }
        }
        return t_orders;
    },

    // get customer count at table
    get_customer_count: function(table) {
        var orders = this.get_table_orders(table);
        var count  = 0;
        for (var i = 0; i < orders.length; i++) {
            count += orders[i].get_customer_count();
        }
        return count;
    },

    // When we validate an order we go back to the floor plan.
    // When we cancel an order and there is multiple orders
    // on the table, stay on the table.
    on_removed_order: function(removed_order,index,reason){
        if (this.config.iface_floorplan) {
            var order_list = this.get_order_list();
            if (reason === 'abandon') {
                this.db.set_order_to_remove_from_server(removed_order);
            }
            if( (reason === 'abandon' || removed_order.temporary) && order_list.length > 0){
                this.set_order(order_list[index] || order_list[order_list.length -1]);
            }else{
                // back to the floor plan
                this.set_table(null);
            }
        } else {
            _super_posmodel.on_removed_order.apply(this,arguments);
        }
    },


});

var TableGuestsButton = screens.ActionButtonWidget.extend({
    template: 'TableGuestsButton',
    guests: function() {
        if (this.pos.get_order()) {
            return this.pos.get_order().customer_count;
        } else {
            return 0;
        }
    },
    button_click: function() {
        var self = this;
        this.gui.show_popup('number', {
            'title':  _t('Guests ?'),
            'cheap': true,
            'value':   this.pos.get_order().customer_count,
            'confirm': function(value) {
                value = Math.max(1,Number(value));
                self.pos.get_order().set_customer_count(value);
                self.renderElement();
            },
        });
    },
});

screens.OrderWidget.include({
    update_summary: function(){
        this._super();
        if (this.getParent().action_buttons &&
            this.getParent().action_buttons.guests) {
            this.getParent().action_buttons.guests.renderElement();
        }
    },
});

screens.define_action_button({
    'name': 'guests',
    'widget': TableGuestsButton,
    'condition': function(){
        return this.pos.config.iface_floorplan;
    },
});

var TransferOrderButton = screens.ActionButtonWidget.extend({
    template: 'TransferOrderButton',
    button_click: function() {
        this.pos.transfer_order_to_different_table();
    },
});

screens.define_action_button({
    'name': 'transfer',
    'widget': TransferOrderButton,
    'condition': function(){
        return this.pos.config.iface_floorplan;
    },
});

return {
    TableGuestsButton: TableGuestsButton,
    TransferOrderButton:TransferOrderButton,
    TableWidget: TableWidget,
    FloorScreenWidget: FloorScreenWidget,
};

});

```

## File: static\src\js\multiprint.js

```javascript
odoo.define('pos_restaurant.multiprint', function (require) {
"use strict";

var models = require('point_of_sale.models');
var screens = require('point_of_sale.screens');
var core = require('web.core');
var Printer = require('point_of_sale.Printer').Printer;

var QWeb = core.qweb;

models.PosModel = models.PosModel.extend({
    create_printer: function (config) {
        var url = config.proxy_ip || '';
        if(url.indexOf('//') < 0) {
            url = window.location.protocol + '//' + url;
        }
        if(url.indexOf(':', url.indexOf('//') + 2) < 0 && window.location.protocol !== 'https:') {
            url = url + ':8069';
        }
        return new Printer(url, this);
    },
});

models.load_models({
    model: 'restaurant.printer',
    fields: ['name','proxy_ip','product_categories_ids', 'printer_type'],
    domain: null,
    loaded: function(self,printers){
        var active_printers = {};
        for (var i = 0; i < self.config.printer_ids.length; i++) {
            active_printers[self.config.printer_ids[i]] = true;
        }

        self.printers = [];
        self.printers_categories = {}; // list of product categories that belong to
                                       // one or more order printer

        for(var i = 0; i < printers.length; i++){
            if(active_printers[printers[i].id]){
                var printer = self.create_printer(printers[i]);
                printer.config = printers[i];
                self.printers.push(printer);

                for (var j = 0; j < printer.config.product_categories_ids.length; j++) {
                    self.printers_categories[printer.config.product_categories_ids[j]] = true;
                }
            }
        }
        self.printers_categories = _.keys(self.printers_categories);
        self.config.iface_printers = !!self.printers.length;
    },
});

var _super_orderline = models.Orderline.prototype;

models.Orderline = models.Orderline.extend({
    initialize: function() {
        _super_orderline.initialize.apply(this,arguments);
        if (!this.pos.config.iface_printers) {
            return;
        }
        if (typeof this.mp_dirty === 'undefined') {
            // mp dirty is true if this orderline has changed
            // since the last kitchen print
            // it's left undefined if the orderline does not
            // need to be printed to a printer.

            this.mp_dirty = this.printable() || undefined;
        }
        if (!this.mp_skip) {
            // mp_skip is true if the cashier want this orderline
            // not to be sent to the kitchen
            this.mp_skip  = false;
        }
    },
    // can this orderline be potentially printed ?
    printable: function() {
        return this.pos.db.is_product_in_category(this.pos.printers_categories, this.get_product().id);
    },
    init_from_JSON: function(json) {
        _super_orderline.init_from_JSON.apply(this,arguments);
        this.mp_dirty = json.mp_dirty;
        this.mp_skip  = json.mp_skip;
    },
    export_as_JSON: function() {
        var json = _super_orderline.export_as_JSON.apply(this,arguments);
        json.mp_dirty = this.mp_dirty;
        json.mp_skip  = this.mp_skip;
        return json;
    },
    set_quantity: function(quantity) {
        if (this.pos.config.iface_printers && quantity !== this.quantity && this.printable()) {
            this.mp_dirty = true;
        }
        _super_orderline.set_quantity.apply(this,arguments);
    },
    can_be_merged_with: function(orderline) {
        return (!this.mp_skip) &&
               (!orderline.mp_skip) &&
               _super_orderline.can_be_merged_with.apply(this,arguments);
    },
    set_skip: function(skip) {
        if (this.mp_dirty && skip && !this.mp_skip) {
            this.mp_skip = true;
            this.trigger('change',this);
        }
        if (this.mp_skip && !skip) {
            this.mp_dirty = true;
            this.mp_skip  = false;
            this.trigger('change',this);
        }
    },
    set_dirty: function(dirty) {
        if (this.mp_dirty !== dirty) {
            this.mp_dirty = dirty;
            this.trigger('change', this);
        }
    },
    get_line_diff_hash: function(){
        if (this.get_note()) {
            return this.id + '|' + this.get_note();
        } else {
            return '' + this.id;
        }
    },
});

var _super_posModel = models.PosModel.prototype;

models.PosModel = models.PosModel.extend({
    _save_to_server: function (orders, options) {
        orders.forEach(function(order){
            if(order.data.multiprint_resume && typeof(order.data.multiprint_resume) === "object")
                order.data.multiprint_resume = JSON.stringify(order.data.multiprint_resume? order.data.multiprint_resume : false);
        });
        return _super_posModel._save_to_server.apply(this,arguments);
    }
});

screens.OrderWidget.include({
    render_orderline: function(orderline) {
        var node = this._super(orderline);
        if (this.pos.config.iface_printers) {
            if (orderline.mp_skip) {
                node.classList.add('skip');
            } else if (orderline.mp_dirty) {
                node.classList.add('dirty');
            }
        }
        return node;
    },
    click_line: function(line, event) {
        if (!this.pos.config.iface_printers) {
            this._super(line, event);
        } else if (this.pos.get_order().selected_orderline !== line) {
            this.mp_dbclk_time = (new Date()).getTime();
        } else if (!this.mp_dbclk_time) {
            this.mp_dbclk_time = (new Date()).getTime();
        } else if (this.mp_dbclk_time + 500 > (new Date()).getTime()) {
            line.set_skip(!line.mp_skip);
            this.mp_dbclk_time = 0;
        } else {
            this.mp_dbclk_time = (new Date()).getTime();
        }

        this._super(line, event);
    },
});

var _super_order = models.Order.prototype;
models.Order = models.Order.extend({
    build_line_resume: function(){
        var resume = {};
        this.orderlines.each(function(line){
            if (line.mp_skip) {
                return;
            }
            var qty  = Number(line.get_quantity());
            var note = line.get_note();
            var product_id = line.get_product().id;
            var product_resume = product_id in resume ? resume[product_id] : {
                product_name_wrapped: line.generate_wrapped_product_name(),
                qties: {},
            };
            if (note in product_resume['qties']) product_resume['qties'][note] += qty;
            else product_resume['qties'][note] = qty;
            resume[product_id] = product_resume;
        });
        return resume;
    },
    saveChanges: function(){
        this.saved_resume = this.build_line_resume();
        this.orderlines.each(function(line){
            line.set_dirty(false);
        });
        this.trigger('change',this);
    },
    computeChanges: function(categories){
        var current_res = this.build_line_resume();
        var old_res     = this.saved_resume || {};
        var json        = this.export_as_JSON();
        var add = [];
        var rem = [];
        var pid, note;

        for (pid in current_res) {
            for (note in current_res[pid]['qties']) {
                var curr = current_res[pid];
                var old  = old_res[pid] || {};
                var found = pid in old_res && note in old_res[pid]['qties'];

                if (!found) {
                    add.push({
                        'id':       pid,
                        'name':     this.pos.db.get_product_by_id(pid).display_name,
                        'name_wrapped': curr.product_name_wrapped,
                        'note':     note,
                        'qty':      curr['qties'][note],
                    });
                } else if (old['qties'][note] < curr['qties'][note]) {
                    add.push({
                        'id':       pid,
                        'name':     this.pos.db.get_product_by_id(pid).display_name,
                        'name_wrapped': curr.product_name_wrapped,
                        'note':     note,
                        'qty':      curr['qties'][note] - old['qties'][note],
                    });
                } else if (old['qties'][note] > curr['qties'][note]) {
                    rem.push({
                        'id':       pid,
                        'name':     this.pos.db.get_product_by_id(pid).display_name,
                        'name_wrapped': curr.product_name_wrapped,
                        'note':     note,
                        'qty':      old['qties'][note] - curr['qties'][note],
                    });
                }
            }
        }

        for (pid in old_res) {
            for (note in old_res[pid]['qties']) {
                var found = pid in current_res && note in current_res[pid]['qties'];
                if (!found) {
                    var old = old_res[pid];
                    rem.push({
                        'id':       pid,
                        'name':     this.pos.db.get_product_by_id(pid).display_name,
                        'name_wrapped': old.product_name_wrapped,
                        'note':     note,
                        'qty':      old['qties'][note],
                    });
                }
            }
        }

        if(categories && categories.length > 0){
            // filter the added and removed orders to only contains
            // products that belong to one of the categories supplied as a parameter

            var self = this;

            var _add = [];
            var _rem = [];

            for(var i = 0; i < add.length; i++){
                if(self.pos.db.is_product_in_category(categories,add[i].id)){
                    _add.push(add[i]);
                }
            }
            add = _add;

            for(var i = 0; i < rem.length; i++){
                if(self.pos.db.is_product_in_category(categories,rem[i].id)){
                    _rem.push(rem[i]);
                }
            }
            rem = _rem;
        }

        var d = new Date();
        var hours   = '' + d.getHours();
            hours   = hours.length < 2 ? ('0' + hours) : hours;
        var minutes = '' + d.getMinutes();
            minutes = minutes.length < 2 ? ('0' + minutes) : minutes;

        return {
            'new': add,
            'cancelled': rem,
            'table': json.table || false,
            'floor': json.floor || false,
            'name': json.name  || 'unknown order',
            'time': {
                'hours':   hours,
                'minutes': minutes,
            },
        };

    },
    printChanges: async function(){
        var printers = this.pos.printers;
        for(var i = 0; i < printers.length; i++){
            var changes = this.computeChanges(printers[i].config.product_categories_ids);
            if ( changes['new'].length > 0 || changes['cancelled'].length > 0){
                var receipt = QWeb.render('OrderChangeReceipt',{changes:changes, widget:this});
                await printers[i].print_receipt(receipt);
            }
        }
    },
    hasChangesToPrint: function(){
        var printers = this.pos.printers;
        for(var i = 0; i < printers.length; i++){
            var changes = this.computeChanges(printers[i].config.product_categories_ids);
            if ( changes['new'].length > 0 || changes['cancelled'].length > 0){
                return true;
            }
        }
        return false;
    },
    hasSkippedChanges: function() {
        var orderlines = this.get_orderlines();
        for (var i = 0; i < orderlines.length; i++) {
            if (orderlines[i].mp_skip) {
                return true;
            }
        }
        return false;
    },
    export_as_JSON: function(){
        var json = _super_order.export_as_JSON.apply(this,arguments);
        json.multiprint_resume = this.saved_resume;
        return json;
    },
    init_from_JSON: function(json){
        _super_order.init_from_JSON.apply(this,arguments);
        this.saved_resume = json.multiprint_resume;
    },
});

var SubmitOrderButton = screens.ActionButtonWidget.extend({
    'template': 'SubmitOrderButton',
    button_click: async function(){
        var order = this.pos.get_order();
        if(order.hasChangesToPrint()){
            await order.printChanges();
            order.saveChanges();
        }
    },
});

screens.define_action_button({
    'name': 'submit_order',
    'widget': SubmitOrderButton,
    'condition': function() {
        return this.pos.printers.length;
    },
});

screens.OrderWidget.include({
    update_summary: function(){
        this._super();
        var changes = this.pos.get_order().hasChangesToPrint();
        var skipped = changes ? false : this.pos.get_order().hasSkippedChanges();
        var buttons = this.getParent().action_buttons;

        if (buttons && buttons.submit_order) {
            buttons.submit_order.highlight(changes);
            buttons.submit_order.altlight(skipped);
        }
    },
});

return {
    SubmitOrderButton: SubmitOrderButton,
}

});

```

## File: static\src\js\notes.js

```javascript
odoo.define('pos_restaurant.notes', function (require) {
"use strict";

var models = require('point_of_sale.models');
var screens = require('point_of_sale.screens');
var core = require('web.core');

var QWeb = core.qweb;
var _t   = core._t;

var _super_orderline = models.Orderline.prototype;

models.Orderline = models.Orderline.extend({
    initialize: function(attr, options) {
        _super_orderline.initialize.call(this,attr,options);
        this.note = this.note || "";
    },
    set_note: function(note){
        this.note = note;
        this.trigger('change',this);
    },
    get_note: function(note){
        return this.note;
    },
    can_be_merged_with: function(orderline) {
        if (orderline.get_note() !== this.get_note()) {
            return false;
        } else {
            return _super_orderline.can_be_merged_with.apply(this,arguments);
        }
    },
    clone: function(){
        var orderline = _super_orderline.clone.call(this);
        orderline.note = this.note;
        return orderline;
    },
    export_as_JSON: function(){
        var json = _super_orderline.export_as_JSON.call(this);
        json.note = this.note;
        return json;
    },
    init_from_JSON: function(json){
        _super_orderline.init_from_JSON.apply(this,arguments);
        this.note = json.note;
    },
});

var OrderlineNoteButton = screens.ActionButtonWidget.extend({
    template: 'OrderlineNoteButton',
    button_click: function(){
        var line = this.pos.get_order().get_selected_orderline();
        if (line) {
            this.gui.show_popup('textarea',{
                title: _t('Add Note'),
                value:   line.get_note(),
                confirm: function(note) {
                    line.set_note(note);
                },
            });
        }
    },
});

screens.define_action_button({
    'name': 'orderline_note',
    'widget': OrderlineNoteButton,
    'condition': function(){
        return this.pos.config.iface_orderline_notes;
    },
});
return {
    OrderlineNoteButton: OrderlineNoteButton,
}
});

```

## File: static\src\js\printbill.js

```javascript
odoo.define('pos_restaurant.printbill', function (require) {
"use strict";

var core = require('web.core');
var screens = require('point_of_sale.screens');
var gui = require('point_of_sale.gui');
var _t = core._t;
var QWeb = core.qweb;

var BillScreenWidget = screens.ReceiptScreenWidget.extend({
    template: 'BillScreenWidget',
    click_next: function(){
        this.gui.show_screen('products');
    },
    click_back: function(){
        this.gui.show_screen('products');
    },
    get_receipt_render_env: function(){
        var render_env = this._super();
        render_env.receipt.bill = true;
        return render_env;
    },
    render_receipt: function(){
        this._super();
        this.$('.receipt-change').remove();
    },
    print_web: function(){
        this._super();
        this.pos.get_order()._printed = false;
    },
    print_html: function(){
        this._super();
        this.pos.get_order()._printed = false;
    },
});

gui.define_screen({name:'bill', widget: BillScreenWidget});

var PrintBillButton = screens.ActionButtonWidget.extend({
    template: 'PrintBillButton',
    button_click: function(){
        var order = this.pos.get('selectedOrder');
        if(order.get_orderlines().length > 0) {
            this.gui.show_screen('bill');
        } else {
          this.gui.show_popup('error', {
              'title': _t('Nothing to Print'),
              'body':  _t('There are no order lines'),
          });
        }
    },
});

screens.define_action_button({
    'name': 'print_bill',
    'widget': PrintBillButton,
    'condition': function(){
        return this.pos.config.iface_printbill;
    },
});
return {
    BillScreenWidget: BillScreenWidget,
    PrintBillButton: PrintBillButton,
};
});

```

## File: static\src\js\splitbill.js

```javascript
odoo.define('pos_restaurant.splitbill', function (require) {
"use strict";

var gui = require('point_of_sale.gui');
var models = require('point_of_sale.models');
var screens = require('point_of_sale.screens');
var core = require('web.core');

var QWeb = core.qweb;

var SplitbillScreenWidget = screens.ScreenWidget.extend({
    template: 'SplitbillScreenWidget',

    previous_screen: 'products',

    renderElement: function(){
        var self = this;
        var linewidget;

        this._super();
        var order = this.pos.get_order();
        if(!order){
            return;
        }
        var orderlines = order.get_orderlines();
        for(var i = 0; i < orderlines.length; i++){
            var line = orderlines[i];
            linewidget = $(QWeb.render('SplitOrderline',{ 
                widget:this, 
                line:line, 
                selected: false,
                quantity: 0,
                id: line.id,
            }));
            linewidget.data('id',line.id);
            this.$('.orderlines').append(linewidget);
        }
        this.$('.back').click(function(){
            self.gui.show_screen(self.previous_screen);
        });
    },

    split_quantity: function(split, line, splitlines) {
        if( !line.get_unit().is_pos_groupable ){
            if( split.quantity !== line.get_quantity()){
                split.quantity = line.get_quantity();
            }else{
                split.quantity = 0;
            }
        }else{
            if( split.quantity < line.get_quantity()){
                split.quantity += line.get_unit().is_pos_groupable ? 1 : line.get_unit().rounding;
                if(split.quantity > line.get_quantity()){
                    split.quantity = line.get_quantity();
                }
            }else{
                split.quantity = 0;
            }
        }
    },

    set_line_on_order: function(neworder, split, line) {
        if( split.quantity ){
            if ( !split.line ){
                split.line = line.clone();
                neworder.add_orderline(split.line);
            }
            split.line.set_quantity(split.quantity, 'do not recompute unit price');
        }else if( split.line ) {
            neworder.remove_orderline(split.line);
            split.line = null;
        }
    },

    lineselect: function($el,order,neworder,splitlines,line_id){
        var split = splitlines[line_id] || {'quantity': 0, line: null};
        var line  = order.get_orderline(line_id);

        this.split_quantity(split, line, null);

        this.set_line_on_order(neworder, split, line);

        splitlines[line_id] = split;
        $el.replaceWith($(QWeb.render('SplitOrderline',{
            widget: this,
            line: line,
            selected: split.quantity !== 0,
            quantity: split.quantity,
            id: line_id,
        })));
        this.$('.order-info .subtotal').text(this.format_currency(neworder.get_subtotal()));
    },

    check_full_pay_order: function (order, splitlines) {
        return _.every(order.get_orderlines(), function(orderLine) {
            var split = splitlines[orderLine.id];
            return split && split.quantity === orderLine.get_quantity();
        });
    },

    set_quantity_on_order: function(splitlines, order) {
        for(var id in splitlines){
            var split = splitlines[id];
            var line  = order.get_orderline(parseInt(id));
            line.set_quantity(line.get_quantity() - split.quantity, 'do not recompute unit price');
            if(Math.abs(line.get_quantity()) < 0.00001){
                order.remove_orderline(line);
            }
            delete splitlines[id];
        }
    },

    pay: function(order,neworder,splitlines){
        if(_.isEmpty(splitlines))    // Splitlines is empty
            return;

        delete neworder.temporary;

        if(this.check_full_pay_order(order, splitlines)){
            this.gui.show_screen('payment');
        }else{
            this.set_quantity_on_order(splitlines, order);

            neworder.set_screen_data('screen','payment');

            // for the kitchen printer we assume that everything
            // has already been sent to the kitchen before splitting
            // the bill. So we save all changes both for the old
            // order and for the new one. This is not entirely correct
            // but avoids flooding the kitchen with unnecessary orders.
            // Not sure what to do in this case.

            if ( neworder.saveChanges ) {
                order.saveChanges();
                neworder.saveChanges();
            }

            neworder.set_customer_count(1);
            order.set_customer_count(order.get_customer_count() - 1);
            order.set_screen_data('screen','products');

            this.pos.get('orders').add(neworder);
            this.pos.set('selectedOrder',neworder);
        }
    },
    show: function(){
        var self = this;
        this._super();
        this.renderElement();

        var order = this.pos.get_order();
        var neworder = new models.Order({},{
            pos: this.pos,
            temporary: true,
        });
        neworder.set('client',order.get('client'));

        var splitlines = {};

        this.$('.orderlines').on('click','.orderline',function(){
            var id = parseInt($(this).data('id'));
            var $el = $(this);
            self.lineselect($el,order,neworder,splitlines,id);
        });

        this.$('.paymentmethods .button').click(function(){
            self.pay(order,neworder,splitlines);
        });
    },
});

gui.define_screen({
    'name': 'splitbill',
    'widget': SplitbillScreenWidget,
    'condition': function(){
        return this.pos.config.iface_splitbill;
    },
});

var SplitbillButton = screens.ActionButtonWidget.extend({
    template: 'SplitbillButton',
    button_click: function(){
        if(this.pos.get_order().get_orderlines().length > 0){
            this.gui.show_screen('splitbill');
        }
    },
});

screens.define_action_button({
    'name': 'splitbill',
    'widget': SplitbillButton,
    'condition': function(){
        return this.pos.config.iface_splitbill;
    },
});

return {
    SplitbillButton: SplitbillButton,
    SplitbillScreenWidget: SplitbillScreenWidget,
}

});


```

## File: static\src\js\tours\pos_restaurant.js

```javascript
odoo.define('pos_reataurant.tour.synchronized_table_management', function (require) {
    "use strict";

    var Tour = require("web_tour.tour");

    function verify_order_total(total_str) {
        return [{
            content: 'order total contains ' + total_str,
            trigger: '.order .total .value:contains("' + total_str + '")',
            run: function () {}, // it's a check
        }];
    }

    function verify_orders_synced(order_count) {
        return [{
            content: "check synced",
            trigger: ".order-sequence",
            run: function() {
                var orders = $('.order-sequence');
                if (orders.length === order_count) {
                    return
                } else {
                    throw "sync failed";
                }
            },
        }];
    }

    function add_product_to_order(product_name) {
        return [{
            content: 'buy ' + product_name,
            trigger: '.product-list .product-name:contains("' + product_name + '")',
        }, {
            content: 'the ' + product_name + ' have been added to the order',
            trigger: '.order .product-name:contains("' + product_name + '")',
            run: function () {}, // it's a check
        }];
    }

    function generate_keypad_steps(amount_str, keypad_selector) {
        var i, steps = [], current_char;
        for (i = 0; i < amount_str.length; ++i) {
            current_char = amount_str[i];
            steps.push({
                content: 'press ' + current_char + ' on payment keypad',
                trigger: keypad_selector + ' .input-button:contains("' + current_char + '"):visible'
            });
        }

        return steps;
    }

    function generate_payment_screen_keypad_steps(amount_str) {
        return generate_keypad_steps(amount_str, '.payment-numpad');
    }

    function generate_product_screen_keypad_steps(amount_str) {
        return generate_keypad_steps(amount_str, '.numpad');
    }

    function goto_payment_screen_and_select_payment_method() {
        return [{
            content: "go to payment screen",
            trigger: '.button.pay',
        }, {
            content: "pay with cash",
            trigger: '.paymentmethod:contains("Cash")',
        }];
    }

    function open_table(table_id, order_count) {
        order_count = order_count || null;
        var steps = [{
            content: 'open table ' + table_id,
            trigger: '.label:contains(' + table_id +')',
            run: 'click',
        }];
        if (order_count !== null){
            steps = steps.concat(verify_orders_synced(order_count));
        }
        return steps;
    }

    function transfer_order_to_table(table_id, order_uid) {
        return [{
            content: 'Click transfer button',
            trigger: '.control-button:contains("Transfer")',
            run: 'click',
        }, {
            content: 'Transfer order to table ' + table_id,
            trigger: '.label:contains(' + table_id +')',
            run: 'click',
        }, {
            content: 'Check if order ' + order_uid + ' is open after transfer',
            trigger: '.order-button.selected .order-sequence:contains("' + order_uid + '")',
            run: function(){} // Check
        }];
    }

    function finish_order() {
        var steps = [{
            content: "validate the order",
            trigger: '.button.next:visible',
        }];
        steps = steps.concat([{
            content: "next order",
            trigger: '.button.next:visible',
        }]);
        return steps;
    }

    /* pos_restaurant_sync
     *
     * Run on new session.
     */
    var steps = [{
        content: 'waiting for loading to finish',
        trigger: 'body:has(.loader:hidden)',
        run: function () {},
    }]


    steps = steps.concat(open_table('T5'));

    steps = steps.concat(add_product_to_order('Coca-Cola'));
    steps = steps.concat(add_product_to_order('Water'));
    steps = steps.concat(verify_order_total('4.40'));
    steps = steps.concat([{
        content: 'start new order',
        trigger: '.neworder-button',
        run: 'click',
    }]);
    steps = steps.concat(add_product_to_order('Coca-Cola'));
    steps = steps.concat(add_product_to_order('Minute Maid'));
    steps = steps.concat(verify_order_total('4.40'));
    steps = steps.concat(goto_payment_screen_and_select_payment_method());
    steps = steps.concat(generate_payment_screen_keypad_steps('6.05'));
    steps = steps.concat(finish_order());
    steps = steps.concat(open_table('T5', 1));
    steps = steps.concat(verify_order_total('4.40'));
    steps = steps.concat([{
        content: 'start new order',
        trigger: '.neworder-button',
        run: 'click',
    }]);
    steps = steps.concat(add_product_to_order('Coca-Cola'));
    steps = steps.concat(add_product_to_order('Minute Maid'));
    steps = steps.concat([{
        content: 'back to floor',
        trigger: '.floor-button',
        run: 'click',
    }]);
    steps = steps.concat(open_table('T5', 2));
    steps = steps.concat([{
        content: 'delete order',
        trigger: '.deleteorder-button',
        run: 'click',
    }, {
        content: 'confirm delete',
        trigger: '.button.confirm',
        run: 'click',
    }, {
        content: 'back to floor',
        trigger: '.floor-button',
        run: 'click',
    }]);
    steps = steps.concat(open_table('T5', 1));

    Tour.register('pos_restaurant_sync', { test: true, url: '/pos/web' }, steps);


    /* pos_restaurant_sync_second_login
     *
     * This tour should be run after the first tour is done.
     */
    var steps = [{
        content: 'waiting for loading to finish',
        trigger: 'body:has(.loader:hidden)',
        run: function () {},
    }];
    steps = steps.concat(open_table('T5', 1));
    steps = steps.concat(verify_order_total('4.40'));
    
    // Test transfering an order
    steps = steps.concat(transfer_order_to_table('T4', '002-0001'));

    // Test if products still get merged after transfering the order
    steps = steps.concat(add_product_to_order('Coca-Cola'));
    steps = steps.concat({
        content: 'check the order-line for Coca-Cola has 2 Units',
        trigger: '.orderlines:has(.orderline .product-name:contains("Coca-Cola")) .info-list:contains("2.000")',
        run: function () {},
    })
    steps = steps.concat(generate_product_screen_keypad_steps('1'));

    steps = steps.concat(goto_payment_screen_and_select_payment_method());
    steps = steps.concat(generate_payment_screen_keypad_steps('4.4'));
    steps = steps.concat(finish_order());
    steps = steps.concat(open_table('T2'));

    // Test transfering an empty order
    steps = steps.concat(transfer_order_to_table('T4', '2'));

    steps = steps.concat(add_product_to_order('Coca-Cola'));
    steps = steps.concat(verify_order_total('2.20'));

    // Take a synced order with products, remove the products 
    // and check if the order is still available in the front-end
    steps = steps.concat([{
        content: 'back to floor',
        trigger: '.floor-button',
        run: 'click',
    }]);
    steps = steps.concat(open_table('T4', 1));
    steps = steps.concat([{
        content: 'click backspace to set quantity to 0',
        trigger: '.numpad-backspace',
        run: 'click',
    }, {
        content: 'click backspace to remove line',
        trigger: '.numpad-backspace',
        run: 'click',
    }]);
    steps = steps.concat([{
        content: 'back to floor',
        trigger: '.floor-button',
        run: 'click',
    }]);
    steps = steps.concat(open_table('T4', 1));
    steps = steps.concat(add_product_to_order('Coca-Cola'));
    steps = steps.concat(verify_order_total('2.20'));
    steps = steps.concat([{
        content: 'back to floor',
        trigger: '.floor-button',
        run: 'click',
    }]);

    Tour.register('pos_restaurant_sync_second_login', { test: true, url: '/pos/web' }, steps);

});

```

## File: static\src\xml\floors.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-extend="OrderReceipt">
        <t t-jquery='.cashier' t-operation='append'>
            <t t-if='receipt.table'>
                at table <t t-esc='receipt.table' />
            </t>
            <t t-if='receipt.table &amp;&amp; receipt.customer_count'>
                <div>Guests: <t t-esc='receipt.customer_count' /></div>
            </t>
        </t>
    </t>

    <t t-extend="BillReceipt">
        <t t-jquery='.cashier' t-operation='append'>
            <t t-if='receipt.table'>
                at table <t t-esc='receipt.table' />
            </t>
            <t t-if='receipt.customer_count'>
                <div>Guests: <t t-esc='receipt.customer_count' /></div>
            </t>
        </t>
    </t>

    <t t-extend="OrderSelectorWidget">
        <t t-jquery=".order-sequence" t-operation="replace">
            <span class="order-sequence">
                <t t-if="order.server_id &amp;&amp; widget.pos.pos_session.login_number != order.uid.split('-')[1]">
                    <t t-esc="order.uid.substring(order.uid.indexOf('-') +1)"/>
                </t>
                <t t-else="">
                    <t t-esc="order.sequence_number"/>
                </t>
            </span>
        </t>
    </t>

    <t t-name="TableGuestsButton">
        <div class='control-button'>
            <span class='control-button-number'>
                <t t-esc="widget.guests()" />
            </span>
            Guests
        </div>
    </t>

    <t t-name="TransferOrderButton">
        <div class='control-button'>
            <i class='fa fa-arrow-right' /> Transfer
        </div>
    </t>

    <t t-name="TableWidget">
        <t t-if='!widget.selected'>
            <div class='table' t-att-style='widget.table_style_str()'>
                <span 
                    t-if="widget.table.shape"
                    t-att-class='"table-cover " + (widget.fill >= 1 ? "full" : "")'
                    t-att-style='"height: " + Math.ceil(widget.fill * 100) + "%;"'
                    ></span>
                <t t-if='widget.order_count'>
                    <span t-att-class='"order-count " + (widget.notifications.printing ? "notify-printing":"") + (widget.notifications.skipped ? "notify-skipped" : "")'><t t-esc='widget.order_count'/></span>
                </t>
                <span class='label'>
                    <t t-esc='widget.table.name' />
                </span>
                <span class="table-seats"><t t-esc="widget.table.seats" /></span>
            </div>
        </t>
        <t t-if='widget.selected'>
            <div class='table selected' t-att-style='widget.table_style_str()'>
                <span class='label'>
                    <t t-esc='widget.table.name' />
                </span>
                <span class="table-seats"><t t-esc="widget.table.seats" /></span>
                <t t-if="widget.table.shape === 'round'">
                    <span class='table-handle top ui-resizable-n'></span>
                    <span class='table-handle bottom ui-resizable-s'></span>
                    <span class='table-handle left ui-resizable-w'></span>
                    <span class='table-handle right ui-resizable-e'></span>
                </t>
                <t t-if="widget.table.shape === 'square'">
                    <span class='table-handle top right ui-resizable-ne'></span>
                    <span class='table-handle top left ui-resizable-nw'></span>
                    <span class='table-handle bottom right ui-resizable-se'></span>
                    <span class='table-handle bottom left ui-resizable-sw'></span>
                </t>
            </div>
        </t>
    </t>

    <t t-name="BackToFloorButton">
        <span class="order-button floor-button">
            <i class='fa fa-angle-double-left' role="img" aria-label="Back to floor" title="Back to floor"/>
            <t t-esc="floor.name"/>
            <span class='table-name'>
                ( <t t-esc="table.name" /> )
            </span>
        </span>
    </t>

    <t t-name="FloorScreenWidget">
        <div class='floor-screen screen'>
            <div class='screen-content-flexbox'>
                <t t-if='widget.pos.floors.length > 1'>
                    <div class='floor-selector'>
                        <t t-foreach="widget.pos.floors" t-as="floor">
                            <t t-if="floor.id === widget.floor.id">
                                <span class='button button-floor active' t-att-data-id="floor.id"><t t-esc="floor.name" /></span>
                            </t>
                            <t t-if="floor.id !== widget.floor.id">
                                <span class='button button-floor' t-att-data-id="floor.id"><t t-esc="floor.name" /></span>
                            </t>
                        </t>
                    </div>
                </t>
                <div class='floor-map' t-att-style='widget.get_floor_style()' >
                    <div class='empty-floor oe_hidden'>
                        This floor has no tables yet, use the <i class="fa fa-plus" role="img" aria-label="Add button" title="Add button"></i> button in the editing toolbar to create new tables.
                    </div>
                    <div class='tables'></div>
                    <span t-if="widget.pos.user.role == 'manager'" class='edit-button editing'><i class='fa fa-pencil' role="img" aria-label="Edit" title="Edit"></i></span>
                    <div class='edit-bar oe_hidden'>
                        <span class='edit-button new-table'>
                            <i class='fa fa-plus' role="img" aria-label="Add" title="Add"></i>
                        </span>
                        <span class='edit-button dup-table needs-selection'>
                            <i class='fa fa-files-o' role="img" aria-label="Duplicate" title="Duplicate"></i>
                        </span>
                        <span class='edit-button rename needs-selection'>
                            <i class='fa fa-font' role="img" aria-label="Rename" title="Rename"></i>
                        </span>
                        <span class='edit-button seats needs-selection'>
                            <i class='fa fa-user' role="img" aria-label="Seats" title="Seats"></i>
                        </span>
                        <span class='edit-button shape needs-selection'>
                            <span class='button-option square'><i class='fa fa-square-o' role="img" aria-label="Square Shape" title="Square Shape"></i></span>
                            <span class='button-option round oe_hidden'><i class='fa fa-circle-o' role="img" aria-label="Round Shape" title="Round Shape"></i></span>
                        </span> 
                        <span class='edit-button color'>
                            <i class='fa fa-tint' role="img" aria-label="Tint" title="Tint"></i>
                            <div class='color-picker fg-picker oe_hidden'>
                                <div  class='close-picker' title="Close" role="img" aria-label="Close">
                                    <i class='fa fa-times' />
                                </div>
                                <span class='color tl'  style='background-color:#EB6D6D' role="img" aria-label="Red" title="Red"/>
                                <span class='color'     style='background-color:#35D374' role="img" aria-label="Green" title="Green"/>
                                <span class='color tr'  style='background-color:#6C6DEC' role="img" aria-label="Blue" title="Blue"/>
                                <span class='color'     style='background-color:#EBBF6D' role="img" aria-label="Orange" title="Orange"/>
                                <span class='color'     style='background-color:#EBEC6D' role="img" aria-label="Yellow" title="Yellow"/>
                                <span class='color'     style='background-color:#AC6DAD' role="img" aria-label="Purple" title="Purple"/>
                                <span class='color bl'  style='background-color:#6C6D6D' role="img" aria-label="Grey" title="Grey"/>
                                <span class='color'     style='background-color:#ACADAD' role="img" aria-label="Light grey" title="Light grey"/>
                                <span class='color br'  style='background-color:#4ED2BE' role="img" aria-label="Turquoise" title="Turquoise"/>
                            </div>
                            <div class='color-picker bg-picker oe_hidden'>
                                <div  class='close-picker' title="Close" role="img" aria-label="Close">
                                    <i class='fa fa-times' />
                                </div>
                                <span class='color tl'  style='background-color:rgb(244, 149, 149)' role="img" aria-label="Red" title="Red"/>
                                <span class='color'     style='background-color:rgb(130, 233, 171)' role="img" aria-label="Green" title="Green"/>
                                <span class='color tr'  style='background-color:rgb(136, 137, 242)' role="img" aria-label="Blue" title="Blue"/>
                                <span class='color'     style='background-color:rgb(255, 214, 136)' role="img" aria-label="Orange" title="Orange"/>
                                <span class='color'     style='background-color:rgb(254, 255, 154)' role="img" aria-label="Yellow" title="Yellow"/>
                                <span class='color'     style='background-color:rgb(209, 171, 210)' role="img" aria-label="Purple" title="Purple"/>
                                <span class='color bl'  style='background-color:rgb(75, 75, 75)'    role="img" aria-label="Grey" title="Grey"/>
                                <span class='color'     style='background-color:rgb(210, 210, 210)' role="img" aria-label="Light grey" title="Light grey"/>
                                <span class='color br'  style='background-color:rgb(127, 221, 236)' role="img" aria-label="Turquoise" title="Turquoise"/>
                            </div>
                        </span>
                        <span class='edit-button trash needs-selection'>
                            <i class='fa fa-trash' role="img" aria-label="Delete" title="Delete"></i>
                        </span>
                    </div>
                    
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\multiprint.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SubmitOrderButton">
        <span class="control-button order-submit">
            <i class="fa fa-cutlery"></i>
            Order
        </span>
    </t>

    <t t-name="NameWrapped">
        <t t-foreach="change.name_wrapped.slice(1)" t-as="wrapped_line">
            <div style="text-align: right">
                <span t-esc="wrapped_line"/>
            </div>
        </t>
    </t>

    <t t-name="OrderChangeReceipt">
        <div class="pos-receipt">
            <div class="pos-receipt-order-data"><t t-esc="changes.name" /></div>
            <t t-if="changes.floor || changes.table">
                <br />
                <div class="pos-receipt-title">
                    <t t-esc="changes.floor" /> / <t t-esc="changes.table"/>
                </div>
            </t>
            <br />
            <br />
            <t t-if="changes.cancelled.length > 0">
                <div class="pos-order-receipt-cancel">
                    <div class="pos-receipt-title">
                        CANCELLED
                        <t t-esc='changes.time.hours'/>:<t t-esc='changes.time.minutes'/>
                    </div>
                    <br />
                    <br />
                    <t t-foreach="changes.cancelled" t-as="change">
                        <div>
                            <t t-esc="change.qty"/>
                            <span t-esc="change.name_wrapped[0]" class="pos-receipt-right-align"/>
                        </div>
                        <t t-call="NameWrapped"/>
                        <t t-if="change.note">
                            <div>
                                NOTE
                                <span class="pos-receipt-right-align">...</span>
                            </div>
                            <div><span class="pos-receipt-left-padding">--- <t t-esc="change.note" /></span></div>
                            <br/>
                        </t>
                    </t>
                    <br />
                    <br />
                </div>
            </t>
            <t t-if="changes.new.length > 0">
                <div class="pos-receipt-title">
                    NEW
                    <t t-esc='changes.time.hours'/>:<t t-esc='changes.time.minutes'/>
                </div>
                <br />
                <br />
                <t t-foreach="changes.new" t-as="change">
                    <div>
                        <t t-esc="change.qty"/>
                        <span t-esc="change.name_wrapped[0]" class="pos-receipt-right-align"/>
                    </div>
                    <t t-call="NameWrapped"/>
                    <t t-if="change.note">
                        <div>
                            NOTE
                            <span class="pos-receipt-right-align">...</span>
                        </div>
                        <div><span class="pos-receipt-left-padding">--- <t t-esc="change.note" /></span></div>
                        <br/>
                    </t>
                </t>
                <br />
                <br />
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\xml\notes.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-extend="Orderline">
        <t t-jquery=".info-list" t-operation="append">
            <t t-if="line.get_note()">
                <li class="info orderline-note">
                    <i class='fa fa-tag' role="img" aria-label="Note" title="Note"/><t t-esc="line.get_note()" />
                </li>
            </t>
        </t>
    </t>

    <t t-name="OrderlineNoteButton">
        <div class='control-button'>
            <i class='fa fa-tag' /> Note
        </div>
    </t>
    
</templates>

```

## File: static\src\xml\printbill.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="BillScreenWidget">
        <div class='receipt-screen screen'>
            <div class='screen-content'>
                <div class='top-content'>
                    <span class='button back'>
                        <i class='fa fa-angle-double-left'></i>
                        Back
                    </span>
                    <h1>Bill Printing</h1>
                    <span class='button next'>
                        Ok
                        <i class='fa fa-angle-double-right'></i>
                    </span>
                </div>
                <div class="centered-content">
                    <div class="button print">
                        <i class='fa fa-print'></i> Print
                    </div>
                    <div class="pos-receipt-container">
                    </div>
                </div>
            </div>
        </div>
    </t>

    <t t-name="PrintBillButton">
        <span class="control-button order-printbill">
            <i class="fa fa-print"></i>
            Bill
        </span>
    </t>

    <t t-extend="OrderReceipt">
        <t t-jquery='.pos-receipt-order-data' t-operation='append'>
            <t t-if='receipt.bill === true'>
                <div>PRO FORMA</div>
            </t>
        </t>
    </t>

</templates>

```

## File: static\src\xml\splitbill.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SplitbillButton">
        <span class="control-button order-split">
            <i class="fa fa-files-o"></i>
            Split
        </span>
    </t>

    <t t-name="SplitOrderline">

        <li t-attf-class="orderline #{ selected ? 'selected' : ''} #{ quantity !== line.get_quantity() ? 'partially' : '' }"
            t-att-data-id="id">
            <span class="product-name">
                <t t-esc="line.get_product().display_name"/>
            </span>
            <span class="price">
                <t t-esc="widget.format_currency(line.get_display_price())"/>
            </span>
            <ul class="info-list">
                <t t-if="line.get_quantity_str() !== '1'">
                    <li class="info">
                        <t t-if='selected and line.get_unit().is_pos_groupable'>
                            <em class='big'>
                                <t t-esc='quantity' />
                            </em>
                            /
                            <t t-esc="line.get_quantity_str()" />
                        </t>
                        <t t-if='!(selected and line.get_unit().is_pos_groupable)'>
                            <em>
                                <t t-esc="line.get_quantity_str()" />
                            </em>
                        </t>
                        <t t-esc="line.get_unit().name" />
                        at
                        <t t-esc="widget.format_currency(line.get_unit_price())" />
                        /
                        <t t-esc="line.get_unit().name" />
                    </li>
                </t>
                <t t-if="line.get_discount_str() !== '0'">
                    <li class="info">
                        With a 
                        <em>
                            <t t-esc="line.get_discount_str()" />%
                        </em>
                        discount
                    </li>
                </t>
            </ul>
        </li>
    </t>

    <t t-name="SplitbillScreenWidget">
        <div class='splitbill-screen screen'>
            <div class='screen-content'>
                <div class='top-content'>
                    <span class='button back'>
                        <i class='fa fa-angle-double-left'></i>
                        Back
                    </span>
                    <h1>Bill Splitting</h1>
                </div>
                <div class='left-content touch-scrollable scrollable-y'>
                    <div class='order'>
                        <ul class='orderlines'>
                        </ul>
                    </div>
                </div>
                <div class='right-content touch-scrollable scrollable-y'>
                    <div class='order-info'>
                        <span class='subtotal'><t t-esc='widget.format_currency(0.0)'/></span>
                    </div>
                    <div class='paymentmethods'>
                        <div class='button payment'>
                            <i class='fa fa-chevron-right' /> Payment
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="pos_config_view_form_inherit_restaurant" model="ir.ui.view">
        <field name="name">pos.config.form.inherit.restaurant</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <button id="btn_use_pos_restaurant" position="replace"/>
            <div id="iface_invoicing" position="before">
                <div class="col-12 col-lg-6 o_setting_box"
                     id="iface_printbill"
                     attrs="{'invisible': [('module_pos_restaurant', '=', False)]}">
                    <div class="o_setting_left_pane">
                        <field name="iface_printbill"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="iface_printbill"/>
                        <span class="fa fa-lg fa-cutlery" title="For bars and restaurants" role="img" aria-label="For bars and restaurants"/>
                        <div class="text-muted">
                            Allow to print bill before payment
                        </div>
                    </div>
                </div>
                <div class="col-12 col-lg-6 o_setting_box"
                     id="iface_splitbill"
                     attrs="{'invisible': [('module_pos_restaurant', '=', False)]}">
                    <div class="o_setting_left_pane">
                        <field name="iface_splitbill"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="iface_splitbill"/>
                        <span class="fa fa-lg fa-cutlery" title="For bars and restaurants" role="img" aria-label="For bars and restaurants"/>
                        <div class="text-muted">
                            Split total or order lines
                        </div>
                    </div>
                </div>
            </div>
            <div id="barcode_scanner" position="after">
                <div class="col-12 col-lg-6 o_setting_box"
                     id="is_order_printer"
                     attrs="{'invisible': [('module_pos_restaurant', '=', False)]}">
                    <div class="o_setting_left_pane">
                        <field name="is_order_printer"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="is_order_printer"/>
                        <span class="fa fa-lg fa-cutlery" title="For bars and restaurants" role="img" aria-label="For bars and restaurants"/>
                        <div class="text-muted">
                            Print orders at the kitchen, at the bar, etc.
                        </div>
                        <div class="content-group" attrs="{'invisible': [('is_order_printer', '=', False)]}">
                            <div class="mt16">
                                <label string="Printers" for="printer_ids" class="o_light_label"/>
                                <field name="printer_ids" widget="many2many_tags"/>
                            </div>
                            <div>
                                <button name="%(pos_restaurant.action_restaurant_printer_form)d" icon="fa-arrow-right" type="action" string="Printers" class="btn-link"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div id="category_reference" position="before">
                <div class="col-12 col-lg-6 o_setting_box"
                     id="is_table_management"
                     attrs="{'invisible': [('module_pos_restaurant', '=', False)]}">
                    <div class="o_setting_left_pane">
                        <field name="is_table_management"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="is_table_management"/>
                        <span class="fa fa-lg fa-cutlery" title="For bars and restaurants" role="img" aria-label="For bars and restaurants"/>
                        <div class="text-muted">
                            Manage table orders
                        </div>
                        <div class="content-group" attrs="{'invisible': [('is_table_management','=',False)]}">
                            <div class="mt16">
                                <label string="Floors" for="floor_ids" class="o_light_label"/>
                                <field name="floor_ids" widget="many2many_tags"/>
                            </div>
                            <div>
                                <button name="%(pos_restaurant.action_restaurant_floor_form)d" icon="fa-arrow-right" type="action" string="Floors" class="btn-link"/>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-12 col-lg-6 o_setting_box"
                     id="iface_orderline_notes"
                     attrs="{'invisible': [('module_pos_restaurant', '=', False)]}">
                    <div class="o_setting_left_pane">
                        <field name="iface_orderline_notes"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="iface_orderline_notes"/>
                        <span class="fa fa-lg fa-cutlery" title="For bars and restaurants" role="img" aria-label="For bars and restaurants"/>
                        <div class="text-muted">
                            Add notes to orderlines
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\pos_order_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="view_pos_pos_form" model="ir.ui.view">
        <field name="name">pos.order.form.view.inherit</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"></field>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='order_fields']" position="inside">
                <field name="table_id"/>
                <field name="customer_count"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\pos_restaurant_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <template id="assets" inherit_id="point_of_sale.assets">
          <xpath expr="." position="inside">
              <script type="text/javascript" src="/pos_restaurant/static/lib/js/jquery.ui.touch-punch.js"></script>
              <script type="text/javascript" src="/pos_restaurant/static/src/js/multiprint.js"></script>
              <script type="text/javascript" src="/pos_restaurant/static/src/js/splitbill.js"></script>
              <script type="text/javascript" src="/pos_restaurant/static/src/js/printbill.js"></script>
              <script type="text/javascript" src="/pos_restaurant/static/src/js/floors.js"></script>
              <script type="text/javascript" src="/pos_restaurant/static/src/js/notes.js"></script>
          </xpath>
          <xpath expr="//link[@id='pos-stylesheet']" position="after">
              <link rel="stylesheet" href="/pos_restaurant/static/src/css/restaurant.css"/>
          </xpath>
        </template>

    <template id="assets_backend" name="hr assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/point_of_sale/static/src/scss/pos_dashboard.scss"/>
        </xpath>
    </template>

    <template id="assets_tests" name="POS Restaurant Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/pos_restaurant/static/src/js/tours/pos_restaurant.js"></script>
        </xpath>
    </template>

</odoo>

```

## File: views\pos_restaurant_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <!--     RESTAURANTS FLOORS  -->

        <record id="view_restaurant_floor_form" model="ir.ui.view">
            <field name="name">Restaurant Floors</field>
            <field name="model">restaurant.floor</field>
            <field name="arch" type="xml">
                <form string="Restaurant Floor">
                    <sheet>
                        <group col="4">
                            <field name="name" />
                            <field name="pos_config_id" />
                            <field name="background_color" groups="base.group_no_one" />
                        </group>
                        <field name="table_ids">
                            <tree string='Tables'>
                                <field name="name" />
                                <field name="seats" />
                                <field name="shape" />
                            </tree>
                        </field>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_restaurant_floor_tree" model="ir.ui.view">
            <field name="name">Restaurant Floors</field>
            <field name="model">restaurant.floor</field>
            <field name="arch" type="xml">
                <tree string="Restaurant Floors">
                    <field name="sequence" widget="handle" />
                    <field name="name" />
                    <field name="pos_config_id" />
                </tree>
            </field>
        </record>

        <record id="view_restaurant_floor_kanban" model="ir.ui.view">
            <field name="name">restaurant.floor.kanban</field>
            <field name="model">restaurant.floor</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="name"/>
                    <field name="pos_config_id" />
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div><strong>Floor Name: </strong><t t-esc="record.name.value"/></div>
                                <div><strong>Point of Sale: </strong><t t-esc="record.pos_config_id.value"/></div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="action_restaurant_floor_form" model="ir.actions.act_window">
            <field name="name">Floor Plans</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">restaurant.floor</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new restaurant floor
              </p><p>
                A restaurant floor represents the place where customers are served, this is where you can
                define and position the tables.
              </p>
            </field>
        </record>

        <record id="view_restaurant_table_form" model="ir.ui.view">
            <field name="name">Restaurant Table</field>
            <field name="model">restaurant.table</field>
            <field name="arch" type="xml">
                <form string="Restaurant Table">
                    <group col="2">
                        <field name="name" />
                        <field name="seats" />
                    </group>
                    <group col="4" string="Appearance" groups="base.group_no_one">
                        <field name="shape" />
                        <field name="color" />
                        <field name="position_h" />
                        <field name="position_v" />
                        <field name="width" />
                        <field name="height" />
                    </group>
                </form>
            </field>
        </record>

        <menuitem id="menu_restaurant_floor_all"
             parent="point_of_sale.menu_point_config_product"
             action="action_restaurant_floor_form"
             sequence="10"
             groups="base.group_no_one"/>

        <!--     RESTAURANT PRINTERS     -->

        <record id="view_restaurant_printer_form" model="ir.ui.view">
            <field name="name">Order Printer</field>
            <field name="model">restaurant.printer</field>
            <field name="arch" type="xml">
                <form string="POS Printer">
                    <sheet>
                        <group>
                            <field name="name" />
                            <field name="printer_type" widget="radio"/>
                            <field name="proxy_ip" attrs="{'invisible': [('printer_type', '!=', 'iot')]}"/>
                            <field name="product_categories_ids" />
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="action_restaurant_printer_form" model="ir.actions.act_window">
            <field name="name">Order Printers</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">restaurant.printer</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new restaurant order printer
              </p><p>
                Order Printers are used by restaurants and bars to print the
                order updates in the kitchen/bar when the waiter updates the order.
              </p><p>
                Each Order Printer has an IP Address that defines the IoT Box/Hardware
                Proxy where the printer can be found, and a list of product categories.
                An Order Printer will only print updates for products belonging to one of
                its categories.
              </p>
            </field>
        </record>

        <record id="view_restaurant_printer" model="ir.ui.view">
            <field name="name">Order Printers</field>
            <field name="model">restaurant.printer</field>
            <field name="arch" type="xml">
                <tree string="Restaurant Order Printers">
                    <field name="name" />
                    <field name="proxy_ip" />
                    <field name="product_categories_ids" />
                </tree>
            </field>
        </record>

        <menuitem id="menu_restaurant_printer_all"
             parent="point_of_sale.menu_point_config_product"
             action="action_restaurant_printer_form"
             sequence="15"
             groups="base.group_no_one"/>

</odoo>

```

