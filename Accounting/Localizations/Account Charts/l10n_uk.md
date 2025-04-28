# Odoo Module: l10n_uk

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2011 Smartmode LTD (<http://www.smartmode.co.uk>).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2011 Smartmode LTD (<http://www.smartmode.co.uk>).

{
    'name': 'UK - Accounting',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest UK Odoo localisation necessary to run Odoo accounting for UK SME's with:
=================================================================================================
    - a CT600-ready chart of accounts
    - VAT100-ready tax structure
    - InfoLogic UK counties listing
    - a few other adaptations""",
    'author': 'SmartMode LTD',
    'website': 'https://www.odoo.com/page/accounting',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
        'data/l10n_uk_chart_data.xml',
        'data/account.account.template.csv',
        'data/account.chart.template.csv',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_data.xml',
        'data/account_chart_template_data.xml',
    ],
    'demo': [
        'demo/l10n_uk_demo.xml',
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","code","name","user_type_id:id","reconcile","chart_template_id:id"
"0010","0010","Software","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0011","0011","Software Depreciation","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0020","0020","Patents & Trademarks","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0021","0021","Patents & Trademarks Depreciation","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0030","0030","Fixtures and fittings","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0031","0031","Fixtures and fittings Depreciation","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0040","0040","Land and buildings","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0041","0041","Land and buildings Depreciation","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0050","0050","Motor vehicles","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0051","0051","Motor vehicles Depreciation","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0060","0060","Office equipment (inc computer equipment)","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0061","0061","Office equipment (inc computer equipment) Depreciation","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0070","0070","Plant and machinery","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"0071","0071","Plant and machinery Depreciation","account.data_account_type_fixed_assets","FALSE","l10n_uk"
"1001","1001","Stock","account.data_account_type_current_assets","TRUE","l10n_uk"
"1002","1002","Work in Progress","account.data_account_type_current_assets","FALSE","l10n_uk"
"1003","1003","Finished Goods","account.data_account_type_current_assets","FALSE","l10n_uk"
"1100","1100","Debtors Control Account","account.data_account_type_receivable","TRUE","l10n_uk"
"1101","1101","Sundry Debtors","account.data_account_type_receivable","TRUE","l10n_uk"
"1102","1102","Other Debtors","account.data_account_type_current_assets","FALSE","l10n_uk"
"1104","1104","Debtors Control Account (PoS)","account.data_account_type_receivable","TRUE","l10n_uk"
"1240","1240","Company Credit Card","account.data_account_type_current_assets","TRUE","l10n_uk"
"1103","1103","Prepayments","account.data_account_type_current_assets","FALSE","l10n_uk"
"2100","2100","Creditors Control Account","account.data_account_type_payable","TRUE","l10n_uk"
"2101","2101","Sundry Creditors","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2102","2102","Other Creditors","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2200","2200","Sales Tax Control Account","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2201","2201","Purchase Tax Control Account","account.data_account_type_current_assets","FALSE","l10n_uk"
"2202","2202","HMRC - VAT Account","account.data_account_type_payable","TRUE","l10n_uk"
"2204","2204","Manual Adjustments ﾖ VAT","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2210","2210","P.A.Y.E. & NI","account.data_account_type_payable","TRUE","l10n_uk"
"2220","2220","Net Wages","account.data_account_type_payable","TRUE","l10n_uk"
"2230","2230","Pension Fund","account.data_account_type_payable","TRUE","l10n_uk"
"2150","2150","Bad debt provision","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2109","2109","Accruals","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2320","2320","Corporation Tax","account.data_account_type_payable","TRUE","l10n_uk"
"2300","2300","Loans","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2310","2310","Hire Purchase","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"2330","2330","Mortgages","account.data_account_type_current_liabilities","FALSE","l10n_uk"
"3000","3000","Called up share capital","account.data_account_type_equity","FALSE","l10n_uk"
"3010","3010","Share premium account","account.data_account_type_equity","FALSE","l10n_uk"
"3020","3020","Revaluation reserve","account.data_account_type_equity","FALSE","l10n_uk"
"3030","3030","Other reserves","account.data_account_type_equity","FALSE","l10n_uk"
"4000","4000","Sales category 1","account.data_account_type_revenue","FALSE","l10n_uk"
"4001","4001","Sales category 2","account.data_account_type_revenue","FALSE","l10n_uk"
"4002","4002","Sales category 3","account.data_account_type_revenue","FALSE","l10n_uk"
"4003","4003","Sales category 4","account.data_account_type_revenue","FALSE","l10n_uk"
"5000","5000","Cost of sales 1","account.data_account_type_expenses","FALSE","l10n_uk"
"5001","5001","Cost of sales 2","account.data_account_type_expenses","FALSE","l10n_uk"
"5002","5002","Cost of sales 3","account.data_account_type_expenses","FALSE","l10n_uk"
"5003","5003","Cost of sales 4","account.data_account_type_expenses","FALSE","l10n_uk"
"6000","6000","Marketing, POS","account.data_account_type_expenses","FALSE","l10n_uk"
"6001","6001","Exhibitions and events","account.data_account_type_expenses","FALSE","l10n_uk"
"6002","6002","PR","account.data_account_type_expenses","FALSE","l10n_uk"
"6010","6010","Distribution vehicles","account.data_account_type_expenses","FALSE","l10n_uk"
"6020","6020","Distribution salaries and wages","account.data_account_type_expenses","FALSE","l10n_uk"
"6030","6030","Shipping","account.data_account_type_expenses","FALSE","l10n_uk"
"7000","7000","Directors pension","account.data_account_type_expenses","FALSE","l10n_uk"
"7001","7001","Directors remuneration","account.data_account_type_expenses","FALSE","l10n_uk"
"7010","7010","Admin gross salaries","account.data_account_type_expenses","FALSE","l10n_uk"
"7011","7011","Management gross salaries","account.data_account_type_expenses","FALSE","l10n_uk"
"7012","7012","Employers NIC","account.data_account_type_expenses","FALSE","l10n_uk"
"7020","7020","Subcontractors payments","account.data_account_type_expenses","FALSE","l10n_uk"
"7610","7610","Consultancy","account.data_account_type_expenses","FALSE","l10n_uk"
"7620","7620","Legal and professional charges","account.data_account_type_expenses","FALSE","l10n_uk"
"7601","7601","Accounting","account.data_account_type_expenses","FALSE","l10n_uk"
"7602","7602","Auditing","account.data_account_type_expenses","FALSE","l10n_uk"
"7110","7110","Light, heat and power","account.data_account_type_expenses","FALSE","l10n_uk"
"7100","7100","Rent and rates","account.data_account_type_expenses","FALSE","l10n_uk"
"7120","7120","Repairs, renewals and maintenance","account.data_account_type_expenses","FALSE","l10n_uk"
"7300","7300","Car hire","account.data_account_type_expenses","FALSE","l10n_uk"
"7301","7301","Car fuel","account.data_account_type_expenses","FALSE","l10n_uk"
"7302","7302","Car maintenance","account.data_account_type_expenses","FALSE","l10n_uk"
"7502","7502","Telephone","account.data_account_type_expenses","FALSE","l10n_uk"
"7503","7503","Internet & hosting","account.data_account_type_expenses","FALSE","l10n_uk"
"7504","7504","Mobiles","account.data_account_type_expenses","FALSE","l10n_uk"
"7505","7505","Stationery","account.data_account_type_expenses","FALSE","l10n_uk"
"7506","7506","Office consumables","account.data_account_type_expenses","FALSE","l10n_uk"
"7507","7507","Postage and Carriage","account.data_account_type_expenses","FALSE","l10n_uk"
"7508","7508","Books","account.data_account_type_expenses","FALSE","l10n_uk"
"7509","7509","Network costs","account.data_account_type_expenses","FALSE","l10n_uk"
"7510","7510","Software expenses","account.data_account_type_expenses","FALSE","l10n_uk"
"7511","7511","Other computer costs","account.data_account_type_expenses","FALSE","l10n_uk"
"7512","7512","Recruitment fees","account.data_account_type_expenses","FALSE","l10n_uk"
"7513","7513","Other admin expenses","account.data_account_type_expenses","FALSE","l10n_uk"
"7700","7700","Exchange gains/losses","account.data_account_type_expenses","FALSE","l10n_uk"
"7710","7710","Other sundry expenses","account.data_account_type_expenses","FALSE","l10n_uk"
"7850","7850","Bad debts","account.data_account_type_expenses","FALSE","l10n_uk"
"7910","7910","Bank, credit card and other financial charges","account.data_account_type_expenses","FALSE","l10n_uk"
"8000","8000","Intangible assets depn","account.data_account_type_expenses","FALSE","l10n_uk"
"8001","8001","Tangible assets depn","account.data_account_type_expenses","FALSE","l10n_uk"
"8200","8200","Donations","account.data_account_type_expenses","FALSE","l10n_uk"
"8300","8300","Entertaining","account.data_account_type_expenses","FALSE","l10n_uk"
"8400","8400","Insurance","account.data_account_type_expenses","FALSE","l10n_uk"
"8500","8500","Travel and subsistence","account.data_account_type_expenses","FALSE","l10n_uk"
"9000","9000","Profits/Losses on disposals of assets","account.data_account_type_revenue","FALSE","l10n_uk"
"4900","4900","Bank Interest received","account.data_account_type_revenue","FALSE","l10n_uk"
"4910","4910","Investment Interest received","account.data_account_type_revenue","FALSE","l10n_uk"
"7900","7900","Interest paid","account.data_account_type_expenses","FALSE","l10n_uk"
"8800","8800","Corporation tax expense","account.data_account_type_expenses","FALSE","l10n_uk"

```

## File: data\account.chart.template.csv

```csv
"id","name","property_account_receivable_id:id","property_account_payable_id:id","property_account_expense_categ_id:id","property_account_income_categ_id:id","income_currency_exchange_account_id:id","expense_currency_exchange_account_id:id","default_pos_receivable_account_id:id","use_anglo_saxon"
"l10n_uk","UK Tax and Account Chart Template (by SmartMode)","1100","2100","5000","4000","7700","7700","1104","True"

```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_0,TAX 0%
tax_group_5,TAX 5%
tax_group_175,TAX 17.5%
tax_group_20,TAX 20%

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_uk.l10n_uk')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="ST0" model="account.tax.template">
        <field name="description">ST0</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Zero rated sales</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
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
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="ST2" model="account.tax.template">
        <field name="description">ST2</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Exempt sales</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
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
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="PT0" model="account.tax.template">
        <field name="description">PT0</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Zero rated purchases</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
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
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="PT2" model="account.tax.template">
        <field name="description">PT2</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Exempt purchases</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
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
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="PT8" model="account.tax.template">
        <field name="description">PT8</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard rated purchases from EC</field>
        <field name="amount_type">percent</field>
        <field name="amount">20</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box9'), ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'plus_report_line_ids': [ref('account_tax_report_line_vat_box2')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'minus_report_line_ids': [ref('account_tax_report_line_vat_box4')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box9'), ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'minus_report_line_ids': [ref('account_tax_report_line_vat_box2')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'plus_report_line_ids': [ref('account_tax_report_line_vat_box4')],
            }),
        ]"/>
    </record>

    <record id="PT5" model="account.tax.template">
        <field name="description">PT5</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Lower rate purchases (5%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_group_id" ref="tax_group_5"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'plus_report_line_ids': [ref('account_tax_report_line_vat_box4')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'minus_report_line_ids': [ref('account_tax_report_line_vat_box4')],
            }),
        ]"/>
    </record>

	<record id="ST5" model="account.tax.template">
        <field name="description">ST5</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Lower rate sales (5%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_vat_box1')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_vat_box1')],
            }),
        ]"/>
    </record>

    <record id="ST4" model="account.tax.template">
        <field name="description">ST4</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Sales to customers in EC</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box8'), ref('account_tax_report_line_exd_vat_box6')],
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
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box8'), ref('account_tax_report_line_exd_vat_box6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="PT7" model="account.tax.template">
        <field name="description">PT7</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Zero rated purchases from EC</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box9'), ref('account_tax_report_line_exd_vat_box7')],
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
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box9'), ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="ST11" model="account.tax.template">
        <field name="description">ST11</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Standard rate sales (20%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">20</field>
        <field name="tax_group_id" ref="tax_group_20"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_vat_box1')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_vat_box1')],
            }),
        ]"/>
    </record>

    <record id="PT11" model="account.tax.template">
        <field name="description">PT11</field>
        <field name="chart_template_id" ref="l10n_uk"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard rate purchases (20%)</field>
        <field name="amount_type">percent</field>
        <field name="amount">20</field>
        <field name="tax_group_id" ref="tax_group_20"/>
		<field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'plus_report_line_ids': [ref('account_tax_report_line_vat_box4')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_exd_vat_box7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2201'),
                'minus_report_line_ids': [ref('account_tax_report_line_vat_box4')],
            }),
        ]"/>
    </record>

</odoo>
```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.uk"/>
    </record>

    <record id="account_tax_report_line_vat_cal" model="account.tax.report.line">
        <field name="name">VAT calculations</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="account_tax_report_line_vat_box1" model="account.tax.report.line">
        <field name="name">[BOX 1] VAT due on sales and other outputs</field>
        <field name="tag_name">1</field>
        <field name="code">UKTAX_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_vat_cal"/>
    </record>

    <record id="account_tax_report_line_vat_box2" model="account.tax.report.line">
        <field name="name">[BOX 2] VAT due on acquisitions from EC</field>
        <field name="tag_name">2</field>
        <field name="code">UKTAX_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_vat_cal"/>
    </record>

    <record id="account_tax_report_line_vat_box3" model="account.tax.report.line">
        <field name="name">[BOX 3] Total VAT due (box 1 + box 2)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="formula">UKTAX_1 + UKTAX_2</field>
        <field name="parent_id" ref="account_tax_report_line_vat_cal"/>
    </record>

    <record id="account_tax_report_line_vat_box4" model="account.tax.report.line">
        <field name="name">[BOX 4] VAT reclaimed on purchases and other inputs (including acquisitions from EC)</field>
        <field name="tag_name">4</field>
        <field name="code">UKTAX_4</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_vat_cal"/>
    </record>

    <record id="account_tax_report_line_vat_box5" model="account.tax.report.line">
        <field name="name">[BOX 5] VAT to pay/reclaim</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="formula">UKTAX_1 + UKTAX_2 - UKTAX_4</field>
        <field name="parent_id" ref="account_tax_report_line_vat_cal"/>
    </record>

    <record id="account_tax_report_line_exd_vat" model="account.tax.report.line">
        <field name="name">Sales and Purchases Excluding VAT</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="account_tax_report_line_exd_vat_box6" model="account.tax.report.line">
        <field name="name">[BOX 6] Total value of sales and other outputs excluding VAT (including EC supplies)</field>
        <field name="tag_name">6</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_exd_vat"/>
    </record>

    <record id="account_tax_report_line_ec_exd_vat" model="account.tax.report.line">
        <field name="name">EC Sales and Purchases excluding VAT</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="account_tax_report_line_exd_vat_box7" model="account.tax.report.line">
        <field name="name">[BOX 7] Total value of purchases and inputs excluding VAT (including EC acquisitions)</field>
        <field name="tag_name">7</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_exd_vat"/>
    </record>

    <record id="account_tax_report_line_exd_vat_box8" model="account.tax.report.line">
        <field name="name">[BOX 8] Total value of EC sales excluding VAT</field>
        <field name="tag_name">8</field>
        <field name="code">UKTAX_8</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_ec_exd_vat"/>
    </record>

    <record id="account_tax_report_line_exd_vat_box9" model="account.tax.report.line">
        <field name="name">[BOX 9] Total value of EC purchases excluding VAT</field>
        <field name="tag_name">9</field>
        <field name="code">UKTAX_9</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_ec_exd_vat"/>
    </record>

</odoo>
```

## File: data\l10n_uk_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_uk_statements_menu" name="United Kingdom" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

        <!-- Chart template -->
        <record id="l10n_uk" model="account.chart.template">
            <field name="name">UK Tax and Account Chart Template (by SmartMode)</field>
            <field name="bank_account_code_prefix">1200</field>
            <field name="cash_account_code_prefix">1210</field>
            <field name="transfer_account_code_prefix">1220</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.GBP"/>
        </record>
</odoo>

```

