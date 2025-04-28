# Odoo Module: product

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Products & Pricelists',
    'version': '1.2',
    'category': 'Sales/Sales',
    'depends': ['base', 'mail', 'uom'],
    'description': """
This is the base module for managing products and pricelists in Odoo.
========================================================================

Products support variants, different pricing methods, vendors information,
make to stock/order, different units of measure, packaging and properties.

Pricelists support:
-------------------
    * Multiple-level of discount (by product, category, quantities)
    * Compute price based on different criteria:
        * Other pricelist
        * Cost price
        * List price
        * Vendor price

Pricelists preferences by product and/or partners.

Print product labels with barcode.
    """,
    'data': [
        'data/product_data.xml',
        'security/product_security.xml',
        'security/ir.model.access.csv',
        'wizard/product_price_list_views.xml',
        'views/res_config_settings_views.xml',
        'views/product_attribute_views.xml',
        'views/product_views.xml',
        'views/product_template_views.xml',
        'views/product_pricelist_views.xml',
        'views/res_partner_views.xml',
        'report/product_reports.xml',
        'report/product_pricelist_templates.xml',
        'report/product_product_templates.xml',
        'report/product_template_templates.xml',
        'report/product_packaging.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\product_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="product_category_all" model="product.category">
            <field name="name">All</field>
        </record>
        <record id="product_category_1" model="product.category">
            <field name="parent_id" ref="product_category_all"/>
            <field name="name">Saleable</field>
        </record>
        <record id="cat_expense" model="product.category">
            <field name="parent_id" ref="product_category_all"/>
            <field name="name">Expenses</field>
        </record>

        <!--
             Precisions
        -->
        <record forcecreate="True" id="decimal_price" model="decimal.precision">
            <field name="name">Product Price</field>
            <field name="digits">2</field>
        </record>
        <record forcecreate="True" id="decimal_discount" model="decimal.precision">
            <field name="name">Discount</field>
            <field name="digits">2</field>
        </record>
        <record forcecreate="True" id="decimal_stock_weight" model="decimal.precision">
            <field name="name">Stock Weight</field>
            <field name="digits">2</field>
        </record>
        <record forcecreate="True" id="decimal_volume" model="decimal.precision">
            <field name="name">Volume</field>
            <field name="digits">2</field>
        </record>
        <record forcecreate="True" id="decimal_product_uom" model="decimal.precision">
            <field name="name">Product Unit of Measure</field>
            <field name="digits" eval="3"/>
        </record>

        <!--
... to here, it should be in product_demo but we cant just move it
there yet otherwise people who have installed the server (even with the without-demo
parameter) will see those record just disappear.
-->

        <!-- Price list -->
        <record id="list0" model="product.pricelist">
            <field name="name">Public Pricelist</field>
            <field name="sequence">1</field>
        </record>

        <!--
        Property
        -->

    </data>
</odoo>

```

## File: data\product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- We want to activate product variant by default for easier demoing. -->
        <record id="base.group_user" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('product.group_product_variant'))]"/>
        </record>

        <record id="product_category_2" model="product.category">
            <field name="parent_id" ref="product.product_category_all"/>
            <field name="name">Internal</field>
        </record>
        <record id="product_category_3" model="product.category">
            <field name="parent_id" ref="product.product_category_1"/>
            <field name="name">Services</field>
        </record>
        <record id="product_category_6" model="product.category">
            <field name="parent_id" ref="product.product_category_3"/>
            <field name="name">Saleable</field>
        </record>
        <record id="product_category_4" model="product.category">
            <field name="parent_id" ref="product.product_category_1"/>
            <field name="name">Software</field>
        </record>
        <record id="product_category_5" model="product.category">
            <field name="parent_id" ref="product_category_1"/>
            <field name="name">Office Furniture</field>
        </record>
        <record id="product_category_consumable" model="product.category">
            <field name="parent_id" ref="product_category_all"/>
            <field name="name">Consumable</field>
        </record>

        <!-- Expensable products -->
        <record id="expense_product" model="product.product">
            <field name="name">Restaurant Expenses</field>
            <field name="list_price">14.0</field>
            <field name="standard_price">8.0</field>
            <field name="type">service</field>
            <field name="default_code">EXP_REST</field>
            <field name="categ_id" ref="product.cat_expense"/>
        </record>

        <record id="expense_hotel" model="product.product">
            <field name="name">Hotel Accommodation</field>
            <field name="list_price">400.0</field>
            <field name="standard_price">400.0</field>
            <field name="type">service</field>
            <field name="default_code">EXP_HA</field>
            <field name="uom_id" ref="uom.product_uom_day"/>
            <field name="uom_po_id" ref="uom.product_uom_day"/>
            <field name="categ_id" ref="cat_expense"/>
        </record>

        <!-- Service products -->
        <record id="product_product_1" model="product.product">
            <field name="name">Virtual Interior Design</field>
            <field name="categ_id" ref="product_category_3"/>
            <field name="standard_price">20.5</field>
            <field name="list_price">30.75</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
        </record>

        <record id="product_product_2" model="product.product">
            <field name="name">Virtual Home Staging</field>
            <field name="categ_id" ref="product_category_3"/>
            <field name="standard_price">25.5</field>
            <field name="list_price">38.25</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
        </record>

        <!-- Physical Products -->

        <record id="product_delivery_01" model="product.product">
            <field name="name">Office Chair</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">55.0</field>
            <field name="list_price">70.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_7777</field>
            <field name="image_1920" type="base64" file="product/static/img/product_chair.png"/>
        </record>

        <record id="product_delivery_02" model="product.product">
            <field name="name">Office Lamp</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">35.0</field>
            <field name="list_price">40.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_8888</field>
            <field name="image_1920" type="base64" file="product/static/img/product_lamp.png"/>
        </record>

        <record id="product_order_01" model="product.product">
            <field name="name">Office Design Software</field>
            <field name="categ_id" ref="product_category_4"/>
            <field name="standard_price">235.0</field>
            <field name="list_price">280.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_9999</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_43-image.jpg"/>
        </record>

        <record id="product_product_3" model="product.product">
            <field name="name">Desk Combination</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="list_price">450.0</field>
            <field name="standard_price">300.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Desk combination, black-brown: chair + desk + drawer.</field>
            <field name="default_code">FURN_7800</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_3-image.jpg"/>
        </record>

        <!-- Variants -->

        <record id="product_attribute_1" model="product.attribute">
            <field name="name">Legs</field>
            <field name="sequence">10</field>
        </record>
        <record id="product_attribute_value_1" model="product.attribute.value">
            <field name="name">Steel</field>
            <field name="attribute_id" ref="product_attribute_1"/>
            <field name="sequence">1</field>
        </record>
        <record id="product_attribute_value_2" model="product.attribute.value">
            <field name="name">Aluminium</field>
            <field name="attribute_id" ref="product_attribute_1"/>
            <field name="sequence">2</field>
        </record>

        <record id="product_attribute_2" model="product.attribute">
            <field name="name">Color</field>
            <field name="sequence">20</field>
        </record>
        <record id="product_attribute_value_3" model="product.attribute.value">
            <field name="name">White</field>
            <field name="attribute_id" ref="product_attribute_2"/>
            <field name="sequence">1</field>
        </record>
        <record id="product_attribute_value_4" model="product.attribute.value">
            <field name="name">Black</field>
            <field name="attribute_id" ref="product_attribute_2"/>
            <field name="sequence">2</field>
        </record>

        <record id="product_attribute_3" model="product.attribute">
            <field name="name">Duration</field>
            <field name="sequence">30</field>
        </record>
        <record id="product_attribute_value_5" model="product.attribute.value">
            <field name="name">1 year</field>
            <field name="attribute_id" ref="product_attribute_3"/>
        </record>
        <record id="product_attribute_value_6" model="product.attribute.value">
            <field name="name">2 year</field>
            <field name="attribute_id" ref="product_attribute_3"/>
        </record>

        <record id="product_product_4_product_template" model="product.template">
            <field name="name">Customizable Desk (CONFIG)</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">500.0</field>
            <field name="list_price">750.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">160x80cm, with large legs.</field>
        </record>

        <!-- the product template attribute lines have to be defined before creating the variants -->
        <record id="product_4_attribute_1_product_template_attribute_line" model="product.template.attribute.line">
            <field name="product_tmpl_id" ref="product_product_4_product_template"/>
            <field name="attribute_id" ref="product_attribute_1"/>
            <field name="value_ids" eval="[(6, 0, [ref('product.product_attribute_value_1'), ref('product.product_attribute_value_2')])]"/>
        </record>
        <record id="product_4_attribute_2_product_template_attribute_line" model="product.template.attribute.line">
            <field name="product_tmpl_id" ref="product_product_4_product_template"/>
            <field name="attribute_id" ref="product_attribute_2"/>
            <field name="value_ids" eval="[(6, 0, [ref('product.product_attribute_value_3'), ref('product.product_attribute_value_4')])]"/>
        </record>

        <!--
        Handle automatically created product.template.attribute.value.
        Meaning that the combination between the "customizable desk" and the attribute value "black" will be materialized
        into a "product.template.attribute.value" with the ref "product.product_4_attribute_1_value_1".
        This will allow setting fields like "price_extra" and "exclude_for"
         -->
        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'product.product_4_attribute_1_value_1',
                'record': obj().env.ref('product.product_4_attribute_1_product_template_attribute_line').product_template_value_ids[0],
                'noupdate': True,
            }, {
                'xml_id': 'product.product_4_attribute_1_value_2',
                'record': obj().env.ref('product.product_4_attribute_1_product_template_attribute_line').product_template_value_ids[1],
                'noupdate': True,
            }, {
                'xml_id': 'product.product_4_attribute_2_value_1',
                'record': obj().env.ref('product.product_4_attribute_2_product_template_attribute_line').product_template_value_ids[0],
                'noupdate': True,
            }, {
                'xml_id': 'product.product_4_attribute_2_value_2',
                'record': obj().env.ref('product.product_4_attribute_2_product_template_attribute_line').product_template_value_ids[1],
                'noupdate': True,
            },]"/>
        </function>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'product.product_product_4',
                'record': obj().env.ref('product.product_product_4_product_template')._get_variant_for_combination(obj().env.ref('product.product_4_attribute_1_value_1') + obj().env.ref('product.product_4_attribute_2_value_1')),
                'noupdate': True,
            }, {
                'xml_id': 'product.product_product_4b',
                'record': obj().env.ref('product.product_product_4_product_template')._get_variant_for_combination(obj().env.ref('product.product_4_attribute_1_value_1') + obj().env.ref('product.product_4_attribute_2_value_2')),
                'noupdate': True,
            }, {
                'xml_id': 'product.product_product_4c',
                'record': obj().env.ref('product.product_product_4_product_template')._get_variant_for_combination(obj().env.ref('product.product_4_attribute_1_value_2') + obj().env.ref('product.product_4_attribute_2_value_1')),
                'noupdate': True,
            }, {
                'xml_id': 'product.product_product_4d',
                'record': obj().env.ref('product.product_product_4_product_template')._get_variant_for_combination(obj().env.ref('product.product_4_attribute_1_value_2') + obj().env.ref('product.product_4_attribute_2_value_2')),
                'noupdate': True,
            },]"/>
        </function>

        <record id="product_product_4" model="product.product">
            <field name="default_code">FURN_0096</field>
            <field name="standard_price">500.0</field>
            <field name="weight">0.01</field>
            <field name="image_1920" type="base64" file="product/static/img/table02.png"/>
        </record>
        <record id="product_product_4b" model="product.product">
            <field name="default_code">FURN_0097</field>
            <field name="weight">0.01</field>
            <field name="standard_price">500.0</field>
            <field name="image_1920" type="base64" file="product/static/img/table04.png"/>
        </record>
        <record id="product_product_4c" model="product.product">
            <field name="default_code">FURN_0098</field>
            <field name="weight">0.01</field>
            <field name="standard_price">500.0</field>
            <field name="image_1920" type="base64" file="product/static/img/table03.png"/>
        </record>
        <record id="product_product_4d" model="product.product">
            <field name="default_code">DESK0004</field>
            <field name="weight">0.01</field>
            <field name="standard_price">500.0</field>
            <field name="image_1920" type="base64" file="product/static/img/table01.png"/>
        </record>

        <record id="product_product_5" model="product.product">
            <field name="name">Corner Desk Right Sit</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">600.0</field>
            <field name="list_price">147.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM06</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_5-image.png"/>
        </record>

        <record id="product_product_6" model="product.product">
            <field name="name">Large Cabinet</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">800.0</field>
            <field name="list_price">320.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM07</field>
            <field name='weight'>0.330</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_6-image.png"/>
        </record>

        <record id="product_product_7" model="product.product">
            <field name="name">Storage Box</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">70.0</field>
            <field name="list_price">79.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM08</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_7-image.png"/>
        </record>

        <record id="product_product_8" model="product.product">
            <field name="name">Large Desk</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">1299.0</field>
            <field name="list_price">1799.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM09</field>
            <field name='weight'>9.54</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_8-image.png"/>
        </record>

        <record id="product_product_9" model="product.product">
            <field name="name">Pedal Bin</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">10.0</field>
            <field name="list_price">47.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM10</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_9-image.png"/>
        </record>

        <record id="product_product_10" model="product.product">
            <field name="name">Cabinet with Doors</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">12.50</field>
            <field name="list_price">14</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM11</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_10-image.png"/>
        </record>

        <record id="product_product_11_product_template" model="product.template">
            <field name="name">Conference Chair (CONFIG)</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">14</field>
            <field name="list_price">16.50</field>
            <field name="type">consu</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="product/static/img/product_product_11-image.png"/>
        </record>

        <!-- the product template attribute lines have to be defined before creating the variants -->
        <record id="product_11_attribute_1_product_template_attribute_line" model="product.template.attribute.line">
            <field name="product_tmpl_id" ref="product_product_11_product_template"/>
            <field name="attribute_id" ref="product_attribute_1"/>
            <field name="value_ids" eval="[(6,0,[ref('product.product_attribute_value_1'), ref('product.product_attribute_value_2')])]"/>
        </record>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'product.product_11_attribute_1_value_1',
                'record': obj().env.ref('product.product_11_attribute_1_product_template_attribute_line').product_template_value_ids[0],
                'noupdate': True,
            }, {
                'xml_id': 'product.product_11_attribute_1_value_2',
                'record': obj().env.ref('product.product_11_attribute_1_product_template_attribute_line').product_template_value_ids[1],
                'noupdate': True,
            }]"/>
        </function>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'product.product_product_11',
                'record': obj().env.ref('product.product_product_11_product_template')._get_variant_for_combination(obj().env.ref('product.product_11_attribute_1_value_1')),
                'noupdate': True,
            }, {
                'xml_id': 'product.product_product_11b',
                'record': obj().env.ref('product.product_product_11_product_template')._get_variant_for_combination(obj().env.ref('product.product_11_attribute_1_value_2')),
                'noupdate': True,
            },]"/>
        </function>

        <record id="product_product_11" model="product.product">
            <field name="default_code">E-COM12</field>
            <field name="weight">0.01</field>
        </record>
        <record id="product_product_11b" model="product.product">
            <field name="default_code">E-COM13</field>
            <field name="weight">0.01</field>
        </record>

        <record id="product.product_4_attribute_1_value_2" model="product.template.attribute.value">
            <field name="price_extra">50.40</field>
        </record>

        <record id="product.product_11_attribute_1_value_2" model="product.template.attribute.value">
            <field name="price_extra">6.40</field>
        </record>

        <!-- MRP Demo Data-->

        <record id="product_product_12" model="product.product">
            <field name="name">Office Chair Black</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">18</field>
            <field name="list_price">12.50</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_0269</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_12-image.png"/>
        </record>

        <record id="product_product_13" model="product.product">
            <field name="name">Corner Desk Black</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">78.0</field>
            <field name="list_price">85.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_1118</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_13-image.png"/>
        </record>

        <record id="product_product_16" model="product.product">
            <field name="name">Drawer Black</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">20.0</field>
            <field name="list_price">25.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_8900</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_16-image.png"/>
        </record>

        <record id="product_product_20" model="product.product">
            <field name="name">Flipover</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">1700.0</field>
            <field name="list_price">1950.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_9001</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_20-image.png"/>
        </record>
        <record id="product_product_22" model="product.product">
            <field name="name">Desk Stand with Screen</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">2010.0</field>
            <field name="list_price">2100.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_7888</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_22-image.png"/>
        </record>

        <record id="product_product_24" model="product.product">
            <field name="name">Individual Workplace</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">876.0</field>
            <field name="list_price">885.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_0789</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_24-image.png"/>
        </record>

        <record id="product_product_25" model="product.product">
            <field name="name">Acoustic Bloc Screens</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">2870.0</field>
            <field name="list_price">2950.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_6666</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_25-image.png"/>
        </record>

        <record id="product_product_27" model="product.product">
            <field name="name">Drawer</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">3300.0</field>
            <field name="list_price">3645.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Drawer with two routing possiblities.</field>
            <field name="default_code">FURN_8855</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_27-image.png"/>
        </record>

        <record id="consu_delivery_03" model="product.product">
            <field name="name">Four Person Desk</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">25000.0</field>
            <field name="list_price">23500.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Four person modern office workstation</field>
            <field name="default_code">FURN_8220</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_d03-image.png"/>
        </record>

        <record id="consu_delivery_02" model="product.product">
            <field name="name">Large Meeting Table</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">45000.0</field>
            <field name="list_price">40000.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Conference room table</field>
            <field name="default_code">FURN_6741</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_46-image.png"/>
        </record>

        <record id="consu_delivery_01" model="product.product">
            <field name="name">Three-Seat Sofa</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">65000.0</field>
            <field name="list_price">60000.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Three Seater Sofa with Lounger in Steel Grey Colour</field>
            <field name="default_code">FURN_8999</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_d01-image.png"/>
        </record>

        <!--
    Resource: product.supplierinfo
    -->

        <record id="product_supplierinfo_1" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_6_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">750</field>
        </record>

        <record id="product_supplierinfo_2" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_6_product_template"/>
            <field name="name" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">790</field>
        </record>

        <record id="product_supplierinfo_2bis" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_6_product_template"/>
            <field name="name" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">3</field>
            <field name="price">785</field>
        </record>

        <record id="product_supplierinfo_3" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_7_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">65</field>
        </record>

        <record id="product_supplierinfo_4" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_7_product_template"/>
            <field name="name" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">72</field>
        </record>

        <record id="product_supplierinfo_5" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_8_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">2</field>
            <field name="min_qty">5</field>
            <field name="price">1299</field>
        </record>

        <record id="product_supplierinfo_6" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_8_product_template"/>
            <field name="name" ref="base.res_partner_12"/>
            <field name="delay">4</field>
            <field name="min_qty">1</field>
            <field name="price">1399</field>
        </record>

        <record id="product_supplierinfo_7" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_10_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">2</field>
            <field name="min_qty">1</field>
            <field name="price">12.50</field>
        </record>

        <record id="product_supplierinfo_8" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_11_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">2</field>
            <field name="min_qty">1</field>
            <field name="price">14</field>
        </record>

        <record id="product_supplierinfo_9" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_13_product_template"/>
            <field name="name" ref="base.res_partner_4"/>
            <field name="delay">5</field>
            <field name="min_qty">1</field>
            <field name="price">78</field>
        </record>

        <record id="product_supplierinfo_10" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_16_product_template"/>
            <field name="name" ref="base.res_partner_3"/>
            <field name="delay">1</field>
            <field name="min_qty">1</field>
            <field name="price">20</field>
        </record>

        <record id="product_supplierinfo_12" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_20_product_template"/>
            <field name="name" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">1700</field>
        </record>

        <record id="product_supplierinfo_13" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_20_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">4</field>
            <field name="min_qty">5</field>
            <field name="price">1720</field>
        </record>

        <record id="product_supplierinfo_14" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_22_product_template"/>
            <field name="name" ref="base.res_partner_2"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">2010</field>
        </record>

        <record id="product_supplierinfo_15" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_24_product_template"/>
            <field name="name" ref="base.res_partner_2"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">876</field>
        </record>

        <record id="product_supplierinfo_16" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_25_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">8</field>
            <field name="min_qty">1</field>
            <field name="price">2870</field>
        </record>

        <record id="product_supplierinfo_17" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_02_product_template"/>
            <field name="name" ref="base.res_partner_2"/>
            <field name="delay">4</field>
            <field name="min_qty">1</field>
            <field name="price">390</field>
        </record>

        <record id="product_supplierinfo_18" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_01_product_template"/>
            <field name="name" ref="base.res_partner_3"/>
            <field name="delay">2</field>
            <field name="min_qty">12</field>
            <field name="price">90</field>
        </record>

        <record id="product_supplierinfo_19" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_01_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">4</field>
            <field name="min_qty">1</field>
            <field name="price">66</field>
        </record>

        <record id="product_supplierinfo_20" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_02_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">5</field>
            <field name="min_qty">1</field>
            <field name="price">35</field>
        </record>

        <record id="product_supplierinfo_21" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_01_product_template"/>
            <field name="name" ref="base.res_partner_12"/>
            <field name="delay">7</field>
            <field name="min_qty">1</field>
            <field name="price">55</field>
        </record>

        <record id="product_supplierinfo_22" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_9_product_template"/>
            <field name="name" ref="base.res_partner_12"/>
            <field name="delay">4</field>
            <field name="min_qty">0</field>
            <field name="price">10</field>
        </record>

        <record id="product_supplierinfo_23" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_27_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">10</field>
            <field name="min_qty">0</field>
            <field name="price">3300</field>
        </record>

        <record id="product_supplierinfo_24" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_12_product_template"/>
            <field name="name" ref="base.res_partner_1"/>
            <field name="delay">3</field>
            <field name="min_qty">0</field>
            <field name="price">12.50</field>
        </record>

        <record id="product_supplierinfo_25" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_12_product_template"/>
            <field name="name" ref="base.res_partner_4"/>
            <field name="delay">2</field>
            <field name="min_qty">0</field>
            <field name="price">13.50</field>
        </record>

        <record forcecreate="True" id="property_product_pricelist_demo" model="ir.property">
            <field name="name">property_product_pricelist</field>
            <field name="fields_id" search="[('model','=','res.partner'),('name','=','property_product_pricelist')]"/>
            <field name="value" eval="'product.pricelist,'+str(ref('list0'))"/>
            <field name="res_id" eval="'res.partner,'+str(ref('base.partner_demo'))"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

    </data>
</odoo>

```

## File: models\decimal_precision.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, tools, _
from odoo.exceptions import ValidationError


class DecimalPrecision(models.Model):
    _inherit = 'decimal.precision'

    @api.constrains('digits')
    def _check_main_currency_rounding(self):
        if any(precision.name == 'Account' and
                tools.float_compare(self.env.company.currency_id.rounding, 10 ** - precision.digits, precision_digits=6) == -1
                for precision in self):
            raise ValidationError(_("You cannot define the decimal precision of 'Account' as greater than the rounding factor of the company's main currency"))
        return True

    @api.onchange('digits')
    def _onchange_digits(self):
        if self.name != "Product Unit of Measure":  # precision_get() relies on this name
            return
        # We are changing the precision of UOM fields; check whether the
        # precision is equal or higher than existing units of measure.
        rounding = 1.0 / 10.0**self.digits
        dangerous_uom = self.env['uom.uom'].search([('rounding', '<', rounding)])
        if dangerous_uom:
            uom_descriptions = [
                " - %s (id=%s, precision=%s)" % (uom.name, uom.id, uom.rounding)
                for uom in dangerous_uom
            ]
            return {'warning': {
                'title': _('Warning!'),
                'message': _(
                    "You are setting a Decimal Accuracy less precise than the UOMs:\n"
                    "%s\n"
                    "This may cause inconsistencies in computations.\n"
                    "Please increase the rounding of those units of measure, or the digits of this Decimal Accuracy."
                 ) % ('\n'.join(uom_descriptions)),
            }}

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError
from odoo.osv import expression


from odoo.tools import float_compare

_logger = logging.getLogger(__name__)



class ProductCategory(models.Model):
    _name = "product.category"
    _description = "Product Category"
    _parent_name = "parent_id"
    _parent_store = True
    _rec_name = 'complete_name'
    _order = 'complete_name'

    name = fields.Char('Name', index=True, required=True)
    complete_name = fields.Char(
        'Complete Name', compute='_compute_complete_name',
        store=True)
    parent_id = fields.Many2one('product.category', 'Parent Category', index=True, ondelete='cascade')
    parent_path = fields.Char(index=True)
    child_id = fields.One2many('product.category', 'parent_id', 'Child Categories')
    product_count = fields.Integer(
        '# Products', compute='_compute_product_count',
        help="The number of products under this category (Does not consider the children categories)")

    @api.depends('name', 'parent_id.complete_name')
    def _compute_complete_name(self):
        for category in self:
            if category.parent_id:
                category.complete_name = '%s / %s' % (category.parent_id.complete_name, category.name)
            else:
                category.complete_name = category.name

    def _compute_product_count(self):
        read_group_res = self.env['product.template'].read_group([('categ_id', 'child_of', self.ids)], ['categ_id'], ['categ_id'])
        group_data = dict((data['categ_id'][0], data['categ_id_count']) for data in read_group_res)
        for categ in self:
            product_count = 0
            for sub_categ_id in categ.search([('id', 'child_of', categ.ids)]).ids:
                product_count += group_data.get(sub_categ_id, 0)
            categ.product_count = product_count

    @api.constrains('parent_id')
    def _check_category_recursion(self):
        if not self._check_recursion():
            raise ValidationError(_('You cannot create recursive categories.'))
        return True

    @api.model
    def name_create(self, name):
        return self.create({'name': name}).name_get()[0]


class ProductProduct(models.Model):
    _name = "product.product"
    _description = "Product"
    _inherits = {'product.template': 'product_tmpl_id'}
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'default_code, name, id'

    # price: total price, context dependent (partner, pricelist, quantity)
    price = fields.Float(
        'Price', compute='_compute_product_price',
        digits='Product Price', inverse='_set_product_price')
    # price_extra: catalog extra value only, sum of variant extra attributes
    price_extra = fields.Float(
        'Variant Price Extra', compute='_compute_product_price_extra',
        digits='Product Price',
        help="This is the sum of the extra price of all attributes")
    # lst_price: catalog value + extra, context dependent (uom)
    lst_price = fields.Float(
        'Public Price', compute='_compute_product_lst_price',
        digits='Product Price', inverse='_set_product_lst_price',
        help="The sale price is managed from the product template. Click on the 'Configure Variants' button to set the extra attribute prices.")

    default_code = fields.Char('Internal Reference', index=True)
    code = fields.Char('Reference', compute='_compute_product_code')
    partner_ref = fields.Char('Customer Ref', compute='_compute_partner_ref')

    active = fields.Boolean(
        'Active', default=True,
        help="If unchecked, it will allow you to hide the product without removing it.")
    product_tmpl_id = fields.Many2one(
        'product.template', 'Product Template',
        auto_join=True, index=True, ondelete="cascade", required=True)
    barcode = fields.Char(
        'Barcode', copy=False,
        help="International Article Number used for product identification.")
    product_template_attribute_value_ids = fields.Many2many('product.template.attribute.value', relation='product_variant_combination', string="Attribute Values", ondelete='restrict')
    combination_indices = fields.Char(compute='_compute_combination_indices', store=True, index=True)
    is_product_variant = fields.Boolean(compute='_compute_is_product_variant')

    standard_price = fields.Float(
        'Cost', company_dependent=True,
        digits='Product Price',
        groups="base.group_user",
        help="""In Standard Price & AVCO: value of the product (automatically computed in AVCO).
        In FIFO: value of the last unit that left the stock (automatically computed).
        Used to value the product when the purchase cost is not known (e.g. inventory adjustment).
        Used to compute margins on sale orders.""")
    volume = fields.Float('Volume', digits='Volume')
    weight = fields.Float('Weight', digits='Stock Weight')

    pricelist_item_count = fields.Integer("Number of price rules", compute="_compute_variant_item_count")

    packaging_ids = fields.One2many(
        'product.packaging', 'product_id', 'Product Packages',
        help="Gives the different ways to package the same product.")

    # all image fields are base64 encoded and PIL-supported

    # all image_variant fields are technical and should not be displayed to the user
    image_variant_1920 = fields.Image("Variant Image", max_width=1920, max_height=1920)

    # resized fields stored (as attachment) for performance
    image_variant_1024 = fields.Image("Variant Image 1024", related="image_variant_1920", max_width=1024, max_height=1024, store=True)
    image_variant_512 = fields.Image("Variant Image 512", related="image_variant_1920", max_width=512, max_height=512, store=True)
    image_variant_256 = fields.Image("Variant Image 256", related="image_variant_1920", max_width=256, max_height=256, store=True)
    image_variant_128 = fields.Image("Variant Image 128", related="image_variant_1920", max_width=128, max_height=128, store=True)
    can_image_variant_1024_be_zoomed = fields.Boolean("Can Variant Image 1024 be zoomed", compute='_compute_can_image_variant_1024_be_zoomed', store=True)

    # Computed fields that are used to create a fallback to the template if
    # necessary, it's recommended to display those fields to the user.
    image_1920 = fields.Image("Image", compute='_compute_image_1920', inverse='_set_image_1920')
    image_1024 = fields.Image("Image 1024", compute='_compute_image_1024')
    image_512 = fields.Image("Image 512", compute='_compute_image_512')
    image_256 = fields.Image("Image 256", compute='_compute_image_256')
    image_128 = fields.Image("Image 128", compute='_compute_image_128')
    can_image_1024_be_zoomed = fields.Boolean("Can Image 1024 be zoomed", compute='_compute_can_image_1024_be_zoomed')

    @api.depends('image_variant_1920', 'image_variant_1024')
    def _compute_can_image_variant_1024_be_zoomed(self):
        for record in self:
            record.can_image_variant_1024_be_zoomed = record.image_variant_1920 and tools.is_image_size_above(record.image_variant_1920, record.image_variant_1024)

    def _compute_image_1920(self):
        """Get the image from the template if no image is set on the variant."""
        for record in self:
            record.image_1920 = record.image_variant_1920 or record.product_tmpl_id.image_1920

    def _set_image_1920(self):
        for record in self:
            if (
                # We are trying to remove an image even though it is already
                # not set, remove it from the template instead.
                not record.image_1920 and not record.image_variant_1920 or
                # We are trying to add an image, but the template image is
                # not set, write on the template instead.
                record.image_1920 and not record.product_tmpl_id.image_1920 or
                # There is only one variant, always write on the template.
                self.search_count([
                    ('product_tmpl_id', '=', record.product_tmpl_id.id),
                    ('active', '=', True),
                ]) <= 1
            ):
                record.image_variant_1920 = False
                record.product_tmpl_id.image_1920 = record.image_1920
            else:
                record.image_variant_1920 = record.image_1920

    def _compute_image_1024(self):
        """Get the image from the template if no image is set on the variant."""
        for record in self:
            record.image_1024 = record.image_variant_1024 or record.product_tmpl_id.image_1024

    def _compute_image_512(self):
        """Get the image from the template if no image is set on the variant."""
        for record in self:
            record.image_512 = record.image_variant_512 or record.product_tmpl_id.image_512

    def _compute_image_256(self):
        """Get the image from the template if no image is set on the variant."""
        for record in self:
            record.image_256 = record.image_variant_256 or record.product_tmpl_id.image_256

    def _compute_image_128(self):
        """Get the image from the template if no image is set on the variant."""
        for record in self:
            record.image_128 = record.image_variant_128 or record.product_tmpl_id.image_128

    def _compute_can_image_1024_be_zoomed(self):
        """Get the image from the template if no image is set on the variant."""
        for record in self:
            record.can_image_1024_be_zoomed = record.can_image_variant_1024_be_zoomed if record.image_variant_1920 else record.product_tmpl_id.can_image_1024_be_zoomed

    def init(self):
        """Ensure there is at most one active variant for each combination.

        There could be no variant for a combination if using dynamic attributes.
        """
        self.env.cr.execute("CREATE UNIQUE INDEX IF NOT EXISTS product_product_combination_unique ON %s (product_tmpl_id, combination_indices) WHERE active is true"
            % self._table)

    _sql_constraints = [
        ('barcode_uniq', 'unique(barcode)', "A barcode can only be assigned to one product !"),
    ]

    def _get_invoice_policy(self):
        return False

    @api.depends('product_template_attribute_value_ids')
    def _compute_combination_indices(self):
        for product in self:
            product.combination_indices = product.product_template_attribute_value_ids._ids2str()

    def _compute_is_product_variant(self):
        for product in self:
            product.is_product_variant = True

    @api.depends_context('pricelist', 'partner', 'quantity', 'uom', 'date', 'no_variant_attributes_price_extra')
    def _compute_product_price(self):
        prices = {}
        pricelist_id_or_name = self._context.get('pricelist')
        if pricelist_id_or_name:
            pricelist = None
            partner = self.env.context.get('partner', False)
            quantity = self.env.context.get('quantity', 1.0)

            # Support context pricelists specified as list, display_name or ID for compatibility
            if isinstance(pricelist_id_or_name, list):
                pricelist_id_or_name = pricelist_id_or_name[0]
            if isinstance(pricelist_id_or_name, str):
                pricelist_name_search = self.env['product.pricelist'].name_search(pricelist_id_or_name, operator='=', limit=1)
                if pricelist_name_search:
                    pricelist = self.env['product.pricelist'].browse([pricelist_name_search[0][0]])
            elif isinstance(pricelist_id_or_name, int):
                pricelist = self.env['product.pricelist'].browse(pricelist_id_or_name)

            if pricelist:
                quantities = [quantity] * len(self)
                partners = [partner] * len(self)
                prices = pricelist.get_products_price(self, quantities, partners)

        for product in self:
            product.price = prices.get(product.id, 0.0)

    def _set_product_price(self):
        for product in self:
            if self._context.get('uom'):
                value = self.env['uom.uom'].browse(self._context['uom'])._compute_price(product.price, product.uom_id)
            else:
                value = product.price
            value -= product.price_extra
            product.write({'list_price': value})

    def _set_product_lst_price(self):
        for product in self:
            if self._context.get('uom'):
                value = self.env['uom.uom'].browse(self._context['uom'])._compute_price(product.lst_price, product.uom_id)
            else:
                value = product.lst_price
            value -= product.price_extra
            product.write({'list_price': value})

    def _compute_product_price_extra(self):
        for product in self:
            product.price_extra = sum(product.product_template_attribute_value_ids.mapped('price_extra'))

    @api.depends('list_price', 'price_extra')
    @api.depends_context('uom')
    def _compute_product_lst_price(self):
        to_uom = None
        if 'uom' in self._context:
            to_uom = self.env['uom.uom'].browse(self._context['uom'])

        for product in self:
            if to_uom:
                list_price = product.uom_id._compute_price(product.list_price, to_uom)
            else:
                list_price = product.list_price
            product.lst_price = list_price + product.price_extra

    @api.depends_context('partner_id')
    def _compute_product_code(self):
        for product in self:
            for supplier_info in product.seller_ids:
                if supplier_info.name.id == product._context.get('partner_id'):
                    product.code = supplier_info.product_code or product.default_code
                    break
            else:
                product.code = product.default_code

    @api.depends_context('partner_id')
    def _compute_partner_ref(self):
        for product in self:
            for supplier_info in product.seller_ids:
                if supplier_info.name.id == product._context.get('partner_id'):
                    product_name = supplier_info.product_name or product.default_code or product.name
                    product.partner_ref = '%s%s' % (product.code and '[%s] ' % product.code or '', product_name)
                    break
            else:
                product.partner_ref = product.display_name

    def _compute_variant_item_count(self):
        for product in self:
            domain = ['|',
                '&', ('product_tmpl_id', '=', product.product_tmpl_id.id), ('applied_on', '=', '1_product'),
                '&', ('product_id', '=', product.id), ('applied_on', '=', '0_product_variant')]
            product.pricelist_item_count = self.env['product.pricelist.item'].search_count(domain)

    @api.onchange('uom_id')
    def _onchange_uom_id(self):
        if self.uom_id:
            self.uom_po_id = self.uom_id.id

    @api.onchange('uom_po_id')
    def _onchange_uom(self):
        if self.uom_id and self.uom_po_id and self.uom_id.category_id != self.uom_po_id.category_id:
            self.uom_po_id = self.uom_id

    @api.model_create_multi
    def create(self, vals_list):
        products = super(ProductProduct, self.with_context(create_product_product=True)).create(vals_list)
        # `_get_variant_id_for_combination` depends on existing variants
        self.clear_caches()
        return products

    def write(self, values):
        res = super(ProductProduct, self).write(values)
        if 'product_template_attribute_value_ids' in values:
            # `_get_variant_id_for_combination` depends on `product_template_attribute_value_ids`
            self.clear_caches()
        elif 'active' in values:
            # `_get_first_possible_variant_id` depends on variants active state
            self.clear_caches()
        return res

    def unlink(self):
        unlink_products = self.env['product.product']
        unlink_templates = self.env['product.template']
        for product in self:
            # If there is an image set on the variant and no image set on the
            # template, move the image to the template.
            if product.image_variant_1920 and not product.product_tmpl_id.image_1920:
                product.product_tmpl_id.image_1920 = product.image_variant_1920
            # Check if product still exists, in case it has been unlinked by unlinking its template
            if not product.exists():
                continue
            # Check if the product is last product of this template...
            other_products = self.search([('product_tmpl_id', '=', product.product_tmpl_id.id), ('id', '!=', product.id)])
            # ... and do not delete product template if it's configured to be created "on demand"
            if not other_products and not product.product_tmpl_id.has_dynamic_attributes():
                unlink_templates |= product.product_tmpl_id
            unlink_products |= product
        res = super(ProductProduct, unlink_products).unlink()
        # delete templates after calling super, as deleting template could lead to deleting
        # products due to ondelete='cascade'
        unlink_templates.unlink()
        # `_get_variant_id_for_combination` depends on existing variants
        self.clear_caches()
        return res

    def _filter_to_unlink(self, check_access=True):
        return self

    def _unlink_or_archive(self, check_access=True):
        """Unlink or archive products.
        Try in batch as much as possible because it is much faster.
        Use dichotomy when an exception occurs.
        """

        # Avoid access errors in case the products is shared amongst companies
        # but the underlying objects are not. If unlink fails because of an
        # AccessError (e.g. while recomputing fields), the 'write' call will
        # fail as well for the same reason since the field has been set to
        # recompute.
        if check_access:
            self.check_access_rights('unlink')
            self.check_access_rule('unlink')
            self.check_access_rights('write')
            self.check_access_rule('write')
            self = self.sudo()
            to_unlink = self._filter_to_unlink()
            to_archive = self - to_unlink
            to_archive.write({'active': False})
            self = to_unlink

        try:
            with self.env.cr.savepoint(), tools.mute_logger('odoo.sql_db'):
                self.unlink()
        except Exception:
            # We catch all kind of exceptions to be sure that the operation
            # doesn't fail.
            if len(self) > 1:
                self[:len(self) // 2]._unlink_or_archive(check_access=False)
                self[len(self) // 2:]._unlink_or_archive(check_access=False)
            else:
                if self.active:
                    # Note: this can still fail if something is preventing
                    # from archiving.
                    # This is the case from existing stock reordering rules.
                    self.write({'active': False})

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        """Variants are generated depending on the configuration of attributes
        and values on the template, so copying them does not make sense.

        For convenience the template is copied instead and its first variant is
        returned.
        """
        # copy variant is disabled in https://github.com/odoo/odoo/pull/38303
        # this returns the first possible combination of variant to make it
        # works for now, need to be fixed to return product_variant_id if it's
        # possible in the future
        template = self.product_tmpl_id.copy(default=default)
        return template.product_variant_id or template._create_first_product_variant()

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        # TDE FIXME: strange
        if self._context.get('search_default_categ_id'):
            args.append((('categ_id', 'child_of', self._context['search_default_categ_id'])))
        return super(ProductProduct, self)._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    @api.depends_context('display_default_code', 'seller_id')
    def _compute_display_name(self):
        # `display_name` is calling `name_get()`` which is overidden on product
        # to depend on `display_default_code` and `seller_id`
        return super()._compute_display_name()

    def name_get(self):
        # TDE: this could be cleaned a bit I think

        def _name_get(d):
            name = d.get('name', '')
            code = self._context.get('display_default_code', True) and d.get('default_code', False) or False
            if code:
                name = '[%s] %s' % (code,name)
            return (d['id'], name)

        partner_id = self._context.get('partner_id')
        if partner_id:
            partner_ids = [partner_id, self.env['res.partner'].browse(partner_id).commercial_partner_id.id]
        else:
            partner_ids = []
        company_id = self.env.context.get('company_id')

        # all user don't have access to seller and partner
        # check access and use superuser
        self.check_access_rights("read")
        self.check_access_rule("read")

        result = []

        # Prefetch the fields used by the `name_get`, so `browse` doesn't fetch other fields
        # Use `load=False` to not call `name_get` for the `product_tmpl_id`
        self.sudo().read(['name', 'default_code', 'product_tmpl_id'], load=False)

        product_template_ids = self.sudo().mapped('product_tmpl_id').ids

        if partner_ids:
            # name_get() of product with different quantities of the same supplier will be ambiguous, so
            # we can pass a supplier_info in the context if we already know which one we want
            supplier_info = self.env.context.get('supplier_info', False)
            if not supplier_info:
                supplier_info = self.env['product.supplierinfo'].sudo().search([
                    ('product_tmpl_id', 'in', product_template_ids),
                    ('name', 'in', partner_ids),
                ])
            # Prefetch the fields used by the `name_get`, so `browse` doesn't fetch other fields
            # Use `load=False` to not call `name_get` for the `product_tmpl_id` and `product_id`
            supplier_info.sudo().read(['product_tmpl_id', 'product_id', 'product_name', 'product_code'], load=False)
            supplier_info_by_template = {}
            for r in supplier_info:
                supplier_info_by_template.setdefault(r.product_tmpl_id, []).append(r)
        for product in self.sudo():
            variant = product.product_template_attribute_value_ids._get_combination_name()

            name = variant and "%s (%s)" % (product.name, variant) or product.name
            sellers = self.env['product.supplierinfo'].sudo().browse(self.env.context.get('seller_id')) or []
            if not sellers and partner_ids:
                product_supplier_info = supplier_info_by_template.get(product.product_tmpl_id, [])
                sellers = [x for x in product_supplier_info if x.product_id and x.product_id == product]
                if not sellers:
                    sellers = [x for x in product_supplier_info if not x.product_id]
                # Filter out sellers based on the company. This is done afterwards for a better
                # code readability. At this point, only a few sellers should remain, so it should
                # not be a performance issue.
                if company_id:
                    sellers = [x for x in sellers if x.company_id.id in [company_id, False]]
            if sellers:
                for s in sellers:
                    seller_variant = s.product_name and (
                        variant and "%s (%s)" % (s.product_name, variant) or s.product_name
                        ) or False
                    mydict = {
                              'id': product.id,
                              'name': seller_variant or name,
                              'default_code': s.product_code or product.default_code,
                              }
                    temp = _name_get(mydict)
                    if temp not in result:
                        result.append(temp)
            else:
                mydict = {
                          'id': product.id,
                          'name': name,
                          'default_code': product.default_code,
                          }
                result.append(_name_get(mydict))
        return result

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        if not args:
            args = []
        if name:
            positive_operators = ['=', 'ilike', '=ilike', 'like', '=like']
            product_ids = []
            if operator in positive_operators:
                product_ids = self._search([('default_code', '=', name)] + args, limit=limit, access_rights_uid=name_get_uid)
                if not product_ids:
                    product_ids = self._search([('barcode', '=', name)] + args, limit=limit, access_rights_uid=name_get_uid)
            if not product_ids and operator not in expression.NEGATIVE_TERM_OPERATORS:
                # Do not merge the 2 next lines into one single search, SQL search performance would be abysmal
                # on a database with thousands of matching products, due to the huge merge+unique needed for the
                # OR operator (and given the fact that the 'name' lookup results come from the ir.translation table
                # Performing a quick memory merge of ids in Python will give much better performance
                product_ids = self._search(args + [('default_code', operator, name)], limit=limit)
                if not limit or len(product_ids) < limit:
                    # we may underrun the limit because of dupes in the results, that's fine
                    limit2 = (limit - len(product_ids)) if limit else False
                    product2_ids = self._search(args + [('name', operator, name), ('id', 'not in', product_ids)], limit=limit2, access_rights_uid=name_get_uid)
                    product_ids.extend(product2_ids)
            elif not product_ids and operator in expression.NEGATIVE_TERM_OPERATORS:
                domain = expression.OR([
                    ['&', ('default_code', operator, name), ('name', operator, name)],
                    ['&', ('default_code', '=', False), ('name', operator, name)],
                ])
                domain = expression.AND([args, domain])
                product_ids = self._search(domain, limit=limit, access_rights_uid=name_get_uid)
            if not product_ids and operator in positive_operators:
                ptrn = re.compile('(\[(.*?)\])')
                res = ptrn.search(name)
                if res:
                    product_ids = self._search([('default_code', '=', res.group(2))] + args, limit=limit, access_rights_uid=name_get_uid)
            # still no results, partner in context: search on supplier info as last hope to find something
            if not product_ids and self._context.get('partner_id'):
                suppliers_ids = self.env['product.supplierinfo']._search([
                    ('name', '=', self._context.get('partner_id')),
                    '|',
                    ('product_code', operator, name),
                    ('product_name', operator, name)], access_rights_uid=name_get_uid)
                if suppliers_ids:
                    product_ids = self._search([('product_tmpl_id.seller_ids', 'in', suppliers_ids)], limit=limit, access_rights_uid=name_get_uid)
        else:
            product_ids = self._search(args, limit=limit, access_rights_uid=name_get_uid)
        return models.lazy_name_get(self.browse(product_ids).with_user(name_get_uid))

    @api.model
    def view_header_get(self, view_id, view_type):
        res = super(ProductProduct, self).view_header_get(view_id, view_type)
        if self._context.get('categ_id'):
            return _('Products: ') + self.env['product.category'].browse(self._context['categ_id']).name
        return res

    def open_pricelist_rules(self):
        self.ensure_one()
        domain = ['|',
            '&', ('product_tmpl_id', '=', self.product_tmpl_id.id), ('applied_on', '=', '1_product'),
            '&', ('product_id', '=', self.id), ('applied_on', '=', '0_product_variant')]
        return {
            'name': _('Price Rules'),
            'view_mode': 'tree,form',
            'views': [(self.env.ref('product.product_pricelist_item_tree_view_from_product').id, 'tree'), (False, 'form')],
            'res_model': 'product.pricelist.item',
            'type': 'ir.actions.act_window',
            'target': 'current',
            'domain': domain,
            'context': {
                'default_product_id': self.id,
                'default_applied_on': '0_product_variant',
            }
        }

    def open_product_template(self):
        """ Utility method used to add an "Open Template" button in product views """
        self.ensure_one()
        return {'type': 'ir.actions.act_window',
                'res_model': 'product.template',
                'view_mode': 'form',
                'res_id': self.product_tmpl_id.id,
                'target': 'new'}

    def _prepare_sellers(self, params):
        # This search is made to avoid retrieving seller_ids from the cache.
        return self.env['product.supplierinfo']\
                   .search([('product_tmpl_id', '=', self.product_tmpl_id.id)])\
                   .filtered(lambda r: r.name.active)\
                   .sorted(lambda s: (s.sequence, -s.min_qty, s.price, s.id))

    def _select_seller(self, partner_id=False, quantity=0.0, date=None, uom_id=False, params=False):
        self.ensure_one()
        if date is None:
            date = fields.Date.context_today(self)
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')

        res = self.env['product.supplierinfo']
        sellers = self._prepare_sellers(params)
        if self.env.context.get('force_company'):
            sellers = sellers.filtered(lambda s: not s.company_id or s.company_id.id == self.env.context['force_company'])
        for seller in sellers:
            # Set quantity in UoM of seller
            quantity_uom_seller = quantity
            if quantity_uom_seller and uom_id and uom_id != seller.product_uom:
                quantity_uom_seller = uom_id._compute_quantity(quantity_uom_seller, seller.product_uom)

            if seller.date_start and seller.date_start > date:
                continue
            if seller.date_end and seller.date_end < date:
                continue
            if partner_id and seller.name not in [partner_id, partner_id.parent_id]:
                continue
            if float_compare(quantity_uom_seller, seller.min_qty, precision_digits=precision) == -1:
                continue
            if seller.product_id and seller.product_id != self:
                continue
            if not res or res.name == seller.name:
                res |= seller
        return res.sorted('price')[:1]

    def price_compute(self, price_type, uom=False, currency=False, company=False):
        # TDE FIXME: delegate to template or not ? fields are reencoded here ...
        # compatibility about context keys used a bit everywhere in the code
        if not uom and self._context.get('uom'):
            uom = self.env['uom.uom'].browse(self._context['uom'])
        if not currency and self._context.get('currency'):
            currency = self.env['res.currency'].browse(self._context['currency'])

        products = self
        if price_type == 'standard_price':
            # standard_price field can only be seen by users in base.group_user
            # Thus, in order to compute the sale price from the cost for users not in this group
            # We fetch the standard price as the superuser
            products = self.with_context(force_company=company and company.id or self._context.get('force_company', self.env.company.id)).sudo()

        prices = dict.fromkeys(self.ids, 0.0)
        for product in products:
            prices[product.id] = product[price_type] or 0.0
            if price_type == 'list_price':
                prices[product.id] += product.price_extra
                # we need to add the price from the attributes that do not generate variants
                # (see field product.attribute create_variant)
                if self._context.get('no_variant_attributes_price_extra'):
                    # we have a list of price_extra that comes from the attribute values, we need to sum all that
                    prices[product.id] += sum(self._context.get('no_variant_attributes_price_extra'))

            if uom:
                prices[product.id] = product.uom_id._compute_price(prices[product.id], uom)

            # Convert from current user company currency to asked one
            # This is right cause a field cannot be in more than one currency
            if currency:
                prices[product.id] = product.currency_id._convert(
                    prices[product.id], currency, product.company_id, fields.Date.today())

        return prices

    @api.model
    def get_empty_list_help(self, help):
        self = self.with_context(
            empty_list_help_document_name=_("product"),
        )
        return super(ProductProduct, self).get_empty_list_help(help)

    def get_product_multiline_description_sale(self):
        """ Compute a multiline description of this product, in the context of sales
                (do not use for purchases or other display reasons that don't intend to use "description_sale").
            It will often be used as the default description of a sale order line referencing this product.
        """
        name = self.display_name
        if self.description_sale:
            name += '\n' + self.description_sale

        return name

    def _is_variant_possible(self, parent_combination=None):
        """Return whether the variant is possible based on its own combination,
        and optionally a parent combination.

        See `_is_combination_possible` for more information.

        :param parent_combination: combination from which `self` is an
            optional or accessory product.
        :type parent_combination: recordset `product.template.attribute.value`

        :return: ẁhether the variant is possible based on its own combination
        :rtype: bool
        """
        self.ensure_one()
        return self.product_tmpl_id._is_combination_possible(self.product_template_attribute_value_ids, parent_combination=parent_combination, ignore_no_variant=True)

    def toggle_active(self):
        """ Archiving related product.template if there is not any more active product.product
        (and vice versa, unarchiving the related product template if there is now an active product.product) """
        result = super().toggle_active()
        # We deactivate product templates which are active with no active variants.
        tmpl_to_deactivate = self.filtered(lambda product: (product.product_tmpl_id.active
                                                            and not product.product_tmpl_id.product_variant_ids)).mapped('product_tmpl_id')
        # We activate product templates which are inactive with active variants.
        tmpl_to_activate = self.filtered(lambda product: (not product.product_tmpl_id.active
                                                          and product.product_tmpl_id.product_variant_ids)).mapped('product_tmpl_id')
        (tmpl_to_deactivate + tmpl_to_activate).toggle_active()
        return result


class ProductPackaging(models.Model):
    _name = "product.packaging"
    _description = "Product Packaging"
    _order = 'sequence'
    _check_company_auto = True

    name = fields.Char('Package Type', required=True)
    sequence = fields.Integer('Sequence', default=1, help="The first in the sequence is the default one.")
    product_id = fields.Many2one('product.product', string='Product', check_company=True)
    qty = fields.Float('Contained Quantity', digits='Product Unit of Measure', help="Quantity of products contained in the packaging.")
    barcode = fields.Char('Barcode', copy=False, help="Barcode used for packaging identification. Scan this packaging barcode from a transfer in the Barcode app to move all the contained units")
    product_uom_id = fields.Many2one('uom.uom', related='product_id.uom_id', readonly=True)
    company_id = fields.Many2one('res.company', 'Company', index=True)


class SupplierInfo(models.Model):
    _name = "product.supplierinfo"
    _description = "Supplier Pricelist"
    _order = 'sequence, min_qty desc, price'

    name = fields.Many2one(
        'res.partner', 'Vendor',
        ondelete='cascade', required=True,
        help="Vendor of this product", check_company=True)
    product_name = fields.Char(
        'Vendor Product Name',
        help="This vendor's product name will be used when printing a request for quotation. Keep empty to use the internal one.")
    product_code = fields.Char(
        'Vendor Product Code',
        help="This vendor's product code will be used when printing a request for quotation. Keep empty to use the internal one.")
    sequence = fields.Integer(
        'Sequence', default=1, help="Assigns the priority to the list of product vendor.")
    product_uom = fields.Many2one(
        'uom.uom', 'Unit of Measure',
        related='product_tmpl_id.uom_po_id',
        help="This comes from the product form.")
    min_qty = fields.Float(
        'Quantity', default=0.0, required=True,
        help="The quantity to purchase from this vendor to benefit from the price, expressed in the vendor Product Unit of Measure if not any, in the default unit of measure of the product otherwise.")
    price = fields.Float(
        'Price', default=0.0, digits='Product Price',
        required=True, help="The price to purchase a product")
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda self: self.env.company.id, index=1)
    currency_id = fields.Many2one(
        'res.currency', 'Currency',
        default=lambda self: self.env.company.currency_id.id,
        required=True)
    date_start = fields.Date('Start Date', help="Start date for this vendor price")
    date_end = fields.Date('End Date', help="End date for this vendor price")
    product_id = fields.Many2one(
        'product.product', 'Product Variant', check_company=True,
        help="If not set, the vendor price will apply to all variants of this product.")
    product_tmpl_id = fields.Many2one(
        'product.template', 'Product Template', check_company=True,
        index=True, ondelete='cascade')
    product_variant_count = fields.Integer('Variant Count', related='product_tmpl_id.product_variant_count', readonly=False)
    delay = fields.Integer(
        'Delivery Lead Time', default=1, required=True,
        help="Lead time in days between the confirmation of the purchase order and the receipt of the products in your warehouse. Used by the scheduler for automatic computation of the purchase order planning.")

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Vendor Pricelists'),
            'template': '/product/static/xls/product_supplierinfo.xls'
        }]

```

## File: models\product_attribute.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError, ValidationError
from odoo.osv import expression


class ProductAttribute(models.Model):
    _name = "product.attribute"
    _description = "Product Attribute"
    # if you change this _order, keep it in sync with the method
    # `_sort_key_attribute_value` in `product.template`
    _order = 'sequence, id'

    name = fields.Char('Attribute', required=True, translate=True)
    value_ids = fields.One2many('product.attribute.value', 'attribute_id', 'Values', copy=True)
    sequence = fields.Integer('Sequence', help="Determine the display order", index=True)
    attribute_line_ids = fields.One2many('product.template.attribute.line', 'attribute_id', 'Lines')
    create_variant = fields.Selection([
        ('always', 'Instantly'),
        ('dynamic', 'Dynamically'),
        ('no_variant', 'Never')],
        default='always',
        string="Variants Creation Mode",
        help="""- Instantly: All possible variants are created as soon as the attribute and its values are added to a product.
        - Dynamically: Each variant is created only when its corresponding attributes and values are added to a sales order.
        - Never: Variants are never created for the attribute.
        Note: the variants creation mode cannot be changed once the attribute is used on at least one product.""",
        required=True)
    is_used_on_products = fields.Boolean('Used on Products', compute='_compute_is_used_on_products')
    product_tmpl_ids = fields.Many2many('product.template', string="Related Products", compute='_compute_products', store=True)

    @api.depends('product_tmpl_ids')
    def _compute_is_used_on_products(self):
        for pa in self:
            pa.is_used_on_products = bool(pa.product_tmpl_ids)

    @api.depends('attribute_line_ids.active', 'attribute_line_ids.product_tmpl_id')
    def _compute_products(self):
        for pa in self:
            pa.with_context(active_test=False).product_tmpl_ids = pa.attribute_line_ids.product_tmpl_id
        # avoid multi-company records in cache (that were an issue for subsequent read)
        self.flush(fnames=['product_tmpl_ids'], records=self)
        self.invalidate_cache(fnames=['product_tmpl_ids'], ids=self.ids)

    def _without_no_variant_attributes(self):
        return self.filtered(lambda pa: pa.create_variant != 'no_variant')

    def write(self, vals):
        """Override to make sure attribute type can't be changed if it's used on
        a product template.

        This is important to prevent because changing the type would make
        existing combinations invalid without recomputing them, and recomputing
        them might take too long and we don't want to change products without
        the user knowing about it."""
        if 'create_variant' in vals:
            for pa in self:
                if vals['create_variant'] != pa.create_variant and pa.is_used_on_products:
                    raise UserError(
                        _("You cannot change the Variants Creation Mode of the attribute %s because it is used on the following products:\n%s") %
                        (pa.display_name, ", ".join(pa.product_tmpl_ids.mapped('display_name')))
                    )
        invalidate_cache = 'sequence' in vals and any(record.sequence != vals['sequence'] for record in self)
        res = super(ProductAttribute, self).write(vals)
        if invalidate_cache:
            # prefetched o2m have to be resequenced
            # (eg. product.template: attribute_line_ids)
            self.flush()
            self.invalidate_cache()
        return res

    def unlink(self):
        for pa in self:
            if pa.is_used_on_products:
                raise UserError(
                    _("You cannot delete the attribute %s because it is used on the following products:\n%s") %
                    (pa.display_name, ", ".join(pa.product_tmpl_ids.mapped('display_name')))
                )
        return super(ProductAttribute, self).unlink()


class ProductAttributeValue(models.Model):
    _name = "product.attribute.value"
    # if you change this _order, keep it in sync with the method
    # `_sort_key_variant` in `product.template'
    _order = 'attribute_id, sequence, id'
    _description = 'Attribute Value'

    name = fields.Char(string='Value', required=True, translate=True)
    sequence = fields.Integer(string='Sequence', help="Determine the display order", index=True)
    attribute_id = fields.Many2one('product.attribute', string="Attribute", ondelete='cascade', required=True, index=True,
        help="The attribute cannot be changed once the value is used on at least one product.")

    pav_attribute_line_ids = fields.Many2many('product.template.attribute.line', string="Lines",
        relation='product_attribute_value_product_template_attribute_line_rel', copy=False)
    is_used_on_products = fields.Boolean('Used on Products', compute='_compute_is_used_on_products')

    _sql_constraints = [
        ('value_company_uniq', 'unique (name, attribute_id)', "You cannot create two values with the same name for the same attribute.")
    ]

    @api.depends('pav_attribute_line_ids')
    def _compute_is_used_on_products(self):
        for pav in self:
            pav.is_used_on_products = bool(pav.pav_attribute_line_ids)

    def name_get(self):
        """Override because in general the name of the value is confusing if it
        is displayed without the name of the corresponding attribute.
        Eg. on product list & kanban views, on BOM form view

        However during variant set up (on the product template form) the name of
        the attribute is already on each line so there is no need to repeat it
        on every value.
        """
        if not self._context.get('show_attribute', True):
            return super(ProductAttributeValue, self).name_get()
        return [(value.id, "%s: %s" % (value.attribute_id.name, value.name)) for value in self]

    def write(self, values):
        if 'attribute_id' in values:
            for pav in self:
                if pav.attribute_id.id != values['attribute_id'] and pav.is_used_on_products:
                    raise UserError(
                        _("You cannot change the attribute of the value %s because it is used on the following products:%s") %
                        (pav.display_name, ", ".join(pav.pav_attribute_line_ids.product_tmpl_id.mapped('display_name')))
                    )

        invalidate_cache = 'sequence' in values and any(record.sequence != values['sequence'] for record in self)
        res = super(ProductAttributeValue, self).write(values)
        if invalidate_cache:
            # prefetched o2m have to be resequenced
            # (eg. product.template.attribute.line: value_ids)
            self.flush()
            self.invalidate_cache()
        return res

    def unlink(self):
        for pav in self:
            if pav.is_used_on_products:
                raise UserError(
                    _("You cannot delete the value %s because it is used on the following products:\n%s") %
                    (pav.display_name, ", ".join(pav.pav_attribute_line_ids.product_tmpl_id.mapped('display_name')))
                )
        return super(ProductAttributeValue, self).unlink()

    def _without_no_variant_attributes(self):
        return self.filtered(lambda pav: pav.attribute_id.create_variant != 'no_variant')


class ProductTemplateAttributeLine(models.Model):
    """Attributes available on product.template with their selected values in a m2m.
    Used as a configuration model to generate the appropriate product.template.attribute.value"""

    _name = "product.template.attribute.line"
    _rec_name = 'attribute_id'
    _description = 'Product Template Attribute Line'
    _order = 'attribute_id, id'

    active = fields.Boolean(default=True)
    product_tmpl_id = fields.Many2one('product.template', string="Product Template", ondelete='cascade', required=True, index=True)
    attribute_id = fields.Many2one('product.attribute', string="Attribute", ondelete='restrict', required=True, index=True)
    value_ids = fields.Many2many('product.attribute.value', string="Values", domain="[('attribute_id', '=', attribute_id)]",
        relation='product_attribute_value_product_template_attribute_line_rel', ondelete='restrict')
    product_template_value_ids = fields.One2many('product.template.attribute.value', 'attribute_line_id', string="Product Attribute Values")

    @api.onchange('attribute_id')
    def _onchange_attribute_id(self):
        self.value_ids = self.value_ids.filtered(lambda pav: pav.attribute_id == self.attribute_id)

    @api.constrains('active', 'value_ids', 'attribute_id')
    def _check_valid_values(self):
        for ptal in self:
            if ptal.active and not ptal.value_ids:
                raise ValidationError(
                    _("The attribute %s must have at least one value for the product %s.") %
                    (ptal.attribute_id.display_name, ptal.product_tmpl_id.display_name)
                )
            for pav in ptal.value_ids:
                if pav.attribute_id != ptal.attribute_id:
                    raise ValidationError(
                        _("On the product %s you cannot associate the value %s with the attribute %s because they do not match.") %
                        (ptal.product_tmpl_id.display_name, pav.display_name, ptal.attribute_id.display_name)
                    )
        return True

    @api.model_create_multi
    def create(self, vals_list):
        """Override to:
        - Activate archived lines having the same configuration (if they exist)
            instead of creating new lines.
        - Set up related values and related variants.

        Reactivating existing lines allows to re-use existing variants when
        possible, keeping their configuration and avoiding duplication.
        """
        create_values = []
        activated_lines = self.env['product.template.attribute.line']
        for value in vals_list:
            vals = dict(value, active=value.get('active', True))
            # While not ideal for peformance, this search has to be done at each
            # step to exclude the lines that might have been activated at a
            # previous step. Since `vals_list` will likely be a small list in
            # all use cases, this is an acceptable trade-off.
            archived_ptal = self.search([
                ('active', '=', False),
                ('product_tmpl_id', '=', vals.pop('product_tmpl_id', 0)),
                ('attribute_id', '=', vals.pop('attribute_id', 0)),
            ], limit=1)
            if archived_ptal:
                # Write given `vals` in addition of `active` to ensure
                # `value_ids` or other fields passed to `create` are saved too,
                # but change the context to avoid updating the values and the
                # variants until all the expected lines are created/updated.
                archived_ptal.with_context(update_product_template_attribute_values=False).write(vals)
                activated_lines += archived_ptal
            else:
                create_values.append(value)
        res = activated_lines + super(ProductTemplateAttributeLine, self).create(create_values)
        res._update_product_template_attribute_values()
        return res

    def write(self, values):
        """Override to:
        - Add constraints to prevent doing changes that are not supported such
            as modifying the template or the attribute of existing lines.
        - Clean up related values and related variants when archiving or when
            updating `value_ids`.
        """
        if 'product_tmpl_id' in values:
            for ptal in self:
                if ptal.product_tmpl_id.id != values['product_tmpl_id']:
                    raise UserError(
                        _("You cannot move the attribute %s from the product %s to the product %s.") %
                        (ptal.attribute_id.display_name, ptal.product_tmpl_id.display_name, values['product_tmpl_id'])
                    )

        if 'attribute_id' in values:
            for ptal in self:
                if ptal.attribute_id.id != values['attribute_id']:
                    raise UserError(
                        _("On the product %s you cannot transform the attribute %s into the attribute %s.") %
                        (ptal.product_tmpl_id.display_name, ptal.attribute_id.display_name, values['attribute_id'])
                    )
        # Remove all values while archiving to make sure the line is clean if it
        # is ever activated again.
        if not values.get('active', True):
            values['value_ids'] = [(5, 0, 0)]
        res = super(ProductTemplateAttributeLine, self).write(values)
        if 'active' in values:
            self.flush()
            self.env['product.template'].invalidate_cache(fnames=['attribute_line_ids'])
        # If coming from `create`, no need to update the values and the variants
        # before all lines are created.
        if self.env.context.get('update_product_template_attribute_values', True):
            self._update_product_template_attribute_values()
        return res

    def unlink(self):
        """Override to:
        - Archive the line if unlink is not possible.
        - Clean up related values and related variants.

        Archiving is typically needed when the line has values that can't be
        deleted because they are referenced elsewhere (on a variant that can't
        be deleted, on a sales order line, ...).
        """
        # Try to remove the values first to remove some potentially blocking
        # references, which typically works:
        # - For single value lines because the values are directly removed from
        #   the variants.
        # - For values that are present on variants that can be deleted.
        self.product_template_value_ids._only_active().unlink()
        # Keep a reference to the related templates before the deletion.
        templates = self.product_tmpl_id
        # Now delete or archive the lines.
        ptal_to_archive = self.env['product.template.attribute.line']
        for ptal in self:
            try:
                with self.env.cr.savepoint(), tools.mute_logger('odoo.sql_db'):
                    super(ProductTemplateAttributeLine, ptal).unlink()
            except Exception:
                # We catch all kind of exceptions to be sure that the operation
                # doesn't fail.
                ptal_to_archive += ptal
        ptal_to_archive.write({'active': False})
        # For archived lines `_update_product_template_attribute_values` is
        # implicitly called during the `write` above, but for products that used
        # unlinked lines `_create_variant_ids` has to be called manually.
        (templates - ptal_to_archive.product_tmpl_id)._create_variant_ids()
        return True

    def _update_product_template_attribute_values(self):
        """Create or unlink `product.template.attribute.value` for each line in
        `self` based on `value_ids`.

        The goal is to delete all values that are not in `value_ids`, to
        activate those in `value_ids` that are currently archived, and to create
        those in `value_ids` that didn't exist.

        This is a trick for the form view and for performance in general,
        because we don't want to generate in advance all possible values for all
        templates, but only those that will be selected.
        """
        ProductTemplateAttributeValue = self.env['product.template.attribute.value']
        ptav_to_create = []
        ptav_to_unlink = ProductTemplateAttributeValue
        for ptal in self:
            ptav_to_activate = ProductTemplateAttributeValue
            remaining_pav = ptal.value_ids
            for ptav in ptal.product_template_value_ids:
                if ptav.product_attribute_value_id not in remaining_pav:
                    # Remove values that existed but don't exist anymore, but
                    # ignore those that are already archived because if they are
                    # archived it means they could not be deleted previously.
                    if ptav.ptav_active:
                        ptav_to_unlink += ptav
                else:
                    # Activate corresponding values that are currently archived.
                    remaining_pav -= ptav.product_attribute_value_id
                    if not ptav.ptav_active:
                        ptav_to_activate += ptav

            for pav in remaining_pav:
                # The previous loop searched for archived values that belonged to
                # the current line, but if the line was deleted and another line
                # was recreated for the same attribute, we need to expand the
                # search to those with matching `attribute_id`.
                # While not ideal for peformance, this search has to be done at
                # each step to exclude the values that might have been activated
                # at a previous step. Since `remaining_pav` will likely be a
                # small list in all use cases, this is an acceptable trade-off.
                ptav = ProductTemplateAttributeValue.search([
                    ('ptav_active', '=', False),
                    ('product_tmpl_id', '=', ptal.product_tmpl_id.id),
                    ('attribute_id', '=', ptal.attribute_id.id),
                    ('product_attribute_value_id', '=', pav.id),
                ], limit=1)
                if ptav:
                    ptav.write({'ptav_active': True, 'attribute_line_id': ptal.id})
                    # If the value was marked for deletion, now keep it.
                    ptav_to_unlink -= ptav
                else:
                    # create values that didn't exist yet
                    ptav_to_create.append({
                        'product_attribute_value_id': pav.id,
                        'attribute_line_id': ptal.id
                    })
            # Handle active at each step in case a following line might want to
            # re-use a value that was archived at a previous step.
            ptav_to_activate.write({'ptav_active': True})
            ptav_to_unlink.write({'ptav_active': False})
        if ptav_to_unlink:
            ptav_to_unlink.unlink()
        ProductTemplateAttributeValue.create(ptav_to_create)
        self.product_tmpl_id._create_variant_ids()

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        # TDE FIXME: currently overriding the domain; however as it includes a
        # search on a m2o and one on a m2m, probably this will quickly become
        # difficult to compute - check if performance optimization is required
        if name and operator in ('=', 'ilike', '=ilike', 'like', '=like'):
            args = args or []
            domain = ['|', ('attribute_id', operator, name), ('value_ids', operator, name)]
            attribute_ids = self._search(expression.AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)
            return models.lazy_name_get(self.browse(attribute_ids).with_user(name_get_uid))
        return super(ProductTemplateAttributeLine, self)._name_search(name=name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)

    def _without_no_variant_attributes(self):
        return self.filtered(lambda ptal: ptal.attribute_id.create_variant != 'no_variant')


class ProductTemplateAttributeValue(models.Model):
    """Materialized relationship between attribute values
    and product template generated by the product.template.attribute.line"""

    _name = "product.template.attribute.value"
    _description = "Product Template Attribute Value"
    _order = 'attribute_line_id, product_attribute_value_id, id'

    # Not just `active` because we always want to show the values except in
    # specific case, as opposed to `active_test`.
    ptav_active = fields.Boolean("Active", default=True)
    name = fields.Char('Value', related="product_attribute_value_id.name")

    # defining fields: the product template attribute line and the product attribute value
    product_attribute_value_id = fields.Many2one(
        'product.attribute.value', string='Attribute Value',
        required=True, ondelete='cascade', index=True)
    attribute_line_id = fields.Many2one('product.template.attribute.line', required=True, ondelete='cascade', index=True)

    # configuration fields: the price_extra and the exclusion rules
    price_extra = fields.Float(
        string="Value Price Extra",
        default=0.0,
        digits='Product Price',
        help="Extra price for the variant with this attribute value on sale price. eg. 200 price extra, 1000 + 200 = 1200.")
    exclude_for = fields.One2many(
        'product.template.attribute.exclusion',
        'product_template_attribute_value_id',
        string="Exclude for",
        relation="product_template_attribute_exclusion",
        help="Make this attribute value not compatible with "
             "other values of the product or some attribute values of optional and accessory products.")

    # related fields: product template and product attribute
    product_tmpl_id = fields.Many2one('product.template', string="Product Template", related='attribute_line_id.product_tmpl_id', store=True, index=True)
    attribute_id = fields.Many2one('product.attribute', string="Attribute", related='attribute_line_id.attribute_id', store=True, index=True)
    ptav_product_variant_ids = fields.Many2many('product.product', relation='product_variant_combination', string="Related Variants", readonly=True)

    _sql_constraints = [
        ('attribute_value_unique', 'unique(attribute_line_id, product_attribute_value_id)', "Each value should be defined only once per attribute per product."),
    ]

    @api.constrains('attribute_line_id', 'product_attribute_value_id')
    def _check_valid_values(self):
        for ptav in self:
            if ptav.product_attribute_value_id not in ptav.attribute_line_id.value_ids:
                raise ValidationError(
                    _("The value %s is not defined for the attribute %s on the product %s.") %
                    (ptav.product_attribute_value_id.display_name, ptav.attribute_id.display_name, ptav.product_tmpl_id.display_name)
                )

    @api.model_create_multi
    def create(self, vals_list):
        if any('ptav_product_variant_ids' in v for v in vals_list):
            # Force write on this relation from `product.product` to properly
            # trigger `_compute_combination_indices`.
            raise UserError(_("You cannot update related variants from the values. Please update related values from the variants."))
        return super(ProductTemplateAttributeValue, self).create(vals_list)

    def write(self, values):
        if 'ptav_product_variant_ids' in values:
            # Force write on this relation from `product.product` to properly
            # trigger `_compute_combination_indices`.
            raise UserError(_("You cannot update related variants from the values. Please update related values from the variants."))
        pav_in_values = 'product_attribute_value_id' in values
        product_in_values = 'product_tmpl_id' in values
        if pav_in_values or product_in_values:
            for ptav in self:
                if pav_in_values and ptav.product_attribute_value_id.id != values['product_attribute_value_id']:
                    raise UserError(
                        _("You cannot change the value of the value %s set on product %s.") %
                        (ptav.display_name, ptav.product_tmpl_id.display_name)
                    )
                if product_in_values and ptav.product_tmpl_id.id != values['product_tmpl_id']:
                    raise UserError(
                        _("You cannot change the product of the value %s set on product %s.") %
                        (ptav.display_name, ptav.product_tmpl_id.display_name)
                    )
        res = super(ProductTemplateAttributeValue, self).write(values)
        if 'exclude_for' in values:
            self.product_tmpl_id._create_variant_ids()
        return res

    def unlink(self):
        """Override to:
        - Clean up the variants that use any of the values in self:
            - Remove the value from the variant if the value belonged to an
                attribute line with only one value.
            - Unlink or archive all related variants.
        - Archive the value if unlink is not possible.

        Archiving is typically needed when the value is referenced elsewhere
        (on a variant that can't be deleted, on a sales order line, ...).
        """
        # Directly remove the values from the variants for lines that had single
        # value (counting also the values that are archived).
        single_values = self.filtered(lambda ptav: len(ptav.attribute_line_id.product_template_value_ids) == 1)
        for ptav in single_values:
            ptav.ptav_product_variant_ids.write({'product_template_attribute_value_ids': [(3, ptav.id, 0)]})
        # Try to remove the variants before deleting to potentially remove some
        # blocking references.
        self.ptav_product_variant_ids._unlink_or_archive()
        # Now delete or archive the values.
        ptav_to_archive = self.env['product.template.attribute.value']
        for ptav in self:
            try:
                with self.env.cr.savepoint(), tools.mute_logger('odoo.sql_db'):
                    super(ProductTemplateAttributeValue, ptav).unlink()
            except Exception:
                # We catch all kind of exceptions to be sure that the operation
                # doesn't fail.
                ptav_to_archive += ptav
        ptav_to_archive.write({'ptav_active': False})
        return True

    def name_get(self):
        """Override because in general the name of the value is confusing if it
        is displayed without the name of the corresponding attribute.
        Eg. on exclusion rules form
        """
        return [(value.id, "%s: %s" % (value.attribute_id.name, value.name)) for value in self]

    def _only_active(self):
        return self.filtered(lambda ptav: ptav.ptav_active)

    def _without_no_variant_attributes(self):
        return self.filtered(lambda ptav: ptav.attribute_id.create_variant != 'no_variant')

    def _ids2str(self):
        return ','.join([str(i) for i in sorted(self.ids)])

    def _get_combination_name(self):
        """Exclude values from single value lines or from no_variant attributes."""
        ptavs = self._without_no_variant_attributes().with_prefetch(self._prefetch_ids)
        ptavs = ptavs._filter_single_value_lines().with_prefetch(self._prefetch_ids)
        return ", ".join([ptav.name for ptav in ptavs])

    def _filter_single_value_lines(self):
        """Return `self` with values from single value lines filtered out
        depending on the active state of all the values in `self`.

        If any value in `self` is archived, archived values are also taken into
        account when checking for single values.
        This allows to display the correct name for archived variants.

        If all values in `self` are active, only active values are taken into
        account when checking for single values.
        This allows to display the correct name for active combinations.
        """
        only_active = all(ptav.ptav_active for ptav in self)
        return self.filtered(lambda ptav: not ptav._is_from_single_value_line(only_active))

    def _is_from_single_value_line(self, only_active=True):
        """Return whether `self` is from a single value line, counting also
        archived values if `only_active` is False.
        """
        self.ensure_one()
        all_values = self.attribute_line_id.product_template_value_ids
        if only_active:
            all_values = all_values._only_active()
        return len(all_values) == 1


class ProductTemplateAttributeExclusion(models.Model):
    _name = "product.template.attribute.exclusion"
    _description = 'Product Template Attribute Exclusion'
    _order = 'product_tmpl_id, id'

    product_template_attribute_value_id = fields.Many2one(
        'product.template.attribute.value', string="Attribute Value", ondelete='cascade', index=True)
    product_tmpl_id = fields.Many2one(
        'product.template', string='Product Template', ondelete='cascade', required=True, index=True)
    value_ids = fields.Many2many(
        'product.template.attribute.value', relation="product_attr_exclusion_value_ids_rel",
        string='Attribute Values', domain="[('product_tmpl_id', '=', product_tmpl_id), ('ptav_active', '=', True)]")

```

## File: models\product_pricelist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from itertools import chain

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_repr
from odoo.tools.misc import get_lang


class Pricelist(models.Model):
    _name = "product.pricelist"
    _description = "Pricelist"
    _order = "sequence asc, id desc"

    def _get_default_currency_id(self):
        return self.env.company.currency_id.id

    name = fields.Char('Pricelist Name', required=True, translate=True)
    active = fields.Boolean('Active', default=True, help="If unchecked, it will allow you to hide the pricelist without removing it.")
    item_ids = fields.One2many(
        'product.pricelist.item', 'pricelist_id', 'Pricelist Items',
        copy=True)
    currency_id = fields.Many2one('res.currency', 'Currency', default=_get_default_currency_id, required=True)
    company_id = fields.Many2one('res.company', 'Company')

    sequence = fields.Integer(default=16)
    country_group_ids = fields.Many2many('res.country.group', 'res_country_group_pricelist_rel',
                                         'pricelist_id', 'res_country_group_id', string='Country Groups')

    discount_policy = fields.Selection([
        ('with_discount', 'Discount included in the price'),
        ('without_discount', 'Show public price & discount to the customer')],
        default='with_discount')

    def name_get(self):
        return [(pricelist.id, '%s (%s)' % (pricelist.name, pricelist.currency_id.name)) for pricelist in self]

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        if name and operator == '=' and not args:
            # search on the name of the pricelist and its currency, opposite of name_get(),
            # Used by the magic context filter in the product search view.
            query_args = {'name': name, 'limit': limit, 'lang': get_lang(self.env).code}
            query = """SELECT p.id
                       FROM ((
                                SELECT pr.id, pr.name
                                FROM product_pricelist pr JOIN
                                     res_currency cur ON
                                         (pr.currency_id = cur.id)
                                WHERE pr.name || ' (' || cur.name || ')' = %(name)s
                            )
                            UNION (
                                SELECT tr.res_id as id, tr.value as name
                                FROM ir_translation tr JOIN
                                     product_pricelist pr ON (
                                        pr.id = tr.res_id AND
                                        tr.type = 'model' AND
                                        tr.name = 'product.pricelist,name' AND
                                        tr.lang = %(lang)s
                                     ) JOIN
                                     res_currency cur ON
                                         (pr.currency_id = cur.id)
                                WHERE tr.value || ' (' || cur.name || ')' = %(name)s
                            )
                        ) p
                       ORDER BY p.name"""
            if limit:
                query += " LIMIT %(limit)s"
            self._cr.execute(query, query_args)
            ids = [r[0] for r in self._cr.fetchall()]
            # regular search() to apply ACLs - may limit results below limit in some cases
            pricelist_ids = self._search([('id', 'in', ids)], limit=limit, access_rights_uid=name_get_uid)
            if pricelist_ids:
                return models.lazy_name_get(self.browse(pricelist_ids).with_user(name_get_uid))
        return super(Pricelist, self)._name_search(name, args, operator=operator, limit=limit, name_get_uid=name_get_uid)

    def _compute_price_rule_multi(self, products_qty_partner, date=False, uom_id=False):
        """ Low-level method - Multi pricelist, multi products
        Returns: dict{product_id: dict{pricelist_id: (price, suitable_rule)} }"""
        if not self.ids:
            pricelists = self.search([])
        else:
            pricelists = self
        results = {}
        for pricelist in pricelists:
            subres = pricelist._compute_price_rule(products_qty_partner, date=date, uom_id=uom_id)
            for product_id, price in subres.items():
                results.setdefault(product_id, {})
                results[product_id][pricelist.id] = price
        return results

    def _compute_price_rule_get_items(self, products_qty_partner, date, uom_id, prod_tmpl_ids, prod_ids, categ_ids):
        self.ensure_one()
        # Load all rules
        self.env['product.pricelist.item'].flush(['price', 'currency_id', 'company_id', 'active'])
        self.env.cr.execute(
            """
            SELECT
                item.id
            FROM
                product_pricelist_item AS item
            LEFT JOIN product_category AS categ ON item.categ_id = categ.id
            WHERE
                (item.product_tmpl_id IS NULL OR item.product_tmpl_id = any(%s))
                AND (item.product_id IS NULL OR item.product_id = any(%s))
                AND (item.categ_id IS NULL OR item.categ_id = any(%s))
                AND (item.pricelist_id = %s)
                AND (item.date_start IS NULL OR item.date_start<=%s)
                AND (item.date_end IS NULL OR item.date_end>=%s)
                AND (item.active = TRUE)
            ORDER BY
                item.applied_on, item.min_quantity desc, categ.complete_name desc, item.id desc
            """,
            (prod_tmpl_ids, prod_ids, categ_ids, self.id, date, date))
        # NOTE: if you change `order by` on that query, make sure it matches
        # _order from model to avoid inconstencies and undeterministic issues.

        item_ids = [x[0] for x in self.env.cr.fetchall()]
        return self.env['product.pricelist.item'].browse(item_ids)

    def _compute_price_rule(self, products_qty_partner, date=False, uom_id=False):
        """ Low-level method - Mono pricelist, multi products
        Returns: dict{product_id: (price, suitable_rule) for the given pricelist}

        Date in context can be a date, datetime, ...

            :param products_qty_partner: list of typles products, quantity, partner
            :param datetime date: validity date
            :param ID uom_id: intermediate unit of measure
        """
        self.ensure_one()
        if not date:
            date = self._context.get('date') or fields.Date.today()
        date = fields.Date.to_date(date)  # boundary conditions differ if we have a datetime
        if not uom_id and self._context.get('uom'):
            uom_id = self._context['uom']
        if uom_id:
            # rebrowse with uom if given
            products = [item[0].with_context(uom=uom_id) for item in products_qty_partner]
            products_qty_partner = [(products[index], data_struct[1], data_struct[2]) for index, data_struct in enumerate(products_qty_partner)]
        else:
            products = [item[0] for item in products_qty_partner]

        if not products:
            return {}

        categ_ids = {}
        for p in products:
            categ = p.categ_id
            while categ:
                categ_ids[categ.id] = True
                categ = categ.parent_id
        categ_ids = list(categ_ids)

        is_product_template = products[0]._name == "product.template"
        if is_product_template:
            prod_tmpl_ids = [tmpl.id for tmpl in products]
            # all variants of all products
            prod_ids = [p.id for p in
                        list(chain.from_iterable([t.product_variant_ids for t in products]))]
        else:
            prod_ids = [product.id for product in products]
            prod_tmpl_ids = [product.product_tmpl_id.id for product in products]

        items = self._compute_price_rule_get_items(products_qty_partner, date, uom_id, prod_tmpl_ids, prod_ids, categ_ids)

        results = {}
        for product, qty, partner in products_qty_partner:
            results[product.id] = 0.0
            suitable_rule = False

            # Final unit price is computed according to `qty` in the `qty_uom_id` UoM.
            # An intermediary unit price may be computed according to a different UoM, in
            # which case the price_uom_id contains that UoM.
            # The final price will be converted to match `qty_uom_id`.
            qty_uom_id = self._context.get('uom') or product.uom_id.id
            qty_in_product_uom = qty
            if qty_uom_id != product.uom_id.id:
                try:
                    qty_in_product_uom = self.env['uom.uom'].browse([self._context['uom']])._compute_quantity(qty, product.uom_id)
                except UserError:
                    # Ignored - incompatible UoM in context, use default product UoM
                    pass

            # if Public user try to access standard price from website sale, need to call price_compute.
            # TDE SURPRISE: product can actually be a template
            price = product.price_compute('list_price')[product.id]

            price_uom = self.env['uom.uom'].browse([qty_uom_id])
            for rule in items:
                if rule.min_quantity and qty_in_product_uom < rule.min_quantity:
                    continue
                if is_product_template:
                    if rule.product_tmpl_id and product.id != rule.product_tmpl_id.id:
                        continue
                    if rule.product_id and not (product.product_variant_count == 1 and product.product_variant_id.id == rule.product_id.id):
                        # product rule acceptable on template if has only one variant
                        continue
                else:
                    if rule.product_tmpl_id and product.product_tmpl_id.id != rule.product_tmpl_id.id:
                        continue
                    if rule.product_id and product.id != rule.product_id.id:
                        continue

                if rule.categ_id:
                    cat = product.categ_id
                    while cat:
                        if cat.id == rule.categ_id.id:
                            break
                        cat = cat.parent_id
                    if not cat:
                        continue

                if rule.base == 'pricelist' and rule.base_pricelist_id:
                    price = rule.base_pricelist_id._compute_price_rule([(product, qty, partner)], date, uom_id)[product.id][0]  # TDE: 0 = price, 1 = rule
                    src_currency = rule.base_pricelist_id.currency_id
                else:
                    # if base option is public price take sale price else cost price of product
                    # price_compute returns the price in the context UoM, i.e. qty_uom_id
                    price = product.price_compute(rule.base)[product.id]
                    if rule.base == 'standard_price':
                        src_currency = product.cost_currency_id
                    else:
                        src_currency = product.currency_id

                if src_currency != self.currency_id:
                    price = src_currency._convert(
                        price, self.currency_id, self.env.company, date, round=False)

                if price is not False:
                    price = rule._compute_price(price, price_uom, product, quantity=qty, partner=partner)
                    suitable_rule = rule
                break

            if not suitable_rule:
                cur = product.currency_id
                price = cur._convert(price, self.currency_id, self.env.company, date, round=False)

            results[product.id] = (price, suitable_rule and suitable_rule.id or False)

        return results

    # New methods: product based
    def get_products_price(self, products, quantities, partners, date=False, uom_id=False):
        """ For a given pricelist, return price for products
        Returns: dict{product_id: product price}, in the given pricelist """
        self.ensure_one()
        return {
            product_id: res_tuple[0]
            for product_id, res_tuple in self._compute_price_rule(
                list(zip(products, quantities, partners)),
                date=date,
                uom_id=uom_id
            ).items()
        }

    def get_product_price(self, product, quantity, partner, date=False, uom_id=False):
        """ For a given pricelist, return price for a given product """
        self.ensure_one()
        return self._compute_price_rule([(product, quantity, partner)], date=date, uom_id=uom_id)[product.id][0]

    def get_product_price_rule(self, product, quantity, partner, date=False, uom_id=False):
        """ For a given pricelist, return price and rule for a given product """
        self.ensure_one()
        return self._compute_price_rule([(product, quantity, partner)], date=date, uom_id=uom_id)[product.id]

    def price_get(self, prod_id, qty, partner=None):
        """ Multi pricelist, mono product - returns price per pricelist """
        return {key: price[0] for key, price in self.price_rule_get(prod_id, qty, partner=partner).items()}

    def price_rule_get_multi(self, products_by_qty_by_partner):
        """ Multi pricelist, multi product  - return tuple """
        return self._compute_price_rule_multi(products_by_qty_by_partner)

    def price_rule_get(self, prod_id, qty, partner=None):
        """ Multi pricelist, mono product - return tuple """
        product = self.env['product.product'].browse([prod_id])
        return self._compute_price_rule_multi([(product, qty, partner)])[prod_id]

    @api.model
    def _price_get_multi(self, pricelist, products_by_qty_by_partner):
        """ Mono pricelist, multi product - return price per product """
        return pricelist.get_products_price(
            list(zip(**products_by_qty_by_partner)))

    def _get_partner_pricelist_multi_search_domain_hook(self, company_id):
        return [
            ('active', '=', True),
            ('company_id', 'in', [company_id, False]),
        ]

    def _get_partner_pricelist_multi_filter_hook(self):
        return self.filtered('active')

    def _get_partner_pricelist_multi(self, partner_ids, company_id=None):
        """ Retrieve the applicable pricelist for given partners in a given company.

            It will return the first found pricelist in this order:
            First, the pricelist of the specific property (res_id set), this one
                   is created when saving a pricelist on the partner form view.
            Else, it will return the pricelist of the partner country group
            Else, it will return the generic property (res_id not set), this one
                  is created on the company creation.
            Else, it will return the first available pricelist

            :param company_id: if passed, used for looking up properties,
                instead of current user's company
            :return: a dict {partner_id: pricelist}
        """
        # `partner_ids` might be ID from inactive uers. We should use active_test
        # as we will do a search() later (real case for website public user).
        Partner = self.env['res.partner'].with_context(active_test=False)
        company_id = company_id or self.env.company.id

        Property = self.env['ir.property'].with_context(force_company=company_id)
        Pricelist = self.env['product.pricelist']
        pl_domain = self._get_partner_pricelist_multi_search_domain_hook(company_id)

        # if no specific property, try to find a fitting pricelist
        result = Property.get_multi('property_product_pricelist', Partner._name, partner_ids)

        remaining_partner_ids = [pid for pid, val in result.items() if not val or
                                 not val._get_partner_pricelist_multi_filter_hook()]
        if remaining_partner_ids:
            # get fallback pricelist when no pricelist for a given country
            pl_fallback = (
                Pricelist.search(pl_domain + [('country_group_ids', '=', False)], limit=1) or
                Property.get('property_product_pricelist', 'res.partner') or
                Pricelist.search(pl_domain, limit=1)
            )
            # group partners by country, and find a pricelist for each country
            domain = [('id', 'in', remaining_partner_ids)]
            groups = Partner.read_group(domain, ['country_id'], ['country_id'])
            for group in groups:
                country_id = group['country_id'] and group['country_id'][0]
                pl = Pricelist.search(pl_domain + [('country_group_ids.country_ids', '=', country_id)], limit=1)
                pl = pl or pl_fallback
                for pid in Partner.search(group['__domain']).ids:
                    result[pid] = pl

        return result

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Pricelists'),
            'template': '/product/static/xls/product_pricelist.xls'
        }]


class ResCountryGroup(models.Model):
    _inherit = 'res.country.group'

    pricelist_ids = fields.Many2many('product.pricelist', 'res_country_group_pricelist_rel',
                                     'res_country_group_id', 'pricelist_id', string='Pricelists')


class PricelistItem(models.Model):
    _name = "product.pricelist.item"
    _description = "Pricelist Rule"
    _order = "applied_on, min_quantity desc, categ_id desc, id desc"
    _check_company_auto = True
    # NOTE: if you change _order on this model, make sure it matches the SQL
    # query built in _compute_price_rule() above in this file to avoid
    # inconstencies and undeterministic issues.

    def _default_pricelist_id(self):
        return self.env['product.pricelist'].search([
            '|', ('company_id', '=', False),
            ('company_id', '=', self.env.company.id)], limit=1)

    product_tmpl_id = fields.Many2one(
        'product.template', 'Product', ondelete='cascade', check_company=True,
        help="Specify a template if this rule only applies to one product template. Keep empty otherwise.")
    product_id = fields.Many2one(
        'product.product', 'Product Variant', ondelete='cascade', check_company=True,
        help="Specify a product if this rule only applies to one product. Keep empty otherwise.")
    categ_id = fields.Many2one(
        'product.category', 'Product Category', ondelete='cascade',
        help="Specify a product category if this rule only applies to products belonging to this category or its children categories. Keep empty otherwise.")
    min_quantity = fields.Integer(
        'Min. Quantity', default=0,
        help="For the rule to apply, bought/sold quantity must be greater "
             "than or equal to the minimum quantity specified in this field.\n"
             "Expressed in the default unit of measure of the product.")
    applied_on = fields.Selection([
        ('3_global', 'All Products'),
        ('2_product_category', 'Product Category'),
        ('1_product', 'Product'),
        ('0_product_variant', 'Product Variant')], "Apply On",
        default='3_global', required=True,
        help='Pricelist Item applicable on selected option')
    base = fields.Selection([
        ('list_price', 'Sales Price'),
        ('standard_price', 'Cost'),
        ('pricelist', 'Other Pricelist')], "Based on",
        default='list_price', required=True,
        help='Base price for computation.\n'
             'Sales Price: The base price will be the Sales Price.\n'
             'Cost Price : The base price will be the cost price.\n'
             'Other Pricelist : Computation of the base price based on another Pricelist.')
    base_pricelist_id = fields.Many2one('product.pricelist', 'Other Pricelist', check_company=True)
    pricelist_id = fields.Many2one('product.pricelist', 'Pricelist', index=True, ondelete='cascade', required=True, default=_default_pricelist_id)
    price_surcharge = fields.Float(
        'Price Surcharge', digits='Product Price',
        help='Specify the fixed amount to add or substract(if negative) to the amount calculated with the discount.')
    price_discount = fields.Float('Price Discount', default=0, digits=(16, 2))
    price_round = fields.Float(
        'Price Rounding', digits='Product Price',
        help="Sets the price so that it is a multiple of this value.\n"
             "Rounding is applied after the discount and before the surcharge.\n"
             "To have prices that end in 9.99, set rounding 10, surcharge -0.01")
    price_min_margin = fields.Float(
        'Min. Price Margin', digits='Product Price',
        help='Specify the minimum amount of margin over the base price.')
    price_max_margin = fields.Float(
        'Max. Price Margin', digits='Product Price',
        help='Specify the maximum amount of margin over the base price.')
    company_id = fields.Many2one(
        'res.company', 'Company',
        readonly=True, related='pricelist_id.company_id', store=True)
    currency_id = fields.Many2one(
        'res.currency', 'Currency',
        readonly=True, related='pricelist_id.currency_id', store=True)
    active = fields.Boolean(
        readonly=True, related="pricelist_id.active", store=True)
    date_start = fields.Date('Start Date', help="Starting date for the pricelist item validation")
    date_end = fields.Date('End Date', help="Ending valid for the pricelist item validation")
    compute_price = fields.Selection([
        ('fixed', 'Fixed Price'),
        ('percentage', 'Percentage (discount)'),
        ('formula', 'Formula')], index=True, default='fixed', required=True)
    fixed_price = fields.Float('Fixed Price', digits='Product Price')
    percent_price = fields.Float('Percentage Price')
    # functional fields used for usability purposes
    name = fields.Char(
        'Name', compute='_get_pricelist_item_name_price',
        help="Explicit rule name for this pricelist line.")
    price = fields.Char(
        'Price', compute='_get_pricelist_item_name_price',
        help="Explicit rule name for this pricelist line.")

    @api.constrains('base_pricelist_id', 'pricelist_id', 'base')
    def _check_recursion(self):
        if any(item.base == 'pricelist' and item.pricelist_id and item.pricelist_id == item.base_pricelist_id for item in self):
            raise ValidationError(_('You cannot assign the Main Pricelist as Other Pricelist in PriceList Item'))
        return True

    @api.constrains('price_min_margin', 'price_max_margin')
    def _check_margin(self):
        if any(item.price_min_margin > item.price_max_margin for item in self):
            raise ValidationError(_('The minimum margin should be lower than the maximum margin.'))
        return True

    @api.constrains('product_id', 'product_tmpl_id', 'categ_id')
    def _check_product_consistency(self):
        for item in self:
            if item.applied_on == "2_product_category" and not item.categ_id:
                raise ValidationError(_("Please specify the category for which this rule should be applied"))
            elif item.applied_on == "1_product" and not item.product_tmpl_id:
                raise ValidationError(_("Please specify the product for which this rule should be applied"))
            elif item.applied_on == "0_product_variant" and not item.product_id:
                raise ValidationError(_("Please specify the product variant for which this rule should be applied"))

    @api.depends('applied_on', 'categ_id', 'product_tmpl_id', 'product_id', 'compute_price', 'fixed_price', \
        'pricelist_id', 'percent_price', 'price_discount', 'price_surcharge')
    def _get_pricelist_item_name_price(self):
        for item in self:
            if item.categ_id and item.applied_on == '2_product_category':
                item.name = _("Category: %s") % (item.categ_id.display_name)
            elif item.product_tmpl_id and item.applied_on == '1_product':
                item.name = _("Product: %s") % (item.product_tmpl_id.display_name)
            elif item.product_id and item.applied_on == '0_product_variant':
                item.name = _("Variant: %s") % (item.product_id.with_context(display_default_code=False).display_name)
            else:
                item.name = _("All Products")

            if item.compute_price == 'fixed':
                decimal_places = self.env['decimal.precision'].precision_get('Product Price')
                if item.currency_id.position == 'after':
                    item.price = "%s %s" % (
                        float_repr(
                            item.fixed_price,
                            decimal_places,
                        ),
                        item.currency_id.symbol,
                    )
                else:
                    item.price = "%s %s" % (
                        item.currency_id.symbol,
                        float_repr(
                            item.fixed_price,
                            decimal_places,
                        ),
                    )
            elif item.compute_price == 'percentage':
                item.price = _("%s %% discount") % (item.percent_price)
            else:
                item.price = _("%s %% discount and %s surcharge") % (item.price_discount, item.price_surcharge)

    @api.onchange('compute_price')
    def _onchange_compute_price(self):
        if self.compute_price != 'fixed':
            self.fixed_price = 0.0
        if self.compute_price != 'percentage':
            self.percent_price = 0.0
        if self.compute_price != 'formula':
            self.update({
                'base': 'list_price',
                'price_discount': 0.0,
                'price_surcharge': 0.0,
                'price_round': 0.0,
                'price_min_margin': 0.0,
                'price_max_margin': 0.0,
            })

    @api.onchange('product_id')
    def _onchange_product_id(self):
        has_product_id = self.filtered('product_id')
        for item in has_product_id:
            item.product_tmpl_id = item.product_id.product_tmpl_id
        if self.env.context.get('default_applied_on', False) == '1_product':
            # If a product variant is specified, apply on variants instead
            # Reset if product variant is removed
            has_product_id.update({'applied_on': '0_product_variant'})
            (self - has_product_id).update({'applied_on': '1_product'})

    @api.onchange('product_tmpl_id')
    def _onchange_product_tmpl_id(self):
        has_tmpl_id = self.filtered('product_tmpl_id')
        for item in has_tmpl_id:
            if item.product_id and item.product_id.product_tmpl_id != item.product_tmpl_id:
                item.product_id = None

    @api.onchange('product_id', 'product_tmpl_id', 'categ_id')
    def _onchane_rule_content(self):
        if not self.user_has_groups('product.group_sale_pricelist') and not self.env.context.get('default_applied_on', False):
            # If advanced pricelists are disabled (applied_on field is not visible)
            # AND we aren't coming from a specific product template/variant.
            variants_rules = self.filtered('product_id')
            template_rules = (self-variants_rules).filtered('product_tmpl_id')
            variants_rules.update({'applied_on': '0_product_variant'})
            template_rules.update({'applied_on': '1_product'})
            (self-variants_rules-template_rules).update({'applied_on': '3_global'})

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('applied_on', False):
                # Ensure item consistency for later searches.
                applied_on = values['applied_on']
                if applied_on == '3_global':
                    values.update(dict(product_id=None, product_tmpl_id=None, categ_id=None))
                elif applied_on == '2_product_category':
                    values.update(dict(product_id=None, product_tmpl_id=None))
                elif applied_on == '1_product':
                    values.update(dict(product_id=None, categ_id=None))
                elif applied_on == '0_product_variant':
                    values.update(dict(categ_id=None))
        return super(PricelistItem, self).create(vals_list)

    def write(self, values):
        if values.get('applied_on', False):
            # Ensure item consistency for later searches.
            applied_on = values['applied_on']
            if applied_on == '3_global':
                values.update(dict(product_id=None, product_tmpl_id=None, categ_id=None))
            elif applied_on == '2_product_category':
                values.update(dict(product_id=None, product_tmpl_id=None))
            elif applied_on == '1_product':
                values.update(dict(product_id=None, categ_id=None))
            elif applied_on == '0_product_variant':
                values.update(dict(categ_id=None))
        res = super(PricelistItem, self).write(values)
        # When the pricelist changes we need the product.template price
        # to be invalided and recomputed.
        self.env['product.template'].invalidate_cache(['price'])
        self.env['product.product'].invalidate_cache(['price'])
        return res

    def toggle_active(self):
        raise ValidationError(_("You cannot disable a pricelist rule, please delete it or archive its pricelist instead."))

    def _compute_price(self, price, price_uom, product, quantity=1.0, partner=False):
        """Compute the unit price of a product in the context of a pricelist application.
           The unused parameters are there to make the full context available for overrides.
        """
        self.ensure_one()
        convert_to_price_uom = (lambda price: product.uom_id._compute_price(price, price_uom))
        if self.compute_price == 'fixed':
            price = convert_to_price_uom(self.fixed_price)
        elif self.compute_price == 'percentage':
            price = (price - (price * (self.percent_price / 100))) or 0.0
        else:
            # complete formula
            price_limit = price
            price = (price - (price * (self.price_discount / 100))) or 0.0

            if self.price_round:
                price = tools.float_round(price, precision_rounding=self.price_round)

            if self.price_surcharge:
                price += convert_to_price_uom(self.price_surcharge)

            if self.price_min_margin:
                price_min_margin = convert_to_price_uom(self.price_min_margin)
                price = max(price, price_limit + price_min_margin)

            if self.price_max_margin:
                price_max_margin = convert_to_price_uom(self.price_max_margin)
                price = min(price, price_limit + price_max_margin)

        return price

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import itertools
import logging
from collections import defaultdict

from odoo import api, fields, models, tools, _, SUPERUSER_ID
from odoo.exceptions import ValidationError, RedirectWarning, UserError
from odoo.osv import expression

_logger = logging.getLogger(__name__)


class ProductTemplate(models.Model):
    _name = "product.template"
    _inherit = ['mail.thread', 'mail.activity.mixin', 'image.mixin']
    _description = "Product Template"
    _order = "name"

    def _get_default_category_id(self):
        if self._context.get('categ_id') or self._context.get('default_categ_id'):
            return self._context.get('categ_id') or self._context.get('default_categ_id')
        category = self.env.ref('product.product_category_all', raise_if_not_found=False)
        if not category:
            category = self.env['product.category'].search([], limit=1)
        if category:
            return category.id
        else:
            err_msg = _('You must define at least one product category in order to be able to create products.')
            redir_msg = _('Go to Internal Categories')
            raise RedirectWarning(err_msg, self.env.ref('product.product_category_action_form').id, redir_msg)

    def _get_default_uom_id(self):
        return self.env["uom.uom"].search([], limit=1, order='id').id

    def _get_default_weight_uom(self):
        return self._get_weight_uom_name_from_ir_config_parameter()

    def _get_default_volume_uom(self):
        return self._get_volume_uom_name_from_ir_config_parameter()

    def _read_group_categ_id(self, categories, domain, order):
        category_ids = self.env.context.get('default_categ_id')
        if not category_ids and self.env.context.get('group_expand'):
            category_ids = categories._search([], order=order, access_rights_uid=SUPERUSER_ID)
        return categories.browse(category_ids)

    name = fields.Char('Name', index=True, required=True, translate=True)
    sequence = fields.Integer('Sequence', default=1, help='Gives the sequence order when displaying a product list')
    description = fields.Text(
        'Description', translate=True)
    description_purchase = fields.Text(
        'Purchase Description', translate=True)
    description_sale = fields.Text(
        'Sales Description', translate=True,
        help="A description of the Product that you want to communicate to your customers. "
             "This description will be copied to every Sales Order, Delivery Order and Customer Invoice/Credit Note")
    type = fields.Selection([
        ('consu', 'Consumable'),
        ('service', 'Service')], string='Product Type', default='consu', required=True,
        help='A storable product is a product for which you manage stock. The Inventory app has to be installed.\n'
             'A consumable product is a product for which stock is not managed.\n'
             'A service is a non-material product you provide.')
    rental = fields.Boolean('Can be Rent')
    categ_id = fields.Many2one(
        'product.category', 'Product Category',
        change_default=True, default=_get_default_category_id, group_expand='_read_group_categ_id',
        required=True, help="Select category for the current product")

    currency_id = fields.Many2one(
        'res.currency', 'Currency', compute='_compute_currency_id')
    cost_currency_id = fields.Many2one(
        'res.currency', 'Cost Currency', compute='_compute_cost_currency_id')

    # price fields
    # price: total template price, context dependent (partner, pricelist, quantity)
    price = fields.Float(
        'Price', compute='_compute_template_price', inverse='_set_template_price',
        digits='Product Price')
    # list_price: catalog price, user defined
    list_price = fields.Float(
        'Sales Price', default=1.0,
        digits='Product Price',
        help="Price at which the product is sold to customers.")
    # lst_price: catalog price for template, but including extra for variants
    lst_price = fields.Float(
        'Public Price', related='list_price', readonly=False,
        digits='Product Price')
    standard_price = fields.Float(
        'Cost', compute='_compute_standard_price',
        inverse='_set_standard_price', search='_search_standard_price',
        digits='Product Price', groups="base.group_user",
        help="""In Standard Price & AVCO: value of the product (automatically computed in AVCO).
        In FIFO: value of the last unit that left the stock (automatically computed).
        Used to value the product when the purchase cost is not known (e.g. inventory adjustment).
        Used to compute margins on sale orders.""")

    volume = fields.Float(
        'Volume', compute='_compute_volume', inverse='_set_volume', digits='Volume', store=True)
    volume_uom_name = fields.Char(string='Volume unit of measure label', compute='_compute_volume_uom_name', default=_get_default_volume_uom)
    weight = fields.Float(
        'Weight', compute='_compute_weight', digits='Stock Weight',
        inverse='_set_weight', store=True)
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name', readonly=True, default=_get_default_weight_uom)

    sale_ok = fields.Boolean('Can be Sold', default=True)
    purchase_ok = fields.Boolean('Can be Purchased', default=True)
    pricelist_id = fields.Many2one(
        'product.pricelist', 'Pricelist', store=False,
        help='Technical field. Used for searching on pricelists, not stored in database.')
    uom_id = fields.Many2one(
        'uom.uom', 'Unit of Measure',
        default=_get_default_uom_id, required=True,
        help="Default unit of measure used for all stock operations.")
    uom_name = fields.Char(string='Unit of Measure Name', related='uom_id.name', readonly=True)
    uom_po_id = fields.Many2one(
        'uom.uom', 'Purchase Unit of Measure',
        default=_get_default_uom_id, required=True,
        help="Default unit of measure used for purchase orders. It must be in the same category as the default unit of measure.")
    company_id = fields.Many2one(
        'res.company', 'Company', index=1)
    packaging_ids = fields.One2many(
        'product.packaging', string="Product Packages", compute="_compute_packaging_ids", inverse="_set_packaging_ids",
        help="Gives the different ways to package the same product.")
    seller_ids = fields.One2many('product.supplierinfo', 'product_tmpl_id', 'Vendors', help="Define vendor pricelists.")
    variant_seller_ids = fields.One2many('product.supplierinfo', 'product_tmpl_id')

    active = fields.Boolean('Active', default=True, help="If unchecked, it will allow you to hide the product without removing it.")
    color = fields.Integer('Color Index')

    is_product_variant = fields.Boolean(string='Is a product variant', compute='_compute_is_product_variant')
    attribute_line_ids = fields.One2many('product.template.attribute.line', 'product_tmpl_id', 'Product Attributes', copy=True)

    valid_product_template_attribute_line_ids = fields.Many2many('product.template.attribute.line',
        compute="_compute_valid_product_template_attribute_line_ids", string='Valid Product Attribute Lines', help="Technical compute")

    product_variant_ids = fields.One2many('product.product', 'product_tmpl_id', 'Products', required=True)
    # performance: product_variant_id provides prefetching on the first product variant only
    product_variant_id = fields.Many2one('product.product', 'Product', compute='_compute_product_variant_id')

    product_variant_count = fields.Integer(
        '# Product Variants', compute='_compute_product_variant_count')

    # related to display product product information if is_product_variant
    barcode = fields.Char('Barcode', related='product_variant_ids.barcode', readonly=False)
    default_code = fields.Char(
        'Internal Reference', compute='_compute_default_code',
        inverse='_set_default_code', store=True)

    pricelist_item_count = fields.Integer("Number of price rules", compute="_compute_item_count")

    can_image_1024_be_zoomed = fields.Boolean("Can Image 1024 be zoomed", compute='_compute_can_image_1024_be_zoomed', store=True)
    has_configurable_attributes = fields.Boolean("Is a configurable product", compute='_compute_has_configurable_attributes', store=True)

    def _compute_item_count(self):
        for template in self:
            # Pricelist item count counts the rules applicable on current template or on its variants.
            template.pricelist_item_count = template.env['product.pricelist.item'].search_count([
                '|', ('product_tmpl_id', '=', template.id), ('product_id', 'in', template.product_variant_ids.ids)])

    @api.depends('image_1920', 'image_1024')
    def _compute_can_image_1024_be_zoomed(self):
        for template in self:
            template.can_image_1024_be_zoomed = template.image_1920 and tools.is_image_size_above(template.image_1920, template.image_1024)

    @api.depends('attribute_line_ids', 'attribute_line_ids.value_ids', 'attribute_line_ids.attribute_id.create_variant')
    def _compute_has_configurable_attributes(self):
        """A product is considered configurable if:
        - It has dynamic attributes
        - It has any attribute line with at least 2 attribute values configured
        """
        for product in self:
            product.has_configurable_attributes = product.has_dynamic_attributes() or any(len(ptal.value_ids) >= 2 for ptal in product.attribute_line_ids)

    @api.depends('product_variant_ids')
    def _compute_product_variant_id(self):
        for p in self:
            p.product_variant_id = p.product_variant_ids[:1].id

    @api.depends('company_id')
    def _compute_currency_id(self):
        main_company = self.env['res.company']._get_main_company()
        for template in self:
            template.currency_id = template.company_id.sudo().currency_id.id or main_company.currency_id.id

    @api.depends_context('force_company')
    def _compute_cost_currency_id(self):
        # Cost_currency_id is the displayed currency for standard_price
        # which is company_dependent and thus depends on force_company
        # context key (or self.env.company)
        company_id = self._context.get('force_company') or self.env.company.id
        company = self.env['res.company'].browse(company_id)
        for template in self:
            template.cost_currency_id = company.currency_id.id

    def _compute_template_price(self):
        prices = self._compute_template_price_no_inverse()
        for template in self:
            template.price = prices.get(template.id, 0.0)

    def _compute_template_price_no_inverse(self):
        """The _compute_template_price writes the 'list_price' field with an inverse method
        This method allows computing the price without writing the 'list_price'
        """
        prices = {}
        pricelist_id_or_name = self._context.get('pricelist')
        if pricelist_id_or_name:
            pricelist = None
            partner = self.env.context.get('partner')
            quantity = self.env.context.get('quantity', 1.0)

            # Support context pricelists specified as list, display_name or ID for compatibility
            if isinstance(pricelist_id_or_name, list):
                pricelist_id_or_name = pricelist_id_or_name[0]
            if isinstance(pricelist_id_or_name, str):
                pricelist_data = self.env['product.pricelist'].name_search(pricelist_id_or_name, operator='=', limit=1)
                if pricelist_data:
                    pricelist = self.env['product.pricelist'].browse(pricelist_data[0][0])
            elif isinstance(pricelist_id_or_name, int):
                pricelist = self.env['product.pricelist'].browse(pricelist_id_or_name)

            if pricelist:
                quantities = [quantity] * len(self)
                partners = [partner] * len(self)
                prices = pricelist.get_products_price(self, quantities, partners)

        return prices

    def _set_template_price(self):
        if self._context.get('uom'):
            for template in self:
                value = self.env['uom.uom'].browse(self._context['uom'])._compute_price(template.price, template.uom_id)
                template.write({'list_price': value})
        else:
            self.write({'list_price': self.price})

    @api.depends_context('force_company')
    @api.depends('product_variant_ids', 'product_variant_ids.standard_price')
    def _compute_standard_price(self):
        # Depends on force_company context because standard_price is company_dependent
        # on the product_product
        unique_variants = self.filtered(lambda template: len(template.product_variant_ids) == 1)
        for template in unique_variants:
            template.standard_price = template.product_variant_ids.standard_price
        for template in (self - unique_variants):
            template.standard_price = 0.0

    def _set_standard_price(self):
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.product_variant_ids.standard_price = template.standard_price

    def _search_standard_price(self, operator, value):
        products = self.env['product.product'].search([('standard_price', operator, value)], limit=None)
        return [('id', 'in', products.mapped('product_tmpl_id').ids)]

    @api.depends('product_variant_ids', 'product_variant_ids.volume')
    def _compute_volume(self):
        unique_variants = self.filtered(lambda template: len(template.product_variant_ids) == 1)
        for template in unique_variants:
            template.volume = template.product_variant_ids.volume
        for template in (self - unique_variants):
            template.volume = 0.0

    def _set_volume(self):
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.product_variant_ids.volume = template.volume

    @api.depends('product_variant_ids', 'product_variant_ids.weight')
    def _compute_weight(self):
        unique_variants = self.filtered(lambda template: len(template.product_variant_ids) == 1)
        for template in unique_variants:
            template.weight = template.product_variant_ids.weight
        for template in (self - unique_variants):
            template.weight = 0.0

    def _compute_is_product_variant(self):
        for template in self:
            template.is_product_variant = False

    @api.model
    def _get_weight_uom_id_from_ir_config_parameter(self):
        """ Get the unit of measure to interpret the `weight` field. By default, we considerer
        that weights are expressed in kilograms. Users can configure to express them in pounds
        by adding an ir.config_parameter record with "product.product_weight_in_lbs" as key
        and "1" as value.
        """
        get_param = self.env['ir.config_parameter'].sudo().get_param
        product_weight_in_lbs_param = get_param('product.weight_in_lbs')
        if product_weight_in_lbs_param == '1':
            return self.env.ref('uom.product_uom_lb', False) or self.env['uom.uom'].search([('measure_type', '=' , 'weight'), ('uom_type', '=', 'reference')], limit=1)
        else:
            return self.env.ref('uom.product_uom_kgm', False) or self.env['uom.uom'].search([('measure_type', '=' , 'weight'), ('uom_type', '=', 'reference')], limit=1)

    @api.model
    def _get_weight_uom_name_from_ir_config_parameter(self):
        return self._get_weight_uom_id_from_ir_config_parameter().display_name

    def _compute_weight_uom_name(self):
        for template in self:
            template.weight_uom_name = self._get_weight_uom_name_from_ir_config_parameter()

    @api.model
    def _get_volume_uom_name_from_ir_config_parameter(self):
        """ Get the unit of measure to interpret the `volume` field. By default, we consider
        that volumes are expressed in cubic meters. Users can configure to express them in cubic feet
        by adding an ir.config_parameter record with "product.volume_in_cubic_feet" as key
        and "1" as value.
        """
        get_param = self.env['ir.config_parameter'].sudo().get_param
        return "ft³" if get_param('product.volume_in_cubic_feet') == '1' else "m³"

    def _compute_volume_uom_name(self):
        for template in self:
            template.volume_uom_name = self._get_volume_uom_name_from_ir_config_parameter()

    def _set_weight(self):
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.product_variant_ids.weight = template.weight

    @api.depends('product_variant_ids.product_tmpl_id')
    def _compute_product_variant_count(self):
        for template in self:
            # do not pollute variants to be prefetched when counting variants
            template.product_variant_count = len(template.with_prefetch().product_variant_ids)

    @api.depends('product_variant_ids', 'product_variant_ids.default_code')
    def _compute_default_code(self):
        unique_variants = self.filtered(lambda template: len(template.product_variant_ids) == 1)
        for template in unique_variants:
            template.default_code = template.product_variant_ids.default_code
        for template in (self - unique_variants):
            template.default_code = False

    def _set_default_code(self):
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.product_variant_ids.default_code = template.default_code

    @api.depends('product_variant_ids', 'product_variant_ids.packaging_ids')
    def _compute_packaging_ids(self):
        for p in self:
            if len(p.product_variant_ids) == 1:
                p.packaging_ids = p.product_variant_ids.packaging_ids
            else:
                p.packaging_ids = False

    def _set_packaging_ids(self):
        for p in self:
            if len(p.product_variant_ids) == 1:
                p.product_variant_ids.packaging_ids = p.packaging_ids

    @api.constrains('uom_id', 'uom_po_id')
    def _check_uom(self):
        if any(template.uom_id and template.uom_po_id and template.uom_id.category_id != template.uom_po_id.category_id for template in self):
            raise ValidationError(_('The default Unit of Measure and the purchase Unit of Measure must be in the same category.'))
        return True

    @api.onchange('uom_id')
    def _onchange_uom_id(self):
        if self.uom_id:
            self.uom_po_id = self.uom_id.id

    @api.onchange('uom_po_id')
    def _onchange_uom(self):
        if self.uom_id and self.uom_po_id and self.uom_id.category_id != self.uom_po_id.category_id:
            self.uom_po_id = self.uom_id

    @api.onchange('type')
    def _onchange_type(self):
        # Do nothing but needed for inheritance
        return {}

    @api.model_create_multi
    def create(self, vals_list):
        ''' Store the initial standard price in order to be able to retrieve the cost of a product template for a given date'''
        templates = super(ProductTemplate, self).create(vals_list)
        if "create_product_product" not in self._context:
            templates._create_variant_ids()

        # This is needed to set given values to first variant after creation
        for template, vals in zip(templates, vals_list):
            related_vals = {}
            if vals.get('barcode'):
                related_vals['barcode'] = vals['barcode']
            if vals.get('default_code'):
                related_vals['default_code'] = vals['default_code']
            if vals.get('standard_price'):
                related_vals['standard_price'] = vals['standard_price']
            if vals.get('volume'):
                related_vals['volume'] = vals['volume']
            if vals.get('weight'):
                related_vals['weight'] = vals['weight']
            # Please do forward port
            if vals.get('packaging_ids'):
                related_vals['packaging_ids'] = vals['packaging_ids']
            if related_vals:
                template.write(related_vals)

        return templates

    def write(self, vals):
        uom = self.env['uom.uom'].browse(vals.get('uom_id')) or self.uom_id
        uom_po = self.env['uom.uom'].browse(vals.get('uom_po_id')) or self.uom_po_id
        if uom and uom_po and uom.category_id != uom_po.category_id:
            vals['uom_po_id'] = uom.id

        res = super(ProductTemplate, self).write(vals)
        if 'attribute_line_ids' in vals or (vals.get('active') and len(self.product_variant_ids) == 0):
            self._create_variant_ids()
        if 'active' in vals and not vals.get('active'):
            self.with_context(active_test=False).mapped('product_variant_ids').write({'active': vals.get('active')})
        if 'image_1920' in vals:
            self.env['product.product'].invalidate_cache(fnames=[
                'image_1920',
                'image_1024',
                'image_512',
                'image_256',
                'image_128',
                'can_image_1024_be_zoomed',
            ])
        return res

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        # TDE FIXME: should probably be copy_data
        self.ensure_one()
        if default is None:
            default = {}
        if 'name' not in default:
            default['name'] = _("%s (copy)") % self.name
        return super(ProductTemplate, self).copy(default=default)

    def name_get(self):
        # Prefetch the fields used by the `name_get`, so `browse` doesn't fetch other fields
        self.browse(self.ids).read(['name', 'default_code'])
        return [(template.id, '%s%s' % (template.default_code and '[%s] ' % template.default_code or '', template.name))
                for template in self]

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        # Only use the product.product heuristics if there is a search term and the domain
        # does not specify a match on `product.template` IDs.
        if not name or any(term[0] == 'id' for term in (args or [])):
            return super(ProductTemplate, self)._name_search(name=name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)

        Product = self.env['product.product']
        templates = self.browse([])
        while True:
            domain = templates and [('product_tmpl_id', 'not in', templates.ids)] or []
            args = args if args is not None else []
            products_ns = Product._name_search(name, args+domain, operator=operator, name_get_uid=name_get_uid)
            products = Product.browse([x[0] for x in products_ns])
            new_templates = products.mapped('product_tmpl_id')
            if new_templates & templates:
                """Product._name_search can bypass the domain we passed (search on supplier info).
                   If this happens, an infinite loop will occur."""
                break
            templates |= new_templates
            if (not products) or (limit and (len(templates) > limit)):
                break

        searched_ids = set(templates.ids)
        # some product.templates do not have product.products yet (dynamic variants configuration),
        # we need to add the base _name_search to the results
        # FIXME awa: this is really not performant at all but after discussing with the team
        # we don't see another way to do it
        tmpl_without_variant_ids = []
        if not limit or len(searched_ids) < limit:
            self._cr.execute("""
                SELECT t.id 
                FROM product_template t         
                LEFT JOIN product_product p ON t.id = p.product_tmpl_id AND p.active = TRUE
                WHERE t.active = TRUE AND p.id is NULL
            """)
            tmpl_without_variant_ids = [row[0] for row in self._cr.fetchall()]
        if tmpl_without_variant_ids:
            domain = expression.AND([args or [], [('id', 'in', tmpl_without_variant_ids)]])
            searched_ids |= set([template_id[0] for template_id in
                super(ProductTemplate, self)._name_search(
                    name,
                    args=domain,
                    operator=operator,
                    limit=limit,
                    name_get_uid=name_get_uid)])

        # re-apply product.template order + name_get
        return super(ProductTemplate, self)._name_search(
            '', args=[('id', 'in', list(searched_ids))],
            operator='ilike', limit=limit, name_get_uid=name_get_uid)

    def open_pricelist_rules(self):
        self.ensure_one()
        domain = ['|',
            ('product_tmpl_id', '=', self.id),
            ('product_id', 'in', self.product_variant_ids.ids)]
        return {
            'name': _('Price Rules'),
            'view_mode': 'tree,form',
            'views': [(self.env.ref('product.product_pricelist_item_tree_view_from_product').id, 'tree'), (False, 'form')],
            'res_model': 'product.pricelist.item',
            'type': 'ir.actions.act_window',
            'target': 'current',
            'domain': domain,
            'context': {
                'default_product_tmpl_id': self.id,
                'default_applied_on': '1_product',
                'product_without_variants': self.product_variant_count == 1,
            },
        }

    def price_compute(self, price_type, uom=False, currency=False, company=False):
        # TDE FIXME: delegate to template or not ? fields are reencoded here ...
        # compatibility about context keys used a bit everywhere in the code
        if not uom and self._context.get('uom'):
            uom = self.env['uom.uom'].browse(self._context['uom'])
        if not currency and self._context.get('currency'):
            currency = self.env['res.currency'].browse(self._context['currency'])

        templates = self
        if price_type == 'standard_price':
            # standard_price field can only be seen by users in base.group_user
            # Thus, in order to compute the sale price from the cost for users not in this group
            # We fetch the standard price as the superuser
            templates = self.with_context(force_company=company and company.id or self._context.get('force_company', self.env.company.id)).sudo()
        if not company:
            if self._context.get('force_company'):
                company = self.env['res.company'].browse(self._context['force_company'])
            else:
                company = self.env.company
        date = self.env.context.get('date') or fields.Date.today()

        prices = dict.fromkeys(self.ids, 0.0)
        for template in templates:
            prices[template.id] = template[price_type] or 0.0
            # yes, there can be attribute values for product template if it's not a variant YET
            # (see field product.attribute create_variant)
            if price_type == 'list_price' and self._context.get('current_attributes_price_extra'):
                # we have a list of price_extra that comes from the attribute values, we need to sum all that
                prices[template.id] += sum(self._context.get('current_attributes_price_extra'))

            if uom:
                prices[template.id] = template.uom_id._compute_price(prices[template.id], uom)

            # Convert from current user company currency to asked one
            # This is right cause a field cannot be in more than one currency
            if currency:
                prices[template.id] = template.currency_id._convert(prices[template.id], currency, company, date)

        return prices

    def _create_variant_ids(self):
        self.flush()
        Product = self.env["product.product"]

        variants_to_create = []
        variants_to_activate = Product
        variants_to_unlink = Product

        for tmpl_id in self:
            lines_without_no_variants = tmpl_id.valid_product_template_attribute_line_ids._without_no_variant_attributes()

            all_variants = tmpl_id.with_context(active_test=False).product_variant_ids.sorted(lambda p: (p.active, -p.id))

            current_variants_to_create = []
            current_variants_to_activate = Product

            # adding an attribute with only one value should not recreate product
            # write this attribute on every product to make sure we don't lose them
            single_value_lines = lines_without_no_variants.filtered(lambda ptal: len(ptal.product_template_value_ids._only_active()) == 1)
            if single_value_lines:
                for variant in all_variants:
                    combination = variant.product_template_attribute_value_ids | single_value_lines.product_template_value_ids._only_active()
                    # Do not add single value if the resulting combination would
                    # be invalid anyway.
                    if (
                        len(combination) == len(lines_without_no_variants) and
                        combination.attribute_line_id == lines_without_no_variants
                    ):
                        variant.product_template_attribute_value_ids = combination

            # Set containing existing `product.template.attribute.value` combination
            existing_variants = {
                variant.product_template_attribute_value_ids: variant for variant in all_variants
            }

            # Determine which product variants need to be created based on the attribute
            # configuration. If any attribute is set to generate variants dynamically, skip the
            # process.
            # Technical note: if there is no attribute, a variant is still created because
            # 'not any([])' and 'set([]) not in set([])' are True.
            if not tmpl_id.has_dynamic_attributes():
                # Iterator containing all possible `product.template.attribute.value` combination
                # The iterator is used to avoid MemoryError in case of a huge number of combination.
                all_combinations = itertools.product(*[
                    ptal.product_template_value_ids._only_active() for ptal in lines_without_no_variants
                ])
                # For each possible variant, create if it doesn't exist yet.
                for combination_tuple in all_combinations:
                    combination = self.env['product.template.attribute.value'].concat(*combination_tuple)
                    is_combination_possible = tmpl_id._is_combination_possible_by_config(combination, ignore_no_variant=True)
                    if not is_combination_possible:
                        continue
                    if combination in existing_variants:
                        current_variants_to_activate += existing_variants[combination]
                    else:
                        current_variants_to_create.append({
                            'product_tmpl_id': tmpl_id.id,
                            'product_template_attribute_value_ids': [(6, 0, combination.ids)],
                            'active': tmpl_id.active,
                        })
                        if len(current_variants_to_create) > 1000:
                            raise UserError(_(
                                'The number of variants to generate is too high. '
                                'You should either not generate variants for each combination or generate them on demand from the sales order. '
                                'To do so, open the form view of attributes and change the mode of *Create Variants*.'))
                variants_to_create += current_variants_to_create
                variants_to_activate += current_variants_to_activate

            else:
                for variant in existing_variants.values():
                    is_combination_possible = self._is_combination_possible_by_config(
                        combination=variant.product_template_attribute_value_ids,
                        ignore_no_variant=True,
                    )
                    if is_combination_possible:
                        current_variants_to_activate += variant
                variants_to_activate += current_variants_to_activate

            variants_to_unlink += all_variants - current_variants_to_activate

        if variants_to_activate:
            variants_to_activate.write({'active': True})
        if variants_to_create:
            Product.create(variants_to_create)
        if variants_to_unlink:
            variants_to_unlink._unlink_or_archive()
            # prevent change if exclusion deleted template by deleting last variant
            if self.exists() != self:
                raise UserError(_("This configuration of product attributes, values, and exclusions would lead to no possible variant. Please archive or delete your product directly if intended."))

        # prefetched o2m have to be reloaded (because of active_test)
        # (eg. product.template: product_variant_ids)
        # We can't rely on existing invalidate_cache because of the savepoint
        # in _unlink_or_archive.
        self.flush()
        self.invalidate_cache()
        return True

    def has_dynamic_attributes(self):
        """Return whether this `product.template` has at least one dynamic
        attribute.

        :return: True if at least one dynamic attribute, False otherwise
        :rtype: bool
        """
        self.ensure_one()
        return any(a.create_variant == 'dynamic' for a in self.valid_product_template_attribute_line_ids.attribute_id)

    @api.depends('attribute_line_ids.value_ids')
    def _compute_valid_product_template_attribute_line_ids(self):
        """A product template attribute line is considered valid if it has at
        least one possible value.

        Those with only one value are considered valid, even though they should
        not appear on the configurator itself (unless they have an is_custom
        value to input), indeed single value attributes can be used to filter
        products among others based on that attribute/value.
        """
        for record in self:
            record.valid_product_template_attribute_line_ids = record.attribute_line_ids.filtered(lambda ptal: ptal.value_ids)

    def _get_possible_variants(self, parent_combination=None):
        """Return the existing variants that are possible.

        For dynamic attributes, it will only return the variants that have been
        created already.

        If there are a lot of variants, this method might be slow. Even if there
        aren't too many variants, for performance reasons, do not call this
        method in a loop over the product templates.

        Therefore this method has a very restricted reasonable use case and you
        should strongly consider doing things differently if you consider using
        this method.

        :param parent_combination: combination from which `self` is an
            optional or accessory product.
        :type parent_combination: recordset `product.template.attribute.value`

        :return: the existing variants that are possible.
        :rtype: recordset of `product.product`
        """
        self.ensure_one()
        return self.product_variant_ids.filtered(lambda p: p._is_variant_possible(parent_combination))

    def _get_attribute_exclusions(self, parent_combination=None, parent_name=None):
        """Return the list of attribute exclusions of a product.

        :param parent_combination: the combination from which
            `self` is an optional or accessory product. Indeed exclusions
            rules on one product can concern another product.
        :type parent_combination: recordset `product.template.attribute.value`
        :param parent_name: the name of the parent product combination.
        :type parent_name: str

        :return: dict of exclusions
            - exclusions: from this product itself
            - parent_combination: ids of the given parent_combination
            - parent_exclusions: from the parent_combination
           - parent_product_name: the name of the parent product if any, used in the interface
               to explain why some combinations are not available.
               (e.g: Not available with Customizable Desk (Legs: Steel))
           - mapped_attribute_names: the name of every attribute values based on their id,
               used to explain in the interface why that combination is not available
               (e.g: Not available with Color: Black)
        """
        self.ensure_one()
        parent_combination = parent_combination or self.env['product.template.attribute.value']
        return {
            'exclusions': self._complete_inverse_exclusions(self._get_own_attribute_exclusions()),
            'parent_exclusions': self._get_parent_attribute_exclusions(parent_combination),
            'parent_combination': parent_combination.ids,
            'parent_product_name': parent_name,
            'mapped_attribute_names': self._get_mapped_attribute_names(parent_combination),
        }

    @api.model
    def _complete_inverse_exclusions(self, exclusions):
        """Will complete the dictionnary of exclusions with their respective inverse
        e.g: Black excludes XL and L
        -> XL excludes Black
        -> L excludes Black"""
        result = dict(exclusions)
        for key, value in exclusions.items():
            for exclusion in value:
                if exclusion in result and key not in result[exclusion]:
                    result[exclusion].append(key)
                else:
                    result[exclusion] = [key]

        return result

    def _get_own_attribute_exclusions(self):
        """Get exclusions coming from the current template.

        Dictionnary, each product template attribute value is a key, and for each of them
        the value is an array with the other ptav that they exclude (empty if no exclusion).
        """
        self.ensure_one()
        product_template_attribute_values = self.valid_product_template_attribute_line_ids.product_template_value_ids
        return {
            ptav.id: [
                value_id
                for filter_line in ptav.exclude_for.filtered(
                    lambda filter_line: filter_line.product_tmpl_id == self
                ) for value_id in filter_line.value_ids.ids
            ]
            for ptav in product_template_attribute_values
        }

    def _get_parent_attribute_exclusions(self, parent_combination):
        """Get exclusions coming from the parent combination.

        Dictionnary, each parent's ptav is a key, and for each of them the value is
        an array with the other ptav that are excluded because of the parent.
        """
        self.ensure_one()
        if not parent_combination:
            return {}

        result = {}
        for product_attribute_value in parent_combination:
            for filter_line in product_attribute_value.exclude_for.filtered(
                lambda filter_line: filter_line.product_tmpl_id == self
            ):
                # Some exclusions don't have attribute value. This means that the template is not
                # compatible with the parent combination. If such an exclusion is found, it means that all
                # attribute values are excluded.
                if filter_line.value_ids:
                    result[product_attribute_value.id] = filter_line.value_ids.ids
                else:
                    result[product_attribute_value.id] = filter_line.product_tmpl_id.mapped('attribute_line_ids.product_template_value_ids').ids

        return result

    def _get_mapped_attribute_names(self, parent_combination=None):
        """ The name of every attribute values based on their id,
        used to explain in the interface why that combination is not available
        (e.g: Not available with Color: Black).

        It contains both attribute value names from this product and from
        the parent combination if provided.
        """
        self.ensure_one()
        all_product_attribute_values = self.valid_product_template_attribute_line_ids.product_template_value_ids
        if parent_combination:
            all_product_attribute_values |= parent_combination

        return {
            attribute_value.id: attribute_value.display_name
            for attribute_value in all_product_attribute_values
        }

    def _is_combination_possible_by_config(self, combination, ignore_no_variant=False):
        """Return whether the given combination is possible according to the config of attributes on the template

        :param combination: the combination to check for possibility
        :type combination: recordset `product.template.attribute.value`

        :param ignore_no_variant: whether no_variant attributes should be ignored
        :type ignore_no_variant: bool

        :return: wether the given combination is possible according to the config of attributes on the template
        :rtype: bool
        """
        self.ensure_one()

        attribute_lines = self.valid_product_template_attribute_line_ids

        if ignore_no_variant:
            attribute_lines = attribute_lines._without_no_variant_attributes()

        if len(combination) != len(attribute_lines):
            # number of attribute values passed is different than the
            # configuration of attributes on the template
            return False

        if attribute_lines != combination.attribute_line_id:
            # combination has different attributes than the ones configured on the template
            return False

        if not (attribute_lines.product_template_value_ids._only_active() >= combination):
            # combination has different values than the ones configured on the template
            return False

        exclusions = self._get_own_attribute_exclusions()
        if exclusions:
            # exclude if the current value is in an exclusion,
            # and the value excluding it is also in the combination
            for ptav in combination:
                for exclusion in exclusions.get(ptav.id):
                    if exclusion in combination.ids:
                        return False

        return True

    def _is_combination_possible(self, combination, parent_combination=None, ignore_no_variant=False):
        """
        The combination is possible if it is not excluded by any rule
        coming from the current template, not excluded by any rule from the
        parent_combination (if given), and there should not be any archived
        variant with the exact same combination.

        If the template does not have any dynamic attribute, the combination
        is also not possible if the matching variant has been deleted.

        Moreover the attributes of the combination must excatly match the
        attributes allowed on the template.

        :param combination: the combination to check for possibility
        :type combination: recordset `product.template.attribute.value`

        :param ignore_no_variant: whether no_variant attributes should be ignored
        :type ignore_no_variant: bool

        :param parent_combination: combination from which `self` is an
            optional or accessory product.
        :type parent_combination: recordset `product.template.attribute.value`

        :return: whether the combination is possible
        :rtype: bool
        """
        self.ensure_one()

        if not self._is_combination_possible_by_config(combination, ignore_no_variant):
            return False

        variant = self._get_variant_for_combination(combination)

        if self.has_dynamic_attributes():
            if variant and not variant.active:
                # dynamic and the variant has been archived
                return False
        else:
            if not variant or not variant.active:
                # not dynamic, the variant has been archived or deleted
                return False

        parent_exclusions = self._get_parent_attribute_exclusions(parent_combination)
        if parent_exclusions:
            # parent_exclusion are mapped by ptav but here we don't need to know
            # where the exclusion comes from so we loop directly on the dict values
            for exclusions_values in parent_exclusions.values():
                for exclusion in exclusions_values:
                    if exclusion in combination.ids:
                        return False

        return True

    def _get_variant_for_combination(self, combination):
        """Get the variant matching the combination.

        All of the values in combination must be present in the variant, and the
        variant should not have more attributes. Ignore the attributes that are
        not supposed to create variants.

        :param combination: recordset of `product.template.attribute.value`

        :return: the variant if found, else empty
        :rtype: recordset `product.product`
        """
        self.ensure_one()
        filtered_combination = combination._without_no_variant_attributes()
        return self.env['product.product'].browse(self._get_variant_id_for_combination(filtered_combination))

    def _create_product_variant(self, combination, log_warning=False):
        """ Create if necessary and possible and return the product variant
        matching the given combination for this template.

        It is possible to create only if the template has dynamic attributes
        and the combination itself is possible.
        If we are in this case and the variant already exists but it is
        archived, it is activated instead of being created again.

        :param combination: the combination for which to get or create variant.
            The combination must contain all necessary attributes, including
            those of type no_variant. Indeed even though those attributes won't
            be included in the variant if newly created, they are needed when
            checking if the combination is possible.
        :type combination: recordset of `product.template.attribute.value`

        :param log_warning: whether a warning should be logged on fail
        :type log_warning: bool

        :return: the product variant matching the combination or none
        :rtype: recordset of `product.product`
        """
        self.ensure_one()

        Product = self.env['product.product']

        product_variant = self._get_variant_for_combination(combination)
        if product_variant:
            if not product_variant.active and self.has_dynamic_attributes() and self._is_combination_possible(combination):
                product_variant.active = True
            return product_variant

        if not self.has_dynamic_attributes():
            if log_warning:
                _logger.warning('The user #%s tried to create a variant for the non-dynamic product %s.' % (self.env.user.id, self.id))
            return Product

        if not self._is_combination_possible(combination):
            if log_warning:
                _logger.warning('The user #%s tried to create an invalid variant for the product %s.' % (self.env.user.id, self.id))
            return Product

        return Product.sudo().create({
            'product_tmpl_id': self.id,
            'product_template_attribute_value_ids': [(6, 0, combination._without_no_variant_attributes().ids)]
        })

    def _create_first_product_variant(self, log_warning=False):
        """Create if necessary and possible and return the first product
        variant for this template.

        :param log_warning: whether a warning should be logged on fail
        :type log_warning: bool

        :return: the first product variant or none
        :rtype: recordset of `product.product`
        """
        return self._create_product_variant(self._get_first_possible_combination(), log_warning)

    @tools.ormcache('self.id', 'frozenset(filtered_combination.ids)')
    def _get_variant_id_for_combination(self, filtered_combination):
        """See `_get_variant_for_combination`. This method returns an ID
        so it can be cached.

        Use sudo because the same result should be cached for all users.
        """
        self.ensure_one()
        domain = [('product_tmpl_id', '=', self.id)]
        combination_indices_ids = filtered_combination._ids2str()

        if combination_indices_ids:
            domain = expression.AND([domain, [('combination_indices', '=', combination_indices_ids)]])
        else:
            domain = expression.AND([domain, [('combination_indices', 'in', ['', False])]])

        return self.env['product.product'].sudo().with_context(active_test=False).search(domain, order='active DESC', limit=1).id

    @tools.ormcache('self.id')
    def _get_first_possible_variant_id(self):
        """See `_create_first_product_variant`. This method returns an ID
        so it can be cached."""
        self.ensure_one()
        return self._create_first_product_variant().id

    def _get_first_possible_combination(self, parent_combination=None, necessary_values=None):
        """See `_get_possible_combinations` (one iteration).

        This method return the same result (empty recordset) if no
        combination is possible at all which would be considered a negative
        result, or if there are no attribute lines on the template in which
        case the "empty combination" is actually a possible combination.
        Therefore the result of this method when empty should be tested
        with `_is_combination_possible` if it's important to know if the
        resulting empty combination is actually possible or not.
        """
        return next(self._get_possible_combinations(parent_combination, necessary_values), self.env['product.template.attribute.value'])

    def _cartesian_product(self, product_template_attribute_values_per_line, parent_combination):
        """
        Generate all possible combination for attributes values (aka cartesian product).
        It is equivalent to itertools.product except it skips invalid partial combinations before they are complete.

        Imagine the cartesian product of 'A', 'CD' and range(1_000_000) and let's say that 'A' and 'C' are incompatible.
        If you use itertools.product or any normal cartesian product, you'll need to filter out of the final result
        the 1_000_000 combinations that start with 'A' and 'C' . Instead, This implementation will test if 'A' and 'C' are
        compatible before even considering range(1_000_000), skip it and and continue with combinations that start
        with 'A' and 'D'.

        It's necessary for performance reason because filtering out invalid combinations from standard Cartesian product
        can be extremely slow

        :param product_template_attribute_values_per_line: the values we want all the possibles combinations of.
        One list of values by attribute line
        :return: a generator of product template attribute value
        """
        if not product_template_attribute_values_per_line:
            return

        all_exclusions = {self.env['product.template.attribute.value'].browse(k):
                          self.env['product.template.attribute.value'].browse(v) for k, v in
                          self._get_own_attribute_exclusions().items()}
        # The following dict uses product template attribute values as keys
        # 0 means the value is acceptable, greater than 0 means it's rejected, it cannot be negative
        # Bear in mind that several values can reject the same value and the latter can only be included in the
        #  considered combination if no value rejects it.
        # This dictionary counts how many times each value is rejected.
        # Each time a value is included in the considered combination, the values it rejects are incremented
        # When a value is discarded from the considered combination, the values it rejects are decremented
        current_exclusions = defaultdict(int)
        for exclusion in self._get_parent_attribute_exclusions(parent_combination):
            current_exclusions[self.env['product.template.attribute.value'].browse(exclusion)] += 1
        partial_combination = self.env['product.template.attribute.value']

        # The following list reflects product_template_attribute_values_per_line
        # For each line, instead of a list of values, it contains the index of the selected value
        # -1 means no value has been picked for the line in the current (partial) combination
        value_index_per_line = [-1] * len(product_template_attribute_values_per_line)
        # determines which line line we're working on
        line_index = 0

        while True:
            current_line_values = product_template_attribute_values_per_line[line_index]
            current_ptav_index = value_index_per_line[line_index]
            current_ptav = current_line_values[current_ptav_index]

            # removing exclusions from current_ptav as we're removing it from partial_combination
            if current_ptav_index >= 0:
                for ptav_to_include_back in all_exclusions[current_ptav]:
                    current_exclusions[ptav_to_include_back] -= 1
                partial_combination -= current_ptav

            if current_ptav_index < len(current_line_values) - 1:
                # go to next value of current line
                value_index_per_line[line_index] += 1
                current_line_values = product_template_attribute_values_per_line[line_index]
                current_ptav_index = value_index_per_line[line_index]
                current_ptav = current_line_values[current_ptav_index]
            elif line_index != 0:
                # reset current line, and then go to previous line
                value_index_per_line[line_index] = - 1
                line_index -= 1
                continue
            else:
                # we're done if we must reset first line
                break

            # adding exclusions from current_ptav as we're incorporating it in partial_combination
            for ptav_to_exclude in all_exclusions[current_ptav]:
                current_exclusions[ptav_to_exclude] += 1
            partial_combination += current_ptav

            # test if included values excludes current value or if current value exclude included values
            if current_exclusions[current_ptav] or \
                    any(intersection in partial_combination for intersection in all_exclusions[current_ptav]):
                continue

            if line_index == len(product_template_attribute_values_per_line) - 1:
                # submit combination if we're on the last line
                yield partial_combination
            else:
                # else we go to the next line
                line_index += 1

    def _get_possible_combinations(self, parent_combination=None, necessary_values=None):
        """Generator returning combinations that are possible, following the
        sequence of attributes and values.

        See `_is_combination_possible` for what is a possible combination.

        When encountering an impossible combination, try to change the value
        of attributes by starting with the further regarding their sequences.

        Ignore attributes that have no values.

        :param parent_combination: combination from which `self` is an
            optional or accessory product.
        :type parent_combination: recordset `product.template.attribute.value`

        :param necessary_values: values that must be in the returned combination
        :type necessary_values: recordset of `product.template.attribute.value`

        :return: the possible combinations
        :rtype: generator of recordset of `product.template.attribute.value`
        """
        self.ensure_one()

        if not self.active:
            return _("The product template is archived so no combination is possible.")

        necessary_values = necessary_values or self.env['product.template.attribute.value']
        necessary_attribute_lines = necessary_values.mapped('attribute_line_id')
        attribute_lines = self.valid_product_template_attribute_line_ids.filtered(lambda ptal: ptal not in necessary_attribute_lines)

        if not attribute_lines and self._is_combination_possible(necessary_values, parent_combination):
            yield necessary_values

        product_template_attribute_values_per_line = [
            ptal.product_template_value_ids._only_active()
            for ptal in attribute_lines
        ]

        for partial_combination in self._cartesian_product(product_template_attribute_values_per_line, parent_combination):
            combination = partial_combination + necessary_values
            if self._is_combination_possible(combination, parent_combination):
                yield combination

        return _("There are no remaining possible combination.")

    def _get_closest_possible_combination(self, combination):
        """See `_get_closest_possible_combinations` (one iteration).

        This method return the same result (empty recordset) if no
        combination is possible at all which would be considered a negative
        result, or if there are no attribute lines on the template in which
        case the "empty combination" is actually a possible combination.
        Therefore the result of this method when empty should be tested
        with `_is_combination_possible` if it's important to know if the
        resulting empty combination is actually possible or not.
        """
        return next(self._get_closest_possible_combinations(combination), self.env['product.template.attribute.value'])

    def _get_closest_possible_combinations(self, combination):
        """Generator returning the possible combinations that are the closest to
        the given combination.

        If the given combination is incomplete, try to complete it.

        If the given combination is invalid, try to remove values from it before
        completing it.

        :param combination: the values to include if they are possible
        :type combination: recordset `product.template.attribute.value`

        :return: the possible combinations that are including as much
            elements as possible from the given combination.
        :rtype: generator of recordset of product.template.attribute.value
        """
        while True:
            res = self._get_possible_combinations(necessary_values=combination)
            try:
                # If there is at least one result for the given combination
                # we consider that combination set, and we yield all the
                # possible combinations for it.
                yield(next(res))
                for cur in res:
                    yield(cur)
                return _("There are no remaining closest combination.")
            except StopIteration:
                # There are no results for the given combination, we try to
                # progressively remove values from it.
                if not combination:
                    return _("There are no possible combination.")
                combination = combination[:-1]

    def _get_current_company(self, **kwargs):
        """Get the most appropriate company for this product.

        If the company is set on the product, directly return it. Otherwise,
        fallback to a contextual company.

        :param kwargs: kwargs forwarded to the fallback method.

        :return: the most appropriate company for this product
        :rtype: recordset of one `res.company`
        """
        self.ensure_one()
        return self.company_id or self._get_current_company_fallback(**kwargs)

    def _get_current_company_fallback(self, **kwargs):
        """Fallback to get the most appropriate company for this product.

        This should only be called from `_get_current_company` but is defined
        separately to allow override.

        The final fallback will be the current user's company.

        :return: the fallback company for this product
        :rtype: recordset of one `res.company`
        """
        self.ensure_one()
        return self.env.company

    def get_single_product_variant(self):
        """ Method used by the product configurator to check if the product is configurable or not.

        We need to open the product configurator if the product:
        - is configurable (see has_configurable_attributes)
        - has optional products (method is extended in sale to return optional products info)
        """
        self.ensure_one()
        if self.product_variant_count == 1 and not self.has_configurable_attributes:
            return {
                'product_id': self.product_variant_id.id,
            }
        return {}

    @api.model
    def get_empty_list_help(self, help):
        self = self.with_context(
            empty_list_help_document_name=_("product"),
        )
        return super(ProductTemplate, self).get_empty_list_help(help)

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Products'),
            'template': '/product/static/xls/product_template.xls'
        }]

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class ResCompany(models.Model):
    _inherit = "res.company"

    @api.model
    def create(self, vals):
        new_company = super(ResCompany, self).create(vals)
        ProductPricelist = self.env['product.pricelist']
        pricelist = ProductPricelist.search([('currency_id', '=', new_company.currency_id.id), ('company_id', '=', False)], limit=1)
        if not pricelist:
            params = {'currency': new_company.currency_id.name}
            pricelist = ProductPricelist.create({
                'name': _("Default %(currency)s pricelist") %  params,
                'currency_id': new_company.currency_id.id,
            })
        field = self.env['ir.model.fields']._get('res.partner', 'property_product_pricelist')
        self.env['ir.property'].sudo().create({
            'name': 'property_product_pricelist',
            'value_reference': 'product.pricelist,%s' % pricelist.id,
            'fields_id': field.id,
            'company_id': new_company.id,
        })
        return new_company

    def write(self, values):
        # When we modify the currency of the company, we reflect the change on the list0 pricelist, if
        # that pricelist is not used by another company. Otherwise, we create a new pricelist for the
        # given currency.
        ProductPricelist = self.env['product.pricelist']
        currency_id = values.get('currency_id')
        main_pricelist = self.env.ref('product.list0', False)
        if currency_id and main_pricelist:
            nb_companies = self.search_count([])
            for company in self:
                existing_pricelist = ProductPricelist.search(
                    [('company_id', 'in', (False, company.id)),
                     ('currency_id', 'in', (currency_id, company.currency_id.id))])
                if existing_pricelist and any(currency_id == x.currency_id.id for x in existing_pricelist):
                    continue
                if currency_id == company.currency_id.id:
                    continue
                currency_match = main_pricelist.currency_id == company.currency_id
                company_match = (main_pricelist.company_id == company or
                                 (main_pricelist.company_id.id is False and nb_companies == 1))
                if currency_match and company_match:
                    main_pricelist.write({'currency_id': currency_id})
                else:
                    params = {'currency': self.env['res.currency'].browse(currency_id).name}
                    pricelist = ProductPricelist.create({
                        'name': _("Default %(currency)s pricelist") %  params,
                        'currency_id': currency_id,
                    })
                    field = self.env['ir.model.fields'].search([('model', '=', 'res.partner'), ('name', '=', 'property_product_pricelist')])
                    self.env['ir.property'].sudo().create({
                        'name': 'property_product_pricelist',
                        'company_id': company.id,
                        'value_reference': 'product.pricelist,%s' % pricelist.id,
                        'fields_id': field.id
                    })
        return super(ResCompany, self).write(values)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_discount_per_so_line = fields.Boolean("Discounts", implied_group='product.group_discount_per_so_line')
    group_uom = fields.Boolean("Units of Measure", implied_group='uom.group_uom')
    group_product_variant = fields.Boolean("Variants", implied_group='product.group_product_variant')
    module_sale_product_configurator = fields.Boolean("Product Configurator")
    module_sale_product_matrix = fields.Boolean("Sales Grid Entry")
    group_stock_packaging = fields.Boolean('Product Packagings',
        implied_group='product.group_stock_packaging')
    group_product_pricelist = fields.Boolean("Pricelists",
        implied_group='product.group_product_pricelist')
    group_sale_pricelist = fields.Boolean("Advanced Pricelists",
        implied_group='product.group_sale_pricelist',
        help="""Allows to manage different prices based on rules per category of customers.
                Example: 10% for retailers, promotion of 5 EUR on this product, etc.""")
    product_pricelist_setting = fields.Selection([
            ('basic', 'Multiple prices per product'),
            ('advanced', 'Advanced price rules (discounts, formulas)')
            ], default='basic', string="Pricelists Method", config_parameter='product.product_pricelist_setting',
            help="Multiple prices: Pricelists with fixed price rules by product,\nAdvanced rules: enables advanced price rules for pricelists.")
    product_weight_in_lbs = fields.Selection([
        ('0', 'Kilogram'),
        ('1', 'Pound'),
    ], 'Weight unit of measure', config_parameter='product.weight_in_lbs', default='0')
    product_volume_volume_in_cubic_feet = fields.Selection([
        ('0', 'Cubic Meters'),
        ('1', 'Cubic Feet'),
    ], 'Volume unit of measure', config_parameter='product.volume_in_cubic_feet', default='0')

    @api.onchange('group_product_variant')
    def _onchange_group_product_variant(self):
        """The product Configurator requires the product variants activated.
        If the user disables the product variants -> disable the product configurator as well"""
        if self.module_sale_product_configurator and not self.group_product_variant:
            self.module_sale_product_configurator = False
        if self.module_sale_product_matrix and not self.group_product_variant:
            self.module_sale_product_matrix = False

    @api.onchange('module_sale_product_configurator')
    def _onchange_module_sale_product_configurator(self):
        """The product Configurator requires the product variants activated
        If the user enables the product configurator -> enable the product variants as well"""
        if self.module_sale_product_configurator and not self.group_product_variant:
            self.group_product_variant = True

    @api.onchange('group_multi_currency')
    def _onchange_group_multi_currency(self):
        if self.group_multi_currency:
            self.group_product_pricelist = True

    @api.onchange('group_product_pricelist')
    def _onchange_group_sale_pricelist(self):
        if not self.group_product_pricelist and self.group_sale_pricelist:
            self.group_sale_pricelist = False

    @api.onchange('product_pricelist_setting')
    def _onchange_product_pricelist_setting(self):
        if self.product_pricelist_setting == 'basic':
            self.group_sale_pricelist = False
        else:
            self.group_sale_pricelist = True

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        if not self.group_discount_per_so_line:
            pl = self.env['product.pricelist'].search([('discount_policy', '=', 'without_discount')])
            pl.write({'discount_policy': 'with_discount'})

    @api.onchange('module_sale_product_matrix')
    def _onchange_module_module_sale_product_matrix(self):
        """The product Grid Configurator requires the product Configurator activated
        If the user enables the Grid Configurator -> enable the product Configurator as well"""
        if self.module_sale_product_matrix and not self.module_sale_product_configurator:
            self.module_sale_product_configurator = True

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class Partner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    # NOT A REAL PROPERTY !!!!
    property_product_pricelist = fields.Many2one(
        'product.pricelist', 'Pricelist', compute='_compute_product_pricelist',
        inverse="_inverse_product_pricelist", company_dependent=False,
        domain=lambda self: [('company_id', 'in', (self.env.company.id, False))],
        help="This pricelist will be used, instead of the default one, for sales to the current partner")

    @api.depends('country_id')
    @api.depends_context('force_company')
    def _compute_product_pricelist(self):
        company = self.env.context.get('force_company', False)
        res = self.env['product.pricelist']._get_partner_pricelist_multi(self.ids, company_id=company)
        for p in self:
            p.property_product_pricelist = res.get(p.id)

    def _inverse_product_pricelist(self):
        for partner in self:
            pls = self.env['product.pricelist'].search(
                [('country_group_ids.country_ids.code', '=', partner.country_id and partner.country_id.code or False)],
                limit=1
            )
            default_for_country = pls and pls[0]
            actual = self.env['ir.property'].get('property_product_pricelist', 'res.partner', 'res.partner,%s' % partner.id)
            # update at each change country, and so erase old pricelist
            if partner.property_product_pricelist or (actual and default_for_country and default_for_country.id != actual.id):
                # keep the company of the current user before sudo
                self.env['ir.property'].with_context(force_company=self._context.get('force_company', self.env.company.id)).sudo().set_multi(
                    'property_product_pricelist',
                    partner._name,
                    {partner.id: partner.property_product_pricelist or default_for_country.id},
                    default_value=default_for_country.id
                )

    def _commercial_fields(self):
        return super(Partner, self)._commercial_fields() + ['property_product_pricelist']

```

## File: models\uom_uom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class UoM(models.Model):
    _inherit = 'uom.uom'

    @api.onchange('rounding')
    def _onchange_rounding(self):
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        if self.rounding < 1.0 / 10.0**precision:
            return {'warning': {
                'title': _('Warning!'),
                'message': _(
                    "This rounding precision is higher than the Decimal Accuracy"
                    " (%s digits).\nThis may cause inconsistencies in computations.\n"
                    "Please set a precision between %s and 1."
                ) % (precision, 1.0 / 10.0**precision),
            }}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# flake8: noqa: F401

from . import res_config_settings
from . import decimal_precision
from . import uom_uom

# don't try to be a good boy and sort imports alphabetically.
# `product.template` should be initialised before `product.product`
from . import product_template
from . import product

from . import product_attribute
from . import product_pricelist
from . import res_company
from . import res_partner

```

## File: report\product_packaging.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <template id="report_packagingbarcode">
            <t t-call="web.basic_layout">
                <div class="page">
                    <t t-foreach="docs" t-as="packaging">
                        <div class="col-4" style="padding:0;">
                            <table class="table table-condensed" style="border-bottom: 0px solid white !important;width: 3in;">
                                <tr>
                                    <th style="text-align: left;">
                                        <strong t-field="packaging.name"/>
                                    </th>
                                </tr>
                                <tr>
                                    <td>
                                        <strong t-field="packaging.product_id.display_name"/>
                                    </td>
                                </tr>
                                <tr>
                                    <td>
                                        <div class="o_row">
                                            <strong>Qty: </strong>
                                            <strong t-field="packaging.qty"/>
                                            <strong t-field="packaging.product_uom_id" groups="uom.group_uom"/>
                                        </div>
                                    </td>
                                </tr>
                                  <t t-if="packaging.barcode">
                                    <tr>
                                    <td style="text-align: center; vertical-align: middle;" class="col-5">
                                        <img alt="Barcode" t-if="len(packaging.barcode) == 13" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN13', packaging.barcode, 600, 150)" style="width:100%;height:20%;"/>
                                        <img alt="Barcode" t-elif="len(packaging.barcode) == 8" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN8', packaging.barcode, 600, 150)" style="width:100%;height:20%;"/>
                                        <img alt="Barcode" t-else="" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', packaging.barcode, 600, 150)" style="width:100%;height:20%;"/>
                                        <span t-field="packaging.barcode"/>
                                    </td>
                                </tr>
                              </t>
                            </table>
                        </div>
                    </t>
                </div>
            </t>
        </template>
    </data>
</odoo>

```

## File: report\product_pricelist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools import float_round


class report_product_pricelist(models.AbstractModel):
    _name = 'report.product.report_pricelist'
    _description = 'Product Price List Report'

    @api.model
    def _get_report_values(self, docids, data=None):
        data = data if data is not None else {}
        pricelist = self.env['product.pricelist'].browse(data.get('form', {}).get('price_list', False))
        products = self.env['product.product'].browse(data.get('ids', data.get('active_ids')))
        quantities = self._get_quantity(data)
        return {
            'doc_ids': data.get('ids', data.get('active_ids')),
            'doc_model': 'product.pricelist',
            'docs': products,
            'data': dict(
                data,
                pricelist=pricelist,
                quantities=quantities,
                categories_data=self._get_categories(pricelist, products, quantities)
            ),
        }

    def _get_quantity(self, data):
        form = data and data.get('form') or {}
        return sorted([form[key] for key in form if key.startswith('qty') and form[key]])

    def _get_categories(self, pricelist, products, quantities):
        categ_data = []
        categories = self.env['product.category']
        for product in products:
            categories |= product.categ_id

        for category in categories:
            categ_products = products.filtered(lambda product: product.categ_id == category)
            prices = {}
            for categ_product in categ_products:
                prices[categ_product.id] = dict.fromkeys(quantities, 0.0)
                for quantity in quantities:
                    prices[categ_product.id][quantity] = self._get_price(pricelist, categ_product, quantity)
            categ_data.append({
                'category': category,
                'products': categ_products,
                'prices': prices,
            })
        return categ_data

    def _get_price(self, pricelist, product, qty):
        sale_price_digits = self.env['decimal.precision'].precision_get('Product Price')
        price = pricelist.get_product_price(product, qty, False)
        if not price:
            price = product.list_price
        return float_round(price, precision_digits=sale_price_digits)

```

## File: report\product_pricelist_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_pricelist">
    <t t-call="web.html_container">
        <t t-call="web.internal_layout">
            <div class="page">
                <h2>Price List</h2>

                <div class="row mt32 mb32">
                    <div class="col-3">
                        <strong>Price List Name</strong>:<br/>
                        <span t-esc="data['pricelist'].name"/>
                    </div>
                    <div class="col-3">
                        <strong>Currency</strong>:<br/>
                        <span t-esc="data['pricelist'].currency_id.name"/>
                    </div>
                    <div class="col-3">
                        <strong>Print date</strong>:<br/>
                        <t t-esc="datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')"/>
                    </div>
                </div>

                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th>
                                <strong>Description</strong>
                            </th>
                            <t t-foreach="data['quantities']" t-as="quantity">
                                <th><strong t-esc="'%s units' % quantity"/></th>
                            </t>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="data['categories_data']" t-as="categ_data">
                            <tr>
                                <td colspan="100">
                                    <strong t-esc="categ_data['category'].name"/>
                                </td>
                            </tr>
                            <tr t-foreach="categ_data['products']" t-as="product">
                                <td>
                                    <t t-if="product.code">
                                        [<span t-esc="product.code"/>]
                                    </t> 
                                    <span t-esc="product.name"/>
                                    <span t-foreach="product.product_template_attribute_value_ids" t-as="attribute_value">
                                        <span t-if="attribute_value_first">-</span>
                                        <span t-if="not attribute_value_last" t-esc="attribute_value.name+','"/>
                                        <span t-else="" t-esc="attribute_value.name"/>
                                    </span>
                                </td>
                                <t t-foreach="data['quantities']" t-as="quantity">
                                    <td><strong t-esc="categ_data['prices'][product.id][quantity]"
                                                t-options="{
                                                    'widget': 'float',
                                                    'decimal_precision': 'Product Price'}"/>
                                    </td>
                                </t>
                            </tr>
                        </t>
                    </tbody>
                </table>
            </div>
        </t>
    </t>
</template>
</odoo>

```

## File: report\product_product_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <template id="report_simple_label">
            <div style="width: 32%; display: inline-table; height:14rem;">
                <table class="table table-bordered mb-0" style="border: 2px solid black;">
                    <tr>
                        <th class="table-active text-left" style="height: 4rem;">
                            <strong t-field="product.display_name"/>
                        </th>
                    </tr>
                    <tr>
                        <td style="height: 2rem">
                            <strong>Price:</strong>
                            <strong t-field="product.lst_price" t-options="{'widget': 'monetary', 'display_currency': product.company_id.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-center align-middle" style="height: 6rem">
                            <t t-if="product.barcode">
                                <img alt="Barcode" t-if="len(product.barcode) == 13" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN13', quote_plus(product.barcode or ''), 600, 150)" style="width:100%;height::4rem;"/>
                                <img alt="Barcode" t-elif="len(product.barcode) == 8" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN8', quote_plus(product.barcode or ''), 600, 150)" style="width:100%;height::4rem;"/>
                                <img alt="Barcode" t-else="" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', quote_plus(product.barcode or ''), 600, 150)" style="width:100%;height::4rem;"/>
                                <span t-field="product.barcode"/>
                            </t>
                            <t t-else=""><span class="text-muted">No barcode available</span></t>
                        </td>
                    </tr>
                </table>
            </div>
        </template>

        <template id="report_productlabel">
            <t t-call="web.basic_layout">
                <div class="page">
                    <t t-foreach="docs" t-as="product">
                        <t t-call="product.report_simple_label">
                            <t t-set="product" t-value="product"/>
                        </t>
                    </t>
                </div>
            </t>
        </template>

        <template id="report_simple_barcode">
            <div style="width: 32%; display: inline-table; height: 10rem;">
                <table class="table table-bordered mb-0" style="border: 2px solid black;">
                    <tr>
                        <th class="table-active text-left" style="height: 4rem;">
                            <strong t-field="product.display_name"/>
                        </th>
                    </tr>
                    <tr>
                        <td class="text-center align-middle" style="height: 6rem;">
                            <t t-if="product.barcode">
                                <img alt="Barcode" t-if="len(product.barcode) == 13" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN13', quote_plus(product.barcode or ''), 600, 150)" style="width:100%;height:4rem;"/>
                                <img alt="Barcode" t-elif="len(product.barcode) == 8" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN8', quote_plus(product.barcode or ''), 600, 150)" style="width:100%;height:4rem;"/>
                                <img alt="Barcode" t-else="" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', quote_plus(product.barcode or ''), 600, 150)" style="width:100%;height:4rem"/>
                                <span t-field="product.barcode"/>
                            </t>
                            <t t-else=""><span class="text-muted">No barcode available</span></t>
                        </td>
                    </tr>
                </table>
            </div>
        </template>

        <template id="report_productbarcode">
            <t t-call="web.basic_layout">
                <div class="page">
                    <t t-foreach="docs" t-as="product">
                        <t t-call="product.report_simple_barcode">
                            <t t-set="product" t-value="product"/>
                        </t>
                    </t>
                </div>
            </t>
        </template>
    </data>
</odoo>

```

## File: report\product_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <report
            id="report_product_label"
            string="Product Label (PDF)"
            model="product.product"
            report_type="qweb-pdf"
            name="product.report_productlabel"
            file="product.report_productlabel"
            print_report_name="'Products Labels - %s' % (object.name)"
        />

        <report
            id="report_product_template_label"
            string="Product Label (PDF)"
            model="product.template"
            report_type="qweb-pdf"
            name="product.report_producttemplatelabel"
            file="product.report_producttemplatelabel"
            print_report_name="'Products Labels - %s' % (object.name)"
        />

        <report
            id="report_product_product_barcode"
            string="Product Barcode (PDF)"
            model="product.product"
            report_type="qweb-pdf"
            name="product.report_productbarcode"
            file="product.report_productbarcode"
            print_report_name="'Products barcode - %s' % (object.name)"
        />

        <report
            id="report_product_template_barcode"
            string="Product Barcode (PDF)"
            model="product.template"
            report_type="qweb-pdf"
            name="product.report_producttemplatebarcode"
            file="product.report_producttemplatebarcode"
            print_report_name="'Products barcode - %s' % (object.name)"
        />

        <report
            id="report_product_packaging"
            string="Product Packaging (PDF)"
            model="product.packaging"
            report_type="qweb-pdf"
            name="product.report_packagingbarcode"
            file="product.report_packagingbarcode"
            print_report_name="'Products packaging - %s' % (object.name)"/>

        <report
            id="action_report_pricelist"
            string="Pricelist"
            model="product.product"
            report_type="qweb-pdf"
            name="product.report_pricelist"
            file="product.report_pricelist"
            menu="False"/>
    </data>
</odoo>

```

## File: report\product_template_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_producttemplatelabel">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-foreach="docs" t-as="template">
                <t t-foreach="template.product_variant_ids" t-as="product">
                    <t t-call="product.report_simple_label">
                        <t t-set="product" t-value="product"/>
                    </t>
                </t>
            </t>
        </div>
    </t>
</template>
<template id="report_producttemplatebarcode">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-foreach="docs" t-as="template">
                <t t-foreach="template.product_variant_ids" t-as="product">
                    <t t-call="product.report_simple_barcode">
                        <t t-set="product" t-value="product"/>
                    </t>
                </t>
            </t>
        </div>
    </t>
</template>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import product_pricelist

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_category_user,product.category.user,model_product_category,base.group_user,1,0,0,0
access_product_template_user,product.template.user,model_product_template,base.group_user,1,0,0,0
access_product_packaging_user,product.packaging.user,model_product_packaging,base.group_user,1,0,0,0
access_product_supplierinfo_user,product.supplierinfo.user,model_product_supplierinfo,base.group_user,1,0,0,0
access_product_pricelist_user,product.pricelist.user,model_product_pricelist,base.group_user,1,0,0,0
access_product_pricelist_item_user,product.pricelist.item.user,model_product_pricelist_item,base.group_user,1,0,0,0
access_product_pricelist_partner_manager,product.pricelist partner manager,model_product_pricelist,base.group_partner_manager,1,0,0,0
access_product_product_employee,product.product employee,model_product_product,base.group_user,1,0,0,0
access_product_attribute,product.attribute,model_product_attribute,base.group_user,1,0,0,0
access_product_attribute_value,product.attribute value,model_product_attribute_value,base.group_user,1,0,0,0
access_product_product_attribute,product.template.attribute value,model_product_template_attribute_value,base.group_user,1,0,0,0
access_product_template_attribute_exclusion,product..template.attribute exclusion,model_product_template_attribute_exclusion,base.group_user,1,0,0,0
access_product_template_attribute_line,product.template.attribute line,model_product_template_attribute_line,base.group_user,1,0,0,0
access_product_category_manager,product.category.manager,model_product_category,base.group_system,1,1,1,1
access_product_template_manager,product.template.manager,model_product_template,base.group_system,1,1,1,1
access_product_packaging_manager,product.packaging.manager,model_product_packaging,base.group_system,1,1,1,1
access_product_supplierinfo_manager,product.supplierinfo.manager,model_product_supplierinfo,base.group_system,1,1,1,1
access_product_pricelist_manager,product.pricelist.manager,model_product_pricelist,base.group_system,1,1,1,1
access_product_pricelist_item_manager,product.pricelist.item.manager,model_product_pricelist_item,base.group_system,1,1,1,1
access_product_product_manager,product.product manager,model_product_product,base.group_system,1,1,1,1
access_product_attribute_manager,product.attribute manager,model_product_attribute,base.group_system,1,1,1,1
access_product_attribute_value_manager,product.attribute value manager,model_product_attribute_value,base.group_system,1,1,1,1
access_product_product_attribute_manager,product.template.attribute value manager,model_product_template_attribute_value,base.group_system,1,1,1,1
access_product_template_attribute_exclusion_manager,product.template.attribute exclusion manager,model_product_template_attribute_exclusion,base.group_system,1,1,1,1
access_product_template_attribute_line_manager,product.template.attribute line manager,model_product_template_attribute_line,base.group_system,1,1,1,1

```

## File: security\product_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">

    <record id="group_product_pricelist" model="res.groups">
        <field name="name">Basic Pricelists</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_sale_pricelist" model="res.groups">
        <field name="name">Advanced Pricelists</field>
        <field name="category_id" ref="base.module_category_hidden"/>
        <field name="implied_ids" eval="[(4, ref('product.group_product_pricelist'))]"/>
    </record>

    <record id="group_stock_packaging" model="res.groups">
        <field name="name">Manage Product Packaging</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_product_variant" model="res.groups">
        <field name="name">Manage Product Variants</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_discount_per_so_line" model="res.groups">
        <field name="name">Discount on lines</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

</data>
<data noupdate="1">

    <record id="product_comp_rule" model="ir.rule">
        <field name="name" >Product multi-company</field>
        <field name="model_id" ref="model_product_template"/>
        <field name="global" eval="True"/>
        <field name="domain_force"> ['|', ('company_id', 'in', company_ids), ('company_id', '=', False)]</field>
    </record>

    <record model="ir.rule" id="product_pricelist_comp_rule">
        <field name="name">product pricelist company rule</field>
        <field name="model_id" ref="model_product_pricelist"/>
        <field name="global" eval="True"/>
        <field name="domain_force"> ['|', ('company_id', 'in', company_ids), ('company_id', '=', False)]</field>
    </record>

    <record model="ir.rule" id="product_pricelist_item_comp_rule">
        <field name="name">product pricelist item company rule</field>
        <field name="model_id" ref="model_product_pricelist_item"/>
        <field name="global" eval="True"/>
        <field name="domain_force"> ['|', ('company_id', 'in', company_ids), ('company_id', '=', False)]</field>
    </record>

    <record model="ir.rule" id="product_supplierinfo_comp_rule">
        <field name="name">product supplierinfo company rule</field>
        <field name="model_id" ref="model_product_supplierinfo"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="product_packaging_comp_rule">
        <field name="name">product packaging company rule</field>
        <field name="model_id" ref="model_product_packaging"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]</field>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient><path id="d" d="M48.466 42.033c0 .837-.277 1.548-.831 2.133L36.606 55.823c-.584.585-1.265.877-2.044.877-.793 0-1.467-.292-2.021-.877l-16.06-16.965c-.569-.585-1.051-1.382-1.448-2.393s-.595-1.935-.595-2.773v-9.857c0-.821.284-1.532.853-2.132.569-.6 1.243-.9 2.021-.9h9.344c.794 0 1.67.209 2.628.627.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156zM24.5 28.385c0-.838-.28-1.552-.842-2.145-.562-.592-1.24-.888-2.033-.888-.794 0-1.471.296-2.033.888-.561.593-.842 1.307-.842 2.145 0 .837.28 1.552.842 2.144.562.592 1.24.889 2.033.889.794 0 1.471-.297 2.033-.889.561-.592.842-1.307.842-2.144zm32.59 13.648c0 .837-.276 1.548-.83 2.133L45.23 55.823c-.584.585-1.265.877-2.044.877-.539 0-.98-.11-1.325-.332-.344-.22-.74-.576-1.19-1.066l10.557-11.136c.554-.585.83-1.296.83-2.133 0-.821-.276-1.54-.83-2.156l-16.06-16.919c-.57-.6-1.333-1.11-2.291-1.528-.958-.418-1.834-.628-2.628-.628h5.031c.794 0 1.67.21 2.628.628.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156z"/><path id="e" d="M48.466 40.033c0 .837-.277 1.548-.831 2.133L36.606 53.823c-.584.585-1.265.877-2.044.877-.793 0-1.467-.292-2.021-.877l-16.06-16.965c-.569-.585-1.051-1.382-1.448-2.393s-.595-1.935-.595-2.773v-9.857c0-.821.284-1.532.853-2.132.569-.6 1.243-.9 2.021-.9h9.344c.794 0 1.67.209 2.628.627.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156zM24.5 26.385c0-.838-.28-1.552-.842-2.145-.562-.592-1.24-.888-2.033-.888-.794 0-1.471.296-2.033.888-.561.593-.842 1.307-.842 2.145 0 .837.28 1.552.842 2.144.562.592 1.24.889 2.033.889.794 0 1.471-.297 2.033-.889.561-.592.842-1.307.842-2.144zm32.59 13.648c0 .837-.276 1.548-.83 2.133L45.23 53.823c-.584.585-1.265.877-2.044.877-.539 0-.98-.11-1.325-.332-.344-.22-.74-.576-1.19-1.066l10.557-11.136c.554-.585.83-1.296.83-2.133 0-.821-.276-1.54-.83-2.156l-16.06-16.919c-.57-.6-1.333-1.11-2.291-1.528-.958-.418-1.834-.628-2.628-.628h5.031c.794 0 1.67.21 2.628.628.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L16.402 19h19.682L56.79 41 39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\product_attribute_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="attribute_tree_view" model="ir.ui.view">
        <field name="name">product.attribute.tree</field>
        <field name="model">product.attribute</field>
        <field name="arch" type="xml">
            <tree string="Variant Values">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="create_variant"/>
            </tree>
        </field>
    </record>

    <record id="product_attribute_view_form" model="ir.ui.view">
        <field name="name">product.attribute.form</field>
        <field name="model">product.attribute</field>
        <field name="arch" type="xml">
            <form string="Product Attribute">
            <field name="is_used_on_products" invisible="1"/>
            <sheet>
                <group name="main_fields" class="o_label_nowrap">
                    <label for="name" string="Attribute Name"/>
                    <field name="name" nolabel="1"/>
                    <field name="create_variant" widget="radio" attrs="{'readonly': [('is_used_on_products', '=', True)]}"/>
                </group>
                <notebook>
                    <page string="Attribute Values">
                        <field name="value_ids" widget="one2many" nolabel="1">
                            <tree string="Values" editable="bottom">
                                <field name="sequence" widget="handle"/>
                                <field name="name"/>
                            </tree>
                        </field>
                    </page>
                    <page string="Related Products" attrs="{'invisible': [('is_used_on_products', '=', False)]}">
                        <field name="product_tmpl_ids">
                            <tree>
                                <field name="name"/>
                            </tree>
                        </field>
                    </page>
                </notebook>
            </sheet>
            </form>
        </field>
    </record>

    <record id="attribute_action" model="ir.actions.act_window">
        <field name="name">Attributes</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.attribute</field>
        <field name="view_mode">tree,form</field>
    </record>

    <record id="product_template_attribute_line_form" model="ir.ui.view">
        <field name="name">product.template.attribute.line.form</field>
        <field name="model">product.template.attribute.line</field>
        <field name="mode">primary</field>
        <field name="priority" eval="8"/>
        <field name="arch" type="xml">
            <form string="Product Attribute and Values">
                <group name="main_field">
                    <label for="attribute_id" string="Attribute Name"/>
                    <field name="attribute_id" nolabel="1"/>
                    <field name="value_ids" widget="one2many">
                        <tree string="Values">
                            <field name="name"/>
                        </tree>
                        <form string="Values">
                            <field name="name"/>
                        </form>
                    </field>
                </group>
            </form>
        </field>
    </record>

    <record id="product_template_attribute_value_view_tree" model="ir.ui.view">
        <field name="name">product.template.attribute.value.view.tree</field>
        <field name="model">product.template.attribute.value</field>
        <field name="type">tree</field>
        <field name="arch" type="xml">
            <tree string="Attributes" create="0" delete="0">
                <field name="attribute_id"/>
                <field name="name"/>
                <field name="ptav_active" optional="hide"/>
                <field name="price_extra"/>
            </tree>
        </field>
    </record>

    <record id="product_template_attribute_value_view_form" model="ir.ui.view">
        <field name="name">product.template.attribute.value.view.form.</field>
        <field name="model">product.template.attribute.value</field>
        <field name="type">form</field>
        <field name="arch" type="xml">
            <form string="Product Attribute" create="0" delete="0">
                <sheet>
                    <group>
                        <field name="ptav_active" readonly="1" attrs="{'invisible': [('ptav_active', '=', True)]}"/>
                        <field name="name"/>
                        <field name="price_extra" />
                        <field name="exclude_for" widget="one2many" mode="tree">
                            <tree editable="bottom">
                                <field name="product_tmpl_id" />
                                <field name="value_ids" widget="many2many_tags" options="{'no_create': True}" />
                            </tree>
                        </field>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="product_template_attribute_value_view_search" model="ir.ui.view">
        <field name="model">product.template.attribute.value</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="Active" name="active" domain="[('ptav_active', '=', True)]"/>
                <filter string="Inactive" name="inactive" domain="[('ptav_active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="product_attribute_value_action" model="ir.actions.act_window">
        <field name="name">Product Variant Values</field>
        <field name="res_model">product.template.attribute.value</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('product_tmpl_id', '=', active_id)]</field>
        <field name="view_ids"
                eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('product.product_template_attribute_value_view_tree')}),
                (0, 0, {'view_mode': 'form', 'view_id': ref('product.product_template_attribute_value_view_form')})]" />
        <field name="context">{
            'default_product_tmpl_id': active_id,
            'search_default_active': 1,
        }</field>
    </record>

</odoo>

```

## File: views\product_pricelist_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record model="ir.ui.view" id="product_pricelist_item_view_search">
            <field name="name">product.pricelist.item.search</field>
            <field name="model">product.pricelist.item</field>
            <field name="arch" type="xml">
                <search string="Products Price Rules Search">
                    <filter name="Product Rule" domain="[('applied_on', '=', '1_product')]"/>
                    <filter name="Variant Rule" domain="[('applied_on', '=', '0_product_variant')]" groups="product.group_product_variant"/>
                    <separator/>
                    <field name="pricelist_id"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="currency_id" groups="base.group_multi_currency"/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Product" name="groupby_product" domain="[]" context="{'group_by': 'product_tmpl_id'}"/>
                        <filter string="Variant"
                          name="groupby_product_variant"
                          domain="[('applied_on', '=', '0_product_variant')]"
                          context="{'group_by': 'product_tmpl_id'}"
                          groups="product.group_product_variant"/>
                        <filter string="Pricelist"
                          name="groupby_vendor"
                          domain="[]"
                          context="{'group_by': 'pricelist_id'}"
                          groups="product.group_product_pricelist"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="product_pricelist_item_tree_view" model="ir.ui.view">
          <field name="name">product.pricelist.item.tree</field>
          <field name="model">product.pricelist.item</field>
          <field name="priority">10</field>
          <field name="arch" type="xml">
            <tree string="Price Rules">
              <field name="pricelist_id"/>
              <field name="name" string="Applied On"/>
              <field name="price"/>
              <field name="min_quantity" colspan="4"/>
              <field name="date_start" optional="hide"/>
              <field name="date_end" optional="hide"/>
              <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </tree>
          </field>
        </record>

        <record id="product_pricelist_item_tree_view_from_product" model="ir.ui.view">
            <!-- Access and edit price rules from a given product/product variant -->
            <field name="name">product.pricelist.item.tree</field>
            <field name="model">product.pricelist.item</field>
            <field name="priority">100</field>
            <field name="arch" type="xml">
                <tree string="Pricelist Items" editable="bottom">
                    <!-- Scope = coming from a product/product template -->
                    <field name="pricelist_id" string="Pricelist" options="{'no_create_edit':1, 'no_open': 1}"/>
                    <field name="name" string="Applied On"/>
                    <field name="categ_id" invisible="1"/>
                    <field name="product_tmpl_id"
                      invisible="context.get('active_model')!='product.category'"
                      attrs="{'required': [('applied_on', '=', '1_product')]}"
                      domain="[('categ_id', '=', context.get('default_categ_id', True)), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"
                      options="{'no_create_edit':1, 'no_open': 1}"/>
                    <field name="product_id"
                      groups="product.group_product_variant"
                      invisible="context.get('product_without_variants', False)"
                      readonly="context.get('active_model')=='product.product'"
                      attrs="{'required': [('applied_on', '=', '0_product_variant')]}"
                      domain="['|', '|',
                        ('id', '=', context.get('default_product_id', 0)),
                        ('product_tmpl_id', '=', context.get('default_product_tmpl_id', 0)),
                        ('categ_id', '=', context.get('default_categ_id', 0)), '|', ('company_id', '=', company_id), ('company_id', '=', False)
                      ]"
                      options="{'no_create_edit':1, 'no_open': 1}"
                      />
                    <field name="min_quantity" colspan="4"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="fixed_price" string="Price" required='1'/>
                    <field name="date_start" optional="show"/>
                    <field name="date_end" optional="show"/>
                    <field name="applied_on" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show" options="{'no_create':1, 'no_open': 1}"/>
                </tree>
            </field>
        </record>

        <record id="product_pricelist_item_form_view" model="ir.ui.view">
            <field name="name">product.pricelist.item.form</field>
            <field name="model">product.pricelist.item</field>
            <field name="arch" type="xml">
                <form string="Pricelist Items">
                    <sheet>
                      <h1><field name="name"/></h1>
                      <group>
                          <group name="pricelist_rule_target">
                              <field name="applied_on" widget="radio"/>
                              <field name="categ_id" attrs="{
                                  'invisible':[('applied_on', '!=', '2_product_category')],
                                  'required':[('applied_on', '=', '2_product_category')]}"
                                options="{'no_create':1}"/>
                              <field name="product_tmpl_id" attrs="{
                                'invisible':[('applied_on', '!=', '1_product')],
                                'required':[('applied_on', '=', '1_product')]}"
                                options="{'no_create':1}"/>
                              <field name="product_id" attrs="{
                                'invisible':[('applied_on', '!=', '0_product_variant')],
                                'required':[('applied_on', '=', '0_product_variant')]}"
                                options="{'no_create':1}"/>
                          </group>
                          <group name="pricelist_rule_limits">
                              <field name="min_quantity"/>
                              <field name="date_start"/>
                              <field name="date_end"/>
                          </group>
                          <group name="pricelist_rule_related" groups="base.group_no_one">
                              <!-- Infos from the pricelist for UI rendering (monetary fields, ...) -->
                              <field name="pricelist_id" invisible="1"/>
                              <field name="currency_id" groups="base.group_multi_currency"/>
                              <field name="company_id" groups="base.group_multi_company"/>
                          </group>
                      </group>
                      <group string="Price Computation" name="pricelist_rule_computation" groups="product.group_sale_pricelist">
                          <group name="pricelist_rule_method">
                              <field name="compute_price" string="Compute Price" widget="radio"/>
                          </group>
                          <group name="pricelist_rule_base">
                              <field name="fixed_price" attrs="{'invisible':[('compute_price', '!=', 'fixed')]}"/>
                              <label for="percent_price" attrs="{'invisible':[('compute_price', '!=', 'percentage')]}"/>
                              <div attrs="{'invisible':[('compute_price', '!=', 'percentage')]}">
                                  <field name="percent_price"
                                    class="oe_inline"
                                    attrs="{'invisible':[('compute_price', '!=', 'percentage')]}"/>
                                  %%
                              </div>
                              <field name="base" attrs="{'invisible':[('compute_price', '!=', 'formula')]}"/>
                              <field name="base_pricelist_id" attrs="{
                                'invisible': ['|', ('compute_price', '!=', 'formula'), ('base', '!=', 'pricelist')],
                                'required': [('compute_price', '=', 'formula'), ('base', '=', 'pricelist')],
                                'readonly': [('base', '!=', 'pricelist')]}"/>
                          </group>
                      </group>
                      <div class="oe_grey" groups="uom.group_uom">
                          <p>The computed price is expressed in the default Unit of Measure of the product.</p>
                      </div>
                      <group name="pricelist_rule_advanced" col="6" attrs="{'invisible':[('compute_price', '!=', 'formula')]}" groups="product.group_sale_pricelist">
                          <label for="" string="New Price ="/>
                          <div>
                              <span attrs="{'invisible':[('base', '!=', 'list_price')]}">Sales Price  -  </span>
                              <span attrs="{'invisible':[('base', '!=', 'standard_price')]}">Cost  -  </span>
                              <span attrs="{'invisible':[('base', '!=', 'pricelist')]}">Other Pricelist  -  </span>
                          </div>
                          <label for="price_discount"/>
                          <div class="o_row">
                              <field name="price_discount"/>
                              <span>%%</span>
                          </div>
                          <label string=" + " for="price_surcharge"/>
                          <field name="price_surcharge" nolabel="1"/>

                          <field name="price_round" string="Rounding Method"/>
                          <field name="price_min_margin" string="Min. Margin"/>
                          <field name="price_max_margin" string="Max. Margin"/>
                      </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="product_pricelist_view_search">
            <field name="name">product.pricelist.search</field>
            <field name="model">product.pricelist</field>
            <field name="arch" type="xml">
                <search string="Products Price Search">
                    <field name="name" string="Products Price"/>
                    <field name="currency_id" groups="base.group_multi_currency"/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
            </field>
        </record>


        <record id="product_pricelist_view_tree" model="ir.ui.view">
            <field name="name">product.pricelist.tree</field>
            <field name="model">product.pricelist</field>
            <field name="arch" type="xml">
                <tree string="Products Price List">
                    <field name="sequence" widget="handle" />
                    <field name="name"/>
                    <field name="discount_policy" groups="product.group_discount_per_so_line"/>
                    <field name="currency_id" groups="base.group_multi_currency"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="product_pricelist_view_kanban" model="ir.ui.view">
            <field name="name">product.pricelist.kanban</field>
            <field name="model">product.pricelist</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div id="product_pricelist" class="o_kanban_record_top mb0">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title"><span><field name="name"/></span></strong>
                                    </div>
                                    <strong><i class="fa fa-money" role="img" aria-label="Currency" title="Currency"></i> <field name="currency_id"/></strong>
                                </div>
                                <field name="discount_policy" groups="product.group_discount_per_so_line"/>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="product_pricelist_view" model="ir.ui.view">
            <field name="name">product.pricelist.form</field>
            <field name="model">product.pricelist</field>
            <field name="arch" type="xml">
                <form string="Products Price List">
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title">
                            <h1><field name="name" placeholder="e.g. USD Retailers"/></h1>
                        </div>
                        <group>
                          <group name="pricelist_settings">
                              <field name="currency_id" groups="base.group_multi_currency"/>
                              <field name="active" invisible="1"/>
                              <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                          </group>
                        </group>
                        <notebook>
                            <page name="pricelist_rules" string="Price Rules">
                              <field name="item_ids" nolabel="1" context="{'default_base':'list_price'}">
                                  <tree string="Pricelist Items" editable="bottom">
                                      <field name="product_tmpl_id" string="Products" required="1"/>
                                      <field name="product_id" string="Variants"
                                        groups="product.group_product_variant"
                                        domain="[('product_tmpl_id', '=', product_tmpl_id)]"
                                        options="{'no_create':1}"/>
                                      <field name="min_quantity"/>
                                      <field name="fixed_price" string="Price"/>
                                      <field name="currency_id" invisible="1"/>
                                      <field name="pricelist_id" invisible="1"/>
                                      <!-- Pricelist ID is here only for related fields to be correctly computed -->
                                      <field name="date_start"/>
                                      <field name="date_end"/>
                                      <field name="base" invisible="1"/>
                                      <field name="applied_on" invisible="1"/>
                                      <field name="company_id" invisible="1"/>
                                  </tree>
                              </field>
                            </page>
                            <page name="pricelist_config" string="Configuration">
                                <group>
                                    <group name="pricelist_availability" string="Availability">
                                        <field name="country_group_ids" widget="many2many_tags"/>
                                    </group>
                                    <group name="pricelist_discounts" groups="product.group_discount_per_so_line" string="Discounts">
                                        <field name="discount_policy" widget="radio"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="product_pricelist_view_inherit" model="ir.ui.view">
            <field name="name">product.pricelist.form.inherit</field>
            <field name="model">product.pricelist</field>
            <field name="inherit_id" ref="product.product_pricelist_view"/>
            <field name="groups_id" eval="[(4, ref('product.group_sale_pricelist'))]"/>
            <field name="arch" type="xml">
                <!-- When in advanced pricelist mode : pricelist rules
                  Should open in a form view and not be editable inline anymore.
                -->
                <field name="item_ids" position="replace">
                  <field name="item_ids" nolabel="1" context="{'default_base':'list_price'}" groups="product.group_product_pricelist">
                      <tree string="Pricelist Items">
                          <field name="name" string="Applicable On"/>
                          <field name="min_quantity"/>
                          <field name="price" string="Price"/>
                          <field name="date_start"/>
                          <field name="date_end"/>
                          <field name="base" invisible="1"/>
                          <field name="price_discount" invisible="1"/>
                          <field name="applied_on" invisible="1"/>
                          <field name="compute_price" invisible="1"/>
                      </tree>
                  </field>
                </field>
            </field>
        </record>

        <record model="ir.ui.view" id="inherits_website_sale_country_group_form">
            <field name="name">website_sale.country_group.form</field>
            <field name="model">res.country.group</field>
            <field name="inherit_id" ref="base.view_country_group_form"/>
            <field name="arch" type="xml">
                <group name="country_group" position="after">
                    <field name="pricelist_ids"/>
                </group>
            </field>
        </record>
        <record id="product_pricelist_action2" model="ir.actions.act_window">
            <field name="name">Pricelists</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">product.pricelist</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="search_view_id" ref="product_pricelist_view_search" />
            <field name="context">{"default_base":'list_price'}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new pricelist
              </p><p>
                A price is a set of sales prices or rules to compute the price of sales order lines based on products, product categories, dates and ordered quantities.
                This is the perfect tool to handle several pricings, seasonal discounts, etc.
              </p><p>
                You can assign pricelists to your customers or select one when creating a new sales quotation.
              </p>
            </field>
        </record>

        <record id="product_pricelist_item_action" model="ir.actions.act_window">
            <field name="name">Price Rules</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">product.pricelist.item</field>
            <field name="view_mode">tree,form</field>
        </record>
    </data>
</odoo>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_tree_view" model="ir.ui.view">
        <field name="name">product.template.product.tree</field>
        <field name="model">product.template</field>
        <field name="arch" type="xml">
            <tree string="Product" multi_edit="1">
                <field name="product_variant_count" invisible="1"/>

                <field name="sequence" widget="handle" readonly="1"/>
                <field name="default_code" optional="show"/>
                <field name="barcode" optional="hide" attrs="{'readonly': [('product_variant_count', '>', 1)]}"/>
                <field name="name"/>
                <field name="company_id" options="{'no_create': True}"
                    groups="base.group_multi_company" optional="hide"/>
                <field name="list_price" string="Sales Price" optional="show"/>
                <field name="standard_price" optional="show" readonly="1"/>
                <field name="categ_id" optional="hide"/>
                <field name="type" optional="hide" readonly="1"/>
                <field name="uom_id" readonly="1" optional="show" groups="uom.group_uom"/>
                <field name="active" invisible="1"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id="product_template_only_form_view" model="ir.ui.view">
        <field name="name">product.template.product.form</field>
        <field name="model">product.template</field>
        <field name="mode">primary</field>
        <field name="priority" eval="8" />
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="attributes">
                <attribute name="name">Product Template</attribute>
            </xpath>
            <field name="categ_id" position="after">
                <field name="default_code" attrs="{'invisible': [('product_variant_count', '>', 1)]}"/>
                <field name="barcode" attrs="{'invisible': [('product_variant_count', '>', 1)]}"/>
            </field>

            <div name="button_box" position="inside">
                <button name="%(product.product_variant_action)d" type="action"
                    icon="fa-sitemap" class="oe_stat_button"
                    attrs="{'invisible': [('product_variant_count', '&lt;=', 1)]}"
                    groups="product.group_product_variant">
                    <field string="Variants" name="product_variant_count" widget="statinfo" />
                </button>
            </div>

            <xpath expr="//page[@name='general_information']" position="after">
                <page name="variants" string="Variants" groups="product.group_product_variant">
                    <field name="attribute_line_ids" widget="one2many" context="{'show_attribute': False}">
                        <tree string="Variants" editable="bottom">
                            <field name="attribute_id" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="value_ids" widget="many2many_tags" options="{'no_create_edit': True}" context="{'default_attribute_id': attribute_id, 'show_attribute': False}"/>
                        </tree>
                    </field>
                        <p class="oe_grey oe_edit_only">
                        <strong>Warning</strong>: adding or deleting attributes
                        will delete and recreate existing variants and lead
                        to the loss of their possible customizations.
                    </p>
                </page>
            </xpath>
        </field>
    </record>

    <record id="product_template_kanban_view" model="ir.ui.view">
        <field name="name">Product.template.product.kanban</field>
        <field name="model">product.template</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="id"/>
                <field name="product_variant_count"/>
                <field name="currency_id"/>
                <field name="activity_state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click">
                            <div class="o_kanban_image">
                                <img t-att-src="kanban_image('product.template', 'image_128', record.id.raw_value)" alt="Product" class="o_image_64_contain"/>
                            </div>
                            <div class="oe_kanban_details">
                                <strong class="o_kanban_record_title">
                                    <field name="name"/>
                                    <small t-if="record.default_code.value">[<field name="default_code"/>]</small>
                                </strong>
                                <div t-if="record.product_variant_count.value &gt; 1" groups="product.group_product_variant">
                                    <strong>
                                        <t t-esc="record.product_variant_count.value"/> Variants
                                    </strong>
                                </div>
                                <div name="tags"/>
                                <ul>
                                    <li>Price: <field name="lst_price" widget="monetary" options="{'currency_field': 'currency_id', 'field_digits': True}"></field></li>
                                </ul>
                                <div name="tags"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="product_template_view_activity" model="ir.ui.view">
        <field name="name">product.template.activity</field>
        <field name="model">product.template</field>
        <field name="arch" type="xml">
            <activity string="Products">
                <field name="id"/>
                <templates>
                    <div t-name="activity-box">
                        <img t-att-src="activity_image('product.template', 'image_128', record.id.raw_value)" role="img" t-att-title="record.id.value" t-att-alt="record.id.value"/>
                        <div>
                            <field name="name" display="full"/>
                            <div t-if="record.default_code.value" class="text-muted">
                                [<field name="default_code"/>]
                            </div>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="product_template_action" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="view_id" ref="product_template_kanban_view"/>
        <field name="search_view_id" ref="product.product_template_search_view"/>
        <field name="context">{"search_default_filter_to_sell":1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new product
            </p><p>
                You must define a product for everything you sell or purchase,
                whether it's a storable product, a consumable or a service.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    <!-- base structure of product.template, common with product.product -->
    <record id="product_template_form_view" model="ir.ui.view">
        <field name="name">product.template.common.form</field>
        <field name="model">product.template</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <form string="Product">
                <header>
                    <button string="Configure Variants" type="action"
                        name="%(product_attribute_value_action)d"
                        attrs="{'invisible': ['|', ('attribute_line_ids', '&lt;=', 0), ('is_product_variant', '=', True)]}"
                        groups="product.group_product_variant"/>
                </header>
                <sheet>
                    <field name='product_variant_count' invisible='1'/>
                    <field name='is_product_variant' invisible='1'/>
                    <field name='attribute_line_ids' invisible='1'/>
                    <div class="oe_button_box" name="button_box"/>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="id" invisible="True"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label class="oe_edit_only" for="name" string="Product Name"/>
                        <h1><field name="name" placeholder="Product Name"/></h1>
                        <div name="options" groups="base.group_user">
                            <div>
                                <field name="sale_ok"/>
                                <label for="sale_ok"/>
                            </div>
                            <div>
                                <field name="purchase_ok"/>
                                <label for="purchase_ok"/>
                            </div>
                        </div>
                    </div>
                    <notebook>
                        <page string="General Information" name="general_information">
                            <group>
                                <group name="group_general">
                                    <field name="active" invisible="1"/>
                                    <field name="type"/>
                                    <field name="categ_id" string="Product Category"/>
                                </group>
                                <group name="group_standard_price">
                                    <label for="list_price"/>
                                    <div name="pricing">
                                      <field name="list_price" class="oe_inline" widget='monetary'
                                        options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                                      <button name="open_pricelist_rules" icon="fa-arrow-right" type="object"
                                        groups="product.group_product_pricelist" class="oe_inline">
                                        <field name="pricelist_item_count" attrs="{'invisible': [('pricelist_item_count', '=', 0)]}"/>
                                        <span attrs="{'invisible': [('pricelist_item_count', '=', 1)]}">
                                          Extra Prices
                                        </span>
                                        <span attrs="{'invisible': [('pricelist_item_count', '!=', 1)]}">
                                          Extra Price
                                        </span>
                                      </button>
                                    </div>
                                    <label for="standard_price" groups="base.group_user" attrs="{'invisible': [('product_variant_count', '&gt;', 1), ('is_product_variant', '=', False)]}"/>
                                    <div name="standard_price_uom" groups="base.group_user" attrs="{'invisible': [('product_variant_count', '&gt;', 1), ('is_product_variant', '=', False)]}" class="o_row">
                                        <field name="standard_price" widget='monetary' options="{'currency_field': 'cost_currency_id', 'field_digits': True}"/>
                                        <span groups="uom.group_uom" class="oe_read_only">per
                                            <field name="uom_name"/>
                                        </span>
                                    </div>
                                    <field name="company_id" groups="base.group_multi_company"
                                        options="{'no_create': True}"/>
                                    <field name="uom_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                    <field name="uom_po_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="cost_currency_id" invisible="1"/>
                                    <field name="product_variant_id" invisible="1"/>
                                </group>
                            </group>
                            <group string="Internal Notes">
                                <field name="description" nolabel="1" placeholder="This note is only for internal purposes."/>
                            </group>
                        </page>
                        <page string="Sales" attrs="{'invisible':[('sale_ok','=',False)]}" name="sales" invisible="1">
                            <group name="sale">
                                <group name="email_template_and_project" invisible="1"/>
                            </group>
                            <group string="Sales Description" name="description">
                                <field name="description_sale" nolabel="1" placeholder="This note is added to sales orders and invoices."/>
                            </group>
                        </page>
                        <page string="Purchase" name="purchase" attrs="{'invisible': [('purchase_ok','=',False)]}" invisible="1">
                            <group name="purchase">
                                <group string="Vendor Bills" name="bill"/>
                            </group>
                        </page>
                        <page string="Inventory" name="inventory" groups="product.group_stock_packaging" attrs="{'invisible':[('type', '=', 'service')]}">
                            <group name="inventory">
                                <group name="group_lots_and_weight" string="Logistics" attrs="{'invisible': [('type', 'not in', ['product', 'consu'])]}">
                                    <label for="weight" attrs="{'invisible':[('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}"/>
                                    <div class="o_row" name="weight" attrs="{'invisible':[('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}">
                                        <field name="weight"/>
                                        <span><field name="weight_uom_name"/></span>
                                    </div>
                                    <label for="volume" attrs="{'invisible':[('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}"/>
                                    <div class="o_row" name="volume" attrs="{'invisible':[('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}">
                                        <field name="volume" string="Volume"/>
                                        <span><field name="volume_uom_name"/></span>
                                    </div>
                                </group>
                            </group>
                            <group name="packaging" string="Packaging"
                                colspan="4"
                                attrs="{'invisible':['|', ('type', 'not in', ['product', 'consu']), ('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}"
                                groups="product.group_stock_packaging">
                                <field name="packaging_ids" nolabel="1" context="{'tree_view_ref':'product.product_packaging_tree_view2', 'form_view_ref':'product.product_packaging_form_view2', 'default_company_id': company_id}"/>
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

    <record id="product_template_search_view" model="ir.ui.view">
        <field name="name">product.template.search</field>
        <field name="model">product.template</field>
        <field name="arch" type="xml">
            <search string="Product">
                <field name="name" string="Product" filter_domain="['|', '|', '|', ('default_code', 'ilike', self), ('product_variant_ids.default_code', 'ilike', self),('name', 'ilike', self), ('barcode', 'ilike', self)]"/>
                <field name="categ_id" filter_domain="[('categ_id', 'child_of', raw_value)]"/>
                <separator/>
                <filter string="Services" name="services" domain="[('type','=','service')]"/>
                <filter string="Products" name="consumable" domain="[('type', 'in', ['consu', 'product'])]"/>
                <separator/>
                <filter string="Can be Sold" name="filter_to_sell" domain="[('sale_ok','=',True)]"/>
                <filter string="Can be Purchased" name="filter_to_purchase" domain="[('purchase_ok', '=', True)]"/>
                <separator/>
                <field string="Attributes" name="attribute_line_ids" groups="product.group_product_variant"/>
                <field name="pricelist_id" context="{'pricelist': self}" filter_domain="[]" groups="product.group_product_pricelist"/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))
                    ]"/>
                <separator/>
                <filter string="Warnings" name="activities_exception"
                        domain="[('activity_exception_decoration', '!=', False)]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                <group expand="1" string="Group By">
                    <filter string="Product Type" name="type" context="{'group_by':'type'}"/>
                    <filter string="Product Category" name="categ_id" context="{'group_by':'categ_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="product_template_action_all" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="context">{}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new product
            </p>
        </field>
    </record>

        <record id="product_search_form_view" model="ir.ui.view">
            <field name="name">product.product.search</field>
            <field name="model">product.product</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="product.product_template_search_view"/>
            <field name="arch" type="xml">
                <field name="name" position="replace">
                    <field name="name" string="Product" filter_domain="['|', '|', ('default_code', 'ilike', self), ('name', 'ilike', self), ('barcode', 'ilike', self)]"/>
                </field>
                <field name="attribute_line_ids" position="replace">
                    <field name="product_template_attribute_value_ids" groups="product.group_product_variant"/>
                    <field name="product_tmpl_id" string="Product Template"/>
                </field>
            </field>
        </record>

        <record id="product_normal_action" model="ir.actions.act_window">
            <field name="name">Product Variants</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">product.product</field>
            <field name="view_mode">tree,form,kanban,activity</field>
            <field name="search_view_id" ref="product_search_form_view"/>
            <field name="view_id" eval="False"/> <!-- Force empty -->
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new product variant
              </p><p>
                You must define a product for everything you sell or purchase,
                whether it's a storable product, a consumable or a service.
              </p>
            </field>
        </record>

        <record id="product_variant_easy_edit_view" model="ir.ui.view">
            <field name="name">product.product.view.form.easy</field>
            <field name="model">product.product</field>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <form string="Variant Information" duplicate="false">
                    <sheet>
                        <div class="oe_button_box" name="button_box"/>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="active" invisible="1"/>
                        <field name="id" invisible="1"/>
                        <field name="company_id" invisible="1"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                        <div class="oe_title">
                            <label class="oe_edit_only" for="name" string="Product Name"/>
                            <h1><field name="name" readonly="1" placeholder="e.g. Odoo Enterprise Subscription"/></h1>
                            <field name="product_template_attribute_value_ids" widget="many2many_tags" readonly="1"/>
                            <p>
                                <span>All general settings about this product are managed on</span>
                                <button name="open_product_template" type="object" string="the product template." class="oe_link oe_link_product pl-0 ml-1 mb-1"/>
                            </p>
                        </div>
                        <group>
                            <group name="codes" string="Codes">
                                <field name="default_code"/>
                                <field name="barcode"/>
                                <field name="type" invisible="1"/>
                            </group>
                            <group name="pricing" string="Pricing">
                                <field name="product_variant_count" invisible="1"/>
                                <label for="lst_price"/>
                                <div class="o_row col-5 pl-0">
                                    <field name="lst_price" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}" attrs="{'readonly': [('product_variant_count', '&gt;', 1)]}"/>
                                </div>
                                <field name="standard_price" widget='monetary' options="{'currency_field': 'cost_currency_id'}"/>
                                <field name="currency_id" invisible='1'/>
                                <field name="cost_currency_id" invisible="1"/>
                            </group>
                        </group>
                        <group>
                            <group name="weight" string="Logistics" attrs="{'invisible':[('type', 'not in', ['product', 'consu'])]}">
                                <label for="volume"/>
                                <div class="o_row">
                                    <field name="volume"/>
                                    <span><field name="volume_uom_name"/></span>
                                </div>
                                <label for="weight"/>
                                <div class="o_row">
                                    <field name="weight"/>
                                    <span><field name="weight_uom_name"/></span>
                                </div>
                            </group>
                            <group name="packaging" string="Packaging" groups="product.group_stock_packaging">
                                <field name="packaging_ids" nolabel="1"
                                    context="{'tree_view_ref':'product.product_packaging_tree_view2', 'form_view_ref':'product.product_packaging_form_view2', 'default_company_id': company_id}"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="product_variant_action" model="ir.actions.act_window">
            <field name="name">Product Variants</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">product.product</field>
            <field name="context">{'search_default_product_tmpl_id': [active_id], 'default_product_tmpl_id': active_id, 'create': False}</field>
            <field name="search_view_id" ref="product_search_form_view"/>
            <field name="view_ids"
                   eval="[(5, 0, 0),
                          (0, 0, {'view_mode': 'tree'}),
                          (0, 0, {'view_mode': 'form', 'view_id': ref('product_variant_easy_edit_view')}),
                          (0, 0, {'view_mode': 'kanban'})]"/>
             <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new product variant
              </p><p>
                You must define a product for everything you sell or purchase,
                whether it's a storable product, a consumable or a service.
                The product form contains information to simplify the sale process:
                price, notes in the quotation, accounting data, procurement methods, etc.
              </p>
            </field>
        </record>

        <record id="product_product_tree_view" model="ir.ui.view">
            <field name="name">product.product.tree</field>
            <field name="model">product.product</field>
            <field eval="7" name="priority"/>
            <field name="arch" type="xml">
                <tree string="Product Variants" multi_edit="1" duplicate="false">
                    <field name="default_code" optional="show" readonly="1"/>
                    <field name="barcode" optional="hide" readonly="1"/>
                    <field name="name" readonly="1"/>
                    <field name="product_template_attribute_value_ids" widget="many2many_tags" groups="product.group_product_variant" readonly="1"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide" readonly="1"/>
                    <field name="lst_price" optional="show" string="Sales Price"/>
                    <field name="standard_price" optional="show" readonly="1"/>
                    <field name="categ_id" optional="hide"/>
                    <field name="type" optional="hide" readonly="1"/>
                    <field name="price" invisible="not context.get('pricelist',False)"/>
                    <field name="uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom" optional="show" readonly="1"/>
                    <field name="product_tmpl_id" invisible="1" readonly="1"/>
                    <field name="active" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="product_normal_form_view" model="ir.ui.view">
            <field name="name">product.product.form</field>
            <field name="model">product.product</field>
            <field name="mode">primary</field>
            <field eval="7" name="priority"/>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <form position="attributes">
                    <attribute name="string">Product Variant</attribute>
                    <attribute name="duplicate">false</attribute>
                </form>
                <field name="type" position="after">
                    <field name="default_code"/>
                    <field name="barcode"/>
                </field>
                <field name="list_price" position="attributes">
                   <attribute name="name">lst_price</attribute>
                   <attribute name="attrs">{'readonly': [('product_variant_count', '&gt;', 1)]}</attribute>
                </field>
                <xpath expr="//label[@for='list_price']" position="replace">
                    <label for="lst_price"/>
                </xpath>
                <group name="packaging" position="attributes">
                    <attribute name="attrs">{'invisible': 0}</attribute>
                </group>
                <field name="name" position="after">
                    <field name="product_tmpl_id" class="oe_inline" readonly="1" invisible="1" attrs="{'required': [('id', '!=', False)]}"/>
                </field>
                <xpath expr="//div[hasclass('oe_title')]" position="inside">
                    <field name="product_template_attribute_value_ids" widget="many2many_tags" readonly="1"
                        groups="product.group_product_variant"/>
                </xpath>
            </field>
        </record>

        <record id="product_kanban_view" model="ir.ui.view">
            <field name="name">Product Kanban</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <kanban>
                    <field name="id"/>
                    <field name="lst_price"/>
                    <field name="activity_state"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div class="o_kanban_image">
                                    <img t-att-src="kanban_image('product.product', 'image_128', record.id.raw_value)" alt="Product" class="o_image_64_contain"/>
                                </div>
                                <div class="oe_kanban_details">
                                    <strong class="o_kanban_record_title">
                                        <field name="name"/>
                                        <small t-if="record.default_code.value">[<field name="default_code"/>]</small>
                                    </strong>
                                    <div class="o_kanban_tags_section">
                                        <field name="product_template_attribute_value_ids" groups="product.group_product_variant"/>
                                    </div>
                                    <ul>
                                        <li><strong>Price: <field name="lst_price"></field></strong></li>
                                    </ul>
                                    <div name="tags"/>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="product_product_view_activity" model="ir.ui.view">
            <field name="name">product.product.activity</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <activity string="Product Variants">
                    <field name="id"/>
                    <field name="default_code"/>
                    <templates>
                        <div t-name="activity-box">
                            <img t-att-src="activity_image('product.product', 'image_128', record.id.raw_value)" role="img" t-att-title="record.id.value" t-att-alt="record.id.value"/>
                            <div>
                                <field name="name" display="full"/>
                                <div t-if="record.default_code.value" class="text-muted">
                                    [<field name="default_code"/>]
                                </div>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="product_normal_action_sell" model="ir.actions.act_window">
            <field name="name">Product Variants</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">product.product</field>
            <field name="view_mode">kanban,tree,form,activity</field>
            <field name="context">{"search_default_filter_to_sell":1}</field>
            <field name="view_id" ref="product_product_tree_view"/>
            <field name="search_view_id" ref="product_search_form_view"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new product variant
              </p><p>
                You must define a product for everything you sell, whether it's a physical product,
                a consumable or a service you offer to customers.
                The product form contains information to simplify the sale process:
                price, notes in the quotation, accounting data, procurement methods, etc.
              </p>
            </field>
        </record>

        <record id="product_category_search_view" model="ir.ui.view">
            <field name="name">product.category.search</field>
            <field name="model">product.category</field>
            <field name="arch" type="xml">
                <search string="Product Categories">
                    <field name="name" string="Product Categories"/>
                    <field name="parent_id"/>
                </search>
            </field>
        </record>
        <record id="product_category_form_view" model="ir.ui.view">
            <field name="name">product.category.form</field>
            <field name="model">product.category</field>
            <field name="arch" type="xml">
                <form class="oe_form_configuration">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button"
                                name="%(product_template_action_all)d"
                                icon="fa-th-list"
                                type="action"
                                context="{'search_default_categ_id': active_id, 'default_categ_id': active_id, 'group_expand': True}">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="product_count"/></span>
                                    <span class="o_stat_text"> Products</span>
                                </div>
                            </button>
                        </div>
                        <div class="oe_title">
                            <label for="name" string="Category name" class="oe_edit_only"/>
                            <h1><field name="name" placeholder="e.g. Lamps"/></h1>
                        </div>
                        <group name="first" col="2">
                            <field name="parent_id" class="oe_inline"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>
        <record id="product_category_list_view" model="ir.ui.view">
            <field name="name">product.category.list</field>
            <field name="model">product.category</field>
            <field name="priority">1</field>
            <field name="arch" type="xml">
                <tree string="Product Categories">
                    <field name="display_name" string="Product Category"/>
                </tree>
            </field>
        </record>
        <record id="product_category_action_form" model="ir.actions.act_window">
            <field name="name">Product Categories</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">product.category</field>
            <field name="search_view_id" ref="product_category_search_view"/>
            <field name="view_id" ref="product_category_list_view"/>
        </record>


        <record id="product_packaging_tree_view" model="ir.ui.view">
            <field name="name">product.packaging.tree.view</field>
            <field name="model">product.packaging</field>
            <field name="arch" type="xml">
                <tree string="Product Packagings">
                    <field name="sequence" widget="handle"/>
                    <field name="product_id"/>
                    <field name="name" string="Packaging"/>
                    <field name="qty"/>
                    <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"/>
                    <field name="company_id" group="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="product_packaging_tree_view2" model="ir.ui.view">
            <field name="name">product.packaging.tree.view2</field>
            <field name="model">product.packaging</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="product.product_packaging_tree_view"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='product_id']" position="replace"/>
                <xpath expr="//field[@name='sequence']" position="replace"/>
            </field>
        </record>

        <record id="product_packaging_form_view" model="ir.ui.view">
            <field name="name">product.packaging.form.view</field>
            <field name="model">product.packaging</field>
            <field name="arch" type="xml">
                <form string="Product Packaging">
                    <sheet>
                        <label for="name" string="Packaging"/>
                        <h1>
                            <field name="name"/>
                        </h1>
                        <group>
                            <field name="id" invisible='1'/>
                            <group name="group_product">
                                <field name="product_id"  required='True' attrs="{'readonly': [('id', '!=', False)]}"/>
                            </group>
                            <group name="qty">
                                <label for="qty" string="Contained quantity"/>
                                <div class="o_row">
                                    <field name="qty"/>
                                    <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"/>
                                </div>
                                <field name="barcode"/>
                                <field name="company_id" group="base.group_multi_company"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="product_packaging_form_view2" model="ir.ui.view">
            <field name="name">product.packaging.form.view2</field>
            <field name="model">product.packaging</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="product.product_packaging_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='group_product']" position="replace"/>
                <xpath expr="//field[@name='id']" position="replace"/>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_packaging_view">
            <field name="name">Product Packagings</field>
            <field name="res_model">product.packaging</field>
            <field name="domain">[('product_id', '!=', False)]</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('product_packaging_tree_view')}),
                (0, 0, {'view_mode': 'form', 'view_id': ref('product_packaging_form_view')})]"/>
        </record>

        <record id="product_supplierinfo_form_view" model="ir.ui.view">
            <field name="name">product.supplierinfo.form.view</field>
            <field name="model">product.supplierinfo</field>
            <field name="arch" type="xml">
                <form string="Vendor Information">
                    <group>
                        <group name="vendor" string="Vendor">
                            <field name="product_variant_count" invisible="1"/>
                            <field name="name" context="{'res_partner_search_mode': 'supplier'}"/>
                            <field name="product_name"/>
                            <field name="product_code"/>
                            <field name="product_id" groups="product.group_product_variant" domain="[('product_tmpl_id', '=', product_tmpl_id)]" options="{'no_create_edit': True}"/>
                            <label for="delay"/>
                            <div>
                                <field name="delay" class="oe_inline"/> days
                            </div>
                        </group>
                        <group string="Price List">
                            <field name="product_tmpl_id" string="Product" invisible="context.get('visible_product_tmpl_id', True)"/>
                            <label for="min_qty"/>
                            <div class="o_row">
                                <field name="min_qty"/>
                                <field name="product_uom" groups="uom.group_uom"/>
                            </div>
                            <label for="price"/>
                            <div class="o_row">
                                <field name="price"/><field name="currency_id" groups="base.group_multi_currency"/>
                            </div>
                            <label for="date_start" string="Validity"/>
                            <div class="o_row"><field name="date_start"/> to <field name="date_end"/></div>
                        </group>
                        <group string="Other Information" groups="base.group_multi_company">
                            <field name="company_id" options="{'no_create': True}"/>
                        </group>
                    </group>
                </form>
            </field>
        </record>

        <record id="product_supplierinfo_search_view" model="ir.ui.view">
            <field name="name">product.supplierinfo.search.view</field>
            <field name="model">product.supplierinfo</field>
            <field name="arch" type="xml">
                <search string="Vendor">
                    <field name="name"/>
                    <field name="product_tmpl_id"/>
                    <filter string="Active" name="active" domain="['|', ('date_end', '=', False), ('date_end', '&gt;=',  (context_today() - datetime.timedelta(days=1)).strftime('%%Y-%%m-%%d'))]"/>
                    <filter string="Archived" name="archived" domain="[('date_end', '&lt;',  (context_today() - datetime.timedelta(days=1)).strftime('%%Y-%%m-%%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Product" name="groupby_product" domain="[]" context="{'group_by': 'product_tmpl_id'}"/>
                        <filter string="Vendor" name="groupby_vendor" domain="[]" context="{'group_by': 'name'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="product_supplierinfo_view_kanban" model="ir.ui.view">
            <field name="name">product.supplierinfo.kanban</field>
            <field name="model">product.supplierinfo</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="min_qty"/>
                    <field name="delay"/>
                    <field name="price"/>
                    <field name="name"/>
                    <field name="currency_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div class="row mb4">
                                    <strong class="col-6">
                                        <span t-esc="record.name.value"/>
                                    </strong>
                                    <strong class="col-6 text-right">
                                        <strong><field name="price" widget="monetary"/></strong>
                                    </strong>
                                    <div class="col-6">
                                        <span t-esc="record.min_qty.value"/>
                                    </div>
                                    <div class="col-6 text-right">
                                        <span t-esc="record.delay.value"/> days
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="product_supplierinfo_tree_view" model="ir.ui.view">
            <field name="name">product.supplierinfo.tree.view</field>
            <field name="model">product.supplierinfo</field>
            <field name="arch" type="xml">
                <tree string="Vendor Information" multi_edit="1">
                    <field name="sequence" widget="handle"/>
                    <field name="name" readonly="1"/>
                    <field name="product_id" readonly="1" optional="hide"
                        invisible="context.get('product_template_invisible_variant', False)"
                        groups="product.group_product_variant"/>
                    <field name="product_tmpl_id" string="Product" readonly="1"
                        invisible="context.get('visible_product_tmpl_id', True)"/>
                    <field name="product_name" optional="hide"/>
                    <field name="product_code" optional="hide"/>
                    <field name="currency_id" groups="base.group_multi_currency"/>
                    <field name="date_start" optional="hide"/>
                    <field name="date_end" optional="hide"/>
                    <field name="company_id" readonly="1" groups="base.group_multi_company"/>
                    <field name="min_qty"/>
                    <field name="product_uom" groups="uom.group_uom"/>
                    <field name="price" string="Price"/>
                </tree>
            </field>
        </record>

    <record id="product_supplierinfo_type_action" model="ir.actions.act_window">
        <field name="name">Vendor Pricelists</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.supplierinfo</field>
        <field name="view_mode">tree,form,kanban</field>
        <field name="context">{'visible_product_tmpl_id':False}</field>
    </record>

    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.product</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@id='multi_company']" position="before">
                    <h2>Products</h2>
                    <div class="row mt16 o_settings_container" id="product_general_settings">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="product_weight_in_lbs" string="Weight Measurement"/>
                                <div class="text-muted">
                                    Choose the unit to measure weight
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="product_weight_in_lbs" class="o_light_label" widget="radio"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_right_pane">
                                <label for="product_volume_volume_in_cubic_feet"/>
                                <div class="text-muted">
                                    In which unit of measure do you manage your volumes
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="product_volume_volume_in_cubic_feet" class="o_light_label" widget="radio"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_partner_property_form" model="ir.ui.view">
            <field name="name">res.partner.product.property.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="groups_id" eval="[(4, ref('product.group_product_pricelist'))]"/>
            <field name="arch" type="xml">
                <group name="sale">
                    <field name="property_product_pricelist" groups="product.group_product_pricelist" attrs="{'invisible': [('is_company','=',False),('parent_id','!=',False)]}"/>
                    <div name="parent_pricelists" groups="product.group_product_pricelist" colspan="2" attrs="{'invisible': ['|',('is_company','=',True),('parent_id','=',False)]}">
                        <p>Pricelists are managed on <button name="open_commercial_entity" type="object" string="the parent company" class="oe_link"/></p>
                    </div>
                </group>
            </field>
        </record>
</odoo>

```

## File: wizard\product_price_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class product_price_list(models.TransientModel):
    _name = 'product.price_list'
    _description = 'Product Price per Unit Based on Pricelist Version'

    price_list = fields.Many2one('product.pricelist', 'PriceList', required=True)
    qty1 = fields.Integer('Quantity-1', default=1)
    qty2 = fields.Integer('Quantity-2', default=5)
    qty3 = fields.Integer('Quantity-3', default=10)
    qty4 = fields.Integer('Quantity-4', default=0)
    qty5 = fields.Integer('Quantity-5', default=0)

    def print_report(self):
        """
        To get the date and print the report
        @return : return report
        """

        datas = {'ids': self.env.context.get('active_ids', [])}
        res = self.read(['price_list', 'qty1', 'qty2', 'qty3', 'qty4', 'qty5'])
        res = res and res[0] or {}
        res['price_list'] = res['price_list'][0]
        datas['form'] = res
        return self.env.ref('product.action_report_pricelist').report_action([], data=datas)

```

## File: wizard\product_price_list_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!--   Product Price List -->
        <record id="view_product_price_list" model="ir.ui.view">
              <field name="name">Price per unit</field>
              <field name="model">product.price_list</field>
              <field name="arch" type="xml">
                <form string="Price List">
                    <group string="Calculate Product Price per Unit Based on Pricelist Version.">
                        <field name="price_list" widget="selection"/>
                        <field name="qty1"/>
                        <field name="qty2"/>
                        <field name="qty3"/>
                        <field name="qty4"/>
                        <field name="qty5"/>
                    </group>
                    <footer>
                        <button name="print_report" string="Print"  type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" />
                    </footer>
                </form>
              </field>
        </record>

        <act_window id="action_product_price_list"
            name="Price List"
            res_model="product.price_list"
            binding_model="product.product"
            binding_type="report"
            groups="product.group_product_pricelist"
            view_mode="form" target="new" />
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_price_list

```

