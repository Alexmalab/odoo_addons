# Odoo Module: l10n_ie

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Ireland - Accounting',
    'version': '1.0',
    'category': 'Localization',
    'description': """
    This module is for all the Irish SMEs who would like to setup their accounting quickly. The module provides:

    - a Chart of Accounts customised to Ireland
    - VAT Rates and Structure""",

    'author': 'Target Integration',
    'website': 'http://www.targetintegration.com',
    'depends': ['account', 'base_iban', 'base_vat'],
    'data': [
        'data/account_chart_template.xml',
        'data/account.account.template.csv',
        'data/account.chart.template.csv',
        'data/account_tax_data.xml',
        'data/account_chart_template_configuration_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,user_type_id:id,reconcile,chart_template_id:id
l10n_ie_a10,10,Software,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a11,11,Software Depreciation,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a20,20,Patents & Trademarks,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a21,21,Patents & Trademarks Depreciation,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a30,30,Fixtures and fittings,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a31,31,Fixtures and fittings Depreciation,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a40,40,Land and buildings,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a41,41,Land and buildings Depreciation,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a50,50,Motor vehicles,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a51,51,Motor vehicles Depreciation,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a60,60,Office equipment (inc computer equipment),account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a61,61,Office equipment (inc computer equipment) Depreciation,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a70,70,Plant and machinery,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a71,71,Plant and machinery Depreciation,account.data_account_type_fixed_assets,FALSE,l10n_ie
l10n_ie_a1001,1001,Stock,account.data_account_type_current_assets,TRUE,l10n_ie
l10n_ie_a1002,1002,Work in Progress,account.data_account_type_current_assets,FALSE,l10n_ie
l10n_ie_a1003,1003,Finished Goods,account.data_account_type_current_assets,FALSE,l10n_ie
l10n_ie_a9999,9999,Debtors Control Account,account.data_account_type_receivable,TRUE,l10n_ie
l10n_ie_a9990,9990,Debtors Control Account (PoS),account.data_account_type_receivable,TRUE,l10n_ie
l10n_ie_a1101,1101,Sundry Debtors,account.data_account_type_receivable,TRUE,l10n_ie
l10n_ie_a1102,1102,Other Debtors,account.data_account_type_current_assets,FALSE,l10n_ie
l10n_ie_a1240,1240,Company Credit Card,account.data_account_type_current_assets,TRUE,l10n_ie
l10n_ie_a1103,1103,Prepayments,account.data_account_type_current_assets,FALSE,l10n_ie
l10n_ie_a9998,9998,Creditors Control Account,account.data_account_type_payable,TRUE,l10n_ie
l10n_ie_a2101,2101,Sundry Creditors,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2102,2102,Other Creditors,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2200,2200,Sales Tax Control Account,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2201,2201,Purchase Tax Control Account,account.data_account_type_current_assets,FALSE,l10n_ie
l10n_ie_a2202,2202,Revenue VAT Account,account.data_account_type_payable,TRUE,l10n_ie
l10n_ie_a2204,2204,Manual Adjustments ﾖ VAT,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2210,2210,PAYE,account.data_account_type_payable,TRUE,l10n_ie
l10n_ie_a2220,2220,Net Wages,account.data_account_type_payable,TRUE,l10n_ie
l10n_ie_a2230,2230,Pension Fund,account.data_account_type_payable,TRUE,l10n_ie
l10n_ie_a2150,2150,Bad debt provision,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2109,2109,Accruals,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2320,2320,Corporation Tax,account.data_account_type_payable,TRUE,l10n_ie
l10n_ie_a2300,2300,Loans,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2310,2310,Hire Purchase,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a2330,2330,Mortgages,account.data_account_type_current_liabilities,FALSE,l10n_ie
l10n_ie_a9997,9997,Called up share capital,account.data_account_type_equity,FALSE,l10n_ie
l10n_ie_a3010,3010,Share premium account,account.data_account_type_equity,FALSE,l10n_ie
l10n_ie_a3020,3020,Revaluation reserve,account.data_account_type_equity,FALSE,l10n_ie
l10n_ie_a3030,3030,Other reserves,account.data_account_type_equity,FALSE,l10n_ie
l10n_ie_a9996,9996,Sales category 1,account.data_account_type_revenue,FALSE,l10n_ie
l10n_ie_a4001,4001,Sales category 2,account.data_account_type_revenue,FALSE,l10n_ie
l10n_ie_a4002,4002,Sales category 3,account.data_account_type_revenue,FALSE,l10n_ie
l10n_ie_a4003,4003,Sales category 4,account.data_account_type_revenue,FALSE,l10n_ie
l10n_ie_a9995,9995,Cost of sales 1,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a5001,5001,Cost of sales 2,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a5002,5002,Cost of sales 3,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a5003,5003,Cost of sales 4,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a9994,9994,"Marketing, POS",account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a6001,6001,Exhibitions and events,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a6002,6002,PR,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a6010,6010,Distribution vehicles,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a6020,6020,Distribution salaries and wages,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a6030,6030,Shipping,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a9993,9993,Directors pension,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7001,7001,Directors remuneration,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7010,7010,Admin gross salaries,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7011,7011,Management gross salaries,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7012,7012,Employers NIC,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7020,7020,Subcontractors payments,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7610,7610,Consultancy,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7620,7620,Legal and professional charges,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7601,7601,Accounting,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7602,7602,Auditing,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7110,7110,"Light, heat and power",account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a9992,9992,Rent and rates,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7120,7120,"Repairs, renewals and maintenance",account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7300,7300,Car hire,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7301,7301,Car fuel,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7302,7302,Car maintenance,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7502,7502,Telephone,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7503,7503,Internet & hosting,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7504,7504,Mobiles,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7505,7505,Stationery,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7506,7506,Office consumables,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7507,7507,Postage and Carriage,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7508,7508,Books,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7509,7509,Network costs,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7510,7510,Software expenses,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7511,7511,Other computer costs,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7512,7512,Recruitment fees,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7513,7513,Other admin expenses,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7700,7700,Exchange gains/losses,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7710,7710,Other sundry expenses,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7850,7850,Bad debts,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a7910,7910,"Bank, credit card and other financial charges",account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a8000,8000,Intangible assets depn,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a8001,8001,Tangible assets depn,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a8200,8200,Donations,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a8300,8300,Entertaining,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a8400,8400,Insurance,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a8500,8500,Travel and subsistence,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a9000,9000,Profits/Losses on disposals of assets,account.data_account_type_revenue,FALSE,l10n_ie
l10n_ie_a4900,4900,Bank Interest received,account.data_account_type_revenue,FALSE,l10n_ie
l10n_ie_a4910,4910,Investment Interest received,account.data_account_type_revenue,FALSE,l10n_ie
l10n_ie_a7900,7900,Interest paid,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a8800,8800,Corporation tax expense,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a2240,2240,Universal Social Charge USC,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a2250,2250,PRSI Employee,account.data_account_type_expenses,FALSE,l10n_ie
l10n_ie_a2260,2260,PRSI Employer,account.data_account_type_expenses,FALSE,l10n_ie

```

## File: data\account.chart.template.csv

```csv
id,name,property_account_receivable_id:id,property_account_payable_id:id,property_account_expense_categ_id:id,property_account_income_categ_id:id,income_currency_exchange_account_id:id,expense_currency_exchange_account_id:id,default_pos_receivable_account_id:id,use_anglo_saxon
l10n_ie,Ireland - Chart of Accounts,l10n_ie_a9999,l10n_ie_a9998,l10n_ie_a9995,l10n_ie_a9996,l10n_ie_a7700,l10n_ie_a7700,l10n_ie_a9990,False

```

## File: data\account_chart_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <menuitem id="account_reports_ie_statements_menu" name="Irish Statements" parent="account.menu_finance_reports" sequence="3" groups="account.group_account_user"/>

    <!-- Chart template -->
    <record id="l10n_ie" model="account.chart.template">
        <field name="name">IE Tax and Account Chart Template</field>
        <field name="bank_account_code_prefix">1200</field>
        <field name="cash_account_code_prefix">1210</field>
        <field name="transfer_account_code_prefix">1220</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.EUR"/>
    </record>

</odoo>

```

## File: data\account_chart_template_configuration_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_ie.l10n_ie')]"/>
    </function>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_ie_tax_st0" model="account.tax.template">
        <field name="description">ST0</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Zero rated sales (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
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

    <record id="l10n_ie_tax_st1" model="account.tax.template">
        <field name="description">ST1</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Standard rate sales (13.5%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">13.5</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2200'),
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
                'account_id': ref('l10n_ie_a2200'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_st2" model="account.tax.template">
        <field name="description">ST2</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Exempt sales (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
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

    <record id="l10n_ie_tax_pt0" model="account.tax.template">
        <field name="description">PT0</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Zero rated purchases (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
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

    <record id="l10n_ie_tax_pt1" model="account.tax.template">
        <field name="description">PT1</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard rate purchases (13.5%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">13.5</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
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
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_pt2" model="account.tax.template">
        <field name="description">PT2</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Exempt purchases (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
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

    <record id="l10n_ie_tax_pt8" model="account.tax.template">
        <field name="description">PT8</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard rated purchases from EU (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">17.5</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
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
                'account_id': ref('l10n_ie_a2201'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_pt9" model="account.tax.template">
        <field name="description">PT9</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Lower rate purchases (9%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
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
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_st4" model="account.tax.template">
        <field name="description">ST4</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Sales to customers in EU (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
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

    <record id="l10n_ie_tax_pt7" model="account.tax.template">
        <field name="description">PT7</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Zero rated purchases from EU (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
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

    <record id="l10n_ie_tax_st11" model="account.tax.template">
        <field name="description">ST11</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Standard rate sales (23%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">23</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2200'),
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
                'account_id': ref('l10n_ie_a2200'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_pt11" model="account.tax.template">
        <field name="description">PT11</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard rate purchases (23%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">23</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
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
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_st9" model="account.tax.template">
        <field name="description">ST9</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Lower rate sale (9%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2202'),
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
                'account_id': ref('l10n_ie_a2202'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_pt6" model="account.tax.template">
        <field name="description">PT6</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Lower rate purchases (4.8%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">4.8</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
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
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
    </record>

    <record id="l10n_ie_tax_st6" model="account.tax.template">
        <field name="description">ST6</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Lower rate sale (4.8%) (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">4.8</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
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
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
    </record>

</odoo>

```

