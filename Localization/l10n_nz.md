# Odoo Module: l10n_nz

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2015 Willow IT Pty Ltd (<http://www.willowit.com.au>).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2015 Willow IT Pty Ltd (<http://www.willowit.com.au>).

{
    'name': 'New Zealand - Accounting',
    'version': '1.1',
    'category': 'Localization',
    'description': """
New Zealand Accounting Module
=============================

New Zealand accounting basic charts and localizations.

Also:
    - activates a number of regional currencies.
    - sets up New Zealand taxes.
    """,
    'author': 'Richard deMeester - Willow IT',
    'website': 'http://www.willowit.com',
    'depends': ['account'],
    'data': [
             'data/l10n_nz_chart_data.xml',
             'data/account.account.template.csv',
             'data/account_chart_template_data.xml',
             'data/account.tax.group.csv',
             'data/account_tax_template_data.xml',
             'data/account_fiscal_position_tax_template_data.xml',
             'data/account_chart_template_configure_data.xml',
             'data/res_currency_data.xml',
             ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,chart_template_id:id,code,name,user_type_id:id,reconcile
nz_11110,l10n_nz.l10n_nz_chart_template,11110,Bank Account,account.data_account_type_liquidity,FALSE
nz_11130,l10n_nz.l10n_nz_chart_template,11130,Petty Cash,account.data_account_type_liquidity,FALSE
nz_11140,l10n_nz.l10n_nz_chart_template,11140,Cash Drawer,account.data_account_type_liquidity,FALSE
nz_11180,l10n_nz.l10n_nz_chart_template,11180,Undeposited Funds,account.data_account_type_current_assets,TRUE
nz_11190,l10n_nz.l10n_nz_chart_template,11190,Electronic Clearing Account,account.data_account_type_current_assets,TRUE
nz_11200,l10n_nz.l10n_nz_chart_template,11200,Trade Debtors,account.data_account_type_receivable,TRUE
nz_11210,l10n_nz.l10n_nz_chart_template,11210,Less Prov'n for Doubtful Debts,account.data_account_type_current_assets,FALSE
nz_11220,l10n_nz.l10n_nz_chart_template,11220,Trade Debtors (PoS),account.data_account_type_receivable,TRUE
nz_11310,l10n_nz.l10n_nz_chart_template,11310,Raw Materials,account.data_account_type_current_assets,FALSE
nz_11320,l10n_nz.l10n_nz_chart_template,11320,Finished Goods,account.data_account_type_current_assets,FALSE
nz_11330,l10n_nz.l10n_nz_chart_template,11330,Trading Stock on Hand,account.data_account_type_current_assets,FALSE
nz_12100,l10n_nz.l10n_nz_chart_template,12100,Deposits Paid,account.data_account_type_prepayments,FALSE
nz_12200,l10n_nz.l10n_nz_chart_template,12200,Prepaid Insurance,account.data_account_type_current_assets,FALSE
nz_13110,l10n_nz.l10n_nz_chart_template,13110,Manufacturing Plant at Cost,account.data_account_type_fixed_assets,FALSE
nz_13120,l10n_nz.l10n_nz_chart_template,13120,Manufac. Plant Accum Dep,account.data_account_type_fixed_assets,FALSE
nz_13130,l10n_nz.l10n_nz_chart_template,13130,Manufacturing Equipment Cost,account.data_account_type_fixed_assets,FALSE
nz_13140,l10n_nz.l10n_nz_chart_template,13140,Manufac. Equip Accum Dep,account.data_account_type_fixed_assets,FALSE
nz_13210,l10n_nz.l10n_nz_chart_template,13210,Furniture & Fixtures at Cost,account.data_account_type_fixed_assets,FALSE
nz_13220,l10n_nz.l10n_nz_chart_template,13220,Furniture & Fixtures Accum Dep,account.data_account_type_fixed_assets,FALSE
nz_13310,l10n_nz.l10n_nz_chart_template,13310,Office Equip at Cost,account.data_account_type_fixed_assets,FALSE
nz_13320,l10n_nz.l10n_nz_chart_template,13320,Office Equip Accum Dep,account.data_account_type_fixed_assets,FALSE
nz_13410,l10n_nz.l10n_nz_chart_template,13410,Motor Vehicles at Cost,account.data_account_type_fixed_assets,FALSE
nz_13420,l10n_nz.l10n_nz_chart_template,13420,Motor Vehicles Accum Dep,account.data_account_type_fixed_assets,FALSE
nz_21110,l10n_nz.l10n_nz_chart_template,21110,Credit Card,account.data_account_type_current_liabilities,FALSE
nz_21200,l10n_nz.l10n_nz_chart_template,21200,Trade Creditors,account.data_account_type_payable,TRUE
nz_21210,l10n_nz.l10n_nz_chart_template,21210,A/P Accrual - Inventory,account.data_account_type_current_liabilities,FALSE
nz_21310,l10n_nz.l10n_nz_chart_template,21310,GST Collected,account.data_account_type_current_liabilities,FALSE
nz_21330,l10n_nz.l10n_nz_chart_template,21330,GST Paid,account.data_account_type_current_assets,FALSE
nz_21350,l10n_nz.l10n_nz_chart_template,21350,Fuel Tax Credits Accrued,account.data_account_type_current_liabilities,FALSE
nz_21360,l10n_nz.l10n_nz_chart_template,21360,Import Duty Payable,account.data_account_type_current_liabilities,FALSE
nz_21370,l10n_nz.l10n_nz_chart_template,21370,Voluntary Withholdings Payable,account.data_account_type_current_liabilities,FALSE
nz_21380,l10n_nz.l10n_nz_chart_template,21380,Tax Withholdings Payable,account.data_account_type_current_liabilities,FALSE
nz_21410,l10n_nz.l10n_nz_chart_template,21410,Payroll Accruals Payable,account.data_account_type_current_liabilities,FALSE
nz_21420,l10n_nz.l10n_nz_chart_template,21420,PAYG Withholding Payable,account.data_account_type_current_liabilities,FALSE
nz_21600,l10n_nz.l10n_nz_chart_template,21600,Customer Deposits,account.data_account_type_current_liabilities,FALSE
nz_21700,l10n_nz.l10n_nz_chart_template,21700,Other Current Liabilities,account.data_account_type_current_liabilities,FALSE
nz_22100,l10n_nz.l10n_nz_chart_template,22100,Mortgages Payable,account.data_account_type_non_current_liabilities,FALSE
nz_22200,l10n_nz.l10n_nz_chart_template,22200,Notes Payable,account.data_account_type_non_current_liabilities,FALSE
nz_22300,l10n_nz.l10n_nz_chart_template,22300,Other Long Term Liabilities,account.data_account_type_non_current_liabilities,FALSE
nz_31100,l10n_nz.l10n_nz_chart_template,31100,Capital Investment,account.data_account_type_equity,FALSE
nz_31200,l10n_nz.l10n_nz_chart_template,31200,Capital Drawings,account.data_account_type_equity,FALSE
nz_38000,l10n_nz.l10n_nz_chart_template,38000,Retained Earnings,account.data_account_type_equity,FALSE
nz_39000,l10n_nz.l10n_nz_chart_template,39000,Current Year Earnings,account.data_unaffected_earnings,FALSE
nz_39999,l10n_nz.l10n_nz_chart_template,39999,Historical Balancing,account.data_account_type_equity,FALSE
nz_41110,l10n_nz.l10n_nz_chart_template,41110,Sales Product #1,account.data_account_type_revenue,FALSE
nz_41120,l10n_nz.l10n_nz_chart_template,41120,Sales Product #2,account.data_account_type_revenue,FALSE
nz_41130,l10n_nz.l10n_nz_chart_template,41130,Sales Product #3,account.data_account_type_revenue,FALSE
nz_42000,l10n_nz.l10n_nz_chart_template,42000,Wholesale Sales,account.data_account_type_revenue,FALSE
nz_43000,l10n_nz.l10n_nz_chart_template,43000,Consignment Sales,account.data_account_type_revenue,FALSE
nz_44000,l10n_nz.l10n_nz_chart_template,44000,Freight Income,account.data_account_type_revenue,FALSE
nz_45000,l10n_nz.l10n_nz_chart_template,45000,Late Fees Collected,account.data_account_type_other_income,FALSE
nz_46000,l10n_nz.l10n_nz_chart_template,46000,Miscellaneous Income,account.data_account_type_other_income,FALSE
nz_47000,l10n_nz.l10n_nz_chart_template,47000,Fuel Tax Credits,account.data_account_type_other_income,FALSE
nz_51110,l10n_nz.l10n_nz_chart_template,51110,Cost of Goods Sold #1,account.data_account_type_direct_costs,FALSE
nz_51120,l10n_nz.l10n_nz_chart_template,51120,Cost of Goods Sold # 2,account.data_account_type_direct_costs,FALSE
nz_51130,l10n_nz.l10n_nz_chart_template,51130,Cost of Goods Sold # 3,account.data_account_type_direct_costs,FALSE
nz_52000,l10n_nz.l10n_nz_chart_template,52000,Wholesale Cost of Sales,account.data_account_type_direct_costs,FALSE
nz_53000,l10n_nz.l10n_nz_chart_template,53000,Consignment Cost of Sales,account.data_account_type_direct_costs,FALSE
nz_54000,l10n_nz.l10n_nz_chart_template,54000,Wages for Production Labour,account.data_account_type_direct_costs,FALSE
nz_55000,l10n_nz.l10n_nz_chart_template,55000,Materials & Supplies,account.data_account_type_direct_costs,FALSE
nz_56000,l10n_nz.l10n_nz_chart_template,56000,Freight,account.data_account_type_direct_costs,FALSE
nz_57000,l10n_nz.l10n_nz_chart_template,57000,Other Costs,account.data_account_type_direct_costs,FALSE
nz_61000,l10n_nz.l10n_nz_chart_template,61000,Advertising,account.data_account_type_expenses,FALSE
nz_61200,l10n_nz.l10n_nz_chart_template,61200,Car & Truck Expenses,account.data_account_type_expenses,FALSE
nz_61300,l10n_nz.l10n_nz_chart_template,61300,Commissions Paid,account.data_account_type_expenses,FALSE
nz_61500,l10n_nz.l10n_nz_chart_template,61500,Depreciation Expense,account.data_account_type_depreciation,FALSE
nz_61610,l10n_nz.l10n_nz_chart_template,61610,Discounts Given,account.data_account_type_expenses,FALSE
nz_61620,l10n_nz.l10n_nz_chart_template,61620,Discounts Taken,account.data_account_type_expenses,FALSE
nz_61630,l10n_nz.l10n_nz_chart_template,61630,Exchange Rate Losses (Gains),account.data_account_type_expenses,FALSE
nz_61700,l10n_nz.l10n_nz_chart_template,61700,Freight Paid,account.data_account_type_expenses,FALSE
nz_61800,l10n_nz.l10n_nz_chart_template,61800,Insurance,account.data_account_type_expenses,FALSE
nz_61910,l10n_nz.l10n_nz_chart_template,61910,Overdraft Interest,account.data_account_type_expenses,FALSE
nz_61920,l10n_nz.l10n_nz_chart_template,61920,Mortgage Interest,account.data_account_type_expenses,FALSE
nz_61930,l10n_nz.l10n_nz_chart_template,61930,Other Interest,account.data_account_type_expenses,FALSE
nz_62000,l10n_nz.l10n_nz_chart_template,62000,Late Fees Paid,account.data_account_type_expenses,FALSE
nz_62110,l10n_nz.l10n_nz_chart_template,62110,Machinery & Equipment,account.data_account_type_expenses,FALSE
nz_62120,l10n_nz.l10n_nz_chart_template,62120,Other Business Property,account.data_account_type_expenses,FALSE
nz_62200,l10n_nz.l10n_nz_chart_template,62200,Legal & Professional Services,account.data_account_type_expenses,FALSE
nz_62300,l10n_nz.l10n_nz_chart_template,62300,Office Expenses,account.data_account_type_expenses,FALSE
nz_62410,l10n_nz.l10n_nz_chart_template,62410,Staff Amenities,account.data_account_type_expenses,FALSE
nz_62420,l10n_nz.l10n_nz_chart_template,62420,Superannuation,account.data_account_type_expenses,FALSE
nz_62430,l10n_nz.l10n_nz_chart_template,62430,Wages & Salaries,account.data_account_type_expenses,FALSE
nz_62440,l10n_nz.l10n_nz_chart_template,62440,Workers' Compensation,account.data_account_type_expenses,FALSE
nz_62450,l10n_nz.l10n_nz_chart_template,62450,Other Employer Expenses,account.data_account_type_expenses,FALSE
nz_62500,l10n_nz.l10n_nz_chart_template,62500,Repairs,account.data_account_type_expenses,FALSE
nz_62550,l10n_nz.l10n_nz_chart_template,62550,Shrinkage/Spoilage,account.data_account_type_expenses,FALSE
nz_62600,l10n_nz.l10n_nz_chart_template,62600,Supplies,account.data_account_type_expenses,FALSE
nz_62700,l10n_nz.l10n_nz_chart_template,62700,Taxes,account.data_account_type_expenses,FALSE
nz_62800,l10n_nz.l10n_nz_chart_template,62800,Telephone,account.data_account_type_expenses,FALSE
nz_62910,l10n_nz.l10n_nz_chart_template,62910,Gas,account.data_account_type_expenses,FALSE
nz_62920,l10n_nz.l10n_nz_chart_template,62920,Electricity,account.data_account_type_expenses,FALSE
nz_62930,l10n_nz.l10n_nz_chart_template,62930,Water,account.data_account_type_expenses,FALSE
nz_63110,l10n_nz.l10n_nz_chart_template,63110,Travel,account.data_account_type_expenses,FALSE
nz_63120,l10n_nz.l10n_nz_chart_template,63120,Meals & Entertainment,account.data_account_type_expenses,FALSE
nz_81000,l10n_nz.l10n_nz_chart_template,81000,Interest Income,account.data_account_type_other_income,FALSE
nz_91000,l10n_nz.l10n_nz_chart_template,91000,Interest Expense,account.data_account_type_expenses,FALSE
nz_92000,l10n_nz.l10n_nz_chart_template,92000,Income Tax Expense,account.data_account_type_expenses,FALSE
```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_0,TAX 0%
tax_group_gst_15,GST 15%
tax_group_15,TAX 15%
tax_group_100000000,GST 100000000%

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_nz.l10n_nz_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_nz_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="nz_11200"/>
        <field name="property_account_payable_id" ref="nz_21200"/>
        <field name="property_account_expense_categ_id" ref="nz_51110"/>
        <field name="property_account_income_categ_id" ref="nz_41110"/>
        <field name="property_stock_account_input_categ_id" ref="nz_21210"/>
        <field name="property_stock_account_output_categ_id" ref="nz_51110"/>
        <field name="property_stock_valuation_account_id" ref="nz_11330"/>
        <field name="expense_currency_exchange_account_id" ref="nz_61630"/>
        <field name="income_currency_exchange_account_id" ref="nz_61630"/>
        <field name="default_pos_receivable_account_id" ref="nz_11220" />
    </record>
</odoo>

```

## File: data\account_fiscal_position_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="fiscal_position_os_partner" model="account.fiscal.position.template">
            <field name="name">OS Partner</field>
            <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_sale1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="nz_tax_sale_15"/>
            <field name="tax_dest_id" ref="nz_tax_sale_0"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_sale2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="nz_tax_sale_inc_15"/>
            <field name="tax_dest_id" ref="nz_tax_sale_0"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_purch1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="nz_tax_purchase_15"/>
            <field name="tax_dest_id" ref="nz_tax_purchase_0"/>
        </record>

        <record id="fiscal_position_tax_template_os_partner_purch2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_os_partner"/>
            <field name="tax_src_id" ref="nz_tax_purchase_inc_15"/>
            <field name="tax_dest_id" ref="nz_tax_purchase_0"/>
        </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record id="nz_tax_sale_15" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">Sale (15%)</field>
        <field name="sequence">1</field>
        <field name="description">GST Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="price_include">FALSE</field>
        <field name="tax_group_id" ref="tax_group_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21310'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21310'),
            }),
        ]"/>
    </record>
    <record id="nz_tax_sale_inc_15" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">GST Inc Sale (15%)</field>
        <field name="sequence">2</field>
        <field name="description">GST Inclusive Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="price_include">TRUE</field>
        <field name="tax_group_id" ref="tax_group_gst_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21310'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21310'),
            }),
        ]"/>
    </record>
    <record id="nz_tax_sale_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">Zero/Export (0%) Sale</field>
        <field name="sequence">3</field>
        <field name="description">Zero Rated (Export) Sales</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include">FALSE</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="nz_tax_purchase_15" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">Purch (15%)</field>
        <field name="sequence">1</field>
        <field name="description">GST Purchases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="price_include">FALSE</field>
        <field name="tax_group_id" ref="tax_group_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21330'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21330'),
            }),
        ]"/>
    </record>
    <record id="nz_tax_purchase_inc_15" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">GST Inc Purch (15%)</field>
        <field name="sequence">2</field>
        <field name="description">GST Inclusive Purchases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="price_include">TRUE</field>
        <field name="tax_group_id" ref="tax_group_gst_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21330'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21330'),
            }),
        ]"/>
    </record>
    <record id="nz_tax_purchase_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">Zero/Import (0%) Purch</field>
        <field name="sequence">3</field>
        <field name="description">Zero Rated Purchases</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include">FALSE</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="nz_tax_purchase_taxable_import" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">Purch (Imports Taxable)</field>
        <field name="sequence">4</field>
        <field name="description">Purchase (Taxable Imports) - Tax Paid Separately</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="price_include">FALSE</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="nz_tax_purchase_gst_only" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_nz_chart_template"/>
        <field name="name">GST Only – Imports</field>
        <field name="sequence">5</field>
        <field name="description">GST Only on Imports</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">100000000000</field>
        <!--
          The tax percentage is so high because on imported goods we
          needed to link the tax line acknowledgment (not to be paid)
          on the customer invoice and what need to actually be
          paid from another invoice given by a clearance house
          (i.e. customs)
          For more info see the complete discussion below
          https://github.com/odoo/odoo/pull/48700#issuecomment-607586417
        -->
        <field name="price_include">TRUE</field>
        <field name="tax_group_id" ref="tax_group_100000000"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21330'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('nz_21330'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_nz_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<data>

    <record id="l10n_nz_chart_template" model="account.chart.template">
        <field name="name">New Zealand Tax and Account Chart Template (by Willow IT)</field>
        <field name="bank_account_code_prefix">1111</field>
        <field name="cash_account_code_prefix">1113</field>
        <field name="transfer_account_code_prefix">11170</field>
        <field name="code_digits">5</field>
        <field name="currency_id" ref="base.NZD"/>
    </record>

</data>
</odoo>

```

## File: data\res_currency_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.NZD" model="res.currency">
            <field name="active" eval="True"/>
        </record>

        <!-- Common trading partner currencies, and especially for AU -->

        <record id="base.AUD" model="res.currency">
            <field name="active" eval="True"/>
        </record>

        <record id="base.JPY" model="res.currency">
            <field name="active" eval="True"/>
        </record>

        <record id="base.INR" model="res.currency">
            <field name="active" eval="True"/>
        </record>

        <record id="base.GBP" model="res.currency">
            <field name="active" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink

```

