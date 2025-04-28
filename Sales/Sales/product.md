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
from . import wizard

```

## File: __manifest__.py

```python
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
        'wizard/update_product_attribute_value_views.xml',
        'views/product_tag_views.xml',
        'views/product_views.xml',  # To keep after product_tag_views.xml because it depends on it.

        'views/res_config_settings_views.xml',
        'views/product_attribute_views.xml',
        'views/product_attribute_value_views.xml',
        'views/product_category_views.xml',
        'views/product_combo_views.xml',
        'views/product_document_views.xml',
        'views/product_packaging_views.xml',
        'views/product_pricelist_item_views.xml',
        'views/product_pricelist_views.xml',
        'views/product_supplierinfo_views.xml',
        'views/product_template_attribute_line_views.xml',
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
        'data/product_attribute_demo.xml',
        'data/product_category_demo.xml',
        'data/product_demo.xml',
        'data/product_document_demo.xml',
        'data/product_supplierinfo_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'product/static/src/js/**/*',
            'product/static/src/product_catalog/**/*.js',
            'product/static/src/product_catalog/**/*.xml',
            'product/static/src/product_catalog/**/*.scss',
            'product/static/src/scss/product_form.scss',
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

## File: controllers\pricelist_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import csv
import io
import json

import xlsxwriter

from odoo import _
from odoo.http import Controller, request, route, content_disposition


class ProductPricelistExportController(Controller):

    @route('/product/export/pricelist/', type='http', auth='user')
    def export_pricelist(self, report_data, export_format):
        json_data = json.loads(report_data)
        report_data = request.env['report.product.report_pricelist']._get_report_data(json_data)
        pricelist_name = report_data['pricelist']['name']
        quantities = report_data['quantities']
        products = report_data['products']
        headers = [
            _("Product"),
            _("UOM"),
        ] + [_("Quantity (%s UoM)", qty) for qty in quantities]
        if export_format == 'csv':
            return self._generate_csv(pricelist_name, quantities, products, headers)
        else:
            return self._generate_xlsx(pricelist_name, quantities, products, headers)

    def _generate_rows(self, products, quantities):
        rows = []
        for product in products:
            variants = product.get('variants', [product])
            for variant in variants:
                row = [
                    variant['name'],
                    variant['uom']
                ] + [variant['price'].get(qty, 0.0) for qty in quantities]
                rows.append(row)
        return rows

    def _generate_csv(self, pricelist_name, quantities, products, headers):
        buffer = io.StringIO()
        writer = csv.writer(buffer)
        writer.writerow(headers)
        rows = self._generate_rows(products, quantities)
        writer.writerows(rows)
        content = buffer.getvalue()
        buffer.close()
        headers = [
            ('Content-Type', 'text/csv'),
            ('Content-Disposition', content_disposition(f'Pricelist - {pricelist_name}.csv'))
        ]
        return request.make_response(content, headers)

    def _generate_xlsx(self, pricelist_name, quantities, products, headers):
        buffer = io.BytesIO()
        workbook = xlsxwriter.Workbook(buffer, {'in_memory': True})
        worksheet = workbook.add_worksheet()
        worksheet.write_row(0, 0, headers)
        rows = self._generate_rows(products, quantities)
        column_widths = [len(header) for header in headers]
        for row_idx, row in enumerate(rows, start=1):
            worksheet.write_row(row_idx, 0, row)
            for col_idx, cell_value in enumerate(row):
                column_widths[col_idx] = max(column_widths[col_idx], len(str(cell_value)))

        for col_idx, width in enumerate(column_widths):
            worksheet.set_column(col_idx, col_idx, width)
        workbook.close()
        content = buffer.getvalue()
        buffer.close()
        headers = [
            ('Content-Type', 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'),
            ('Content-Disposition', content_disposition(f'Pricelist - {pricelist_name}.xlsx'))
        ]
        return request.make_response(content, headers)

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
    def upload_document(self, ufile, res_model, res_id, **kwargs):
        if not self.is_model_valid(res_model):
            return

        record = request.env[res_model].browse(int(res_id)).exists()

        if not record or not record.browse().has_access('write'):
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
                    **self.get_additional_create_params(**kwargs)
                })
            except Exception as e:
                logger.exception("Failed to upload document %s", ufile.filename)
                result = {'error': str(e)}
        return json.dumps(result)

    # mrp hook
    def get_additional_create_params(self, **kwargs):
        return {}

    # eco hook
    def is_model_valid(self, res_model):
        return res_model in ('product.product', 'product.template')

```

## File: controllers\__init__.py

```python
from . import catalog
from . import product_document
from . import pricelist_report

```

## File: data\product_attribute_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

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
    <record id="product_attribute_value_color_wood" model="product.attribute.value">
        <field name="name">Wood</field>
        <field name="attribute_id" ref="product_attribute_2"/>
        <field name="sequence">0</field>
        <field name="image" type="base64" file="product/static/img/wood.jpg"/>
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

    <record id="pa_extra_options" model="product.attribute">
        <field name="name">Extra Options</field>
        <field name="display_type">multi</field>
        <field name="create_variant">no_variant</field>
        <field name="sequence">40</field>
    </record>
    <record id="pav_warranty" model="product.attribute.value">
        <field name="name">1 year warranty extension</field>
        <field name="default_extra_price">50</field>
        <field name="attribute_id" ref="pa_extra_options"/>
    </record>
    <record id="pav_cleaning_kit" model="product.attribute.value">
        <field name="name">Cleaning kit</field>
        <field name="default_extra_price">5</field>
        <field name="attribute_id" ref="pa_extra_options"/>
    </record>
    <record id="pav_protection_kit" model="product.attribute.value">
        <field name="name">Protection kit</field>
        <field name="default_extra_price">10</field>
        <field name="attribute_id" ref="pa_extra_options"/>
    </record>

    <record id="size_attribute" model="product.attribute">
        <field name="name">Size</field>
        <field name="sequence">30</field>
        <field name="display_type">radio</field>
        <field name="create_variant">no_variant</field>
    </record>
    <record id="size_attribute_s" model="product.attribute.value">
        <field name="name">S</field>
        <field name="sequence">1</field>
        <field name="attribute_id" ref="size_attribute"/>
    </record>
    <record id="size_attribute_m" model="product.attribute.value">
        <field name="name">M</field>
        <field name="sequence">2</field>
        <field name="attribute_id" ref="size_attribute"/>
    </record>
    <record id="size_attribute_l" model="product.attribute.value">
        <field name="name">L</field>
        <field name="sequence">3</field>
        <field name="attribute_id" ref="size_attribute"/>
    </record>

    <record id="fabric_attribute" model="product.attribute">
        <field name="name">Fabric</field>
        <field name="sequence">40</field>
        <field name="display_type">select</field>
        <field name="create_variant">no_variant</field>
    </record>
    <record id="fabric_attribute_plastic" model="product.attribute.value">
        <field name="name">Plastic</field>
        <field name="sequence">1</field>
        <field name="attribute_id" ref="fabric_attribute"/>
    </record>
    <record id="fabric_attribute_leather" model="product.attribute.value">
        <field name="name">Leather</field>
        <field name="sequence">2</field>
        <field name="attribute_id" ref="fabric_attribute"/>
    </record>
    <record id="fabric_attribute_custom" model="product.attribute.value">
        <field name="name">Custom</field>
        <field name="sequence">3</field>
        <field name="attribute_id" ref="fabric_attribute"/>
        <field name="is_custom">True</field>
    </record>

</odoo>

```

## File: data\product_category_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

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
    <record id="product_category_construction" model="product.category">
        <field name="parent_id" ref="product_category_all"/>
        <field name="name">Home Construction</field>
    </record>
    <record id="product_category_outdoor_furniture" model="product.category">
        <field name="parent_id" ref="product_category_1"/>
        <field name="name">Outdoor furniture</field>
    </record>

</odoo>

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

        <!-- Expensable products -->
        <record id="expense_product" model="product.product">
            <field name="name">Restaurant Expenses</field>
            <field name="list_price">14.0</field>
            <field name="standard_price">8.0</field>
            <field name="type">service</field>
            <field name="categ_id" ref="product.cat_expense"/>
        </record>

        <record id="expense_hotel" model="product.product">
            <field name="name">Hotel Accommodation</field>
            <field name="list_price">400.0</field>
            <field name="standard_price">400.0</field>
            <field name="type">service</field>
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
            <field name="image_1920" type="base64" file="product/static/img/product_chair.jpg"/>
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

        <record id="product_product_4_product_template" model="product.template">
            <field name="name">Customizable Desk</field>
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
            <field name="type">consu</field>
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
            <field name="type">consu</field>
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
            <field name="image_1920" type="base64" file="product/static/img/product_product_9-image.jpg"/>
        </record>

        <record id="product_product_10" model="product.product">
            <field name="name">Cabinet with Doors</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">120.50</field>
            <field name="list_price">140</field>
            <field name="type">consu</field>
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
            <field name="type">consu</field>
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
            <field name="type">consu</field>
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
            <field name="type">consu</field>
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
            <field name="image_1920" type="base64" file="product/static/img/product_product_24-image.jpg"/>
        </record>

        <record id="product_template_acoustic_bloc_screens" model="product.template">
            <field name="name">Acoustic Bloc Screens</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">287.0</field>
            <field name="list_price">295.0</field>
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="product/static/img/product_product_25-image.png"/>
            <field name="attribute_line_ids" eval="[
                Command.create({
                    'attribute_id': ref('product.product_attribute_2'),
                    'value_ids': [
                        Command.set([
                            ref('product.product_attribute_value_3'),
                            ref('product.product_attribute_value_color_wood')
                        ])
                    ]
                })
            ]"/>
        </record>

        <record id="product_product_27" model="product.product">
            <field name="name">Drawer</field>
            <field name="categ_id" ref="product_category_5"/>
            <field name="standard_price">100.0</field>
            <field name="list_price">110.50</field>
            <field name="type">consu</field>
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
            <field name="standard_price">4500.0</field>
            <field name="list_price">4000.0</field>
            <field name="type">consu</field>
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
            <field name="type">consu</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Three Seater Sofa with Lounger in Steel Grey Colour</field>
            <field name="default_code">FURN_8999</field>
            <field name="image_1920" type="base64" file="product/static/img/product_product_d01-image.jpg"/>
        </record>

        <record id="product_product_local_delivery" model="product.product">
            <field name="name">Local Delivery</field>
            <field name="default_code">Delivery_010</field>
            <field name="type">service</field>
            <field name="categ_id" ref="product_category_3"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="list_price">10.0</field>
        </record>

        <record id="product_product_furniture" model="product.product">
            <field name="name">Furniture Assembly</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="list_price">2000.00</field>
            <field name="standard_price">2500.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
        </record>

        <record id="product_template_dining_table" model="product.template">
            <field name="name">Outdoor dining table</field>
            <field name="categ_id" ref="product_category_outdoor_furniture"/>
            <field name="list_price">589</field>
            <field name="image_1920" type="base64" file="product/static/img/dining_table.png"/>
            <field name="attribute_line_ids" eval="[
                Command.create({
                    'attribute_id': ref('pa_extra_options'),
                    'value_ids': [Command.set([ref('pav_warranty'), ref('pav_cleaning_kit'), ref('pav_protection_kit')])],
                }),
            ]"/>
        </record>

        <record id="desk_organizer" model="product.product">
            <field name="list_price">5.10</field>
            <field name="name">Desk Organizer</field>
            <field name="description_sale">The desk organiser is perfect for storing all kinds of small things and since the 5 boxes are loose, you can move and place them in the way that suits you and your things best.</field>
            <field name="default_code">FURN_0001</field>
            <field name="barcode">2300001000008</field>
            <field name="weight">0.01</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="product/static/img/desk_organizer.png"/>
        </record>
        <function model="ir.model.data" name="_update_xmlids">
            <value
                model="base"
                eval="[{
                    'xml_id': 'product.desk_organizer_product_template',
                    'record': obj().env.ref('product.desk_organizer').product_tmpl_id,
                    'noupdate': True,
                }]"
            />
        </function>
        <record id="desk_organizer_size" model="product.template.attribute.line">
            <field name="product_tmpl_id" ref="product.desk_organizer_product_template"/>
            <field name="attribute_id" ref="size_attribute"/>
            <field
                name="value_ids"
                eval="[(6, 0, [
                    ref('size_attribute_s'),
                    ref('size_attribute_m'),
                    ref('size_attribute_l'),
                ])]"
            />
        </record>
        <record id="desk_organizer_fabric" model="product.template.attribute.line">
            <field name="product_tmpl_id" ref="product.desk_organizer_product_template"/>
            <field name="attribute_id" ref="fabric_attribute"/>
            <field
                name="value_ids"
                eval="[(6, 0, [
                    ref('fabric_attribute_plastic'),
                    ref('fabric_attribute_leather'),
                    ref('fabric_attribute_custom'),
                ])]"
            />
        </record>

        <record id="desk_pad" model="product.product">
            <field name="list_price">1.98</field>
            <field name="name">Desk Pad</field>
            <field name="default_code">FURN_0002</field>
            <field name="weight">0.01</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="product/static/img/desk_pad.png"/>
        </record>

        <record id="monitor_stand" model="product.product">
            <field name="list_price">3.19</field>
            <field name="name">Monitor Stand</field>
            <field name="default_code">FURN_0006</field>
            <field name="weight">0.01</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="product/static/img/monitor_stand.png"/>
        </record>

        <record id="desk_accessories_combo" model="product.combo">
            <field name="name">Desk Accessories Combo</field>
            <field
                name="combo_item_ids"
                eval="[
                    Command.clear(),
                    Command.create({
                        'product_id': ref('product.desk_organizer'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('product.desk_pad'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('product.monitor_stand'),
                        'extra_price': 2,
                    }),
                ]"
            />
        </record>

        <record id="desks_combo" model="product.combo">
            <field name="name">Desks Combo</field>
            <field
                name="combo_item_ids"
                eval="[
                    Command.clear(),
                    Command.create({
                        'product_id': ref('product.product_product_3'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('product.product_product_5'),
                        'extra_price': 0,
                    }),
                ]"
            />
        </record>

        <record id="chairs_combo" model="product.combo">
            <field name="name">Chairs Combo</field>
            <field
                name="combo_item_ids"
                eval="[
                    Command.clear(),
                    Command.create({
                        'product_id': ref('product.product_product_11'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('product.product_product_11b'),
                        'extra_price': 0,
                    }),
                    Command.create({
                        'product_id': ref('product.product_product_12'),
                        'extra_price': 0,
                    }),
                ]"
            />
        </record>

        <record id="office_combo" model="product.product">
            <field name="list_price">160</field>
            <field name="name">Office Combo</field>
            <field name="type">combo</field>
            <field name="purchase_ok">False</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="image_1920" type="base64" file="product/static/img/office_combo.png"/>
            <field
                name="combo_ids"
                eval="[(6, 0, [
                    ref('desks_combo'),
                    ref('chairs_combo'),
                    ref('desk_accessories_combo'),
                ])]"
            />
        </record>

        <function model="product.template" name="_demo_configure_variants"/>
    </data>
</odoo>

```

## File: data\product_document_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <record id="product_product_4_product_template_document" model="product.document">
        <field name="name">Customizable Desk Document</field>
        <field name="datas" type="base64" file="product/static/demo/customizable_desk_document.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="res_model">product.template</field>
        <field name="res_id" ref="product.product_product_4_product_template"/>
    </record>

    <record id="product_product_25_document" model="product.document">
        <field name="name">Acoustic Bloc Screens Document</field>
        <field name="datas" type="base64" file="product/static/demo/acoustic_bloc_screen_document.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="res_model">product.product</field>
        <field name="res_id" ref="product.product_product_25"/>
    </record>
</odoo>

```

## File: data\product_supplierinfo_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

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
        <field name="product_tmpl_id" ref="product_template_acoustic_bloc_screens"/>
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
    active = fields.Boolean(
        default=True,
        help="If unchecked, it will allow you to hide the attribute without removing it.",
    )
    create_variant = fields.Selection(
        selection=[
            ('always', 'Instantly'),
            ('dynamic', 'Dynamically'),
            ('no_variant', 'Never'),
        ],
        default='always',
        string="Variant Creation",
        help="""- Instantly: All possible variants are created as soon as the attribute and its values are added to a product.
        - Dynamically: Each variant is created only when its corresponding attributes and values are added to a sales order.
        - Never: Variants are never created for the attribute.
        Note: this cannot be changed once the attribute is used on a product.""",
        required=True)
    display_type = fields.Selection(
        selection=[
            ('radio', 'Radio'),
            ('pills', 'Pills'),
            ('select', 'Select'),
            ('color', 'Color'),
            ('multi', 'Multi-checkbox'),
        ],
        default='radio',
        required=True,
        help="The display type used in the Product Configurator.")
    sequence = fields.Integer(string="Sequence", help="Determine the display order", index=True, default=20)

    value_ids = fields.One2many(
        comodel_name='product.attribute.value',
        inverse_name='attribute_id',
        string="Values", copy=True)
    template_value_ids = fields.One2many(
        comodel_name='product.template.attribute.value',
        inverse_name='attribute_id',
        string="Template Values")
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

    # === COMPUTE METHODS === #

    @api.depends('product_tmpl_ids')
    def _compute_number_related_products(self):
        res = {
            attribute.id: count
            for attribute, count in self.env['product.template.attribute.line']._read_group(
                domain=[('attribute_id', 'in', self.ids), ('product_tmpl_id.active', '=', 'True')],
                groupby=['attribute_id'],
                aggregates=['__count'],
            )
        }
        for pa in self:
            pa.number_related_products = res.get(pa.id, 0)

    @api.depends('attribute_line_ids.active', 'attribute_line_ids.product_tmpl_id')
    def _compute_products(self):
        templates_by_attribute = {
            attribute.id: templates
            for attribute, templates in self.env['product.template.attribute.line']._read_group(
                domain=[('attribute_id', 'in', self.ids)],
                groupby=['attribute_id'],
                aggregates=['product_tmpl_id:recordset']
            )
        }
        for pa in self:
            pa.with_context(active_test=False).product_tmpl_ids = templates_by_attribute.get(pa.id, False)

    # === ONCHANGE METHODS === #

    @api.onchange('display_type')
    def _onchange_display_type(self):
        if self.display_type == 'multi' and self.number_related_products == 0:
            self.create_variant = 'no_variant'

    # === CRUD METHODS === #

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

    # === ACTION METHODS === #

    def action_archive(self):
        for attribute in self:
            if attribute.number_related_products:
                raise UserError(_(
                    "You cannot archive this attribute as there are still products linked to it",
                ))
        return super().action_archive()

    def action_open_product_template_attribute_lines(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _("Products"),
            'res_model': 'product.template.attribute.line',
            'view_mode': 'list,form',
            'domain': [('attribute_id', '=', self.id), ('product_tmpl_id.active', '=', 'True')],
        }

    # === TOOLING === #

    def _without_no_variant_attributes(self):
        return self.filtered(lambda pa: pa.create_variant != 'no_variant')

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

from odoo import _, api, fields, models
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

    default_extra_price = fields.Float()
    is_custom = fields.Boolean(
        string="Free text",
        help="Allow customers to set their own value")
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
    active = fields.Boolean(default=True)

    is_used_on_products = fields.Boolean(
        string="Used on Products", compute='_compute_is_used_on_products')
    default_extra_price_changed = fields.Boolean(compute='_compute_default_extra_price_changed')

    # === COMPUTE METHODS === #

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

    @api.depends('pav_attribute_line_ids')
    def _compute_is_used_on_products(self):
        for pav in self:
            pav.is_used_on_products = bool(pav.pav_attribute_line_ids.filtered('product_tmpl_id.active'))

    @api.depends('default_extra_price')
    def _compute_default_extra_price_changed(self):
        for pav in self:
            pav.default_extra_price_changed = (
                pav.default_extra_price != pav._origin.default_extra_price
            )

    # === CRUD METHODS === #

    def write(self, vals):
        if 'attribute_id' in vals:
            for pav in self:
                if pav.attribute_id.id != vals['attribute_id'] and pav.is_used_on_products:
                    raise UserError(_(
                        "You cannot change the attribute of the value %(value)s because it is used"
                        " on the following products: %(products)s",
                        value=pav.display_name,
                        products=", ".join(pav.pav_attribute_line_ids.product_tmpl_id.mapped('display_name')),
                    ))

        invalidate = 'sequence' in vals and any(record.sequence != vals['sequence'] for record in self)
        res = super().write(vals)
        if invalidate:
            # prefetched o2m have to be resequenced
            # (eg. product.template.attribute.line: value_ids)
            self.env.flush_all()
            self.env.invalidate_all()
        return res

    def check_is_used_on_products(self):
        for pav in self.filtered('is_used_on_products'):
            return _(
                "You cannot delete the value %(value)s because it is used on the following"
                " products:\n%(products)s\n",
                value=pav.display_name,
                products=", ".join(pav.pav_attribute_line_ids.product_tmpl_id.mapped('display_name')),
            )
        return False

    @api.ondelete(at_uninstall=False)
    def _unlink_except_used_on_product(self):
        if is_used_on_products := self.check_is_used_on_products():
            raise UserError(is_used_on_products)

    def unlink(self):
        pavs_to_archive = self.env['product.attribute.value']
        for pav in self:
            linked_products = pav.env['product.template.attribute.value'].search(
                [('product_attribute_value_id', '=', pav.id)]
            ).with_context(active_test=False).ptav_product_variant_ids
            active_linked_products = linked_products.filtered('active')
            if not active_linked_products and linked_products:
                # If product attribute value found on non-active product variants
                # archive PAV instead of deleting
                pavs_to_archive |= pav
        if pavs_to_archive:
            pavs_to_archive.action_archive()
        return super(ProductAttributeValue, self - pavs_to_archive).unlink()

    def _without_no_variant_attributes(self):
        return self.filtered(lambda pav: pav.attribute_id.create_variant != 'no_variant')

    # === ACTION METHODS === #

    def action_add_to_products(self):
        return {
            'name': _("Add to all products"),
            'type': 'ir.actions.act_window',
            'res_model': 'update.product.attribute.value',
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'default_attribute_value_id': self.id,
                'default_mode': 'add',
                'dialog_size': 'medium',
            },
        }

    def action_update_prices(self):
        return {
            'name': _("Update product extra prices"),
            'type': 'ir.actions.act_window',
            'res_model': 'update.product.attribute.value',
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'default_attribute_value_id': self.id,
                'default_mode': 'update_extra_price',
                'dialog_size': 'medium',
            },
        }

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

    def _default_order_line_values(self, child_field=False):
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
        return ['|', ('company_id', '=', False), ('company_id', 'parent_of', self.company_id.id), ('type', '!=', 'combo')]

    def _get_product_catalog_record_lines(self, product_ids, child_field=False, **kwargs):
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
        return {product.id: {'productType': product.type} for product in products}

    def _get_product_catalog_order_line_info(self, product_ids, child_field=False, **kwargs):
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
        default_data = self._default_order_line_values(child_field)

        for product, record_lines in self._get_product_catalog_record_lines(product_ids, child_field=child_field, **kwargs).items():
            order_line_info[product.id] = {
               **record_lines._get_product_catalog_lines_data(parent_record=self, **kwargs),
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
    _inherit = ['mail.thread']
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
    parent_path = fields.Char(index=True)
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
        if self._has_cycle():
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

## File: models\product_combo.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class ProductCombo(models.Model):
    _name = 'product.combo'
    _description = "Product Combo"
    _order = 'sequence, id'

    name = fields.Char(string="Name", required=True)
    sequence = fields.Integer(default=10, copy=False)
    company_id = fields.Many2one(string="Company", comodel_name='res.company', index=True)
    combo_item_ids = fields.One2many(
        comodel_name='product.combo.item',
        inverse_name='combo_id',
        copy=True,
    )
    combo_item_count = fields.Integer(string="Product Count", compute='_compute_combo_item_count')
    currency_id = fields.Many2one(comodel_name='res.currency', compute='_compute_currency_id')
    base_price = fields.Float(
        string="Combo Price",
        help="The minimum price among the products in this combo. This value will be used to"
             " prorate the price of this combo with respect to the other combos in a combo product."
             " This heuristic ensures that whatever product the user chooses in a combo, it will"
             " always be the same price.",
        digits='Product Price',
        compute='_compute_base_price',
    )

    @api.depends('combo_item_ids')
    def _compute_combo_item_count(self):
        # Initialize combo_item_count to 0 as _read_group won't return any results for new combos.
        self.combo_item_count = 0
        # Optimization to count the number of combo items in each combo.
        for combo, item_count in self.env['product.combo.item']._read_group(
            domain=[('combo_id', 'in', self.ids)],
            groupby=['combo_id'],
            aggregates=['__count'],
        ):
            combo.combo_item_count = item_count

    @api.depends('company_id')
    def _compute_currency_id(self):
        main_company = self.env['res.company']._get_main_company()
        for combo in self:
            combo.currency_id = (
                combo.company_id.sudo().currency_id or main_company.currency_id
            )

    @api.depends('combo_item_ids')
    def _compute_base_price(self):
        for combo in self:
            combo.base_price = min(combo.combo_item_ids.mapped(
                lambda item: item.currency_id._convert(
                    from_amount=item.lst_price,
                    to_currency=combo.currency_id,
                    company=combo.company_id or self.env.company,
                    date=self.env.cr.now(),
                )
            )) if combo.combo_item_ids else 0

    @api.constrains('combo_item_ids')
    def _check_combo_item_ids_not_empty(self):
        if any(not combo.combo_item_ids for combo in self):
            raise ValidationError(_("A combo choice must contain at least 1 product."))

    @api.constrains('combo_item_ids')
    def _check_combo_item_ids_no_duplicates(self):
        for combo in self:
            if len(combo.combo_item_ids.mapped('product_id')) < len(combo.combo_item_ids):
                raise ValidationError(_("A combo choice can't contain duplicate products."))

    @api.constrains('company_id')
    def _check_company_id(self):
        templates = self.env['product.template'].sudo().search([('combo_ids', 'in', self.ids)])
        templates._check_company(fnames=['combo_ids'])

```

## File: models\product_combo_item.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class ProductComboItem(models.Model):
    _name = 'product.combo.item'
    _description = "Product Combo Item"
    _check_company_auto = True

    company_id = fields.Many2one(related='combo_id.company_id', precompute=True, store=True)
    combo_id = fields.Many2one(comodel_name='product.combo', ondelete='cascade', required=True)
    product_id = fields.Many2one(
        string="Product",
        comodel_name='product.product',
        ondelete='cascade',
        domain=[('type', '!=', 'combo')],
        required=True,
        check_company=True,
    )
    currency_id = fields.Many2one(comodel_name='res.currency', related='product_id.currency_id')
    lst_price = fields.Float(
        string="Original Price",
        digits='Product Price',
        related='product_id.lst_price',
    )
    extra_price = fields.Float(string="Extra Price", digits='Product Price', default=0.0)

    @api.constrains('product_id')
    def _check_product_id_no_combo(self):
        if any(combo_item.product_id.type == 'combo' for combo_item in self):
            raise ValidationError(_("A combo choice can't contain products of type \"combo\"."))

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
    _order = 'sequence, name'

    ir_attachment_id = fields.Many2one(
        'ir.attachment',
        string="Related attachment",
        required=True,
        ondelete='cascade')

    active = fields.Boolean(default=True)
    sequence = fields.Integer(default=10)

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

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        ir_default = default
        if ir_default:
            ir_fields = list(self.env['ir.attachment']._fields)
            ir_default = {field : default[field] for field in default if field in ir_fields}
        for document, vals in zip(self, vals_list):
            vals['ir_attachment_id'] = document.ir_attachment_id.with_context(
                no_document=True,
                disable_product_documents_creation=True,
            ).copy(ir_default).id
        return vals_list

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
        if self.env['product.product'].search_count(domain, limit=1):
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class Pricelist(models.Model):
    _name = "product.pricelist"
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = "Pricelist"
    _rec_names_search = ['name', 'currency_id']  # TODO check if should be removed
    _order = "sequence, id, name"

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
            pricelist_name = pricelist.name and pricelist.name or _('New')
            pricelist.display_name = f'{pricelist_name} ({pricelist.currency_id.name})'

    def write(self, values):
        res = super().write(values)

        # Make sure that there is no multi-company issue in the existing rules after the company
        # change.
        if 'company_id' in values and len(self) == 1:
            self.item_ids._check_company()

        return res

    def copy_data(self, default=None):
        default = dict(default or {})
        vals_list = super().copy_data(default=default)
        if 'name' not in default:
            for pricelist, vals in zip(self, vals_list):
                vals['name'] = _("%s (copy)", pricelist.name)
        return vals_list

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

        IrConfigParameter = self.env['ir.config_parameter'].sudo()
        Pricelist = self.env['product.pricelist']
        pl_domain = self._get_partner_pricelist_multi_search_domain_hook(company_id)

        # if no specific property, try to find a fitting pricelist
        result = {}
        remaining_partner_ids = []
        for partner in Partner.browse(partner_ids):
            if partner.specific_property_product_pricelist._get_partner_pricelist_multi_filter_hook():
                result[partner.id] = partner.specific_property_product_pricelist
            else:
                remaining_partner_ids.append(partner.id)

        if remaining_partner_ids:
            def convert_to_int(string_value):
                try:
                    return int(string_value)
                except (TypeError, ValueError, OverflowError):
                    return None
            # get fallback pricelist when no pricelist for a given country
            pl_fallback = (
                Pricelist.search(pl_domain + [('country_group_ids', '=', False)], limit=1) or
                # save data in ir.config_parameter instead of ir.default for
                # res.partner.property_product_pricelist
                # otherwise the data will become the default value while
                # creating without specifying the property_product_pricelist
                # however if the property_product_pricelist is not specified
                # the result of the previous line should have high priority
                # when computing
                Pricelist.browse(convert_to_int(IrConfigParameter.get_param(f'res.partner.property_product_pricelist_{company_id}'))) or
                Pricelist.browse(convert_to_int(IrConfigParameter.get_param('res.partner.property_product_pricelist'))) or
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
                'You cannot delete pricelist(s):\n(%(pricelists)s)\nThey are used within pricelist(s):\n%(other_pricelists)s',
                pricelists='\n'.join(linked_items.base_pricelist_id.mapped('display_name')),
                other_pricelists='\n'.join(linked_items.pricelist_id.mapped('display_name')),
            ))

    def action_open_pricelist_report(self):
        self.ensure_one()
        return {
            'name': _("Pricelist Report Preview"),
            'type': 'ir.actions.client',
            'tag': 'generate_pricelist_report',
        }

```

## File: models\product_pricelist_item.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import SUPERUSER_ID, _, api, fields, models, tools
from odoo.exceptions import ValidationError
from odoo.tools import format_amount, format_datetime, formatLang


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

    display_applied_on = fields.Selection(
        selection=[
            ('1_product', "Product"),
            ('2_product_category', "Category"),
        ],
        default='1_product',
        required=True,
        help="Pricelist Item applicable on selected option")

    categ_id = fields.Many2one(
        comodel_name='product.category',
        string="Category",
        ondelete='cascade',
        help="Specify a product category if this rule only applies to products belonging to this category or its children categories. Keep empty otherwise.")
    product_tmpl_id = fields.Many2one(
        comodel_name='product.template',
        string="Product",
        ondelete='cascade', check_company=True,
        help="Specify a template if this rule only applies to one product template. Keep empty otherwise.")
    product_id = fields.Many2one(
        comodel_name='product.product',
        string="Variant",
        ondelete='cascade', check_company=True,
        domain="[('product_tmpl_id', '=', product_tmpl_id)]",
        help="Specify a product if this rule only applies to one product. Keep empty otherwise.")
    product_uom = fields.Char(related='product_tmpl_id.uom_name')
    product_variant_count = fields.Integer(related='product_tmpl_id.product_variant_count')

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
            ('percentage', "Discount"),
            ('formula', "Formula"),
            ('fixed', "Fixed Price"),
        ],
        help="Use the discount rules and activate the discount settings"
             " in order to show discount to customer.",
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
             "To have prices that end in 9.99, round off to 10.00 and set an extra at -0.01")
    price_surcharge = fields.Float(
        string="Extra Fee",
        digits='Product Price',
        help="Specify the fixed amount to add or subtract (if negative) to the amount calculated with the discount.")

    price_markup = fields.Float(
        string="Markup",
        digits=(16, 2),
        compute='_compute_price_markup',
        inverse='_inverse_price_markup',
        store=True,
        help="You can apply a mark-up on the cost")

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
        compute='_compute_name',
        help="Explicit rule name for this pricelist line.")
    price = fields.Char(
        string="Price",
        compute='_compute_price_label',
        help="Explicit rule name for this pricelist line.")
    rule_tip = fields.Char(compute='_compute_rule_tip')

    #=== COMPUTE METHODS ===#

    @api.depends('applied_on', 'categ_id', 'product_tmpl_id', 'product_id')
    def _compute_name(self):
        for item in self:
            if item.categ_id and item.applied_on == '2_product_category':
                item.name = _("Category: %s", item.categ_id.display_name)
            elif item.product_tmpl_id and item.applied_on == '1_product':
                item.name = item.product_tmpl_id.display_name
            elif item.product_id and item.applied_on == '0_product_variant':
                item.name = _("Variant: %s", item.product_id.display_name)
            elif item.display_applied_on == '2_product_category':
                item.name = _("All Categories")
            else:
                item.name = _("All Products")

    @api.depends(
        'compute_price', 'fixed_price', 'pricelist_id', 'percent_price', 'price_discount',
        'price_markup', 'price_surcharge', 'base', 'base_pricelist_id',
    )
    def _compute_price_label(self):
        for item in self:
            if item.compute_price == 'fixed':
                item.price = formatLang(
                    item.env, item.fixed_price, dp="Product Price", currency_obj=item.currency_id)
            elif item.compute_price == 'percentage':
                percentage = self._get_integer(item.percent_price)
                if item.base_pricelist_id:
                    item.price = _(
                        "%(percentage)s %% discount on %(pricelist)s",
                        percentage=percentage,
                        pricelist=item.base_pricelist_id.display_name
                    )
                else:
                    item.price = _(
                        "%(percentage)s %% discount on sales price",
                        percentage=percentage
                    )
            else:
                base_str = ""
                if item.base == 'pricelist' and item.base_pricelist_id:
                    base_str = item.base_pricelist_id.display_name
                elif item.base == 'standard_price':
                    base_str = _("product cost")
                else:
                    base_str = _("sales price")

                extra_fee_str = ""
                if item.price_surcharge > 0:
                    extra_fee_str = _(
                        "+ %(amount)s extra fee",
                        amount=format_amount(
                            item.env,
                            abs(item.price_surcharge),
                            currency=item.currency_id,
                        ),
                    )
                elif item.price_surcharge < 0:
                    extra_fee_str = _(
                        "- %(amount)s rebate",
                        amount=format_amount(
                            item.env,
                            abs(item.price_surcharge),
                            currency=item.currency_id,
                        ),
                    )
                discount_type, percentage = self._get_displayed_discount(item)
                item.price = _("%(percentage)s %% %(discount_type)s on %(base)s %(extra)s",
                    percentage=percentage,
                    discount_type=discount_type,
                    base=base_str,
                    extra=extra_fee_str,
                )

    @api.depends('price_discount')
    def _compute_price_markup(self):
        for item in self:
            item.price_markup = -item.price_discount

    def _inverse_price_markup(self):
        for item in self:
            item.price_discount = -item.price_markup

    @api.depends_context('lang')
    @api.depends(
        'base', 'compute_price', 'price_discount', 'price_markup', 'price_round', 'price_surcharge',
    )
    def _compute_rule_tip(self):
        base_selection_vals = {elem[0]: elem[1] for elem in self._fields['base']._description_selection(self.env)}
        self.rule_tip = False
        for item in self:
            if item.compute_price != 'formula':
                continue
            base_amount = 100
            discount = item.price_discount if item.base != 'standard_price' else -item.price_markup
            discount_factor = (100 - discount) / 100
            discounted_price = base_amount * discount_factor
            if item.price_round:
                discounted_price = tools.float_round(discounted_price, precision_rounding=item.price_round)
            surcharge = tools.format_amount(item.env, item.price_surcharge, item.currency_id)
            discount_type, discount = self._get_displayed_discount(item)

            item.rule_tip = _(
                "%(base)s with a %(discount)s %% %(discount_type)s and %(surcharge)s extra fee\n"
                "Example: %(amount)s * %(discount_charge)s + %(price_surcharge)s → %(total_amount)s",
                base=base_selection_vals[item.base],
                discount=discount,
                discount_type=discount_type,
                surcharge=surcharge,
                amount=tools.format_amount(item.env, 100, item.currency_id),
                discount_charge=discount_factor,
                price_surcharge=surcharge,
                total_amount=tools.format_amount(
                    item.env, discounted_price + item.price_surcharge, item.currency_id),
            )

    def _get_integer(self, percentage):
        return int(percentage) if percentage.is_integer() else percentage

    def _get_displayed_discount(self, item):
        if item.base == 'standard_price':
            return _("markup"), self._get_integer(item.price_markup)
        return _("discount"), self._get_integer(item.price_discount)

    #=== CONSTRAINT METHODS ===#

    @api.constrains('base_pricelist_id', 'pricelist_id', 'base')
    def _check_pricelist_recursion(self):
        if any(item.base == 'pricelist' and item.pricelist_id and item.pricelist_id == item.base_pricelist_id for item in self):
            raise ValidationError(_('You cannot assign the Main Pricelist as Other Pricelist in PriceList Item'))

    @api.constrains('date_start', 'date_end')
    def _check_date_range(self):
        for item in self:
            if item.date_start and item.date_end and item.date_start >= item.date_end:
                raise ValidationError(_(
                    '%(item_name)s: end date (%(end_date)s) should be after start date (%(start_date)s)',
                    item_name=item.display_name,
                    end_date=format_datetime(self.env, item.date_end),
                    start_date=format_datetime(self.env, item.date_start),
                ))
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

    @api.onchange('base')
    def _onchange_base(self):
        for item in self:
            item.update({
                'price_discount': 0.0,
                'price_markup': 0.0,
            })

    @api.onchange('base_pricelist_id')
    def _onchange_base_pricelist_id(self):
        for item in self:
            if item.compute_price == 'percentage':
                item.base = bool(item.base_pricelist_id) and 'pricelist' or 'list_price'

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
                'price_markup': 0.0,
                'price_round': 0.0,
                'price_min_margin': 0.0,
                'price_max_margin': 0.0,
            })

    @api.onchange('display_applied_on')
    def _onchange_display_applied_on(self):
        for item in self:
            if not (item.product_tmpl_id or item.categ_id):
                item.update(dict(
                    applied_on='3_global',
                ))
            elif item.display_applied_on == '1_product':
                item.update(dict(
                    applied_on='1_product',
                    categ_id=None,
                ))
            elif item.display_applied_on == '2_product_category':
                item.update(dict(
                    product_id=None,
                    product_tmpl_id=None,
                    applied_on='2_product_category',
                    product_uom=None,
                ))

    @api.onchange('price_markup')
    def _onchange_price_markup(self):
        pass  # TODO: remove in master

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
        if not self.env.context.get('default_applied_on', False):
            # If we aren't coming from a specific product template/variant.
            variants_rules = self.filtered('product_id')
            template_rules = (self-variants_rules).filtered('product_tmpl_id')
            category_rules = self.filtered(lambda cat: cat.categ_id and cat.categ_id.name != 'All')
            variants_rules.update({'applied_on': '0_product_variant'})
            template_rules.update({'applied_on': '1_product'})
            category_rules.update({'applied_on': '2_product_category'})
            global_rules = self - variants_rules - template_rules - category_rules
            global_rules.update({'applied_on': '3_global'})

    @api.onchange('price_round')
    def _onchange_price_round(self):
        if any(item.price_round and item.price_round < 0.0 for item in self):
            raise ValidationError(_("The rounding method must be strictly positive."))

    @api.onchange('date_start', 'date_end')
    def _onchange_validity_period(self):
        self._check_date_range()

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('product_id') and not values.get('product_tmpl_id'):
                # Deduce product template from product variant if not specified.
                # Ensures that the pricelist rule is properly configured and displayed in the UX
                # even in case of partial/incomplete data (mostly for imports).
                values['product_tmpl_id'] = self.env['product.product'].browse(
                    values.get('product_id')
                ).product_tmpl_id.id

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
            discount = self.price_discount if self.base != 'standard_price' else -self.price_markup
            price = base_price - (base_price * (discount / 100))
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
        """Compute the base price of the lowest pricelist rule,
        discount is shown by default if computation method is a percentage rule.

        :param product: recordset of product (product.product/product.template)
        :param float qty: quantity of products requested (in given uom)
        :param uom: unit of measure (uom.uom record)
        :param datetime date: date to use for price computation and currency conversions
        :param currency: currency in which the returned price must be expressed

        :returns: base price, expressed in provided pricelist currency
        :rtype: float
        """
        pricelist_item = self
        # Find the lowest pricelist rule whose pricelist is configured to show the discount to the
        # customer.
        while pricelist_item.base == 'pricelist':
            rule_id = pricelist_item.base_pricelist_id._get_product_rule(*args, **kwargs)
            rule_pricelist_item = self.env['product.pricelist.item'].browse(rule_id)
            if rule_pricelist_item and rule_pricelist_item.compute_price == 'percentage':
                pricelist_item = rule_pricelist_item
            else:
                break

        return pricelist_item._compute_base_price(*args, **kwargs)

    @api.model
    def _is_discount_feature_enabled(self):
        superuser = self.env['res.users'].browse(SUPERUSER_ID)
        return superuser.has_group('sale.group_discount_per_so_line')

    def _show_discount(self):
        if not self:
            return False

        self.ensure_one()
        return self._is_discount_feature_enabled() and self.compute_price == 'percentage'

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from operator import itemgetter

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError
from odoo.osv import expression
from odoo.tools import float_compare, format_list, groupby
from odoo.tools.image import is_image_size_above
from odoo.tools.misc import unique


class ProductProduct(models.Model):
    _name = "product.product"
    _description = "Product Variant"
    _inherits = {'product.template': 'product_tmpl_id'}
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'is_favorite desc, default_code, name, id'
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
        string="Variant Tags",
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
            record.can_image_variant_1024_be_zoomed = record.image_variant_1920 and is_image_size_above(record.image_variant_1920, record.image_variant_1024)

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
                "- Barcode \"%(barcode)s\" already assigned to product(s): %(product_list)s",
                barcode=record['barcode'], product_list=format_list(self.env, [p.display_name for p in self.search([('id', 'in', record['id'])])]),
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
        if self.env['product.packaging'].sudo().search_count(packaging_domain, limit=1):
            raise ValidationError(_("A packaging already uses the barcode"))

    @api.constrains('barcode')
    def _check_barcode_uniqueness(self):
        """ With GS1 nomenclature, products and packagings use the same pattern. Therefore, we need
        to ensure the uniqueness between products' barcodes and packagings' ones"""
        # Barcodes should only be unique within a company
        for company_id, barcodes_within_company in self._get_barcodes_by_company():
            self._check_duplicated_product_barcodes(barcodes_within_company, company_id)
            self._check_duplicated_packaging_barcodes(barcodes_within_company, company_id)

    @api.constrains('company_id')
    def _check_company_id(self):
        combo_items = self.env['product.combo.item'].sudo().search([('product_id', 'in', self.ids)])
        combo_items._check_company(fnames=['product_id'])

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
                ('compute_price', '=', 'fixed'),
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
            product.all_product_tag_ids = (
                product.product_tag_ids | product.additional_product_tag_ids
            ).sorted('sequence')

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

        if self.env['product.product'].search_count(domain, limit=1):
            return {'warning': {
                'title': _("Note:"),
                'message': _("The Reference '%s' already exists.", self.default_code),
            }}

    @api.model_create_multi
    def create(self, vals_list):
        products = super(ProductProduct, self.with_context(create_product_product=False)).create(vals_list)
        # `_get_variant_id_for_combination` depends on existing variants
        self.env.registry.clear_cache()
        return products

    def write(self, values):
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
            self.check_access('unlink')
            self.check_access('write')
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

        # Use tmp recordset in case we copy several variants from the same template
        templates = [product.product_tmpl_id for product in self]
        templates_to_copy = self.env['product.template'].concat(*templates)
        new_templates = templates_to_copy.copy(default=default)
        new_products = self.env['product.product']
        for new_template in new_templates:
            new_products += new_template.product_variant_id or new_template._create_first_product_variant()
        return new_products

    @api.model
    def _search(self, domain, offset=0, limit=None, order=None):
        # TDE FIXME: strange
        if self._context.get('search_default_categ_id'):
            domain = domain.copy()
            domain.append((('categ_id', 'child_of', self._context['search_default_categ_id'])))
        return super()._search(domain, offset, limit, order)

    @api.depends('name', 'default_code', 'product_tmpl_id')
    @api.depends_context('display_default_code', 'seller_id', 'company_id', 'partner_id')
    def _compute_display_name(self):

        def get_display_name(name, code):
            if self._context.get('display_default_code', True) and code:
                return f'[{code}] {name}'
            return name

        partner_id = self._context.get('partner_id')
        if partner_id:
            partner_ids = [partner_id, self.env['res.partner'].browse(partner_id).commercial_partner_id.id]
        else:
            partner_ids = []
        company_id = self.env.context.get('company_id')

        # all user don't have access to seller and partner
        # check access and use superuser
        self.check_access("read")

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
    def _search_display_name(self, operator, value):
        is_positive = operator not in expression.NEGATIVE_TERM_OPERATORS
        combine = expression.OR if is_positive else expression.AND
        domains = [
            [('name', operator, value)],
            [('default_code', operator, value)],
        ]
        if operator in ('=', 'in') or (operator.endswith('like') and is_positive):
            barcode_values = [value] if operator != 'in' else value
            domains.append([('barcode', 'in', barcode_values)])
        if operator == '=' and isinstance(value, str) and (m := re.search(r'(\[(.*?)\])', value)):
            domains.append([('default_code', '=', m.group(2))])
        if partner_id := self.env.context.get('partner_id'):
            supplier_domain = [
                ('partner_id', '=', partner_id),
                '|',
                ('product_code', operator, value),
                ('product_name', operator, value),
            ]
            domains.append([('product_tmpl_id.seller_ids', 'any', supplier_domain)])
        return combine(domains)

    @api.model
    def name_search(self, name='', args=None, operator='ilike', limit=100):
        if not name:
            return super().name_search(name, args, operator, limit)
        # search progressively by the most specific attributes
        positive_operators = ['=', 'ilike', '=ilike', 'like', '=like']
        is_positive = operator not in expression.NEGATIVE_TERM_OPERATORS
        products = self.browse()
        domain = args or []
        if operator in positive_operators:
            products = self.search_fetch(expression.AND([domain, [('default_code', '=', name)]]), ['display_name'], limit=limit) \
                or self.search_fetch(expression.AND([domain, [('barcode', '=', name)]]), ['display_name'], limit=limit)
        if not products:
            if is_positive:
                # Do not merge the 2 next lines into one single search, SQL search performance would be abysmal
                # on a database with thousands of matching products, due to the huge merge+unique needed for the
                # OR operator (and given the fact that the 'name' lookup results come from the ir.translation table
                # Performing a quick memory merge of ids in Python will give much better performance
                products = self.search_fetch(expression.AND([domain, [('default_code', operator, name)]]), ['display_name'], limit=limit)
                limit_rest = limit and limit - len(products)
                if limit_rest is None or limit_rest > 0:
                    products |= self.search_fetch(expression.AND([domain, [('id', 'not in', products.ids)], [('name', operator, name)]]), ['display_name'], limit=limit_rest)
            else:
                domain_neg = [
                    ('name', operator, name),
                    '|', ('default_code', operator, name), ('default_code', '=', False),
                ]
                products = self.search_fetch(expression.AND([domain, domain_neg]), ['display_name'], limit=limit)
        if not products and operator in positive_operators and (m := re.search(r'(\[(.*?)\])', name)):
            match_domain = [('default_code', '=', m.group(2))]
            products = self.search_fetch(expression.AND([domain, match_domain]), ['display_name'], limit=limit)
        if not products and (partner_id := self.env.context.get('partner_id')):
            # still no results, partner in context: search on supplier info as last hope to find something
            supplier_domain = [
                ('partner_id', '=', partner_id),
                '|',
                ('product_code', operator, name),
                ('product_name', operator, name),
            ]
            match_domain = [('product_tmpl_id.seller_ids', 'any', supplier_domain)]
            products = self.search_fetch(expression.AND([domain, match_domain]), ['display_name'], limit=limit)
        return [(product.id, product.display_name) for product in products.sudo()]

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
            '&', ('product_id', '=', self.id), ('applied_on', '=', '0_product_variant'),
            ('compute_price', '=', 'fixed'),
        ]
        return {
            'name': _('Price Rules'),
            'view_mode': 'list,form',
            'views': [(self.env.ref('product.product_pricelist_item_tree_view_from_product').id, 'list')],
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
        if not date:
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

        no_variant_attributes_price_extra = self._get_no_variant_attributes_price_extra(combination)

        if no_variant_attributes_price_extra:
            res['no_variant_attributes_price_extra'] = no_variant_attributes_price_extra

        return res

    def _get_no_variant_attributes_price_extra(self, combination):
        # It is possible that a no_variant attribute is still in a variant if
        # the type of the attribute has been changed after creation.
        return sum(
            ptav.price_extra for ptav in combination.filtered(
                lambda ptav:
                    ptav.price_extra
                    and ptav.product_tmpl_id == self.product_tmpl_id
                    and ptav not in self.product_template_attribute_value_ids
            )
        )

    def _get_attributes_extra_price(self):
        self.ensure_one()

        return self.price_extra + self.env.context.get('no_variant_attributes_price_extra', 0)

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
        # FIXME VFE this won't consider ptavs extra prices, since we rely on the template price
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

from odoo import api, fields, models
from odoo.osv import expression

class ProductTag(models.Model):
    _name = 'product.tag'
    _description = 'Product Tag'
    _order = 'sequence, id'

    def _get_default_template_id(self):
        return self.env['product.template'].browse(self.env.context.get('product_template_id'))

    def _get_default_variant_id(self):
        return self.env['product.product'].browse(self.env.context.get('product_variant_id'))

    name = fields.Char(string="Name", required=True, translate=True)
    sequence = fields.Integer(default=10)
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

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", tag.name)) for tag, vals in zip(self, vals_list)]

    def _search_product_ids(self, operator, operand):
        if operator in expression.NEGATIVE_TERM_OPERATORS:
            return [('product_template_ids.product_variant_ids', operator, operand), ('product_product_ids', operator, operand)]
        return ['|', ('product_template_ids.product_variant_ids', operator, operand), ('product_product_ids', operator, operand)]

```

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import itertools
import logging

from collections import defaultdict

from odoo import _, api, fields, models, tools
from odoo.exceptions import UserError, ValidationError
from odoo.osv import expression
from odoo.tools.image import is_image_size_above

_logger = logging.getLogger(__name__)
PRICE_CONTEXT_KEYS = ['pricelist', 'quantity', 'uom', 'date']


class ProductTemplate(models.Model):
    _name = "product.template"
    _inherit = ['mail.thread', 'mail.activity.mixin', 'image.mixin']
    _description = "Product"
    _order = "is_favorite desc, name"
    _check_company_auto = True
    _check_company_domain = models.check_company_domain_parent_of

    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        if 'uom_id' in fields_list and not res.get('uom_id') or self.env.context.get('default_uom_id') is False:
            res['uom_id'] = self._get_default_uom_id().id
        return res

    @tools.ormcache()
    def _get_default_category_id(self):
        # Deletion forbidden (at least through unlink)
        return self.env.ref('product.product_category_all')

    @tools.ormcache()
    def _get_default_uom_id(self):
        # Deletion forbidden (at least through unlink)
        return self.env.ref('uom.product_uom_unit')

    def _read_group_categ_id(self, categories, domain):
        category_ids = self.env.context.get('default_categ_id')
        if not category_ids and self.env.context.get('group_expand'):
            category_ids = categories.sudo()._search([], order=categories._order)
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
    type = fields.Selection(
        string="Product Type",
        help="Goods are tangible materials and merchandise you provide.\n"
             "A service is a non-material product you provide.",
        selection=[
            ('consu', "Goods"),
            ('service', "Service"),
            ('combo', "Combo"),
        ],
        required=True,
        default='consu',
    )
    combo_ids = fields.Many2many(
        string="Combo Choices", comodel_name='product.combo', check_company=True
    )
    service_tracking = fields.Selection(selection=[
            ('no', 'Nothing'),
        ],
        string="Create on Order",
        default="no",
        compute="_compute_service_tracking",
        required=True,
        store=True,
        readonly=False,
    )
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
        tracking=True,
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

    sale_ok = fields.Boolean('Sales', default=True)
    purchase_ok = fields.Boolean('Purchase', default=True, compute='_compute_purchase_ok', store=True, readonly=False)
    uom_id = fields.Many2one(
        'uom.uom', 'Unit of Measure',
        default=_get_default_uom_id, required=True,
        help="Default unit of measure used for all stock operations.")
    uom_name = fields.Char(string='Unit of Measure Name', related='uom_id.name', readonly=True)
    uom_category_id = fields.Many2one('uom.category', string='UoM Category', related="uom_id.category_id")
    uom_po_id = fields.Many2one(
        'uom.uom', 'Purchase Unit',
        compute='_compute_uom_po_id', required=True, readonly=False, store=True, precompute=True,
        domain="[('category_id', '=', uom_category_id)]",
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

    is_favorite = fields.Boolean(string="Favorite")

    product_tag_ids = fields.Many2many(
        string="Tags", comodel_name='product.tag', relation='product_tag_product_template_rel'
    )
    # Properties
    product_properties = fields.Properties('Properties', definition='categ_id.product_properties_definition', copy=True)

    @api.depends('type')
    def _compute_service_tracking(self):
        self.filtered(lambda product: product.type != 'service').service_tracking = 'no'

    def _compute_purchase_ok(self):
        pass

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
                ('compute_price', '=', 'fixed'),
            ])

    def _compute_product_document_count(self):
        for template in self:
            template.product_document_count = template.env['product.document'].search_count(
                template._get_product_document_domain()
            )

    @api.depends('image_1920', 'image_1024')
    def _compute_can_image_1024_be_zoomed(self):
        for template in self.with_context(bin_size=False):
            template.can_image_1024_be_zoomed = template.image_1920 and is_image_size_above(template.image_1920, template.image_1024)

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
                    ptal._is_configurable()
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

        if self.env['product.template'].search_count(domain, limit=1):
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
        self.product_tooltip = False
        for template in self:
            template.product_tooltip = template._prepare_tooltip()

    def _prepare_tooltip(self):
        self.ensure_one()
        tooltip = ""
        if self.type == 'combo':
            tooltip = _(
                "Combos allow to choose one product amongst a selection of choices per category."
            )
        return tooltip

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
        if self.type == 'combo':
            if self.attribute_line_ids:
                raise UserError(_("Combo products can't have attributes."))
            combo_items = self.env['product.combo.item'].sudo().search([
                ('product_id', 'in', self.product_variant_ids.ids)
            ])
            if combo_items:
                raise UserError(_(
                    "This product is part of a combo, so its type can't be changed to \"combo\"."
                ))
            self.purchase_ok = False
        return {}

    @api.constrains('type', 'combo_ids')
    def _check_combo_ids_not_empty(self):
        for template in self:
            if template.type == 'combo' and not template.combo_ids:
                raise ValidationError(_("A combo product must contain at least 1 combo choice."))

    @api.constrains('type', 'combo_ids', 'sale_ok')
    def _check_sale_combo_ids(self):
        for template in self:
            if (
                template.type == 'combo'
                and template.sale_ok
                and any(
                    not product.sale_ok for product in template.combo_ids.combo_item_ids.product_id
                )
            ):
                raise ValidationError(
                    _("A sellable combo product can only contain sellable products.")
                )

    def _get_related_fields_variant_template(self):
        """ Return a list of fields present on template and variants models and that are related"""
        return ['barcode', 'default_code', 'standard_price', 'volume', 'weight', 'packaging_ids', 'product_properties']

    @api.model_create_multi
    def create(self, vals_list):
        ''' Store the initial standard price in order to be able to retrieve the cost of a product template for a given date'''
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
        for product_template in self:
            if "type" in vals and vals.get("type") != "combo":
                product_template.combo_ids = False
        return res

    def copy_data(self, default=None):
        default = dict(default or {})
        vals_list = super().copy_data(default=default)
        if 'name' not in default:
            for template, vals in zip(self, vals_list):
                vals['name'] = _("%s (copy)", template.name)
        return vals_list

    def copy(self, default=None):
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
    def _search_display_name(self, operator, value):
        domain = super()._search_display_name(operator, value)
        if self.env.context.get('search_product_product', bool(value)):
            combine = expression.OR if operator not in expression.NEGATIVE_TERM_OPERATORS else expression.AND
            domain = combine([domain, [('product_variant_ids', operator, value)]])
        return domain

    @api.model
    def name_search(self, name='', args=None, operator='ilike', limit=100):
        # Only use the product.product heuristics if there is a search term and the domain
        # does not specify a match on `product.template` IDs.
        self_obj = self
        if 'search_product_product' not in self.env.context and any(term[0] == 'id' for term in (args or [])):
            self_obj = self_obj.with_context(search_product_product=False)
        return super(ProductTemplate, self_obj).name_search(name, args, operator, limit)

    #=== ACTION METHODS ===#

    def action_open_label_layout(self):
        action = self.env['ir.actions.act_window']._for_xml_id('product.action_open_label_layout')
        action['context'] = {'default_product_tmpl_ids': self.ids}
        return action

    def open_pricelist_rules(self):
        self.ensure_one()
        domain = ['|',
            ('product_tmpl_id', '=', self.id),
            ('product_id', 'in', self.product_variant_ids.ids),
            ('compute_price', '=', 'fixed'),
        ]
        return {
            'name': _('Price Rules'),
            'view_mode': 'list,form',
            'views': [(self.env.ref('product.product_pricelist_item_tree_view_from_product').id, 'list')],
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
            'view_mode': 'kanban,list,form',
            'context': {
                'default_res_model': self._name,
                'default_res_id': self.id,
                'default_company_id': self.company_id.id,
            },
            'domain': self._get_product_document_domain(),
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
                    <a class="oe_link" href="https://www.odoo.com/documentation/18.0/_downloads/c2c6ce32294dfddffcfefcf2775f7a09/pdfquotebuilderexamples.zip">
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
        for variant in variants_to_unlink:
            combo_items_to_unlink = self.env['product.combo.item'].search([
                ('product_id', '=', variant.id)
            ])
            # Unlink all combo items which reference unlinked variants.
            combo_items_to_unlink.unlink()

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

    def _get_product_document_domain(self):
        self.ensure_one()
        return expression.OR([
            expression.AND([[('res_model', '=', 'product.template')], [('res_id', '=', self.id)]]),
            expression.AND([
                [('res_model', '=', 'product.product')],
                [('res_id', 'in', self.product_variant_ids.ids)],
            ])
        ])

    ###################
    # DEMO DATA SETUP #
    ###################

    @api.model
    def _demo_configure_variants(self):
        acoustic_bloc_screens = self.env.ref(
            'product.product_template_acoustic_bloc_screens', raise_if_not_found=False
        )
        if acoustic_bloc_screens:
            acoustic_bloc_screens.product_variant_ids[0].default_code = 'FURN_6666'
            acoustic_bloc_screens.product_variant_ids[1].default_code = 'FURN_6667'
            self.env['ir.model.data']._update_xmlids([{
                'xml_id': 'product.product_product_25',
                'record': acoustic_bloc_screens.product_variant_ids[1],
                'noupdate': True,
            }])

    def _get_list_price(self, price):
        """ Get the product sales price from a public price based on taxes defined on the product.
        To be overridden in accounting module."""
        self.ensure_one()
        return price

    @api.model
    def _service_tracking_blacklist(self):
        """ Service tracking field is used to distinguish some specific categories of products.
        Those products shouldn't be displayed or used in unrelated applications.
        This method returns a domain targeting all those specific products (events, courses, ...).
        """
        return []

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

from odoo import _, api, fields, models, tools
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
        if self.attribute_id.create_variant == 'no_variant':
            self.value_ids = self.env['product.attribute.value'].search([
                ('attribute_id', '=', self.attribute_id.id),
            ])
        else:
            self.value_ids = self.value_ids.filtered(
                lambda pav: pav.attribute_id == self.attribute_id
            )

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

    def _is_configurable(self):
        self.ensure_one()
        return (
            len(self.value_ids) >= 2
            or self.attribute_id.display_type == 'multi'
            or self.value_ids.is_custom
        )

    def action_open_attribute_values(self):
        return {
            'type': 'ir.actions.act_window',
            'name': _("Product Variant Values"),
            'res_model': 'product.template.attribute.value',
            'view_mode': 'list,form',
            'domain': [('id', 'in', self.product_template_value_ids.ids)],
            'views': [
                (self.env.ref('product.product_template_attribute_value_view_tree').id, 'list'),
                (self.env.ref('product.product_template_attribute_value_view_form').id, 'form'),
            ],
            'context': {
                'search_default_active': 1,
                'product_invisible': True,
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
        string="Extra Price",
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

        if self.env.user.has_group('product.group_product_pricelist'):
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
            'name': _("Default"),
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

        enabled_pricelists = self.env.user.has_group('product.group_product_pricelist')
        res = super(
            ResCompany, self.with_context(disable_company_pricelist_creation=True)
        ).write(vals)
        if not enabled_pricelists and self.env.user.has_group('product.group_product_pricelist'):
            self.browse()._activate_or_create_pricelists()

        return res

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_uom = fields.Boolean("Units of Measure", implied_group='uom.group_uom')
    group_product_variant = fields.Boolean("Variants", implied_group='product.group_product_variant')
    module_loyalty = fields.Boolean("Promotions, Coupons, Gift Card & Loyalty Program")
    group_stock_packaging = fields.Boolean('Product Packagings',
        implied_group='product.group_stock_packaging')
    group_product_pricelist = fields.Boolean("Pricelists",
        implied_group='product.group_product_pricelist')
    product_weight_in_lbs = fields.Selection([
        ('0', 'Kilograms'),
        ('1', 'Pounds'),
    ], 'Weight unit of measure', config_parameter='product.weight_in_lbs', default='0')
    product_volume_volume_in_cubic_feet = fields.Selection([
        ('0', 'Cubic Meters'),
        ('1', 'Cubic Feet'),
    ], 'Volume unit of measure', config_parameter='product.volume_in_cubic_feet', default='0')

    @api.onchange('group_product_pricelist')
    def _onchange_group_sale_pricelist(self):
        if not self.group_product_pricelist:
            active_pricelist = self.env['product.pricelist'].sudo().search_count(
                [('active', '=', True)], limit=1
            )
            if active_pricelist:
                return {
                    'warning': {
                    'message': _("You are deactivating the pricelist feature. "
                                 "Every active pricelist will be archived.")
                }}

    def set_values(self):
        had_group_pl = self.default_get(['group_product_pricelist'])['group_product_pricelist']
        super().set_values()

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
        if not self.env.user.has_group('product.group_product_pricelist'):
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

    # when the specific_property_product_pricelist is not defined
    # the fallback value may be computed with 2 ir.config_parameter
    # in self.env['product.pricelist']._get_partner_pricelist_multi
    # 1. res.partner.property_product_pricelist_{company_id}  # fallback for current company
    # 2. res.partner.property_product_pricelist               # fallback for all companies
    property_product_pricelist = fields.Many2one(
        comodel_name='product.pricelist',
        string="Pricelist",
        compute='_compute_product_pricelist',
        inverse="_inverse_product_pricelist",
        company_dependent=False,  # behave like company dependent field but is not company_dependent
        domain=lambda self: [('company_id', 'in', (self.env.company.id, False))],
        help="This pricelist will be used, instead of the default one, for sales to the current partner")

    # the specific pricelist to compute property_product_pricelist
    # this company dependent field shouldn't have any fallback in ir.default
    specific_property_product_pricelist = fields.Many2one(
        comodel_name='product.pricelist',
        company_dependent=True,
    )

    @api.depends('country_id', 'specific_property_product_pricelist')
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
            actual = partner.specific_property_product_pricelist
            # update at each change country, and so erase old pricelist
            if partner.property_product_pricelist or (actual and default_for_country and default_for_country.id != actual.id):
                partner.specific_property_product_pricelist = False if partner.property_product_pricelist.id == default_for_country.id else partner.property_product_pricelist.id

    def _commercial_fields(self):
        return super()._commercial_fields() + ['property_product_pricelist']

    def _company_dependent_commercial_fields(self):
        return [
            *super()._company_dependent_commercial_fields(),
            'specific_property_product_pricelist'
        ]

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
                    " (%(digits)s digits).\nThis may cause inconsistencies in computations.\n"
                    "Please set a precision between %(min_precision)s and 1.",
                    digits=precision, min_precision=1.0 / 10.0**precision),
            }}

```

## File: models\__init__.py

```python
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
from . import product_combo
from . import product_combo_item
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

        products = ProductClass.browse(active_ids) if active_ids else []
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
            'docs': pricelist,
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
            <div class="row my-3">
                <div class="col-12">
                    <h2 t-if="is_html_type">
                        Pricelist <span t-if="display_pricelist_title">:</span>
                        <a t-if="display_pricelist_title" href="#" class="o_action"
                            data-model="product.pricelist" t-att-data-res-id="pricelist.id">
                            <span t-field="pricelist.display_name">Gold Member Pricelist</span>
                        </a>
                    </h2>
                    <h2 t-else="">
                        Pricelist <span t-if="display_pricelist_title">:</span>
                        <span t-if="display_pricelist_title" t-field="pricelist.display_name">Gold Member Pricelist</span>
                    </h2>
                </div>
            </div>
            <div class="oe_structure"></div>
            <div class="row">
                <div class="col-12">
                    <table class="table table-sm">
                        <thead>
                            <th class="text-end px-1" colspan="100%">Quantities (Price)</th>
                            <tr>
                                <th class="px-1">Products</th>
                                <th class="px-1" groups="uom.group_uom">UOM</th>
                                <t t-foreach="quantities" t-as="qty">
                                    <th class="text-end px-1"><span t-out="qty">10 Units</span></th>
                                </t>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="products" t-as="product">
                                <tr>
                                    <td t-att-class="is_product_tmpl and 'fw-bold px-1' or None">
                                        <a t-if="is_html_type" href="#" class="o_action" t-att-data-model="is_product_tmpl and 'product.template' or 'product.product'" t-att-data-res-id="product['id']">
                                            <span t-out="product['name']">Virtual Interior Design</span>
                                        </a>
                                        <span t-else="" t-out="product['name']">Acme Widget</span>
                                    </td>
                                    <td groups="uom.group_uom" class="px-1" t-out="product['uom']"/>
                                    <t t-foreach="quantities" t-as="qty">
                                        <td class="text-end px-1">
                                            <span t-out="product['price'][qty]" t-options='{"widget": "monetary", "display_currency": pricelist.currency_id}'>$15.00</span>
                                        </td>
                                    </t>
                                </tr>
                                <t t-if="is_product_tmpl and 'variants' in product">
                                    <tr t-foreach="product['variants']" t-as="variant">
                                        <td class="px-1">
                                            <a t-if="is_html_type" href="#" class="o_action ms-4" data-model="product.product" t-att-data-res-id="variant['id']">
                                                <span t-out="variant['name']">Acme Widget - Blue</span>
                                            </a>
                                            <span t-else="" class="ms-4" t-out="variant['name']">Acme Widget - Blue</span>
                                        </td>
                                        <td groups="uom.group_uom" class="px-1" t-out="product['uom']"/>
                                        <t t-foreach="quantities" t-as="qty">
                                            <td class="text-end px-1">
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
        <t t-call="web.html_container">
            <div class="page">
                <t t-if="is_html_type">
                    <t t-call="product.report_pricelist_page"/>
                </t>
                <t t-else="">
                    <t t-call="web.external_layout">
                        <t t-call="product.report_pricelist_page"/>
                    </t>
                </t>
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
                        <small class="text-nowrap" t-field="product.default_code" style="font-size: 0.875em;"/>
                    </div>
                    <div class="text-end" style="padding: 0 4px;">
                        <strong class="o_label_price_small" t-out="pricelist._get_product_price(product, 1, pricelist.currency_id or product.currency_id)"
                                t-options="{'widget': 'monetary', 'display_currency': pricelist.currency_id or product.currency_id, 'label_price': True}"/>
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
access_product_combo_user,product.combo.user,model_product_combo,base.group_user,1,0,0,0
access_product_combo_item_user,product.combo.item.user,model_product_combo_item,base.group_user,1,0,0,0
access_product_category_manager,product.category.manager,model_product_category,group_product_manager,1,1,1,1
access_product_template_manager,product.template.manager,model_product_template,group_product_manager,1,1,1,1
access_product_packaging_manager,product.packaging.manager,model_product_packaging,group_product_manager,1,1,1,1
access_product_supplierinfo_manager,product.supplierinfo.manager,model_product_supplierinfo,group_product_manager,1,1,1,1
access_product_pricelist_manager,product.pricelist.manager,model_product_pricelist,group_product_manager,1,1,1,1
access_product_pricelist_item_manager,product.pricelist.item.manager,model_product_pricelist_item,group_product_manager,1,1,1,1
access_product_product_manager,product.product manager,model_product_product,group_product_manager,1,1,1,1
access_product_attribute_manager,product.attribute manager,model_product_attribute,group_product_manager,1,1,1,1
access_product_attribute_value_manager,product.attribute value manager,model_product_attribute_value,group_product_manager,1,1,1,1
access_product_attribute_custom_value_manager,product.attribute.custom value manager,model_product_attribute_custom_value,group_product_manager,1,1,1,1
access_product_product_attribute_manager,product.template.attribute value manager,model_product_template_attribute_value,group_product_manager,1,1,1,1
access_product_template_attribute_exclusion_manager,product.template.attribute exclusion manager,model_product_template_attribute_exclusion,group_product_manager,1,1,1,1
access_product_template_attribute_line_manager,product.template.attribute line manager,model_product_template_attribute_line,group_product_manager,1,1,1,1
access_product_label_layout_user,product.label.layout.user,model_product_label_layout,base.group_user,1,1,1,1
access_product_tag_manager,product.tag.manager,model_product_tag,group_product_manager,1,1,1,1
access_product_combo_manager,product.combo.manager,model_product_combo,group_product_manager,1,1,1,1
access_product_combo_item_manager,product.combo.item.manager,model_product_combo_item,group_product_manager,1,1,1,1
access_product_document_user,access_product_document_user,model_product_document,base.group_user,1,1,1,1
access_update_product_attribute_value_manager,update.product.attribute.value,model_update_product_attribute_value,group_product_manager,1,1,1,0

```

## File: security\product_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="group_product_pricelist" model="res.groups">
        <field name="name">Basic Pricelists</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_stock_packaging" model="res.groups">
        <field name="name">Manage Product Packaging</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_product_variant" model="res.groups">
        <field name="name">Manage Product Variants</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_product_manager" model="res.groups">
        <field name="name">Product Creation</field>
        <field name="users" eval="[Command.link(ref('base.user_root')), Command.link(ref('base.user_admin'))]"/>
        <field name="category_id" ref="base.module_category_usability"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[Command.link(ref('group_product_manager'))]"/>
    </record>
    <record id="base.group_system" model="res.groups">
        <field name="implied_ids" eval="[Command.link(ref('group_product_manager'))]"/>
    </record>

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

    <record model="ir.rule" id="product_combo_comp_rule">
        <field name="name">Product combo multi-company rule</field>
        <field name="model_id" ref="model_product_combo"/>
        <field name="domain_force">
            ['|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]
        </field>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M28.235 7.242a6.487 6.487 0 0 0-7.242 2.823l-3.884 6.252a2.418 2.418 0 0 0-.198 2.159l10.002 25.523 17.025-6.636a3.228 3.228 0 0 0 1.839-4.185l-8.823-22.515a2.427 2.427 0 0 0-1.613-1.452l-7.106-1.969Zm-1.092 7.865a1.787 1.787 0 0 0 1.017-2.317 1.795 1.795 0 0 0-2.323-1.015 1.787 1.787 0 0 0-1.017 2.317c.36.92 1.4 1.374 2.323 1.015Z" fill="#2EBCFA"/><path fill-rule="evenodd" clip-rule="evenodd" d="M32.423 10.924a6.482 6.482 0 0 0-6.734-3.877l-7.322.88a2.43 2.43 0 0 0-1.814 1.194L4.435 30.055a3.226 3.226 0 0 0 1.185 4.413l15.831 9.115L35.19 19.852c.382-.66.43-1.462.13-2.163l-2.897-6.764Zm-6.84 4.062c.857.493 1.954.2 2.45-.655a1.786 1.786 0 0 0-.657-2.443 1.796 1.796 0 0 0-2.45.655 1.786 1.786 0 0 0 .657 2.443Z" fill="#985184"/><path fill-rule="evenodd" clip-rule="evenodd" d="M27.78 7.134a6.476 6.476 0 0 1 4.643 3.789l2.897 6.764c.3.701.252 1.503-.13 2.163L24.61 38.124l-7.7-19.649a2.418 2.418 0 0 1 .198-2.158l3.885-6.252a6.487 6.487 0 0 1 6.788-2.931Zm-2.027 7.937a1.786 1.786 0 0 1-.827-2.53 1.796 1.796 0 0 1 2.322-.721 1.787 1.787 0 0 1-.105 3.287 1.792 1.792 0 0 1-1.39-.036Z" fill="#144496"/></svg>

```

## File: static\src\js\product_attribute_value_list.js

```javascript
/** @odoo-module */

import { _t } from '@web/core/l10n/translation';
import { ConfirmationDialog, deleteConfirmationMessage } from '@web/core/confirmation_dialog/confirmation_dialog';
import { ListRenderer } from '@web/views/list/list_renderer';
import { registry } from '@web/core/registry';
import { useService } from '@web/core/utils/hooks';
import { X2ManyField, x2ManyField } from '@web/views/fields/x2many/x2many_field';


export class PAVListRenderer extends ListRenderer {
    setup() {
        super.setup();
        this.dialog = useService("dialog");
        this.orm = useService("orm");
    }

    async onDeleteRecord(record) {
        const message = await this.orm.call(
            'product.attribute.value',
            'check_is_used_on_products',
            [record.resId],
        )
        if (message) {
            return this.dialog.add(ConfirmationDialog, {
                title: _t("Invalid Operation"),
                body: message,
            });
        }
        if (record.isNew) {
            return super.onDeleteRecord(...arguments);
        }
        this.dialog.add(ConfirmationDialog, {
            title: _t("Bye-bye, record!"),
            body: deleteConfirmationMessage,
            confirmLabel: _t("Delete"),
            confirm: () => this.onConfirmDelete(record),
            cancel: () => { },
            cancelLabel: _t("No, keep it"),
        });
    }

    async onConfirmDelete(record) {
        await this.orm.unlink('product.attribute.value', [record.resId])
        const res = await super.onDeleteRecord(record);
        await this.props.list.model.root.save();
        return res;
    }
}

export class PAVOne2ManyField extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: PAVListRenderer,
    };
}

export const pavOne2ManyField = {
    ...x2ManyField,
    component: PAVOne2ManyField,
}

registry.category("fields").add("pavs_one2many", pavOne2ManyField);

```

## File: static\src\js\pricelist_report\product_pricelist_report.js

```javascript
/** @odoo-module **/

import { Component, markup, onRendered, onWillStart, useState } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { download } from "@web/core/network/download";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { useSetupAction } from "@web/search/action_hook";
import { Layout } from "@web/search/layout";
import { SelectCreateDialog } from "@web/views/view_dialogs/select_create_dialog";
import { standardActionServiceProps } from "@web/webclient/actions/action_service";

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
    static props = { ...standardActionServiceProps };
    static components = { Layout };
    static template = "product.ProductPricelistReport";

    setup() {
        this.action = useService("action");
        this.orm = useService("orm");
        this.dialog = useService("dialog");

        this.MAX_QTY = 5;
        const pastState = this.props.state || {};

        const active_model = pastState.activeModel || this.props.action.context.active_model;
        this.noProducts = active_model === 'product.pricelist';
        this.activeIds = this.noProducts ? [] : pastState.activeIds || this.props.action.context.active_ids;
        this.activeModel = this.noProducts ? 'product.template' : active_model;
        this.defaultPricelistId = this.noProducts ? this.props.action.context.active_id : false;

        this.state = useState({
            displayPricelistTitle: pastState.displayPricelistTitle || false,
            html: "",
            pricelists: [],
            _quantities: pastState.quantities || [1, 5, 10],
            selectedPricelist: {},
        });

        onWillStart(async () => {
            this.state.pricelists = await this.getPricelists();
            if (this.defaultPricelistId) {
                this.state.selectedPricelist = this.pricelists.find(p => p.id === this.defaultPricelistId) || this.pricelists[0];
            } else {
                this.state.selectedPricelist = pastState.selectedPricelist || this.pricelists[0];
            }
            if(this.noProducts){
                await this.onClickAddProducts();
            }
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
                    activeModel: this.activeModel,
                    activeIds: this.activeIds,
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
        if (this.noProducts) {
            // do not make an rpc to get empty report data
            this.state.html = "";
            return
        }
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

        const qty = parseInt(ev.target.previousSibling.value);
        if (qty > 0) {
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

        const parent = ev.target.parentElement;

        let classes = parent.getAttribute("class", "");
        let resModel = parent.getAttribute("data-model", "");
        let resId = parent.getAttribute("data-res-id", "");

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

    async onClickPrint() {
        if (this.noProducts) {
            this.action.doAction(
                sendCustomNotification("warning", _t("Please select some products first."))
            );
            return;
        }
        const selectedFormat = document.getElementById('formats').value;
        if (selectedFormat === 'pdf') {
            this.export_pdf();
        } else {
            await this.export_pricelist_csv_xlsx(selectedFormat);
        }
    }

    export_pdf() {
        this.action.doAction({
            type: 'ir.actions.report',
            report_type: 'qweb-pdf',
            report_name: 'product.report_pricelist',
            report_file: 'product.report_pricelist',
            data: this.reportParams,
        });
    }

    async export_pricelist_csv_xlsx(format) {
        try {
            await download({
                url: `/product/export/pricelist/`,
                data: {
                    report_data: JSON.stringify(this.reportParams),
                    export_format: format,
                }
            });
        } catch (error) {
            console.error(`Error exporting ${format.toUpperCase()} file:`, error);
            await this.action.doAction(
                sendCustomNotification(
                    "danger",
                    _t("Error exporting file. Please try again.")
                )
            );
        }
    }

    async onClickAddProducts() {
        this.dialog.add(SelectCreateDialog, {
            resModel: this.activeModel || 'product.template',
            title: _t("Add Products to pricelist report"),
            noCreate: true,
            onSelected: async (resIds) => {
                resIds.forEach((id) => {
                    if (!this.activeIds.includes(id)) {
                        this.activeIds.push(id);
                    }
                });
                this.noProducts = false;
                await this.renderHtml();
            },
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

registry.category("actions").add("generate_pricelist_report", ProductPricelistReport);

```

## File: static\src\js\pricelist_report\product_pricelist_report.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>

    <t t-name="product.ProductPricelistReport">
        <div class="o_action">
            <Layout display="{ controlPanel: {} }">
                <t t-set-slot="control-panel-always-buttons">
                    <button t-on-click="onClickPrint" type="button" class="btn btn-primary" title="Print">Print</button>
                    <select name="formats" id="formats" class="form-select border-1 w-auto">
                        <option value="pdf">pdf</option>
                        <option value="csv">csv</option>
                        <option value="xlsx">xlsx</option>
                    </select>
                </t>
                <t t-set-slot="layout-actions">
                    <form class="o_pricelist_report_form d-flex flex-row gap-1">
                        <div class="d-flex align-items-center gap-3 w-100">
                            <label class="visually-hidden" for="pricelists">Username</label>
                            <div class="input-group w-auto">
                                <div class="input-group-text fw-bold">Pricelist</div>
                                <select
                                    name="pricelists"
                                    id="pricelists"
                                    class="o_select form-select border-0"
                                    t-on-change="onSelectPricelist"
                                >
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
                                <label for="display_pricelist_title" class="form-check-label">Show Name</label>
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
                                <span class="o_badges_list d-flex">
                                    <t t-foreach="quantities" t-as="qty" t-key="qty">
                                        <span class="text-bg-300 o_remove_qty badge rounded-pill me-2 py-1 border " t-att-value="qty">
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
                    <div style="margin-left:300px;">
                        <button t-on-click="onClickAddProducts" string="All Products" class="btn btn-link">
                            <i class="fa fa-pencil"/>
                            Add Products
                        </button>
                    </div>
                </t>
                <div t-on-click="onClickLink">
                    <t t-out="html"/>
                </div>
                <t t-if="noProducts">
                    <div class="o_view_nocontent" role="alert">
                        <div class="o_nocontent_help">
                            <p class="o_view_nocontent_smiling_face">
                                No products found in the report
                            </p>
                            <p class="fs-5">
                                Add products using the "Add Products" button at the top right to
                                include them in the report.
                            </p>
                        </div>
                    </div>
                </t>
           </Layout>
        </div>
    </t>

</templates>

```

## File: static\src\js\product_document_kanban\product_document_kanban_controller.js

```javascript
/** @odoo-module **/

import { UploadButton } from '@product/js/product_document_kanban/upload_button/upload_button';
import { KanbanController } from '@web/views/kanban/kanban_controller';

export class ProductDocumentKanbanController extends KanbanController {
    static components = { ...KanbanController.components, UploadButton };

    setup() {
        super.setup();
        this.uploadRoute = '/product/document/upload';
        this.formData = {
            'res_model': this.props.context.default_res_model,
            'res_id': this.props.context.default_res_id,
        };
    }
}

```

## File: static\src\js\product_document_kanban\product_document_kanban_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t
        t-name="product.ProductDocumentKanbanView.Buttons"
        t-inherit="web.KanbanView.Buttons"
        t-inherit-mode="primary"
    >
        <xpath expr="//div[hasclass('o_cp_buttons')]" position="inside">
            <UploadButton
                formData="formData"
                allowedMIMETypes="allowedMIMETypes"
                load.bind="() => this.model.root.load()"
                uploadRoute="uploadRoute"
            />
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
    static components = {
        ...KanbanRenderer.components,
        FileUploadProgressContainer,
        FileUploadProgressKanbanRecord,
        KanbanRecord: ProductDocumentKanbanRecord,
    };
    static template = "product.ProductDocumentKanbanRenderer";
    setup() {
        super.setup();
        this.fileUploadService = useService("file_upload");
    }
}


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

## File: static\src\js\product_document_kanban\upload_button\upload_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { Component, useRef } from "@odoo/owl";
import { useBus, useService } from "@web/core/utils/hooks";

export class UploadButton extends Component {
    static template = "product.UploadButton";
    static props = {
        formData: { type: Object, optional: true},
        // See https://www.iana.org/assignments/media-types/media-types.xhtml
        allowedMIMETypes: { type: String, optional: true},
        load: Function,
        uploadRoute: String,
    }
    static defaultProps = {
        formData: {},
    }

    setup() {
        this.uploadFileInputRef = useRef("uploadFileInput");
        this.fileUploadService = useService("file_upload");
        this.notification = useService('notification');
        useBus(
            this.fileUploadService.bus,
            "FILE_UPLOAD_LOADED",
            async () => {
                await this.props.load();
            },
        );
    }

    async onFileInputChange(ev) {
        const files = [...ev.target.files].filter(file => this.validFileType(file));
        if (!files.length) {
            return;
        }
        await this.fileUploadService.upload(
            this.props.uploadRoute,
            files,
            {
                buildFormData: (formData) => this.buildFormData(formData)
            },
        );
        // Reset the file input's value so that the same file may be uploaded twice.
        ev.target.value = "";
    }

    /**
     * The `allowedMIMETypes` prop can restrict the file types users are guided to select. However,
     * the `accept` attribute doesn't enforce strict validation; it only suggests file types for
     * browsers.
     *
     * @param {File} file
     * @returns Whether the upload file's type is in the whitelist (`allowedMIMETypes`).
     */
    validFileType(file) {
        if (this.props.allowedMIMETypes && !this.props.allowedMIMETypes.includes(file.type)) {
            this.notification.add(
                _t(`Oops! '%(fileName)s' didn’t upload since its format isn’t allowed.`, {
                    fileName: file.name,
                }),
                {
                    type: "danger",
                }
            );
            return false;
        }
        return true;
    }

    buildFormData(formData) {
        for (const [key, value] of Object.entries(this.props.formData)) {
            formData.append(key, value);
        }
    }

}

```

## File: static\src\js\product_document_kanban\upload_button\upload_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="product.UploadButton">
        <input
            type="file"
            multiple="true"
            t-ref="uploadFileInput"
            t-att-accept="props.allowedMIMETypes"
            class="o_input_file o_hidden"
            t-on-change.stop="onFileInputChange"
        />
        <button
            type="button"
            name="product_upload_document"
            t-attf-class="btn btn-primary"
            t-on-click.stop.prevent="() => this.uploadFileInputRef.el.click()"
        >
            Upload
        </button>
    </t>
</templates>

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

import { rpc } from "@web/core/network/rpc";
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
            const orderLinesInfo = await rpc("/product/catalog/order_lines_info", this._getOrderLinesInfoParams(params, result.records.map((rec) => rec.id)));
            for (const record of result.records) {
                record.productCatalogData = orderLinesInfo[record.id];
            }
        }
        return result;
    }

    _getOrderLinesInfoParams(params, productIds) {
        return {
            order_id: params.context.order_id,
            product_ids: productIds,
            res_model: params.context.product_catalog_order_model,
            child_field: params.context?.child_field,
        }
    }
}

```

## File: static\src\product_catalog\kanban_record.js

```javascript
/** @odoo-module */
import { useSubEnv } from "@odoo/owl";
import { rpc } from "@web/core/network/rpc";
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
            childField: this.props.record.context?.child_field
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
        return rpc("/product/catalog/update_order_line_info", this._getUpdateQuantityAndGetPriceParams());
    }

    _getUpdateQuantityAndGetPriceParams() {
        return {
            order_id: this.env.orderId,
            product_id: this.env.productId,
            quantity: this.productCatalogData.quantity,
            res_model: this.env.orderResModel,
            child_field: this.env.childField,
        }
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
        <article
             t-att-class="getRecordClasses() + (productCatalogData.quantity ? ' o_product_added' : '')"
             t-att-data-id="props.record.id"
             t-att-tabindex="props.record.model.useSampleModel ? -1 : 0"
             t-on-click="onGlobalClick"
             t-ref="root">
            <div class="d-flex flex-column h-100">
                <t t-call="{{ templates[this.constructor.KANBAN_CARD_ATTRIBUTE] }}"
                   t-call-context="this.renderingContext"/>
                <t t-component="orderLineComponent" productId="props.record.resId" t-props="productCatalogData"/>
            </div>
            <t t-call="{{ this.constructor.menuTemplate }}"/>
        </article>
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
    static template = "product.ProductCatalogOrderLine";
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

    get showPrice() {
        return true;
    }
}

```

## File: static\src\product_catalog\order_line\order_line.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="product.ProductCatalogOrderLine">
        <!-- Replace the element found using the css selector by the content of the portalled
             template.  -->
        <t t-portal="`#product-${props.productId}-price`" t-if="showPrice">
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
        <span t-elif="props.readOnly" class="m-2 pt-3 border-top" t-out="props.warning">
            You can't edit this product in the catalog.
        </span>
        <div t-else="" class="d-flex justify-content-end align-items-center m-2">
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
    static subTemplates = {
        ...SearchPanel.subTemplates,
        filtersGroup: "ProductCatalogSearchPanel.FiltersGroup",
    };

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
            <list string="Attribute Values">
                <field name="name"/>
                <field name="default_extra_price"/>
            </list>
        </field>
    </record>

</odoo>

```

## File: views\product_attribute_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="attribute_tree_view" model="ir.ui.view">
        <field name="name">product.attribute.list</field>
        <field name="model">product.attribute</field>
        <field name="arch" type="xml">
            <list string="Variant Values">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="display_type"/>
                <field name="create_variant"/>
            </list>
        </field>
    </record>

    <record id="product_attribute_view_form" model="ir.ui.view">
        <field name="name">product.attribute.form</field>
        <field name="model">product.attribute</field>
        <field name="arch" type="xml">
            <form string="Product Attribute">
            <field name="number_related_products" invisible="1"/>
            <sheet>
                <widget
                    name="web_ribbon"
                    title="Archived"
                    bg_color="text-bg-danger"
                    invisible="active"
                />
                <div class="oe_button_box" name="button_box">
                    <button name="action_open_product_template_attribute_lines"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-bars"
                            invisible="not number_related_products">
                        <div class="o_stat_info">
                            <span class="o_stat_value"><field name="number_related_products"/></span>
                            <span class="o_stat_text">Products</span>
                        </div>
                    </button>
                </div>
                <group name="main_fields">
                    <group name="sale_main_fields">
                        <label for="name" string="Attribute Name"/>
                        <field name="name" nolabel="1"/>
                        <field name="display_type" widget="radio" options="{'horizontal': True}"/>
                        <field name="create_variant"
                               widget="radio"
                               options="{'horizontal': True}"
                               readonly="number_related_products != 0 or display_type == 'multi'"
                               force_save="1"/>
                    </group>
                </group>
                <notebook>
                    <page string="Attribute Values" name="attribute_values">
                        <field name="value_ids" widget="pavs_one2many" nolabel="1">
                            <list string="Values" editable="bottom">
                                <field name="sequence" widget="handle"/>
                                <field name="name"/>
                                <field name="display_type" column_invisible="True"/>
                                <field name="is_custom" width="120px"
                                       column_invisible="parent.display_type == 'multi'"/>
                                <field name="html_color"
                                       widget="color"
                                       column_invisible="parent.display_type != 'color'"
                                       invisible="image"/>
                                <field name="image"
                                       widget="image"
                                       options="{'size': [70, 70]}"
                                       class="oe_avatar text-start float-none"
                                       column_invisible="parent.display_type != 'color'"/>
                                <field name="default_extra_price" width="160px"/>
                                <button
                                    name="action_add_to_products"
                                    string="Add to products"
                                    type="object"
                                    icon="fa-plus"
                                    column_invisible="parent.number_related_products &lt; 1"
                                    invisible="pav_attribute_line_ids"
                                />
                                <button
                                    name="action_update_prices"
                                    string="Update extra prices"
                                    type="object"
                                    icon="fa-refresh"
                                    column_invisible="parent.number_related_products &lt; 1"
                                    invisible="not pav_attribute_line_ids or not default_extra_price_changed"
                                />
                            </list>
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
        <field name="view_mode">list,form</field>
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
                        <list string="Values">
                            <field name="name"/>
                            <field name="html_color"/>
                        </list>
                        <form string="Values">
                            <field name="name"/>
                        </form>
                    </field>
                </group>
            </form>
        </field>
    </record>

    <record id="product_template_attribute_value_view_tree" model="ir.ui.view">
        <field name="name">product.template.attribute.value.view.list</field>
        <field name="model">product.template.attribute.value</field>
        <field name="arch" type="xml">
            <list string="Attributes" create="0" delete="0" multi_edit="1">
                <field name="product_tmpl_id" string="Product" column_invisible="context.get('product_invisible', False)"/>
                <field name="attribute_id" optional="hide"/>
                <field name="name"/>
                <field name="display_type" optional="hide"/>
                <field name="html_color" invisible="display_type != 'color' or image" widget="color"/>
                <field name="image" invisible="display_type != 'color'" widget="image" options="{'size': [70, 70]}"/>
                <field name="ptav_active" optional="hide"/>
                <field name="price_extra" widget="monetary" options="{'field_digits': True}"/>
                <field name="currency_id" column_invisible="True"/>
            </list>
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
                        <field name="exclude_for" widget="one2many" mode="list">
                            <list editable="bottom">
                                <field name="product_tmpl_id" />
                                <field name="value_ids" widget="many2many_tags" options="{'no_create': True}" />
                            </list>
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

    <record id="product_attribute_search" model="ir.ui.view">
        <field name="name">product.attribute.view.search</field>
        <field name="model">product.attribute</field>
        <field name="arch" type="xml">
            <search>
                <filter string="Inactive" name="inactive" domain="[('active', '=', False)]"/>
                <field name="name"/>
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
                <chatter/>
            </form>
        </field>
    </record>

    <record id="product_category_list_view" model="ir.ui.view">
        <field name="name">product.category.list</field>
        <field name="model">product.category</field>
        <field name="priority">1</field>
        <field name="arch" type="xml">
            <list string="Product Categories">
                <field name="display_name" string="Product Category"/>
            </list>
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

## File: views\product_combo_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product.product_combo_view_form" model="ir.ui.view">
        <field name="name">product.combo.form</field>
        <field name="model">product.combo</field>
        <field name="arch" type="xml">
            <form string="Combo Choice">
                <div class="oe_title">
                    <label for="name" string="Combo Choice"/>
                    <h1>
                        <field
                            name="name"
                            placeholder="e.g. Burger Choice"
                            widget="text"
                            options="{'line_breaks': False}"
                            class="text-break"
                        />
                    </h1>
                </div>
                <group>
                    <field
                        name="company_id"
                        placeholder="Visible to all"
                        groups="base.group_multi_company"
                        options="{'no_create': True}"
                        class="oe_inline"
                    />
                </group>
                <field name="combo_item_ids">
                    <list editable="bottom">
                        <field name="currency_id" column_invisible="True"/>
                        <field name="product_id"/>
                        <field
                            name="lst_price"
                            widget="monetary"
                            options="{'field_digits': True}"
                        />
                        <field
                            name="extra_price"
                            widget="monetary"
                            options="{'field_digits': True}"
                        />
                    </list>
                </field>
            </form>
        </field>
    </record>

    <record id="product.product_combo_view_tree" model="ir.ui.view">
        <field name="name">product.combo.list</field>
        <field name="model">product.combo</field>
        <field name="arch" type="xml">
            <list string="Combo Choices">
                <field name="currency_id" column_invisible="True"/>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="base_price" widget="monetary" options="{'field_digits': True}"/>
                <field name="combo_item_count"/>
            </list>
        </field>
    </record>

    <record id="product.product_combo_action" model="ir.actions.act_window">
        <field name="name">Combo Choices</field>
        <field name="res_model">product.combo</field>
        <field name="view_mode">list,form</field>
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
                    <h1>
                        <field name="name" readonly="not datas and type != 'url'"/>
                    </h1>
                    <group>
                        <group>
                            <field name="type" readonly="datas or id" class="oe_inline"/>
                            <label for="datas"/>
                            <div class="o_row">
                                <field name="datas"
                                       filename="name"
                                       invisible="type == 'url'"
                                       required="type == 'binary'"
                                       class="oe_inline"/>
                            </div>
                            <field name="url" widget="url" invisible="type == 'binary'" required="type == 'url'"/>
                            <field name="res_name" string="Product Variant" invisible="res_model != 'product.product'"/>
                        </group>
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
            <kanban js_class="product_documents_kanban" can_open="0">
                <field name="sequence" widget="handle"/>
                <field name="ir_attachment_id"/>
                <field name="mimetype"/>
                <field name="type"/>
                <field name="name"/>
                <field name="active"/>
                <field name="res_model"/>
                <templates>
                    <t t-name="menu">
                        <a t-if="widget.editable" type="open" class="dropdown-item">Edit</a>
                        <a t-if="widget.deletable" type="delete" class="dropdown-item">Delete</a>
                        <a t-attf-href="/web/content/#{record.ir_attachment_id.raw_value}?download=true" download="" class="dropdown-item">Download</a>
                    </t>
                    <t t-name="card" class="o_kanban_attachment flex-row">
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <widget name="web_ribbon" title="Variant" bg_color="text-bg-secondary" invisible="not active or res_model != 'product.product'" groups="product.group_product_variant"/>
                        <aside class="o_kanban_image m-2">
                            <t t-set="webimage" t-value="new RegExp('image.*(gif|jpeg|jpg|png|webp)').test(record.mimetype.value)"/>
                            <t t-set="binaryPreviewable"
                                t-value="new RegExp('(image|video|application/pdf|text)').test(record.mimetype.value) &amp;&amp; record.type.raw_value === 'binary'"/>
                            <div t-attf-class="o_kanban_image_wrapper #{(webimage or binaryPreviewable) ? 'o_kanban_previewer' : ''}">
                                <div t-if="record.type.raw_value == 'url'" class="fa fa-link fa-3x text-muted" aria-label="Image is a link"/>
                                <img t-elif="webimage" t-attf-src="/web/image/#{record.ir_attachment_id.raw_value}" width="100" height="100" alt="Document" class="o_attachment_image"/>
                                <div t-else="" class="o_image o_image_thumbnail" t-att-data-mimetype="record.mimetype.value"/>
                            </div>
                        </aside>
                        <main class="ms-2 p-2">
                            <field name="name" class=" fw-bolder fs-5 text-truncate"/>
                            <field  t-if="record.type.raw_value == 'url'" name="url" widget="url"/>
                            <div class="d-flex flex-column mt-2">
                                <field name="res_name" class="text-decoration-underline fst-italic" invisible="res_model != 'product.product'"/>
                            </div>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="product_document_list" model="ir.ui.view">
        <field name="name">product.document.list</field>
        <field name="model">product.document</field>
        <field name="arch" type="xml">
            <list editable="top" multi_edit="true">
                <field name="res_model" column_invisible="True"/>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </list>
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
                <filter name="all" string="All" domain="['|', ('active', '=', True), ('active', '=', False)]"/>
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
        <field name="name">product.packaging.list.view</field>
        <field name="model">product.packaging</field>
        <field name="arch" type="xml">
            <list string="Product Packagings" name="packaging">
                <field name="sequence" widget="handle"/>
                <field name="product_id"/>
                <field name="name" string="Packaging"/>
                <field name="qty"/>
                <field name="product_uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"/>
                <field name="barcode" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                <field name="company_id" groups="!base.group_multi_company" invisible="1"/>
            </list>
        </field>
    </record>

    <record id="product_packaging_search_view" model="ir.ui.view">
        <field name="name">product.packaging.search</field>
        <field name="model">product.packaging</field>
        <field name="arch" type="xml">
            <search string="Product Packagings">
                <field name="name"/>
                <field name="product_id" string="Product"/>
            </search>
        </field>
    </record>

    <record id="product_packaging_tree_view2" model="ir.ui.view">
        <field name="name">product.packaging.list.view2</field>
        <field name="model">product.packaging</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="product.product_packaging_tree_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="replace"/>
            <xpath expr="//list[@name='packaging']" position="attributes">
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
            (0, 0, {'view_mode': 'list', 'view_id': ref('product_packaging_tree_view')}),
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
      <field name="name">product.pricelist.item.list</field>
      <field name="model">product.pricelist.item</field>
      <field name="priority">10</field>
      <field name="arch" type="xml">
        <list string="Price Rules">
          <field name="pricelist_id"/>
          <field name="name" string="Applied On"/>
          <field name="price"/>
          <field name="min_quantity" colspan="4"/>
          <field name="date_start" optional="hide"/>
          <field name="date_end" optional="hide"/>
          <field name="company_id" groups="base.group_multi_company" optional="show"/>
        </list>
      </field>
    </record>

    <record id="product_pricelist_item_tree_view_from_product" model="ir.ui.view">
        <!-- Access and edit price rules from a given product/product variant -->
        <field name="name">product.pricelist.item.list</field>
        <field name="model">product.pricelist.item</field>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <list string="Pricelist Rules" editable="bottom">
                <!-- Scope = coming from a product/product template -->
                <field name="pricelist_id" string="Pricelist" options="{'no_create_edit':1, 'no_open': 1}"/>
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
                    placeholder="All variants"
                    options="{'no_create_edit':1, 'no_open': 1}"/>
                <field name="company_id" column_invisible="True"/>
                <field name="categ_id" column_invisible="True"/>
                <field name="product_tmpl_id"
                  column_invisible="context.get('active_model') != 'product.category'"
                  required="applied_on == '1_product'"
                  domain="[('categ_id', '=', context.get('default_categ_id', True)), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"
                  options="{'no_create_edit':1, 'no_open': 1}"/>
                <field name="fixed_price"
                        widget="monetary"
                        string="Price"
                        options="{'currency_field': 'currency_id', 'field_digits': True}"
                        required='1'/>
                <field name="min_quantity" colspan="4"/>
                <field name="currency_id" column_invisible="True"/>
                <field name="date_start" optional="show"/>
                <field name="date_end" optional="show"/>
                <field name="applied_on" column_invisible="True"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide" options="{'no_create':1, 'no_open': 1}"/>
            </list>
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
                    <field name="price" invisible="1"/>
                    <group name="pricelist_rule_computation">
                        <group name="pricelist_rule_method">
                            <field name="display_applied_on"
                                string="Apply To"
                                widget="radio"
                                options="{'horizontal': true}"/>
                            <field name="categ_id"
                                options="{'no_create':1}"
                                invisible="display_applied_on != '2_product_category'"
                                placeholder="All categories"/>
                            <field name="product_tmpl_id"
                                options="{'no_create':1}"
                                invisible="display_applied_on != '1_product'"
                                placeholder="All products"/>
                            <field name="product_variant_count" invisible="1"/>
                            <field name="product_id"
                                options="{'no_create':1}"
                                invisible="product_variant_count &lt; 2"
                                placeholder="All variants"/>
                            <field name="compute_price"
                                string="Price Type"
                                widget="radio"
                                options="{'horizontal': true}"
                                invisible="display_applied_on == '1_product' and compute_price == 'fixed_price'"/>
                            <label for="fixed_price" invisible="compute_price != 'fixed'"/>
                            <div class="o_row" invisible="compute_price != 'fixed'" style="width: 60% !important;">
                                <field name="fixed_price"
                                    widget="monetary"
                                    options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                                <span class="d-flex gap-2 p-0" invisible="not product_uom ">per<field
                                        name="product_uom"/></span>
                            </div>
                            <label for="percent_price" string="Discount"
                                invisible="compute_price != 'percentage'"/>
                            <div class="o_row gap-3" invisible="compute_price != 'percentage'">
                                <field name="percent_price"/>
                                <span>%</span>
                                <span>on</span>
                                <field name="base_pricelist_id" placeholder="sales price"/>
                            </div>
                        </group>
                        <group name="pricelist_rule_limits">
                            <field name="min_quantity" string="Min Qty"/>
                            <field name="date_start"
                                string="Validity Period"
                                widget="daterange"
                                options="{'end_date_field': 'date_end'}"/>
                            <field name="date_end" invisible="1"/>
                        </group>
                    </group>
                    <group name="pricelist_rule_base" invisible="compute_price != 'formula'">
                        <group>
                            <field name="base" string="Based price"/>
                            <field name="base_pricelist_id"
                                invisible="base != 'pricelist'"
                                readonly="base != 'pricelist'"
                                required="compute_price == 'formula' and base == 'pricelist'"
                                string="Other Pricelist"/>
                            <label for="price_discount" string="Discount"
                                invisible="base == 'standard_price'"/>
                            <div class="o_row" invisible="base == 'standard_price'" style="width: 40% !important;">
                                <field name="price_discount"/>
                                <span>%</span>
                            </div>
                            <label for="price_markup" string="Markup"
                                invisible="base != 'standard_price'"/>
                            <div class="o_row" invisible="base != 'standard_price'" style="width: 40% !important;">
                                <field name="price_markup"/>
                                <span>%</span>
                            </div>
                            <field name="price_round" string="Round off to"/>
                            <field name="price_surcharge"
                                widget="monetary"
                                options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                            <label string="Margins" for="price_min_margin"
                                groups="base.group_no_one"/>
                            <div class="d-flex align-items-baseline" groups="base.group_no_one">
                                <field name="price_min_margin"
                                    string="Min. Margin"
                                    class="oe_inline"
                                    widget="monetary"
                                    options="{'field_digits': True}"/>
                                <i class="fa fa-long-arrow-right mx-2 oe_edit_only"
                                    aria-label="Arrow icon" title="Arrow"/>
                                <field name="price_max_margin"
                                    string="Max. Margin"
                                    class="oe_inline"
                                    widget="monetary"
                                    nolabel="1"
                                    options="{'field_digits': True}"/>
                            </div>
                        </group>
                        <div>
                            <div class="alert alert-info" role="alert" style="white-space: pre;">
                                <field name="rule_tip"/>
                            </div>
                            <div class="alert alert-info"
                                role="alert"
                                style="white-space: pre;">
                                <b>Tip: want to round at 9.99?</b>
                                <div>round off to 10.00 and set an extra at -0.01</div>
                            </div>
                        </div>
                    </group>

                    <group string="Company Settings" groups="base.group_no_one">
                        <group name="pricelist_rule_related">
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
        <field name="name">product.pricelist.list</field>
        <field name="model">product.pricelist</field>
        <field name="arch" type="xml">
            <list string="Products Price List" sample="1">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="country_group_ids"
                    widget="many2many_tags"
                    placeholder="All countries"/>
                <field name="currency_id" groups="base.group_multi_currency"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </list>
        </field>
    </record>

    <record id="product_pricelist_view_kanban" model="ir.ui.view">
        <field name="name">product.pricelist.kanban</field>
        <field name="model">product.pricelist</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
                <templates>
                    <t t-name="card" class="flex-row fw-bolder">
                        <field name="name"/>
                        <div class="ms-auto align-items-center">
                            <i class="fa fa-money" role="img" aria-label="Currency" title="Currency"></i> <field name="currency_id"/>
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
                <header>
                    <button name="action_open_pricelist_report" type="object" string="Print"/>
                </header>
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
                        <group>
                            <field name="country_group_ids" widget="many2many_tags"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="pricelist_rules" string="Price Rules">
                            <field name="item_ids" nolabel="1" context="{'default_base':'list_price'}">
                                <list string="Pricelist Rules">
                                    <field name="product_tmpl_id" column_invisible="True"/>
                                    <field name="name" string="Apply on"/>
                                    <field name="price" string="Price"/>
                                    <field name="min_quantity"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="base" column_invisible="True"/>
                                    <field name="price_discount" column_invisible="True"/>
                                    <field name="applied_on" column_invisible="True"/>
                                    <field name="compute_price" column_invisible="True"/>
                                </list>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <chatter/>
            </form>
        </field>
    </record>

    <record id="product_pricelist_action2" model="ir.actions.act_window">
        <field name="name">Pricelists</field>
        <field name="res_model">product.pricelist</field>
        <field name="path">pricelists</field>
        <field name="view_mode">list,kanban,form</field>
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
        <field name="view_mode">list,form</field>
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
                                <field name="product_uom" groups="uom.group_uom" options="{'no_open': True}"/>
                            </div>
                            <label for="price" string="Unit Price"/>
                            <div class="o_row">
                                <field name="price" class="oe_inline" /><field name="currency_id" groups="base.group_multi_currency" options="{'no_open': True}"/>
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
                <field name="currency_id"/>
                <templates>
                    <t t-name="card">
                        <div class="d-flex fw-bolder mb4">
                            <field name="partner_id" />
                            <field name="price" widget="monetary" class="ms-auto"/>
                        </div>
                        <div class="d-flex">
                            <field name="min_qty"/>
                            <field name="delay" class="ms-auto me-1"/>days
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="product_supplierinfo_tree_view" model="ir.ui.view">
        <field name="name">product.supplierinfo.list.view</field>
        <field name="model">product.supplierinfo</field>
        <field name="arch" type="xml">
            <list string="Vendor Information" multi_edit="1">
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
            </list>
        </field>
    </record>

    <record id="product_supplierinfo_type_action" model="ir.actions.act_window">
        <field name="name">Vendor Pricelists</field>
        <field name="res_model">product.supplierinfo</field>
        <field name="view_mode">list,form,kanban</field>
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
                                   context="{'list_view_ref':'product.product_template_view_tree_tag'}"
                            />
                        </page>
                        <page string="Product Variants" name="page_product_variants">
                            <field name="product_product_ids"
                                   context="{'list_view_ref':'product.product_product_view_tree_tag'}"
                            />
                        </page>
                        <page string="All Products" name="page_all_products" invisible="not product_ids">
                            <field name="product_ids"
                                   context="{'list_view_ref':'product.product_product_view_tree_tag'}"
                            />
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record id="product_tag_tree_view" model="ir.ui.view">
        <field name="name">product.tag.list</field>
        <field name="model">product.tag</field>
        <field name="arch" type="xml">
            <list string="Product Tags" multi_edit="1">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="product_template_ids" widget="many2many_tags" optional="show"/>
                <field name="product_product_ids" widget="many2many_tags" string="Product Variant" optional="show"/>
            </list>
        </field>
    </record>
    <record id="product_tag_action" model="ir.actions.act_window">
        <field name="name">Product Tags</field>
        <field name="res_model">product.tag</field>
        <field name="view_mode">list,form</field>
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

## File: views\product_template_attribute_line_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="product_template_attribute_line_view_tree" model="ir.ui.view">
        <field name="name">product.template.attribute.line.view.list</field>
        <field name="model">product.template.attribute.line</field>
        <field name="arch" type="xml">
            <list editable="bottom" multi_edit="1" create="false">
                <field name="product_tmpl_id" readonly="1"/>
                <field name="attribute_id" readonly="1"/>
                <field name="value_ids"
                       widget="many2many_tags"
                       options="{'no_create_edit': True}"
                       context="{'default_attribute_id': attribute_id}"/>
            </list>
        </field>
    </record>

</odoo>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_tree_view" model="ir.ui.view">
        <field name="name">product.template.product.list</field>
        <field name="model">product.template</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <list string="Product" multi_edit="1" sample="1">
            <header>
                <button string="Print Labels" type="object" name="action_open_label_layout"/>
            </header>
                <field name="product_variant_count" column_invisible="True"/>
                <field name="sale_ok" column_invisible="True"/>
                <field name="currency_id" column_invisible="True"/>
                <field name="cost_currency_id" column_invisible="True"/>
                <field name="is_favorite" widget="boolean_favorite" optional="show" nolabel="1"/>
                <field name="name" string="Product Name"/>
                <field name="default_code" optional="show"/>
                <field name="product_tag_ids" widget="many2many_tags" optional="show"/>
                <field name="barcode" optional="hide" readonly="product_variant_count != 1"/>
                <field name="company_id" options="{'no_create': True}"
                    groups="base.group_multi_company" optional="hide"/>
                <field name="list_price" string="Sales Price" widget='monetary' options="{'currency_field': 'currency_id'}" optional="show" decoration-muted="not sale_ok"/>
                <field name="standard_price" widget='monetary' options="{'currency_field': 'cost_currency_id'}" optional="show" readonly="1"/>
                <field name="categ_id" optional="hide"/>
                <field name="type" optional="hide" readonly="1"/>
                <field name="uom_id" string="Unit" readonly="1" optional="show" groups="uom.group_uom"/>
                <field name="active" column_invisible="True"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </list>
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
                <field name="default_code" string="Reference" invisible="product_variant_count &gt; 1"/>
                <field name="valid_product_template_attribute_line_ids" invisible="1"/>
                <field name="barcode"
                    invisible="product_variant_count &gt; 1
                        or (product_variant_count == 0
                        and valid_product_template_attribute_line_ids)
                        or type in ['service', 'combo']"/>
            </field>

            <button name="action_open_documents" position="before">
                <button name="%(product.product_variant_action)d" type="action"
                    icon="fa-sitemap" class="oe_stat_button"
                    invisible="product_variant_count &lt;= 1"
                    groups="product.group_product_variant">
                    <field string="Variants" name="product_variant_count" widget="statinfo" />
                </button>
            </button>

            <xpath expr="//page[@name='general_information']" position="after">
                <page
                    name="variants"
                    string="Attributes &amp; Variants"
                    groups="product.group_product_variant"
                    invisible="type == 'combo'"
                >
                    <field name="attribute_line_ids" widget="one2many" context="{'show_attribute': False}">
                        <list string="Variants" editable="bottom" decoration-info="value_count &lt;= 1">
                            <field name="value_count" column_invisible="True"/>
                            <field name="sequence" widget="handle"/>
                            <field name="attribute_id" readonly="id"/>
                            <field name="value_ids" widget="many2many_tags" options="{'no_create_edit': True, 'color_field': 'color'}" context="{'default_attribute_id': attribute_id, 'show_attribute': False}"/>
                            <button string="Configure" class="float-end btn-secondary"
                                type="object" name="action_open_attribute_values"
                                groups="product.group_product_variant"/>
                        </list>
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
                <field name="currency_id"/>
                <field name="activity_state"/>
                <field name="categ_id"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="card" class="flex-row">
                        <main class="pe-2">
                            <div class="mb-1">
                                <div class="d-flex mb-0 h5">
                                    <field class="me-1" name="is_favorite" widget="boolean_favorite" nolabel="1"/>
                                    <field name="name"/>
                                </div>
                                <span t-if="record.default_code.value">
                                    [<field name="default_code"/>]
                                </span>
                                <strong t-if="record.product_variant_count.value &gt; 1">
                                    <field name="product_variant_count"/> Variants
                                </strong>
                            </div>
                            <span>
                                Price: <field name="list_price" widget="monetary" options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                            </span>
                            <field name="product_properties" widget="properties"/>
                        </main>
                        <aside>
                            <field
                                name="image_128"
                                widget="image"
                                alt="Product"
                                options="{'img_class': 'w-100 object-fit-contain'}"
                                invisible="not image_128"
                            />
                        </aside>
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
        <field name="view_mode">kanban,list,form</field>
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
        <field name="name">Pricelist Report</field>
        <field name="groups_id" eval="[(4, ref('product.group_product_pricelist'))]"/>
        <field name="model_id" ref="product.model_product_template"/>
        <field name="binding_model_id" ref="product.model_product_template"/>
        <field name="state">code</field>
        <field name="code">
action = {
    'name': 'Pricelist Report',
    'type': 'ir.actions.client',
    'tag': 'generate_pricelist_report',
    'context': env.context,
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
                    <button string="Print Labels"
                        type="object"
                        name="action_open_label_layout"
                        invisible="type == 'service'"
                    />
                </header>
                <sheet name="product_form">
                    <field name='product_variant_count' invisible='1'/>
                    <field name='is_product_variant' invisible='1'/>
                    <field name='attribute_line_ids' invisible='1'/>
                    <field name="company_id" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button"
                               name="open_pricelist_rules"
                               icon="fa-list-ul"
                               groups="product.group_product_pricelist"
                               type="object">
                               <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value o_row">
                                        <field name="pricelist_item_count"/>
                                        <span class="o_stat_text" invisible="pricelist_item_count == 1">
                                            Rules
                                        </span>
                                        <span class="o_stat_text" invisible="pricelist_item_count != 1">
                                            Rule
                                        </span>
                                    </span>
                                    <span class="o_stat_text" invisible="pricelist_item_count == 1">
                                        Pricelists
                                    </span>
                                    <span class="o_stat_text" invisible="pricelist_item_count != 1">
                                        Pricelist
                                    </span>
                               </div>
                        </button>
                        <!-- Dummy tag for organizing buttons, using position='replace' when inheriting -->
                        <span id="button_website" invisible="1"/>
                        <button class="oe_stat_button"
                                name="action_open_documents"
                                type="object"
                                icon="fa-file-text-o">
                            <field string="Documents" name="product_document_count" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <field name="id" invisible="True"/>
                    <field
                        name="image_1920"
                        widget="image"
                        class="oe_avatar"
                        options="{'convert_to_webp': True, 'preview_image': 'image_128'}"
                    />
                    <div class="oe_title">
                        <label for="name" string="Product"/>
                        <h1>
                            <div class="d-flex">
                                <field name="is_favorite" widget="boolean_favorite" class="me-3" nolabel="1"/>
                                <field class="text-break" name="name" options="{'line_breaks': False}" widget="text" placeholder="e.g. Cheese Burger"/>
                            </div>
                        </h1>
                    </div>
                    <div name="options">
                        <span class="d-inline-flex">
                            <field name="sale_ok"/>
                            <label for="sale_ok"/>
                        </span>
                    </div>
                    <notebook>
                        <page string="General Information" name="general_information">
                            <group>
                                <group name="group_general">
                                    <field name="active" invisible="1"/>
                                    <field name="type" widget="radio" options="{'horizontal': True}"/>
                                    <field
                                        name="combo_ids"
                                        widget="many2many_tags"
                                        placeholder="e.g. Starter - Meal - Desert"
                                        invisible="type != 'combo'"
                                        options="{
                                            'no_quick_create': True,
                                            'edit_tags': True,
                                        }"
                                    />
                                    <field name="service_tracking" invisible="1 or type != 'service'"/>
                                    <field name="product_tooltip"
                                        class="fst-italic text-muted"
                                        string=""
                                        invisible="not product_tooltip"
                                    />
                                </group>
                                <group name="group_standard_price">
                                    <label for="list_price"/>
                                    <div name="list_price_uom">
                                        <field name="list_price"
                                            class="oe_inline"
                                            widget="monetary"
                                            options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                                        <span
                                            name="uom_span"
                                            groups="uom.group_uom"
                                            invisible="type == 'combo'"
                                        >
                                            per
                                            <field
                                                name="uom_id"
                                                class="oe_inline" style="max-width:136px"
                                                options="{'no_create': True}"
                                            />
                                        </span>
                                    </div>

                                    <!--
                                        `id` condition to prevent invisibility on new products
                                        when `product_variant_count` is 0.
                                        `product_variant_count != 1` to handle cases with dynamic
                                        / multiple variants since id will be set after saving.
                                        #TODO : revert this and create a compute field in master
                                        to ensure invisivility for dynamic variants only.
                                    -->
                                    <label
                                        for="standard_price"
                                        invisible="(id and product_variant_count != 1
                                                    and not is_product_variant) or type == 'combo'"
                                        id="standard_price_label"
                                    />
                                    <div
                                        name="standard_price_uom"
                                        invisible="(id and product_variant_count != 1
                                                    and not is_product_variant) or type == 'combo'"
                                    >
                                        <field
                                            name="standard_price"
                                            class="oe_inline"
                                            widget="monetary"
                                            options="{
                                                'currency_field': 'cost_currency_id',
                                                'field_digits': True,
                                            }"
                                        />
                                        <span groups="uom.group_uom">
                                            per
                                            <field name="uom_name" class="oe_inline"/>
                                        </span>
                                    </div>
                                    <field name="categ_id" string="Category"/>
                                    <field name="company_id"
                                        groups="base.group_multi_company"
                                        options="{'no_create': True}"
                                        placeholder="Visible to all"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="cost_currency_id" invisible="1"/>
                                    <field name="product_variant_id" invisible="1"/>
                                </group>
                            </group>
                            <group name="internal_notes" string="Internal Notes">
                                <field colspan="2" name="description" nolabel="1" placeholder="This note is only for internal purposes."/>
                            </group>
                        </page>
                        <page string="Sales" name="sales" invisible="1 or not sale_ok">
                            <group name="sale">
                                <group string="Upsell &amp; Cross-Sell" name="upsell" invisible="1"/>
                                <group name="extra_info" string="Extra Info">
                                    <field name="product_tag_ids" widget="many2many_tags" context="{'product_template_id': id}"/>
                                </group>
                            </group>
                            <group>
                                <group string="Quotation Description" name="description">
                                    <field colspan="2" name="description_sale" nolabel="1" placeholder="This note is added to sales orders and invoices."/>
                                </group>
                            </group>
                        </page>
                        <page
                            string="Purchase"
                            name="purchase"
                            invisible="1 or not purchase_ok or type == 'combo'"
                            groups="uom.group_uom"
                        >
                            <group name="purchase">
                                <group string="Vendor Bills" name="bill">
                                    <field name="uom_po_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                </group>
                            </group>
                        </page>
                        <page
                            string="Inventory"
                            name="inventory"
                            groups="product.group_stock_packaging"
                            invisible="type in ['service', 'combo']"
                        >
                            <group name="inventory">
                                <group name="group_lots_and_weight" string="Logistics" invisible="type != 'consu'">
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
                                invisible="(type != 'consu' or product_variant_count &gt; 1) and not is_product_variant"
                                groups="product.group_stock_packaging">
                                <field colspan="2" name="packaging_ids" nolabel="1" context="{'list_view_ref':'product.product_packaging_tree_view2', 'default_company_id': company_id}"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <chatter/>
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
                <field name="product_tag_ids" string="Tags"/>
                <separator/>
                <filter string="Services" name="services" domain="[('type','=','service')]"/>
                <filter string="Goods" name="goods" domain="[('type', '=', 'consu')]"/>
                <filter string="Combo" name="combo" domain="[('type', '=', 'combo')]"/>
                <separator/>
                <filter string="Sales" name="filter_to_sell" domain="[('sale_ok','=',True)]"/>
                <filter string="Purchase" name="filter_to_purchase" domain="[('purchase_ok', '=', True)]"/>
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
                <filter string="Favorites" name="favorites" domain="[('is_favorite', '=', True)]"/>
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
        <field name="view_mode">kanban,list,form</field>
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
                <field name="product_tag_ids" position="replace">
                    <field name="all_product_tag_ids" string="Tags"/>
                </field>
                <field name="name" position="replace">
                    <field name="name" string="Product" filter_domain="['|', '|', ('default_code', 'ilike', self), ('name', 'ilike', self), ('barcode', 'ilike', self)]"/>
                </field>
                <field name="attribute_line_ids" position="replace">
                    <field name="product_template_attribute_value_ids" groups="product.group_product_variant"/>
                    <field name="product_tmpl_id" string="Product Template"/>
                </field>
                <filter name="categ_id" position="after">
                    <filter string="Product Template" name="group_by_product_tmpl_id" context="{'group_by':'product_tmpl_id'}"/>
                </filter>
            </field>
        </record>

        <record id="product_normal_action" model="ir.actions.act_window">
            <field name="name">Product Variants</field>
            <field name="res_model">product.product</field>
            <field name="view_mode">list,form,kanban,activity</field>
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
                        <field name="type" invisible="1"/>
                        <button string="Print Labels"
                            type="object"
                            name="action_open_label_layout"
                            invisible="type == 'service'"
                        />
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box"/>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name="active" invisible="1"/>
                        <field name="id" invisible="1"/>
                        <field name="company_id" invisible="1"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'convert_to_webp': True,'preview_image': 'image_128'}"/>
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
                                    <field name="standard_price"
                                        class="oe_inline"
                                        widget="monetary"
                                        options="{'currency_field': 'cost_currency_id', 'field_digits': True}"/>
                                </div>
                                <field name="currency_id" invisible='1'/>
                                <field name="cost_currency_id" invisible="1"/>
                            </group>
                        </group>
                        <group>
                            <group name="weight" string="Logistics" invisible="type != 'consu'">
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
                            <group name="sales" string="Sales">
                                <field name="product_tag_ids" widget="many2many_tags" readonly="1"/>
                                <field name="additional_product_tag_ids" widget="many2many_tags" context="{'product_variant_id': id}"/>
                            </group>
                        </group>
                        <group>
                            <group name="packaging" string="Packaging" groups="product.group_stock_packaging">
                                <field colspan="2" name="packaging_ids" nolabel="1"
                                    context="{'list_view_ref':'product.product_packaging_tree_view2', 'default_company_id': company_id}"/>
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
                          (0, 0, {'view_mode': 'list'}),
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
            <field name="name">product.product.list</field>
            <field name="model">product.product</field>
            <field eval="7" name="priority"/>
            <field name="arch" type="xml">
                <list string="Product Variants" multi_edit="1" duplicate="false" sample="1">
                <header>
                    <button string="Print Labels" type="object" name="action_open_label_layout"/>
                </header>
                    <field name="is_favorite" widget="boolean_favorite" nolabel="1" readonly="1"/>
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
                </list>
            </field>
        </record>

        <record model="ir.ui.view" id="product_template_view_tree_tag">
            <field name="name">product.template.view.list.tag</field>
            <field name="model">product.template</field>
            <field name="arch" type="xml">
                <list string="Product Templates" editable="bottom">
                    <field name="name" readonly="1"/>
                    <field name="default_code" readonly="1" optional="show"/>
                    <field name="description" readonly="1" optional="show"/>
                </list>
            </field>
        </record>

        <record model="ir.ui.view" id="product_product_view_tree_tag">
            <field name="name">product.product.view.list.tag</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <list string="Product Variants" editable="bottom">
                    <field name="name" readonly="1"/>
                    <field name="default_code" readonly="1" optional="show"/>
                    <field name="product_template_variant_value_ids"
                           widget="many2many_tags"
                           groups="product.group_product_variant"
                           readonly="1"
                    />
                    <field name="description" readonly="1" optional="show"/>
                </list>
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
                <xpath expr="//field[@name='is_favorite']" position="attributes">
                    <attribute name="readonly">1</attribute>
                </xpath>
                <field name="list_price" position="attributes">
                   <attribute name="invisible">1</attribute>
                   <attribute name="readonly">product_variant_count &gt; 1</attribute>
                </field>
                <label for="list_price" position="replace">
                    <label for="lst_price"/>
                </label>
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
                    <field name="product_template_variant_value_ids" widget="many2many_tags" readonly="1" invisible="not product_template_variant_value_ids" groups="product.group_product_variant"/>
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

        <record id="product_product_view_form_normalized" model="ir.ui.view">
            <field name="name">product.product.view.form.normalized</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <form>
                    <sheet>
                        <field name="image_1920" widget="image" class="oe_avatar" nolabel="1" options="{'convert_to_webp': True,'preview_image': 'image_128'}"/>
                        <group name="name">
                                <field name="company_id" invisible="1"/>
                                <field name="currency_id" invisible="1"/>
                                <field name="cost_currency_id" invisible="1"/>
                                <field name="name" class="oe_inline" placeholder="e.g. Cheese Burger" string="Product Name"/>
                                <field name="barcode" class="oe_inline" placeholder="e.g. 1234567890"/>
                                <field name="list_price" class="oe_inline" widget="monetary"
                                        options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                                <field name="description" invisible="1"/>  <!-- Hide the description field for set data in product barcodelookup -->
                                <field name="weight" invisible="1"/>       <!-- Hide the weight field for set data in product barcodelookup -->
                                <field name="categ_id" invisible="1"/>     <!-- Hide the categ_id field set data in product barcodelookup -->
                        </group>
                    </sheet>
                </form>
            </field>
        </record>


        <record id="product_kanban_view" model="ir.ui.view">
            <field name="name">Product Kanban</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <kanban sample="1">
                    <field name="activity_state"/>
                    <field name="color"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="card" class="row g-0">
                            <main class="col-10 pe-2">
                                <div class="d-flex mb-1 h5">
                                    <field class="me-1" name="is_favorite" widget="boolean_favorite" nolabel="1"/>
                                    <field name="name"/>
                                </div>
                                <span t-if="record.default_code.value">
                                    [<field name="default_code"/>]
                                </span>
                                <field name="product_template_variant_value_ids" groups="product.group_product_variant" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                <span>
                                    Price: <field name="lst_price"></field>
                                </span>
                            </main>
                            <aside class="col-2">
                                <field name="image_128" widget="image" options="{'img_class': 'o_image_64_contain mw-100'}" alt="Product"/>
                            </aside>
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
            <field name="view_mode">kanban,list,form,activity</field>
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
        <field name="name">Pricelist Report</field>
        <field name="groups_id" eval="[(4, ref('group_product_pricelist'))]"/>
        <field name="model_id" ref="product.model_product_product"/>
        <field name="binding_model_id" ref="product.model_product_product"/>
        <field name="state">code</field>
        <field name="code">
action = {
    'name': 'Pricelist Report',
    'type': 'ir.actions.client',
    'tag': 'generate_pricelist_report',
    'context': env.context,
}
        </field>
    </record>

    <record id="product_view_kanban_catalog" model="ir.ui.view">
        <field name="name">product.view.kanban.catalog</field>
        <field name="model">product.product</field>
        <field name="arch" type="xml">
            <kanban records_draggable="0" js_class="product_kanban_catalog">
                <field name="id"/>
                <templates>
                    <t t-name="menu">
                        <div class="o_product_catalog_cancel_global_click">
                            <a role="menuitem" type="open" class="dropdown-item border-top-0">Edit</a>
                        </div>
                    </t>
                    <t t-name="card" class="p-0">
                    <!-- TODO : not getting border when select record -->
                        <div class="d-flex flex-grow-1 m-2">
                            <div class="p-2 pt-1 d-flex flex-column m-0"
                                    t-attf-id="product-{{record.id.raw_value}}">
                                <div class="d-flex">
                                    <field class="mt-1 me-1"
                                        name="is_favorite"
                                        widget="boolean_favorite"
                                        nolabel="1"
                                        readonly="1"/>
                                    <field name="name" class="fw-bolder fs-4 text-reset mb-1"/>
                                </div>
                                <div t-if="record.default_code.value">
                                    [<field name="default_code"/>]
                                </div>
                                <!-- Used by @web/product/js/product_catalog/order_line to
                                        show the price using a t-portal. -->
                                <div name="o_kanban_price"
                                        t-attf-id="product-{{record.id.raw_value}}-price"/>
                                <field name="product_template_attribute_value_ids"
                                        widget="many2many_tags"
                                        domain="[('id', 'in', parent.ids)]"
                                        groups="product.group_product_variant"
                                        options="{'color_field': 'color'}"/>
                            </div>
                            <field name="image_128" widget="image" alt="Product" invisible="not image_128" class="ms-auto" options="{'size': [55, 55]}"/>
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
                <filter string="Favorites" name="favorites" domain="[('is_favorite', '=', True)]"/>
                <separator/>
                <filter string="Services" name="services" domain="[('type', '=', 'service')]"/>
                <filter string="Goods"
                        name="goods"
                        domain="[('type', '=', 'consu')]"/>
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
                        <field name="extra_html" invisible="print_format != '2x7xprice'"/>
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

## File: wizard\update_product_attribute_value.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.fields import Command


class UpdateProductAttributeValue(models.TransientModel):
    _name = 'update.product.attribute.value'
    _description = "Update product attribute value"

    attribute_value_id = fields.Many2one('product.attribute.value', required=True)

    mode = fields.Selection(
        selection=[
            ('add', "Add to existing products"),
            ('update_extra_price', "Update the extra price on existing products")
        ]
    )
    message = fields.Char(compute='_compute_message')
    product_count = fields.Integer(compute='_compute_product_count')

    @api.depends('product_count', 'mode', 'attribute_value_id')
    def _compute_message(self):
        self.message = ''
        for wizard in self:
            if wizard.mode == 'add':
                wizard.message = _(
                    "You are about to add the value \"%(attribute_value)s\" to %(product_count)s products.",
                    attribute_value=wizard.attribute_value_id.name,
                    product_count=wizard.product_count,
                )
            elif wizard.mode == 'update_extra_price':
                wizard.message = _(
                    "You are about to update the extra price of %s products.",
                    wizard.product_count,
                )

    @api.depends('mode')
    def _compute_product_count(self):
        self.product_count = 0
        ProductTemplate = self.env['product.template']
        for wizard in self:
            if wizard.mode == 'add':
                wizard.product_count = ProductTemplate.search_count([
                    ('attribute_line_ids.attribute_id', '=', wizard.attribute_value_id.attribute_id.id),
                ])
            elif wizard.mode == 'update_extra_price':
                wizard.product_count = ProductTemplate.search_count([
                    ('attribute_line_ids.value_ids', '=', wizard.attribute_value_id.id),
                ])

    def action_confirm(self):
        self.ensure_one()
        if self.mode == 'add':
            self._add_value_to_existing_attribute_lines()
        elif self.mode == 'update_extra_price':
            self._update_extra_price_on_existing_products()

    def _add_value_to_existing_attribute_lines(self):
        ptals = self.env['product.template.attribute.line'].search([
            ('attribute_id', '=', self.attribute_value_id.attribute_id.id),
            # Make sure we do not impact products belonging to other companies
            ('product_tmpl_id.company_id', 'in', self.env.companies.ids + [False]),
        ])
        ptals.write({'value_ids': [Command.link(self.attribute_value_id.id)]})

    def _update_extra_price_on_existing_products(self):
        ptavs = self.env['product.template.attribute.value'].search([
            ('product_attribute_value_id', '=', self.attribute_value_id.id),
            # Make sure we do not impact products belonging to other companies
            ('product_tmpl_id.company_id', 'in', self.env.companies.ids + [False]),
        ])
        ptavs.write({'price_extra': self.attribute_value_id.default_extra_price})

```

## File: wizard\update_product_attribute_value_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="update_product_attribute_value_form" model="ir.ui.view">
        <field name="name">update.product.attribute.value.form</field>
        <field name="model">update.product.attribute.value</field>
        <field name="arch" type="xml">
            <form>
                <field name="product_count" invisible="1"/>
                <field name="mode" invisible="1"/>
                <field name="message"/>
                <footer>
                    <button string="Cancel" special="cancel"/>
                    <button
                        name="action_confirm"
                        string="Confirm"
                        class="btn-primary"
                        type="object"
                    />
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_label_layout
from . import update_product_attribute_value

```

