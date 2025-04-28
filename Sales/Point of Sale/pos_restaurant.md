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
        'data/demo_data.xml',
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
        'point_of_sale.assets_debug': [
            'pos_restaurant/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\demo_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="pos.config" name="load_onboarding_restaurant_scenario" />
        <function model="pos.config" name="load_onboarding_bar_scenario" />
    </data>
</odoo>

```

## File: data\restaurant_session_floor.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="restaurant.floor" name="unlink">
            <value model="restaurant.floor" eval="obj().search([
                    ('pos_config_ids', 'in', ref('pos_config_main_restaurant')),
                ]).id"/>
        </function>

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
            <field name="name">OpenSession/0004</field>
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
            <field name="name">OpenSession/0005</field>
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
            <field name="background_color">rgb(255,255,255,0.75)</field>
            <field name="pos_config_ids" eval="[(6, 0, [ref('pos_restaurant.pos_config_main_restaurant')])]" />
            <field name="floor_background_image" type="base64" file="pos_restaurant/static/img/floor_main.jpeg" />
        </record>

        <record id="table_01" model="restaurant.table">
            <field name="table_number">1</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">407</field>
            <field name="position_v">88</field>
        </record>

        <record id="table_02" model="restaurant.table">
            <field name="table_number">2</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">582</field>
            <field name="position_v">88</field>
        </record>

        <record id="table_03" model="restaurant.table">
            <field name="table_number">3</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">165</field>
            <field name="height">100</field>
            <field name="position_h">762</field>
            <field name="position_v">83</field>
        </record>

        <record id="table_04" model="restaurant.table">
            <field name="table_number">4</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">407</field>
            <field name="position_v">247</field>
        </record>

        <record id="table_05" model="restaurant.table">
            <field name="table_number">5</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">582</field>
            <field name="position_v">247</field>
        </record>

        <record id="table_06" model="restaurant.table">
            <field name="table_number">6</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(53,211,116)</field>
            <field name="shape">square</field>
            <field name="width">165</field>
            <field name="height">100</field>
            <field name="position_h">762</field>
            <field name="position_v">325</field>
        </record>

        <record id="table_07" model="restaurant.table">
            <field name="table_number">7</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">407</field>
            <field name="position_v">406</field>
        </record>

        <record id="table_08" model="restaurant.table">
            <field name="table_number">8</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">4</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">582</field>
            <field name="position_v">406</field>
        </record>

        <record id="table_09" model="restaurant.table">
            <field name="table_number">9</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">6</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">165</field>
            <field name="height">100</field>
            <field name="position_h">120</field>
            <field name="position_v">560</field>
        </record>

        <record id="table_10" model="restaurant.table">
            <field name="table_number">10</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">6</field>
            <field name="color">rgb(235,109,109)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">407</field>
            <field name="position_v">565</field>
        </record>

        <record id="table_11" model="restaurant.table">
            <field name="table_number">11</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">2</field>
            <field name="color">rgb(172,109,173)</field>
            <field name="shape">square</field>
            <field name="width">90</field>
            <field name="height">90</field>
            <field name="position_h">582</field>
            <field name="position_v">565</field>
        </record>

        <record id="table_12" model="restaurant.table">
            <field name="table_number">12</field>
            <field name="floor_id" ref="pos_restaurant.floor_main" />
            <field name="seats">2</field>
            <field name="color">rgb(172,109,173)</field>
            <field name="shape">square</field>
            <field name="width">165</field>
            <field name="height">100</field>
            <field name="position_h">762</field>
            <field name="position_v">560</field>
        </record>

        <!-- Restaurant Floor: Patio -->

        <record id="floor_patio" model="restaurant.floor">
            <field name="name">Patio</field>
            <field name="background_color">rgb(130, 233, 171)</field>
            <field name="pos_config_ids" eval="[(6, 0, [ref('pos_restaurant.pos_config_main_restaurant')])]" />
        </record>

        <!-- Patio: Left table row -->

        <record id="table_21" model="restaurant.table">
            <field name="table_number">101</field>
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
            <field name="table_number">102</field>
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
            <field name="table_number">103</field>
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
            <field name="table_number">104</field>
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
            <field name="table_number">105</field>
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
            <field name="table_number">106</field>
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
            <field name="table_number">107</field>
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
            <field name="table_number">108</field>
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
            <field name="table_number">109</field>
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
            <field name="table_number">110</field>
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
            <field name="table_number">111</field>
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
            <field name="table_number">112</field>
            <field name="floor_id" ref="pos_restaurant.floor_patio" />
            <field name="seats">4</field>
            <field name="color">rgb(235,191,109)</field>
            <field name="shape">square</field>
            <field name="width">130</field>
            <field name="height">120</field>
            <field name="position_h">560</field>
            <field name="position_v">315</field>
        </record>

        <!-- Open Session -->
        <record id="customer_1" model="res.partner">
            <field name="name">John Doe</field>
        </record>

        <record id="pos_open_session_2" model="pos.session" forcecreate="False">
            <field name="name">OpenSession/0003</field>
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
            <field name="partner_id" ref="customer_1" />
            <field name="table_id" ref="table_01" />
            <field name="customer_count">8</field>
            <field name="uuid">b3abf526-e575-4c29-a1b7-0264e21c6dda</field>
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
            <field name="uuid">42ca3fb9-dc7a-4b4b-bb42-9027f07569e6</field>
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
            <field name="uuid">e5b8c7fc-d279-4285-a5c3-5e289043d9d8</field>
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
            <field name="partner_id" ref="customer_1" />
            <field name="table_id" ref="table_02" />
            <field name="customer_count">3</field>
            <field name="uuid">b3abf526-e575-4c29-a1b7-0264e21c6ddb</field>
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
            <field name="uuid">42ca3fb9-dc7a-4b4b-bb42-9027f07569e7</field>
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
            <field name="uuid">e5b8c7fc-d279-4285-a5c3-5e289043d9d9</field>
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
            <field name="partner_id" ref="customer_1" />
            <field name="table_id" ref="table_04" />
            <field name="customer_count">5</field>
            <field name="uuid">b3abf526-e575-4c29-a1b7-0264e21c6ddc</field>
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
            <field name="uuid">42ca3fb9-dc7a-4b4b-bb42-9027f07569e8</field>
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
            <field name="uuid">e5b8c7fc-d279-4285-a5c3-5e289043d9da</field>
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
            <field name="partner_id" ref="customer_1" />
            <field name="table_id" ref="table_06" />
            <field name="customer_count">1</field>
            <field name="uuid">b3abf526-e575-4c29-a1b7-0264e21c6ddd</field>
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
            <field name="uuid">42ca3fb9-dc7a-4b4b-bb42-9027f07569e9</field>
        </record>

        <function model="pos.session" name="_set_last_order_preparation_change"
            eval="[[ref('pos_open_order_2'), ref('pos_open_order_3'), ref('pos_open_order_4')]]"/>
    </data>
</odoo>

```

## File: data\scenarios\bar_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
		<record id="pos_category_cocktails" model="pos.category">
			<field name="name">Cocktails</field>
			<field name="image_128" type="base64" file="point_of_sale/static/img/cocktail-icon.png" />
			<field name="sequence">11</field>
		</record>

        <record id="pos_category_soft_drinks" model="pos.category">
			<field name="name">Soft drinks</field>
			<field name="image_128" type="base64" file="pos_restaurant/static/img/soft-drink-icon.png" />
			<field name="sequence">12</field>
		</record>

		<!-- Cocktails products -->
		<record model="product.product" id="product_cosmopolitan">
			<field name="name">Cosmopolitan</field>
			<field name="list_price">12.00</field>
			<field name="standard_price">10.8</field>
			<field name="description_sale">Cranberry Jus, lime jus, vodka and Cointreau</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_cosmopolitan.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_margarita">
			<field name="name">Margarita</field>
			<field name="list_price">12.00</field>
			<field name="standard_price">10.8</field>
			<field name="description_sale">Tequila Jose Cuervo, lime jus, sugar cane Cointreau</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_margarita.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_moscow_mule">
			<field name="name">Moscow Mule</field>
			<field name="list_price">10.00</field>
			<field name="standard_price">9.0</field>
			<field name="description_sale">Vodka 42 Below, lime, sugar, ginger beer</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_moscow_mule.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_pina_colada">
			<field name="name">Pina colada</field>
			<field name="list_price">13.00</field>
			<field name="standard_price">11.7</field>
			<field name="description_sale">White rhum, Malibu, Batida de coco, coconut liqueur, pineapple juice</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_pina_colada.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_aperol_spritz">
			<field name="name">Aperol Spritz</field>
			<field name="list_price">9.00</field>
			<field name="standard_price">8.1</field>
			<field name="description_sale">Prosecco, aperol, soda</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_aperol_spritz.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_old_fashioned">
			<field name="name">Old Fashioned</field>
			<field name="list_price">14.00</field>
			<field name="standard_price">12.6</field>
			<field name="description_sale">Bourbon, bitters, sugar, and a twist of citrus zest.</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_old_fashioned.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_mojito">
			<field name="name">Mojito</field>
			<field name="list_price">11.00</field>
			<field name="standard_price">9.9</field>
			<field name="description_sale">White rum, sugar, lime juice, soda water, and mint.</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_mojito.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_mai_tai">
			<field name="name">Mai Tai</field>
			<field name="list_price">13.00</field>
			<field name="standard_price">11.7</field>
			<field name="description_sale">Rum, lime juice, orgeat syrup, and orange liqueur.</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_mai_tai.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_whiskey_sour">
			<field name="name">Whiskey Sour</field>
			<field name="list_price">12.00</field>
			<field name="standard_price">10.8</field>
			<field name="description_sale">Whiskey, lemon juice, sugar, and a dash of egg white.</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_whiskey_sour.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>
		<record model="product.product" id="product_negroni">
			<field name="name">Negroni</field>
			<field name="list_price">12.00</field>
			<field name="standard_price">10.8</field>
			<field name="description_sale">Gin, vermouth rosso, Campari, and an orange peel.</field>
			<field name="type">consu</field>
			<field name="weight">0.01</field>
			<field name="uom_id" ref="uom.product_uom_unit"/>
			<field name="uom_po_id" ref="uom.product_uom_unit"/>
			<field name="image_1920" type="base64" file="pos_restaurant/static/img/product_negroni.png"/>
			<field name="available_in_pos" eval="True"/>
			<field name="categ_id" ref="product.product_category_1"/>
			<field name="pos_categ_ids" eval="[(6, 0, [ref('pos_category_cocktails')])]" />
		</record>

		<!-- Drinks (use drinks from restaurant scenario) -->
        <record id="coke" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Coca-Cola</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(4, ref('pos_category_soft_drinks'))]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-coke.png"/>
        </record>
        <record id="water" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Water</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(4, ref('pos_category_soft_drinks'))]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-water.png"/>
        </record>
        <record id="minute_maid" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Minute Maid</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(4, ref('pos_category_soft_drinks'))]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-minute_maid.png"/>
        </record>
        <record id="green_tea" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.70</field>
            <field name="name">Green Tea</field>
            <field name="pos_categ_ids" eval="[(4, ref('pos_category_soft_drinks'))]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-green_tea.png"/>
        </record>
        <record id="ice_tea" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Ice Tea</field>
            <field name="pos_categ_ids" eval="[(4, ref('pos_category_soft_drinks'))]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-ice_tea.png"/>
        </record>
        <record id="schweppes" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Schweppes</field>
            <field name="pos_categ_ids" eval="[(4, ref('pos_category_soft_drinks'))]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-schweppes.png"/>
        </record>
        <record id="fanta" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Fanta</field>
            <field name="pos_categ_ids" eval="[(4, ref('pos_category_soft_drinks'))]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-fanta.png"/>
        </record>
	</data>
</odoo>

```

## File: data\scenarios\restaurant_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.group_user" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('product.group_product_variant'))]"/>
        </record>

        <!-- Restaurant data scenario -->
        <record id="food" model="pos.category">
            <field name="name">Food</field>
            <field name="image_128" type="base64" file="pos_restaurant/static/img/food_category.png" />
            <field name="sequence">9</field>
        </record>

        <record id="drinks" model="pos.category">
            <field name="name">Drinks</field>
            <field name="image_128" type="base64" file="pos_restaurant/static/img/drink_category.png" />
            <field name="sequence">10</field>
        </record>

        <record id="product_category_pos_food" model="product.category">
            <field name="parent_id" ref="point_of_sale.product_category_pos"/>
            <field name="name">Food</field>
        </record>

        <!-- Food products -->
        <record model="product.product" id="pos_food_bacon">
            <field name="name">Bacon Burger</field>
            <field name="list_price">15.50</field>
            <field name="standard_price">13.95</field>
            <field name="description_sale">200G Irish Black Angus beef, caramelized onions with paprika, chopped iceberg salad, red onions, grilled bacon, tomato sauce, pickles, barbecue sauce</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-burger.png"/>
            <field name="available_in_pos" eval="True"/>
            <field name="categ_id" ref="product.product_category_1"/>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                        'xml_id': 'pos_restaurant.product_bacon_burger_template',
                        'record': obj().env.ref('pos_restaurant.pos_food_bacon').product_tmpl_id,
                        'noupdate': True,
                    }]" />
        </function>

        <record model="product.attribute" id="product_sides_buns_pizza">
            <field name="name">Sides</field>
            <field name="create_variant">no_variant</field>
            <field name="display_type">pills</field>
        </record>

        <record model="product.attribute.value" id="product_attribute_fries">
            <field name="name">Belgian fresh homemade fries</field>
            <field name="attribute_id" ref="product_sides_buns_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_sweet_potato">
            <field name="name">Sweet potato fries</field>
            <field name="attribute_id" ref="product_sides_buns_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_smashed_sweet_potatoes">
            <field name="name">Smashed sweet potatoes</field>
            <field name="attribute_id" ref="product_sides_buns_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_potato_thyme">
            <field name="name">Potatoes with thyme</field>
            <field name="attribute_id" ref="product_sides_buns_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_grilled_vegetables">
            <field name="name">Grilled vegetables</field>
            <field name="attribute_id" ref="product_sides_buns_pizza"/>
        </record>

        <record model="product.template.attribute.line" id="product_attribute_line_bacon_sides">
            <field name="product_tmpl_id" ref="pos_restaurant.product_bacon_burger_template"/>
            <field name="attribute_id" ref="product_sides_buns_pizza"/>
            <field name="value_ids" eval="[(6, 0, [ref('product_attribute_fries'), ref('product_attribute_sweet_potato'), ref('product_attribute_value_smashed_sweet_potatoes'), ref('product_attribute_value_potato_thyme'), ref('product_attribute_value_grilled_vegetables')])]" />
        </record>

        <record model="product.product" id="pos_food_cheeseburger">
            <field name="name">Cheese Burger</field>
            <field name="list_price">13.00</field>
            <field name="standard_price">11.7</field>
            <field name="description_sale">200G Irish Black Angus beef, 9-month matured cheddar cheese, shredded iceberg lettuce, caramelised onions, crushed tomatoes and Chef’s sauce.</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-cheeseburger.png"/>
            <field name="available_in_pos" eval="True"/>
            <field name="categ_id" ref="product.product_category_1"/>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                        'xml_id': 'pos_restaurant.product_cheese_burger_template',
                        'record': obj().env.ref('pos_restaurant.pos_food_cheeseburger').product_tmpl_id,
                        'noupdate': True,
                    }]" />
        </function>

        <record model="product.template.attribute.line" id="product_attribute_line_cheese_side">
            <field name="product_tmpl_id" ref="pos_restaurant.product_cheese_burger_template"/>
            <field name="attribute_id" ref="product_sides_buns_pizza"/>
            <field name="value_ids" eval="[(6, 0, [ref('product_attribute_fries'), ref('product_attribute_sweet_potato'), ref('product_attribute_value_smashed_sweet_potatoes'), ref('product_attribute_value_potato_thyme'), ref('product_attribute_value_grilled_vegetables')])]" />
        </record>

        <record model="product.product" id="pos_food_margherita">
            <field name="name">Pizza Margherita</field>
            <field name="list_price">11.50</field>
            <field name="standard_price">10.35</field>
            <field name="description_sale">Tomato sauce, Agerola mozzarella &quot;fior di latte&quot;, fresh basil</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-ma.png"/>
            <field name="available_in_pos" eval="True"/>
            <field name="categ_id" ref="product.product_category_1"/>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                        'xml_id': 'pos_restaurant.product_pizza_margherita_template',
                        'record': obj().env.ref('pos_restaurant.pos_food_margherita').product_tmpl_id,
                        'noupdate': True,
                    }]" />
        </function>

        <record model="product.attribute" id="product_extra_pizza">
            <field name="name">Extras</field>
            <field name="create_variant">no_variant</field>
            <field name="display_type">multi</field>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_peperoni">
            <field name="name">Pepperoni</field>
            <field name="attribute_id" ref="product_extra_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_mushroom">
            <field name="name">Mushroom</field>
            <field name="attribute_id" ref="product_extra_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_black_olives">
            <field name="name">Black olives</field>
            <field name="attribute_id" ref="product_extra_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_anchovy">
            <field name="name">Anchovy</field>
            <field name="attribute_id" ref="product_extra_pizza"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_extra_cheese">
            <field name="name">Extra cheese</field>
            <field name="attribute_id" ref="product_extra_pizza"/>
        </record>

        <record model="product.template.attribute.line" id="product_attribute_line_pizza_extra">
            <field name="product_tmpl_id" ref="pos_restaurant.product_pizza_margherita_template"/>
            <field name="attribute_id" ref="product_extra_pizza"/>
            <field name="value_ids" eval="[(6, 0, [ref('product_attribute_value_peperoni'), ref('product_attribute_value_mushroom'), ref('product_attribute_value_black_olives'), ref('product_attribute_value_anchovy'), ref('product_attribute_value_extra_cheese')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'pos_restaurant.product_pizza_extra_1',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_extra').product_template_value_ids[0],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_extra_2',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_extra').product_template_value_ids[1],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_extra_3',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_extra').product_template_value_ids[2],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_extra_4',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_extra').product_template_value_ids[3],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_extra_5',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_extra').product_template_value_ids[4],
                'noupdate': True,
            },
            ]"
            />
        </function>

        <record id="pos_restaurant.product_pizza_extra_1" model="product.template.attribute.value">
            <field name="price_extra">3</field>
        </record>
        <record id="pos_restaurant.product_pizza_extra_2" model="product.template.attribute.value">
            <field name="price_extra">2</field>
        </record>
        <record id="pos_restaurant.product_pizza_extra_3" model="product.template.attribute.value">
            <field name="price_extra">1.5</field>
        </record>
        <record id="pos_restaurant.product_pizza_extra_4" model="product.template.attribute.value">
            <field name="price_extra">1.5</field>
        </record>
        <record id="pos_restaurant.product_pizza_extra_5" model="product.template.attribute.value">
            <field name="price_extra">1.5</field>
        </record>

        <record model="product.product" id="pos_food_vege">
            <field name="name">Pizza Vegetarian</field>
            <field name="list_price">16.00</field>
            <field name="standard_price">14.4</field>
            <field name="description_sale">Pizza Vegetarian</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-ve.png"/>
            <field name="available_in_pos" eval="True"/>
            <field name="categ_id" ref="product.product_category_1"/>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                        'xml_id': 'pos_restaurant.product_pizza_vegetarian_template',
                        'record': obj().env.ref('pos_restaurant.pos_food_vege').product_tmpl_id,
                        'noupdate': True,
                    }]" />
        </function>

        <record model="product.template.attribute.line" id="product_attribute_line_pizza_vege_extra">
            <field name="product_tmpl_id" ref="pos_restaurant.product_pizza_vegetarian_template"/>
            <field name="attribute_id" ref="product_extra_pizza"/>
            <field name="value_ids" eval="[(6, 0, [ref('product_attribute_value_peperoni'), ref('product_attribute_value_mushroom'), ref('product_attribute_value_black_olives'), ref('product_attribute_value_anchovy'), ref('product_attribute_value_extra_cheese')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'pos_restaurant.product_pizza_vg_extra_1',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_vege_extra').product_template_value_ids[0],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_vg_extra_2',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_vege_extra').product_template_value_ids[1],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_vg_extra_3',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_vege_extra').product_template_value_ids[2],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_vg_extra_4',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_vege_extra').product_template_value_ids[3],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pizza_vg_extra_5',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pizza_vege_extra').product_template_value_ids[4],
                'noupdate': True,
            },
            ]"
            />
        </function>

        <record id="pos_restaurant.product_pizza_vg_extra_1" model="product.template.attribute.value">
            <field name="price_extra">3</field>
        </record>
        <record id="pos_restaurant.product_pizza_vg_extra_2" model="product.template.attribute.value">
            <field name="price_extra">2</field>
        </record>
        <record id="pos_restaurant.product_pizza_vg_extra_3" model="product.template.attribute.value">
            <field name="price_extra">1.5</field>
        </record>
        <record id="pos_restaurant.product_pizza_vg_extra_4" model="product.template.attribute.value">
            <field name="price_extra">1.5</field>
        </record>
        <record id="pos_restaurant.product_pizza_vg_extra_5" model="product.template.attribute.value">
            <field name="price_extra">1.5</field>
        </record>

        <record model="product.product" id="pos_food_4formaggi">
            <field name="name">Pasta 4 Formaggi</field>
            <field name="list_price">9.50</field>
            <field name="standard_price">8.55</field>
            <field name="description_sale">Pepe, latte, gorgonzola dolce, taleggio, parmigiano reggiano</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta-4f.png"/>
            <field name="available_in_pos" eval="True"/>
            <field name="categ_id" ref="product.product_category_1"/>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                        'xml_id': 'pos_restaurant.product_pasta_4_formaggi_template',
                        'record': obj().env.ref('pos_restaurant.pos_food_4formaggi').product_tmpl_id,
                        'noupdate': True,
                    }]" />
        </function>

        <record model="product.attribute" id="product_extra_pasta">
            <field name="name">Extras</field>
            <field name="create_variant">no_variant</field>
            <field name="display_type">multi</field>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_mushroom_pasta">
            <field name="name">Mushroom</field>
            <field name="attribute_id" ref="product_extra_pasta"/>
        </record>

        <record model="product.attribute.value" id="product_attribute_value_extra_cheese_pasta">
            <field name="name">Extra cheese</field>
            <field name="attribute_id" ref="product_extra_pasta"/>
        </record>

        <record model="product.template.attribute.line" id="product_attribute_line_pasta_extra">
            <field name="product_tmpl_id" ref="pos_restaurant.product_pasta_4_formaggi_template"/>
            <field name="attribute_id" ref="product_extra_pasta"/>
            <field name="value_ids" eval="[(6, 0, [ref('product_attribute_value_extra_cheese_pasta'), ref('product_attribute_value_mushroom_pasta')])]" />
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'pos_restaurant.product_pasta_extra_1',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pasta_extra').product_template_value_ids[0],
                'noupdate': True,
            },
            {
                'xml_id': 'pos_restaurant.product_pasta_extra_2',
                'record': obj().env.ref('pos_restaurant.product_attribute_line_pasta_extra').product_template_value_ids[1],
                'noupdate': True,
            },
            ]"
            />
        </function>

        <record id="pos_restaurant.product_pasta_extra_1" model="product.template.attribute.value">
            <field name="price_extra">2</field>
        </record>
        <record id="pos_restaurant.product_pasta_extra_2" model="product.template.attribute.value">
            <field name="price_extra">1.5</field>
        </record>

        <record id="pos_food_funghi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">7.0</field>
            <field name="name">Funghi</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pizza-fu.png"/>
        </record>
        <record id="pos_food_bolo" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.5</field>
            <field name="name">Pasta Bolognese</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-pasta.png"/>
        </record>
        <record id="pos_food_chicken" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.0</field>
            <field name="name">Chicken Curry Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-sandwich.png"/>
        </record>
        <record id="pos_food_tuna" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.0</field>
            <field name="name">Spicy Tuna Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-tuna.png"/>
        </record>
        <record id="pos_food_mozza" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.9</field>
            <field name="name">Mozzarella Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-mozza.png"/>
        </record>
        <record id="pos_food_club" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.4</field>
            <field name="name">Club Sandwich</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-club.png"/>
        </record>
        <record id="pos_food_maki" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">12.0</field>
            <field name="name">Lunch Maki 18pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-maki.png"/>
        </record>
        <record id="pos_food_salmon" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">13.80</field>
            <field name="name">Lunch Salmon 20pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon.png"/>
        </record>
        <record id="pos_food_temaki" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">14.0</field>
            <field name="name">Lunch Temaki mix 3pc</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-temaki.png"/>
        </record>
        <record id="pos_food_chirashi" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">9.25</field>
            <field name="name">Salmon and Avocado</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="categ_id" ref="pos_restaurant.product_category_pos_food"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-salmon-avocado.png"/>
        </record>

        <!-- Drinks -->
        <record id="coke" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Coca-Cola</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-coke.png"/>
        </record>

        <record id="water" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Water</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-water.png"/>
        </record>

        <record id="minute_maid" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Minute Maid</field>
            <field name="weight">0.01</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-minute_maid.png"/>
        </record>

        <record id="espresso" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.70</field>
            <field name="name">Espresso</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-espresso.png"/>
        </record>

        <record id="green_tea" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">4.70</field>
            <field name="name">Green Tea</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-green_tea.png"/>
        </record>

        <record id="milkshake_banana" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">3.60</field>
            <field name="name">Milkshake Banana</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-milkshake_banana.png"/>
        </record>

        <record id="ice_tea" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Ice Tea</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-ice_tea.png"/>
        </record>

        <record id="schweppes" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Schweppes</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-schweppes.png"/>
        </record>

        <record id="fanta" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">2.20</field>
            <field name="name">Fanta</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('drinks')])]"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/th-fanta.png"/>
        </record>

        <!-- Combo -->
        <record id="burger_combo" model="product.combo">
            <field name="name">Burgers Choice</field>
            <field
                name="combo_item_ids"
                eval="[
                    Command.clear(),
                    Command.create({
                        'product_id': ref('pos_food_cheeseburger'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('pos_food_bacon'),
                        'extra_price': 0,
                    }),
                ]"
            />
        </record>

        <record id="drink_combo" model="product.combo">
            <field name="name">Drinks choice</field>
            <field
                name="combo_item_ids"
                eval="[
                    Command.clear(),
                    Command.create({
                        'product_id': ref('coke'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('water'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('minute_maid'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('milkshake_banana'),
                        'extra_price': 2,
                    }),
                ]"
            />
        </record>

        <record id="burger_drink_combo" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="list_price">10</field>
            <field name="name">Burger Menu Combo</field>
            <field name="type">combo</field>
            <field name="purchase_ok">False</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="pos_restaurant/static/img/combo-hamb.png"/>
            <field name="combo_ids" eval="[(6, 0, [ref('drink_combo'), ref('burger_combo')])]"/>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('food')])]"/>
            <field name="taxes_id" eval="[(5,)]"/>  <!-- no taxes -->
            <field name="supplier_taxes_id" eval="[(5,)]"/>
        </record>
    </data>
</odoo>

```

## File: models\account_fiscal_position.py

```python
from odoo import models, api
from odoo.osv.expression import OR


class AccountFiscalPosition(models.Model):
    _inherit = 'account.fiscal.position'

    @api.model
    def _load_pos_data_domain(self, data):
        params = super()._load_pos_data_domain(data)
        params = OR([params, [('id', '=', data['pos.config']['data'][0]['takeaway_fp_id'])]])
        return params

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
import json
from collections import defaultdict
from odoo.tools import convert


class PosConfig(models.Model):
    _inherit = 'pos.config'

    iface_splitbill = fields.Boolean(string='Bill Splitting', help='Enables Bill Splitting in the Point of Sale.')
    iface_printbill = fields.Boolean(string='Bill Printing', help='Allows to print the Bill before payment.')
    floor_ids = fields.Many2many('restaurant.floor', string='Restaurant Floors', help='The restaurant floors served by this point of sale.', copy=False)
    set_tip_after_payment = fields.Boolean('Set Tip After Payment', help="Adjust the amount authorized by payment terminals to add a tip after the customers left or at the end of the day.")
    module_pos_restaurant_appointment = fields.Boolean("Table Booking")
    takeaway = fields.Boolean("Takeaway", help="Allow to create orders for takeaway customers.")
    takeaway_fp_id = fields.Many2one(
        'account.fiscal.position',
        string='Alternative Fiscal Position',
        help='This is useful for restaurants with onsite and take-away services that imply specific tax rates.',
    )

    def _get_forbidden_change_fields(self):
        forbidden_keys = super(PosConfig, self)._get_forbidden_change_fields()
        forbidden_keys.append('floor_ids')
        return forbidden_keys

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            is_restaurant = 'module_pos_restaurant' in vals and vals['module_pos_restaurant']
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

    def _setup_default_floor(self, pos_config):
        if not pos_config.floor_ids:
            main_floor = self.env['restaurant.floor'].create({
                'name': pos_config.company_id.name,
                'pos_config_ids': [(4, pos_config.id)],
            })
            self.env['restaurant.table'].create({
                'table_number': 1,
                'floor_id': main_floor.id,
                'seats': 1,
                'position_h': 100,
                'position_v': 100,
                'width': 130,
                'height': 130,
            })

    @api.model
    def _load_bar_data(self):
        convert.convert_file(self.env, 'pos_restaurant', 'data/scenarios/bar_data.xml', None, noupdate=True, mode='init', kind='data')

    @api.model
    def _load_restaurant_data(self):
        convert.convert_file(self.env, 'pos_restaurant', 'data/scenarios/restaurant_data.xml', None, noupdate=True, mode='init', kind='data')

    @api.model
    def load_onboarding_bar_scenario(self):
        ref_name = 'pos_restaurant.pos_config_main_bar'
        if not self.env.ref(ref_name, raise_if_not_found=False):
            self._load_bar_data()
        journal, payment_methods_ids = self._create_journal_and_payment_methods(cash_journal_vals={'name': 'Cash Bar', 'show_on_dashboard': False})
        bar_categories = self.get_categories([
            'pos_restaurant.pos_category_cocktails',
            'pos_restaurant.pos_category_soft_drinks',
        ])
        config = self.env['pos.config'].create({
            'name': 'Bar',
            'company_id': self.env.company.id,
            'journal_id': journal.id,
            'payment_method_ids': payment_methods_ids,
            'limit_categories': True,
            'iface_available_categ_ids': bar_categories,
            'iface_splitbill': True,
            'module_pos_restaurant': True,
        })
        self.env['ir.model.data']._update_xmlids([{
            'xml_id': self._get_suffixed_ref_name(ref_name),
            'record': config,
            'noupdate': True,
        }])

    @api.model
    def load_onboarding_restaurant_scenario(self):
        ref_name = 'pos_restaurant.pos_config_main_restaurant'
        if not self.env.ref(ref_name, raise_if_not_found=False):
            self._load_restaurant_data()

        journal, payment_methods_ids = self._create_journal_and_payment_methods(cash_journal_vals={'name': 'Cash Restaurant', 'show_on_dashboard': False})
        restaurant_categories = self.get_categories([
            'pos_restaurant.food',
            'pos_restaurant.drinks',
        ])
        config = self.env['pos.config'].create({
            'name': _('Restaurant'),
            'company_id': self.env.company.id,
            'journal_id': journal.id,
            'payment_method_ids': payment_methods_ids,
            'limit_categories': True,
            'iface_available_categ_ids': restaurant_categories,
            'iface_splitbill': True,
            'module_pos_restaurant': True,
        })
        self.env['ir.model.data']._update_xmlids([{
            'xml_id': self._get_suffixed_ref_name(ref_name),
            'record': config,
            'noupdate': True,
        }])
        if self.env.company.id == self.env.ref('base.main_company').id:
            existing_session = self.env.ref('pos_restaurant.pos_closed_session_3', raise_if_not_found=False)
            if not existing_session:
                convert.convert_file(self.env, 'pos_restaurant', 'data/restaurant_session_floor.xml', None, noupdate=True, mode='init', kind='data')

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models

class PosOrder(models.Model):
    _inherit = 'pos.order'

    table_id = fields.Many2one('restaurant.table', string='Table', help='The table where this order was served', index='btree_not_null', readonly=True)
    customer_count = fields.Integer(string='Guests', help='The amount of customers that have been served by this order.', readonly=True)
    takeaway = fields.Boolean(string="Take Away", default=False)

    def _get_open_order(self, order):
        config_id = self.env['pos.session'].browse(order.get('session_id')).config_id
        if not config_id.module_pos_restaurant:
            return super()._get_open_order(order)

        domain = []
        if order.get('table_id', False) and order.get('state') == 'draft':
            domain += ['|', ('uuid', '=', order.get('uuid')), ('table_id', '=', order.get('table_id')), ('state', '=', 'draft')]
        else:
            domain += [('uuid', '=', order.get('uuid'))]
        return self.env["pos.order"].search(domain, limit=1)

    @api.model
    def remove_from_ui(self, server_ids):
        tables = self.env['pos.order'].search([('id', 'in', server_ids)]).table_id
        order_ids = super().remove_from_ui(server_ids)
        self.send_table_count_notification(tables)
        return order_ids

    @api.model
    def sync_from_ui(self, orders):
        result = super().sync_from_ui(orders)

        if self.env.context.get('table_ids'):
            order_ids = [order['id'] for order in result['pos.order']]
            table_orders = self.search([
                "&",
                ('table_id', 'in', self.env.context['table_ids']),
                ('state', '=', 'draft'),
                ('id', 'not in', order_ids)
            ])

            if len(table_orders) > 0:
                config_id = table_orders[0].config_id.id
                result['pos.order'].extend(table_orders.read(table_orders._load_pos_data_fields(config_id), load=False))
                result['pos.payment'].extend(table_orders.payment_ids.read(table_orders.payment_ids._load_pos_data_fields(config_id), load=False))
                result['pos.order.line'].extend(table_orders.lines.read(table_orders.lines._load_pos_data_fields(config_id), load=False))
                result['pos.pack.operation.lot'].extend(table_orders.lines.pack_lot_ids.read(table_orders.lines.pack_lot_ids._load_pos_data_fields(config_id), load=False))
                result["product.attribute.custom.value"].extend(table_orders.lines.custom_attribute_value_ids.read(table_orders.lines.custom_attribute_value_ids._load_pos_data_fields(config_id), load=False))

        return result

    def send_table_count_notification(self, table_ids):
         # Cannot remove the method in stable
        pass

    def action_pos_order_cancel(self):
        result = super().action_pos_order_cancel()
        if self.table_id:
            self.send_table_count_notification(self.table_id)
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

from odoo import api, fields, models, _, Command
from odoo.exceptions import UserError


class RestaurantFloor(models.Model):

    _name = 'restaurant.floor'
    _description = 'Restaurant Floor'
    _order = "sequence, name"
    _inherit = ['pos.load.mixin']

    name = fields.Char('Floor Name', required=True)
    pos_config_ids = fields.Many2many('pos.config', string='Point of Sales', domain="[('module_pos_restaurant', '=', True)]")
    background_image = fields.Binary('Background Image')
    background_color = fields.Char('Background Color', help='The background color of the floor in a html-compatible format', default='rgb(249,250,251)')
    table_ids = fields.One2many('restaurant.table', 'floor_id', string='Tables')
    sequence = fields.Integer('Sequence', default=1)
    active = fields.Boolean(default=True)
    floor_background_image = fields.Image(string='Floor Background Image')

    @api.model
    def _load_pos_data_domain(self, data):
        return [('pos_config_ids', '=', data['pos.config']['data'][0]['id'])]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['name', 'background_color', 'table_ids', 'sequence', 'pos_config_ids', 'floor_background_image']

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_pos_session(self):
        confs = self.mapped('pos_config_ids').filtered(lambda c: c.module_pos_restaurant)
        opened_session = self.env['pos.session'].search([('config_id', 'in', confs.ids), ('state', '!=', 'closed')])
        if opened_session and confs:
            error_msg = _("You cannot remove a floor that is used in a PoS session, close the session(s) first: \n")
            for floor in self:
                for session in opened_session:
                    if floor in session.config_id.floor_ids:
                        error_msg += _("Floor: %(floor)s - PoS Config: %(config)s \n", floor=floor.name, config=session.config_id.name)
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
    def sync_from_ui(self, name, background_color, config_id):
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

        return True

class RestaurantTable(models.Model):

    _name = 'restaurant.table'
    _description = 'Restaurant Table'
    _inherit = ['pos.load.mixin']

    floor_id = fields.Many2one('restaurant.floor', string='Floor')
    table_number = fields.Integer('Table Number', required=True, help='The number of the table as displayed on the floor plan', default=0)
    shape = fields.Selection([('square', 'Square'), ('round', 'Round')], string='Shape', required=True, default='square')
    position_h = fields.Float('Horizontal Position', default=10,
        help="The table's horizontal position from the left side to the table's center, in pixels")
    position_v = fields.Float('Vertical Position', default=10,
        help="The table's vertical position from the top to the table's center, in pixels")
    width = fields.Float('Width', default=50, help="The table's width in pixels")
    height = fields.Float('Height', default=50, help="The table's height in pixels")
    seats = fields.Integer('Seats', default=1, help="The default number of customer served at this table.")
    color = fields.Char('Color', help="The table's color, expressed as a valid 'background' CSS property value", default="#35D374")
    parent_id = fields.Many2one('restaurant.table', string='Parent Table', help="The parent table if this table is part of a group of tables")
    active = fields.Boolean('Active', default=True, help='If false, the table is deactivated and will not be available in the point of sale')

    @api.depends('table_number', 'floor_id')
    def _compute_display_name(self):
        for table in self:
            table.display_name = f"{table.floor_id.name}, {table.table_number}"

    @api.model
    def _load_pos_data_domain(self, data):
        return [('active', '=', True), ('floor_id', 'in', [floor['id'] for floor in data['restaurant.floor']['data']])]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['table_number', 'width', 'height', 'position_h', 'position_v', 'parent_id', 'shape', 'floor_id', 'color', 'seats', 'active']

    def are_orders_still_in_draft(self):
        draft_orders_count = self.env['pos.order'].search_count([('table_id', 'in', self.ids), ('state', '=', 'draft')])

        if draft_orders_count > 0:
            raise UserError(_("You cannot delete a table when orders are still in draft for this table."))

        return True

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

from odoo import models, api
import json

class PosSession(models.Model):
    _inherit = 'pos.session'

    @api.model
    def _load_pos_data_models(self, config_id):
        data = super()._load_pos_data_models(config_id)
        if self.config_id.module_pos_restaurant:
            data += ['restaurant.floor', 'restaurant.table']
        return data

    @api.model
    def _set_last_order_preparation_change(self, order_ids):
        for order_id in order_ids:
            order = self.env['pos.order'].browse(order_id)
            last_order_preparation_change = {
                'lines': {},
                'generalNote': '',
            }
            for orderline in order['lines']:
                last_order_preparation_change['lines'][orderline.uuid + " - "] = {
                    "uuid": orderline.uuid,
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
    pos_iface_printbill = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_iface_splitbill = fields.Boolean(compute='_compute_pos_module_pos_restaurant', store=True, readonly=False)
    pos_set_tip_after_payment = fields.Boolean(compute='_compute_pos_set_tip_after_payment', store=True, readonly=False)
    pos_module_pos_restaurant_appointment = fields.Boolean(related="pos_config_id.module_pos_restaurant_appointment", readonly=False)
    pos_takeaway = fields.Boolean(related="pos_config_id.takeaway", readonly=False)
    pos_takeaway_fp_id = fields.Many2one(related="pos_config_id.takeaway_fp_id", readonly=False)

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
from . import account_fiscal_position

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

## File: static\img\plan.svg

```svg
<svg width="23" height="23" fill="none" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M8.214 6.16a.41.41 0 0 1-.41.411H.41A.41.41 0 0 0 0 6.982V22.59c0 .227.184.411.41.411h22.18a.41.41 0 0 0 .41-.41V.41a.41.41 0 0 0-.41-.41H8.624a.41.41 0 0 0-.41.41v5.75Zm13.143-4.517h-11.5V6.16c0 1.134-.92 2.053-2.053 2.053H1.643v13.143h6.571v-7.393h9.036v1.643H9.857v5.75h11.5V1.643Z" fill="#000"/></svg>
```

## File: static\img\table.svg

```svg
<svg width="23" height="23" fill="none" xmlns="http://www.w3.org/2000/svg"><g clip-path="url(#a)" fill="#000"><path d="M13.79 4.356c.443-.743.71-1.617.71-2.356 0-1.657-1.343-2-3-2s-3 .343-3 2c0 .738.267 1.613.709 2.356A7.495 7.495 0 0 1 11.499 4c.8 0 1.57.125 2.292.356ZM17.25 11.5a5.75 5.75 0 1 1-11.5 0 5.75 5.75 0 0 1 11.5 0ZM9.21 18.644c-.443.743-.71 1.618-.71 2.356 0 1.657 1.343 2 3 2s3-.343 3-2c0-.738-.267-1.613-.71-2.356A7.496 7.496 0 0 1 11.5 19a7.495 7.495 0 0 1-2.29-.356ZM18.993 11.5c0 .952-.177 1.863-.5 2.7.771.497 1.713.8 2.5.8 1.657 0 2-1.343 2-3s-.343-3-2-3c-.698 0-1.518.238-2.233.638.152.596.233 1.22.233 1.862ZM2 9c.697 0 1.517.238 2.231.637a7.515 7.515 0 0 0-.233 1.863c0 .953.178 1.864.501 2.702C3.728 14.697 2.787 15 2 15c-1.657 0-2-1.343-2-3s.343-3 2-3Z"/></g><defs><clipPath id="a"><path fill="#fff" d="M0 0h23v23H0z"/></clipPath></defs></svg>
```

## File: static\src\app\bill_screen\bill_screen.js

```javascript
import { Dialog } from "@web/core/dialog/dialog";
import { OrderReceipt } from "@point_of_sale/app/screens/receipt_screen/receipt/order_receipt";
import { Component, useState } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { useService } from "@web/core/utils/hooks";

export class BillScreen extends Component {
    static template = "pos_restaurant.BillScreen";
    static components = { OrderReceipt, Dialog };
    static props = {
        close: Function,
    };
    setup() {
        this.pos = usePos();
        this.printer = useState(useService("printer"));
    }
    async print() {
        await this.pos.printReceipt({
            printBillActionTriggered: true,
        });
    }
}

```

## File: static\src\app\bill_screen\bill_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.BillScreen">
        <Dialog title.translate="Bill Printing" bodyClass="'text-center'">
            <div class="d-inline-block m-3 p-3 border rounded bg-view">
                <OrderReceipt data="{...pos.orderExportForPrinting(pos.get_order()), isBill: true, show_change: false }" formatCurrency="env.utils.formatCurrency" />
            </div>
            <t t-set-slot="footer">
                <div class="d-flex w-100 justify-content-start gap-2">
                    <div class="button print btn btn-lg btn-primary" t-on-click="print">
                        <i t-attf-class="fa {{printer.state.isPrinting ? 'fa-fw fa-spin fa-circle-o-notch' : 'fa-print'}} me-1" />
                        Print
                    </div>
                </div>
            </t>
        </Dialog>
    </t>

</templates>

```

## File: static\src\app\control_buttons\control_buttons.js

```javascript
import { ControlButtons } from "@point_of_sale/app/screens/product_screen/control_buttons/control_buttons";
import { SelectPartnerButton } from "@point_of_sale/app/screens/product_screen/control_buttons/select_partner_button/select_partner_button";
import { OrderReceipt } from "@point_of_sale/app/screens/receipt_screen/receipt/order_receipt";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
import { useAsyncLockedMethod } from "@point_of_sale/app/utils/hooks";
import { patch } from "@web/core/utils/patch";
import { BillScreen } from "@pos_restaurant/app/bill_screen/bill_screen";
import { TextInputPopup } from "@point_of_sale/app/utils/input_popups/text_input_popup";

patch(ControlButtons.prototype, {
    setup() {
        super.setup(...arguments);
        this.alert = useService("alert");
        this.printer = useService("printer");
        this.clickPrintBill = useAsyncLockedMethod(this.clickPrintBill);
    },
    async clickPrintBill() {
        // Need to await to have the result in case of automatic skip screen.
        (await this.printer.print(OrderReceipt, {
            data: this.pos.orderExportForPrinting(this.pos.get_order()),
            formatCurrency: this.env.utils.formatCurrency,
        })) || this.dialog.add(BillScreen);
    },
    clickTableGuests() {
        this.dialog.add(NumberPopup, {
            startingValue: this.currentOrder?.getCustomerCount() || 0,
            title: _t("Guests?"),
            feedback: (buffer) => {
                const value = this.env.utils.formatCurrency(
                    this.currentOrder?.amountPerGuest(parseInt(buffer, 10) || 0) || 0
                );
                return value ? `${value} / ${_t("Guest")}` : "";
            },
            getPayload: (inputNumber) => {
                const guestCount = parseInt(inputNumber, 10) || 0;
                if (guestCount == 0 && this.currentOrder.lines.length === 0) {
                    this.pos.removeOrder(this.currentOrder);
                    this.pos.showScreen("FloorScreen");
                    return;
                }
                this.currentOrder.setCustomerCount(guestCount);
                this.pos.addPendingOrder([this.currentOrder.id]);
            },
        });
    },
    clickTransferOrder() {
        this.dialog.closeAll();
        this.pos.isOrderTransferMode = true;
        const orderUuid = this.pos.get_order().uuid;
        this.pos.get_order().setBooked(true);
        this.pos.showScreen("FloorScreen");
        const onClickWhileTransfer = async (ev) => {
            if (ev.target.closest(".button-floor")) {
                return;
            }
            this.pos.isOrderTransferMode = false;
            const tableElement = ev.target.closest(".table");
            if (!tableElement) {
                return;
            }
            const table = this.pos.getTableFromElement(tableElement);
            await this.pos.transferOrder(orderUuid, table);
            this.pos.setTableFromUi(table);
            document.removeEventListener("click", onClickWhileTransfer);
        };
        document.addEventListener("click", onClickWhileTransfer);
    },
    async clickTakeAway() {
        const isTakeAway = !this.currentOrder.takeaway;
        const defaultFp = this.pos.config?.default_fiscal_position_id ?? false;
        const takeawayFp = this.pos.config.takeaway_fp_id;

        this.currentOrder.takeaway = isTakeAway;
        this.currentOrder.update({ fiscal_position_id: isTakeAway ? takeawayFp : defaultFp });
        if (typeof this.currentOrder.id == "number") {
            this.pos.data.write("pos.order", [this.currentOrder.id], {
                takeaway: isTakeAway ? true : false,
            });
        }
    },
    editFloatingOrderName(order) {
        this.dialog.add(TextInputPopup, {
            title: _t("Edit Order Name"),
            placeholder: _t("18:45 John 4P"),
            startingValue: order.floating_order_name || "",
            getPayload: async (newName) => {
                if (typeof order.id == "number") {
                    this.pos.data.write("pos.order", [order.id], {
                        floating_order_name: newName,
                    });
                } else {
                    order.floating_order_name = newName;
                }
            },
        });
    },
});
patch(ControlButtons, {
    components: {
        ...ControlButtons.components,
        SelectPartnerButton,
    },
});

```

## File: static\src\app\control_buttons\control_buttons.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.ControlButtons" t-inherit="point_of_sale.ControlButtons" t-inherit-mode="extension">
        <xpath expr="//t[@t-if='props.showRemainingButtons']/div/OrderlineNoteButton" position="after">
            <t t-if="pos.config.module_pos_restaurant">
                <!-- All buttons always displayed -->
                <t t-if="pos.config.iface_printbill">
                    <button t-att-class="buttonClass"
                        t-att-disabled="!pos.get_order()?.get_orderlines()?.length"
                        t-on-click="clickPrintBill">
                        <i class="fa fa-print me-1"></i>Bill
                    </button>
                </t>
                <button t-att-class="buttonClass" t-on-click="clickTableGuests">
                    <span t-esc="currentOrder?.getCustomerCount() || 0" class="px-2 py-1 rounded-circle text-bg-dark fw-bolder small me-1"/>
                    <span>Guests</span>
                </button>
            </t>
        </xpath>
        <xpath expr="//t[@t-if='props.showRemainingButtons']/div/OrderlineNoteButton" position="after">
            <!-- All these buttons will only be displayed in a dialog -->
            <t t-if="pos.config.module_pos_restaurant">
                <button class="btn btn-secondary btn-lg py-5"
                    t-att-disabled="pos.get_order()?.get_orderlines()?.reduce((sum, line) => sum + line.qty, 0) lt 2"
                    t-on-click="() => pos.showScreen('SplitBillScreen')">
                    <i class="fa fa-files-o me-1"/>Split
                </button>
                <button class="btn btn-secondary btn-lg py-5" t-on-click.stop="() => this.clickTransferOrder()">
                    <i class="oi oi-arrow-right me-1" />Transfer / Merge
                </button>
                <button t-if="!pos.get_order()?.table_id" class="btn btn-secondary btn-lg py-5" t-on-click="() => this.editFloatingOrderName(this.pos.get_order())">
                    <i class="fa fa-pencil-square-o me-1" />Edit Order Name
                </button>
            </t>
        </xpath>
        <xpath expr="//button[hasclass('o_fiscal_position_button')]" position="after">
            <button t-if="pos.config.takeaway"
                t-attf-class="btn-secondary btn btn-lg py-5"
                t-on-click="clickTakeAway">
                <i t-attf-class="{{ currentOrder.takeaway ? 'fa fa-cutlery' : 'fa fa-car' }} me-1"></i>
                <span t-if="currentOrder.takeaway">Switch to Dine in</span>
                <span t-else="">Switch to Takeaway</span>
            </button>
        </xpath>
        <xpath expr="//button[hasclass('o_fiscal_position_button')]" position="attributes">
            <attribute name="t-if">!pos.config.takeaway</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\app\floor_screen\floor_screen.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";
import { debounce } from "@web/core/utils/timing";
import { registry } from "@web/core/registry";

import { TextInputPopup } from "@point_of_sale/app/utils/input_popups/text_input_popup";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { useService } from "@web/core/utils/hooks";
import { Component, onMounted, useRef, useState, useEffect, useExternalListener } from "@odoo/owl";
import { ask } from "@point_of_sale/app/store/make_awaitable_dialog";
import { loadImage } from "@point_of_sale/utils";
import { getDataURLFromFile } from "@web/core/utils/urls";
import { hasTouch } from "@web/core/browser/feature_detection";
import {
    getButtons,
    DECIMAL,
    ZERO,
    BACKSPACE,
} from "@point_of_sale/app/generic_components/numpad/numpad";
import { makeDraggableHook } from "@web/core/utils/draggable_hook_builder_owl";
import { pick } from "@web/core/utils/objects";
import { getOrderChanges } from "@point_of_sale/app/models/utils/order_change";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { useTrackedAsync } from "@point_of_sale/app/utils/hooks";

function constrain(num, min, max) {
    return Math.min(Math.max(num, min), max);
}
/**
 * Gives the minimum and maximum x and y value for an element to prevent it from
 * overflowing outside of another element.
 *
 * @param {HTMLElement} el the element for which we want to get the position
 *  limits
 * @param {HTMLElement} limitEl the element outside of which the main element
 *  shouldn't overflow
 * @returns {{ minX: number, maxX: number, minY: number, maxY: number }} limits
 */
function getLimits(el, limitEl) {
    const limitRect = limitEl.getBoundingClientRect();
    const offsetParentRect = el.offsetParent.getBoundingClientRect();
    return {
        minX: limitRect.left - offsetParentRect.left,
        maxX: limitRect.left - offsetParentRect.left + limitRect.width,
        minY: limitRect.top - offsetParentRect.top,
        maxY: limitRect.top - offsetParentRect.top + limitRect.height,
    };
}
const areElementsIntersecting = (el1, el2) => {
    const rect1 = el1.getBoundingClientRect();
    const rect2 = el2.getBoundingClientRect();
    return !(
        rect1.right < rect2.left ||
        rect1.left > rect2.right ||
        rect1.bottom < rect2.top ||
        rect1.top > rect2.bottom
    );
};
const useDraggable = makeDraggableHook({
    name: "useDraggable",
    onComputeParams({ ctx }) {
        ctx.followCursor = false;
    },
    onWillStartDrag: ({ ctx }) => pick(ctx.current, "element"),
    onDragStart: ({ ctx }) => pick(ctx.current, "element"),
    onDrag: ({ ctx }) => pick(ctx.current, "element"),
    onDrop: ({ ctx }) => pick(ctx.current, "element"),
});

const GRID_SIZE = 10;

export class FloorScreen extends Component {
    static components = { Dropdown, DropdownItem };
    static template = "pos_restaurant.FloorScreen";
    static props = { floor: { type: true, optional: true } };
    static storeOnOrder = false;

    setup() {
        this.pos = usePos();
        this.dialog = useService("dialog");
        this.ui = useService("ui");
        const floor = this.pos.currentFloor;
        this.state = useState({
            selectedFloorId: floor ? floor.id : null,
            floorHeight: "100%",
            floorWidth: "100%",
            selectedTableIds: [],
            potentialLink: null,
        });

        this.doCreateTable = useTrackedAsync(async () => {
            await this.createTable();
        });
        this.floorMapRef = useRef("floor-map-ref");
        this.floorScrollBox = useRef("floor-map-scroll");
        this.map = useRef("map");
        this.alert = useService("alert");
        const getTableElem = (table) => {
            return this.map.el.querySelector(`.tableId-${table.id}`);
        };
        const findIntersectingTableElem = (tableElem) => {
            const table = this.getPosTable(tableElem);
            return [...tableElem.parentElement.getElementsByClassName("table")].find(
                (t) =>
                    t !== tableElem &&
                    areElementsIntersecting(t, tableElem) &&
                    !table.isParent(this.getPosTable(t))
            );
        };
        const TABLE_LINKING_DELAY = 400;
        let offsetX, offsetY;
        const suggestLinkingPositions = () =>
            this.state.potentialLink?.parent &&
            this.state.potentialLink.time + TABLE_LINKING_DELAY < Date.now();

        useExternalListener(window, "keydown", (ev) => {
            const overlayElements = document.querySelectorAll(".o-overlay-item");
            if (
                overlayElements.length == 0 &&
                ev.key === "Escape" &&
                this.pos.isEditMode &&
                this.state.selectedTableIds.length == 0 &&
                !this.state.potentialLink
            ) {
                this.pos.isEditMode = false;
            }
        });
        useDraggable({
            ref: this.map,
            elements: ".table",
            ignore: "span.table-handle",
            enabled: !this.pos.isEditMode,
            onDragStart: (ctx) => {
                ctx.addClass(ctx.element, "shadow");
                if (this.pos.isEditMode) {
                    return;
                }
                const table = this.getPosTable(ctx.element);
                this.state.potentialLink = { child: table };
                table.uiState.initialPosition = pick(table, "position_h", "position_v");
                // This helps when unlinking tables ( to keep the position )
                table.position_h = table.getX();
                table.position_v = table.getY();
                if (table.parent_id) {
                    this.pos.data.write("restaurant.table", [table.id], {
                        parent_id: null,
                    });
                }
            },
            onWillStartDrag: ({ element, x, y }) => {
                offsetX = x - element.getBoundingClientRect().left;
                offsetY = y - element.getBoundingClientRect().top;
            },
            onDrag: ({ element, x, y }) => {
                const table = this.getPosTable(element);
                if (!suggestLinkingPositions()) {
                    table.position_h =
                        x - offsetX + this.map.el.parentElement.parentElement.scrollLeft;
                    table.position_v = y - offsetY - this.map.el.getBoundingClientRect().top;
                    if (this.pos.isEditMode && !this.activeFloor.floor_background_image) {
                        table.position_h -= table.position_h % GRID_SIZE;
                        table.position_v -= table.position_v % GRID_SIZE;
                    }
                    if (this.pos.isEditMode || this.state.potentialLink?.parent) {
                        return;
                    }
                    const potentialParentElem = findIntersectingTableElem(element);
                    if (!potentialParentElem) {
                        this.alert.add("Link Table");
                        return;
                    }
                    this.state.potentialLink = {
                        child: table,
                        parent: this.getPosTable(potentialParentElem),
                        time: Date.now(),
                    };
                    this.alert.add(
                        `Link Table ${table.table_number} with ${this.state.potentialLink.parent.table_number}`
                    );
                    return;
                }
                const { child, parent } = this.state.potentialLink;
                const { left, top, width, height } = getTableElem(parent).getBoundingClientRect();
                const dx = x - left - width / 2;
                const dy = y - top - height / 2;
                if (
                    Math.abs(dx) > parent.width / 2 + child.width / 2 ||
                    Math.abs(dy) > parent.height / 2 + child.height / 2
                ) {
                    this.state.potentialLink = { child: table };
                    return;
                }
                table.setPositionAsIfLinked(
                    this.state.potentialLink.parent,
                    Math.abs(dx) > Math.abs(dy)
                        ? dx > 0
                            ? "left"
                            : "right"
                        : dy < 0
                        ? "top"
                        : "bottom"
                );
            },
            onDrop: ({ element }) => {
                this.alert.dismiss();
                const table = this.getPosTable(element);
                if (this.pos.isEditMode) {
                    this.pos.data.write("restaurant.table", [table.id], {
                        position_h: table.position_h,
                        position_v: table.position_v,
                    });
                    return;
                }
                table.position_h = table.uiState.initialPosition.position_h;
                table.position_v = table.uiState.initialPosition.position_v;
                if (!suggestLinkingPositions()) {
                    this.state.potentialLink = null;
                    return;
                }
                const oToTrans = this.pos.getActiveOrdersOnTable(table)[0];
                if (oToTrans) {
                    this.pos.transferOrder(oToTrans.uuid, this.state.potentialLink.parent);
                }
                this.pos.data.write("restaurant.table", [table.id], {
                    parent_id: this.state.potentialLink.parent.id,
                });
                this.state.potentialLink = null;
            },
        });
        this.useResizeHook();
        onMounted(() => {
            this.pos.openOpeningControl();
            this.pos.searchProductWord = "";
            this.pos.unsetTable();
        });
        useEffect(
            () => {
                this.computeFloorSize();
            },
            () => [this.activeFloor, this.pos.floorPlanStyle]
        );
        useEffect(
            (tableL) => {
                if (hasTouch()) {
                    if (tableL) {
                        this.floorScrollBox.el.classList.remove("overflow-auto");
                        this.floorScrollBox.el.classList.add("overflow-hidden");
                    } else {
                        this.floorScrollBox.el.classList.remove("overflow-hidden");
                        this.floorScrollBox.el.classList.add("overflow-auto");
                    }
                }
            },
            () => [this.state.selectedTableIds.length]
        );
    }
    getPosTable(el) {
        return this.pos.getTableFromElement(el);
    }
    useResizeHook() {
        let startX, startY;
        let startPosH, startPosV, startWidth, startHeight;
        let table;
        useDraggable({
            ref: this.map,
            elements: "span.table-handle",
            onDragStart: (ctx) => {
                table = this.getPosTable(ctx.element.parentElement);
                startX = ctx.x;
                startY = ctx.y;
                startPosH = table.position_h;
                startPosV = table.position_v;
                startWidth = table.width;
                startHeight = table.height;
            },
            onDrag: (ctx) => {
                const newPosition = {
                    minX: startPosH,
                    minY: startPosV,
                    maxX: startPosH + startWidth,
                    maxY: startPosV + startHeight,
                };
                const dx = ctx.x - startX;
                const dy = ctx.y - startY;
                const limits = getLimits(ctx.element.parentElement, this.map.el);
                const MIN_TABLE_SIZE = 30;
                const bounds = {
                    maxX: [startPosH + MIN_TABLE_SIZE, limits.maxX + startWidth],
                    minX: [limits.minX, newPosition.maxX - MIN_TABLE_SIZE],
                    maxY: [startPosV + MIN_TABLE_SIZE, limits.maxY + startHeight],
                    minY: [limits.minY, newPosition.maxY - MIN_TABLE_SIZE],
                };
                const moveX = ctx.element.classList.contains("left") ? "minX" : "maxX";
                const moveY = ctx.element.classList.contains("top") ? "minY" : "maxY";
                newPosition[moveX] = constrain(newPosition[moveX] + dx, ...bounds[moveX]);
                newPosition[moveY] = constrain(newPosition[moveY] + dy, ...bounds[moveY]);
                if (!this.activeFloor.floor_background_image) {
                    newPosition[moveX] -= newPosition[moveX] % GRID_SIZE;
                    newPosition[moveY] -= newPosition[moveY] % GRID_SIZE;
                }
                table.position_h = newPosition.minX;
                table.position_v = newPosition.minY;
                table.width = newPosition.maxX - newPosition.minX;
                table.height = newPosition.maxY - newPosition.minY;
            },
            onDrop: (ctx) => {
                const table = this.getPosTable(ctx.element.parentElement);
                this.pos.data.write(
                    "restaurant.table",
                    [table.id],
                    pick(table, "position_h", "position_v", "width", "height")
                );
            },
        });
    }
    computeFloorSize() {
        if (this.pos.floorPlanStyle === "kanban") {
            this.state.floorHeight = "100%";
            this.state.floorWidth = window.innerWidth + "px";
            return;
        }

        if (!this.activeFloor) {
            return;
        }

        const tables = this.activeFloor.table_ids;
        const floorV = this.floorMapRef.el.clientHeight;
        const floorH = this.floorMapRef.el.offsetWidth;
        const positionH = Math.max(
            ...tables.map((table) => table.position_h + table.width),
            floorH
        );
        const positionV = Math.max(
            ...tables.map((table) => table.position_v + table.height),
            floorV
        );

        if (this.activeFloor.floor_background_image) {
            const img = new Image();
            img.onload = () => {
                const height = Math.max(img.height, positionV);
                const width = Math.max(img.width, positionH);
                this.state.floorHeight = `${height}px`;
                this.state.floorWidth = `${width}px`;
            };
            img.src = "data:image/png;base64," + this.activeFloor.floor_background_image;
        } else {
            this.state.floorHeight = `${positionV}px`;
            this.state.floorWidth = `${positionH}px`;
        }
    }
    get floorBackround() {
        return this.activeFloor.floor_background_image
            ? "data:image/png;base64," + this.activeFloor.floor_background_image
            : "none";
    }
    getTableHandleOffset(table) {
        // min(width/2, height/2) is the real border radius
        // 0.2929 is (1 - cos(45°)) to get in the middle of the border's arc
        return table.shape === "round"
            ? -12 + Math.min(table.width / 2, table.height / 2) * 0.2929
            : -12;
    }
    onClickFloorMap(ev) {
        if (ev.target.closest(".table")) {
            return;
        }
        for (const tableId of this.state.selectedTableIds) {
            const table = this.pos.models["restaurant.table"].get(tableId);
            this.pos.data.write("restaurant.table", [tableId], {
                ...table.serialize({ orm: true }),
            });
        }
        this.state.selectedTableIds = [];
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
    async _createTableHelper(copyTable, duplicateFloor = false) {
        const existingTable = this.activeFloor.table_ids;
        let newTableData;
        if (copyTable) {
            newTableData = copyTable.serialize({ orm: true });
            if (!duplicateFloor) {
                newTableData.position_h += 10;
                newTableData.position_v += 10;
            }
            delete newTableData.id;
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

            newTableData = {
                active: true,
                position_v: posV,
                position_h: posH,
                width: widthTable,
                height: heightTable,
                shape: "square",
                seats: 2,
                color: "rgb(53, 211, 116)",
                floor_id: this.activeFloor.id,
            };
        }
        if (!duplicateFloor) {
            newTableData.table_number = this._getNewTableNumber();
        }
        const table = await this.createTableFromRaw(newTableData);
        return table;
    }
    async createTableFromRaw(newTableData) {
        newTableData.active = true;
        const table = await this.pos.data.create("restaurant.table", [newTableData]);
        return table[0];
    }
    _getNewTableNumber() {
        let firstNum = 1;
        const tablesNumber = this.activeTables
            .map((table) => table.table_number)
            .sort(function (a, b) {
                return a - b;
            });

        for (let i = 0; i < tablesNumber.length; i++) {
            if (tablesNumber[i] == firstNum) {
                firstNum += 1;
            } else {
                break;
            }
        }
        return firstNum;
    }
    get activeFloor() {
        return this.state.selectedFloorId
            ? this.pos.models["restaurant.floor"].get(this.state.selectedFloorId)
            : null;
    }
    get activeTables() {
        return this.activeFloor?.table_ids;
    }
    get selectedTables() {
        return this.state.selectedTableIds.map((id) => this.pos.models["restaurant.table"].get(id));
    }
    get nbrFloors() {
        return this.pos.models["restaurant.floor"].length;
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
        this.unselectTables();
    }
    async onClickTable(table, ev) {
        if (this.pos.isEditMode) {
            if (this.state.selectedTableIds.includes(table.id)) {
                this.state.selectedTableIds = this.state.selectedTableIds.filter(
                    (id) => id !== table.id
                );
                return;
            }
            if (!ev.ctrlKey && !ev.metaKey) {
                this.unselectTables();
            }
            this.state.selectedTableIds.push(table.id);
            return;
        }
        if (table.parent_id) {
            this.onClickTable(table.parent_id, ev);
            return;
        }
        if (!this.pos.isOrderTransferMode) {
            await this.pos.setTableFromUi(table);
        }
    }
    unselectTables() {
        if (this.selectedTables.length) {
            for (const table of this.selectedTables) {
                this.pos.data.write("restaurant.table", [table.id], table.serialize({ orm: true }));
            }
        }
        this.state.selectedTableIds = [];
    }
    closeEditMode() {
        this.pos.isEditMode = false;
        this.unselectTables();
    }
    async addFloor() {
        this.dialog.add(TextInputPopup, {
            title: _t("New Floor"),
            placeholder: _t("Floor name"),
            getPayload: async (newName) => {
                const floor = await this.pos.data.create(
                    "restaurant.floor",
                    [
                        {
                            name: newName,
                            background_color: "#FFFFFF",
                            pos_config_ids: [this.pos.config.id],
                        },
                    ],
                    false
                );

                this.selectFloor(floor[0]);
                this.pos.isEditMode = true;
            },
        });
    }

    async createTable() {
        const newTable = await this._createTableHelper();
        if (newTable) {
            this.state.selectedTableIds = [newTable.id];
        }
    }
    async duplicateFloor() {
        const floor = this.activeFloor;
        const tables = this.activeFloor.table_ids;
        const newFloorName = floor.name + " (copy)";
        const copyFloor = await this.pos.data.create("restaurant.floor", [
            {
                name: newFloorName,
                background_color: "#ACADAD",
                pos_config_ids: [this.pos.config.id],
            },
        ]);

        this.selectFloor(copyFloor[0]);
        this.pos.isEditMode = true;

        for (const table of tables) {
            const tableSerialized = table.serialize({ orm: true });
            tableSerialized.floor_id = copyFloor[0].id;
            await this.createTableFromRaw(tableSerialized);
        }
    }
    async duplicateTable() {
        const selectedTables = this.selectedTables;
        this.state.selectedTableIds = [];

        for (const table of selectedTables) {
            const newTable = await this._createTableHelper(table);
            if (newTable) {
                this.state.selectedTableIds.push(newTable.id);
            }
        }
    }
    async renameFloor() {
        this.dialog.add(TextInputPopup, {
            startingValue: this.activeFloor.name,
            title: _t("Floor Name ?"),
            getPayload: (newName) => {
                if (newName !== this.activeFloor.name) {
                    this.activeFloor.name = newName;
                    this.pos.data.write("restaurant.floor", [this.activeFloor.id], {
                        name: newName,
                    });
                }
            },
        });
    }
    async renameTable() {
        if (this.selectedTables.length > 1) {
            return;
        }
        if (this.selectedTables.length === 1) {
            this.dialog.add(NumberPopup, {
                startingValue: parseInt(this.selectedTables[0].table_number) || false,
                title: _t("Change table number?"),
                placeholder: _t("Enter a table number"),
                buttons: getButtons([{ ...DECIMAL, disabled: true }, ZERO, BACKSPACE]),
                isValid: (x) => x,
                getPayload: (newNumber) => {
                    if (parseInt(newNumber) !== this.selectedTables[0].table_number) {
                        this.pos.data.write("restaurant.table", [this.selectedTables[0].id], {
                            table_number: parseInt(newNumber),
                        });
                    }
                },
            });
        } else {
            this.dialog.add(TextInputPopup, {
                startingValue: this.activeFloor.name,
                title: _t("Floor Name ?"),
                getPayload: (newName) => {
                    if (newName !== this.activeFloor.name) {
                        this.activeFloor.name = newName;
                        this.pos.data.write("restaurant.floor", [this.activeFloor.id], {
                            name: newName,
                        });
                    }
                },
            });
        }
    }
    async changeSeatsNum() {
        const selectedTables = this.selectedTables;
        if (selectedTables.length == 0) {
            return;
        }
        this.dialog.add(NumberPopup, {
            title: _t("Number of Seats?"),
            getPayload: (num) => {
                const newSeatsNum = parseInt(num, 10);
                selectedTables.forEach((selectedTable) => {
                    if (newSeatsNum !== selectedTable.seats) {
                        this.pos.data.write("restaurant.table", [selectedTable.id], {
                            seats: newSeatsNum,
                        });
                    }
                });
            },
        });
    }
    changeShape(form) {
        for (const table of this.selectedTables) {
            this.pos.data.write("restaurant.table", [table.id], { shape: form });
        }
    }

    setFloorColor(color) {
        this.activeFloor.background_color = color;
        this.pos.data.write("restaurant.floor", [this.activeFloor.id], {
            background_color: color,
            floor_background_image: false,
        });
    }

    setTableColor(color) {
        if (this.selectedTables.length > 0) {
            for (const table of this.selectedTables) {
                this.pos.data.write("restaurant.table", [table.id], { color: color });
            }
        }
    }
    _getColors() {
        return {
            white: [255, 255, 255],
            red: [235, 109, 109],
            green: [53, 211, 116],
            blue: [108, 109, 236],
            orange: [235, 191, 109],
            yellow: [235, 236, 109],
            purple: [172, 109, 173],
            grey: [108, 109, 109],
            lightGrey: [172, 173, 173],
            turquoise: [78, 210, 190],
        };
    }
    formatColor(color) {
        return `rgb(${color})`;
    }
    getColors() {
        return Object.fromEntries(
            Object.entries(this._getColors()).map(([k, v]) => [k, this.formatColor(v)])
        );
    }
    getLighterShade(color) {
        return this.formatColor([...this._getColors()[color], 0.75]);
    }
    async deleteFloor() {
        const confirmed = await ask(this.dialog, {
            title: `Removing floor ${this.activeFloor.name}`,
            body: sprintf(
                _t("Removing a floor cannot be undone. Do you still want to remove %s?"),
                this.activeFloor.name
            ),
        });
        if (!confirmed) {
            return;
        }
        const activeFloor = this.activeFloor;
        try {
            await this.pos.data.call("restaurant.floor", "deactivate_floor", [
                activeFloor.id,
                this.pos.session.id,
            ]);
        } catch {
            this.dialog.add(AlertDialog, {
                title: _t("Delete Error"),
                body: _t("You cannot delete a floor with orders still in draft for this floor."),
            });
            return;
        }

        const orderList = [...this.pos.get_open_orders()];
        for (const order of orderList) {
            if (activeFloor.table_ids.includes(order.tableId)) {
                this.pos.removeOrder(order, false);
            }
        }

        for (const table_id of activeFloor.table_ids) {
            table_id.delete();
        }

        activeFloor.delete();

        if (this.pos.models["restaurant.floor"].length > 0) {
            this.selectFloor(this.pos.models["restaurant.floor"].getAll()[0]);
        } else {
            this.pos.isEditMode = false;
            this.pos.floorPlanStyle = "default";
        }
        return;
    }
    async deleteTable() {
        const confirmed = await ask(this.dialog, {
            title: _t("Are you sure?"),
            body: _t("Removing a table cannot be undone"),
        });
        if (!confirmed) {
            return;
        }
        const originalSelectedTableIds = [...this.state.selectedTableIds];

        try {
            const response = await this.pos.data.call(
                "restaurant.table",
                "are_orders_still_in_draft",
                [originalSelectedTableIds]
            );

            if (response) {
                for (const id of originalSelectedTableIds) {
                    //remove order not send to server
                    for (const order of this.pos.get_open_orders()) {
                        if (order.table_id == id) {
                            this.pos.removeOrder(order, false);
                        }
                    }
                    const records = this.pos.data.write("restaurant.table", [id], {
                        active: false,
                    });
                    records[0].delete();
                }
            }
        } catch {
            this.dialog.add(AlertDialog, {
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
    }
    getFloorChangeCount(floor) {
        let changeCount = 0;
        if (!floor) {
            return changeCount;
        }
        const table_ids = floor.table_ids;
        for (const table of table_ids) {
            changeCount += this.getChangeCount(table) || 0;
        }

        return changeCount;
    }
    getChildren(table) {
        return this.pos.models["restaurant.table"].filter((t) => t.parent_id?.id === table.id);
    }
    async uploadImage(event) {
        const file = event.target.files[0];
        if (!file) {
            // Don't proceed if there are no selected files.
            return;
        }
        if (!file.type.match(/image.*/)) {
            this.dialog.add(AlertDialog, {
                title: _t("Unsupported File Format"),
                body: _t("Only web-compatible Image formats such as .png or .jpeg are supported."),
            });
        } else {
            const imageUrl = await getDataURLFromFile(file);
            const loadedImage = await loadImage(imageUrl);
            if (loadedImage) {
                this.env.services.ui.block();
                await this.pos.data.ormWrite("restaurant.floor", [this.activeFloor.id], {
                    floor_background_image: imageUrl.split(",")[1],
                });
                // A read is added to be sure that we have the same image as the one in backend
                await this.pos.data.read("restaurant.floor", [this.activeFloor.id]);
                this.env.services.ui.unblock();
            } else {
                this.dialog.add(AlertDialog, {
                    title: _t("Loading Image Error"),
                    body: _t("Encountered error when loading image. Please try again."),
                });
            }
        }
    }
    getChangeCount(table) {
        // This information in uiState came by websocket
        // If the table is not synced, we need to count the unsynced orders
        let changeCount = 0;
        let skipCount = 0;
        const tableOrders = this.pos.models["pos.order"].filter(
            (o) => o.table_id?.id === table.id && !o.finalized
        );

        for (const order of tableOrders) {
            const changes = getOrderChanges(order, false, this.pos.orderPreparationCategories);
            changeCount += changes.nbrOfChanges;
            skipCount += changes.nbrOfSkipped;
        }

        return { changes: changeCount, skip: skipCount };
    }
    setColor(hasSelectedTable, color) {
        if (hasSelectedTable) {
            return this.setTableColor(color);
        } else {
            return this.setFloorColor(color);
        }
    }
    rename(hasSelectedTable) {
        if (hasSelectedTable) {
            return this.renameTable();
        } else {
            return this.renameFloor();
        }
    }
    duplicate(hasSelectedTable) {
        if (hasSelectedTable) {
            return this.duplicateTable();
        } else {
            return this.duplicateFloor();
        }
    }
    delete(hasSelectedTable) {
        if (hasSelectedTable) {
            return this.deleteTable();
        } else {
            return this.deleteFloor();
        }
    }
}

registry.category("pos_screens").add("FloorScreen", FloorScreen);

```

## File: static\src\app\floor_screen\floor_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.FloorScreen">
        <div class="floor-screen screen h-100 position-relative d-flex flex-column flex-nowrap m-0 bg-300 text-start overflow-hidden">
            <t t-set="editButtonClass" t-value="'btn btn-lg'" />
            <t t-set="hasSelectedTable" t-value="selectedTables.length > 0" />
            <t t-set="firstSelectedTable" t-value="selectedTables.length ? selectedTables[0] : null" />
            <div class="d-flex flex-row flex-wrap justify-content-between border-bottom bg-light">
                <!-- Left Side Div -->
                <div class="floor-selector d-flex gap-2 p-2 align-items-center overflow-auto">
                    <t t-foreach="pos.models['restaurant.floor'].getAll()" t-as="floor" t-key="floor.id">
                        <button class="button button-floor btn btn-outline-secondary btn-lg px-3 lh-lg text-nowrap" t-attf-class="{{ floor.id === state.selectedFloorId ? 'active' : '' }}" t-on-click="() => this.selectFloor(floor)">
                            <t t-esc="floor.name" />
                            <t t-set="changeCount" t-value="this.getFloorChangeCount(floor)"/>
                            <span t-if="changeCount > 0" class="badge rounded-pill text-bg-danger ms-2 py-1 smaller fw-bolder" t-esc="changeCount"/>
                        </button>
                    </t>
                    <button t-attf-class="{{editButtonClass}} btn-secondary lh-lg" t-if="pos.isEditMode or pos.config.floor_ids?.length === 0" t-on-click="addFloor" >
                        <i class="fa fa-plus fa-fw" role="img" aria-label="Add Floor" title="Add Floor" />
                    </button>
                </div>
                <!-- Right Side Div -->
                <div t-if="pos.isEditMode" class="edit-buttons d-flex gap-2 m-2 px-2 px-sm-0 overflow-x-auto" t-att-class="{'mx-auto': ui.isSmall}">
                    <div class="d-flex gap-1 p-1 rounded-3 bg-200">
                        <t t-if="hasSelectedTable">
                            <span class="mx-2 align-self-center text-uppercase smaller fw-bolder">Table <t t-esc="firstSelectedTable.table_number" /></span>
                            <button t-attf-class="{{editButtonClass}} btn-light" t-on-click.stop="changeSeatsNum" t-att-disabled="!hasSelectedTable">
                                <i class="fa fa-user fa-fw" role="img" aria-label="Seats" title="Seats" />
                            </button>
                            <t t-if="selectedTables.some((t) => t.shape === 'square')">
                                <button t-attf-class="{{editButtonClass}} btn-light" t-on-click.stop="() => this.changeShape('round')" t-att-disabled="!hasSelectedTable">
                                    <i class="fa fa-circle-o fa-fw" role="img" aria-label="Make Round" title="Round Shape" />
                                </button>
                            </t>
                            <t t-else="">
                                <button t-attf-class="{{editButtonClass}} btn-light" t-on-click.stop="() => this.changeShape('square')" t-att-disabled="!hasSelectedTable">
                                    <i class="fa fa-square-o fa-fw" role="img" aria-label="Make Square" title="Square Shape" />
                                </button>
                            </t>
                        </t>
                        <Dropdown menuClass="'pos-dropdown-menu px-1'">
                            <button t-attf-class="{{editButtonClass}} btn-light">
                                <i class="fa fa-paint-brush fa-fw" role="img" aria-label="Change Floor Background" title="Change Floor Background"/>
                            </button>
                            <t t-set-slot="content">
                                <div class="d-grid gap-1 py-2 px-1" style="grid-template-columns: repeat(3, 1fr);">
                                    <t t-foreach="Object.entries(getColors())" t-as="color" t-key="color[0]">
                                        <t t-set="adaptColor" t-value="!hasSelectedTable ? this.getLighterShade(color[0]) : color[0]" />
                                        <DropdownItem closingMode="'none'" onSelected="() => this.setColor(hasSelectedTable, adaptColor)">
                                            <button
                                                class="p-4 border-1 rounded"
                                                t-attf-style="background-color: {{adaptColor}}"
                                            />
                                        </DropdownItem>
                                    </t>
                                    <DropdownItem closingMode="'none'">
                                        <button class="floor-picture border-1 rounded position-relative text-center overflow-hidden d-flex flex-column align-items-center justify-content-center">
                                            <i class="fa fa-camera" role="img" aria-label="Picture" title="Picture"></i>
                                            File
                                            <input type="file" class="image-uploader" t-on-change="uploadImage" />
                                        </button>
                                    </DropdownItem>
                                </div>
                            </t>
                        </Dropdown>
                        <button t-attf-class="{{editButtonClass}} btn-light" t-on-click.stop="() => this.rename(hasSelectedTable)">
                            <i class="fa fa-pencil-square-o fa-fw" role="img" aria-label="Rename" title="Rename"/>
                        </button>
                        <button t-attf-class="{{editButtonClass}} btn-light" t-on-click.stop="() => this.duplicate(hasSelectedTable)">
                            <i class="fa fa-copy fa-fw" role="img" aria-label="Clone" title="Clone"/>
                        </button>
                        <button t-attf-class="{{editButtonClass}} btn-light" t-on-click.stop="() => this.delete(hasSelectedTable)">
                            <i class="fa fa-trash fa-fw" role="img" aria-label="Delete" title="Delete"/>
                        </button>
                    </div>
                    <button t-attf-class="btn btn-outline-secondary btn-lg d-flex align-items-center justify-content-center gap-2 lh-lg" t-on-click.stop="doCreateTable.call" t-att-disabled="doCreateTable.status === 'loading'">
                        <t t-if="doCreateTable.status === 'loading'">
                            <i class="fa fa-spinner fa-spin icon-button" role="img" aria-label="Loading" title="Loading"></i>
                        </t>
                        <t t-else="">
                            <i class="fa fa-plus-circle" role="img" aria-label="Add Table" title="Add Table"/>
                            <t t-if="!ui.isSmall">Table</t>
                        </t>
                    </button>
                    <button t-attf-class="btn btn-primary btn-lg" t-on-click.stop="closeEditMode">
                        <t t-if="!ui.isSmall">Save</t>
                        <i t-else="" class="fa fa-floppy-o" role="img" aria-label="Save" title="Save"/>
                    </button>
                </div>
            </div>
            <t t-set="isKanban" t-value="pos.floorPlanStyle == 'kanban'"/>
            <div t-ref="floor-map-scroll" class="overflow-auto flex-grow-1 flex-shrink-1 flex-basis-0 w-auto" t-attf-style="background: {{activeFloor?.background_color}}"> 
                <div t-on-click="onClickFloorMap" t-on-touchstart="_onPinchStart" t-on-touchmove="_onPinchMove" t-on-touchend="_onPinchEnd"
                    t-attf-class="floor-map position-relative w-100 h-100 {{ pos.isEditMode ? 'floor-grid' : ''}}"
                    t-ref="floor-map-ref"
                    t-attf-style="
                        -webkit-touch-callout: none;
                        height: {{state.floorHeight}} !important;
                        width: {{state.floorWidth}} !important;
                        {{ activeFloor?.floor_background_image and !isKanban ?
                            'background-image: url(' + floorBackround + '); background-size: auto; background-repeat: no-repeat; background-attachment: local;' :
                            ''
                        }}">
                    <t t-if="pos.config.floor_ids?.length > 0">
                        <div t-if="!activeTables?.length" class="empty-floor d-flex align-items-center justify-content-center h-100 fs-3 text-center text-muted" t-ref="map">
                            <span>Oops! No tables available.<br/>Add a new table to get started.</span>
                        </div>
                        <div t-else="" t-ref="map" t-att-class="{'floor-kanban d-grid gap-3 p-3': isKanban, 'h-100': !isKanban}">
                           <t t-foreach="activeTables.sort((a,b)=>a.id-b.id)" t-as="table" t-key="table.id" >
                                <t t-set="isOccupied" t-value="pos.tableHasOrders(table)"/>
                                <t t-set="isIntersecting" t-value="state.potentialLink?.child?.id === table.id"/>
                                <t t-set="isIntersected" t-value="state.potentialLink?.parent?.id === table.id"/>
                                <div
                                    t-on-click="(ev) => this.onClickTable(table, ev)"
                                    class="table o_draggable d-flex flex-column align-items-center justify-content-between cursor-pointer"
                                    t-att-class="{
                                        'position-relative m-0': isKanban,
                                        'position-absolute': pos.floorPlanStyle !== 'kanban',
                                        'selected': state.selectedTableIds.includes(table.id),
                                    }"
                                    t-attf-class="tableId-{{table.id}}"
                                    t-attf-style="
                                                border: 3px solid {{table.color}};
                                                border-radius: {{table.shape === 'round' ? 1000 : 6}}px;
                                                background: {{isOccupied ? table.color || 'rgb(53, 211, 116)' : '#00000020'}};
                                                color: {{!hasBg ? 'black' : 'white'}};
                                                opacity: {{state.potentialLink ? (isIntersecting or isIntersected ? 1 : 0.25) : 1}};
                                                {{isKanban ?
                                                    `
                                                        width: 100%;
                                                        min-height: 120px;
                                                    ` :
                                                    `
                                                        width: ${table.width}px;
                                                        height: ${table.height}px;
                                                        top: ${table.getY()}px;
                                                        left: ${table.getX()}px;
                                                    `
                                                }}
                                            "
                                    >
                                    <t t-set="offset" t-value="getTableHandleOffset(table)"/>
                                    <div
                                        class="info position-relative w-100 h-100 overflow-hidden"
                                        t-att-class="{'opacity-25': table.parent_id}"
                                        t-attf-style="border-radius: {{table.shape === 'round' ? 1000 : 3}}px;"
                                    >
                                        <div t-esc="table.table_number" class="label fw-bolder fs-4 position-absolute top-50 start-50 translate-middle" />
                                    </div>
                                    <t t-set="data" t-value="getChangeCount(table)"/>
                                    <div
                                        t-if="data.changes > 0 || data.skip > 0"
                                        t-esc="this.env.utils.formatProductQty(data.changes > 0 ? data.changes : data.skip, false)"
                                        t-att-class="{
                                            'text-bg-danger': data.changes,
                                            'text-bg-info'  : !data.changes and data.skip,
                                        }"
                                        class="order-count d-flex align-items-center justify-content-center position-absolute top-0 start-100 translate-middle rounded-circle smaller fw-bolder z-2"
                                        t-attf-style="width: 2rem; height: 2rem;"
                                    />
                                    <t t-if="state.selectedTableIds.includes(table.id)">
                                        <span t-attf-class="tableId-{{table.id}}" class="table-handle position-absolute top left" t-attf-style="top: {{offset}}px; left: {{offset}}px"/>
                                        <span t-attf-class="tableId-{{table.id}}" class="table-handle position-absolute top right" t-attf-style="top: {{offset}}px; right: {{offset}}px"/>
                                        <span t-attf-class="tableId-{{table.id}}" class="table-handle position-absolute bottom right" t-attf-style="bottom: {{offset}}px; right: {{offset}}px"/>
                                        <span t-attf-class="tableId-{{table.id}}" class="table-handle position-absolute bottom left" t-attf-style="bottom: {{offset}}px; left: {{offset}}px"/>
                                    </t>
                                </div>
                            </t>
                        </div>
                    </t>
                    <div t-else="" class="empty-floor d-flex align-items-center justify-content-center h-100 fs-3 text-center text-muted" t-ref="map">
                        <span>Oops! No floors available.<br/>Add a new floor to get started.</span>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\app\models\restaurant_table.js

```javascript
import { registry } from "@web/core/registry";
import { Base } from "@point_of_sale/app/models/related_models";

export class RestaurantTable extends Base {
    static pythonModel = "restaurant.table";

    setup(vals) {
        super.setup(vals);

        this.table_number = vals.table_number || 0;
        this.uiState = {
            initialPosition: {},
            orderCount: 0,
            changeCount: 0,
            skipCount: 0,
        };
    }
    isParent(t) {
        return t.parent_id && (t.parent_id.id === this.id || this.isParent(t.parent_id));
    }
    getParent() {
        return this.parent_id?.getParent() || this;
    }
    getParentSide() {
        if (!this.parent_id) {
            return;
        }
        const dx = this.position_h - this.parent_id.getX();
        const dy = this.position_v - this.parent_id.getY();
        if (Math.abs(dx) > Math.abs(dy)) {
            return dx < 0 ? "right" : "left";
        }
        return dy > 0 ? "bottom" : "top";
    }
    getX() {
        if (!this.parent_id) {
            return this.position_h;
        }
        const parent_side = this.parent_side || this.getParentSide();
        if (["top", "bottom"].includes(parent_side)) {
            return this.parent_id.getX();
        }
        if (parent_side === "left") {
            return this.parent_id.getX() + this.parent_id.width;
        }
        return this.parent_id.getX() - this.width;
    }
    getY() {
        if (!this.parent_id) {
            return this.position_v;
        }
        const parent_side = this.parent_side || this.getParentSide();
        this.parent_side = parent_side;
        if (["left", "right"].includes(parent_side)) {
            return this.parent_id.getY();
        }
        if (parent_side === "bottom") {
            return this.parent_id.getY() + this.parent_id.height;
        }
        return this.parent_id.getY() - this.height;
    }
    getCenter() {
        return {
            x: this.getX() + this.width / 2,
            y: this.getY() + this.height / 2,
        };
    }
    get orders() {
        return this.models["pos.order"].filter(
            (o) =>
                o.table_id?.id === this.id &&
                // Include the orders that are in tipping state.
                (!o.finalized || o.uiState.screen_data?.value?.name === "TipScreen")
        );
    }
    getOrder() {
        return (
            this.parent_id?.getOrder?.() || this["<-pos.order.table_id"].find((o) => !o.finalized)
        );
    }
    setPositionAsIfLinked(parent, side) {
        this.parent_id = parent;
        this.parent_side = side;
        this.position_h = this.getX();
        this.position_v = this.getY();
        this.parent_id = null;
    }
    getName() {
        return this.table_number.toString();
    }
}
registry.category("pos_available_models").add(RestaurantTable.pythonModel, RestaurantTable);

```

## File: static\src\app\product_screen\order_summary\order_summary.js

```javascript
import { patch } from "@web/core/utils/patch";
import { OrderSummary } from "@point_of_sale/app/screens/product_screen/order_summary/order_summary";

patch(OrderSummary.prototype, {
    bookTable() {
        this.pos.get_order().setBooked(true);
        this.pos.showScreen("FloorScreen");
    },
    showBookButton() {
        if (!this.pos.selectedTable) {
            return false;
        }
        return (
            this.pos.config.module_pos_restaurant &&
            !this.pos.models["pos.order"].some(
                (o) =>
                    o.table_id?.id === this.pos.selectedTable.id &&
                    o.finalized === false &&
                    o.isBooked
            )
        );
    },
    async unbookTable() {
        const order = this.pos.get_order();
        await this.pos._onBeforeDeleteOrder(order);
        order.state = "cancel";
        this.pos.showScreen("FloorScreen");
    },
    showUnbookButton() {
        if (this.pos.selectedTable) {
            return (
                this.pos.config.module_pos_restaurant &&
                !this.pos.models["pos.order"].some(
                    (o) =>
                        o.table_id?.id === this.pos.selectedTable.id &&
                        o.finalized === false &&
                        !o.isBooked
                ) &&
                this.pos.get_order().lines.length === 0
            );
        }
        const currentOrder = this.pos.get_order();
        return (
            this.pos.config.module_pos_restaurant &&
            !currentOrder.finalized &&
            currentOrder.isBooked &&
            currentOrder.lines.length === 0
        );
    },
});

```

## File: static\src\app\product_screen\order_summary\order_summary.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.OrderSummary" t-inherit="point_of_sale.OrderSummary" t-inherit-mode="extension">
        <xpath expr="//OrderWidget/t[@t-set-slot='details']" position="inside">
            <div class="d-flex flex-column align-items-center gap-2">
                <button t-if="showBookButton()" class="btn btn-primary btn-lg py-2 book-table" style="border:none; font-size: 20px;" t-on-click="bookTable">Book table</button>
                <button t-if="showUnbookButton()" class="btn btn-primary btn-lg py-2 unbook-table" style="border:none; font-size: 20px;" t-on-click="unbookTable">
                    <t t-if="pos.selectedTable">Release table</t>
                    <t t-else="">Release Order</t>
                </button>    
            </div>
        </xpath>
        <xpath expr="//OrderWidget" position="attributes">
            <attribute name="isConfigRestaurant">pos.config.module_pos_restaurant</attribute>
            <attribute name="isOrderBooked">!!currentOrder.isBooked</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\app\split_bill_screen\split_bill_screen.js

```javascript
import { registry } from "@web/core/registry";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { Component, useState } from "@odoo/owl";
import { Orderline } from "@point_of_sale/app/generic_components/orderline/orderline";
import { OrderWidget } from "@point_of_sale/app/generic_components/order_widget/order_widget";

export class SplitBillScreen extends Component {
    static template = "pos_restaurant.SplitBillScreen";
    static components = { Orderline, OrderWidget };
    static props = {};

    setup() {
        this.pos = usePos();
        this.qtyTracker = useState({});
        this.priceTracker = useState({});
    }

    get currentOrder() {
        return this.pos.get_order();
    }

    get orderlines() {
        return this.currentOrder.get_orderlines();
    }

    get newOrderPrice() {
        return Object.values(this.priceTracker).reduce((a, b) => a + b, 0);
    }

    onClickLine(line) {
        const lines = line.getAllLinesInCombo();

        for (const line of lines) {
            if (!line.is_pos_groupable()) {
                if (this.qtyTracker[line.uuid] === line.get_quantity()) {
                    this.qtyTracker[line.uuid] = 0;
                } else {
                    this.qtyTracker[line.uuid] = line.get_quantity();
                }
            } else if (!this.qtyTracker[line.uuid]) {
                this.qtyTracker[line.uuid] = 1;
            } else if (this.qtyTracker[line.uuid] === line.get_quantity()) {
                this.qtyTracker[line.uuid] = 0;
            } else {
                this.qtyTracker[line.uuid] += 1;
            }
            // We need this split for decimal quantities (e.g. 0.5 kg)
            if (this.qtyTracker[line.uuid] > line.get_quantity()) {
                this.qtyTracker[line.uuid] = line.get_quantity();
            }
            this.priceTracker[line.uuid] =
                (line.get_price_with_tax() / line.qty) * this.qtyTracker[line.uuid];
        }
    }

    _getOrderName(order) {
        return order.table_id?.table_number.toString() || order.getFloatingOrderName() || "";
    }

    _getLatestOrderNameStartingWith(name) {
        return (
            this.pos
                .get_open_orders()
                .map((order) => this._getOrderName(order))
                .filter((orderName) => orderName.slice(0, -1) === name)
                .sort((a, b) => a.slice(-1).localeCompare(b.slice(-1)))
                .at(-1) || name
        );
    }

    _getSplitOrderName(originalOrderName) {
        const latestOrderName = this._getLatestOrderNameStartingWith(originalOrderName);
        if (latestOrderName === originalOrderName) {
            return `${originalOrderName}B`;
        }
        const lastChar = latestOrderName[latestOrderName.length - 1];
        if (lastChar === "Z") {
            throw new Error("You cannot split the order into more than 26 parts!");
        }
        const nextChar = String.fromCharCode(lastChar.charCodeAt(0) + 1);
        return `${latestOrderName.slice(0, -1)}${nextChar}`;
    }

    // Meant to be overridden
    async preSplitOrder(originalOrder, newOrder) {}
    async postSplitOrder(originalOrder, newOrder) {}

    // Calculates the sent quantities for both orders and adjusts for last_order_preparation_change.
    _getSentQty(ogLine, newLine, orderedQty) {
        const unorderedQty = ogLine.qty - orderedQty;

        const delta = newLine.qty - unorderedQty;
        const newQty = delta > 0 ? delta : 0;

        return {
            [ogLine.preparationKey]: orderedQty - newQty,
            [newLine.preparationKey]: newQty,
        };
    }

    async createSplittedOrder() {
        const curOrderUuid = this.currentOrder.uuid;
        const originalOrder = this.pos.models["pos.order"].find((o) => o.uuid === curOrderUuid);
        this.pos.selectedTable = null;
        const originalOrderName = this._getOrderName(originalOrder);
        const newOrderName = this._getSplitOrderName(originalOrderName);

        const newOrder = this.pos.createNewOrder();
        newOrder.floating_order_name = newOrderName;
        newOrder.uiState.splittedOrderUuid = curOrderUuid;
        await this.preSplitOrder(originalOrder, newOrder);

        // Create lines for the new order
        const lineToDel = [];
        for (const line of originalOrder.lines) {
            if (this.qtyTracker[line.uuid]) {
                const data = line.serialize();
                delete data.uuid;
                const newLine = this.pos.models["pos.order.line"].create(
                    {
                        ...data,
                        qty: this.qtyTracker[line.uuid],
                        order_id: newOrder.id,
                    },
                    false,
                    true
                );

                const ordered =
                    originalOrder.last_order_preparation_change.lines[line.preparationKey];
                if (line.get_quantity() === this.qtyTracker[line.uuid]) {
                    delete originalOrder.last_order_preparation_change.lines[line.preparationKey];
                    lineToDel.push(line);

                    if (ordered) {
                        const newOrdered = { ...ordered };
                        newOrdered.uuid = newLine.uuid;
                        newOrder.last_order_preparation_change.lines[newLine.preparationKey] =
                            newOrdered;
                    }
                } else {
                    const newQty = line.get_quantity() - this.qtyTracker[line.uuid];
                    line.update({ qty: newQty });

                    if (ordered) {
                        const orderedQty = ordered["quantity"];
                        const newOrderedQty = orderedQty > newQty ? newQty : orderedQty;
                        ordered["quantity"] = newOrderedQty;

                        if (orderedQty > newQty) {
                            const newOrdered = { ...ordered };

                            newOrdered.uuid = newLine.uuid;
                            newOrdered.quantity = orderedQty - newQty;
                            newOrder.last_order_preparation_change.lines[newLine.preparationKey] =
                                newOrdered;
                        }
                    }
                }
            }
        }

        for (const line of lineToDel) {
            line.delete();
        }

        await this.pos.syncAllOrders({ orders: [originalOrder, newOrder] });
        originalOrder.customer_count -= 1;
        await this.postSplitOrder(originalOrder, newOrder);
        originalOrder.set_screen_data({ name: "ProductScreen" });
        this.pos.selectedOrderUuid = null;
        this.pos.set_order(newOrder);
        this.back();
    }

    getLineData(line) {
        const splitQty = this.qtyTracker[line.uuid];

        if (!splitQty) {
            return line.getDisplayData();
        }

        return { ...line.getDisplayData(), qty: `${splitQty} / ${line.get_quantity_str()}` };
    }

    back() {
        this.pos.showScreen("ProductScreen");
    }
}

registry.category("pos_screens").add("SplitBillScreen", SplitBillScreen);

```

## File: static\src\app\split_bill_screen\split_bill_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.SplitBillScreen">
        <div class="splitbill-screen screen h-100 bg-200">
            <div class="contents d-flex flex-column flex-nowrap h-100 my-0 mx-auto">
                <div class="top-content d-flex align-items-center p-2 border-bottom text-center bg-view">
                    <button class="button back back-button btn btn-secondary lh-lg" t-on-click="back">
                        <i class="oi oi-chevron-left fa-fw"/>
                    </button>
                    <div class="top-content-center flex-grow-1 pe-5">
                        <h4 class="mb-0">Bill Splitting</h4>
                    </div>
                </div>

                <div class="main d-flex flex-column flex-lg-row flex-grow-1 gap-2 p-2 overflow-hidden">
                    <div class="flex-grow-1 w-100 w-lg-50 me-0 me-lg-2 rounded-3 bg-view overflow-auto">
                        <OrderWidget lines="orderlines" t-slot-scope="scope" generalNote="currentOrder.general_note or ''">
                            <t t-set="line" t-value="scope.line" />
                            <Orderline line="getLineData(line)"
                                t-on-click="() => this.onClickLine(line)"
                                class="{'selected active': qtyTracker[line.uuid] and qtyTracker[line.uuid] !== 0}" />
                        </OrderWidget>
                    </div>
                    <div class="controls flex-column flex-nowrap w-100 w-lg-50">
                        <div class="order-info mb-2 mt-2 mt-lg-0 py-3 py-lg-4 rounded-3 text-center bg-view">
                            <span class="subtotal text-success">
                                <t t-esc="env.utils.formatCurrency(newOrderPrice)" />
                            </span>
                        </div>
                        <div class="pay-button">
                            <div class="button btn btn-lg btn-primary py-3 py-lg-5 w-100" t-on-click="createSplittedOrder">
                                <span>Split Order</span>
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
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { useService } from "@web/core/utils/hooks";
import { Component, useRef, onMounted } from "@odoo/owl";
import { TipReceipt } from "@pos_restaurant/app/tip_receipt/tip_receipt";
import { ask } from "@point_of_sale/app/store/make_awaitable_dialog";

export class TipScreen extends Component {
    static template = "pos_restaurant.TipScreen";
    static props = {};
    setup() {
        this.pos = usePos();
        this.posReceiptContainer = useRef("pos-receipt-container");
        this.dialog = useService("dialog");
        this.printer = useService("printer");
        this.state = this.currentOrder.uiState.TipScreen;
        this._totalAmount = this.currentOrder.get_total_with_tax();

        onMounted(async () => {
            await this.printTipReceipt();
        });
    }
    get overallAmountStr() {
        const tipAmount = this.env.utils.parseValidFloat(this.state.inputTipAmount);
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
        const amount = this.env.utils.parseValidFloat(this.state.inputTipAmount);
        const order = this.pos.get_order();
        const serverId = typeof order.id === "number" && order.id;

        if (!serverId) {
            this.dialog.add(AlertDialog, {
                title: _t("Unsynced order"),
                body: _t(
                    "This order is not yet synced to server. Make sure it is synced then try again."
                ),
            });
            return;
        }

        if (!amount) {
            await this.pos.data.write("pos.order", [serverId], { is_tipped: true, tip_amount: 0 });
            this.goNextScreen();
            return;
        }

        if (amount > 0.25 * this.totalAmount) {
            const confirmed = await ask(this.dialog, {
                title: "Are you sure?",
                body: `${this.env.utils.formatCurrency(
                    amount
                )} is more than 25% of the order's total amount. Are you sure of this tip amount?`,
            });
            if (!confirmed) {
                return;
            }
        }

        order.state = "draft";
        await this.pos.set_tip(amount);
        order.state = "paid";

        const paymentline = this.pos.get_order().payment_ids[0];
        if (paymentline.payment_method_id.payment_terminal) {
            paymentline.amount += amount;
            await paymentline.payment_method_id.payment_terminal.send_payment_adjust(
                paymentline.uuid
            );
        }

        const serializedTipLine = order.get_selected_orderline().serialize({ orm: true });
        order.get_selected_orderline().delete();
        const serverTipLine = await this.pos.data.create("pos.order.line", [serializedTipLine]);
        await this.pos.data.write("pos.order", [serverId], {
            is_tipped: true,
            tip_amount: serverTipLine[0].price_subtotal_incl,
        });
        this.goNextScreen();
    }
    goNextScreen() {
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
            order.get_selected_paymentline().ticket,
            order.get_selected_paymentline().cashier_receipt,
        ];
        for (let i = 0; i < receipts.length; i++) {
            await this.printer.print(
                TipReceipt,
                {
                    headerData: this.pos.getReceiptHeaderData(order),
                    data: receipts[i] || {},
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
import { Navbar } from "@point_of_sale/app/navbar/navbar";
import { patch } from "@web/core/utils/patch";
import { _t } from "@web/core/l10n/translation";
import {
    getButtons,
    EMPTY,
    ZERO,
    BACKSPACE,
} from "@point_of_sale/app/generic_components/numpad/numpad";
import { TableSelector } from "./table_selector/table_selector";

patch(Navbar.prototype, {
    /**
     * If no table is set to pos, which means the current main screen
     * is floor screen, then the order count should be based on all the orders.
     */
    get orderCount() {
        if (this.pos.config.module_pos_restaurant && this.pos.selectedTable) {
            return this.pos.getTableOrders(this.pos.selectedTable.id).length;
        }
        return super.orderCount;
    },
    onSwitchButtonClick() {
        const mode = this.pos.floorPlanStyle === "kanban" ? "default" : "kanban";
        localStorage.setItem("floorPlanStyle", mode);
        this.pos.floorPlanStyle = mode;
    },
    get showEditPlanButton() {
        return true;
    },
    setFloatingOrder(floatingOrder) {
        this.pos.selectedTable = null;
        this.pos.set_order(floatingOrder);
        this.pos.showScreen("ProductScreen");
    },
    async onClickTableTab() {
        await this.pos.syncAllOrders();
        this.dialog.add(TableSelector, {
            title: _t("Table Selector"),
            placeholder: _t("Enter a table number"),
            buttons: getButtons([
                EMPTY,
                ZERO,
                { ...BACKSPACE, class: "o_colorlist_item_color_transparent_1" },
            ]),
            confirmButtonLabel: _t("Jump"),
            getPayload: async (table_number) => {
                const find_table = (t) => t.table_number === parseInt(table_number);
                const table =
                    this.pos.currentFloor?.table_ids.find(find_table) ||
                    this.pos.models["restaurant.table"].find(find_table);
                if (table) {
                    return this.pos.setTableFromUi(table);
                }
                const floating_order = this.pos
                    .get_open_orders()
                    .find((o) => o.getFloatingOrderName() === table_number);
                if (floating_order) {
                    return this.setFloatingOrder(floating_order);
                }
                if (!table && !floating_order) {
                    this.pos.selectedTable = null;
                    const newOrder = this.pos.add_new_order();
                    newOrder.floating_order_name = table_number;
                    newOrder.setBooked(true);
                    return this.setFloatingOrder(newOrder);
                }
            },
        });
    },
    onClickPlanButton() {
        this.pos.showScreen("FloorScreen", { floor: this.floor });
    },
});

```

## File: static\src\overrides\components\navbar\navbar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.Navbar" t-inherit="point_of_sale.Navbar" t-inherit-mode="extension">
        <xpath expr="//DropdownItem[contains(text(), 'Backend')]" position="before">
            <t t-if="pos.mainScreen.component.name == 'FloorScreen'">
                <DropdownItem t-if="showEditPlanButton and this.pos.config.floor_ids.length" onSelected="() => this.pos.toggleEditMode()">
                    Edit Plan
                </DropdownItem>
            </t>
            <DropdownItem t-if="pos.mainScreen.component.name == 'FloorScreen'" onSelected="() => this.onSwitchButtonClick()">
                Switch Floor View
            </DropdownItem>
        </xpath>
        <xpath expr="//div[hasclass('pos-leftheader')]/OrderTabs" position="attributes">
            <attribute name="t-if">!pos.config.module_pos_restaurant or !ui.isSmall</attribute>
        </xpath>
        <xpath expr="//div[hasclass('pos-leftheader')]/OrderTabs" position="before">
            <div t-if="pos.config.module_pos_restaurant" class="d-flex flex-shrink-0 gap-1 position-relative">
               <div class="navbar-menu d-flex d-lg-grid gap-1">
                    <t t-set="screen" t-value="pos.mainScreen.component.name" />
                    <button class="back-button btn btn-lg lh-lg" t-att-class="{'btn-primary': screen === 'FloorScreen'}" t-on-click="() => this.onClickPlanButton()">
                        <span t-if="!ui.isSmall">Plan</span>
                        <img t-else="" src="/pos_restaurant/static/img/plan.svg" class="navbar-icon" alt="Floor Plan"/>
                    </button>
                    <button class="table-free-order-label btn btn-lg lh-lg" t-att-class="{'btn-primary': !['ActionScreen', 'FloorScreen'].includes(screen)}" t-on-click="() => this.onClickTableTab()">
                        <span t-if="pos.get_order()" t-esc="pos.get_order().getName().slice(0, 7)"/>
                        <span t-elif="!ui.isSmall">Table</span>
                        <img t-else="" src="/pos_restaurant/static/img/table.svg" class="navbar-icon" alt="Table Selector"/>
                    </button>
                </div>
                <div t-if="!ui.isSmall" class="ms-1 me-2 my-2 border-start border"/>
                <div class="d-flex align-items-center" t-if="pos.isOrderTransferMode">
                    <strong class="mx-2 text-warning">
                        Select table to transfer order
                    </strong>
                </div>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\navbar\table_selector\table_selector.js

```javascript
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { OrderTabs } from "@point_of_sale/app/components/order_tabs/order_tabs";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";

export class TableSelector extends NumberPopup {
    static template = "pos_restaurant.TableSelector";
    static components = { ...super.components, OrderTabs };
    setup() {
        this.pos = usePos();
        super.setup();
    }
}

```

## File: static\src\overrides\components\navbar\table_selector\table_selector.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.TableSelector" t-inherit="point_of_sale.NumberPopup" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('input-symbol')]" position="before">
            <OrderTabs orders="pos.get_open_orders().filter((o) => !o.table_id)" class="'mb-3'"/>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\order_tabs\order_tabs.js

```javascript
import { OrderTabs } from "@point_of_sale/app/components/order_tabs/order_tabs";
import { patch } from "@web/core/utils/patch";

patch(OrderTabs.prototype, {
    newFloatingOrder() {
        const order = super.newFloatingOrder(...arguments);

        if (this.pos.config.module_pos_restaurant) {
            order.setBooked(true);
        }
    },
});

```

## File: static\src\overrides\components\order_widget\order_widget.js

```javascript
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
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";

patch(PaymentScreen.prototype, {
    get nextScreen() {
        const order = this.currentOrder;
        if (!this.pos.config.set_tip_after_payment || order.is_tipped) {
            return super.nextScreen;
        }
        // Take the first payment method as the main payment.
        const mainPayment = order.payment_ids[0];
        if (mainPayment && mainPayment.canBeAdjusted()) {
            return "TipScreen";
        }
        return super.nextScreen;
    },
    async afterOrderValidation(suggestToSync = true) {
        // After the order has been validated the tables have no reason to be merged anymore.
        const changedTables = this.pos.models["restaurant.table"]?.filter(
            (t) => t.parent_id && t.parent_id.id === this.currentOrder.table_id?.id
        );
        if (changedTables?.length) {
            for (const table of changedTables) {
                this.pos.data.write("restaurant.table", [table.id], { parent_id: null });
            }
        }
        return await super.afterOrderValidation(...arguments);
    },
});

```

## File: static\src\overrides\components\payment_screen\payment_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_restaurant.PaymentScreenValidate" t-inherit="point_of_sale.PaymentScreenValidate" t-inherit-mode="extension">
        <xpath expr="//button[hasclass('button') and hasclass('next')]" position="attributes">
            <attribute name="t-att-hidden">pos.config.set_tip_after_payment and !currentOrder.is_paid()</attribute>
        </xpath>

        <xpath expr="//button[hasclass('button') and hasclass('next')]/span[hasclass('next_text')]" position="replace">
            <t t-if="pos.config.set_tip_after_payment and currentOrder.is_paid()">
                <span class="back_text">Close Tab</span>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
    <t t-name="pos_restaurant.PaymentScreenBack" t-inherit="point_of_sale.PaymentScreenBack" t-inherit-mode="extension">
        <xpath expr="//button[hasclass('button') and hasclass('back')]/span[hasclass('back_text')]" position="replace">
            <t t-if="pos.config.set_tip_after_payment and currentOrder.is_paid()">
                <span class="back_text">Keep Open</span>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
    <t t-name="pos_restaurant.PaymentScreenDue" t-inherit="point_of_sale.PaymentScreenDue" t-inherit-mode="extension">
        <xpath expr="//section[hasclass('paymentlines-container')]" position="inside">
            <div t-if="currentOrder.getCustomerCount() > 1" class="message text-center fs-4">
                <t t-esc="this.env.utils.formatCurrency(currentOrder.amountPerGuest())" /> / Guest
            </div>
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
            <t t-if="line.canBeAdjusted() &amp;&amp; line.pos_order_id.get_total_paid() &lt; line.pos_order_id.get_total_with_tax()">
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
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { patch } from "@web/core/utils/patch";
import { useState } from "@odoo/owl";

patch(ProductScreen.prototype, {
    /**
     * @override
     */
    setup() {
        super.setup(...arguments);
        this.uiState = useState({
            clicked: false,
        });
    },
    get selectedOrderlineQuantity() {
        const order = this.pos.get_order();
        const orderline = order.get_selected_orderline();
        const isForPreparation = orderline.product_id.pos_categ_ids
            .map((categ) => categ.id)
            .some((id) => this.pos.orderPreparationCategories.has(id));
        if (
            this.pos.config.module_pos_restaurant &&
            this.pos.orderPreparationCategories.size &&
            isForPreparation
        ) {
            const changes = Object.values(this.pos.getOrderChanges().orderlines).find(
                (change) => change.name == orderline.get_full_product_name()
            );
            return changes ? changes.quantity : false;
        }
        return super.selectedOrderlineQuantity;
    },
    get nbrOfChanges() {
        return this.pos.getOrderChanges().nbrOfChanges;
    },
    get swapButton() {
        return this.pos.config.module_pos_restaurant && this.pos.orderPreparationCategories.size;
    },
    get displayCategoryCount() {
        return this.pos.categoryCount.slice(0, 3);
    },
    async submitOrder() {
        if (!this.uiState.clicked) {
            this.uiState.clicked = true;
            try {
                await this.pos.sendOrderInPreparationUpdateLastChange(this.currentOrder);
                this.pos.addPendingOrder([this.currentOrder.id]);
            } finally {
                this.uiState.clicked = false;
            }
        }
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
            this.pos.getOrderChanges().nbrOfChanges !== 0 && this.pos.config.module_pos_restaurant
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
            <t t-if="!pos.scanning">
                <button
                    t-if="this.swapButton"
                    class="btn-switchpane pay-button btn btn-lg w-50 position-relative lh-sm overflow-hidden"
                    t-on-click="submitOrder"
                    t-attf-class="{{ primaryOrderButton ? 'btn-primary' : 'btn-light' }}">
                    <!-- Replace the payment button by the order button -->
                    <span class="d-block">Order</span>
                    <small><t t-esc="nbrOfChanges"/> changes</small>
                </button>
                <t t-else="">
                    <button
                        class="btn-switchpane pay-button btn btn-lg w-50 lh-sm"
                        t-attf-class="{{ currentOrder.is_empty() ? 'btn-light disabled' : 'btn-primary' }}"
                        t-on-click="() => this.pos.pay()">
                        <span class="d-block">Pay</span>
                        <small><t t-esc="total" /></small>
                    </button>
                </t>
            </t>
        </xpath>
        <xpath expr="//button[hasclass('review-button')]" position="replace">
            <button class="btn-switchpane btn btn-lg w-50 review-button lh-sm" t-attf-class="{{ primaryReviewButton ? 'btn-primary' : 'btn-secondary' }}" t-on-click="switchPane">
                <span class="d-block">Cart</span>
                <small t-if="this.swapButton"><t t-esc="total" /></small>
                <small t-else=""><t t-esc="items"/> items</small>
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\product_screen\actionpad_widget\actionpad_widget.js

```javascript
import { patch } from "@web/core/utils/patch";
import { ActionpadWidget } from "@point_of_sale/app/screens/product_screen/action_pad/action_pad";
import { useState } from "@odoo/owl";
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
/**
 * @props partner
 */

patch(ActionpadWidget.prototype, {
    setup() {
        super.setup();
        this.uiState = useState({
            clicked: false,
        });
    },
    get swapButton() {
        return (
            this.pos.config.module_pos_restaurant && this.pos.mainScreen.component !== TicketScreen
        );
    },
    get currentOrder() {
        return this.pos.get_order();
    },
    get hasChangesToPrint() {
        let hasChange = this.pos.getOrderChanges();
        hasChange =
            hasChange.generalNote == ""
                ? true // for the case when removed all general note
                : hasChange.count || hasChange.generalNote || hasChange.modeUpdate;
        return hasChange;
    },
    get swapButtonClasses() {
        return {
            "highlight btn-primary justify-content-between": this.displayCategoryCount.length,
            "btn-light pe-none disabled justify-content-center": !this.displayCategoryCount.length,
            altlight: !this.hasChangesToPrint && this.currentOrder?.hasSkippedChanges(),
        };
    },
    async submitOrder() {
        if (!this.uiState.clicked) {
            this.uiState.clicked = true;
            try {
                await this.pos.sendOrderInPreparationUpdateLastChange(this.currentOrder);
            } finally {
                this.uiState.clicked = false;
            }
        }
    },
    hasQuantity(order) {
        if (!order) {
            return false;
        } else {
            return order.lines.reduce((totalQty, line) => totalQty + line.get_quantity(), 0) > 0;
        }
    },
    get highlightPay() {
        return (
            this.currentOrder?.lines?.length &&
            !this.hasChangesToPrint &&
            this.hasQuantity(this.currentOrder)
        );
    },
    get displayCategoryCount() {
        return this.pos.categoryCount.slice(0, 4);
    },
    get isCategoryCountOverflow() {
        if (this.pos.categoryCount.length > 4) {
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
        <xpath expr="//BackButton" position="attributes">
            <attribute name="t-if">!this.swapButton and !props.showActionButton and pos.showBackButton()</attribute>
        </xpath>
        <xpath expr="//div[hasclass('validation')]" position="attributes">
            <attribute name="t-if">this.swapButton or props.showActionButton</attribute>
        </xpath>
        <xpath expr="//div[hasclass('validation')]//button[hasclass('pay-order-button')]" position="attributes">
            <attribute name="t-if">!this.swapButton</attribute>
        </xpath>
        <!-- Replace the payment button by the order button -->
        <xpath expr="//div[hasclass('validation')]" position="inside">
            <div t-if="this.swapButton" class="d-flex gap-2 flex-fill">
                <button
                    class="submit-order h-100 button btn btn-lg d-flex align-items-center w-50 flex-fill position-relative px-3"
                    t-att-class="swapButtonClasses"
                    t-on-click="submitOrder"
                >
                    <t t-if="!(ui.isSmall or displayCategoryCount.length > 2) or (!displayCategoryCount.length and ui.isSmall)">Order</t>
                    <div t-attf-class="{{ !(displayCategoryCount.length > 2) ? 'd-flex flex-column align-items-end gap-1' : 'row row-cols-2 g-1 gx-2' }} {{ isCategoryCountOverflow ? 'mt-n3' : ''}}">
                        <t t-if="displayCategoryCount.length">
                            <t t-foreach="displayCategoryCount" t-as="categoryCountLine"  t-key="categoryCountLine_index">
                                <div class="d-flex align-items-center justify-content-between small" t-att-class="{ 'gap-2' : !(displayCategoryCount.length > 2) }">
                                    <label class="text-truncate"><t t-esc="categoryCountLine.name"/></label>
                                    <label class="rounded px-2 py-0" style="background-color:rgba(0, 0, 0, 0.3);"><t t-esc="this.env.utils.formatProductQty(categoryCountLine.count, false)"/></label>
                                </div>
                            </t>
                        </t>
                        <t t-if="isCategoryCountOverflow">
                            <div class="position-absolute bottom-0 start-50 translate-middle-x">...</div>
                        </t>
                    </div>
                </button>
                <button t-if="!currentOrder.is_empty()"
                    t-on-click="() => pos.pay()" 
                    class="button pay-order-button btn btn-lg w-50"
                    t-attf-class="{{ this.highlightPay ? 'highlight btn-primary' : 'btn-light' }}" 
                >
                    Payment
                </button>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\product_screen\order_summary\order_summary.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.OrderSummary" t-inherit="point_of_sale.OrderSummary" t-inherit-mode="extension">
        <xpath expr="//Orderline" position="attributes">
            <attribute name="t-on-dblclick">
                () => line.toggleSkipChange()
            </attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\receipt_screen\receipt_screen.js

```javascript
import { ReceiptScreen } from "@point_of_sale/app/screens/receipt_screen/receipt_screen";
import { patch } from "@web/core/utils/patch";

patch(ReceiptScreen.prototype, {
    //@override
    _addNewOrder() {
        if (!this.pos.config.module_pos_restaurant) {
            super._addNewOrder(...arguments);
        }
    },
    continueSplitting() {
        const originalOrderUuid = this.currentOrder.uiState.splittedOrderUuid;
        this.currentOrder.uiState.screen_data.value = "";
        this.currentOrder.uiState.locked = true;
        this.pos.selectedOrderUuid = originalOrderUuid;
        this.pos.showScreen("ProductScreen");
    },
    isContinueSplitting() {
        if (this.pos.config.module_pos_restaurant && this.currentOrder.originalSplittedOrder) {
            const o = this.currentOrder.originalSplittedOrder;
            return !o.finalized && o.lines.length;
        } else {
            return false;
        }
    },
    //@override
    get nextScreen() {
        if (this.pos.config.module_pos_restaurant) {
            const table = this.pos.selectedTable;
            return { name: "FloorScreen", props: { floor: table ? table.floor_id : null } };
        } else {
            return super.nextScreen;
        }
    },
});

```

## File: static\src\overrides\components\receipt_screen\receipt_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_restaurant.ReceiptScreen" t-inherit="point_of_sale.ReceiptScreen" t-inherit-mode="extension">
        <xpath expr="//div[@id='action_btn_desktop']" position="inside">
            <button t-if="isContinueSplitting()" class="button next validation btn btn-primary w-100 py-5 rounded-0 fs-2" t-att-class="{ highlight: !locked }" t-on-click="continueSplitting" name="resume">
                <i class="fa fa-chevron-right" role="img" aria-label="Pay" title="Pay" />
                Continue
            </button>
        </xpath>
        <xpath expr="//div[@id='action_btn_mobile']" position="inside">
            <div t-if="isContinueSplitting()" class="btn-switchpane validation-button btn btn-primary flex-fill d-flex justify-content-center align-items-center rounded-0 fw-bolder fs-1" t-att-class="{ highlight: !locked }" t-on-click="continueSplitting" name="resume">
                Continue
            </div>
        </xpath>
        <xpath expr="//div[@id='action_btn_desktop']/button[hasclass('validation')]" position="attributes">
            <attribute name="t-if">!isContinueSplitting()</attribute>
        </xpath>
        <xpath expr="//div[@id='action_btn_mobile']/div[hasclass('validation-button')]" position="attributes">
            <attribute name="t-if">!isContinueSplitting()</attribute>
        </xpath>
    </t>
</templates>

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
            <t t-if="props.data.table">Table <t t-esc="props.data.table" /></t>
            <t t-if="props.data.table and props.data.customer_count">, Guests: <t t-esc="props.data.customer_count" /></t>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
import { useAutofocus } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";
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

            if (this.pos.models["restaurant.floor"].length > 0) {
                floorAndTable = `${table.floor_id.name}/`;
            }

            floorAndTable += table.getName();
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
                repr: (order) => order.table_id?.getName() || "",
                displayName: _t("Table"),
                modelField: "table_id.table_number",
            },
        });
    },
    async _setOrder(order) {
        const shouldBeOverridden = this.pos.config.module_pos_restaurant && order.table_id;
        if (!shouldBeOverridden) {
            return super._setOrder(...arguments);
        }
        // we came from the FloorScreen
        const orderTable = order.getTable();
        await this.pos.setTable(orderTable, order.uuid);
        this.closeTicketScreen();
    },
    async settleTips() {
        const promises = [];
        for (const order of this.getFilteredOrderList()) {
            const amount = this.env.utils.parseValidFloat(order.uiState.TipScreen.inputTipAmount);

            if (typeof order.id === "string") {
                console.warn(
                    `${order.name} is not yet sync. Sync it to server before setting a tip.`
                );
                continue;
            }

            order.state = "draft";
            this.pos.selectedOrderUuid = order.uuid;
            this.pos.set_tip(amount);
            order.state = "paid";
            order.uiState.screen_data.value = { name: "", props: {} };

            const serializedTipLine = order.get_selected_orderline().serialize({ orm: true });
            order.get_selected_orderline().delete();

            promises.push(
                new Promise((resolve) => {
                    const fn = async () => {
                        const tipLine = await this.pos.data.create("pos.order.line", [
                            serializedTipLine,
                        ]);
                        const state = await this.pos.data.ormWrite("pos.order", [order.id], {
                            is_tipped: true,
                            tip_amount: tipLine[0].price_unit,
                        });

                        if (state) {
                            order.update({
                                isTipped: true,
                                tipAmount: tipLine[0].price_unit,
                            });
                        }
                        resolve();
                    };
                    fn();
                })
            );
        }

        await Promise.all(promises);
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
        if (this.pos.config.module_pos_restaurant && order && !this.pos.selectedTable) {
            await this.pos.setTable(
                order.table ? order.table : this.pos.models["restaurant.table"].getAll()[0]
            );
        }
        await super.onDoRefund(...arguments);
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
    static props = {
        order: Object,
    };

    setup() {
        this.state = useState({ isEditing: false });
        this.orderUiState = this.props.order.uiState.TipScreen;
        useAutofocus();
    }
    get tipAmountStr() {
        return this.env.utils.formatCurrency(
            this.env.utils.parseValidFloat(this.orderUiState.inputTipAmount)
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
            <div t-if="state.filter == 'TIPPING'" class="col end narrow p-2" name="tip">Tip</div>
        </xpath>
        <xpath expr="//div[hasclass('order-row')]//div[@name='delete']" position="before">
            <div t-if="pos.config.module_pos_restaurant" class="col p-2" name="table">
                <t t-if="order.table_id">
                    <div t-if="ui.isSmall">Table</div>
                    <div><t t-esc="getTable(order)"></t></div>
                </t>
                <div t-elif="pos.config.module_pos_restaurant" t-esc="order.getFloatingOrderName()" />
            </div>
            <div t-if="state.filter == 'TIPPING'" class="col end narrow p-2" name="tip">
                <div t-if="ui.isSmall">Tip</div>
                <div><TipCell order="order" /></div>
            </div>
        </xpath>
        <xpath expr="//div[hasclass('mobileOrderList')]//div[hasclass('orderStatus')]" position="before">
            <div t-if="order.table_id" t-esc="getTable(order)" />
            <div t-elif="pos.config.module_pos_restaurant" t-esc="order.getFloatingOrderName()" />
        </xpath>
        <xpath expr="//div[hasclass('buttons')]" position="inside">
            <button class="settle-tips btn btn-lg btn-primary" t-if="state.filter == 'TIPPING'" t-on-click="settleTips">Settle</button>
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

## File: static\src\overrides\models\devices_synchronisation.js

```javascript
import DevicesSynchronisation from "@point_of_sale/app/store/devices_synchronisation";
import { patch } from "@web/core/utils/patch";

patch(DevicesSynchronisation.prototype, {
    processDynamicRecords(dynamicRecords) {
        const result = super.processDynamicRecords(dynamicRecords);
        if (!dynamicRecords["pos.order"]?.length) {
            return result;
        }
        // Verify if there is only 1 order by table.
        const orderByTableId = this.models["pos.order"].reduce((acc, order) => {
            // Floating order doesn't need to be verified.
            if (!order.finalized && order.table_id?.id) {
                acc[order.table_id.id] = acc[order.table_id.id] || [];
                acc[order.table_id.id].push(order);
            }
            return acc;
        }, {});

        for (const orders of Object.values(orderByTableId)) {
            if (orders.length > 1) {
                // The only way to get here is if there is several waiters on the same table.
                // In this case we take orderline of the local order and we add it to the synced order.
                const localOrders = orders.filter((order) => typeof order.id !== "number");
                const syncedOrder = orders
                    .filter((order) => typeof order.id === "number")
                    .sort((a, b) => a.id - b.id);

                if (
                    (syncedOrder.length === 0 || localOrders.length === 0) &&
                    syncedOrder.length <= 1 &&
                    localOrders.length <= 1
                ) {
                    continue;
                }

                const uniqOrder = syncedOrder.pop();
                for (const order of [...localOrders, ...syncedOrder]) {
                    let watcher = 0;
                    while (order.lines.length > 0) {
                        if (watcher > 1000) {
                            break;
                        }

                        const line = order.lines.pop();
                        line.update({ order_id: uniqOrder });
                        line.setDirty();
                        watcher++;
                    }
                }

                const localIds = [
                    ...localOrders.map((order) => order.uuid),
                    ...syncedOrder.map((order) => order.uuid),
                ];
                if (localIds.includes(this.pos.selectedOrderUuid)) {
                    this.pos.set_order(uniqOrder);
                    this.pos.addPendingOrder([uniqOrder.id]);
                }

                this.pos.deleteOrders(syncedOrder);
                this.pos.models["pos.order"].deleteMany(localOrders);
            }
        }
    },
});

```

## File: static\src\overrides\models\payment.js

```javascript
import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { patch } from "@web/core/utils/patch";

patch(PaymentInterface.prototype, {
    /**
     * Return true if the amount that was authorized can be modified,
     * false otherwise
     * @param {string} uuid - The id of the paymentline
     */
    canBeAdjusted(uuid) {
        return false;
    },

    /**
     * Called when the amount authorized by a payment request should
     * be adjusted to account for a new order line, it can only be called if
     * canBeAdjusted returns True
     * @param {string} uuid - The id of the paymentline
     */
    send_payment_adjust(uuid) {},
});

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";

patch(PosOrder.prototype, {
    setup(_defaultObj, options) {
        super.setup(...arguments);
        if (this.config.module_pos_restaurant) {
            this.customer_count = this.customer_count || 1;
        }
    },
    getCustomerCount() {
        return this.customer_count;
    },
    setCustomerCount(count) {
        this.customer_count = Math.max(count, 0);
    },
    getTable() {
        return this.table_id;
    },
    amountPerGuest(numCustomers = this.customer_count) {
        if (numCustomers === 0) {
            return 0;
        }
        return this.getTotalDue() / numCustomers;
    },
    export_for_printing(baseUrl, headerData) {
        return {
            ...super.export_for_printing(...arguments),
            set_tip_after_payment: this.config.set_tip_after_payment,
            isRestaurant: this.config.module_pos_restaurant,
        };
    },
    setBooked(booked) {
        this.uiState.booked = booked;
    },
    getName() {
        if (this.config.module_pos_restaurant && this.getTable()) {
            const table = this.getTable();
            const child_tables = this.models["restaurant.table"].filter((t) => {
                if (t.floor_id.id === table.floor_id.id) {
                    return table.isParent(t);
                }
            });
            let name = table.table_number.toString();
            for (const child_table of child_tables) {
                name += ` & ${child_table.table_number}`;
            }
            return name;
        }
        return super.getName(...arguments);
    },
});

```

## File: static\src\overrides\models\pos_order_line.js

```javascript
import { PosOrderline } from "@point_of_sale/app/models/pos_order_line";
import { patch } from "@web/core/utils/patch";

patch(PosOrderline.prototype, {
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
    get_line_diff_hash() {
        if (this.getNote()) {
            return this.id + "|" + this.getNote();
        } else {
            return "" + this.id;
        }
    },
    toggleSkipChange() {
        if (this.uiState.hasChange || this.skip_change) {
            this.setDirty();
            this.skip_change = !this.skip_change;
        }
    },
    getDisplayClasses() {
        return {
            ...super.getDisplayClasses(),
            "has-change": this.uiState.hasChange && this.config.module_pos_restaurant,
            "skip-change": this.skip_change && this.config.module_pos_restaurant,
        };
    },
});

```

## File: static\src\overrides\models\pos_payment.js

```javascript
import { PosPayment } from "@point_of_sale/app/models/pos_payment";
import { patch } from "@web/core/utils/patch";

patch(PosPayment.prototype, {
    //@override
    canBeAdjusted() {
        if (this.payment_method_id.payment_terminal) {
            return this.payment_method_id.payment_terminal.canBeAdjusted(this.uuid);
        }
        return !this.payment_method_id.is_cash_count;
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { ConnectionLostError } from "@web/core/network/rpc";
import { _t } from "@web/core/l10n/translation";

patch(PosStore.prototype, {
    /**
     * @override
     */
    async setup() {
        this.isEditMode = false;
        this.tableSyncing = false;
        await super.setup(...arguments);
    },
    get idleTimeout() {
        return [
            ...super.idleTimeout,
            {
                timeout: 180000, // 3 minutes
                action: () =>
                    this.dialog.closeAll() &&
                    this.config.module_pos_restaurant &&
                    this.mainScreen.component.name !== "PaymentScreen" &&
                    this.showScreen("FloorScreen"),
            },
        ];
    },
    get firstScreen() {
        const screen = super.firstScreen;

        if (!this.config.module_pos_restaurant) {
            return screen;
        }

        return screen === "LoginScreen" ? "LoginScreen" : "FloorScreen";
    },
    async onDeleteOrder(order) {
        const orderIsDeleted = await super.onDeleteOrder(...arguments);
        if (
            this.config.module_pos_restaurant &&
            orderIsDeleted &&
            this.mainScreen.component.name !== "TicketScreen"
        ) {
            this.showScreen("FloorScreen");
        }
        return orderIsDeleted;
    },
    async closingSessionNotification() {
        await super.closingSessionNotification(...arguments);
        this.computeTableCount();
    },
    computeTableCount() {
        const tables = this.models["restaurant.table"].getAll();
        const orders = this.get_open_orders();
        for (const table of tables) {
            const tableOrders = orders.filter(
                (order) => order.table_id?.id === table.id && !order.finalized
            );
            const qtyChange = tableOrders.reduce(
                (acc, order) => {
                    const quantitySkipped = this.getOrderChanges(true, order);
                    const quantityChange = this.getOrderChanges(false, order);
                    acc.changed += quantityChange.count;
                    acc.skipped += quantitySkipped.count;
                    return acc;
                },
                { changed: 0, skipped: 0 }
            );

            table.uiState.orderCount = tableOrders.length;
            table.uiState.changeCount = qtyChange.changed;
        }
    },
    get categoryCount() {
        const orderChanges = this.getOrderChanges();
        const linesChanges = orderChanges.orderlines;

        const categories = Object.values(linesChanges).reduce((acc, curr) => {
            const categories =
                this.models["product.product"].get(curr.product_id)?.pos_categ_ids || [];

            for (const category of categories.slice(0, 1)) {
                if (!acc[category.id]) {
                    acc[category.id] = {
                        count: curr.quantity,
                        name: category.name,
                    };
                } else {
                    acc[category.id].count += curr.quantity;
                }
            }

            return acc;
        }, {});

        const nbNoteChange = Object.keys(orderChanges.noteUpdated).length;
        if (nbNoteChange) {
            categories["noteUpdate"] = { count: nbNoteChange, name: _t("Note") };
        }
        // Only send modeUpdate if there's already an older mode in progress.
        const currentOrder = this.get_order();
        if (
            orderChanges.modeUpdate &&
            Object.keys(currentOrder.last_order_preparation_change.lines).length
        ) {
            const displayName = currentOrder.takeaway ? _t("Take out") : _t("Dine in");
            categories["modeUpdate"] = { count: 1, name: displayName };
        }

        return [
            ...Object.values(categories),
            ...("generalNote" in orderChanges ? [{ count: 1, name: _t("Message") }] : []),
        ];
    },
    createNewOrder() {
        const order = super.createNewOrder(...arguments);

        if (this.config.module_pos_restaurant && this.selectedTable && !order.table_id) {
            order.update({ table_id: this.selectedTable });
        }

        return order;
    },
    getReceiptHeaderData(order) {
        const json = super.getReceiptHeaderData(...arguments);
        if (this.config.module_pos_restaurant && order) {
            if (order.getTable()) {
                json.table = order.getTable().table_number;
            }
            json.customer_count = order.getCustomerCount();
        }
        return json;
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
    //@override
    async afterProcessServerData() {
        this.floorPlanStyle =
            localStorage.getItem("floorPlanStyle") || (this.ui.isSmall ? "kanban" : "default");
        if (this.config.module_pos_restaurant) {
            this.currentFloor = this.config.floor_ids?.length > 0 ? this.config.floor_ids[0] : null;
        }

        const res = await super.afterProcessServerData(...arguments);
        if (this.config.module_pos_restaurant) {
            this.selectedTable = null;
        }
        return res;
    },
    //@override
    add_new_order() {
        const order = super.add_new_order(...arguments);
        if (this.config.module_pos_restaurant) {
            this.addPendingOrder([order.id]);
        }
        return order;
    },
    async addLineToCurrentOrder(vals, opts = {}, configure = true) {
        if (this.config.module_pos_restaurant) {
            const order = this.get_order();
            this.addPendingOrder([order.id]);
            if (!this.get_order().uiState.booked) {
                this.get_order().setBooked(true);
            }
        }
        return super.addLineToCurrentOrder(vals, opts, configure);
    },
    async getServerOrders() {
        if (this.config.module_pos_restaurant) {
            const tableIds = [].concat(
                ...this.models["restaurant.floor"].map((floor) =>
                    floor.table_ids.map((table) => table.id)
                )
            );
            await this.syncAllOrders({ table_ids: tableIds });
        }
        //Need product details from backand to UI for urbanpiper
        return await super.getServerOrders();
    },
    getDefaultSearchDetails() {
        if (this.selectedTable && this.selectedTable.id) {
            return {
                fieldName: "TABLE",
                searchTerm: this.selectedTable.getName(),
            };
        }
        return super.getDefaultSearchDetails();
    },
    async setTable(table, orderUuid = null) {
        this.deviceSync.readDataFromServer();
        this.selectedTable = table;
        let currentOrder = table
            ? table.orders.find((o) => o.uuid === orderUuid || !o.finalized)
            : null;

        if (currentOrder) {
            this.set_order(currentOrder);
        } else {
            const potentialsOrders = this.models["pos.order"].filter(
                (o) => !o.table_id && !o.finalized && o.lines.length === 0
            );

            if (potentialsOrders.length) {
                currentOrder = potentialsOrders[0];
                currentOrder.update({ table_id: table });
                this.selectedOrderUuid = currentOrder.uuid;
            } else {
                this.add_new_order();
            }
        }
    },
    async setTableFromUi(table, orderUuid = null) {
        try {
            this.tableSyncing = true;
            if (table.parent_id) {
                table = table.getParent();
            }
            await this.setTable(table, orderUuid);
        } catch (e) {
            if (!(e instanceof ConnectionLostError)) {
                throw e;
            }
            // Reject error in a separate stack to display the offline popup, but continue the flow
            Promise.reject(e);
        } finally {
            this.tableSyncing = false;
            const orders = this.getTableOrders(table.id);
            if (orders.length > 0) {
                this.set_order(orders[0]);
                const props = {};
                if (orders[0].get_screen_data().name === "PaymentScreen") {
                    props.orderUuid = orders[0].uuid;
                }
                this.showScreen(orders[0].get_screen_data().name, props);
            } else {
                this.add_new_order();
                this.showScreen("ProductScreen");
            }
        }
    },
    getTableOrders(tableId) {
        return this.get_open_orders().filter((order) => order.table_id?.id === tableId);
    },
    async unsetTable() {
        this.selectedTable = null;
        const order = this.get_order();
        if (order && !order.isBooked) {
            this.removeOrder(order);
        } else if (order && this.previousScreen !== "ReceiptScreen") {
            if (!this.orderToTransferUuid) {
                this.syncAllOrders({ orders: [order] });
            } else {
                await this.syncAllOrders({ orders: [order] });
            }
        }
        this.set_order(null);
    },
    getActiveOrdersOnTable(table) {
        return this.models["pos.order"].filter(
            (o) => o.table_id?.id === table.id && !o.finalized && o.lines.length
        );
    },
    tableHasOrders(table) {
        return Boolean(table.getOrder());
    },
    getTableFromElement(el) {
        return this.models["restaurant.table"].get(
            [...el.classList].find((c) => c.includes("tableId")).split("-")[1]
        );
    },
    mergePreparationLines(preparationLine, destPreparationLine, destinationOrder, destOrderLine) {
        if (preparationLine && destPreparationLine) {
            destPreparationLine.quantity += preparationLine.quantity;
            preparationLine.quantity = 0;
        } else if (preparationLine) {
            const preparationLineCopy = { ...preparationLine };
            preparationLineCopy.order_id = destinationOrder.id;
            preparationLineCopy.uuid = destOrderLine.uuid;
            destinationOrder.last_order_preparation_change.lines[destOrderLine.preparationKey] =
                preparationLineCopy;
            preparationLine.quantity = 0;
        }
    },
    async transferOrder(orderUuid, destinationTable) {
        const order = this.models["pos.order"].getBy("uuid", orderUuid);
        const destinationOrder = this.getActiveOrdersOnTable(destinationTable)[0];
        await this.syncAllOrders({ orders: [destinationOrder || order] });
        const originalTable = order.table_id;
        this.loadingOrderState = false;
        this.alert.dismiss();
        if (destinationTable.id === originalTable?.id) {
            this.set_order(order);
            await this.setTable(destinationTable);
            return;
        }
        if (!this.tableHasOrders(destinationTable)) {
            order.update({ table_id: destinationTable });
            this.set_order(order);
            this.addPendingOrder([order.id]);
        } else {
            for (const orphanLine of order.lines) {
                const adoptingLine = destinationOrder.lines.find((l) =>
                    l.can_be_merged_with(orphanLine)
                );
                if (adoptingLine) {
                    adoptingLine.merge(orphanLine);
                    this.mergePreparationLines(
                        order.last_order_preparation_change.lines[orphanLine.preparationKey],
                        destinationOrder.last_order_preparation_change.lines[
                            adoptingLine.preparationKey
                        ],
                        destinationOrder,
                        adoptingLine
                    );
                } else {
                    const serialized = orphanLine.serialize();
                    serialized.order_id = destinationOrder.id;
                    delete serialized.uuid;
                    delete serialized.id;
                    const newOrderLine = this.models["pos.order.line"].create(
                        serialized,
                        false,
                        true
                    );

                    const preparationLine =
                        order.last_order_preparation_change.lines[orphanLine.preparationKey];
                    if (preparationLine) {
                        const preparationLineCopy = { ...preparationLine };
                        preparationLineCopy.order_id = destinationOrder.id;
                        destinationOrder.last_order_preparation_change.lines[
                            newOrderLine.preparationKey
                        ] = preparationLineCopy;
                        preparationLine.quantity = 0;
                    }
                }
            }

            this.set_order(destinationOrder);
            await this.deleteOrders([order]);
        }

        await this.syncAllOrders({ orders: [destinationOrder || order] });
        await this.setTable(destinationTable);
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
    _shouldLoadOrders() {
        return super._shouldLoadOrders() || this.config.module_pos_restaurant;
    },
    get showSaveOrderButton() {
        return super.showSaveOrderButton && !this.config.module_pos_restaurant;
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
                        <field name="floor_background_image" widget='image' class="oe_avatar" options='{"preview_image": "floor_background_image"}'/>
                        <group col="4">
                            <field name="name" />
                            <field name="pos_config_ids" widget="many2many_tags"/>
                            <field name="background_color" groups="base.group_no_one" />
                        </group>
                        <field name="table_ids">
                            <list string='Tables' editable="bottom">
                                <field name="table_number" />
                                <field name="seats" />
                                <field name="shape" />
                                <field name="height" optional="hide" />
                                <field name="width" optional="hide" />
                                <field name="color" widget="color" optional="hide" />
                                <field name="active" optional="hide" />
                            </list>
                        </field>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_restaurant_floor_tree" model="ir.ui.view">
            <field name="name">Restaurant Floors</field>
            <field name="model">restaurant.floor</field>
            <field name="arch" type="xml">
                <list string="Restaurant Floors">
                    <field name="sequence" widget="handle" />
                    <field name="name" />
                    <field name="pos_config_ids" widget="many2many_tags"/>
                </list>
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
                    <templates>
                        <t t-name="card">
                            <div><strong>Floor Name: </strong><field name="name"/></div>
                            <div class="d-flex">
                                <strong>Point of Sales:</strong><field name="pos_config_ids" class="ms-1"/>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="action_restaurant_floor_form" model="ir.actions.act_window">
            <field name="name">Floor Plans</field>
            <field name="res_model">restaurant.floor</field>
            <field name="view_mode">list,kanban,form</field>
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
                            <field name="table_number" />
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
                <setting id="floor_and_table_map" string="Floors &amp; Tables Map" help="Design floors and assign orders to tables" documentation="/applications/sales/point_of_sale/restaurant/floors_tables.html" invisible="is_kiosk_mode or not pos_module_pos_restaurant">
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
                <setting string="Eat in / Take out" help="Adjust the tax rate based on whether customers are dining in or opting for takeout."  invisible="not pos_module_pos_restaurant">
                    <field name="pos_takeaway"/>
                    <div class="text-warning mb16" invisible="not pos_takeaway">Taxes must be included in price.</div>
                    <div class="content-group" invisible="not pos_takeaway">
                        <label string="" for="pos_takeaway_fp_id" class="me-2"/>
                        <field name="pos_takeaway_fp_id" placeholder="Alternative Fiscal Position"/>
                        <div>
                            <button name="%(account.action_account_fiscal_position_form)d" icon="oi-arrow-right" type="action" string="Fiscal Positions" class="btn-link"/>
                        </div>
                    </div>
                </setting>
                <setting string="Early Receipt Printing" help="Allow to print receipt before payment" id="iface_printbill"  invisible="not pos_module_pos_restaurant or is_kiosk_mode">
                    <field name="pos_iface_printbill"/>
                </setting>
                <setting help="Split total or order lines" id="iface_splitbill"  documentation="/applications/sales/point_of_sale/restaurant/bill_printing.html" invisible="not pos_module_pos_restaurant or is_kiosk_mode">
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
            <setting id="flexible_taxes" position="attributes">
                <attribute name="invisible">pos_takeaway</attribute>
            </setting>
        </field>
    </record>
</odoo>

```

