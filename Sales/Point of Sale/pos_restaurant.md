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
        'data/pos_restaurant_data.xml',
    ],
    'demo': [
        'data/pos_restaurant_demo.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_restaurant/static/src/**/*',
            ('after', 'point_of_sale/static/src/scss/pos.scss', 'pos_restaurant/static/src/scss/restaurant.scss'),
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

## File: data\pos_restaurant_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="pos_config_main_restaurant" model="pos.config">
            <field name="name">Restaurant</field>
            <field name="module_pos_restaurant">True</field>
            <field name="iface_splitbill">True</field>
            <field name="iface_printbill">False</field>
            <field name="iface_orderline_notes">True</field>
            <field name="iface_tipproduct">False</field>
            <field name="start_category">False</field>
            <field name="limit_categories">False</field>
        </record>

        <function model="pos.config" name="_setup_main_restaurant_defaults">
            <value eval="[ref('pos_config_main_restaurant')]"/>
        </function>
    </data>
</odoo>

```

## File: data\pos_restaurant_demo.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="drinks" model="pos.category">
            <field name="name">Drinks</field>
            <field name="image_128" type="base64" file="pos_restaurant/static/img/drink_category.png" />
        </record>

        <record id="product_category_pos_food" model="product.category">
            <field name="parent_id" ref="point_of_sale.product_category_pos"/>
            <field name="name">Food</field>
        </record>

        <record id="food" model="pos.category">
            <field name="name">Food</field>
            <field name="image_128" type="base64" file="pos_restaurant/static/img/food_category.png" />
        </record>

        <!-- Food -->
        <record id="pos_food_margherita" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Margherita</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-ma.jpg"/>
        </record>
        <record id="pos_food_funghi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Funghi</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-fu.jpg"/>
        </record>
        <record id="pos_food_vege" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Vegetarian</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-ve.jpg"/>
        </record>
        <record id="pos_food_bolo" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.5</field>
            <field name="name">Pasta Bolognese</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta.jpg"/>
        </record>
        <record id="pos_food_4formaggi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">5.5</field>
            <field name="name">Pasta 4 formaggi </field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta-4f.jpg"/>
        </record>
        <record id="pos_food_bacon" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.5</field>
            <field name="name">Bacon Burger</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-burger.jpg"/>
        </record>
        <record id="pos_food_cheeseburger" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Cheese Burger</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-cheeseburger.jpg"/>
        </record>
        <record id="pos_food_chicken" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.0</field>
            <field name="name">Chicken Curry Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-sandwich.jpg"/>
        </record>
        <record id="pos_food_tuna" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.0</field>
            <field name="name">Spicy Tuna Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-tuna.jpg"/>
        </record>
        <record id="pos_food_mozza" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.9</field>
            <field name="name">Mozzarella Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-mozza.jpg"/>
        </record>
        <record id="pos_food_club" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.4</field>
            <field name="name">Club Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-club.jpg"/>
        </record>
        <record id="pos_food_maki" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">12.0</field>
            <field name="name">Lunch Maki 18pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-maki.jpg"/>
        </record>
        <record id="pos_food_salmon" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">13.80</field>
            <field name="name">Lunch Salmon 20pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon.jpg"/>
        </record>
        <record id="pos_food_temaki" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">14.0</field>
            <field name="name">Lunch Temaki mix 3pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-temaki.jpg"/>
        </record>
        <record id="pos_food_chirashi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">9.25</field>
            <field name="name">Salmon and Avocado</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon-avocado.jpg"/>
        </record>

        <!-- Drinks -->
        <record id="coke" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Coca-Cola</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-coke.jpg"/>
        </record>

        <record id="water" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Water</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-water.jpg"/>
        </record>

        <record id="minute_maid" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Minute Maid</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-minute_maid.jpg"/>
        </record>

        <record id="espresso" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.70</field>
            <field name="name">Espresso</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-espresso.jpg"/>
        </record>

        <record id="green_tea" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.70</field>
            <field name="name">Green Tea</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-green_tea.jpg"/>
        </record>

        <record id="milkshake_banana" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.60</field>
            <field name="name">Milkshake Banana</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-milkshake_banana.jpg"/>
        </record>

        <record id="ice_tea" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Ice Tea</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-ice_tea.jpg"/>
        </record>

        <record id="schweppes" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Schweppes</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-schweppes.jpg"/>
        </record>

        <record id="fanta" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Fanta</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-fanta.jpg"/>
        </record>

        <!-- Combo -->
        <record id="cheeseburger_combo_line" model="pos.combo.line">
            <field name="product_id" ref="pos_food_cheeseburger"/>
            <field name="combo_price">0</field>
        </record>
        <record id="bacon_burger_combo_line" model="pos.combo.line">
            <field name="product_id" ref="pos_food_bacon"/>
            <field name="combo_price">0</field>
        </record>
        <record id="burger_combo" model="pos.combo">
            <field name="name">Burgers Choice</field>
            <field name="combo_line_ids" eval="[(6, 0, [ref('cheeseburger_combo_line'), ref('bacon_burger_combo_line')])]"/>
        </record>

        <record id="coke_combo_line" model="pos.combo.line">
            <field name="product_id" ref="coke"/>
            <field name="combo_price">0</field>
        </record>
        <record id="water_combo_line" model="pos.combo.line">
            <field name="product_id" ref="water"/>
            <field name="combo_price">0</field>
        </record>
        <record id="maid_combo_line" model="pos.combo.line">
            <field name="product_id" ref="minute_maid"/>
            <field name="combo_price">0</field>
        </record>
        <record id="milkshake_combo_line" model="pos.combo.line">
            <field name="product_id" ref="milkshake_banana"/>
            <field name="combo_price">2</field>
        </record>
        <record id="drink_combo" model="pos.combo">
            <field name="name">Drinks choice</field>
            <field name="combo_line_ids" eval="[(6, 0, [ref('coke_combo_line'), ref('water_combo_line'), ref('maid_combo_line'), ref('milkshake_combo_line')])]"/>
        </record>

        <record id="burger_drink_combo" model="product.product">
          <field name="available_in_pos">True</field>
          <field name="list_price">10</field>
          <field name="name">Burger Menu Combo</field>
          <field name="type">combo</field>
          <field name="uom_id" ref="uom.product_uom_unit"/>
          <field name="uom_po_id" ref="uom.product_uom_unit"/>
          <field name="image_1920" type="base64" file="pos_restaurant/static/img/combo-hamb.jpg"/>
          <field name="combo_ids" eval="[(6, 0, [ref('drink_combo'), ref('burger_combo')])]"/>
          <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
          <field name="taxes_id" eval="[(5,)]"/>  <!-- no taxes -->
        </record>

        <function model="restaurant.floor" name="unlink">
            <value model="restaurant.floor" eval="obj().search([
                    ('pos_config_ids', 'in', ref('pos_config_main_restaurant')),
                ]).id"/>
        </function>

        <!-- Pos Config -->
        <record model="pos.config" id="pos_config_main_restaurant">
            <field name="iface_printbill">True</field>
            <field name="limit_categories">True</field>
            <field name="iface_available_categ_ids"
                eval="[(6, 0, [ref('drinks'), ref('food')])]" />
        </record>

        <!-- Closed Sessions -->
        <!-- forcecreate is set to false in order to not create record when updating the db -->

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                    'xml_id': 'pos_restaurant.payment_method',
                    'record': obj().env.ref('pos_restaurant.pos_config_main_restaurant')._get_payment_method('bank'),
                    'noupdate': True,
                }]" />
        </function>

        <!-- Closed Session 3 -->

        <record id="pos_closed_session_3" model="pos.session" forcecreate="False">
            <field name="config_id" ref="pos_config_main_restaurant" />
            <field name="user_id" ref="base.user_admin" />
            <field name="start_at" eval="(DateTime.today() + relativedelta(days=-1)).strftime('%Y-%m-%d %H:%M:%S')" />
            <field name="stop_at"
                eval="(DateTime.today() + relativedelta(days=-1, hours=1)).strftime('%Y-%m-%d %H:%M:%S')" />
        </record>

        <record id="pos_closed_order_3_1" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_closed_session_3" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">ClosedDemo/0005</field>
            <field name="state">paid</field>
            <field name="amount_total">14.0</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">14.0</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00000-003-1001</field>
        </record>

        <record id="pos_closed_orderline_3_1_1" model="pos.order.line" forcecreate="False">
            <field name="name">Closed Orderline 3.1.1</field>
            <field name="product_id" ref="pos_food_margherita" />
            <field name="price_subtotal">7.0</field>
            <field name="price_subtotal_incl">7.0</field>
            <field name="price_unit">7.0</field>
            <field name="order_id" ref="pos_closed_order_3_1" />
            <field name="full_product_name">Margherita</field>
        </record>

        <record id="pos_closed_orderline_3_1_2" model="pos.order.line" forcecreate="False">
            <field name="name">Closed Orderline 3.1.2</field>
            <field name="product_id" ref="pos_food_funghi" />
            <field name="price_subtotal">7.0</field>
            <field name="price_subtotal_incl">7.0</field>
            <field name="price_unit">7.0</field>
            <field name="order_id" ref="pos_closed_order_3_1" />
            <field name="full_product_name">Funghi</field>
        </record>

        <record id="pos_payment_1" model="pos.payment" forcecreate="False">
            <field name="payment_method_id" ref="pos_restaurant.payment_method" />
            <field name="pos_order_id" ref="pos_closed_order_3_1" />
            <field name="amount">14.0</field>
        </record>

        <record id="pos_closed_order_3_2" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_closed_session_3" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">ClosedDemo/0006</field>
            <field name="state">paid</field>
            <field name="amount_total">7.0</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">7.0</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00000-003-1002</field>
        </record>

        <record id="pos_closed_orderline_3_2_1" model="pos.order.line" forcecreate="False">
            <field name="name">Closed Orderline 3.2.1</field>
            <field name="product_id" ref="pos_food_vege" />
            <field name="price_subtotal">7.0</field>
            <field name="price_subtotal_incl">7.0</field>
            <field name="price_unit">7.0</field>
            <field name="order_id" ref="pos_closed_order_3_2" />
            <field name="full_product_name">Vegetarian</field>
        </record>

        <record id="pos_payment_2" model="pos.payment" forcecreate="False">
            <field name="payment_method_id" ref="pos_restaurant.payment_method" />
            <field name="pos_order_id" ref="pos_closed_order_3_2" />
            <field name="amount">7.0</field>
        </record>

        <function model="pos.session" name="action_pos_session_closing_control"
            eval="[[ref('pos_closed_session_3')]]" />

        <!-- Closed Session 4 -->

        <record id="pos_closed_session_4" model="pos.session" forcecreate="False">
            <field name="config_id" ref="pos_config_main_restaurant" />
            <field name="user_id" ref="base.user_admin" />
            <field name="start_at" eval="(DateTime.today() + relativedelta(days=-1)).strftime('%Y-%m-%d %H:%M:%S')" />
            <field name="stop_at"
                eval="(DateTime.today() + relativedelta(days=-1, hours=1)).strftime('%Y-%m-%d %H:%M:%S')" />
        </record>

        <record id="pos_closed_order_4_1" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_closed_session_4" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">ClosedDemo/0007</field>
            <field name="state">paid</field>
            <field name="amount_total">6.7</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">6.7</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00000-004-1001</field>
        </record>

        <record id="pos_closed_orderline_4_1_1" model="pos.order.line" forcecreate="False">
            <field name="name">Closed Orderline 4.1.1</field>
            <field name="product_id" ref="water" />
            <field name="price_subtotal">2.20</field>
            <field name="price_subtotal_incl">2.20</field>
            <field name="price_unit">2.20</field>
            <field name="order_id" ref="pos_closed_order_4_1" />
            <field name="full_product_name">Water</field>
        </record>

        <record id="pos_closed_orderline_4_1_2" model="pos.order.line" forcecreate="False">
            <field name="name">Closed Orderline 4.1.2</field>
            <field name="product_id" ref="pos_food_bolo" />
            <field name="price_subtotal">4.5</field>
            <field name="price_subtotal_incl">4.5</field>
            <field name="price_unit">4.5</field>
            <field name="order_id" ref="pos_closed_order_4_1" />
            <field name="full_product_name">Pasta Bolognese</field>
        </record>

        <record id="pos_payment_3" model="pos.payment" forcecreate="False">
            <field name="payment_method_id" ref="pos_restaurant.payment_method" />
            <field name="pos_order_id" ref="pos_closed_order_4_1" />
            <field name="amount">6.7</field>
        </record>

        <record id="pos_closed_order_4_2" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_closed_session_4" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">ClosedDemo/0008</field>
            <field name="state">paid</field>
            <field name="amount_total">28.0</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">28.0</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00000-004-1002</field>
        </record>

        <record id="pos_closed_orderline_4_2_1" model="pos.order.line" forcecreate="False">
            <field name="name">Closed Orderline 4.2.1</field>
            <field name="product_id" ref="pos_food_cheeseburger" />
            <field name="price_subtotal">28.0</field>
            <field name="price_subtotal_incl">28.0</field>
            <field name="price_unit">7.0</field>
            <field name="qty">4</field>
            <field name="order_id" ref="pos_closed_order_4_2" />
            <field name="full_product_name">Cheese Burger</field>
        </record>

        <record id="pos_payment_4" model="pos.payment" forcecreate="False">
            <field name="payment_method_id" ref="pos_restaurant.payment_method" />
            <field name="pos_order_id" ref="pos_closed_order_4_2" />
            <field name="amount">28.0</field>
        </record>

        <function model="pos.session" name="action_pos_session_closing_control"
            eval="[[ref('pos_closed_session_4')]]" />

        <!-- Floors: Main Floor -->
        <record id="floor_main" model="restaurant.floor">
            <field name="name">Main Floor</field>
            <field name="background_color">rgb(249,250,251)</field>
            <field name="pos_config_ids" eval="[(6, 0, [ref('pos_restaurant.pos_config_main_restaurant')])]" />
        </record>

        <record id="table_01" model="restaurant.table">
            <field name="name">1</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">50</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_02" model="restaurant.table">
            <field name="name">2</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">212</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_03" model="restaurant.table">
            <field name="name">3</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">374</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_04" model="restaurant.table">
            <field name="name">4</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">536</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_05" model="restaurant.table">
            <field name="name">5</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">698</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_06" model="restaurant.table">
            <field name="name">6</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">860</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_07" model="restaurant.table">
            <field name="name">7</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">50</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_08" model="restaurant.table">
            <field name="name">8</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">212</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_09" model="restaurant.table">
            <field name="name">9</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">6</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">698</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_10" model="restaurant.table">
            <field name="name">10</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">6</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">100</field>
            <field name="height">100</field>
            <field name="position_h">860</field>
            <field name="position_v">280</field>
        </record>

        <record id="table_11" model="restaurant.table">
            <field name="name">11</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">6</field>
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
            <field name="pos_config_ids" eval="[(6, 0, [ref('pos_restaurant.pos_config_main_restaurant')])]" />
        </record>

        <!-- Patio: Left table row -->

        <record id="table_21" model="restaurant.table">
            <field name="name">1</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">100</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_22" model="restaurant.table">
            <field name="name">2</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">100</field>
            <field name="position_v">166</field>
        </record>

        <record id="table_23" model="restaurant.table">
            <field name="name">3</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">100</field>
            <field name="position_v">283</field>
        </record>

        <record id="table_24" model="restaurant.table">
            <field name="name">4</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">100</field>
            <field name="position_v">400</field>
        </record>

        <!-- Patio: Right table row -->

        <record id="table_25" model="restaurant.table">
            <field name="name">5</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">800</field>
            <field name="position_v">50</field>
        </record>

        <record id="table_26" model="restaurant.table">
            <field name="name">6</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">800</field>
            <field name="position_v">166</field>
        </record>

        <record id="table_27" model="restaurant.table">
            <field name="name">7</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">800</field>
            <field name="position_v">283</field>
        </record>

        <record id="table_28" model="restaurant.table">
            <field name="name">8</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">2</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">85</field>
            <field name="position_h">800</field>
            <field name="position_v">400</field>
        </record>

        <!-- Patio: Center table block -->

        <record id="table_29" model="restaurant.table">
            <field name="name">9</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">330</field>
            <field name="position_v">100</field>
        </record>

        <record id="table_29" model="restaurant.table">
            <field name="name">9</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">330</field>
            <field name="position_v">100</field>
        </record>

        <record id="table_30" model="restaurant.table">
            <field name="name">10</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">560</field>
            <field name="position_v">100</field>
        </record>

        <record id="table_31" model="restaurant.table">
            <field name="name">11</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">330</field>
            <field name="position_v">315</field>
        </record>

        <record id="table_32" model="restaurant.table">
            <field name="name">12</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">560</field>
            <field name="position_v">315</field>
        </record>

        <function model="pos.config" name="add_cash_payment_method" />

        <!-- Open Session -->

        <record id="pos_open_session_2" model="pos.session" forcecreate="False">
            <field name="config_id" ref="pos_config_main_restaurant" />
            <field name="user_id" ref="base.user_admin" />
        </record>

        <record id="pos_open_order_2" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_open_session_2" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">Restaurant/00001</field>
            <field name="state">draft</field>
            <field name="amount_total">22.90</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">0.0</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00002-001-0000</field>
            <field name="partner_id" ref="base.res_partner_1" />
            <field name="table_id" ref="table_01" />
            <field name="customer_count">8</field>
        </record>

        <record id="pos_orderline_2" model="pos.order.line" forcecreate="False">
            <field name="name">Orderline 2</field>
            <field name="product_id" ref="coke" />
            <field name="price_subtotal">4.40</field>
            <field name="price_subtotal_incl">4.40</field>
            <field name="price_unit">2.20</field>
            <field name="qty">2</field>
            <field name="order_id" ref="pos_open_order_2" />
            <field name="full_product_name">Coca-Cola</field>
            <field name="uuid">00000000-0000-4000-000000000000</field>
        </record>

        <record id="pos_orderline_3" model="pos.order.line" forcecreate="False">
            <field name="name">Orderline 3</field>
            <field name="product_id" ref="pos_food_chirashi" />
            <field name="price_subtotal">18.5</field>
            <field name="price_subtotal_incl">18.5</field>
            <field name="price_unit">9.25</field>
            <field name="qty">2</field>
            <field name="order_id" ref="pos_open_order_2" />
            <field name="full_product_name">Salmon and Avocado</field>
            <field name="uuid">00000000-0000-4000-000000000001</field>
        </record>

        <record id="pos_open_order_3" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_open_session_2" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">Restaurant/00002</field>
            <field name="state">draft</field>
            <field name="amount_total">21.8</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">0.0</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00002-002-0000</field>
            <field name="partner_id" ref="base.res_partner_2" />
            <field name="table_id" ref="table_02" />
            <field name="customer_count">3</field>
        </record>

        <record id="pos_orderline_4" model="pos.order.line" forcecreate="False">
            <field name="name">Orderline 4</field>
            <field name="product_id" ref="pos_food_temaki" />
            <field name="price_subtotal">14.0</field>
            <field name="price_subtotal_incl">14.0</field>
            <field name="price_unit">14.0</field>
            <field name="qty">1</field>
            <field name="order_id" ref="pos_open_order_3" />
            <field name="full_product_name">Lunch Temaki mix 3pc</field>
            <field name="uuid">00000000-0000-4000-000000000002</field>
        </record>

        <record id="pos_orderline_5" model="pos.order.line" forcecreate="False">
            <field name="name">Orderline 5</field>
            <field name="product_id" ref="pos_food_mozza" />
            <field name="price_subtotal">7.8</field>
            <field name="price_subtotal_incl">7.8</field>
            <field name="price_unit">3.9</field>
            <field name="qty">2</field>
            <field name="order_id" ref="pos_open_order_3" />
            <field name="full_product_name">Mozzarella Sandwich</field>
            <field name="uuid">00000000-0000-4000-000000000003</field>
        </record>

        <record id="pos_open_order_4" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_open_session_2" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">Restaurant/00003</field>
            <field name="state">draft</field>
            <field name="amount_total">10.5</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">0.0</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00002-003-0000</field>
            <field name="partner_id" ref="base.res_partner_4" />
            <field name="table_id" ref="table_04" />
            <field name="customer_count">5</field>
        </record>

        <record id="pos_orderline_6" model="pos.order.line" forcecreate="False">
            <field name="name">Orderline 6</field>
            <field name="product_id" ref="pos_food_chicken" />
            <field name="price_subtotal">3.0</field>
            <field name="price_subtotal_incl">3.0</field>
            <field name="price_unit">3.0</field>
            <field name="qty">1</field>
            <field name="order_id" ref="pos_open_order_4" />
            <field name="full_product_name">Chicken Curry Sandwich</field>
            <field name="uuid">00000000-0000-4000-000000000004</field>
        </record>

        <record id="pos_orderline_7" model="pos.order.line" forcecreate="False">
            <field name="name">Orderline 7</field>
            <field name="product_id" ref="pos_food_bacon" />
            <field name="price_subtotal">7.5</field>
            <field name="price_subtotal_incl">7.5</field>
            <field name="price_unit">7.5</field>
            <field name="qty">1</field>
            <field name="order_id" ref="pos_open_order_4" />
            <field name="full_product_name">Bacon Burger</field>
            <field name="uuid">00000000-0000-4000-000000000005</field>
        </record>

        <record id="pos_open_order_5" model="pos.order" forcecreate="False">
            <field name="session_id" ref="pos_open_session_2" />
            <field name="company_id" ref="base.main_company" />
            <field name="name">Restaurant/00004</field>
            <field name="state">draft</field>
            <field name="amount_total">5.5</field>
            <field name="amount_tax">0.0</field>
            <field name="amount_paid">0.0</field>
            <field name="amount_return">0.0</field>
            <field name="pos_reference">Order 00002-004-0000</field>
            <field name="partner_id" ref="base.res_partner_10" />
            <field name="table_id" ref="table_06" />
            <field name="customer_count">1</field>
        </record>

        <record id="pos_orderline_8" model="pos.order.line" forcecreate="False">
            <field name="name">Orderline 8</field>
            <field name="product_id" ref="pos_food_4formaggi" />
            <field name="price_subtotal">5.5</field>
            <field name="price_subtotal_incl">5.5</field>
            <field name="price_unit">5.5</field>
            <field name="qty">1</field>
            <field name="order_id" ref="pos_open_order_5" />
            <field name="full_product_name">Pizza 4 Formaggi</field>
        </record>

        <function model="pos.session" name="_set_last_order_preparation_change"
            eval="[[ref('pos_open_order_2'), ref('pos_open_order_3'), ref('pos_open_order_4')]]"/>
    </data>
</odoo>

```

## File: data\pos_restaurant_onboarding.xml

```xml
<odoo noupdate="1">

    <record id="drinks" model="pos.category">
        <field name="name">Drinks</field>
        <field name="image_128" type="base64" file="pos_restaurant/static/img/drink_category.png" />
    </record>

    <record id="product_category_pos_food" model="product.category">
        <field name="parent_id" ref="point_of_sale.product_category_pos"/>
        <field name="name">Food</field>
    </record>

    <record id="food" model="pos.category">
        <field name="name">Food</field>
        <field name="image_128" type="base64" file="pos_restaurant/static/img/food_category.png" />
    </record>

    <!-- Food -->
    <record id="pos_food_margherita" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">7.0</field>
        <field name="name">Margherita</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-ma.jpg"/>
    </record>
    <record id="pos_food_funghi" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">7.0</field>
        <field name="name">Funghi</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-fu.jpg"/>
    </record>
    <record id="pos_food_vege" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">7.0</field>
        <field name="name">Vegetarian</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-ve.jpg"/>
    </record>
    <record id="pos_food_bolo" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">4.5</field>
        <field name="name">Pasta Bolognese</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta.jpg"/>
    </record>
    <record id="pos_food_4formaggi" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">5.5</field>
        <field name="name">Pasta 4 formaggi </field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta-4f.jpg"/>
    </record>
    <record id="pos_food_bacon" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">7.5</field>
        <field name="name">Bacon Burger</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-burger.jpg"/>
    </record>
    <record id="pos_food_cheeseburger" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">7.0</field>
        <field name="name">Cheese Burger</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-cheeseburger.jpg"/>
    </record>
    <record id="pos_food_chicken" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">3.0</field>
        <field name="name">Chicken Curry Sandwich</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-sandwich.jpg"/>
    </record>
    <record id="pos_food_tuna" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">3.0</field>
        <field name="name">Spicy Tuna Sandwich</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-tuna.jpg"/>
    </record>
    <record id="pos_food_mozza" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">3.9</field>
        <field name="name">Mozzarella Sandwich</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-mozza.jpg"/>
    </record>
    <record id="pos_food_club" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">3.4</field>
        <field name="name">Club Sandwich</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-club.jpg"/>
    </record>
    <record id="pos_food_maki" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">12.0</field>
        <field name="name">Lunch Maki 18pc</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-maki.jpg"/>
    </record>
    <record id="pos_food_salmon" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">13.80</field>
        <field name="name">Lunch Salmon 20pc</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon.jpg"/>
    </record>
    <record id="pos_food_temaki" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">14.0</field>
        <field name="name">Lunch Temaki mix 3pc</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-temaki.jpg"/>
    </record>
    <record id="pos_food_chirashi" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">9.25</field>
        <field name="name">Salmon and Avocado</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
        <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon-avocado.jpg"/>
    </record>

    <!-- Drinks -->
    <record id="coke" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">2.20</field>
        <field name="name">Coca-Cola</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="categ_id" ref="point_of_sale.product_category_pos"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-coke.jpg"/>
    </record>

    <record id="water" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">2.20</field>
        <field name="name">Water</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="categ_id" ref="point_of_sale.product_category_pos"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-water.jpg"/>
    </record>

    <record id="minute_maid" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">2.20</field>
        <field name="name">Minute Maid</field>
        <field name="weight">0.01</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="categ_id" ref="point_of_sale.product_category_pos"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-minute_maid.jpg"/>
    </record>

    <record id="espresso" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">4.70</field>
        <field name="name">Espresso</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-espresso.jpg"/>
    </record>

    <record id="green_tea" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">4.70</field>
        <field name="name">Green Tea</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-green_tea.jpg"/>
    </record>

    <record id="milkshake_banana" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">3.60</field>
        <field name="name">Milkshake Banana</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-milkshake_banana.jpg"/>
    </record>

    <record id="ice_tea" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">2.20</field>
        <field name="name">Ice Tea</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-ice_tea.jpg"/>
    </record>

    <record id="schweppes" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">2.20</field>
        <field name="name">Schweppes</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-schweppes.jpg"/>
    </record>

    <record id="fanta" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">2.20</field>
        <field name="name">Fanta</field>
        <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
        <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-fanta.jpg"/>
    </record>

    <function model="pos.config" name="add_cash_payment_method" />
</odoo>

```

## File: data\pos_restaurant_onboarding_main_config.xml

```xml
<odoo noupdate="1">
     <!-- Pos Config -->
    <record model="pos.config" id="pos_config_main_restaurant">
        <field name="name">Restaurant</field>
        <field name="iface_printbill">True</field>
    </record>

    <!-- Closed Sessions -->
    <!-- forcecreate is set to false in order to not create record when updating the db -->

    <function model="ir.model.data" name="_update_xmlids">
        <value model="base" eval="[{
                'xml_id': 'pos_restaurant.payment_method',
                'record': obj().env.ref('pos_restaurant.pos_config_main_restaurant')._get_payment_method('bank'),
                'noupdate': True,
            }]" />
    </function>

    <!-- Closed Session 3 -->

    <record id="pos_closed_session_3" model="pos.session" forcecreate="False" context="{'onboarding_creation': True}">
        <field name="config_id" ref="pos_config_main_restaurant" />
        <field name="user_id" ref="base.user_admin" />
        <field name="start_at" eval="(DateTime.today() + relativedelta(days=-1)).strftime('%Y-%m-%d %H:%M:%S')" />
        <field name="stop_at"
            eval="(DateTime.today() + relativedelta(days=-1, hours=1)).strftime('%Y-%m-%d %H:%M:%S')" />
    </record>

    <record id="pos_closed_order_3_1" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_closed_session_3" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">ClosedDemo/0005</field>
        <field name="state">paid</field>
        <field name="amount_total">14.0</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">14.0</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00000-003-1001</field>
    </record>

    <record id="pos_closed_orderline_3_1_1" model="pos.order.line" forcecreate="False">
        <field name="name">Closed Orderline 3.1.1</field>
        <field name="product_id" ref="pos_food_margherita" />
        <field name="price_subtotal">7.0</field>
        <field name="price_subtotal_incl">7.0</field>
        <field name="price_unit">7.0</field>
        <field name="order_id" ref="pos_closed_order_3_1" />
        <field name="full_product_name">Margherita</field>
    </record>

    <record id="pos_closed_orderline_3_1_2" model="pos.order.line" forcecreate="False">
        <field name="name">Closed Orderline 3.1.2</field>
        <field name="product_id" ref="pos_food_funghi" />
        <field name="price_subtotal">7.0</field>
        <field name="price_subtotal_incl">7.0</field>
        <field name="price_unit">7.0</field>
        <field name="order_id" ref="pos_closed_order_3_1" />
        <field name="full_product_name">Funghi</field>
    </record>

    <record id="pos_payment_1" model="pos.payment" forcecreate="False">
        <field name="payment_method_id" ref="pos_restaurant.payment_method" />
        <field name="pos_order_id" ref="pos_closed_order_3_1" />
        <field name="amount">14.0</field>
    </record>

    <record id="pos_closed_order_3_2" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_closed_session_3" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">ClosedDemo/0006</field>
        <field name="state">paid</field>
        <field name="amount_total">7.0</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">7.0</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00000-003-1002</field>
    </record>

    <record id="pos_closed_orderline_3_2_1" model="pos.order.line" forcecreate="False">
        <field name="name">Closed Orderline 3.2.1</field>
        <field name="product_id" ref="pos_food_vege" />
        <field name="price_subtotal">7.0</field>
        <field name="price_subtotal_incl">7.0</field>
        <field name="price_unit">7.0</field>
        <field name="order_id" ref="pos_closed_order_3_2" />
        <field name="full_product_name">Vegetarian</field>
    </record>

    <record id="pos_payment_2" model="pos.payment" forcecreate="False">
        <field name="payment_method_id" ref="pos_restaurant.payment_method" />
        <field name="pos_order_id" ref="pos_closed_order_3_2" />
        <field name="amount">7.0</field>
    </record>

    <function model="pos.session" name="action_pos_session_closing_control"
        eval="[[ref('pos_closed_session_3')]]" />

    <!-- Closed Session 4 -->

    <record id="pos_closed_session_4" model="pos.session" forcecreate="False" context="{'onboarding_creation': True}">
        <field name="config_id" ref="pos_config_main_restaurant" />
        <field name="user_id" ref="base.user_admin" />
        <field name="start_at" eval="(DateTime.today() + relativedelta(days=-1)).strftime('%Y-%m-%d %H:%M:%S')" />
        <field name="stop_at"
            eval="(DateTime.today() + relativedelta(days=-1, hours=1)).strftime('%Y-%m-%d %H:%M:%S')" />
    </record>

    <record id="pos_closed_order_4_1" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_closed_session_4" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">ClosedDemo/0007</field>
        <field name="state">paid</field>
        <field name="amount_total">6.7</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">6.7</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00000-004-1001</field>
    </record>

    <record id="pos_closed_orderline_4_1_1" model="pos.order.line" forcecreate="False">
        <field name="name">Closed Orderline 4.1.1</field>
        <field name="product_id" ref="water" />
        <field name="price_subtotal">2.20</field>
        <field name="price_subtotal_incl">2.20</field>
        <field name="price_unit">2.20</field>
        <field name="order_id" ref="pos_closed_order_4_1" />
        <field name="full_product_name">Water</field>
    </record>

    <record id="pos_closed_orderline_4_1_2" model="pos.order.line" forcecreate="False">
        <field name="name">Closed Orderline 4.1.2</field>
        <field name="product_id" ref="pos_food_bolo" />
        <field name="price_subtotal">4.5</field>
        <field name="price_subtotal_incl">4.5</field>
        <field name="price_unit">4.5</field>
        <field name="order_id" ref="pos_closed_order_4_1" />
        <field name="full_product_name">Pasta Bolognese</field>
    </record>

    <record id="pos_payment_3" model="pos.payment" forcecreate="False">
        <field name="payment_method_id" ref="pos_restaurant.payment_method" />
        <field name="pos_order_id" ref="pos_closed_order_4_1" />
        <field name="amount">6.7</field>
    </record>

    <record id="pos_closed_order_4_2" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_closed_session_4" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">ClosedDemo/0008</field>
        <field name="state">paid</field>
        <field name="amount_total">28.0</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">28.0</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00000-004-1002</field>
    </record>

    <record id="pos_closed_orderline_4_2_1" model="pos.order.line" forcecreate="False">
        <field name="name">Closed Orderline 4.2.1</field>
        <field name="product_id" ref="pos_food_cheeseburger" />
        <field name="price_subtotal">28.0</field>
        <field name="price_subtotal_incl">28.0</field>
        <field name="price_unit">7.0</field>
        <field name="qty">4</field>
        <field name="order_id" ref="pos_closed_order_4_2" />
        <field name="full_product_name">Cheese Burger</field>
    </record>

    <record id="pos_payment_4" model="pos.payment" forcecreate="False">
        <field name="payment_method_id" ref="pos_restaurant.payment_method" />
        <field name="pos_order_id" ref="pos_closed_order_4_2" />
        <field name="amount">28.0</field>
    </record>

    <function model="pos.session" name="action_pos_session_closing_control"
        eval="[[ref('pos_closed_session_4')]]" />

    <!-- Floors: Main Floor -->
    <record id="floor_main" model="restaurant.floor">
        <field name="name">Main Floor</field>
        <field name="background_color">rgb(249,250,251)</field>
        <field name="pos_config_ids" eval="[(6, 0, [ref('pos_restaurant.pos_config_main_restaurant')])]" />
    </record>

    <record id="table_01" model="restaurant.table">
        <field name="name">1</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">50</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_02" model="restaurant.table">
        <field name="name">2</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">212</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_03" model="restaurant.table">
        <field name="name">3</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">374</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_04" model="restaurant.table">
        <field name="name">4</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">536</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_05" model="restaurant.table">
        <field name="name">5</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">698</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_06" model="restaurant.table">
        <field name="name">6</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">860</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_07" model="restaurant.table">
        <field name="name">7</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(235,109,109)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">50</field>
        <field name="position_v">280</field>
    </record>

    <record id="table_08" model="restaurant.table">
        <field name="name">8</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(235,109,109)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">212</field>
        <field name="position_v">280</field>
    </record>

    <record id="table_09" model="restaurant.table">
        <field name="name">9</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(235,109,109)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">698</field>
        <field name="position_v">280</field>
    </record>

    <record id="table_10" model="restaurant.table">
        <field name="name">10</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
        <field name="seats">4</field>
        <field name="color">rgb(235,109,109)</field>
        <field name="shape">square</field>
        <field name="width">100</field>
        <field name="height">100</field>
        <field name="position_h">860</field>
        <field name="position_v">280</field>
    </record>

    <record id="table_11" model="restaurant.table">
        <field name="name">11</field>
        <field name="floor_id" ref="pos_restaurant.floor_main" />
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
        <field name="pos_config_ids" eval="[(6, 0, [ref('pos_restaurant.pos_config_main_restaurant')])]" />
    </record>

    <!-- Patio: Left table row -->

    <record id="table_21" model="restaurant.table">
        <field name="name">1</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">100</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_22" model="restaurant.table">
        <field name="name">2</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">100</field>
        <field name="position_v">166</field>
    </record>

    <record id="table_23" model="restaurant.table">
        <field name="name">3</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">100</field>
        <field name="position_v">283</field>
    </record>

    <record id="table_24" model="restaurant.table">
        <field name="name">4</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">100</field>
        <field name="position_v">400</field>
    </record>

    <!-- Patio: Right table row -->

    <record id="table_25" model="restaurant.table">
        <field name="name">5</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">800</field>
        <field name="position_v">50</field>
    </record>

    <record id="table_26" model="restaurant.table">
        <field name="name">6</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">800</field>
        <field name="position_v">166</field>
    </record>

    <record id="table_27" model="restaurant.table">
        <field name="name">7</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">800</field>
        <field name="position_v">283</field>
    </record>

    <record id="table_28" model="restaurant.table">
        <field name="name">8</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">2</field>
        <field name="color">rgb(53,211,116)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">85</field>
        <field name="position_h">800</field>
        <field name="position_v">400</field>
    </record>

    <!-- Patio: Center table block -->

    <record id="table_29" model="restaurant.table">
        <field name="name">9</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">4</field>
        <field name="color">rgb(235,191,109)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">120</field>
        <field name="position_h">330</field>
        <field name="position_v">100</field>
    </record>

    <record id="table_29" model="restaurant.table">
        <field name="name">9</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">4</field>
        <field name="color">rgb(235,191,109)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">120</field>
        <field name="position_h">330</field>
        <field name="position_v">100</field>
    </record>

    <record id="table_30" model="restaurant.table">
        <field name="name">10</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">4</field>
        <field name="color">rgb(235,191,109)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">120</field>
        <field name="position_h">560</field>
        <field name="position_v">100</field>
    </record>

    <record id="table_31" model="restaurant.table">
        <field name="name">11</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
        <field name="seats">4</field>
        <field name="color">rgb(235,191,109)</field>
        <field name="shape">square</field>
        <field name="width">130</field>
        <field name="height">120</field>
        <field name="position_h">330</field>
        <field name="position_v">315</field>
    </record>

    <record id="table_32" model="restaurant.table">
        <field name="name">12</field>
        <field name="floor_id" ref="pos_restaurant.floor_patio" />
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

## File: data\pos_restaurant_onboarding_open_session.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Open Session -->

    <function model="ir.model.data" name="_update_xmlids">
        <value model="base" eval="[{
                'xml_id': 'pos_restaurant.pos_open_session_2',
                'record': obj().env.ref('pos_restaurant.pos_config_main_restaurant').current_session_id,
                'noupdate': True,
                }]" />
    </function>

    <record id="pos_open_order_2" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_open_session_2" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">Restaurant/00001</field>
        <field name="state">draft</field>
        <field name="amount_total">22.90</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">0.0</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00002-001-0000</field>
        <field name="table_id" ref="table_01" />
        <field name="customer_count">8</field>
    </record>

    <record id="pos_orderline_2" model="pos.order.line" forcecreate="False">
        <field name="name">Orderline 2</field>
        <field name="product_id" ref="coke" />
        <field name="price_subtotal">4.40</field>
        <field name="price_subtotal_incl">4.40</field>
        <field name="price_unit">2.20</field>
        <field name="qty">2</field>
        <field name="order_id" ref="pos_open_order_2" />
        <field name="full_product_name">Coca-Cola</field>
        <field name="uuid">00000000-0000-4000-000000000000</field>
    </record>

    <record id="pos_orderline_3" model="pos.order.line" forcecreate="False">
        <field name="name">Orderline 3</field>
        <field name="product_id" ref="pos_food_chirashi" />
        <field name="price_subtotal">18.5</field>
        <field name="price_subtotal_incl">18.5</field>
        <field name="price_unit">9.25</field>
        <field name="qty">2</field>
        <field name="order_id" ref="pos_open_order_2" />
        <field name="full_product_name">Salmon and Avocado</field>
        <field name="uuid">00000000-0000-4000-000000000001</field>
    </record>

    <record id="pos_open_order_3" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_open_session_2" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">Restaurant/00002</field>
        <field name="state">draft</field>
        <field name="amount_total">21.8</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">0.0</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00002-002-0000</field>
        <field name="table_id" ref="table_02" />
        <field name="customer_count">3</field>
    </record>

    <record id="pos_orderline_4" model="pos.order.line" forcecreate="False">
        <field name="name">Orderline 4</field>
        <field name="product_id" ref="pos_food_temaki" />
        <field name="price_subtotal">14.0</field>
        <field name="price_subtotal_incl">14.0</field>
        <field name="price_unit">14.0</field>
        <field name="qty">1</field>
        <field name="order_id" ref="pos_open_order_3" />
        <field name="full_product_name">Lunch Temaki mix 3pc</field>
        <field name="uuid">00000000-0000-4000-000000000002</field>
    </record>

    <record id="pos_orderline_5" model="pos.order.line" forcecreate="False">
        <field name="name">Orderline 5</field>
        <field name="product_id" ref="pos_food_mozza" />
        <field name="price_subtotal">7.8</field>
        <field name="price_subtotal_incl">7.8</field>
        <field name="price_unit">3.9</field>
        <field name="qty">2</field>
        <field name="order_id" ref="pos_open_order_3" />
        <field name="full_product_name">Mozzarella Sandwich</field>
        <field name="uuid">00000000-0000-4000-000000000003</field>
    </record>

    <record id="pos_open_order_4" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_open_session_2" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">Restaurant/00003</field>
        <field name="state">draft</field>
        <field name="amount_total">10.5</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">0.0</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00002-003-0000</field>
        <field name="table_id" ref="table_04" />
        <field name="customer_count">5</field>
    </record>

    <record id="pos_orderline_6" model="pos.order.line" forcecreate="False">
        <field name="name">Orderline 6</field>
        <field name="product_id" ref="pos_food_chicken" />
        <field name="price_subtotal">3.0</field>
        <field name="price_subtotal_incl">3.0</field>
        <field name="price_unit">3.0</field>
        <field name="qty">1</field>
        <field name="order_id" ref="pos_open_order_4" />
        <field name="full_product_name">Chicken Curry Sandwich</field>
        <field name="uuid">00000000-0000-4000-000000000004</field>
    </record>

    <record id="pos_orderline_7" model="pos.order.line" forcecreate="False">
        <field name="name">Orderline 7</field>
        <field name="product_id" ref="pos_food_bacon" />
        <field name="price_subtotal">7.5</field>
        <field name="price_subtotal_incl">7.5</field>
        <field name="price_unit">7.5</field>
        <field name="qty">1</field>
        <field name="order_id" ref="pos_open_order_4" />
        <field name="full_product_name">Bacon Burger</field>
        <field name="uuid">00000000-0000-4000-000000000005</field>
    </record>

    <record id="pos_open_order_5" model="pos.order" forcecreate="False">
        <field name="session_id" ref="pos_open_session_2" />
        <field name="company_id" ref="base.main_company" />
        <field name="name">Restaurant/00004</field>
        <field name="state">draft</field>
        <field name="amount_total">5.5</field>
        <field name="amount_tax">0.0</field>
        <field name="amount_paid">0.0</field>
        <field name="amount_return">0.0</field>
        <field name="pos_reference">Order 00002-004-0000</field>
        <field name="table_id" ref="table_06" />
        <field name="customer_count">1</field>
    </record>

    <record id="pos_orderline_8" model="pos.order.line" forcecreate="False">
        <field name="name">Orderline 8</field>
        <field name="product_id" ref="pos_food_4formaggi" />
        <field name="price_subtotal">5.5</field>
        <field name="price_subtotal_incl">5.5</field>
        <field name="price_unit">5.5</field>
        <field name="qty">1</field>
        <field name="order_id" ref="pos_open_order_5" />
        <field name="full_product_name">Pizza 4 Formaggi</field>
    </record>

    <function model="pos.session" name="_set_last_order_preparation_change"
            eval="[[ref('pos_open_order_2'), ref('pos_open_order_3'), ref('pos_open_order_4')]]"/>
</odoo>

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
import json
from collections import defaultdict


class PosConfig(models.Model):
    _inherit = 'pos.config'

    iface_splitbill = fields.Boolean(string='Bill Splitting', help='Enables Bill Splitting in the Point of Sale.')
    iface_printbill = fields.Boolean(string='Bill Printing', help='Allows to print the Bill before payment.')
    iface_orderline_notes = fields.Boolean(string='Internal Notes', help='Allow custom Internal notes on Orderlines.')
    floor_ids = fields.Many2many('restaurant.floor', string='Restaurant Floors', help='The restaurant floors served by this point of sale.')
    set_tip_after_payment = fields.Boolean('Set Tip After Payment', help="Adjust the amount authorized by payment terminals to add a tip after the customers left or at the end of the day.")
    module_pos_restaurant = fields.Boolean(default=True)
    module_pos_restaurant_appointment = fields.Boolean("Table Booking")

    def get_tables_order_count_and_printing_changes(self):
        self.ensure_one()
        floors = self.env['restaurant.floor'].search([('pos_config_ids', '=', self.id)])
        tables = self.env['restaurant.table'].search([('floor_id', 'in', floors.ids)])
        domain = [('state', '=', 'draft'), ('table_id', 'in', tables.ids)]

        order_stats = self.env['pos.order']._read_group(domain, ['table_id'], ['__count'])
        linked_orderlines = self.env['pos.order.line'].search([('order_id.state', '=', 'draft'), ('order_id.table_id', 'in', tables.ids)])
        orders_map = {table.id: count for table, count in order_stats}
        changes_map = defaultdict(lambda: 0)
        skip_changes_map = defaultdict(lambda: 0)

        for line in linked_orderlines:
            # For the moment, as this feature is not compatible with pos_self_order,
            # we ignore last_order_preparation_change when it is set to false.
            # In future, pos_self_order will send the various changes to the order.
            if not line.order_id.last_order_preparation_change:
                line.order_id.last_order_preparation_change = '{}'

            last_order_preparation_change = json.loads(line.order_id.last_order_preparation_change)
            prep_change = {}
            for line_uuid in last_order_preparation_change:
                prep_change[last_order_preparation_change[line_uuid]['line_uuid']] = last_order_preparation_change[line_uuid]
            quantity_changed = 0
            if line.uuid in prep_change:
                quantity_changed = line.qty - prep_change[line.uuid]['quantity']
            else:
                quantity_changed = line.qty

            if line.skip_change:
                skip_changes_map[line.order_id.table_id.id] += quantity_changed
            else:
                changes_map[line.order_id.table_id.id] += quantity_changed

        result = []
        for table in tables:
            result.append({'id': table.id, 'orders': orders_map.get(table.id, 0), 'changes': changes_map.get(table.id, 0), 'skip_changes': skip_changes_map.get(table.id, 0)})
        return result

    def _get_forbidden_change_fields(self):
        forbidden_keys = super(PosConfig, self)._get_forbidden_change_fields()
        forbidden_keys.append('floor_ids')
        return forbidden_keys

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            is_restaurant = 'module_pos_restaurant' not in vals or vals['module_pos_restaurant']
            if is_restaurant and 'iface_splitbill' not in vals:
                vals['iface_splitbill'] = True
            if not is_restaurant or not vals.get('iface_tipproduct', False):
                vals['set_tip_after_payment'] = False
        pos_configs = super().create(vals_list)
        for config in pos_configs:
            if config.module_pos_restaurant:
                self._setup_default_floor(config)
        return pos_configs

    def write(self, vals):
        if ('module_pos_restaurant' in vals and vals['module_pos_restaurant'] is False):
            vals['floor_ids'] = [(5, 0, 0)]

        if ('module_pos_restaurant' in vals and not vals['module_pos_restaurant']) or ('iface_tipproduct' in vals and not vals['iface_tipproduct']):
            vals['set_tip_after_payment'] = False

        if ('module_pos_restaurant' in vals and vals['module_pos_restaurant']):
            self._setup_default_floor(self)

        return super().write(vals)

    @api.model
    def post_install_pos_localisation(self, companies=False):
        self = self.sudo()
        if not companies:
            companies = self.env['res.company'].search([])
        super(PosConfig, self).post_install_pos_localisation(companies)
        for company in companies.filtered('chart_template'):
            pos_configs = self.search([
                *self.env['account.journal']._check_company_domain(company),
                ('module_pos_restaurant', '=', True),
            ])
            if not pos_configs:
                pos_configs = self.env['pos.config'].with_company(company).create({
                'name': _('Bar'),
                'company_id': company.id,
                'module_pos_restaurant': True,
                'iface_splitbill': True,
                'iface_printbill': True,
                'iface_orderline_notes': True,

            })
            pos_configs.setup_defaults(company)

    def setup_defaults(self, company):
        main_restaurant = self.env.ref('pos_restaurant.pos_config_main_restaurant', raise_if_not_found=False)
        main_restaurant_is_present = main_restaurant and not main_restaurant.has_active_session and self.filtered(lambda cfg: cfg.id == main_restaurant.id)
        if main_restaurant_is_present:
            non_main_restaurant_configs = self - main_restaurant
            non_main_restaurant_configs.assign_payment_journals(company)
            main_restaurant._setup_main_restaurant_defaults()
            self.generate_pos_journal(company)
            self.setup_invoice_journal(company)
        else:
            super().setup_defaults(company)

    def _setup_main_restaurant_defaults(self):
        self.ensure_one()
        self._link_same_non_cash_payment_methods_if_exists('point_of_sale.pos_config_main')
        self._ensure_cash_payment_method('MRCSH', _('Cash Restaurant'))
        self._archive_shop()

    def _archive_shop(self):
        shop = self.env.ref('point_of_sale.pos_config_main', raise_if_not_found=False)
        if shop:
            session_count = self.env['pos.session'].search_count([('config_id', '=', shop.id)])
            if session_count == 0:
                shop.update({'active': False})

    def _setup_default_floor(self, pos_config):
        if not pos_config.floor_ids:
            main_floor = self.env['restaurant.floor'].create({
                'name': pos_config.company_id.name,
                'pos_config_ids': [(4, pos_config.id)],
            })
            self.env['restaurant.table'].create({
                'name': '1',
                'floor_id': main_floor.id,
                'seats': 1,
                'position_h': 100,
                'position_v': 100,
                'width': 100,
                'height': 100,
            })

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from functools import partial

from odoo import api, fields, models


class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    note = fields.Char('Internal Note added by the waiter.')

    def _export_for_ui(self, orderline):
        return {
            'note': orderline.note,
            **super()._export_for_ui(orderline),
        }


class PosOrder(models.Model):
    _inherit = 'pos.order'

    table_id = fields.Many2one('restaurant.table', string='Table', help='The table where this order was served', index='btree_not_null', readonly=True)
    customer_count = fields.Integer(string='Guests', help='The amount of customers that have been served by this order.', readonly=True)

    @api.model
    def remove_from_ui(self, server_ids):
        tables = self.env['pos.order'].search([('id', 'in', server_ids)]).table_id
        order_ids = super().remove_from_ui(server_ids)
        self.send_table_count_notification(tables)
        return order_ids

    def _process_saved_order(self, draft):
        order_id = super()._process_saved_order(draft)
        self.send_table_count_notification(self.table_id)
        return order_id

    def send_table_count_notification(self, table_ids):
        messages = []
        for config in self.env['pos.config'].search([('floor_ids', 'in', table_ids.floor_id.ids)]):
            config_cur_session = config.current_session_id
            if config_cur_session:
                order_count = config.get_tables_order_count_and_printing_changes()
                messages.append((config_cur_session._get_bus_channel_name(), 'TABLE_ORDER_COUNT', order_count))
        self.env['bus.bus']._sendmany(messages)

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
        return order_fields

    def _export_for_ui(self, order):
        result = super(PosOrder, self)._export_for_ui(order)
        result['table_id'] = order.table_id.id
        result['customer_count'] = order.customer_count
        return result

    @api.model
    def export_for_ui_table_draft(self, table_ids):
        orders = self.env['pos.order'].search([('state', '=', 'draft'), ('table_id', 'in', table_ids)])
        return orders.export_for_ui()

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

from odoo import api, fields, models, _, Command
from odoo.exceptions import UserError


class RestaurantFloor(models.Model):

    _name = 'restaurant.floor'
    _description = 'Restaurant Floor'
    _order = "sequence, name"

    name = fields.Char('Floor Name', required=True)
    pos_config_ids = fields.Many2many('pos.config', string='Point of Sales', domain="[('module_pos_restaurant', '=', True)]")
    background_image = fields.Binary('Background Image')
    background_color = fields.Char('Background Color', help='The background color of the floor in a html-compatible format', default='rgb(210, 210, 210)')
    table_ids = fields.One2many('restaurant.table', 'floor_id', string='Tables')
    sequence = fields.Integer('Sequence', default=1)
    active = fields.Boolean(default=True)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_pos_session(self):
        confs = self.mapped('pos_config_ids').filtered(lambda c: c.module_pos_restaurant)
        opened_session = self.env['pos.session'].search([('config_id', 'in', confs.ids), ('state', '!=', 'closed')])
        if opened_session and confs:
            error_msg = _("You cannot remove a floor that is used in a PoS session, close the session(s) first: \n")
            for floor in self:
                for session in opened_session:
                    if floor in session.config_id.floor_ids:
                        error_msg += _("Floor: %s - PoS Config: %s \n", floor.name, session.config_id.name)
            raise UserError(error_msg)

    def write(self, vals):
        for floor in self:
            for config in floor.pos_config_ids:
                if config.has_active_session and (vals.get('pos_config_ids') or vals.get('active')):
                    raise UserError(
                        'Please close and validate the following open PoS Session before modifying this floor.\n'
                        'Open session: %s' % (' '.join(config.mapped('name')),))
        return super(RestaurantFloor, self).write(vals)

    def rename_floor(self, new_name):
        for floor in self:
            floor.name = new_name

    @api.model
    def create_from_ui(self, name, background_color, config_id):
        floor_fields = {
            "name": name,
            "background_color": background_color,
        }
        pos_floor = self.create(floor_fields)
        pos_floor.pos_config_ids = [Command.link(config_id)]
        return {
            'id': pos_floor.id,
            'name': pos_floor.name,
            'background_color': pos_floor.background_color,
            'table_ids': [],
            'sequence': pos_floor.sequence,
            'tables': [],
        }

    def deactivate_floor(self, session_id):
        draft_orders = self.env['pos.order'].search([('session_id', '=', session_id), ('state', '=', 'draft'), ('table_id.floor_id', '=', self.id)])
        if draft_orders:
            raise UserError(_("You cannot delete a floor when orders are still in draft for this floor."))
        for table in self.table_ids:
            table.active = False
        self.active = False

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
    color = fields.Char('Color', help="The table's color, expressed as a valid 'background' CSS property value", default="#35D374")
    active = fields.Boolean('Active', default=True, help='If false, the table is deactivated and will not be available in the point of sale')

    def are_orders_still_in_draft(self):
        draft_orders_count = self.env['pos.order'].search_count([('table_id', 'in', self.ids), ('state', '=', 'draft')])
        return draft_orders_count > 0

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_pos_session(self):
        confs = self.mapped('floor_id.pos_config_ids').filtered(lambda c: c.module_pos_restaurant)
        opened_session = self.env['pos.session'].search([('config_id', 'in', confs.ids), ('state', '!=', 'closed')])
        if opened_session:
            error_msg = _("You cannot remove a table that is used in a PoS session, close the session(s) first.")
            if confs:
                raise UserError(error_msg)

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, Command, api
from odoo.tools import convert
from itertools import groupby
from odoo.osv.expression import AND
import json

class PosSession(models.Model):
    _inherit = 'pos.session'

    def _pos_ui_models_to_load(self):
        result = super()._pos_ui_models_to_load()
        if self.config_id.module_pos_restaurant:
            result.append('restaurant.floor')
        return result

    def _loader_params_restaurant_floor(self):
        return {
            'search_params': {
                'domain': [('pos_config_ids', '=', self.config_id.id)],
                'fields': ['name', 'background_color', 'table_ids', 'sequence'],
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

    def get_pos_ui_restaurant_floor(self):
        return self._get_pos_ui_restaurant_floor(self._loader_params_restaurant_floor())

    def get_onboarding_data(self):
        results = super().get_onboarding_data()
        if self.config_id.module_pos_restaurant:
            results.update({
                'restaurant.floor': self._load_model('restaurant.floor'),
            })
        return results

    @api.model
    def _load_onboarding_data(self):
        super()._load_onboarding_data()
        convert.convert_file(self.env, 'pos_restaurant', 'data/pos_restaurant_onboarding.xml', None, mode='init', noupdate=True, kind='data')
        restaurant_config = self.env.ref('pos_restaurant.pos_config_main_restaurant', raise_if_not_found=False)
        if restaurant_config:
            convert.convert_file(self.env, 'pos_restaurant', 'data/pos_restaurant_onboarding_main_config.xml', None, mode='init', noupdate=True, kind='data')
            if len(restaurant_config.session_ids.filtered(lambda s: s.state == 'opened')) == 0:
                self.env['pos.session'].create({
                    'config_id': restaurant_config.id,
                    'user_id': self.env.ref('base.user_admin').id,
                })
            convert.convert_file(self.env, 'pos_restaurant', 'data/pos_restaurant_onboarding_open_session.xml', None, mode='init', noupdate=True, kind='data')

    def _after_load_onboarding_data(self):
        super()._after_load_onboarding_data()
        configs = self.config_id.filtered('module_pos_restaurant')
        if configs:
            configs.with_context(bypass_categories_forbidden_change=True).write({
                'limit_categories': True,
                'iface_available_categ_ids': [Command.link(self.env.ref('pos_restaurant.food').id), Command.link(self.env.ref('pos_restaurant.drinks').id)]
            })

    @api.model
    def _set_last_order_preparation_change(self, order_ids):
        for order_id in order_ids:
            order = self.env['pos.order'].browse(order_id)
            last_order_preparation_change = {}
            for orderline in order['lines']:
                last_order_preparation_change[orderline.uuid + " - "] = {
                    "line_uuid": orderline.uuid,
                    "name": orderline.full_product_name,
                    "note": "",
                    "product_id": orderline.product_id.id,
                    "quantity": orderline.qty,
                    "attribute_value_ids": orderline.attribute_value_ids.ids,
                }
            order.write({'last_order_preparation_change': json.dumps(last_order_preparation_change)})

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pos_floor_ids = fields.Many2many(related='pos_config_id.floor_ids', readonly=False)
    pos_iface_orderline_notes = fields.Boolean(related='pos_config_id.iface_orderline_notes', readonly=False)
    pos_iface_printbill = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_iface_splitbill = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_set_tip_after_payment = fields.Boolean(compute='_compute_pos_set_tip_after_payment', store=True, readonly=False)
    pos_module_pos_restaurant_appointment = fields.Boolean(related="pos_config_id.module_pos_restaurant_appointment", readonly=False)

    @api.depends('pos_module_pos_restaurant', 'pos_config_id')
    def _compute_pos_module_pos_restaurant(self):
        for res_config in self:
            if not res_config.pos_module_pos_restaurant:
                res_config.update({
                    'pos_iface_printbill': False,
                    'pos_iface_splitbill': False,
                })
            else:
                res_config.update({
                    'pos_iface_printbill': res_config.pos_config_id.iface_printbill,
                    'pos_iface_splitbill': res_config.pos_config_id.iface_splitbill,
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
access_restaurant_floor,restaurant.floor.user,model_restaurant_floor,point_of_sale.group_pos_user,1,0,0,0
access_restaurant_floor_manager,restaurant.floor.manager,model_restaurant_floor,point_of_sale.group_pos_manager,1,1,1,1
access_restaurant_table,restaurant.table.user,model_restaurant_table,point_of_sale.group_pos_user,1,0,0,0
access_restaurant_table_manager,restaurant.table.manager,model_restaurant_table,point_of_sale.group_pos_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><circle cx="25" cy="25.001" r="17" fill="#985184"/><path d="m42.586 15.587-8.172-8.172c-1.26-1.26-.367-3.414 1.414-3.414H42a4 4 0 0 1 4 4v6.171c0 1.782-2.154 2.675-3.414 1.415ZM7.414 34.415l8.172 8.172c1.26 1.26.367 3.414-1.414 3.414H8a4 4 0 0 1-4-4v-6.172c0-1.781 2.154-2.674 3.414-1.414Zm27 8.172 8.172-8.172c1.26-1.26 3.414-.367 3.414 1.414v6.172a4 4 0 0 1-4 4h-6.172c-1.781 0-2.674-2.154-1.414-3.414ZM15.586 7.415l-8.172 8.172C6.154 16.847 4 15.954 4 14.172v-6.17a4 4 0 0 1 4-4h6.172c1.781 0 2.674 2.154 1.414 3.414Z" fill="#FBB945"/></svg>

```

## File: static\src\app\bill_screen\bill_screen.js

```javascript
/** @odoo-module */

import { ReceiptScreen } from "@point_of_sale/app/screens/receipt_screen/receipt_screen";
import { OrderReceipt } from "@point_of_sale/app/screens/receipt_screen/receipt/order_receipt";
import { registry } from "@web/core/registry";

export class BillScreen extends ReceiptScreen {
    static template = "pos_restaurant.BillScreen";
    static components = { OrderReceipt };
    confirm() {
        if (!this.env.isMobile) {
            this.props.resolve({ confirmed: true, payload: null });
            this.pos.closeTempScreen();
        }
    }
    /**
     * @override
     */
    async printReceipt() {
        const order = this.currentOrder;
        await super.printReceipt();
        order._printed = false;
    }

    get isBill() {
        return true;
    }
}

registry.category("pos_screens").add("BillScreen", BillScreen);

```

## File: static\src\app\bill_screen\bill_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.BillScreen">
        <div class="receipt-screen screen h-100 bg-100">
            <div class="screen-content d-flex flex-column h-100">
                <div class="top-content d-flex align-items-center p-2 border-bottom text-center">
                    <div class="top-content-center flex-grow-1">
                        <h2 class="mb-0 text-start">Bill Printing</h2>
                    </div>
                    <button class="button next highlight btn btn-lg btn-outline-primary" t-on-click="confirm">
                        <span>Ok</span>
                        <i class="fa fa-angle-double-right ms-2"></i>
                    </button>
                </div>
                <div class="centered-content mx-auto mt-3 border-start border-end text-center overflow-x-hidden overflow-y-auto">
                    <div class="button print btn btn-lg btn-primary" t-on-click="printReceipt">
                        <i class="fa fa-print" t-ref="order-print-receipt-button"></i>
                        <span> </span>
                        <span>Print</span>
                    </div>
                    <div class="pos-receipt-container text-center">
                        <div class="d-inline-block m-3 p-3 border rounded bg-view text-start overflow-hidden">
                            <OrderReceipt data="{...pos.get_order().export_for_printing(), isBill: isBill}" formatCurrency="env.utils.formatCurrency" />
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\app\control_buttons\orderline_note_button\orderline_note_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { useService } from "@web/core/utils/hooks";
import { TextAreaPopup } from "@point_of_sale/app/utils/input_popups/textarea_popup";
import { Component } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

export class OrderlineNoteButton extends Component {
    static template = "pos_restaurant.OrderlineNoteButton";

    setup() {
        this.pos = usePos();
        this.popup = useService("popup");
    }
    get selectedOrderline() {
        return this.pos.get_order().get_selected_orderline();
    }
    async click() {
        if (!this.selectedOrderline) {
            return;
        }

        const oldNote = this.selectedOrderline.getNote();
        const { confirmed, payload: inputNote } = await this.popup.add(TextAreaPopup, {
            startingValue: this.selectedOrderline.getNote(),
            title: _t("Add internal Note"),
        });

        if (confirmed) {
            this.selectedOrderline.setNote(inputNote);
        }

        return { confirmed, inputNote, oldNote };
    }
}

ProductScreen.addControlButton({
    component: OrderlineNoteButton,
    condition: function () {
        return this.pos.config.iface_orderline_notes;
    },
});

```

## File: static\src\app\control_buttons\orderline_note_button\orderline_note_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.OrderlineNoteButton">
        <button class="control-button btn btn-light rounded-0 fw-bolder" t-on-click="() => this.click()" t-att-disabled="!selectedOrderline">
            <i class="fa fa-tag me-1" />
            <span> </span>
            <span>Internal Note</span>
        </button>
    </t>

</templates>

```

## File: static\src\app\control_buttons\print_bill_button\print_bill_button.js

```javascript
/** @odoo-module */

import { usePos } from "@point_of_sale/app/store/pos_hook";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { Component } from "@odoo/owl";
import { OrderReceipt } from "@point_of_sale/app/screens/receipt_screen/receipt/order_receipt";
import { useService } from "@web/core/utils/hooks";
import { useAsyncLockedMethod } from "@point_of_sale/app/utils/hooks";

export class PrintBillButton extends Component {
    static template = "pos_restaurant.PrintBillButton";

    setup() {
        this.pos = usePos();
        this.printer = useService("printer");
        this.click = useAsyncLockedMethod(this.click);
    }

    _isDisabled() {
        const order = this.pos.get_order();
        if (!order) {
            return false;
        }
        return order.get_orderlines().length === 0;
    }

    async click() {
        // Need to await to have the result in case of automatic skip screen.
        (await this.printer.print(OrderReceipt, {
            data: this.pos.get_order().export_for_printing(),
            formatCurrency: this.env.utils.formatCurrency,
        })) || this.pos.showTempScreen("BillScreen");
    }
}

ProductScreen.addControlButton({
    component: PrintBillButton,
    condition: function () {
        return this.pos.config.iface_printbill;
    },
});

```

## File: static\src\app\control_buttons\print_bill_button\print_bill_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.PrintBillButton">
        <span class="control-button order-printbill btn btn-light rounded-0 fw-bolder" t-att-class="{'disabled': _isDisabled()}" t-on-click="() => this.click()">
            <i class="fa fa-print me-1"></i>
            <span> </span>
            <span>Bill</span>
        </span>
    </t>

</templates>

```

## File: static\src\app\control_buttons\split_bill_button\split_bill_button.js

```javascript
/** @odoo-module */

import { usePos } from "@point_of_sale/app/store/pos_hook";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { Component } from "@odoo/owl";

export class SplitBillButton extends Component {
    static template = "pos_restaurant.SplitBillButton";

    setup() {
        this.pos = usePos();
    }
    _isDisabled() {
        const order = this.pos.get_order();
        return (
            order
                .get_orderlines()
                .reduce((totalProduct, orderline) => totalProduct + orderline.quantity, 0) < 2
        );
    }
    async click() {
        this.pos.showScreen("SplitBillScreen");
    }
}

ProductScreen.addControlButton({
    component: SplitBillButton,
    condition: function () {
        return this.pos.config.iface_splitbill;
    },
});

```

## File: static\src\app\control_buttons\split_bill_button\split_bill_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.SplitBillButton">
        <span class="control-button order-split btn btn-light rounded-0 fw-bolder" t-att-class="{'disabled': _isDisabled()}" t-on-click="() => this.click()">
            <i class="fa fa-files-o me-1"></i>
            <span> </span>
            <span>Split</span>
        </span>
    </t>

</templates>

```

## File: static\src\app\control_buttons\table_guests_button\table_guests_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { useService } from "@web/core/utils/hooks";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { Component } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

export class TableGuestsButton extends Component {
    static template = "pos_restaurant.TableGuestsButton";

    setup() {
        this.pos = usePos();
        this.popup = useService("popup");
    }
    get currentOrder() {
        return this.pos.get_order();
    }
    get nGuests() {
        return this.currentOrder ? this.currentOrder.getCustomerCount() : 0;
    }
    async click() {
        const { confirmed, payload: inputNumber } = await this.popup.add(NumberPopup, {
            startingValue: this.nGuests,
            cheap: true,
            title: _t("Guests?"),
            isInputSelected: true,
        });

        if (confirmed) {
            const guestCount = parseInt(inputNumber, 10) || 0;
            // Set the maximum number possible for an integer
            const max_capacity = 2 ** 31 - 1;
            if (guestCount > max_capacity) {
                await this.popup.add(ErrorPopup, {
                    title: _t("Blocked action"),
                    body: _t("You cannot put a number that exceeds %s ", max_capacity),
                });
                return;
            }

            if (guestCount == 0 && this.currentOrder.orderlines.length === 0) {
                this.pos.removeOrder(this.currentOrder);
                this.pos.showScreen("FloorScreen");
            }

            this.currentOrder.setCustomerCount(guestCount);
        }
    }
}

ProductScreen.addControlButton({
    component: TableGuestsButton,
    condition: function () {
        return this.pos.config.module_pos_restaurant;
    },
});

```

## File: static\src\app\control_buttons\table_guests_button\table_guests_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.TableGuestsButton">
        <button class="control-button btn btn-light rounded-0 fw-bolder" t-on-click="() => this.click()">
            <span class="control-button-number px-2 py-1 rounded-circle text-bg-dark fw-bolder small me-1">
                <t t-esc="nGuests" />
            </span>
            <span> </span>
            <span>Dine-in Guests</span>
        </button>
    </t>

</templates>

```

## File: static\src\app\control_buttons\transfer_order_button\transfer_order_button.js

```javascript
/** @odoo-module */

import { usePos } from "@point_of_sale/app/store/pos_hook";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { Component } from "@odoo/owl";

export class TransferOrderButton extends Component {
    static template = "pos_restaurant.TransferOrderButton";

    setup() {
        this.pos = usePos();
    }
    async click() {
        this.pos.setCurrentOrderToTransfer();
        this.pos.showScreen("FloorScreen");
    }
}

ProductScreen.addControlButton({
    component: TransferOrderButton,
    condition: function () {
        return this.pos.config.module_pos_restaurant;
    },
});

```

## File: static\src\app\control_buttons\transfer_order_button\transfer_order_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.TransferOrderButton">
        <button class="control-button btn btn-light rounded-0 fw-bolder" t-on-click="() => this.click()">
            <i class="oi oi-arrow-right me-1" />
            <span> </span>
            <span>Transfer</span>
        </button>
    </t>

</templates>

```

## File: static\src\app\floor_screen\editable_table.js

```javascript
/** @odoo-module */

import { getLimits, useMovable, constrain } from "@point_of_sale/app/utils/movable_hook";
import { onWillUnmount, useEffect, useRef, Component } from "@odoo/owl";
import { Table } from "@pos_restaurant/app/floor_screen/table";
import { usePos } from "@point_of_sale/app/store/pos_hook";

const MIN_TABLE_SIZE = 30; // px

export class EditableTable extends Component {
    static template = "pos_restaurant.EditableTable";
    static props = {
        onSaveTable: Function,
        limit: { type: Object, shape: { el: [HTMLElement, { value: null }] } },
        table: Table.props.table,
        selectedTables: Array,
    };

    setup() {
        this.pos = usePos();
        useEffect(this._setElementStyle.bind(this));
        this.root = useRef("root");
        this.handles = {
            "top left": ["minX", "minY"],
            "top right": ["maxX", "minY"],
            "bottom left": ["minX", "maxY"],
            "bottom right": ["maxX", "maxY"],
        };
        // make table draggable
        useMovable({
            ref: this.root,
            onMoveStart: () => this.onMoveStart(),
            onMove: (delta) => this.onMove(delta),
        });
        // make table resizable
        for (const [handle, toMove] of Object.entries(this.handles)) {
            useMovable({
                ref: useRef(handle),
                onMoveStart: () => this.onMoveStart(),
                onMove: (delta) => this.onResizeHandleMove(toMove, delta),
            });
        }
        onWillUnmount(() => this.props.onSaveTable(this.props.table));
    }

    onMoveStart() {
        if (this.pos.floorPlanStyle == "kanban") {
            return;
        }
        this.startTable = { ...this.props.table };
        this.selectedTablesCopy = {};
        for (let i = 0; i < this.props.selectedTables.length; i++) {
            this.selectedTablesCopy[i] = { ...this.props.selectedTables[i] };
        }
        // stop the next click event from the touch/click release from unselecting the table
        document.addEventListener("click", (ev) => ev.stopPropagation(), {
            capture: true,
            once: true,
        });
    }

    onMove({ dx, dy }) {
        if (this.pos.floorPlanStyle == "kanban") {
            return;
        }
        const { minX, minY, maxX, maxY } = getLimits(this.root.el, this.props.limit.el);

        for (const [index, table] of Object.entries(this.selectedTablesCopy)) {
            const position_h = table.position_h;
            const position_v = table.position_v;
            this.props.selectedTables[index].position_h = constrain(position_h + dx, minX, maxX);
            this.props.selectedTables[index].position_v = constrain(position_v + dy, minY, maxY);
        }

        this._setElementStyle();
    }

    onResizeHandleMove([moveX, moveY], { dx, dy }) {
        if (this.pos.floorPlanStyle == "kanban") {
            return;
        }
        // Working with min/max x and y makes constraints much easier to apply uniformly
        const { width, height, position_h: minX, position_v: minY } = this.startTable;
        const newTable = { minX, minY, maxX: minX + width, maxY: minY + height };

        const limits = getLimits(this.root.el, this.props.limit.el);
        const { width: elWidth, height: elHeight } = this.root.el.getBoundingClientRect();
        const bounds = {
            maxX: [minX + MIN_TABLE_SIZE, limits.maxX + elWidth],
            minX: [limits.minX, newTable.maxX - MIN_TABLE_SIZE],
            maxY: [minY + MIN_TABLE_SIZE, limits.maxY + elHeight],
            minY: [limits.minY, newTable.maxY - MIN_TABLE_SIZE],
        };
        newTable[moveX] = constrain(newTable[moveX] + dx, ...bounds[moveX]);
        newTable[moveY] = constrain(newTable[moveY] + dy, ...bounds[moveY]);

        // Convert back to server format at the end
        this.props.table.position_h = newTable.minX;
        this.props.table.position_v = newTable.minY;
        this.props.table.width = newTable.maxX - newTable.minX;
        this.props.table.height = newTable.maxY - newTable.minY;
        this._setElementStyle();
    }
    /**
     * Offsets the resize handles from the edge of the table. For square tables,
     * the offset is half the width of the handle (we just want a quarter circle
     * to be visible), for round tables it's half the width plus the distance of
     * the middle of the rounded border's arc to the edge.
     *
     * @param {`${'top'|'bottom'} ${'left'|'right'}`} handleName the handle for
     *  which to compute the style
     * @returns {string} the value of the style attribute for the given handle
     */
    computeHandleStyle(handleName) {
        const table = this.props.table;
        // 24 is half the handle's width
        let offset = -24;
        if (table.shape === "round") {
            // min(width/2, height/2) is the real border radius
            // 0.2929 is (1 - cos(45°)) to get in the middle of the border's arc
            offset += Math.min(table.width / 2, table.height / 2) * 0.2929;
        }
        return handleName
            .split(" ")
            .map((dir) => `${dir}: ${offset}px;`)
            .join(" ");
    }

    _setElementStyle() {
        const table = this.props.table;
        if (this.pos.floorPlanStyle == "kanban") {
            const floor = table.floor;
            const index = floor.tables.indexOf(table);
            const minWidth = 100 + 20;
            const nbrHorizontal = Math.floor(window.innerWidth / minWidth);
            const widthTable = (window.innerWidth - nbrHorizontal * 10) / nbrHorizontal;
            const position_h =
                widthTable * (index % nbrHorizontal) + 5 + (index % nbrHorizontal) * 10;
            const position_v =
                (widthTable + 25) * Math.floor(index / nbrHorizontal) +
                10 +
                Math.floor(index / nbrHorizontal) * 10;

            Object.assign(this.root.el.style, {
                left: `${position_h}px`,
                top: `${position_v}px`,
                width: `${widthTable}px`,
                height: `${widthTable}px`,
                background: table.color || "rgb(53, 211, 116)",
                "line-height": `${widthTable}px`,
                "border-radius": table.shape === "round" ? "1000px" : "3px",
                "font-size": widthTable >= 150 ? "32px" : "16px",
                opacity: "0.7",
            });
            return;
        }
        Object.assign(this.root.el.style, {
            left: `${table.position_h}px`,
            top: `${table.position_v}px`,
            width: `${table.width}px`,
            height: `${table.height}px`,
            background: table.color || "rgb(53, 211, 116)",
            "line-height": `${table.height}px`,
            "border-radius": table.shape === "round" ? "1000px" : "3px",
            "font-size": table.height >= 150 && table.width >= 150 ? "32px" : "16px",
        });
    }
}

```

## File: static\src\app\floor_screen\editable_table.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.EditableTable">
        <div class="table selected position-absolute d-flex flex-column align-items-center justify-content-between overflow-hidden" 
            t-ref="root">
            <span class="label drag-handle d-flex align-items-center flex-grow-1 d fw-bolder fs-2">
                <t t-esc="props.table.name" />
            </span>
            <t t-if="pos.floorPlanStyle != 'kanban'" t-foreach="handles" t-as="handle" t-key="handle">
                <span class="table-handle" t-ref="{{handle}}" t-att-style="computeHandleStyle(handle)"/>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\app\floor_screen\edit_bar.js

```javascript
/** @odoo-module */

import { Component, useExternalListener, useState } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { useTrackedAsync } from "@point_of_sale/app/utils/hooks";

export class EditBar extends Component {
    static template = "pos_restaurant.EditBar";
    static props = {
        selectedTables: Object,
        nbrFloors: Number,
        floorMapScrollTop: Number,
        isColorPicker: Boolean,
        toggleColorPicker: Function,
        createTable: Function,
        duplicateTableOrFloor: Function,
        renameTable: Function,
        changeSeatsNum: Function,
        changeToCircle: Function,
        changeToSquare: Function,
        setTableColor: Function,
        setFloorColor: Function,
        deleteFloorOrTable: Function,
        toggleEditMode: Function,
    };

    setup() {
        this.ui = useState(useService("ui"));
        useExternalListener(window, "click", this.onOutsideClick);
        this.doCreateTable = useTrackedAsync(this.props.createTable);
    }

    onOutsideClick() {
        if (this.props.isColorPicker) {
            this.props.isColorPicker = false;
        }
    }

    getSelectedTablesShape() {
        let shape = "round";
        this.props.selectedTables.forEach((table) => {
            if (table.shape == "square") {
                shape = "square";
            }
        });
        return shape;
    }
}

```

## File: static\src\app\floor_screen\edit_bar.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.EditBar">
        <div class="edit-bar-top d-flex align-items-center justify-content-between px-3 border-bottom bg-view overflow-x-auto h-54 w-100">
            <div class="flex-fill d-flex justify-content-center">
                <span role="button" class="edit-button text-center d-flex flex-column btn btn-light text-uppercase" t-on-click.stop="doCreateTable.call" t-att-disabled="doCreateTable.status === 'loading'">
                    <t t-if="doCreateTable.status === 'loading'">
                        <i class="fa fa-spinner fa-spin icon-button" role="img" aria-label="Loading" title="Loading"></i>
                    </t>
                    <t t-else="">
                        <i class="fa fa-plus icon-button" role="img" aria-label="Add" title="Add"></i>
                    </t>
                    <span class="text-button d-block">Table</span>
                </span>
                <span role="button" class="edit-button text-center d-flex flex-column btn btn-light text-uppercase" t-att-class="{ 'disabled': props.selectedTables.length == 0 }" t-on-click.stop="props.changeSeatsNum">
                    <i class="fa fa-user icon-button" role="img" aria-label="Seats" title="Seats"></i>
                    <span class="text-button d-block">Seats</span>
                </span>
                <span role="button" class="edit-button text-center d-flex flex-column btn btn-light text-uppercase button-option round" t-if="getSelectedTablesShape() == 'square'" t-att-class="{ disabled: props.selectedTables.length == 0 }" t-on-click.stop="props.changeToCircle">
                    <i class="fa fa-circle-o icon-button" role="img" aria-label="Round Shape" title="Round Shape"></i>
                    <span class="text-button d-block">Shape</span>
                </span>
                <span role="button" class="edit-button text-center d-flex flex-column btn btn-light text-uppercase button-option square" t-else="" t-att-class="{ disabled: props.selectedTables.length == 0 }" t-on-click.stop="props.changeToSquare">
                    <i class="fa fa-square-o icon-button" role="img" aria-label="Square Shape" title="Square Shape"></i>
                    <span class="text-button d-block">Shape</span>
                </span>
                <span t-if="!props.isColorPicker" role="button" class="edit-button text-center d-flex flex-column btn btn-light text-uppercase" t-on-click.stop="props.toggleColorPicker">
                    <i class="fa fa-paint-brush icon-button" role="img" aria-label="Tint" title="Tint"></i>
                    <span class="text-button d-block">Fill</span>
                </span>
                <span t-else="" role="button" class="edit-button text-center d-flex flex-column is-active btn btn-secondary text-uppercase" t-on-click.stop="props.toggleColorPicker">
                    <i class="fa fa-paint-brush icon-button" role="img" aria-label="Tint" title="Tint"></i>
                    <span class="text-button d-block">Fill</span>
                </span>
                <span role="button" class="edit-button text-center d-flex flex-column btn btn-light text-uppercase" t-att-class="{ disabled: props.selectedTables.length > 1 }" t-on-click.stop="props.renameTable">
                    <i class="fa fa-pencil-square-o icon-button" role="img" aria-label="Rename" title="Rename"></i>
                    <span class="text-button d-block">Rename</span>
                </span>
                <span role="button" class="edit-button text-center d-flex flex-column btn btn-light text-uppercase" t-on-click.stop="props.duplicateTableOrFloor">
                    <i class="fa fa-clone icon-button" role="img" aria-label="Copy" title="Copy"></i>
                    <span class="text-button d-block">Copy</span>
                </span>
                <span role="button" class="edit-button text-center d-flex flex-column trash btn btn-light text-uppercase" t-on-click.stop="props.deleteFloorOrTable">
                    <i class="fa fa-trash icon-button" role="img" aria-label="Delete" title="Delete"></i>
                    <span class="text-button d-block">Delete</span>
                </span>
            </div>
            <span class="edit-button d-flex flex-row-reverse border-0 flex-shrink-1 text-uppercase text-end last-edit-button">
                <div class="d-flex justify-content-end">
                    <div role="button" class="btn btn-light d-flex flex-column align-items-center close-edit-button" t-on-click.stop="props.toggleEditMode">
                        <i class="fa fa-times icon-button" role="img" aria-label="Close" title="Close"></i>
                        <span class="text-button d-block">Close</span>
                    </div>
                </div>
            </span>
        </div>
        <div t-if="props.isColorPicker and props.selectedTables.length > 0" class="color-picker d-flex justify-content-center align-items-center bg-200 h-54">
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#FFFFFF" role="img" aria-label="White" title="White" t-on-click.stop="() => props.setTableColor('#FFFFFF')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#EB6D6D" role="img" aria-label="Red" title="Red" t-on-click.stop="() => props.setTableColor('#EB6D6D')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#35D374" role="img" aria-label="Green" title="Green" t-on-click.stop="() => props.setTableColor('#35D374')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#6C6DEC" role="img" aria-label="Blue" title="Blue" t-on-click.stop="() => props.setTableColor('#6C6DEC')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#EBBF6D" role="img" aria-label="Orange" title="Orange" t-on-click.stop="() => props.setTableColor('#EBBF6D')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#EBEC6D" role="img" aria-label="Yellow" title="Yellow" t-on-click.stop="() => props.setTableColor('#EBEC6D')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#AC6DAD" role="img" aria-label="Purple" title="Purple" t-on-click.stop="() => props.setTableColor('#AC6DAD')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#6C6D6D" role="img" aria-label="Grey" title="Grey" t-on-click.stop="() => props.setTableColor('#6C6D6D')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#ACADAD" role="img" aria-label="Light grey" title="Light grey" t-on-click.stop="() => props.setTableColor('#ACADAD')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:#4ED2BE" role="img" aria-label="Turquoise" title="Turquoise" t-on-click.stop="() => props.setTableColor('#4ED2BE')" />
        </div>
        <div t-if="props.isColorPicker and props.selectedTables.length == 0" class="color-picker d-flex justify-content-center align-items-center bg-200 h-54">    
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(249, 250, 251)" role="img" aria-label="White" title="White" t-on-click.stop="() => props.setFloorColor('rgb(249, 250, 251)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(244, 149, 149)" role="img" aria-label="Red" title="Red" t-on-click.stop="() => props.setFloorColor('rgb(244, 149, 149)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(130, 233, 171)" role="img" aria-label="Green" title="Green" t-on-click.stop="() => props.setFloorColor('rgb(130, 233, 171)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(136, 137, 242)" role="img" aria-label="Blue" title="Blue" t-on-click.stop="() => props.setFloorColor('rgb(136, 137, 242)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(255, 214, 136)" role="img" aria-label="Orange" title="Orange" t-on-click.stop="() => props.setFloorColor('rgb(255, 214, 136)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(254, 255, 154)" role="img" aria-label="Yellow" title="Yellow" t-on-click.stop="() => props.setFloorColor('rgb(254, 255, 154)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(209, 171, 210)" role="img" aria-label="Purple" title="Purple" t-on-click.stop="() => props.setFloorColor('rgb(209, 171, 210)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(75, 75, 75)"    role="img" aria-label="Grey" title="Grey" t-on-click.stop="() => props.setFloorColor('rgb(75, 75, 75)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(210, 210, 210)" role="img" aria-label="Light grey" title="Light grey" t-on-click.stop="() => props.setFloorColor('rgb(210, 210, 210)')" />
            <span class="color py-4 cursor-pointer float-start"  style="background-color:rgb(127, 221, 236)" role="img" aria-label="Turquoise" title="Turquoise" t-on-click.stop="() => props.setFloorColor('rgb(127, 221, 236)')" />
        </div>
    </t>

</templates>

```

## File: static\src\app\floor_screen\floor_screen.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";
import { ConnectionLostError } from "@web/core/network/rpc_service";
import { debounce } from "@web/core/utils/timing";
import { registry } from "@web/core/registry";

import { TextInputPopup } from "@point_of_sale/app/utils/input_popups/text_input_popup";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { ConfirmPopup } from "@point_of_sale/app/utils/confirm_popup/confirm_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";

import { EditableTable } from "@pos_restaurant/app/floor_screen/editable_table";
import { EditBar } from "@pos_restaurant/app/floor_screen/edit_bar";
import { Table } from "@pos_restaurant/app/floor_screen/table";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { useService } from "@web/core/utils/hooks";
import {
    Component,
    onPatched,
    onMounted,
    onWillUnmount,
    useRef,
    useState,
    onWillStart,
} from "@odoo/owl";

export class FloorScreen extends Component {
    static components = { EditableTable, EditBar, Table };
    static template = "pos_restaurant.FloorScreen";
    static props = { isShown: Boolean, floor: { type: true, optional: true } };
    static storeOnOrder = false;

    setup() {
        this.pos = usePos();
        this.popup = useService("popup");
        this.orm = useService("orm");
        const floor = this.pos.currentFloor;
        this.state = useState({
            selectedFloorId: floor ? floor.id : null,
            selectedTableIds: [],
            floorBackground: floor ? floor.background_color : null,
            floorMapScrollTop: 0,
            isColorPicker: false,
        });
        this.floorMapRef = useRef("floor-map-ref");
        this.addFloorRef = useRef("add-floor-ref");
        this.map = useRef("map");
        onPatched(this.onPatched);
        onMounted(this.onMounted);
        onWillUnmount(this.onWillUnmount);
        onWillStart(this.onWillStart);
    }
    onPatched() {
        this.floorMapRef.el.style.background = this.state.floorBackground;
        if (!this.pos.isEditMode && this.pos.floors.length > 0) {
            this.addFloorRef.el.style.display = "none";
        } else {
            this.addFloorRef.el.style.display = "initial";
        }
        this.state.floorMapScrollTop = this.floorMapRef.el.getBoundingClientRect().top;
    }
    async onWillStart() {
        this.pos.searchProductWord = "";
        const table = this.pos.table;
        if (table) {
            const orders = this.pos.get_order_list();
            const tableOrders = orders.filter(
                (order) => order.tableId === table.id && !order.finalized
            );
            const qtyChange = tableOrders.reduce(
                (acc, order) => {
                    const quantityChange = order.getOrderChanges();
                    const quantitySkipped = order.getOrderChanges(true);
                    acc.changed += quantityChange.count;
                    acc.skipped += quantitySkipped.count;
                    return acc;
                },
                { changed: 0, skipped: 0 }
            );

            table.changes_count = qtyChange.changed;
            table.skip_changes = qtyChange.skipped;
        }
        await this.pos.unsetTable();
    }
    onMounted() {
        this.pos.openCashControl();
        this.floorMapRef.el.style.background = this.state.floorBackground;
        if (!this.pos.isEditMode && this.pos.floors.length > 0) {
            this.addFloorRef.el.style.display = "none";
        } else {
            this.addFloorRef.el.style.display = "initial";
        }
        this.state.floorMapScrollTop = this.floorMapRef.el.getBoundingClientRect().top;
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
            callbackFunction(Math.hypot(deltaX, deltaY));
        }
    }
    _onPinchStart(ev) {
        ev.currentTarget.style.setProperty("touch-action", "none");
        this._computePinchHypo(ev, this.startPinch.bind(this));
    }
    _onPinchEnd(ev) {
        ev.currentTarget.style.removeProperty("touch-action");
    }
    _onPinchMove(ev) {
        debounce(this._computePinchHypo, 10, true)(ev, this.movePinch.bind(this));
    }
    _onDeselectTable() {
        this.state.selectedTableIds = [];
    }
    async _createTableHelper(copyTable, duplicateFloor = false) {
        const existingTable = this.activeFloor.tables;
        let newTable;
        if (copyTable) {
            newTable = Object.assign({}, copyTable);
            if (!duplicateFloor) {
                newTable.position_h += 10;
                newTable.position_v += 10;
            }
            delete newTable.id;
            newTable.order_count = 0;
        } else {
            let posV = 0;
            let posH = 10;
            const referenceScreenWidth = 1180;
            const spaceBetweenTable = 15 * (screen.width / referenceScreenWidth);
            const h_min = spaceBetweenTable;
            const h_max = screen.width;
            const v_max = screen.height;
            let potentialWidth = 100 * (h_max / referenceScreenWidth);
            if (potentialWidth > 130) {
                potentialWidth = 130;
            } else if (potentialWidth < 75) {
                potentialWidth = 75;
            }
            const heightTable = potentialWidth;
            const widthTable = potentialWidth;
            const positionTable = [];

            existingTable.forEach((table) => {
                positionTable.push([
                    table.position_v,
                    table.position_v + table.height,
                    table.position_h,
                    table.position_h + table.width,
                ]);
            });

            positionTable.sort((tableA, tableB) => {
                if (tableA[0] < tableB[0]) {
                    return -1;
                } else if (tableA[0] > tableB[0]) {
                    return 1;
                } else if (tableA[2] < tableB[2]) {
                    return -1;
                } else {
                    return 1;
                }
            });

            let actualHeight = 100;
            let impossible = true;

            while (actualHeight <= v_max - heightTable - spaceBetweenTable && impossible) {
                const tableIntervals = [
                    [h_min, h_min, v_max],
                    [h_max, h_max, v_max],
                ];
                for (let i = 0; i < positionTable.length; i++) {
                    if (positionTable[i][0] >= actualHeight + heightTable + spaceBetweenTable) {
                        continue;
                    } else if (positionTable[i][1] + spaceBetweenTable <= actualHeight) {
                        continue;
                    } else {
                        tableIntervals.push([
                            positionTable[i][2],
                            positionTable[i][3],
                            positionTable[i][1],
                        ]);
                    }
                }

                tableIntervals.sort((a, b) => {
                    if (a[0] < b[0]) {
                        return -1;
                    } else if (a[0] > b[0]) {
                        return 1;
                    } else if (a[1] < b[1]) {
                        return -1;
                    } else {
                        return 1;
                    }
                });

                let nextHeight = v_max;
                for (let i = 0; i < tableIntervals.length - 1; i++) {
                    if (tableIntervals[i][2] < nextHeight) {
                        nextHeight = tableIntervals[i][2];
                    }

                    if (
                        tableIntervals[i + 1][0] - tableIntervals[i][1] >
                        widthTable + spaceBetweenTable
                    ) {
                        impossible = false;
                        posV = actualHeight;
                        posH = tableIntervals[i][1] + spaceBetweenTable;
                        break;
                    }
                }
                actualHeight = nextHeight + spaceBetweenTable;
            }

            if (impossible) {
                posV = positionTable[0][0] + 10;
                posH = positionTable[0][2] + 10;
            }

            newTable = {
                position_v: posV,
                position_h: posH,
                width: widthTable,
                height: heightTable,
                shape: "square",
                seats: 2,
                color: "rgb(53, 211, 116)",
            };
        }
        if (!duplicateFloor) {
            newTable.name = this._getNewTableName();
        }
        newTable.floor_id = [this.activeFloor.id, ""];
        newTable.floor = this.activeFloor;
        await this._save(newTable);
        this.activeTables.push(newTable);
        this.activeFloor.table_ids.push(newTable.id);
        return newTable;
    }
    _getNewTableName() {
        let firstNum = 1;
        const tablesNameNumber = this.activeTables
            .map((table) => +table.name)
            .sort(function (a, b) {
                return a - b;
            });

        for (let i = 0; i < tablesNameNumber.length; i++) {
            if (tablesNameNumber[i] == firstNum) {
                firstNum += 1;
            } else {
                break;
            }
        }
        return firstNum.toString();
    }
    async _save(table) {
        const tableCopy = {
            floor_id: table.floor.id,
            color: table.color,
            height: table.height,
            name: table.name,
            position_h: table.position_h,
            position_v: table.position_v,
            seats: table.seats,
            shape: table.shape,
            width: table.width,
        };

        if (table.id) {
            await this.orm.write("restaurant.table", [table.id], tableCopy);
        } else {
            const tableId = await this.orm.create("restaurant.table", [tableCopy]);

            table.id = tableId[0];
            this.pos.tables_by_id[tableId] = table;
        }
    }
    async _renameFloor(floorId, newName) {
        await this.orm.call("restaurant.floor", "rename_floor", [floorId, newName]);
    }
    get activeFloor() {
        return this.state.selectedFloorId
            ? this.pos.floors_by_id[this.state.selectedFloorId]
            : null;
    }
    get activeTables() {
        return this.activeFloor ? this.activeFloor.tables : null;
    }
    get isFloorEmpty() {
        return this.activeTables ? this.activeTables.length === 0 : true;
    }
    get selectedTables() {
        const tables = [];
        this.state.selectedTableIds.forEach((id) => {
            tables.push(this.pos.tables_by_id[id]);
        });
        return tables;
    }
    get nbrFloors() {
        return this.pos.floors.length;
    }
    movePinch(hypot) {
        const delta = hypot / this.scalehypot;
        const value = this.initalScale * delta;
        this.setScale(value);
    }
    startPinch(hypot) {
        this.scalehypot = hypot;
        this.initalScale = this.getScale();
    }
    getScale() {
        const scale = this.map.el.style.getPropertyValue("--scale");
        const parsedScaleValue = parseFloat(scale);
        return isNaN(parsedScaleValue) ? 1 : parsedScaleValue;
    }
    setScale(value) {
        // a scale can't be a negative number
        if (value > 0) {
            this.map.el.style.setProperty("--scale", value);
        }
    }
    selectFloor(floor) {
        this.pos.currentFloor = floor;
        this.state.selectedFloorId = floor.id;
        this.state.floorBackground = this.activeFloor.background_color;
        this.state.selectedTableIds = [];
    }
    toggleEditMode() {
        this.pos.toggleEditMode();
        if (!this.pos.isEditMode && this.pos.floors.length > 0) {
            this.addFloorRef.el.style.display = "none";
        } else {
            this.addFloorRef.el.style.display = "initial";
        }
        this.state.selectedTableIds = [];
    }
    async onSelectTable(table, ev) {
        if (this.pos.isEditMode) {
            if (ev.ctrlKey || ev.metaKey) {
                this.state.selectedTableIds.push(table.id);
            } else {
                this.state.selectedTableIds = [];
                this.state.selectedTableIds.push(table.id);
            }
        } else {
            if (this.pos.orderToTransfer && table.order_count > 0) {
                const { confirmed } = await this.popup.add(ConfirmPopup, {
                    title: _t("Table is not empty"),
                    body: _t(
                        "The table already contains an order. Do you want to proceed and transfer the order here?"
                    ),
                    confirmText: _t("Yes"),
                });
                if (!confirmed) {
                    // We don't want to change the table if the transfer is not done.
                    table = this.pos.tables_by_id[this.pos.orderToTransfer.tableId];
                    this.pos.orderToTransfer = null;
                }
            }
            if (this.pos.orderToTransfer) {
                await this.pos.transferTable(table);
            } else {
                try {
                    await this.pos.setTable(table);
                } catch (e) {
                    if (!(e instanceof ConnectionLostError)) {
                        throw e;
                    }
                    // Reject error in a separate stack to display the offline popup, but continue the flow
                    Promise.reject(e);
                }
            }
            const order = this.pos.get_order();
            this.pos.showScreen(order.get_screen_data().name);
        }
    }
    async onSaveTable(table) {
        if (this.pos.tables_by_id[table.id] && this.pos.tables_by_id[table.id].active) {
            await this._save(table);
        }
    }
    async addFloor() {
        const { confirmed, payload: newName } = await this.popup.add(TextInputPopup, {
            title: _t("New Floor"),
            placeholder: _t("Floor name"),
        });
        if (!confirmed) {
            return;
        }
        const floor = await this.orm.call("restaurant.floor", "create_from_ui", [
            newName,
            "#ACADAD",
            this.pos.config.id,
        ]);
        this.pos.floors_by_id[floor.id] = floor;
        this.pos.floors.push(floor);
        this.selectFloor(floor);
        this.pos.isEditMode = true;
    }
    async createTable() {
        const newTable = await this._createTableHelper();
        newTable.skip_changes = 0;
        newTable.changes_count = 0;
        newTable.order_count = 0;
        if (newTable) {
            this.state.selectedTableIds = [];
            this.state.selectedTableIds.push(newTable.id);
        }
    }
    async duplicateTableOrFloor() {
        if (this.selectedTables.length == 0) {
            const floor = this.activeFloor;
            const tables = this.activeFloor.tables;
            const newFloorName = floor.name + " (copy)";
            const newFloor = await this.orm.call("restaurant.floor", "create_from_ui", [
                newFloorName,
                floor.background_color,
                this.pos.config.id,
            ]);
            this.pos.floors_by_id[newFloor.id] = newFloor;
            this.pos.floors.push(newFloor);
            this.selectFloor(newFloor);
            for (const table of tables) {
                await this._createTableHelper(table, true);
            }
            return;
        }
        const selectedTables = this.selectedTables;
        this._onDeselectTable();

        for (const table of selectedTables) {
            const newTable = await this._createTableHelper(table);
            if (newTable) {
                this.state.selectedTableIds.push(newTable.id);
            }
        }
    }
    async renameTable() {
        const selectedTables = this.selectedTables;
        const selectedFloor = this.activeFloor;
        if (selectedTables.length > 1) {
            return;
        }
        if (selectedTables.length == 0) {
            const { confirmed, payload: newName } = await this.popup.add(TextInputPopup, {
                startingValue: selectedFloor.name,
                title: _t("Floor Name ?"),
            });
            if (!confirmed) {
                return;
            }
            if (newName !== selectedFloor.name) {
                selectedFloor.name = newName;
                await this._renameFloor(selectedFloor.id, newName);
            }
            return;
        }
        const selectedTable = selectedTables[0];
        const { confirmed, payload: newName } = await this.popup.add(TextInputPopup, {
            startingValue: selectedTable.name,
            title: _t("Table Name?"),
        });
        if (!confirmed) {
            return;
        }
        if (newName !== selectedTable.name) {
            selectedTable.name = newName;
            await this._save(selectedTable);
        }
    }
    async changeSeatsNum() {
        const selectedTables = this.selectedTables;
        if (selectedTables.length == 0) {
            return;
        }
        const { confirmed, payload: inputNumber } = await this.popup.add(NumberPopup, {
            startingValue: 0,
            cheap: true,
            title: _t("Number of Seats?"),
            isInputSelected: true,
        });
        if (!confirmed) {
            return;
        }
        const newSeatsNum = parseInt(inputNumber, 10);
        selectedTables.forEach(async (selectedTable) => {
            if (newSeatsNum !== selectedTable.seats) {
                selectedTable.seats = newSeatsNum;
                await this._save(selectedTable);
            }
        });
    }
    async changeToCircle() {
        await this.changeShape("round");
    }
    async changeToSquare() {
        await this.changeShape("square");
    }
    async changeShape(form) {
        if (this.selectedTables.length == 0) {
            return;
        }
        this.selectedTables.forEach(async (selectedTable) => {
            selectedTable.shape = form;
            await this._save(selectedTable);
        });
    }
    async setTableColor(color) {
        const selectedTables = this.selectedTables;
        selectedTables.forEach(async (selectedTable) => {
            selectedTable.color = color;
            await this._save(selectedTable);
        });
        this.state.isColorPicker = false;
    }
    async setFloorColor(color) {
        this.state.floorBackground = color;
        this.activeFloor.background_color = color;
        await this.orm.write("restaurant.floor", [this.activeFloor.id], {
            background_color: color,
        });
        this.state.isColorPicker = false;
    }
    toggleColorPicker() {
        this.state.isColorPicker = !this.state.isColorPicker;
    }
    async deleteFloorOrTable() {
        if (this.selectedTables.length == 0) {
            const { confirmed } = await this.popup.add(ConfirmPopup, {
                title: `Removing floor ${this.activeFloor.name}`,
                body: sprintf(
                    _t("Removing a floor cannot be undone. Do you still want to remove %s?"),
                    this.activeFloor.name
                ),
            });
            if (!confirmed) {
                return;
            }
            const originalSelectedFloorId = this.activeFloor.id;
            await this.orm.call("restaurant.floor", "deactivate_floor", [
                originalSelectedFloorId,
                this.pos.pos_session.id,
            ]);
            const floor = this.pos.floors_by_id[originalSelectedFloorId];
            const orderList = [...this.pos.get_order_list()];
            for (const order of orderList) {
                if (floor.table_ids.includes(order.tableId)) {
                    this.pos.removeOrder(order, false);
                }
            }
            floor.table_ids.forEach((tableId) => {
                delete this.pos.tables_by_id[tableId];
            });
            delete this.pos.floors_by_id[originalSelectedFloorId];
            this.pos.floors = this.pos.floors.filter(
                (floor) => floor.id != originalSelectedFloorId
            );
            this.pos.TICKET_SCREEN_STATE.syncedOrders.cache = {};
            if (this.pos.floors.length > 0) {
                this.selectFloor(this.pos.floors[0]);
            } else {
                this.pos.isEditMode = false;
                this.pos.floorPlanStyle = "default";
                this.state.floorBackground = null;
            }
            return;
        }
        const { confirmed } = await this.popup.add(ConfirmPopup, {
            title: _t("Are you sure?"),
            body: _t("Removing a table cannot be undone"),
        });
        if (!confirmed) {
            return;
        }
        const originalSelectedTableIds = [...this.state.selectedTableIds];
        const response = await this.orm.call("restaurant.table", "are_orders_still_in_draft", [
            originalSelectedTableIds,
        ]);
        if (!response) {
            for (const id of originalSelectedTableIds) {
                //remove order not send to server
                for (const order of this.pos.get_order_list()) {
                    if (order.tableId == id) {
                        this.pos.removeOrder(order, false);
                    }
                }
                this.pos.tables_by_id[id].active = false;
                this.orm.write("restaurant.table", [id], { active: false });
                this.activeFloor.tables = this.activeTables.filter((table) => table.id !== id);
                delete this.pos.tables_by_id[id];
            }
        } else {
            await this.popup.add(ErrorPopup, {
                title: _t("Delete Error"),
                body: _t("You cannot delete a table with orders still in draft for this table."),
            });
        }
        // Value of an object can change inside async function call.
        //   Which means that in this code block, the value of `state.selectedTableId`
        //   before the await call can be different after the finishing the await call.
        // Since we wanted to disable the selected table after deletion, we should be
        //   setting the selectedTableId to null. However, we only do this if nothing
        //   else is selected during the rpc call.
        const equalsCheck = (a, b) => {
            return JSON.stringify(a) === JSON.stringify(b);
        };
        if (equalsCheck(this.state.selectedTableIds, originalSelectedTableIds)) {
            this.state.selectedTableIds = [];
        }
        this.pos.TICKET_SCREEN_STATE.syncedOrders.cache = {};
    }
}

registry.category("pos_screens").add("FloorScreen", FloorScreen);

```

## File: static\src\app\floor_screen\floor_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.FloorScreen">
        <div class="floor-screen screen h-100 position-relative d-flex flex-column flex-nowrap m-0 bg-100 text-start overflow-hidden">
            <EditBar t-if="pos.isEditMode" selectedTables="selectedTables" nbrFloors="nbrFloors"
                        floorMapScrollTop="state.floorMapScrollTop" isColorPicker="state.isColorPicker" toggleColorPicker.bind="toggleColorPicker"
                        createTable.bind="createTable" duplicateTableOrFloor.bind="duplicateTableOrFloor" renameTable.bind="renameTable"
                        changeSeatsNum.bind="changeSeatsNum" changeToCircle.bind="changeToCircle" changeToSquare.bind="changeToSquare"
                        setTableColor.bind="setTableColor" setFloorColor.bind="setFloorColor" deleteFloorOrTable.bind="deleteFloorOrTable"
                        toggleEditMode.bind="toggleEditMode"
            />
            <div class="floor-selector d-flex text-center bg-100 fs-3 w-100 h-54">
                <t t-foreach="pos.floors" t-as="floor" t-key="floor.id">
                    <button class="button button-floor btn p-3 rounded-0 flex-fill border-start shadow-none d-flex align-items-center justify-content-center" t-attf-class="{{ floor.id === state.selectedFloorId ? 'btn-primary border-start-0' : 'btn-light' }}" t-on-click="() => this.selectFloor(floor)">
                        <t t-esc="floor.name" />
                        <span t-if="activeFloor.id !== floor.id and floor.changes_count > 0" class="mx-1 badge bg-danger text-white rounded-pill py-1 px-2 fs-5" t-esc="floor.changes_count"/>
                    </button>
                </t>
                <button class="button button-add btn btn-secondary p-3 ms-auto rounded-0" t-ref="add-floor-ref" t-on-click="addFloor">
                    <i class="fa fa-plus icon-button me-2" role="img" aria-label="Add" title="Add"></i>
                    Add Floor
                </button>
            </div>

            <div
                t-on-click="_onDeselectTable"
                t-on-touchstart="_onPinchStart"
                t-on-touchmove="_onPinchMove"
                t-on-touchend="_onPinchEnd"
                class="floor-map position-relative flex-grow-1 flex-shrink-1 flex-basis-0 w-auto h-100 overflow-auto"
                t-ref="floor-map-ref"
            >
                <t t-if="pos.floors.length > 0">
                    <div t-if="isFloorEmpty" class="empty-floor d-flex align-items-center justify-content-center h-100 fs-3 text-center text-muted" t-ref="map">
                        <span>Oops! No tables available.<br/>Add a new table to get started.</span>
                    </div>
                    <div t-else="" t-ref="map">
                        <t t-foreach="activeTables" t-as="table" t-key="table.id">
                            <Table t-if="!state.selectedTableIds.includes(table.id)" onClick.bind="onSelectTable" table="table" />
                            <EditableTable t-else="" table="table" selectedTables="selectedTables" onSaveTable.bind="onSaveTable" limit="floorMapRef" />
                        </t>
                    </div>
                </t>
                <t t-else="">
                    <div class="empty-floor d-flex align-items-center justify-content-center h-100 fs-3 text-center text-muted" t-ref="map">
                        <span>Oops! No floors available.<br/>Add a new floor to get started.</span>
                    </div>
                </t>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\floor_screen\table.js

```javascript
/** @odoo-module */

import { Component, useState } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

export class Table extends Component {
    static template = "pos_restaurant.Table";
    static props = {
        onClick: Function,
        table: {
            type: Object,
            shape: {
                position_h: Number,
                position_v: Number,
                width: Number,
                height: Number,
                shape: String,
                color: [String, { value: false }],
                name: String,
                seats: Number,
                "*": true,
            },
        },
    };

    setup() {
        this.pos = usePos();
        this.state = useState({
            containerHeight: 0,
            containerWidth: 0,
        });
    }
    get fontSize() {
        const size = this.state.containerHeight / 3;
        return size > 20 ? 20 : size;
    }
    get badgeStyle() {
        if (this.props.table.shape !== "round") {
            return `top: -6px; right: -6px;`;
        }

        const tableHeight = this.state.containerHeight;
        const tableWidth = this.state.containerWidth;
        const radius = Math.min(tableWidth, tableHeight) / 2;

        let left = 0;
        let bottom = 0;

        if (tableHeight > tableWidth) {
            left = radius;
            bottom = radius + (tableHeight - tableWidth);
        } else {
            bottom = radius;
            left = radius + (tableWidth - tableHeight);
        }

        bottom += 0.7 * radius - 8;
        left += 0.7 * radius - 8;

        return `bottom: ${bottom}px; left: ${left}px;`;
    }
    computePosition(index, nbrHorizontal, widthTable) {
        const position_h = widthTable * (index % nbrHorizontal) + 5 + (index % nbrHorizontal) * 10;
        const position_v =
            widthTable * Math.floor(index / nbrHorizontal) +
            10 +
            Math.floor(index / nbrHorizontal) * 10;
        return { position_h, position_v };
    }
    get style() {
        const table = this.props.table;
        let style = "";
        let background = table.color ? table.color : "rgb(53, 211, 116)";
        let textColor = "white";

        if (!this.isOccupied()) {
            background = "#00000020";
            const rgb = table.floor.background_color
                .substring(4, table.floor.background_color.length - 1)
                .replace(/ /g, "")
                .split(",");
            textColor =
                (0.299 * rgb[0] + 0.587 * rgb[1] + 0.114 * rgb[2]) / 255 > 0.5 ? "black" : "white";
        }

        style += `
            border: 3px solid ${table.color};
            border-radius: ${table.shape === "round" ? 1000 : 3}px;
            background: ${background};
            box-shadow: 0px 3px rgba(0,0,0,0.07);
            padding: ${table.shape === "round" ? "4px 10px" : "4px 8px"};
            color: ${textColor};`;

        if (this.pos.floorPlanStyle == "kanban") {
            const floor = table.floor;
            const index = floor.tables.indexOf(table);
            const minWidth = 120;
            const nbrHorizontal = Math.floor(window.innerWidth / minWidth);
            const widthTable = (window.innerWidth - nbrHorizontal * 10) / nbrHorizontal;
            const { position_h, position_v } = this.computePosition(
                index,
                nbrHorizontal,
                widthTable
            );

            this.state.containerHeight = widthTable;
            this.state.containerWidth = widthTable;

            style += `
                width: ${widthTable}px;
                height: ${widthTable}px;
                top: ${position_v}px;
                left: ${position_h}px;
            `;
        } else {
            this.state.containerHeight = table.height;
            this.state.containerWidth = table.width;

            style += `
                width: ${table.width}px;
                height: ${table.height}px;
                top: ${table.position_v}px;
                left: ${table.position_h}px;
            `;
        }

        style += `
            font-size: ${this.fontSize}px;
            line-height: ${this.fontSize}px;`;

        return style;
    }
    get fill() {
        const customerCount = this.pos.getCustomerCount(this.props.table.id);
        return Math.min(1, Math.max(0, customerCount / this.props.table.seats));
    }
    get orderCount() {
        const table = this.props.table;
        const unsynced_orders = this.pos.getTableOrders(table.id).filter(
            (o) =>
                o.server_id === undefined &&
                (o.orderlines.length !== 0 || o.paymentlines.length !== 0) &&
                // do not count the orders that are already finalized
                !o.finalized
        );
        let result;
        if (table.changes_count > 0) {
            result = table.changes_count;
        } else if (table.skip_changes > 0) {
            result = table.skip_changes;
        } else {
            result = table.order_count + unsynced_orders.length;
        }
        return !Number.isNaN(result) ? result : 0;
    }
    get orderCountClass() {
        const notifications = this._getNotifications();
        const countClass = {
            "order-count": true,
            "notify-printing text-bg-danger": notifications.printing,
            "notify-skipped text-bg-info": notifications.skipped,
            "text-bg-dark": !notifications.printing && !notifications.skipped,
        };
        return countClass;
    }
    get customerCountDisplay() {
        const customerCount = this.pos.getCustomerCount(this.props.table.id);
        if (customerCount == 0) {
            return `${this.props.table.seats}`;
        } else {
            return `${customerCount}/${this.props.table.seats}`;
        }
    }
    _getNotifications() {
        const table = this.props.table;

        const hasChangesCount = table.changes_count;
        const hasSkippedCount = table.skip_changes;

        return hasChangesCount ? { printing: true } : hasSkippedCount ? { skipped: true } : {};
    }
    isOccupied() {
        return (
            this.pos.getCustomerCount(this.props.table.id) > 0 || this.props.table.order_count > 0
        );
    }
}

```

## File: static\src\app\floor_screen\table.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.Table">
        <div class="table position-absolute d-flex flex-column align-items-center justify-content-between cursor-pointer" 
            t-on-click.stop="(ev) => props.onClick(props.table, ev)" 
            t-att-style="style">
            <div class="infos d-flex align-items-center flex-grow-1">
                <span class="label fw-bolder fs-4"
                    t-esc="props.table.name" />
                <span 
                    t-att-class="orderCountClass" 
                    class="badge d-flex align-items-center justify-content-center position-absolute rounded-pill" 
                    t-attf-class="{{ orderCount === 0 or pos.isEditMode ? 'd-none' : ''}}"
                    t-att-style="badgeStyle"
                    t-esc="this.env.utils.formatProductQty(orderCount, false)"/>
            </div>
            <span class="table-seats position-absolute bottom-0 start-50 translate-middle-x mb-1 px-2 py-1 rounded text-bg-dark bg-opacity-25 fs-4">
                <div class="cover" t-att-style="`width: ${Math.ceil(fill * 100)}%`" />
                <t t-esc="customerCountDisplay" />
            </span>
        </div>
    </t>
</templates>

```

## File: static\src\app\product_screen\product_screen.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";

patch(ProductScreen.prototype, {
    bookTable() {
        this.pos.get_order().setBooked(true);
    },
    showBookButton() {
        return (
            this.pos.config.module_pos_restaurant &&
            this.pos.table &&
            !this.pos.orders.some(
                (o) => o.tableId === this.pos.table.id && o.finalized === false && o.isBooked
            )
        );
    },
});

```

## File: static\src\app\product_screen\product_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.ProductScreen" t-inherit="point_of_sale.ProductScreen" t-inherit-mode="extension">
        <xpath expr="//OrderWidget/t[@t-set-slot='details']" position="inside">
            <button t-if="showBookButton()" class="btn btn-primary py-2 rounded-0 book-table" style="border:none; font-size: 20px;" t-on-click="bookTable">Book table</button>
        </xpath>
        <xpath expr="//OrderWidget" position="attributes">
            <attribute name="isConfigRestaurant">pos.config.module_pos_restaurant</attribute>
            <attribute name="isOrderBooked">currentOrder.isBooked</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\app\split_bill_screen\split_bill_screen.js

```javascript
/** @odoo-module */

import { Order } from "@point_of_sale/app/store/models";

import { registry } from "@web/core/registry";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { Component, useState } from "@odoo/owl";
import { Orderline } from "@point_of_sale/app/generic_components/orderline/orderline";
import { OrderWidget } from "@point_of_sale/app/generic_components/order_widget/order_widget";
import { groupBy } from "@web/core/utils/arrays";

export class SplitBillScreen extends Component {
    static template = "pos_restaurant.SplitBillScreen";
    static components = { Orderline, OrderWidget };

    setup() {
        this.pos = usePos();
        this.splitlines = useState(this._initSplitLines(this.pos.get_order()));
        this.newOrderLines = {};
        this.newOrder = undefined;
        this._isFinal = false;
        this.newOrder = new Order(
            { env: this.env },
            {
                pos: this.pos,
                temporary: true,
            }
        );
    }
    get currentOrder() {
        return this.pos.get_order();
    }
    get orderlines() {
        return this.currentOrder.get_orderlines();
    }
    onClickLine(line) {
        for (const l of line.getAllLinesInCombo()) {
            this._splitQuantity(l);
            this._updateNewOrder(l);
        }
    }
    getLineData(line) {
        const splitQty = this.splitlines[line.id].quantity;
        if (!splitQty) {
            return line.getDisplayData();
        }
        return { ...line.getDisplayData(), qty: `${splitQty} / ${line.get_quantity_str()}` };
    }
    back() {
        this.pos.showScreen("ProductScreen");
    }
    proceed() {
        if (Object.keys(this.splitlines || {})?.length === 0) {
            // Splitlines is empty
            return;
        }

        this._isFinal = true;
        delete this.newOrder.temporary;

        if (!this._isFullPayOrder()) {
            this._setQuantityOnCurrentOrder();

            this.newOrder.set_screen_data({ name: "PaymentScreen" });

            // for the kitchen printer we assume that everything
            // has already been sent to the kitchen before splitting
            // the bill. So we save all changes both for the old
            // order and for the new one. This is not entirely correct
            // but avoids flooding the kitchen with unnecessary orders.
            // Not sure what to do in this case.
            if (this.pos.orderPreparationCategories.size) {
                this.currentOrder.updateLastOrderChange();
                this.newOrder.updateLastOrderChange();
            }

            this.newOrder.setCustomerCount(1);
            this.newOrder.originalSplittedOrder = this.currentOrder;
            const newCustomerCount = this.currentOrder.getCustomerCount() - 1;
            this.currentOrder.setCustomerCount(newCustomerCount || 1);
            this.currentOrder.set_screen_data({ name: "ProductScreen" });

            const reactiveNewOrder = this.pos.makeOrderReactive(this.newOrder);
            this.pos.orders.add(reactiveNewOrder);
            this.pos.selectedOrder = reactiveNewOrder;
        }
        this.pos.showScreen("PaymentScreen");
    }
    /**
     * @param {models.Order} order
     * @returns {Object<{ quantity: number }>} splitlines
     */
    _initSplitLines(order) {
        const splitlines = {};
        for (const line of order.get_orderlines()) {
            splitlines[line.id] = { product: line.get_product().id, quantity: 0 };
        }
        return splitlines;
    }
    /**
     * @param {Orderline} line
     * side effect: update `this.splitlines[line.id].quantity` depending on
     * - it's current value
     * - the total quantity of the product in the order
     * - the value of `line.is_pos_groupable()`
     */
    _splitQuantity(line) {
        const split = this.splitlines[line.id];
        // total quantity of the product in this line
        // we add up the quantities of all the lines that have this product
        let totalQuantity = 0;

        this.pos
            .get_order()
            .get_orderlines()
            .forEach(function (orderLine) {
                if (orderLine.get_product().id === split.product) {
                    totalQuantity += orderLine.get_quantity();
                }
            });

        if (line.get_quantity() > 0) {
            if (!line.is_pos_groupable()) {
                if (split.quantity !== line.get_quantity()) {
                    split.quantity = line.get_quantity();
                } else {
                    split.quantity = 0;
                }
            } else {
                if (split.quantity < totalQuantity) {
                    split.quantity += 1;
                    // TODO: why do we need this `if`?
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
            orderline.set_quantity(split.quantity, "do not recompute unit price");
        } else if (orderline) {
            this.newOrder.removeOrderline(orderline);
            this.newOrderLines[line.id] = null;
        }
    }
    _isFullPayOrder() {
        const order = this.pos.get_order();
        let full = true;
        const splitlines = this.splitlines;
        const groupedLines = groupBy(order.get_orderlines(), (line) => line.get_product().id);

        Object.keys(groupedLines).forEach(function (lineId) {
            var maxQuantity = groupedLines[lineId].reduce(
                (quantity, line) => quantity + line.get_quantity(),
                0
            );
            Object.keys(splitlines).forEach((id) => {
                const split = splitlines[id];
                if (split.product === groupedLines[lineId][0].get_product().id) {
                    maxQuantity -= split.quantity;
                }
            });
            if (maxQuantity !== 0) {
                full = false;
            }
        });

        return full;
    }
    _setQuantityOnCurrentOrder() {
        const order = this.pos.get_order();
        for (var id in this.splitlines) {
            var split = this.splitlines[id];
            var line = this.currentOrder.get_orderline(parseInt(id));

            if (!this.pos.disallowLineQuantityChange()) {
                line.set_quantity(
                    line.get_quantity() - split.quantity,
                    "do not recompute unit price"
                );
            } else {
                if (split.quantity) {
                    const decreaseLine = line.clone();
                    decreaseLine.order = order;
                    decreaseLine.noDecrease = true;
                    decreaseLine.set_quantity(-split.quantity);
                    order.add_orderline(decreaseLine);
                }
            }
        }
        if (!this.pos.disallowLineQuantityChange()) {
            for (id in this.splitlines) {
                line = this.currentOrder.get_orderline(parseInt(id));
                if (line && Math.abs(line.get_quantity()) < 0.00001) {
                    this.currentOrder.removeOrderline(line);
                }
            }
        }
    }
}

registry.category("pos_screens").add("SplitBillScreen", SplitBillScreen);

```

## File: static\src\app\split_bill_screen\split_bill_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.SplitBillScreen">
        <div class="splitbill-screen screen h-100 bg-100">
            <div class="contents d-flex flex-column flex-nowrap h-100 my-0 mx-auto">
                <div class="top-content d-flex align-items-center p-2 border-bottom text-center">
                    <button class="button back btn btn-lg btn-outline-primary" t-on-click="back">
                        <i class="fa fa-angle-double-left me-2"></i>
                        <span>Back</span>
                    </button>
                    <div class="top-content-center flex-grow-1">
                        <h2 class="mb-0">Bill Splitting</h2>
                    </div>
                </div>

                <div t-if="newOrder" class="main d-flex flex-nowrap flex-grow-1 overflow-hidden">
                    <div class="flex-grow-1 mw-50 m-3 bg-view border rounded w-50 overflow-auto">
                        <OrderWidget lines="orderlines" t-slot-scope="scope">
                            <t t-set="line" t-value="scope.line" />
                            <Orderline line="getLineData(line)"
                                t-on-click="() => this.onClickLine(line)"
                                class="{'selected text-bg-primary': splitlines[line.id].quantity !== 0}" />
                        </OrderWidget>
                    </div>
                    <div class="controls border-start flex-column flex-nowrap flex-grow-1 flex-shrink-1 flex-basis-0">
                        <div class="order-info py-4 border-bottom text-center text-success">
                            <span class="subtotal">
                                <t t-esc="env.utils.formatCurrency(newOrder.get_subtotal())" />
                            </span>
                        </div>
                        <div class="pay-button m-3">
                            <div class="button btn btn-lg btn-secondary py-3 w-100" t-on-click="proceed">
                                <i class="oi oi-chevron-right me-2" />
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

## File: static\src\app\tip_receipt\tip_receipt.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { ReceiptHeader } from "@point_of_sale/app/screens/receipt_screen/receipt/receipt_header/receipt_header";

export class TipReceipt extends Component {
    static template = "pos_restaurant.TipReceipt";
    static components = { ReceiptHeader };
    static props = ["headerData", "data", "total"];

    get total() {
        return this.props.total;
    }
}

```

## File: static\src\app\tip_receipt\tip_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.TipReceipt">
        <div class="pos-receipt">
            <ReceiptHeader data="props.headerData" />
            <div class="pos-payment-terminal-receipt">
                <pre t-esc="data" />
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

## File: static\src\app\tip_screen\tip_screen.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { parseFloat } from "@web/views/fields/parsers";
import { registry } from "@web/core/registry";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { ConfirmPopup } from "@point_of_sale/app/utils/confirm_popup/confirm_popup";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { useService } from "@web/core/utils/hooks";
import { Component, useRef, onMounted } from "@odoo/owl";
import { TipReceipt } from "@pos_restaurant/app/tip_receipt/tip_receipt";

export class TipScreen extends Component {
    static template = "pos_restaurant.TipScreen";
    setup() {
        this.pos = usePos();
        this.posReceiptContainer = useRef("pos-receipt-container");
        this.popup = useService("popup");
        this.orm = useService("orm");
        this.printer = useService("printer");
        this.state = this.currentOrder.uiState.TipScreen;
        this._totalAmount = this.currentOrder.get_total_with_tax();

        onMounted(async () => {
            await this.printTipReceipt();
        });
    }
    get overallAmountStr() {
        const tipAmount = this.env.utils.isValidFloat(this.state.inputTipAmount)
            ? parseFloat(this.state.inputTipAmount)
            : 0;
        const original = this.env.utils.formatCurrency(this.totalAmount);
        const tip = this.env.utils.formatCurrency(tipAmount);
        const overall = this.env.utils.formatCurrency(this.totalAmount + tipAmount);
        return `${original} + ${tip} tip = ${overall}`;
    }
    get totalAmount() {
        return this._totalAmount;
    }
    get currentOrder() {
        return this.pos.get_order();
    }
    get percentageTips() {
        return [
            { percentage: "15%", amount: 0.15 * this.totalAmount },
            { percentage: "20%", amount: 0.2 * this.totalAmount },
            { percentage: "25%", amount: 0.25 * this.totalAmount },
        ];
    }
    async validateTip() {
        const amount = this.env.utils.isValidFloat(this.state.inputTipAmount)
            ? parseFloat(this.state.inputTipAmount)
            : 0;
        const order = this.pos.get_order();
        const serverId = this.pos.validated_orders_name_server_id_map[order.name];

        if (!serverId) {
            this.popup.add(ErrorPopup, {
                title: _t("Unsynced order"),
                body: _t(
                    "This order is not yet synced to server. Make sure it is synced then try again."
                ),
            });
            return;
        }

        if (!amount) {
            await this.orm.call("pos.order", "set_no_tip", [serverId]);
            this.goNextScreen();
            return;
        }

        if (amount > 0.25 * this.totalAmount) {
            const { confirmed } = await this.popup.add(ConfirmPopup, {
                title: "Are you sure?",
                body: `${this.env.utils.formatCurrency(
                    amount
                )} is more than 25% of the order's total amount. Are you sure of this tip amount?`,
            });
            if (!confirmed) {
                return;
            }
        }

        // set the tip by temporarily allowing order modification
        order.finalized = false;
        order.set_tip(amount);
        order.finalized = true;

        const paymentline = this.pos.get_order().get_paymentlines()[0];
        if (paymentline.payment_method.payment_terminal) {
            paymentline.amount += amount;
            await paymentline.payment_method.payment_terminal.send_payment_adjust(paymentline.cid);
        }

        // set_tip calls add_product which sets the new line as the selected_orderline
        const tip_line = order.selected_orderline;
        await this.orm.call("pos.order", "set_tip", [serverId, tip_line.export_as_JSON()]);
        this.goNextScreen();
    }
    goNextScreen() {
        this.pos.removeOrder(this.currentOrder);
        if (!this.pos.config.module_pos_restaurant) {
            this.pos.add_new_order();
        }
        const { name, props } = this.nextScreen;
        this.pos.showScreen(name, props);
    }
    get nextScreen() {
        return { name: "ReceiptScreen" };
    }
    async printTipReceipt() {
        const order = this.currentOrder;
        const receipts = [
            order.selected_paymentline.ticket,
            order.selected_paymentline.cashier_receipt,
        ];
        for (let i = 0; i < receipts.length; i++) {
            await this.printer.print(
                TipReceipt,
                {
                    headerData: this.pos.getReceiptHeaderData(order),
                    data: receipts[i],
                    total: this.env.utils.formatCurrency(this.totalAmount),
                },
                { webPrintFallback: false }
            );
        }
    }
}

registry.category("pos_screens").add("TipScreen", TipScreen);

```

## File: static\src\app\tip_screen\tip_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.TipScreen">
        <div class="tip-screen screen h-100 bg-100">
            <div class="pos-receipt-container text-center" t-ref="pos-receipt-container"/>
            <div class="screen-content d-flex flex-column h-100">
                <div class="top-content d-flex align-items-center p-2 border-bottom text-center">
                    <button class="button btn btn-lg btn-outline-primary" t-on-click="() => this.pos.showScreen('FloorScreen')">
                        <i class="fa fa-angle-double-left me-2"/>
                        <span>Back</span>
                    </button>
                    <button class="button btn btn-lg btn-outline-primary" t-if="printer.is()" t-on-click="printTipReceipt">
                        <i class="fa fa-print"></i>
                        <span> </span>
                        <span>Reprint receipts</span>
                    </button>
                    <div class="top-content-center flex-grow-1">
                        <h2 class="mb-0">Add a tip</h2>
                    </div>
                    <div class="button highlight next btn btn-lg btn-primary" t-on-click="validateTip">
                        Settle <i class="fa fa-angle-double-right"></i>
                    </div>
                </div>
                <div class="tip-options">
                    <div class="total-amount my-4 fs-2 text-center">
                        <t t-esc="overallAmountStr" />
                    </div>
                    <div class="tip-amount-options d-flex flex-column gap-2 mx-4 p-3 rounded bg-view">
                        <div class="percentage-amounts d-flex gap-2">
                            <t t-foreach="percentageTips" t-as="tip" t-key="tip.percentage">
                                <button class="button btn btn-lg btn-secondary flex-fill py-5" t-on-click="() => { state.inputTipAmount = env.utils.formatCurrency(tip.amount,false); }">
                                    <div class="percentage fs-1 text-primary text-center">
                                        <t t-esc="tip.percentage"></t>
                                    </div>
                                    <div class="amount text-muted">
                                        <t t-esc="env.utils.formatCurrency(tip.amount)" />
                                    </div>
                                </button>
                            </t>
                        </div>
                        <div class="no-tip d-grid">
                            <button class="button btn btn-lg btn-secondary flex-fill py-5 fs-1 text-primary"  t-on-click="() => { state.inputTipAmount = '0'; }">No Tip</button>
                        </div>
                        <div class="custom-amount-form d-flex">
                            <div class="input-group input-group-lg">
                                <span class="input-group-text bg-secondary">Tip Amount</span>
                                <input type="text" class="item form-control fs-1" aria-label="Username" t-model="state.inputTipAmount" t-att-data-amount="state.inputTipAmount"/>
                                <span class="input-group-text">
                                    <div class="currency">
                                        <t t-esc="pos.getCurrencySymbol()" />
                                    </div>
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\overrides\components\navbar\navbar.js

```javascript
/** @odoo-module */

import { Navbar } from "@point_of_sale/app/navbar/navbar";
import { patch } from "@web/core/utils/patch";

patch(Navbar.prototype, {
    /**
     * If no table is set to pos, which means the current main screen
     * is floor screen, then the order count should be based on all the orders.
     */
    get orderCount() {
        if (this.pos.config.module_pos_restaurant && this.pos.table) {
            return this.pos.getTableOrders(this.pos.table.id).length;
        }
        return super.orderCount;
    },
    _shouldLoadOrders() {
        return super._shouldLoadOrders() || this.pos.config.module_pos_restaurant;
    },
    onSwitchButtonClick() {
        const mode = this.pos.floorPlanStyle == "kanban" ? "default" : "kanban";
        localStorage.setItem("floorPlanStyle", mode);
        this.pos.floorPlanStyle = mode;
    },
    toggleEditMode() {
        this.pos.toggleEditMode();
    },
    showBackButton() {
        return (
            super.showBackButton(...arguments) ||
            (this.pos.showBackButton() && this.pos.config.module_pos_restaurant)
        );
    },
});

```

## File: static\src\overrides\components\navbar\navbar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.Navbar" t-inherit="point_of_sale.Navbar" t-inherit-mode="extension">
        <xpath expr="//li[hasclass('backend-button')]" position="before">
            <li t-if="pos.mainScreen.component.name == 'FloorScreen' and pos.floors.length" class="menu-item navbar-button edit-button" t-on-click="toggleEditMode">
                <a class="dropdown-item py-2">Edit Plan</a>
            </li>
            <li t-if="pos.mainScreen.component.name == 'FloorScreen'" class="menu-item navbar-button" t-on-click="onSwitchButtonClick">
                <a class="dropdown-item py-2">Switch Floor View</a>
            </li>
        </xpath>
        <xpath expr="//BackButton" position="before">
            <span t-if="pos.table?.name" t-esc="pos.table.name" t-attf-style="background-color: {{pos.table.color}};border-radius: 0.25rem;" class="table-name text-white fw-bolder my-2 px-3 d-flex align-items-center" />
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\navbar\back_button\back_button.js

```javascript
/** @odoo-module */

import { BackButton } from "@point_of_sale/app/navbar/back_button/back_button";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { TipScreen } from "@pos_restaurant/app/tip_screen/tip_screen";
import { patch } from "@web/core/utils/patch";

patch(BackButton.prototype, {
    get floor() {
        return this.table?.floor;
    },
    get hasTable() {
        return this.table != null;
    },
    /**
     * @override
     * If we have a floor screen,
     * the logic of the back button changes a bit.
     */
    async onClick() {
        if (this.pos.mainScreen.component && this.pos.config.module_pos_restaurant) {
            if (
                (this.pos.mainScreen.component === ProductScreen &&
                    this.pos.mobile_pane == "right") ||
                this.pos.mainScreen.component === TipScreen
            ) {
                this.pos.showScreen("FloorScreen", { floor: this.floor });
            } else {
                super.onClick(...arguments);
            }
            return;
        }
        super.onClick(...arguments);
    },
});

```

## File: static\src\overrides\components\navbar\back_button\back_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.BackButton" t-inherit="point_of_sale.BackButton" t-inherit-mode="extension">
         <xpath expr="//span//span" position="replace">
             <t t-if="!ui.isSmall and pos.config.module_pos_restaurant">
                 <span>Change table</span>
             </t>
             <t t-else="">
                 <span t-if="!ui.isSmall">BACK</span>
             </t>
         </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\order_widget\order_widget.js

```javascript
/** @odoo-module */

import { OrderWidget } from "@point_of_sale/app/generic_components/order_widget/order_widget";
import { patch } from "@web/core/utils/patch";
import { _t } from "@web/core/l10n/translation";

patch(OrderWidget, {
    props: {
        ...OrderWidget.props,
        isConfigRestaurant: { type: Boolean, optional: true },
        isOrderBooked: { type: Boolean, optional: true },
    },
});

patch(OrderWidget.prototype, {
    emptyCartText() {
        let text = super.emptyCartText(...arguments);
        if (this.props.isConfigRestaurant && !this.props.isOrderBooked) {
            text += " " + _t("or book the table for later");
        }
        return text;
    },
});

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
/** @odoo-module */

import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";

patch(PaymentScreen.prototype, {
    get nextScreen() {
        const order = this.currentOrder;
        if (!this.pos.config.set_tip_after_payment || order.is_tipped) {
            return super.nextScreen;
        }
        // Take the first payment method as the main payment.
        const mainPayment = order.get_paymentlines()[0];
        if (mainPayment && mainPayment.canBeAdjusted()) {
            return "TipScreen";
        }
        return super.nextScreen;
    },
});

```

## File: static\src\overrides\components\payment_screen\payment_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.PaymentScreenValidate" t-inherit="point_of_sale.PaymentScreenValidate" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('button') and hasclass('next')]" position="attributes">
            <attribute name="t-att-hidden">pos.config.set_tip_after_payment and !currentOrder.is_paid()</attribute>
        </xpath>

        <xpath expr="//div[hasclass('button') and hasclass('next')]/span[hasclass('next_text')]" position="replace">
            <t t-if="pos.config.set_tip_after_payment and currentOrder.is_paid()">
                <span class="back_text">Close Tab</span>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
    <t t-name="pos_restaurant.PaymentScreenTop" t-inherit="point_of_sale.PaymentScreenTop" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('button') and hasclass('back')]/span[hasclass('back_text')]" position="replace">
            <t t-if="pos.config.set_tip_after_payment and currentOrder.is_paid()">
                <span class="back_text">Keep Open</span>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>


</templates>

```

## File: static\src\overrides\components\payment_screen\payment_screen_payment_lines\payment_screen_payment_lines.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.PaymentScreenPaymentLines" t-inherit="point_of_sale.PaymentScreenPaymentLines" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('send_payment_reversal')]/.." position="replace">
            <t t-if="line.canBeAdjusted() &amp;&amp; line.order.get_total_paid() &lt; line.order.get_total_with_tax()">
                <div class="button send_adjust_amount" title="Adjust Amount" t-on-click="() => this.sendPaymentAdjust(line)">
                    Adjust Amount
                </div>
            </t>
            <t t-elif="line.can_be_reversed">
                <div class="button send_payment_reversal" title="Reverse Payment" t-on-click="() => this.props.sendPaymentReverse(line)">
                    Reverse
                </div>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\product_screen\product_screen.js

```javascript
/** @odoo-module */

import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { patch } from "@web/core/utils/patch";

patch(ProductScreen.prototype, {
    /**
     * @override
     */
    get selectedOrderlineQuantity() {
        const order = this.pos.get_order();
        const orderline = order.get_selected_orderline();
        if (this.pos.config.module_pos_restaurant && this.pos.orderPreparationCategories.size) {
            let orderline_name = orderline.product.display_name;
            if (orderline.description) {
                orderline_name += " (" + orderline.description + ")";
            }
            const changes = Object.values(order.getOrderChanges().orderlines).find(
                (change) => change.name == orderline_name
            );
            return changes ? changes.quantity : false;
        }
        return super.selectedOrderlineQuantity;
    },
    get selectedOrderlineTotal() {
        return this.env.utils.formatCurrency(
            this.pos.get_order().get_selected_orderline().get_display_price()
        );
    },
    get nbrOfChanges() {
        return this.currentOrder.getOrderChanges().nbrOfChanges;
    },
    get swapButton() {
        return this.pos.config.module_pos_restaurant && this.pos.orderPreparationCategories.size;
    },
    submitOrder() {
        this.pos.sendOrderInPreparationUpdateLastChange(this.pos.get_order());
    },
    get primaryReviewButton() {
        return (
            !this.primaryOrderButton &&
            !this.pos.get_order().is_empty() &&
            this.pos.config.module_pos_restaurant
        );
    },
    get primaryOrderButton() {
        return (
            this.pos.get_order().getOrderChanges().nbrOfChanges !== 0 &&
            this.pos.config.module_pos_restaurant
        );
    },
});

```

## File: static\src\overrides\components\product_screen\product_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.ProductScreen" t-inherit="point_of_sale.ProductScreen" t-inherit-mode="extension">
        <!-- add a showOrderButton here (using the computeOrderChange method) -->
        <xpath expr="//button[hasclass('pay-button')]" position="replace">
            <button
                t-if="this.swapButton"
                class="btn-switchpane btn flex-fill rounded-0 fw-bolder"
                t-on-click="submitOrder"
                t-attf-class="{{ primaryOrderButton ? 'btn-primary' : 'btn-secondary' }}">
                <!-- Replace the payment button by the order button -->
                <span class="fs-1 d-block">Order</span>
                <span><t t-esc="nbrOfChanges"/> changes</span>
            </button>
            <t t-else="">
                <button
                    class="btn-switchpane btn flex-fill rounded-0 fw-bolder"
                    t-attf-class="{{ primaryPayButton ? 'btn-primary' : 'btn-secondary' }}"
                    t-on-click="() => currentOrder.pay()">
                    <span class="fs-1 d-block">Pay</span>
                    <span><t t-esc="total" /></span>
                </button>
            </t>
        </xpath>
        <xpath expr="//button[hasclass('review-button')]" position="replace">
            <button class="btn-switchpane btn w-50 rounded-0 fw-bolder review-button" t-attf-class="{{ primaryReviewButton ? 'btn-primary' : 'btn-secondary' }}" t-on-click="switchPane">
                <span class="fs-1 d-block">Payment</span>
                <span t-if="this.swapButton"><t t-esc="total" /></span>
                <span t-else=""><t t-esc="items"/> items</span>
            </button>
        </xpath>
        <xpath expr="//Orderline" position="attributes">
            <attribute name="t-on-dblclick">
                () => line.toggleSkipChange()
            </attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\product_screen\actionpad_widget\actionpad_widget.js

```javascript
/** @odoo-module */
import { patch } from "@web/core/utils/patch";
import { ActionpadWidget } from "@point_of_sale/app/screens/product_screen/action_pad/action_pad";
/**
 * @props partner
 */

patch(ActionpadWidget.prototype, {
    get swapButton() {
        return this.props.actionType === "payment" && this.pos.config.module_pos_restaurant;
    },
    get currentOrder() {
        return this.pos.get_order();
    },
    get swapButtonClasses() {
        return {
            "highlight btn-primary": this.currentOrder?.hasChangesToPrint(),
            altlight:
                !this.currentOrder?.hasChangesToPrint() && this.currentOrder?.hasSkippedChanges(),
        };
    },
    async submitOrder() {
        if (!this.clicked) {
            this.clicked = true;
            try {
                await this.pos.sendOrderInPreparationUpdateLastChange(this.currentOrder);
            } finally {
                this.clicked = false;
            }
        }
    },
    hasQuantity(order) {
        if (!order) {
            return false;
        } else {
            return (
                order.orderlines.reduce((totalQty, line) => totalQty + line.get_quantity(), 0) > 0
            );
        }
    },
    get highlightPay() {
        return (
            super.highlightPay &&
            !this.currentOrder.hasChangesToPrint() &&
            this.hasQuantity(this.currentOrder)
        );
    },
    get categoryCount() {
        const orderChange = this.currentOrder.getOrderChanges().orderlines;

        const categories = Object.values(orderChange).reduce((acc, curr) => {
            const categoryId = this.pos.db.product_by_id[curr.product_id].pos_categ_ids[0];
            const category = this.pos.db.category_by_id[categoryId];
            if (category) {
                if (!acc[category.id]) {
                    acc[category.id] = {
                        count: curr.quantity,
                        name: category.name,
                        id: category.id,
                    };
                } else {
                    acc[category.id].count += curr.quantity;
                }
            }
            return acc;
        }, {});
        return Object.values(categories);
    },
    get displayCategoryCount() {
        return this.categoryCount.slice(0, 3);
    },
    get isCategoryCountOverflow() {
        if (this.categoryCount.length > 3) {
            return true;
        }
        return false;
    },
});

```

## File: static\src\overrides\components\product_screen\actionpad_widget\actionpad_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.ActionpadWidget" t-inherit="point_of_sale.ActionpadWidget" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('actionpad')]" position="attributes">
            <attribute name="t-att-class">{'w-50' : this.swapButton}</attribute>
        </xpath>

        <!-- Replace the payment button by the order button -->
        <xpath expr="//button[hasclass('validation')]" position="after">
            <button
                t-if="this.swapButton"
                t-attf-class="submit-order h-100 {{getMainButtonClasses()}}"
                t-att-class="swapButtonClasses"
                t-on-click="submitOrder"
                style="width: 170px;">
                <i class="fa fa-cutlery"></i>
                Order
                <div t-if="displayCategoryCount.length" class="break-line fw-normal fs-6 w-100  border-top mt-1 pt-1">
                    <t t-foreach="displayCategoryCount" t-as="categoryCountLine"  t-key="categoryCountLine_index">
                        <div class="d-flex justify-content-start g-0 my-1 w-100">
                            <label class="rounded px-2 py-0 me-1" style="background-color:rgba(0, 0, 0, 0.3);"><t t-esc="this.env.utils.formatProductQty(categoryCountLine.count, false)"/></label>
                            <label class="text-truncate ps-0" ><t t-esc="categoryCountLine.name"/></label>
                        </div>
                    </t>
                </div>
                <t t-if="isCategoryCountOverflow">
                    <div class="position-absolute bottom-0">...</div>
                </t>
            </button>
        </xpath>
        <xpath expr="//button[hasclass('validation')]" position="attributes">
            <attribute name="t-if">!this.swapButton</attribute>
        </xpath>

        <!-- Replace the customer button by the payment button, the customer button will be added in the mixins -->
        <xpath expr="//button[hasclass('set-partner')]" position="after">
            <button t-if="this.swapButton"
                t-on-click="() => pos.get_order().pay()" 
                class="button pay-order-button btn btn-lg rounded-0" 
                t-attf-class="{{ this.highlightPay ? 'highlight btn-primary' : 'btn-secondary' }}" 
                >
                <i class="oi oi-chevron-right" role="img" aria-label="Pay" title="Pay" />
                Payment
            </button>
        </xpath>
        <xpath expr="//button[hasclass('set-partner')]" position="attributes">
            <attribute name="t-if">!this.swapButton</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\receipt_screen\receipt_screen.js

```javascript
/** @odoo-module */

import { ReceiptScreen } from "@point_of_sale/app/screens/receipt_screen/receipt_screen";
import { patch } from "@web/core/utils/patch";
import { onWillUnmount } from "@odoo/owl";
import { FloorScreen } from "@pos_restaurant/app/floor_screen/floor_screen";

patch(ReceiptScreen.prototype, {
    setup() {
        super.setup(...arguments);
        onWillUnmount(() => {
            // When leaving the receipt screen to the floor screen the order is paid and can be removed
            if (this.pos.mainScreen.component === FloorScreen && this.currentOrder.finalized) {
                this.pos.removeOrder(this.currentOrder);
            }
        });
    },
    //@override
    _addNewOrder() {
        if (!this.pos.config.module_pos_restaurant) {
            super._addNewOrder(...arguments);
        }
    },
    isResumeVisible() {
        if (this.pos.config.module_pos_restaurant && this.pos.table) {
            return this.pos.getTableOrders(this.pos.table.id).length > 1;
        }
        return super.isResumeVisible(...arguments);
    },
    //@override
    get nextScreen() {
        if (this.pos.config.module_pos_restaurant) {
            const table = this.pos.table;
            return { name: "FloorScreen", props: { floor: table ? table.floor : null } };
        } else {
            return super.nextScreen;
        }
    },
});

```

## File: static\src\overrides\components\receipt_screen\order_receipt\order_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('pos-receipt-order-data')]" position="inside">
            <t t-if="props.data.isBill">
                <div>PRO FORMA</div>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('receipt-change')]" position="attributes">
            <attribute name="t-if">!props.data.isBill</attribute>
        </xpath>
        <xpath expr="//div[hasclass('before-footer')]" position="after">
            <t t-if="props.data.isBill and props.data.set_tip_after_payment">
                <div class="tip-form py-3">
                    <div class="title text-center mt-3">For convenience, we are providing the following gratuity calculations:</div>
                    <div class="percentage-options percentage-options d-flex flex-nowrap mt-3">
                        <div class="option d-flex flex-column flex-nowrap align-items-center justify-content-center flex-grow-1">
                            <div>15%</div>
                            <div class="amount">
                                <t t-esc="props.formatCurrency(props.data.amount_total * 0.15)"></t>
                            </div>
                        </div>
                        <div class="option d-flex flex-column flex-nowrap align-items-center justify-content-center flex-grow-1">
                            <div>20%</div>
                            <div class="amount">
                                <t t-esc="props.formatCurrency(props.data.amount_total * 0.20)"></t>
                            </div>
                        </div>
                        <div class="option d-flex flex-column flex-nowrap align-items-center justify-content-center flex-grow-1">
                            <div>25%</div>
                            <div class="amount">
                                <t t-esc="props.formatCurrency(props.data.amount_total * 0.25)"></t>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </xpath>
    </t>
    <t t-name="pos_restaurant.ReceiptHeader" t-inherit="point_of_sale.ReceiptHeader" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('cashier')]" position="after">
            <t t-if="props.data.table">
                at table <t t-esc="props.data.table" />
            </t>
            <t t-if="props.data.table and props.data.customer_count">
                <div>Guests: <t t-esc="props.data.customer_count" /></div>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
import { useAutofocus } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";
import { parseFloat } from "@web/views/fields/parsers";
import { ConfirmPopup } from "@point_of_sale/app/utils/confirm_popup/confirm_popup";
import { Component, useState } from "@odoo/owl";

patch(TicketScreen.prototype, {
    _getScreenToStatusMap() {
        return Object.assign(super._getScreenToStatusMap(...arguments), {
            PaymentScreen: this.pos.config.set_tip_after_payment
                ? "OPEN"
                : super._getScreenToStatusMap(...arguments).PaymentScreen,
            TipScreen: "TIPPING",
        });
    },
    getTable(order) {
        const table = order.getTable();
        if (table) {
            let floorAndTable = "";

            if (this.pos.floors && this.pos.floors.length > 1) {
                floorAndTable = `${table.floor.name}/`;
            }

            floorAndTable += table.name;
            return floorAndTable;
        }
    },
    //@override
    _getSearchFields() {
        if (!this.pos.config.module_pos_restaurant) {
            return super._getSearchFields(...arguments);
        }
        return Object.assign({}, super._getSearchFields(...arguments), {
            TABLE: {
                repr: this.getTable.bind(this),
                displayName: _t("Table"),
                modelField: "table_id.name",
            },
        });
    },
    async _setOrder(order) {
        if (!this.pos.config.module_pos_restaurant || this.pos.table || !order.tableId) {
            return super._setOrder(...arguments);
        }
        // we came from the FloorScreen
        const orderTable = order.getTable();
        await this.pos.setTable(orderTable, order.uid);
        this.closeTicketScreen();
    },
    async onDeleteOrder(order) {
        const confirmed = await super.onDeleteOrder(...arguments);
        if (
            confirmed &&
            this.pos.config.module_pos_restaurant &&
            this.pos.table &&
            !this.pos.orders.some((order) => order.tableId === this.pos.table.id)
        ) {
            return this.pos.showScreen("FloorScreen");
        }
    },
    get allowNewOrders() {
        return this.pos.config.module_pos_restaurant
            ? Boolean(this.pos.table)
            : super.allowNewOrders;
    },
    async settleTips() {
        // set tip in each order
        for (const order of this.getFilteredOrderList()) {
            const tipAmount = this.env.utils.isValidFloat(order.uiState.TipScreen.inputTipAmount)
                ? parseFloat(order.uiState.TipScreen.inputTipAmount)
                : 0;
            const serverId = this.pos.validated_orders_name_server_id_map[order.name];
            if (!serverId) {
                console.warn(
                    `${order.name} is not yet sync. Sync it to server before setting a tip.`
                );
            } else {
                const result = await this.setTip(order, serverId, tipAmount);
                if (!result) {
                    break;
                }
            }
        }
    },
    async setTip(order, serverId, amount) {
        try {
            const paymentline = order.get_paymentlines()[0];
            if (paymentline.payment_method.payment_terminal) {
                paymentline.amount += amount;
                this.pos.set_order(order, { silent: true });
                await paymentline.payment_method.payment_terminal.send_payment_adjust(
                    paymentline.cid
                );
            }

            if (!amount) {
                await this.setNoTip(serverId);
            } else {
                order.finalized = false;
                order.set_tip(amount);
                order.finalized = true;
                const tip_line = order.selected_orderline;
                await this.orm.call("pos.order", "set_tip", [serverId, tip_line.export_as_JSON()]);
            }
            if (order === this.pos.get_order()) {
                this._selectNextOrder(order);
            }
            this.pos.removeOrder(order);
            return true;
        } catch {
            const { confirmed } = await this.popup.add(ConfirmPopup, {
                title: "Failed to set tip",
                body: `Failed to set tip to ${order.name}. Do you want to proceed on setting the tips of the remaining?`,
            });
            return confirmed;
        }
    },
    async setNoTip(serverId) {
        await this.orm.call("pos.order", "set_no_tip", [serverId]);
    },
    _getOrderStates() {
        const result = super._getOrderStates(...arguments);
        if (this.pos.config.set_tip_after_payment) {
            result.delete("PAYMENT");
            result.set("OPEN", { text: _t("Open"), indented: true });
            result.set("TIPPING", { text: _t("Tipping"), indented: true });
        }
        return result;
    },
    async onDoRefund() {
        const order = this.getSelectedOrder();
        if (this.pos.config.module_pos_restaurant && order && !this.pos.table) {
            this.pos.setTable(order.table ? order.table : Object.values(this.pos.tables_by_id)[0]);
        }
        super.onDoRefund(...arguments);
    },
    isDefaultOrderEmpty(order) {
        if (this.pos.config.module_pos_restaurant) {
            return false;
        }
        return super.isDefaultOrderEmpty(...arguments);
    },
});

export class TipCell extends Component {
    static template = "pos_restaurant.TipCell";

    setup() {
        this.state = useState({ isEditing: false });
        this.orderUiState = this.props.order.uiState.TipScreen;
        useAutofocus();
    }
    get tipAmountStr() {
        return this.env.utils.formatCurrency(
            this.env.utils.isValidFloat(this.orderUiState.inputTipAmount)
                ? parseFloat(this.orderUiState.inputTipAmount)
                : 0
        );
    }
    onBlur() {
        this.state.isEditing = false;
    }
    onKeydown(event) {
        if (event.key === "Enter") {
            this.state.isEditing = false;
        }
    }
    editTip() {
        this.state.isEditing = true;
    }
}

patch(TicketScreen, {
    components: { ...TicketScreen.components, TipCell },
});

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.TicketScreen" t-inherit="point_of_sale.TicketScreen" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('header-row')]//div[@name='delete']" position="before">
            <div t-if="pos.config.module_pos_restaurant" class="col p-2" name="table">Table</div>
            <div t-if="_state.ui.filter == 'TIPPING'" class="col end narrow p-2" name="tip">Tip</div>
        </xpath>
        <xpath expr="//div[hasclass('order-row')]//div[@name='delete']" position="before">
            <div t-if="pos.config.module_pos_restaurant" class="col p-2" name="table">
                <t t-if="order.tableId">
                    <div t-if="ui.isSmall">Table</div>
                    <div><t t-esc="getTable(order)"></t></div>
                </t>
            </div>
            <div t-if="_state.ui.filter == 'TIPPING'" class="col end narrow p-2" name="tip">
                <div t-if="ui.isSmall">Tip</div>
                <div><TipCell order="order" /></div>
            </div>
        </xpath>
        <xpath expr="//div[hasclass('mobileOrderList')]//div[hasclass('orderStatus')]" position="before">
            <t t-if="order.tableId">
                <div><t t-esc="getTable(order)"></t></div>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('buttons')]" position="inside">
            <button class="settle-tips btn btn-lg btn-primary" t-if="_state.ui.filter == 'TIPPING'" t-on-click="settleTips">Settle</button>
        </xpath>
    </t>

    <t t-name="pos_restaurant.TipCell">
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

## File: static\src\overrides\models\models.js

```javascript
/** @odoo-module */

import { Order, Orderline, Payment } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

// New orders are now associated with the current table, if any.
patch(Order.prototype, {
    setup(_defaultObj, options) {
        super.setup(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            if (this.defaultTableNeeded(options)) {
                this.tableId = this.pos.table.id;
            }
            this.booked = false;
            this.customerCount = this.customerCount || 1;
        }
    },
    //@override
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            json.table_id = this.tableId;
            json.customer_count = this.customerCount;
            json.booked = this.booked;
        }

        return json;
    },
    //@override
    init_from_JSON(json) {
        super.init_from_JSON(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            this.tableId = json.table_id;
            this.customerCount = json.customer_count;
        }
    },
    getCustomerCount() {
        return this.customerCount;
    },
    setCustomerCount(count) {
        this.customerCount = Math.max(count, 0);
    },
    getTable() {
        if (this.pos.config.module_pos_restaurant) {
            return this.pos.tables_by_id[this.tableId];
        }
        return null;
    },
    defaultTableNeeded(options) {
        return !this.tableId && !options.json && this.pos.table;
    },
    export_for_printing() {
        return {
            ...super.export_for_printing(...arguments),
            set_tip_after_payment: this.pos.config.set_tip_after_payment,
            isRestaurant: this.pos.config.module_pos_restaurant,
        };
    },
    setBooked(booked) {
        this.booked = booked;
        if (booked) {
            this.save_to_db();
            this.pos.ordersToUpdateSet.add(this);
        }
    },
    async add_product(product, options) {
        const result = await super.add_product(...arguments);
        if (this.pos.config.module_pos_restaurant) {
            this.setBooked(true);
        }
        return result;
    },
});

patch(Orderline.prototype, {
    setup() {
        super.setup(...arguments);
        this.note = this.note || "";
    },
    //@override
    clone() {
        const orderline = super.clone(...arguments);
        orderline.note = this.note;
        return orderline;
    },
    //@override
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        json.note = this.note;
        if (this.pos.config.iface_printers) {
            json.uuid = this.uuid;
        }
        return json;
    },
    //@override
    init_from_JSON(json) {
        super.init_from_JSON(...arguments);
        this.note = json.note;
        if (this.pos.config.iface_printers) {
            this.uuid = json.uuid;
        }
    },
    get_line_diff_hash() {
        if (this.getNote()) {
            return this.id + "|" + this.getNote();
        } else {
            return "" + this.id;
        }
    },
    toggleSkipChange() {
        if (this.hasChange || this.skipChange) {
            this.skipChange = !this.skipChange;
        }
    },
    getDisplayClasses() {
        return {
            ...super.getDisplayClasses(),
            "has-change text-success border-start border-success border-4": this.hasChange,
            "skip-change text-primary border-start border-primary border-4": this.skipChange,
        };
    },
});

patch(Payment.prototype, {
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
    },
});

```

## File: static\src\overrides\models\payment.js

```javascript
/** @odoo-module */

import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { patch } from "@web/core/utils/patch";

patch(PaymentInterface.prototype, {
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
    send_payment_adjust(cid) {},
});

```

## File: static\src\overrides\models\popup_service.js

```javascript
/* @odoo-module */

import { patch } from "@web/core/utils/patch";
import { popupService } from "@point_of_sale/app/popup/popup_service";

patch(popupService, {
    start() {
        return Object.assign(super.start(...arguments), {
            closePopupsButError() {
                const popups = Object.values(this.popups);
                const isErrorPopupOpen = popups.some((popup) =>
                    // FIXME POSREF: this seems very brittle.
                    popup.component.name.toLowerCase().includes("error")
                );
                if (!isErrorPopupOpen) {
                    for (const popup of popups) {
                        popup.props.close(false);
                    }
                }
                return !isErrorPopupOpen;
            },
        });
    },
});

```

## File: static\src\overrides\models\pos_bus.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PosBus } from "@point_of_sale/app/bus/pos_bus_service";

patch(PosBus.prototype, {
    // Override
    setup() {
        super.setup(...arguments);

        if (this.pos.config.module_pos_restaurant) {
            this.initTableOrderCount();
        }
    },

    async initTableOrderCount() {
        const result = await this.orm.call(
            "pos.config",
            "get_tables_order_count_and_printing_changes",
            [this.pos.config.id]
        );

        this.ws_syncTableCount(result);
    },

    // Override
    dispatch(message) {
        super.dispatch(...arguments);

        if (message.type === "TABLE_ORDER_COUNT" && this.pos.config.module_pos_restaurant) {
            this.ws_syncTableCount(message.payload);
        }
    },

    // Sync the number of orders on each table with other PoS
    // using the same floorplan.
    async ws_syncTableCount(data) {
        const missingTable = data.find((table) => !(table.id in this.pos.tables_by_id));

        if (missingTable) {
            const result = await this.orm.call("pos.session", "get_pos_ui_restaurant_floor", [
                [odoo.pos_session_id],
            ]);

            if (this.pos.config.module_pos_restaurant) {
                this.pos.floors = result;
                this.pos.loadRestaurantFloor();
            }
        }

        for (const floor of this.pos.floors) {
            floor.changes_count = 0;
        }
        for (const table of data) {
            const table_obj = this.pos.tables_by_id[table.id];
            if (table_obj) {
                table_obj.order_count = table.orders;
                table_obj.changes_count = table.changes;
                table_obj.skip_changes = table.skip_changes;
                table_obj.floor.changes_count += table.changes;
            }
        }
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { FloorScreen } from "@pos_restaurant/app/floor_screen/floor_screen";
import { TipScreen } from "@pos_restaurant/app/tip_screen/tip_screen";
import { ConnectionLostError } from "@web/core/network/rpc_service";

const NON_IDLE_EVENTS = [
    "mousemove",
    "mousedown",
    "touchstart",
    "touchend",
    "touchmove",
    "click",
    "scroll",
    "keypress",
];
let IDLE_TIMER_SETTER;

patch(PosStore.prototype, {
    /**
     * @override
     */
    async setup() {
        this.orderToTransfer = null; // table transfer feature
        this.transferredOrdersSet = new Set(); // used to know which orders has been transferred but not sent to the back end yet
        this.isEditMode = false;
        await super.setup(...arguments);
        this.floorPlanStyle =
            localStorage.getItem("floorPlanStyle") || (this.ui.isSmall ? "kanban" : "default");
        if (this.config.module_pos_restaurant) {
            this.setActivityListeners();
            this.showScreen("FloorScreen", { floor: this.table?.floor || null });
        }
        this.currentFloor = this.floors?.length > 0 ? this.floors[0] : null;
    },
    setActivityListeners() {
        IDLE_TIMER_SETTER = this.setIdleTimer.bind(this);
        for (const event of NON_IDLE_EVENTS) {
            window.addEventListener(event, IDLE_TIMER_SETTER);
        }
    },
    setIdleTimer() {
        clearTimeout(this.idleTimer);
        if (this.shouldResetIdleTimer()) {
            this.idleTimer = setTimeout(() => this.actionAfterIdle(), 180000);
        }
    },
    async actionAfterIdle() {
        const isPopupClosed = this.popup.closePopupsButError();
        if (isPopupClosed) {
            this.closeTempScreen();
            const table = this.table;
            const order = this.get_order();
            if (order && order.get_screen_data().name === "ReceiptScreen") {
                // When the order is finalized, we can safely remove it from the memory
                // We check that it's in ReceiptScreen because we want to keep the order if it's in a tipping state
                this.removeOrder(order);
            }
            this.showScreen("FloorScreen", { floor: table?.floor });
        }
    },
    getReceiptHeaderData(order) {
        const json = super.getReceiptHeaderData(...arguments);
        if (this.config.module_pos_restaurant && order) {
            if (order.getTable()) {
                json.table = order.getTable().name;
            }
            json.customer_count = order.getCustomerCount();
        }
        return json;
    },
    shouldResetIdleTimer() {
        const stayPaymentScreen =
            this.mainScreen.component === PaymentScreen && this.get_order().paymentlines.length > 0;
        return (
            this.config.module_pos_restaurant &&
            !stayPaymentScreen &&
            this.mainScreen.component !== FloorScreen
        );
    },
    showScreen(screenName) {
        super.showScreen(...arguments);
        this.setIdleTimer();
    },
    closeScreen() {
        if (this.config.module_pos_restaurant && !this.get_order()) {
            return this.showScreen("FloorScreen");
        }
        return super.closeScreen(...arguments);
    },
    addOrderIfEmpty() {
        if (!this.config.module_pos_restaurant) {
            return super.addOrderIfEmpty(...arguments);
        }
    },
    /**
     * @override
     * Before closing pos, we remove the event listeners set on window
     * for detecting activities outside FloorScreen.
     */
    async closePos() {
        if (IDLE_TIMER_SETTER) {
            for (const event of NON_IDLE_EVENTS) {
                window.removeEventListener(event, IDLE_TIMER_SETTER);
            }
        }
        return super.closePos(...arguments);
    },
    showBackButton() {
        return (
            super.showBackButton(...arguments) ||
            this.mainScreen.component === TipScreen ||
            (this.mainScreen.component === ProductScreen && this.config.module_pos_restaurant)
        );
    },
    //@override
    async _processData(loadedData) {
        await super._processData(...arguments);
        if (this.config.module_pos_restaurant) {
            this.floors = loadedData["restaurant.floor"];
            this.loadRestaurantFloor();
        }
    },
    //@override
    async after_load_server_data() {
        var res = await super.after_load_server_data(...arguments);
        if (this.config.module_pos_restaurant) {
            this.table = null;
        }
        return res;
    },
    //@override
    // if we have tables, we do not load a default order, as the default order will be
    // set when the user selects a table.
    set_start_order() {
        if (!this.config.module_pos_restaurant) {
            super.set_start_order(...arguments);
        }
    },
    //@override
    add_new_order() {
        const order = super.add_new_order(...arguments);
        this.ordersToUpdateSet.add(order);
        return order;
    },
    async _getTableOrdersFromServer(tableIds) {
        this.set_synch("connecting", 1);
        try {
            // FIXME POSREF timeout
            const orders = await this.env.services.orm.silent.call(
                "pos.order",
                "export_for_ui_table_draft",
                [tableIds]
            );
            this.set_synch("connected");
            return orders;
        } catch (error) {
            this.set_synch("error");
            throw error;
        }
    },
    /**
     * Sync orders that got updated to the back end
     * @param tableId ID of the table we want to sync
     */
    async _syncTableOrdersToServer() {
        await this.sendDraftToServer();
        await this._removeOrdersFromServer();
        // This need to be called here otherwise _onReactiveOrderUpdated() will be called after the set is being cleared
        this.ordersToUpdateSet.clear();
        this.transferredOrdersSet.clear();
    },
    /**
     * Replace all the orders of a table by orders fetched from the backend
     * @param tableId ID of the table
     * @throws error
     */
    async _syncTableOrdersFromServer(tableId) {
        await this.push_orders({ show_error: true }); // in case we were offline and we paid orders in the mean time
        await this._removeOrdersFromServer(); // in case we were offline and we deleted orders in the mean time
        const ordersJsons = await this._getTableOrdersFromServer([tableId]);
        await this._addPricelists(ordersJsons);
        await this._addFiscalPositions(ordersJsons);
        const tableOrders = this.getTableOrders(tableId);
        this._replaceOrders(tableOrders, ordersJsons);
    },
    async _getOrdersJson() {
        if (this.config.module_pos_restaurant) {
            const tableIds = [].concat(
                ...this.floors.map((floor) => floor.tables.map((table) => table.id))
            );
            await this._syncTableOrdersToServer(); // to prevent losing the transferred orders
            const ordersJsons = await this._getTableOrdersFromServer(tableIds); // get all orders
            await this._loadMissingProducts(ordersJsons);
            await this._loadMissingPartners(ordersJsons);
            return ordersJsons;
        } else {
            return await super._getOrdersJson();
        }
    },
    _shouldRemoveOrder(order) {
        return super._shouldRemoveOrder(...arguments) && !this.transferredOrdersSet.has(order);
    },
    _shouldCreateOrder(json) {
        return (
            (!this._transferredOrder(json) || this._isSameTable(json)) &&
            (!this.selectedOrder || super._shouldCreateOrder(...arguments))
        );
    },
    _shouldRemoveSelectedOrder(removeSelected) {
        return this.selectedOrder && super._shouldRemoveSelectedOrder(...arguments);
    },
    _isSelectedOrder(json) {
        return !this.selectedOrder || super._isSelectedOrder(...arguments);
    },
    _isSameTable(json) {
        const transferredOrder = this._transferredOrder(json);
        return transferredOrder && transferredOrder.tableId === json.tableId;
    },
    _transferredOrder(json) {
        return [...this.transferredOrdersSet].find((order) => order.uid === json.uid);
    },
    _createOrder(json) {
        const transferredOrder = this._transferredOrder(json);
        if (this._isSameTable(json)) {
            // this means we transferred back to the original table, we'll prioritize the server state
            this.removeOrder(transferredOrder, false);
        }
        return super._createOrder(...arguments);
    },
    getDefaultSearchDetails() {
        if (this.table && this.table.id) {
            return {
                fieldName: "TABLE",
                searchTerm: this.table.name,
            };
        }
        return super.getDefaultSearchDetails();
    },
    loadRestaurantFloor() {
        // we do this in the front end due to the circular/recursive reference needed
        // Ignore floorplan features if no floor specified.
        this.floors_by_id = {};
        this.tables_by_id = {};
        for (const floor of this.floors) {
            this.floors_by_id[floor.id] = floor;
            for (const table of floor.tables) {
                this.tables_by_id[table.id] = table;
                table.floor = floor;
            }
        }
    },
    async setTable(table, orderUid = null) {
        this.table = table;
        try {
            this.loadingOrderState = true;
            await this._syncTableOrdersFromServer(table.id);
        } finally {
            this.loadingOrderState = false;
            const currentOrder = this.getTableOrders(table.id).find((order) =>
                orderUid ? order.uid === orderUid : !order.finalized
            );
            if (currentOrder) {
                this.set_order(currentOrder);
            } else {
                this.add_new_order();
            }
        }
    },
    getTableOrders(tableId) {
        return this.get_order_list().filter((order) => order.tableId === tableId);
    },
    async unsetTable() {
        try {
            await this._syncTableOrdersToServer();
        } catch (e) {
            if (!(e instanceof ConnectionLostError)) {
                throw e;
            }
            Promise.reject(e);
        }
        this.table = null;
        const order = this.get_order();
        if (order && !order.isBooked) {
            this.removeOrder(order);
        }
        this.set_order(null);
    },
    setCurrentOrderToTransfer() {
        this.selectedOrder.setBooked(true);
        this.orderToTransfer = this.selectedOrder;
    },
    async transferTable(table) {
        this.table = table;
        try {
            this.loadingOrderState = true;
            await this._syncTableOrdersFromServer(table.id);
        } finally {
            this.loadingOrderState = false;
            this.orderToTransfer.tableId = table.id;
            this.set_order(this.orderToTransfer);
            this.transferredOrdersSet.add(this.orderToTransfer);
            this.orderToTransfer = null;
        }
    },
    getCustomerCount(tableId) {
        const tableOrders = this.getTableOrders(tableId).filter((order) => !order.finalized);
        return tableOrders.reduce((count, order) => count + order.getCustomerCount(), 0);
    },
    isOpenOrderShareable() {
        return super.isOpenOrderShareable(...arguments) || this.config.module_pos_restaurant;
    },
    toggleEditMode() {
        this.isEditMode = !this.isEditMode;
    },
    async updateModelsData(models_data) {
        const floors = models_data["restaurant.floor"];
        if (floors) {
            this.floors = floors;
            this.loadRestaurantFloor();
            const result = await this.orm.call(
                "pos.config",
                "get_tables_order_count_and_printing_changes",
                [this.config.id]
            );
            for (const table of result) {
                const table_obj = this.tables_by_id[table.id];
                if (table_obj) {
                    table_obj.order_count = table.orders;
                    table_obj.changes_count = table.changes;
                    table_obj.skip_changes = table.skip_changes;
                }
            }
        }
        return super.updateModelsData(models_data);
    },
    async addProductToCurrentOrder(product, options = {}) {
        if (this.config.module_pos_restaurant && !this.get_order().booked) {
            this.get_order().setBooked(true);
        }
        return super.addProductToCurrentOrder(...arguments);
    },
});

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
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name="active" invisible="1"/>
                        <group col="4">
                            <field name="name" />
                            <field name="pos_config_ids" widget="many2many_tags"/>
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
                    <field name="pos_config_ids" widget="many2many_tags"/>
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
                    <field name="pos_config_ids" />
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div><strong>Floor Name: </strong><t t-esc="record.name.value"/></div>
                                <div><strong>Point of Sales: </strong><t t-esc="record.pos_config_ids.value"/></div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="action_restaurant_floor_form" model="ir.actions.act_window">
            <field name="name">Floor Plans</field>
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
                    <sheet>
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
                    </sheet>
                </form>
            </field>
        </record>

        <menuitem id="menu_restaurant_floor_all"
             parent="point_of_sale.menu_point_config_product"
             action="action_restaurant_floor_form"
             sequence="10"
             groups="point_of_sale.group_pos_user"/>
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
            <block id="restaurant_section" position="inside">
                <setting id="floor_and_table_map" string="Floors &amp; Tables Map" help="Design floors and assign orders to tables" invisible="is_kiosk_mode or not pos_module_pos_restaurant">
                    <div class="content-group">
                        <div class="mt16">
                            <label string="Floors" for="pos_floor_ids" class="o_light_label me-2"/>
                            <field name="pos_floor_ids" widget="many2many_tags" readonly="pos_has_active_session" />
                        </div>
                        <div>
                            <button name="%(pos_restaurant.action_restaurant_floor_form)d" icon="oi-arrow-right" type="action" string="Floors" class="btn-link"/>
                        </div>
                    </div>
                </setting>
                <setting string="Early Receipt Printing" help="Allow to print receipt before payment" id="iface_printbill"  invisible="not pos_module_pos_restaurant or is_kiosk_mode">
                    <field name="pos_iface_printbill"/>
                </setting>
                <setting help="Split total or order lines" id="iface_splitbill"  invisible="not pos_module_pos_restaurant or is_kiosk_mode">
                    <field name="pos_iface_splitbill" string="Allow Bill Splitting"/>
                </setting>
                <setting help="Online reservation for restaurant"  invisible="not pos_module_pos_restaurant or is_kiosk_mode">
                    <field name="pos_module_pos_restaurant_appointment" string="Table Booking" widget="upgrade_boolean" />
                    <div class="content-group" id="pos_table_booking" invisible="not pos_module_pos_restaurant_appointment">
                        <div class="text-warning mt16 mb4">
                            Save this page and come back here to set up the feature.
                        </div>
                    </div>
                </setting>
            </block>
            <div id="tip_product" position="after">
                <div invisible="not pos_module_pos_restaurant or not pos_iface_tipproduct">
                    <field name="pos_set_tip_after_payment" class="oe_inline"/>
                    <label class="fw-normal" for="pos_set_tip_after_payment" string="Add tip after payment"/>
                </div>
            </div>
            <block id="pos_interface_section" position="inside">
                <setting string="Internal Notes" help="Add internal notes on order lines for the kitchen" id="iface_orderline_notes">
                    <field name="pos_iface_orderline_notes"/>
                </setting>
            </block>
        </field>
    </record>
</odoo>

```

