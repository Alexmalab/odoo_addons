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
        'views/assets.xml',
        'views/product_template_views.xml',
        'views/sale_views.xml',
        'report/sale_report_templates.xml',
    ],
    'demo': [
        'data/product_matrix_demo.xml'
    ],
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
from odoo import api, models, fields


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    product_add_mode = fields.Selection([
        ('configurator', 'Product Configurator'),
        ('matrix', 'Order Grid Entry'),
    ], string='Add product mode', default='configurator', help="Configurator: choose attribute values to add the matching \
        product variant to the order.\nGrid: add several variants at once from the grid of attribute values")

    def get_single_product_variant(self):
        res = super(ProductTemplate, self).get_single_product_variant()
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

    report_grids = fields.Boolean(
        string="Print Variant Grids", default=True,
        help="If set, the matrix of the products configurable by matrix will be shown on the report of the order.")

    """ Matrix loading and update: fields and methods :

    NOTE: The matrix functionality was done in python, server side, to avoid js
        restriction.  Indeed, the js framework only loads the x first lines displayed
        in the client, which means in case of big matrices and lots of so_lines,
        the js doesn't have access to the 41nth and following lines.

        To force the loading, a 'hack' of the js framework would have been needed...
    """

    grid_product_tmpl_id = fields.Many2one(
        'product.template', store=False,
        help="Technical field for product_matrix functionalities.")
    grid_update = fields.Boolean(
        default=False, store=False,
        help="Whether the grid field contains a new matrix to apply or not.")
    grid = fields.Char(
        "Matrix local storage", store=False,
        help="Technical local storage of grid. \nIf grid_update, will be loaded on the SO. \nIf not, represents the matrix to open.")

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
            product_ids = set()
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

                product_ids.add(product.id)

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
            if product_ids:
                res = False
                if new_lines:
                    # Add new SO lines
                    self.update(dict(order_line=new_lines))

                # Recompute prices for new/modified lines
                for line in self.order_line.filtered(lambda line: line.product_id.id in product_ids):
                    res = line.product_id_change() or res
                    line._onchange_discount()
                    line._onchange_product_id_set_customer_lead()
                return res

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
                    # TODO do we really want the whole matrix even if there isn't a lot of lines ??
                    matrixes.append(self._get_matrix(template))
        return matrixes

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import sale_order
from . import product_template

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

## File: static\src\js\product_matrix_configurator.js

```javascript
odoo.define('sale_product_matrix.product_configurator', function (require) {
var ProductConfiguratorWidget = require('sale_product_configurator.product_configurator');

/**
 * Extension of the ProductConfiguratorWidget to support product configuration
 * as variant batches to add to the SO..
 * It opens when a configurable product_template is set
 * (multiple variants, or custom attributes)
 * and its configuration mode is matrix.
 *
 */
ProductConfiguratorWidget.include({

    /**
     * @override
     */
     _openConfigurator: function (result, productTemplateId, dataPointId) {
        var self = this;
        var mode = result.mode;
        this._super.apply(this, arguments).then(function (configuratorOpened) {
            if (!configuratorOpened && mode === 'matrix') {
                self._openGridConfigurator(productTemplateId, dataPointId);
                return Promise.resolve(true);
            }
            return Promise.resolve(configuratorOpened);
        });
    },

    _openGridConfigurator: function (productTemplateId, dataPointId, edit) {
        var attribs = edit ? this._getPTAVS() : [];
        this.trigger_up('open_matrix', {
            product_template_id: productTemplateId,
            model: 'sale.order',
            dataPointId: dataPointId,
            edit: edit,
            editedCellAttributes: attribs,
        });
    },

    _onEditProductConfiguration: function () {
        if (!this.recordData.is_configurable_product) {
            // if line should be edited by another configurator
            // or simply inline.
            this._super.apply(this, arguments);
            return;
        }
        var self = this;
        var productTemplateId = this.recordData.product_template_id.data.id;
        this._rpc({
            model: 'product.template',
            method: 'read',
            args: [productTemplateId, ['product_add_mode']],
        }).then(function (result) {
            if (result && result[0].product_add_mode === 'matrix') {
                self._openGridConfigurator(productTemplateId, self.dataPointID, true);
            } else {
                self.restoreProductTemplateId = self.recordData.product_template_id;
                // Call super only if product_add_mode different than matrix
                // to avoid product configurator opening (which is the default case).
                self._openProductConfigurator({
                        configuratorMode: 'edit',
                        default_product_template_id: self.recordData.product_template_id.data.id,
                        default_pricelist_id: self._getPricelistId(),
                        default_product_template_attribute_value_ids: self._convertFromMany2Many(
                            self.recordData.product_template_attribute_value_ids
                        ),
                        default_product_no_variant_attribute_value_ids: self._convertFromMany2Many(
                            self.recordData.product_no_variant_attribute_value_ids
                        ),
                        default_product_custom_attribute_value_ids: self._convertFromOne2Many(
                            self.recordData.product_custom_attribute_value_ids
                        ),
                        default_quantity: self.recordData.product_uom_qty
                    },
                    self.dataPointID
                );
            }
        });
    },

    /**
     * Returns the list of attribute ids (product.template.attribute.value)
     * from the current SOLine.
    */
    _getPTAVS: function () {
        var PTAVSIDS = [];
        _.each(this.recordData.product_no_variant_attribute_value_ids.res_ids, function (id) {
            PTAVSIDS.push(id);
        });
        _.each(this.recordData.product_template_attribute_value_ids.res_ids, function (id) {
            PTAVSIDS.push(id);
        });
        return PTAVSIDS.sort(function (a, b) {return a - b;});
    }

});

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend_inherit_sale" inherit_id="web.assets_backend" name="Sale Grid assets">
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/sale_product_matrix/static/src/js/product_matrix_configurator.js"/>
        </xpath>
    </template>

    <template id="qunit_suite" name="sale_product_matrix tests" inherit_id="web.qunit_suite_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/sale_product_matrix/static/tests/section_and_note_widget_tests.js"></script>
        </xpath>
    </template>

    <template id="assets_tests" name="Sale Product Matrix Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/sale_product_matrix/static/tests/tours/sale_product_matrix_tour.js"/>
        </xpath>
    </template>
</odoo>

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
                <group name="product_mode" attrs="{'invisible': [('has_configurable_attributes', '=', False)]}">
                    <group string="Sales Variant Selection">
                        <field name="has_configurable_attributes" invisible="1"/>
                        <field name="product_add_mode" widget="radio" nolabel="1"/>
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
            <xpath expr="//group[@name='options']" position="inside">
                <field name="product_add_mode" widget="radio" invisible="1"/>
                <!--
                  Field is only needed as invisible on product variants view
                -->
            </xpath>
            <xpath expr="//group[@name='options']" position="attributes">
                <!-- HIDE optional products many2many when product is added through the grid to SOLines -->
                <attribute name="attrs">{'invisible': [('product_add_mode', '=', 'grid')]}</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\sale_views.xml

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
          <xpath expr="//notebook//group[@name='sales_person']" position="inside">
              <field name="report_grids" groups="base.group_no_one"/>
          </xpath>
      </field>
  </record>
</odoo>

```

