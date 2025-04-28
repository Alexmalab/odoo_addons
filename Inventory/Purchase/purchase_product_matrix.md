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
        'views/purchase_views.xml',
        'report/purchase_quotation_templates.xml',
        'report/purchase_order_templates.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'purchase_product_matrix/static/src/**/*',
        ],
        'web.assets_tests': [
            'purchase_product_matrix/static/tests/tours/**/*',
        ],
    },
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

## File: static\src\js\purchase_product_field.js

```javascript
/** @odoo-module **/

import Dialog from 'web.Dialog';
import { qweb } from "web.core";
import { registry } from '@web/core/registry';
import { Many2OneField } from '@web/views/fields/many2one/many2one_field';
import { formatMonetary } from "@web/views/fields/formatters";
import { useEffect } from '@odoo/owl';

const { markup } = owl;


export class PurchaseOrderLineProductField extends Many2OneField {

    setup() {
        super.setup();
        let isMounted = false;
        let isInternalUpdate = false;
        const super_update = this.update;
        this.update = (recordlist) => {
            isInternalUpdate = true;
            super_update(recordlist);
        };
        if (this.props.canQuickCreate) {
            this.quickCreate = (name) => {
                isInternalUpdate = true;
                return this.props.update([false, name]);
            };
        }
        useEffect(value => {
            if (!isMounted) {
                isMounted = true;
            } else if (value && isInternalUpdate) {
                // we don't want to trigger product update when update comes from an external sources,
                // such as an onchange, or the product configuration dialog itself
                this._onProductTemplateUpdate();
            }
            isInternalUpdate = false;
        }, () => [Array.isArray(this.value) && this.value[0]]);
    }

    get configurationButtonHelp() {
        return this.env._t("Edit Configuration");
    }
    get isConfigurableTemplate() {
        return this.props.record.data.is_configurable_product;
    }

    async _onProductTemplateUpdate() {
        const result = await this.orm.call(
            'product.template',
            'get_single_product_variant',
            [this.props.record.data.product_template_id[0]],
        );
        if(result && result.product_id) {
            if (this.props.record.data.product_id != result.product_id.id) {
                this.props.record.update({
                    // TODO right name get (same problem as configurator)
                    product_id: [result.product_id, 'whatever'],
                });
            }
        } else {
            this._openGridConfigurator(false);
        }
    }

    onEditConfiguration() {
        if (this.props.record.data.is_configurable_product) {
            this._openGridConfigurator(true);
        }
    }

    async _openGridConfigurator(edit) {
        const PurchaseOrderRecord = this.props.record.model.root;

        // fetch matrix information from server;
        await PurchaseOrderRecord.update({
            grid_product_tmpl_id: this.props.record.data.product_template_id,
        });

        let updatedLineAttributes = [];
        if (edit) {
            // provide attributes of edited line to automatically focus on matching cell in the matrix
            for (let ptnvav of this.props.record.data.product_no_variant_attribute_value_ids.records) {
                updatedLineAttributes.push(ptnvav.data.id);
            }
            for (let ptav of this.props.record.data.product_template_attribute_value_ids.records) {
                updatedLineAttributes.push(ptav.data.id);
            }
            updatedLineAttributes.sort((a, b) => { return a - b; });
        }

        this._openMatrixConfigurator(
            PurchaseOrderRecord.data.grid,
            this.props.record.data.product_template_id[0],
            updatedLineAttributes,
        );

        if (!edit) {
            // remove new line used to open the matrix
            PurchaseOrderRecord.data.order_line.removeRecord(this.props.record);
        }
    }

    _openMatrixConfigurator(jsonInfo, productTemplateId, editedCellAttributes) {
        const infos = JSON.parse(jsonInfo);
        const saleOrderRecord = this.props.record.model.root;
        const MatrixDialog = new Dialog(this, {
            title: this.env._t('Choose Product Variants'),
            size: 'extra-large', // adapt size depending on matrix size?
            $content: $(qweb.render(
                'product_matrix.matrix', {
                    header: infos.header,
                    rows: infos.matrix,
                    format({price, currency_id}) {
                        if (!price) { return ""; }
                        const sign = price < 0 ? '-' : '+';
                        const formatted = formatMonetary(
                            Math.abs(price),
                            {
                                currencyId: currency_id,
                            },
                        );
                        return markup(`${sign}&nbsp;${formatted}`);
                    }
                }
            )),
            buttons: [
                {text: this.env._t('Confirm'), classes: 'btn-primary', close: true, click: function (result) {
                    const $inputs = this.$('.o_matrix_input');
                    var matrixChanges = [];
                    _.each($inputs, function (matrixInput) {
                        if (matrixInput.value && matrixInput.value !== matrixInput.attributes.value.nodeValue) {
                            matrixChanges.push({
                                qty: parseFloat(matrixInput.value),
                                ptav_ids: matrixInput.attributes.ptav_ids.nodeValue.split(",").map(function (id) {
                                      return parseInt(id);
                                }),
                            });
                        }
                    });
                    if (matrixChanges.length > 0) {
                        // NB: server also removes current line opening the matrix
                        saleOrderRecord.update({
                            grid: JSON.stringify({changes: matrixChanges, product_template_id: productTemplateId}),
                            grid_update: true // to say that the changes to grid have to be applied to the SO.
                        });
                    }
                }},
                {text: this.env._t('Close'), close: true},
            ],
        }).open();

        MatrixDialog.opened(function () {
            MatrixDialog.$content.closest('.o_dialog_container').removeClass('d-none');
            if (editedCellAttributes.length > 0) {
                const str = editedCellAttributes.toString();
                MatrixDialog.$content.find('.o_matrix_input').filter((k, v) => v.attributes.ptav_ids.nodeValue === str)[0].focus();
            } else {
                MatrixDialog.$content.find('.o_matrix_input:first()').focus();
            }
        });
    }
}

PurchaseOrderLineProductField.template = "purchase.PurchaseProductField";

registry.category("fields").add("pol_product_many2one", PurchaseOrderLineProductField);

```

## File: static\src\js\purchase_product_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="purchase.PurchaseProductField" t-inherit="web.Many2OneField" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-if='hasExternalButton']" position="before">
            <t t-if="isConfigurableTemplate">
                <button
                    type="button"
                    class="btn btn-secondary fa fa-pencil"
                    tabindex="-1"
                    draggable="false"
                    t-att-aria-label="configurationButtonHelp"
                    t-att-data-tooltip="configurationButtonHelp"
                    t-on-click="onEditConfiguration"/>
            </t>
        </xpath>
    </t>

</templates>

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="purchase_order_form_matrix" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit.matrix</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//tree/field[@name='product_id']" position="attributes">
                <attribute name="string">Product Variant</attribute>
                <attribute name="optional">hide</attribute>
            </xpath>
            <xpath expr="//tree/field[@name='product_id']" position="after">
                <field name="product_template_id"
                    string="Product"
                    attrs="{
                        'readonly': [('state', 'in', ('purchase', 'to approve','done', 'cancel'))],
                        'required': [('display_type', '=', False)],
                    }"
                    context="{'partner_id': parent.partner_id}"
                    widget="pol_product_many2one"/>
                <field name="product_template_attribute_value_ids" invisible="1"/>
                <field name="product_no_variant_attribute_value_ids" invisible="1"/>
                <field name="is_configurable_product" invisible="1"/>
            </xpath>
            <field name="partner_id" position="after">
                <field name="grid" invisible="1"/>
                <field name="grid_product_tmpl_id" invisible="1"/>
                <field name="grid_update" invisible="1"/>
            </field>
            <group name="other_info" position="inside">
                <field name="report_grids" groups="base.group_no_one"/>
            </group>
        </field>
    </record>

</odoo>

```

