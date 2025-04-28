# Odoo Module: test_website_slides_full

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Test Full eLearning Flow',
    'version': '1.0',
    'category': 'Tools',
    'description': """
This module will test the main certification flow of Odoo.
It will install the e-learning, survey and e-commerce apps and make a complete
certification flow including purchase, certification, failure and success.
""",
    'depends': [
        'website_sale_product_configurator',
        'website_sale_slides',
        'website_slides_forum',
        'website_slides_survey',
        'payment_test'
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'data': [
        'views/assets.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="product_course_channel_1_option_0" model="product.product">
        <field name="name">Water can</field>
        <field name="standard_price">12.0</field>
        <field name="list_price">12.0</field>
        <field name="type">consu</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_slides/static/img/water_can.jpg"/>
    </record>

    <record id="product_course_channel_1_option_1" model="product.product">
        <field name="name">Flower pot</field>
        <field name="standard_price">4.5</field>
        <field name="list_price">4.5</field>
        <field name="type">consu</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_slides/static/img/flower_pot.jpg"/>
    </record>

    <record id="website_sale_slides.product_course_channel_1_product_template" model="product.template">
        <field name="optional_product_ids" eval="[
            (6, 0, [ref('test_website_slides_full.product_course_channel_1_option_0_product_template'),
                    ref('test_website_slides_full.product_course_channel_1_option_1_product_template')]
        )]"/>
    </record>
</data></odoo>

```

## File: views\assets.xml

```xml
<?xml version="1.0" ?>
<odoo><data>
    <template id="assets_tests" inherit_id="web.assets_tests" name="Full Slides Tests Assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/test_website_slides_full/tests/tours/slides_certification_member.js"/>
        </xpath>
    </template>
</data></odoo>

```

