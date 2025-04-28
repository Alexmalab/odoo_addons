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
        'data/sale_expense_data.xml',
        'views/product_view.xml',
        'views/hr_expense_views.xml',
        'views/sale_order_views.xml',
        'views/hr_expense_sheet_views.xml',
    ],
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

## File: data\sale_expense_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="hr_expense.expense_product_meal" model="product.product">
            <field name="expense_policy">sales_price</field>
        </record>

        <record id="hr_expense.expense_product_mileage" model="product.product">
            <field name="invoice_policy">delivery</field>
            <field name="expense_policy">sales_price</field>
        </record>

        <record id="hr_expense.expense_product_travel_accommodation" model="product.product">
            <field name="invoice_policy">delivery</field>
            <field name="expense_policy">cost</field>
        </record>

        <record id="hr_expense.expense_product_communication" model="product.product">
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
        self.expense_sheet_id._sale_expense_reset_sol_quantities()
        res = super()._reverse_moves(default_values_list, cancel)
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

    @api.depends('product_id.expense_policy')
    def _compute_can_be_reinvoiced(self):
        for expense in self:
            expense.can_be_reinvoiced = expense.product_id.expense_policy in ['sales_price', 'cost']

    @api.depends('can_be_reinvoiced')
    def _compute_sale_order_id(self):
        for expense in self.filtered(lambda e: not e.can_be_reinvoiced):
            expense.sale_order_id = False

    def _compute_analytic_distribution(self):
        super(Expense, self)._compute_analytic_distribution()
        for expense in self.filtered('sale_order_id'):
            if expense.sale_order_id.sudo().analytic_account_id:
                expense.analytic_distribution = {expense.sale_order_id.sudo().analytic_account_id.id: 100}  # `sudo` required for normal employee without sale access rights

    @api.onchange('sale_order_id')
    def _onchange_sale_order_id(self):
        to_reset = self.filtered(lambda line: not self.env.is_protected(self._fields['analytic_distribution'], line))
        to_reset.invalidate_recordset(['analytic_distribution'])
        self.env.add_to_compute(self._fields['analytic_distribution'], to_reset)

    def _get_split_values(self):
        vals = super(Expense, self)._get_split_values()
        for split_value in vals:
            split_value['sale_order_id'] = self.sale_order_id.id
        return vals

    def action_move_create(self):
        """ When posting expense, if the AA is given, we will track cost in that
            If a SO is set, this means we want to reinvoice the expense. But to do so, we
            need the analytic entries to be generated, so a AA is required to reinvoice. So,
            we ensure the AA if a SO is given.
        """
        for expense in self.filtered(lambda expense: expense.sale_order_id and not expense.analytic_distribution):
            if not expense.sale_order_id.analytic_account_id:
                expense.sale_order_id._create_analytic_account()
            expense.write({
                'analytic_distribution': {expense.sale_order_id.analytic_account_id.id: 100}
            })
        return super(Expense, self).action_move_create()

```

## File: models\hr_expense_sheet.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from collections import Counter

from psycopg2.extras import execute_values

from odoo import fields, models, _


class HrExpenseSheet(models.Model):
    _inherit = "hr.expense.sheet"

    sale_order_count = fields.Integer(compute='_compute_sale_order_count')

    def _compute_sale_order_count(self):
        for sheet in self:
            sheet.sale_order_count = len(sheet.expense_line_ids.sale_order_id)

    def _get_sale_order_lines(self):
        """
            This method is used to try to find the sale order lines created by expense sheets.
            It is used to reset the quantities of the sale order lines when the expense sheet is reset.
            It uses several shared fields to try to find the sale order lines:
                - order_id
                - product_id
                - product_uom_qty
                - sale order line's price_unit (computed from the product_id, then rounded to the currency's rounding)
                - name
        """
        # Get the product account move lines created by an expense
        expensed_amls = self.account_move_id.line_ids.filtered(lambda aml: aml.expense_id.sale_order_id and aml.balance >= 0 and not aml.tax_line_id)
        if not expensed_amls:
            return self.env['sale.order.line']

        # Get the sale orders linked to the related expenses
        aml_to_so_map = expensed_amls._sale_determine_order()

        self.env['sale.order.line'].flush_model(['order_id', 'product_id', 'product_uom_qty', 'price_unit', 'name'])
        self.env['res.company'].flush_model(['currency_id'])
        self.env['res.currency'].flush_model(['rounding'])
        query = """
              WITH aml(key_id, key_count, order_id, product_id, product_uom_qty, price_unit, name) AS (VALUES %s)
            SELECT ARRAY_AGG(sol.id ORDER BY sol.id), aml.key_count
              FROM aml,
                   sale_order_line AS sol
              JOIN res_company AS company ON sol.company_id = company.id
              JOIN res_currency AS company_currency ON company.currency_id = company_currency.id
         LEFT JOIN res_currency AS currency ON sol.currency_id = currency.id
             WHERE sol.is_expense = TRUE
               AND sol.order_id = aml.order_id
               AND sol.product_id = aml.product_id
               AND sol.product_uom_qty = aml.product_uom_qty
               AND sol.name = aml.name
               AND ROUND(sol.price_unit::numeric, COALESCE(currency.rounding, company_currency.rounding)::int)
                   = ROUND(aml.price_unit::numeric, COALESCE(currency.rounding, company_currency.rounding)::int)
               GROUP BY aml.key_id, aml.key_count
        """

        # Get the keys used to fetch the corresponding sale order lines, and the number of times they are used
        # We need the occurrences count to filter out the sale order lines so that we keep exactly one per expense
        expense_keys_counter = Counter(expensed_amls.mapped(lambda aml: (
            aml.expense_id.sale_order_id.id,
            aml.product_id.id,
            aml.quantity,
            aml.currency_id.round(aml._sale_get_invoice_price(aml_to_so_map[aml.id])),
            aml.name,
        )))
        expensed_amls_keys_and_count = tuple(
            (key_id, key_count, *key) for key_id, (key, key_count) in enumerate(expense_keys_counter.items())
        )
        execute_values(
            self.env.cr._obj,
            query,
            expensed_amls_keys_and_count,
        )

        # Filters out the sale order lines so that we only keep one per expense
        sol_ids = []
        for all_sol_ids_per_key, expense_count_per_key in self.env.cr.fetchall():
            sol_ids += all_sol_ids_per_key[:expense_count_per_key]
        return self.env['sale.order.line'].browse(sol_ids)

    def _sale_expense_reset_sol_quantities(self):
        sale_order_lines = self._get_sale_order_lines()
        sale_order_lines.write({'qty_delivered': 0.0, 'product_uom_qty': 0.0})

    def reset_expense_sheets(self):
        super().reset_expense_sheets()
        self._sale_expense_reset_sol_quantities()
        return True

    def action_open_sale_orders(self):
        self.ensure_one()
        if self.sale_order_count == 1:
            return {
                'type': 'ir.actions.act_window',
                'res_model': 'sale.order',
                'views': [(self.env.ref("sale.view_order_form").id, 'form')],
                'view_mode': 'form',
                'target': 'current',
                'name': self.expense_line_ids.sale_order_id.display_name,
                'res_id': self.expense_line_ids.sale_order_id.id,
            }
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'sale.order',
            'views': [(self.env.ref('sale.view_order_tree').id, 'list'), (self.env.ref("sale.view_order_form").id, 'form')],
            'view_mode': 'list,form',
            'target': 'current',
            'name': _('Reinvoiced Sales Orders'),
            'domain': [('id', 'in', self.expense_line_ids.sale_order_id.ids)],
        }

```

## File: models\hr_expense_split.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class HrExpenseSplit(models.TransientModel):
    _inherit = "hr.expense.split"

    def default_get(self, fields):
        result = super(HrExpenseSplit, self).default_get(fields)
        if 'expense_id' in result:
            expense = self.env['hr.expense'].browse(result['expense_id'])
            result['sale_order_id'] = expense.sale_order_id
        return result

    sale_order_id = fields.Many2one('sale.order', string="Customer to Reinvoice", compute='_compute_sale_order_id', readonly=False, store=True, domain="[('state', '=', 'sale'), ('company_id', '=', company_id)]")
    can_be_reinvoiced = fields.Boolean("Can be reinvoiced", compute='_compute_can_be_reinvoiced')

    def _get_values(self):
        self.ensure_one()
        vals = super(HrExpenseSplit, self)._get_values()
        vals['sale_order_id'] = self.sale_order_id.id
        return vals

    @api.depends('product_id')
    def _compute_can_be_reinvoiced(self):
        for split in self:
            split.can_be_reinvoiced = split.product_id.expense_policy in ['sales_price', 'cost']

    @api.depends('can_be_reinvoiced')
    def _compute_sale_order_id(self):
        for split in self:
            split.sale_order_id = split.sale_order_id if split.can_be_reinvoiced else False

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    expense_policy_tooltip = fields.Char(compute='_compute_expense_policy_tooltip')

    @api.depends_context('lang')
    @api.depends('expense_policy')
    def _compute_expense_policy_tooltip(self):
        for product_template in self:
            if not product_template.can_be_expensed or not product_template.expense_policy:
                product_template.expense_policy_tooltip = False
            elif product_template.expense_policy == 'no':
                product_template.expense_policy_tooltip = _(
                    "Expenses of this category may not be added to a Sales Order."
                )
            elif product_template.expense_policy == 'cost':
                product_template.expense_policy_tooltip = _(
                    "Expenses will be added to the Sales Order at their actual cost when posted."
                )
            elif product_template.expense_policy == 'sales_price':
                product_template.expense_policy_tooltip = _(
                    "Expenses will be added to the Sales Order at their sales price (product price, pricelist, etc.) when posted."
                )

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

    @api.depends('can_be_expensed')
    def _compute_expense_policy(self):
        super()._compute_expense_policy()
        self.filtered(lambda t: not t.can_be_expensed).expense_policy = 'no'

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
        expense_data = self.env['hr.expense']._read_group([('sale_order_id', 'in', self.ids)], ['sale_order_id'], ['sale_order_id'])
        mapped_data = dict([(item['sale_order_id'][0], item['sale_order_id_count']) for item in expense_data])
        for sale_order in self:
            sale_order.expense_count = mapped_data.get(sale_order.id, 0)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_expense
from . import hr_expense_sheet
from . import hr_expense_split
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
/** @odoo-module alias=sale_expense.sale_order_many2one **/

import { Many2OneField } from '@web/views/fields/many2one/many2one_field';

import { registry } from "@web/core/registry";

export class OrderField extends Many2OneField {
    setup() {
        super.setup();
    }

    /**
     * @override
     */
    get Many2XAutocompleteProps() {
        // hide the search more option from the dropdown menu
       return {
           ...super.Many2XAutocompleteProps,
           noSearchMore: true,
       }
    }
}

registry.category('fields').add('sale_order_many2one', OrderField)
registry.add('sale_order_many2one', OrderField);

```

## File: views\hr_expense_sheet_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_expense_sheet_view_form" model="ir.ui.view">
        <field name="name">hr.expense.sheet.view.form.inherit.sale.expense</field>
        <field name="model">hr.expense.sheet</field>
        <field name="inherit_id" ref="hr_expense.view_hr_expense_sheet_form"/>
        <field name="arch" type="xml">
            <button name="action_get_expense_view" position="after">
                <button name="action_open_sale_orders"
                    class="oe_stat_button"
                    icon="fa-money"
                    type="object"
                    attrs="{'invisible': [('sale_order_count', '=', 0)]}">
                    <field name="sale_order_count" widget="statinfo" string="Sales Orders"/>
                </button>
            </button>
        </field>
    </record>
</odoo>

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
            <xpath expr="//field[@name='analytic_distribution']" position="before">
                <field name="sale_order_id" groups="!sales_team.group_sale_salesman,!account.group_account_manager"
                    attrs="{'invisible': [('can_be_reinvoiced', '=', False)], 'readonly': [('sheet_is_editable', '=', False)]}"
                    options="{'no_create_edit': True, 'no_create': True, 'no_open': True}"
                    context="{'sale_show_partner_name': True, 'sale_expense_all_order': True}"
                    widget="sale_order_many2one"/>
                <field name="sale_order_id" groups="sales_team.group_sale_salesman,!account.group_account_manager"
                    attrs="{'invisible': [('can_be_reinvoiced', '=', False)], 'readonly': [('sheet_is_editable', '=', False)]}"
                    options="{'no_create_edit': True, 'no_create': True}"
                    context="{'sale_show_partner_name': True, 'sale_expense_all_order': True}"
                    widget="many2one"/>
                <field name="sale_order_id" groups="account.group_account_manager"
                    widget="many2one"
                    attrs="{'invisible':[['can_be_reinvoiced','=',False]],'readonly':['|', ['state','in',['done']], ['sheet_is_editable', '=', False]]}"
                    options="{'no_create_edit': True, 'no_create': True, 'no_open': True}"
                    context="{'sale_show_partner_name': True, 'sale_expense_all_order': True}"
                    />
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

    <record id="hr_expense_split_view_inherit_sale_expense" model="ir.ui.view">
        <field name="name">hr.expense.split.view.inherit.sale.expense</field>
        <field name="model">hr.expense.split.wizard</field>
        <field name="inherit_id" ref="hr_expense.hr_expense_split"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='employee_id']" position="after">
                <field name="can_be_reinvoiced" invisible="1"/>
                <field name="sale_order_id" force_save="1" attrs="{'readonly': [('can_be_reinvoiced', '=', False)]}"/>
            </xpath>
        </field>
    </record>

    <record id="hr_expense_sheet_form_view_inherit_sale_expense" model="ir.ui.view">
        <field name="name">hr.expense.sheet.form.inherit.sale.expense</field>
        <field name="model">hr.expense.sheet</field>
        <field name="inherit_id" ref="hr_expense.view_hr_expense_sheet_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='expense_line_ids']/tree/field[@name='state']" position="after">
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
                    <field name="expense_policy" widget="radio"/>
                    <label for="expense_policy_tooltip" string=""/>
                    <div class="o_row">
                        <field name="expense_policy_tooltip" class="fst-italic text-muted"/>
                    </div>
                </group>
            </xpath>
            <xpath expr="//field[@name='standard_price']" position="before">
                <field name="list_price" attrs="{'invisible':[('expense_policy', '!=', 'sales_price')]}"/>
            </xpath>
            <xpath expr="//field[@name='supplier_taxes_id']" position="after">
                <field name="taxes_id" widget="many2many_tags" attrs="{'invisible':[('expense_policy', '=', 'no')]}"
                       options="{'no_quick_create': True}"/>
            </xpath>
        </field>
    </record>

    <record id="product_product_view_list_inherit_sale_expense" model="ir.ui.view">
        <field name="name">product.product.view.list.inherit.sale.expense</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="hr_expense.product_product_expense_categories_tree_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='supplier_taxes_id']" position="after">
                <field name="expense_policy" optional="show"/>
                <field name="taxes_id" optional="hide"/>
            </xpath>
        </field>
    </record>

    <record id="hr_expense.hr_expense_product" model="ir.actions.act_window">
        <field name="context">{"default_can_be_expensed": 1, 'default_detailed_type': 'service',
            'default_expense_policy' : 'cost'}</field>
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

