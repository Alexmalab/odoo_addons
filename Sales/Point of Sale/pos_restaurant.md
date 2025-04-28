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
    'website': 'https://www.odoo.com/page/point-of-sale-restaurant',
    'data': [
        'security/ir.model.access.csv',
        'views/pos_order_views.xml',
        'views/pos_restaurant_views.xml',
        'views/pos_config_views.xml',
        'views/pos_restaurant_templates.xml',
    ],
    'qweb': [
        'static/src/xml/Resizeable.xml',
        'static/src/xml/Chrome.xml',
        'static/src/xml/Screens/TicketScreen.xml',
        'static/src/xml/Screens/OrderManagementScreen/OrderList.xml',
        'static/src/xml/Screens/OrderManagementScreen/OrderRow.xml',
        'static/src/xml/Screens/ProductScreen/ControlButtons/OrderlineNoteButton.xml',
        'static/src/xml/Screens/ProductScreen/ControlButtons/TableGuestsButton.xml',
        'static/src/xml/Screens/ProductScreen/ControlButtons/PrintBillButton.xml',
        'static/src/xml/Screens/ProductScreen/ControlButtons/SubmitOrderButton.xml',
        'static/src/xml/Screens/ProductScreen/ControlButtons/SplitBillButton.xml',
        'static/src/xml/Screens/ProductScreen/ControlButtons/TransferOrderButton.xml',
        'static/src/xml/Screens/BillScreen.xml',
        'static/src/xml/Screens/SplitBillScreen/SplitBillScreen.xml',
        'static/src/xml/Screens/SplitBillScreen/SplitOrderline.xml',
        'static/src/xml/Screens/ReceiptScreen/OrderReceipt.xml',
        'static/src/xml/Screens/ProductScreen/Orderline.xml',
        'static/src/xml/Screens/PaymentScreen/PaymentScreen.xml',
        'static/src/xml/Screens/PaymentScreen/PaymentScreenElectronicPayment.xml',
        'static/src/xml/Screens/FloorScreen/FloorScreen.xml',
        'static/src/xml/Screens/FloorScreen/EditBar.xml',
        'static/src/xml/Screens/FloorScreen/TableWidget.xml',
        'static/src/xml/Screens/FloorScreen/EditableTable.xml',
        'static/src/xml/Screens/TipScreen.xml',
        'static/src/xml/ChromeWidgets/BackToFloorButton.xml',
        'static/src/xml/multiprint.xml',
        'static/src/xml/TipReceipt.xml',
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
    iface_orderline_notes = fields.Boolean(string='Notes', help='Allow custom notes on Orderlines.')
    floor_ids = fields.One2many('restaurant.floor', 'pos_config_id', string='Restaurant Floors', help='The restaurant floors served by this point of sale.')
    printer_ids = fields.Many2many('restaurant.printer', 'pos_config_printer_rel', 'config_id', 'printer_id', string='Order Printers')
    is_table_management = fields.Boolean('Floors & Tables')
    is_order_printer = fields.Boolean('Order Printer')
    set_tip_after_payment = fields.Boolean('Set Tip After Payment', help="Adjust the amount authorized by payment terminals to add a tip after the customers left or at the end of the day.")
    module_pos_restaurant = fields.Boolean(default=True)

    @api.onchange('module_pos_restaurant')
    def _onchange_module_pos_restaurant(self):
        if not self.module_pos_restaurant:
            self.update({'iface_printbill': False,
            'iface_splitbill': False,
            'is_order_printer': False,
            'is_table_management': False,
            'iface_orderline_notes': False})

    @api.onchange('iface_tipproduct')
    def _onchange_iface_tipproduct(self):
        if not self.iface_tipproduct:
            self.set_tip_after_payment = False

    def _force_http(self):
        enforce_https = self.env['ir.config_parameter'].sudo().get_param('point_of_sale.enforce_https')
        if not enforce_https and self.printer_ids.filtered(lambda pt: pt.printer_type == 'epson_epos'):
            return True
        return super(PosConfig, self)._force_http()

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

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.tools import groupby
from re import search
from functools import partial

from odoo import api, fields, models


class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    note = fields.Char('Note added by the waiter.')
    mp_skip = fields.Boolean('Skip line when sending ticket to kitchen printers.')
    mp_dirty = fields.Boolean()


class PosOrder(models.Model):
    _inherit = 'pos.order'

    table_id = fields.Many2one('restaurant.table', string='Table', help='The table where this order was served', index=True)
    customer_count = fields.Integer(string='Guests', help='The amount of customers that have been served by this order.')
    multiprint_resume = fields.Char()

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
        return [
            'id',
            'discount',
            'product_id',
            'price_unit',
            'order_id',
            'qty',
            'note',
            'mp_skip',
            'mp_dirty',
            'full_product_name',
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
            else:
                order_line['pack_lot_ids'] = [[0, 0, lot] for lot in order_line['pack_lot_ids']]
            extended_order_lines.append([0, 0, order_line])

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

    def _get_payment_lines(self, orders):
        """Add account_bank_statement_lines to the orders.

        The function doesn't return anything but adds the results directly to the orders.

        :param orders: orders for which the payment_lines are to be requested.
        :type orders: pos.order.
        """
        payment_lines = self.env['pos.payment'].search_read(
                domain = [('pos_order_id', 'in', [po['id'] for po in orders])],
                fields = self._get_fields_for_payment_lines())

        extended_payment_lines = []
        for payment_line in payment_lines:
            payment_line['server_id'] = payment_line['id']
            payment_line['payment_method_id'] = payment_line['payment_method_id'][0]

            del payment_line['id']
            extended_payment_lines.append([0, 0, payment_line])
        for order_id, payment_lines in groupby(extended_payment_lines, key=lambda x:x[2]['pos_order_id']):
            next(order for order in orders if order['id'] == order_id[0])['statement_ids'] = list(payment_lines)

    def _get_fields_for_draft_order(self):
        fields = super(PosOrder, self)._get_fields_for_draft_order()
        fields.extend([
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
                    ])
        return fields

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
                domain=[('state', '=', 'draft'), ('table_id', '=', table_id)],
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
            if 'employee_id' in order:
                order['employee_id'] = order['employee_id'][0] if order['employee_id'] else False

            if not 'lines' in order:
                order['lines'] = []
            if not 'statement_ids' in order:
                order['statement_ids'] = []

            del order['id']
            del order['session_id']
            del order['pos_reference']
            del order['create_date']

        return table_orders

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

    name = fields.Char('Floor Name', required=True, help='An internal identification of the restaurant floor')
    pos_config_id = fields.Many2one('pos.config', string='Point of Sale')
    background_image = fields.Binary('Background Image', help='A background image used to display a floor layout in the point of sale interface')
    background_color = fields.Char('Background Color', help='The background color of the floor layout, (must be specified in a html-compatible format)', default='rgb(210, 210, 210)')
    table_ids = fields.One2many('restaurant.table', 'floor_id', string='Tables', help='The list of tables in this floor')
    sequence = fields.Integer('Sequence', help='Used to sort Floors', default=1)
    active = fields.Boolean(default=True)

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

    def write(self, vals):
        for floor in self:
            if floor.pos_config_id.has_active_session and (vals.get('pos_config_id') or vals.get('active')) :
                raise UserError(
                    'Please close and validate the following open PoS Session before modifying this floor.\n'
                    'Open session: %s' % (' '.join(floor.pos_config_id.mapped('name')),))
            if vals.get('pos_config_id') and floor.pos_config_id.id and vals.get('pos_config_id') != floor.pos_config_id.id:
                raise UserError('The %s is already used in another Pos Config.' % floor.name)
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

        table_id = table.pop('id', False)
        if table_id:
            self.browse(table_id).write(table)
        else:
            table_id = self.create(table).id
        return table_id

    def unlink(self):
        confs = self.mapped('floor_id').mapped('pos_config_id').filtered(lambda c: c.is_table_management == True)
        opened_session = self.env['pos.session'].search([('config_id', 'in', confs.ids), ('state', '!=', 'closed')])
        if opened_session:
            error_msg = _("You cannot remove a table that is used in a PoS session, close the session(s) first.")
            if confs:
                raise UserError(error_msg)
        return super(RestaurantTable, self).unlink()


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
from . import pos_payment
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
            /**
             * @override
             * Order is set to null when table is selected. There is no saved
             * screen for null order so show `FloorScreen` instead.
             */
            _showSavedScreen(pos, newSelectedOrder) {
                if (!newSelectedOrder) {
                    this.showScreen('FloorScreen', { floor: pos.table ? pos.table.floor : null });
                } else {
                    super._showSavedScreen(pos, newSelectedOrder);
                }
            }
            _setActivityListeners() {
                IDLE_TIMER_SETTER = this._setIdleTimer.bind(this);
                for (const event of NON_IDLE_EVENTS) {
                    window.addEventListener(event, IDLE_TIMER_SETTER);
                }
            }
            _setIdleTimer() {
                if (this._shouldResetIdleTimer()) {
                    clearTimeout(this.idleTimer);
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
                this.showScreen('FloorScreen', { floor: table ? table.floor : null });
            }
            _shouldResetIdleTimer() {
                return this.env.pos.config.iface_floorplan && this.mainScreen.name !== 'FloorScreen';
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

## File: static\src\js\floors.js

```javascript
odoo.define('pos_restaurant.floors', function (require) {
"use strict";

var models = require('point_of_sale.models');
const { Gui } = require('point_of_sale.Gui');
const { posbus } = require('point_of_sale.utils');

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

// New orders are now associated with the current table, if any.
var _super_order = models.Order.prototype;
models.Order = models.Order.extend({
    initialize: function(attr,options) {
        _super_order.initialize.apply(this,arguments);
        if (!this.table && !options.json) {
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

// We need to change the way the regular UI sees the orders, it
// needs to only see the orders associated with the current table,
// and when an order is validated, it needs to go back to the floor map.
//
// And when we change the table, we must create an order for that table
// if there is none.
var _super_posmodel = models.PosModel.prototype;
models.PosModel = models.PosModel.extend({
    after_load_server_data: async function() {
        var res = await _super_posmodel.after_load_server_data.call(this);
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
        return self._remove_from_server(ids_to_remove)
            .then(function(server_ids) {
                self.set_synch('connected');
            }).catch(function(reason){
                self.set_synch('error');
                throw reason;
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
        var timeout = typeof options.timeout === 'number' ? options.timeout : 7500;
        return this.rpc({
                model: 'pos.order',
                method: 'get_table_draft_orders',
                args: [table_id],
                kwargs: {context: this.session.user_context},
            }, {
                timeout: timeout,
                shadow: false,
            })
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

    /**
     * @param {models.Order} order order to set
     */
    set_order_on_table: function(order) {
        var orders = this.get_order_list();
        if (orders.length) {
            order = order ? orders.find((o) => o.uid === order.uid) : null;
            if (order) {
                this.set_order(order);
            } else {
                // do not mindlessly set the first order in the list.
                orders = orders.filter(order => !order.finalized);
                if (orders.length) {
                    this.set_order(orders[0]);
                } else {
                    this.add_new_order();
                }
            }
        } else {
            this.add_new_order();  // or create a new order with the current table
        }
    },

    sync_to_server: function(table, order) {
        var self = this;
        var ids_to_remove = this.db.get_ids_to_remove_from_server();

        this.set_synch('connecting', 1);
        this._get_from_server(table.id).then(function (server_orders) {
            var orders = self.get_order_list();
            self._replace_orders(orders, server_orders);
            if (!ids_to_remove.length) {
                self.set_synch('connected');
            } else {
                self.remove_from_server_and_set_sync_state(ids_to_remove);
            }
        }).catch(function(reason){
            self.set_synch('error');
        }).finally(function(){
            self.set_order_on_table(order);
        });
    },
    _replace_orders: function(orders_to_replace, new_orders) {
        var self = this;
        orders_to_replace.forEach(function(order){
            // We don't remove the validated orders because we still want to see them
            // in the ticket screen. Orders in 'ReceiptScreen' or 'TipScreen' are validated
            // orders.
            if (order.server_id && !order.finalized){
                self.get("orders").remove(order);
                order.destroy();
            }
        });
        new_orders.forEach(function(server_order){
            var new_order = new models.Order({},{pos: self, json: server_order});
            self.get("orders").add(new_order);
            new_order.save_to_db();
        });
    },
    //@throw error
    replace_table_orders_from_server: async function(table) {
        const server_orders = await this._get_from_server(table.id);
        const orders = this.get_table_orders(table);
        this._replace_orders(orders, server_orders);
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
     * @param {models.Order|undefined} order if provided, set to this order
     */
    set_table: function(table, order) {
        if(!table){
            this.sync_from_server(table, this.get_order_list(), this.get_order_with_uid());
            this.set_order(null);
            this.table = null;
        } else if (this.order_to_transfer_to_different_table) {
            var order_ids = this.get_order_with_uid();

            this.transfer_order_to_table(table);
            this.push_order_for_transfer(order_ids, this.get_order_list());

            this.sync_from_server(table, this.get_order_list(), order_ids);
            this.set_order(null);
        } else {
            this.table = table;
            this.sync_to_server(table, order);
        }
        posbus.trigger('table-set');
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
                return _super_posmodel.add_new_order.apply(this, arguments);
            } else {
                Gui.showPopup('ConfirmPopup', {
                    title: 'Unable to create order',
                    body: 'Orders cannot be created when there is no active table in restaurant mode',
                });
                return undefined;
            }
        } else {
            return _super_posmodel.add_new_order.apply(this,arguments);
        }
    },


    // get the list of unpaid orders (associated to the current table)
    get_order_list: function() {
        var orders = _super_posmodel.get_order_list.call(this);
        if (!(this.config && this.config.iface_floorplan)) {
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
        var orders = this.get_table_orders(table).filter(order => !order.finalized);
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
                this.set_order(order_list[index] || order_list[order_list.length - 1], { silent: true });
            } else if (order_list.length === 0) {
                this.table ? this.set_order(null) : this.set_table(null);
            }
        } else {
            _super_posmodel.on_removed_order.apply(this,arguments);
        }
    },


});


var _super_paymentline = models.Paymentline.prototype;
models.Paymentline = models.Paymentline.extend({
    /**
     * Override this method to be able to show the 'Adjust Authorisation' button
     * on a validated payment_line and to show the tip screen which allow
     * tipping even after payment. By default, this returns true for all
     * non-cash payment.
     */
    canBeAdjusted: function() {
        if (this.payment_method.payment_terminal) {
            return this.payment_method.payment_terminal.canBeAdjusted(this.cid);
        }
        return !this.payment_method.is_cash_count;
    },
});

});

```

## File: static\src\js\multiprint.js

```javascript
odoo.define('pos_restaurant.multiprint', function (require) {
"use strict";

var models = require('point_of_sale.models');
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
    set_quantity: function(quantity, keep_price) {
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
            var product_name = line.get_full_product_name();
            var p_key = product_id + " - " + product_name;
            var product_resume = p_key in resume ? resume[p_key] : {
                pid: product_id,
                product_name_wrapped: line.generate_wrapped_product_name(),
                qties: {},
            };
            if (note in product_resume['qties']) product_resume['qties'][note] += qty;
            else product_resume['qties'][note] = qty;
            resume[p_key] = product_resume;
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
        var p_key, note;

        for (p_key in current_res) {
            for (note in current_res[p_key]['qties']) {
                var curr = current_res[p_key];
                var old  = old_res[p_key] || {};
                var pid = curr.pid;
                var found = p_key in old_res && note in old_res[p_key]['qties'];

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

        for (p_key in old_res) {
            for (note in old_res[p_key]['qties']) {
                var found = p_key in current_res && note in current_res[p_key]['qties'];
                if (!found) {
                    var old = old_res[p_key];
                    var pid = old.pid;
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
        let isPrintSuccessful = true;
        for(var i = 0; i < printers.length; i++){
            var changes = this.computeChanges(printers[i].config.product_categories_ids);
            if ( changes['new'].length > 0 || changes['cancelled'].length > 0){
                var receipt = QWeb.render('OrderChangeReceipt',{changes:changes, widget:this});
                const result = await printers[i].print_receipt(receipt);
                if (!result.successful) {
                    isPrintSuccessful = false;
                }
            }
        }
        return isPrintSuccessful;
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
        json.multiprint_resume = JSON.stringify(this.saved_resume);
        return json;
    },
    init_from_JSON: function(json){
        _super_order.init_from_JSON.apply(this,arguments);
        this.saved_resume = json.multiprint_resume && JSON.parse(json.multiprint_resume);
    },
});


});

```

## File: static\src\js\notes.js

```javascript
odoo.define('pos_restaurant.notes', function (require) {
"use strict";

var models = require('point_of_sale.models');

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

    const { useExternalListener } = owl.hooks;
    const { useListener } = require('web.custom_hooks');
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    class Resizeable extends PosComponent {
        constructor() {
            super(...arguments);

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
        }
        mounted() {
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
    const { posbus } = require('point_of_sale.utils');

    class BackToFloorButton extends PosComponent {
        mounted() {
            posbus.on('table-set', this, this.render);
        }
        willUnmount() {
            posbus.on('table-set', this);
        }
        get table() {
            return (this.env.pos && this.env.pos.table) || null;
        }
        get floor() {
            const table = this.table;
            return table ? table.floor : null;
        }
        get hasTable() {
            return this.table !== null;
        }
        backToFloorScreen() {
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
    const { posbus } = require('point_of_sale.utils');

    const PosResTicketButton = (TicketButton) =>
        class extends TicketButton {
            async onClick() {
                if (this.env.pos.config.iface_floorplan && !this.props.isTicketScreenShown && !this.env.pos.table) {
                    await this._syncAllFromServer();
                    this.showScreen('TicketScreen');
                } else {
                    super.onClick();
                }
            }
            async _syncAllFromServer() {
                const pos = this.env.pos;
                try {
                    for (const floor of pos.floors) {
                        for (const table of floor.tables) {
                            await pos.replace_table_orders_from_server(table);
                        }
                    }
                } catch (e) {
                    await this.showPopup('ErrorPopup', {
                        title: this.env._t('Connection Error'),
                        body: this.env._t('Due to a connection error, the orders are not synchronized.'),
                    });
                }
            }
            mounted() {
                posbus.on('table-set', this, this.render);
            }
            willUnmount() {
                posbus.off('table-set', this);
            }
            /**
             * If no table is set to pos, which means the current main screen
             * is floor screen, then the order count should be based on all the orders.
             */
            get count() {
                if (!this.env.pos || !this.env.pos.config) return 0;
                if (this.env.pos.config.iface_floorplan && !this.env.pos.table) {
                    return this.env.pos.get('orders').models.length;
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
                await super.printReceipt();
                this.currentOrder._printed = false;
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
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    const PosResPaymentScreen = (PaymentScreen) =>
        class extends PaymentScreen {
            constructor() {
                super(...arguments);
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
    const { useAutofocus } = require('web.custom_hooks');
    const { posbus } = require('point_of_sale.utils');
    const { parse } = require('web.field_utils');
    const { useState, useContext } = owl.hooks;

    const PosResTicketScreen = (TicketScreen) =>
        class extends TicketScreen {
            close() {
                super.close();
                if (!this.env.pos.config.iface_floorplan) {
                    // Make sure the 'table-set' event is triggered
                    // to properly rerender the components that listens to it.
                    posbus.trigger('table-set');
                }
            }
            get filterOptions() {
                const { Payment, Open, Tipping } = this.getOrderStates();
                var filterOptions = super.filterOptions;
                if (this.env.pos.config.set_tip_after_payment) {
                    var idx = filterOptions.indexOf(Payment);
                    filterOptions[idx] = Open;
                }
                return [...filterOptions, Tipping];
            }
            get _screenToStatusMap() {
                const { Open, Tipping } = this.getOrderStates();
                return Object.assign(super._screenToStatusMap, {
                    PaymentScreen: this.env.pos.config.set_tip_after_payment ? Open : super._screenToStatusMap.PaymentScreen,
                    TipScreen: Tipping,
                });
            }
            getTable(order) {
                return `${order.table.floor.name} (${order.table.name})`;
            }
            get _searchFields() {
                if (!this.env.pos.config.iface_floorplan) {
                    return super._searchFields;
                }
                return Object.assign({}, super._searchFields, {
                    Table: (order) => `${order.table.floor.name} (${order.table.name})`,
                });
            }
            _setOrder(order) {
                if (!this.env.pos.config.iface_floorplan || this.env.pos.table) {
                    super._setOrder(order);
                } else {
                    this.env.pos.set_table(order.table, order);
                }
            }
            get showNewTicketButton() {
                return this.env.pos.config.iface_floorplan ? Boolean(this.env.pos.table) : super.showNewTicketButton;
            }
            get orderList() {
                if (this.env.pos.table) {
                    return super.orderList;
                } else {
                    return this.env.pos.get('orders').models;
                }
            }
            async settleTips() {
                // set tip in each order
                for (const order of this.filteredOrderList) {
                    const tipAmount = parse.float(order.uiState.TipScreen.state.inputTipAmount || '0');
                    const serverId = this.env.pos.validated_orders_name_server_id_map[order.name];
                    if (!serverId) {
                        console.warn(`${order.name} is not yet sync. Sync it to server before setting a tip.`);
                    } else {
                        const result = await this.setTip(order, serverId, tipAmount);
                        if (!result) break;
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
                        await this.setNoTip();
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
                    order.finalize();
                    return true;
                } catch (error) {
                    const { confirmed } = await this.showPopup('ConfirmPopup', {
                        title: 'Failed to set tip',
                        body: `Failed to set tip to ${order.name}. Do you want to proceed on setting the tips of the remaining?`,
                    });
                    return confirmed;
                }
            }
            async setNoTip() {
                await this.rpc({
                    method: 'set_no_tip',
                    model: 'pos.order',
                    args: [serverId],
                });
            }
            getOrderStates() {
                return Object.assign(super.getOrderStates(), {
                    Tipping: this.env._t('Tipping'),
                    Open: this.env._t('Open'),
                });
            }
        };

    Registries.Component.extend(TicketScreen, PosResTicketScreen);

    class TipCell extends PosComponent {
        constructor() {
            super(...arguments);
            this.state = useState({ isEditing: false });
            this.orderUiState = useContext(this.props.order.uiState.TipScreen);
            useAutofocus({ selector: 'input' });
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
    const { useContext } = owl.hooks;

    class TipScreen extends PosComponent {
        constructor() {
            super(...arguments);
            this.state = useContext(this.currentOrder.uiState.TipScreen);
            this._totalAmount = this.currentOrder.get_total_with_tax();
        }
        mounted () {
            this.printTipReceipt();
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
                    title: 'Unsynced order',
                    body: 'This order is not yet synced to server. Make sure it is synced then try again.',
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
            this.env.pos.get_order().finalize();
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
                var receipt = this.env.qweb.renderToString('TipReceipt', {
                    receipt: this.currentOrder.getOrderReceiptEnv().receipt,
                    data: data,
                    total: this.env.pos.format_currency(this.totalAmount),
                });

                if (this.env.pos.proxy.printer) {
                    await this._printIoT(receipt);
                } else {
                    await this._printWeb(receipt);
                }
            }
        }

        async _printIoT(receipt) {
            const printResult = await this.env.pos.proxy.printer.print_receipt(receipt);
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
            } catch (err) {
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

    const { onPatched, onMounted } = owl.hooks;
    const { useListener } = require('web.custom_hooks');
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    class EditableTable extends PosComponent {
        constructor() {
            super(...arguments);
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
            this.trigger('save-table', this.props.table);
        }
        _onDragEnd(event) {
            const { loc } = event.detail;
            const table = this.props.table;
            table.position_v = loc.top;
            table.position_h = loc.left;
            this.trigger('save-table', this.props.table);
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
    const { useState } = owl.hooks;

    class EditBar extends PosComponent {
        constructor() {
            super(...arguments);
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

    const { debounce } = owl.utils;
    const PosComponent = require('point_of_sale.PosComponent');
    const { useState, useRef } = owl.hooks;
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class FloorScreen extends PosComponent {
        /**
         * @param {Object} props
         * @param {Object} props.floor
         */
        constructor() {
            super(...arguments);
            this._setTableColor = debounce(this._setTableColor, 70);
            this._setFloorColor = debounce(this._setFloorColor, 70);
            useListener('select-table', this._onSelectTable);
            useListener('deselect-table', this._onDeselectTable);
            useListener('save-table', this._onSaveTable);
            useListener('create-table', this._createTable);
            useListener('duplicate-table', this._duplicateTable);
            useListener('rename-table', this._renameTable);
            useListener('change-seats-num', this._changeSeatsNum);
            useListener('change-shape', this._changeShape);
            useListener('set-table-color', this._setTableColor);
            useListener('set-floor-color', this._setFloorColor);
            useListener('delete-table', this._deleteTable);
            const floor = this.props.floor ? this.props.floor : this.env.pos.floors[0];
            this.state = useState({
                selectedFloorId: floor.id,
                selectedTableId: null,
                isEditMode: false,
                floorBackground: floor.background_color,
                floorMapScrollTop: 0,
            });
            this.floorMapRef = useRef('floor-map-ref');
        }
        patched() {
            this.floorMapRef.el.style.background = this.state.floorBackground;
            this.state.floorMapScrollTop = this.floorMapRef.el.getBoundingClientRect().top;
        }
        mounted() {
            if (this.env.pos.table) {
                this.env.pos.set_table(null);
            }
            this.floorMapRef.el.style.background = this.state.floorBackground;
            this.state.floorMapScrollTop = this.floorMapRef.el.getBoundingClientRect().top;
            // call _tableLongpolling once then set interval of 5sec.
            this._tableLongpolling();
            this.tableLongpolling = setInterval(this._tableLongpolling.bind(this), 5000);
        }
        willUnmount() {
            clearInterval(this.tableLongpolling);
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
        async _createTable() {
            const newTable = await this._createTableHelper();
            if (newTable) {
                this.state.selectedTableId = newTable.id;
            }
        }
        async _duplicateTable() {
            if (!this.selectedTable) return;
            const newTable = await this._createTableHelper(this.selectedTable);
            if (newTable) {
                this.state.selectedTableId = newTable.id;
            }
        }
        async _changeSeatsNum() {
            const selectedTable = this.selectedTable
            if (!selectedTable) return;
            const { confirmed, payload: inputNumber } = await this.showPopup('NumberPopup', {
                startingValue: selectedTable.seats,
                cheap: true,
                title: this.env._t('Number of Seats ?'),
            });
            if (!confirmed) return;
            const newSeatsNum = parseInt(inputNumber, 10) || selectedTable.seats;
            if (newSeatsNum !== selectedTable.seats) {
                selectedTable.seats = newSeatsNum;
                await this._save(selectedTable);
            }
        }
        async _changeShape() {
            if (!this.selectedTable) return;
            this.selectedTable.shape = this.selectedTable.shape === 'square' ? 'round' : 'square';
            this.render();
            await this._save(this.selectedTable);
        }
        async _renameTable() {
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
        async _setTableColor({ detail: color }) {
            this.selectedTable.color = color;
            this.render();
            await this._save(this.selectedTable);
        }
        async _setFloorColor({ detail: color }) {
            this.state.floorBackground = color;
            this.activeFloor.background_color = color;
            try {
                await this.rpc({
                    model: 'restaurant.floor',
                    method: 'write',
                    args: [[this.activeFloor.id], { background_color: color }],
                });
            } catch (error) {
                if (error.message.code < 0) {
                    await this.showPopup('OfflineErrorPopup', {
                        title: this.env._t('Offline'),
                        body: this.env._t('Unable to change background color'),
                    });
                } else {
                    throw error;
                }
            }
        }
        async _deleteTable() {
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
            } catch (error) {
                if (error.message.code < 0) {
                    await this.showPopup('OfflineErrorPopup', {
                        title: this.env._t('Offline'),
                        body: this.env._t('Unable to delete table'),
                    });
                } else {
                    throw error;
                }
            }
        }
        _onSelectTable(event) {
            const table = event.detail;
            if (this.state.isEditMode) {
                this.state.selectedTableId = table.id;
            } else {
                this.env.pos.set_table(table);
            }
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
                if (error.message.code < 0) {
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
            const fields = this.env.pos.models.find((model) => model.model === 'restaurant.table')
                .fields;
            const serializeTable = {};
            for (let field of fields) {
                if (typeof table[field] !== 'undefined') {
                    serializeTable[field] = table[field];
                }
            }
            serializeTable.id = table.id;
            const tableId = await this.rpc({
                model: 'restaurant.table',
                method: 'create_from_ui',
                args: [serializeTable],
            });
            table.id = tableId;
            this.env.pos.tables_by_id[tableId] = table;
        }
        async _onSaveTable(event) {
            const table = event.detail;
            await this._save(table);
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
                    const unsynced_orders = this.env.pos
                        .get_table_orders(table_obj)
                        .filter(
                            (o) =>
                                o.server_id === undefined &&
                                (o.orderlines.length !== 0 || o.paymentlines.length !== 0) &&
                                // do not count the orders that are already finalized
                                !o.finalized
                        ).length;
                    table_obj.order_count = table.orders + unsynced_orders;
                });
                this.render();
            } catch (error) {
                if (error.message.code < 0) {
                    await this.showPopup('OfflineErrorPopup', {
                        title: 'Offline',
                        body: 'Unable to get orders count',
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

    class TableWidget extends PosComponent {
        mounted() {
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
            const customerCount = this.env.pos.get_customer_count(this.props.table);
            return Math.min(1, Math.max(0, customerCount / this.props.table.seats));
        }
        get orderCount() {
            const table = this.props.table;
            return table.order_count !== undefined
                ? table.order_count
                : this.env.pos
                      .get_table_orders(table)
                      .filter(o => o.orderlines.length !== 0 || o.paymentlines.length !== 0).length;
        }
        get orderCountClass() {
            const notifications = this._getNotifications();
            return {
                'order-count': true,
                'notify-printing': notifications.printing,
                'notify-skipped': notifications.skipped,
            };
        }
        _getNotifications() {
            const orders = this.env.pos.get_table_orders(this.props.table);

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

## File: static\src\js\Screens\OrderManagementScreen\OrderManagementScreen.js

```javascript
odoo.define('pos_restaurant.OrderManagementScreen', function (require) {
    'use strict';

    const OrderManagementScreen = require('point_of_sale.OrderManagementScreen');
    const Registries = require('point_of_sale.Registries');

    const PosResOrderManagementScreen = (OrderManagementScreen) =>
        class extends OrderManagementScreen {
            /**
             * @override
             */
            _setOrder(order) {
                if (this.env.pos.config.module_pos_restaurant) {
                    const currentOrder = this.env.pos.get_order();
                    this.env.pos.set_table(order.table, order);
                    if (currentOrder && currentOrder.uid === order.uid) {
                        this.close();
                    }
                } else {
                    super._setOrder(order);
                }
            }
        };

    Registries.Component.extend(OrderManagementScreen, PosResOrderManagementScreen);

    return OrderManagementScreen;
});

```

## File: static\src\js\Screens\OrderManagementScreen\OrderRow.js

```javascript
odoo.define('pos_restaurant.OrderRow', function (require) {
    'use strict';

    const OrderRow = require('point_of_sale.OrderRow');
    const Registries = require('point_of_sale.Registries');

    const PosResOrderRow = (OrderRow) =>
        class extends OrderRow {
            get table() {
                return this.order.table ? this.order.table.name : '';
            }
        };

    Registries.Component.extend(OrderRow, PosResOrderRow);

    return OrderRow;
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
                if (this.env.pos.get_order().selected_orderline !== line) {
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
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class OrderlineNoteButton extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click', this.onClick);
        }
        get selectedOrderline() {
            return this.env.pos.get_order().get_selected_orderline();
        }
        async onClick() {
            if (!this.selectedOrderline) return;

            const { confirmed, payload: inputNote } = await this.showPopup('TextAreaPopup', {
                startingValue: this.selectedOrderline.get_note(),
                title: this.env._t('Add Note'),
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
            return this.env.pos.config.module_pos_restaurant;
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
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class PrintBillButton extends PosComponent {
        constructor() {
            super(...arguments);
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
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class SplitBillButton extends PosComponent {
        constructor() {
            super(...arguments);
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
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    /**
     * IMPROVEMENT: Perhaps this class is quite complicated for its worth.
     * This is because it needs to listen to changes to the current order.
     * Also, the current order changes when the selectedOrder in pos is changed.
     * After setting new current order, we update the listeners.
     */
    class SubmitOrderButton extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click', this.onClick);
            this._currentOrder = this.env.pos.get_order();
            this._currentOrder.orderlines.on('change', this.render, this);
            this.env.pos.on('change:selectedOrder', this._updateCurrentOrder, this);
        }
        willUnmount() {
            this._currentOrder.orderlines.off('change', null, this);
            this.env.pos.off('change:selectedOrder', null, this);
        }
        async onClick() {
            const order = this.env.pos.get_order();
            if (order.hasChangesToPrint()) {
                const isPrintSuccessful = await order.printChanges();
                if (isPrintSuccessful) {
                    order.saveChanges();
                } else {
                    await this.showPopup('ErrorPopup', {
                        title: 'Printing failed',
                        body: 'Failed in printing the changes in the order',
                    });
                }
            }
        }
        get addedClasses() {
            if (!this._currentOrder) return {};
            const changes = this._currentOrder.hasChangesToPrint();
            const skipped = changes ? false : this._currentOrder.hasSkippedChanges();
            return {
                highlight: changes,
                altlight: skipped,
            };
        }
        _updateCurrentOrder(pos, newSelectedOrder) {
            this._currentOrder.orderlines.off('change', null, this);
            if (newSelectedOrder) {
                this._currentOrder = newSelectedOrder;
                this._currentOrder.orderlines.on('change', this.render, this);
            }
        }
    }
    SubmitOrderButton.template = 'SubmitOrderButton';

    ProductScreen.addControlButton({
        component: SubmitOrderButton,
        condition: function() {
            return this.env.pos.printers.length;
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
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class TableGuestsButton extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click', this.onClick);
        }
        get currentOrder() {
            return this.env.pos.get_order();
        }
        get nGuests() {
            return this.currentOrder ? this.currentOrder.get_customer_count() : 0;
        }
        async onClick() {
            const { confirmed, payload: inputNumber } = await this.showPopup('NumberPopup', {
                startingValue: this.nGuests,
                cheap: true,
                title: this.env._t('Guests ?'),
            });

            if (confirmed) {
                this.env.pos.get_order().set_customer_count(parseInt(inputNumber, 10) || 1);
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
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class TransferOrderButton extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click', this.onClick);
        }
        onClick() {
            this.env.pos.transfer_order_to_different_table();
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
            /**
             * @override
             */
            get nextScreen() {
                if (
                    this.env.pos.config.module_pos_restaurant &&
                    this.env.pos.config.iface_floorplan
                ) {
                    const table = this.env.pos.table;
                    return { name: 'FloorScreen', props: { floor: table ? table.floor : null } };
                } else {
                    return super.nextScreen;
                }
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
    const { useState } = owl.hooks;
    const { useListener } = require('web.custom_hooks');
    const models = require('point_of_sale.models');
    const Registries = require('point_of_sale.Registries');

    class SplitBillScreen extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click-line', this.onClickLine);
            this.splitlines = useState(this._initSplitLines(this.env.pos.get_order()));
            this.newOrderLines = {};
            this.newOrder = new models.Order(
                {},
                {
                    pos: this.env.pos,
                    temporary: true,
                }
            );
            this._isFinal = false;
        }
        mounted() {
            this.env.pos.on('change:selectedOrder', this._resetState, this);
        }
        willUnmount() {
            this.env.pos.off('change:selectedOrder', null, this);
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

            if (this._isFullPayOrder()) {
                this.showScreen('PaymentScreen');
            } else {
                this._setQuantityOnCurrentOrder();

                this.newOrder.set_screen_data({ name: 'PaymentScreen' });

                // for the kitchen printer we assume that everything
                // has already been sent to the kitchen before splitting
                // the bill. So we save all changes both for the old
                // order and for the new one. This is not entirely correct
                // but avoids flooding the kitchen with unnecessary orders.
                // Not sure what to do in this case.

                if (this.newOrder.saveChanges) {
                    this.currentOrder.saveChanges();
                    this.newOrder.saveChanges();
                }

                this.newOrder.set_customer_count(1);
                const newCustomerCount = this.currentOrder.get_customer_count() - 1;
                this.currentOrder.set_customer_count(newCustomerCount || 1);
                this.currentOrder.set_screen_data({ name: 'ProductScreen' });

                this.env.pos.get('orders').add(this.newOrder);
                this.env.pos.set('selectedOrder', this.newOrder);
            }
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

                if(!this.props.disallow) {
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
        _resetState() {
            if (this._isFinal) return;

            for (let id in this.splitlines) {
                delete this.splitlines[id];
            }
            for (let line of this.currentOrder.get_orderlines()) {
                this.splitlines[line.id] = { quantity: 0 };
            }
            this.newOrder.orderlines.reset();
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

    const { useListener } = require('web.custom_hooks');
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    class SplitOrderline extends PosComponent {
        constructor() {
            super(...arguments);
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
        <xpath expr="//div[hasclass('search-bar-portal')]" position="before">
            <BackToFloorButton />
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\multiprint.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

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
                        <div class="multiprint-flex">
                            <t t-esc="change.qty"/>
                            <span t-esc="change.name_wrapped[0]"/>
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
                    <div class="multiprint-flex">
                        <t t-esc="change.qty"/>
                        <span t-esc="change.name_wrapped[0]"/>
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
                    <t t-raw="receipt.header_html" />
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
                <t t-raw="data"/>
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
        <span t-if="hasTable" class="order-button floor-button" t-on-click="backToFloorScreen">
            <i class="fa fa-angle-double-left" role="img" aria-label="Back to floor" title="Back to floor" />
            <span> </span>
            <span t-esc="floor.name" />
            <span> </span>
            <span class="table-name">
                <span>( </span>
                <span t-esc="table.name" />
                <span> )</span>
            </span>
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
                    <span class="button back" t-on-click="confirm">
                        <i class="fa fa-angle-double-left"></i>
                        <span> </span>
                        <span>Back</span>
                    </span>
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
                    <div class="pos-receipt-container">
                        <OrderReceipt order="currentOrder" isBill="true" t-ref="order-receipt"/>
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
            <div t-if="env.pos.config.iface_floorplan" class="col start" name="table">Table</div>
            <div t-if="filter === 'Tipping'" class="col end narrow" name="tip">Tip</div>
        </xpath>
        <xpath expr="//div[hasclass('order-row')]//div[@name='delete']" position="before">
            <div t-if="env.pos.config.iface_floorplan" class="col start" name="table">
                <t t-esc="getTable(order)"></t>
            </div>
            <div t-if="filter === 'Tipping'" class="col end narrow" name="tip">
                <TipCell order="order" />
            </div>
        </xpath>
        <xpath expr="//div[hasclass('buttons')]" position="inside">
            <button class="settle-tips" t-if="filter === 'Tipping'" t-on-click="settleTips">Settle</button>
        </xpath>
    </t>

    <t t-name="TipCell" owl="1">
        <div class="tip-cell" t-on-click.stop="editTip">
            <t t-if="state.isEditing">
                <input type="text" name="tip-amount" t-model="orderUiState.inputTipAmount" t-on-blur="onBlur" t-on-keydown="onKeydown" />
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
                    <span class="button back" t-on-click="showScreen('FloorScreen')">
                        <i class="fa fa-angle-double-left"></i>
                        <span> </span>
                        <span>Back</span>
                    </span>
                    <span class="button" t-if="env.pos.proxy.printer" t-on-click="printTipReceipt()">
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
                                <div class="button" t-on-click="state.inputTipAmount = tip.amount.toFixed(2)">
                                    <div class="percentage">
                                        <t t-esc="tip.percentage"></t>
                                    </div>
                                    <div class="amount">
                                        <t t-esc="env.pos.format_currency(tip.amount)" />
                                    </div>
                                </div>
                            </t>
                        </div>
                        <div class="no-tip" t-on-click="state.inputTipAmount = '0'">
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
            <span class="edit-button" t-on-click.stop="trigger('create-table')">
                <i class="fa fa-plus" role="img" aria-label="Add" title="Add"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="trigger('duplicate-table')">
                <i class="fa fa-files-o" role="img" aria-label="Duplicate" title="Duplicate"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="trigger('rename-table')">
                <i class="fa fa-font" role="img" aria-label="Rename" title="Rename"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="trigger('change-seats-num')">
                <i class="fa fa-user" role="img" aria-label="Seats" title="Seats"></i>
            </span>
            <span class="edit-button" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="trigger('change-shape')">
                <span t-if="!props.selectedTable or props.selectedTable.shape == 'square'" class="button-option square">
                    <i class="fa fa-square-o" role="img" aria-label="Square Shape" title="Square Shape"></i>
                </span>
                <span t-else="" class="button-option round">
                    <i class="fa fa-circle-o" role="img" aria-label="Round Shape" title="Round Shape"></i>
                </span>
            </span>
            <span class="edit-button" t-on-click.stop="state.isColorPicker = !state.isColorPicker">
                <i class="fa fa-tint" role="img" aria-label="Tint" title="Tint"></i>
            </span>
            <div t-if="state.isColorPicker and props.selectedTable" class="color-picker fg-picker">
                <div  class="close-picker" title="Close" role="img" aria-label="Close" t-on-click.stop="state.isColorPicker = false">
                    <i class="fa fa-times" />
                </div>
                <span class="color tl"  style="background-color:#EB6D6D" role="img" aria-label="Red" title="Red" t-on-click.stop="trigger('set-table-color', '#EB6D6D')" />
                <span class="color"     style="background-color:#35D374" role="img" aria-label="Green" title="Green" t-on-click.stop="trigger('set-table-color', '#35D374')" />
                <span class="color tr"  style="background-color:#6C6DEC" role="img" aria-label="Blue" title="Blue" t-on-click.stop="trigger('set-table-color', '#6C6DEC')" />
                <span class="color"     style="background-color:#EBBF6D" role="img" aria-label="Orange" title="Orange" t-on-click.stop="trigger('set-table-color', '#EBBF6D')" />
                <span class="color"     style="background-color:#EBEC6D" role="img" aria-label="Yellow" title="Yellow" t-on-click.stop="trigger('set-table-color', '#EBEC6D')" />
                <span class="color"     style="background-color:#AC6DAD" role="img" aria-label="Purple" title="Purple" t-on-click.stop="trigger('set-table-color', '#AC6DAD')" />
                <span class="color bl"  style="background-color:#6C6D6D" role="img" aria-label="Grey" title="Grey" t-on-click.stop="trigger('set-table-color', '#6C6D6D')" />
                <span class="color"     style="background-color:#ACADAD" role="img" aria-label="Light grey" title="Light grey" t-on-click.stop="trigger('set-table-color', '#ACADAD')" />
                <span class="color br"  style="background-color:#4ED2BE" role="img" aria-label="Turquoise" title="Turquoise" t-on-click.stop="trigger('set-table-color', '#4ED2BE')" />
            </div>
            <div t-if="state.isColorPicker and !props.selectedTable" class="color-picker bg-picker">
                <div  class="close-picker" title="Close" role="img" aria-label="Close" t-on-click.stop="state.isColorPicker = false">
                    <i class="fa fa-times" />
                </div>
                <span class="color tl"  style="background-color:rgb(244, 149, 149)" role="img" aria-label="Red" title="Red" t-on-click.stop="trigger('set-floor-color', 'rgb(244, 149, 149)')" />
                <span class="color"     style="background-color:rgb(130, 233, 171)" role="img" aria-label="Green" title="Green" t-on-click.stop="trigger('set-floor-color', 'rgb(130, 233, 171)')" />
                <span class="color tr"  style="background-color:rgb(136, 137, 242)" role="img" aria-label="Blue" title="Blue" t-on-click.stop="trigger('set-floor-color', 'rgb(136, 137, 242)')" />
                <span class="color"     style="background-color:rgb(255, 214, 136)" role="img" aria-label="Orange" title="Orange" t-on-click.stop="trigger('set-floor-color', 'rgb(255, 214, 136)')" />
                <span class="color"     style="background-color:rgb(254, 255, 154)" role="img" aria-label="Yellow" title="Yellow" t-on-click.stop="trigger('set-floor-color', 'rgb(254, 255, 154)')" />
                <span class="color"     style="background-color:rgb(209, 171, 210)" role="img" aria-label="Purple" title="Purple" t-on-click.stop="trigger('set-floor-color', 'rgb(209, 171, 210)')" />
                <span class="color bl"  style="background-color:rgb(75, 75, 75)"    role="img" aria-label="Grey" title="Grey" t-on-click.stop="trigger('set-floor-color', 'rgb(75, 75, 75)')" />
                <span class="color"     style="background-color:rgb(210, 210, 210)" role="img" aria-label="Light grey" title="Light grey" t-on-click.stop="trigger('set-floor-color', 'rgb(210, 210, 210)')" />
                <span class="color br"  style="background-color:rgb(127, 221, 236)" role="img" aria-label="Turquoise" title="Turquoise" t-on-click.stop="trigger('set-floor-color', 'rgb(127, 221, 236)')" />
            </div>
            <span class="edit-button trash" t-att-class="{ disabled: !props.selectedTable }" t-on-click.stop="trigger('delete-table')">
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
                            <span class="button button-floor" t-att-class="{ active: floor.id === state.selectedFloorId }" t-on-click="selectFloor(floor)">
                                <t t-esc="floor.name" />
                            </span>
                        </t>
                    </div>
                </t>
                <div class="floor-map" t-on-click="trigger('deselect-table')" t-ref="floor-map-ref">
                    <div t-if="isFloorEmpty" class="empty-floor">
                        <span>This floor has no tables yet, use the </span>
                        <i class="fa fa-plus" role="img" aria-label="Add button" title="Add button"></i>
                        <span> button in the editing toolbar to create new tables.</span>
                    </div>
                    <div t-else="" class="tables">
                        <t t-foreach="activeTables" t-as="table" t-key="table.id">
                            <TableWidget t-if="table.id !== state.selectedTableId" table="table" />
                            <EditableTable t-else="" table="table" />
                        </t>
                    </div>
                    <span t-if="env.pos.user.role == 'manager'" class="edit-button editing" t-att-class="{ active: state.isEditMode }" t-on-click.stop="toggleEditMode"
                          t-attf-style="top:{{state.floorMapScrollTop}}px;">
                        <i class="fa fa-pencil" role="img" aria-label="Edit" title="Edit"></i>
                    </span>
                    <EditBar t-if="state.isEditMode" selectedTable="selectedTable" floorMapScrollTop="state.floorMapScrollTop"/>
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
        <div t-if="!props.isSelected" class="table" t-on-click.stop="trigger('select-table', props.table)">
            <span class="table-cover" t-att-class="{ full: fill >= 1 }"></span>
            <span t-att-class="orderCountClass" t-att-hidden="orderCount === 0">
                <t t-esc="orderCount" />
            </span>
            <span class="label">
                <t t-esc="props.table.name" />
            </span>
            <span class="table-seats">
                <t t-esc="props.table.seats" />
            </span>
        </div>
    </t>

</templates>

```

## File: static\src\xml\Screens\OrderManagementScreen\OrderList.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="OrderList" t-inherit="point_of_sale.OrderList" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('order-row')]//div[hasclass('customer')]" position="after">
            <div t-if="env.pos.config.module_pos_restaurant" class="header table">
                Table
            </div>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\Screens\OrderManagementScreen\OrderRow.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="OrderRow" t-inherit="point_of_sale.OrderRow" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('order-row')]//div[hasclass('customer')]" position="after">
            <div t-if="env.pos.config.module_pos_restaurant" class="item table">
                <t t-esc="table" />
            </div>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\Screens\PaymentScreen\PaymentScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="PaymentScreen" t-inherit="point_of_sale.PaymentScreen" t-inherit-mode="extension" owl="1">
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

## File: static\src\xml\Screens\PaymentScreen\PaymentScreenElectronicPayment.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="PaymentScreenElectronicPayment" t-inherit="point_of_sale.PaymentScreenElectronicPayment" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('send_payment_reversal')]/.." position="replace">
            <t t-if="props.line.canBeAdjusted() &amp;&amp; props.line.order.get_total_paid() &lt; props.line.order.get_total_with_tax()">
                <div class="button send_adjust_amount" title="Adjust Amount" t-on-click="trigger('send-payment-adjust', props.line)">
                    Adjust Amount
                </div>
            </t>
            <t t-elif="props.line.can_be_reversed">
                <div class="button send_payment_reversal" title="Reverse Payment" t-on-click="trigger('send-payment-reverse', props.line)">
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

## File: static\src\xml\Screens\ProductScreen\ControlButtons\OrderlineNoteButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="OrderlineNoteButton" owl="1">
        <div class="control-button">
            <i class="fa fa-tag" />
            <span> </span>
            <span>Note</span>
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
        <span class="control-button" t-att-class="addedClasses">
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
                <div class="main">
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
            <div id="tip_product" position="after">
                <div attrs="{'invisible': ['|', ('module_pos_restaurant', '=', False), ('iface_tipproduct', '=', False)]}">
                    <field name="set_tip_after_payment" class="oe_inline"/>
                    <label class="font-weight-normal" for="set_tip_after_payment" string="Add tip after payment (North America specific)"/>
                    <span class="fa fa-lg fa-cutlery" title="For bars and restaurants" role="img" aria-label="For bars and restaurants"/>
                </div>
            </div>
            <div id="category_reference" position="before">
                <div class="col-12 col-lg-6 o_setting_box"
                     id="is_table_management"
                     attrs="{'invisible': [('module_pos_restaurant', '=', False)]}">
                    <div class="o_setting_left_pane">
                        <field name="is_table_management" attrs="{'readonly': [('has_active_session','=', True)]}"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="is_table_management"/>
                        <span class="fa fa-lg fa-cutlery" title="For bars and restaurants" role="img" aria-label="For bars and restaurants"/>
                        <div class="text-muted">
                            Design floors and assign orders to tables
                        </div>
                        <div class="content-group" attrs="{'invisible': [('is_table_management','=',False)]}">
                            <div class="mt16">
                                <label string="Floors" for="floor_ids" class="o_light_label"/>
                                <field name="floor_ids" widget="many2many_tags" attrs="{'readonly': [('has_active_session','=', True)]}" domain="['|', ('pos_config_id', '=', False), ('pos_config_id', '=', active_id)]"/>
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
                            Add notes on order lines
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
            <script type="text/javascript" src="/pos_restaurant/static/src/js/floors.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/notes.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/payment.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Resizeable.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ProductScreen/ControlButtons/OrderlineNoteButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ProductScreen/ControlButtons/TableGuestsButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ProductScreen/ControlButtons/PrintBillButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ProductScreen/ControlButtons/SubmitOrderButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ProductScreen/ControlButtons/SplitBillButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ProductScreen/ControlButtons/TransferOrderButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ProductScreen/Orderline.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/BillScreen.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/SplitBillScreen/SplitBillScreen.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/SplitBillScreen/SplitOrderline.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/FloorScreen/FloorScreen.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/FloorScreen/EditBar.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/FloorScreen/TableWidget.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/FloorScreen/EditableTable.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/OrderManagementScreen/OrderManagementScreen.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/OrderManagementScreen/OrderRow.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/TicketScreen.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/ChromeWidgets/BackToFloorButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/ChromeWidgets/TicketButton.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Chrome.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/ReceiptScreen/ReceiptScreen.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/PaymentScreen.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/src/js/Screens/TipScreen.js"></script>
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
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/ProductScreenTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/ChromeTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/FloorScreenTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/SplitBillScreenTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/TextAreaPopupTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/TextInputPopupTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/BillScreenTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/helpers/TipScreenTourMethods.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/pos_restaurant.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/SplitBillScreen.tour.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/ControlButtons.tour.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/FloorScreen.tour.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/OrderManagementScreen.tour.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/TicketScreen.tour.js"></script>
            <script type="text/javascript" src="/pos_restaurant/static/tests/tours/TipScreen.tour.js"></script>
        </xpath>
    </template>

    <template id="pos_restaurant.qunit_suite_tests" inherit_id="point_of_sale.qunit_suite_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/pos_restaurant/static/tests/unit/test_FloorScreen.js"></script>
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

