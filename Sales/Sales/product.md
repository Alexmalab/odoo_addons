# Odoo Module: product

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
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
        'views/product_tag_views.xml',
        'views/product_views.xml',  # To keep after product_tag_views.xml because it depends on it.

        'views/res_config_settings_views.xml',
        'views/product_attribute_views.xml',
        'views/product_attribute_value_views.xml',
        'views/product_category_views.xml',
        'views/product_document_views.xml',
        'views/product_packaging_views.xml',
        'views/product_pricelist_item_views.xml',
        'views/product_pricelist_views.xml',
        'views/product_supplierinfo_views.xml',
        'views/product_template_views.xml',
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
            'product/static/src/product_catalog/**/*.js',
            'product/static/src/product_catalog/**/*.xml',
            'product/static/src/product_catalog/**/*.scss',
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

## File: controllers\catalog.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route, Controller


class ProductCatalogController(Controller):

    @route('/product/catalog/order_lines_info', auth='user', type='json')
    def product_catalog_get_order_lines_info(self, res_model, order_id, product_ids, **kwargs):
        """ Returns products information to be shown in the catalog.

        :param string res_model: The order model.
        :param int order_id: The order id.
        :param list product_ids: The products currently displayed in the product catalog, as a list
                                 of `product.product` ids.
        :rtype: dict
        :return: A dict with the following structure:
            {
                product.id: {
                    'productId': int
                    'quantity': float (optional)
                    'price': float
                    'readOnly': bool (optional)
                }
            }
        """
        order = request.env[res_model].browse(order_id)
        return order.with_company(order.company_id)._get_product_catalog_order_line_info(
            product_ids, **kwargs,
        )

    @route('/product/catalog/update_order_line_info', auth='user', type='json')
    def product_catalog_update_order_line_info(self, res_model, order_id, product_id, quantity=0, **kwargs):
        """ Update order line information on a given order for a given product.

        :param string res_model: The order model.
        :param int order_id: The order id.
        :param int product_id: The product, as a `product.product` id.
        :return: The unit price price of the product, based on the pricelist of the order and
                 the quantity selected.
        :rtype: float
        """
        order = request.env[res_model].browse(order_id)
        return order.with_company(order.company_id)._update_order_line_info(
            product_id, quantity, **kwargs,
        )

```

## File: controllers\product_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging

from odoo import _
from odoo.http import request, route, Controller

logger = logging.getLogger(__name__)


class ProductDocumentController(Controller):

    @route('/product/document/upload', type='http', methods=['POST'], auth='user')
    def upload_document(self, ufile, res_model, res_id):
        if res_model not in ('product.product', 'product.template'):
            return

        record = request.env[res_model].browse(int(res_id)).exists()

        if not record or not record.check_access_rights('write'):
            return

        files = request.httprequest.files.getlist('ufile')
        result = {'success': _("All files uploaded")}
        for ufile in files:
            try:
                mimetype = ufile.content_type
                request.env['product.document'].create({
                    'name': ufile.filename,
                    'res_model': record._name,
                    'res_id': record.id,
                    'company_id': record.company_id.id,
                    'mimetype': mimetype,
                    'raw': ufile.read(),
                })
            except Exception as e:
                logger.exception("Failed to upload document %s", ufile.filename)
                result = {'error': str(e)}

        return json.dumps(result)

```

## File: controllers\__init__.py

```python
from . import catalog
from . import product_document

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

        <record id="product_product_4_product_template_document" model="product.document">
            <field name="name">Customizable Desk Document.pdf</field>
            <field name="datas" type="base64" file="product/static/demo/customizable_desk_document.pdf"/>
            <field name="mimetype">application/pdf</field>
            <field name="res_model">product.template</field>
            <field name="res_id" ref="product.product_product_4_product_template"/>
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

        <record id="product_template_attribute_exclusion_1" model="product.template.attribute.exclusion">
            <field name="product_tmpl_id" ref="product.product_product_4_product_template" />
            <field name="value_ids" eval="[(6, 0, [ref('product.product_4_attribute_2_value_2')])]"/>
        </record>
        <record id="product_template_attribute_exclusion_2" model="product.template.attribute.exclusion">
            <field name="product_tmpl_id" ref="product.product_product_11_product_template" />
            <field name="value_ids" eval="[(6, 0, [ref('product.product_11_attribute_1_value_1')])]"/>
        </record>
        <record id="product_template_attribute_exclusion_3" model="product.template.attribute.exclusion">
            <field name="product_tmpl_id" ref="product.product_product_11_product_template" />
            <field name="value_ids" eval="[(6, 0, [ref('product.product_11_attribute_1_value_2')])]"/>
        </record>

        <!--
        The "Customizable Desk's Aluminium" attribute value will excude:
        - The "Customizable Desk's Black" attribute
        - The "Office Chair's Steel" attribute
         -->
        <record id="product_4_attribute_1_value_2" model="product.template.attribute.value">
            <field name="exclude_for" eval="[(6, 0, [ref('product.product_template_attribute_exclusion_1'), ref('product.product_template_attribute_exclusion_2')])]" />
        </record>
        <!--
        The "Customizable Desk's Steel" attribute value will excude:
        - The "Office Chair's Aluminium" attribute
        -->
        <record id="product_4_attribute_1_value_1" model="product.template.attribute.value">
            <field name="exclude_for" eval="[(6, 0, [ref('product.product_template_attribute_exclusion_3')])]" />
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

        <record id="product_product_25_document" model="product.document">
            <field name="name">Acoustic Bloc Screens Document.pdf</field>
            <field name="datas" type="base64" file="product/static/demo/acoustic_bloc_screen_document.pdf"/>
            <field name="mimetype">application/pdf</field>
            <field name="res_model">product.product</field>
            <field name="res_id" ref="product.product_product_25"/>
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

        <record id="consu_delivery_02_document" model="product.document">
            <field name="name">Large Meeting Table Document.pdf</field>
            <field name="datas" type="base64" file="product/static/demo/large_meeting_table_document.pdf"/>
            <field name="mimetype">application/pdf</field>
            <field name="res_model">product.template</field>
            <field name="res_id" ref="consu_delivery_02_product_template"/>
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
                    "Please increase the rounding of those units of measure, or the digits of this Decimal Accuracy.",
                    '\n'.join(uom_descriptions)),
            }}

```

## File: models\ir_attachment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'

    @api.model_create_multi
    def create(self, vals_list):
        """Create product.document for attachments added in products chatters"""
        attachments = super().create(vals_list)
        if not self.env.context.get('disable_product_documents_creation'):
            product_attachments = attachments.filtered(
                lambda attachment:
                    attachment.res_model in ('product.product', 'product.template')
                    and not attachment.res_field
            )
            if product_attachments:
                self.env['product.document'].sudo().create(
                    {
                        'ir_attachment_id': attachment.id
                    } for attachment in product_attachments
                )
        return attachments

```

## File: models\product_attribute.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ProductAttribute(models.Model):
    _name = "product.attribute"
    _description = "Product Attribute"
    # if you change this _order, keep it in sync with the method
    # `_sort_key_attribute_value` in `product.template`
    _order = 'sequence, id'

    _sql_constraints = [
        (
            'check_multi_checkbox_no_variant',
            "CHECK(display_type != 'multi' OR create_variant = 'no_variant')",
            "Multi-checkbox display type is not compatible with the creation of variants"
        ),
    ]

    name = fields.Char(string="Attribute", required=True, translate=True)
    create_variant = fields.Selection(
        selection=[
            ('always', 'Instantly'),
            ('dynamic', 'Dynamically'),
            ('no_variant', 'Never (option)'),
        ],
        default='always',
        string="Variants Creation Mode",
        help="""- Instantly: All possible variants are created as soon as the attribute and its values are added to a product.
        - Dynamically: Each variant is created only when its corresponding attributes and values are added to a sales order.
        - Never: Variants are never created for the attribute.
        Note: the variants creation mode cannot be changed once the attribute is used on at least one product.""",
        required=True)
    display_type = fields.Selection(
        selection=[
            ('radio', 'Radio'),
            ('pills', 'Pills'),
            ('select', 'Select'),
            ('color', 'Color'),
            ('multi', 'Multi-checkbox (option)'),
        ],
        default='radio',
        required=True,
        help="The display type used in the Product Configurator.")
    sequence = fields.Integer(string="Sequence", help="Determine the display order", index=True, default=20)

    value_ids = fields.One2many(
        comodel_name='product.attribute.value',
        inverse_name='attribute_id',
        string="Values", copy=True)

    attribute_line_ids = fields.One2many(
        comodel_name='product.template.attribute.line',
        inverse_name='attribute_id',
        string="Lines")
    product_tmpl_ids = fields.Many2many(
        comodel_name='product.template',
        string="Related Products",
        compute='_compute_products',
        store=True)
    number_related_products = fields.Integer(compute='_compute_number_related_products')

    @api.depends('product_tmpl_ids')
    def _compute_number_related_products(self):
        res = {
            attribute.id: count
            for attribute, count in self.env['product.template.attribute.line']._read_group(
                domain=[('attribute_id', 'in', self.ids)],
                groupby=['attribute_id'],
                aggregates=['product_tmpl_id:count_distinct'],
            )
        }
        for pa in self:
            pa.number_related_products = res.get(pa.id, 0)

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
                    raise UserError(_(
                        "You cannot change the Variants Creation Mode of the attribute %(attribute)s"
                        " because it is used on the following products:\n%(products)s",
                        attribute=pa.display_name,
                        products=", ".join(pa.product_tmpl_ids.mapped('display_name')),
                    ))
        invalidate = 'sequence' in vals and any(record.sequence != vals['sequence'] for record in self)
        res = super().write(vals)
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
                raise UserError(_(
                    "You cannot delete the attribute %(attribute)s because it is used on the"
                    " following products:\n%(products)s",
                    attribute=pa.display_name,
                    products=", ".join(pa.product_tmpl_ids.mapped('display_name')),
                ))

    def action_open_related_products(self):
        return {
            'type': 'ir.actions.act_window',
            'name': _("Related Products"),
            'res_model': 'product.template',
            'view_mode': 'tree,form',
            'domain': [('id', 'in', self.product_tmpl_ids.ids)],
        }

```

## File: models\product_attribute_custom_value.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProductAttributeCustomValue(models.Model):
    _name = 'product.attribute.custom.value'
    _description = "Product Attribute Custom Value"
    _order = 'custom_product_template_attribute_value_id, id'

    name = fields.Char(string="Name", compute='_compute_name')
    custom_product_template_attribute_value_id = fields.Many2one(
        comodel_name='product.template.attribute.value',
        string="Attribute Value",
        required=True,
        ondelete='restrict')
    custom_value = fields.Char(string="Custom Value")

    @api.depends('custom_product_template_attribute_value_id.name', 'custom_value')
    def _compute_name(self):
        for record in self:
            name = (record.custom_value or '').strip()
            if record.custom_product_template_attribute_value_id.display_name:
                name = "%s: %s" % (record.custom_product_template_attribute_value_id.display_name, name)
            record.name = name

```

## File: models\product_attribute_value.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ProductAttributeValue(models.Model):
    _name = 'product.attribute.value'
    # if you change this _order, keep it in sync with the method
    # `_sort_key_variant` in `product.template'
    _order = 'attribute_id, sequence, id'
    _description = "Attribute Value"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char(string="Value", required=True, translate=True)
    sequence = fields.Integer(string="Sequence", help="Determine the display order", index=True)
    attribute_id = fields.Many2one(
        comodel_name='product.attribute',
        string="Attribute",
        help="The attribute cannot be changed once the value is used on at least one product.",
        ondelete='cascade',
        required=True,
        index=True)

    pav_attribute_line_ids = fields.Many2many(
        comodel_name='product.template.attribute.line',
        relation='product_attribute_value_product_template_attribute_line_rel',
        string="Lines",
        copy=False)
    is_used_on_products = fields.Boolean(
        string="Used on Products", compute='_compute_is_used_on_products')

    default_extra_price = fields.Float()
    is_custom = fields.Boolean(
        string="Is custom value",
        help="Allow users to input custom values for this attribute value")
    html_color = fields.Char(
        string="Color",
        help="Here you can set a specific HTML color index (e.g. #ff0000)"
            " to display the color if the attribute type is 'Color'.")
    display_type = fields.Selection(related='attribute_id.display_type')
    color = fields.Integer(string="Color Index", default=_get_default_color)
    image = fields.Image(
        string="Image",
        help="You can upload an image that will be used as the color of the attribute value.",
        max_width=70,
        max_height=70,
    )

    _sql_constraints = [
        ('value_company_uniq',
         'unique (name, attribute_id)',
         "You cannot create two values with the same name for the same attribute.")
    ]

    @api.depends('pav_attribute_line_ids')
    def _compute_is_used_on_products(self):
        for pav in self:
            pav.is_used_on_products = bool(pav.pav_attribute_line_ids)

    @api.depends('attribute_id')
    @api.depends_context('show_attribute')
    def _compute_display_name(self):
        """Override because in general the name of the value is confusing if it
        is displayed without the name of the corresponding attribute.
        Eg. on product list & kanban views, on BOM form view

        However during variant set up (on the product template form) the name of
        the attribute is already on each line so there is no need to repeat it
        on every value.
        """
        if not self.env.context.get('show_attribute', True):
            return super()._compute_display_name()
        for value in self:
            value.display_name = f"{value.attribute_id.name}: {value.name}"

    def write(self, values):
        if 'attribute_id' in values:
            for pav in self:
                if pav.attribute_id.id != values['attribute_id'] and pav.is_used_on_products:
                    raise UserError(_(
                        "You cannot change the attribute of the value %(value)s because it is used"
                        " on the following products: %(products)s",
                        value=pav.display_name,
                        products=", ".join(pav.pav_attribute_line_ids.product_tmpl_id.mapped('display_name')),
                    ))

        invalidate = 'sequence' in values and any(record.sequence != values['sequence'] for record in self)
        res = super().write(values)
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
                raise UserError(_(
                    "You cannot delete the value %(value)s because it is used on the following "
                    "products:\n%(products)s\n If the value has been associated to a product in the"
                    " past, you will not be able to delete it.",
                    value=pav.display_name,
                    products=", ".join(pav.pav_attribute_line_ids.product_tmpl_id.mapped('display_name')),
                ))
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

```

## File: models\product_catalog_mixin.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class ProductCatalogMixin(models.AbstractModel):
    """ This mixin should be inherited when the model should be able to work
    with the product catalog.
    It assumes the model using this mixin has a O2M field where the products are added/removed and
    this field's co-related model should has a method named `_get_product_catalog_lines_data`.
    """
    _name = 'product.catalog.mixin'
    _description = 'Product Catalog Mixin'

    def action_add_from_catalog(self):
        kanban_view_id = self.env.ref('product.product_view_kanban_catalog').id
        search_view_id = self.env.ref('product.product_view_search_catalog').id
        additional_context = self._get_action_add_from_catalog_extra_context()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Products'),
            'res_model': 'product.product',
            'views': [(kanban_view_id, 'kanban'), (False, 'form')],
            'search_view_id': [search_view_id, 'search'],
            'domain': self._get_product_catalog_domain(),
            'context': {**self.env.context, **additional_context},
        }

    def _default_order_line_values(self):
        return {
            'quantity': 0,
            'readOnly': self._is_readonly() if self else False,
        }

    def _get_product_catalog_domain(self):
        """Get the domain to search for products in the catalog.

        For a model that uses products that has to be hidden in the catalog, it
        must override this method and extend the appropriate domain.
        :returns: A list of tuples that represents a domain.
        :rtype: list
        """
        return ['|', ('company_id', '=', False), ('company_id', 'parent_of', self.company_id.id)]

    def _get_product_catalog_record_lines(self, product_ids):
        """ Returns the record's lines grouped by product.
        Must be overrided by each model using this mixin.

        :param list product_ids: The ids of the products currently displayed in the product catalog.
        :rtype: dict
        """
        return {}

    def _get_product_catalog_order_data(self, products, **kwargs):
        """ Returns a dict containing the products' data. Those data are for products who aren't in
        the record yet. For products already in the record, see `_get_product_catalog_lines_data`.

        For each product, its id is the key and the value is another dict with all needed data.
        By default, the price is the only needed data but each model is free to add more data.
        Must be overrided by each model using this mixin.

        :param products: Recordset of `product.product`.
        :param dict kwargs: additional values given for inherited models.
        :rtype: dict
        :return: A dict with the following structure:
            {
                'productId': int
                'quantity': float (optional)
                'productType': string
                'price': float
                'readOnly': bool (optional)
            }
        """
        res = {}
        for product in products:
            res[product.id] = {'productType': product.type}
        return res

    def _get_product_catalog_order_line_info(self, product_ids, **kwargs):
        """ Returns products information to be shown in the catalog.
        :param list product_ids: The products currently displayed in the product catalog, as a list
                                 of `product.product` ids.
        :param dict kwargs: additional values given for inherited models.
        :rtype: dict
        :return: A dict with the following structure:
            {
                'productId': int
                'quantity': float (optional)
                'productType': string
                'price': float
                'readOnly': bool (optional)
            }
        """
        order_line_info = {}
        default_data = self._default_order_line_values()

        for product, record_lines in self._get_product_catalog_record_lines(product_ids).items():
            order_line_info[product.id] = {
               **record_lines._get_product_catalog_lines_data(**kwargs),
               'productType': product.type,
            }
            product_ids.remove(product.id)

        products = self.env['product.product'].browse(product_ids)
        product_data = self._get_product_catalog_order_data(products, **kwargs)
        for product_id, data in product_data.items():
            order_line_info[product_id] = {**default_data, **data}
        return order_line_info

    def _get_action_add_from_catalog_extra_context(self):
        return {
            'product_catalog_order_id': self.id,
            'product_catalog_order_model': self._name,
        }

    def _is_readonly(self):
        """ Must be overrided by each model using this mixin.
        :return: Whether the record is read-only or not.
        :rtype: bool
        """
        return False

    def _update_order_line_info(self, product_id, quantity, **kwargs):
        """ Update the line information for a given product or create a new one if none exists yet.
        Must be overrided by each model using this mixin.
        :param int product_id: The product, as a `product.product` id.
        :param int quantity: The product's quantity.
        :param dict kwargs: additional values given for inherited models.
        :return: The unit price of the product, based on the pricelist of the
                 purchase order and the quantity selected.
        :rtype: float
        """
        return 0

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
    product_properties_definition = fields.PropertiesDefinition('Product Properties')

    @api.depends('name', 'parent_id.complete_name')
    def _compute_complete_name(self):
        for category in self:
            if category.parent_id:
                category.complete_name = '%s / %s' % (category.parent_id.complete_name, category.name)
            else:
                category.complete_name = category.name

    def _compute_product_count(self):
        read_group_res = self.env['product.template']._read_group([('categ_id', 'child_of', self.ids)], ['categ_id'], ['__count'])
        group_data = {categ.id: count for categ, count in read_group_res}
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
        category = self.create({'name': name})
        return category.id, category.display_name

    @api.depends_context('hierarchical_naming')
    def _compute_display_name(self):
        if self.env.context.get('hierarchical_naming', True):
            return super()._compute_display_name()
        for record in self:
            record.display_name = record.name

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

## File: models\product_document.py

```python

# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class ProductDocument(models.Model):
    _name = 'product.document'
    _description = "Product Document"
    _inherits = {
        'ir.attachment': 'ir_attachment_id',
    }
    _order = 'name'

    ir_attachment_id = fields.Many2one(
        'ir.attachment',
        string="Related attachment",
        required=True,
        ondelete='cascade')

    active = fields.Boolean(default=True)

    @api.onchange('url')
    def _onchange_url(self):
        for attachment in self:
            if attachment.type == 'url' and attachment.url and\
                not attachment.url.startswith(('https://', 'http://', 'ftp://')):
                raise ValidationError(_(
                    "Please enter a valid URL.\nExample: https://www.odoo.com\n\nInvalid URL: %s",
                    attachment.url
                ))

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, vals_list):
        return super(
            ProductDocument,
            self.with_context(disable_product_documents_creation=True),
        ).create(vals_list)

    def copy(self, default=None):
        default = default if default is not None else {}
        ir_default = default
        if ir_default:
            ir_fields = list(self.env['ir.attachment']._fields)
            ir_default = {field : default[field] for field in default if field in ir_fields}
        new_attach = self.ir_attachment_id.with_context(
            no_document=True,
            disable_product_documents_creation=True,
        ).copy(ir_default)
        return super().copy(dict(default, ir_attachment_id=new_attach.id))

    def unlink(self):
        attachments = self.ir_attachment_id
        res = super().unlink()
        return res and attachments.unlink()

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
    product_id = fields.Many2one('product.product', string='Product', check_company=True, required=True, ondelete="cascade")
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

    def _compute_qty(self, qty, qty_uom=False):
        """Returns the qty of this packaging that qty converts to.
        A float is returned because there are edge cases where some users use
        "part" of a packaging

        :param qty: float of product quantity (given in product UoM if no qty_uom provided)
        :param qty_uom: Optional uom of quantity
        :returns: float of packaging qty
        """
        self.ensure_one()
        if qty_uom:
            qty = qty_uom._compute_quantity(qty, self.product_uom_id)
        return float_round(qty / self.qty, precision_rounding=self.product_uom_id.rounding)

```

## File: models\product_pricelist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class Pricelist(models.Model):
    _name = "product.pricelist"
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = "Pricelist"
    _rec_names_search = ['name', 'currency_id']  # TODO check if should be removed
    _order = "sequence asc, id asc"

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
        required=True,
        tracking=1,
    )

    company_id = fields.Many2one(
        comodel_name='res.company',
        tracking=5,
        default=lambda self: self.env.company,
    )

    country_group_ids = fields.Many2many(
        comodel_name='res.country.group',
        relation='res_country_group_pricelist_rel',
        column1='pricelist_id',
        column2='res_country_group_id',
        string="Country Groups",
        tracking=10,
    )

    discount_policy = fields.Selection(
        selection=[
            ('with_discount', "Discount included in the price"),
            ('without_discount', "Show public price & discount to the customer"),
        ],
        default='with_discount',
        required=True,
        tracking=15,
    )

    item_ids = fields.One2many(
        comodel_name='product.pricelist.item',
        inverse_name='pricelist_id',
        string="Pricelist Rules",
        domain=[
            '&',
            '|', ('product_tmpl_id', '=', None), ('product_tmpl_id.active', '=', True),
            '|', ('product_id', '=', None), ('product_id.active', '=', True),
        ],
        copy=True)

    @api.depends('currency_id')
    def _compute_display_name(self):
        for pricelist in self:
            pricelist.display_name = f'{pricelist.name} ({pricelist.currency_id.name})'

    def write(self, values):
        res = super().write(values)

        # Make sure that there is no multi-company issue in the existing rules after the company
        # change.
        if 'company_id' in values and len(self) == 1:
            self.item_ids._check_company()

        return res

    def copy_data(self, default=None):
        default = default or {}
        if not default.get('name'):
            default['name'] = _('%s (copy)', self.name)
        return super().copy_data(default=default)

    def _get_products_price(self, products, *args, **kwargs):
        """Compute the pricelist prices for the specified products, quantity & uom.

        Note: self and self.ensure_one()

        :param products: recordset of products (product.product/product.template)
        :param float quantity: quantity of products requested (in given uom)
        :param currency: record of currency (res.currency) (optional)
        :param uom: unit of measure (uom.uom record) (optional)
            If not specified, prices returned are expressed in product uoms
        :param date: date to use for price computation and currency conversions (optional)
        :type date: date or datetime

        :returns: {product_id: product price}, considering the current pricelist if any
        :rtype: dict(int, float)
        """
        self and self.ensure_one()  # self is at most one record
        return {
            product_id: res_tuple[0]
            for product_id, res_tuple in self._compute_price_rule(products, *args, **kwargs).items()
        }

    def _get_product_price(self, product, *args, **kwargs):
        """Compute the pricelist price for the specified product, qty & uom.

        Note: self and self.ensure_one()

        :param product: product record (product.product/product.template)
        :param float quantity: quantity of products requested (in given uom)
        :param currency: record of currency (res.currency) (optional)
        :param uom: unit of measure (uom.uom record) (optional)
            If not specified, prices returned are expressed in product uoms
        :param date: date to use for price computation and currency conversions (optional)
        :type date: date or datetime

        :returns: unit price of the product, considering pricelist rules if any
        :rtype: float
        """
        self and self.ensure_one()  # self is at most one record
        return self._compute_price_rule(product, *args, **kwargs)[product.id][0]

    def _get_product_price_rule(self, product, *args, **kwargs):
        """Compute the pricelist price & rule for the specified product, qty & uom.

        Note: self and self.ensure_one()

        :param product: product record (product.product/product.template)
        :param float quantity: quantity of products requested (in given uom)
        :param currency: record of currency (res.currency) (optional)
        :param uom: unit of measure (uom.uom record) (optional)
            If not specified, prices returned are expressed in product uoms
        :param date: date to use for price computation and currency conversions (optional)
        :type date: date or datetime

        :returns: (product unit price, applied pricelist rule id)
        :rtype: tuple(float, int)
        """
        self and self.ensure_one()  # self is at most one record
        return self._compute_price_rule(product, *args, **kwargs)[product.id]

    def _get_product_rule(self, product, *args, **kwargs):
        """Compute the pricelist price & rule for the specified product, qty & uom.

        Note: self and self.ensure_one()

        :param product: product record (product.product/product.template)
        :param float quantity: quantity of products requested (in given uom)
        :param currency: record of currency (res.currency) (optional)
        :param uom: unit of measure (uom.uom record) (optional)
            If not specified, prices returned are expressed in product uoms
        :param date: date to use for price computation and currency conversions (optional)
        :type date: date or datetime

        :returns: applied pricelist rule id
        :rtype: int or False
        """
        self and self.ensure_one()  # self is at most one record
        return self._compute_price_rule(product, *args, compute_price=False, **kwargs)[product.id][1]

    def _compute_price_rule(
            self, products, quantity, currency=None, uom=None, date=False, compute_price=True,
            **kwargs
    ):
        """ Low-level method - Mono pricelist, multi products
        Returns: dict{product_id: (price, suitable_rule) for the given pricelist}

        Note: self and self.ensure_one()

        :param products: recordset of products (product.product/product.template)
        :param float quantity: quantity of products requested (in given uom)
        :param currency: record of currency (res.currency)
                         note: currency.ensure_one()
        :param uom: unit of measure (uom.uom record)
            If not specified, prices returned are expressed in product uoms
        :param date: date to use for price computation and currency conversions
        :type date: date or datetime
        :param bool compute_price: whether the price should be computed (default: True)

        :returns: product_id: (price, pricelist_rule)
        :rtype: dict
        """
        self and self.ensure_one()  # self is at most one record

        currency = currency or self.currency_id or self.env.company.currency_id
        currency.ensure_one()

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
                qty_in_product_uom = target_uom._compute_quantity(
                    quantity, product_uom, raise_if_failure=False
                )
            else:
                qty_in_product_uom = quantity

            for rule in rules:
                if rule._is_applicable_for(product, qty_in_product_uom):
                    suitable_rule = rule
                    break

            if compute_price:
                price = suitable_rule._compute_price(
                    product, quantity, target_uom, date=date, currency=currency)
            else:
                # Skip price computation when only the rule is requested.
                price = 0.0
            results[product.id] = (price, suitable_rule.id)

        return results

    # Split methods to ease (community) overrides
    def _get_applicable_rules(self, products, date, **kwargs):
        self and self.ensure_one()  # self is at most one record
        if not self:
            return self.env['product.pricelist.item']

        # Do not filter out archived pricelist items, since it means current pricelist is also archived
        # We do not want the computation of prices for archived pricelist to always fallback on the Sales price
        # because no rule was found (thanks to the automatic orm filtering on active field)
        return self.env['product.pricelist.item'].with_context(active_test=False).search(
            self._get_applicable_rules_domain(products=products, date=date, **kwargs)
        ).with_context(self.env.context)

    def _get_applicable_rules_domain(self, products, date, **kwargs):
        self and self.ensure_one()  # self is at most one record
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
    def _price_get(self, product, quantity, **kwargs):
        """ Multi pricelist, mono product - returns price per pricelist """
        return {
            key: price[0]
            for key, price in self._compute_price_rule_multi(product, quantity, **kwargs)[product.id].items()}

    def _compute_price_rule_multi(self, products, quantity, uom=None, date=False, **kwargs):
        """ Low-level method - Multi pricelist, multi products
        Returns: dict{product_id: dict{pricelist_id: (price, suitable_rule)} }"""
        if not self.ids:
            pricelists = self.search([])
        else:
            pricelists = self
        results = {}
        for pricelist in pricelists:
            subres = pricelist._compute_price_rule(products, quantity, uom=uom, date=date, **kwargs)
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
        Else, it will return the generic property (res_id not set)
        Else, it will return the first available pricelist if any

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
        specific_properties = Property._get_multi(
            'property_product_pricelist', Partner._name,
            list(models.origin_ids(partner_ids)),  # Some NewID can be in the partner_ids
        )
        result = {}
        remaining_partner_ids = []
        for pid in partner_ids:
            if (
                specific_properties.get(pid)
                and specific_properties[pid]._get_partner_pricelist_multi_filter_hook()
            ):
                result[pid] = specific_properties[pid]
            elif (
                isinstance(pid, models.NewId) and specific_properties.get(pid.origin)
                and specific_properties[pid.origin]._get_partner_pricelist_multi_filter_hook()
            ):
                result[pid] = specific_properties[pid.origin]
            else:
                remaining_partner_ids.append(pid)

        if remaining_partner_ids:
            # get fallback pricelist when no pricelist for a given country
            pl_fallback = (
                Pricelist.search(pl_domain + [('country_group_ids', '=', False)], limit=1) or
                Property._get('property_product_pricelist', 'res.partner') or
                Pricelist.search(pl_domain, limit=1)
            )
            # group partners by country, and find a pricelist for each country
            remaining_partners = self.env['res.partner'].browse(remaining_partner_ids)
            partners_by_country = remaining_partners.grouped('country_id')
            for country, partners in partners_by_country.items():
                if not country and (country_code := self.env.context.get('country_code')):
                    country = self.env['res.country'].search([('code', '=', country_code)], limit=1)
                pl = Pricelist.search(pl_domain + [('country_group_ids.country_ids', '=', country.id if country else False)], limit=1)
                pl = pl or pl_fallback
                result.update(dict.fromkeys(partners._ids, pl))

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
             "Cost Price: The base price will be the cost price.\n"
             "Other Pricelist: Computation of the base price based on another Pricelist.")
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
                item.name = _("Category: %s", item.categ_id.display_name)
            elif item.product_tmpl_id and item.applied_on == '1_product':
                item.name = _("Product: %s", item.product_tmpl_id.display_name)
            elif item.product_id and item.applied_on == '0_product_variant':
                item.name = _("Variant: %s", item.product_id.display_name)
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
                raise ValidationError(_('%s: end date (%s) should be greater than start date (%s)', item.display_name, format_datetime(self.env, item.date_end), format_datetime(self.env, item.date_start)))
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

        Note: self and self.ensure_one()

        :param product: recordset of product (product.product/product.template)
        :param float qty: quantity of products requested (in given uom)
        :param uom: unit of measure (uom.uom record)
        :param datetime date: date to use for price computation and currency conversions
        :param currency: currency (for the case where self is empty)

        :returns: price according to pricelist rule or the product price, expressed in the param
                  currency, the pricelist currency or the company currency
        :rtype: float
        """
        self and self.ensure_one()  # self is at most one record
        product.ensure_one()
        uom.ensure_one()

        currency = currency or self.currency_id or self.env.company.currency_id
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

    def _compute_base_price(self, product, quantity, uom, date, currency):
        """ Compute the base price for a given rule

        :param product: recordset of product (product.product/product.template)
        :param float qty: quantity of products requested (in given uom)
        :param uom: unit of measure (uom.uom record)
        :param datetime date: date to use for price computation and currency conversions
        :param currency: currency in which the returned price must be expressed

        :returns: base price, expressed in provided pricelist currency
        :rtype: float
        """
        currency.ensure_one()

        rule_base = self.base or 'list_price'
        if rule_base == 'pricelist' and self.base_pricelist_id:
            price = self.base_pricelist_id._get_product_price(
                product, quantity, currency=self.base_pricelist_id.currency_id, uom=uom, date=date
            )
            src_currency = self.base_pricelist_id.currency_id
        elif rule_base == "standard_price":
            src_currency = product.cost_currency_id
            price = product._price_compute(rule_base, uom=uom, date=date)[product.id]
        else: # list_price
            src_currency = product.currency_id
            price = product._price_compute(rule_base, uom=uom, date=date)[product.id]

        if src_currency != currency:
            price = src_currency._convert(price, currency, self.env.company, date, round=False)

        return price

    def _compute_price_before_discount(self, *args, **kwargs):
        """Compute the base price of the lowest pricelist rule whose pricelist discount_policy
        is set to show the discount to the customer.

        :param product: recordset of product (product.product/product.template)
        :param float qty: quantity of products requested (in given uom)
        :param uom: unit of measure (uom.uom record)
        :param datetime date: date to use for price computation and currency conversions
        :param currency: currency in which the returned price must be expressed

        :returns: base price, expressed in provided pricelist currency
        :rtype: float
        """
        pricelist_rule = self
        if pricelist_rule and pricelist_rule.pricelist_id.discount_policy == 'without_discount':
            pricelist_item = pricelist_rule
            # Find the lowest pricelist rule whose pricelist is configured to show the discount
            # to the customer.
            while (
                pricelist_item.base == 'pricelist'
                and pricelist_item.base_pricelist_id.discount_policy == 'without_discount'
            ):
                rule_id = pricelist_item.base_pricelist_id._get_product_rule(*args, **kwargs)
                pricelist_item = self.env['product.pricelist.item'].browse(rule_id)

            pricelist_rule = pricelist_item

        return pricelist_rule._compute_base_price(*args, **kwargs)

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from collections import defaultdict
from operator import itemgetter

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError
from odoo.osv import expression
from odoo.tools import float_compare, groupby
from odoo.tools.misc import unique


class ProductProduct(models.Model):
    _name = "product.product"
    _description = "Product Variant"
    _inherits = {'product.template': 'product_tmpl_id'}
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'priority desc, default_code, name, id'
    _check_company_domain = models.check_company_domain_parent_of

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
        help="""Value of the product (automatically computed in AVCO).
        Used to value the product when the purchase cost is not known (e.g. inventory adjustment).
        Used to compute margins on sale orders.""")
    volume = fields.Float('Volume', digits='Volume')
    weight = fields.Float('Weight', digits='Stock Weight')

    pricelist_item_count = fields.Integer("Number of price rules", compute="_compute_variant_item_count")

    product_document_ids = fields.One2many(
        string="Documents",
        comodel_name='product.document',
        inverse_name='res_id',
        domain=lambda self: [('res_model', '=', self._name)])
    product_document_count = fields.Integer(
        string="Documents Count", compute='_compute_product_document_count')

    packaging_ids = fields.One2many(
        'product.packaging', 'product_id', 'Product Packages',
        help="Gives the different ways to package the same product.")

    additional_product_tag_ids = fields.Many2many(
        string="Additional Product Tags",
        comodel_name='product.tag',
        relation='product_tag_product_product_rel',
        domain="[('id', 'not in', product_tag_ids)]",
    )
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
    write_date = fields.Datetime(compute='_compute_write_date', store=True)

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

    @api.depends("product_tmpl_id.write_date")
    def _compute_write_date(self):
        """
        First, the purpose of this computation is to update a product's
        write_date whenever its template's write_date is updated.  Indeed,
        when a template's image is modified, updating its products'
        write_date will invalidate the browser's cache for the products'
        image, which may be the same as the template's.  This guarantees UI
        consistency.

        Second, the field 'write_date' is automatically updated by the
        framework when the product is modified.  The recomputation of the
        field supplements that behavior to keep the product's write_date
        up-to-date with its template's write_date.

        Third, the framework normally prevents us from updating write_date
        because it is a "magic" field.  However, the assignment inside the
        compute method is not subject to this restriction.  It therefore
        works as intended :-)
        """
        for record in self:
            record.write_date = max(record.write_date or self.env.cr.now(), record.product_tmpl_id.write_date)

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
            return 'product/static/img/placeholder_thumbnail.png'
        return super()._get_placeholder_filename(field)

    def init(self):
        """Ensure there is at most one active variant for each combination.

        There could be no variant for a combination if using dynamic attributes.
        """
        self.env.cr.execute("CREATE UNIQUE INDEX IF NOT EXISTS product_product_combination_unique ON %s (product_tmpl_id, combination_indices) WHERE active is true"
            % self._table)

    def _get_barcodes_by_company(self):
        return [
            (company_id, [p.barcode for p in products if p.barcode])
            for company_id, products in groupby(self, lambda p: p.company_id.id)
        ]

    def _get_barcode_search_domain(self, barcodes_within_company, company_id):
        domain = [('barcode', 'in', barcodes_within_company)]
        if company_id:
            domain.append(('company_id', 'in', (False, company_id)))
        return domain

    def _check_duplicated_product_barcodes(self, barcodes_within_company, company_id):
        domain = self._get_barcode_search_domain(barcodes_within_company, company_id)
        products_by_barcode = self.sudo().read_group(domain, ['barcode', 'id:array_agg'], ['barcode'])

        duplicates_as_str = "\n".join(
            _(
                "- Barcode \"%s\" already assigned to product(s): %s",
                record['barcode'], ", ".join(p.display_name for p in self.search([('id', 'in', record['id'])]))
            )
            for record in products_by_barcode if len(record['id']) > 1
        )
        if duplicates_as_str.strip():
            duplicates_as_str += _(
                "\n\nNote: products that you don't have access to will not be shown above."
            )
            raise ValidationError(_("Barcode(s) already assigned:\n\n%s", duplicates_as_str))

    def _check_duplicated_packaging_barcodes(self, barcodes_within_company, company_id):
        packaging_domain = self._get_barcode_search_domain(barcodes_within_company, company_id)
        if self.env['product.packaging'].sudo().search(packaging_domain, order="id", limit=1):
            raise ValidationError(_("A packaging already uses the barcode"))

    @api.constrains('barcode')
    def _check_barcode_uniqueness(self):
        """ With GS1 nomenclature, products and packagings use the same pattern. Therefore, we need
        to ensure the uniqueness between products' barcodes and packagings' ones"""
        # Barcodes should only be unique within a company
        for company_id, barcodes_within_company in self._get_barcodes_by_company():
            self._check_duplicated_product_barcodes(barcodes_within_company, company_id)
            self._check_duplicated_packaging_barcodes(barcodes_within_company, company_id)

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
            domain = [
                ('pricelist_id.active', '=', True),
                '|',
                    '&', ('product_tmpl_id', '=', product.product_tmpl_id.id), ('applied_on', '=', '1_product'),
                    '&', ('product_id', '=', product.id), ('applied_on', '=', '0_product_variant'),
            ]
            product.pricelist_item_count = self.env['product.pricelist.item'].search_count(domain)

    def _compute_product_document_count(self):
        for product in self:
            product.product_document_count = product.env['product.document'].search_count([
                ('res_model', '=', 'product.product'),
                ('res_id', '=', product.id),
            ])

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
        products = super(ProductProduct, self.with_context(create_product_product=False)).create(vals_list)
        # `_get_variant_id_for_combination` depends on existing variants
        self.env.registry.clear_cache()
        return products

    def write(self, values):
        self.product_tmpl_id._sanitize_vals(values)
        res = super(ProductProduct, self).write(values)
        if 'product_template_attribute_value_ids' in values:
            # `_get_variant_id_for_combination` depends on `product_template_attribute_value_ids`
            self.env.registry.clear_cache()
        elif 'active' in values:
            # `_get_first_possible_variant_id` depends on variants active state
            self.env.registry.clear_cache()
        return res

    def unlink(self):
        unlink_products_ids = set()
        unlink_templates_ids = set()

        # Check if products still exists, in case they've been unlinked by unlinking their template
        existing_products = self.exists()
        product_ids_by_template_id = {template.id: set(ids) for template, ids in self._read_group(
            domain=[('product_tmpl_id', 'in', existing_products.product_tmpl_id.ids)],
            groupby=['product_tmpl_id'],
            aggregates=['id:array_agg'],
        )}
        for product in existing_products:
            # If there is an image set on the variant and no image set on the
            # template, move the image to the template.
            if product.image_variant_1920 and not product.product_tmpl_id.image_1920:
                product.product_tmpl_id.image_1920 = product.image_variant_1920
            # Check if the product is last product of this template...
            has_other_products = product_ids_by_template_id.get(product.product_tmpl_id.id, set()) - {product.id}
            # ... and do not delete product template if it's configured to be created "on demand"
            if not has_other_products and not product.product_tmpl_id.has_dynamic_attributes():
                unlink_templates_ids.add(product.product_tmpl_id.id)
            unlink_products_ids.add(product.id)
        unlink_products = self.env['product.product'].browse(unlink_products_ids)
        res = super(ProductProduct, unlink_products).unlink()
        # delete templates after calling super, as deleting template could lead to deleting
        # products due to ondelete='cascade'
        unlink_templates = self.env['product.template'].browse(unlink_templates_ids)
        unlink_templates.unlink()
        # `_get_variant_id_for_combination` depends on existing variants
        self.env.registry.clear_cache()
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
    def _search(self, domain, offset=0, limit=None, order=None, access_rights_uid=None):
        # TDE FIXME: strange
        if self._context.get('search_default_categ_id'):
            domain = domain.copy()
            domain.append((('categ_id', 'child_of', self._context['search_default_categ_id'])))
        return super()._search(domain, offset, limit, order, access_rights_uid)

    @api.depends('name', 'default_code', 'product_tmpl_id')
    @api.depends_context('display_default_code', 'seller_id', 'company_id', 'partner_id', 'use_partner_name')
    def _compute_display_name(self):

        def get_display_name(name, code):
            if self._context.get('display_default_code', True) and code:
                return f'[{code}] {name}'
            return name

        partner_id = self._context.get('partner_id') if self.env.context.get('use_partner_name', True) else self.env['res.partner']
        if partner_id:
            partner_ids = [partner_id, self.env['res.partner'].browse(partner_id).commercial_partner_id.id]
        else:
            partner_ids = []
        company_id = self.env.context.get('company_id')

        # all user don't have access to seller and partner
        # check access and use superuser
        self.check_access_rights("read")
        self.check_access_rule("read")

        product_template_ids = self.sudo().product_tmpl_id.ids

        if partner_ids:
            # prefetch the fields used by the `display_name`
            supplier_info = self.env['product.supplierinfo'].sudo().search_fetch(
                [('product_tmpl_id', 'in', product_template_ids), ('partner_id', 'in', partner_ids)],
                ['product_tmpl_id', 'product_id', 'company_id', 'product_name', 'product_code'],
            )
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
                temp = []
                for s in sellers:
                    seller_variant = s.product_name and (
                        variant and "%s (%s)" % (s.product_name, variant) or s.product_name
                        ) or False
                    temp.append(get_display_name(seller_variant or name, s.product_code or product.default_code))

                # => Feature drop here, one record can only have one display_name now, instead separate with `,`
                # Remove this comment
                product.display_name = ", ".join(unique(temp))
            else:
                product.display_name = get_display_name(name, product.default_code)

    @api.model
    def _name_search(self, name, domain=None, operator='ilike', limit=None, order=None):
        domain = domain or []
        if name:
            positive_operators = ['=', 'ilike', '=ilike', 'like', '=like']
            product_ids = []
            if operator in positive_operators:
                product_ids = list(self._search([('default_code', '=', name)] + domain, limit=limit, order=order))
                if not product_ids:
                    product_ids = list(self._search([('barcode', '=', name)] + domain, limit=limit, order=order))
            if not product_ids and operator not in expression.NEGATIVE_TERM_OPERATORS:
                # Do not merge the 2 next lines into one single search, SQL search performance would be abysmal
                # on a database with thousands of matching products, due to the huge merge+unique needed for the
                # OR operator (and given the fact that the 'name' lookup results come from the ir.translation table
                # Performing a quick memory merge of ids in Python will give much better performance
                product_ids = list(self._search(domain + [('default_code', operator, name)], limit=limit, order=order))
                if not limit or len(product_ids) < limit:
                    # we may underrun the limit because of dupes in the results, that's fine
                    limit2 = (limit - len(product_ids)) if limit else False
                    product2_ids = self._search(domain + [('name', operator, name), ('id', 'not in', product_ids)], limit=limit2, order=order)
                    product_ids.extend(product2_ids)
            elif not product_ids and operator in expression.NEGATIVE_TERM_OPERATORS:
                domain2 = expression.OR([
                    ['&', ('default_code', operator, name), ('name', operator, name)],
                    ['&', ('default_code', '=', False), ('name', operator, name)],
                ])
                domain2 = expression.AND([domain, domain2])
                product_ids = list(self._search(domain2, limit=limit, order=order))
            if not product_ids and operator in positive_operators:
                ptrn = re.compile(r'(\[(.*?)\])')
                res = ptrn.search(name)
                if res:
                    product_ids = list(self._search([('default_code', '=', res.group(2))] + domain, limit=limit, order=order))
            # still no results, partner in context: search on supplier info as last hope to find something
            if not product_ids and self._context.get('partner_id'):
                suppliers_ids = self.env['product.supplierinfo']._search([
                    ('partner_id', '=', self._context.get('partner_id')),
                    '|',
                    ('product_code', operator, name),
                    ('product_name', operator, name)])
                if suppliers_ids:
                    product_ids = self._search([('product_tmpl_id.seller_ids', 'in', suppliers_ids)], limit=limit, order=order)
        else:
            product_ids = self._search(domain, limit=limit, order=order)
        return product_ids

    @api.model
    def view_header_get(self, view_id, view_type):
        if self._context.get('categ_id'):
            return _(
                'Products: %(category)s',
                category=self.env['product.category'].browse(self.env.context['categ_id']).name,
            )
        return super().view_header_get(view_id, view_type)

    #=== ACTION METHODS ===#

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
                'search_default_visible': True,
            }
        }

    def open_product_template(self):
        """ Utility method used to add an "Open Template" button in product views """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'product.template',
            'view_mode': 'form',
            'res_id': self.product_tmpl_id.id,
            'target': 'new'
        }

    def action_open_documents(self):
        res = self.product_tmpl_id.action_open_documents()
        res['context'].update({
            'default_res_model': self._name,
            'default_res_id': self.id,
            'search_default_context_variant': True,
        })
        return res

    #=== BUSINESS METHODS ===#

    def _prepare_sellers(self, params=False):
        sellers = self.seller_ids._get_filtered_supplier(self.env.company, self, params)
        return sellers.sorted(lambda s: (s.sequence, -s.min_qty, s.price, s.id))

    def _get_filtered_sellers(self, partner_id=False, quantity=0.0, date=None, uom_id=False, params=False):
        self.ensure_one()
        if date is None:
            date = fields.Date.context_today(self)
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')

        sellers_filtered = self._prepare_sellers(params)
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

    def _select_seller(self, partner_id=False, quantity=0.0, date=None, uom_id=False, ordered_by='price_discounted', params=False):
        # Always sort by discounted price but another field can take the primacy through the `ordered_by` param.
        sort_key = ('price_discounted', 'sequence', 'id')
        if ordered_by != 'price_discounted':
            sort_key = (ordered_by, 'price_discounted', 'sequence', 'id')

        def sort_function(record):
            vals = {
                'price_discounted': record.currency_id._convert(record.price_discounted, record.env.company.currency_id, record.env.company, date or fields.Date.context_today(self))
            }
            return [vals.get(key, record[key]) for key in sort_key]
        sellers = self._get_filtered_sellers(partner_id=partner_id, quantity=quantity, date=date, uom_id=uom_id, params=params)
        res = self.env['product.supplierinfo']
        for seller in sellers:
            if not res or res.partner_id == seller.partner_id:
                res |= seller
        return res and res.sorted(sort_function)[:1]

    def _get_product_price_context(self, combination):
        self.ensure_one()
        res = {}

        # It is possible that a no_variant attribute is still in a variant if
        # the type of the attribute has been changed after creation.
        no_variant_attributes_price_extra = [
            ptav.price_extra for ptav in combination.filtered(
                lambda ptav:
                    ptav.price_extra
                    and ptav.product_tmpl_id == self.product_tmpl_id
                    and ptav not in self.product_template_attribute_value_ids
            )
        ]
        if no_variant_attributes_price_extra:
            res['no_variant_attributes_price_extra'] = tuple(no_variant_attributes_price_extra)

        return res

    def _get_attributes_extra_price(self):
        self.ensure_one()

        return self.price_extra + sum(
            self.env.context.get('no_variant_attributes_price_extra', []))

    def _price_compute(self, price_type, uom=None, currency=None, company=None, date=False):
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
            elif price_type == 'list_price':
                price += product._get_attributes_extra_price()

            if uom:
                price = product.uom_id._compute_price(price, uom)

            # Convert from current user company currency to asked one
            # This is right cause a field cannot be in more than one currency
            if currency:
                price = price_currency._convert(price, currency, company, date)

            prices[product.id] = price

        return prices

    @api.model
    def get_empty_list_help(self, help_message):
        self = self.with_context(
            empty_list_help_document_name=_("product"),
        )
        return super(ProductProduct, self).get_empty_list_help(help_message)

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

    def _get_contextual_discount(self):
        self.ensure_one()

        pricelist = self.product_tmpl_id._get_contextual_pricelist()
        if not pricelist:
            # No pricelist = no discount
            return 0.0

        lst_price = self.currency_id._convert(
            self.lst_price,
            pricelist.currency_id,
            self.env.company,
            fields.Datetime.now(),
            round=False
        )
        if lst_price:
            return (lst_price - self._get_contextual_price()) / lst_price
        return 0.0

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
    price_discounted = fields.Float('Discounted Price', compute='_compute_price_discounted')
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
        domain="[('product_tmpl_id', '=', product_tmpl_id)] if product_tmpl_id else []",
        default=_default_product_id,
        help="If not set, the vendor price will apply to all variants of this product.")
    product_tmpl_id = fields.Many2one(
        'product.template', 'Product Template', check_company=True,
        index=True, ondelete='cascade')
    product_variant_count = fields.Integer('Variant Count', related='product_tmpl_id.product_variant_count')
    delay = fields.Integer(
        'Delivery Lead Time', default=1, required=True,
        help="Lead time in days between the confirmation of the purchase order and the receipt of the products in your warehouse. Used by the scheduler for automatic computation of the purchase order planning.")
    discount = fields.Float(
        string="Discount (%)",
        digits='Discount',
        readonly=False)

    @api.depends('discount', 'price')
    def _compute_price_discounted(self):
        for rec in self:
            rec.price_discounted = rec.price * (1 - rec.discount / 100)

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

    def _get_filtered_supplier(self, company_id, product_id, params=False):
        return self.filtered(lambda s: (not s.company_id or s.company_id.id == company_id.id) and (s.partner_id.active and (not s.product_id or s.product_id == product_id)))

```

## File: models\product_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.osv import expression

class ProductTag(models.Model):
    _name = 'product.tag'
    _description = 'Product Tag'

    def _get_default_template_id(self):
        return self.env['product.template'].browse(self.env.context.get('product_template_id'))

    def _get_default_variant_id(self):
        return self.env['product.product'].browse(self.env.context.get('product_variant_id'))

    name = fields.Char(string="Name", required=True, translate=True)
    color = fields.Char(string="Color", default='#3C3C3C')
    product_template_ids = fields.Many2many(
        string="Product Templates",
        comodel_name='product.template',
        relation='product_tag_product_template_rel',
        default=_get_default_template_id,
    )
    product_product_ids = fields.Many2many(
        string="Product Variants",
        comodel_name='product.product',
        relation='product_tag_product_product_rel',
        domain="[('attribute_line_ids', '!=', False), ('product_tmpl_id', 'not in', product_template_ids)]",
        default=_get_default_variant_id,
    )
    product_ids = fields.Many2many(
        'product.product', string='All Product Variants using this Tag',
        compute='_compute_product_ids', search='_search_product_ids'
    )

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

    @api.depends('product_template_ids', 'product_product_ids')
    def _compute_product_ids(self):
        for tag in self:
            tag.product_ids = tag.product_template_ids.product_variant_ids | tag.product_product_ids

    @api.returns(None, lambda value: value[0])
    def copy_data(self, default=None):
        default = default or {}
        if not default.get('name'):
            default['name'] = _('%s (copy)', self.name)
        return super().copy_data(default=default)

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
from odoo.exceptions import UserError, ValidationError
from odoo.models import PREFETCH_MAX
from odoo.osv import expression

_logger = logging.getLogger(__name__)
PRICE_CONTEXT_KEYS = ['pricelist', 'quantity', 'uom', 'date']


class ProductTemplate(models.Model):
    _name = "product.template"
    _inherit = ['mail.thread', 'mail.activity.mixin', 'image.mixin']
    _description = "Product"
    _order = "priority desc, name"
    _check_company_domain = models.check_company_domain_parent_of

    @tools.ormcache()
    def _get_default_category_id(self):
        # Deletion forbidden (at least through unlink)
        return self.env.ref('product.product_category_all')

    @tools.ormcache()
    def _get_default_uom_id(self):
        # Deletion forbidden (at least through unlink)
        return self.env.ref('uom.product_uom_unit')

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
        help="""Value of the product (automatically computed in AVCO).
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
        compute='_compute_uom_po_id', required=True, readonly=False, store=True, precompute=True,
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

    product_document_ids = fields.One2many(
        string="Documents",
        comodel_name='product.document',
        inverse_name='res_id',
        domain=lambda self: [('res_model', '=', self._name)])
    product_document_count = fields.Integer(
        string="Documents Count", compute='_compute_product_document_count')

    can_image_1024_be_zoomed = fields.Boolean("Can Image 1024 be zoomed", compute='_compute_can_image_1024_be_zoomed', store=True)
    has_configurable_attributes = fields.Boolean("Is a configurable product", compute='_compute_has_configurable_attributes', store=True)

    product_tooltip = fields.Char(compute='_compute_product_tooltip')

    priority = fields.Selection([
        ('0', 'Normal'),
        ('1', 'Favorite'),
    ], default='0', string="Favorite")

    product_tag_ids = fields.Many2many(
        string="Product Template Tags",
        comodel_name='product.tag',
        relation='product_tag_product_template_rel',
    )
    # Properties
    product_properties = fields.Properties('Properties', definition='categ_id.product_properties_definition', copy=True)

    @api.depends('uom_id')
    def _compute_uom_po_id(self):
        for template in self:
            if not template.uom_po_id or template.uom_id.category_id != template.uom_po_id.category_id:
                template.uom_po_id = template.uom_id

    def _compute_item_count(self):
        for template in self:
            # Pricelist item count counts the rules applicable on current template or on its variants.
            template.pricelist_item_count = template.env['product.pricelist.item'].search_count([
                '&',
                '|', ('product_tmpl_id', '=', template.id), ('product_id', 'in', template.product_variant_ids.ids),
                ('pricelist_id.active', '=', True),
            ])

    def _compute_product_document_count(self):
        for template in self:
            template.product_document_count = template.env['product.document'].search_count([
                '|',
                    '&', ('res_model', '=', 'product.template'), ('res_id', '=', template.id),
                    '&',
                        ('res_model', '=', 'product.product'),
                        ('res_id', 'in', template.product_variant_ids.ids),
            ])

    @api.depends('image_1920', 'image_1024')
    def _compute_can_image_1024_be_zoomed(self):
        for template in self.with_context(bin_size=False):
            template.can_image_1024_be_zoomed = template.image_1920 and tools.is_image_size_above(template.image_1920, template.image_1024)

    @api.depends(
        'attribute_line_ids',
        'attribute_line_ids.value_ids',
        'attribute_line_ids.attribute_id.create_variant',
        'attribute_line_ids.attribute_id.display_type',
        'attribute_line_ids.value_ids.is_custom',
    )
    def _compute_has_configurable_attributes(self):
        """A product is considered configurable if:
        - It has dynamic attributes
        - It has any attribute line with at least 2 attribute values configured
        - It has multi-checkbox display type
        - It has at least one custom attribute value
        """
        for product in self:
            product.has_configurable_attributes = (
                product.has_dynamic_attributes() or any(
                    ptal.attribute_id.display_type == 'multi'
                    or len(ptal.value_ids) >= 2
                    or ptal.value_ids.is_custom
                    for ptal in product.attribute_line_ids
                )
            )

    @api.depends('product_variant_ids')
    def _compute_product_variant_id(self):
        for p in self:
            p.product_variant_id = p.product_variant_ids[:1].id

    @api.constrains('company_id')
    def _check_barcode_uniqueness(self):
        for template in self:
            template.product_variant_ids._check_barcode_uniqueness()

    @api.depends('company_id')
    def _compute_currency_id(self):
        main_company = self.env['res.company']._get_main_company()
        for template in self:
            template.currency_id = template.company_id.sudo().currency_id.id or main_company.currency_id.id

    @api.depends('company_id')
    @api.depends_context('company')
    def _compute_cost_currency_id(self):
        env_currency_id = self.env.company.currency_id.id
        for template in self:
            template.cost_currency_id = template.company_id.currency_id.id or env_currency_id

    def _compute_template_field_from_variant_field(self, fname, default=False):
        """Sets the value of the given field based on the template variant values

        Equals to product_variant_ids[fname] if it's a single variant product.
        Otherwise, sets the value specified in ``default``.
        It's used to compute fields like barcode, weight, volume..

        :param str fname: name of the field to compute
            (field name must be identical between product.product & product.template models)
        :param default: default value to set when there are multiple or no variants on the template
        :return: None
        """
        for template in self:
            variant_count = len(template.product_variant_ids)
            if variant_count == 1:
                template[fname] = template.product_variant_ids[fname]
            elif variant_count == 0 and self.env.context.get("active_test", True):
                # If the product has no active variants, retry without the active_test
                template_ctx = template.with_context(active_test=False)
                template_ctx._compute_template_field_from_variant_field(fname, default=default)
            else:
                template[fname] = default

    def _set_product_variant_field(self, fname):
        """Propagate the value of the given field from the templates to their unique variant.

        Only if it's a single variant product.
        It's used to set fields like barcode, weight, volume..

        :param str fname: name of the field whose value should be propagated to the variant.
            (field name must be identical between product.product & product.template models)
        """
        for template in self:
            count = len(template.product_variant_ids)
            if count == 1:
                template.product_variant_ids[fname] = template[fname]
            elif count == 0:
                archived_variants = self.with_context(active_test=False).product_variant_ids
                if len(archived_variants) == 1:
                    archived_variants[fname] = template[fname]

    @api.depends_context('company')
    @api.depends('product_variant_ids.standard_price')
    def _compute_standard_price(self):
        # Depends on force_company context because standard_price is company_dependent
        # on the product_product
        self._compute_template_field_from_variant_field('standard_price')

    def _set_standard_price(self):
        self._set_product_variant_field('standard_price')

    def _search_standard_price(self, operator, value):
        return [('product_variant_ids.standard_price', operator, value)]

    @api.depends('product_variant_ids.volume')
    def _compute_volume(self):
        self._compute_template_field_from_variant_field('volume')

    def _set_volume(self):
        self._set_product_variant_field('volume')

    @api.depends('product_variant_ids.weight')
    def _compute_weight(self):
        self._compute_template_field_from_variant_field('weight')

    def _set_weight(self):
        self._set_product_variant_field('weight')

    def _compute_is_product_variant(self):
        self.is_product_variant = False

    @api.depends('product_variant_ids.barcode')
    def _compute_barcode(self):
        self._compute_template_field_from_variant_field('barcode')

    def _search_barcode(self, operator, value):
        subquery = self.with_context(active_test=False)._search([
            ('product_variant_ids.barcode', operator, value),
        ])
        return [('id', 'in', subquery)]

    def _set_barcode(self):
        self._set_product_variant_field('barcode')

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

    @api.depends('type')
    def _compute_weight_uom_name(self):
        self.weight_uom_name = self._get_weight_uom_name_from_ir_config_parameter()

    @api.depends('type')
    def _compute_volume_uom_name(self):
        self.volume_uom_name = self._get_volume_uom_name_from_ir_config_parameter()

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

    @api.depends('product_variant_ids.default_code')
    def _compute_default_code(self):
        self._compute_template_field_from_variant_field('default_code')

    def _set_default_code(self):
        self._set_product_variant_field('default_code')

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
        return ['barcode', 'default_code', 'standard_price', 'volume', 'weight', 'packaging_ids', 'product_properties']

    @api.model_create_multi
    def create(self, vals_list):
        ''' Store the initial standard price in order to be able to retrieve the cost of a product template for a given date'''
        for vals in vals_list:
            self._sanitize_vals(vals)
        templates = super(ProductTemplate, self).create(vals_list)
        if self._context.get("create_product_product", True):
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
        res = super(ProductTemplate, self).write(vals)
        if self._context.get("create_product_product", True) and 'attribute_line_ids' in vals or (vals.get('active') and len(self.product_variant_ids) == 0):
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

    @api.depends('name', 'default_code')
    def _compute_display_name(self):
        for template in self:
            template.display_name = False if not template.name else (
                '{}{}'.format(
                    template.default_code and '[%s] ' % template.default_code or '', template.name
                ))

    @api.model
    def _name_search(self, name, domain=None, operator='ilike', limit=None, order=None):
        # Only use the product.product heuristics if there is a search term and the domain
        # does not specify a match on `product.template` IDs.
        domain = domain or []
        if not name or any(term[0] == 'id' for term in domain):
            return super()._name_search(name, domain, operator, limit, order)

        Product = self.env['product.product']
        templates = self.browse()
        while True:
            extra = templates and [('product_tmpl_id', 'not in', templates.ids)] or []
            # Pathological case: there is no limit, so we'll need to search on all products.
            # We iteratively _name_search with a larger bound, but not unbounded to avoid
            # performance regressions or OOM errors while manipulating extremely large list of ids.
            # For other cases, we use PREFETCH_MAX as an upper bound.
            search_limit = PREFETCH_MAX * 10 if not limit else PREFETCH_MAX
            products_ids = Product._name_search(name, domain + extra, operator, limit=search_limit)
            products = Product.browse(products_ids)
            new_templates = products.product_tmpl_id
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
            domain2 = expression.AND([domain, [('id', 'in', tmpl_without_variant_ids)]])
            searched_ids |= set(super()._name_search(name, domain2, operator, limit, order))

        # re-apply product.template order + display_name
        domain = [('id', 'in', list(searched_ids))]
        return super()._name_search('', domain, 'ilike', limit, order)

    #=== ACTION METHODS ===#

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
                'search_default_visible': True,
            },
        }

    def action_open_documents(self):
        self.ensure_one()
        return {
            'name': _('Documents'),
            'type': 'ir.actions.act_window',
            'res_model': 'product.document',
            'view_mode': 'kanban,tree,form',
            'context': {
                'default_res_model': self._name,
                'default_res_id': self.id,
                'default_company_id': self.company_id.id,
            },
            'domain': [
                '|',
                    '&', ('res_model', '=', 'product.template'), ('res_id', '=', self.id),
                    '&',
                        ('res_model', '=', 'product.product'),
                        ('res_id', 'in', self.product_variant_ids.ids),
            ],
            'target': 'current',
            'help': """
                <p class="o_view_nocontent_smiling_face">
                    %s
                </p>
                <p>
                    %s
                    <br/>
                    %s
                </p>
                <p>
                    <a class="oe_link" href="https://www.odoo.com/documentation/17.0/_downloads/5f0840ed187116c425fdac2ab4b592e1/pdfquotebuilderexamples.zip">
                    %s
                    </a>
                </p>
            """ % (
                _("Upload files to your product"),
                _("Use this feature to store any files you would like to share with your customers"),
                _("(e.g: product description, ebook, legal notice, ...)."),
                _("Download examples")
            )
        }

    #=== BUSINESS METHODS ===#

    def _get_product_price_context(self, combination):
        self.ensure_one()
        res = {}

        current_attributes_price_extra = [
            ptav.price_extra for ptav in combination.filtered(
                lambda ptav:
                    ptav.price_extra
                    and ptav.product_tmpl_id == self
            )
        ]
        if current_attributes_price_extra:
            res['current_attributes_price_extra'] = tuple(current_attributes_price_extra)

        return res

    def _get_attributes_extra_price(self):
        self.ensure_one()

        return sum(self.env.context.get('current_attributes_price_extra', []))

    def _price_compute(self, price_type, uom=None, currency=None, company=None, date=False):
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
            elif price_type == 'list_price':
                price += template._get_attributes_extra_price()

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
                for combination in tmpl_id._filter_combinations_impossible_by_config(
                    all_combinations, ignore_no_variant=True,
                ):
                    if combination in existing_variants:
                        current_variants_to_activate += existing_variants[combination]
                    else:
                        current_variants_to_create.append(tmpl_id._prepare_variant_values(combination))
                        variant_limit = self.env['ir.config_parameter'].sudo().get_param('product.dynamic_variant_limit', 1000)
                        if len(current_variants_to_create) > int(variant_limit):
                            raise UserError(_(
                                'The number of variants to generate is above allowed limit. '
                                'You should either not generate variants for each combination or generate them on demand from the sales order. '
                                'To do so, open the form view of attributes and change the mode of *Create Variants*.'))
                variants_to_create += current_variants_to_create
                variants_to_activate += current_variants_to_activate

            elif existing_variants:
                variants_combinations = [variant.product_template_attribute_value_ids for variant in existing_variants.values()]
                current_variants_to_activate += Product.concat(*[existing_variants[possible_combination]
                    for possible_combination in tmpl_id._filter_combinations_impossible_by_config(variants_combinations, ignore_no_variant=True)
                ])
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

    def _get_attribute_exclusions(
        self, parent_combination=None, parent_name=None, combination_ids=None
    ):
        """Return the list of attribute exclusions of a product.

        :param parent_combination: the combination from which
            `self` is an optional or accessory product. Indeed exclusions
            rules on one product can concern another product.
        :type parent_combination: recordset `product.template.attribute.value`
        :param parent_name: the name of the parent product combination.
        :type parent_name: str
        :param list combination: The combination of the product, as a
            list of `product.template.attribute.value` ids.

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
            'exclusions': self._complete_inverse_exclusions(
                self._get_own_attribute_exclusions(combination_ids=combination_ids)
            ),
            'archived_combinations': list(set(
                tuple(product.product_template_attribute_value_ids.ids)
                for product in archived_products
                if product.product_template_attribute_value_ids and all(
                    ptav.ptav_active or combination_ids and ptav.id in combination_ids
                    for ptav in product.product_template_attribute_value_ids
                )
            ) - active_combinations),
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

    def _get_own_attribute_exclusions(self, combination_ids=None):
        """Get exclusions coming from the current template.

        :param list combination: The combination of the product, as a
            list of `product.template.attribute.value` ids.
        Dictionnary, each product template attribute value is a key, and for each of them
        the value is an array with the other ptav that they exclude (empty if no exclusion).
        """
        self.ensure_one()
        product_template_attribute_values = self.valid_product_template_attribute_line_ids.product_template_value_ids
        return {
            ptav.id: [
                value.id
                for filter_line in ptav.exclude_for.filtered(
                    lambda filter_line: filter_line.product_tmpl_id == self
                ) for value in filter_line.value_ids if value.ptav_active
            ]
            for ptav in product_template_attribute_values if (
                ptav.ptav_active or combination_ids and ptav.id in combination_ids
            )
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

    def _filter_combinations_impossible_by_config(self, combination_tuples, ignore_no_variant=False):
        """ Filter combination_tuples according to the config of attributes on the template

        :return: iterator over possible combinations
        :rtype: generator
        """
        self.ensure_one()
        attribute_lines = self.valid_product_template_attribute_line_ids
        attribute_lines_active_values = attribute_lines.product_template_value_ids._only_active()
        if ignore_no_variant:
            attribute_lines = attribute_lines._without_no_variant_attributes()
        attribute_lines_without_multi = attribute_lines.filtered(
            lambda l: l.attribute_id.display_type != 'multi')
        exclusions = self._get_own_attribute_exclusions()
        for combination_tuple in combination_tuples:
            combination = self.env['product.template.attribute.value'].concat(*combination_tuple)
            combination_without_multi = combination.filtered(
                lambda l: l.attribute_line_id.attribute_id.display_type != 'multi')
            if len(combination_without_multi) != len(attribute_lines_without_multi):
                # number of attribute values passed is different than the
                # configuration of attributes on the template
                continue
            if attribute_lines_without_multi != combination_without_multi.attribute_line_id:
                # combination has different attributes than the ones configured on the template
                continue
            if not (attribute_lines_active_values >= combination):
                # combination has different values than the ones configured on the template
                continue
            if exclusions:
                # exclude if the current value is in an exclusion,
                # and the value excluding it is also in the combination
                combination_ids = set(combination.ids)
                combination_excluded_ids = set(itertools.chain(*[exclusions.get(ptav_id) for ptav_id in combination.ids]))
                if combination_ids & combination_excluded_ids:
                    continue
            yield combination

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
        # Returns False on StopIteration. Empty combination should return True.
        return isinstance(next(self._filter_combinations_impossible_by_config([combination], ignore_no_variant), False), models.BaseModel)

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

            # For multi-checkbox attribute, the list is empty as we want to start without any selected value
            if not current_line_values:
                if line_index == len(product_template_attribute_values_per_line) - 1:
                    # submit combination if we're on the last line
                    yield partial_combination
                else:
                    line_index += 1
                    continue

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
        attribute_lines = self.valid_product_template_attribute_line_ids.filtered(
            lambda ptal: ptal not in necessary_attribute_lines)

        if not attribute_lines and self._is_combination_possible(necessary_values, parent_combination):
            yield necessary_values

        product_template_attribute_values_per_line = []
        for ptal in attribute_lines:
            if ptal.attribute_id.display_type != 'multi':
                values_to_add = ptal.product_template_value_ids._only_active()
            else:
                values_to_add = self.env['product.template.attribute.value']
            product_template_attribute_values_per_line.append(values_to_add)

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

    def _get_placeholder_filename(self, field):
        image_fields = ['image_%s' % size for size in [1920, 1024, 512, 256, 128]]
        if field in image_fields:
            return 'product/static/img/placeholder_thumbnail.png'
        return super()._get_placeholder_filename(field)

    def get_single_product_variant(self):
        """ Method used by the product configurator to check if the product is configurable or not.

        We need to open the product configurator if the product:
        - is configurable (see has_configurable_attributes)
        - has optional products (method is extended in sale to return optional products info)

        Note: self.ensure_one()
        """
        self.ensure_one()
        if self.product_variant_count == 1 and not self.has_configurable_attributes:
            return {
                'product_id': self.product_variant_id.id,
                'product_name': self.product_variant_id.display_name,
            }
        return {}

    @api.model
    def get_empty_list_help(self, help_message):
        self = self.with_context(
            empty_list_help_document_name=_("product"),
        )
        return super(ProductTemplate, self).get_empty_list_help(help_message)

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
        pricelist = self._get_contextual_pricelist()
        quantity = self.env.context.get('quantity', 1.0)
        uom = self.env['uom.uom'].browse(self.env.context.get('uom'))
        date = self.env.context.get('date')
        return pricelist._get_product_price(product or self, quantity, uom=uom, date=date)

    def _get_contextual_pricelist(self):
        """ Get the contextual pricelist

        This method is meant to be overriden in other standard modules.
        """
        return self.env['product.pricelist'].browse(self.env.context.get('pricelist'))

```

## File: models\product_template_attribute_exclusion.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProductTemplateAttributeExclusion(models.Model):
    _name = 'product.template.attribute.exclusion'
    _description = "Product Template Attribute Exclusion"
    _order = 'product_tmpl_id, id'

    product_template_attribute_value_id = fields.Many2one(
        comodel_name='product.template.attribute.value',
        string="Attribute Value",
        ondelete='cascade',
        index=True)
    product_tmpl_id = fields.Many2one(
        comodel_name='product.template',
        string="Product Template",
        ondelete='cascade',
        required=True,
        index=True)
    value_ids = fields.Many2many(
        comodel_name='product.template.attribute.value',
        relation='product_attr_exclusion_value_ids_rel',
        string="Attribute Values",
        domain="[('product_tmpl_id', '=', product_tmpl_id), ('ptav_active', '=', True)]")

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

```

## File: models\product_template_attribute_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError, ValidationError
from odoo.fields import Command


class ProductTemplateAttributeLine(models.Model):
    """Attributes available on product.template with their selected values in a m2m.
    Used as a configuration model to generate the appropriate product.template.attribute.value"""

    _name = 'product.template.attribute.line'
    _rec_name = 'attribute_id'
    _rec_names_search = ['attribute_id', 'value_ids']
    _description = "Product Template Attribute Line"
    _order = 'sequence, attribute_id, id'

    active = fields.Boolean(default=True)
    product_tmpl_id = fields.Many2one(
        comodel_name='product.template',
        string="Product Template",
        ondelete='cascade',
        required=True,
        index=True)
    sequence = fields.Integer("Sequence", default=10)
    attribute_id = fields.Many2one(
        comodel_name='product.attribute',
        string="Attribute",
        ondelete='restrict',
        required=True,
        index=True)
    value_ids = fields.Many2many(
        comodel_name='product.attribute.value',
        relation='product_attribute_value_product_template_attribute_line_rel',
        string="Values",
        domain="[('attribute_id', '=', attribute_id)]",
        ondelete='restrict')
    value_count = fields.Integer(compute='_compute_value_count', store=True)
    product_template_value_ids = fields.One2many(
        comodel_name='product.template.attribute.value',
        inverse_name='attribute_line_id',
        string="Product Attribute Values")

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
                raise ValidationError(_(
                    "The attribute %(attribute)s must have at least one value for the product %(product)s.",
                    attribute=ptal.attribute_id.display_name,
                    product=ptal.product_tmpl_id.display_name,
                ))
            for pav in ptal.value_ids:
                if pav.attribute_id != ptal.attribute_id:
                    raise ValidationError(_(
                        "On the product %(product)s you cannot associate the value %(value)s"
                        " with the attribute %(attribute)s because they do not match.",
                        product=ptal.product_tmpl_id.display_name,
                        value=pav.display_name,
                        attribute=ptal.attribute_id.display_name,
                    ))
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
        res = activated_lines + super().create(create_values)
        if self._context.get("create_product_product", True):
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
                    raise UserError(_(
                        "You cannot move the attribute %(attribute)s from the product"
                        " %(product_src)s to the product %(product_dest)s.",
                        attribute=ptal.attribute_id.display_name,
                        product_src=ptal.product_tmpl_id.display_name,
                        product_dest=values['product_tmpl_id'],
                    ))

        if 'attribute_id' in values:
            for ptal in self:
                if ptal.attribute_id.id != values['attribute_id']:
                    raise UserError(_(
                        "On the product %(product)s you cannot transform the attribute"
                        " %(attribute_src)s into the attribute %(attribute_dest)s.",
                        product=ptal.product_tmpl_id.display_name,
                        attribute_src=ptal.attribute_id.display_name,
                        attribute_dest=values['attribute_id'],
                    ))
        # Remove all values while archiving to make sure the line is clean if it
        # is ever activated again.
        if not values.get('active', True):
            values['value_ids'] = [Command.clear()]
        res = super().write(values)
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
                        'attribute_line_id': ptal.id,
                        'price_extra': pav.default_extra_price,
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

```

## File: models\product_template_attribute_value.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError, ValidationError
from odoo.fields import Command


class ProductTemplateAttributeValue(models.Model):
    """Materialized relationship between attribute values
    and product template generated by the product.template.attribute.line"""

    _name = 'product.template.attribute.value'
    _description = "Product Template Attribute Value"
    _order = 'attribute_line_id, product_attribute_value_id, id'

    def _get_default_color(self):
        return randint(1, 11)

    # Not just `active` because we always want to show the values except in
    # specific case, as opposed to `active_test`.
    ptav_active = fields.Boolean(string="Active", default=True)
    name = fields.Char(string="Value", related="product_attribute_value_id.name")

    # defining fields: the product template attribute line and the product attribute value
    product_attribute_value_id = fields.Many2one(
        comodel_name='product.attribute.value',
        string="Attribute Value",
        required=True, ondelete='cascade', index=True)
    attribute_line_id = fields.Many2one(
        comodel_name='product.template.attribute.line',
        required=True, ondelete='cascade', index=True)
    # configuration fields: the price_extra and the exclusion rules
    price_extra = fields.Float(
        string="Value Price Extra",
        default=0.0,
        digits='Product Price',
        help="Extra price for the variant with this attribute value on sale price."
            " eg. 200 price extra, 1000 + 200 = 1200.")
    currency_id = fields.Many2one(related='attribute_line_id.product_tmpl_id.currency_id')

    exclude_for = fields.One2many(
        comodel_name='product.template.attribute.exclusion',
        inverse_name='product_template_attribute_value_id',
        string="Exclude for",
        help="Make this attribute value not compatible with "
             "other values of the product or some attribute values of optional and accessory products.")

    # related fields: product template and product attribute
    product_tmpl_id = fields.Many2one(
        related='attribute_line_id.product_tmpl_id', store=True, index=True)
    attribute_id = fields.Many2one(
        related='attribute_line_id.attribute_id', store=True, index=True)
    ptav_product_variant_ids = fields.Many2many(
        comodel_name='product.product', relation='product_variant_combination',
        string="Related Variants", readonly=True)

    html_color = fields.Char(string="HTML Color Index", related='product_attribute_value_id.html_color')
    is_custom = fields.Boolean(related='product_attribute_value_id.is_custom')
    display_type = fields.Selection(related='product_attribute_value_id.display_type')
    color = fields.Integer(string="Color", default=_get_default_color)
    image = fields.Image(related='product_attribute_value_id.image')

    _sql_constraints = [
        ('attribute_value_unique',
         'unique(attribute_line_id, product_attribute_value_id)',
         "Each value should be defined only once per attribute per product."),
    ]

    @api.constrains('attribute_line_id', 'product_attribute_value_id')
    def _check_valid_values(self):
        for ptav in self:
            if ptav.ptav_active and ptav.product_attribute_value_id not in ptav.attribute_line_id.value_ids:
                raise ValidationError(_(
                    "The value %(value)s is not defined for the attribute %(attribute)s"
                    " on the product %(product)s.",
                    value=ptav.product_attribute_value_id.display_name,
                    attribute=ptav.attribute_id.display_name,
                    product=ptav.product_tmpl_id.display_name,
                ))

    @api.model_create_multi
    def create(self, vals_list):
        if any('ptav_product_variant_ids' in v for v in vals_list):
            # Force write on this relation from `product.product` to properly
            # trigger `_compute_combination_indices`.
            raise UserError(_("You cannot update related variants from the values. Please update related values from the variants."))
        return super().create(vals_list)

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
                    raise UserError(_(
                        "You cannot change the value of the value %(value)s set on product %(product)s.",
                        value=ptav.display_name,
                        product=ptav.product_tmpl_id.display_name,
                    ))
                if product_in_values and ptav.product_tmpl_id.id != values['product_tmpl_id']:
                    raise UserError(_(
                        "You cannot change the product of the value %(value)s set on product %(product)s.",
                        value=ptav.display_name,
                        product=ptav.product_tmpl_id.display_name,
                    ))
        res = super().write(values)
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
            ptav.ptav_product_variant_ids.write({
                'product_template_attribute_value_ids': [Command.unlink(ptav.id)],
            })
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

    @api.depends('attribute_id')
    def _compute_display_name(self):
        """Override because in general the name of the value is confusing if it
        is displayed without the name of the corresponding attribute.
        Eg. on exclusion rules form
        """
        for value in self:
            value.display_name = f"{value.attribute_id.name}: {value.name}"

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

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models


class ResCompany(models.Model):
    _inherit = "res.company"

    @api.model_create_multi
    def create(self, vals_list):
        companies = super().create(vals_list)
        companies._activate_or_create_pricelists()
        return companies

    def _activate_or_create_pricelists(self):
        """ Manage the default pricelists for needed companies. """
        if self.env.context.get('disable_company_pricelist_creation'):
            return

        if self.user_has_groups('product.group_product_pricelist'):
            companies = self or self.env['res.company'].search([])
            ProductPricelist = self.env['product.pricelist'].sudo()
            # Activate existing default pricelists
            default_pricelists_sudo = ProductPricelist.with_context(active_test=False).search(
                [('item_ids', '=', False), ('company_id', 'in', companies.ids)]
            ).filtered(lambda pl: pl.currency_id == pl.company_id.currency_id)
            default_pricelists_sudo.action_unarchive()
            companies_without_pricelist = companies.filtered(
                lambda c: c.id not in default_pricelists_sudo.company_id.ids
            )
            # Create missing default pricelists
            ProductPricelist.create([
                company._get_default_pricelist_vals() for company in companies_without_pricelist
            ])

    def _get_default_pricelist_vals(self):
        """Add values to the default pricelist at company creation or activation of the pricelist

        Note: self.ensure_one()

        :rtype: dict
        """
        self.ensure_one()
        values = {}
        values.update({
            'name': _("Default %s pricelist", self.currency_id.name),
            'currency_id': self.currency_id.id,
            'company_id': self.id,
            'sequence': 10,
        })
        return values

    def write(self, vals):
        """Delay the automatic creation of pricelists post-company update.

        This makes sure that the pricelist(s) automatically created are created with the right
        currency.
        """
        if not vals.get('currency_id'):
            return super().write(vals)

        enabled_pricelists = self.user_has_groups('product.group_product_pricelist')
        res = super(
            ResCompany, self.with_context(disable_company_pricelist_creation=True)
        ).write(vals)
        if not enabled_pricelists and self.user_has_groups('product.group_product_pricelist'):
            self.browse()._activate_or_create_pricelists()

        return res

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


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
        if not self.group_product_pricelist:
            if self.group_sale_pricelist:
                self.group_sale_pricelist = False
            active_pricelist = self.env['product.pricelist'].sudo().search_count(
                [('active', '=', True)], limit=1
            )
            if active_pricelist:
                return {
                    'warning': {
                    'message': _("You are deactivating the pricelist feature. "
                                 "Every active pricelist will be archived.")
                }}

    @api.onchange('product_pricelist_setting')
    def _onchange_product_pricelist_setting(self):
        if self.product_pricelist_setting == 'basic':
            self.group_sale_pricelist = False
        else:
            self.group_sale_pricelist = True

    def set_values(self):
        had_group_pl = self.default_get(['group_product_pricelist'])['group_product_pricelist']
        had_discount_group = self.default_get(['group_discount_per_so_line'])['group_discount_per_so_line']
        super().set_values()
        if had_discount_group and not self.group_discount_per_so_line:
            pl = self.env['product.pricelist'].search([('discount_policy', '=', 'without_discount')])
            pl.write({'discount_policy': 'with_discount'})

        if self.group_product_pricelist and not had_group_pl:
            self.env['res.company']._activate_or_create_pricelists()
        elif not self.group_product_pricelist:
            self.env['product.pricelist'].sudo().search([]).action_archive()

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
        if not self.user_has_groups('product.group_product_pricelist'):
            group_user = self.env.ref('base.group_user').sudo()
            group_user._apply_group(self.env.ref('product.group_product_pricelist'))
            self.env['res.company']._activate_or_create_pricelists()

    def write(self, vals):
        """ Archive pricelist when the linked currency is archived. """
        res = super().write(vals)

        if self and 'active' in vals and not vals['active']:
            self.env['product.pricelist'].search([('currency_id', 'in', self.ids)]).action_archive()

        return res

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
    @api.depends_context('company', 'country_code')
    def _compute_product_pricelist(self):
        res = self.env['product.pricelist']._get_partner_pricelist_multi(self._ids)
        for partner in self:
            partner.property_product_pricelist = res.get(partner.id)

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

    def _company_dependent_commercial_fields(self):
        # property_product_pricelist is not a company_dependent field but technically behaves as one
        return super()._company_dependent_commercial_fields() + ['property_product_pricelist']

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
                    "Please set a precision between %s and 1.",
                    precision, 1.0 / 10.0**precision),
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
from . import ir_attachment
from . import product_attribute
from . import product_attribute_custom_value
from . import product_attribute_value
from . import product_catalog_mixin
from . import product_category
from . import product_document
from . import product_packaging
from . import product_pricelist
from . import product_pricelist_item
from . import product_supplierinfo
from . import product_tag
from . import product_template_attribute_line
from . import product_template_attribute_exclusion
from . import product_template_attribute_value
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
from odoo.tools import SQL

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
        # Generate a unique `barcode` prefix so consecutive populate runs don't
        # raise a unique constraint error on `barcode`.
        barcode_prefix = 'BARCODE-PP-'
        query = SQL("""
            SELECT barcode
              FROM product_product
             WHERE barcode LIKE %s
          ORDER BY id DESC
             LIMIT 1
        """, barcode_prefix + '%')
        self.env.cr.execute(query)
        barcode = self.env.cr.fetchone()
        if barcode:
            barcode_prefix = barcode[0] + '-'

        return [
            ("name", populate.constant('product_product_name_{counter}')),
            ("description", populate.constant('product_product_description_{counter}')),
            ("default_code", populate.constant('PP-{counter}')),
            ("barcode", populate.constant(barcode_prefix + '{counter}')),
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


def _prepare_data(env, docids, data):
    # change product ids by actual product object to get access to fields in xml template
    # we needed to pass ids because reports only accepts native python types (int, float, strings, ...)

    layout_wizard = env['product.label.layout'].browse(data.get('layout_wizard'))
    if data.get('active_model') == 'product.template':
        Product = env['product.template'].with_context(display_default_code=False)
    elif data.get('active_model') == 'product.product':
        Product = env['product.product'].with_context(display_default_code=False)
    elif data.get("studio") and docids:
        # special case: users trying to customize labels
        products = env['product.template'].with_context(display_default_code=False).browse(docids)
        quantity_by_product = defaultdict(list)
        for product in products:
            quantity_by_product[product].append((product.barcode, 1))
        return {
            'quantity': quantity_by_product,
            'page_numbers': 1,
            'pricelist': layout_wizard.pricelist_id,
        }
    else:
        raise UserError(_('Product model not defined, Please contact your administrator.'))

    if not layout_wizard:
        return {}

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

    return {
        'quantity': quantity_by_product,
        'page_numbers': (total - 1) // (layout_wizard.rows * layout_wizard.columns) + 1,
        'price_included': data.get('price_included'),
        'extra_html': layout_wizard.extra_html,
        'pricelist': layout_wizard.pricelist_id,
    }


class ReportProductTemplateLabel2x7(models.AbstractModel):
    _name = 'report.product.report_producttemplatelabel2x7'
    _description = 'Product Label Report 2x7'

    def _get_report_values(self, docids, data):
        return _prepare_data(self.env, docids, data)


class ReportProductTemplateLabel4x7(models.AbstractModel):
    _name = 'report.product.report_producttemplatelabel4x7'
    _description = 'Product Label Report 4x7'

    def _get_report_values(self, docids, data):
        return _prepare_data(self.env, docids, data)


class ReportProductTemplateLabel4x12(models.AbstractModel):
    _name = 'report.product.report_producttemplatelabel4x12'
    _description = 'Product Label Report 4x12'

    def _get_report_values(self, docids, data):
        return _prepare_data(self.env, docids, data)


class ReportProductTemplateLabel4x12NoPrice(models.AbstractModel):
    _name = 'report.product.report_producttemplatelabel4x12noprice'
    _description = 'Product Label Report 4x12 No Price'

    def _get_report_values(self, docids, data):
        return _prepare_data(self.env, docids, data)


class ReportProductTemplateLabelDymo(models.AbstractModel):
    _name = 'report.product.report_producttemplatelabel_dymo'
    _description = 'Product Label Report'

    def _get_report_values(self, docids, data):
        return _prepare_data(self.env, docids, data)

```

## File: report\product_packaging.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <template id="report_packagingbarcode">
            <t t-call="web.basic_layout">
                <div class="page">
                    <div class="oe_structure"></div>
                    <t t-foreach="docs" t-as="packaging">
                        <div class="col-4" style="padding:0;">
                            <div class="oe_structure"></div>
                            <table class="table table-condensed" style="border-bottom: 0px solid white !important;width: 3in;">
                                <tr>
                                    <th style="text-align: left;">
                                        <strong><span t-field="packaging.name">Package Type A</span></strong>
                                    </th>
                                </tr>
                                <tr>
                                    <td>
                                        <strong><span t-field="packaging.product_id.display_name">Eco-friendly Wooden Chair</span></strong>
                                    </td>
                                </tr>
                                <tr>
                                    <td>
                                        <div class="o_row">
                                            <strong>Qty: </strong>
                                            <strong><span t-field="packaging.qty">10</span></strong>
                                            <strong><span t-field="packaging.product_uom_id" groups="uom.group_uom">Units</span></strong>
                                        </div>
                                    </td>
                                </tr>
                                  <t t-if="packaging.barcode">
                                    <tr>
                                    <td style="text-align: center; vertical-align: middle;" class="col-5">
                                        <div t-field="packaging.barcode" t-options="{'widget': 'barcode', 'symbology': 'auto', 'width': 600, 'height': 150, 'img_style': 'width:100%;height:20%;'}"/>
                                        <span t-field="packaging.barcode">123456789012</span>
                                    </td>
                                </tr>
                              </t>
                            </table>
                            <div class="oe_structure"></div>
                        </div>
                    </t>
                    <div class="oe_structure"></div>
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
            'display_pricelist_title': data.get('display_pricelist_title', False) and bool(data['display_pricelist_title']),
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
            <div class="oe_structure"></div>
            <div class="row my-3" t-if="display_pricelist_title">
                <div class="col-12">
                    <h2 t-if="is_html_type">
                        Pricelist:
                        <a href="#" class="o_action" data-model="product.pricelist" t-att-data-res-id="pricelist.id">
                            <span t-field="pricelist.display_name">Gold Member Pricelist</span>
                        </a>
                    </h2>
                    <h2 t-else="">
                        Pricelist: <span t-field="pricelist.display_name">Gold Member Pricelist</span>
                    </h2>
                </div>
            </div>
            <div class="oe_structure"></div>
            <div class="row">
                <div class="col-12">
                    <table class="table table-sm">
                        <thead>
                            <th class="text-end" colspan="100%">Quantities (Price)</th>
                            <tr>
                                <th>Products</th>
                                <th groups="uom.group_uom">UOM</th>
                                <t t-foreach="quantities" t-as="qty">
                                    <th class="text-end"><span t-out="qty">10 Units</span></th>
                                </t>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="products" t-as="product">
                                <tr>
                                    <td t-att-class="is_product_tmpl and 'fw-bold' or None">
                                        <a t-if="is_html_type" href="#" class="o_action" t-att-data-model="is_product_tmpl and 'product.template' or 'product.product'" t-att-data-res-id="product['id']">
                                            <span t-out="product['name']">Virtual Interior Design</span>
                                        </a>
                                        <span t-else="" t-out="product['name']">Acme Widget</span>
                                    </td>
                                    <td groups="uom.group_uom" t-out="product['uom']"/>
                                    <t t-foreach="quantities" t-as="qty">
                                        <td class="text-end">
                                            <span t-out="product['price'][qty]" t-options='{"widget": "monetary", "display_currency": pricelist.currency_id}'>$15.00</span>
                                        </td>
                                    </t>
                                </tr>
                                <t t-if="is_product_tmpl and 'variants' in product">
                                    <tr t-foreach="product['variants']" t-as="variant">
                                        <td>
                                            <a t-if="is_html_type" href="#" class="o_action ms-4" data-model="product.product" t-att-data-res-id="variant['id']">
                                                <span t-out="variant['name']">Acme Widget - Blue</span>
                                            </a>
                                            <span t-else="" class="ms-4" t-out="variant['name']">Acme Widget - Blue</span>
                                        </td>
                                        <td groups="uom.group_uom" t-out="product['uom']"/>
                                        <t t-foreach="quantities" t-as="qty">
                                            <td class="text-end">
                                                <span t-out="variant['price'][qty]" t-options='{"widget": "monetary", "display_currency": pricelist.currency_id}'>$14.00</span>
                                            </td>
                                        </t>
                                    </tr>
                                </t>
                            </t>
                        </tbody>
                    </table>
                </div>
            </div>
            <div class="oe_structure"></div>
        </div>
    </template>

    <template id="report_pricelist">
        <t t-call="web.basic_layout">
            <div class="page">
                <t t-call="product.report_pricelist_page"/>
            </div>
            <p style="page-break-before:always;"> </p>
            <div class="oe_structure"></div>
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
                            <strong class="o_label_price" t-out="pricelist._get_product_price(product, 1, pricelist.currency_id or product.currency_id)"
                                    t-options="{'widget': 'monetary', 'display_currency': pricelist.currency_id or product.currency_id, 'label_price': True}"/>
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
                        <strong class="o_label_price_medium" t-out="pricelist._get_product_price(product, 1, pricelist.currency_id or product.currency_id)"
                                t-options="{'widget': 'monetary', 'display_currency': pricelist.currency_id or product.currency_id, 'label_price': True}"/>
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
                    <div class="o_label_left_column">
                        <span class="text-nowrap" t-field="product.default_code"/>
                    </div>
                    <div class="o_label_price_medium text-end">
                        <strong t-out="pricelist._get_product_price(product, 1, pricelist.currency_id or product.currency_id)"
                            t-options="{'widget': 'monetary', 'display_currency': pricelist.currency_id or product.currency_id, 'label_price': True}"/>
                    </div>
                    <div class= "text-center o_label_small_barcode">
                        <t t-if="barcode">
                            <div t-out="barcode" style="padding:0" t-options="{'widget': 'barcode', 'symbology': 'auto', 'img_style': barcode_size}"/>
                            <span class="text-center" t-out="barcode"/>
                        </t>
                    </div>
                </div>
            </td>
        </template>

        <template id="report_simple_label4x12_no_price">
            <t t-set="barcode_size" t-value="'width:33mm;height:4mm'"/>
            <t t-set="table_style" t-value="'width:43mm;height:19mm;' + table_style"/>
            <td t-att-style="make_invisible and 'visibility:hidden;'" >
                <div class="o_label_full o_label_small_text" t-att-style="table_style">
                    <div class="o_label_name">
                        <strong t-field="product.display_name"/>
                    </div>
                    <div class="o_label_left_column o_label_full_with">
                        <span class="text-nowrap" t-field="product.default_code"/>
                    </div>
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
                        <strong class="o_label_price_small" t-out="pricelist._get_product_price(product, 1, pricelist.currency_id or product.currency_id)"
                                t-options="{'widget': 'monetary', 'display_currency': pricelist.currency_id or product.currency_id, 'label_price': True}"/>
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
                        <t t-set="report_to_call" t-value="'2x7'"/>
                    </t>
                    <t t-if="columns == 4 and rows == 7">
                        <t t-set="padding_page" t-value="'padding: 14mm 3mm'"/>
                        <t t-set="report_to_call" t-value="'4x7'"/>
                    </t>
                    <t t-if="columns == 4 and rows == 12">
                        <t t-set="padding_page" t-value="'padding: 20mm 8mm'"/>
                        <t t-set="report_to_call" t-value="'4x12'"/>
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
                                            <!-- IMPORTANT: explictly call templates in place of {{template}} otherwise studio won't load report correctly -->
                                            <t t-if="report_to_call == '2x7'">
                                                <t t-call="product.report_simple_label2x7"/>
                                            </t>
                                            <t t-elif="report_to_call == '4x7'">
                                                <t t-call="product.report_simple_label4x7"/>
                                            </t>
                                            <t t-elif="report_to_call == '4x12'">
                                                <t t-if="price_included">
                                                    <t t-call="product.report_simple_label4x12"/>
                                                </t>
                                                <t t-else="">
                                                    <t t-call="product.report_simple_label4x12_no_price"/>
                                                </t>
                                            </t>
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
        <record id="report_product_template_label_2x7" model="ir.actions.report">
            <field name="name">Product Label 2x7 (PDF)</field>
            <field name="model">product.template</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_producttemplatelabel2x7</field>
            <field name="report_file">product.report_producttemplatelabel2x7</field>
            <field name="paperformat_id" ref="product.paperformat_label_sheet"/>
            <field name="print_report_name">'Products Labels - %s' % (object.name)</field>
            <field name="binding_model_id" eval="False"/>
            <field name="binding_type">report</field>
        </record>
        <record id="report_product_template_label_4x7" model="ir.actions.report">
            <field name="name">Product Label 4x7 (PDF)</field>
            <field name="model">product.template</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_producttemplatelabel4x7</field>
            <field name="report_file">product.report_producttemplatelabel4x7</field>
            <field name="paperformat_id" ref="product.paperformat_label_sheet"/>
            <field name="print_report_name">'Products Labels - %s' % (object.name)</field>
            <field name="binding_model_id" eval="False"/>
            <field name="binding_type">report</field>
        </record>
        <record id="report_product_template_label_4x12" model="ir.actions.report">
            <field name="name">Product Label 4x12 (PDF)</field>
            <field name="model">product.template</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_producttemplatelabel4x12</field>
            <field name="report_file">product.report_producttemplatelabel4x12</field>
            <field name="paperformat_id" ref="product.paperformat_label_sheet"/>
            <field name="print_report_name">'Products Labels - %s' % (object.name)</field>
            <field name="binding_model_id" eval="False"/>
            <field name="binding_type">report</field>
        </record>
        <record id="report_product_template_label_4x12_noprice" model="ir.actions.report">
            <field name="name">Product Label 4x12 No Price (PDF)</field>
            <field name="model">product.template</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">product.report_producttemplatelabel4x12noprice</field>
            <field name="report_file">product.report_producttemplatelabel4x12noprice</field>
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

<!-- IMPORTANT: explictly include col/row otherwise studio won't load report correctly -->
<template id="report_producttemplatelabel2x7">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-set="columns" t-value="2"/>
            <t t-set="rows" t-value="7"/>
            <t t-call="product.report_productlabel"/>
        </div>
    </t>
</template>

<template id="report_producttemplatelabel4x7">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-set="columns" t-value="4"/>
            <t t-set="rows" t-value="7"/>
            <t t-call="product.report_productlabel"/>
        </div>
    </t>
</template>

<template id="report_producttemplatelabel4x12">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-set="columns" t-value="4"/>
            <t t-set="rows" t-value="12"/>
            <t t-set="price_included" t-value="True"/>
            <t t-call="product.report_productlabel"/>
        </div>
    </t>
</template>

<template id="report_producttemplatelabel4x12noprice">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-set="columns" t-value="4"/>
            <t t-set="rows" t-value="12"/>
            <t t-set="price_included" t-value="False"/>
            <t t-call="product.report_productlabel"/>
        </div>
    </t>
</template>

<template id="report_producttemplatelabel_dymo">
    <t t-call="web.basic_layout">
        <div class="page">
            <t t-call="product.report_productlabel_dymo"/>
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
access_product_document_user,access_product_document_user,model_product_document,base.group_user,1,1,1,1

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
        <field name="domain_force"> ['|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]</field>
    </record>

    <record id="product_document_comp_rule" model="ir.rule">
        <field name="name" >Product multi-company</field>
        <field name="model_id" ref="model_product_document"/>
        <field name="domain_force"> ['|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]</field>
    </record>

    <record model="ir.rule" id="product_pricelist_comp_rule">
        <field name="name">product pricelist company rule</field>
        <field name="model_id" ref="model_product_pricelist"/>
        <field name="domain_force"> ['|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]</field>
    </record>

    <record model="ir.rule" id="product_pricelist_item_comp_rule">
        <field name="name">product pricelist item company rule</field>
        <field name="model_id" ref="model_product_pricelist_item"/>
        <field name="domain_force"> ['|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]</field>
    </record>

    <record model="ir.rule" id="product_supplierinfo_comp_rule">
        <field name="name">product supplierinfo company rule</field>
        <field name="model_id" ref="model_product_supplierinfo"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]</field>
    </record>

    <record model="ir.rule" id="product_packaging_comp_rule">
        <field name="name">product packaging company rule</field>
        <field name="model_id" ref="model_product_packaging"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]</field>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M28.235 7.242a6.487 6.487 0 0 0-7.242 2.823l-3.884 6.252a2.418 2.418 0 0 0-.198 2.159l10.002 25.523 17.025-6.636a3.228 3.228 0 0 0 1.839-4.185l-8.823-22.515a2.427 2.427 0 0 0-1.613-1.452l-7.106-1.969Zm-1.092 7.865a1.787 1.787 0 0 0 1.017-2.317 1.795 1.795 0 0 0-2.323-1.015 1.787 1.787 0 0 0-1.017 2.317c.36.92 1.4 1.374 2.323 1.015Z" fill="#2EBCFA"/><path fill-rule="evenodd" clip-rule="evenodd" d="M32.423 10.924a6.482 6.482 0 0 0-6.734-3.877l-7.322.88a2.43 2.43 0 0 0-1.814 1.194L4.435 30.055a3.226 3.226 0 0 0 1.185 4.413l15.831 9.115L35.19 19.852c.382-.66.43-1.462.13-2.163l-2.897-6.764Zm-6.84 4.062c.857.493 1.954.2 2.45-.655a1.786 1.786 0 0 0-.657-2.443 1.796 1.796 0 0 0-2.45.655 1.786 1.786 0 0 0 .657 2.443Z" fill="#985184"/><path fill-rule="evenodd" clip-rule="evenodd" d="M27.78 7.134a6.476 6.476 0 0 1 4.643 3.789l2.897 6.764c.3.701.252 1.503-.13 2.163L24.61 38.124l-7.7-19.649a2.418 2.418 0 0 1 .198-2.158l3.885-6.252a6.487 6.487 0 0 1 6.788-2.931Zm-2.027 7.937a1.786 1.786 0 0 1-.827-2.53 1.796 1.796 0 0 1 2.322-.721 1.787 1.787 0 0 1-.105 3.287 1.792 1.792 0 0 1-1.39-.036Z" fill="#144496"/></svg>

```

## File: static\src\js\pricelist_report\product_pricelist_report.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { Component, markup, onRendered, onWillStart, useState } from "@odoo/owl";
import { Layout } from "@web/search/layout";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { useSetupAction } from "@web/webclient/actions/action_hook";


function sendCustomNotification(type, message) {
    return {
        type: "ir.actions.client",
        tag: "display_notification",
        params: {
            "type": type,
            "message": message
        },
    }
}

export class ProductPricelistReport extends Component {
    setup() {
        this.action = useService("action");
        this.orm = useService("orm");

        this.MAX_QTY = 5;
        const pastState = this.props.state || {};

        this.activeIds = this.props.action.context.active_ids;
        this.activeModel = this.props.action.context.active_model;

        this.state = useState({
            displayPricelistTitle: pastState.displayPricelistTitle || false,
            html: "",
            pricelists: [],
            _quantities: pastState.quantities || [1, 5, 10],
            selectedPricelist: {},
        });

        onWillStart(async () => {
            this.state.pricelists = await this.getPricelists()
            this.state.selectedPricelist = pastState.selectedPricelist || this.pricelists[0];

            this.renderHtml();
        });

        onRendered(() => {
            this.env.config.setDisplayName(_t("Pricelist Report"));
        });

        /*
        When following the link of a product and coming back we need to keep the 
        precedent state:
            - if the pricelist was being showed
            - wich pricelist is selected at the moment
            - which quantities
        */
        useSetupAction({
            getLocalState: () => {
                return {
                    displayPricelistTitle: this.displayPricelistTitle,
                    quantities: this.quantities,
                    selectedPricelist: this.selectedPricelist,
              };
            },
        });
    }

    // getters and setters

    get displayPricelistTitle() {
        return this.state.displayPricelistTitle;
    }

    get html() {
        return this.state.html;
    }

    get pricelists() {
        return this.state.pricelists;
    }

    get quantities() {
        return this.state._quantities;
    }

    set quantities(value) {
        this.state._quantities = value;
    }

    get reportParams() {
        return {
            active_model: this.activeModel || 'product.template',
            active_ids: this.activeIds || [],
            display_pricelist_title: this.displayPricelistTitle || '',
            pricelist_id: this.selectedPricelist.id || '',
            quantities: this.quantities || [1],
        };
    }

    get selectedPricelist() {
        return this.state.selectedPricelist;
    }

    // orm calls

    getPricelists() {
        return this.orm.searchRead("product.pricelist", [], ["id", "name"]);
    }

    async renderHtml() {
        let html = await this.orm.call(
            "report.product.report_pricelist", "get_html", [], {data: this.reportParams}
        );
        this.state.html = markup(html);
    }

    // events

    async onClickAddQty(ev) {
        ev.preventDefault(); // avoid automatic reloading of the page

        if (this.quantities.length >= this.MAX_QTY) {
            let message = _t(
                "At most %s quantities can be displayed simultaneously. Remove a selected quantity to add others.",
                this.MAX_QTY
            );
            await this.action.doAction(sendCustomNotification("warning", message));
            return;
        }

        const qty = parseInt($("input.add-quantity-input")[0].value);
        if (qty && qty > 0) {
            // Check qty already exist.
            if (this.quantities.indexOf(qty) === -1) {
                this.quantities.push(qty);
                this.quantities = this.quantities.sort((a, b) => a - b);
                this.renderHtml();
            } else {
                let message = _t("Quantity already present (%s).", qty);
                await this.action.doAction(sendCustomNotification("info", message));
            }
        } else {
            await this.action.doAction(
                sendCustomNotification("info", _t("Please enter a positive whole number."))
            );
        }
    }

    onClickLink(ev) {
        ev.preventDefault();

        let classes = ev.target.getAttribute("class", "");
        let resModel = ev.target.getAttribute("data-model", "");
        let resId = ev.target.getAttribute("data-res-id", "");

        if (classes && classes.includes("o_action") && resModel && resId) {
            this.action.doAction({
                type: 'ir.actions.act_window',
                res_model: resModel,
                res_id: parseInt(resId),
                views: [[false, 'form']],
                target: 'self',
            });
        }
    }

    onClickPrint() {
        this.action.doAction({
            type: 'ir.actions.report',
            report_type: 'qweb-pdf',
            report_name: 'product.report_pricelist',
            report_file: 'product.report_pricelist',
            data: this.reportParams,
        });
    }

    async onClickRemoveQty(ev) {
        if (this.quantities.length <= 1) {
            await this.action.doAction(
                sendCustomNotification("warning", _t("You must leave at least one quantity."))
            );
            return;
        }

        const qty = parseInt(ev.srcElement.parentElement.childNodes[0].data);
        this.quantities = this.quantities.filter(q => q !== qty);
        this.renderHtml();
    }

    onSelectPricelist(ev) {
        this.state.selectedPricelist = this.pricelists.filter(pricelist => 
            pricelist.id === parseInt(ev.target.value)
        )[0];

        this.renderHtml();
    }

    onToggleDisplayPricelist() {
        this.state.displayPricelistTitle = !this.displayPricelistTitle;
        this.renderHtml();
    }
}

ProductPricelistReport.props = {
    action: { type: Object },
    "*": true,
}
ProductPricelistReport.components = { Layout };
ProductPricelistReport.template = "product.ProductPricelistReport";
registry.category("actions").add("generate_pricelist_report", ProductPricelistReport);

```

## File: static\src\js\pricelist_report\product_pricelist_report.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>

    <t t-name="product.ProductPricelistReport">
        <div class="o_action">
            <Layout display="{ controlPanel: { 'bottom-right': false } }">
                <t t-set-slot="control-panel-always-buttons">
                    <button t-on-click="onClickPrint" type="button" class="btn btn-primary" title="Print">Print</button>
                </t>
                <t t-set-slot="layout-actions">
                    <form class="o_pricelist_report_form d-flex flex-column gap-1">
                        <div class="d-flex align-items-center gap-3 w-100">
                            <label class="visually-hidden" for="pricelists">Username</label>
                            <div class="input-group w-auto">
                                <div class="input-group-text fw-bold">Pricelist</div>
                                <select name="pricelists"
                                            id="pricelists"
                                            class="o_select form-select border-0"
                                            t-on-change="onSelectPricelist">
                                        <option t-out="selectedPricelist.name" t-att-value="selectedPricelist.id"/>
                                        <t t-foreach="pricelists" t-as="pricelist" t-key="pricelist.id">
                                            <t t-if="pricelist.id != selectedPricelist.id">
                                                <option t-out="pricelist.name" t-att-value="pricelist.id"/>
                                            </t>
                                    </t>
                                    </select>
                            </div>
                            <div class="form-check w-auto">
                                <input class="o_display_pricelist_title form-check-input ms-0 me-2"
                                    id="display_pricelist_title"
                                    type="checkbox"
                                    t-att-checked="displayPricelistTitle"
                                    t-on-click="onToggleDisplayPricelist"/>
                                <label for="display_pricelist_title" class="form-check-label">Display Pricelist</label>
                            </div>
                        </div>
                        <div class="d-flex align-items-center gap-3 w-100">
                            <div class="input-group d-flex flex-nowrap w-50" style="min-width:210px;">
                                <span class="input-group-text fw-bold"> 
                                    Quantities
                                </span>
                                <input type="number"
                                    class="form-control add-quantity-input"
                                    value="1"
                                    min="1"/>
                                <button class="o_add_qty btn btn-primary fa fa-plus"
                                        type="submit"
                                        t-on-click="onClickAddQty"
                                        title="Add a quantity"/>
                            </div>
                            <div class="d-flex align-items-center w-50">
                                <span class="o_badges_list">
                                    <t t-foreach="quantities" t-as="qty" t-key="qty">
                                        <span class="o_field_badge o_remove_qty badge rounded-pill me-2 py-1 border " t-att-value="qty">
                                            <t class="me-2" t-esc="qty"/>
                                            <i class="oi oi-close ms-1 opacity-50 opacity-100-hover text-900 cursor-pointer"
                                            title="Remove quantity"
                                            t-on-click="onClickRemoveQty"/>
                                        </span>
                                    </t>
                                </span>
                            </div>
                        </div>
                    </form>
                </t>
                <div t-on-click="onClickLink">
                    <t t-out="html"/>
                </div>
           </Layout>
        </div>
    </t>

</templates>

```

## File: static\src\js\product_document_kanban\product_document_kanban_controller.js

```javascript
/** @odoo-module **/

import { KanbanController } from "@web/views/kanban/kanban_controller";
import { useBus, useService } from "@web/core/utils/hooks";
import { useRef } from "@odoo/owl";

export class ProductDocumentKanbanController extends KanbanController {
    setup() {
        super.setup();
        this.uploadFileInputRef = useRef("uploadFileInput");
        this.fileUploadService = useService("file_upload");
        useBus(
            this.fileUploadService.bus,
            "FILE_UPLOAD_LOADED",
            async () => {
                await this.model.root.load();
            },
        );
    }

    async onFileInputChange(ev) {
        if (!ev.target.files.length) {
            return;
        }
        await this.fileUploadService.upload(
            "/product/document/upload",
            ev.target.files,
            {
                buildFormData: (formData) => {
                    formData.append("res_model", this.props.context.active_model);
                    formData.append("res_id", this.props.context.active_id);
                },
            },
        );
        // Reset the file input's value so that the same file may be uploaded twice.
        ev.target.value = "";
    }
}

```

## File: static\src\js\product_document_kanban\product_document_kanban_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="product.ProductDocumentKanbanView.Buttons" t-inherit="web.KanbanView.Buttons" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_cp_buttons')]" position="inside">
            <input type="file" multiple="true" t-ref="uploadFileInput" class="o_input_file o_hidden" t-on-change.stop="onFileInputChange"/>
            <button type="button" t-attf-class="btn btn-primary o_mrp_documents_kanban_upload"
                t-on-click.stop.prevent="() => this.uploadFileInputRef.el.click()">
                Upload
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\js\product_document_kanban\product_document_kanban_record.js

```javascript
/** @odoo-module **/

import { CANCEL_GLOBAL_CLICK, KanbanRecord } from "@web/views/kanban/kanban_record";
import { useService } from "@web/core/utils/hooks";
import { useFileViewer } from "@web/core/file_viewer/file_viewer_hook";

export class ProductDocumentKanbanRecord extends KanbanRecord {
    setup() {
        super.setup();
        this.store = useService("mail.store");
        this.fileViewer = useFileViewer();
    }
    /**
     * @override
     *
     * Override to open the preview upon clicking the image, if compatible.
     */
    onGlobalClick(ev) {
        if (ev.target.closest(CANCEL_GLOBAL_CLICK)) {
            return;
        } else if (ev.target.closest(".o_kanban_previewer")) {
            const attachment = this.store.Attachment.insert({
                id: this.props.record.data.ir_attachment_id[0],
                filename: this.props.record.data.name,
                name: this.props.record.data.name,
                mimetype: this.props.record.data.mimetype,
            });
            this.fileViewer.open(attachment)
            return;
        }
        return super.onGlobalClick(...arguments);
    }
}

```

## File: static\src\js\product_document_kanban\product_document_kanban_renderer.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";
import { KanbanRenderer } from "@web/views/kanban/kanban_renderer";
import { ProductDocumentKanbanRecord } from "@product/js/product_document_kanban/product_document_kanban_record";
import { FileUploadProgressContainer } from "@web/core/file_upload/file_upload_progress_container";
import { FileUploadProgressKanbanRecord } from "@web/core/file_upload/file_upload_progress_record";

export class ProductDocumentKanbanRenderer extends KanbanRenderer {
    setup() {
        super.setup();
        this.fileUploadService = useService("file_upload");
    }
}

ProductDocumentKanbanRenderer.components = {
    ...KanbanRenderer.components,
    FileUploadProgressContainer,
    FileUploadProgressKanbanRecord,
    KanbanRecord: ProductDocumentKanbanRecord,
};
ProductDocumentKanbanRenderer.template = "product.ProductDocumentKanbanRenderer";

```

## File: static\src\js\product_document_kanban\product_document_kanban_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="product.ProductDocumentKanbanRenderer" t-inherit-mode="primary" t-inherit="web.KanbanRenderer">
        <!-- Before the first t-foreach -->
        <xpath expr="//t[@t-key='groupOrRecord.key']" position="before">
            <FileUploadProgressContainer fileUploads="fileUploadService.uploads" Component="constructor.components.FileUploadProgressKanbanRecord"/>
        </xpath>
    </t>
</templates>

```

## File: static\src\js\product_document_kanban\product_document_kanban_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";

import { kanbanView } from "@web/views/kanban/kanban_view";
import { ProductDocumentKanbanController } from "@product/js/product_document_kanban/product_document_kanban_controller";
import { ProductDocumentKanbanRenderer } from "@product/js/product_document_kanban/product_document_kanban_renderer";

export const productDocumentKanbanView = {
    ...kanbanView,
    Controller: ProductDocumentKanbanController,
    Renderer: ProductDocumentKanbanRenderer,
    buttonTemplate: "product.ProductDocumentKanbanView.Buttons",
};

registry.category("views").add("product_documents_kanban", productDocumentKanbanView);

```

## File: static\src\product_catalog\kanban_controller.js

```javascript
/** @odoo-module **/

import { KanbanController } from "@web/views/kanban/kanban_controller";
import { onWillStart } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { useDebounced } from "@web/core/utils/timing";
import { _t } from "@web/core/l10n/translation";

export class ProductCatalogKanbanController extends KanbanController {
    static template = "ProductCatalogKanbanController";

    setup() {
        super.setup();
        this.action = useService("action");
        this.orm = useService("orm");
        this.orderId = this.props.context.order_id;
        this.orderResModel = this.props.context.product_catalog_order_model;
        this.backToQuotationDebounced = useDebounced(this.backToQuotation, 500)

        onWillStart(async () => this._defineButtonContent());
    }

    // Force the slot for the "Back to Quotation" button to always be shown.
    get canCreate() {
        return true;
    }

    async _defineButtonContent() {
        // Define the button's label depending of the order's state.
        const orderStateInfo = await this.orm.searchRead(
            this.orderResModel, [["id", "=", this.orderId]], ["state"]
        );
        const orderIsQuotation = ["draft", "sent"].includes(orderStateInfo[0].state);
        if (orderIsQuotation) {
            this.buttonString = _t("Back to Quotation");
        } else {
            this.buttonString = _t("Back to Order");
        }
    }

    async backToQuotation() {
        // Restore the last form view from the breadcrumbs if breadcrumbs are available.
        // If, for some weird reason, the user reloads the page then the breadcrumbs are
        // lost, and we fall back to the form view ourselves.
        if (this.env.config.breadcrumbs.length > 1) {
            await this.action.restore();
        } else {
            await this.action.doAction({
                type: "ir.actions.act_window",
                res_model: this.orderResModel,
                views: [[false, "form"]],
                view_mode: "form",
                res_id: this.orderId,
            });
        }
    }
}

```

## File: static\src\product_catalog\kanban_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ProductCatalogKanbanController" t-inherit="web.KanbanView" t-inherit-mode="primary">
        <xpath expr="//button[hasclass('o-kanban-button-new')]" position="replace">
            <button t-out="this.buttonString" type="button" class="btn btn-secondary o-kanban-button-back" t-on-click="this.backToQuotationDebounced"/>
        </xpath>
    </t>
</templates>

```

## File: static\src\product_catalog\kanban_model.js

```javascript
/** @odoo-module */

import { Record } from "@web/model/relational_model/record";
import { RelationalModel } from "@web/model/relational_model/relational_model";

class ProductCatalogRecord extends Record {
    setup(config, data, options = {}) {
        this.productCatalogData = data.productCatalogData;
        data = { ...data };
        delete data.productCatalogData;
        super.setup(config, data, options);
    }
}

export class ProductCatalogKanbanModel extends RelationalModel {
    static Record = ProductCatalogRecord;

    async _loadData(params) {
        const result = await super._loadData(...arguments);
        if (!params.isMonoRecord && !params.groupBy.length) {
            const orderLinesInfo = await this.rpc("/product/catalog/order_lines_info", {
                order_id: params.context.order_id,
                product_ids: result.records.map((rec) => rec.id),
                res_model: params.context.product_catalog_order_model,
            });
            for (const record of result.records) {
                record.productCatalogData = orderLinesInfo[record.id];
            }
        }
        return result;
    }
}

```

## File: static\src\product_catalog\kanban_record.js

```javascript
/** @odoo-module */
import { useSubEnv } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { useDebounced } from "@web/core/utils/timing";
import { KanbanRecord } from "@web/views/kanban/kanban_record";
import { ProductCatalogOrderLine } from "./order_line/order_line";

export class ProductCatalogKanbanRecord extends KanbanRecord {
    static template = "ProductCatalogKanbanRecord";
    static components = {
        ...KanbanRecord.components,
        ProductCatalogOrderLine,
    };

    setup() {
        super.setup();
        this.rpc = useService("rpc");
        this.debouncedUpdateQuantity = useDebounced(this._updateQuantity, 500, {
            execBeforeUnmount: true,
        });

        useSubEnv({
            currencyId: this.props.record.context.product_catalog_currency_id,
            orderId: this.props.record.context.product_catalog_order_id,
            orderResModel: this.props.record.context.product_catalog_order_model,
            digits: this.props.record.context.product_catalog_digits,
            displayUoM: this.props.record.context.display_uom,
            precision: this.props.record.context.precision,
            productId: this.props.record.resId,
            addProduct: this.addProduct.bind(this),
            removeProduct: this.removeProduct.bind(this),
            increaseQuantity: this.increaseQuantity.bind(this),
            setQuantity: this.setQuantity.bind(this),
            decreaseQuantity: this.decreaseQuantity.bind(this),
        });
    }

    get orderLineComponent() {
        return ProductCatalogOrderLine;
    }

    get productCatalogData() {
        return this.props.record.productCatalogData;
    }

    onGlobalClick(ev) {
        // avoid a concurrent update when clicking on the buttons (that are inside the record)
        if (ev.target.closest(".o_product_catalog_cancel_global_click")) {
            return;
        }
        if (this.productCatalogData.quantity === 0) {
            this.addProduct();
        } else {
            this.increaseQuantity();
        }
    }

    //--------------------------------------------------------------------------
    // Data Exchanges
    //--------------------------------------------------------------------------

    async _updateQuantity() {
        const price = await this._updateQuantityAndGetPrice();
        this.productCatalogData.price = parseFloat(price);
    }

    _updateQuantityAndGetPrice() {
        return this.rpc("/product/catalog/update_order_line_info", this._getUpdateQuantityAndGetPrice());
    }

    _getUpdateQuantityAndGetPrice() {
        return {
            order_id: this.env.orderId,
            product_id: this.env.productId,
            quantity: this.productCatalogData.quantity,
            res_model: this.env.orderResModel,
        };
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    updateQuantity(quantity) {
        if (this.productCatalogData.readOnly) {
            return;
        }
        this.productCatalogData.quantity = quantity || 0;
        this.debouncedUpdateQuantity();
    }

    /**
     * Add the product to the order
     */
    addProduct(qty=1) {
        this.updateQuantity(qty);
    }

    /**
     * Remove the product to the order
     */
    removeProduct() {
        this.updateQuantity(0);
    }

    /**
     * Increase the quantity of the product on the order line.
     */
    increaseQuantity(qty=1) {
        this.updateQuantity(this.productCatalogData.quantity + qty);
    }

    /**
     * Set the quantity of the product on the order line.
     *
     * @param {Event} event
     */
    setQuantity(event) {
        this.updateQuantity(parseFloat(event.target.value));
    }

    /**
     * Decrease the quantity of the product on the order line.
     */
    decreaseQuantity() {
        this.updateQuantity(parseFloat(this.productCatalogData.quantity - 1));
    }
}

```

## File: static\src\product_catalog\kanban_record.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ProductCatalogKanbanRecord">
        <div role="article"
             t-att-class="getRecordClasses()"
             t-att-data-id="props.record.id"
             t-att-tabindex="props.record.model.useSampleModel ? -1 : 0"
             t-on-click="onGlobalClick"
             t-ref="root">
            <div class="d-flex flex-column h-100"
                 t-att-class="{'o_product_added': productCatalogData.quantity}">
                <t t-call="{{ templates[this.constructor.KANBAN_BOX_ATTRIBUTE] }}"
                   t-call-context="this.renderingContext"/>
                <t t-component="orderLineComponent" productId="props.record.resId" t-props="productCatalogData"/>
            </div>
            <t t-call="{{ this.constructor.menuTemplate }}"/>
        </div>
    </t>
</templates>

```

## File: static\src\product_catalog\kanban_renderer.js

```javascript
/** @odoo-module **/

import { KanbanRenderer } from "@web/views/kanban/kanban_renderer";
import { useService } from "@web/core/utils/hooks";

import { ProductCatalogKanbanRecord } from "./kanban_record";

export class ProductCatalogKanbanRenderer extends KanbanRenderer {
    static template = "ProductCatalogKanbanRenderer";
    static components = {
        ...KanbanRenderer.components,
        KanbanRecord: ProductCatalogKanbanRecord,
    };

    setup() {
        super.setup();
        this.action = useService("action");
    }

    get createProductContext() {
        return {};
    }

    async createProduct() {
        await this.action.doAction(
            {
                type: "ir.actions.act_window",
                res_model: "product.product",
                target: "new",
                views: [[false, "form"]],
                view_mode: "form",
                context: this.createProductContext,
            },
            {
                onClose: () => this.props.list.model.load(),
            }
        );
    }
}

```

## File: static\src\product_catalog\kanban_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">
    <t t-name="ProductCatalogKanbanRenderer" t-inherit="web.KanbanRenderer" t-inherit-mode="primary">
        <t t-if="showNoContentHelper" position="replace">
            <t t-if="showNoContentHelper">
                <div class="o_view_nocontent" role="alert">
                    <div class="o_nocontent_help">
                        <p class="o_view_nocontent_smiling_face">
                            No products could be found.<br/>
                            <button class="mt-2 btn btn-primary" t-on-click="this.createProduct">Create a product</button>
                        </p>
                        <p>
                            You must define a product for everything you sell or purchase,
                            whether it's a storable product, a consumable or a service.
                        </p>
                    </div>
                </div>
            </t>
        </t>
    </t>
</templates>

```

## File: static\src\product_catalog\kanban_view.js

```javascript
/** @odoo-module **/

import { kanbanView } from "@web/views/kanban/kanban_view";
import { registry } from "@web/core/registry";

import { ProductCatalogKanbanController } from "./kanban_controller";
import { ProductCatalogKanbanModel } from "./kanban_model";
import { ProductCatalogKanbanRenderer } from "./kanban_renderer";
import { ProductCatalogSearchPanel} from "./search/search_panel";


export const productCatalogKanbanView = {
    ...kanbanView,
    Controller: ProductCatalogKanbanController,
    Model: ProductCatalogKanbanModel,
    Renderer: ProductCatalogKanbanRenderer,
    SearchPanel: ProductCatalogSearchPanel,
};

registry.category("views").add("product_kanban_catalog", productCatalogKanbanView);

```

## File: static\src\product_catalog\order_line\order_line.js

```javascript
/** @odoo-module */
import { Component } from "@odoo/owl";
import { formatFloat, formatMonetary } from "@web/views/fields/formatters";

export class ProductCatalogOrderLine extends Component {
    static template = "ProductCatalogOrderLine";
    static props = {
        productId: Number,
        quantity: Number,
        price: Number,
        productType: String,
        readOnly: { type: Boolean, optional: true },
        warning: { type: String, optional: true},
    };

    /**
     * Focus input text when clicked
     * @param {Event} ev 
     */
    _onFocus(ev) {
        ev.target.select();
    }

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    isInOrder() {
        return this.props.quantity !== 0;
    }

    get disableRemove() {
        return false;
    }

    get disabledButtonTooltip() {
        return "";
    }

    get price() {
        const { currencyId, digits } = this.env;
        return formatMonetary(this.props.price, { currencyId, digits });
    }

    get quantity() {
        const digits = [false, this.env.precision];
        const options = { digits, decimalPoint: ".", thousandsSep: "" };
        return parseFloat(formatFloat(this.props.quantity, options));
    }
}

```

## File: static\src\product_catalog\order_line\order_line.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="ProductCatalogOrderLine">
        <!-- Replace the element found using the css selector by the content of the portalled
             template.  -->
        <t t-portal="`#product-${props.productId}-price`">
            <span class="o_product_catalog_price">Unit price: <t t-out="price"/></span>
        </t>
        <div 
            t-if="props.readOnly and props.warning"
            class="text-danger text-truncate my-2 pt-3 border-top">
            <i class="fa fa-warning" t-att-title="props.warning"/>
            <span 
                class="px-1" 
                t-att-title="props.warning"
                t-out="props.warning"/>
        </div>
        <span t-elif="props.readOnly" class="my-2 pt-3 border-top" t-out="props.warning">
            You can't edit this product in the catalog.
        </span>
        <div t-else="" class="d-flex justify-content-end align-items-center">
            <div t-if="isInOrder()"
                    class="input-group o_product_catalog_quantity o_product_catalog_cancel_global_click w-50">
                <button class="btn btn-primary border"
                        t-on-click.stop="this.env.decreaseQuantity"
                        t-att-disabled="disableRemove"
                        t-att-data-tooltip="disabledButtonTooltip">
                    <i class="fa fa-minus center"/>
                </button>
                <input class="o_input form-control border text-center text-bg-light"
                        type="number"
                        t-att-value="quantity"
                        t-on-change="this.env.setQuantity"
                        t-on-focus="_onFocus"
                />
                <button class="btn btn-primary border"
                        t-on-click.stop="(ev) => this.env.increaseQuantity()">
                    <i class="fa fa-plus"/>
                </button>
            </div>
            <t t-elif="props.warning">
                <i class="fa fa-warning text-warning" t-att-title="props.warning"/>
                <span 
                    class="text-truncate text-warning px-1" 
                    t-att-title="props.warning"
                    t-out="props.warning"/>
            </t>
            <div 
                class="ms-auto o_product_catalog_buttons o_product_catalog_cancel_global_click"
                style="min-width: max-content;">
                <button t-if="!isInOrder()"
                        t-on-click.stop="() => this.env.addProduct()"
                        class="btn btn-secondary">
                    <i class="fa fa-shopping-cart"/>
                    Add
                </button>
                <div t-else="" class="o_tooltip_div_remove" t-att-data-tooltip="this.disabledButtonTooltip">
                    <button t-on-click.stop="this.env.removeProduct"
                            t-att-disabled="disableRemove"
                            class="btn btn-light border">
                        <i class="fa fa-trash"/>
                        Remove
                    </button>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\product_catalog\search\search_panel.js

```javascript
/** @odoo-module **/

import { SearchPanel } from "@web/search/search_panel/search_panel";
import { useState } from "@odoo/owl";


export class ProductCatalogSearchPanel extends SearchPanel {
    setup() {
        super.setup();

        this.state = useState({
            ...this.state,
            sectionOfAttributes: {},
        });
    }

    updateActiveValues() {
        super.updateActiveValues();
        this.state.sectionOfAttributes = this.buildSection();
    }

    buildSection() {
        const values = this.env.searchModel.filters[0].values;
        let sections = new Map();

        values.forEach(element => {
            const name = element.display_name;
            const id = element.id;
            const count = element.__count;

            if (sections.has(name)) {
                let currentAttr = sections.get(name);
                currentAttr.get('ids').push(id);
                currentAttr.set('count', currentAttr.get('count') + count);
            } else if (count > 0) {
                let newAttr = new Map();
                newAttr.set('ids', [id]);
                newAttr.set('count', count);
                sections.set(name, newAttr);
            }
        });

        return sections;
    }

    toggleSectionFilterValue(filterId, attrIds, { currentTarget }) {
        attrIds.forEach(id => {
            this.toggleFilterValue(filterId, id, { currentTarget });
        })
    }
}

ProductCatalogSearchPanel.subTemplates = {
    ...SearchPanel.subTemplates,
    filtersGroup: "ProductCatalogSearchPanel.FiltersGroup",
}

```

## File: static\src\product_catalog\search\search_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-name="ProductCatalogSearchPanel.FiltersGroup" t-inherit="web.SearchPanel.FiltersGroup" t-inherit-mode="primary">
        <li t-foreach="[...values.keys()]" position="replace">
            <li t-foreach="[...this.state.sectionOfAttributes.keys()]" t-as="attrName" t-key="attrName"
                class="o_search_panel_filter_value list-group-item p-0 mb-1 border-0 o_cursor_pointer"
                t-att-class="{ 'ps-2' : isChildList }">
                <t t-set="attrCount" t-value="this.state.sectionOfAttributes.get(attrName).get('count')"/>
                <t t-set="attrIds" t-value="this.state.sectionOfAttributes.get(attrName).get('ids')"/>
                <div class="form-check w-100">
                    <input type="checkbox"
                           t-attf-id="{{ section.id }}_input_{{ attrName }}"
                           class="form-check-input"
                           t-on-click="ev => this.toggleSectionFilterValue(section.id, attrIds, ev)"/>
                    <label class="o_search_panel_label form-check-label d-flex align-items-center justify-content-between w-100 o_cursor_pointer"
                           t-attf-for="{{ section.id }}_input_{{ attrName }}"
                           t-att-title="(group and group.tooltip) or false">
                        <span class="o_search_panel_label_title text-truncate" t-esc="attrName"/>
                        <span t-if="section.enableCounters and attrCount gt 0"
                              class="o_search_panel_counter text-muted mx-2 small"
                              t-esc="attrCount"/>
                    </label>
                </div>
            </li>
        </li>
    </t>
</templates>

```

## File: views\product_attribute_value_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_attribute_value_list" model="ir.ui.view">
        <field name="name">product.attribute.value.list</field>
        <field name="model">product.attribute.value</field>
        <field name="arch" type="xml">
            <tree string="Attribute Values">
                <field name="name"/>
                <field name="default_extra_price"/>
            </tree>
        </field>
    </record>

</odoo>

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
                            invisible="not number_related_products">
                        <div class="o_stat_info">
                            <span class="o_stat_value"><field name="number_related_products"/></span>
                            <span class="o_stat_text">Related Products</span>
                        </div>
                    </button>
                </div>
                <group name="main_fields">
                    <group name="sale_main_fields">
                        <label for="name" string="Attribute Name"/>
                        <field name="name" nolabel="1"/>
                        <field name="display_type" widget="radio"/>
                        <field name="create_variant" widget="radio" readonly="number_related_products != 0"/>
                    </group>
                </group>
                <notebook>
                    <page string="Attribute Values" name="attribute_values">
                        <field name="value_ids" widget="one2many" nolabel="1">
                            <tree string="Values" editable="bottom">
                                <field name="sequence" widget="handle"/>
                                <field name="name"/>
                                <field name="display_type" column_invisible="True"/>
                                <field name="is_custom" groups="product.group_product_variant" column_invisible="parent.display_type == 'multi'"/>
                                <field name="html_color" column_invisible="parent.display_type != 'color'" invisible="image" widget="color"/>
                                <field name="image"
                                       class="oe_avatar text-start float-none"
                                       column_invisible="parent.display_type != 'color'"
                                       options="{'size': [70, 70]}"
                                       widget="image"/>
                                <field name="default_extra_price"/>
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
        <field name="arch" type="xml">
            <tree string="Attributes" create="0" delete="0">
                <field name="attribute_id" optional="hide"/>
                <field name="name"/>
                <field name="display_type" optional="hide"/>
                <field name="html_color" invisible="display_type != 'color' or image" widget="color"/>
                <field name="image" invisible="display_type != 'color'" widget="image" options="{'size': [70, 70]}"/>
                <field name="ptav_active" optional="hide"/>
                <field name="price_extra" widget="monetary" options="{'field_digits': True}"/>
                <field name="currency_id" column_invisible="True"/>
            </tree>
        </field>
    </record>

    <record id="product_template_attribute_value_view_form" model="ir.ui.view">
        <field name="name">product.template.attribute.value.view.form.</field>
        <field name="model">product.template.attribute.value</field>
        <field name="arch" type="xml">
            <form string="Product Attribute" create="0" delete="0">
                <sheet>
                    <group>
                        <field name="ptav_active" readonly="1" invisible="ptav_active"/>
                        <field name="name"/>
                        <field name="display_type" invisible="1"/>
                        <field name="html_color" invisible="display_type != 'color' or image"/>
                        <field name="image" invisible="display_type != 'color' or not image" widget="image" options="{'size': [70, 70]}"/>
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
                            context="{'search_default_categ_id': id, 'default_categ_id': id, 'group_expand': True}">
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
        <field name="res_model">product.category</field>
        <field name="search_view_id" ref="product_category_search_view"/>
        <field name="view_id" ref="product_category_list_view"/>
    </record>

</odoo>

```

## File: views\product_document_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="product_document_form" model="ir.ui.view">
        <field name="name">product.document.form</field>
        <field name="model">product.document</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <field name="res_model" invisible="True"/>
                    <label for="name"/>
                    <h1>
                        <field name="name" readonly="not datas and type != 'url'"/>
                    </h1>
                    <group>
                        <field name="type" readonly="datas or id" class="oe_inline"/>
                        <field name="datas"
                               filename="name"
                               invisible="type == 'url'"
                               required="type == 'binary'"
                               class="oe_inline"/>
                        <field name="url" widget="url" invisible="type == 'binary'" required="type == 'url'"/>
                        <field name="res_name" string="Product Variant" invisible="res_model != 'product.product'"/>
                    </group>
                    <group string="Attached To" groups="base.group_no_one">
                        <field name="res_name"/>
                        <field name="company_id"
                               groups="base.group_multi_company"
                               options="{'no_create': True}"
                               class="oe_inline"/>
                    </group>
                    <group string="History" groups="base.group_no_one" invisible="not create_date">
                        <label for="create_uid" string="Creation"/>
                        <div name="creation_div">
                            <field name="create_uid" readonly="1" class="oe_inline"/> on
                            <field name="create_date" readonly="1" class="oe_inline"/>
                        </div>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="product_document_kanban" model="ir.ui.view">
        <field name="name">product.document.kanban</field>
        <field name="model">product.document</field>
        <field name="arch" type="xml">
            <kanban js_class="product_documents_kanban">
                <field name="ir_attachment_id"/>
                <field name="mimetype"/>
                <field name="type"/>
                <field name="name"/>
                <field name="create_uid"/>
                <field name="active"/>
                <templates>
                    <t t-name="kanban-menu">
                        <a t-if="widget.editable" type="edit" class="dropdown-item">Edit</a>
                        <a t-if="widget.deletable" type="delete" class="dropdown-item">Delete</a>
                        <a t-attf-href="/web/content/#{record.ir_attachment_id.raw_value}?download=true" download="" class="dropdown-item">Download</a>
                    </t>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_area o_kanban_attachment">
                            <field name="res_model" invisible="True"/>
                            <div class="ribbon ribbon-top-right" invisible="active">
                                <span class="text-bg-danger">Archived</span>
                            </div>
                            <div class="ribbon ribbon-top-right" invisible="not active or res_model != 'product.product'">
                                <span class="text-bg-secondary">Variant</span>
                            </div>
                            <div class="o_kanban_image">
                                <t t-set="webimage" t-value="new RegExp('image.*(gif|jpeg|jpg|png|webp)').test(record.mimetype.value)"/>
                                <t t-set="binaryPreviewable"
                                   t-value="new RegExp('(image|video|application/pdf|text)').test(record.mimetype.value) &amp;&amp; record.type.raw_value === 'binary'"/>
                                <div t-attf-class="o_kanban_image_wrapper #{(webimage or binaryPreviewable) ? 'o_kanban_previewer' : ''}">
                                    <div t-if="record.type.raw_value == 'url'" class="o_url_image fa fa-link fa-3x text-muted" aria-label="Image is a link"/>
                                    <img t-elif="webimage" t-attf-src="/web/image/#{record.ir_attachment_id.raw_value}" width="100" height="100" alt="Document" class="o_attachment_image"/>
                                    <div t-else="" class="o_image o_image_thumbnail" t-att-data-mimetype="record.mimetype.value"/>
                                </div>
                            </div>
                            <div class="o_kanban_details h-100">
                                <div class="o_kanban_details_wrapper h-100 justify-content-between">
                                    <div class="o_kanban_record_title">
                                        <field name="name" class="o_text_overflow"/>
                                    </div>
                                    <div class="o_kanban_record_body"
                                         name="body"
                                         t-if="record.type.raw_value == 'url'">
                                        <field name="url" widget="url" invisible="type == 'binary'"/>
                                    </div>
                                    <div class="o_kanban_record_bottom flex-column" name="bottom">
                                        <div class="mt-2" invisible="res_model != 'product.product'">
                                            <span>Variant </span>
                                            <field name="res_name" style="font-style: italic; text-decoration-line: underline;"/>
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

    <record id="product_document_list" model="ir.ui.view">
        <field name="name">product.document.list</field>
        <field name="model">product.document</field>
        <field name="arch" type="xml">
            <tree editable="top" multi_edit="true">
                <field name="name"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="product_document_search" model="ir.ui.view">
        <field name="name">product.document.search</field>
        <field name="model">product.document</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                <filter
                    string="Documents of this variant"
                    name="context_variant"
                    domain="[('res_model', '=', 'product.product'), ('res_id', '=', context.get('default_res_id'))]"/>
            </search>
        </field>
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
                            <field name="product_id" readonly="id"/>
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
        <field name="domain">[]</field>
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
                <filter string="Active" name="visible" domain="[('pricelist_id.active', '=', True)]"/>
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
                <field name="company_id" column_invisible="True"/>
                <field name="categ_id" column_invisible="True"/>
                <field name="product_tmpl_id"
                  column_invisible="context.get('active_model') != 'product.category'"
                  required="applied_on == '1_product'"
                  domain="[('categ_id', '=', context.get('default_categ_id', True)), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"
                  options="{'no_create_edit':1, 'no_open': 1}"/>
                <field name="product_id"
                  groups="product.group_product_variant"
                  readonly="context.get('active_model') == 'product.product'"
                  column_invisible="context.get('product_without_variants', False)"
                  required="applied_on == '0_product_variant'"
                  domain="['|', '|',
                    ('id', '=', context.get('default_product_id', 0)),
                    ('product_tmpl_id', '=', context.get('default_product_tmpl_id', 0)),
                    ('categ_id', '=', context.get('default_categ_id', 0)), '|', ('company_id', '=', company_id), ('company_id', '=', False)
                  ]"
                  options="{'no_create_edit':1, 'no_open': 1}"/>
                <field name="min_quantity" colspan="4"/>
                <field name="currency_id" column_invisible="True"/>
                <field name="fixed_price" string="Price" required='1'/>
                <field name="date_start" optional="show"/>
                <field name="date_end" optional="show"/>
                <field name="applied_on" column_invisible="True"/>
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
                            <field name="fixed_price"
                                widget="monetary"
                                invisible="compute_price != 'fixed'"
                                options="{'field_digits': True}"/>
                            <label for="percent_price" string="Discount" invisible="compute_price != 'percentage'"/>
                            <div class="o_row" invisible="compute_price != 'percentage'">
                                <field name="percent_price" class="oe_inline" invisible="compute_price != 'percentage'"/>%
                            </div>
                            <field name="base" invisible="compute_price != 'formula'"/>
                            <field name="base_pricelist_id" invisible="compute_price != 'formula' or base != 'pricelist'" readonly="base != 'pricelist'" required="compute_price == 'formula' and base == 'pricelist'"/>
                            <label for="price_discount" string="Discount" invisible="compute_price != 'formula'"/>
                            <div class="o_row" invisible="compute_price != 'formula'">
                                <field name="price_discount"/>
                                <span>%</span>
                            </div>
                            <field name="price_surcharge" widget="monetary" string="Extra Fee" invisible="compute_price != 'formula'"/>
                            <field name="price_round" string="Rounding Method" invisible="compute_price != 'formula'"/>
                            <label string="Margins" for="price_min_margin" invisible="compute_price != 'formula'"/>
                            <div class="o_row" invisible="compute_price != 'formula'">
                                <field name="price_min_margin"
                                    string="Min. Margin"
                                    class="oe_inline"
                                    widget="monetary"
                                    nolabel="1"
                                    options="{'field_digits': True}"/>
                                <i class="fa fa-long-arrow-right mx-2 oe_edit_only" aria-label="Arrow icon" title="Arrow"/>
                                <field name="price_max_margin"
                                    string="Max. Margin"
                                    class="oe_inline"
                                    widget="monetary"
                                    nolabel="1"
                                    options="{'field_digits': True}"/>
                            </div>
                        </group>
                        <div class="alert alert-info" role="alert" style="white-space: pre;" invisible="compute_price != 'formula'">
                            <field name="rule_tip"/>
                        </div>
                    </group>

                    <group string="Conditions">
                        <group name="pricelist_rule_target">
                            <field name="applied_on" widget="radio"/>
                            <field name="categ_id" options="{'no_create':1}" invisible="applied_on != '2_product_category'" required="applied_on == '2_product_category'"/>
                            <field name="product_tmpl_id" options="{'no_create':1}" invisible="applied_on != '1_product'" required="applied_on == '1_product'"/>
                            <field name="product_id" options="{'no_create':1}" invisible="applied_on != '0_product_variant'" required="applied_on == '0_product_variant'"/>
                        </group>
                        <group name="pricelist_rule_limits">
                            <field name="min_quantity"/>
                            <field name="date_start" string="Validity" widget="daterange" options="{'end_date_field': 'date_end'}" />
                            <field name="date_end" invisible="1" />
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
                <field name="discount_policy"/>
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
                            <field name="discount_policy"/>
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
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
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
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="pricelist_id" column_invisible="True"/>
                                    <!-- Pricelist ID is here only for related fields to be correctly computed -->
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="base" column_invisible="True"/>
                                    <field name="applied_on" column_invisible="True"/>
                                    <field name="company_id" column_invisible="True"/>
                                </tree>
                                <!-- When in advanced pricelist mode : pricelist rules
                                    Should open in a form view and not be editable inline anymore.
                                -->
                                <tree groups="product.group_sale_pricelist" string="Pricelist Rules">
                                    <field name="product_tmpl_id" column_invisible="True"/>
                                    <field name="name" string="Applicable On"/>
                                    <field name="min_quantity"/>
                                    <field name="price" string="Price"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="base" column_invisible="True"/>
                                    <field name="price_discount" column_invisible="True"/>
                                    <field name="applied_on" column_invisible="True"/>
                                    <field name="compute_price" column_invisible="True"/>
                                </tree>
                            </field>
                        </page>
                        <page name="pricelist_config" string="Configuration">
                            <group>
                                <group name="pricelist_availability" string="Availability">
                                    <field name="country_group_ids" widget="many2many_tags"/>
                                </group>
                                <group name="pricelist_discounts" string="Discounts">
                                    <field name="discount_policy" widget="radio"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="activity_ids"/>
                    <field name="message_follower_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="product_pricelist_action2" model="ir.actions.act_window">
        <field name="name">Pricelists</field>
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
                            <field name="discount"/>
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
                <field name="product_name"/>
                <field name="product_code"/>
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
                <field name="product_id" optional="hide"
                    readonly="1"
                    column_invisible="context.get('product_template_invisible_variant', False)"
                    groups="product.group_product_variant"
                    domain="[('product_tmpl_id', '=', context.get('default_product_tmpl_id'))] if context.get('default_product_tmpl_id') else [('product_tmpl_id', '=', product_tmpl_id)]"/>
                <field name="product_tmpl_id" string="Product"
                    readonly="1"
                    column_invisible="context.get('visible_product_tmpl_id', True)"/>
                <field name="product_name" optional="hide"/>
                <field name="product_code" optional="hide"/>
                <field name="date_start" optional="hide"/>
                <field name="date_end" optional="hide"/>
                <field name="company_id" readonly="1" groups="base.group_multi_company"/>
                <field name="min_qty" optional="hide"/>
                <field name="product_uom" groups="uom.group_uom" optional="hide"/>
                <field name="price" string="Price"/>
                <field name="discount" optional="hide"/>
                <field name="currency_id" groups="base.group_multi_currency"/>
                <field name="delay" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="product_supplierinfo_type_action" model="ir.actions.act_window">
        <field name="name">Vendor Pricelists</field>
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
                        <field name="name"/>
                    </group>
                    <notebook>
                        <page string="Product Templates" name="page_product_templates">
                            <field name="product_template_ids"
                                   context="{'tree_view_ref':'product.product_template_view_tree_tag'}"
                            />
                        </page>
                        <page string="Product Variants" name="page_product_variants">
                            <field name="product_product_ids"
                                   context="{'tree_view_ref':'product.product_product_view_tree_tag'}"
                            />
                        </page>
                        <page string="All Products" name="page_all_products" invisible="not product_ids">
                            <field name="product_ids"
                                   context="{'tree_view_ref':'product.product_product_view_tree_tag'}"
                            />
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record id="product_tag_tree_view" model="ir.ui.view">
        <field name="name">product.tag.tree</field>
        <field name="model">product.tag</field>
        <field name="arch" type="xml">
            <tree string="Product Tags" multi_edit="1">
                <field name="name"/>
                <field name="product_template_ids" widget="many2many_tags" optional="show"/>
                <field name="product_product_ids" widget="many2many_tags" string="Product Variant" optional="show"/>
            </tree>
        </field>
    </record>
    <record id="product_tag_action" model="ir.actions.act_window">
        <field name="name">Product Tags</field>
        <field name="res_model">product.tag</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" eval="False"/>
        <field name="context">{'create': True}</field>
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
                <field name="product_variant_count" column_invisible="True"/>
                <field name="sale_ok" column_invisible="True"/>
                <field name="currency_id" column_invisible="True"/>
                <field name="cost_currency_id" column_invisible="True"/>
                <field name="priority" widget="priority" optional="show" nolabel="1"/>
                <field name="name" string="Product Name"/>
                <field name="default_code" optional="show"/>
                <field name="product_tag_ids" widget="many2many_tags" optional="show"/>
                <field name="barcode" optional="hide" readonly="product_variant_count != 1"/>
                <field name="company_id" options="{'no_create': True}"
                    groups="base.group_multi_company" optional="hide"/>
                <field name="list_price" string="Sales Price" widget='monetary' options="{'currency_field': 'currency_id'}" optional="show" decoration-muted="not sale_ok"/>
                <field name="standard_price" widget='monetary' options="{'currency_field': 'cost_currency_id'}" optional="show" readonly="1"/>
                <field name="categ_id" optional="hide"/>
                <field name="detailed_type" optional="hide" readonly="1"/>
                <field name="type" column_invisible="True"/>
                <field name="uom_id" string="Unit" readonly="1" optional="show" groups="uom.group_uom"/>
                <field name="active" column_invisible="True"/>
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
                <field name="default_code" invisible="product_variant_count &gt; 1"/>
                <field name="valid_product_template_attribute_line_ids" invisible="1"/>
                <field name="barcode" invisible="product_variant_count &gt; 1 or (product_variant_count == 0 and valid_product_template_attribute_line_ids)"/>
            </field>

            <div name="button_box" position="inside">
                <button name="%(product.product_variant_action)d" type="action"
                    icon="fa-sitemap" class="oe_stat_button"
                    invisible="product_variant_count &lt;= 1"
                    groups="product.group_product_variant">
                    <field string="Variants" name="product_variant_count" widget="statinfo" />
                </button>
            </div>

            <xpath expr="//page[@name='general_information']" position="after">
                <page name="variants" string="Attributes &amp; Variants" groups="product.group_product_variant">
                    <field name="attribute_line_ids" widget="one2many" context="{'show_attribute': False}">
                        <tree string="Variants" editable="bottom" decoration-info="value_count &lt;= 1">
                            <field name="value_count" column_invisible="True"/>
                            <field name="sequence" widget="handle"/>
                            <field name="attribute_id" readonly="id"/>
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
            <xpath expr="//group[@name='group_standard_price']" position="after">
                <field name="product_properties" columns="2"/>
            </xpath>
        </field>
    </record>

    <record id="product_template_kanban_view" model="ir.ui.view">
        <field name="name">Product.template.product.kanban</field>
        <field name="model">product.template</field>
        <field name="arch" type="xml">
            <kanban sample="1">
                <field name="id"/>
                <field name="product_variant_count"/>
                <field name="currency_id"/>
                <field name="activity_state"/>
                <field name="categ_id"/>
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
                                <div>
                                    <field name="product_properties" widget="properties"/>
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
                        <div class="ms-2">
                            <field name="name" display="full" class="o_text_block"/>
                            <div t-if="record.default_code.value" class="o_text_block text-muted">
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
    'tag': 'generate_pricelist_report',
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
                    <button string="Print Labels" type="object" name="action_open_label_layout" invisible="detailed_type not in ['consu', 'product', 'combo']"/>
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
                                    <span class="o_stat_text" invisible="pricelist_item_count == 1">
                                        Extra Prices
                                    </span>
                                    <span class="o_stat_text" invisible="pricelist_item_count != 1">
                                        Extra Price
                                    </span>
                               </div>
                        </button>
                        <button class="oe_stat_button"
                                name="action_open_documents"
                                type="object"
                                icon="fa-file-text-o">
                            <field string="Documents" name="product_document_count" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <field name="id" invisible="True"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="name" string="Product Name"/>
                        <h1>
                            <div class="d-flex">
                                <field name="priority" widget="priority" class="me-3"/>
                                <field class="text-break" name="name" options="{'line_breaks': False}" widget="text" placeholder="e.g. Cheese Burger"/>
                            </div>
                        </h1>
                    </div>
                    <div name="options">
                        <span class="d-inline-flex">
                            <field name="sale_ok"/>
                            <label for="sale_ok"/>
                        </span>
                        <span class="d-inline-flex">
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
                                    <field name="product_tooltip" string="" class="fst-italic text-muted" invisible="type == 'service' and not sale_ok"/>
                                    <field name="uom_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                    <field name="uom_po_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                </group>
                                <group name="group_standard_price">
                                    <label for="list_price"/>
                                    <div name="pricing" class="o_row">
                                      <field name="list_price" class="oe_inline" widget='monetary'
                                        options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                                    </div>

                                    <!--
                                        `id` condition to prevent invisibility on new products
                                        when `product_variant_count` is 0.
                                        `product_variant_count != 1` to handle cases with dynamic
                                        / multiple variants since id will be set after saving.
                                        #TODO : revert this and create a compute field in master
                                        to ensure invisivility for dynamic variants only.
                                    -->
                                    <label for="standard_price" invisible="id and product_variant_count != 1 and not is_product_variant"/>
                                    <div name="standard_price_uom" invisible="id and product_variant_count != 1 and not is_product_variant">
                                        <field name="standard_price" class="oe_inline" widget='monetary' options="{'currency_field': 'cost_currency_id', 'field_digits': True}"/>
                                        <span groups="uom.group_uom" >per
                                            <field name="uom_name" class="oe_inline"/>
                                        </span>
                                    </div>
                                    <field name="categ_id" string="Product Category"/>
                                    <field name="product_tag_ids" widget="many2many_tags" context="{'product_template_id': id}"/>
                                    <button name="%(product_tag_action)d" icon="oi-arrow-right" type="action" class="btn-link ps-0" colspan="2" string="Configure tags"/>
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
                        <page string="Sales" name="sales" invisible="1 or not sale_ok">
                            <group name="sale">
                                <group string="Upsell &amp; Cross-Sell" name="upsell" invisible="1"/>
                            </group>
                            <group>
                                <group string="Sales Description" name="description">
                                    <field colspan="2" name="description_sale" nolabel="1" placeholder="This note is added to sales orders and invoices."/>
                                </group>
                            </group>
                        </page>
                        <page string="Purchase" name="purchase" invisible="1 or not purchase_ok">
                            <group name="purchase">
                                <group string="Vendor Bills" name="bill"/>
                            </group>
                        </page>
                        <page string="Inventory" name="inventory" groups="product.group_stock_packaging" invisible="type == 'service'">
                            <group name="inventory">
                                <group name="group_lots_and_weight" string="Logistics" invisible="type not in ['product', 'consu']">
                                    <label for="weight" invisible="product_variant_count &gt; 1 and not is_product_variant"/>
                                    <div class="o_row" name="weight" invisible="product_variant_count &gt; 1 and not is_product_variant">
                                        <field name="weight" class="oe_inline"/>
                                        <field name="weight_uom_name"/>
                                    </div>
                                    <label for="volume" invisible="product_variant_count &gt; 1 and not is_product_variant"/>
                                    <div class="o_row" name="volume" invisible="product_variant_count &gt; 1 and not is_product_variant">
                                        <field name="volume" string="Volume" class="oe_inline"/>
                                        <field name="volume_uom_name"/>
                                    </div>
                                </group>
                            </group>
                            <group name="packaging" string="Packaging"
                                colspan="4"
                                invisible="(type not in ['product', 'consu'] or product_variant_count &gt; 1) and not is_product_variant"
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
                        <field name="detailed_type" invisible="1"/>
                        <button string="Print Labels" type="object" name="action_open_label_layout" invisible="detailed_type not in ['consu', 'product', 'combo']"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box"/>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
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
                                    <field name="lst_price" class="oe_inline" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}" readonly="product_variant_count &gt; 1"/>
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
                            <group name="weight" string="Logistics" invisible="type not in ['product', 'consu']">
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
                                <field name="product_tag_ids" widget="many2many_tags" readonly="1"/>
                                <field name="additional_product_tag_ids" widget="many2many_tags" context="{'product_variant_id': id}"/>
                                <button name="%(product_tag_action)d" icon="oi-arrow-right" type="action" class="btn-link ps-0" colspan="2" string="Configure tags"/>
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
                    <field name="product_tag_ids" widget="many2many_tags" readonly="1" optional="hide"/>
                    <field name="additional_product_tag_ids" widget="many2many_tags" optional="hide"/>
                    <field name="type" optional="hide" readonly="1"/>
                    <field name="uom_id" string="Unit" groups="uom.group_uom" optional="show" readonly="1"/>
                    <field name="product_tmpl_id" readonly="1" column_invisible="True"/>
                    <field name="active" column_invisible="True"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="product_template_view_tree_tag">
            <field name="name">product.template.view.tree.tag</field>
            <field name="model">product.template</field>
            <field name="arch" type="xml">
                <tree string="Product Templates" editable="bottom">
                    <field name="name" readonly="1"/>
                    <field name="default_code" readonly="1" optional="show"/>
                    <field name="description" readonly="1" optional="show"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="product_product_view_tree_tag">
            <field name="name">product.product.view.tree.tag</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <tree string="Product Variants" editable="bottom">
                    <field name="name" readonly="1"/>
                    <field name="default_code" readonly="1" optional="show"/>
                    <field name="product_template_variant_value_ids"
                           widget="many2many_tags"
                           groups="product.group_product_variant"
                           readonly="1"
                    />
                    <field name="description" readonly="1" optional="show"/>
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
                   <attribute name="invisible">1</attribute>
                   <attribute name="readonly">product_variant_count &gt; 1</attribute>
                   </field>
                <xpath expr="//label[@for='list_price']" position="replace">
                    <label for="lst_price"/>
                </xpath>
                <field name="list_price" position="after">
                   <field name="lst_price" class="oe_inline" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                </field>
                <group name="packaging" position="attributes">
                    <attribute name="invisible">0</attribute>
                </group>
                <field name="name" position="after">
                    <field name="product_tmpl_id" class="oe_inline" readonly="1" invisible="1" required="id"/>
                </field>
                <xpath expr="//div[hasclass('oe_title')]" position="inside">
                    <field name="product_template_variant_value_ids" widget="many2many_tags" readonly="1" groups="product.group_product_variant"/>
                </xpath>
                <field name="product_tag_ids" position="attributes">
                    <attribute name="options">{'no_open': True}</attribute>
                    <attribute name="context">{'product_template_id': product_tmpl_id}</attribute>
                </field>
                <field name="product_tag_ids" position="after">
                    <field name="additional_product_tag_ids" widget="many2many_tags"/>
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
                            <div class="oe_kanban_card oe_kanban_global_click">
                                <div class="o_kanban_image me-1">
                                    <img t-att-src="kanban_image('product.product', 'image_128', record.id.raw_value)" alt="Product" class="o_image_64_contain"/>
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
                                    <div class="o_kanban_tags_section">
                                        <field name="product_template_variant_value_ids" groups="product.group_product_variant" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                    </div>
                                    <div name="product_lst_price" class="mt-1">
                                        Price: <field name="lst_price"></field>
                                    </div>
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
                            <div class="ms-2">
                                <field name="name" display="full" class="o_text_block"/>
                                <div t-if="record.default_code.value" class="o_text_block text-muted">
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
        <field name="name">Generate Pricelist Report</field>
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
    'tag': 'generate_pricelist_report',
    'context': ctx,
}
        </field>
    </record>

    <record id="product_view_kanban_catalog" model="ir.ui.view">
        <field name="name">product.view.kanban.catalog</field>
        <field name="model">product.product</field>
        <field name="arch" type="xml">
            <kanban records_draggable="0" js_class="product_kanban_catalog">
                <field name="id" invisible="1"/>
                <field name="default_code" invisible="1"/>
                <templates>
                    <t t-name="kanban-menu">
                        <div class="o_product_catalog_cancel_global_click">
                            <a role="menuitem" type="edit" class="dropdown-item border-top-0">Edit</a>
                        </div>
                    </t>
                    <t t-name="kanban-box">
                        <div class="d-flex flex-grow-1">
                            <div class="o_kanban_image">
                                <img t-att-src="kanban_image('product.product', 'image_128', record.id.raw_value)"
                                     alt="Product"/>
                            </div>
                            <div class="oe_kanban_details p-2 d-flex">
                                <div class="o_kanban_record_top flex-column m-0"
                                     t-attf-id="product-{{record.id.raw_value}}">
                                    <div class="d-flex">
                                        <field style="margin-top: 2px;" class="me-1" name="priority" widget="priority"/>
                                        <h4 class="text-reset">
                                            <strong class="o_kanban_record_title"><field name="name"/></strong>
                                        </h4>
                                    </div>
                                    <div t-if="record.default_code.value">
                                        [<field name="default_code"/>]
                                    </div>
                                    <!-- Used by @web/product/js/product_catalog/order_line to
                                         show the price using a t-portal. -->
                                    <div name="o_kanban_price"
                                         t-attf-id="product-{{record.id.raw_value}}-price"
                                         class="d-flex flex-column"/>
                                    <field name="product_template_attribute_value_ids"
                                           widget="many2many_tags"
                                           domain="[('id', 'in', parent.ids)]"
                                           groups="product.group_product_variant"
                                           options="{'color_field': 'color'}"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="product_view_search_catalog" model="ir.ui.view">
        <field name="name">product.view.search.catalog</field>
        <field name="model">product.product</field>
        <field name="arch" type="xml">
            <search string="Product">
                <!-- Search field -->
                <field name="name"
                       string="Product"
                       filter_domain="['|', '|', ('default_code', 'ilike', self), ('name', 'ilike', self), ('barcode', 'ilike', self)]"/>
                <field name="categ_id" filter_domain="[('categ_id', 'child_of', raw_value)]"/>
                <field name="product_template_attribute_value_ids"
                       groups="product.group_product_variant"/>
                <field name="product_tmpl_id" string="Product Template"/>
                <!-- Filter -->
                <filter string="Favorites" name="favorites" domain="[('priority', '=', '1')]"/>
                <separator/>
                <filter string="Services" name="services" domain="[('type', '=', 'service')]"/>
                <filter string="Products"
                        name="products"
                        domain="[('type', 'in', ['consu', 'product'])]"/>
                <!-- Group By -->
                <group expand="1" string="Group By">
                    <filter string="Product Type" name="type" context="{'group_by':'type'}"/>
                    <filter string="Product Category"
                            name="categ_id"
                            context="{'group_by':'categ_id'}"/>
                </group>
                <!-- searchpanel -->
                <searchpanel>
                    <field name="categ_id"
                           string="Product Category"
                           icon="fa-th-list"/>
                    <field name="product_template_attribute_value_ids"
                           string="Attributes"
                           icon="fa-th-list"
                           domain="[('ptav_active', '=', True), ('product_tmpl_id.active', '=', True)]"
                           enable_counters="1"
                           select="multi"/>
                </searchpanel>
            </search>
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
                    <block title="Units of Measure" id="product_general_settings">
                        <setting id="weight_uom_setting" string="Weight" help="Define your weight unit of measure">
                            <field name="product_weight_in_lbs" class="o_light_label" widget="radio" options="{'horizontal': true}"/>
                        </setting>
                        <setting id="manage_volume_uom_setting" string="Volume" help="Define your volume unit of measure">
                            <field name="product_volume_volume_in_cubic_feet" class="o_light_label" widget="radio" options="{'horizontal': true}"/>
                        </setting>
                    </block>
                </xpath>
                 <xpath expr="//div[@id='product_get_pic_setting']" position="replace">
                    <setting id="product_get_pic_setting" string="Google Images" help="Get product pictures using Barcode" documentation="/applications/sales/sales/products_prices/products/product_images.html">
                        <field name="module_product_images"/>
                        <div class="content-group mt16"
                            invisible="not module_product_images"
                            id="msg_module_product_images">
                            <div class="mt16 text-warning">
                                <strong>Save</strong> this page and come back
                                here to set up the feature.
                            </div>
                        </div>
                    </setting>
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
                    <field name="property_product_pricelist" groups="product.group_product_pricelist" invisible="not is_company and parent_id"/>
                    <div name="parent_pricelists" groups="product.group_product_pricelist" colspan="2" invisible="is_company or not parent_id">
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
    pricelist_id = fields.Many2one('product.pricelist', string="Pricelist")

    @api.depends('print_format')
    def _compute_dimensions(self):
        for wizard in self:
            if 'x' in wizard.print_format:
                columns, rows = wizard.print_format.split('x')[:2]
                wizard.columns = columns.isdigit() and int(columns) or 1
                wizard.rows = rows.isdigit() and int(rows) or 1
            else:
                wizard.columns, wizard.rows = 1, 1

    def _prepare_report_data(self):
        if self.custom_quantity <= 0:
            raise UserError(_('You need to set a positive quantity.'))

        # Get layout grid
        if self.print_format == 'dymo':
            xml_id = 'product.report_product_template_label_dymo'
        elif 'x' in self.print_format:
            xml_id = 'product.report_product_template_label_%sx%s' % (self.columns, self.rows)
            if 'xprice' not in self.print_format:
                xml_id += '_noprice'
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
        report_action = self.env.ref(xml_id).report_action(None, data=data, config=False)
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
                        <field name="pricelist_id" groups="product.group_product_pricelist"/>
                        <field name="extra_html" widget="html" invisible="print_format != '2x7xprice'"/>
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

