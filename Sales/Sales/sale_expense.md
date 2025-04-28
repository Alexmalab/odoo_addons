# Odoo Module: sale_expense

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
    'name': 'Sales Expense',
    'version': '1.0',
    'category': 'Sales/Sales',
    'summary': 'Quotation, Sales Orders, Delivery & Invoicing Control',
    'description': """
Reinvoice Employee Expense
==========================

Create some products for which you can re-invoice the costs.
This module allow to reinvoice employee expense, by setting the SO directly on the expense.
""",
    'depends': ['sale_management', 'hr_expense'],
    'data': [
        'views/product_view.xml',
        'views/hr_expense_views.xml',
        'views/sale_order_views.xml',
    ],
    'demo': ['data/sale_expense_demo.xml'],
    'test': [],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'sale_expense/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'sale_expense/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\sale_expense_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="hr_expense.food_expense_product" model="product.product">
            <field name="expense_policy">sales_price</field>
        </record>

        <record id="hr_expense.mileage_expense_product" model="product.product">
            <field name="invoice_policy">delivery</field>
            <field name="expense_policy">sales_price</field>
        </record>

        <record id="hr_expense.accomodation_expense_product" model="product.product">
            <field name="invoice_policy">delivery</field>
            <field name="expense_policy">cost</field>
        </record>

        <record id="hr_expense.allowance_expense_product" model="product.product">
            <field name="expense_policy">cost</field>
        </record>

    </data>
</odoo>

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _sale_can_be_reinvoice(self):
        """ determine if the generated analytic line should be reinvoiced or not.
            For Expense flow, if the product has a 'reinvoice policy' and a Sales Order is set on the expense, then we will reinvoice the AAL
        """
        self.ensure_one()
        if self.expense_id:  # expense flow is different from vendor bill reinvoice flow
            return self.expense_id.product_id.expense_policy in ['sales_price', 'cost'] and self.expense_id.sale_order_id
        return super(AccountMoveLine, self)._sale_can_be_reinvoice()

    def _sale_determine_order(self):
        """ For move lines created from expense, we override the normal behavior.
            Note: if no SO but an AA is given on the expense, we will determine anyway the SO from the AA, using the same
            mecanism as in Vendor Bills.
        """
        mapping_from_invoice = super(AccountMoveLine, self)._sale_determine_order()

        mapping_from_expense = {}
        for move_line in self.filtered(lambda move_line: move_line.expense_id):
            mapping_from_expense[move_line.id] = move_line.expense_id.sale_order_id or None

        mapping_from_invoice.update(mapping_from_expense)
        return mapping_from_invoice

    def _sale_prepare_sale_line_values(self, order, price):
        # Add expense quantity to sales order line and update the sales order price because it will be charged to the customer in the end.
        self.ensure_one()
        res = super()._sale_prepare_sale_line_values(order, price)
        if self.expense_id:
            res.update({'product_uom_qty': self.expense_id.quantity})
        return res

    def _sale_create_reinvoice_sale_line(self):
        expensed_lines = self.filtered('expense_id')
        res = super(AccountMoveLine, self - expensed_lines)._sale_create_reinvoice_sale_line()
        res.update(super(AccountMoveLine, expensed_lines.with_context({'force_split_lines': True}))._sale_create_reinvoice_sale_line())
        return res


class AccountMove(models.Model):
    _inherit = 'account.move'

    expense_sheet_id = fields.One2many(
        comodel_name='hr.expense.sheet',
        inverse_name='account_move_id',
        string='Expense Sheet',
        readonly=True
    )

    def _reverse_moves(self, default_values_list=None, cancel=False):
        res = super()._reverse_moves(default_values_list, cancel)
        self.expense_sheet_id._sale_expense_reset_sol_quantities()
        return res

    def button_draft(self):
        res = super().button_draft()
        self.expense_sheet_id._sale_expense_reset_sol_quantities()
        return res

```

## File: models\hr_expense.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Expense(models.Model):
    _inherit = "hr.expense"

    sale_order_id = fields.Many2one('sale.order', compute='_compute_sale_order_id', store=True, string='Customer to Reinvoice', readonly=False, tracking=True,
        states={'done': [('readonly', True)], 'refused': [('readonly', True)]},
        # NOTE: only confirmed SO can be selected, but this domain in activated throught the name search with the `sale_expense_all_order`
        # context key. So, this domain is not the one applied.
        domain="[('state', '=', 'sale'), ('company_id', '=', company_id)]",
        help="If the category has an expense policy, it will be reinvoiced on this sales order")
    can_be_reinvoiced = fields.Boolean("Can be reinvoiced", compute='_compute_can_be_reinvoiced')
    analytic_account_id = fields.Many2one(compute='_compute_analytic_account_id', store=True, readonly=False)

    @api.depends('product_id.expense_policy')
    def _compute_can_be_reinvoiced(self):
        for expense in self:
            expense.can_be_reinvoiced = expense.product_id.expense_policy in ['sales_price', 'cost']

    @api.depends('can_be_reinvoiced')
    def _compute_sale_order_id(self):
        for expense in self.filtered(lambda e: not e.can_be_reinvoiced):
            expense.sale_order_id = False

    @api.depends('sale_order_id')
    def _compute_analytic_account_id(self):
        for expense in self.filtered('sale_order_id'):
            expense.analytic_account_id = expense.sale_order_id.sudo().analytic_account_id  # `sudo` required for normal employee without sale access rights

    def action_move_create(self):
        """ When posting expense, if the AA is given, we will track cost in that
            If a SO is set, this means we want to reinvoice the expense. But to do so, we
            need the analytic entries to be generated, so a AA is required to reinvoice. So,
            we ensure the AA if a SO is given.
        """
        for expense in self.filtered(lambda expense: expense.sale_order_id and not expense.analytic_account_id):
            if not expense.sale_order_id.analytic_account_id:
                expense.sale_order_id._create_analytic_account()
            expense.write({
                'analytic_account_id': expense.sale_order_id.analytic_account_id.id
            })
        return super(Expense, self).action_move_create()

```

## File: models\hr_expense_sheet.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class HrExpenseSheet(models.Model):
    _inherit = "hr.expense.sheet"

    def _get_sale_order_lines(self):
        """
            This method is used to try to find the sale order lines created by expense sheets.

            :return: sale.order.line
            :rtype: recordset
        """
        expensed_amls = self.account_move_id.line_ids.filtered(lambda aml: aml.expense_id.sale_order_id and aml.balance >= 0)
        if not expensed_amls:
            return self.env['sale.order.line']
        aml_to_so_map = expensed_amls._sale_determine_order()
        sale_order_ids = tuple(set(aml_to_so_map[aml.id].id for aml in expensed_amls))
        aml_sol_unit_price_map = dict(expensed_amls.mapped(lambda aml: (aml.id, aml._sale_get_invoice_price(aml_to_so_map[aml.id]))))
        product_ids = tuple(expensed_amls.product_id.ids)
        quantities = tuple(expensed_amls.mapped('quantity'))
        names = tuple(expensed_amls.mapped('name'))
        self.env['sale.order.line'].flush(['order_id', 'product_id', 'product_uom_qty', 'price_unit', 'name'])
        query = """
            SELECT 
                DISTINCT ON (sol.order_id, sol.product_id, sol.product_uom_qty, sol.price_unit, sol.name)
                sol.order_id, sol.product_id, sol.product_uom_qty, sol.price_unit, sol.name, sol.id
            FROM sale_order_line AS sol
            WHERE sol.is_expense = TRUE
                AND sol.order_id IN %s
                AND sol.product_id IN %s
                AND sol.product_uom_qty IN %s
                AND sol.price_unit IN %s
                AND sol.name IN %s
            ORDER BY sol.order_id, sol.product_id, sol.product_uom_qty, sol.price_unit, sol.name
        """
        self.env.cr.execute(query, (sale_order_ids, product_ids, quantities, tuple(set(aml_sol_unit_price_map.values())), names))
        potential_sols_map = {
            (row['order_id'], row['product_id'], row['product_uom_qty'], row['price_unit'], row['name']): row['id']
            for row in self.env.cr.dictfetchall()
        }
        expensed_amls_keys = set(expensed_amls.mapped(
            lambda aml: (aml.expense_id.sale_order_id.id, aml.product_id.id, aml.quantity, aml_sol_unit_price_map[aml.id], aml.name)
        ))
        return self.env['sale.order.line'].browse(sol_id for key, sol_id in potential_sols_map.items() if key in expensed_amls_keys)

    def _sale_expense_reset_sol_quantities(self):
        sale_order_lines = self._get_sale_order_lines()
        sale_order_lines.write({'qty_delivered': 0.0, 'product_uom_qty': 0.0})

    def reset_expense_sheets(self):
        super().reset_expense_sheets()
        self._sale_expense_reset_sol_quantities()
        return True

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    @api.depends('can_be_expensed')
    def _compute_visible_expense_policy(self):
        expense_products = self.filtered(lambda p: p.can_be_expensed)
        for product_template in self - expense_products:
            product_template.visible_expense_policy = False

        super(ProductTemplate, expense_products)._compute_visible_expense_policy()
        visibility = self.user_has_groups('hr_expense.group_hr_expense_user')
        for product_template in expense_products:
            if not product_template.visible_expense_policy:
                product_template.visible_expense_policy = visibility

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo import SUPERUSER_ID
from odoo.osv import expression


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    expense_ids = fields.One2many('hr.expense', 'sale_order_id', string='Expenses', domain=[('state', '=', 'done')], readonly=True, copy=False)
    expense_count = fields.Integer("# of Expenses", compute='_compute_expense_count', compute_sudo=True)

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        """ For expense, we want to show all sales order but only their name_get (no ir.rule applied), this is the only way to do it. """
        if self._context.get('sale_expense_all_order') and self.user_has_groups('sales_team.group_sale_salesman') and not self.user_has_groups('sales_team.group_sale_salesman_all_leads'):
            domain = expression.AND([args or [], ['&', ('state', '=', 'sale'), ('company_id', 'in', self.env.companies.ids)]])
            return super(SaleOrder, self.sudo())._name_search(name=name, args=domain, operator=operator, limit=limit, name_get_uid=SUPERUSER_ID)
        return super(SaleOrder, self)._name_search(name=name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)

    @api.depends('expense_ids')
    def _compute_expense_count(self):
        expense_data = self.env['hr.expense'].read_group([('sale_order_id', 'in', self.ids)], ['sale_order_id'], ['sale_order_id'])
        mapped_data = dict([(item['sale_order_id'][0], item['sale_order_id_count']) for item in expense_data])
        for sale_order in self:
            sale_order.expense_count = mapped_data.get(sale_order.id, 0)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_expense
from . import hr_expense_sheet
from . import sale_order
from . import product_template
from . import account_move

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sale_order_employee,sale.order.employee.expense,sale.model_sale_order,base.group_user,0,0,0,0
```

## File: security\sale_expense_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Required simply to set SO on expense before submitting. Used to avoid crash on name_get/name_search -->
    <record id="sale_order_rule_expense_user" model="ir.rule">
        <field name="name">Expense Employee can read confirmed SO</field>
        <field ref="sale.model_sale_order" name="model_id"/>
        <field name="domain_force">[('state', '=', 'sale')]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="active" eval="False"/> <!-- opw-2027005: this rules breaks sale "see own document" -->
    </record>

</odoo>

```

## File: static\src\js\sale_order_many2one.js

```javascript
odoo.define('sale_expense.sale_order_many2one', function (require) {
"use strict";

var FieldMany2One = require('web.relational_fields').FieldMany2One;
var FieldRegistry = require('web.field_registry');


var OrderField = FieldMany2One.extend({
    /**
     * hide the search more option from the dropdown menu
     * @override
     * @private
     * @returns {Object}
     */
    _manageSearchMore: function (values) {
        return values;
    }
});
FieldRegistry.add('sale_order_many2one', OrderField);
return OrderField;
});

```

## File: views\hr_expense_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_expense_form_view_inherit_sale_expense" model="ir.ui.view">
        <field name="name">hr.expense.form.inherit.sale.expense</field>
        <field name="model">hr.expense</field>
        <field name="inherit_id" ref="hr_expense.hr_expense_view_form"/>
        <field name="priority">30</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='analytic_account_id']" position="before">
                <field name="sale_order_id" attrs="{'invisible': [('can_be_reinvoiced', '=', False)], 'readonly': [('sheet_is_editable', '=', False)]}" options="{'no_create_edit': True, 'no_create': True, 'no_open': True}" context="{'sale_show_partner_name': True, 'sale_expense_all_order': True}" widget="sale_order_many2one"/>
                <field name="can_be_reinvoiced" invisible="1"/>
            </xpath>
        </field>
    </record>
    <record id="hr_expense_tree_view_inherit_sale_expense" model="ir.ui.view">
        <field name="name">hr.expense.tree.inherit.sale.expense</field>
        <field name="model">hr.expense</field>
        <field name="inherit_id" ref="hr_expense.view_expenses_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='reference']" position="after">
                <field name="sale_order_id" optional="hide" attrs="{'invisible': [('can_be_reinvoiced', '=', False)]}" options="{'no_create_edit': True, 'no_create': True, 'no_open': True}"  context="{'sale_show_partner_name': True, 'sale_expense_all_order': True}"/>
                <field name="can_be_reinvoiced" invisible="1" readonly="1"/>
            </xpath>
        </field>
    </record>
    <record id="hr_expense_form_view_inherit_account_manager" model="ir.ui.view">
        <field name="name">hr.expense.form.inherit.sale.expense</field>
        <field name="model">hr.expense</field>
        <field name="inherit_id" ref="sale_expense.hr_expense_form_view_inherit_sale_expense"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='sale_order_id']" position="attributes">
                <attribute name="attrs">{'invisible':[['can_be_reinvoiced','=',False]],'readonly':['|', ['state','in',['done']], ['sheet_is_editable', '=', False]]}</attribute>
                <attribute name="widget">many2one</attribute>
            </xpath>
        </field>
        <field name="groups_id" eval="[(6, 0, [ref('account.group_account_manager')])]"/>
    </record>
    <record id="hr_expense_form_view_inherit_saleman" model="ir.ui.view">
        <field name="name">hr.expense.form.inherit.saleman</field>
        <field name="model">hr.expense</field>
        <field name="inherit_id" ref="sale_expense.hr_expense_form_view_inherit_sale_expense"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='sale_order_id']" position="attributes">
                <attribute name="options">{'no_create_edit': True, 'no_create': True}</attribute>
                <attribute name="widget">many2one</attribute>
            </xpath>
        </field>
        <field name="groups_id" eval="[(6, 0, [ref('sales_team.group_sale_salesman')])]"/>
    </record>

    <record id="hr_expense_sheet_form_view_inherit_sale_expense" model="ir.ui.view">
        <field name="name">hr.expense.sheet.form.inherit.sale.expense</field>
        <field name="model">hr.expense.sheet</field>
        <field name="inherit_id" ref="hr_expense.view_hr_expense_sheet_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='expense_line_ids']/tree/field[@name='name']" position="after">
                <field name="sale_order_id" attrs="{'invisible': [('can_be_reinvoiced', '=', False)]}" optional="show" options="{'no_create_edit': True, 'no_create': True, 'no_open': True}" context="{'sale_show_partner_name': True, 'sale_expense_all_order': True}"/>
                <field name="can_be_reinvoiced" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="hr_expense_action_from_sale_order" model="ir.actions.act_window">
        <field name="name">Expenses</field>
        <field name="res_model">hr.expense</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('sale_order_id', '=', active_id)]</field>
        <field name="context">{'default_sale_order_id': active_id}</field>
    </record>

</odoo>

```

## File: views\product_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_product_view_form_inherit_sale_expense" model="ir.ui.view">
        <field name="name">product.template.expense</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="hr_expense.product_product_expense_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='product_details']" position="inside">
                <group string="Invoicing">
                    <field name="invoice_policy" widget="radio"/>
                    <field name="expense_policy" widget="radio"/>
                </group>
            </xpath>
            <xpath expr="//field[@name='standard_price']" position="before">
                <field name="list_price" attrs="{'invisible':[('expense_policy', '!=', 'sales_price')]}"/>
            </xpath>
            <xpath expr="//field[@name='taxes_id']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_expense.hr_expense_product" model="ir.actions.act_window">
        <field name="context">{"default_can_be_expensed": 1, 'default_detailed_type': 'service',
            'default_invoice_policy':'delivery', 'default_expense_policy' : 'cost'}</field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="sale_order_form_view_inherit" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.sale.expense</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <data>
               <xpath expr="//button[@name='action_view_invoice']" position="before">
                   <button type="action"
                       name="%(sale_expense.hr_expense_action_from_sale_order)d"
                       class="oe_stat_button"
                       icon="fa-money"
                       attrs="{'invisible': [('expense_count', '=', 0)]}">
                       <field name="expense_count" widget="statinfo" string="Expenses"/>
                   </button>
                </xpath>
            </data>
        </field>
    </record>

</odoo>

```

