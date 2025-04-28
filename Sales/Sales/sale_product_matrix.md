# Odoo Module: sale_product_matrix

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
    'name': "Sale Matrix",
    'summary': "Add variants to Sales Order through a grid entry.",
    'description': """
This module allows to fill Sales Order rapidly
by choosing product variants quantity through a Grid Entry.
    """,
    'category': 'Sales/Sales',
    'version': '1.0',
    'depends': ['sale', 'product_matrix', 'sale_product_configurator'],
    'data': [
        'views/product_template_views.xml',
        'views/sale_order_views.xml',
        'report/sale_report_templates.xml',
    ],
    'demo': [
        'data/product_matrix_demo.xml'
    ],
    'assets': {
        'web.assets_backend': [
            'sale_product_matrix/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\product_matrix_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="product_matrix.matrix_product_template_shirt" model="product.template">
        <field name="product_add_mode">matrix</field>
    </record>
</odoo>

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    product_add_mode = fields.Selection(
        selection=[
            ('configurator', "Product Configurator"),
            ('matrix', "Order Grid Entry"),
        ],
        string="Add product mode",
        default='configurator',
        help="Configurator: choose attribute values to add the matching product variant to the order."
             "\nGrid: add several variants at once from the grid of attribute values")

    def get_single_product_variant(self):
        res = super().get_single_product_variant()
        if self.has_configurable_attributes:
            res['mode'] = self.product_add_mode
        else:
            res['mode'] = 'configurator'
        return res

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json
from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    # if set, the matrix of the products configurable by matrix will be shown
    # on the report of the order.
    report_grids = fields.Boolean(string="Print Variant Grids", default=True)

    """ Matrix loading and update: fields and methods :

    NOTE: The matrix functionality was done in python, server side, to avoid js
        restriction.  Indeed, the js framework only loads the x first lines displayed
        in the client, which means in case of big matrices and lots of so_lines,
        the js doesn't have access to the 41nth and following lines.

        To force the loading, a 'hack' of the js framework would have been needed...
    """

    grid_product_tmpl_id = fields.Many2one(
        'product.template', store=False)
    # Whether the grid field contains a new matrix to apply or not
    grid_update = fields.Boolean(default=False, store=False)
    grid = fields.Char(
        "Matrix local storage", store=False,
        help="Technical local storage of grid. "
        "\nIf grid_update, will be loaded on the SO."
        "\nIf not, represents the matrix to open.")

    @api.onchange('grid_product_tmpl_id')
    def _set_grid_up(self):
        """Save locally the matrix of the given product.template, to be used by the matrix configurator."""
        if self.grid_product_tmpl_id:
            self.grid_update = False
            self.grid = json.dumps(self._get_matrix(self.grid_product_tmpl_id))

    @api.onchange('grid')
    def _apply_grid(self):
        """Apply the given list of changed matrix cells to the current SO."""
        if self.grid and self.grid_update:
            grid = json.loads(self.grid)
            product_template = self.env['product.template'].browse(grid['product_template_id'])
            dirty_cells = grid['changes']
            Attrib = self.env['product.template.attribute.value']
            default_so_line_vals = {}
            new_lines = []
            for cell in dirty_cells:
                combination = Attrib.browse(cell['ptav_ids'])
                no_variant_attribute_values = combination - combination._without_no_variant_attributes()

                # create or find product variant from combination
                product = product_template._create_product_variant(combination)
                order_lines = self.order_line.filtered(
                    lambda line: line.product_id.id == product.id
                    and line.product_no_variant_attribute_value_ids.ids == no_variant_attribute_values.ids
                )

                # if product variant already exist in order lines
                old_qty = sum(order_lines.mapped('product_uom_qty'))
                qty = cell['qty']
                diff = qty - old_qty

                if not diff:
                    continue

                # TODO keep qty check? cannot be 0 because we only get cell changes ...
                if order_lines:
                    if qty == 0:
                        if self.state in ['draft', 'sent']:
                            # Remove lines if qty was set to 0 in matrix
                            # only if SO state = draft/sent
                            self.order_line -= order_lines
                        else:
                            order_lines.update({'product_uom_qty': 0.0})
                    else:
                        """
                        When there are multiple lines for same product and its quantity was changed in the matrix,
                        An error is raised.

                        A 'good' strategy would be to:
                            * Sets the quantity of the first found line to the cell value
                            * Remove the other lines.

                        But this would remove all business logic linked to the other lines...
                        Therefore, it only raises an Error for now.
                        """
                        if len(order_lines) > 1:
                            raise ValidationError(_("You cannot change the quantity of a product present in multiple sale lines."))
                        else:
                            order_lines[0].product_uom_qty = qty
                            # If we want to support multiple lines edition:
                            # removal of other lines.
                            # For now, an error is raised instead
                            # if len(order_lines) > 1:
                            #     # Remove 1+ lines
                            #     self.order_line -= order_lines[1:]
                else:
                    if not default_so_line_vals:
                        OrderLine = self.env['sale.order.line']
                        default_so_line_vals = OrderLine.default_get(OrderLine._fields.keys())
                    last_sequence = self.order_line[-1:].sequence
                    if last_sequence:
                        default_so_line_vals['sequence'] = last_sequence
                    new_lines.append((0, 0, dict(
                        default_so_line_vals,
                        product_id=product.id,
                        product_uom_qty=qty,
                        product_no_variant_attribute_value_ids=no_variant_attribute_values.ids)
                    ))
            if new_lines:
                # Add new SO lines
                self.update(dict(order_line=new_lines))

    def _get_matrix(self, product_template):
        """Return the matrix of the given product, updated with current SOLines quantities.

        :param product.template product_template:
        :return: matrix to display
        :rtype dict:
        """
        def has_ptavs(line, sorted_attr_ids):
            # TODO instead of sorting on ids, use odoo-defined order for matrix ?
            ptav = line.product_template_attribute_value_ids.ids
            pnav = line.product_no_variant_attribute_value_ids.ids
            pav = pnav + ptav
            pav.sort()
            return pav == sorted_attr_ids
        matrix = product_template._get_template_matrix(
            company_id=self.company_id,
            currency_id=self.currency_id,
            display_extra_price=True)
        if self.order_line:
            lines = matrix['matrix']
            order_lines = self.order_line.filtered(lambda line: line.product_template_id == product_template)
            for line in lines:
                for cell in line:
                    if not cell.get('name', False):
                        line = order_lines.filtered(lambda line: has_ptavs(line, cell['ptav_ids']))
                        if line:
                            cell.update({
                                'qty': sum(line.mapped('product_uom_qty'))
                            })
        return matrix

    def get_report_matrixes(self):
        """Reporting method.

        :return: array of matrices to display in the report
        :rtype: list
        """
        matrixes = []
        if self.report_grids:
            grid_configured_templates = self.order_line.filtered('is_configurable_product').product_template_id.filtered(lambda ptmpl: ptmpl.product_add_mode == 'matrix')
            for template in grid_configured_templates:
                if len(self.order_line.filtered(lambda line: line.product_template_id == template)) > 1:
                    matrix = self._get_matrix(template)
                    matrix_data = []
                    for row in matrix['matrix']:
                        if any(column['qty'] != 0 for column in row[1:]):
                            matrix_data.append(row)
                    matrix['matrix'] = matrix_data
                    matrixes.append(matrix)
        return matrixes

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    product_add_mode = fields.Selection(related='product_template_id.product_add_mode', depends=['product_template_id'])

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_template
from . import sale_order
from . import sale_order_line

```

## File: report\sale_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="grid_report_saleorder_inherit" inherit_id="sale.report_saleorder_document">
        <xpath expr="//table[hasclass('o_main_table')]" position="before">
            <t t-call="product_matrix.matrix">
                <t t-set="order" t-value="doc"/>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: static\src\js\sale_product_field.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { SaleOrderLineProductField } from '@sale/js/sale_product_field';
import { ProductMatrixDialog } from "@product_matrix/js/product_matrix_dialog";
import { useService } from "@web/core/utils/hooks";
import "@sale_product_configurator/js/sale_product_field";


patch(SaleOrderLineProductField.prototype, {

    setup() {
        super.setup(...arguments);
        this.dialog = useService("dialog");
    },

    async _openGridConfigurator(edit=false) {
        const saleOrderRecord = this.props.record.model.root;

        // fetch matrix information from server;
        await saleOrderRecord.update({
            grid_product_tmpl_id: this.props.record.data.product_template_id,
        });

        let updatedLineAttributes = [];
        if (edit) {
            // provide attributes of edited line to automatically focus on matching cell in the matrix
            for (let ptnvav of this.props.record.data.product_no_variant_attribute_value_ids.records) {
                updatedLineAttributes.push(ptnvav.resId);
            }
            for (let ptav of this.props.record.data.product_template_attribute_value_ids.records) {
                updatedLineAttributes.push(ptav.resId);
            }
            updatedLineAttributes.sort((a, b) => { return a - b; });
        }

        this._openMatrixConfigurator(
            saleOrderRecord.data.grid,
            this.props.record.data.product_template_id[0],
            updatedLineAttributes,
        );

        if (!edit) {
            // remove new line used to open the matrix
            saleOrderRecord.data.order_line.delete(this.props.record);
        }
    },

    async _openProductConfigurator(edit=false) {
        if (edit && this.props.record.data.product_add_mode == 'matrix') {
            this._openGridConfigurator(true);
        } else {
            super._openProductConfigurator(...arguments);
        }
    },

    /**
     * Triggers Matrix Dialog opening
     *
     * @param {String} jsonInfo matrix dialog content
     * @param {integer} productTemplateId product.template id
     * @param {editedCellAttributes} list of product.template.attribute.value ids
     *  used to focus on the matrix cell representing the edited line.
     *
     * @private
    */
    _openMatrixConfigurator(jsonInfo, productTemplateId, editedCellAttributes) {
        const infos = JSON.parse(jsonInfo);
        this.dialog.add(ProductMatrixDialog, {
            header: infos.header,
            rows: infos.matrix,
            editedCellAttributes: editedCellAttributes.toString(),
            product_template_id: productTemplateId,
            record: this.props.record.model.root,
        });
    },
});

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_grid_view_form" model="ir.ui.view">
        <field name="name">product.template.form.inherit.sale.product.matrix</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_only_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='variants']" position="inside">
                <group name="product_mode" invisible="not has_configurable_attributes">
                    <group string="Sales Variant Selection">
                        <field name="has_configurable_attributes" invisible="1"/>
                        <field name="product_add_mode" widget="radio" nolabel="1" colspan="2"/>
                    </group>
                </group>
            </xpath>
        </field>
    </record>

    <record id="product_template_view_form" model="ir.ui.view">
        <field name="name">product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="sale_product_configurator.product_template_view_form"/>
        <field name="arch" type="xml">
            <field name="optional_product_ids" position="before">
                <field name="product_add_mode" widget="radio" invisible="1"/>
            </field>
            <field name="optional_product_ids" position="attributes">
                <attribute name="invisible">product_add_mode == 'grid'</attribute>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_order_form_with_variant_grid" model="ir.ui.view">
        <field name="name">sale.order.line.variant.grid</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <!-- Technical non stored fields for Order Grid Entry -->
                <field name="grid" invisible="1"/>
                <field name="grid_product_tmpl_id" invisible="1"/>
                <field name="grid_update" invisible="1"/>
            </field>
            <xpath expr="//tree/field[@name='product_template_id']" position="after">
                <field name="product_add_mode" column_invisible="True"/>
            </xpath>
            <xpath expr="//notebook//group[@name='sales_person']" position="inside">
                <field name="report_grids" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>
</odoo>

```

