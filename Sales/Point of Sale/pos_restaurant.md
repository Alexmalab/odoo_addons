# Odoo Module: pos_restaurant

Category: Sales/Point of Sale

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
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Restaurant extensions for the Point of Sale ',
    'description': """

This module adds several features to the Point of Sale that are specific to restaurant management:
- Bill Printing: Allows you to print a receipt before the order is paid
- Bill Splitting: Allows you to split an order into different orders
- Kitchen Order Printing: allows you to print orders updates to kitchen or bar printers

""",
    'depends': ['point_of_sale'],
    'website': 'https://www.odoo.com/app/point-of-sale-restaurant',
    'data': [
        'security/ir.model.access.csv',
        'views/pos_order_views.xml',
        'views/pos_restaurant_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'demo': [
        'data/pos_restaurant_demo.xml',
    ],
    'installable': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_restaurant/static/lib/**/*.js',
            'pos_restaurant/static/src/js/**/*.js',
            ('after', 'point_of_sale/static/src/scss/pos.scss', 'pos_restaurant/static/src/scss/restaurant.scss'),
            'pos_restaurant/static/src/xml/**/*',
        ],
        'web.assets_backend': [
            'point_of_sale/static/src/scss/pos_dashboard.scss',
        ],
        'web.assets_tests': [
            'pos_restaurant/static/tests/tours/**/*',
        ],
    },
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
            <field name="pos_config_id" ref="pos_restaurant.pos_config_restaurant"/>
        </record>

        <record id="table_01" model="restaurant.table">
            <field name="name">T1</field>
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_main"/>
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
            <field name="pos_config_id" ref="pos_restaurant.pos_config_restaurant"/>
        </record>

        <!-- Patio: Left table row -->

        <record id="table_21" model="restaurant.table">
            <field name="name">T1</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
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
            <field name="floor_id" ref="pos_restaurant.floor_patio"/>
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">560</field>
            <field name="position_v">315</field>
        </record>

        <function model="pos.config" name="add_cash_payment_method" />
</odoo>

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class PosConfig(models.Model):
    _inherit = 'pos.config'

    iface_splitbill = fields.Boolean(string='Bill Splitting', help='Enables Bill Splitting in the Point of Sale.')
    iface_printbill = fields.Boolean(string='Bill Printing', help='Allows to print the Bill before payment.')
    iface_orderline_notes = fields.Boolean(string='Internal Notes', help='Allow custom internal notes on Orderlines.')
    floor_ids = fields.One2many('restaurant.floor', 'pos_config_id', string='Restaurant Floors', help='The restaurant floors served by this point of sale.')
    printer_ids = fields.Many2many('restaurant.printer', 'pos_config_printer_rel', 'config_id', 'printer_id', string='Order Printers')
    is_table_management = fields.Boolean('Floors & Tables')
    is_order_printer = fields.Boolean('Order Printer')
    set_tip_after_payment = fields.Boolean('Set Tip After Payment', help="Adjust the amount authorized by payment terminals to add a tip after the customers left or at the end of the day.")
    module_pos_restaurant = fields.Boolean(default=True)

    def _force_http(self):
        enforce_https = self.env['ir.config_parameter'].sudo().get_param('point_of_sale.enforce_https')
        if not enforce_https and self.printer_ids.filtered(lambda pt: pt.printer_type == 'epson_epos'):
            return True
        return super(PosConfig, self)._force_http()

    def get_tables_order_count(self):
        """         """
        self.ensure_one()
        floors = self.env['restaurant.floor'].search([('pos_config_id', 'in', self.ids)])
        tables = self.env['restaurant.table'].search([('floor_id', 'in', floors.ids)])
        domain = [('state', '=', 'draft'), ('table_id', 'in', tables.ids)]

        order_stats = self.env['pos.order'].read_group(domain, ['table_id'], 'table_id')
        orders_map = dict((s['table_id'][0], s['table_id_count']) for s in order_stats)

        result = []
        for table in tables:
            result.append({'id': table.id, 'orders': orders_map.get(table.id, 0)})
        return result

    def _get_forbidden_change_fields(self):
        forbidden_keys = super(PosConfig, self)._get_forbidden_change_fields()
        forbidden_keys.append('is_table_management')
        forbidden_keys.append('floor_ids')
        return forbidden_keys

    def write(self, vals):
        if ('is_table_management' in vals and vals['is_table_management'] == False):
            vals['floor_ids'] = [(5, 0, 0)]
        if ('is_order_printer' in vals and vals['is_order_printer'] == False):
            vals['printer_ids'] = [(5, 0, 0)]
        return super(PosConfig, self).write(vals)

    @api.model
    def add_cash_payment_method(self):
        companies = self.env['res.company'].search([])
        for company in companies.filtered('chart_template_id'):
            pos_configs = self.search([('company_id', '=', company.id), ('module_pos_restaurant', '=', True)])
            journal_counter = 2
            for pos_config in pos_configs:
                if pos_config.payment_method_ids.filtered('is_cash_count'):
                    continue
                cash_journal = self.env['account.journal'].search([('company_id', '=', company.id), ('type', '=', 'cash'), ('pos_payment_method_ids', '=', False)], limit=1)
                if not cash_journal:
                    cash_journal = self.env['account.journal'].create({
                        'name': _('Cash %s', journal_counter),
                        'code': 'RCSH%s' % journal_counter,
                        'type': 'cash',
                        'company_id': company.id
                    })
                    journal_counter += 1
                payment_methods = pos_config.payment_method_ids
                payment_methods |= self.env['pos.payment.method'].create({
                    'name': _('Cash Bar'),
                    'journal_id': cash_journal.id,
                    'company_id': company.id,
                })
                pos_config.write({'payment_method_ids': [(6, 0, payment_methods.ids)]})

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.tools import groupby
from re import search
from functools import partial

import pytz

from odoo import api, fields, models


class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    note = fields.Char('Internal Note added by the waiter.')
    uuid = fields.Char(string='Uuid', readonly=True, copy=False)
    mp_skip = fields.Boolean('Skip line when sending ticket to kitchen printers.')


class PosOrder(models.Model):
    _inherit = 'pos.order'

    table_id = fields.Many2one('restaurant.table', string='Table', help='The table where this order was served', index='btree_not_null')
    customer_count = fields.Integer(string='Guests', help='The amount of customers that have been served by this order.')
    multiprint_resume = fields.Char(string='Multiprint Resume', help="Last printed state of the order")

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
            next(order_line for order_line in order_lines if order_line['id'] == order_line_id)['pack_lot_ids'] = list(pack_lot_ids)

    def _get_fields_for_order_line(self):
        fields = super(PosOrder, self)._get_fields_for_order_line()
        fields.extend([
            'id',
            'discount',
            'product_id',
            'price_unit',
            'order_id',
            'qty',
            'note',
            'uuid',
            'mp_skip',
            'full_product_name',
            'customer_note',
            'price_extra',
        ])
        return fields

    def _prepare_order_line(self, order_line):
        """Method that will allow the cleaning of values to send the correct information.
        :param order_line: order_line that will be cleaned.
        :type order_line: pos.order.line.
        :returns: dict -- dict representing the order line's values.
        """
        order_line = super()._prepare_order_line(order_line)
        order_line["product_id"] = order_line["product_id"][0]
        order_line["server_id"] = order_line["id"]

        del order_line["id"]
        if not "pack_lot_ids" in order_line:
            order_line["pack_lot_ids"] = []
        else:
            order_line["pack_lot_ids"] = [[0, 0, lot] for lot in order_line["pack_lot_ids"]]
        return order_line

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
            extended_order_lines.append([0, 0, self._prepare_order_line(order_line)])

        for order_id, order_lines in groupby(extended_order_lines, key=lambda x:x[2]['order_id']):
            next(order for order in orders if order['id'] == order_id[0])['lines'] = list(order_lines)

    def _get_fields_for_payment_lines(self):
        return [
            'id',
            'amount',
            'pos_order_id',
            'payment_method_id',
            'card_type',
            'cardholder_name',
            'transaction_id',
            'payment_status'
            ]

    def _get_payments_lines_list(self, orders):
        payment_lines = self.env['pos.payment'].search_read(
                domain = [('pos_order_id', 'in', [po['id'] for po in orders])],
                fields = self._get_fields_for_payment_lines())

        extended_payment_lines = []
        for payment_line in payment_lines:
            payment_line['server_id'] = payment_line['id']
            payment_line['payment_method_id'] = payment_line['payment_method_id'][0]

            del payment_line['id']
            extended_payment_lines.append([0, 0, payment_line])
        return extended_payment_lines

    def _get_payment_lines(self, orders):
        """Add account_bank_statement_lines to the orders.

        The function doesn't return anything but adds the results directly to the orders.

        :param orders: orders for which the payment_lines are to be requested.
        :type orders: pos.order.
        """
        extended_payment_lines = self._get_payments_lines_list(orders)
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
            'multiprint_resume',
            'access_token',
        ]

    def _get_domain_for_draft_orders(self, table_ids):
        """ Get the domain to search for draft orders on a table.
        :param table_ids: Ids of the selected tables.
        :type table_ids: list of int.
        "returns: list -- list of tuples that represents a domain.
        """
        return [('state', '=', 'draft'), ('table_id', 'in', table_ids)]

    def _add_activated_coupon_to_draft_orders(self, table_orders):
        table_orders = super()._add_activated_coupon_to_draft_orders(table_orders)
        return table_orders

    @api.model
    def get_table_draft_orders(self, table_ids):
        """Generate an object of all draft orders for the given table.

        Generate and return an JSON object with all draft orders for the given table, to send to the
        front end application.

        :param table_ids: Ids of the selected tables.
        :type table_ids: list of int.
        :returns: list -- list of dict representing the table orders
        """
        table_orders = self.search_read(
                domain=self._get_domain_for_draft_orders(table_ids),
                fields=self._get_fields_for_draft_order())

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

        return self._add_activated_coupon_to_draft_orders(table_orders)

    @api.model
    def remove_from_ui(self, server_ids):
        """ Remove orders from the frontend PoS application

        Remove orders from the server by id.
        :param server_ids: list of the id's of orders to remove from the server.
        :type server_ids: list.
        :returns: list -- list of db-ids for the removed orders.
        """
        orders = self.search([('id', 'in', server_ids), ('state', '=', 'draft')])
        orders.write({'state': 'cancel'})
        # TODO Looks like delete cascade is a better solution.
        orders.mapped('payment_ids').sudo().unlink()
        orders.sudo().unlink()
        return orders.ids

    def set_tip(self, tip_line_vals):
        """Set tip to `self` based on values in `tip_line_vals`."""

        self.ensure_one()
        PosOrderLine = self.env['pos.order.line']
        process_line = partial(PosOrderLine._order_line_fields, session_id=self.session_id.id)

        # 1. add/modify tip orderline
        processed_tip_line_vals = process_line([0, 0, tip_line_vals])[2]
        processed_tip_line_vals.update({ "order_id": self.id })
        tip_line = self.lines.filtered(lambda line: line.product_id == self.session_id.config_id.tip_product_id)
        if not tip_line:
            tip_line = PosOrderLine.create(processed_tip_line_vals)
        else:
            tip_line.write(processed_tip_line_vals)

        # 2. modify payment
        payment_line = self.payment_ids.filtered(lambda line: not line.is_change)[0]
        # TODO it would be better to throw error if there are multiple payment lines
        # then ask the user to select which payment to update, no?
        payment_line._update_payment_line_for_tip(tip_line.price_subtotal_incl)

        # 3. flag order as tipped and update order fields
        self.write({
            "is_tipped": True,
            "tip_amount": tip_line.price_subtotal_incl,
            "amount_total": self.amount_total + tip_line.price_subtotal_incl,
            "amount_paid": self.amount_paid + tip_line.price_subtotal_incl,
        })

    def set_no_tip(self):
        """Override this method to introduce action when setting no tip."""
        self.ensure_one()
        self.write({
            "is_tipped": True,
            "tip_amount": 0,
        })

    @api.model
    def _order_fields(self, ui_order):
        order_fields = super(PosOrder, self)._order_fields(ui_order)
        order_fields['table_id'] = ui_order.get('table_id', False)
        order_fields['customer_count'] = ui_order.get('customer_count', 0)
        order_fields['multiprint_resume'] = ui_order.get('multiprint_resume', False)
        return order_fields

    def _export_for_ui(self, order):
        result = super(PosOrder, self)._export_for_ui(order)
        result['table_id'] = order.table_id.id
        return result

```

## File: models\pos_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosConfig(models.Model):
    _inherit = 'pos.payment'

    def _update_payment_line_for_tip(self, tip_amount):
        """Inherit this method to perform reauthorization or capture on electronic payment."""
        self.ensure_one()
        self.write({
            "amount": self.amount + tip_amount,
        })

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
    _order = "sequence, name"

    name = fields.Char('Floor Name', required=True)
    pos_config_id = fields.Many2one('pos.config', string='Point of Sale')
    background_image = fields.Binary('Background Image')
    background_color = fields.Char('Background Color', help='The background color of the floor in a html-compatible format', default='rgb(210, 210, 210)')
    table_ids = fields.One2many('restaurant.table', 'floor_id', string='Tables')
    sequence = fields.Integer('Sequence', default=1)
    active = fields.Boolean(default=True)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_pos_session(self):
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

    def write(self, vals):
        for floor in self:
            if floor.pos_config_id.has_active_session and (vals.get('pos_config_id') or vals.get('active')) :
                raise UserError(
                    'Please close and validate the following open PoS Session before modifying this floor.\n'
                    'Open session: %s' % (' '.join(floor.pos_config_id.mapped('name')),))
            if vals.get('pos_config_id') and floor.pos_config_id.id and vals.get('pos_config_id') != floor.pos_config_id.id:
                raise UserError(_('The %s is already used in another Pos Config.', floor.name))
        return super(RestaurantFloor, self).write(vals)


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

        sanitized_table = dict([(key, val) for key, val in table.items() if key in self._fields and val is not None])
        table_id = sanitized_table.pop('id', False)
        if table_id:
            self.browse(table_id).write(sanitized_table)
        else:
            table_id = self.create(sanitized_table).id
        return table_id

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_pos_session(self):
        confs = self.mapped('floor_id').mapped('pos_config_id').filtered(lambda c: c.is_table_management == True)
        opened_session = self.env['pos.session'].search([('config_id', 'in', confs.ids), ('state', '!=', 'closed')])
        if opened_session:
            error_msg = _("You cannot remove a table that is used in a PoS session, close the session(s) first.")
            if confs:
                raise UserError(error_msg)


class RestaurantPrinter(models.Model):

    _name = 'restaurant.printer'
    _description = 'Restaurant Printer'

    name = fields.Char('Printer Name', required=True, default='Printer', help='An internal identification of the printer')
    printer_type = fields.Selection(string='Printer Type', default='iot',
        selection=[('iot', ' Use a printer connected to the IoT Box')])
    proxy_ip = fields.Char('Proxy IP Address', help="The IP Address or hostname of the Printer's hardware proxy")
    product_categories_ids = fields.Many2many('pos.category', 'printer_category_rel', 'printer_id', 'category_id', string='Printed Product Categories')

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from itertools import groupby
from odoo.osv.expression import AND

class PosSession(models.Model):
    _inherit = 'pos.session'

    def _pos_ui_models_to_load(self):
        result = super()._pos_ui_models_to_load()
        if self.config_id.module_pos_restaurant:
            result.append('restaurant.printer')
            if self.config_id.is_table_management:
                result.append('restaurant.floor')
        return result

    def _loader_params_restaurant_floor(self):
        return {
            'search_params': {
                'domain': [('pos_config_id', '=', self.config_id.id)],
                'fields': ['name', 'background_color', 'table_ids', 'sequence'],
                'order': 'sequence',
            },
        }

    def _loader_params_restaurant_table(self):
        return {
            'search_params': {
                'domain': [('active', '=', True)],
                'fields': [
                    'name', 'width', 'height', 'position_h', 'position_v',
                    'shape', 'floor_id', 'color', 'seats', 'active'
                ],
            },
        }

    def _get_pos_ui_restaurant_floor(self, params):
        floors = self.env['restaurant.floor'].search_read(**params['search_params'])
        floor_ids = [floor['id'] for floor in floors]

        table_params = self._loader_params_restaurant_table()
        table_params['search_params']['domain'] = AND([table_params['search_params']['domain'], [('floor_id', 'in', floor_ids)]])
        tables = self.env['restaurant.table'].search(table_params['search_params']['domain'], order='floor_id')
        tables_by_floor_id = {}
        for floor_id, table_group in groupby(tables, key=lambda table: table.floor_id):
            floor_tables = self.env['restaurant.table'].concat(*table_group)
            tables_by_floor_id[floor_id.id] = floor_tables.read(table_params['search_params']['fields'])

        for floor in floors:
            floor['tables'] = tables_by_floor_id.get(floor['id'], [])

        return floors

    def _loader_params_restaurant_printer(self):
        return {
            'search_params': {
                'domain': [('id', 'in', self.config_id.printer_ids.ids)],
                'fields': ['name', 'proxy_ip', 'product_categories_ids', 'printer_type'],
            },
        }
    def _get_pos_ui_restaurant_printer(self, params):
        return self.env['restaurant.printer'].search_read(**params['search_params'])

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def _get_floors_domain(self):
        return ['|', ('pos_config_id', 'in', self.pos_config_id.ids), ('pos_config_id', '=', False)]

    pos_floor_ids = fields.One2many(related='pos_config_id.floor_ids', readonly=False, domain=lambda self: self._get_floors_domain())
    pos_iface_orderline_notes = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_iface_printbill = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_iface_splitbill = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_is_order_printer = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_is_table_management = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_printer_ids = fields.Many2many(related='pos_config_id.printer_ids', readonly=False)
    pos_set_tip_after_payment = fields.Boolean(compute='_compute_pos_set_tip_after_payment', store=True, readonly=False)

    @api.depends('pos_module_pos_restaurant', 'pos_config_id')
    def _compute_pos_module_pos_restaurant(self):
        for res_config in self:
            if not res_config.pos_module_pos_restaurant:
                res_config.update({
                    'pos_iface_orderline_notes': False,
                    'pos_iface_printbill': False,
                    'pos_iface_splitbill': False,
                    'pos_is_order_printer': False,
                    'pos_is_table_management': False,
                })
            else:
                res_config.update({
                    'pos_iface_orderline_notes': res_config.pos_config_id.iface_orderline_notes,
                    'pos_iface_printbill': res_config.pos_config_id.iface_printbill,
                    'pos_iface_splitbill': res_config.pos_config_id.iface_splitbill,
                    'pos_is_order_printer': res_config.pos_config_id.is_order_printer,
                    'pos_is_table_management': res_config.pos_config_id.is_table_management,
                })

    @api.depends('pos_iface_tipproduct', 'pos_config_id')
    def _compute_pos_set_tip_after_payment(self):
        for res_config in self:
            if res_config.pos_iface_tipproduct:
                res_config.pos_set_tip_after_payment = res_config.pos_config_id.set_tip_after_payment
            else:
                res_config.pos_set_tip_after_payment = False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_order
from . import pos_payment
from . import pos_restaurant
from . import pos_session
from . import res_config_settings

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

## File: static\src\js\Chrome.js

```javascript
odoo.define('pos_restaurant.chrome', function (require) {
    'use strict';

    const Chrome = require('point_of_sale.Chrome');
    const Registries = require('point_of_sale.Registries');

    const NON_IDLE_EVENTS = 'mousemove mousedown touchstart touchend touchmove click scroll keypress'.split(/\s+/);
    let IDLE_TIMER_SETTER;

    const PosResChrome = (Chrome) =>
        class extends Chrome {
            /**
             * @override
             */
            async start() {
                await super.start();
                if (this.env.pos.config.iface_floorplan) {
                    this._setActivityListeners();
                }
            }
            /**
             * @override
             * Do not set `FloorScreen` to the order.
             */
            _setScreenData(name) {
                if (name === 'FloorScreen') return;
                super._setScreenData(...arguments);
            }
            /**
             * @override
             * `FloorScreen` is the start screen if there are floors.
             */
            get startScreen() {
                if (this.env.pos.config.iface_floorplan) {
                    const table = this.env.pos.table;
                    return { name: 'FloorScreen', props: { floor: table ? table.floor : null } };
                } else {
                    return super.startScreen;
                }
            }
            _setActivityListeners() {
                IDLE_TIMER_SETTER = this._setIdleTimer.bind(this);
                for (const event of NON_IDLE_EVENTS) {
                    window.addEventListener(event, IDLE_TIMER_SETTER);
                }
            }
            _setIdleTimer() {
                clearTimeout(this.idleTimer);
                if (this._shouldResetIdleTimer()) {
                    this.idleTimer = setTimeout(() => {
                        this._actionAfterIdle();
                    }, 60000);
                }
            }
            _actionAfterIdle() {
                if (this.tempScreen.isShown) {
                    this.trigger('close-temp-screen');
                }
                const table = this.env.pos.table;
                const order = this.env.pos.get_order();
                if (order && order.get_screen_data().name === 'ReceiptScreen') {
                    // When the order is finalized, we can safely remove it from the memory
                    // We check that it's in ReceiptScreen because we want to keep the order if it's in a tipping state
                    this.env.pos.removeOrder(order);
                }
                this.showScreen('FloorScreen', { floor: table ? table.floor : null });
            }
            _shouldResetIdleTimer() {
                const stayPaymentScreen = this.mainScreen.name === 'PaymentScreen' && this.env.pos.get_order().paymentlines.length > 0;
                return this.env.pos.config.iface_floorplan && !stayPaymentScreen && this.mainScreen.name !== 'FloorScreen';
            }
            __showScreen() {
                super.__showScreen(...arguments);
                this._setIdleTimer();
            }
            /**
             * @override
             * Before closing pos, we remove the event listeners set on window
             * for detecting activities outside FloorScreen.
             */
            async _closePos() {
                if (IDLE_TIMER_SETTER) {
                    for (const event of NON_IDLE_EVENTS) {
                        window.removeEventListener(event, IDLE_TIMER_SETTER);
                    }
                }
                await super._closePos();
            }
        };

    Registries.Component.extend(Chrome, PosResChrome);

    return Chrome;
});

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_restaurant.models', function (require) {
"use strict";

const { PosGlobalState, Order, Orderline, Payment } = require('point_of_sale.models');
const Registries = require('point_of_sale.Registries');
const { uuidv4 } = require('point_of_sale.utils');
const core = require('web.core');
const Printer = require('point_of_sale.Printer').Printer;
const { batched } = require('point_of_sale.utils')
const QWeb = core.qweb;

const TIMEOUT = 7500;

const PosRestaurantPosGlobalState = (PosGlobalState) => class PosRestaurantPosGlobalState extends PosGlobalState {
    constructor(obj) {
        super(obj);
        this.orderToTransfer = null; // table transfer feature
        this.ordersToUpdateSet = new Set(); // used to know which orders need to be sent to the back end when syncing
        this.transferredOrdersSet = new Set(); // used to know which orders has been transferred but not sent to the back end yet
        this.loadingOrderState = false; // used to prevent orders fetched to be put in the update set during the reactive change
    }
    //@override
    async _processData(loadedData) {
        await super._processData(...arguments);
        if (this.config.is_table_management) {
            this.floors = loadedData['restaurant.floor'];
            this.loadRestaurantFloor();
        }
        if (this.config.module_pos_restaurant) {
            this._loadRestaurantPrinter(loadedData['restaurant.printer']);
        }
    }
    //@override
    _onReactiveOrderUpdated(order) {
        super._onReactiveOrderUpdated(...arguments)
        if (this.config.iface_floorplan && !this.loadingOrderState) {
            this.ordersToUpdateSet.add(order);
        }
    }
   //@override
    removeOrder(order, removeFromServer=true) {
        super.removeOrder(...arguments);
        if (this.config.iface_floorplan && removeFromServer) {
            if (this.ordersToUpdateSet.has(order)) {
                this.ordersToUpdateSet.delete(order)
            }
            if (order.server_id && !order.finalized) {
                this.db.set_order_to_remove_from_server(order);
            }
        }
    }
    //@override
    async after_load_server_data() {
        var res = await super.after_load_server_data(...arguments);
        if (this.config.iface_floorplan) {
            this.table = null;
        }
        return res;
    }
    //@override
    // if we have tables, we do not load a default order, as the default order will be
    // set when the user selects a table.
    set_start_order() {
        if (!this.config.iface_floorplan) {
            super.set_start_order(...arguments);
        }
    }
    //@override
    add_new_order() {
        const order = super.add_new_order();
        this.ordersToUpdateSet.add(order);
        return order;
    }
    //@override
    createReactiveOrder(json) {
        let reactiveOrder = super.createReactiveOrder(...arguments);
        if (this.config.iface_printers) {
            const updateOrderChanges = () => {
                if (reactiveOrder.get_screen_data().name === 'ProductScreen') {
                    reactiveOrder.updateChangesToPrint();
                }
            }
            reactiveOrder = owl.reactive(reactiveOrder, batched(updateOrderChanges));
            reactiveOrder.updateChangesToPrint();
        }
        return reactiveOrder;
    }
    //@override
    async load_orders() {
        this.loadingOrderState = true;
        await super.load_orders();
        this.loadingOrderState = false;
    }
    _loadRestaurantPrinter(printers) {
        this.unwatched.printers = [];
        // list of product categories that belong to one or more order printer
        this.printers_category_ids_set = new Set();
        for (let printerConfig of printers) {
            let printer = this.create_printer(printerConfig);
            printer.config = printerConfig;
            this.unwatched.printers.push(printer);
            for (let id of printer.config.product_categories_ids) {
                this.printers_category_ids_set.add(id);
            }
        }
        this.config.iface_printers = !!this.unwatched.printers.length;
    }
    async _getTableOrdersFromServer(tableIds) {
        this.set_synch('connecting', 1);
        try {
            const orders = await this.env.services.rpc({
                model: 'pos.order',
                method: 'get_table_draft_orders',
                args: [tableIds],
            }, {
                timeout: TIMEOUT,
                shadow: true,
            });
            this.set_synch('connected');
            return orders;
        } catch (error) {
            this.set_synch('error');
            throw error;
        }
    }
    /**
     * Sync orders that got updated to the back end
     * @param tableId ID of the table we want to sync
     */
    async _syncTableOrdersToServer() {
        await this._pushOrdersToServer();
        await this._removeOrdersFromServer();
        // This need to be called here otherwise _onReactiveOrderUpdated() will be called after the set is being cleared
        this.ordersToUpdateSet.clear();
        this.transferredOrdersSet.clear();
    }
    /**
     * Send the orders to be saved to the back end
     * @throw error
     */
    async _pushOrdersToServer() {
        const ordersUidsToSync = [...this.ordersToUpdateSet].map(order => order.uid);
        const ordersToSync = this.db.get_unpaid_orders_to_sync(ordersUidsToSync);
        const ordersResponse = await this._save_to_server(ordersToSync, {'draft': true});
        const tableOrders = [...this.ordersToUpdateSet].map(order => order);
        ordersResponse.forEach(orderResponseData => this._updateTableOrder(orderResponseData, tableOrders));
    }
    // created this hook for modularity
    _updateTableOrder(ordersResponseData, tableOrders) {
        const order = tableOrders.find(order => order.name === ordersResponseData.pos_reference);
        order.server_id = ordersResponseData.id;
        return order;
    }
    /**
    * Remove the deleted orders from the backend.
    * @throw error
    */
    async _removeOrdersFromServer() {
        const removedOrdersIds = this.db.get_ids_to_remove_from_server();
        if (removedOrdersIds.length === 0) {
            return;
        }

        const timeout = TIMEOUT * removedOrdersIds.length;
        this.set_synch('connecting', removedOrdersIds.length);
        try {
            const removeOrdersResponseData = await this.env.services.rpc({
                model: 'pos.order',
                method: 'remove_from_ui',
                args: [removedOrdersIds],
            }, {
                timeout: timeout,
                shadow: true,
            });
            this.set_synch('connected');
            this._postRemoveFromServer(removedOrdersIds, removeOrdersResponseData);
        } catch (reason) {
            let error = reason.message;
            if (error.code === 200) {
                // Business Logic Error, not a connection problem
                //if warning do not need to display traceback!!
                if (error.data.exception_type == 'warning') {
                    delete error.data.debug;
                }
            }
            // important to throw error here and let the rendering component handle the error
            console.warn('Failed to remove orders:', removedOrdersIds);
            throw error;
        }
    }
    // to override
    _postRemoveFromServer(serverIds, data) {
        this.db.set_ids_removed_from_server(serverIds);
    }
    /**
     * Replace all the orders of a table by orders fetched from the backend
     * @param tableId ID of the table
     * @throws error
     */
    async _syncTableOrdersFromServer(tableId) {
        await this._removeOrdersFromServer(); // in case we were offline and we deleted orders in the mean time
        const ordersJsons = await this._getTableOrdersFromServer([tableId]);
        const tableOrders = this.getTableOrders(tableId);
        this._replaceOrders(tableOrders, ordersJsons);
    }
    async _syncAllOrdersFromServer() {
        await this._removeOrdersFromServer(); // in case we were offline and we deleted orders in the mean time
        const tableIds = [].concat(...this.floors.map(floor => floor.tables.map(table => table.id)));
        const ordersJsons = await this._getTableOrdersFromServer(tableIds); // get all orders
        await this._syncTableOrdersToServer(); // to prevent losing the transferred orders
        const allOrders = [...this.get_order_list()];
        this._replaceOrders(allOrders, ordersJsons);
    }
    _replaceOrders(ordersToReplace, newOrdersJsons) {
        ordersToReplace.forEach(order => {
            // We don't remove the validated orders because we still want to see them in the ticket screen.
            // Orders in 'ReceiptScreen' or 'TipScreen' are validated orders.
            if (order.server_id && !order.finalized && !this.transferredOrdersSet.has(order)){
                this.removeOrder(order, false);
            }
        });
        newOrdersJsons.forEach(json => {
            // Because of the offline feature, some draft orders fetched from the backend will appear
            // to belong in different table, but in fact they are already moved.
            const transferredOrder = [...this.transferredOrdersSet].find(order => order.uid === json.uid)
            const isSameTable = transferredOrder && transferredOrder.tableId === json.tableId;
            if (isSameTable) {
                // this means we transferred back to the original table, we'll prioritize the server state
                this.removeOrder(transferredOrder, false);
            }
            if (!transferredOrder || isSameTable) {
                const order = this.createReactiveOrder(json);
                this.orders.add(order);
            }
        });
    }
    setLoadingOrderState(bool) {
        this.loadingOrderState = bool;
    }
    loadRestaurantFloor() {
        // we do this in the front end due to the circular/recursive reference needed
        // Ignore floorplan features if no floor specified.
        this.config.iface_floorplan = !!(this.floors && this.floors.length > 0);
        if (this.config.iface_floorplan) {
            this.floors_by_id = {};
            this.tables_by_id = {};
            for (let floor of this.floors) {
                this.floors_by_id[floor.id] = floor;
                for (let table of floor.tables) {
                    this.tables_by_id[table.id] = table;
                    table.floor = floor;
                }
            }
        }
    }
    async setTable(table, orderUid=null) {
        this.table = table;
        try {
            this.loadingOrderState = true;
            await this._syncTableOrdersFromServer(table.id);
        } catch (error) {
            throw error;
        } finally {
            this.loadingOrderState = false;
            const currentOrder = this.getTableOrders(table.id).find(order => orderUid ? order.uid === orderUid : !order.finalized);
            if (currentOrder) {
                this.set_order(currentOrder);
            } else {
                this.add_new_order();
            }
        }
    }
    getTableOrders(tableId) {
        return this.get_order_list().filter(order => order.tableId === tableId);
    }
    unsetTable() {
        this._syncTableOrdersToServer();
        this.table = null;
        this.set_order(null);
    }
    setCurrentOrderToTransfer() {
        this.orderToTransfer = this.selectedOrder;
    }
    async transferTable(table) {
        this.table = table;
        try {
            this.loadingOrderState = true;
            await this._syncTableOrdersFromServer(table.id);
        } catch (error) {
            throw error;
        } finally {
            this.loadingOrderState = false;
            this.orderToTransfer.tableId = table.id;
            this.set_order(this.orderToTransfer);
            this.transferredOrdersSet.add(this.orderToTransfer);
            this.orderToTransfer = null;
        }
    }
    getCustomerCount(tableId) {
        const tableOrders = this.getTableOrders(tableId).filter(order => !order.finalized);
        return tableOrders.reduce((count, order) => count + order.getCustomerCount(), 0);
    }
    create_printer(config) {
        var url = config.proxy_ip || '';
        if(url.indexOf('//') < 0) {
            url = window.location.protocol + '//' + url;
        }
        if(url.indexOf(':', url.indexOf('//') + 2) < 0 && window.location.protocol !== 'https:') {
            url = url + ':8069';
        }
        return new Printer(url, this);
    }
}
Registries.Model.extend(PosGlobalState, PosRestaurantPosGlobalState);

// New orders are now associated with the current table, if any.
const PosRestaurantOrder = (Order) => class PosRestaurantOrder extends Order {
    constructor(obj, options) {
        super(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            if (this.pos.config.iface_floorplan && !this.tableId && !options.json) {
                this.tableId = this.pos.table.id;
            }
            this.customerCount = this.customerCount || 1;
        }
        if (this.pos.config.iface_printers) {
            // printedResume will store the previous state of the orderlines (when there were no skip), it will
            // store all the orderlines even if the product are not printable. This way, when we add a new category in
            // the printers, the already added products of the newly added category are not printed.
            this.printedResume = owl.markRaw(this.printedResume || {}); // we don't wanna track it and re-render
            // no need to store this in the backend, we can just compute it once the order is fetched from clicking a table
            if (!this.printingChanges) {
                this._resetPrintingChanges();
            }
        }
    }
    //@override
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            if (this.pos.config.iface_floorplan) {
                json.table_id = this.tableId
            }
            json.customer_count = this.customerCount;
        }
        if (this.pos.config.iface_printers) {
            json.multiprint_resume = JSON.stringify(this.printedResume);
            // so that it can be stored in local storage and be used when loading the pos in the floorscreen
            json.printing_changes = JSON.stringify(this.printingChanges);
        }
        return json;
    }
    //@override
    init_from_JSON(json) {
        super.init_from_JSON(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            if (this.pos.config.iface_floorplan) {
                this.tableId = json.table_id;
                this.validation_date = moment.utc(json.creation_date).local().toDate();
            }
            this.customerCount = json.customer_count;
        }
        if (this.pos.config.iface_printers) {
            this.printedResume = json.multiprint_resume && JSON.parse(json.multiprint_resume);
            this.printingChanges = json.printing_changes && JSON.parse(json.printing_changes);
        }
    }
    //@override
    export_for_printing() {
        const json = super.export_for_printing(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            if (this.pos.config.iface_floorplan) {
                json.table = this.getTable().name;
            }
            json.customer_count = this.getCustomerCount();
        }
        return json;
    }
    _resetPrintingChanges() {
        this.printingChanges = { new:[], cancelled:[] };
    }
    /**
     * @returns {{ [productKey: string]: { product_id: number, name: string, note: string, quantity: number } }}
     */
    _computePrintChanges() {
        const changes = {};

        // If there's a new orderline, we add it otherwise we add the change if there's one
        this.orderlines.forEach(line => {
            if (!line.mp_skip) {
                const productId = line.get_product().id;
                const note = line.get_note();
                const productKey = `${productId} - ${line.get_full_product_name()} - ${note}`;
                const lineKey = `${line.uuid} - ${note}`;
                const quantityDiff = line.get_quantity() - (this.printedResume[lineKey] ? this.printedResume[lineKey]['quantity'] : 0);
                if (quantityDiff) {
                    if (!changes[productKey]) {
                        changes[productKey] = {
                            product_id: productId,
                            name: line.get_full_product_name(),
                            note: note,
                            quantity: quantityDiff,
                        }
                    } else {
                        changes[productKey]['quantity'] += quantityDiff;
                    }
                    line.set_dirty(true);
                } else {
                    line.set_dirty(false);
                }
            }
        })

        // If there's an orderline that's not present anymore, we consider it as removed (even if note changed)
        for (const [lineKey, lineResume] of Object.entries(this.printedResume)) {
            if (!this._getPrintedLine(lineKey)) {
                const productKey = `${lineResume['product_id']} - ${lineResume['name']} - ${lineResume['note']}`;
                if (!changes[productKey]) {
                    changes[productKey] = {
                        product_id: lineResume['product_id'],
                        name: lineResume['name'],
                        note: lineResume['note'],
                        quantity: -lineResume['quantity'],
                    }
                } else {
                    changes[productKey]['quantity'] -= lineResume['quantity'];
                }
            }
        }

        return changes;
    }
    _getPrintingCategoriesChanges(categories) {
        return {
            new: this.printingChanges['new'].filter(change => this.pos.db.is_product_in_category(categories, change['product_id'])),
            cancelled: this.printingChanges['cancelled'].filter(change => this.pos.db.is_product_in_category(categories, change['product_id'])),
        }
    }
    _getPrintedLine(lineKey) {
        return this.orderlines.find(line => line.uuid === this.printedResume[lineKey]['line_uuid'] &&
            line.note === this.printedResume[lineKey]['note']);
    }
    getCustomerCount(){
        return this.customerCount;
    }
    setCustomerCount(count) {
        this.customerCount = Math.max(count,0);
    }
    getTable() {
        if (this.pos.config.iface_floorplan) {
            return this.pos.tables_by_id[this.tableId];
        }
        return null;
    }
    updatePrintedResume(){
        // we first remove the removed orderlines
        for (const lineKey in this.printedResume) {
            if (!this._getPrintedLine(lineKey)) {
                delete this.printedResume[lineKey];
            }
        }
        // we then update the added orderline or product quantity change
        this.orderlines.forEach(line => {
            if (!line.mp_skip) {
                const note = line.get_note();
                const lineKey = `${line.uuid} - ${note}`;
                if (this.printedResume[lineKey]) {
                    this.printedResume[lineKey]['quantity'] = line.get_quantity();
                } else {
                    this.printedResume[lineKey] = {
                        line_uuid: line.uuid,
                        product_id: line.get_product().id,
                        name: line.get_full_product_name(),
                        note: note,
                        quantity: line.get_quantity()
                    }
                }
                line.set_dirty(false);
            }
        });
        this._resetPrintingChanges();
    }
    updateChangesToPrint() {
        const changes = this._computePrintChanges(); // it's possible to have a change's quantity of 0
        // we thoroughly parse the changes we just computed to properly separate them into two
        const toAdd = [];
        const toRemove = [];

        for (const lineChange of Object.values(changes)) {
            if (lineChange['quantity'] > 0) {
                toAdd.push(lineChange);
            } else if (lineChange['quantity'] < 0) {
                lineChange['quantity'] *= -1; // we change the sign because that's how it is
                toRemove.push(lineChange);
            }
        }

        this.printingChanges = { new: toAdd, cancelled: toRemove };
    }
    hasChangesToPrint(){
        for (const printer of this.pos.unwatched.printers) {
            const changes = this._getPrintingCategoriesChanges(printer.config.product_categories_ids);
            if (changes['new'].length > 0 || changes['cancelled'].length > 0) {
                return true;
            }
        }
        return false;
    }
    hasSkippedChanges() {
        var orderlines = this.get_orderlines();
        for (var i = 0; i < orderlines.length; i++) {
            if (orderlines[i].mp_skip) {
                return true;
            }
        }
        return false;
    }
    async printChanges(){
        let isPrintSuccessful = true;
        const d = new Date();
        let hours = '' + d.getHours();
        hours = hours.length < 2 ? ('0' + hours) : hours;
        let minutes = '' + d.getMinutes();
        minutes = minutes.length < 2 ? ('0' + minutes) : minutes;


        for (const printer of this.pos.unwatched.printers) {
            const changes = this._getPrintingCategoriesChanges(printer.config.product_categories_ids);
            if (changes['new'].length > 0 || changes['cancelled'].length > 0) {
                const printingChanges = {
                    new: changes['new'],
                    cancelled: changes['cancelled'],
                    table_name: this.pos.config.iface_floorplan ? this.getTable().name : false,
                    floor_name: this.pos.config.iface_floorplan ? this.getTable().floor.name : false,
                    name: this.name || 'unknown order',
                    time: {
                        hours,
                        minutes,
                    },
                };
                const receipt = QWeb.render('OrderChangeReceipt', { changes: printingChanges });
                const result = await printer.print_receipt(receipt);
                if (!result.successful) {
                    isPrintSuccessful = false;
                }
            }
        }
       return isPrintSuccessful;
    }
}
Registries.Model.extend(Order, PosRestaurantOrder);


const PosRestaurantOrderline = (Orderline) => class PosRestaurantOrderline extends Orderline {
    constructor() {
        super(...arguments);
        this.note = this.note || "";
        if (this.pos.config.iface_printers) {
            this.uuid = this.uuid || uuidv4();
            // mp dirty is true if this orderline has changed since the last kitchen print
            this.mp_dirty = false
            if (!this.mp_skip) {
                // mp_skip is true if the cashier want this orderline
                // not to be sent to the kitchen
                this.mp_skip  = false;
            }
        }
    }
    //@override
    can_be_merged_with(orderline) {
        if (orderline.get_note() !== this.get_note()) {
            return false;
        } else {
            return (!this.mp_skip) && (!orderline.mp_skip) && super.can_be_merged_with(...arguments);
        }
    }
    //@override
    clone(){
        const orderline = super.clone(...arguments);
        orderline.note = this.note;
        return orderline;
    }
    //@override
    export_as_JSON(){
        const json = super.export_as_JSON(...arguments);
        json.note = this.note;
        if (this.pos.config.iface_printers) {
            json.uuid = this.uuid;
            json.mp_skip = this.mp_skip;
        }
        return json;
    }
    //@override
    init_from_JSON(json){
        super.init_from_JSON(...arguments);
        this.note = json.note;
        if (this.pos.config.iface_printers) {
            this.uuid = json.uuid;
            this.mp_skip = json.mp_skip;
        }
    }
    set_note(note){
        this.note = note;
    }
    get_note(){
        return this.note;
    }
    set_skip(skip) {
        if (this.mp_dirty && skip && !this.mp_skip) {
            this.mp_skip = true;
        }
        if (this.mp_skip && !skip) {
            this.mp_skip  = false;
        }
    }
    set_dirty(dirty) {
        if (this.printable()) {
            this.mp_dirty = dirty;
        }
    }
    get_line_diff_hash(){
        if (this.get_note()) {
            return this.id + '|' + this.get_note();
        } else {
            return '' + this.id;
        }
    }
    // can this orderline be potentially printed ?
    printable() {
        return this.pos.db.is_product_in_category(this.pos.printers_category_ids_set, this.get_product().id);
    }
}
Registries.Model.extend(Orderline, PosRestaurantOrderline);

const PosRestaurantPayment = (Payment) => class PosRestaurantPayment extends Payment {
    /**
     * Override this method to be able to show the 'Adjust Authorisation' button
     * on a validated payment_line and to show the tip screen which allow
     * tipping even after payment. By default, this returns true for all
     * non-cash payment.
     */
    canBeAdjusted() {
        if (this.payment_method.payment_terminal) {
            return this.payment_method.payment_terminal.canBeAdjusted(this.cid);
        }
        return !this.payment_method.is_cash_count;
    }
}
Registries.Model.extend(Payment, PosRestaurantPayment);

});

```

## File: static\src\js\payment.js

```javascript
odoo.define('pos_restaurant.PaymentInterface', function (require) {
    "use strict";

    var PaymentInterface = require('point_of_sale.PaymentInterface');

    PaymentInterface.include({
        /**
         * Return true if the amount that was authorized can be modified,
         * false otherwise
         * @param {string} cid - The id of the paymentline
         */
        canBeAdjusted(cid) {
            return false;
        },

        /**
         * Called when the amount authorized by a payment request should
         * be adjusted to account for a new order line, it can only be called if
         * canBeAdjusted returns True
         * @param {string} cid - The id of the paymentline
         */
        send_payment_adjust: function (cid) {},
    });
});

```

## File: static\src\js\Resizeable.js

```javascript
odoo.define('pos_restaurant.Resizeable', function(require) {
    'use strict';

    const { useListener } = require("@web/core/utils/hooks");
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    const { onMounted, useExternalListener } = owl;

    class Resizeable extends PosComponent {
        setup() {
            super.setup();
            useExternalListener(document, 'mousemove', this.resizeN);
            useExternalListener(document, 'mouseup', this.endResizeN);
            useListener('mousedown', '.resize-handle-n', this.startResizeN);

            useExternalListener(document, 'mousemove', this.resizeS);
            useExternalListener(document, 'mouseup', this.endResizeS);
            useListener('mousedown', '.resize-handle-s', this.startResizeS);

            useExternalListener(document, 'mousemove', this.resizeW);
            useExternalListener(document, 'mouseup', this.endResizeW);
            useListener('mousedown', '.resize-handle-w', this.startResizeW);

            useExternalListener(document, 'mousemove', this.resizeE);
            useExternalListener(document, 'mouseup', this.endResizeE);
            useListener('mousedown', '.resize-handle-e', this.startResizeE);

            useExternalListener(document, 'mousemove', this.resizeNW);
            useExternalListener(document, 'mouseup', this.endResizeNW);
            useListener('mousedown', '.resize-handle-nw', this.startResizeNW);

            useExternalListener(document, 'mousemove', this.resizeNE);
            useExternalListener(document, 'mouseup', this.endResizeNE);
            useListener('mousedown', '.resize-handle-ne', this.startResizeNE);

            useExternalListener(document, 'mousemove', this.resizeSW);
            useExternalListener(document, 'mouseup', this.endResizeSW);
            useListener('mousedown', '.resize-handle-sw', this.startResizeSW);

            useExternalListener(document, 'mousemove', this.resizeSE);
            useExternalListener(document, 'mouseup', this.endResizeSE);
            useListener('mousedown', '.resize-handle-se', this.startResizeSE);

            useExternalListener(document, 'touchmove', this.resizeN);
            useExternalListener(document, 'touchend', this.endResizeN);
            useListener('touchstart', '.resize-handle-n', this.startResizeN);

            useExternalListener(document, 'touchmove', this.resizeS);
            useExternalListener(document, 'touchend', this.endResizeS);
            useListener('touchstart', '.resize-handle-s', this.startResizeS);

            useExternalListener(document, 'touchmove', this.resizeW);
            useExternalListener(document, 'touchend', this.endResizeW);
            useListener('touchstart', '.resize-handle-w', this.startResizeW);

            useExternalListener(document, 'touchmove', this.resizeE);
            useExternalListener(document, 'touchend', this.endResizeE);
            useListener('touchstart', '.resize-handle-e', this.startResizeE);

            useExternalListener(document, 'touchmove', this.resizeNW);
            useExternalListener(document, 'touchend', this.endResizeNW);
            useListener('touchstart', '.resize-handle-nw', this.startResizeNW);

            useExternalListener(document, 'touchmove', this.resizeNE);
            useExternalListener(document, 'touchend', this.endResizeNE);
            useListener('touchstart', '.resize-handle-ne', this.startResizeNE);

            useExternalListener(document, 'touchmove', this.resizeSW);
            useExternalListener(document, 'touchend', this.endResizeSW);
            useListener('touchstart', '.resize-handle-sw', this.startResizeSW);

            useExternalListener(document, 'touchmove', this.resizeSE);
            useExternalListener(document, 'touchend', this.endResizeSE);
            useListener('touchstart', '.resize-handle-se', this.startResizeSE);

            this.size = { height: 0, width: 0 };
            this.loc = { top: 0, left: 0 };
            this.tempSize = {};

            onMounted(() => {
                this.limitArea = this.props.limitArea
                    ? document.querySelector(this.props.limitArea)
                    : this.el.offsetParent;
                this.limitAreaBoundingRect = this.limitArea.getBoundingClientRect();
                if (this.limitArea === this.el.offsetParent) {
                    this.limitLeft = 0;
                    this.limitTop = 0;
                    this.limitRight = this.limitAreaBoundingRect.width;
                    this.limitBottom = this.limitAreaBoundingRect.height;
                } else {
                    this.limitLeft = -this.el.offsetParent.offsetLeft;
                    this.limitTop = -this.el.offsetParent.offsetTop;
                    this.limitRight =
                        this.limitAreaBoundingRect.width - this.el.offsetParent.offsetLeft;
                    this.limitBottom =
                        this.limitAreaBoundingRect.height - this.el.offsetParent.offsetTop;
                }
                this.limitAreaWidth = this.limitAreaBoundingRect.width;
                this.limitAreaHeight = this.limitAreaBoundingRect.height;
            });
        }
        startResizeN(event) {
            let realEvent;
            if (event instanceof CustomEvent) {
                realEvent = event.detail;
            } else {
                realEvent = event;
            }
            const { y } = this._getEventLoc(realEvent);
            this.isResizingN = true;
            this.startY = y;
            this.size.height = this.el.offsetHeight;
            this.loc.top = this.el.offsetTop;
            event.stopPropagation();
        }
        resizeN(event) {
            if (this.isResizingN) {
                const { y: newY } = this._getEventLoc(event);
                let dY = newY - this.startY;
                if (dY < 0 && Math.abs(dY) > this.loc.top) {
                    dY = -this.loc.top;
                } else if (dY > 0 && dY > this.size.height) {
                    dY = this.size.height;
                }
                this.el.style.height = `${this.size.height - dY}px`;
                this.el.style.top = `${this.loc.top + dY}px`;
            }
        }
        endResizeN() {
            if (this.isResizingN && !this.isResizingE && !this.isResizingW && !this.isResizingS) {
                this.isResizingN = false;
                this._triggerResizeEnd();
            }
        }
        startResizeS(event) {
            let realEvent;
            if (event instanceof CustomEvent) {
                realEvent = event.detail;
            } else {
                realEvent = event;
            }
            const { y } = this._getEventLoc(realEvent);
            this.isResizingS = true;
            this.startY = y;
            this.size.height = this.el.offsetHeight;
            this.loc.top = this.el.offsetTop;
            event.stopPropagation();
        }
        resizeS(event) {
            if (this.isResizingS) {
                const { y: newY } = this._getEventLoc(event);
                let dY = newY - this.startY;
                if (dY > 0 && dY > this.limitAreaHeight - (this.size.height + this.loc.top)) {
                    dY = this.limitAreaHeight - (this.size.height + this.loc.top);
                } else if (dY < 0 && Math.abs(dY) > this.size.height) {
                    dY = -this.size.height;
                }
                this.el.style.height = `${this.size.height + dY}px`;
            }
        }
        endResizeS() {
            if (!this.isResizingN && !this.isResizingE && !this.isResizingW && this.isResizingS) {
                this.isResizingS = false;
                this._triggerResizeEnd();
            }
        }
        startResizeW(event) {
            let realEvent;
            if (event instanceof CustomEvent) {
                realEvent = event.detail;
            } else {
                realEvent = event;
            }
            const { x } = this._getEventLoc(realEvent);
            this.isResizingW = true;
            this.startX = x;
            this.size.width = this.el.offsetWidth;
            this.loc.left = this.el.offsetLeft;
            event.stopPropagation();
        }
        resizeW(event) {
            if (this.isResizingW) {
                const { x: newX } = this._getEventLoc(event);
                let dX = newX - this.startX;
                if (dX > 0 && dX > this.size.width) {
                    dX = this.size.width;
                } else if (dX < 0 && Math.abs(dX) > this.loc.left + Math.abs(this.limitLeft)) {
                    dX = -this.loc.left + this.limitLeft;
                }
                this.el.style.width = `${this.size.width - dX}px`;
                this.el.style.left = `${this.loc.left + dX}px`;
            }
        }
        endResizeW() {
            if (!this.isResizingN && !this.isResizingE && this.isResizingW && !this.isResizingS) {
                this.isResizingW = false;
                this._triggerResizeEnd();
            }
        }
        startResizeE(event) {
            let realEvent;
            if (event instanceof CustomEvent) {
                realEvent = event.detail;
            } else {
                realEvent = event;
            }
            const { x } = this._getEventLoc(realEvent);
            this.isResizingE = true;
            this.startX = x;
            this.size.width = this.el.offsetWidth;
            this.loc.left = this.el.offsetLeft;
            event.stopPropagation();
        }
        resizeE(event) {
            if (this.isResizingE) {
                const { x: newX } = this._getEventLoc(event);
                let dX = newX - this.startX;
                if (
                    dX > 0 &&
                    dX >
                        this.limitAreaWidth -
                            (this.size.width + this.loc.left + Math.abs(this.limitLeft))
                ) {
                    dX =
                        this.limitAreaWidth -
                        (this.size.width + this.loc.left + Math.abs(this.limitLeft));
                } else if (dX < 0 && Math.abs(dX) > this.size.width) {
                    dX = -this.size.width;
                }
                this.el.style.width = `${this.size.width + dX}px`;
            }
        }
        endResizeE() {
            if (!this.isResizingN && this.isResizingE && !this.isResizingW && !this.isResizingS) {
                this.isResizingE = false;
                this._triggerResizeEnd();
            }
        }
        startResizeNW(event) {
            this.startResizeN(event);
            this.startResizeW(event);
        }
        resizeNW(event) {
            this.resizeN(event);
            this.resizeW(event);
        }
        endResizeNW() {
            if (this.isResizingN && !this.isResizingE && this.isResizingW && !this.isResizingS) {
                this.isResizingN = false;
                this.isResizingW = false;
                this._triggerResizeEnd();
            }
        }
        startResizeNE(event) {
            this.startResizeN(event);
            this.startResizeE(event);
        }
        resizeNE(event) {
            this.resizeN(event);
            this.resizeE(event);
        }
        endResizeNE() {
            if (this.isResizingN && this.isResizingE && !this.isResizingW && !this.isResizingS) {
                this.isResizingN = false;
                this.isResizingE = false;
                this._triggerResizeEnd();
            }
        }
        startResizeSE(event) {
            this.startResizeS(event);
            this.startResizeE(event);
        }
        resizeSE(event) {
            this.resizeS(event);
            this.resizeE(event);
        }
        endResizeSE() {
            if (!this.isResizingN && this.isResizingE && !this.isResizingW && this.isResizingS) {
                this.isResizingS = false;
                this.isResizingE = false;
                this._triggerResizeEnd();
            }
        }
        startResizeSW(event) {
            this.startResizeS(event);
            this.startResizeW(event);
        }
        resizeSW(event) {
            this.resizeS(event);
            this.resizeW(event);
        }
        endResizeSW() {
            if (!this.isResizingN && !this.isResizingE && this.isResizingW && this.isResizingS) {
                this.isResizingS = false;
                this.isResizingW = false;
                this._triggerResizeEnd();
            }
        }
        _getEventLoc(event) {
            let coordX, coordY;
            if (event.touches && event.touches[0]) {
                coordX = event.touches[0].clientX;
                coordY = event.touches[0].clientY;
            } else {
                coordX = event.clientX;
                coordY = event.clientY;
            }
            return {
                x: coordX,
                y: coordY,
            };
        }
        _triggerResizeEnd() {
            const size = {
                height: this.el.offsetHeight,
                width: this.el.offsetWidth,
            };
            const loc = {
                top: this.el.offsetTop,
                left: this.el.offsetLeft,
            };
            this.trigger('resize-end', { size, loc });
        }
    }
    Resizeable.template = 'Resizeable';

    Registries.Component.add(Resizeable);

    return Resizeable;
});

```

## File: static\src\js\ChromeWidgets\BackToFloorButton.js

```javascript
odoo.define('pos_restaurant.BackToFloorButton', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    /**
     * Props: {
     *     onClick: callback
     * }
     */
    class BackToFloorButton extends PosComponent {
        get table() {
            return this.env.pos.table;
        }
        get floor() {
            return this.table ? this.table.floor : null;
        }
        get hasTable() {
            return this.table != null;
        }
        backToFloorScreen() {
            if (this.props.onClick) {
                this.props.onClick();
            }
            this.showScreen('FloorScreen', { floor: this.floor });
        }
    }
    BackToFloorButton.template = 'BackToFloorButton';

    Registries.Component.add(BackToFloorButton);

    return BackToFloorButton;
});

```

## File: static\src\js\ChromeWidgets\TicketButton.js

```javascript
odoo.define('pos_restaurant.TicketButton', function (require) {
    'use strict';

    const TicketButton = require('point_of_sale.TicketButton');
    const Registries = require('point_of_sale.Registries');
    const { isConnectionError } = require('point_of_sale.utils');

    const PosResTicketButton = (TicketButton) =>
        class extends TicketButton {
            async onClick() {
                if (this.env.pos.config.iface_floorplan && !this.props.isTicketScreenShown && !this.env.pos.table) {
                    try {
                        this.env.pos.setLoadingOrderState(true);
                        await this.env.pos._syncAllOrdersFromServer();
                    } catch (error) {
                        if (isConnectionError(error)) {
                            await this.showPopup('OfflineErrorPopup', {
                                title: this.env._t('Offline'),
                                body: this.env._t('Due to a connection error, the orders are not synchronized.'),
                            });
                        } else {
                            this.showPopup('ErrorPopup', {
                                title: this.env._t('Unknown error'),
                                body: error.message,
                            });
                        }
                    } finally {
                        this.env.pos.setLoadingOrderState(false);
                        this.showScreen('TicketScreen');
                    }
                } else {
                    super.onClick();
                }
            }
            /**
             * If no table is set to pos, which means the current main screen
             * is floor screen, then the order count should be based on all the orders.
             */
            get count() {
                if (!this.env.pos || !this.env.pos.config) return 0;
                if (this.env.pos.config.iface_floorplan && this.env.pos.table) {
                    return this.env.pos.getTableOrders(this.env.pos.table.id).length;
                } else {
                    return super.count;
                }
            }
        };

    Registries.Component.extend(TicketButton, PosResTicketButton);

    return TicketButton;
});

```

## File: static\src\js\Screens\BillScreen.js

```javascript
odoo.define('pos_restaurant.BillScreen', function (require) {
    'use strict';

    const ReceiptScreen = require('point_of_sale.ReceiptScreen');
    const Registries = require('point_of_sale.Registries');

    const BillScreen = (ReceiptScreen) => {
        class BillScreen extends ReceiptScreen {
            confirm() {
                this.props.resolve({ confirmed: true, payload: null });
                this.trigger('close-temp-screen');
            }
            whenClosing() {
                this.confirm();
            }
            /**
             * @override
             */
            async printReceipt() {
                const currentOrder = this.currentOrder;
                await super.printReceipt();
                currentOrder._printed = false;
                if (this.env.pos.config.iface_print_skip_screen && !this.env.isMobile) {
                    this.confirm();
                }
            }
        }
        BillScreen.template = 'BillScreen';
        return BillScreen;
    };

    Registries.Component.addByExtending(BillScreen, ReceiptScreen);

    return BillScreen;
});

```

## File: static\src\js\Screens\PaymentScreen.js

```javascript
odoo.define('pos_restaurant.PosResPaymentScreen', function (require) {
    'use strict';

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const { useListener } = require("@web/core/utils/hooks");
    const Registries = require('point_of_sale.Registries');

    const PosResPaymentScreen = (PaymentScreen) =>
        class extends PaymentScreen {
            setup() {
                super.setup();
                useListener('send-payment-adjust', this._sendPaymentAdjust);
            }

            async _sendPaymentAdjust({ detail: line }) {
                const previous_amount = line.get_amount();
                const amount_diff = line.order.get_total_with_tax() - line.order.get_total_paid();
                line.set_amount(previous_amount + amount_diff);
                line.set_payment_status('waiting');

                const payment_terminal = line.payment_method.payment_terminal;
                const isAdjustSuccessful = await payment_terminal.send_payment_adjust(line.cid);
                if (isAdjustSuccessful) {
                    line.set_payment_status('done');
                } else {
                    line.set_amount(previous_amount);
                    line.set_payment_status('done');
                }
            }

            get nextScreen() {
                const order = this.currentOrder;
                if (!this.env.pos.config.set_tip_after_payment || order.is_tipped) {
                    return super.nextScreen;
                }
                // Take the first payment method as the main payment.
                const mainPayment = order.get_paymentlines()[0];
                if (mainPayment.canBeAdjusted()) {
                    return 'TipScreen';
                }
                return super.nextScreen;
            }
        };

    Registries.Component.extend(PaymentScreen, PosResPaymentScreen);

    return PosResPaymentScreen;
});

```

## File: static\src\js\Screens\TicketScreen.js

```javascript
odoo.define('pos_restaurant.TicketScreen', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const TicketScreen = require('point_of_sale.TicketScreen');
    const Registries = require('point_of_sale.Registries');
    const { useAutofocus } = require("@web/core/utils/hooks");
    const { parse } = require('web.field_utils');

    const { useState } = owl;

    const PosResTicketScreen = (TicketScreen) =>
        class extends TicketScreen {
            close() {
                if (!this.env.pos.config.iface_floorplan) {
                    super.close();
                } else {
                    const order = this.env.pos.get_order();
                    if (order) {
                        const { name: screenName } = order.get_screen_data();
                        this.showScreen(screenName);
                    } else {
                        this.showScreen('FloorScreen');
                    }
                }
            }
            _getScreenToStatusMap() {
                return Object.assign(super._getScreenToStatusMap(), {
                    PaymentScreen: this.env.pos.config.set_tip_after_payment ? 'OPEN' : super._getScreenToStatusMap().PaymentScreen,
                    TipScreen: 'TIPPING',
                });
            }
            getTable(order) {
                const table = order.getTable();
                return table ? `${table.floor.name} (${table.name})` : '';
            }
            //@override
            _getSearchFields() {
                if (!this.env.pos.config.iface_floorplan) {
                    return super._getSearchFields();
                }
                return Object.assign({}, super._getSearchFields(), {
                    TABLE: {
                        repr: this.getTable.bind(this),
                        displayName: this.env._t('Table'),
                        modelField: 'table_id.name',
                    }
                });
            }
            async _setOrder(order) {
                if (!this.env.pos.config.iface_floorplan || this.env.pos.table) {
                    super._setOrder(order);
                } else {
                    // we came from the FloorScreen
                    const orderTable = order.getTable();
                    await this.env.pos.setTable(orderTable, order.uid);
                    this.close();
                }
            }
            shouldShowNewOrderButton() {
                return this.env.pos.config.iface_floorplan ? Boolean(this.env.pos.table) : super.shouldShowNewOrderButton();
            }
            _getOrderList() {
                if (this.env.pos.table) {
                    return this.env.pos.getTableOrders(this.env.pos.table.id);
                }
                return super._getOrderList();
            }
            async settleTips() {
                // set tip in each order
                for (const order of this.getFilteredOrderList()) {
                    const tipAmount = parse.float(order.uiState.TipScreen.inputTipAmount || '0');
                    const serverId = this.env.pos.validated_orders_name_server_id_map[order.name];
                    if (!serverId) {
                        console.warn(`${order.name} is not yet sync. Sync it to server before setting a tip.`);
                    } else {
                        const result = await this.setTip(order, serverId, tipAmount);
                        if (!result) break;
                    }
                }
            }
            //@override
            _selectNextOrder(currentOrder) {
                if (this.env.pos.config.iface_floorplan && this.env.pos.table) {
                    return super._selectNextOrder(...arguments);
                }
            }
            //@override
            async _onDeleteOrder() {
                await super._onDeleteOrder(...arguments);
                if (this.env.pos.config.iface_floorplan) {
                    if (!this.env.pos.table) {
                        this.env.pos._removeOrdersFromServer();
                    }
                    const orderList = this.env.pos.table ? this.env.pos.getTableOrders(this.env.pos.table.id) : this.env.pos.orders;
                    if (orderList.length == 0) {
                        this.showScreen('FloorScreen');
                    }
                }
            }
            async setTip(order, serverId, amount) {
                try {
                    const paymentline = order.get_paymentlines()[0];
                    if (paymentline.payment_method.payment_terminal) {
                        paymentline.amount += amount;
                        this.env.pos.set_order(order, {silent: true});
                        await paymentline.payment_method.payment_terminal.send_payment_adjust(paymentline.cid);
                    }

                    if (!amount) {
                        await this.setNoTip(serverId);
                    } else {
                        order.finalized = false;
                        order.set_tip(amount);
                        order.finalized = true;
                        const tip_line = order.selected_orderline;
                        await this.rpc({
                            method: 'set_tip',
                            model: 'pos.order',
                            args: [serverId, tip_line.export_as_JSON()],
                        });
                    }
                    if (order === this.env.pos.get_order()) {
                        this._selectNextOrder(order);
                    }
                    this.env.pos.removeOrder(order);
                    return true;
                } catch (_error) {
                    const { confirmed } = await this.showPopup('ConfirmPopup', {
                        title: 'Failed to set tip',
                        body: `Failed to set tip to ${order.name}. Do you want to proceed on setting the tips of the remaining?`,
                    });
                    return confirmed;
                }
            }
            async setNoTip(serverId) {
                await this.rpc({
                    method: 'set_no_tip',
                    model: 'pos.order',
                    args: [serverId],
                });
            }
            _getOrderStates() {
                const result = super._getOrderStates();
                if (this.env.pos.config.set_tip_after_payment) {
                    result.delete('PAYMENT');
                    result.set('OPEN', { text: this.env._t('Open'), indented: true });
                    result.set('TIPPING', { text: this.env._t('Tipping'), indented: true });
                }
                return result;
            }
            async _onDoRefund() {
                const order = this.getSelectedSyncedOrder();
                if(order && this.env.pos.config.iface_floorplan && !this.env.pos.table) {
                    this.env.pos.setTable(order.table ? order.table : Object.values(this.env.pos.tables_by_id)[0]);
                }
                super._onDoRefund();
            }
            isDefaultOrderEmpty(order) {
                if (this.env.pos.config.iface_floorplan) {
                    return false;
                }
                return super.isDefaultOrderEmpty(...arguments);
            }
        };

    Registries.Component.extend(TicketScreen, PosResTicketScreen);

    class TipCell extends PosComponent {
        setup() {
            super.setup();
            this.state = useState({ isEditing: false });
            this.orderUiState = this.props.order.uiState.TipScreen;
            useAutofocus();
        }
        get tipAmountStr() {
            return this.env.pos.format_currency(parse.float(this.orderUiState.inputTipAmount || '0'));
        }
        onBlur() {
            this.state.isEditing = false;
        }
        onKeydown(event) {
            if (event.key === 'Enter') {
                this.state.isEditing = false;
            }
        }
        editTip() {
            this.state.isEditing = true;
        }
    }
    TipCell.template = 'TipCell';

    Registries.Component.add(TipCell);

    return { TicketScreen, TipCell };
});

```

## File: static\src\js\Screens\TipScreen.js

```javascript
odoo.define('pos_restaurant.TipScreen', function (require) {
    'use strict';

    const Registries = require('point_of_sale.Registries');
    const PosComponent = require('point_of_sale.PosComponent');
    const { parse } = require('web.field_utils');
    const { renderToString } = require('@web/core/utils/render');

    const { onMounted } = owl;

    class TipScreen extends PosComponent {
        setup() {
            super.setup();
            this.state = this.currentOrder.uiState.TipScreen;
            this._totalAmount = this.currentOrder.get_total_with_tax();

            onMounted(() => {
                this.printTipReceipt();
            });
        }
        get overallAmountStr() {
            const tipAmount = parse.float(this.state.inputTipAmount || '0');
            const original = this.env.pos.format_currency(this.totalAmount);
            const tip = this.env.pos.format_currency(tipAmount);
            const overall = this.env.pos.format_currency(this.totalAmount + tipAmount);
            return `${original} + ${tip} tip = ${overall}`;
        }
        get totalAmount() {
            return this._totalAmount;
        }
        get currentOrder() {
            return this.env.pos.get_order();
        }
        get percentageTips() {
            return [
                { percentage: '15%', amount: 0.15 * this.totalAmount },
                { percentage: '20%', amount: 0.2 * this.totalAmount },
                { percentage: '25%', amount: 0.25 * this.totalAmount },
            ];
        }
        async validateTip() {
            const amount = parse.float(this.state.inputTipAmount) || 0;
            const order = this.env.pos.get_order();
            const serverId = this.env.pos.validated_orders_name_server_id_map[order.name];

            if (!serverId) {
                this.showPopup('ErrorPopup', {
                    title: this.env._t('Unsynced order'),
                    body: this.env._t('This order is not yet synced to server. Make sure it is synced then try again.'),
                });
                return;
            }

            if (!amount) {
                await this.rpc({
                    method: 'set_no_tip',
                    model: 'pos.order',
                    args: [serverId],
                });
                this.goNextScreen();
                return;
            }

            if (amount > 0.25 * this.totalAmount) {
                const { confirmed } = await this.showPopup('ConfirmPopup', {
                    title: 'Are you sure?',
                    body: `${this.env.pos.format_currency(
                        amount
                    )} is more than 25% of the order's total amount. Are you sure of this tip amount?`,
                });
                if (!confirmed) return;
            }

            // set the tip by temporarily allowing order modification
            order.finalized = false;
            order.set_tip(amount);
            order.finalized = true;

            const paymentline = this.env.pos.get_order().get_paymentlines()[0];
            if (paymentline.payment_method.payment_terminal) {
                paymentline.amount += amount;
                await paymentline.payment_method.payment_terminal.send_payment_adjust(paymentline.cid);
            }

            // set_tip calls add_product which sets the new line as the selected_orderline
            const tip_line = order.selected_orderline;
            await this.rpc({
                method: 'set_tip',
                model: 'pos.order',
                args: [serverId, tip_line.export_as_JSON()],
            });
            this.goNextScreen();
        }
        goNextScreen() {
            this.env.pos.removeOrder(this.currentOrder);
            if (!this.env.pos.config.iface_floorplan) {
                this.env.pos.add_new_order();
            }
            const { name, props } = this.nextScreen;
            this.showScreen(name, props);
        }
        get nextScreen() {
            if (this.env.pos.config.module_pos_restaurant && this.env.pos.config.iface_floorplan) {
                const table = this.env.pos.table;
                return { name: 'FloorScreen', props: { floor: table ? table.floor : null } };
            } else {
                return { name: 'ProductScreen' };
            }
        }
        async printTipReceipt() {
            const receipts = [
                this.currentOrder.selected_paymentline.ticket,
                this.currentOrder.selected_paymentline.cashier_receipt
            ];

            for (let i = 0; i < receipts.length; i++) {
                const data = receipts[i];
                var receipt = renderToString('TipReceipt', {
                    receipt: this.currentOrder.getOrderReceiptEnv().receipt,
                    data: data,
                    total: this.env.pos.format_currency(this.totalAmount),
                });

                if (this.env.proxy.printer) {
                    await this._printIoT(receipt);
                } else {
                    await this._printWeb(receipt);
                }
            }
        }

        async _printIoT(receipt) {
            const printResult = await this.env.proxy.printer.print_receipt(receipt);
            if (!printResult.successful) {
                await this.showPopup('ErrorPopup', {
                    title: printResult.message.title,
                    body: printResult.message.body,
                });
            }
        }

        async _printWeb(receipt) {
            try {
                $(this.el).find('.pos-receipt-container').html(receipt);
                window.print();
            } catch (_err) {
                await this.showPopup('ErrorPopup', {
                    title: this.env._t('Printing is not supported on some browsers'),
                    body: this.env._t(
                        'Printing is not supported on some browsers due to no default printing protocol ' +
                            'is available. It is possible to print your tickets by making use of an IoT Box.'
                    ),
                });
            }
        }
    }
    TipScreen.template = 'pos_restaurant.TipScreen';

    Registries.Component.add(TipScreen);

    return TipScreen;
});

```

## File: static\src\js\Screens\FloorScreen\EditableTable.js

```javascript
odoo.define('pos_restaurant.EditableTable', function(require) {
    'use strict';

    const { useListener } = require("@web/core/utils/hooks");
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    const { onMounted, onPatched } = owl;

    class EditableTable extends PosComponent {
        setup() {
            super.setup();
            useListener('resize-end', this._onResizeEnd);
            useListener('drag-end', this._onDragEnd);
            onPatched(this._setElementStyle.bind(this));
            onMounted(this._setElementStyle.bind(this));
        }
        _setElementStyle() {
            const table = this.props.table;
            function unit(val) {
                return `${val}px`;
            }
            const style = {
                width: unit(table.width),
                height: unit(table.height),
                'line-height': unit(table.height),
                top: unit(table.position_v),
                left: unit(table.position_h),
                'border-radius': table.shape === 'round' ? unit(1000) : '3px',
            };
            if (table.color) {
                style.background = table.color;
            }
            if (table.height >= 150 && table.width >= 150) {
                style['font-size'] = '32px';
            }
            Object.assign(this.el.style, style);
        }
        _onResizeEnd(event) {
            const { size, loc } = event.detail;
            const table = this.props.table;
            table.width = size.width;
            table.height = size.height;
            table.position_v = loc.top;
            table.position_h = loc.left;
            this.props.onSaveTable(this.props.table);
        }
        _onDragEnd(event) {
            const { loc } = event.detail;
            const table = this.props.table;
            table.position_v = loc.top;
            table.position_h = loc.left;
            this.props.onSaveTable(this.props.table);
        }
    }
    EditableTable.template = 'EditableTable';

    Registries.Component.add(EditableTable);

    return EditableTable;
});

```

## File: static\src\js\Screens\FloorScreen\EditBar.js

```javascript
odoo.define('pos_restaurant.EditBar', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    const { useState } = owl;

    class EditBar extends PosComponent {
        setup() {
            super.setup();
            this.state = useState({ isColorPicker: false })
        }
    }
    EditBar.template = 'EditBar';

    Registries.Component.add(EditBar);

    return EditBar;
});

```

## File: static\src\js\Screens\FloorScreen\FloorScreen.js

```javascript
odoo.define('pos_restaurant.FloorScreen', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');
    const { debounce } = require("@web/core/utils/timing");
    const { isConnectionError } = require('point_of_sale.utils');

    const { onPatched, onMounted, onWillUnmount, useRef, useState } = owl;

    class FloorScreen extends PosComponent {
        /**
         * @param {Object} props
         * @param {Object} props.floor
         */
        setup() {
            super.setup();
            const floor = this.props.floor ? this.props.floor : this.env.pos.floors[0];
            this.state = useState({
                selectedFloorId: floor.id,
                selectedTableId: null,
                isEditMode: false,
                floorBackground: floor.background_color,
                floorMapScrollTop: 0,
            });
            this.floorMapRef = useRef('floor-map-ref');
            onPatched(this.onPatched);
            onMounted(this.onMounted);
            onWillUnmount(this.onWillUnmount);
        }
        onPatched() {
            this.floorMapRef.el.style.background = this.state.floorBackground;
            this.state.floorMapScrollTop = this.floorMapRef.el.getBoundingClientRect().top;
        }
        onMounted() {
            if (this.env.pos.table) {
                this.env.pos.unsetTable();
            }
            this.env.posbus.trigger('start-cash-control');
            this.floorMapRef.el.style.background = this.state.floorBackground;
            this.state.floorMapScrollTop = this.floorMapRef.el.getBoundingClientRect().top;
            // call _tableLongpolling once then set interval of 5sec.
            this._tableLongpolling();
            this.tableLongpolling = setInterval(this._tableLongpolling.bind(this), 5000);
        }
        onWillUnmount() {
            clearInterval(this.tableLongpolling);
        }
        _computePinchHypo(ev, callbackFunction) {
            const touches = ev.touches;
            // If two pointers are down, check for pinch gestures
            if (touches.length === 2) {
                const deltaX = touches[0].pageX - touches[1].pageX;
                const deltaY = touches[0].pageY - touches[1].pageY;
                callbackFunction(Math.hypot(deltaX, deltaY))
            }
        }
        _onPinchStart(ev) {
            ev.currentTarget.style.setProperty('touch-action', 'none');
            this._computePinchHypo(ev, this.startPinch.bind(this));
        }
        _onPinchEnd(ev) {
            ev.currentTarget.style.removeProperty('touch-action');
        }
        _onPinchMove(ev) {
            debounce(this._computePinchHypo, 10, true)(ev, this.movePinch.bind(this));
        }
        _onDeselectTable() {
            this.state.selectedTableId = null;
        }
        async _createTableHelper(copyTable) {
            let newTable;
            if (copyTable) {
                newTable = Object.assign({}, copyTable);
                newTable.position_h += 10;
                newTable.position_v += 10;
            } else {
                newTable = {
                    position_v: 100,
                    position_h: 100,
                    width: 75,
                    height: 75,
                    shape: 'square',
                    seats: 1,
                };
            }
            newTable.name = this._getNewTableName(newTable.name);
            delete newTable.id;
            newTable.floor_id = [this.activeFloor.id, ''];
            newTable.floor = this.activeFloor;
            try {
                await this._save(newTable);
                this.activeTables.push(newTable);
                return newTable;
            } catch (error) {
                if (isConnectionError(error)) {
                    await this.showPopup('ErrorPopup', {
                        title: this.env._t('Offline'),
                        body: this.env._t('Unable to create table because you are offline.'),
                    });
                    return;
                } else {
                    throw error;
                }
            }
        }
        _getNewTableName(name) {
            if (name) {
                const num = Number((name.match(/\d+/g) || [])[0] || 0);
                const str = name.replace(/\d+/g, '');
                const n = { num: num, str: str };
                n.num += 1;
                this._lastName = n;
            } else if (this._lastName) {
                this._lastName.num += 1;
            } else {
                this._lastName = { num: 1, str: 'T' };
            }
            return '' + this._lastName.str + this._lastName.num;
        }
        async _save(table) {
            const tableCopy = { ...table };
            delete tableCopy.floor;
            const tableId = await this.rpc({
                model: 'restaurant.table',
                method: 'create_from_ui',
                args: [tableCopy],
            });
            table.id = tableId;
            this.env.pos.tables_by_id[tableId] = table;
        }
        async _tableLongpolling() {
            if (this.state.isEditMode) {
                return;
            }
            try {
                const result = await this.rpc({
                    model: 'pos.config',
                    method: 'get_tables_order_count',
                    args: [this.env.pos.config.id],
                });
                result.forEach((table) => {
                    const table_obj = this.env.pos.tables_by_id[table.id];
                    if (table_obj === undefined) {
                        console.warn(`Table with id ${table.id} is not found in the POS`);
                        return; // skip the table
                    }
                    const unsynced_orders = this.env.pos
                        .getTableOrders(table_obj.id)
                        .filter(
                            (o) =>
                                o.server_id === undefined &&
                                (o.orderlines.length !== 0 || o.paymentlines.length !== 0) &&
                                // do not count the orders that are already finalized
                                !o.finalized
                        ).length;
                    table_obj.order_count = table.orders + unsynced_orders;
                });
            } catch (error) {
                if (isConnectionError(error)) {
                    await this.showPopup('OfflineErrorPopup', {
                        title: this.env._t('Offline'),
                        body: this.env._t('Unable to get orders count'),
                    });
                } else {
                    throw error;
                }
            }
        }
        get activeFloor() {
            return this.env.pos.floors_by_id[this.state.selectedFloorId];
        }
        get activeTables() {
            return this.activeFloor.tables;
        }
        get isFloorEmpty() {
            return this.activeTables.length === 0;
        }
        get selectedTable() {
            return this.state.selectedTableId !== null
                ? this.env.pos.tables_by_id[this.state.selectedTableId]
                : false;
        }
        movePinch(hypot) {
            const delta = hypot / this.scalehypot ;
            const value = this.initalScale * delta;
            this.setScale(value);
        }
        startPinch(hypot) {
            this.scalehypot = hypot;
            this.initalScale = this.getScale();
        }
        getMapNode() {
            return this.el.querySelector('.floor-map > .tables, .floor-map > .empty-floor');
        }
        getScale() {
            const scale = this.getMapNode().style.getPropertyValue('--scale');
            const parsedScaleValue = parseFloat(scale);
            return isNaN(parsedScaleValue) ? 1 : parsedScaleValue;
        }
        setScale(value) {
            // a scale can't be a negative number
            if (value > 0) {
                this.getMapNode().style.setProperty('--scale', value);
            }
        }
        selectFloor(floor) {
            this.state.selectedFloorId = floor.id;
            this.state.floorBackground = this.activeFloor.background_color;
            this.state.isEditMode = false;
            this.state.selectedTableId = null;
        }
        toggleEditMode() {
            this.state.isEditMode = !this.state.isEditMode;
            this.state.selectedTableId = null;
        }
        async onSelectTable(table) {
            if (this.state.isEditMode) {
                this.state.selectedTableId = table.id;
            } else {
                try {
                    if (this.env.pos.orderToTransfer) {
                        await this.env.pos.transferTable(table);
                    } else {
                        await this.env.pos.setTable(table);
                    }
                } catch (error) {
                    if (isConnectionError(error)) {
                        await this.showPopup('OfflineErrorPopup', {
                            title: this.env._t('Offline'),
                            body: this.env._t('Unable to fetch orders'),
                        });
                    } else {
                        throw error;
                    }
                }
                const order = this.env.pos.get_order();
                this.showScreen(order.get_screen_data().name);
            }
        }
        async onSaveTable(table) {
            await this._save(table);
        }
        async createTable() {
            const newTable = await this._createTableHelper();
            if (newTable) {
                this.state.selectedTableId = newTable.id;
            }
        }
        async duplicateTable() {
            if (!this.selectedTable) return;
            const newTable = await this._createTableHelper(this.selectedTable);
            if (newTable) {
                this.state.selectedTableId = newTable.id;
            }
        }
        async renameTable() {
            const selectedTable = this.selectedTable;
            if (!selectedTable) return;
            const { confirmed, payload: newName } = await this.showPopup('TextInputPopup', {
                startingValue: selectedTable.name,
                title: this.env._t('Table Name ?'),
            });
            if (!confirmed) return;
            if (newName !== selectedTable.name) {
                selectedTable.name = newName;
                await this._save(selectedTable);
            }
        }
        async changeSeatsNum() {
            const selectedTable = this.selectedTable
            if (!selectedTable) return;
            const { confirmed, payload: inputNumber } = await this.showPopup('NumberPopup', {
                startingValue: selectedTable.seats,
                cheap: true,
                title: this.env._t('Number of Seats ?'),
                isInputSelected: true,
            });
            if (!confirmed) return;
            const newSeatsNum = parseInt(inputNumber, 10) || selectedTable.seats;
            if (newSeatsNum !== selectedTable.seats) {
                selectedTable.seats = newSeatsNum;
                await this._save(selectedTable);
            }
        }
        async changeShape() {
            if (!this.selectedTable) return;
            this.selectedTable.shape = this.selectedTable.shape === 'square' ? 'round' : 'square';
            this.render();
            await this._save(this.selectedTable);
        }
        async setTableColor(color) {
            this.selectedTable.color = color;
            this.render();
            await this._save(this.selectedTable);
        }
        async setFloorColor(color) {
            this.state.floorBackground = color;
            this.activeFloor.background_color = color;
            try {
                await this.rpc({
                    model: 'restaurant.floor',
                    method: 'write',
                    args: [[this.activeFloor.id], { background_color: color }],
                });
            } catch (error) {
                if (isConnectionError(error)) {
                    await this.showPopup('OfflineErrorPopup', {
                        title: this.env._t('Offline'),
                        body: this.env._t('Unable to change background color'),
                    });
                } else {
                    throw error;
                }
            }
        }
        async deleteTable() {
            if (!this.selectedTable) return;
            const { confirmed } = await this.showPopup('ConfirmPopup', {
                title: this.env._t('Are you sure ?'),
                body: this.env._t('Removing a table cannot be undone'),
            });
            if (!confirmed) return;
            try {
                const originalSelectedTableId = this.state.selectedTableId;
                await this.rpc({
                    model: 'restaurant.table',
                    method: 'create_from_ui',
                    args: [{ active: false, id: originalSelectedTableId }],
                });
                this.activeFloor.tables = this.activeTables.filter(
                    (table) => table.id !== originalSelectedTableId
                );
                // Value of an object can change inside async function call.
                //   Which means that in this code block, the value of `state.selectedTableId`
                //   before the await call can be different after the finishing the await call.
                // Since we wanted to disable the selected table after deletion, we should be
                //   setting the selectedTableId to null. However, we only do this if nothing
                //   else is selected during the rpc call.
                if (this.state.selectedTableId === originalSelectedTableId) {
                    this.state.selectedTableId = null;
                }
                delete this.env.pos.tables_by_id[originalSelectedTableId];
                this.env.pos.TICKET_SCREEN_STATE.syncedOrders.cache = {};
            } catch (error) {
                if (isConnectionError(error)) {
                    await this.showPopup('OfflineErrorPopup', {
                        title: this.env._t('Offline'),
                        body: this.env._t('Unable to delete table'),
                    });
                } else {
                    throw error;
                }
            }
        }
    }
    FloorScreen.template = 'FloorScreen';
    FloorScreen.hideOrderSelector = true;

    Registries.Component.add(FloorScreen);

    return FloorScreen;
});

```

## File: static\src\js\Screens\FloorScreen\TableWidget.js

```javascript
odoo.define('pos_restaurant.TableWidget', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    /**
     * props: {
     *  onClick: callback,
     *  table: table object,
     * }
     */
    class TableWidget extends PosComponent {
        setup() {
            owl.onMounted(this.onMounted);
        }
        onMounted() {
            const table = this.props.table;
            function unit(val) {
                return `${val}px`;
            }
            const style = {
                width: unit(table.width),
                height: unit(table.height),
                'line-height': unit(table.height),
                top: unit(table.position_v),
                left: unit(table.position_h),
                'border-radius': table.shape === 'round' ? unit(1000) : '3px',
            };
            if (table.color) {
                style.background = table.color;
            }
            if (table.height >= 150 && table.width >= 150) {
                style['font-size'] = '32px';
            }
            Object.assign(this.el.style, style);

            const tableCover = this.el.querySelector('.table-cover');
            Object.assign(tableCover.style, { height: `${Math.ceil(this.fill * 100)}%` });
        }
        get fill() {
            const customerCount = this.env.pos.getCustomerCount(this.props.table.id);
            return Math.min(1, Math.max(0, customerCount / this.props.table.seats));
        }
        get orderCount() {
            const table = this.props.table;
            return table.order_count !== undefined
                ? table.order_count
                : this.env.pos
                      .getTableOrders(table.id)
                      .filter(o => o.orderlines.length !== 0 || o.paymentlines.length !== 0).length;
        }
        get orderCountClass() {
            const countClass = { 'order-count': true }
            if (this.env.pos.config.iface_printers) {
                const notifications = this._getNotifications();
                countClass['notify-printing'] = notifications.printing;
                countClass['notify-skipped'] = notifications.skipped;
            }
            return countClass;
        }
        get customerCountDisplay() {
            return `${this.env.pos.getCustomerCount(this.props.table.id)}/${this.props.table.seats}`;
        }
        _getNotifications() {
            const orders = this.env.pos.getTableOrders(this.props.table.id);

            let hasChangesCount = 0;
            let hasSkippedCount = 0;
            for (let i = 0; i < orders.length; i++) {
                if (orders[i].hasChangesToPrint()) {
                    hasChangesCount++;
                } else if (orders[i].hasSkippedChanges()) {
                    hasSkippedCount++;
                }
            }

            return hasChangesCount ? { printing: true } : hasSkippedCount ? { skipped: true } : {};
        }
    }
    TableWidget.template = 'TableWidget';

    Registries.Component.add(TableWidget);

    return TableWidget;
});

```

## File: static\src\js\Screens\ProductScreen\Orderline.js

```javascript
odoo.define('pos_restaurant.Orderline', function(require) {
    'use strict';

    const Orderline = require('point_of_sale.Orderline');
    const Registries = require('point_of_sale.Registries');

    const PosResOrderline = Orderline =>
        class extends Orderline {
            /**
             * @override
             */
            get addedClasses() {
                const res = super.addedClasses;
                Object.assign(res, {
                    dirty: this.props.line.mp_dirty,
                    skip: this.props.line.mp_skip,
                });
                return res;
            }
            /**
             * @override
             * if doubleclick, change mp_dirty to mp_skip
             *
             * IMPROVEMENT: Instead of handling both double click and click in single
             * method, perhaps we can separate double click from single click.
             */
            selectLine() {
                const line = this.props.line; // the orderline
                if (this.env.pos.get_order().selected_orderline.id !== line.id) {
                    this.mp_dbclk_time = new Date().getTime();
                } else if (!this.mp_dbclk_time) {
                    this.mp_dbclk_time = new Date().getTime();
                } else if (this.mp_dbclk_time + 500 > new Date().getTime()) {
                    line.set_skip(!line.mp_skip);
                    this.mp_dbclk_time = 0;
                } else {
                    this.mp_dbclk_time = new Date().getTime();
                }
                super.selectLine();
            }
        };

    Registries.Component.extend(Orderline, PosResOrderline);

    return Orderline;
});

```

## File: static\src\js\Screens\ProductScreen\ControlButtons\OrderlineNoteButton.js

```javascript
odoo.define('pos_restaurant.OrderlineNoteButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require("@web/core/utils/hooks");
    const Registries = require('point_of_sale.Registries');

    class OrderlineNoteButton extends PosComponent {
        setup() {
            super.setup();
            useListener('click', this.onClick);
        }
        get selectedOrderline() {
            return this.env.pos.get_order().get_selected_orderline();
        }
        async onClick() {
            if (!this.selectedOrderline) return;

            const { confirmed, payload: inputNote } = await this.showPopup('TextAreaPopup', {
                startingValue: this.selectedOrderline.get_note(),
                title: this.env._t('Add Internal Note'),
            });

            if (confirmed) {
                this.selectedOrderline.set_note(inputNote);
            }
        }
    }
    OrderlineNoteButton.template = 'OrderlineNoteButton';

    ProductScreen.addControlButton({
        component: OrderlineNoteButton,
        condition: function() {
            return this.env.pos.config.iface_orderline_notes;
        },
    });

    Registries.Component.add(OrderlineNoteButton);

    return OrderlineNoteButton;
});

```

## File: static\src\js\Screens\ProductScreen\ControlButtons\PrintBillButton.js

```javascript
odoo.define('pos_restaurant.PrintBillButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require("@web/core/utils/hooks");
    const Registries = require('point_of_sale.Registries');

    class PrintBillButton extends PosComponent {
        setup() {
            super.setup();
            useListener('click', this.onClick);
        }
        async onClick() {
            const order = this.env.pos.get_order();
            if (order.get_orderlines().length > 0) {
                order.initialize_validation_date();
                await this.showTempScreen('BillScreen');
            } else {
                await this.showPopup('ErrorPopup', {
                    title: this.env._t('Nothing to Print'),
                    body: this.env._t('There are no order lines'),
                });
            }
        }
    }
    PrintBillButton.template = 'PrintBillButton';

    ProductScreen.addControlButton({
        component: PrintBillButton,
        condition: function() {
            return this.env.pos.config.iface_printbill;
        },
    });

    Registries.Component.add(PrintBillButton);

    return PrintBillButton;
});

```

## File: static\src\js\Screens\ProductScreen\ControlButtons\SplitBillButton.js

```javascript
odoo.define('pos_restaurant.SplitBillButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require("@web/core/utils/hooks");
    const Registries = require('point_of_sale.Registries');

    class SplitBillButton extends PosComponent {
        setup() {
            super.setup();
            useListener('click', this.onClick);
        }
        async onClick() {
            const order = this.env.pos.get_order();
            if (order.get_orderlines().length > 0) {
                this.showScreen('SplitBillScreen');
            }
        }
    }
    SplitBillButton.template = 'SplitBillButton';

    ProductScreen.addControlButton({
        component: SplitBillButton,
        condition: function() {
            return this.env.pos.config.iface_splitbill;
        },
    });

    Registries.Component.add(SplitBillButton);

    return SplitBillButton;
});

```

## File: static\src\js\Screens\ProductScreen\ControlButtons\SubmitOrderButton.js

```javascript
odoo.define('pos_restaurant.SubmitOrderButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const Registries = require('point_of_sale.Registries');

    /**
     * IMPROVEMENT: Perhaps this class is quite complicated for its worth.
     * This is because it needs to listen to changes to the current order.
     * Also, the current order changes when the selectedOrder in pos is changed.
     * After setting new current order, we update the listeners.
     */
    class SubmitOrderButton extends PosComponent {
        setup() {
            super.setup();
            this.clicked = false; //mutex, we don't want to be able to spam the printers
        }
        async _onClick() {
            if (!this.clicked) {
                try {
                    this.clicked = true;
                    const order = this.env.pos.get_order();
                    if (order.hasChangesToPrint()) {
                        const isPrintSuccessful = await order.printChanges();
                        if (isPrintSuccessful) {
                            order.updatePrintedResume();
                        } else {
                            this.showPopup('ErrorPopup', {
                                title: this.env._t('Printing failed'),
                                body: this.env._t('Failed in printing the changes in the order'),
                            });
                        }
                    }
                } finally {
                    this.clicked = false;
                }
            }
        }
        get currentOrder() {
            return this.env.pos.get_order();
        }
        get addedClasses() {
            if (!this.currentOrder) return {};
            const hasChanges = this.currentOrder.hasChangesToPrint();
            const skipped = hasChanges ? false : this.currentOrder.hasSkippedChanges();
            return {
                highlight: hasChanges,
                altlight: skipped,
            };
        }
    }
    SubmitOrderButton.template = 'SubmitOrderButton';

    ProductScreen.addControlButton({
        component: SubmitOrderButton,
        condition: function() {
            return this.env.pos.config.module_pos_restaurant && this.env.pos.unwatched.printers.length;
        },
    });

    Registries.Component.add(SubmitOrderButton);

    return SubmitOrderButton;
});

```

## File: static\src\js\Screens\ProductScreen\ControlButtons\TableGuestsButton.js

```javascript
odoo.define('pos_restaurant.TableGuestsButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require("@web/core/utils/hooks");
    const Registries = require('point_of_sale.Registries');

    class TableGuestsButton extends PosComponent {
        setup() {
            super.setup();
            useListener('click', this.onClick);
        }
        get currentOrder() {
            return this.env.pos.get_order();
        }
        get nGuests() {
            return this.currentOrder ? this.currentOrder.getCustomerCount() : 0;
        }
        async onClick() {
            const { confirmed, payload: inputNumber } = await this.showPopup('NumberPopup', {
                startingValue: this.nGuests,
                cheap: true,
                title: this.env._t('Guests ?'),
                isInputSelected: true
            });

            if (confirmed) {
                const guestCount = parseInt(inputNumber, 10) || 1;
                // Set the maximum number possible for an integer
                const max_capacity = 2**31 - 1;
                if (guestCount > max_capacity) {
                    await this.showPopup('ErrorPopup', {
                        title: this.env._t('Blocked action'),
                        body: _.str.sprintf(
                            this.env._t('You cannot put a number that exceeds %s '),
                            max_capacity,
                        ),
                    });
                    return;
                }
                this.env.pos.get_order().setCustomerCount(guestCount);
            }
        }
    }
    TableGuestsButton.template = 'TableGuestsButton';

    ProductScreen.addControlButton({
        component: TableGuestsButton,
        condition: function() {
            return this.env.pos.config.module_pos_restaurant;
        },
    });

    Registries.Component.add(TableGuestsButton);

    return TableGuestsButton;
});

```

## File: static\src\js\Screens\ProductScreen\ControlButtons\TransferOrderButton.js

```javascript
odoo.define('pos_restaurant.TransferOrderButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require("@web/core/utils/hooks");
    const Registries = require('point_of_sale.Registries');

    class TransferOrderButton extends PosComponent {
        setup() {
            super.setup();
            useListener('click', this.onClick);
        }
        async onClick() {
            this.env.pos.setCurrentOrderToTransfer();
            this.showScreen('FloorScreen');
        }
    }
    TransferOrderButton.template = 'TransferOrderButton';

    ProductScreen.addControlButton({
        component: TransferOrderButton,
        condition: function() {
            return this.env.pos.config.iface_floorplan;
        },
    });

    Registries.Component.add(TransferOrderButton);

    return TransferOrderButton;
});

```

## File: static\src\js\Screens\ReceiptScreen\ReceiptScreen.js

```javascript
odoo.define('pos_restaurant.ReceiptScreen', function(require) {
    'use strict';

    const ReceiptScreen = require('point_of_sale.ReceiptScreen');
    const Registries = require('point_of_sale.Registries');

    const PosResReceiptScreen = ReceiptScreen =>
        class extends ReceiptScreen {
            //@override
            _addNewOrder() {
                if (!this.env.pos.config.iface_floorplan) {
                    super._addNewOrder();
                }
            }
            //@override
            get nextScreen() {
                if (this.env.pos.config.iface_floorplan) {
                    const table = this.env.pos.table;
                    return { name: 'FloorScreen', props: { floor: table ? table.floor : null } };
                } else {
                    return super.nextScreen;
                }
            }
            onBackToFloorButtonClick() {
                // If we're here and the order is paid, we can remove it from the orders
                this.env.pos.removeOrder(this.currentOrder);
            }
        };

    Registries.Component.extend(ReceiptScreen, PosResReceiptScreen);

    return ReceiptScreen;
});

```

## File: static\src\js\Screens\SplitBillScreen\SplitBillScreen.js

```javascript
odoo.define('pos_restaurant.SplitBillScreen', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const { useListener } = require("@web/core/utils/hooks");
    const { Order } = require('point_of_sale.models');
    const Registries = require('point_of_sale.Registries');

    const { useState, onMounted } = owl;

    class SplitBillScreen extends PosComponent {
        setup() {
            super.setup();
            useListener('click-line', this.onClickLine);
            this.splitlines = useState(this._initSplitLines(this.env.pos.get_order()));
            this.newOrderLines = {};
            this.newOrder = undefined;
            this._isFinal = false;
            onMounted(() => {
                // Should create the new order outside of the constructor because
                // sequence_number of pos_session is modified. which will trigger
                // rerendering which will rerender this screen and will be infinite loop.
                this.newOrder = Order.create(
                    {},
                    {
                        pos: this.env.pos,
                        temporary: true,
                    }
                );
                this.render();
            });
        }
        get disallow() {
            return false;
        }
        get currentOrder() {
            return this.env.pos.get_order();
        }
        get orderlines() {
            return this.currentOrder.get_orderlines();
        }
        onClickLine(event) {
            const line = event.detail;
            this._splitQuantity(line);
            this._updateNewOrder(line);
        }
        back() {
            this.showScreen('ProductScreen');
        }
        proceed() {
            if (_.isEmpty(this.splitlines))
                // Splitlines is empty
                return;

            this._isFinal = true;
            delete this.newOrder.temporary;

            if (!this._isFullPayOrder()) {
                this._setQuantityOnCurrentOrder();

                this.newOrder.set_screen_data({ name: 'PaymentScreen' });

                // for the kitchen printer we assume that everything
                // has already been sent to the kitchen before splitting
                // the bill. So we save all changes both for the old
                // order and for the new one. This is not entirely correct
                // but avoids flooding the kitchen with unnecessary orders.
                // Not sure what to do in this case.
                if (this.env.pos.config.iface_printers) {
                    this.currentOrder.updatePrintedResume();
                    this.newOrder.updatePrintedResume();
                }

                this.newOrder.setCustomerCount(1);
                const newCustomerCount = this.currentOrder.getCustomerCount() - 1;
                this.currentOrder.setCustomerCount(newCustomerCount || 1);
                this.currentOrder.set_screen_data({ name: 'ProductScreen' });

                const reactiveNewOrder = this.env.pos.makeOrderReactive(this.newOrder);
                this.env.pos.orders.add(reactiveNewOrder);
                this.env.pos.selectedOrder = reactiveNewOrder;
            }
            this.showScreen('PaymentScreen');
        }
        /**
         * @param {models.Order} order
         * @returns {Object<{ quantity: number }>} splitlines
         */
        _initSplitLines(order) {
            const splitlines = {};
            for (let line of order.get_orderlines()) {
                splitlines[line.id] = { product: line.get_product().id, quantity: 0 };
            }
            return splitlines;
        }
        _splitQuantity(line) {
            const split = this.splitlines[line.id];

            let totalQuantity = 0;

            this.env.pos.get_order().get_orderlines().forEach(function(orderLine) {
                if(orderLine.get_product().id === split.product)
                    totalQuantity += orderLine.get_quantity();
            });

            if(line.get_quantity() > 0) {
                if (!line.get_unit().is_pos_groupable) {
                    if (split.quantity !== line.get_quantity()) {
                        split.quantity = line.get_quantity();
                    } else {
                        split.quantity = 0;
                    }
                } else {
                    if (split.quantity < totalQuantity) {
                        split.quantity += line.get_unit().is_pos_groupable? 1: line.get_unit().rounding;
                        if (split.quantity > line.get_quantity()) {
                            split.quantity = line.get_quantity();
                        }
                    } else {
                        split.quantity = 0;
                    }
                }
            }
        }
        _updateNewOrder(line) {
            const split = this.splitlines[line.id];
            let orderline = this.newOrderLines[line.id];
            if (split.quantity) {
                if (!orderline) {
                    orderline = line.clone();
                    this.newOrder.add_orderline(orderline);
                    this.newOrderLines[line.id] = orderline;
                }
                orderline.set_quantity(split.quantity, 'do not recompute unit price');
            } else if (orderline) {
                this.newOrder.remove_orderline(orderline);
                this.newOrderLines[line.id] = null;
            }
        }
        _isFullPayOrder() {
            let order = this.env.pos.get_order();
            let full = true;
            let splitlines = this.splitlines;
            let groupedLines = _.groupBy(order.get_orderlines(), line => line.get_product().id);

            Object.keys(groupedLines).forEach(function (lineId) {
                var maxQuantity = groupedLines[lineId].reduce(((quantity, line) => quantity + line.get_quantity()), 0);
                Object.keys(splitlines).forEach(id => {
                    let split = splitlines[id];
                    if(split.product === groupedLines[lineId][0].get_product().id)
                        maxQuantity -= split.quantity;
                });
                if(maxQuantity !== 0)
                    full = false;
            });

            return full;
        }
        _setQuantityOnCurrentOrder() {
            let order = this.env.pos.get_order();
            for (var id in this.splitlines) {
                var split = this.splitlines[id];
                var line = this.currentOrder.get_orderline(parseInt(id));

                if(!this.disallow) {
                    line.set_quantity(
                        line.get_quantity() - split.quantity,
                        'do not recompute unit price'
                    );
                    if (Math.abs(line.get_quantity()) < 0.00001) {
                        this.currentOrder.remove_orderline(line);
                    }
                } else {
                    if(split.quantity) {
                        let decreaseLine = line.clone();
                        decreaseLine.order = order;
                        decreaseLine.noDecrease = true;
                        decreaseLine.set_quantity(-split.quantity);
                        order.add_orderline(decreaseLine);
                    }
                }
            }
        }
    }
    SplitBillScreen.template = 'SplitBillScreen';

    Registries.Component.add(SplitBillScreen);

    return SplitBillScreen;
});

```

## File: static\src\js\Screens\SplitBillScreen\SplitOrderline.js

```javascript
odoo.define('pos_restaurant.SplitOrderline', function(require) {
    'use strict';

    const { useListener } = require("@web/core/utils/hooks");
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    class SplitOrderline extends PosComponent {
        setup() {
            super.setup();
            useListener('click', this.onClick);
        }
        get isSelected() {
            return this.props.split.quantity !== 0;
        }
        onClick() {
            this.trigger('click-line', this.props.line);
        }
    }
    SplitOrderline.template = 'SplitOrderline';

    Registries.Component.add(SplitOrderline);

    return SplitOrderline;
});

```

## File: static\src\xml\Chrome.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="Chrome" t-inherit="point_of_sale.Chrome" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('status-buttons')]" position="before">
            <div class="back-to-floor-portal"/>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\multiprint.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="OrderChangeReceipt">
        <div class="pos-receipt">
            <div class="pos-receipt-order-data"><t t-esc="changes.name" /></div>
            <t t-if="changes.floor_name || changes.table_name">
                <br />
                <div class="pos-receipt-title">
                    <t t-esc="changes.floor_name" /> / <t t-esc="changes.table_name"/>
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
                        <div class="multiprint-flex">
                            <span class="product-quantity" t-esc="change.quantity"/>
                            <span class="product-name" t-esc="change.name"/>
                        </div>
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
                    <div class="multiprint-flex">
                        <span class="product-quantity" t-esc="change.quantity"/>
                        <span class="product-name" t-esc="change.name"/>
                    </div>
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

## File: static\src\xml\Resizeable.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="Resizeable" owl="1">
        <t t-slot="default"></t>
    </t>

</templates>

```

## File: static\src\xml\TipReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="TipReceipt" owl="1">
        <div class="pos-receipt">
            <t t-if="receipt.company.logo">
                <img class="pos-receipt-logo" t-att-src="receipt.company.logo" alt="Logo"/>
                <br/>
            </t>
            <t t-if="!receipt.company.logo">
                <h2 class="pos-receipt-center-align">
                    <t t-esc="receipt.company.name" />
                </h2>
                <br/>
            </t>
            <div class="pos-receipt-contact">
                <t t-if="receipt.company.contact_address">
                    <div><t t-esc="receipt.company.contact_address" /></div>
                </t>
                <t t-if="receipt.company.phone">
                    <div>Tel:<t t-esc="receipt.company.phone" /></div>
                </t>
                <t t-if="receipt.company.vat">
                    <div>VAT:<t t-esc="receipt.company.vat" /></div>
                </t>
                <t t-if="receipt.company.email">
                    <div><t t-esc="receipt.company.email" /></div>
                </t>
                <t t-if="receipt.company.website">
                    <div><t t-esc="receipt.company.website" /></div>
                </t>
                <t t-if="receipt.header_html">
                    <t t-out="receipt.header_html" />
                </t>
                <t t-if="!receipt.header_html and receipt.header">
                    <div><t t-esc="receipt.header" /></div>
                </t>
                <t t-if="receipt.cashier">
                    <div class="cashier">
                        <div>--------------------------------</div>
                        <div>Served by <t t-esc="receipt.cashier" /></div>
                    </div>
                </t>
            </div>
            <br/>

            <div class="pos-payment-terminal-receipt">
                <t t-out="data"/>
            </div>
            <br/>


            <div class="subtotal">
                <span>Subtotal</span>
                <div class="pos-receipt-right-align"><t t-esc="total"/></div>
            </div>
            <br/>

            <div class="tip">
                <span>Tip:</span>
                <div class="pos-receipt-right-align">________________________</div>
            </div>
            <br/>

            <div class="total">
                <span>Total:</span>
                <div class="pos-receipt-right-align">________________________</div>
            </div>
            <br/>
            <br/>

            <div class="signature">
                <div>______________________________________________</div>
                <div>Signature</div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\ChromeWidgets\BackToFloorButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="BackToFloorButton" owl="1">
        <span
            t-if="hasTable"
            class="order-button floor-button"
            t-att-class="{ oe_hidden: props.mobileSearchBarIsShown }"
            t-on-click="backToFloorScreen"
        >
            <i class="fa fa-angle-double-left" role="img" aria-label="Back to floor" title="Back to floor" />
            <t t-if="env.isMobile">
                <span class="table-name" t-esc="table.name"/>
            </t>
            <t t-else="">
                <span t-esc="floor.name" /><span class="table-name">(<t t-esc="table.name" />)</span>
            </t>
        </span>
        <span t-else=""></span>
    </t>

</templates>

```

## File: static\src\xml\Screens\BillScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="BillScreen" owl="1">
        <div class="receipt-screen screen">
            <div class="screen-content">
                <div class="top-content">
                    <div class="top-content-center">
                        <h1>Bill Printing</h1>
                    </div>
                    <span class="button next highlight" t-on-click="confirm">
                        <span>Ok</span>
                        <span> </span>
                        <i class="fa fa-angle-double-right"></i>
                    </span>
                </div>
                <div class="centered-content">
                    <div class="button print" t-on-click="printReceipt">
                        <i class="fa fa-print"></i>
                        <span> </span>
                        <span>Print</span>
                    </div>
                    <div class="pos-receipt-container" t-ref="order-receipt">
                        <OrderReceipt order="currentOrder" isBill="true"/>
                    </div>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\TicketScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="TicketScreen" t-inherit="point_of_sale.TicketScreen" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('header-row')]//div[@name='delete']" position="before">
            <div t-if="env.pos.config.iface_floorplan" class="col" name="table">Table</div>
            <div t-if="_state.ui.filter == 'TIPPING'" class="col end narrow" name="tip">Tip</div>
        </xpath>
        <xpath expr="//div[hasclass('order-row')]//div[@name='delete']" position="before">
            <div t-if="env.pos.config.iface_floorplan" class="col" name="table">
                <t t-if="order.tableId">
                    <div t-if="env.isMobile">Table</div>
                    <div><t t-esc="getTable(order)"></t></div>
                </t>
            </div>
            <div t-if="_state.ui.filter == 'TIPPING'" class="col end narrow" name="tip">
                <div t-if="env.isMobile">Tip</div>
                <div><TipCell order="order" /></div>
            </div>
        </xpath>
        <xpath expr="//div[hasclass('buttons')]" position="inside">
            <button class="settle-tips" t-if="_state.ui.filter == 'TIPPING'" t-on-click="settleTips">Settle</button>
        </xpath>
    </t>

    <t t-name="TipCell" owl="1">
        <div class="tip-cell" t-on-click.stop="editTip">
            <t t-if="state.isEditing">
                <input type="text" name="tip-amount" t-ref="autofocus" t-model="orderUiState.inputTipAmount" t-on-blur="onBlur" t-on-keydown="onKeydown" />
            </t>
            <div t-else="">
                <t t-esc="tipAmountStr"></t>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\TipScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.TipScreen" owl="1">
        <div class="tip-screen screen">
            <div class="pos-receipt-container"/>
            <div class="screen-content">
                <div class="top-content">
                    <span class="button back" t-on-click="() => this.showScreen('FloorScreen')">
                        <i class="fa fa-angle-double-left"></i>
                        <span> </span>
                        <span>Back</span>
                    </span>
                    <span class="button" t-if="env.proxy.printer" t-on-click="printTipReceipt">
                        <i class="fa fa-print"></i>
                        <span> </span>
                        <span>Reprint receipts</span>
                    </span>
                    <div class="top-content-center">
                        <h1>Add a tip</h1>
                    </div>
                    <div class="button highlight next" t-on-click="validateTip">
                        Settle <i class="fa fa-angle-double-right"></i>
                    </div>
                </div>
                <div class="tip-options">
                    <div class="total-amount">
                        <t t-esc="overallAmountStr" />
                    </div>
                    <div class="tip-amount-options">
                        <div class="percentage-amounts">
                            <t t-foreach="percentageTips" t-as="tip" t-key="tip.percentage">
                                <div class="button" t-on-click="() => { state.inputTipAmount = tip.amount.toFixed(2); }">
                                    <div class="percentage">
                                        <t t-esc="tip.percentage"></t>
                                    </div>
                                    <div class="amount">
                                        <t t-esc="env.pos.format_currency(tip.amount)" />
                                    </div>
                                </div>
                            </t>
                        </div>
                        <div class="no-tip" t-on-click="() => { state.inputTipAmount = '0'; }">
                            <div class="button">No Tip</div>
                        </div>
                        <div class="custom-amount-form">
                            <div class="item label">Amount</div>
                            <div class="item input">
                                <input type="text" t-model="state.inputTipAmount" t-att-data-amount="state.inputTipAmount" />
                                <div class="currency">
                                    <t t-esc="env.pos.getCurrencySymbol()" />
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <t t-portal="'.pos .back-to-floor-portal'" position="before">
                <BackToFloorButton mobileSearchBarIsShown="props.mobileSearchBarIsShown"/>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\FloorScreen\EditableTable.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">

    <t t-name="EditableTable" owl="1">
        <Draggable limitArea="'.floor-map'">
            <Resizeable limitArea="'.floor-map'">
                <div class="table selected" t-on-click.stop="">
                    <span class="label drag-handle">
                        <t t-esc="props.table.name" />
                    </span>
                    <span class="table-seats">
                        <t t-esc="props.table.seats" />
                    </span>
                    <t t-if="props.table.shape === 'round'">
                        <div class="table-handle top resize-handle-n"></div>
                        <div class="table-handle bottom resize-handle-s"></div>
                        <div class="table-handle left resize-handle-w"></div>
                        <div class="table-handle right resize-handle-e"></div>
                    </t>
                    <t t-if="props.table.shape === 'square'">
                        <span class='table-handle top right resize-handle-ne'></span>
                        <span class='table-handle top left resize-handle-nw'></span>
                        <span class='table-handle bottom right resize-handle-se'></span>
                        <span class='table-handle bottom left resize-handle-sw'></span>
                    </t>
                </div>
            </Resizeable>
        </Draggable>
    </t>

</templates>

```

## File: static\src\xml\Screens\FloorScreen\EditBar.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">

    <t t-name="EditBar" owl="1">
        <div class="edit-bar" t-attf-style="top:{{props.floorMapScrollTop}}px;">
            <span class="edit-button" t-on-click.stop="props.createTable">
                <i class="fa fa-plus" role="img" aria-label="Add" title="Add"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="props.duplicateTable">
                <i class="fa fa-files-o" role="img" aria-label="Duplicate" title="Duplicate"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="props.renameTable">
                <i class="fa fa-font" role="img" aria-label="Rename" title="Rename"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="props.changeSeatsNum">
                <i class="fa fa-user" role="img" aria-label="Seats" title="Seats"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="props.changeShape">
                <span t-if="!props.selectedTable or props.selectedTable.shape == 'square'" class="button-option square">
                    <i class="fa fa-square-o" role="img" aria-label="Square Shape" title="Square Shape"></i>
                </span>
                <span t-else="" class="button-option round">
                    <i class="fa fa-circle-o" role="img" aria-label="Round Shape" title="Round Shape"></i>
                </span>
            </span>
            <span class="edit-button" t-on-click.stop="() => { state.isColorPicker = !state.isColorPicker }">
                <i class="fa fa-tint" role="img" aria-label="Tint" title="Tint"></i>
            </span>
            <div t-if="state.isColorPicker and props.selectedTable" class="color-picker fg-picker">
                <div  class="close-picker" title="Close" role="img" aria-label="Close" t-on-click.stop="() => { state.isColorPicker = false; }">
                    <i class="fa fa-times" />
                </div>
                <span class="color tl"  style="background-color:#EB6D6D" role="img" aria-label="Red" title="Red" t-on-click.stop="() => props.setTableColor('#EB6D6D')" />
                <span class="color"     style="background-color:#35D374" role="img" aria-label="Green" title="Green" t-on-click.stop="() => props.setTableColor('#35D374')" />
                <span class="color tr"  style="background-color:#6C6DEC" role="img" aria-label="Blue" title="Blue" t-on-click.stop="() => props.setTableColor('#6C6DEC')" />
                <span class="color"     style="background-color:#EBBF6D" role="img" aria-label="Orange" title="Orange" t-on-click.stop="() => props.setTableColor('#EBBF6D')" />
                <span class="color"     style="background-color:#EBEC6D" role="img" aria-label="Yellow" title="Yellow" t-on-click.stop="() => props.setTableColor('#EBEC6D')" />
                <span class="color"     style="background-color:#AC6DAD" role="img" aria-label="Purple" title="Purple" t-on-click.stop="() => props.setTableColor('#AC6DAD')" />
                <span class="color bl"  style="background-color:#6C6D6D" role="img" aria-label="Grey" title="Grey" t-on-click.stop="() => props.setTableColor('#6C6D6D')" />
                <span class="color"     style="background-color:#ACADAD" role="img" aria-label="Light grey" title="Light grey" t-on-click.stop="() => props.setTableColor('#ACADAD')" />
                <span class="color br"  style="background-color:#4ED2BE" role="img" aria-label="Turquoise" title="Turquoise" t-on-click.stop="() => props.setTableColor('#4ED2BE')" />
            </div>
            <div t-if="state.isColorPicker and !props.selectedTable" class="color-picker bg-picker">
                <div  class="close-picker" title="Close" role="img" aria-label="Close" t-on-click.stop="() => { state.isColorPicker = false; }">
                    <i class="fa fa-times" />
                </div>
                <span class="color tl"  style="background-color:rgb(244, 149, 149)" role="img" aria-label="Red" title="Red" t-on-click.stop="() => props.setFloorColor('rgb(244, 149, 149)')" />
                <span class="color"     style="background-color:rgb(130, 233, 171)" role="img" aria-label="Green" title="Green" t-on-click.stop="() => props.setFloorColor('rgb(130, 233, 171)')" />
                <span class="color tr"  style="background-color:rgb(136, 137, 242)" role="img" aria-label="Blue" title="Blue" t-on-click.stop="() => props.setFloorColor('rgb(136, 137, 242)')" />
                <span class="color"     style="background-color:rgb(255, 214, 136)" role="img" aria-label="Orange" title="Orange" t-on-click.stop="() => props.setFloorColor('rgb(255, 214, 136)')" />
                <span class="color"     style="background-color:rgb(254, 255, 154)" role="img" aria-label="Yellow" title="Yellow" t-on-click.stop="() => props.setFloorColor('rgb(254, 255, 154)')" />
                <span class="color"     style="background-color:rgb(209, 171, 210)" role="img" aria-label="Purple" title="Purple" t-on-click.stop="() => props.setFloorColor('rgb(209, 171, 210)')" />
                <span class="color bl"  style="background-color:rgb(75, 75, 75)"    role="img" aria-label="Grey" title="Grey" t-on-click.stop="() => props.setFloorColor('rgb(75, 75, 75)')" />
                <span class="color"     style="background-color:rgb(210, 210, 210)" role="img" aria-label="Light grey" title="Light grey" t-on-click.stop="() => props.setFloorColor('rgb(210, 210, 210)')" />
                <span class="color br"  style="background-color:rgb(127, 221, 236)" role="img" aria-label="Turquoise" title="Turquoise" t-on-click.stop="() => props.setFloorColor('rgb(127, 221, 236)')" />
            </div>
            <span class="edit-button trash" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="props.deleteTable">
                <i class="fa fa-trash" role="img" aria-label="Delete" title="Delete"></i>
            </span>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\FloorScreen\FloorScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="FloorScreen" owl="1">
        <div class="floor-screen screen">
            <div class="screen-content-flexbox">
                <t t-if="env.pos.floors.length > 1">
                    <div class="floor-selector">
                        <t t-foreach="env.pos.floors" t-as="floor" t-key="floor.id">
                            <span class="button button-floor" t-att-class="{ active: floor.id === state.selectedFloorId }" t-on-click="() => this.selectFloor(floor)">
                                <t t-esc="floor.name" />
                            </span>
                        </t>
                    </div>
                </t>

                <div
                    t-on-click="_onDeselectTable"
                    t-on-touchstart="_onPinchStart"
                    t-on-touchmove="_onPinchMove"
                    t-on-touchend="_onPinchEnd"
                    class="floor-map"
                    t-ref="floor-map-ref"
                >
                    <div t-if="isFloorEmpty" class="empty-floor">
                        <span>This floor has no tables yet, use the </span>
                        <i class="fa fa-plus" role="img" aria-label="Add button" title="Add button"></i>
                        <span> button in the editing toolbar to create new tables.</span>
                    </div>
                    <div t-else="" class="tables">
                        <t t-foreach="activeTables" t-as="table" t-key="table.id">
                            <TableWidget t-if="table.id !== state.selectedTableId" onClick.bind="onSelectTable" table="table" />
                            <EditableTable t-else="" table="table" onSaveTable.bind="onSaveTable" />
                        </t>
                    </div>
                    <span t-if="env.pos.user.role == 'manager'" class="edit-button editing" t-att-class="{ active: state.isEditMode }" t-on-click.stop="toggleEditMode"
                          t-attf-style="top:{{state.floorMapScrollTop}}px;">
                        <i class="fa fa-pencil" role="img" aria-label="Edit" title="Edit"></i>
                    </span>
                    <EditBar t-if="state.isEditMode" selectedTable="selectedTable" floorMapScrollTop="state.floorMapScrollTop"
                             createTable.bind="createTable" duplicateTable.bind="duplicateTable" renameTable.bind="renameTable"
                             changeSeatsNum.bind="changeSeatsNum" changeShape.bind="changeShape" setTableColor.bind="setTableColor"
                             setFloorColor.bind="setFloorColor" deleteTable.bind="deleteTable"
                    />
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\Screens\FloorScreen\TableWidget.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">

    <t t-name="TableWidget" owl="1">
        <div class="table" t-on-click.stop="() => props.onClick(props.table)">
            <span class="table-cover" t-att-class="{ full: fill >= 1 }"></span>
            <span t-att-class="orderCountClass" t-att-hidden="orderCount === 0">
                <t t-esc="orderCount" />
            </span>
            <span class="label">
                <t t-esc="props.table.name" />
            </span>
            <span class="table-seats">
                <t t-esc="customerCountDisplay" />
            </span>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\PaymentScreen\PaymentScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="PaymentScreen" t-inherit="point_of_sale.PaymentScreen" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('payment-screen')]" position="inside">
            <t t-portal="'.pos .back-to-floor-portal'" position="before">
                <BackToFloorButton mobileSearchBarIsShown="props.mobileSearchBarIsShown"/>
            </t>
        </xpath>

        <xpath expr="//div[hasclass('button') and hasclass('next')]" position="attributes">
            <attribute name="t-att-hidden">env.pos.config.set_tip_after_payment and !currentOrder.is_paid()</attribute>
        </xpath>

        <xpath expr="//div[hasclass('button') and hasclass('back')]/span[hasclass('back_text')]" position="replace">
            <t t-if="env.pos.config.set_tip_after_payment and currentOrder.is_paid()">
                <span class="back_text">Keep Open</span>
            </t>
            <t t-else="">$0</t>
        </xpath>

        <xpath expr="//div[hasclass('button') and hasclass('next')]/span[hasclass('next_text')]" position="replace">
            <t t-if="env.pos.config.set_tip_after_payment and currentOrder.is_paid()">
                <span class="back_text">Close Tab</span>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\Screens\PaymentScreen\PaymentScreenPaymentLines.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="PaymentScreenPaymentLines" t-inherit="point_of_sale.PaymentScreenPaymentLines" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('send_payment_reversal')]/.." position="replace">
            <t t-if="line.canBeAdjusted() &amp;&amp; line.order.get_total_paid() &lt; line.order.get_total_with_tax()">
                <div class="button send_adjust_amount" title="Adjust Amount" t-on-click="() => this.trigger('send-payment-adjust', line)">
                    Adjust Amount
                </div>
            </t>
            <t t-elif="line.can_be_reversed">
                <div class="button send_payment_reversal" title="Reverse Payment" t-on-click="() => this.trigger('send-payment-reverse', line)">
                    Reverse
                </div>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\Screens\ProductScreen\Orderline.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="Orderline" t-inherit="point_of_sale.Orderline" t-inherit-mode="extension" owl="1">
        <xpath expr="//ul[hasclass('info-list')]" position="inside">
            <t t-if="props.line.get_note()">
                <li class="info orderline-note">
                    <i class="fa fa-tag" role="img" aria-label="Note" title="Note"/>
                    <t t-esc="props.line.get_note()" />
                </li>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\Screens\ProductScreen\ProductScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="ProductScreen" t-inherit="point_of_sale.ProductScreen" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('product-screen')]" position="inside">
            <t t-portal="'.pos .back-to-floor-portal'" position="before">
                <BackToFloorButton mobileSearchBarIsShown="props.mobileSearchBarIsShown"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\Screens\ProductScreen\ControlButtons\OrderlineNoteButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="OrderlineNoteButton" owl="1">
        <div class="control-button">
            <i class="fa fa-tag" />
            <span> </span>
            <span>Internal Note</span>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\ProductScreen\ControlButtons\PrintBillButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="PrintBillButton" owl="1">
        <span class="control-button order-printbill">
            <i class="fa fa-print"></i>
            <span> </span>
            <span>Bill</span>
        </span>
    </t>

</templates>

```

## File: static\src\xml\Screens\ProductScreen\ControlButtons\SplitBillButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SplitBillButton" owl="1">
        <span class="control-button order-split">
            <i class="fa fa-files-o"></i>
            <span> </span>
            <span>Split</span>
        </span>
    </t>

</templates>

```

## File: static\src\xml\Screens\ProductScreen\ControlButtons\SubmitOrderButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SubmitOrderButton" owl="1">
        <span class="control-button" t-att-class="addedClasses" t-on-click="_onClick">
            <i class="fa fa-cutlery"></i>
            <span> </span>
            <span>Order</span>
        </span>
    </t>

</templates>

```

## File: static\src\xml\Screens\ProductScreen\ControlButtons\TableGuestsButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="TableGuestsButton" owl="1">
        <div class="control-button">
            <span class="control-button-number">
                <t t-esc="nGuests" />
            </span>
            <span> </span>
            <span>Guests</span>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\ProductScreen\ControlButtons\TransferOrderButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="TransferOrderButton" owl="1">
        <div class="control-button">
            <i class="fa fa-arrow-right" />
            <span> </span>
            <span>Transfer</span>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\ReceiptScreen\OrderReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('pos-receipt-order-data')]" position="inside">
            <t t-if="props.isBill">
                <div>PRO FORMA</div>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('receipt-change')]" position="attributes">
            <attribute name="t-if">!props.isBill</attribute>
        </xpath>
        <xpath expr="//div[hasclass('cashier')]" position="after">
            <t t-if="receipt.table">
                at table <t t-esc="receipt.table" />
            </t>
            <t t-if="receipt.table and receipt.customer_count">
                <div>Guests: <t t-esc="receipt.customer_count" /></div>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('before-footer')]" position="after">
            <t t-if="props.isBill and env.pos.config.set_tip_after_payment">
                <div class="tip-form">
                    <div class="title">For convenience, we are providing the following gratuity calculations:</div>
                    <div class="percentage-options">
                        <div class="option">
                            <div>15%</div>
                            <div class="amount">
                                <t t-esc="env.pos.format_currency(receipt.total_with_tax * 0.15)"></t>
                            </div>
                        </div>
                        <div class="option">
                            <div>20%</div>
                            <div class="amount">
                                <t t-esc="env.pos.format_currency(receipt.total_with_tax * 0.20)"></t>
                            </div>
                        </div>
                        <div class="option">
                            <div>25%</div>
                            <div class="amount">
                                <t t-esc="env.pos.format_currency(receipt.total_with_tax * 0.25)"></t>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\Screens\ReceiptScreen\ReceiptScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="ReceiptScreen" t-inherit="point_of_sale.ReceiptScreen" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('receipt-screen')]" position="inside">
            <t t-portal="'.pos .back-to-floor-portal'" position="before">
                <BackToFloorButton mobileSearchBarIsShown="props.mobileSearchBarIsShown"
                                   onClick.bind="onBackToFloorButtonClick"
                />
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\Screens\SplitBillScreen\SplitBillScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SplitBillScreen" owl="1">
        <div class="splitbill-screen screen">
            <div class="contents">
                <div class="top-content">
                    <span class="button back" t-on-click="back">
                        <i class="fa fa-angle-double-left"></i>
                        <span> </span>
                        <span>Back</span>
                    </span>
                    <div class="top-content-center">
                        <h1>Bill Splitting</h1>
                    </div>
                </div>
                <div t-if="newOrder" class="main">
                    <div class="lines">
                        <div class="order">
                            <ul class="orderlines">
                                <t t-foreach="orderlines" t-as="line" t-key="line.cid">
                                    <SplitOrderline line="line" split="splitlines[line.id]" />
                                </t>
                            </ul>
                        </div>
                    </div>
                    <div class="controls">
                        <div class="order-info">
                            <span class="subtotal">
                                <t t-esc="env.pos.format_currency(newOrder.get_subtotal())" />
                            </span>
                        </div>
                        <div class="pay-button">
                            <div class="button" t-on-click="proceed">
                                <i class="fa fa-chevron-right" />
                                <span> </span>
                                <span>Payment</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\SplitBillScreen\SplitOrderline.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SplitOrderline" owl="1">
        <li class="orderline" t-att-class="{ selected: isSelected, partially: props.split.quantity !== props.line.get_quantity() }">
            <span class="product-name">
                <t t-esc="props.line.get_product().display_name" />
            </span>
            <span class="price">
                <t t-esc="env.pos.format_currency(props.line.get_display_price())" />
            </span>
            <ul class="info-list">
                <t t-if="props.line.get_quantity_str() !== '1'">
                    <li class="info">
                        <t t-if="isSelected and props.line.get_unit().is_pos_groupable">
                            <em class="big">
                                <t t-esc="props.split.quantity" />
                            </em>
                            /
                            <t t-esc="props.line.get_quantity_str()" />
                        </t>
                        <t t-if="!(isSelected and props.line.get_unit().is_pos_groupable)">
                            <em>
                                <t t-esc="props.line.get_quantity_str()" />
                            </em>
                        </t>
                        <t t-esc="props.line.get_unit().name" />
                        at
                        <t t-esc="env.pos.format_currency(props.line.get_unit_price())" />
                        /
                        <t t-esc="props.line.get_unit().name" />
                    </li>
                </t>
                <t t-if="props.line.get_discount_str() !== '0'">
                    <li class="info">
                        <span>With a </span>
                        <em>
                            <t t-esc="props.line.get_discount_str()" />
                            <span>%</span>
                        </em>
                        <span> discount</span>
                    </li>
                </t>
            </ul>
        </li>
    </t>

</templates>

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
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="active" invisible="1"/>
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

        <record id="view_restaurant_floor_search" model="ir.ui.view">
            <field name="name">restaurant.floor.search</field>
            <field name="model">restaurant.floor</field>
            <field name="arch" type="xml">
                <search>
                    <field name="name"/>
                    <filter string="Archived" name="active" domain="[('active', '=', False)]"/>
                </search>
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
                    <field name="product_categories_ids" widget="many2many_tags"/>
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

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_restaurant</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="warning_text_pos_restaurant" position="replace"/>
            <div id="pos_interface_section" position="after">
                <h2 class="mt16" attrs="{'invisible': [('pos_module_pos_restaurant', '=', False)]}">Restaurant &amp; Bar</h2>
                <div class="row mt16 o_settings_container" id="restaurant_section" attrs="{'invisible': [('pos_module_pos_restaurant', '=', False)]}">
                    <div class="col-12 col-lg-6 o_setting_box"
                         id="is_table_management"
                         attrs="{'invisible': [('pos_module_pos_restaurant', '=', False)]}">
                        <div class="o_setting_left_pane">
                            <field name="pos_is_table_management" attrs="{'readonly': [('pos_has_active_session','=', True)]}"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="pos_is_table_management" string="Floors &amp; Tables Map"/>
                            <div class="text-muted">
                                Design floors and assign orders to tables
                            </div>
                            <div class="content-group" attrs="{'invisible': [('pos_is_table_management','=',False)]}">
                                <div class="mt16">
                                    <label string="Floors" for="pos_floor_ids" class="o_light_label"/>
                                    <field name="pos_floor_ids" widget="many2many_tags" attrs="{'readonly': [('pos_has_active_session','=', True)]}" />
                                </div>
                                <div>
                                    <button name="%(pos_restaurant.action_restaurant_floor_form)d" icon="fa-arrow-right" type="action" string="Floors" class="btn-link"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box"
                         id="iface_orderline_notes"
                         attrs="{'invisible': [('pos_module_pos_restaurant', '=', False)]}">
                        <div class="o_setting_left_pane">
                            <field name="pos_iface_orderline_notes"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="pos_iface_orderline_notes" string="Kitchen Notes"/>
                            <div class="text-muted">
                                Add internal notes on order lines for the kitchen
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box"
                         id="iface_printbill"
                         attrs="{'invisible': [('pos_module_pos_restaurant', '=', False)]}">
                        <div class="o_setting_left_pane">
                            <field name="pos_iface_printbill"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="pos_iface_printbill" string="Early Receipt Printing" />
                            <div class="text-muted">
                                Allow to print receipt before payment
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box"
                         id="iface_splitbill"
                         attrs="{'invisible': [('pos_module_pos_restaurant', '=', False)]}">
                        <div class="o_setting_left_pane">
                            <field name="pos_iface_splitbill" string="Allow Bill Splitting"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="pos_iface_splitbill" string="Allow Bill Splitting"/>
                            <div class="text-muted">
                                Split total or order lines
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box"
                         id="is_order_printer"
                         attrs="{'invisible': [('pos_module_pos_restaurant', '=', False)]}">
                        <div class="o_setting_left_pane">
                            <field name="pos_is_order_printer"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="pos_is_order_printer" string="Kitchen Printers"/>
                            <div class="text-muted">
                                Print orders at the kitchen, at the bar, etc.
                            </div>
                            <div class="content-group" attrs="{'invisible': [('pos_is_order_printer', '=', False)]}">
                                <div class="mt16">
                                    <label string="Printers" for="pos_printer_ids" class="o_light_label"/>
                                    <field name="pos_printer_ids" widget="many2many_tags"/>
                                </div>
                                <div>
                                    <button name="%(pos_restaurant.action_restaurant_printer_form)d" icon="fa-arrow-right" type="action" string="Printers" class="btn-link"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div id="tip_product" position="after">
                <div attrs="{'invisible': ['|', ('pos_module_pos_restaurant', '=', False), ('pos_iface_tipproduct', '=', False)]}">
                    <field name="pos_set_tip_after_payment" class="oe_inline"/>
                    <label class="fw-normal" for="pos_set_tip_after_payment" string="Add tip after payment (North America specific)"/>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

