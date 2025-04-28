# Odoo Module: product

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import populate
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

        'wizard/product_label_layout_views.xml',

        'views/product_views.xml',

        'views/res_config_settings_views.xml',
        'views/product_attribute_views.xml',
        'views/product_category_views.xml',
        'views/product_packaging_views.xml',
        'views/product_pricelist_item_views.xml',
        'views/product_pricelist_views.xml',
        'views/product_supplierinfo_views.xml',
        'views/product_template_views.xml',
        'views/product_tag_views.xml',
        'views/res_country_group_views.xml',
        'views/res_partner_views.xml',

        'report/product_reports.xml',
        'report/product_product_templates.xml',
        'report/product_template_templates.xml',
        'report/product_packaging.xml',
        'report/product_pricelist_report_templates.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'product/static/src/js/**/*',
            'product/static/src/xml/**/*',
        ],
        'web.report_assets_common': [
            'product/static/src/scss/report_label_sheet.scss',
        ],
        'web.qunit_suite_tests': [
            'product/static/tests/**/*',
        ],
    },
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
            <field name="digits" eval="2"/>
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
            <field name="detailed_type">service</field>
            <field name="categ_id" ref="product.cat_expense"/>
        </record>

        <record id="expense_hotel" model="product.product">
            <field name="name">Hotel Accommodation</field>
            <field name="list_price">400.0</field>
            <field name="standard_price">400.0</field>
            <field name="detailed_type">service</field>
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
            <field name="detailed_type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
        </record>

        <record id="product_product_2" model="product.product">
            <field name="name">Virtual Home Staging</field>
            <field name="categ_id" ref="product_category_3"/>
            <field name="standard_price">25.5</field>
            <field name="list_price">38.25</field>
            <field name="detailed_type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
        </record>

        <!-- Physical Products -->

        <record id="product_delivery_01" model="product.product">
            <field name="name">Office Chair</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">55.0</field>
            <field name="list_price">70.0</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_7777</field>
            <field name="image_1920" type="base64" file="product/static/img/product_chair.jpg"/>
        </record>

        <record id="product_delivery_02" model="product.product">
            <field name="name">Office Lamp</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">35.0</field>
            <field name="list_price">40.0</field>
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">consu</field>
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
            <field name="name">Customizable Desk</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">500.0</field>
            <field name="list_price">750.0</field>
            <field name="detailed_type">consu</field>
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
            },]"/>
        </function>

        <record id="product_product_4" model="product.product">
            <field name="default_code">FURN_0096</field>
            <field name="standard_price">500.0</field>
            <field name="weight">0.01</field>
            <field name="image_1920" type="base64" file="product/static/img/table02.jpg"/>
        </record>
        <record id="product_product_4b" model="product.product">
            <field name="default_code">FURN_0097</field>
            <field name="weight">0.01</field>
            <field name="standard_price">500.0</field>
            <field name="image_1920" type="base64" file="product/static/img/table04.jpg"/>
        </record>
        <record id="product_product_4c" model="product.product">
            <field name="default_code">FURN_0098</field>
            <field name="weight">0.01</field>
            <field name="standard_price">500.0</field>
            <field name="image_1920" type="base64" file="product/static/img/table03.jpg"/>
        </record>

        <record id="product_product_5" model="product.product">
            <field name="name">Corner Desk Right Sit</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">600.0</field>
            <field name="list_price">147.0</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM06</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_5-image.jpg"/>
        </record>

        <record id="product_product_6" model="product.product">
            <field name="name">Large Cabinet</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">800.0</field>
            <field name="list_price">320.0</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM07</field>
            <field name='weight'>0.330</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_6-image.jpg"/>
        </record>

        <record id="product_product_7" model="product.product">
            <field name="name">Storage Box</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">14.0</field>
            <field name="list_price">15.8</field>
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM10</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_9-image.jpg"/>
        </record>

        <record id="product_product_10" model="product.product">
            <field name="name">Cabinet with Doors</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">120.50</field>
            <field name="list_price">140</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">E-COM11</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_10-image.jpg"/>
        </record>

        <record id="product_product_11_product_template" model="product.template">
            <field name="name">Conference Chair</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">28</field>
            <field name="list_price">33</field>
            <field name="detailed_type">consu</field>
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
            <field name="standard_price">180</field>
            <field name="list_price">120.50</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_0269</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_12-image.png"/>
        </record>

        <record id="product_product_13" model="product.product">
            <field name="name">Corner Desk Left Sit</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">78.0</field>
            <field name="list_price">85.0</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_1118</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_13-image.jpg"/>
        </record>

        <record id="product_product_16" model="product.product">
            <field name="name">Drawer Black</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">20.0</field>
            <field name="list_price">25.0</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_8900</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_16-image.jpg"/>
        </record>

        <record id="product_product_20" model="product.product">
            <field name="name">Flipover</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">1700.0</field>
            <field name="list_price">1950.0</field>
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">consu</field>
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
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_0789</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_24-image.jpg"/>
        </record>

        <record id="product_product_25" model="product.product">
            <field name="name">Acoustic Bloc Screens</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">287.0</field>
            <field name="list_price">295.0</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FURN_6666</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_25-image.png"/>
        </record>

        <record id="product_product_27" model="product.product">
            <field name="name">Drawer</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">100.0</field>
            <field name="list_price">110.50</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description">Drawer with two routing possiblities.</field>
            <field name="default_code">FURN_8855</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_27-image.jpg"/>
        </record>

        <record id="consu_delivery_03" model="product.product">
            <field name="name">Four Person Desk</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">2500.0</field>
            <field name="list_price">2350.0</field>
            <field name="detailed_type">consu</field>
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
            <field name="standard_price">4500.0</field>
            <field name="list_price">4000.0</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Conference room table</field>
            <field name="default_code">FURN_6741</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_46-image.jpg"/>
        </record>

        <record id="consu_delivery_01" model="product.product">
            <field name="name">Three-Seat Sofa</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">1000</field>
            <field name="list_price">1500</field>
            <field name="detailed_type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Three Seater Sofa with Lounger in Steel Grey Colour</field>
            <field name="default_code">FURN_8999</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_d01-image.jpg"/>
        </record>

        <!--
    Resource: product.supplierinfo
    -->

        <record id="product_supplierinfo_1" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_6_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">750</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_2" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_6_product_template"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">790</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_2bis" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_6_product_template"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">3</field>
            <field name="price">785</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_3" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_7_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">13.0</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_4" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_7_product_template"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">14.4</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_5" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_8_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">2</field>
            <field name="min_qty">5</field>
            <field name="price">1299</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_6" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_8_product_template"/>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="delay">4</field>
            <field name="min_qty">1</field>
            <field name="price">1399</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_7" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_10_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">2</field>
            <field name="min_qty">1</field>
            <field name="price">120.50</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_8" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_11_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">2</field>
            <field name="min_qty">1</field>
            <field name="price">28</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_9" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_13_product_template"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="delay">5</field>
            <field name="min_qty">1</field>
            <field name="price">78</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_10" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_16_product_template"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="delay">1</field>
            <field name="min_qty">1</field>
            <field name="price">20</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_12" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_20_product_template"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">1700</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_13" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_20_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">4</field>
            <field name="min_qty">5</field>
            <field name="price">1720</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_14" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_22_product_template"/>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">2010</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_15" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_24_product_template"/>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="delay">3</field>
            <field name="min_qty">1</field>
            <field name="price">876</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_16" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_25_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">8</field>
            <field name="min_qty">1</field>
            <field name="price">287</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_17" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_02_product_template"/>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="delay">4</field>
            <field name="min_qty">1</field>
            <field name="price">390</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_18" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_01_product_template"/>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="delay">2</field>
            <field name="min_qty">12</field>
            <field name="price">90</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_19" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_01_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">4</field>
            <field name="min_qty">1</field>
            <field name="price">66</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_20" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_02_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">5</field>
            <field name="min_qty">1</field>
            <field name="price">35</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_21" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_delivery_01_product_template"/>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="delay">7</field>
            <field name="min_qty">1</field>
            <field name="price">55</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_22" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_9_product_template"/>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="delay">4</field>
            <field name="min_qty">0</field>
            <field name="price">10</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_23" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_27_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">10</field>
            <field name="min_qty">0</field>
            <field name="price">95.50</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_24" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_12_product_template"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="delay">3</field>
            <field name="min_qty">0</field>
            <field name="price">120.50</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_25" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_12_product_template"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="delay">2</field>
            <field name="min_qty">0</field>
            <field name="price">130.50</field>
            <field name="currency_id" ref="base.USD"/>
        </record>

        <record id="product_supplierinfo_26" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product_product_5_product_template"/>
            <field name="partner_id" ref="base.res_partner_10"/>
            <field name="delay">1</field>
            <field name="min_qty">0</field>
            <field name="price">145</field>
            <field name="currency_id" ref="base.USD"/>
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

## File: models\product_attribute.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

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
    sequence = fields.Integer('Sequence', help="Determine the display order", index=True, default=20)
    attribute_line_ids = fields.One2many('product.template.attribute.line', 'attribute_id', 'Lines')
    create_variant = fields.Selection([
        ('always', 'Instantly'),
        ('dynamic', 'Dynamically'),
        ('no_variant', 'Never (option)')],
        default='always',
        string="Variants Creation Mode",
        help="""- Instantly: All possible variants are created as soon as the attribute and its values are added to a product.
        - Dynamically: Each variant is created only when its corresponding attributes and values are added to a sales order.
        - Never: Variants are never created for the attribute.
        Note: the variants creation mode cannot be changed once the attribute is used on at least one product.""",
        required=True)
    number_related_products = fields.Integer(compute='_compute_number_related_products')
    product_tmpl_ids = fields.Many2many('product.template', string="Related Products", compute='_compute_products', store=True)
    display_type = fields.Selection([
        ('radio', 'Radio'),
        ('pills', 'Pills'),
        ('select', 'Select'),
        ('color', 'Color')], default='radio', required=True, help="The display type used in the Product Configurator.")

    @api.depends('product_tmpl_ids')
    def _compute_number_related_products(self):
        for pa in self:
            pa.number_related_products = len(pa.product_tmpl_ids)

    @api.depends('attribute_line_ids.active', 'attribute_line_ids.product_tmpl_id')
    def _compute_products(self):
        for pa in self:
            pa.with_context(active_test=False).product_tmpl_ids = pa.attribute_line_ids.product_tmpl_id

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
                if vals['create_variant'] != pa.create_variant and pa.number_related_products:
                    raise UserError(
                        _("You cannot change the Variants Creation Mode of the attribute %s because it is used on the following products:\n%s") %
                        (pa.display_name, ", ".join(pa.product_tmpl_ids.mapped('display_name')))
                    )
        invalidate = 'sequence' in vals and any(record.sequence != vals['sequence'] for record in self)
        res = super(ProductAttribute, self).write(vals)
        if invalidate:
            # prefetched o2m have to be resequenced
            # (eg. product.template: attribute_line_ids)
            self.env.flush_all()
            self.env.invalidate_all()
        return res

    @api.ondelete(at_uninstall=False)
    def _unlink_except_used_on_product(self):
        for pa in self:
            if pa.number_related_products:
                raise UserError(
                    _("You cannot delete the attribute %s because it is used on the following products:\n%s") %
                    (pa.display_name, ", ".join(pa.product_tmpl_ids.mapped('display_name')))
                )

    def action_open_related_products(self):
        return {
            'type': 'ir.actions.act_window',
            'name': _("Related Products"),
            'res_model': 'product.template',
            'view_mode': 'tree,form',
            'domain': [('id', 'in', self.with_context(active_test=False).product_tmpl_ids.ids)],
        }


class ProductAttributeValue(models.Model):
    _name = "product.attribute.value"
    # if you change this _order, keep it in sync with the method
    # `_sort_key_variant` in `product.template'
    _order = 'attribute_id, sequence, id'
    _description = 'Attribute Value'

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char(string='Value', required=True, translate=True)
    sequence = fields.Integer(string='Sequence', help="Determine the display order", index=True)
    attribute_id = fields.Many2one('product.attribute', string="Attribute", ondelete='cascade', required=True, index=True,
        help="The attribute cannot be changed once the value is used on at least one product.")

    pav_attribute_line_ids = fields.Many2many('product.template.attribute.line', string="Lines",
        relation='product_attribute_value_product_template_attribute_line_rel', copy=False)
    is_used_on_products = fields.Boolean('Used on Products', compute='_compute_is_used_on_products')

    is_custom = fields.Boolean('Is custom value', help="Allow users to input custom values for this attribute value")
    html_color = fields.Char(
        string='Color',
        help="Here you can set a specific HTML color index (e.g. #ff0000) to display the color if the attribute type is 'Color'.")
    display_type = fields.Selection(related='attribute_id.display_type', readonly=True)
    color = fields.Integer('Color Index', default=_get_default_color)

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

        invalidate = 'sequence' in values and any(record.sequence != values['sequence'] for record in self)
        res = super(ProductAttributeValue, self).write(values)
        if invalidate:
            # prefetched o2m have to be resequenced
            # (eg. product.template.attribute.line: value_ids)
            self.env.flush_all()
            self.env.invalidate_all()
        return res

    @api.ondelete(at_uninstall=False)
    def _unlink_except_used_on_product(self):
        for pav in self:
            if pav.is_used_on_products:
                raise UserError(
                    _("You cannot delete the value %s because it is used on the following products:"
                      "\n%s\n If the value has been associated to a product in the past, you will "
                      "not be able to delete it.") %
                    (pav.display_name, ", ".join(
                        pav.pav_attribute_line_ids.product_tmpl_id.mapped('display_name')
                    ))
                )
            linked_products = pav.env['product.template.attribute.value'].search(
                [('product_attribute_value_id', '=', pav.id)]
            ).with_context(active_test=False).ptav_product_variant_ids
            unlinkable_products = linked_products._filter_to_unlink()
            if linked_products != unlinkable_products:
                raise UserError(_(
                    "You cannot delete value %s because it was used in some products.",
                    pav.display_name
                ))

    def _without_no_variant_attributes(self):
        return self.filtered(lambda pav: pav.attribute_id.create_variant != 'no_variant')


class ProductTemplateAttributeLine(models.Model):
    """Attributes available on product.template with their selected values in a m2m.
    Used as a configuration model to generate the appropriate product.template.attribute.value"""

    _name = "product.template.attribute.line"
    _rec_name = 'attribute_id'
    _rec_names_search = ['attribute_id', 'value_ids']
    _description = 'Product Template Attribute Line'
    _order = 'attribute_id, id'

    active = fields.Boolean(default=True)
    product_tmpl_id = fields.Many2one('product.template', string="Product Template", ondelete='cascade', required=True, index=True)
    attribute_id = fields.Many2one('product.attribute', string="Attribute", ondelete='restrict', required=True, index=True)
    value_ids = fields.Many2many('product.attribute.value', string="Values", domain="[('attribute_id', '=', attribute_id)]",
                                 relation='product_attribute_value_product_template_attribute_line_rel', ondelete='restrict')
    value_count = fields.Integer(compute='_compute_value_count', store=True, readonly=True)
    product_template_value_ids = fields.One2many('product.template.attribute.value', 'attribute_line_id', string="Product Attribute Values")

    @api.depends('value_ids')
    def _compute_value_count(self):
        for record in self:
            record.value_count = len(record.value_ids)

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
            self.env.flush_all()
            self.env['product.template'].invalidate_model(['attribute_line_ids'])
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
        ptal_to_archive.action_archive()  # only calls write if there are records
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

    def _without_no_variant_attributes(self):
        return self.filtered(lambda ptal: ptal.attribute_id.create_variant != 'no_variant')

    def action_open_attribute_values(self):
        return {
            'type': 'ir.actions.act_window',
            'name': _("Product Variant Values"),
            'res_model': 'product.template.attribute.value',
            'view_mode': 'tree,form',
            'domain': [('id', 'in', self.product_template_value_ids.ids)],
            'views': [
                (self.env.ref('product.product_template_attribute_value_view_tree').id, 'list'),
                (self.env.ref('product.product_template_attribute_value_view_form').id, 'form'),
            ],
            'context': {
                'search_default_active': 1,
            },
        }


class ProductTemplateAttributeValue(models.Model):
    """Materialized relationship between attribute values
    and product template generated by the product.template.attribute.line"""

    _name = "product.template.attribute.value"
    _description = "Product Template Attribute Value"
    _order = 'attribute_line_id, product_attribute_value_id, id'

    def _get_default_color(self):
        return randint(1, 11)

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
    currency_id = fields.Many2one(related='attribute_line_id.product_tmpl_id.currency_id')

    exclude_for = fields.One2many(
        'product.template.attribute.exclusion',
        'product_template_attribute_value_id',
        string="Exclude for",
        help="Make this attribute value not compatible with "
             "other values of the product or some attribute values of optional and accessory products.")

    # related fields: product template and product attribute
    product_tmpl_id = fields.Many2one('product.template', string="Product Template", related='attribute_line_id.product_tmpl_id', store=True, index=True)
    attribute_id = fields.Many2one('product.attribute', string="Attribute", related='attribute_line_id.attribute_id', store=True, index=True)
    ptav_product_variant_ids = fields.Many2many('product.product', relation='product_variant_combination', string="Related Variants", readonly=True)

    html_color = fields.Char('HTML Color Index', related="product_attribute_value_id.html_color")
    is_custom = fields.Boolean('Is custom value', related="product_attribute_value_id.is_custom")
    display_type = fields.Selection(related='product_attribute_value_id.display_type', readonly=True)
    color = fields.Integer('Color', default=_get_default_color)

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

    @api.model_create_multi
    def create(self, vals_list):
        exclusions = super().create(vals_list)
        exclusions.product_tmpl_id._create_variant_ids()
        return exclusions

    def unlink(self):
        # Keep a reference to the related templates before the deletion.
        templates = self.product_tmpl_id
        res = super().unlink()
        templates._create_variant_ids()
        return res

    def write(self, values):
        templates = self.env['product.template']
        if 'product_tmpl_id' in values:
            templates = self.product_tmpl_id
        res = super().write(values)
        (templates | self.product_tmpl_id)._create_variant_ids()
        return res


class ProductAttributeCustomValue(models.Model):
    _name = "product.attribute.custom.value"
    _description = 'Product Attribute Custom Value'
    _order = 'custom_product_template_attribute_value_id, id'

    name = fields.Char("Name", compute='_compute_name')
    custom_product_template_attribute_value_id = fields.Many2one('product.template.attribute.value', string="Attribute Value", required=True, ondelete='restrict')
    custom_value = fields.Char("Custom Value")

    @api.depends('custom_product_template_attribute_value_id.name', 'custom_value')
    def _compute_name(self):
        for record in self:
            name = (record.custom_value or '').strip()
            if record.custom_product_template_attribute_value_id.display_name:
                name = "%s: %s" % (record.custom_product_template_attribute_value_id.display_name, name)
            record.name = name

```

## File: models\product_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError


class ProductCategory(models.Model):
    _name = "product.category"
    _description = "Product Category"
    _parent_name = "parent_id"
    _parent_store = True
    _rec_name = 'complete_name'
    _order = 'complete_name'

    name = fields.Char('Name', index='trigram', required=True)
    complete_name = fields.Char(
        'Complete Name', compute='_compute_complete_name', recursive=True,
        store=True)
    parent_id = fields.Many2one('product.category', 'Parent Category', index=True, ondelete='cascade')
    parent_path = fields.Char(index=True, unaccent=False)
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

    @api.model
    def name_create(self, name):
        return self.create({'name': name}).name_get()[0]

    def name_get(self):
        if not self.env.context.get('hierarchical_naming', True):
            return [(record.id, record.name) for record in self]
        return super().name_get()

    @api.ondelete(at_uninstall=False)
    def _unlink_except_default_category(self):
        main_category = self.env.ref('product.product_category_all', raise_if_not_found=False)
        if main_category and main_category in self:
            raise UserError(_("You cannot delete this product category, it is the default generic category."))
        expense_category = self.env.ref('product.cat_expense', raise_if_not_found=False)
        if expense_category and expense_category in self:
            raise UserError(_("You cannot delete the %s product category.", expense_category.name))
        saleable_category = self.env.ref('product.product_category_1', raise_if_not_found=False)
        if saleable_category and saleable_category in self:
            raise UserError(_("You cannot delete the %s product category.", saleable_category.name))

```

## File: models\product_packaging.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.osv import expression


from odoo.tools import float_compare, float_round


class ProductPackaging(models.Model):
    _name = "product.packaging"
    _description = "Product Packaging"
    _order = 'product_id, sequence, id'
    _check_company_auto = True

    name = fields.Char('Product Packaging', required=True)
    sequence = fields.Integer('Sequence', default=1, help="The first in the sequence is the default one.")
    product_id = fields.Many2one('product.product', string='Product', check_company=True)
    qty = fields.Float('Contained Quantity', default=1, digits='Product Unit of Measure', help="Quantity of products contained in the packaging.")
    barcode = fields.Char('Barcode', copy=False, help="Barcode used for packaging identification. Scan this packaging barcode from a transfer in the Barcode app to move all the contained units")
    product_uom_id = fields.Many2one('uom.uom', related='product_id.uom_id', readonly=True)
    company_id = fields.Many2one('res.company', 'Company', index=True)

    _sql_constraints = [
        ('positive_qty', 'CHECK(qty > 0)', 'Contained Quantity should be positive.'),
        ('barcode_uniq', 'unique(barcode)', 'A barcode can only be assigned to one packaging.'),
    ]

    @api.constrains('barcode')
    def _check_barcode_uniqueness(self):
        """ With GS1 nomenclature, products and packagings use the same pattern. Therefore, we need
        to ensure the uniqueness between products' barcodes and packagings' ones"""
        domain = [('barcode', 'in', [b for b in self.mapped('barcode') if b])]
        if self.env['product.product'].search(domain, order="id", limit=1):
            raise ValidationError(_("A product already uses the barcode"))

    def _check_qty(self, product_qty, uom_id, rounding_method="HALF-UP"):
        """Check if product_qty in given uom is a multiple of the packaging qty.
        If not, rounding the product_qty to closest multiple of the packaging qty
        according to the rounding_method "UP", "HALF-UP or "DOWN".
        """
        self.ensure_one()
        default_uom = self.product_id.uom_id
        packaging_qty = default_uom._compute_quantity(self.qty, uom_id)
        # We do not use the modulo operator to check if qty is a mltiple of q. Indeed the quantity
        # per package might be a float, leading to incorrect results. For example:
        # 8 % 1.6 = 1.5999999999999996
        # 5.4 % 1.8 = 2.220446049250313e-16
        if product_qty and packaging_qty:
            rounded_qty = float_round(product_qty / packaging_qty, precision_rounding=1.0,
                                  rounding_method=rounding_method) * packaging_qty
            return rounded_qty if float_compare(rounded_qty, product_qty, precision_rounding=default_uom.rounding) else product_qty
        return product_qty

    def _find_suitable_product_packaging(self, product_qty, uom_id):
        """ try find in `self` if a packaging's qty in given uom is a divisor of
        the given product_qty. If so, return the one with greatest divisor.
        """
        packagings = self.sorted(lambda p: p.qty, reverse=True)
        for packaging in packagings:
            new_qty = packaging._check_qty(product_qty, uom_id)
            if new_qty == product_qty:
                return packaging
        return self.env['product.packaging']

    def write(self, vals):
        res = super().write(vals)
        if res and not vals.get('product_id', True):
            self.unlink()
        return res

```

## File: models\product_pricelist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class Pricelist(models.Model):
    _name = "product.pricelist"
    _description = "Pricelist"
    _rec_names_search = ['name', 'currency_id']  # TODO check if should be removed
    _order = "sequence asc, id desc"

    def _default_currency_id(self):
        return self.env.company.currency_id.id

    name = fields.Char(string="Pricelist Name", required=True, translate=True)

    active = fields.Boolean(
        string="Active",
        default=True,
        help="If unchecked, it will allow you to hide the pricelist without removing it.")
    sequence = fields.Integer(default=16)

    currency_id = fields.Many2one(
        comodel_name='res.currency',
        default=_default_currency_id,
        required=True)

    company_id = fields.Many2one(
        comodel_name='res.company')
    country_group_ids = fields.Many2many(
        comodel_name='res.country.group',
        relation='res_country_group_pricelist_rel',
        column1='pricelist_id',
        column2='res_country_group_id',
        string="Country Groups")

    discount_policy = fields.Selection(
        selection=[
            ('with_discount', "Discount included in the price"),
            ('without_discount', "Show public price & discount to the customer"),
        ],
        default='with_discount',
        required=True)

    item_ids = fields.One2many(
        comodel_name='product.pricelist.item',
        inverse_name='pricelist_id',
        string="Pricelist Rules",
        copy=True)

    def name_get(self):
        return [(pricelist.id, '%s (%s)' % (pricelist.name, pricelist.currency_id.name)) for pricelist in self]

    def _get_products_price(self, products, quantity, uom=None, date=False, **kwargs):
        """Compute the pricelist prices for the specified products, qty & uom.

        Note: self.ensure_one()

        :returns: dict{product_id: product price}, considering the current pricelist
        :rtype: dict
        """
        self.ensure_one()
        return {
            product_id: res_tuple[0]
            for product_id, res_tuple in self._compute_price_rule(
                products,
                quantity,
                uom=uom,
                date=date,
                **kwargs
            ).items()
        }

    def _get_product_price(self, product, quantity, uom=None, date=False, **kwargs):
        """Compute the pricelist price for the specified product, qty & uom.

        Note: self.ensure_one()

        :returns: unit price of the product, considering pricelist rules
        :rtype: float
        """
        self.ensure_one()
        return self._compute_price_rule(
            product, quantity, uom=uom, date=date, **kwargs
        )[product.id][0]

    def _get_product_price_rule(self, product, quantity, uom=None, date=False, **kwargs):
        """Compute the pricelist price & rule for the specified product, qty & uom.

        Note: self.ensure_one()

        :returns: (product unit price, applied pricelist rule id)
        :rtype: tuple(float, int)
        """
        self.ensure_one()
        return self._compute_price_rule(product, quantity, uom=uom, date=date, **kwargs)[product.id]

    def _get_product_rule(self, product, quantity, uom=None, date=False, **kwargs):
        """Compute the pricelist price & rule for the specified product, qty & uom.

        Note: self.ensure_one()

        :returns: applied pricelist rule id
        :rtype: int or False
        """
        self.ensure_one()
        return self._compute_price_rule(
            product, quantity, uom=uom, date=date, **kwargs
        )[product.id][1]

    def _compute_price_rule(self, products, qty, uom=None, date=False, **kwargs):
        """ Low-level method - Mono pricelist, multi products
        Returns: dict{product_id: (price, suitable_rule) for the given pricelist}

        :param products: recordset of products (product.product/product.template)
        :param float qty: quantity of products requested (in given uom)
        :param uom: unit of measure (uom.uom record)
            If not specified, prices returned are expressed in product uoms
        :param date: date to use for price computation and currency conversions
        :type date: date or datetime

        :returns: product_id: (price, pricelist_rule)
        :rtype: dict
        """
        self.ensure_one()

        if not products:
            return {}

        if not date:
            # Used to fetch pricelist rules and currency rates
            date = fields.Datetime.now()

        # Fetch all rules potentially matching specified products/templates/categories and date
        rules = self._get_applicable_rules(products, date, **kwargs)

        results = {}
        for product in products:
            suitable_rule = self.env['product.pricelist.item']

            product_uom = product.uom_id
            target_uom = uom or product_uom  # If no uom is specified, fall back on the product uom

            # Compute quantity in product uom because pricelist rules are specified
            # w.r.t product default UoM (min_quantity, price_surchage, ...)
            if target_uom != product_uom:
                qty_in_product_uom = target_uom._compute_quantity(qty, product_uom, raise_if_failure=False)
            else:
                qty_in_product_uom = qty

            for rule in rules:
                if rule._is_applicable_for(product, qty_in_product_uom):
                    suitable_rule = rule
                    break

            kwargs['pricelist'] = self
            price = suitable_rule._compute_price(product, qty, target_uom, date=date, currency=self.currency_id)
            results[product.id] = (price, suitable_rule.id)

        return results

    # Split methods to ease (community) overrides
    def _get_applicable_rules(self, products, date, **kwargs):
        self.ensure_one()
        # Do not filter out archived pricelist items, since it means current pricelist is also archived
        # We do not want the computation of prices for archived pricelist to always fallback on the Sales price
        # because no rule was found (thanks to the automatic orm filtering on active field)
        return self.env['product.pricelist.item'].with_context(active_test=False).search(
            self._get_applicable_rules_domain(products=products, date=date, **kwargs)
        )

    def _get_applicable_rules_domain(self, products, date, **kwargs):
        if products._name == 'product.template':
            templates_domain = ('product_tmpl_id', 'in', products.ids)
            products_domain = ('product_id.product_tmpl_id', 'in', products.ids)
        else:
            templates_domain = ('product_tmpl_id', 'in', products.product_tmpl_id.ids)
            products_domain = ('product_id', 'in', products.ids)

        return [
            ('pricelist_id', '=', self.id),
            '|', ('categ_id', '=', False), ('categ_id', 'parent_of', products.categ_id.ids),
            '|', ('product_tmpl_id', '=', False), templates_domain,
            '|', ('product_id', '=', False), products_domain,
            '|', ('date_start', '=', False), ('date_start', '<=', date),
            '|', ('date_end', '=', False), ('date_end', '>=', date),
        ]

    # Multi pricelists price|rule computation
    def _price_get(self, product, qty, **kwargs):
        """ Multi pricelist, mono product - returns price per pricelist """
        return {
            key: price[0]
            for key, price in self._compute_price_rule_multi(product, qty, **kwargs)[product.id].items()}

    def _compute_price_rule_multi(self, products, qty, uom=None, date=False, **kwargs):
        """ Low-level method - Multi pricelist, multi products
        Returns: dict{product_id: dict{pricelist_id: (price, suitable_rule)} }"""
        if not self.ids:
            pricelists = self.search([])
        else:
            pricelists = self
        results = {}
        for pricelist in pricelists:
            subres = pricelist._compute_price_rule(products, qty, uom=uom, date=date, **kwargs)
            for product_id, price in subres.items():
                results.setdefault(product_id, {})
                results[product_id][pricelist.id] = price
        return results

    # res.partner.property_product_pricelist field computation
    @api.model
    def _get_partner_pricelist_multi(self, partner_ids):
        """ Retrieve the applicable pricelist for given partners in a given company.

        It will return the first found pricelist in this order:
        First, the pricelist of the specific property (res_id set), this one
                is created when saving a pricelist on the partner form view.
        Else, it will return the pricelist of the partner country group
        Else, it will return the generic property (res_id not set), this one
                is created on the company creation.
        Else, it will return the first available pricelist

        :param int company_id: if passed, used for looking up properties,
            instead of current user's company
        :return: a dict {partner_id: pricelist}
        """
        # `partner_ids` might be ID from inactive users. We should use active_test
        # as we will do a search() later (real case for website public user).
        Partner = self.env['res.partner'].with_context(active_test=False)
        company_id = self.env.company.id

        Property = self.env['ir.property'].with_company(company_id)
        Pricelist = self.env['product.pricelist']
        pl_domain = self._get_partner_pricelist_multi_search_domain_hook(company_id)

        # if no specific property, try to find a fitting pricelist
        result = Property._get_multi('property_product_pricelist', Partner._name, partner_ids)

        remaining_partner_ids = [pid for pid, val in result.items() if not val or
                                 not val._get_partner_pricelist_multi_filter_hook()]
        if remaining_partner_ids:
            # get fallback pricelist when no pricelist for a given country
            pl_fallback = (
                Pricelist.search(pl_domain + [('country_group_ids', '=', False)], limit=1) or
                Property._get('property_product_pricelist', 'res.partner') or
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

    def _get_partner_pricelist_multi_search_domain_hook(self, company_id):
        return [
            ('active', '=', True),
            ('company_id', 'in', [company_id, False]),
        ]

    def _get_partner_pricelist_multi_filter_hook(self):
        return self.filtered('active')

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Pricelists'),
            'template': '/product/static/xls/product_pricelist.xls'
        }]

    @api.ondelete(at_uninstall=False)
    def _unlink_except_used_as_rule_base(self):
        linked_items = self.env['product.pricelist.item'].sudo().with_context(active_test=False).search([
            ('base', '=', 'pricelist'),
            ('base_pricelist_id', 'in', self.ids),
            ('pricelist_id', 'not in', self.ids),
        ])
        if linked_items:
            raise UserError(_(
                'You cannot delete those pricelist(s):\n(%s)\n, they are used in other pricelist(s):\n%s',
                '\n'.join(linked_items.base_pricelist_id.mapped('display_name')),
                '\n'.join(linked_items.pricelist_id.mapped('display_name'))
            ))

```

## File: models\product_pricelist_item.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError
from odoo.tools import format_datetime, formatLang


class PricelistItem(models.Model):
    _name = "product.pricelist.item"
    _description = "Pricelist Rule"
    _order = "applied_on, min_quantity desc, categ_id desc, id desc"
    _check_company_auto = True

    def _default_pricelist_id(self):
        return self.env['product.pricelist'].search([
            '|', ('company_id', '=', False),
            ('company_id', '=', self.env.company.id)], limit=1)

    pricelist_id = fields.Many2one(
        comodel_name='product.pricelist',
        string="Pricelist",
        index=True, ondelete='cascade',
        required=True,
        default=_default_pricelist_id)

    active = fields.Boolean(related='pricelist_id.active', store=True)
    company_id = fields.Many2one(related='pricelist_id.company_id', store=True)
    currency_id = fields.Many2one(related='pricelist_id.currency_id', store=True)

    date_start = fields.Datetime(
        string="Start Date",
        help="Starting datetime for the pricelist item validation\n"
            "The displayed value depends on the timezone set in your preferences.")
    date_end = fields.Datetime(
        string="End Date",
        help="Ending datetime for the pricelist item validation\n"
            "The displayed value depends on the timezone set in your preferences.")

    min_quantity = fields.Float(
        string="Min. Quantity",
        default=0,
        digits='Product Unit of Measure',
        help="For the rule to apply, bought/sold quantity must be greater "
             "than or equal to the minimum quantity specified in this field.\n"
             "Expressed in the default unit of measure of the product.")

    applied_on = fields.Selection(
        selection=[
            ('3_global', "All Products"),
            ('2_product_category', "Product Category"),
            ('1_product', "Product"),
            ('0_product_variant', "Product Variant"),
        ],
        string="Apply On",
        default='3_global',
        required=True,
        help="Pricelist Item applicable on selected option")

    categ_id = fields.Many2one(
        comodel_name='product.category',
        string="Product Category",
        ondelete='cascade',
        help="Specify a product category if this rule only applies to products belonging to this category or its children categories. Keep empty otherwise.")
    product_tmpl_id = fields.Many2one(
        comodel_name='product.template',
        string="Product",
        ondelete='cascade', check_company=True,
        help="Specify a template if this rule only applies to one product template. Keep empty otherwise.")
    product_id = fields.Many2one(
        comodel_name='product.product',
        string="Product Variant",
        ondelete='cascade', check_company=True,
        help="Specify a product if this rule only applies to one product. Keep empty otherwise.")

    base = fields.Selection(
        selection=[
            ('list_price', 'Sales Price'),
            ('standard_price', 'Cost'),
            ('pricelist', 'Other Pricelist'),
        ],
        string="Based on",
        default='list_price',
        required=True,
        help="Base price for computation.\n"
             "Sales Price: The base price will be the Sales Price.\n"
             "Cost Price : The base price will be the cost price.\n"
             "Other Pricelist : Computation of the base price based on another Pricelist.")
    base_pricelist_id = fields.Many2one('product.pricelist', 'Other Pricelist', check_company=True)

    compute_price = fields.Selection(
        selection=[
            ('fixed', "Fixed Price"),
            ('percentage', "Discount"),
            ('formula', "Formula"),
        ],
        index=True, default='fixed', required=True)

    fixed_price = fields.Float(string="Fixed Price", digits='Product Price')
    percent_price = fields.Float(
        string="Percentage Price",
        help="You can apply a mark-up by setting a negative discount.")

    price_discount = fields.Float(
        string="Price Discount",
        default=0,
        digits=(16, 2),
        help="You can apply a mark-up by setting a negative discount.")
    price_round = fields.Float(
        string="Price Rounding",
        digits='Product Price',
        help="Sets the price so that it is a multiple of this value.\n"
             "Rounding is applied after the discount and before the surcharge.\n"
             "To have prices that end in 9.99, set rounding 10, surcharge -0.01")
    price_surcharge = fields.Float(
        string="Price Surcharge",
        digits='Product Price',
        help="Specify the fixed amount to add or subtract (if negative) to the amount calculated with the discount.")

    price_min_margin = fields.Float(
        string="Min. Price Margin",
        digits='Product Price',
        help="Specify the minimum amount of margin over the base price.")
    price_max_margin = fields.Float(
        string="Max. Price Margin",
        digits='Product Price',
        help="Specify the maximum amount of margin over the base price.")

    # functional fields used for usability purposes
    name = fields.Char(
        string="Name",
        compute='_compute_name_and_price',
        help="Explicit rule name for this pricelist line.")
    price = fields.Char(
        string="Price",
        compute='_compute_name_and_price',
        help="Explicit rule name for this pricelist line.")
    rule_tip = fields.Char(compute='_compute_rule_tip')

    #=== COMPUTE METHODS ===#

    @api.depends('applied_on', 'categ_id', 'product_tmpl_id', 'product_id', 'compute_price', 'fixed_price', \
        'pricelist_id', 'percent_price', 'price_discount', 'price_surcharge')
    def _compute_name_and_price(self):
        for item in self:
            if item.categ_id and item.applied_on == '2_product_category':
                item.name = _("Category: %s") % (item.categ_id.display_name)
            elif item.product_tmpl_id and item.applied_on == '1_product':
                item.name = _("Product: %s") % (item.product_tmpl_id.display_name)
            elif item.product_id and item.applied_on == '0_product_variant':
                item.name = _("Variant: %s") % (item.product_id.display_name)
            else:
                item.name = _("All Products")

            if item.compute_price == 'fixed':
                item.price = formatLang(
                    item.env, item.fixed_price, monetary=True, dp="Product Price", currency_obj=item.currency_id)
            elif item.compute_price == 'percentage':
                item.price = _("%s %% discount", item.percent_price)
            else:
                item.price = _("%(percentage)s %% discount and %(price)s surcharge", percentage=item.price_discount, price=item.price_surcharge)

    @api.depends_context('lang')
    @api.depends('compute_price', 'price_discount', 'price_surcharge', 'base', 'price_round')
    def _compute_rule_tip(self):
        base_selection_vals = {elem[0]: elem[1] for elem in self._fields['base']._description_selection(self.env)}
        self.rule_tip = False
        for item in self:
            if item.compute_price != 'formula':
                continue
            base_amount = 100
            discount_factor = (100 - item.price_discount) / 100
            discounted_price = base_amount * discount_factor
            if item.price_round:
                discounted_price = tools.float_round(discounted_price, precision_rounding=item.price_round)
            surcharge = tools.format_amount(item.env, item.price_surcharge, item.currency_id)
            item.rule_tip = _(
                "%(base)s with a %(discount)s %% discount and %(surcharge)s extra fee\n"
                "Example: %(amount)s * %(discount_charge)s + %(price_surcharge)s → %(total_amount)s",
                base=base_selection_vals[item.base],
                discount=item.price_discount,
                surcharge=surcharge,
                amount=tools.format_amount(item.env, 100, item.currency_id),
                discount_charge=discount_factor,
                price_surcharge=surcharge,
                total_amount=tools.format_amount(
                    item.env, discounted_price + item.price_surcharge, item.currency_id),
            )

    #=== CONSTRAINT METHODS ===#

    @api.constrains('base_pricelist_id', 'pricelist_id', 'base')
    def _check_recursion(self):
        if any(item.base == 'pricelist' and item.pricelist_id and item.pricelist_id == item.base_pricelist_id for item in self):
            raise ValidationError(_('You cannot assign the Main Pricelist as Other Pricelist in PriceList Item'))

    @api.constrains('date_start', 'date_end')
    def _check_date_range(self):
        for item in self:
            if item.date_start and item.date_end and item.date_start >= item.date_end:
                raise ValidationError(_('%s : end date (%s) should be greater than start date (%s)', item.display_name, format_datetime(self.env, item.date_end), format_datetime(self.env, item.date_start)))
        return True

    @api.constrains('price_min_margin', 'price_max_margin')
    def _check_margin(self):
        if any(item.price_min_margin > item.price_max_margin for item in self):
            raise ValidationError(_('The minimum margin should be lower than the maximum margin.'))

    @api.constrains('product_id', 'product_tmpl_id', 'categ_id')
    def _check_product_consistency(self):
        for item in self:
            if item.applied_on == "2_product_category" and not item.categ_id:
                raise ValidationError(_("Please specify the category for which this rule should be applied"))
            elif item.applied_on == "1_product" and not item.product_tmpl_id:
                raise ValidationError(_("Please specify the product for which this rule should be applied"))
            elif item.applied_on == "0_product_variant" and not item.product_id:
                raise ValidationError(_("Please specify the product variant for which this rule should be applied"))

    #=== ONCHANGE METHODS ===#

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
    def _onchange_rule_content(self):
        if not self.user_has_groups('product.group_sale_pricelist') and not self.env.context.get('default_applied_on', False):
            # If advanced pricelists are disabled (applied_on field is not visible)
            # AND we aren't coming from a specific product template/variant.
            variants_rules = self.filtered('product_id')
            template_rules = (self-variants_rules).filtered('product_tmpl_id')
            variants_rules.update({'applied_on': '0_product_variant'})
            template_rules.update({'applied_on': '1_product'})
            (self-variants_rules-template_rules).update({'applied_on': '3_global'})

    @api.onchange('price_round')
    def _onchange_price_round(self):
        if any(item.price_round and item.price_round < 0.0 for item in self):
            raise ValidationError(_("The rounding method must be strictly positive."))

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if not values.get('applied_on'):
                values['applied_on'] = (
                    '0_product_variant' if values.get('product_id') else
                    '1_product' if values.get('product_tmpl_id') else
                    '2_product_category' if values.get('categ_id') else
                    '3_global'
                )

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
        return super().create(vals_list)

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
        return super().write(values)

    def toggle_active(self):
        raise ValidationError(_("You cannot disable a pricelist rule, please delete it or archive its pricelist instead."))

    #=== BUSINESS METHODS ===#

    def _is_applicable_for(self, product, qty_in_product_uom):
        """Check whether the current rule is valid for the given product & qty.

        Note: self.ensure_one()

        :param product: product record (product.product/product.template)
        :param float qty_in_product_uom: quantity, expressed in product UoM
        :returns: Whether rules is valid or not
        :rtype: bool
        """
        self.ensure_one()
        product.ensure_one()
        res = True

        is_product_template = product._name == 'product.template'
        if self.min_quantity and qty_in_product_uom < self.min_quantity:
            res = False

        elif self.applied_on == "2_product_category":
            if (
                product.categ_id != self.categ_id
                and not product.categ_id.parent_path.startswith(self.categ_id.parent_path)
            ):
                res = False
        else:
            # Applied on a specific product template/variant
            if is_product_template:
                if self.applied_on == "1_product" and product.id != self.product_tmpl_id.id:
                    res = False
                elif self.applied_on == "0_product_variant" and not (
                    product.product_variant_count == 1
                    and product.product_variant_id.id == self.product_id.id
                ):
                    # product self acceptable on template if has only one variant
                    res = False
            else:
                if self.applied_on == "1_product" and product.product_tmpl_id.id != self.product_tmpl_id.id:
                    res = False
                elif self.applied_on == "0_product_variant" and product.id != self.product_id.id:
                    res = False

        return res

    def _compute_price(self, product, quantity, uom, date, currency=None):
        """Compute the unit price of a product in the context of a pricelist application.

        :param product: recordset of product (product.product/product.template)
        :param float qty: quantity of products requested (in given uom)
        :param uom: unit of measure (uom.uom record)
        :param datetime date: date to use for price computation and currency conversions
        :param currency: pricelist currency (for the specific case where self is empty)

        :returns: price according to pricelist rule, expressed in pricelist currency
        :rtype: float
        """
        product.ensure_one()
        uom.ensure_one()

        currency = currency or self.currency_id
        currency.ensure_one()

        # Pricelist specific values are specified according to product UoM
        # and must be multiplied according to the factor between uoms
        product_uom = product.uom_id
        if product_uom != uom:
            convert = lambda p: product_uom._compute_price(p, uom)
        else:
            convert = lambda p: p

        if self.compute_price == 'fixed':
            price = convert(self.fixed_price)
        elif self.compute_price == 'percentage':
            base_price = self._compute_base_price(product, quantity, uom, date, currency)
            price = (base_price - (base_price * (self.percent_price / 100))) or 0.0
        elif self.compute_price == 'formula':
            base_price = self._compute_base_price(product, quantity, uom, date, currency)
            # complete formula
            price_limit = base_price
            price = (base_price - (base_price * (self.price_discount / 100))) or 0.0
            if self.price_round:
                price = tools.float_round(price, precision_rounding=self.price_round)

            if self.price_surcharge:
                price += convert(self.price_surcharge)

            if self.price_min_margin:
                price = max(price, price_limit + convert(self.price_min_margin))

            if self.price_max_margin:
                price = min(price, price_limit + convert(self.price_max_margin))
        else:  # empty self, or extended pricelist price computation logic
            price = self._compute_base_price(product, quantity, uom, date, currency)

        return price

    def _compute_base_price(self, product, quantity, uom, date, target_currency):
        """ Compute the base price for a given rule

        :param product: recordset of product (product.product/product.template)
        :param float qty: quantity of products requested (in given uom)
        :param uom: unit of measure (uom.uom record)
        :param datetime date: date to use for price computation and currency conversions
        :param target_currency: pricelist currency

        :returns: base price, expressed in provided pricelist currency
        :rtype: float
        """
        target_currency.ensure_one()

        rule_base = self.base or 'list_price'
        if rule_base == 'pricelist' and self.base_pricelist_id:
            price = self.base_pricelist_id._get_product_price(product, quantity, uom, date)
            src_currency = self.base_pricelist_id.currency_id
        elif rule_base == "standard_price":
            src_currency = product.cost_currency_id
            price = product.price_compute(rule_base, uom=uom, date=date)[product.id]
        else: # list_price
            src_currency = product.currency_id
            price = product.price_compute(rule_base, uom=uom, date=date)[product.id]

        if src_currency != target_currency:
            price = src_currency._convert(price, target_currency, self.env.company, date, round=False)

        return price

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from collections import defaultdict

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError
from odoo.osv import expression
from odoo.tools import float_compare


class ProductProduct(models.Model):
    _name = "product.product"
    _description = "Product Variant"
    _inherits = {'product.template': 'product_tmpl_id'}
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'priority desc, default_code, name, id'

    # price_extra: catalog extra value only, sum of variant extra attributes
    price_extra = fields.Float(
        'Variant Price Extra', compute='_compute_product_price_extra',
        digits='Product Price',
        help="This is the sum of the extra price of all attributes")
    # lst_price: catalog value + extra, context dependent (uom)
    lst_price = fields.Float(
        'Sales Price', compute='_compute_product_lst_price',
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
        'Barcode', copy=False, index='btree_not_null',
        help="International Article Number used for product identification.")
    product_template_attribute_value_ids = fields.Many2many('product.template.attribute.value', relation='product_variant_combination', string="Attribute Values", ondelete='restrict')
    product_template_variant_value_ids = fields.Many2many('product.template.attribute.value', relation='product_variant_combination',
                                                          domain=[('attribute_line_id.value_count', '>', 1)], string="Variant Values", ondelete='restrict')
    combination_indices = fields.Char(compute='_compute_combination_indices', store=True, index=True)
    is_product_variant = fields.Boolean(compute='_compute_is_product_variant')

    standard_price = fields.Float(
        'Cost', company_dependent=True,
        digits='Product Price',
        groups="base.group_user",
        help="""In Standard Price & AVCO: value of the product (automatically computed in AVCO).
        In FIFO: value of the next unit that will leave the stock (automatically computed).
        Used to value the product when the purchase cost is not known (e.g. inventory adjustment).
        Used to compute margins on sale orders.""")
    volume = fields.Float('Volume', digits='Volume')
    weight = fields.Float('Weight', digits='Stock Weight')

    pricelist_item_count = fields.Integer("Number of price rules", compute="_compute_variant_item_count")

    packaging_ids = fields.One2many(
        'product.packaging', 'product_id', 'Product Packages',
        help="Gives the different ways to package the same product.")

    additional_product_tag_ids = fields.Many2many('product.tag', 'product_tag_product_product_rel')
    all_product_tag_ids = fields.Many2many('product.tag', compute='_compute_all_product_tag_ids', search='_search_all_product_tag_ids')

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

    def _set_template_field(self, template_field, variant_field):
        for record in self:
            if (
                # We are trying to remove a field from the variant even though it is already
                # not set on the variant, remove it from the template instead.
                (not record[template_field] and not record[variant_field])
                # We are trying to add a field to the variant, but the template field is
                # not set, write on the template instead.
                or (record[template_field] and not record.product_tmpl_id[template_field])
                # There is only one variant, always write on the template.
                or self.search_count([
                    ('product_tmpl_id', '=', record.product_tmpl_id.id),
                    ('active', '=', True),
                ]) <= 1
            ):
                record[variant_field] = False
                record.product_tmpl_id[template_field] = record[template_field]
            else:
                record[variant_field] = record[template_field]

    @api.depends("create_date", "write_date", "product_tmpl_id.create_date", "product_tmpl_id.write_date")
    def _compute_concurrency_field(self):
        # Intentionally not calling super() to involve all fields explicitly
        for record in self:
            record[self.CONCURRENCY_CHECK_FIELD] = max(filter(None, (
                record.product_tmpl_id.write_date or record.product_tmpl_id.create_date,
                record.write_date or record.create_date or fields.Datetime.now(),
            )))

    def _compute_image_1920(self):
        """Get the image from the template if no image is set on the variant."""
        for record in self:
            record.image_1920 = record.image_variant_1920 or record.product_tmpl_id.image_1920

    def _set_image_1920(self):
        return self._set_template_field('image_1920', 'image_variant_1920')

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

    def _get_placeholder_filename(self, field):
        image_fields = ['image_%s' % size for size in [1920, 1024, 512, 256, 128]]
        if field in image_fields:
            return 'product/static/img/placeholder.png'
        return super()._get_placeholder_filename(field)

    def init(self):
        """Ensure there is at most one active variant for each combination.

        There could be no variant for a combination if using dynamic attributes.
        """
        self.env.cr.execute("CREATE UNIQUE INDEX IF NOT EXISTS product_product_combination_unique ON %s (product_tmpl_id, combination_indices) WHERE active is true"
            % self._table)

    @api.constrains('barcode')
    def _check_barcode_uniqueness(self):
        """ With GS1 nomenclature, products and packagings use the same pattern. Therefore, we need
        to ensure the uniqueness between products' barcodes and packagings' ones"""
        all_barcode = [b for b in self.mapped('barcode') if b]
        domain = [('barcode', 'in', all_barcode)]
        matched_products = self.sudo().search(domain, order='id')
        if len(matched_products) > len(all_barcode):  # It means that you find more than `self` -> there are duplicates
            products_by_barcode = defaultdict(list)
            for product in matched_products:
                products_by_barcode[product.barcode].append(product)

            duplicates_as_str = "\n".join(
                _("- Barcode \"%s\" already assigned to product(s): %s", barcode, ", ".join(p.display_name for p in products))
                for barcode, products in products_by_barcode.items() if len(products) > 1
            )
            raise ValidationError(_("Barcode(s) already assigned:\n\n%s", duplicates_as_str))

        if self.env['product.packaging'].search(domain, order="id", limit=1):
            raise ValidationError(_("A packaging already uses the barcode"))

    def _get_invoice_policy(self):
        return False

    @api.depends('product_template_attribute_value_ids')
    def _compute_combination_indices(self):
        for product in self:
            product.combination_indices = product.product_template_attribute_value_ids._ids2str()

    def _compute_is_product_variant(self):
        self.is_product_variant = True

    @api.onchange('lst_price')
    def _set_product_lst_price(self):
        for product in self:
            if self._context.get('uom'):
                value = self.env['uom.uom'].browse(self._context['uom'])._compute_price(product.lst_price, product.uom_id)
            else:
                value = product.lst_price
            value -= product.price_extra
            product.write({'list_price': value})

    @api.depends("product_template_attribute_value_ids.price_extra")
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
        read_access = self.env['ir.model.access'].check('product.supplierinfo', 'read', False)
        for product in self:
            product.code = product.default_code
            if read_access:
                for supplier_info in product.seller_ids:
                    if supplier_info.partner_id.id == product._context.get('partner_id'):
                        if supplier_info.product_id and supplier_info.product_id != product:
                            # Supplier info specific for another variant.
                            continue
                        product.code = supplier_info.product_code or product.default_code
                        if product == supplier_info.product_id:
                            # Supplier info specific for this variant.
                            break

    @api.depends_context('partner_id')
    def _compute_partner_ref(self):
        for product in self:
            for supplier_info in product.seller_ids:
                if supplier_info.partner_id.id == product._context.get('partner_id'):
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

    @api.depends('product_tag_ids', 'additional_product_tag_ids')
    def _compute_all_product_tag_ids(self):
        for product in self:
            product.all_product_tag_ids = product.product_tag_ids | product.additional_product_tag_ids

    def _search_all_product_tag_ids(self, operator, operand):
        if operator in expression.NEGATIVE_TERM_OPERATORS:
            return [('product_tag_ids', operator, operand), ('additional_product_tag_ids', operator, operand)]
        return ['|', ('product_tag_ids', operator, operand), ('additional_product_tag_ids', operator, operand)]

    @api.onchange('uom_id')
    def _onchange_uom_id(self):
        if self.uom_id:
            self.uom_po_id = self.uom_id.id

    @api.onchange('uom_po_id')
    def _onchange_uom(self):
        if self.uom_id and self.uom_po_id and self.uom_id.category_id != self.uom_po_id.category_id:
            self.uom_po_id = self.uom_id

    @api.onchange('default_code')
    def _onchange_default_code(self):
        if not self.default_code:
            return

        domain = [('default_code', '=', self.default_code)]
        if self.id.origin:
            domain.append(('id', '!=', self.id.origin))

        if self.env['product.product'].search(domain, limit=1):
            return {'warning': {
                'title': _("Note:"),
                'message': _("The Internal Reference '%s' already exists.", self.default_code),
            }}

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            self.product_tmpl_id._sanitize_vals(vals)
        products = super(ProductProduct, self.with_context(create_product_product=True)).create(vals_list)
        # `_get_variant_id_for_combination` depends on existing variants
        self.clear_caches()
        return products

    def write(self, values):
        self.product_tmpl_id._sanitize_vals(values)
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
        self.packaging_ids.unlink()
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
            args = args.copy()
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
            supplier_info = self.env['product.supplierinfo'].sudo().search([
                ('product_tmpl_id', 'in', product_template_ids),
                ('partner_id', 'in', partner_ids),
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
                product_ids = list(self._search([('default_code', '=', name)] + args, limit=limit, access_rights_uid=name_get_uid))
                if not product_ids:
                    product_ids = list(self._search([('barcode', '=', name)] + args, limit=limit, access_rights_uid=name_get_uid))
            if not product_ids and operator not in expression.NEGATIVE_TERM_OPERATORS:
                # Do not merge the 2 next lines into one single search, SQL search performance would be abysmal
                # on a database with thousands of matching products, due to the huge merge+unique needed for the
                # OR operator (and given the fact that the 'name' lookup results come from the ir.translation table
                # Performing a quick memory merge of ids in Python will give much better performance
                product_ids = list(self._search(args + [('default_code', operator, name)], limit=limit))
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
                product_ids = list(self._search(domain, limit=limit, access_rights_uid=name_get_uid))
            if not product_ids and operator in positive_operators:
                ptrn = re.compile(r'(\[(.*?)\])')
                res = ptrn.search(name)
                if res:
                    product_ids = list(self._search([('default_code', '=', res.group(2))] + args, limit=limit, access_rights_uid=name_get_uid))
            # still no results, partner in context: search on supplier info as last hope to find something
            if not product_ids and self._context.get('partner_id'):
                suppliers_ids = self.env['product.supplierinfo']._search([
                    ('partner_id', '=', self._context.get('partner_id')),
                    '|',
                    ('product_code', operator, name),
                    ('product_name', operator, name)], access_rights_uid=name_get_uid)
                if suppliers_ids:
                    product_ids = self._search([('product_tmpl_id.seller_ids', 'in', suppliers_ids)], limit=limit, access_rights_uid=name_get_uid)
        else:
            product_ids = self._search(args, limit=limit, access_rights_uid=name_get_uid)
        return product_ids

    @api.model
    def view_header_get(self, view_id, view_type):
        if self._context.get('categ_id'):
            return _(
                'Products: %(category)s',
                category=self.env['product.category'].browse(self.env.context['categ_id']).name,
            )
        return super().view_header_get(view_id, view_type)

    def action_open_label_layout(self):
        action = self.env['ir.actions.act_window']._for_xml_id('product.action_open_label_layout')
        action['context'] = {'default_product_ids': self.ids}
        return action

    def open_pricelist_rules(self):
        self.ensure_one()
        domain = ['|',
            '&', ('product_tmpl_id', '=', self.product_tmpl_id.id), ('applied_on', '=', '1_product'),
            '&', ('product_id', '=', self.id), ('applied_on', '=', '0_product_variant')]
        return {
            'name': _('Price Rules'),
            'view_mode': 'tree,form',
            'views': [(self.env.ref('product.product_pricelist_item_tree_view_from_product').id, 'tree')],
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

    def _prepare_sellers(self, params=False):
        return self.seller_ids.filtered(lambda s: s.partner_id.active).sorted(lambda s: (s.sequence, -s.min_qty, s.price, s.id))

    def _get_filtered_sellers(self, partner_id=False, quantity=0.0, date=None, uom_id=False, params=False):
        self.ensure_one()
        if date is None:
            date = fields.Date.context_today(self)
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')

        sellers_filtered = self._prepare_sellers(params)
        sellers_filtered = sellers_filtered.filtered(lambda s: not s.company_id or s.company_id.id == self.env.company.id)
        sellers = self.env['product.supplierinfo']
        for seller in sellers_filtered:
            # Set quantity in UoM of seller
            quantity_uom_seller = quantity
            if quantity_uom_seller and uom_id and uom_id != seller.product_uom:
                quantity_uom_seller = uom_id._compute_quantity(quantity_uom_seller, seller.product_uom)

            if seller.date_start and seller.date_start > date:
                continue
            if seller.date_end and seller.date_end < date:
                continue
            if partner_id and seller.partner_id not in [partner_id, partner_id.parent_id]:
                continue
            if quantity is not None and float_compare(quantity_uom_seller, seller.min_qty, precision_digits=precision) == -1:
                continue
            if seller.product_id and seller.product_id != self:
                continue
            sellers |= seller
        return sellers

    def _select_seller(self, partner_id=False, quantity=0.0, date=None, uom_id=False, params=False):
        sellers = self._get_filtered_sellers(partner_id=partner_id, quantity=quantity, date=date, uom_id=uom_id, params=params)
        res = self.env['product.supplierinfo']
        for seller in sellers:
            if not res or res.partner_id == seller.partner_id:
                res |= seller
        return res and res.sorted('price')[:1]

    def price_compute(self, price_type, uom=None, currency=None, company=None, date=False):
        company = company or self.env.company
        date = date or fields.Date.context_today(self)

        self = self.with_company(company)
        if price_type == 'standard_price':
            # standard_price field can only be seen by users in base.group_user
            # Thus, in order to compute the sale price from the cost for users not in this group
            # We fetch the standard price as the superuser
            self = self.sudo()

        prices = dict.fromkeys(self.ids, 0.0)
        for product in self:
            price = product[price_type] or 0.0
            price_currency = product.currency_id
            if price_type == 'standard_price':
                price_currency = product.cost_currency_id

            if price_type == 'list_price':
                price += product.price_extra
                # we need to add the price from the attributes that do not generate variants
                # (see field product.attribute create_variant)
                if self._context.get('no_variant_attributes_price_extra'):
                    # we have a list of price_extra that comes from the attribute values, we need to sum all that
                    price += sum(self._context.get('no_variant_attributes_price_extra'))

            if uom:
                price = product.uom_id._compute_price(price, uom)

            # Convert from current user company currency to asked one
            # This is right cause a field cannot be in more than one currency
            if currency:
                price = price_currency._convert(price, currency, company, date)

            prices[product.id] = price

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

    def get_contextual_price(self):
        return self._get_contextual_price()

    def _get_contextual_price(self):
        self.ensure_one()
        return self.product_tmpl_id._get_contextual_price(self)

```

## File: models\product_supplierinfo.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class SupplierInfo(models.Model):
    _name = "product.supplierinfo"
    _description = "Supplier Pricelist"
    _order = 'sequence, min_qty DESC, price, id'
    _rec_name = 'partner_id'

    def _default_product_id(self):
        product_id = self.env.get('default_product_id')
        if not product_id:
            model, active_id = [self.env.context.get(k) for k in ['model', 'active_id']]
            if model == 'product.product' and active_id:
                product_id = self.env[model].browse(active_id).exists()
        return product_id

    def _domain_product_id(self):
        domain = "product_tmpl_id and [('product_tmpl_id', '=', product_tmpl_id)] or []"
        if self.env.context.get('base_model_name') == 'product.template':
            domain = "[('product_tmpl_id', '=', parent.id)]"
        elif self.env.context.get('base_model_name') == 'product.product':
            domain = "[('product_tmpl_id', '=', parent.product_tmpl_id)]"
        return domain

    partner_id = fields.Many2one(
        'res.partner', 'Vendor',
        ondelete='cascade', required=True,
        check_company=True)
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
        related='product_tmpl_id.uom_po_id')
    min_qty = fields.Float(
        'Quantity', default=0.0, required=True, digits="Product Unit of Measure",
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
        domain=_domain_product_id, default=_default_product_id,
        help="If not set, the vendor price will apply to all variants of this product.")
    product_tmpl_id = fields.Many2one(
        'product.template', 'Product Template', check_company=True,
        index=True, ondelete='cascade')
    product_variant_count = fields.Integer('Variant Count', related='product_tmpl_id.product_variant_count')
    delay = fields.Integer(
        'Delivery Lead Time', default=1, required=True,
        help="Lead time in days between the confirmation of the purchase order and the receipt of the products in your warehouse. Used by the scheduler for automatic computation of the purchase order planning.")

    @api.onchange('product_tmpl_id')
    def _onchange_product_tmpl_id(self):
        """Clear product variant if it no longer matches the product template."""
        if self.product_id and self.product_id not in self.product_tmpl_id.product_variant_ids:
            self.product_id = False

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Vendor Pricelists'),
            'template': '/product/static/xls/product_supplierinfo.xls'
        }]

    def _sanitize_vals(self, vals):
        """Sanitize vals to sync product variant & template on read/write."""
        # add product's product_tmpl_id if none present in vals
        if  vals.get('product_id') and not vals.get('product_tmpl_id'):
            product = self.env['product.product'].browse(vals['product_id'])
            vals['product_tmpl_id'] = product.product_tmpl_id.id

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            self._sanitize_vals(vals)
        return super().create(vals_list)

    def write(self, vals):
        self._sanitize_vals(vals)
        return super().write(vals)

```

## File: models\product_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from random import randint

from odoo import api, fields, models
from odoo.osv import expression

class ProductTag(models.Model):
    _name = 'product.tag'
    _description = 'Product Tag'

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char('Tag Name', required=True, translate=True)
    color = fields.Integer('Color', default=_get_default_color)

    product_template_ids = fields.Many2many('product.template', 'product_tag_product_template_rel')
    product_product_ids = fields.Many2many('product.product', 'product_tag_product_product_rel')
    product_ids = fields.Many2many(
        'product.product', string='All Product Variants using this Tag',
        compute='_compute_product_ids', search='_search_product_ids'
    )

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists !"),
    ]

    @api.depends('product_template_ids', 'product_product_ids')
    def _compute_product_ids(self):
        for tag in self:
            tag.product_ids = tag.product_template_ids.product_variant_ids | tag.product_product_ids

    def _search_product_ids(self, operator, operand):
        if operator in expression.NEGATIVE_TERM_OPERATORS:
            return [('product_template_ids.product_variant_ids', operator, operand), ('product_product_ids', operator, operand)]
        return ['|', ('product_template_ids.product_variant_ids', operator, operand), ('product_product_ids', operator, operand)]

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
from odoo.models import PREFETCH_MAX
from odoo.osv import expression

_logger = logging.getLogger(__name__)


class ProductTemplate(models.Model):
    _name = "product.template"
    _inherit = ['mail.thread', 'mail.activity.mixin', 'image.mixin']
    _description = "Product"
    _order = "priority desc, name"

    @tools.ormcache()
    def _get_default_category_id(self):
        # Deletion forbidden (at least through unlink)
        return self.env.ref('product.product_category_all')

    @tools.ormcache()
    def _get_default_uom_id(self):
        # Deletion forbidden (at least through unlink)
        return self.env.ref('uom.product_uom_unit')

    def _get_default_uom_po_id(self):
        return self.default_get(['uom_id']).get('uom_id') or self._get_default_uom_id()

    def _read_group_categ_id(self, categories, domain, order):
        category_ids = self.env.context.get('default_categ_id')
        if not category_ids and self.env.context.get('group_expand'):
            category_ids = categories._search([], order=order, access_rights_uid=SUPERUSER_ID)
        return categories.browse(category_ids)

    name = fields.Char('Name', index='trigram', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=1, help='Gives the sequence order when displaying a product list')
    description = fields.Html(
        'Description', translate=True)
    description_purchase = fields.Text(
        'Purchase Description', translate=True)
    description_sale = fields.Text(
        'Sales Description', translate=True,
        help="A description of the Product that you want to communicate to your customers. "
             "This description will be copied to every Sales Order, Delivery Order and Customer Invoice/Credit Note")
    detailed_type = fields.Selection([
        ('consu', 'Consumable'),
        ('service', 'Service')], string='Product Type', default='consu', required=True,
        help='A storable product is a product for which you manage stock. The Inventory app has to be installed.\n'
             'A consumable product is a product for which stock is not managed.\n'
             'A service is a non-material product you provide.')
    type = fields.Selection(
        [('consu', 'Consumable'),
         ('service', 'Service')],
        compute='_compute_type', store=True, readonly=False, precompute=True)
    categ_id = fields.Many2one(
        'product.category', 'Product Category',
        change_default=True, default=_get_default_category_id, group_expand='_read_group_categ_id',
        required=True)

    currency_id = fields.Many2one(
        'res.currency', 'Currency', compute='_compute_currency_id')
    cost_currency_id = fields.Many2one(
        'res.currency', 'Cost Currency', compute='_compute_cost_currency_id')

    # list_price: catalog price, user defined
    list_price = fields.Float(
        'Sales Price', default=1.0,
        digits='Product Price',
        help="Price at which the product is sold to customers.",
    )
    standard_price = fields.Float(
        'Cost', compute='_compute_standard_price',
        inverse='_set_standard_price', search='_search_standard_price',
        digits='Product Price', groups="base.group_user",
        help="""In Standard Price & AVCO: value of the product (automatically computed in AVCO).
        In FIFO: value of the next unit that will leave the stock (automatically computed).
        Used to value the product when the purchase cost is not known (e.g. inventory adjustment).
        Used to compute margins on sale orders.""")

    volume = fields.Float(
        'Volume', compute='_compute_volume', inverse='_set_volume', digits='Volume', store=True)
    volume_uom_name = fields.Char(string='Volume unit of measure label', compute='_compute_volume_uom_name')
    weight = fields.Float(
        'Weight', compute='_compute_weight', digits='Stock Weight',
        inverse='_set_weight', store=True)
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name')

    sale_ok = fields.Boolean('Can be Sold', default=True)
    purchase_ok = fields.Boolean('Can be Purchased', default=True)
    uom_id = fields.Many2one(
        'uom.uom', 'Unit of Measure',
        default=_get_default_uom_id, required=True,
        help="Default unit of measure used for all stock operations.")
    uom_name = fields.Char(string='Unit of Measure Name', related='uom_id.name', readonly=True)
    uom_po_id = fields.Many2one(
        'uom.uom', 'Purchase UoM',
        default=_get_default_uom_po_id, required=True,
        help="Default unit of measure used for purchase orders. It must be in the same category as the default unit of measure.")
    company_id = fields.Many2one(
        'res.company', 'Company', index=True)
    packaging_ids = fields.One2many(
        'product.packaging', string="Product Packages", compute="_compute_packaging_ids", inverse="_set_packaging_ids",
        help="Gives the different ways to package the same product.")
    seller_ids = fields.One2many('product.supplierinfo', 'product_tmpl_id', 'Vendors', depends_context=('company',))
    variant_seller_ids = fields.One2many('product.supplierinfo', 'product_tmpl_id')

    active = fields.Boolean('Active', default=True, help="If unchecked, it will allow you to hide the product without removing it.")
    color = fields.Integer('Color Index')

    is_product_variant = fields.Boolean(string='Is a product variant', compute='_compute_is_product_variant')
    attribute_line_ids = fields.One2many('product.template.attribute.line', 'product_tmpl_id', 'Product Attributes', copy=True)

    valid_product_template_attribute_line_ids = fields.Many2many('product.template.attribute.line',
        compute="_compute_valid_product_template_attribute_line_ids", string='Valid Product Attribute Lines')

    product_variant_ids = fields.One2many('product.product', 'product_tmpl_id', 'Products', required=True)
    # performance: product_variant_id provides prefetching on the first product variant only
    product_variant_id = fields.Many2one('product.product', 'Product', compute='_compute_product_variant_id')

    product_variant_count = fields.Integer(
        '# Product Variants', compute='_compute_product_variant_count')

    # related to display product product information if is_product_variant
    barcode = fields.Char('Barcode', compute='_compute_barcode', inverse='_set_barcode', search='_search_barcode')
    default_code = fields.Char(
        'Internal Reference', compute='_compute_default_code',
        inverse='_set_default_code', store=True)

    pricelist_item_count = fields.Integer("Number of price rules", compute="_compute_item_count")

    can_image_1024_be_zoomed = fields.Boolean("Can Image 1024 be zoomed", compute='_compute_can_image_1024_be_zoomed', store=True)
    has_configurable_attributes = fields.Boolean("Is a configurable product", compute='_compute_has_configurable_attributes', store=True)

    product_tooltip = fields.Char(compute='_compute_product_tooltip')

    priority = fields.Selection([
        ('0', 'Normal'),
        ('1', 'Favorite'),
    ], default='0', string="Favorite")

    product_tag_ids = fields.Many2many('product.tag', 'product_tag_product_template_rel', string='Product Tags')

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

    @api.depends_context('company')
    def _compute_cost_currency_id(self):
        self.cost_currency_id = self.env.company.currency_id.id

    @api.depends_context('company')
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
        self.is_product_variant = False

    @api.depends('product_variant_ids.barcode')
    def _compute_barcode(self):
        self.barcode = False
        for template in self:
            # TODO master: update product_variant_count depends and use it instead
            variant_count = len(template.product_variant_ids)
            if variant_count == 1:
                template.barcode = template.product_variant_ids.barcode
            elif variant_count == 0:
                archived_variants = template.with_context(active_test=False).product_variant_ids
                if len(archived_variants) == 1:
                    template.barcode = archived_variants.barcode

    def _search_barcode(self, operator, value):
        query = self.with_context(active_test=False)._search([('product_variant_ids.barcode', operator, value)])
        return [('id', 'in', query)]

    def _set_barcode(self):
        variant_count = len(self.product_variant_ids)
        if variant_count == 1:
            self.product_variant_ids.barcode = self.barcode
        elif variant_count == 0:
            archived_variants = self.with_context(active_test=False).product_variant_ids
            if len(archived_variants) == 1:
                archived_variants.barcode = self.barcode

    @api.model
    def _get_weight_uom_id_from_ir_config_parameter(self):
        """ Get the unit of measure to interpret the `weight` field. By default, we considerer
        that weights are expressed in kilograms. Users can configure to express them in pounds
        by adding an ir.config_parameter record with "product.product_weight_in_lbs" as key
        and "1" as value.
        """
        product_weight_in_lbs_param = self.env['ir.config_parameter'].sudo().get_param('product.weight_in_lbs')
        if product_weight_in_lbs_param == '1':
            return self.env.ref('uom.product_uom_lb')
        else:
            return self.env.ref('uom.product_uom_kgm')

    @api.model
    def _get_length_uom_id_from_ir_config_parameter(self):
        """ Get the unit of measure to interpret the `length`, 'width', 'height' field.
        By default, we considerer that length are expressed in millimeters. Users can configure
        to express them in feet by adding an ir.config_parameter record with "product.volume_in_cubic_feet"
        as key and "1" as value.
        """
        product_length_in_feet_param = self.env['ir.config_parameter'].sudo().get_param('product.volume_in_cubic_feet')
        if product_length_in_feet_param == '1':
            return self.env.ref('uom.product_uom_foot')
        else:
            return self.env.ref('uom.product_uom_millimeter')

    @api.model
    def _get_volume_uom_id_from_ir_config_parameter(self):
        """ Get the unit of measure to interpret the `volume` field. By default, we consider
        that volumes are expressed in cubic meters. Users can configure to express them in cubic feet
        by adding an ir.config_parameter record with "product.volume_in_cubic_feet" as key
        and "1" as value.
        """
        product_length_in_feet_param = self.env['ir.config_parameter'].sudo().get_param('product.volume_in_cubic_feet')
        if product_length_in_feet_param == '1':
            return self.env.ref('uom.product_uom_cubic_foot')
        else:
            return self.env.ref('uom.product_uom_cubic_meter')

    @api.model
    def _get_weight_uom_name_from_ir_config_parameter(self):
        return self._get_weight_uom_id_from_ir_config_parameter().display_name

    @api.model
    def _get_length_uom_name_from_ir_config_parameter(self):
        return self._get_length_uom_id_from_ir_config_parameter().display_name

    @api.model
    def _get_volume_uom_name_from_ir_config_parameter(self):
        return self._get_volume_uom_id_from_ir_config_parameter().display_name

    def _compute_weight_uom_name(self):
        self.weight_uom_name = self._get_weight_uom_name_from_ir_config_parameter()

    def _compute_volume_uom_name(self):
        self.volume_uom_name = self._get_volume_uom_name_from_ir_config_parameter()

    def _set_weight(self):
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.product_variant_ids.weight = template.weight

    @api.depends('product_variant_ids.product_tmpl_id')
    def _compute_product_variant_count(self):
        for template in self:
            template.product_variant_count = len(template.product_variant_ids)

    @api.onchange('default_code')
    def _onchange_default_code(self):
        if not self.default_code:
            return

        domain = [('default_code', '=', self.default_code)]
        if self.id.origin:
            domain.append(('id', '!=', self.id.origin))

        if self.env['product.template'].search(domain, limit=1):
            return {'warning': {
                'title': _("Note:"),
                'message': _("The Internal Reference '%s' already exists.", self.default_code),
            }}

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

    @api.depends('type')
    def _compute_product_tooltip(self):
        for record in self:
            if record.type == 'consu':
                record.product_tooltip = _(
                    "Consumables are physical products for which you don't manage the inventory "
                    "level: they are always available."
                )
            else:
                record.product_tooltip = ""

    def _detailed_type_mapping(self):
        return {}

    @api.depends('detailed_type')
    def _compute_type(self):
        type_mapping = self._detailed_type_mapping()
        for record in self:
            record.type = type_mapping.get(record.detailed_type, record.detailed_type)

    @api.constrains('type', 'detailed_type')
    def _constrains_detailed_type(self):
        type_mapping = self._detailed_type_mapping()
        for record in self:
            if record.type != type_mapping.get(record.detailed_type, record.detailed_type):
                raise ValidationError(_("The Type of this product doesn't match the Detailed Type"))

    @api.constrains('uom_id', 'uom_po_id')
    def _check_uom(self):
        if any(template.uom_id and template.uom_po_id and template.uom_id.category_id != template.uom_po_id.category_id for template in self):
            raise ValidationError(_('The default Unit of Measure and the purchase Unit of Measure must be in the same category.'))

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

    def _sanitize_vals(self, vals):
        """Sanitize vales for writing/creating product templates and variants.

        Values need to be sanitized to keep values synchronized, and to be able to preprocess the
        vals in extensions of create/write.
        :param vals: create/write values dictionary
        """
        if 'type' in vals and 'detailed_type' not in vals:
            if vals['type'] not in self.mapped('type'):
                vals['detailed_type'] = vals['type']
        if 'detailed_type' in vals and 'type' not in vals:
            type_mapping = self._detailed_type_mapping()
            vals['type'] = type_mapping.get(vals['detailed_type'], vals['detailed_type'])

    def _get_related_fields_variant_template(self):
        """ Return a list of fields present on template and variants models and that are related"""
        return ['barcode', 'default_code', 'standard_price', 'volume', 'weight', 'packaging_ids']

    @api.model_create_multi
    def create(self, vals_list):
        ''' Store the initial standard price in order to be able to retrieve the cost of a product template for a given date'''
        for vals in vals_list:
            self._sanitize_vals(vals)
        templates = super(ProductTemplate, self).create(vals_list)
        if "create_product_product" not in self._context:
            templates._create_variant_ids()

        # This is needed to set given values to first variant after creation
        for template, vals in zip(templates, vals_list):
            related_vals = {}
            for field_name in self._get_related_fields_variant_template():
                if vals.get(field_name):
                    related_vals[field_name] = vals[field_name]
            if related_vals:
                template.write(related_vals)

        return templates

    def write(self, vals):
        self._sanitize_vals(vals)
        if 'uom_id' in vals or 'uom_po_id' in vals:
            uom_id = self.env['uom.uom'].browse(vals.get('uom_id')) or self.uom_id
            uom_po_id = self.env['uom.uom'].browse(vals.get('uom_po_id')) or self.uom_po_id
            if uom_id and uom_po_id and uom_id.category_id != uom_po_id.category_id:
                vals['uom_po_id'] = uom_id.id
        res = super(ProductTemplate, self).write(vals)
        if 'attribute_line_ids' in vals or (vals.get('active') and len(self.product_variant_ids) == 0):
            self._create_variant_ids()
        if 'active' in vals and not vals.get('active'):
            self.with_context(active_test=False).mapped('product_variant_ids').write({'active': vals.get('active')})
        if 'image_1920' in vals:
            self.env['product.product'].invalidate_model([
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
            default['name'] = _("%s (copy)", self.name)

        res = super().copy(default=default)

        # Since we don't copy the product template attribute values, we need to match the extra prices.
        for ptal, copied_ptal in zip(self.attribute_line_ids, res.attribute_line_ids):
            for ptav, copied_ptav in zip(ptal.product_template_value_ids, copied_ptal.product_template_value_ids):
                if not ptav.price_extra:
                    continue
                # security check
                if ptav.attribute_id == copied_ptav.attribute_id and ptav.product_attribute_value_id == copied_ptav.product_attribute_value_id:
                    copied_ptav.price_extra = ptav.price_extra
        return res

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
            # Pathological case: there is no limit, so we'll need to search on all products.
            # We iteratively _name_search with a larger bound, but not unbounded to avoid
            # performance regressions or OOM errors while manipulating extremely large list of ids.
            # For other cases, we use PREFETCH_MAX as an upper bound.
            search_limit = PREFETCH_MAX * 10 if not limit else PREFETCH_MAX
            products_ids = Product._name_search(name, args+domain, operator=operator, limit=search_limit, name_get_uid=name_get_uid)
            products = Product.browse(products_ids)
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
        tmpl_without_variant_ids = []
        # Useless if variants is not set up as no tmpl_without_variant_ids could exist.
        if self.env.user.has_group('product.group_product_variant') and (not limit or len(searched_ids) < limit):
            # The ORM has to be bypassed because it would require a NOT IN which is inefficient.
            self.env['product.product'].flush_model(['product_tmpl_id', 'active'])
            tmpl_without_variant_ids = self.env['product.template']._search([], order='id')
            tmpl_without_variant_ids.add_where("""
                NOT EXISTS (
                    SELECT product_tmpl_id
                    FROM product_product
                    WHERE product_product.active = true
                        AND product_template.id = product_product.product_tmpl_id
                )
            """)
        if tmpl_without_variant_ids:
            domain = expression.AND([args or [], [('id', 'in', tmpl_without_variant_ids)]])
            searched_ids |= set(super(ProductTemplate, self)._name_search(
                    name,
                    args=domain,
                    operator=operator,
                    limit=limit,
                    name_get_uid=name_get_uid))

        # re-apply product.template order + name_get
        return super(ProductTemplate, self)._name_search(
            '', args=[('id', 'in', list(searched_ids))],
            operator='ilike', limit=limit, name_get_uid=name_get_uid)

    def action_open_label_layout(self):
        action = self.env['ir.actions.act_window']._for_xml_id('product.action_open_label_layout')
        action['context'] = {'default_product_tmpl_ids': self.ids}
        return action

    def open_pricelist_rules(self):
        self.ensure_one()
        domain = ['|',
            ('product_tmpl_id', '=', self.id),
            ('product_id', 'in', self.product_variant_ids.ids)]
        return {
            'name': _('Price Rules'),
            'view_mode': 'tree,form',
            'views': [(self.env.ref('product.product_pricelist_item_tree_view_from_product').id, 'tree')],
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

    def price_compute(self, price_type, uom=None, currency=None, company=None, date=False):
        company = company or self.env.company
        date = date or fields.Date.context_today(self)

        self = self.with_company(company)
        if price_type == 'standard_price':
            # standard_price field can only be seen by users in base.group_user
            # Thus, in order to compute the sale price from the cost for users not in this group
            # We fetch the standard price as the superuser
            self = self.sudo()

        prices = dict.fromkeys(self.ids, 0.0)
        for template in self:
            price = template[price_type] or 0.0
            price_currency = template.currency_id
            if price_type == 'standard_price':
                if not price and template.product_variant_ids:
                    price = template.product_variant_ids[0].standard_price
                price_currency = template.cost_currency_id

            # yes, there can be attribute values for product template if it's not a variant YET
            # (see field product.attribute create_variant)
            if price_type == 'list_price' and self._context.get('current_attributes_price_extra'):
                # we have a list of price_extra that comes from the attribute values, we need to sum all that
                price += sum(self._context.get('current_attributes_price_extra'))

            if uom:
                price = template.uom_id._compute_price(price, uom)

            # Convert from current user company currency to asked one
            # This is right cause a field cannot be in more than one currency
            if currency:
                price = price_currency._convert(price, currency, company, date)

            prices[template.id] = price
        return prices

    def _create_variant_ids(self):
        if not self:
            return

        self.env.flush_all()
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
                        current_variants_to_create.append(tmpl_id._prepare_variant_values(combination))
                        if len(current_variants_to_create) > 1000:
                            raise UserError(_(
                                'The number of variants to generate is too high. '
                                'You should either not generate variants for each combination or generate them on demand from the sales order. '
                                'To do so, open the form view of attributes and change the mode of *Create Variants*.'))
                variants_to_create += current_variants_to_create
                variants_to_activate += current_variants_to_activate

            else:
                for variant in existing_variants.values():
                    is_combination_possible = tmpl_id._is_combination_possible_by_config(
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
        # We can't rely on existing invalidate because of the savepoint
        # in _unlink_or_archive.
        self.env.flush_all()
        self.env.invalidate_all()
        return True

    def _prepare_variant_values(self, combination):
        self.ensure_one()
        return {
            'product_tmpl_id': self.id,
            'product_template_attribute_value_ids': [(6, 0, combination.ids)],
            'active': self.active
        }

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
            - archived_combinations: list of archived combinations
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
        archived_products = self.with_context(active_test=False).product_variant_ids.filtered(lambda l: not l.active)
        active_combinations = set(tuple(product.product_template_attribute_value_ids.ids) for product in self.product_variant_ids)
        return {
            'exclusions': self._complete_inverse_exclusions(self._get_own_attribute_exclusions()),
            'archived_combinations': list(set(tuple(product.product_template_attribute_value_ids.ids) for product in archived_products) - active_combinations),
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

    def _get_placeholder_filename(self, field):
        image_fields = ['image_%s' % size for size in [1920, 1024, 512, 256, 128]]
        if field in image_fields:
            return 'product/static/img/placeholder.png'
        return super()._get_placeholder_filename(field)

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
                'product_name': self.product_variant_id.display_name,
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

    def get_contextual_price(self, product=None):
        return self._get_contextual_price(product=product)

    def _get_contextual_price(self, product=None):
        self.ensure_one()
        # YTI TODO: During website_sale cleaning, we should get rid of those crappy context thing
        pricelist = self._get_contextual_pricelist()
        if not pricelist:
            return 0.0

        quantity = self.env.context.get('quantity', 1.0)
        uom = self.env['uom.uom'].browse(self.env.context.get('uom'))
        date = self.env.context.get('date')
        return pricelist._get_product_price(product or self, quantity, uom=uom, date=date)

    def _get_contextual_pricelist(self):
        """ Get the contextual pricelist

        This method is meant to be overriden in other standard modules.
        """
        if self._context.get('pricelist'):
            return self.env['product.pricelist'].browse(self._context.get('pricelist'))
        return self.env['product.pricelist']

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class ResCompany(models.Model):
    _inherit = "res.company"

    @api.model_create_multi
    def create(self, vals_list):
        companies = super(ResCompany, self).create(vals_list)
        ProductPricelist = self.env['product.pricelist']
        for new_company in companies:
            pricelist = ProductPricelist.search([
                ('currency_id', '=', new_company.currency_id.id),
                ('company_id', '=', False)
            ], limit=1)
            if not pricelist:
                params = {'currency': new_company.currency_id.name}
                pricelist = ProductPricelist.create({
                    'name': _("Default %(currency)s pricelist") %  params,
                    'currency_id': new_company.currency_id.id,
                })
            self.env['ir.property']._set_default(
                'property_product_pricelist',
                'res.partner',
                pricelist,
                new_company,
            )
        return companies

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
                    self.env['ir.property']._set_default(
                        'property_product_pricelist',
                        'res.partner',
                        pricelist,
                        company,
                    )
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
    module_sale_product_matrix = fields.Boolean("Sales Grid Entry")
    module_loyalty = fields.Boolean("Promotions, Coupons, Gift Card & Loyalty Program")
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
        ('0', 'Kilograms'),
        ('1', 'Pounds'),
    ], 'Weight unit of measure', config_parameter='product.weight_in_lbs', default='0')
    product_volume_volume_in_cubic_feet = fields.Selection([
        ('0', 'Cubic Meters'),
        ('1', 'Cubic Feet'),
    ], 'Volume unit of measure', config_parameter='product.volume_in_cubic_feet', default='0')

    @api.onchange('group_product_variant')
    def _onchange_group_product_variant(self):
        """The product Configurator requires the product variants activated.
        If the user disables the product variants -> disable the product configurator as well"""
        if self.module_sale_product_matrix and not self.group_product_variant:
            self.module_sale_product_matrix = False

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
        super().set_values()
        if not self.group_discount_per_so_line:
            pl = self.env['product.pricelist'].search([('discount_policy', '=', 'without_discount')])
            pl.write({'discount_policy': 'with_discount'})

```

## File: models\res_country_group.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCountryGroup(models.Model):
    _inherit = 'res.country.group'

    pricelist_ids = fields.Many2many(
        comodel_name='product.pricelist',
        relation='res_country_group_pricelist_rel',
        column1='res_country_group_id',
        column2='pricelist_id',
        string="Pricelists")

```

## File: models\res_currency.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResCurrency(models.Model):
    _inherit = 'res.currency'

    def _activate_group_multi_currency(self):
        # for Sale/ POS - Multi currency flows require pricelists
        super()._activate_group_multi_currency()
        group_user = self.env.ref('base.group_user').sudo()
        group_user._apply_group(self.env.ref('product.group_product_pricelist'))

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    # NOT A REAL PROPERTY !!!!
    property_product_pricelist = fields.Many2one(
        comodel_name='product.pricelist',
        string="Pricelist",
        compute='_compute_product_pricelist',
        inverse="_inverse_product_pricelist",
        company_dependent=False,
        domain=lambda self: [('company_id', 'in', (self.env.company.id, False))],
        help="This pricelist will be used, instead of the default one, for sales to the current partner")

    @api.depends('country_id')
    @api.depends_context('company')
    def _compute_product_pricelist(self):
        res = self.env['product.pricelist']._get_partner_pricelist_multi(self.ids)
        for partner in self:
            partner.property_product_pricelist = res.get(partner._origin.id)

    def _inverse_product_pricelist(self):
        for partner in self:
            pls = self.env['product.pricelist'].search(
                [('country_group_ids.country_ids.code', '=', partner.country_id and partner.country_id.code or False)],
                limit=1
            )
            default_for_country = pls
            actual = self.env['ir.property']._get(
                'property_product_pricelist',
                'res.partner',
                'res.partner,%s' % partner.id)
            # update at each change country, and so erase old pricelist
            if partner.property_product_pricelist or (actual and default_for_country and default_for_country.id != actual.id):
                # keep the company of the current user before sudo
                self.env['ir.property']._set_multi(
                    'property_product_pricelist',
                    partner._name,
                    {partner.id: partner.property_product_pricelist or default_for_country.id},
                    default_value=default_for_country.id
                )

    def _commercial_fields(self):
        return super()._commercial_fields() + ['property_product_pricelist']

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

# don't try to be a good boy and sort imports alphabetically.
# `product.template` should be initialised before `product.product`
from . import product_template
from . import product_product

from . import decimal_precision
from . import product_attribute
from . import product_category
from . import product_packaging
from . import product_pricelist
from . import product_pricelist_item
from . import product_supplierinfo
from . import product_tag
from . import res_company
from . import res_config_settings
from . import res_country_group
from . import res_currency
from . import res_partner
from . import uom_uom

```

## File: populate\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import collections

from odoo import models
from odoo.tools import populate
from odoo.addons.stock.populate.stock import COMPANY_NB_WITH_STOCK

_logger = logging.getLogger(__name__)


class ProductCategory(models.Model):
    _inherit = "product.category"
    _populate_sizes = {"small": 50, "medium": 500, "large": 5_000}

    def _populate_factories(self):
        return [("name", populate.constant('PC_{counter}'))]

    def _populate(self, size):
        categories = super()._populate(size)
        # Set parent/child relation
        self._populate_set_parents(categories, size)
        return categories

    def _populate_set_parents(self, categories, size):
        _logger.info('Set parent/child relation of product categories')
        parent_ids = []
        rand = populate.Random('product.category+parent_generator')

        for category in categories:
            if rand.random() < 0.25:
                parent_ids.append(category.id)

        categories -= self.browse(parent_ids)  # Avoid recursion in parent-child relations.
        parent_childs = collections.defaultdict(lambda: self.env['product.category'])
        for category in categories:
            if rand.random() < 0.25:  # 1/4 of remaining categories have a parent.
                parent_childs[rand.choice(parent_ids)] |= category

        for count, (parent, children) in enumerate(parent_childs.items()):
            if (count + 1) % 1000 == 0:
                _logger.info('Setting parent: %s/%s', count + 1, len(parent_childs))
            children.write({'parent_id': parent})

class ProductProduct(models.Model):
    _inherit = "product.product"
    _populate_sizes = {"small": 150, "medium": 5_000, "large": 50_000}
    _populate_dependencies = ["product.category"]

    def _populate_get_types(self):
        return ["consu", "service"], [2, 1]

    def _populate_get_product_factories(self):
        category_ids = self.env.registry.populated_models["product.category"]
        types, types_distribution = self._populate_get_types()

        def get_rand_float(values, counter, random):
            return random.randrange(0, 1500) * random.random()

        # TODO sale & purchase uoms

        return [
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("active", populate.randomize([True, False], [0.8, 0.2])),
            ("type", populate.randomize(types, types_distribution)),
            ("categ_id", populate.randomize(category_ids)),
            ("list_price", populate.compute(get_rand_float)),
            ("standard_price", populate.compute(get_rand_float)),
        ]

    def _populate_factories(self):
        return [
            ("name", populate.constant('product_product_name_{counter}')),
            ("description", populate.constant('product_product_description_{counter}')),
            ("default_code", populate.constant('PP-{counter}')),
            ("barcode", populate.constant('BARCODE-PP-{counter}')),
        ] + self._populate_get_product_factories()


class SupplierInfo(models.Model):
    _inherit = 'product.supplierinfo'

    _populate_sizes = {'small': 450, 'medium': 15_000, 'large': 180_000}
    _populate_dependencies = ['res.partner', 'product.product', 'product.template']

    def _populate_factories(self):
        random = populate.Random('product_with_supplierinfo')
        company_ids = self.env.registry.populated_models['res.company'][:COMPANY_NB_WITH_STOCK] + [False]
        partner_ids = self.env.registry.populated_models['res.partner']
        product_templates_ids = self.env['product.product'].browse(self.env.registry.populated_models['product.product']).product_tmpl_id.ids
        product_templates_ids += self.env.registry.populated_models['product.template']
        product_templates_ids = random.sample(product_templates_ids, int(len(product_templates_ids) * 0.95))

        def get_company_id(values, counter, random):
            partner = self.env['res.partner'].browse(values['partner_id'])
            if partner.company_id:
                return partner.company_id.id
            return random.choice(company_ids)

        def get_delay(values, counter, random):
            # 5 % with huge delay (between 5 month and 6 month), otherwise between 1 and 10 days
            if random.random() > 0.95:
                return random.randint(150, 210)
            return random.randint(1, 10)

        return [
            ('partner_id', populate.randomize(partner_ids)),
            ('company_id', populate.compute(get_company_id)),
            ('product_tmpl_id', populate.iterate(product_templates_ids)),
            ('product_name', populate.constant("SI-{counter}")),
            ('sequence', populate.randint(1, 10)),
            ('min_qty', populate.randint(0, 10)),
            ('price', populate.randint(10, 100)),
            ('delay', populate.compute(get_delay)),
        ]

```

## File: populate\product_pricelist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import populate
from datetime import timedelta, date


class Pricelist(models.Model):
    _inherit = "product.pricelist"
    _populate_sizes = {"small": 20, "medium": 100, "large": 1_500}
    _populate_dependencies = ["res.company"]

    def _populate(self, size):

        # Reflect the settings with data created
        self.env['res.config.settings'].create({
            'group_product_pricelist': True,  # Activate pricelist
            'group_sale_pricelist': True,  # Activate advanced pricelist
        }).execute()

        return super()._populate(size)

    def _populate_factories(self):
        company_ids = self.env.registry.populated_models["res.company"]

        return [
            ("company_id", populate.iterate(company_ids + [False for i in range(len(company_ids))])),
            ("name", populate.constant('product_pricelist_{counter}')),
            ("currency_id", populate.randomize(self.env["res.currency"].search([("active", "=", True)]).ids)),
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("discount_policy", populate.randomize(["with_discount", "without_discount"])),
            ("active", populate.randomize([True, False], [0.8, 0.2])),
        ]


class PricelistItem(models.Model):
    _inherit = "product.pricelist.item"
    _populate_sizes = {"small": 500, "medium": 5_000, "large": 50_000}
    _populate_dependencies = ["product.product", "product.template", "product.pricelist"]

    def _populate_factories(self):
        pricelist_ids = self.env.registry.populated_models["product.pricelist"]
        product_ids = self.env.registry.populated_models["product.product"]
        p_tmpl_ids = self.env.registry.populated_models["product.template"]
        categ_ids = self.env.registry.populated_models["product.category"]

        def get_target_info(iterator, field_name, model_name):
            random = populate.Random("pricelist_target")
            for values in iterator:
                # If product population is updated to consider multi company
                # the company of product would have to be considered
                # for product_id & product_tmpl_id
                applied_on = values["applied_on"]
                if applied_on == "0_product_variant":
                    values["product_id"] = random.choice(product_ids)
                elif applied_on == "1_product":
                    values["product_tmpl_id"] = random.choice(p_tmpl_ids)
                elif applied_on == "2_product_category":
                    values["categ_id"] = random.choice(categ_ids)
                yield values

        def get_prices(iterator, field_name, model_name):
            random = populate.Random("pricelist_prices")
            for values in iterator:
                # Fixed price, percentage, formula
                compute_price = values["compute_price"]
                if compute_price == "fixed":
                    # base = "list_price" = default
                    # fixed_price
                    values["fixed_price"] = random.randint(1, 1000)
                elif compute_price == "percentage":
                    # base = "list_price" = default
                    # percent_price
                    values["percent_price"] = random.randint(1, 100)
                else:  # formula
                    # pricelist base not considered atm.
                    values["base"] = random.choice(["list_price", "standard_price"])
                    values["price_discount"] = random.randint(0, 100)
                    # price_min_margin, price_max_margin
                    # price_round ??? price_discount, price_surcharge
                yield values

        now = date.today()

        def get_date_start(values, counter, random):
            if random.random() > 0.5:  # 50 % of chance to have validation dates
                return now + timedelta(days=random.randint(-20, 20))
            else:
                False

        def get_date_end(values, counter, random):
            if values['date_start']:  # 50 % of chance to have validation dates
                return values['date_start'] + timedelta(days=random.randint(5, 100))
            else:
                False

        return [
            ("pricelist_id", populate.randomize(pricelist_ids)),
            ("applied_on", populate.randomize(
                ["3_global", "2_product_category", "1_product", "0_product_variant"],
                [5, 3, 2, 1],
            )),
            ("compute_price", populate.randomize(
                ["fixed", "percentage", "formula"],
                [5, 3, 1],
            )),
            ("_price", get_prices),
            ("_target", get_target_info),
            ("min_quantity", populate.randint(0, 50)),
            ("date_start", populate.compute(get_date_start)),
            ("date_end", populate.compute(get_date_end)),
        ]

```

## File: populate\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
from collections import defaultdict
from functools import reduce

from odoo import models
from odoo.tools import populate

_logger = logging.getLogger(__name__)


class ProductAttribute(models.Model):
    _inherit = "product.attribute"
    _populate_sizes = {"small": 20, "medium": 150, "large": 750}

    def _populate(self, size):

        # Reflect the settings with data created
        self.env['res.config.settings'].create({
            'group_product_variant': True,  # Activate variant
        }).execute()

        return super()._populate(size)

    def _populate_factories(self):
        return [
            ("name", populate.constant('PA_{counter}')),
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("create_variant", populate.randomize(["always", "dynamic", "no_variant"])),
        ]


class ProductAttributeValue(models.Model):
    _inherit = "product.attribute.value"
    _populate_dependencies = ["product.attribute"]
    _populate_sizes = {"small": 100, "medium": 1_000, "large": 10_000}

    def _populate_factories(self):
        attribute_ids = self.env.registry.populated_models["product.attribute"]

        return [
            ("name", populate.constant('PAV_{counter}')),
            ("sequence", populate.randomize([False] + [i for i in range(1, 101)])),
            ("attribute_id", populate.randomize(attribute_ids)),
        ]


class ProductTemplate(models.Model):
    _inherit = "product.template"
    _populate_sizes = {"small": 150, "medium": 5_000, "large": 50_000}
    _populate_dependencies = ["product.attribute.value", "product.category"]

    def _populate(self, size):
        res = super()._populate(size)

        def set_barcode_variant(sample_ratio):
            random = populate.Random('barcode_product_template')
            product_variants_ids = res.product_variant_ids.ids
            product_variants_ids = random.sample(product_variants_ids, int(len(product_variants_ids) * sample_ratio))
            product_variants = self.env['product.product'].browse(product_variants_ids)
            _logger.info('Set barcode on product variants (%s)', len(product_variants))
            for product in product_variants:
                product.barcode = "BARCODE-PT-%s" % product.id

        set_barcode_variant(0.85)

        return res

    def _populate_factories(self):
        attribute_ids = self.env.registry.populated_models["product.attribute"]
        attribute_ids_by_types = defaultdict(list)
        attributes = self.env["product.attribute"].browse(attribute_ids)
        for attr in attributes:
            attribute_ids_by_types[attr.create_variant].append(attr.id)

        def get_attributes(values, counter, random):
            if random.random() < 0.20:  # 20 % chance to have no attributes
                return False
            attributes_qty = random.choices(
                [1, 2, 3, 4, 5, 6, 8, 10],
                [10, 9, 8, 7, 6, 4, 1, 0.5],
            )[0]
            attr_line_vals = []
            attribute_used_ids = attribute_ids
            if random.random() < 0.20:  # 20 % chance of using only always attributes (to test when product has lot of variant)
                attribute_used_ids = attribute_ids_by_types["always"]

            no_variant = False
            values_count = [0 for i in range(attributes_qty)]

            def will_exceed(i):
                return not no_variant and reduce((lambda x, y: (x or 1) * (y or 1)), values_count[i:] + [values_count[i] + 1] + values_count[:i]) > 1000

            for i in range(attributes_qty):
                if will_exceed(i):
                    return attr_line_vals
                attr_id = random.choice(attribute_used_ids)
                attr = self.env["product.attribute"].browse(attr_id)
                if attr.create_variant == "dynamic":
                    no_variant = True
                if not attr.value_ids:
                    # attribute without any value
                    continue
                nb_values = len(attr.value_ids)
                vals_qty = random.randrange(nb_values) + 1
                value_ids = set()
                for __ in range(vals_qty):
                    # Ensure that we wouldn't have > 1k variants with the generated attributes combination
                    if will_exceed(i):
                        break
                    random_value_id = attr.value_ids[random.randrange(nb_values)].id
                    if random_value_id not in value_ids:
                        values_count[i] += 1
                        value_ids.add(random_value_id)

                attr_line_vals.append((0, 0, {
                    "attribute_id": attr_id,
                    "value_ids": [(6, 0, list(value_ids))],
                }))

            return attr_line_vals

        return [
            ("name", populate.constant('product_template_name_{counter}')),
            ("description", populate.constant('product_template_description_{counter}')),
            ("default_code", populate.constant('PT-{counter}')),
            ("attribute_line_ids", populate.compute(get_attributes)),
        ] + self.env['product.product']._populate_get_product_factories()


class ProductTemplateAttributeExclusion(models.Model):
    _inherit = "product.template.attribute.exclusion"
    _populate_dependencies = ["product.template"]
    _populate_sizes = {"small": 200, "medium": 1_000, "large": 5_000}

    def _populate_factories(self):
        p_tmpl_ids = self.env.registry.populated_models["product.template"]

        configurable_templates = self.env["product.template"].search([
            ('id', 'in', p_tmpl_ids),
            ('has_configurable_attributes', '=', True),
        ])
        tmpl_ids_possible = []
        multi_values_attribute_lines_by_tmpl = {}
        for template in configurable_templates:
            multi_values_attribute_lines = template.attribute_line_ids.filtered(
                lambda l: len(l.value_ids) > 1
            )
            if len(multi_values_attribute_lines) < 2:
                continue
            tmpl_ids_possible.append(template.id)
            multi_values_attribute_lines_by_tmpl[template.id] = multi_values_attribute_lines

        def get_product_template_attribute_value_id(values, counter, random):
            return random.choice(multi_values_attribute_lines_by_tmpl[values['product_tmpl_id']].product_template_value_ids.ids)

        def get_value_ids(values, counter, random):
            attr_val = self.env['product.template.attribute.value'].browse(values['product_template_attribute_value_id']).attribute_line_id
            remaining_lines = multi_values_attribute_lines_by_tmpl[values['product_tmpl_id']] - attr_val
            return [(
                # TODO: multiple values
                6, 0, [random.choice(remaining_lines.product_template_value_ids).id]
            )]

        return [
            ("product_tmpl_id", populate.randomize(tmpl_ids_possible)),
            ("product_template_attribute_value_id", populate.compute(get_product_template_attribute_value_id)),
            ("value_ids", populate.compute(get_value_ids)),
        ]


class ProductTemplateAttributeValue(models.Model):
    _inherit = "product.template.attribute.value"
    _populate_dependencies = ["product.template"]

    def _populate(self, size):
        p_tmpl_ids = self.env.registry.populated_models["product.template"]
        ptavs = self.search([('product_tmpl_id', 'in', p_tmpl_ids)])
        # ptavs are automatically created when specifying attribute lines on product templates.

        rand = populate.Random("ptav_extra_price_generator")
        for ptav in ptavs:
            if rand.random() < 0.50:  # 50% of having a extra price
                ptav.price_extra = rand.randrange(500) * rand.random()

        return ptavs

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product
from . import product_template
from . import product_pricelist

```

## File: report\product_label_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import _, models
from odoo.exceptions import UserError

def _prepare_data(env, data):
    # change product ids by actual product object to get access to fields in xml template
    # we needed to pass ids because reports only accepts native python types (int, float, strings, ...)
    if data.get('active_model') == 'product.template':
        Product = env['product.template'].with_context(display_default_code=False)
    elif data.get('active_model') == 'product.product':
        Product = env['product.product'].with_context(display_default_code=False)
    else:
        raise UserError(_('Product model not defined, Please contact your administrator.'))

    total = 0
    qty_by_product_in = data.get('quantity_by_product')
    # search for products all at once, ordered by name desc since popitem() used in xml to print the labels
    # is LIFO, which results in ordering by product name in the report
    products = Product.search([('id', 'in', [int(p) for p in qty_by_product_in.keys()])], order='name desc')
    quantity_by_product = defaultdict(list)
    for product in products:
        q = qty_by_product_in[str(product.id)]
        quantity_by_product[product].append((product.barcode, q))
        total += q
    if data.get('custom_barcodes'):
        # we expect custom barcodes format as: {product: [(barcode, qty_of_barcode)]}
        for product, barcodes_qtys in data.get('custom_barcodes').items():
            quantity_by_product[Product.browse(int(product))] += (barcodes_qtys)
            total += sum(qty for _, qty in barcodes_qtys)

    layout_wizard = env['product.label.layout'].browse(data.get('layout_wizard'))
    if not layout_wizard:
        return {}

    return {
        'quantity': quantity_by_product,
        'rows': layout_wizard.rows,
        'columns': layout_wizard.columns,
        'page_numbers': (total - 1) // (layout_wizard.rows * layout_wizard.columns) + 1,
        'price_included': data.get('price_included'),
        'extra_html': layout_wizard.extra_html,
    }

class ReportProductTemplateLabel(models.AbstractModel):
    _name = 'report.product.report_producttemplatelabel'
    _description = 'Product Label Report'

    def _get_report_values(self, docids, data):
        return _prepare_data(self.env, data)

class ReportProductTemplateLabelDymo(models.AbstractModel):
    _name = 'report.product.report_producttemplatelabel_dymo'
    _description = 'Product Label Report'

    def _get_report_values(self, docids, data):
        return _prepare_data(self.env, data)

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
                                        <div t-field="packaging.barcode" t-options="{'widget': 'barcode', 'symbology': 'auto', 'width': 600, 'height': 150, 'img_style': 'width:100%;height:20%;'}"/>
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

## File: report\product_pricelist_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProductPricelistReport(models.AbstractModel):
    _name = 'report.product.report_pricelist'
    _description = 'Pricelist Report'

    def _get_report_values(self, docids, data):
        return self._get_report_data(data, 'pdf')

    @api.model
    def get_html(self, data):
        render_values = self._get_report_data(data, 'html')
        return self.env['ir.qweb']._render('product.report_pricelist_page', render_values)

    def _get_report_data(self, data, report_type='html'):
        quantities = data.get('quantities', [1])

        data_pricelist_id = data.get('pricelist_id')
        pricelist_id = data_pricelist_id and int(data_pricelist_id)
        pricelist = self.env['product.pricelist'].browse(pricelist_id).exists()
        if not pricelist:
            pricelist = self.env['product.pricelist'].search([], limit=1)

        active_model = data.get('active_model', 'product.template')
        active_ids = data.get('active_ids') or []
        is_product_tmpl = active_model == 'product.template'
        ProductClass = self.env[active_model]

        products = ProductClass.browse(active_ids) if active_ids else ProductClass.search([('sale_ok', '=', True)])
        products_data = [
            self._get_product_data(is_product_tmpl, product, pricelist, quantities)
            for product in products
        ]

        return {
            'is_html_type': report_type == 'html',
            'is_product_tmpl': is_product_tmpl,
            'is_visible_title': data.get('is_visible_title', False) and bool(data['is_visible_title']),
            'pricelist': pricelist,
            'products': products_data,
            'quantities': quantities,
        }

    def _get_product_data(self, is_product_tmpl, product, pricelist, quantities):
        data = {
            'id': product.id,
            'name': is_product_tmpl and product.name or product.display_name,
            'price': dict.fromkeys(quantities, 0.0),
            'uom': product.uom_id.name,
        }
        for qty in quantities:
            data['price'][qty] = pricelist._get_product_price(product, qty)

        if is_product_tmpl and product.product_variant_count > 1:
            data['variants'] = [
                self._get_product_data(False, variant, pricelist, quantities)
                for variant in product.product_variant_ids
            ]

        return data

```

## File: report\product_pricelist_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="report_pricelist_page">
        <div class="container bg-white p-4 my-4">
            <div class="row my-3">
                <div class="col-12" t-if="is_visible_title">
                    <h2 t-if="is_html_type">
                        Pricelist:
                        <a href="#" class="o_action" data-model="product.pricelist" t-att-data-res-id="pricelist.id">
                            <t t-esc="pricelist.display_name"/>
                        </a>
                    </h2>
                    <h2 t-else="">
                        Pricelist: <t t-esc="pricelist.display_name"/>
                    </h2>
                </div>
            </div>
            <div class="row">
                <div t-att-class="'text-center' + (' offset-4' if is_html_type else ' offset-3')">
                    <strong>Sales Order Line Quantities (price per unit)</strong>
                </div>
            </div>
            <div class="row">
                <div class="col-12">
                    <table class="table table-sm">
                        <thead>
                            <tr>
                                <th>Products</th>
                                <th groups="uom.group_uom">UoM</th>
                                <t t-foreach="quantities" t-as="qty">
                                    <th class="text-end"><t t-esc="qty"/></th>
                                </t>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="products" t-as="product">
                                <tr>
                                    <td t-att-class="is_product_tmpl and 'fw-bold' or None">
                                        <a t-if="is_html_type" href="#" class="o_action" t-att-data-model="is_product_tmpl and 'product.template' or 'product.product'" t-att-data-res-id="product['id']">
                                            <t t-esc="product['name']"/>
                                        </a>
                                        <t t-else="">
                                            <t t-esc="product['name']"/>
                                        </t>
                                    </td>
                                    <td groups="uom.group_uom">
                                        <t t-esc="product['uom']"/>
                                    </td>
                                    <t t-foreach="quantities" t-as="qty">
                                        <td class="text-end">
                                            <t t-esc="product['price'][qty]" t-options='{"widget": "monetary", "display_currency": pricelist.currency_id}'/>
                                        </td>
                                    </t>
                                </tr>
                                <t t-if="is_product_tmpl and 'variants' in product">
                                    <tr t-foreach="product['variants']" t-as="variant">
                                        <td>
                                            <a t-if="is_html_type" href="#" class="o_action ms-4" data-model="product.product" t-att-data-res-id="variant['id']">
                                                <t t-esc="variant['name']"/>
                                            </a>
                                            <span t-else="" class="ms-4" t-esc="variant['name']"/>
                                        </td>
                                        <td groups="uom.group_uom">
                                            <t t-esc="product['uom']"/>
                                        </td>
                                        <t t-foreach="quantities" t-as="qty">
                                            <td class="text-end">
                                                <t t-esc="variant['price'][qty]" t-options='{"widget": "monetary", "display_currency": pricelist.currency_id}'/>
                                            </td>
                                        </t>
                                    </tr>
                                </t>
                            </t>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </template>

    <template id="report_pricelist">
        <t t-call="web.basic_layout">
            <div class="page">
                <t t-call="product.report_pricelist_page"/>
            </div>
            <p style="page-break-before:always;"> </p>
        </t>
    </template>

</odoo>

```

## File: report\product_product_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <template id="report_simple_label2x7">
            <t t-set="barcode_size" t-value="'width:33mm;height:14mm'"/>
            <t t-set="table_style" t-value="'width:97mm;height:37.1mm;' + table_style"/>
            <td t-att-style="make_invisible and 'visibility:hidden;'" >
                <div class="o_label_full" t-att-style="table_style">
                    <div class="o_label_name">
                        <strong t-field="product.display_name"/>
                    </div>
                    <div class="o_label_data">
                        <div class="text-center o_label_left_column">
                            <span class="text-nowrap" t-field="product.default_code"/>
                            <t t-if="barcode">
                                <div t-out="barcode" t-options="{'widget': 'barcode', 'symbology': 'auto', 'img_style': barcode_size}"/>
                                <span class="text-center" t-out="barcode"/>
                            </t>
                        </div>
                        <div class="text-end" style="line-height:normal">
                            <div class="o_label_extra_data">
                                <t t-out="extra_html"/>
                            </div>
                            <t t-if="product.is_product_variant">
                                <strong class="o_label_price" t-field="product.lst_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                            </t>
                            <t t-else="">
                                <strong class="o_label_price" t-field="product.list_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                            </t>
                        </div>
                        <div class="o_label_clear"></div>
                    </div>
                </div>
            </td>
        </template>

        <template id="report_simple_label4x7">
            <t t-set="barcode_size" t-value="'width:33mm;height:8mm'"/>
            <t t-set="table_style" t-value="'width:47mm;height:37.1mm;' + table_style"/>
            <td t-att-style="make_invisible and 'visibility:hidden;'" >
                <div class="o_label_full" t-att-style="table_style">
                    <div class="o_label_name">
                        <strong t-field="product.display_name"/>
                    </div>
                    <div class="text-end" style="padding-top:0;padding-bottom:0">
                        <t t-if="product.is_product_variant">
                            <strong class="o_label_price_medium" t-field="product.lst_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                        </t>
                        <t t-else="">
                            <strong class="o_label_price_medium" t-field="product.list_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                        </t>
                    </div>
                    <div class= "text-center o_label_small_barcode">
                        <span class="text-nowrap" t-field="product.default_code"/>
                        <t t-if="barcode">
                            <div t-out="barcode" style="padding:0" t-options="{'widget': 'barcode', 'symbology': 'auto', 'img_style': barcode_size}"/>
                            <span class="text-center" t-out="barcode"/>
                        </t>
                    </div>
                </div>
            </td>
        </template>

        <template id="report_simple_label4x12">
            <t t-set="barcode_size" t-value="'width:33mm;height:4mm'"/>
            <t t-set="table_style" t-value="'width:43mm;height:19mm;' + table_style"/>
            <td t-att-style="make_invisible and 'visibility:hidden;'" >
                <div class="o_label_full o_label_small_text" t-att-style="table_style">
                    <div class="o_label_name">
                        <strong t-field="product.display_name"/>
                    </div>
                    <t t-if="price_included">
                        <div class="o_label_left_column">
                            <span class="text-nowrap" t-field="product.default_code"/>
                        </div>
                        <div class="o_label_price_medium text-end">
                            <t t-if="product.is_product_variant">
                                <strong t-field="product.lst_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                            </t>
                            <t t-else="">
                                <strong t-field="product.list_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                            </t>
                        </div>
                    </t>
                    <t t-else="">
                        <div class="o_label_left_column o_label_full_with">
                            <span class="text-nowrap" t-field="product.default_code"/>
                        </div>
                    </t>
                    <div class= "text-center o_label_small_barcode">
                        <t t-if="barcode">
                            <div t-out="barcode" style="padding:0" t-options="{'widget': 'barcode', 'symbology': 'auto', 'img_style': barcode_size}"/>
                            <span class="text-center" t-out="barcode"/>
                        </t>
                    </div>
                </div>
            </td>
        </template>

        <template id="report_simple_label_dymo">
            <div class="o_label_sheet o_label_dymo" t-att-style="padding_page">
                <div class="o_label_full" t-att-style="table_style">
                    <div class= "text-start o_label_small_barcode">
                        <t t-if="barcode">
                            <!-- `quiet=0` to remove the left and right margins on the barcode -->
                            <div t-out="barcode" style="padding:0" t-options="{'widget': 'barcode', 'quiet': 0, 'symbology': 'auto', 'img_style': barcode_size}"/>
                            <div class="o_label_name" style="height:1.7em;background-color: transparent;">
                                <span t-out="barcode"/>
                            </div>
                        </t>
                    </div>
                    <div class="o_label_name" style="line-height: 100%;background-color: transparent;padding-top: 1px;">
                        <span t-if="product.is_product_variant" t-field="product.display_name"/>
                        <span t-else="" t-field="product.name"/>
                    </div>
                    <div class="o_label_left_column">
                        <small class="text-nowrap" t-field="product.default_code"/>
                    </div>
                    <div class="text-end" style="padding: 0 4px;">
                        <t t-if="product.is_product_variant">
                            <strong class="o_label_price_small" t-field="product.lst_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                        </t>
                        <t t-else="">
                            <strong class="o_label_price_small" t-field="product.list_price" t-options="{'widget': 'monetary', 'label_price': True}"/>
                        </t>
                        <div t-if="False" class="o_label_extra_data">
                            <t t-out="extra_html"/>
                        </div>
                    </div>
                </div>
            </div>
        </template>

        <template id="report_productlabel">
            <t t-call="web.html_container">
                <t t-if="columns and rows">
                    <t t-if="columns == 2 and rows == 7">
                        <t t-set="padding_page" t-value="'padding: 14mm 3mm'"/>
                        <t t-set="report_to_call" t-value="'product.report_simple_label2x7'"/>
                    </t>
                    <t t-if="columns == 4 and rows == 7">
                        <t t-set="padding_page" t-value="'padding: 14mm 3mm'"/>
                        <t t-set="report_to_call" t-value="'product.report_simple_label4x7'"/>
                    </t>
                    <t t-if="columns == 4 and rows == 12">
                        <t t-set="padding_page" t-value="'padding: 20mm 8mm'"/>
                        <t t-set="report_to_call" t-value="'product.report_simple_label4x12'"/>
                    </t>
                    <t t-foreach="range(page_numbers)" t-as="page">
                        <div class="o_label_sheet" t-att-style="padding_page">
                            <table class="my-0 table table-sm table-borderless">
                                <t t-foreach="range(rows)" t-as="row">
                                    <tr>
                                        <t t-foreach="range(columns)" t-as="column">
                                            <t t-if="not current_quantity and quantity and not (current_data and current_data[1])">
                                                <t t-set="current_data" t-value="quantity.popitem()"/>
                                                <t t-set="product" t-value="current_data[0]"/>
                                                <t t-set="barcode_and_qty" t-value="current_data[1].pop()"/>
                                                <t t-set="barcode" t-value="barcode_and_qty[0]"/>
                                                <t t-set="current_quantity" t-value="barcode_and_qty[1]"/>
                                            </t>
                                            <t t-if="current_quantity">
                                                <t t-set="make_invisible" t-value="False"/>
                                                <t t-set="current_quantity" t-value="current_quantity - 1"/>
                                            </t>
                                            <t t-elif="current_data and current_data[1]">
                                                <t t-set="barcode_and_qty" t-value="current_data[1].pop()"/>
                                                <t t-set="barcode" t-value="barcode_and_qty[0]"/>
                                                <t t-set="current_quantity" t-value="barcode_and_qty[1] - 1"/>
                                            </t>
                                            <t t-else="">
                                                <t t-set="make_invisible" t-value="True"/>
                                            </t>
                                            <t t-set="table_style" t-value="'border: 1px solid %s;' % (product.env.user.company_id.primary_color or 'black')"/>
                                            <t t-call="{{report_to_call}}"/>
                                        </t>
                                    </tr>
                                </t>
                            </table>
                        </div>
                    </t>
                </t>
            </t>
        </template>

        <template id="report_productlabel_dymo">
            <t t-call="web.html_container">
                <t t-set="barcode_size" t-value="'width:45.5mm;height:7.5mm'"/>
                <t t-set="table_style" t-value="'width:100%;height:32mm;'"/>
                <t t-set="padding_page" t-value="'padding: 2mm'"/>
                <t t-foreach="quantity.items()" t-as="barcode_and_qty_by_product">
                    <t t-set="product" t-value="barcode_and_qty_by_product[0]"/>
                    <t t-foreach="barcode_and_qty_by_product[1]" t-as="barcode_and_qty">
                        <t t-set="barcode" t-value="barcode_and_qty[0]"/>
                        <t t-foreach="range(barcode_and_qty[1])" t-as="qty">
                            <t t-call="product.report_simple_label_dymo"/>
                        </t>
                    </t>
                </t>
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
        <record id="paperformat_label_sheet" model="report.paperformat">
            <field name="name">A4 Label Sheet</field>
            <field name="default" eval="True" />
            <field name="format">A4</field>
            <field name="page_height">0</field>
            <field name="page_width">0</field>
            <field name="orientation">Portrait</field>
            <field name="margin_top">0</field>
            <field name="margin_bottom">0</field>
            <field name="margin_left">0</field>
            <field name="margin_right">0</field>
            <field name="disable_shrinking" eval="True"/>
            <field name="dpi">96</field>
        </record>
        <record id="report_product_template_label" model="ir.actions.report">
            <field name="name">Product Label (PDF)</field>
            <field name="model">product.template</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_producttemplatelabel</field>
            <field name="report_file">product.report_producttemplatelabel</field>
            <field name="paperformat_id" ref="product.paperformat_label_sheet"/>
            <field name="print_report_name">'Products Labels - %s' % (object.name)</field>
            <field name="binding_model_id" eval="False"/>
            <field name="binding_type">report</field>
        </record>

        <record id="report_product_packaging" model="ir.actions.report">
            <field name="name">Product Packaging (PDF)</field>
            <field name="model">product.packaging</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_packagingbarcode</field>
            <field name="report_file">product.report_packagingbarcode</field>
            <field name="print_report_name">'Products packaging - %s' % (object.name)</field>
            <field name="binding_model_id" ref="product.model_product_packaging"/>
            <field name="binding_type">report</field>
        </record>

        <record id="action_report_pricelist" model="ir.actions.report">
            <field name="name">Pricelist</field>
            <field name="model">product.product</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_pricelist</field>
            <field name="report_file">product.report_pricelist</field>
        </record>

        <record id="paperformat_label_sheet_dymo" model="report.paperformat">
            <field name="name">Dymo Label Sheet</field>
            <field name="default" eval="True" />
            <field name="format">custom</field>
            <field name="page_height">57</field>
            <field name="page_width">32</field>
            <field name="orientation">Landscape</field>
            <field name="margin_top">0</field>
            <field name="margin_bottom">0</field>
            <field name="margin_left">0</field>
            <field name="margin_right">0</field>
            <field name="disable_shrinking" eval="True"/>
            <field name="dpi">96</field>
        </record>

        <record id="report_product_template_label_dymo" model="ir.actions.report">
            <field name="name">Product Label (PDF)</field>
            <field name="model">product.template</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_producttemplatelabel_dymo</field>
            <field name="report_file">product.report_producttemplatelabel_dymo</field>
            <field name="paperformat_id" ref="product.paperformat_label_sheet_dymo"/>
            <field name="print_report_name">'Products Labels - %s' % (object.name)</field>
            <field name="binding_type">report</field>
        </record>
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
            <t t-call="product.report_productlabel">
                <t t-set="products" t-value="products"/>
            </t>
        </div>
    </t>
</template>

<template id="report_producttemplatelabel_dymo">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-call="product.report_productlabel_dymo">
                <t t-set="products" t-value="products"/>
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

from . import product_label_report
from . import product_pricelist_report

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
access_product_attribute_custom_value_user,product.attribute.custom value manager,model_product_attribute_custom_value,base.group_user,1,0,0,0
access_product_product_attribute,product.template.attribute value,model_product_template_attribute_value,base.group_user,1,0,0,0
access_product_template_attribute_exclusion,product..template.attribute exclusion,model_product_template_attribute_exclusion,base.group_user,1,0,0,0
access_product_template_attribute_line,product.template.attribute line,model_product_template_attribute_line,base.group_user,1,0,0,0
access_product_tag_user,product.tag.user,model_product_tag,base.group_user,1,0,0,0
access_product_category_manager,product.category.manager,model_product_category,base.group_system,1,1,1,1
access_product_template_manager,product.template.manager,model_product_template,base.group_system,1,1,1,1
access_product_packaging_manager,product.packaging.manager,model_product_packaging,base.group_system,1,1,1,1
access_product_supplierinfo_manager,product.supplierinfo.manager,model_product_supplierinfo,base.group_system,1,1,1,1
access_product_pricelist_manager,product.pricelist.manager,model_product_pricelist,base.group_system,1,1,1,1
access_product_pricelist_item_manager,product.pricelist.item.manager,model_product_pricelist_item,base.group_system,1,1,1,1
access_product_product_manager,product.product manager,model_product_product,base.group_system,1,1,1,1
access_product_attribute_manager,product.attribute manager,model_product_attribute,base.group_system,1,1,1,1
access_product_attribute_value_manager,product.attribute value manager,model_product_attribute_value,base.group_system,1,1,1,1
access_product_attribute_custom_value_manager,product.attribute.custom value manager,model_product_attribute_custom_value,base.group_system,1,1,1,1
access_product_product_attribute_manager,product.template.attribute value manager,model_product_template_attribute_value,base.group_system,1,1,1,1
access_product_template_attribute_exclusion_manager,product.template.attribute exclusion manager,model_product_template_attribute_exclusion,base.group_system,1,1,1,1
access_product_template_attribute_line_manager,product.template.attribute line manager,model_product_template_attribute_line,base.group_system,1,1,1,1
access_product_label_layout_user,product.label.layout.user,model_product_label_layout,base.group_user,1,1,1,1
access_product_tag_manager,product.tag.manager,model_product_tag,base.group_system,1,1,1,1

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
        <field name="domain_force"> [('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="product_pricelist_comp_rule">
        <field name="name">product pricelist company rule</field>
        <field name="model_id" ref="model_product_pricelist"/>
        <field name="domain_force"> [('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="product_pricelist_item_comp_rule">
        <field name="name">product pricelist item company rule</field>
        <field name="model_id" ref="model_product_pricelist_item"/>
        <field name="domain_force"> [('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="product_supplierinfo_comp_rule">
        <field name="name">product supplierinfo company rule</field>
        <field name="model_id" ref="model_product_supplierinfo"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="product_packaging_comp_rule">
        <field name="name">product packaging company rule</field>
        <field name="model_id" ref="model_product_packaging"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient><path id="d" d="M48.466 42.033c0 .837-.277 1.548-.831 2.133L36.606 55.823c-.584.585-1.265.877-2.044.877-.793 0-1.467-.292-2.021-.877l-16.06-16.965c-.569-.585-1.051-1.382-1.448-2.393s-.595-1.935-.595-2.773v-9.857c0-.821.284-1.532.853-2.132.569-.6 1.243-.9 2.021-.9h9.344c.794 0 1.67.209 2.628.627.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156zM24.5 28.385c0-.838-.28-1.552-.842-2.145-.562-.592-1.24-.888-2.033-.888-.794 0-1.471.296-2.033.888-.561.593-.842 1.307-.842 2.145 0 .837.28 1.552.842 2.144.562.592 1.24.889 2.033.889.794 0 1.471-.297 2.033-.889.561-.592.842-1.307.842-2.144zm32.59 13.648c0 .837-.276 1.548-.83 2.133L45.23 55.823c-.584.585-1.265.877-2.044.877-.539 0-.98-.11-1.325-.332-.344-.22-.74-.576-1.19-1.066l10.557-11.136c.554-.585.83-1.296.83-2.133 0-.821-.276-1.54-.83-2.156l-16.06-16.919c-.57-.6-1.333-1.11-2.291-1.528-.958-.418-1.834-.628-2.628-.628h5.031c.794 0 1.67.21 2.628.628.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156z"/><path id="e" d="M48.466 40.033c0 .837-.277 1.548-.831 2.133L36.606 53.823c-.584.585-1.265.877-2.044.877-.793 0-1.467-.292-2.021-.877l-16.06-16.965c-.569-.585-1.051-1.382-1.448-2.393s-.595-1.935-.595-2.773v-9.857c0-.821.284-1.532.853-2.132.569-.6 1.243-.9 2.021-.9h9.344c.794 0 1.67.209 2.628.627.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156zM24.5 26.385c0-.838-.28-1.552-.842-2.145-.562-.592-1.24-.888-2.033-.888-.794 0-1.471.296-2.033.888-.561.593-.842 1.307-.842 2.145 0 .837.28 1.552.842 2.144.562.592 1.24.889 2.033.889.794 0 1.471-.297 2.033-.889.561-.592.842-1.307.842-2.144zm32.59 13.648c0 .837-.276 1.548-.83 2.133L45.23 53.823c-.584.585-1.265.877-2.044.877-.539 0-.98-.11-1.325-.332-.344-.22-.74-.576-1.19-1.066l10.557-11.136c.554-.585.83-1.296.83-2.133 0-.821-.276-1.54-.83-2.156l-16.06-16.919c-.57-.6-1.333-1.11-2.291-1.528-.958-.418-1.834-.628-2.628-.628h5.031c.794 0 1.67.21 2.628.628.959.419 1.722.928 2.291 1.528l16.06 16.919c.554.616.83 1.335.83 2.156z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L16.402 19h19.682L56.79 41 39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\product_pricelist_report.js

```javascript
odoo.define('product.generate_pricelist', function (require) {
'use strict';

var AbstractAction = require('web.AbstractAction');
var core = require('web.core');
var FieldMany2One = require('web.relational_fields').FieldMany2One;
var StandaloneFieldManagerMixin = require('web.StandaloneFieldManagerMixin');
var Widget = require('web.Widget');

var QWeb = core.qweb;
var _t = core._t;

var QtyTagWidget = Widget.extend({
    template: 'product.report_pricelist_qty',
    events: {
        'click .o_remove_qty': '_onClickRemoveQty',
    },
    /**
     * @override
     */
    init: function (parent, defaulQuantities) {
        this._super.apply(this, arguments);
        this.quantities = defaulQuantities;
        this.MAX_QTY = 5;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Add a quantity when add(+) button clicked.
     *
     * @private
     */
    _onClickAddQty: function () {
        if (this.quantities.length >= this.MAX_QTY) {
            this.displayNotification({ message: _.str.sprintf(
                _t("At most %d quantities can be displayed simultaneously. Remove a selected quantity to add others."),
                this.MAX_QTY
            ) });
            return;
        }
        const qty = parseInt(this.$('.o_product_qty').val());
        if (qty && qty > 0) {
            // Check qty already exist
            if (this.quantities.indexOf(qty) === -1) {
                this.quantities.push(qty);
                this.quantities = this.quantities.sort((a, b) => a - b);
                this.trigger_up('qty_changed', {quantities: this.quantities});
                this.renderElement();
            } else {
                this.displayNotification({
                    message: _.str.sprintf(_t("Quantity already present (%d)."), qty),
                    type: 'info'
                });
            }
        } else {
            this.displayNotification({ message: _t("Please enter a positive whole number") });
        }
    },
    /**
     * Remove quantity.
     *
     * @private
     * @param {jQueryEvent} ev
     */
    _onClickRemoveQty: function (ev) {
        const qty = parseInt($(ev.currentTarget).closest('.badge').data('qty'));
        this.quantities = this.quantities.filter(q => q !== qty);
        this.trigger_up('qty_changed', {quantities: this.quantities});
        this.renderElement();
    },
});

var GeneratePriceList = AbstractAction.extend(StandaloneFieldManagerMixin, {
    hasControlPanel: true,
    events: {
        'click .o_action': '_onClickAction',
        'submit form': '_onSubmitForm',
    },
    custom_events: Object.assign({}, StandaloneFieldManagerMixin.custom_events, {
        field_changed: '_onFieldChanged',
        qty_changed: '_onQtyChanged',
    }),
    /**
     * @override
     */
    init: function (parent, params) {
        this._super.apply(this, arguments);
        StandaloneFieldManagerMixin.init.call(this);
        this.context = params.context;
        // in case the window got refreshed
        if (params.params && params.params.active_ids && typeof(params.params.active_ids === 'string')) {
            try {
                this.context.active_ids = params.params.active_ids.split(',').map(id => parseInt(id));
                this.context.active_model = params.params.active_model;
            } catch(_e) {
                console.log('unable to load ids from the url fragment 🙁');
            }
        }
        if (!this.context.active_model) {
            // started without an active module, assume product templates
            this.context.active_model = 'product.template';
        }
        this.context.quantities = [1, 5, 10];
    },
    /**
     * @override
     */
    willStart: function () {
        let getPricelist;
        // started without a selected pricelist in context? just get the first one
        if (this.context.default_pricelist) {
            getPricelist = Promise.resolve([this.context.default_pricelist]);
        } else {
            getPricelist = this._rpc({
                model: 'product.pricelist',
                method: 'search',
                args: [[]],
                kwargs: {limit: 1}
            });
        }
        const fieldSetup = getPricelist.then(pricelistIds => {
            return this.model.makeRecord('report.product.report_pricelist', [{
                name: 'pricelist_id',
                type: 'many2one',
                relation: 'product.pricelist',
                value: pricelistIds[0],
            }]);
        }).then(recordID => {
            const record = this.model.get(recordID);
            this.many2one = new FieldMany2One(this, 'pricelist_id', record, {
                mode: 'edit',
                attrs: {
                    can_create: false,
                    can_write: false,
                    options: {no_open: true},
                },
            });
            this._registerWidget(recordID, 'pricelist_id', this.many2one);
        });
        return Promise.all([fieldSetup, this._getHtml(), this._super()]);
    },
    /**
     * @override
     */
    start: function () {
        this.controlPanelProps.cp_content = this._renderComponent();
        const $content = this.controlPanelProps.cp_content;
        $content["$searchview"][0].querySelector('.o_is_visible_title').addEventListener('click', this._onClickVisibleTitle.bind(this));
        return this._super.apply(this, arguments).then(() => {
            this.$('.o_content').html(this.reportHtml);
        });
    },
    /**
     * Include the current model (template/variant) in the state to allow refreshing without losing
     * the proper context.
     * @override
     */
    getState: function () {
        return {
            active_model: this.context.active_model,
        };
    },
    getTitle: function () {
        return _t('Pricelist Report');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Returns the expected data for the report rendering call (html or pdf)
     *
     * @private
     * @returns {Object}
     */
    _prepareActionReportParams: function () {
        return {
            active_model: this.context.active_model,
            active_ids: this.context.active_ids || '',
            is_visible_title: this.context.is_visible_title || '',
            pricelist_id: this.context.pricelist_id || '',
            quantities: this.context.quantities || [1],
        };
    },
    /**
     * Get template to display report.
     *
     * @private
     * @returns {Promise}
     */
    _getHtml: function () {
        return this._rpc({
            model: 'report.product.report_pricelist',
            method: 'get_html',
            kwargs: {
                data: this._prepareActionReportParams(),
                context: this.context,
            },
        }).then(result => {
            this.reportHtml = result;
        });
    },
    /**
     * Reload report.
     *
     * @private
     * @returns {Promise}
     */
    _reload: function () {
        return this._getHtml().then(() => {
            this.$('.o_content').html(this.reportHtml);
        });
    },
    /**
     * Render search view and print button.
     *
     * @private
     */
    _renderComponent: function () {
        const $buttons = $('<button>', {
            class: 'btn btn-primary',
            text: _t("Print"),
        }).on('click', this._onClickPrint.bind(this));

        const $searchview = $(QWeb.render('product.report_pricelist_search'));
        this.many2one.appendTo($searchview.find('.o_pricelist'));

        this.qtyTagWidget = new QtyTagWidget(this, this.context.quantities);
        this.qtyTagWidget.replace($searchview.find('.o_product_qty'));
        return { $buttons, $searchview };
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Checkbox is checked, the report title will show.
     *
     * @private
     * @param {Event} ev
     */
    _onClickVisibleTitle(ev) {
        this.context.is_visible_title = ev.currentTarget.checked;
        this._reload();
    },

    /**
     * Open form view of particular record when link clicked.
     *
     * @private
     * @param {jQueryEvent} ev
     */
    _onClickAction: function (ev) {
        ev.preventDefault();
        this.do_action({
            type: 'ir.actions.act_window',
            res_model: $(ev.currentTarget).data('model'),
            res_id: $(ev.currentTarget).data('res-id'),
            views: [[false, 'form']],
            target: 'self',
        });
    },
    /**
     * Print report in PDF when button clicked.
     *
     * @private
     */
    _onClickPrint: function () {
        return this.do_action({
            type: 'ir.actions.report',
            report_type: 'qweb-pdf',
            report_name: 'product.report_pricelist',
            report_file: 'product.report_pricelist',
            data: this._prepareActionReportParams(),
        });
    },
    /**
     * Reload report when pricelist changed.
     *
     * @override
     */
    _onFieldChanged: function (event) {
        this.context.pricelist_id = event.data.changes.pricelist_id.id;
        StandaloneFieldManagerMixin._onFieldChanged.apply(this, arguments);
        this._reload();
    },
    /**
     * Reload report when quantities changed.
     *
     * @private
     * @param {OdooEvent} ev
     * @param {integer[]} event.data.quantities
     */
    _onQtyChanged: function (ev) {
        this.context.quantities = ev.data.quantities;
        this._reload();
    },
    _onSubmitForm: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this.qtyTagWidget._onClickAddQty();
    },
});

core.action_registry.add('generate_pricelist', GeneratePriceList);

return {
    GeneratePriceList,
    QtyTagWidget
};

});

```

## File: static\src\xml\pricelist_report.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>

    <t t-name="product.report_pricelist_qty">
        <span>
            <div class="input-group flex-nowrap w-75">
                <input type="number" name="qty_to_add" class="o_input o_product_qty form-control text-end w-auto" value="1" min="1"/>
                <button class="btn btn-secondary o_add_qty text-end form-control" type="submit" title="Add a quantity">
                    <i class="fa fa-plus"/>
                </button>
            </div>
            <span class="o_badges">
                <t t-set="quantities" t-value="widget.quantities"/>
                <t t-call="product.report_pricelist_qty_badges"/>
            </span>
        </span>
    </t>

    <t t-name="product.report_pricelist_search">
        <form class="d-flex justify-content-around align-items-center o_pricelist_report_form">
            <div>
                <label class="fw-bold">Pricelist:</label>
                <span class="o_pricelist"/>
            </div>
            <div class="d-flex align-items-center">
                <label class="fw-bold mb-4" for="qty_to_add">Quantities:</label>
                <div class="o_product_qty"/>
            </div>
            <div class="form-check">
                <input class="form-check-input o_is_visible_title ms-2" type="checkbox"/>
                <label class="form-check-label">Display Pricelist</label>
            </div>
        </form>
    </t>

    <t t-name="product.report_pricelist_qty_badges">
        <t t-foreach="quantities" t-as="qty">
            <span class="badge rounded-pill border" t-att-data-qty="qty">
                <t t-esc="qty"/>
                <i class="fa fa-close o_remove_qty" title="Remove quantity"/>
            </span>
        </t>
    </t>

</templates>

```

## File: views\product_attribute_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="attribute_tree_view" model="ir.ui.view">
        <field name="name">product.attribute.tree</field>
        <field name="model">product.attribute</field>
        <field name="arch" type="xml">
            <tree string="Variant Values" default_order="sequence, id">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="display_type"/>
                <field name="create_variant"/>
            </tree>
        </field>
    </record>

    <record id="product_attribute_view_form" model="ir.ui.view">
        <field name="name">product.attribute.form</field>
        <field name="model">product.attribute</field>
        <field name="arch" type="xml">
            <form string="Product Attribute">
            <field name="number_related_products" invisible="1"/>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button class="oe_stat_button" name="action_open_related_products"
                            type="object" icon="fa-bars"
                            attrs="{'invisible': [('number_related_products', '=', [])]}">
                        <div class="o_stat_info">
                            <span class="o_stat_value"><field name="number_related_products"/></span>
                            <span class="o_stat_text">Related Products</span>
                        </div>
                    </button>
                </div>
                <group name="main_fields" class="o_label_nowrap">
                    <label for="name" string="Attribute Name"/>
                    <field name="name" nolabel="1"/>
                    <field name="display_type" widget="radio"/>
                    <field name="create_variant" widget="radio" attrs="{'readonly': [('number_related_products', '!=', 0)]}"/>
                </group>
                <notebook>
                    <page string="Attribute Values" name="attribute_values">
                        <field name="value_ids" widget="one2many" nolabel="1">
                            <tree string="Values" editable="bottom">
                                <field name="sequence" widget="handle"/>
                                <field name="name"/>
                                <field name="display_type" invisible="1"/>
                                <field name="is_custom" groups="product.group_product_variant"/>
                                <field name="html_color" attrs="{'column_invisible': [('parent.display_type', '!=', 'color')]}" widget="color"/>
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
                            <field name="html_color"/>
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
                <field name="attribute_id" optional="hide"/>
                <field name="name"/>
                <field name="display_type" optional="hide"/>
                <field name="html_color" attrs="{'invisible': [('display_type', '!=', 'color')]}" widget="color"/>
                <field name="ptav_active" optional="hide"/>
                <field name="price_extra" widget="monetary" options="{'field_digits': True}"/>
                <field name="currency_id" invisible="1"/>
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
                        <field name="display_type" invisible="1"/>
                        <field name="html_color" attrs="{'invisible': [('display_type', '!=', 'color')]}"/>
                        <field name="price_extra" widget="monetary" options="{'field_digits': True}"/>
                        <field name="currency_id" invisible="1"/>
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

</odoo>

```

## File: views\product_category_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

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
                        <label for="name" string="Category"/>
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

    <record id="product_category_action_form" model="ir.actions.act_window">
        <field name="name">Product Categories</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.category</field>
        <field name="search_view_id" ref="product_category_search_view"/>
        <field name="view_id" ref="product_category_list_view"/>
    </record>

</odoo>

```

## File: views\product_packaging_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="product_packaging_tree_view" model="ir.ui.view">
        <field name="name">product.packaging.tree.view</field>
        <field name="model">product.packaging</field>
        <field name="arch" type="xml">
            <tree string="Product Packagings" name="packaging">
                <field name="sequence" widget="handle"/>
                <field name="product_id"/>
                <field name="name" string="Packaging"/>
                <field name="qty"/>
                <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"/>
                <field name="barcode" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                <field name="company_id" groups="!base.group_multi_company" invisible="1"/>
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
            <xpath expr="//tree[@name='packaging']" position="attributes">
                <attribute name="editable">bottom</attribute>
            </xpath>
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
                        <field name="company_id" invisible="1"/>
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
                            <field name="company_id" groups="base.group_multi_company"/>
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

</odoo>

```

## File: views\product_pricelist_item_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

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
                      context="{'group_by': 'product_id'}"
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
            <tree string="Pricelist Rules" editable="bottom">
                <!-- Scope = coming from a product/product template -->
                <field name="pricelist_id" string="Pricelist" options="{'no_create_edit':1, 'no_open': 1}"/>
                <field name="name" string="Applied On"/>
                <field name="company_id" invisible="1"/>
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
            <form string="Pricelist Rule">
                <sheet>
                    <field name="name" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <group name="pricelist_rule_computation" groups="product.group_sale_pricelist" string="Price Computation">
                        <group name="pricelist_rule_method">
                            <field name="compute_price" string="Computation" widget="radio"/>
                        </group>
                        <div class="alert alert-info" role="alert" groups="uom.group_uom">
                            The computed price is expressed in the default Unit of Measure of the product.
                        </div>
                    </group>
                    <group name="pricelist_rule_base" groups="product.group_sale_pricelist">
                        <group>
                            <field name="price" invisible="1"/>
                            <field name="fixed_price" widget="monetary"
                                attrs="{'invisible': [('compute_price', '!=', 'fixed')]}"
                                options="{'field_digits': True}"/>
                            <label for="percent_price" string="Discount" attrs="{'invisible':[('compute_price', '!=', 'percentage')]}"/>
                            <div class="o_row" attrs="{'invisible':[('compute_price', '!=', 'percentage')]}">
                                <field name="percent_price" class="oe_inline" attrs="{'invisible':[('compute_price', '!=', 'percentage')]}"/>%
                            </div>
                            <field name="base" attrs="{'invisible':[('compute_price', '!=', 'formula')]}"/>
                            <field name="base_pricelist_id" attrs="{
                                'invisible': ['|', ('compute_price', '!=', 'formula'), ('base', '!=', 'pricelist')],
                                'required': [('compute_price', '=', 'formula'), ('base', '=', 'pricelist')],
                                'readonly': [('base', '!=', 'pricelist')]}"/>
                            <label for="price_discount" string="Discount" attrs="{'invisible':[('compute_price', '!=', 'formula')]}"/>
                            <div class="o_row" attrs="{'invisible':[('compute_price', '!=', 'formula')]}">
                                <field name="price_discount"/>
                                <span>%</span>
                            </div>
                            <field name="price_surcharge"
                                widget="monetary"
                                string="Extra Fee"
                                attrs="{'invisible':[('compute_price', '!=', 'formula')]}"
                                options="{'field_digits': True}"/>
                            <field name="price_round" string="Rounding Method" attrs="{'invisible':[('compute_price', '!=', 'formula')]}"/>
                            <label string="Margins" for="price_min_margin" attrs="{'invisible':[('compute_price', '!=', 'formula')]}"/>
                            <div class="o_row" attrs="{'invisible':[('compute_price', '!=', 'formula')]}">
                                <field name="price_min_margin" string="Min. Margin" class="oe_inline"
                                    widget="monetary"
                                    nolabel="1"
                                    options="{'field_digits': True}"/>
                                <i class="fa fa-long-arrow-right mx-2 oe_edit_only" aria-label="Arrow icon" title="Arrow"/>
                                <field name="price_max_margin" string="Max. Margin" class="oe_inline"
                                    widget="monetary"
                                    nolabel="1"
                                    options="{'field_digits': True}"/>
                            </div>
                        </group>
                        <div class="alert alert-info" role="alert" style="white-space: pre;" attrs="{'invisible': [('compute_price', '!=', 'formula')]}">
                            <field name="rule_tip"/>
                        </div>
                    </group>

                    <group string="Conditions">
                        <group name="pricelist_rule_target">
                            <field name="applied_on" widget="radio"/>
                            <field name="categ_id" options="{'no_create':1}" attrs="{
                                'invisible':[('applied_on', '!=', '2_product_category')],
                                'required':[('applied_on', '=', '2_product_category')]}"/>
                            <field name="product_tmpl_id" options="{'no_create':1}" attrs="{
                                'invisible':[('applied_on', '!=', '1_product')],
                                'required':[('applied_on', '=', '1_product')]}"/>
                            <field name="product_id" options="{'no_create':1}" attrs="{
                                'invisible':[('applied_on', '!=', '0_product_variant')],
                                'required':[('applied_on', '=', '0_product_variant')]}"/>
                        </group>
                        <group name="pricelist_rule_limits">
                            <field name="min_quantity"/>
                            <label for="date_start" string="Validity"/>
                            <div class="o_row">
                                <field name="date_start" widget="daterange" options='{"related_end_date": "date_end"}'/>
                                <i class="fa fa-long-arrow-right mx-2 oe_edit_only" aria-label="Arrow icon" title="Arrow"/>
                                <field name="date_end" widget="daterange" options='{"related_start_date": "date_start"}'/>
                            </div>
                        </group>
                        <group name="pricelist_rule_related" groups="base.group_no_one">
                            <field name="pricelist_id" invisible="1"/>
                            <field name="currency_id" groups="base.group_multi_currency"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

</odoo>

```

## File: views\product_pricelist_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

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
            <tree string="Products Price List" sample="1">
                <field name="sequence" widget="handle"/>
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
            <kanban class="o_kanban_mobile" sample="1">
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div id="product_pricelist" class="o_kanban_record_top mb0">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title">
                                        <span><field name="name"/></span>
                                    </strong>
                                </div>
                                <strong>
                                    <i class="fa fa-money" role="img" aria-label="Currency" title="Currency"></i> <field name="currency_id"/>
                                </strong>
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
                                <tree groups="!product.group_sale_pricelist" string="Pricelist Rules" editable="bottom">
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
                                <!-- When in advanced pricelist mode : pricelist rules
                                    Should open in a form view and not be editable inline anymore.
                                -->
                                <tree groups="product.group_sale_pricelist" string="Pricelist Rules">
                                    <field name="product_tmpl_id" invisible="1"/>
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

</odoo>

```

## File: views\product_supplierinfo_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="product_supplierinfo_form_view" model="ir.ui.view">
        <field name="name">product.supplierinfo.form.view</field>
        <field name="model">product.supplierinfo</field>
        <field name="arch" type="xml">
            <form string="Vendor Information">
                <sheet>
                    <group>
                        <group name="vendor" string="Vendor">
                            <field name="product_variant_count" invisible="1"/>
                            <field name="partner_id" context="{'res_partner_search_mode': 'supplier'}"/>
                            <field name="product_name"/>
                            <field name="product_code"/>
                            <label for="delay"/>
                            <div>
                                <field name="delay" class="oe_inline"/> days
                            </div>
                        </group>
                        <group string="Pricelist">
                            <field name="product_tmpl_id" string="Product" invisible="context.get('visible_product_tmpl_id', True)"/>
                            <field name="product_id" groups="product.group_product_variant" options="{'no_create': True}"/>
                            <label for="min_qty"/>
                            <div class="o_row">
                                <field name="min_qty"/>
                                <field name="product_uom" groups="uom.group_uom"/>
                            </div>
                            <label for="price" string="Unit Price"/>
                            <div class="o_row">
                                <field name="price" class="oe_inline" /><field name="currency_id" groups="base.group_multi_currency"/>
                            </div>
                            <label for="date_start" string="Validity"/>
                            <div class="o_row"><field name="date_start" class="oe_inline"/> to <field name="date_end" class="oe_inline"/></div>
                            <field name="company_id" options="{'no_create': True}"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="product_supplierinfo_search_view" model="ir.ui.view">
        <field name="name">product.supplierinfo.search.view</field>
        <field name="model">product.supplierinfo</field>
        <field name="arch" type="xml">
            <search string="Vendor">
                <field name="partner_id"/>
                <field name="product_tmpl_id"/>
                <filter string="Active Products" name="active_products" domain="['|', ('product_tmpl_id.active', '=', True),('product_id.active', '=', True)]"/>
                <separator />
                <filter string="Active" name="active" domain="['|', ('date_end', '=', False), ('date_end', '&gt;=',  (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d'))]"/>
                <filter string="Archived" name="archived" domain="[('date_end', '&lt;',  (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Product" name="groupby_product" domain="[]" context="{'group_by': 'product_tmpl_id'}"/>
                    <filter string="Vendor" name="groupby_vendor" domain="[]" context="{'group_by': 'partner_id'}"/>
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
                <field name="partner_id"/>
                <field name="currency_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click">
                            <div class="row mb4">
                                <strong class="col-6">
                                    <span t-esc="record.partner_id.value"/>
                                </strong>
                                <strong class="col-6 text-end">
                                    <strong><field name="price" widget="monetary"/></strong>
                                </strong>
                                <div class="col-6">
                                    <span t-esc="record.min_qty.value"/>
                                </div>
                                <div class="col-6 text-end">
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
                <field name="partner_id" readonly="1"/>
                <field name="product_id" readonly="1" optional="hide"
                    invisible="context.get('product_template_invisible_variant', False)"
                    groups="product.group_product_variant"/>
                <field name="product_tmpl_id" string="Product" readonly="1"
                    invisible="context.get('visible_product_tmpl_id', True)"/>
                <field name="product_name" optional="hide"/>
                <field name="product_code" optional="hide"/>
                <field name="date_start" optional="hide"/>
                <field name="date_end" optional="hide"/>
                <field name="company_id" readonly="1" groups="base.group_multi_company"/>
                <field name="min_qty" optional="hide"/>
                <field name="product_uom" groups="uom.group_uom" optional="hide"/>
                <field name="price" string="Price"/>
                <field name="currency_id" groups="base.group_multi_currency"/>
                <field name="delay" optional="hide"/>
            </tree>
        </field>
    </record>

    <record id="product_supplierinfo_type_action" model="ir.actions.act_window">
        <field name="name">Vendor Pricelists</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.supplierinfo</field>
        <field name="view_mode">tree,form,kanban</field>
        <field name="context">{'visible_product_tmpl_id': False, 'search_default_active_products': True}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No vendor pricelist found
            </p><p>
                Register the prices requested by your vendors for each product, based on the quantity and the period.
            </p>
        </field>
    </record>

</odoo>

```

## File: views\product_tag_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Product Tags -->
    <record id="product_tag_form_view" model="ir.ui.view">
        <field name="name">product.tag.form</field>
        <field name="model">product.tag</field>
        <field name="arch" type="xml">
            <form string="Product Tag">
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="color" widget="color_picker"/>
                        </group>
                    </group>
                    <group>
                        <field name="product_ids" widget="many2many_tags"
                               attrs="{'invisible':[('product_ids','=',[])]}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="product_tag_tree_view" model="ir.ui.view">
        <field name="name">product.tag.tree</field>
        <field name="model">product.tag</field>
        <field name="arch" type="xml">
            <tree string="Product Tags">
                <field name="name"/>
                <field name="color" widget="color_picker"/>
            </tree>
        </field>
    </record>
    <record id="product_tag_action" model="ir.actions.act_window">
        <field name="name">Product Tags</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.tag</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" eval="False"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Define a new tag
          </p><p>
            Tags are used to search product for a given theme.
          </p>
        </field>
    </record>
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
            <tree string="Product" multi_edit="1" sample="1">
            <header>
                <button string="Print Labels" type="object" name="action_open_label_layout"/>
            </header>
                <field name="product_variant_count" invisible="1"/>
                <field name="sale_ok" invisible="1"/>
                <field name="currency_id" invisible="1"/>
                <field name="cost_currency_id" invisible="1"/>
                <field name="priority" widget="priority" optional="show" nolabel="1"/>
                <field name="name" string="Product Name"/>
                <field name="default_code" optional="show"/>
                <field name="product_tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                <field name="barcode" optional="hide" attrs="{'readonly': [('product_variant_count', '>', 1)]}"/>
                <field name="company_id" options="{'no_create': True}"
                    groups="base.group_multi_company" optional="hide"/>
                <field name="list_price" string="Sales Price" widget='monetary' options="{'currency_field': 'currency_id'}" optional="show" decoration-muted="not sale_ok"/>
                <field name="standard_price" widget='monetary' options="{'currency_field': 'cost_currency_id'}" optional="show" readonly="1"/>
                <field name="categ_id" optional="hide"/>
                <field name="detailed_type" optional="hide" readonly="1"/>
                <field name="type" invisible="1"/>
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
                <page name="variants" string="Attributes &amp; Variants" groups="product.group_product_variant">
                    <field name="attribute_line_ids" widget="one2many" context="{'show_attribute': False}">
                        <tree string="Variants" editable="bottom" decoration-info="value_count &lt;= 1">
                            <field name="value_count" invisible="1"/>
                            <field name="attribute_id" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="value_ids" widget="many2many_tags" options="{'no_create_edit': True, 'color_field': 'color'}" context="{'default_attribute_id': attribute_id, 'show_attribute': False}"/>
                            <button string="Configure" class="float-end btn-secondary"
                                type="object" name="action_open_attribute_values"
                                groups="product.group_product_variant"/>
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
            <kanban sample="1" class="o_kanban_product_template">
                <field name="id"/>
                <field name="product_variant_count"/>
                <field name="currency_id"/>
                <field name="activity_state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_card oe_kanban_global_click">
                            <div class="o_kanban_image me-1">
                                <img t-att-src="kanban_image('product.template', 'image_128', record.id.raw_value)" alt="Product" class="o_image_64_contain"/>
                            </div>
                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top mb-0">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <field name="name"/>
                                        </strong>
                                    </div>
                                    <field name="priority" widget="priority"/>
                                </div>
                                <t t-if="record.default_code.value">[<field name="default_code"/>]</t>
                                <div t-if="record.product_variant_count.value &gt; 1" groups="product.group_product_variant">
                                    <strong>
                                        <t t-esc="record.product_variant_count.value"/> Variants
                                    </strong>
                                </div>
                                <div name="product_lst_price" class="mt-1">
                                    Price: <field name="list_price" widget="monetary" options="{'currency_field': 'currency_id', 'field_digits': True}"></field>
                                </div>
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

    <record id="action_product_template_price_list_report" model="ir.actions.server">
        <field name="name">Generate Pricelist Report</field>
        <field name="groups_id" eval="[(4, ref('product.group_product_pricelist'))]"/>
        <field name="model_id" ref="product.model_product_template"/>
        <field name="binding_model_id" ref="product.model_product_template"/>
        <field name="state">code</field>
        <field name="code">
ctx = env.context
ctx.update({'default_pricelist': env['product.pricelist'].search([], limit=1).id})
action = {
    'name': 'Pricelist Report',
    'type': 'ir.actions.client',
    'tag': 'generate_pricelist',
    'context': ctx,
}
        </field>
    </record>

</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- base structure of product.template, common with product.product -->
    <record id="product_template_form_view" model="ir.ui.view">
        <field name="name">product.template.common.form</field>
        <field name="model">product.template</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <form string="Product">
                <header>
                    <button string="Print Labels" type="object" name="action_open_label_layout" attrs="{'invisible': [('detailed_type', '==', 'service')]}"/>
                </header>
                <sheet name="product_form">
                    <field name='product_variant_count' invisible='1'/>
                    <field name='is_product_variant' invisible='1'/>
                    <field name='attribute_line_ids' invisible='1'/>
                    <field name="type" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button"
                               name="open_pricelist_rules"
                               icon="fa-list-ul"
                               groups="product.group_product_pricelist"
                               type="object">
                               <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value">
                                        <field name="pricelist_item_count"/>
                                    </span>
                                    <span attrs="{'invisible': [('pricelist_item_count', '=', 1)]}">
                                        Extra Prices
                                    </span>
                                    <span attrs="{'invisible': [('pricelist_item_count', '!=', 1)]}">
                                        Extra Price
                                    </span>
                               </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="id" invisible="True"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="name" string="Product Name"/>
                        <h1>
                            <div class="d-flex">
                                <field name="priority" widget="priority" class="me-3"/>
                                <field class="text-break" name="name" placeholder="e.g. Cheese Burger"/>
                            </div>
                        </h1>
                    </div>
                    <div name="options">
                        <span class="d-inline-block">
                            <field name="sale_ok"/>
                            <label for="sale_ok"/>
                        </span>
                        <span class="d-inline-block">
                            <field name="purchase_ok"/>
                            <label for="purchase_ok"/>
                        </span>
                    </div>
                    <notebook>
                        <page string="General Information" name="general_information">
                            <group>
                                <group name="group_general">
                                    <field name="active" invisible="1"/>
                                    <field name="detailed_type"/>
                                    <field name="product_tooltip" string="" class="fst-italic text-muted"/>
                                    <field name="uom_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                    <field name="uom_po_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                </group>
                                <group name="group_standard_price">
                                    <label for="list_price"/>
                                    <div name="pricing">
                                      <field name="list_price" class="oe_inline" widget='monetary'
                                        options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                                    </div>
                                    <label for="standard_price" attrs="{'invisible': [('product_variant_count', '&gt;', 1), ('is_product_variant', '=', False)]}"/>
                                    <div name="standard_price_uom" attrs="{'invisible': [('product_variant_count', '&gt;', 1), ('is_product_variant', '=', False)]}">
                                        <field name="standard_price" class="oe_inline" widget='monetary' options="{'currency_field': 'cost_currency_id', 'field_digits': True}"/>
                                        <span groups="uom.group_uom" >per
                                            <field name="uom_name" class="oe_inline"/>
                                        </span>
                                    </div>
                                    <field name="categ_id" string="Product Category"/>
                                    <field name="product_tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                    <field name="company_id" groups="base.group_multi_company"
                                        options="{'no_create': True}"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="cost_currency_id" invisible="1"/>
                                    <field name="product_variant_id" invisible="1"/>
                                </group>
                            </group>
                            <group string="Internal Notes">
                                <field colspan="2" name="description" nolabel="1" placeholder="This note is only for internal purposes."/>
                            </group>
                        </page>
                        <page string="Sales" attrs="{'invisible':[('sale_ok','=',False)]}" name="sales" invisible="1">
                            <group name="sale">
                                <group string="Upsell &amp; Cross-Sell" name="upsell" invisible="1"/>
                            </group>
                            <group>
                                <group string="Sales Description" name="description">
                                    <field colspan="2" name="description_sale" nolabel="1" placeholder="This note is added to sales orders and invoices."/>
                                </group>
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
                                        <field name="weight" class="oe_inline"/>
                                        <field name="weight_uom_name"/>
                                    </div>
                                    <label for="volume" attrs="{'invisible':[('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}"/>
                                    <div class="o_row" name="volume" attrs="{'invisible':[('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}">
                                        <field name="volume" string="Volume" class="oe_inline"/>
                                        <field name="volume_uom_name"/>
                                    </div>
                                </group>
                            </group>
                            <group name="packaging" string="Packaging"
                                colspan="4"
                                attrs="{'invisible':['|', ('type', 'not in', ['product', 'consu']), ('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}"
                                groups="product.group_stock_packaging">
                                <field colspan="2" name="packaging_ids" nolabel="1" context="{'tree_view_ref':'product.product_packaging_tree_view2', 'default_company_id': company_id}"/>
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
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))
                    ]"/>
                <separator/>
                <filter string="Favorites" name="favorites" domain="[('priority','=','1')]"/>
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
                    <header>
                        <button string="Print Labels" type="object" name="action_open_label_layout"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box"/>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="active" invisible="1"/>
                        <field name="id" invisible="1"/>
                        <field name="company_id" invisible="1"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                        <div class="oe_title">
                            <label for="name" string="Product Name"/>
                            <h1><field name="name" readonly="1" placeholder="e.g. Odoo Enterprise Subscription"/></h1>
                            <field name="product_template_attribute_value_ids" widget="many2many_tags" readonly="1"/>
                            <p>
                                <span>All general settings about this product are managed on</span>
                                <button name="open_product_template" type="object" string="the product template." class="oe_link oe_link_product ps-0 ms-1 mb-1"/>
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
                                <label for="lst_price" string="Sales Price"/>
                                <div class="o_row">
                                    <field name="lst_price" class="oe_inline" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}" attrs="{'readonly': [('product_variant_count', '&gt;', 1)]}"/>
                                </div>
                                <label for="standard_price"/>
                                <div class="o_row">
                                    <field name="standard_price" widget='monetary' class="oe_inline" options="{'currency_field': 'cost_currency_id'}"/>
                                </div>
                                <field name="currency_id" invisible='1'/>
                                <field name="cost_currency_id" invisible="1"/>
                            </group>
                        </group>
                        <group>
                            <group name="weight" string="Logistics" attrs="{'invisible':[('type', 'not in', ['product', 'consu'])]}">
                                <label for="volume"/>
                                <div class="o_row">
                                    <field name="volume" class="oe_inline"/>
                                    <span><field name="volume_uom_name"/></span>
                                </div>
                                <label for="weight"/>
                                <div class="o_row">
                                    <field name="weight" class="oe_inline"/>
                                    <span><field name="weight_uom_name"/></span>
                                </div>
                            </group>
                            <group name="tags" string="Tags">
                                <field name="product_tag_ids" string="Product Template Tags" widget="many2many_tags" readonly="1" options="{'no_open': True, 'color_field': 'color'}"/>
                                <field name="additional_product_tag_ids" widget="many2many_tags" options="{'no_open': True, 'color_field': 'color'}"/>
                            </group>
                        </group>
                        <group>
                            <group name="packaging" string="Packaging" groups="product.group_stock_packaging">
                                <field colspan="2" name="packaging_ids" nolabel="1"
                                    context="{'tree_view_ref':'product.product_packaging_tree_view2', 'default_company_id': company_id}"/>
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
                <tree string="Product Variants" multi_edit="1" duplicate="false" sample="1">
                <header>
                    <button string="Print Labels" type="object" name="action_open_label_layout"/>
                </header>
                    <field name="priority" widget="priority" nolabel="1" readonly="1"/>
                    <field name="default_code" optional="show" readonly="1"/>
                    <field name="barcode" optional="hide" readonly="1"/>
                    <field name="name" readonly="1"/>
                    <field name="product_template_variant_value_ids" widget="many2many_tags" groups="product.group_product_variant" readonly="1"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide" readonly="1"/>
                    <field name="lst_price" optional="show" string="Sales Price"/>
                    <field name="standard_price" optional="show"/>
                    <field name="categ_id" optional="hide"/>
                    <field name="product_tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_edit_color': 1}" optional="hide"/>
                    <field name="type" optional="hide" readonly="1"/>
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
                <xpath expr="//div[@name='standard_price_uom']" position="after">
                    <field name="default_code"/>
                    <field name="barcode"/>
                </xpath>
                <xpath expr="//field[@name='priority']" position="attributes">
                    <attribute name="readonly">1</attribute>
                </xpath>
                <field name="list_price" position="attributes">
                   <attribute name="attrs">{'readonly': [('product_variant_count', '&gt;', 1)]}</attribute>
                   <attribute name="invisible">1</attribute>
                </field>
                <xpath expr="//label[@for='list_price']" position="replace">
                    <label for="lst_price"/>
                </xpath>
                <field name="list_price" position="after">
                   <field name="lst_price" class="oe_inline" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                </field>
                <group name="packaging" position="attributes">
                    <attribute name="attrs">{'invisible': 0}</attribute>
                </group>
                <field name="name" position="after">
                    <field name="product_tmpl_id" class="oe_inline" readonly="1" invisible="1" attrs="{'required': [('id', '!=', False)]}"/>
                </field>
                <xpath expr="//div[hasclass('oe_title')]" position="inside">
                    <field name="product_template_variant_value_ids" widget="many2many_tags" readonly="1" groups="product.group_product_variant"/>
                </xpath>
                <field name="product_tag_ids" position="attributes">
                    <attribute name="options">{'no_open': True, 'color_field': 'color'}</attribute>
                </field>
                <field name="product_tag_ids" position="after">
                    <field name="additional_product_tag_ids" widget="many2many_tags" options="{'no_open': True, 'color_field': 'color'}"/>
                </field>
            </field>
        </record>

        <record id="product_kanban_view" model="ir.ui.view">
            <field name="name">Product Kanban</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <kanban sample="1">
                    <field name="id"/>
                    <field name="lst_price"/>
                    <field name="activity_state"/>
                    <field name="color"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div class="o_kanban_image">
                                    <img t-att-src="kanban_image('product.product', 'image_128', record.id.raw_value)" alt="Product" class="o_image_64_contain"/>
                                </div>
                                <div class="oe_kanban_details">
                                    <field name="priority" widget="priority" readonly="1"/>
                                    <strong class="o_kanban_record_title">
                                        <field name="name"/>
                                        <small t-if="record.default_code.value">[<field name="default_code"/>]</small>
                                    </strong>
                                    <div class="o_kanban_tags_section">
                                        <field name="product_template_variant_value_ids" groups="product.group_product_variant" widget="many2many_tags" options="{'color_field': 'color'}"/>
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

    <record id="action_product_price_list_report" model="ir.actions.server">
        <field name="name">Generate Pricelist</field>
        <field name="groups_id" eval="[(4, ref('group_product_pricelist'))]"/>
        <field name="model_id" ref="product.model_product_product"/>
        <field name="binding_model_id" ref="product.model_product_product"/>
        <field name="state">code</field>
        <field name="code">
ctx = env.context
ctx.update({'default_pricelist': env['product.pricelist'].search([], limit=1).id})
action = {
    'name': 'Pricelist Report',
    'type': 'ir.actions.client',
    'tag': 'generate_pricelist',
    'context': ctx,
}
        </field>
    </record>

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
                <xpath expr="//div[@id='companies']" position="after">
                    <h2>Units of Measure</h2>
                    <div class="row mt16 o_settings_container" id="product_general_settings">
                        <div class="col-12 col-lg-6 o_setting_box" id="weight_uom_setting">
                            <div class="o_setting_left_pane">
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="product_weight_in_lbs" string="Weight"/>
                                <div class="text-muted">
                                    Define your weight unit of measure
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="product_weight_in_lbs" class="o_light_label" widget="radio" options="{'horizontal': true}"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="manage_volume_uom_setting">
                            <div class="o_setting_right_pane">
                                <label for="product_volume_volume_in_cubic_feet" string="Volume"/>
                                <div class="text-muted">
                                    Define your volume unit of measure
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="product_volume_volume_in_cubic_feet" class="o_light_label" widget="radio" options="{'horizontal': true}"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
                 <xpath expr="//div[@id='product_get_pic_setting']" position="replace">
                    <div class="col-12 col-lg-6 o_setting_box" id="product_get_pic_setting">
                        <div class="o_setting_left_pane">
                            <field name="module_product_images"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="module_product_images" string="Google Images"/>
                            <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/products_prices/products/product_images.html"
                               title="Documentation" class="o_doc_link" target="_blank"/>
                            <div class="text-muted">
                                Get product pictures using Barcode
                            </div>
                            <div class="content-group mt16"
                                attrs="{'invisible': [('module_product_images','=',False)]}"
                                id="msg_module_product_images">
                                <div class="mt16 text-warning">
                                    <strong>Save</strong> this page and come back
                                    here to set up the feature.
                                </div>
                            </div>
                        </div>
                    </div>
                 </xpath>
            </field>
        </record>
</odoo>

```

## File: views\res_country_group_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="inherits_website_sale_country_group_form" model="ir.ui.view">
        <field name="name">res.country.group.form.inherit.product</field>
        <field name="model">res.country.group</field>
        <field name="inherit_id" ref="base.view_country_group_form"/>
        <field name="arch" type="xml">
            <group name="country_group" position="after">
                <field name="pricelist_ids"/>
            </group>
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

## File: wizard\product_label_layout.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import _, api, fields, models
from odoo.exceptions import UserError


class ProductLabelLayout(models.TransientModel):
    _name = 'product.label.layout'
    _description = 'Choose the sheet layout to print the labels'

    print_format = fields.Selection([
        ('dymo', 'Dymo'),
        ('2x7xprice', '2 x 7 with price'),
        ('4x7xprice', '4 x 7 with price'),
        ('4x12', '4 x 12'),
        ('4x12xprice', '4 x 12 with price')], string="Format", default='2x7xprice', required=True)
    custom_quantity = fields.Integer('Quantity', default=1, required=True)
    product_ids = fields.Many2many('product.product')
    product_tmpl_ids = fields.Many2many('product.template')
    extra_html = fields.Html('Extra Content', default='')
    rows = fields.Integer(compute='_compute_dimensions')
    columns = fields.Integer(compute='_compute_dimensions')

    @api.depends('print_format')
    def _compute_dimensions(self):
        for wizard in self:
            if 'x' in wizard.print_format:
                columns, rows = wizard.print_format.split('x')[:2]
                wizard.columns = int(columns)
                wizard.rows = int(rows)
            else:
                wizard.columns, wizard.rows = 1, 1

    def _prepare_report_data(self):
        if self.custom_quantity <= 0:
            raise UserError(_('You need to set a positive quantity.'))

        # Get layout grid
        if self.print_format == 'dymo':
            xml_id = 'product.report_product_template_label_dymo'
        elif 'x' in self.print_format:
            xml_id = 'product.report_product_template_label'
        else:
            xml_id = ''

        active_model = ''
        if self.product_tmpl_ids:
            products = self.product_tmpl_ids.ids
            active_model = 'product.template'
        elif self.product_ids:
            products = self.product_ids.ids
            active_model = 'product.product'
        else:
            raise UserError(_("No product to print, if the product is archived please unarchive it before printing its label."))

        # Build data to pass to the report
        data = {
            'active_model': active_model,
            'quantity_by_product': {p: self.custom_quantity for p in products},
            'layout_wizard': self.id,
            'price_included': 'xprice' in self.print_format,
        }
        return xml_id, data

    def process(self):
        self.ensure_one()
        xml_id, data = self._prepare_report_data()
        if not xml_id:
            raise UserError(_('Unable to find report template for %s format', self.print_format))
        report_action = self.env.ref(xml_id).report_action(None, data=data)
        report_action.update({'close_on_report_download': True})
        return report_action

```

## File: wizard\product_label_layout_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_label_layout_form" model="ir.ui.view">
        <field name="name">product.label.layout.form</field>
        <field name="model">product.label.layout</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <group>
                        <field name="product_ids" invisible="1"/>
                        <field name="product_tmpl_ids" invisible="1"/>
                        <field name="custom_quantity"/>
                        <field name="print_format" widget="radio"/>
                    </group>
                    <group>
                        <field name="extra_html" widget="html" attrs="{'invisible': [('print_format', '!=', '2x7xprice')]}"/>
                    </group>
                </group>
                <footer>
                    <button name="process" string="Confirm" type="object" class="btn-primary"/>
                    <button string="Discard" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_open_label_layout" model="ir.actions.act_window">
        <field name="name">Choose Labels Layout</field>
        <field name="res_model">product.label.layout</field>
        <field name="view_ids"
                eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'form', 'view_id': ref('product_label_layout_form')})]" />
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_label_layout

```

