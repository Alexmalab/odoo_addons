# Odoo Module: purchase_product_matrix

Category: Inventory/Purchase

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
    'name': "Purchase Matrix",
    'summary': """
       Add variants to your purchase orders through an Order Grid Entry.
    """,
    'description': """
        This module allows to fill Purchase Orders rapidly
        by choosing product variants quantity through a Grid Entry.
    """,
    'category': 'Inventory/Purchase',
    'version': '1.0',
    'depends': ['purchase', 'product_matrix'],
    'data': [
        'views/assets.xml',
        'views/purchase_views.xml',
        'report/purchase_quotation_templates.xml',
        'report/purchase_order_templates.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json
from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    report_grids = fields.Boolean(string="Print Variant Grids", default=True, help="If set, the matrix of configurable products will be shown on the report of this order.")

    """ Matrix loading and update: fields and methods :

    NOTE: The matrix functionality was done in python, server side, to avoid js
        restriction.  Indeed, the js framework only loads the x first lines displayed
        in the client, which means in case of big matrices and lots of po_lines,
        the js doesn't have access to the 41st and following lines.

        To force the loading, a 'hack' of the js framework would have been needed...
    """

    grid_product_tmpl_id = fields.Many2one('product.template', store=False, help="Technical field for product_matrix functionalities.")
    grid_update = fields.Boolean(default=False, store=False, help="Whether the grid field contains a new matrix to apply or not.")
    grid = fields.Char(store=False, help="Technical storage of grid. \nIf grid_update, will be loaded on the PO. \nIf not, represents the matrix to open.")

    @api.onchange('grid_product_tmpl_id')
    def _set_grid_up(self):
        if self.grid_product_tmpl_id:
            self.grid_update = False
            self.grid = json.dumps(self._get_matrix(self.grid_product_tmpl_id))

    def _must_delete_date_planned(self, field_name):
        return super()._must_delete_date_planned(field_name) or field_name == "grid"

    @api.onchange('grid')
    def _apply_grid(self):
        if self.grid and self.grid_update:
            grid = json.loads(self.grid)
            product_template = self.env['product.template'].browse(grid['product_template_id'])
            product_ids = set()
            dirty_cells = grid['changes']
            Attrib = self.env['product.template.attribute.value']
            default_po_line_vals = {}
            new_lines = []
            for cell in dirty_cells:
                combination = Attrib.browse(cell['ptav_ids'])
                no_variant_attribute_values = combination - combination._without_no_variant_attributes()

                # create or find product variant from combination
                product = product_template._create_product_variant(combination)
                # TODO replace the check on product_id by a first check on the ptavs and pnavs?
                # and only create/require variant after no line has been found ???
                order_lines = self.order_line.filtered(lambda line: (line._origin or line).product_id == product and (line._origin or line).product_no_variant_attribute_value_ids == no_variant_attribute_values)

                # if product variant already exist in order lines
                old_qty = sum(order_lines.mapped('product_qty'))
                qty = cell['qty']
                diff = qty - old_qty

                if not diff:
                    continue

                product_ids.add(product.id)

                if order_lines:
                    if qty == 0:
                        if self.state in ['draft', 'sent']:
                            # Remove lines if qty was set to 0 in matrix
                            # only if PO state = draft/sent
                            self.order_line -= order_lines
                        else:
                            order_lines.update({'product_qty': 0.0})
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
                            raise ValidationError(_("You cannot change the quantity of a product present in multiple purchase lines."))
                        else:
                            order_lines[0].product_qty = qty
                            order_lines[0]._onchange_quantity()
                            # If we want to support multiple lines edition:
                            # removal of other lines.
                            # For now, an error is raised instead
                            # if len(order_lines) > 1:
                            #     # Remove 1+ lines
                            #     self.order_line -= order_lines[1:]
                else:
                    if not default_po_line_vals:
                        OrderLine = self.env['purchase.order.line']
                        default_po_line_vals = OrderLine.default_get(OrderLine._fields.keys())
                    last_sequence = self.order_line[-1:].sequence
                    if last_sequence:
                        default_po_line_vals['sequence'] = last_sequence
                    new_lines.append((0, 0, dict(
                        default_po_line_vals,
                        product_id=product.id,
                        product_qty=qty,
                        product_no_variant_attribute_value_ids=no_variant_attribute_values.ids)
                    ))
            if product_ids:
                res = False
                if new_lines:
                    # Add new PO lines
                    self.update(dict(order_line=new_lines))

                # Recompute prices for new/modified lines:
                for line in self.order_line.filtered(lambda line: line.product_id.id in product_ids):
                    line._product_id_change()
                    line._onchange_quantity()
                    res = line.onchange_product_id_warning() or res
                return res

    def _get_matrix(self, product_template):
        def has_ptavs(line, sorted_attr_ids):
            ptav = line.product_template_attribute_value_ids.ids
            pnav = line.product_no_variant_attribute_value_ids.ids
            pav = pnav + ptav
            pav.sort()
            return pav == sorted_attr_ids
        matrix = product_template._get_template_matrix(
            company_id=self.company_id,
            currency_id=self.currency_id)
        if self.order_line:
            lines = matrix['matrix']
            order_lines = self.order_line.filtered(lambda line: line.product_template_id == product_template)
            for line in lines:
                for cell in line:
                    if not cell.get('name', False):
                        line = order_lines.filtered(lambda line: has_ptavs(line, cell['ptav_ids']))
                        if line:
                            cell.update({
                                'qty': sum(line.mapped('product_qty'))
                            })
        return matrix

    def get_report_matrixes(self):
        """Reporting method."""
        matrixes = []
        if self.report_grids:
            grid_configured_templates = self.order_line.filtered('is_configurable_product').product_template_id
            # TODO is configurable product and product_variant_count > 1
            # configurable products are only configured through the matrix in purchase, so no need to check product_add_mode.
            for template in grid_configured_templates:
                if len(self.order_line.filtered(lambda line: line.product_template_id == template)) > 1:
                    matrixes.append(self._get_matrix(template))
        return matrixes


class PurchaseOrderLine(models.Model):
    _inherit = "purchase.order.line"

    product_template_id = fields.Many2one('product.template', string='Product Template', related="product_id.product_tmpl_id", domain=[('purchase_ok', '=', True)])
    is_configurable_product = fields.Boolean('Is the product configurable?', related="product_template_id.has_configurable_attributes")
    product_template_attribute_value_ids = fields.Many2many(related='product_id.product_template_attribute_value_ids', readonly=True)
    product_no_variant_attribute_value_ids = fields.Many2many('product.template.attribute.value', string='Product attribute values that do not create variants', ondelete='restrict')

    def _get_product_purchase_description(self, product):
        name = super(PurchaseOrderLine, self)._get_product_purchase_description(product)
        for no_variant_attribute_value in self.product_no_variant_attribute_value_ids:
            name += "\n" + no_variant_attribute_value.attribute_id.name + ': ' + no_variant_attribute_value.name

        return name

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import purchase

```

## File: report\purchase_order_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="grid_purchaseorder_inherit" inherit_id="purchase.report_purchaseorder_document">
        <xpath expr="//div[@id='informations']" position="after">
            <t t-call="product_matrix.matrix">
                <t t-set="order" t-value="o"/>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: report\purchase_quotation_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="grid_report_purchaseorder_template_inherit" inherit_id="purchase.report_purchasequotation_document">
        <xpath expr="//table[hasclass('table')]" position="before">
            <t t-call="product_matrix.matrix">
                <t t-set="order" t-value="o"/>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: static\src\js\product_matrix_configurator.js

```javascript
odoo.define('purchase.product_matrix_configurator', function (require) {

var relationalFields = require('web.relational_fields');
var FieldsRegistry = require('web.field_registry');
var core = require('web.core');
var _t = core._t;

/**
 * The purchase.product_matrix_configurator widget is a widget extending FieldMany2One
 * It triggers the opening of the matrix edition when the product has multiple variants.
 *
 *
 * !!! WARNING !!!
 *
 * This widget is only designed for Purchase Order Lines.
 * !!! It should only be used on a product_template field !!!
 */
var MatrixConfiguratorWidget = relationalFields.FieldMany2One.extend({
    events: _.extend({}, relationalFields.FieldMany2One.prototype.events, {
        'click .o_edit_product_configuration': '_onEditProductConfiguration'
    }),

    /**
     * @override
     */
    _render: function () {
       this._super.apply(this, arguments);
       if (this.mode === 'edit' && this.value &&
       (this._isConfigurableProduct())) {
           this._addProductLinkButton();
           this._addConfigurationEditButton();
       } else if (this.mode === 'edit' && this.value) {
           this._addProductLinkButton();
       } else {
           this.$('.o_edit_product_configuration').hide();
       }
    },

    /**
    * Add button linking to product_id/product_template_id form.
    */
    _addProductLinkButton: function () {
       if (this.$('.o_external_button').length === 0) {
           var $productLinkButton = $('<button>', {
               type: 'button',
               class: 'fa fa-external-link btn btn-secondary o_external_button',
               tabindex: '-1',
               draggable: false,
               'aria-label': _t('External Link'),
               title: _t('External Link')
           });

           var $inputDropdown = this.$('.o_input_dropdown');
           $inputDropdown.after($productLinkButton);
       }
    },

    /**
    * If current product is configurable,
    * Show edit button (in Edit Mode) after the product/product_template
    */
    _addConfigurationEditButton: function () {
       var $inputDropdown = this.$('.o_input_dropdown');

       if ($inputDropdown.length !== 0 &&
           this.$('.o_edit_product_configuration').length === 0) {
           var $editConfigurationButton = $('<button>', {
               type: 'button',
               class: 'fa fa-pencil btn btn-secondary o_edit_product_configuration',
               tabindex: '-1',
               draggable: false,
               'aria-label': _t('Edit Configuration'),
               title: _t('Edit Configuration')
           });

           $inputDropdown.after($editConfigurationButton);
       }
    },

    /**
     * Hook to override with _onEditProductConfiguration
     * to know if edit pencil button has to be put next to the field
     *
     * @private
     */
    _isConfigurableProduct: function () {
        return this.recordData.is_configurable_product;
    },

    /**
     * Override catching changes on product_id or product_template_id.
     * Calls _onTemplateChange in case of product_template change.
     * Calls _onProductChange in case of product change.
     * Shouldn't be overridden by product configurators
     * or only to setup some data for further computation
     * before calling super.
     *
     * @override
     */
    reset: async function (record, ev) {
        await this._super(...arguments);
        if (ev && ev.target === this && ev.data.changes && ev.data.changes.product_template_id && record.data.product_template_id.data.id) {
            this._onTemplateChange(record.data.product_template_id.data.id, ev.data.dataPointID);
        }
    },

    /**
     * Hook for product_template based configurators
     * (product configurator, matrix, ...).
     *
     * @param {integer} productTemplateId
     * @param {String} dataPointID
     *
     * @private
     */
    _onTemplateChange: function (productTemplateId, dataPointId) {
        var self = this;
        this._rpc({
            model: 'product.template',
            method: 'get_single_product_variant',
            args: [
                productTemplateId
            ]
        }).then(function (result) {
            if (result.product_id) {
                self.trigger_up('field_changed', {
                    dataPointID: dataPointId,
                    changes: {
                        product_id: {id: result.product_id},
                    },
                });
            } else {
                self._openMatrix(productTemplateId, dataPointId, false);
            }
        });
    },

    /**
     * Hook for editing a configured line.
     * The button triggering this function is only shown in Edit mode,
     * when _isConfigurableProduct is True.
     *
     * @private
     */
    _onEditProductConfiguration: function () {
        if (this.recordData.is_configurable_product) {
            this._openMatrix(this.recordData.product_template_id.data.id, this.dataPointID, true);
        }
    },

    _openMatrix: function (productTemplateId, dataPointId, edit) {
        var attribs = edit ? this._getPTAVS() : [];
        this.trigger_up('open_matrix', {
            product_template_id: productTemplateId,
            model: 'purchase.order',
            dataPointId: dataPointId,
            edit: edit,
            editedCellAttributes: attribs,
            // used to focus the cell representing the line on which the pencil was clicked.
        });
    },

    /**
     * Returns the list of attribute ids (product.template.attribute.value)
     * from the current POLine.
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

FieldsRegistry.add('matrix_configurator', MatrixConfiguratorWidget);

return MatrixConfiguratorWidget;

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

  <template id="assets_backend_inherit_purchase" inherit_id="web.assets_backend" name="Purchase Grid assets">
      <xpath expr="script[last()]" position="after">
          <script type="text/javascript" src="/purchase_product_matrix/static/src/js/product_matrix_configurator.js"/>
      </xpath>
  </template>

  <template id="qunit_suite" name="purchase_product_matrix tests" inherit_id="web.qunit_suite_tests">
      <xpath expr="." position="inside">
          <script type="text/javascript" src="/purchase_product_matrix/static/tests/section_and_note_widget_tests.js"></script>
      </xpath>
  </template>

  <template id="assets_tests" name="Purchase Product Matrix Assets Tests" inherit_id="web.assets_tests">
      <xpath expr="." position="inside">
          <script type="text/javascript" src="/purchase_product_matrix/static/tests/tours/purchase_product_matrix_tour.js"/>
      </xpath>
  </template>
</odoo>

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

  <!-- TODO ensure barcode working with matrix (configurator mobile task ?) -->

  <record id="purchase_order_form_matrix" model="ir.ui.view">
    <field name="name">purchase.order.form.inherit.matrix</field>
    <field name="model">purchase.order</field>
    <field name="inherit_id" ref="purchase.purchase_order_form"/>
    <field name="arch" type="xml">
      <xpath expr="//tree/field[@name='product_id']" position="attributes">
          <attribute name="invisible">1</attribute>
      </xpath>
      <xpath expr="//tree/field[@name='product_id']" position="after">
          <field name="product_template_id"
            string="Product"
            attrs="{
                'readonly': [('state', 'in', ('purchase', 'to approve','done', 'cancel'))],
                'required': [('display_type', '=', False)],
            }"
            options="{'no_open': True}"
            context="{'partner_id': parent.partner_id}"
            widget="matrix_configurator"/>
          <field name="product_template_attribute_value_ids" invisible="1" />
          <field name="product_no_variant_attribute_value_ids" invisible="1" />
          <field name="is_configurable_product" invisible="1" />
      </xpath>
      <field name="partner_id" position="after">
          <field name="grid" invisible="1"/>
          <field name="grid_product_tmpl_id" invisible="1"/>
          <field name="grid_update" invisible="1"/>
      </field>
      <xpath expr="//group[@name='other_info']" position="inside">
          <field name="report_grids" groups="base.group_no_one"/>
      </xpath>
    </field>
  </record>

</odoo>

```

