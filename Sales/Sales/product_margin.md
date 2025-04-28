# Odoo Module: product_margin

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import wizard
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Margins by Products',
    'category': 'Sales/Sales',
    'description': """
Adds a reporting menu in products that computes sales, purchases, margins and other interesting indicators based on invoices.
=============================================================================================================================

The wizard to launch the report has several options to help you get the data you need.
""",
    'depends': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'wizard/product_margin_view.xml',
        'views/product_product_views.xml'
    ],
    'license': 'LGPL-3',
}

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import itertools
import time

from odoo import api, fields, models
from odoo.tools.sql import SQL


class ProductProduct(models.Model):
    _inherit = "product.product"

    date_from = fields.Date(compute='_compute_product_margin_fields_values', string='Margin Date From')
    date_to = fields.Date(compute='_compute_product_margin_fields_values', string='Margin Date To')
    invoice_state = fields.Selection(compute='_compute_product_margin_fields_values',
        selection=[
            ('paid', 'Paid'),
            ('open_paid', 'Open and Paid'),
            ('draft_open_paid', 'Draft, Open and Paid')
        ], string='Invoice State', readonly=True)
    sale_avg_price = fields.Float(compute='_compute_product_margin_fields_values', string='Avg. Sale Unit Price',
        help="Avg. Price in Customer Invoices.")
    purchase_avg_price = fields.Float(compute='_compute_product_margin_fields_values', string='Avg. Purchase Unit Price',
        help="Avg. Price in Vendor Bills")
    sale_num_invoiced = fields.Float(compute='_compute_product_margin_fields_values', string='# Invoiced in Sale',
        help="Sum of Quantity in Customer Invoices")
    purchase_num_invoiced = fields.Float(compute='_compute_product_margin_fields_values', string='# Invoiced in Purchase',
        help="Sum of Quantity in Vendor Bills")
    sales_gap = fields.Float(compute='_compute_product_margin_fields_values', string='Sales Gap',
        help="Expected Sale - Turn Over")
    purchase_gap = fields.Float(compute='_compute_product_margin_fields_values', string='Purchase Gap',
        help="Normal Cost - Total Cost")
    turnover = fields.Float(compute='_compute_product_margin_fields_values', string='Turnover',
        help="Sum of Multiplication of Invoice price and quantity of Customer Invoices")
    total_cost = fields.Float(compute='_compute_product_margin_fields_values', string='Total Cost',
        help="Sum of Multiplication of Invoice price and quantity of Vendor Bills ")
    sale_expected = fields.Float(compute='_compute_product_margin_fields_values', string='Expected Sale',
        help="Sum of Multiplication of Sale Catalog price and quantity of Customer Invoices")
    normal_cost = fields.Float(compute='_compute_product_margin_fields_values', string='Normal Cost',
        help="Sum of Multiplication of Cost price and quantity of Vendor Bills")
    total_margin = fields.Float(compute='_compute_product_margin_fields_values', string='Total Margin',
        help="Turnover - Total cost")
    expected_margin = fields.Float(compute='_compute_product_margin_fields_values', string='Expected Margin',
        help="Expected Sale - Normal Cost")
    total_margin_rate = fields.Float(compute='_compute_product_margin_fields_values', string='Total Margin Rate(%)',
        help="Total margin * 100 / Turnover")
    expected_margin_rate = fields.Float(compute='_compute_product_margin_fields_values', string='Expected Margin (%)',
        help="Expected margin * 100 / Expected Sale")

    _SPECIAL_SUM_AGGREGATES = {f"{name}:sum" for name in (
        'turnover', 'sale_avg_price', 'sale_num_invoiced', 'purchase_num_invoiced',
        'sales_gap', 'purchase_gap', 'total_cost', 'sale_expected', 'normal_cost',
        'total_margin', 'expected_margin', 'total_margin_rate', 'expected_margin_rate',
    )}

    def _read_group_select(self, aggregate_spec, query):
        # the purpose of this override is to flag the aggregates above as such:
        # field._description_aggregator() should simply not fail
        if aggregate_spec in self._SPECIAL_SUM_AGGREGATES:
            return SQL()
        return super()._read_group_select(aggregate_spec, query)

    @api.model
    def _read_group(self, domain, groupby=(), aggregates=(), having=(), offset=0, limit=None, order=None):
        """
            Inherit _read_group to calculate the sum of the non-stored fields, as it is not automatically done anymore through the XML.
        """
        if self._SPECIAL_SUM_AGGREGATES.isdisjoint(aggregates):
            return super()._read_group(domain, groupby, aggregates, having, offset, limit, order)

        base_aggregates = [*(agg for agg in aggregates if agg not in self._SPECIAL_SUM_AGGREGATES), 'id:recordset']
        base_result = super()._read_group(domain, groupby, base_aggregates, having, offset, limit, order)

        # Force the compute of all records to bypass the limit compute batching (PREFETCH_MAX)
        all_records = self.browse().union(*(item[-1] for item in base_result))
        # This line will compute all fields having _compute_product_margin_fields_values
        # as compute method.
        self._fields['turnover'].compute_value(all_records)

        # base_result = [(a1, b1, records), (a2, b2, records), ...]
        result = []
        for *other, records in base_result:
            for index, spec in enumerate(itertools.chain(groupby, aggregates)):
                if spec in self._SPECIAL_SUM_AGGREGATES:
                    field_name = spec.split(':')[0]
                    other.insert(index, sum(records.mapped(field_name)))
            result.append(tuple(other))

        return result

    def _compute_product_margin_fields_values(self):
        date_from = self.env.context.get('date_from', time.strftime('%Y-01-01'))
        date_to = self.env.context.get('date_to', time.strftime('%Y-12-31'))
        invoice_state = self.env.context.get('invoice_state', 'open_paid')
        res = {
            product_id: {'date_from': date_from, 'date_to': date_to, 'invoice_state': invoice_state, 'turnover': 0.0,
                'sale_avg_price': 0.0, 'purchase_avg_price': 0.0, 'sale_num_invoiced': 0.0, 'purchase_num_invoiced': 0.0,
                'sales_gap': 0.0, 'purchase_gap': 0.0, 'total_cost': 0.0, 'sale_expected': 0.0, 'normal_cost': 0.0, 'total_margin': 0.0,
                'expected_margin': 0.0, 'total_margin_rate': 0.0, 'expected_margin_rate': 0.0}
            for product_id in self.ids
        }
        states = ()
        payment_states = ()
        if invoice_state == 'paid':
            states = ('posted',)
            payment_states = ('in_payment', 'paid', 'reversed')
        elif invoice_state == 'open_paid':
            states = ('posted',)
            payment_states = ('not_paid', 'in_payment', 'paid', 'reversed', 'partial')
        elif invoice_state == 'draft_open_paid':
            states = ('posted', 'draft')
            payment_states = ('not_paid', 'in_payment', 'paid', 'reversed', 'partial')
        if "force_company" in self.env.context:
            company_id = self.env.context['force_company']
        else:
            company_id = self.env.company.id
        self.env['account.move.line'].flush_model(['price_unit', 'quantity', 'balance', 'product_id', 'display_type'])
        self.env['account.move'].flush_model(['state', 'payment_state', 'move_type', 'invoice_date', 'company_id'])
        self.env['product.template'].flush_model(['list_price'])
        sqlstr = """
                WITH currency_rate AS MATERIALIZED ({})
                SELECT
                    l.product_id as product_id,
                    SUM(
                        l.price_unit / (CASE COALESCE(cr.rate, 0) WHEN 0 THEN 1.0 ELSE cr.rate END) *
                        l.quantity * (CASE WHEN i.move_type IN ('out_invoice', 'in_invoice') THEN 1 ELSE -1 END) * ((100 - l.discount) * 0.01)
                    ) / NULLIF(SUM(l.quantity * (CASE WHEN i.move_type IN ('out_invoice', 'in_invoice') THEN 1 ELSE -1 END)), 0) AS avg_unit_price,
                    SUM(l.quantity * (CASE WHEN i.move_type IN ('out_invoice', 'in_invoice') THEN 1 ELSE -1 END)) AS num_qty,
                    SUM(CASE WHEN i.move_type = 'out_invoice' THEN -l.balance WHEN i.move_type = 'in_invoice' THEN l.balance ELSE -ABS(l.balance) END) AS total,
                    SUM(l.quantity * pt.list_price * (CASE WHEN i.move_type IN ('out_invoice', 'in_invoice') THEN 1 ELSE -1 END)) AS sale_expected
                FROM account_move_line l
                LEFT JOIN account_move i ON (l.move_id = i.id)
                LEFT JOIN product_product product ON (product.id=l.product_id)
                LEFT JOIN product_template pt ON (pt.id = product.product_tmpl_id)
                left join currency_rate cr on
                (cr.currency_id = i.currency_id and
                 cr.company_id = i.company_id and
                 cr.date_start <= COALESCE(i.invoice_date, NOW()) and
                 (cr.date_end IS NULL OR cr.date_end > COALESCE(i.invoice_date, NOW())))
                WHERE l.product_id IN %s
                AND i.state IN %s
                AND i.payment_state IN %s
                AND i.move_type IN %s
                AND i.invoice_date BETWEEN %s AND  %s
                AND i.company_id = %s
                AND l.display_type = 'product'
                GROUP BY l.product_id
                """.format(self.env['res.currency']._select_companies_rates())
        invoice_types = ('out_invoice', 'out_refund')
        self.env.cr.execute(sqlstr, (tuple(self.ids), states, payment_states, invoice_types, date_from, date_to, company_id))
        for product_id, avg, qty, total, sale in self.env.cr.fetchall():
            res[product_id]['sale_avg_price'] = avg and avg or 0.0
            res[product_id]['sale_num_invoiced'] = qty and qty or 0.0
            res[product_id]['turnover'] = total and total or 0.0
            res[product_id]['sale_expected'] = sale and sale or 0.0
            res[product_id]['sales_gap'] = res[product_id]['sale_expected'] - res[product_id]['turnover']
            res[product_id]['total_margin'] = res[product_id]['turnover']
            res[product_id]['expected_margin'] = res[product_id]['sale_expected']
            res[product_id]['total_margin_rate'] = res[product_id]['turnover'] and res[product_id]['total_margin'] * 100 / res[product_id]['turnover'] or 0.0
            res[product_id]['expected_margin_rate'] = res[product_id]['sale_expected'] and res[product_id]['expected_margin'] * 100 / res[product_id]['sale_expected'] or 0.0

        ctx = self.env.context.copy()
        ctx['force_company'] = company_id
        invoice_types = ('in_invoice', 'in_refund')
        self.env.cr.execute(sqlstr, (tuple(self.ids), states, payment_states, invoice_types, date_from, date_to, company_id))
        for product_id, avg, qty, total, dummy in self.env.cr.fetchall():
            res[product_id]['purchase_avg_price'] = avg and avg or 0.0
            res[product_id]['purchase_num_invoiced'] = qty and qty or 0.0
            res[product_id]['total_cost'] = total and total or 0.0
            res[product_id]['total_margin'] = res[product_id].get('turnover', 0.0) - res[product_id]['total_cost']
            res[product_id]['total_margin_rate'] = res[product_id].get('turnover', 0.0) and res[product_id]['total_margin'] * 100 / res[product_id].get('turnover', 0.0) or 0.0
        for product in self:
            res[product.id]['normal_cost'] = product.standard_price * res[product.id]['purchase_num_invoiced']
            res[product.id]['purchase_gap'] = res[product.id]['normal_cost'] - res[product.id]['total_cost']
            res[product.id]['expected_margin'] = res[product.id].get('sale_expected', 0.0) - res[product.id]['normal_cost']
            res[product.id]['expected_margin_rate'] = res[product.id].get('sale_expected', 0.0) and res[product.id]['expected_margin'] * 100 / res[product.id].get('sale_expected', 0.0) or 0.0
            product.update(res[product.id])
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_product
```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_margin,access.product.margin,model_product_margin,account.group_account_user,1,1,1,0

```

## File: views\product_product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="view_product_margin_graph">
            <field name="name">product.margin.graph</field>
            <field name="model">product.product</field>
            <field name="priority" eval="50"/>
            <field name="arch" type="xml">
                <graph string="Product Margins" sample="1">
                    <field name="product_tmpl_id"/>
                    <field name="total_margin" type="measure" operator="+"/>
                </graph>
            </field>
    </record>

     <record id="view_product_margin_form" model="ir.ui.view">
            <field name="name">product.margin.form.inherit</field>
            <field name="model">product.product</field>
            <field name="priority">50</field>
            <field name="arch" type="xml">
                 <form string="Product Margins">
                    <group col="4">
                        <field name="name"/>
                        <field name="default_code"/>
                    </group>
                    <notebook>
                    <page string="Margins" name="margins">
                        <group string="Analysis Criteria">
                            <field name="date_from"/>
                            <field name="date_to"/>
                            <newline/>
                            <field name="invoice_state" />

                            <separator string="Sales" colspan="4"/>
                            <field name="sale_avg_price" string="Avg. Unit Price"/>
                            <field name="list_price" string="Catalog Price" readonly="1"/>
                            <field name="sale_num_invoiced" />
                            <field name="sales_gap" />
                            <field name="turnover" />
                            <field name="sale_expected" />

                            <separator string="Purchases" colspan="4"/>
                            <field name="purchase_avg_price" string="Avg. Unit Price"/>
                            <field name="standard_price" string="Standard Price"/>
                            <field name="purchase_num_invoiced" />
                            <field name="purchase_gap" />
                            <field name="total_cost" />
                            <field name="normal_cost" />

                            <separator string="Margins" colspan="4"/>
                            <field name="total_margin"/>
                            <field name="expected_margin"/>
                            <field name="total_margin_rate" widget="progressbar"/>
                            <field name="expected_margin_rate" widget="progressbar"/>
                         </group>
                    </page>
                    </notebook>
                </form>
            </field>
        </record>

        <record id="view_product_margin_tree" model="ir.ui.view">
            <field name="name">product.margin.list</field>
            <field name="model">product.product</field>
            <field name="priority" eval="50"/>
            <field name="arch" type="xml">
                <list string="Product Margins">
                    <field name="name"/>
                    <field name="default_code"/>
                    <field name="sale_avg_price"/>
                    <field name="sale_num_invoiced"/>
                    <field name="turnover" sum="Turnover"/>
                    <field name="sales_gap" sum="Sales Gap"/>
                    <field name="total_cost" sum="Total Cost"/>
                    <field name="purchase_num_invoiced" string="# Purchased"/>
                    <field name="total_margin" sum="Total Margin"/>
                    <field name="expected_margin"/>
                    <field name="total_margin_rate"  widget="progressbar"/>
                    <field name="expected_margin_rate" widget="progressbar"/>
                    <field name="categ_id" column_invisible="True"/>
                    <field name="uom_id" column_invisible="True"/>
                    <field name="type" column_invisible="True"/>
                    <field name="company_id" column_invisible="True"/>
            </list>
            </field>
        </record>

    <menuitem action="product_margin_act_window" id="menu_action_product_margin" name="Product Margins" sequence="20" parent="account.account_reports_management_menu"/>

</odoo>

```

## File: wizard\product_margin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import time

from odoo import api, fields, models, _


class ProductMargin(models.TransientModel):
    _name = 'product.margin'
    _description = 'Product Margin'

    from_date = fields.Date('From', default=time.strftime('%Y-01-01'))
    to_date = fields.Date('To', default=time.strftime('%Y-12-31'))
    invoice_state = fields.Selection([
        ('paid', 'Paid'),
        ('open_paid', 'Open and Paid'),
        ('draft_open_paid', 'Draft, Open and Paid'),
    ], 'Invoice State', required=True, default="open_paid")

    def action_open_window(self):
        self.ensure_one()
        context = dict(self.env.context, create=False, edit=False)

        def ref(xml_id):
            proxy = self.env['ir.model.data']
            return proxy._xmlid_lookup(xml_id)[1]

        search_view_id = ref('product.product_search_form_view')
        graph_view_id = ref('product_margin.view_product_margin_graph')
        form_view_id = ref('product_margin.view_product_margin_form')
        tree_view_id = ref('product_margin.view_product_margin_tree')

        context.update(invoice_state=self.invoice_state)

        if self.from_date:
            context.update(date_from=self.from_date)

        if self.to_date:
            context.update(date_to=self.to_date)

        views = [
            (tree_view_id, 'list'),
            (form_view_id, 'form'),
            (graph_view_id, 'graph')
        ]
        return {
            'name': _('Product Margins'),
            'context': context,
            "view_mode": 'list,form,graph',
            'res_model': 'product.product',
            'type': 'ir.actions.act_window',
            'views': views,
            'view_id': False,
            'search_view_id': [search_view_id],
        }

```

## File: wizard\product_margin_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
         <record id="product_margin_form_view" model="ir.ui.view">
            <field name="name">product.margin.form</field>
            <field name="model">product.margin</field>
            <field name="arch" type="xml">
                <form string="Properties categories">
                    <group col="4" string="General Information">
                        <field name="from_date" />
                        <field name="to_date" />
                        <field name="invoice_state" />
                    </group>
                    <footer>
                        <button name="action_open_window" string="Open Margins" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x" />
                    </footer>
                </form>
            </field>
        </record>

        <record id="product_margin_act_window" model="ir.actions.act_window">
            <field name="name">Product Margins</field>
            <field name="res_model">product.margin</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_margin

```

