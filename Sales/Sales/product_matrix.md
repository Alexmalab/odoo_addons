# Odoo Module: product_matrix

Category: Sales/Sales

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
    'name': "Product Matrix",
    'summary': """
       Technical module: Matrix Implementation
    """,
    'description': """
Please refer to Sale Matrix or Purchase Matrix for the use of this module.
    """,
    'category': 'Sales/Sales',
    'version': '1.0',
    'depends': ['account'],
    # Account dependency for section_and_note widget.
    'data': [
        'views/matrix_templates.xml',
    ],
    'demo': [
        'data/product_matrix_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'product_matrix/static/src/scss/product_matrix.scss',
            'product_matrix/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\product_matrix_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <!-- Size -->
    <record id="product_attribute_size" model="product.attribute">
        <field name="name">Size</field>
        <field name="sequence">19</field>
    </record>
    <record id="product_attribute_value_size_xs" model="product.attribute.value">
        <field name="name">XS</field>
        <field name="attribute_id" ref="product_attribute_size"/>
        <field name="sequence">1</field>
    </record>
    <record id="product_attribute_value_size_s" model="product.attribute.value">
        <field name="name">S</field>
        <field name="attribute_id" ref="product_attribute_size"/>
        <field name="sequence">2</field>
    </record>
    <record id="product_attribute_value_size_m" model="product.attribute.value">
        <field name="name">M</field>
        <field name="attribute_id" ref="product_attribute_size"/>
        <field name="sequence">3</field>
    </record>
    <record id="product_attribute_value_size_l" model="product.attribute.value">
        <field name="name">L</field>
        <field name="attribute_id" ref="product_attribute_size"/>
        <field name="sequence">4</field>
    </record>
    <record id="product_attribute_value_size_xl" model="product.attribute.value">
        <field name="name">XL</field>
        <field name="attribute_id" ref="product_attribute_size"/>
        <field name="sequence">5</field>
    </record>

    <!-- Color -->
    <record id="product_attribute_value_color_1" model="product.attribute.value">
        <field name="name">Blue</field>
        <field name="attribute_id" ref="product.product_attribute_2"/>
        <field name="sequence">3</field>
    </record>
    <record id="product_attribute_value_color_2" model="product.attribute.value">
        <field name="name">Pink</field>
        <field name="attribute_id" ref="product.product_attribute_2"/>
        <field name="sequence">4</field>
    </record>
    <record id="product_attribute_value_color_3" model="product.attribute.value">
        <field name="name">Yellow</field>
        <field name="attribute_id" ref="product.product_attribute_2"/>
        <field name="sequence">5</field>
    </record>
    <record id="product_attribute_value_color_4" model="product.attribute.value">
        <field name="name">Rainbow</field>
        <field name="attribute_id" ref="product.product_attribute_2"/>
        <field name="sequence">6</field>
    </record>

    <!-- Gender -->
    <record id="product_attribute_gender" model="product.attribute">
        <field name="name">Gender</field>
        <field name="sequence">21</field>
    </record>
    <record id="product_attribute_value_m" model="product.attribute.value">
        <field name="name">Men</field>
        <field name="attribute_id" ref="product_attribute_gender"/>
        <field name="sequence">1</field>
    </record>
    <record id="product_attribute_value_w" model="product.attribute.value">
        <field name="name">Women</field>
        <field name="attribute_id" ref="product_attribute_gender"/>
        <field name="sequence">2</field>
    </record>

    <record id="matrix_product_template_shirt" model="product.template">
        <field name="name">My Company Tshirt (GRID)</field>
        <field name="categ_id" ref="product.product_category_6"/>
        <field name="standard_price">7.0</field>
        <field name="list_price">15.0</field>
        <field name="detailed_type">consu</field>
        <field name="uom_id" ref="uom.product_uom_unit"/>
        <field name="uom_po_id" ref="uom.product_uom_unit"/>
        <field name="description_sale">Show your company love around you =).</field>
        <field name="image_1920" type="base64" file="product_matrix/static/img/matrix_mycompany_tshirt.jpeg"/>
    </record>

    <record id="product_template_attribute_line_size" model="product.template.attribute.line">
        <field name="product_tmpl_id" ref="matrix_product_template_shirt"/>
        <field name="attribute_id" ref="product_attribute_size"/>
        <field name="value_ids" eval="[
          (6, 0, [
          ref('product_attribute_value_size_xs'),
          ref('product_attribute_value_size_s'),
          ref('product_attribute_value_size_m'),
          ref('product_attribute_value_size_l'),
          ref('product_attribute_value_size_xl')])]"/>
    </record>
    <record id="product_template_attribute_line_gender" model="product.template.attribute.line">
        <field name="product_tmpl_id" ref="matrix_product_template_shirt"/>
        <field name="attribute_id" ref="product_attribute_gender"/>
        <field name="value_ids" eval="[(6, 0, [ref('product_attribute_value_m'), ref('product_attribute_value_w')])]"/>
    </record>
    <record id="product_template_attribute_line_color" model="product.template.attribute.line">
        <field name="product_tmpl_id" ref="matrix_product_template_shirt"/>
        <field name="attribute_id" ref="product.product_attribute_2"/>
        <field name="value_ids" eval="[(6, 0, [ref('product_attribute_value_color_1'), ref('product_attribute_value_color_2'), ref('product_attribute_value_color_3'), ref('product_attribute_value_color_4')])]"/>
    </record>
</odoo>

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import itertools

from odoo import models, fields


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    def _get_template_matrix(self, **kwargs):
        self.ensure_one()
        company_id = kwargs.get('company_id', None) or self.company_id or self.env.company
        currency_id = kwargs.get('currency_id', None) or self.currency_id
        display_extra = kwargs.get('display_extra_price', False)
        attribute_lines = self.valid_product_template_attribute_line_ids

        Attrib = self.env['product.template.attribute.value']
        first_line_attributes = attribute_lines[0].product_template_value_ids._only_active()
        attribute_ids_by_line = [line.product_template_value_ids._only_active().ids for line in attribute_lines]

        header = [{"name": self.display_name}] + [
            attr._grid_header_cell(
                fro_currency=self.currency_id,
                to_currency=currency_id,
                company=company_id,
                display_extra=display_extra
            ) for attr in first_line_attributes]

        result = [[]]
        for pool in attribute_ids_by_line:
            result = [x + [y] for y in pool for x in result]
        args = [iter(result)] * len(first_line_attributes)
        rows = itertools.zip_longest(*args)

        matrix = []
        for row in rows:
            row_attributes = Attrib.browse(row[0][1:])
            row_header_cell = row_attributes._grid_header_cell(
                fro_currency=self.currency_id,
                to_currency=currency_id,
                company=company_id,
                display_extra=display_extra)
            result = [row_header_cell]

            for cell in row:
                combination = Attrib.browse(cell)
                is_possible_combination = self._is_combination_possible(combination)
                cell.sort()
                result.append({
                    "ptav_ids": cell,
                    "qty": 0,
                    "is_possible_combination": is_possible_combination
                })
            matrix.append(result)

        return {
            "header": header,
            "matrix": matrix,
        }


class ProductTemplateAttributeValue(models.Model):
    _inherit = "product.template.attribute.value"

    def _grid_header_cell(self, fro_currency, to_currency, company, display_extra=True):
        """Generate a header matrix cell for 1 or multiple attributes.

        :param res.currency fro_currency:
        :param res.currency to_currency:
        :param res.company company:
        :param bool display_extra: whether extra prices should be displayed in the cell
            True by default, used to avoid showing extra prices on purchases.
        :returns: cell with name (and price if any price_extra is defined on self)
        :rtype: dict
        """
        header_cell = {
            'name': ' • '.join([attr.name for attr in self]) if self else " "
        }  # The " " is to avoid having 'Not available' if the template has only one attribute line.
        extra_price = sum(self.mapped('price_extra')) if display_extra else 0
        if extra_price:
            header_cell['currency_id'] = to_currency.id
            header_cell['price'] = fro_currency._convert(
                extra_price, to_currency, company, fields.Date.today())
        return header_cell

```

## File: models\__init__.py

```python
from . import product_template

```

## File: static\src\xml\product_matrix.xml

```xml
<template>
    <div t-name="product_matrix.matrix">
        <table class="o_matrix_input_table o_product_variant_matrix table table-sm table-striped table-bordered cursor-default">
            <thead>
                <tr>
                    <t t-foreach="header" t-as="column_header">
                        <th t-attf-class="o_matrix_title_header {{column_header_first?'text-start':'text-center'}}">
                            <span t-esc="column_header.name"/>
                            <t t-call="product_matrix.extra_price">
                                <t t-set="cell" t-value="column_header"/>
                            </t>
                        </th>
                    </t>
                </tr>
            </thead>
            <tbody>
                <tr t-foreach="rows" t-as="row">
                    <t t-foreach="row" t-as="cell">
                        <th t-if="cell.name" class="text-start">
                            <strong t-esc="cell.name"/>
                            <t t-call="product_matrix.extra_price"/>
                        </th>
                        <td t-else="">
                            <div t-if="cell.is_possible_combination" class="input-group">
                                <input type="number"
                                  class="o_matrix_input"
                                  t-att-ptav_ids="cell.ptav_ids"
                                  t-att-value="cell.qty"/>
                            </div>
                            <span t-else="" class="o_matrix_cell o_matrix_text_muted o_matrix_nocontent_container"> Not available </span>
                        </td>
                    </t>
                </tr>
            </tbody>
        </table>
    </div>
    <span t-name="product_matrix.extra_price" t-if="cell.price" class="badge rounded-pill text-bg-secondary">
            <!--
                price_extra is displayed as catalog price instead of
                price after pricelist because it is impossible to
                compute. Indeed, the pricelist rule might depend on the
                selected variant, so the price_extra will be different
                depending on the selected combination. The price of an
                attribute is therefore variable and it's not very
                accurate to display it.
                -->
            <span class="variant_price_extra" style="white-space: nowrap;">
                <t t-out="format(cell)"/>
            </span>
        </span>
</template>

```

## File: views\matrix_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="matrix">
          <t t-foreach="order.get_report_matrixes()" t-as="grid">
              <table class="o_view_grid o_product_variant_grid table table-sm table-striped table-bordered">
                  <thead>
                      <tr>
                          <th t-foreach="grid['header']" t-as="column_header" class="o_grid_title_header text-center">
                              <span t-esc="column_header['name']"/>
                              <t t-call="product_matrix.extra_price">
                                  <t t-set="price" t-value="column_header.get('price', False)"/>
                              </t>
                          </th>
                      </tr>
                  </thead>
                  <tbody>
                      <tr t-foreach="grid['matrix']" t-as="row">
                          <t t-foreach="row" t-as="cell">
                              <td t-if="cell.get('name', False)" class="text-start">
                                  <strong t-esc="cell['name']"/>
                                  <t t-call="product_matrix.extra_price">
                                      <t t-set="price" t-value="cell.get('price', False)"/>
                                  </t>
                              </td>
                              <td t-else="" class="text-end">
                                  <span t-esc="cell.get('qty', 0)" class="o_grid_cell_container"/>
                              </td>
                          </t>
                      </tr>
                  </tbody>
              </table>
          </t>
    </template>

    <template id="extra_price">
      <span t-if="price" class="badge rounded-pill text-bg-secondary">
              <!--
                  price_extra is displayed as catalog price instead of
                  price after pricelist because it is impossible to
                  compute. Indeed, the pricelist rule might depend on the
                  selected variant, so the price_extra will be different
                  depending on the selected combination. The price of an
                  attribute is therefore variable and it's not very
                  accurate to display it.
                  -->
              <span class="variant_price_extra" style="white-space: nowrap;">
                  <t t-out="price"/>
              </span>
          </span>
    </template>
</odoo>

```

