# Odoo Module: l10n_za

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-

# Copyright (C) 2017 Paradigm Digital (<http://www.paradigmdigital.co.za>).

{
    'name': 'South Africa - Accounting',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest basic South African localisation necessary to run Odoo in ZA:
================================================================================
    - a generic chart of accounts
    - SARS VAT Ready Structure""",
    'author': 'Paradigm Digital',
    'website': 'https://www.paradigmdigital.co.za',
    'depends': ['account', 'base_vat'],
    'data': [
        'data/account.account.tag.csv',
        'data/account_tax_report_data.xml',
        'data/account.tax.group.csv',
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_post_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
id,name,applicability,country_id:id
tag_ST1,ST1,taxes,base.za
tag_ST1A,ST1A,taxes,base.za
tag_ST2,ST2,taxes,base.za
tag_ST2A,ST2A,taxes,base.za
tag_ST3,ST3,taxes,base.za
tag_ST5,ST5,taxes,base.za
tag_ST7,ST7,taxes,base.za
tag_ST10,ST10,taxes,base.za
tag_ST12,ST12,taxes,base.za
tag_PT14,PT14,taxes,base.za
tag_PT14A,PT14A,taxes,base.za
tag_PT15,PT15,taxes,base.za
tag_PT15A,PT15A,taxes,base.za
tag_PT16,PT16,taxes,base.za
tag_PT17,PT17,taxes,base.za
tag_PT18,PT18,taxes,base.za

```

## File: data\account.account.template.csv

```csv
id,code,name,user_type_id/id,reconcile,chart_template_id/id
100020,100020,Stock Valuation Account,account.data_account_type_current_assets,True,default_chart_template
100030,100030,Stock Work In Progress,account.data_account_type_current_assets,False,default_chart_template
100040,100040,Stock Finished Goods,account.data_account_type_current_assets,False,default_chart_template
100050,100050,Stock Delivered Control Account,account.data_account_type_current_assets,True,default_chart_template
100060,100060,Purchase Tax Control Account,account.data_account_type_current_assets,False,default_chart_template
100070,100070,Other Current Assets,account.data_account_type_current_assets,False,default_chart_template
110010,110010,Debtors Control Account,account.data_account_type_receivable,True,default_chart_template
110020,110020,Sundry Debtors,account.data_account_type_receivable,True,default_chart_template
110030,110030,Debtors Control Account (PoS),account.data_account_type_receivable,True,default_chart_template
124010,124010,Credit Card Merchant Account,account.data_account_type_liquidity,False,default_chart_template
126010,126010,Cash In Hand,account.data_account_type_liquidity,False,default_chart_template
130010,130010,Prepayments,account.data_account_type_prepayments,False,default_chart_template
140010,140010,Software,account.data_account_type_fixed_assets,False,default_chart_template
140020,140020,Patents & Trademarks,account.data_account_type_fixed_assets,False,default_chart_template
140030,140030,Fixtures & Fittings,account.data_account_type_fixed_assets,False,default_chart_template
140040,140040,Land & Buildings,account.data_account_type_fixed_assets,False,default_chart_template
140050,140050,Motor Vehicles,account.data_account_type_fixed_assets,False,default_chart_template
140060,140060,Office Equipment (incl computer equipment),account.data_account_type_fixed_assets,False,default_chart_template
140070,140070,Plant & Machinery,account.data_account_type_fixed_assets,False,default_chart_template
150010,150010,Non-current assets,account.data_account_type_non_current_assets,False,default_chart_template
200010,200010,Stock Received Control Account,account.data_account_type_current_liabilities,True,default_chart_template
200020,200020,Sundry Creditors,account.data_account_type_current_liabilities,False,default_chart_template
200030,200030,Other Creditors,account.data_account_type_current_liabilities,False,default_chart_template
200040,200040,Accruals,account.data_account_type_current_liabilities,False,default_chart_template
200050,200050,Bad debt provision,account.data_account_type_current_liabilities,False,default_chart_template
200060,200060,Sales Tax Control Account,account.data_account_type_current_liabilities,False,default_chart_template
200070,200070,Manual Adjustments & VAT,account.data_account_type_current_liabilities,False,default_chart_template
200080,200080,Loans,account.data_account_type_current_liabilities,False,default_chart_template
200090,200090,Hire Purchase,account.data_account_type_current_liabilities,False,default_chart_template
200100,200100,Mortgages,account.data_account_type_current_liabilities,False,default_chart_template
210010,210010,Company Credit Card,account.data_account_type_credit_card,False,default_chart_template
220010,220010,Creditors Control Account,account.data_account_type_payable,True,default_chart_template
220020,220020,SARS - VAT Account,account.data_account_type_payable,True,default_chart_template
220030,220030,P.A.Y.E. & UIF,account.data_account_type_payable,True,default_chart_template
220040,220040,Net Wages,account.data_account_type_payable,True,default_chart_template
220050,220050,Pension Fund,account.data_account_type_payable,True,default_chart_template
220060,220060,Corporation Tax,account.data_account_type_payable,True,default_chart_template
300010,300010,Called up share capital,account.data_account_type_equity,False,default_chart_template
300020,300020,Share premium account,account.data_account_type_equity,False,default_chart_template
300030,300030,Revaluation reserve,account.data_account_type_equity,False,default_chart_template
300040,300040,Other reserves,account.data_account_type_equity,False,default_chart_template
300050,300050,Capital,account.data_account_type_equity,False,default_chart_template
300060,300060,Dividends,account.data_account_type_equity,False,default_chart_template
300070,300070,Drawings,account.data_account_type_equity,False,default_chart_template
400010,400010,Undistributed Profits/Losses,account.data_unaffected_earnings,False,default_chart_template
500010,500010,Sales category 1,account.data_account_type_revenue,False,default_chart_template
500020,500020,Sales category 2,account.data_account_type_revenue,False,default_chart_template
500030,500030,Sales category 3,account.data_account_type_revenue,False,default_chart_template
500040,500040,Sales category 4,account.data_account_type_revenue,False,default_chart_template
500050,500050,Bank Interest received,account.data_account_type_revenue,False,default_chart_template
500060,500060,Investment Interest received,account.data_account_type_revenue,False,default_chart_template
500070,500070,Profits/Losses on disposals of assets,account.data_account_type_revenue,False,default_chart_template
500080,500080,Rental Income,account.data_account_type_revenue,False,default_chart_template
510010,510010,Other Income,account.data_account_type_other_income,False,default_chart_template
600010,600010,Cost of sales 1,account.data_account_type_direct_costs,False,default_chart_template
600020,600020,Cost of sales 2,account.data_account_type_direct_costs,False,default_chart_template
600030,600030,Cost of sales 3,account.data_account_type_direct_costs,False,default_chart_template
600040,600040,Cost of sales 4,account.data_account_type_direct_costs,False,default_chart_template
610010,610010,Marketing,account.data_account_type_expenses,False,default_chart_template
610020,610020,Exhibitions and events,account.data_account_type_expenses,False,default_chart_template
610030,610030,PR,account.data_account_type_expenses,False,default_chart_template
610040,610040,Distribution vehicles,account.data_account_type_expenses,False,default_chart_template
610050,610050,Distribution salaries and wages,account.data_account_type_expenses,False,default_chart_template
610060,610060,Shipping,account.data_account_type_expenses,False,default_chart_template
610070,610070,Directors pension,account.data_account_type_expenses,False,default_chart_template
610080,610080,Directors remuneration,account.data_account_type_expenses,False,default_chart_template
610090,610090,Gross Salaries,account.data_account_type_expenses,False,default_chart_template
610100,610100,Employers SDL & UIF,account.data_account_type_expenses,False,default_chart_template
610110,610110,Subcontractors payments,account.data_account_type_expenses,False,default_chart_template
610120,610120,Rent and rates,account.data_account_type_expenses,False,default_chart_template
610130,610130,Light / heat and power,account.data_account_type_expenses,False,default_chart_template
610140,610140,Repairs and maintenance,account.data_account_type_expenses,False,default_chart_template
610150,610150,Car hire,account.data_account_type_expenses,False,default_chart_template
610160,610160,Car fuel,account.data_account_type_expenses,False,default_chart_template
610170,610170,Car maintenance,account.data_account_type_expenses,False,default_chart_template
610180,610180,Telephone,account.data_account_type_expenses,False,default_chart_template
610190,610190,Internet & hosting,account.data_account_type_expenses,False,default_chart_template
610200,610200,Mobiles,account.data_account_type_expenses,False,default_chart_template
610210,610210,Stationery,account.data_account_type_expenses,False,default_chart_template
610220,610220,Office consumables,account.data_account_type_expenses,False,default_chart_template
610230,610230,Postage and Carriage,account.data_account_type_expenses,False,default_chart_template
610240,610240,Books,account.data_account_type_expenses,False,default_chart_template
610250,610250,Network costs,account.data_account_type_expenses,False,default_chart_template
610260,610260,Software expenses,account.data_account_type_expenses,False,default_chart_template
610270,610270,Other computer costs,account.data_account_type_expenses,False,default_chart_template
610280,610280,Recruitment fees,account.data_account_type_expenses,False,default_chart_template
610290,610290,Other admin expenses,account.data_account_type_expenses,False,default_chart_template
610300,610300,Accounting,account.data_account_type_expenses,False,default_chart_template
610310,610310,Auditing,account.data_account_type_expenses,False,default_chart_template
610320,610320,Consultancy,account.data_account_type_expenses,False,default_chart_template
610330,610330,Legal and professional charges,account.data_account_type_expenses,False,default_chart_template
610340,610340,Exchange gains/losses,account.data_account_type_expenses,False,default_chart_template
610350,610350,Other sundry expenses,account.data_account_type_expenses,False,default_chart_template
610360,610360,Bad debts,account.data_account_type_expenses,False,default_chart_template
610370,610370,Interest paid,account.data_account_type_expenses,False,default_chart_template
610380,610380,Bank Charges,account.data_account_type_expenses,False,default_chart_template
610390,610390,Donations,account.data_account_type_expenses,False,default_chart_template
610400,610400,Entertaining,account.data_account_type_expenses,False,default_chart_template
610410,610410,Insurance,account.data_account_type_expenses,False,default_chart_template
610420,610420,Travel and subsistence,account.data_account_type_expenses,False,default_chart_template
610430,610430,Corporation tax expense,account.data_account_type_expenses,False,default_chart_template
610440,610440,Foreign Exchange Gains/Losses,account.data_account_type_expenses,False,default_chart_template
610450,610450,Price Differences Control Account,account.data_account_type_expenses,False,default_chart_template
610460,610460,Cash Register Gains/Losses,account.data_account_type_expenses,False,default_chart_template
620010,620010,Software Depreciation,account.data_account_type_depreciation,False,default_chart_template
620020,620020,Patents & Trademarks Depreciation,account.data_account_type_depreciation,False,default_chart_template
620030,620030,Fixtures and fittings Depreciation,account.data_account_type_depreciation,False,default_chart_template
620040,620040,Land and buildings Depreciation,account.data_account_type_depreciation,False,default_chart_template
620050,620050,Motor vehicles Depreciation,account.data_account_type_depreciation,False,default_chart_template
620060,620060,Office equipment (inc computer equipment) Depreciation,account.data_account_type_depreciation,False,default_chart_template
620070,620070,Plant and machinery Depreciation,account.data_account_type_depreciation,False,default_chart_template
```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_0,VAT 0%
tax_group_1,VAT 15%

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_za.default_chart_template')]"/>
    </function>

</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem id="account_reports_za_statements_menu" name="South Africa" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

    <record id="default_chart_template" model="account.chart.template">
        <field name="name">South African Tax and Account Chart Template (by Paradigm Digital)</field>
        <field name="bank_account_code_prefix">1200</field>
        <field name="cash_account_code_prefix">1250</field>
        <field name="transfer_account_code_prefix">1010</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.ZAR"/>
    </record>

</odoo>

```

## File: data\account_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="default_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="110010"/>
        <field name="property_account_payable_id" ref="220010"/>
        <field name="property_account_expense_categ_id" ref="600010"/>
        <field name="property_account_income_categ_id" ref="500010"/>
        <field name="property_stock_account_input_categ_id" ref="200010"/>
        <field name="property_stock_account_output_categ_id" ref="100050"/>
        <field name="property_stock_valuation_account_id" ref="100020"/>
        <field name="income_currency_exchange_account_id" ref="610440"/>
        <field name="expense_currency_exchange_account_id" ref="610440"/>
        <field name="default_pos_receivable_account_id" ref="110030" />
        <field name="use_anglo_saxon" eval="True"/>
    </record>

</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.za"/>
    </record>

    <record id="total_vat_payable" model="account.tax.report.line">
        <field name="name">[20] VAT PAYABLE/REFUNDABLE (Total A - Total B)</field>
        <field name="formula"> (VAT4 + VAT4A + (SEC6 * 60/100 + SEC7) + VAT11 + VAT12) - (VAT14 + VAT14A + VAT15 + VAT16 + VAT17 + VAT18)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="total_output_tax" model="account.tax.report.line">
        <field name="name">[13] Total A: TOTAL OUTPUT TAX (4 + 4A + 9 + 11 + 12)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="formula">VAT4 + VAT4A + (SEC6 * 60/100 + SEC7) + VAT11 + VAT12</field>
        <field name="parent_id" ref='total_vat_payable'/>
    </record>

    <record id="total_input_tax" model="account.tax.report.line">
        <field name="name">[19] Total B: TOTAL INPUT TAX (14 + 14A + 15 + 16 + 17 + 18)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="formula">VAT14 + VAT14A + VAT15 + VAT16 + VAT17 + VAT18</field>
        <field name="parent_id" ref='total_vat_payable'/>
    </record>

    <record id="standard_rate_exclude_capital_goods_service" model="account.tax.report.line">
        <field name="name">[1] Standard Rate (Excluding Capital goods and/or services and accomodation)</field>
        <field name="tag_name">[1] Standard Rate (Excluding Capital goods and/or services and accomodation)</field>
        <field name="code">VAT1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="vat_on_standard_rate_exclude_capital_goods_service" model="account.tax.report.line">
        <field name="name">[4] x 15/ (100 + 15)</field>
        <field name="tag_name">[4] x 15/ (100 + 15)</field>
        <field name="code">VAT4</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="standard_rate_only_capital_goods_service" model="account.tax.report.line">
        <field name="name">[1A] Standard Rate (Only Capital goods and/or services)</field>
        <field name="tag_name">[1A] Standard Rate (Only Capital goods and/or services)</field>
        <field name="code">VAT1A</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="vat_on_standard_rate_only_capital_goods_service" model="account.tax.report.line">
        <field name="name">[4A] x 15/ (100 + 15)</field>
        <field name="tag_name">[4A] x 15/ (100 + 15)</field>
        <field name="code">VAT4A</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="zero_rate_exclude_goods_exported" model="account.tax.report.line">
        <field name="name">[2] Zero Rate (excluding goods exported)</field>
        <field name="tag_name">[2] Zero Rate (excluding goods exported)</field>
        <field name="code">VAT2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="zero_rate_only_goods_exported" model="account.tax.report.line">
        <field name="name">[2A] Zero Rate (Only goods exported)</field>
        <field name="tag_name">[2A] Zero Rate (Only goods exported)</field>
        <field name="code">VAT2A</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="exempt_and_non_supplies" model="account.tax.report.line">
        <field name="name">[3] Exempt and Non supplies</field>
        <field name="tag_name">[3] Exempt and Non supplies</field>
        <field name="code">VAT3</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="accomodation_exceeding_28_days" model="account.tax.report.line">
        <field name="name">[5] Accomodation exceeding 28 days</field>
        <field name="tag_name">[5] Accomodation exceeding 28 days</field>
        <field name="code">VAT5</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="accomodation_exceeding_28_days_60_percent" model="account.tax.report.line">
        <field name="name">[6] x 60%</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="formula"> VAT5 * 60 / 100</field>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="accomodation_under_28_days" model="account.tax.report.line">
        <field name="name">[7] Accomodation under 28 days</field>
        <field name="tag_name">[7] Accomodation under 28 days</field>
        <field name="code">VAT7</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="accomodation_28_days" model="account.tax.report.line">
        <field name="name">[8] Total (6 + 7)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="14"/>
        <field name="formula">(VAT5 * 60 / 100) + VAT7</field>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="vat_on_accomodation_28_days" model="account.tax.report.line">
        <field name="name">[9] x 15 / (100 ??) </field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="15"/>
        <field name="formula"> SEC6 * 60/100 + SEC7 </field>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="vat_on_accomodation_exceeding_28_days" model="account.tax.report.line">
        <field name="name">VAT on Accomodation exceeding 28 days</field>
        <field name="tag_name">VAT on Accomodation exceeding 28 days</field>
        <field name="code">SEC6</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref='vat_on_accomodation_28_days'/>
    </record>

    <record id="vat_on_accomodation_under_28_days" model="account.tax.report.line">
        <field name="name">VAT on Accomodation under 28 days</field>
        <field name="tag_name">VAT on Accomodation under 28 days</field>
        <field name="code">SEC7</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref='vat_on_accomodation_28_days'/>
    </record>

    <record id="vat_plus_base_of_second_hand_goods_export" model="account.tax.report.line">
        <field name="name">[10] Change in use and export of second-hand goods</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="17"/>
        <field name="formula"> VAT10a + VAT11</field>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="second_hand_goods_export" model="account.tax.report.line">
        <field name="name">[10] Base Amount: Change in use and export of second-hand goods</field>
        <field name="tag_name">[10] Change in use and export of second-hand goods</field>
        <field name="code">VAT10a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref='vat_plus_base_of_second_hand_goods_export'/>
    </record>

    <record id="vat_on_second_hand_goods_export" model="account.tax.report.line">
        <field name="name">[11] x 15 / (100 + 15)</field>
        <field name="tag_name">[11] x 15 / (100 + 15)</field>
        <field name="code">VAT11</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="18"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="other_imported_services" model="account.tax.report.line">
        <field name="name">[12] Other and imported services</field>
        <field name="tag_name">[12] Other and imported services</field>
        <field name="code">VAT12</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="19"/>
        <field name="parent_id" ref='total_output_tax'/>
    </record>

    <record id="capital_goods_services_supplied" model="account.tax.report.line">
        <field name="name">[14] Capital Goods and/or services supplied to you</field>
        <field name="tag_name">[14] Capital Goods and/or services supplied to you</field>
        <field name="code">VAT14</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="21"/>
        <field name="parent_id" ref='total_input_tax'/>
    </record>

    <record id="capital_goods_imported" model="account.tax.report.line">
        <field name="name">[14A] Capital Goods imported by you</field>
        <field name="tag_name">[14A] Capital Goods imported by you</field>
        <field name="code">VAT14A</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="22"/>
        <field name="parent_id" ref='total_input_tax'/>
    </record>

    <record id="other_goods_services_supplied" model="account.tax.report.line">
        <field name="name">[15] Other goods and/or services supplied to you (not Capital Goods)</field>
        <field name="tag_name">[15] Other goods and/or services supplied to you (not Capital Goods)</field>
        <field name="code">VAT15</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="23"/>
        <field name="parent_id" ref='total_input_tax'/>
    </record>

    <record id="other_goods_imported" model="account.tax.report.line">
        <field name="name">[15A] Other goods imported by you (not Capital Goods)</field>
        <field name="tag_name">[15A] Other goods imported by you (not Capital Goods)</field>
        <field name="code">VAT15A</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="24"/>
        <field name="parent_id" ref='total_input_tax'/>
    </record>

    <record id="change_in_use" model="account.tax.report.line">
        <field name="name">[16] Change in Use</field>
        <field name="tag_name">[16] Change in Use</field>
        <field name="code">VAT16</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="25"/>
        <field name="parent_id" ref='total_input_tax'/>
    </record>

    <record id="bad_debts" model="account.tax.report.line">
        <field name="name">[17] Bad Debts</field>
        <field name="tag_name">[17] Bad Debts</field>
        <field name="code">VAT17</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="26"/>
        <field name="parent_id" ref='total_input_tax'/>
    </record>

    <record id="others" model="account.tax.report.line">
        <field name="name">[18] Other</field>
        <field name="tag_name">[18] Other</field>
        <field name="code">VAT18</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="27"/>
        <field name="parent_id" ref='total_input_tax'/>
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="ST1" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Standard Rate</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('standard_rate_exclude_capital_goods_service')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_line_ids': [ref('standard_rate_exclude_capital_goods_service'), ref('vat_on_standard_rate_exclude_capital_goods_service')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('standard_rate_exclude_capital_goods_service')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_line_ids': [ref('standard_rate_exclude_capital_goods_service'), ref('vat_on_standard_rate_exclude_capital_goods_service')],
            }),
        ]"/>
    </record>

    <record id="ST1A" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Standard Rate (Capital Goods)</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('standard_rate_only_capital_goods_service')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_line_ids': [ref('standard_rate_only_capital_goods_service'), ref('vat_on_standard_rate_only_capital_goods_service')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('standard_rate_only_capital_goods_service')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_line_ids': [ref('standard_rate_only_capital_goods_service'), ref('vat_on_standard_rate_only_capital_goods_service')],
            }),
        ]"/>
    </record>

    <record id="ST2" model="account.tax.template">
        <field name="description">0%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Zero Rate</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('zero_rate_exclude_goods_exported')],
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
                'minus_report_line_ids': [ref('zero_rate_exclude_goods_exported')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="ST2A" model="account.tax.template">
        <field name="description">0%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Zero Rate Exports</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('zero_rate_only_goods_exported')],
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
                'minus_report_line_ids': [ref('zero_rate_only_goods_exported')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="ST3" model="account.tax.template">
        <field name="description">0%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Exempt and Non-Supplies</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('exempt_and_non_supplies')],
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
                'minus_report_line_ids': [ref('exempt_and_non_supplies')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="ST5" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Accommodation (28+ days)</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('accomodation_exceeding_28_days')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_line_ids': [ref('vat_on_accomodation_exceeding_28_days')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('accomodation_exceeding_28_days')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_line_ids': [ref('vat_on_accomodation_exceeding_28_days')],
            }),
        ]"/>
    </record>

    <record id="ST7" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Accommodation (Under 28 days)</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('accomodation_under_28_days')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_line_ids': [ref('vat_on_accomodation_under_28_days')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('accomodation_under_28_days')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_line_ids': [ref('vat_on_accomodation_under_28_days')],
            }),
        ]"/>
    </record>

    <record id="ST10" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Export of Second-hand Goods/ Change in Use</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('second_hand_goods_export')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_line_ids': [ref('vat_on_second_hand_goods_export')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('second_hand_goods_export')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_line_ids': [ref('vat_on_second_hand_goods_export')],
            }),
        ]"/>
    </record>

    <record id="ST12" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">VAT Adjustments and Manual VAT</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_line_ids': [ref('other_imported_services')],
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
                'account_id': ref('200060'),
                'minus_report_line_ids': [ref('other_imported_services')],
            }),
        ]"/>
    </record>

    <record id="PT15" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard Rate</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_line_ids': [ref('other_goods_services_supplied')],
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
                'account_id': ref('100060'),
                'minus_report_line_ids': [ref('other_goods_services_supplied')],
            }),
        ]"/>
    </record>

    <record id="PT14" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Standard Rate (Capital Goods)</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_line_ids': [ref('capital_goods_services_supplied')],
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
                'account_id': ref('100060'),
                'minus_report_line_ids': [ref('capital_goods_services_supplied')],
            }),
        ]"/>
    </record>

    <record id="PT14A" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Capital Goods Imported</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_line_ids': [ref('capital_goods_imported')],
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
                'account_id': ref('100060'),
                'minus_report_line_ids': [ref('capital_goods_imported')],
            }),
        ]"/>
    </record>

    <record id="PT15A" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Goods and Services Imported</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_line_ids': [ref('other_goods_imported')],
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
                'account_id': ref('100060'),
                'minus_report_line_ids': [ref('other_goods_imported')],
            }),
        ]"/>
    </record>

    <record id="PT16" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Change in Use</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_line_ids': [ref('change_in_use')],
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
                'account_id': ref('100060'),
                'minus_report_line_ids': [ref('change_in_use')],
            }),
        ]"/>
    </record>

    <record id="PT17" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Bad Debts</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_line_ids': [ref('bad_debts')],
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
                'account_id': ref('100060'),
                'minus_report_line_ids': [ref('bad_debts')],
            }),
        ]"/>
    </record>

    <record id="PT18" model="account.tax.template">
        <field name="description">15%</field>
        <field name="chart_template_id" ref="default_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Other Adjustments</field>
        <field name="amount_type">percent</field>
        <field name="amount">15</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_line_ids': [ref('others')],
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
                'account_id': ref('100060'),
                'minus_report_line_ids': [ref('others')],
            }),
        ]"/>
    </record>

</odoo>

```

