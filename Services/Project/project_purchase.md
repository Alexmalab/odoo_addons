# Odoo Module: project_purchase

Category: Services/Project

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Project Purchase",
    'version': '1.0',
    'summary': "Monitor purchase in project",
    'category': 'Services/Project',
    'depends': ['purchase', 'project_account'],
    'demo': [
        'data/project_purchase_demo.xml',
    ],
    'data': [
        'views/project_project.xml',
        'views/purchase_order.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'project_purchase/static/src/product_catalog/kanban_record.js',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\catalog.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route
from odoo.addons.product.controllers.catalog import ProductCatalogController


class ProjectPurchaseCatalogController(ProductCatalogController):

    @route()
    def product_catalog_update_order_line_info(self, res_model, order_id, product_id, quantity=0, **kwargs):
        """ Override to update context with project_id.

        :param string res_model: The order model.
        :param int order_id: The order id.
        :param int product_id: The product, as a `product.product` id.
        :return: The unit price price of the product, based on the pricelist of the order and
                 the quantity selected.
        :rtype: float
        """
        if (project_id := kwargs.get('project_id')):
            request.update_context(project_id=project_id)
        return super().product_catalog_update_order_line_info(res_model, order_id, product_id, quantity, **kwargs)

```

## File: controllers\__init__.py

```python
from . import catalog

```

## File: data\project_purchase_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="purchase.order.line" name="write">
            <value model="purchase.order.line" search="[('product_id', 'in', [ref('product.product_delivery_01'), ref('product.product_product_27')]), ('order_id', '=', ref('purchase.purchase_order_1'))]"/>
            <value eval="{'analytic_distribution': {ref('analytic.analytic_our_super_product'): 100}}"/>
        </function>

        <record id="product_product_cement" model="product.product">
            <field name="name">Cement</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="standard_price">100.0</field>
            <field name="list_price">110.0</field>
            <field name="type">consu</field>
            <field name="weight">1.00</field>
            <field name="uom_id" ref="uom.product_uom_ton"/>
            <field name="uom_po_id" ref="uom.product_uom_ton"/>
        </record>

        <record id="product_product_sand" model="product.product">
            <field name="name">Sand</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="standard_price">80.0</field>
            <field name="list_price">70.0</field>
            <field name="type">consu</field>
            <field name="weight">1.00</field>
            <field name="uom_id" ref="uom.product_uom_ton"/>
            <field name="uom_po_id" ref="uom.product_uom_ton"/>
        </record>

        <record id="product_product_bricks" model="product.product">
            <field name="name">Bricks</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="standard_price">50.0</field>
            <field name="list_price">50.0</field>
            <field name="type">consu</field>
            <field name="weight">1.00</field>
            <field name="uom_id" ref="uom.product_uom_ton"/>
            <field name="uom_po_id" ref="uom.product_uom_ton"/>
        </record>

        <!-- Purchase order for project update -->
        <record id="purchase_order_for_home_construction" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="state">draft</field>
            <field name="date_order" eval="(datetime.today()).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_planned" eval="(datetime.today()).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="purchase_order_line_for_home_construction_1" model="purchase.order.line">
            <field name="order_id" ref="purchase_order_for_home_construction"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('project_purchase.product_product_cement').partner_ref"/>
            <field name="product_id" ref="project_purchase.product_product_cement"/>
            <field name="product_uom" ref="uom.product_uom_ton"/>
            <field name="price_unit">150</field>
            <field name="product_qty">5</field>
            <field name="analytic_distribution" eval="{ref('project.analytic_construction'): 100}"/>
            <field name="date_planned" eval="(datetime.today()).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="purchase_order_line_for_home_construction_2" model="purchase.order.line">
            <field name="order_id" ref="purchase_order_for_home_construction"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('project_purchase.product_product_sand').partner_ref"/>
            <field name="product_id" ref="project_purchase.product_product_sand"/>
            <field name="product_uom" ref="uom.product_uom_ton"/>
            <field name="price_unit">100</field>
            <field name="product_qty">10</field>
            <field name="analytic_distribution" eval="{ref('project.analytic_construction'): 100}"/>
            <field name="date_planned" eval="(datetime.today()).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="purchase_order_line_for_home_construction_3" model="purchase.order.line">
            <field name="order_id" ref="purchase_order_for_home_construction"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('project_purchase.product_product_bricks').partner_ref"/>
            <field name="product_id" ref="project_purchase.product_product_bricks"/>
            <field name="product_uom" ref="uom.product_uom_ton"/>
            <field name="price_unit">50</field>
            <field name="product_qty">15</field>
            <field name="analytic_distribution" eval="{ref('project.analytic_construction'): 100}"/>
            <field name="date_planned" eval="(datetime.today()).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
    </data>
</odoo>

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import fields, models, _
from odoo.osv import expression


class Project(models.Model):
    _inherit = "project.project"

    purchase_orders_count = fields.Integer('# Purchase Orders', compute='_compute_purchase_orders_count', groups='purchase.group_purchase_user', export_string_translation=False)

    def _compute_purchase_orders_count(self):
        purchase_orders_per_project = dict(
            self.env['purchase.order']._read_group(
                domain=[
                    ('project_id', 'in', self.ids),
                    ('order_line', '!=', False),
                ],
                groupby=['project_id'],
                aggregates=['id:array_agg'],
            )
        )
        purchase_orders_count_per_project_from_lines = dict(
            self.env['purchase.order.line']._read_group(
                domain=[
                    ('order_id', 'not in', [order_id for values in purchase_orders_per_project.values() for order_id in values]),
                    ('analytic_distribution', 'in', self.account_id.ids),
                ],
                groupby=['analytic_distribution'],
                aggregates=['__count'],
            )
        )

        projects_no_account = self.filtered(lambda project: not project.account_id)
        for project in projects_no_account:
            project.purchase_orders_count = len(purchase_orders_per_project.get(project, []))

        purchase_orders_per_project = {project.account_id.id: len(orders) for project, orders in purchase_orders_per_project.items()}
        for project in (self - projects_no_account):
            project.purchase_orders_count = purchase_orders_per_project.get(project.account_id.id, 0) + purchase_orders_count_per_project_from_lines.get(project.account_id.id, 0)

    # ----------------------------
    #  Actions
    # ----------------------------

    def action_open_project_purchase_orders(self):
        purchase_orders = self.env['purchase.order.line'].search([
            '|',
                ('analytic_distribution', 'in', self.account_id.ids),
                ('order_id.project_id', '=', self.id),
        ]).order_id
        action_window = {
            'name': self.env._('Purchase Orders'),
            'type': 'ir.actions.act_window',
            'res_model': 'purchase.order',
            'views': [
                [False, 'list'], [self.env.ref('purchase.purchase_order_view_kanban_without_dashboard').id, 'kanban'],
                [False, 'form'], [False, 'calendar'], [False, 'pivot'], [False, 'graph'], [False, 'activity'],
            ],
            'domain': [('id', 'in', purchase_orders.ids)],
            'context': {
                'default_project_id': self.id,
            },
            'help': "<p class='o_view_nocontent_smiling_face'>%s</p><p>%s</p>" % (
                _("No purchase order found. Let's create one."),
                _("Once you ordered your products from your supplier, confirm your request for quotation and it will turn "
                    "into a purchase order."),
            ),
        }
        if len(purchase_orders) == 1 and not self.env.context.get('from_embedded_action'):
            action_window['views'] = [[False, 'form']]
            action_window['res_id'] = purchase_orders.id
        return action_window

    def action_profitability_items(self, section_name, domain=None, res_id=False):
        if section_name == 'purchase_order':
            action = {
                'name': self.env._('Purchase Order Items'),
                'type': 'ir.actions.act_window',
                'res_model': 'purchase.order.line',
                'views': [[False, 'list'], [False, 'form']],
                'domain': domain,
                'context': {
                    'create': False,
                    'edit': False,
                },
            }
            if res_id:
                action['res_id'] = res_id
                if 'views' in action:
                    action['views'] = [
                        (view_id, view_type)
                        for view_id, view_type in action['views']
                        if view_type == 'form'
                    ] or [False, 'form']
                action['view_mode'] = 'form'
            return action
        return super().action_profitability_items(section_name, domain, res_id)

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.env.user.has_group('purchase.group_purchase_user'):
            self_sudo = self.sudo()
            buttons.append({
                'icon': 'credit-card',
                'text': self.env._('Purchase Orders'),
                'number': self_sudo.purchase_orders_count,
                'action_type': 'object',
                'action': 'action_open_project_purchase_orders',
                'show': self_sudo.purchase_orders_count > 0,
                'sequence': 36,
            })
        return buttons

    def _get_profitability_aal_domain(self):
        return expression.AND([
            super()._get_profitability_aal_domain(),
            ['|', ('move_line_id', '=', False), ('move_line_id.purchase_line_id', '=', False)],
        ])

    def _add_purchase_items(self, profitability_items, with_action=True):
        return False

    def _get_profitability_labels(self):
        labels = super()._get_profitability_labels()
        labels['purchase_order'] = self.env._('Purchase Orders')
        return labels

    def _get_profitability_sequence_per_invoice_type(self):
        sequence_per_invoice_type = super()._get_profitability_sequence_per_invoice_type()
        sequence_per_invoice_type['purchase_order'] = 10
        return sequence_per_invoice_type

    def _get_profitability_items(self, with_action=True):
        profitability_items = super()._get_profitability_items(with_action)
        if self.account_id:
            invoice_lines = self.env['account.move.line'].sudo().search_fetch([
                ('parent_state', 'in', ['draft', 'posted']),
                ('analytic_distribution', 'in', self.account_id.ids),
                ('purchase_line_id', '!=', False),
            ], ['parent_state', 'currency_id', 'price_subtotal', 'analytic_distribution'])
            purchase_order_line_invoice_line_ids = self._get_already_included_profitability_invoice_line_ids()
            with_action = with_action and (
                self.env.user.has_group('purchase.group_purchase_user')
                or self.env.user.has_group('account.group_account_invoice')
                or self.env.user.has_group('account.group_account_readonly')
            )
            if invoice_lines:
                amount_invoiced = amount_to_invoice = 0.0
                purchase_order_line_invoice_line_ids.extend(invoice_lines.ids)
                for line in invoice_lines:
                    price_subtotal = line.currency_id._convert(line.price_subtotal, self.currency_id, self.company_id)
                    # an analytic account can appear several time in an analytic distribution with different repartition percentage
                    analytic_contribution = sum(
                        percentage for ids, percentage in line.analytic_distribution.items()
                        if str(self.account_id.id) in ids.split(',')
                    ) / 100.
                    cost = price_subtotal * analytic_contribution * (-1 if line.is_refund else 1)
                    if line.parent_state == 'posted':
                        amount_invoiced -= cost
                    else:
                        amount_to_invoice -= cost
                costs = profitability_items['costs']
                section_id = 'purchase_order'
                purchase_order_costs = {'id': section_id, 'sequence': self._get_profitability_sequence_per_invoice_type()[section_id], 'billed': amount_invoiced, 'to_bill': amount_to_invoice}
                if with_action:
                    args = [section_id, [('id', 'in', invoice_lines.purchase_line_id.ids)]]
                    if len(invoice_lines.purchase_line_id) == 1:
                        args.append(invoice_lines.purchase_line_id.id)
                    action = {'name': 'action_profitability_items', 'type': 'object', 'args': json.dumps(args)}
                    purchase_order_costs['action'] = action
                costs['data'].append(purchase_order_costs)
                costs['total']['billed'] += amount_invoiced
                costs['total']['to_bill'] += amount_to_invoice
            domain = [
                ('move_id.move_type', 'in', ['in_invoice', 'in_refund']),
                ('parent_state', 'in', ['draft', 'posted']),
                ('id', 'not in', purchase_order_line_invoice_line_ids),
            ]
            self._get_costs_items_from_purchase(domain, profitability_items, with_action=with_action)
        return profitability_items

```

## File: models\purchase_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    project_id = fields.Many2one('project.project')

```

## File: models\purchase_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    @api.depends('product_id', 'order_id.partner_id', 'order_id.project_id')
    def _compute_analytic_distribution(self):
        super()._compute_analytic_distribution()
        ProjectProject = self.env['project.project']
        for line in self:
            if line.display_type or line.analytic_distribution:
                continue
            project_id = line._context.get('project_id')
            project = ProjectProject.browse(project_id) if project_id else line.order_id.project_id
            if project:
                line.analytic_distribution = project._get_analytic_distribution()

    @api.model_create_multi
    def create(self, vals_list):
        lines = super().create(vals_list)
        lines._recompute_recordset(fnames=['analytic_distribution'])
        return lines

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import project_project
from . import purchase_order_line
from . import purchase_order

```

## File: static\src\product_catalog\kanban_record.js

```javascript
/** @odoo-module **/
import { patch } from "@web/core/utils/patch";

import { ProductCatalogKanbanRecord } from "@product/product_catalog/kanban_record";

patch(ProductCatalogKanbanRecord.prototype, {
    _getUpdateQuantityAndGetPriceParams() {
        return {
            ...super._getUpdateQuantityAndGetPriceParams(),
            project_id: this.props.record.context.project_id,
        };
    },
});

```

## File: views\project_project.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_embedded_action_purchase_orders" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">70</field>
        <field name="name">Purchase Orders</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_open_project_purchase_orders</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('purchase.group_purchase_user'))]"/>
    </record>
</odoo>

```

## File: views\purchase_order.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_order_form_inherit_project_purchase" model="ir.ui.view">
        <field name="name">purchase.order.form.purchase.project</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <div name="reminder" position="after">
                <field name="project_id" groups="project.group_project_user"/>
            </div>
        </field>
    </record>
</odoo>

```

