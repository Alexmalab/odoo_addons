# Odoo Module: l10n_ie

Category: Accounting/Localizations/Account Charts

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
    'category': 'Accounting/Localizations/Account Charts',
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
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,account_type,reconcile,chart_template_id:id
l10n_ie_a10,10,Software,asset_fixed,FALSE,l10n_ie
l10n_ie_a11,11,Software Depreciation,asset_fixed,FALSE,l10n_ie
l10n_ie_a20,20,Patents & Trademarks,asset_fixed,FALSE,l10n_ie
l10n_ie_a21,21,Patents & Trademarks Depreciation,asset_fixed,FALSE,l10n_ie
l10n_ie_a30,30,Fixtures and fittings,asset_fixed,FALSE,l10n_ie
l10n_ie_a31,31,Fixtures and fittings Depreciation,asset_fixed,FALSE,l10n_ie
l10n_ie_a40,40,Land and buildings,asset_fixed,FALSE,l10n_ie
l10n_ie_a41,41,Land and buildings Depreciation,asset_fixed,FALSE,l10n_ie
l10n_ie_a50,50,Motor vehicles,asset_fixed,FALSE,l10n_ie
l10n_ie_a51,51,Motor vehicles Depreciation,asset_fixed,FALSE,l10n_ie
l10n_ie_a60,60,Office equipment (inc computer equipment),asset_fixed,FALSE,l10n_ie
l10n_ie_a61,61,Office equipment (inc computer equipment) Depreciation,asset_fixed,FALSE,l10n_ie
l10n_ie_a70,70,Plant and machinery,asset_fixed,FALSE,l10n_ie
l10n_ie_a71,71,Plant and machinery Depreciation,asset_fixed,FALSE,l10n_ie
l10n_ie_a1001,1001,Stock,asset_current,TRUE,l10n_ie
l10n_ie_a1002,1002,Work in Progress,asset_current,FALSE,l10n_ie
l10n_ie_a1003,1003,Finished Goods,asset_current,FALSE,l10n_ie
l10n_ie_a9999,9999,Debtors Control Account,asset_receivable,TRUE,l10n_ie
l10n_ie_a9990,9990,Debtors Control Account (PoS),asset_receivable,TRUE,l10n_ie
l10n_ie_a1101,1101,Sundry Debtors,asset_receivable,TRUE,l10n_ie
l10n_ie_a1102,1102,Other Debtors,asset_current,FALSE,l10n_ie
l10n_ie_a1240,1240,Company Credit Card,asset_current,TRUE,l10n_ie
l10n_ie_a1103,1103,Prepayments,asset_current,FALSE,l10n_ie
l10n_ie_a9998,9998,Creditors Control Account,liability_payable,TRUE,l10n_ie
l10n_ie_a2101,2101,Sundry Creditors,liability_current,FALSE,l10n_ie
l10n_ie_a2102,2102,Other Creditors,liability_current,FALSE,l10n_ie
l10n_ie_a2200,2200,Sales Tax Control Account,liability_current,FALSE,l10n_ie
l10n_ie_a2201,2201,Purchase Tax Control Account,asset_current,FALSE,l10n_ie
l10n_ie_a2202,2202,Revenue VAT Account,liability_payable,TRUE,l10n_ie
l10n_ie_a2204,2204,Manual Adjustments ﾖ VAT,liability_current,FALSE,l10n_ie
l10n_ie_a2210,2210,PAYE,liability_payable,TRUE,l10n_ie
l10n_ie_a2220,2220,Net Wages,liability_payable,TRUE,l10n_ie
l10n_ie_a2230,2230,Pension Fund,liability_payable,TRUE,l10n_ie
l10n_ie_a2150,2150,Bad debt provision,liability_current,FALSE,l10n_ie
l10n_ie_a2109,2109,Accruals,liability_current,FALSE,l10n_ie
l10n_ie_a2320,2320,Corporation Tax,liability_payable,TRUE,l10n_ie
l10n_ie_a2300,2300,Loans,liability_current,FALSE,l10n_ie
l10n_ie_a2310,2310,Hire Purchase,liability_current,FALSE,l10n_ie
l10n_ie_a2330,2330,Mortgages,liability_current,FALSE,l10n_ie
l10n_ie_a9997,9997,Called up share capital,equity,FALSE,l10n_ie
l10n_ie_a3010,3010,Share premium account,equity,FALSE,l10n_ie
l10n_ie_a3020,3020,Revaluation reserve,equity,FALSE,l10n_ie
l10n_ie_a3030,3030,Other reserves,equity,FALSE,l10n_ie
l10n_ie_a9996,9996,Sales category 1,income,FALSE,l10n_ie
l10n_ie_a4001,4001,Sales category 2,income,FALSE,l10n_ie
l10n_ie_a4002,4002,Sales category 3,income,FALSE,l10n_ie
l10n_ie_a4003,4003,Sales category 4,income,FALSE,l10n_ie
l10n_ie_a9995,9995,Cost of sales 1,expense,FALSE,l10n_ie
l10n_ie_a5001,5001,Cost of sales 2,expense,FALSE,l10n_ie
l10n_ie_a5002,5002,Cost of sales 3,expense,FALSE,l10n_ie
l10n_ie_a5003,5003,Cost of sales 4,expense,FALSE,l10n_ie
l10n_ie_a9994,9994,"Marketing, POS",expense,FALSE,l10n_ie
l10n_ie_a6001,6001,Exhibitions and events,expense,FALSE,l10n_ie
l10n_ie_a6002,6002,PR,expense,FALSE,l10n_ie
l10n_ie_a6010,6010,Distribution vehicles,expense,FALSE,l10n_ie
l10n_ie_a6020,6020,Distribution salaries and wages,expense,FALSE,l10n_ie
l10n_ie_a6030,6030,Shipping,expense,FALSE,l10n_ie
l10n_ie_a9993,9993,Directors pension,expense,FALSE,l10n_ie
l10n_ie_a7001,7001,Directors remuneration,expense,FALSE,l10n_ie
l10n_ie_a7010,7010,Admin gross salaries,expense,FALSE,l10n_ie
l10n_ie_a7011,7011,Management gross salaries,expense,FALSE,l10n_ie
l10n_ie_a7012,7012,Employers NIC,expense,FALSE,l10n_ie
l10n_ie_a7020,7020,Subcontractors payments,expense,FALSE,l10n_ie
l10n_ie_a7610,7610,Consultancy,expense,FALSE,l10n_ie
l10n_ie_a7620,7620,Legal and professional charges,expense,FALSE,l10n_ie
l10n_ie_a7601,7601,Accounting,expense,FALSE,l10n_ie
l10n_ie_a7602,7602,Auditing,expense,FALSE,l10n_ie
l10n_ie_a7110,7110,"Light, heat and power",expense,FALSE,l10n_ie
l10n_ie_a9992,9992,Rent and rates,expense,FALSE,l10n_ie
l10n_ie_a7120,7120,"Repairs, renewals and maintenance",expense,FALSE,l10n_ie
l10n_ie_a7300,7300,Car hire,expense,FALSE,l10n_ie
l10n_ie_a7301,7301,Car fuel,expense,FALSE,l10n_ie
l10n_ie_a7302,7302,Car maintenance,expense,FALSE,l10n_ie
l10n_ie_a7502,7502,Telephone,expense,FALSE,l10n_ie
l10n_ie_a7503,7503,Internet & hosting,expense,FALSE,l10n_ie
l10n_ie_a7504,7504,Mobiles,expense,FALSE,l10n_ie
l10n_ie_a7505,7505,Stationery,expense,FALSE,l10n_ie
l10n_ie_a7506,7506,Office consumables,expense,FALSE,l10n_ie
l10n_ie_a7507,7507,Postage and Carriage,expense,FALSE,l10n_ie
l10n_ie_a7508,7508,Books,expense,FALSE,l10n_ie
l10n_ie_a7509,7509,Network costs,expense,FALSE,l10n_ie
l10n_ie_a7510,7510,Software expenses,expense,FALSE,l10n_ie
l10n_ie_a7511,7511,Other computer costs,expense,FALSE,l10n_ie
l10n_ie_a7512,7512,Recruitment fees,expense,FALSE,l10n_ie
l10n_ie_a7513,7513,Other admin expenses,expense,FALSE,l10n_ie
l10n_ie_a7700,7700,Exchange gains/losses,expense,FALSE,l10n_ie
l10n_ie_a7710,7710,Other sundry expenses,expense,FALSE,l10n_ie
l10n_ie_a7850,7850,Bad debts,expense,FALSE,l10n_ie
l10n_ie_a7910,7910,"Bank, credit card and other financial charges",expense,FALSE,l10n_ie
l10n_ie_a8000,8000,Intangible assets depn,expense,FALSE,l10n_ie
l10n_ie_a8001,8001,Tangible assets depn,expense,FALSE,l10n_ie
l10n_ie_a8200,8200,Donations,expense,FALSE,l10n_ie
l10n_ie_a8300,8300,Entertaining,expense,FALSE,l10n_ie
l10n_ie_a8400,8400,Insurance,expense,FALSE,l10n_ie
l10n_ie_a8500,8500,Travel and subsistence,expense,FALSE,l10n_ie
l10n_ie_a9000,9000,Profits/Losses on disposals of assets,income,FALSE,l10n_ie
l10n_ie_a4900,4900,Bank Interest received,income,FALSE,l10n_ie
l10n_ie_a4910,4910,Investment Interest received,income,FALSE,l10n_ie
l10n_ie_a7900,7900,Interest paid,expense,FALSE,l10n_ie
l10n_ie_a8800,8800,Corporation tax expense,expense,FALSE,l10n_ie
l10n_ie_a2240,2240,Universal Social Charge USC,expense,FALSE,l10n_ie
l10n_ie_a2250,2250,PRSI Employee,expense,FALSE,l10n_ie
l10n_ie_a2260,2260,PRSI Employer,expense,FALSE,l10n_ie

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
    <!-- Chart template -->
    <record id="l10n_ie" model="account.chart.template">
        <field name="name">IE Tax and Account Chart Template</field>
        <field name="bank_account_code_prefix">1200</field>
        <field name="cash_account_code_prefix">1210</field>
        <field name="transfer_account_code_prefix">1220</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.EUR"/>
        <field name="country_id" ref="base.ie"/>
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2200'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="l10n_ie_tax_pt8" model="account.tax.template">
        <field name="description">PT8</field>
        <field name="chart_template_id" ref="l10n_ie"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard rated purchases from EU (IE)</field>
        <field name="amount_type">percent</field>
        <field name="amount">23</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2200'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2202'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ie_a2201'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="7.22" width="50.4" height="32.5" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.5" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
    </mask>
    <symbol id="c" data-name="account icon" viewBox="0 0 106 106">
      <g style="mask: url(#a)">
        <g>
          <path d="M0,0H106V106H0Z" style="fill: #5a5a64;fill-rule: evenodd"/>
          <path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill: #fff;fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <g>
            <path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill: #393939;fill-rule: evenodd;opacity: 0.324000000953674;isolation: isolate"/>
            <g style="opacity: 0.30000000000000004">
              <g>
                <path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/>
                <path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/>
                <path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/>
                <path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/>
                <path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/>
                <path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/>
                <path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/>
              </g>
              <path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/>
            </g>
            <path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill: #a8a9ab"/>
            <g>
              <path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill: #a8a9ab"/>
              <path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill: #a8a9ab"/>
              <path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill: #a8a9ab"/>
              <path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill: #a8a9ab"/>
              <path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill: #a8a9ab"/>
              <path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill: #a8a9ab"/>
              <path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill: #a8a9ab"/>
            </g>
          </g>
        </g>
      </g>
    </symbol>
  </defs>
  <g>
    <use width="106" height="106" transform="translate(-0.07 0)" xlink:href="#c"/>
    <rect x="6.2" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="1200" height="600" transform="translate(4.8 7.22) scale(0.04 0.05)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABLAAAAMGCAYAAADiFX0pAAAACXBIWXMAAQeYAAEHmAEWNs1oAAAVYklEQVR4Xu3YMRHCUBQAwQeDDpRRoAYbFCiJsiAgxZ9UuWK3PgV3m+9rHwA4aX//VgkAHH2eqwIADu6rAAAAAACuZGABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQZmABAAAAkGZgAQAAAJBmYAEAAACQ9piZbRUBAAAAwFX+4BULb2f5h14AAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

