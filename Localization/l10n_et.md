# Odoo Module: l10n_et

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2012 Michael Telahun Makonnen <mmakonnen@gmail.com>.

```

## File: __manifest__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2012 Michael Telahun Makonnen <mmakonnen@gmail.com>.

{
    'name': 'Ethiopia - Accounting',
    'version': '2.0',
    'category': 'Localization',
    'description': """
Base Module for Ethiopian Localization
======================================

This is the latest Ethiopian Odoo localization and consists of:
    - Chart of Accounts
    - VAT tax structure
    - Withholding tax structure
    - Regional State listings
    """,
    'author':'Michael Telahun Makonnen <mmakonnen@gmail.com>',
    'website':'http://miketelahun.wordpress.com',
    'depends': [
        'account',
    ],
    'data': [
        'data/l10n_et_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_data.xml',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","code","name","user_type_id:id","reconcile","chart_template_id:id"
"l10n_et1100","1100","Sales of Goods and Services","account.data_account_type_revenue","",l10n_et.l10n_et
"l10n_et2102","2102","Cash at bank in foreigh currency","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2104","2104","Letter of Credit restricted account","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2201","2201","Suspense","account.data_account_type_receivable","True",l10n_et.l10n_et
"l10n_et2202","2202","Cash shortage","account.data_account_type_receivable","True",l10n_et.l10n_et
"l10n_et2203","2203","Advance to staff","account.data_account_type_receivable","True",l10n_et.l10n_et
"l10n_et2204","2204","Cash Registers","account.data_account_type_receivable","True",l10n_et.l10n_et
"l10n_et2211","2211","Trade Debtors","account.data_account_type_receivable","True",l10n_et.l10n_et
"l10n_et2212","2212","VAT Receivable on Purchases","account.data_account_type_current_liabilities","",l10n_et.l10n_et
"l10n_et2213","2213","Withholding Receivable on Sales","account.data_account_type_current_liabilities","",l10n_et.l10n_et
"l10n_et2214","2214","VAT Withholding Receivable on Sales","account.data_account_type_current_liabilities","",l10n_et.l10n_et
"l10n_et2215","2215","Trade Debtors (PoS)","account.data_account_type_receivable","True",l10n_et.l10n_et
"l10n_et2251","2251","Advance to contractors","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2252","2252","Advance to consultant","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2253","2253","Advance to supplier","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2274","2254","Other Debtors","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2301","2301","Goods in Transit","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2351","2351","Stock","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2411","2411","Work in Progress","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2412","2412","Finished Goods","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et2501","2501","Construction of buildings","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2503","2503","Construction of infrastructure","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2521","2521","Vehicles and other vehicular transport","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2522","2522","Aircraft, etc","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2523","2523","Plant machinery and equipment","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2525","2525","Buildings","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2527","2527","Infrastructure","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2529","2529","Furnishings and fixtures","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et2530","2530","Livestock and tansport animals","account.data_account_type_fixed_assets","",l10n_et.l10n_et
"l10n_et3001","3001","Grace period payables","account.data_account_type_payable","True",l10n_et.l10n_et
"l10n_et3002","3002","Trade Creditors","account.data_account_type_payable","True",l10n_et.l10n_et
"l10n_et3003","3003","Pension contribution payable","account.data_account_type_payable","True",l10n_et.l10n_et
"l10n_et3004","3004","Salary payable","account.data_account_type_payable","True",l10n_et.l10n_et
"l10n_et3006","3006","Witholding Payable","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et3007","3007","VAT Payable","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et3008","3008","Federal Income Tax","account.data_account_type_current_assets","",l10n_et.l10n_et
"l10n_et3054","3054","Other deposits","account.data_account_type_current_liabilities","",l10n_et.l10n_et
"l10n_et3061","3061","Retention on contract","account.data_account_type_current_liabilities","",l10n_et.l10n_et
"l10n_et3103","3103","Commercial Loan","account.data_account_type_current_liabilities","",l10n_et.l10n_et
"l10n_et3153","3153","Commercial Loan","account.data_account_type_current_liabilities","",l10n_et.l10n_et
"l10n_et4001","4010","Share capital / equity","account.data_account_type_equity","",l10n_et.l10n_et
"l10n_et4004","4020","Reserves","account.data_account_type_equity","",l10n_et.l10n_et
"l10n_et5111","5111","Cost of Goods and Services","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et5901","5901","Inventory Adjustments","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et5911","5911","Purchase Returns and Allowances","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et5921","5921","Other","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6111","6111","Salaries to permanent staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6113","6113","Wages to contract staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6114","6114","Wages to casual staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6115","6115","Wages to external contract staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6116","6116","Miscellaneous payments to staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6121","6121","Allowances to permanent staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6123","6123","Allowances to contract staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6124","6124","Allowances to external contract staff","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6131","6131","Contribution to permanent staff pensions","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6211","6211","Uniforms, bedding","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6212","6212","Office supplies","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6213","6213","Printing","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6214","6214","Medical supplies","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6215","6215","Educational supplies","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6216","6216","Food","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6217","6217","Fuel and lubricants","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6218","6218","Other material and supplies","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6219","6219","Miscellaneous equipment","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6221","6221","Agriculture, forestry and marine inputs","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6222","6222","Veterinary supplies and drugs","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6223","6223","Research and development supplies","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6231","6231","Per diem","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6232","6232","Transport fees","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6233","6233","Official entertainment","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6241","6241","Maintenance and repair of vehicles and other transport","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6243","6243","Maintenance and repair of plant, and equipment","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6244","6244","Maintenance and repair of buildings, furnishings and fixtures","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6245","6245","Maintenance and repair of infrastructure","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6251","6251","Contracted professional services","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6252","6252","Rent","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6253","6253","Advertising","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6254","6254","Insurance","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6255","6255","Freight","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6256","6256","Fees and charges","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6257","6257","Electricity charges","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6258","6258","Telecommunication charges","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6259","6259","Water and other utilities","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6271","6271","Local training","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6272","6272","External training","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6311","6311","Depreciation of vehicles and other vehicular transport","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6313","6313","Depreciation of plant, machinery and equipment","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6314","6314","Depreciation of buildings, furnishings and fixtures","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6315","6315","Depreciation of livestock and transport animals","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6321","6321","Pre-construction activities","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6322","6322","Construction of buildings","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6324","6324","Construction of infrastructure","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6431","6431","Payments on the principal of foreign debt","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6432","6432","Payments of interest and bank charges on foreign debt","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6433","6433","Payments on the principal of local debt","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6434","6434","Payments of interest and bank charges on local debt","account.data_account_type_expenses","",l10n_et.l10n_et
"l10n_et6435","1200","Foreign Exchange Currency Gain Account","account.data_account_type_other_income","",l10n_et.l10n_et
"l10n_et6436","6260","Foreign Exchange Currency Loss Account","account.data_account_type_expenses","",l10n_et.l10n_et

```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_vat_0,VAT 0%
tax_group_vat_15,VAT 15%
tax_group_withh_2,Withholding 2%
tax_group_withh_15,Withholding 15%
tax_group_withh_35,Withholding 35%

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_et.l10n_et')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_et" model="account.chart.template">
        <field name="property_account_receivable_id" ref="l10n_et2211"/>
        <field name="property_account_payable_id" ref="l10n_et3002"/>
        <field name="property_account_expense_categ_id" ref="l10n_et2301"/>
        <field name="property_account_income_categ_id" ref="l10n_et1100"/>
        <field name="income_currency_exchange_account_id" ref="l10n_et6435"/>
        <field name="expense_currency_exchange_account_id" ref="l10n_et6436"/>
        <field name="default_pos_receivable_account_id" ref="l10n_et2215" />
    </record>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>

        <record id="id_tax03" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT 15% rated sales</field>
            <field name="description">tax03</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_vat_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_sale_vat_rated_15')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3007'),
                    'minus_report_line_ids': [ref('account_tax_report_net_sale_vat_15')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_sale_vat_rated_15')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3007'),
                    'plus_report_line_ids': [ref('account_tax_report_net_sale_vat_15')],
                }),
            ]"/>
        </record>

        <record id="id_tax04" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT 0% rated sales</field>
            <field name="description">tax04</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_sale_vat_rated_0')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_sale_vat_rated_0')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="id_tax06" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT Exempt rated sales</field>
            <field name="description">tax06</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_sale_vat_exmpt')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_sale_vat_exmpt')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="id_tax11" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT Out of Scope rated sales</field>
            <field name="description">tax11</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_sale_vat_out_scope')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_sale_vat_out_scope')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="id_tax02" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">Withholding 2% rated sales</field>
            <field name="amount">-2</field>
            <field name="amount_type">percent</field>
            <field name="description">tax02</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_withh_2"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_sale_withholding_2')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3006'),
                    'minus_report_line_ids': [ref('account_tax_report_net_sale_witheld_2')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_sale_withholding_2')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3006'),
                    'plus_report_line_ids': [ref('account_tax_report_net_sale_witheld_2')],
                }),
            ]"/>
        </record>

        <record id="id_tax13" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">Withholding 35% rated sales</field>
            <field name="description">tax13</field>
            <field name="amount">-35</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_withh_35"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_sale_withholding_35')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3006'),
                    'plus_report_line_ids': [ref('account_tax_report_net_sale_witheld_35')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_sale_withholding_35')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3006'),
                    'minus_report_line_ids': [ref('account_tax_report_net_sale_witheld_35')],
                }),
            ]"/>
        </record>

        <record id="id_tax14" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">Withholding VAT 15% rated sales</field>
            <field name="description">tax14</field>
            <field name="amount">-15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_withh_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_sale_vat_withholding')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3006'),
                    'plus_report_line_ids': [ref('account_tax_report_net_sale_vat_witheld')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_sale_vat_withholding')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et3006'),
                    'minus_report_line_ids': [ref('account_tax_report_net_sale_vat_witheld')],
                }),
            ]"/>
        </record>

        <record id="id_tax08" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT 15% rated purchases</field>
            <field name="description">tax08</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_vat_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_purch_vt_ratd_15')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et2212'),
                    'plus_report_line_ids': [ref('account_tax_report_purch_vat_rated_15')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_purch_vt_ratd_15')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et2212'),
                    'minus_report_line_ids': [ref('account_tax_report_purch_vat_rated_15')],
                }),
            ]"/>
        </record>

        <record id="id_tax07" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT 0% rated purchases</field>
            <field name="description">tax07</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_purch_vt_ratd_0')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_purch_vt_ratd_0')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="id_tax10" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT Exempt rated purchases</field>
            <field name="description">tax10</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_purch_vt_exmpt')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_purch_vt_exmpt')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="id_tax09" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">VAT Out of Scope rated purchases</field>
            <field name="description">tax09</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_purch_vt_out_scope')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_purch_vt_out_scope')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="id_tax05" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">Withholding 2% rated purchases</field>
            <field name="description">tax05</field>
            <field name="amount">-2</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_withh_2"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_purch_withholding_2')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et2213'),
                    'plus_report_line_ids': [ref('account_tax_report_net_purch_witholding_2')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_purch_withholding_2')],
                }),
                    (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et2213'),
                    'minus_report_line_ids': [ref('account_tax_report_net_purch_witholding_2')],
                }),
            ]"/>
        </record>

        <record id="id_tax12" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_et"/>
            <field name="name">Withholding 35% rated purchases</field>
            <field name="description">tax12</field>
            <field name="amount">-35</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_withh_35"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_purch_withholding_35')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et2213'),
                    'plus_report_line_ids': [ref('account_tax_report_net_purch_witholding_35')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_purch_withholding_35')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_et2213'),
                    'minus_report_line_ids': [ref('account_tax_report_net_purch_witholding_35')],
                }),
            ]"/>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="account_tax_report_purch_vt" model="account.tax.report.line">
            <field name="name">Taxable Purchases - VAT</field>
            <field name="sequence" eval="1"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_vt_out_scope" model="account.tax.report.line">
            <field name="name">Taxable Purchase VAT Out of Scope</field>
            <field name="tag_name">Taxable Purchase VAT Out of Scope</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_purch_vt"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_vt_exmpt" model="account.tax.report.line">
            <field name="name">Taxable Purchase VAT Exempt</field>
            <field name="tag_name">Taxable Purchase VAT Exempt</field>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="account_tax_report_purch_vt"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_vt_ratd_0" model="account.tax.report.line">
            <field name="name">Taxable Purchase VAT Rated 0%</field>
            <field name="tag_name">Taxable Purchase VAT Rated 0%</field>
            <field name="sequence" eval="3"/>
            <field name="parent_id" ref="account_tax_report_purch_vt"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_vt_ratd_15" model="account.tax.report.line">
            <field name="name">Taxable Purchase VAT Rated 15%</field>
            <field name="tag_name">Taxable Purchase VAT Rated 15%</field>
            <field name="sequence" eval="4"/>
            <field name="parent_id" ref="account_tax_report_purch_vt"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_withholding" model="account.tax.report.line">
            <field name="name">Taxable Purchases - Witholding</field>
            <field name="sequence" eval="2"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_withholding_2" model="account.tax.report.line">
            <field name="name">Taxable 2% Withholding on Purchases</field>
            <field name="tag_name">Taxable 2% Withholding on Purchases</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_purch_withholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_withholding_35" model="account.tax.report.line">
            <field name="name">Taxable 35% Withholding on Purchases</field>
            <field name="tag_name">Taxable 35% Withholding on Purchases</field>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="account_tax_report_purch_withholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_vat" model="account.tax.report.line">
            <field name="name">Taxable Sales - VAT</field>
            <field name="sequence" eval="3"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_vat_out_scope" model="account.tax.report.line">
            <field name="name">Taxable Sales VAT Out of Scope (Sales)</field>
            <field name="tag_name">Taxable Sales VAT Out of Scope (Sales)</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_sale_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_vat_exmpt" model="account.tax.report.line">
            <field name="name">Taxable Sales VAT Exempt</field>
            <field name="tag_name">Taxable Sales VAT Exempt</field>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="account_tax_report_sale_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_vat_rated_0" model="account.tax.report.line">
            <field name="name">Taxable Sales VAT Rated 0%</field>
            <field name="tag_name">Taxable Sales VAT Rated 0%</field>
            <field name="sequence" eval="3"/>
            <field name="parent_id" ref="account_tax_report_sale_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_vat_rated_15" model="account.tax.report.line">
            <field name="name">Taxable Sales VAT Rated 15%</field>
            <field name="tag_name">Taxable Sales VAT Rated 15%</field>
            <field name="sequence" eval="4"/>
            <field name="parent_id" ref="account_tax_report_sale_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_withholding" model="account.tax.report.line">
            <field name="name">Taxable Sales - Withholding</field>
            <field name="sequence" eval="4"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_withholding_2" model="account.tax.report.line">
            <field name="name">Taxable 2% Withholding on Sales</field>
            <field name="tag_name">Taxable 2% Withholding on Sales</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_sale_withholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_withholding_35" model="account.tax.report.line">
            <field name="name">Taxable 35% Withholding on Sales</field>
            <field name="tag_name">Taxable 35% Withholding on Sales</field>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="account_tax_report_sale_withholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_sale_vat_withholding" model="account.tax.report.line">
            <field name="name">Taxable VAT Withholding on Sales</field>
            <field name="tag_name">Taxable VAT Withholding on Sales</field>
            <field name="sequence" eval="3"/>
            <field name="parent_id" ref="account_tax_report_sale_withholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_vat" model="account.tax.report.line">
            <field name="name">Net VAT to be Paid/Reclaimed</field>
            <field name="sequence" eval="5"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_vat" model="account.tax.report.line">
            <field name="name">Purchase VAT</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_net_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_purch_vat_rated_15" model="account.tax.report.line">
            <field name="name">Purchase VAT Rated 15%</field>
            <field name="tag_name">Purchase VAT Rated 15%</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_purch_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_sale_vat" model="account.tax.report.line">
            <field name="name">Sales VAT</field>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="account_tax_report_net_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_sale_vat_15" model="account.tax.report.line">
            <field name="name">Sales VAT Rated 15%</field>
            <field name="tag_name">Sales VAT Rated 15%</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_net_sale_vat"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_purch_witholding" model="account.tax.report.line">
            <field name="name">Withholding on Purchases</field>
            <field name="sequence" eval="6"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_purch_witholding_2" model="account.tax.report.line">
            <field name="name">2% Withholding on Purchases</field>
            <field name="tag_name">2% Withholding on Purchases</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_net_purch_witholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_purch_witholding_35" model="account.tax.report.line">
            <field name="name">35% Withholding on Purchases</field>
            <field name="tag_name">35% Withholding on Purchases</field>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="account_tax_report_net_purch_witholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_sale_witholding" model="account.tax.report.line">
            <field name="name">Withholding on Sales</field>
            <field name="sequence" eval="7"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_sale_witheld_2" model="account.tax.report.line">
            <field name="name">2% Withheld on Sales</field>
            <field name="tag_name">2% Withheld on Sales</field>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="account_tax_report_net_sale_witholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_sale_witheld_35" model="account.tax.report.line">
            <field name="name">35% Withheld on Sales</field>
            <field name="tag_name">35% Withheld on Sales</field>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="account_tax_report_net_sale_witholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

        <record id="account_tax_report_net_sale_vat_witheld" model="account.tax.report.line">
            <field name="name">VAT Withheld on Sales</field>
            <field name="tag_name">VAT Withheld on Sales</field>
            <field name="sequence" eval="3"/>
            <field name="parent_id" ref="account_tax_report_net_sale_witholding"/>
            <field name="country_id" ref="base.et"/>
        </record>

    </data>
</odoo>
```

## File: data\l10n_et_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_et_statements_menu" name="Ethiopia" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>

    <record id="l10n_et" model="account.chart.template">
        <field name="name">Ethiopia Tax and Account Chart Template</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.ETB"/>
        <field name="bank_account_code_prefix">211</field>
        <field name="cash_account_code_prefix">211</field>
        <field name="transfer_account_code_prefix">212</field>
    </record>
</odoo>

```

