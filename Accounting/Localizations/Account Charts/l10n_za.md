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
id,code,name,account_type,reconcile,chart_template_id/id
100020,100020,Stock Valuation,asset_current,True,default_chart_template
100030,100030,Stock Work In Progress,asset_current,False,default_chart_template
100040,100040,Stock Finished Goods,asset_current,False,default_chart_template
100050,100050,Stock Delivered Control,asset_current,True,default_chart_template
100060,100060,Purchase Tax Control,asset_current,False,default_chart_template
100070,100070,Other Current Assets,asset_current,False,default_chart_template
110010,110010,Debtors Control,asset_receivable,True,default_chart_template
110020,110020,Sundry Debtors,asset_receivable,True,default_chart_template
110030,110030,Debtors Control Account (PoS),asset_receivable,True,default_chart_template
124010,124010,Credit Card Merchant,asset_cash,False,default_chart_template
126010,126010,Cash In Hand,asset_cash,False,default_chart_template
130010,130010,Prepayments,asset_prepayments,False,default_chart_template
140010,140010,Software,asset_fixed,False,default_chart_template
140020,140020,Patents & Trademarks,asset_fixed,False,default_chart_template
140030,140030,Fixtures & Fittings,asset_fixed,False,default_chart_template
140040,140040,Land & Buildings,asset_fixed,False,default_chart_template
140050,140050,Motor Vehicles,asset_fixed,False,default_chart_template
140060,140060,Office Equipment (incl computer equipment),asset_fixed,False,default_chart_template
140070,140070,Plant & Machinery,asset_fixed,False,default_chart_template
150010,150010,Non-current assets,asset_non_current,False,default_chart_template
200010,200010,Stock Received Control,liability_current,True,default_chart_template
200020,200020,Sundry Creditors,liability_current,False,default_chart_template
200030,200030,Other Creditors,liability_current,False,default_chart_template
200040,200040,Accruals,liability_current,False,default_chart_template
200050,200050,Bad debt provision,liability_current,False,default_chart_template
200060,200060,Sales Tax Control,liability_current,False,default_chart_template
200070,200070,Manual Adjustments & VAT,liability_current,False,default_chart_template
200080,200080,Loans,liability_current,False,default_chart_template
200090,200090,Hire Purchase,liability_current,False,default_chart_template
200100,200100,Mortgages,liability_current,False,default_chart_template
210010,210010,Company Credit Card,liability_credit_card,False,default_chart_template
220010,220010,Creditors Control,liability_payable,True,default_chart_template
220020,220020,SARS - VAT,liability_payable,True,default_chart_template
220030,220030,P.A.Y.E. & UIF,liability_payable,True,default_chart_template
220040,220040,Net Wages,liability_payable,True,default_chart_template
220050,220050,Pension Fund,liability_payable,True,default_chart_template
220060,220060,Corporation Tax,liability_payable,True,default_chart_template
300010,300010,Called up share capital,equity,False,default_chart_template
300020,300020,Share premium,equity,False,default_chart_template
300030,300030,Revaluation reserve,equity,False,default_chart_template
300040,300040,Other reserves,equity,False,default_chart_template
300050,300050,Capital,equity,False,default_chart_template
300060,300060,Dividends,equity,False,default_chart_template
300070,300070,Drawings,equity,False,default_chart_template
400010,400010,Undistributed Profits/Losses,equity_unaffected,False,default_chart_template
500010,500010,Sales category 1,income,False,default_chart_template
500020,500020,Sales category 2,income,False,default_chart_template
500030,500030,Sales category 3,income,False,default_chart_template
500040,500040,Sales category 4,income,False,default_chart_template
500050,500050,Bank Interest received,income,False,default_chart_template
500060,500060,Investment Interest received,income,False,default_chart_template
500070,500070,Profits/Losses on disposals of assets,income,False,default_chart_template
500080,500080,Rental Income,income,False,default_chart_template
510010,510010,Other Income,income_other,False,default_chart_template
600010,600010,Cost of sales 1,expense_direct_cost,False,default_chart_template
600020,600020,Cost of sales 2,expense_direct_cost,False,default_chart_template
600030,600030,Cost of sales 3,expense_direct_cost,False,default_chart_template
600040,600040,Cost of sales 4,expense_direct_cost,False,default_chart_template
610010,610010,Marketing,expense,False,default_chart_template
610020,610020,Exhibitions and events,expense,False,default_chart_template
610030,610030,PR,expense,False,default_chart_template
610040,610040,Distribution vehicles,expense,False,default_chart_template
610050,610050,Distribution salaries and wages,expense,False,default_chart_template
610060,610060,Shipping,expense,False,default_chart_template
610070,610070,Directors pension,expense,False,default_chart_template
610080,610080,Directors remuneration,expense,False,default_chart_template
610090,610090,Gross Salaries,expense,False,default_chart_template
610100,610100,Employers SDL & UIF,expense,False,default_chart_template
610110,610110,Subcontractors payments,expense,False,default_chart_template
610120,610120,Rent and rates,expense,False,default_chart_template
610130,610130,Light / heat and power,expense,False,default_chart_template
610140,610140,Repairs and maintenance,expense,False,default_chart_template
610150,610150,Car hire,expense,False,default_chart_template
610160,610160,Car fuel,expense,False,default_chart_template
610170,610170,Car maintenance,expense,False,default_chart_template
610180,610180,Telephone,expense,False,default_chart_template
610190,610190,Internet & hosting,expense,False,default_chart_template
610200,610200,Mobiles,expense,False,default_chart_template
610210,610210,Stationery,expense,False,default_chart_template
610220,610220,Office consumables,expense,False,default_chart_template
610230,610230,Postage and Carriage,expense,False,default_chart_template
610240,610240,Books,expense,False,default_chart_template
610250,610250,Network costs,expense,False,default_chart_template
610260,610260,Software expenses,expense,False,default_chart_template
610270,610270,Other computer costs,expense,False,default_chart_template
610280,610280,Recruitment fees,expense,False,default_chart_template
610290,610290,Other admin expenses,expense,False,default_chart_template
610300,610300,Accounting,expense,False,default_chart_template
610310,610310,Auditing,expense,False,default_chart_template
610320,610320,Consultancy,expense,False,default_chart_template
610330,610330,Legal and professional charges,expense,False,default_chart_template
610340,610340,Exchange gains/losses,expense,False,default_chart_template
610350,610350,Other sundry expenses,expense,False,default_chart_template
610360,610360,Bad debts,expense,False,default_chart_template
610370,610370,Interest paid,expense,False,default_chart_template
610380,610380,Bank Charges,expense,False,default_chart_template
610390,610390,Donations,expense,False,default_chart_template
610400,610400,Entertaining,expense,False,default_chart_template
610410,610410,Insurance,expense,False,default_chart_template
610420,610420,Travel and subsistence,expense,False,default_chart_template
610430,610430,Corporation tax expense,expense,False,default_chart_template
610440,610440,Foreign Exchange Gains/Losses,expense,False,default_chart_template
610450,610450,Price Differences Control,expense,False,default_chart_template
610460,610460,Cash Register Gains/Losses,expense,False,default_chart_template
620010,620010,Software Depreciation,expense_depreciation,False,default_chart_template
620020,620020,Patents & Trademarks Depreciation,expense_depreciation,False,default_chart_template
620030,620030,Fixtures and fittings Depreciation,expense_depreciation,False,default_chart_template
620040,620040,Land and buildings Depreciation,expense_depreciation,False,default_chart_template
620050,620050,Motor vehicles Depreciation,expense_depreciation,False,default_chart_template
620060,620060,Office equipment (inc computer equipment) Depreciation,expense_depreciation,False,default_chart_template
620070,620070,Plant and machinery Depreciation,expense_depreciation,False,default_chart_template

```

## File: data\account.tax.group.csv

```csv
id,name,country_id/id
tax_group_0,VAT 0%,base.za
tax_group_1,VAT 15%,base.za

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

    <record id="default_chart_template" model="account.chart.template">
        <field name="name">South African Tax and Account Chart Template (by Paradigm Digital)</field>
        <field name="bank_account_code_prefix">1200</field>
        <field name="cash_account_code_prefix">1250</field>
        <field name="transfer_account_code_prefix">1010</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.ZAR"/>
        <field name="country_id" ref="base.za"/>
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
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.za"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="total_vat_payable" model="account.report.line">
                <field name="name">[20] VAT PAYABLE/REFUNDABLE (Total A - Total B)</field>
                <field name="aggregation_formula">TotalA.balance - TotalB.balance</field>
                <field name="children_ids">
                    <record id="total_output_tax" model="account.report.line">
                        <field name="name">[13] Total A: TOTAL OUTPUT TAX (4 + 4A + 9 + 11 + 12)</field>
                        <field name="code">TotalA</field>
                        <field name="aggregation_formula">VAT4.balance + VAT4A.balance + (SEC6.balance * 0.6 + SEC7.balance) + VAT11.balance + VAT12.balance</field>
                        <field name="children_ids">
                            <record id="standard_rate_exclude_capital_goods_service" model="account.report.line">
                                <field name="name">[1] Standard Rate (Excluding Capital goods and/or services and accomodation)</field>
                                <field name="code">VAT1</field>
                                <field name="expression_ids">
                                    <record id="standard_rate_exclude_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[1] Standard Rate (Excluding Capital goods and/or services and accomodation)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_on_standard_rate_exclude_capital_goods_service" model="account.report.line">
                                <field name="name">[4] x 15/ (100 + 15)</field>
                                <field name="code">VAT4</field>
                                <field name="expression_ids">
                                    <record id="vat_on_standard_rate_exclude_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[4] x 15/ (100 + 15)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="standard_rate_only_capital_goods_service" model="account.report.line">
                                <field name="name">[1A] Standard Rate (Only Capital goods and/or services)</field>
                                <field name="code">VAT1A</field>
                                <field name="expression_ids">
                                    <record id="standard_rate_only_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[1A] Standard Rate (Only Capital goods and/or services)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_on_standard_rate_only_capital_goods_service" model="account.report.line">
                                <field name="name">[4A] x 15/ (100 + 15)</field>
                                <field name="code">VAT4A</field>
                                <field name="expression_ids">
                                    <record id="vat_on_standard_rate_only_capital_goods_service_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[4A] x 15/ (100 + 15)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="zero_rate_exclude_goods_exported" model="account.report.line">
                                <field name="name">[2] Zero Rate (excluding goods exported)</field>
                                <field name="code">VAT2</field>
                                <field name="expression_ids">
                                    <record id="zero_rate_exclude_goods_exported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[2] Zero Rate (excluding goods exported)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="zero_rate_only_goods_exported" model="account.report.line">
                                <field name="name">[2A] Zero Rate (Only goods exported)</field>
                                <field name="code">VAT2A</field>
                                <field name="expression_ids">
                                    <record id="zero_rate_only_goods_exported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[2A] Zero Rate (Only goods exported)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="exempt_and_non_supplies" model="account.report.line">
                                <field name="name">[3] Exempt and Non supplies</field>
                                <field name="code">VAT3</field>
                                <field name="expression_ids">
                                    <record id="exempt_and_non_supplies_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[3] Exempt and Non supplies</field>
                                    </record>
                                </field>
                            </record>
                            <record id="accomodation_exceeding_28_days" model="account.report.line">
                                <field name="name">[5] Accomodation exceeding 28 days</field>
                                <field name="code">VAT5</field>
                                <field name="expression_ids">
                                    <record id="accomodation_exceeding_28_days_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[5] Accomodation exceeding 28 days</field>
                                    </record>
                                </field>
                            </record>
                            <record id="accomodation_exceeding_28_days_60_percent" model="account.report.line">
                                <field name="name">[6] x 60%</field>
                                <field name="aggregation_formula">VAT5.balance * 0.6</field>
                            </record>
                            <record id="accomodation_under_28_days" model="account.report.line">
                                <field name="name">[7] Accomodation under 28 days</field>
                                <field name="code">VAT7</field>
                                <field name="expression_ids">
                                    <record id="accomodation_under_28_days_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[7] Accomodation under 28 days</field>
                                    </record>
                                </field>
                            </record>
                            <record id="accomodation_28_days" model="account.report.line">
                                <field name="name">[8] Total (6 + 7)</field>
                                <field name="aggregation_formula">(VAT5.balance * 0.6) + VAT7.balance</field>
                            </record>
                            <record id="vat_on_accomodation_28_days" model="account.report.line">
                                <field name="name">[9] x 15 / (100 ??) </field>
                                <field name="aggregation_formula">SEC6.balance * 0.6 + SEC7.balance </field>
                                <field name="children_ids">
                                    <record id="vat_on_accomodation_exceeding_28_days" model="account.report.line">
                                        <field name="name">VAT on Accomodation exceeding 28 days</field>
                                        <field name="code">SEC6</field>
                                        <field name="expression_ids">
                                            <record id="vat_on_accomodation_exceeding_28_days_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">VAT on Accomodation exceeding 28 days</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="vat_on_accomodation_under_28_days" model="account.report.line">
                                        <field name="name">VAT on Accomodation under 28 days</field>
                                        <field name="code">SEC7</field>
                                        <field name="expression_ids">
                                            <record id="vat_on_accomodation_under_28_days_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">VAT on Accomodation under 28 days</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_plus_base_of_second_hand_goods_export" model="account.report.line">
                                <field name="name">[10] Change in use and export of second-hand goods</field>
                                <field name="aggregation_formula">VAT10a.balance + VAT11.balance</field>
                                <field name="children_ids">
                                    <record id="second_hand_goods_export" model="account.report.line">
                                        <field name="name">[10] Base Amount: Change in use and export of second-hand goods</field>
                                        <field name="code">VAT10a</field>
                                        <field name="expression_ids">
                                            <record id="second_hand_goods_export_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">[10] Change in use and export of second-hand goods</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_on_second_hand_goods_export" model="account.report.line">
                                <field name="name">[11] x 15 / (100 + 15)</field>
                                <field name="code">VAT11</field>
                                <field name="expression_ids">
                                    <record id="vat_on_second_hand_goods_export_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[11] x 15 / (100 + 15)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="other_imported_services" model="account.report.line">
                                <field name="name">[12] Other and imported services</field>
                                <field name="code">VAT12</field>
                                <field name="expression_ids">
                                    <record id="other_imported_services_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[12] Other and imported services</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="total_input_tax" model="account.report.line">
                        <field name="name">[19] Total B: TOTAL INPUT TAX (14 + 14A + 15 + 15A + 16 + 17 + 18)</field>
                        <field name="code">TotalB</field>
                        <field name="aggregation_formula">VAT14.balance + VAT14A.balance + VAT15.balance + VAT15A.balance + VAT16.balance + VAT17.balance + VAT18.balance</field>
                        <field name="children_ids">
                            <record id="capital_goods_services_supplied" model="account.report.line">
                                <field name="name">[14] Capital Goods and/or services supplied to you</field>
                                <field name="code">VAT14</field>
                                <field name="expression_ids">
                                    <record id="capital_goods_services_supplied_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[14] Capital Goods and/or services supplied to you</field>
                                    </record>
                                </field>
                            </record>
                            <record id="capital_goods_imported" model="account.report.line">
                                <field name="name">[14A] Capital Goods imported by you</field>
                                <field name="code">VAT14A</field>
                                <field name="expression_ids">
                                    <record id="capital_goods_imported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[14A] Capital Goods imported by you</field>
                                    </record>
                                </field>
                            </record>
                            <record id="other_goods_services_supplied" model="account.report.line">
                                <field name="name">[15] Other goods and/or services supplied to you (not Capital Goods)</field>
                                <field name="code">VAT15</field>
                                <field name="expression_ids">
                                    <record id="other_goods_services_supplied_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[15] Other goods and/or services supplied to you (not Capital Goods)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="other_goods_imported" model="account.report.line">
                                <field name="name">[15A] Other goods imported by you (not Capital Goods)</field>
                                <field name="code">VAT15A</field>
                                <field name="expression_ids">
                                    <record id="other_goods_imported_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[15A] Other goods imported by you (not Capital Goods)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="change_in_use" model="account.report.line">
                                <field name="name">[16] Change in Use</field>
                                <field name="code">VAT16</field>
                                <field name="expression_ids">
                                    <record id="change_in_use_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[16] Change in Use</field>
                                    </record>
                                </field>
                            </record>
                            <record id="bad_debts" model="account.report.line">
                                <field name="name">[17] Bad Debts</field>
                                <field name="code">VAT17</field>
                                <field name="expression_ids">
                                    <record id="bad_debts_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[17] Bad Debts</field>
                                    </record>
                                </field>
                            </record>
                            <record id="others" model="account.report.line">
                                <field name="name">[18] Other</field>
                                <field name="code">VAT18</field>
                                <field name="expression_ids">
                                    <record id="others_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">[18] Other</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('standard_rate_exclude_capital_goods_service_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_expression_ids': [ref('standard_rate_exclude_capital_goods_service_tag'), ref('vat_on_standard_rate_exclude_capital_goods_service_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('standard_rate_exclude_capital_goods_service_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_expression_ids': [ref('standard_rate_exclude_capital_goods_service_tag'), ref('vat_on_standard_rate_exclude_capital_goods_service_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('standard_rate_only_capital_goods_service_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_expression_ids': [ref('standard_rate_only_capital_goods_service_tag'), ref('vat_on_standard_rate_only_capital_goods_service_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('standard_rate_only_capital_goods_service_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_expression_ids': [ref('standard_rate_only_capital_goods_service_tag'), ref('vat_on_standard_rate_only_capital_goods_service_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('zero_rate_exclude_goods_exported_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('zero_rate_exclude_goods_exported_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('zero_rate_only_goods_exported_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('zero_rate_only_goods_exported_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('exempt_and_non_supplies_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('exempt_and_non_supplies_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('accomodation_exceeding_28_days_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_expression_ids': [ref('vat_on_accomodation_exceeding_28_days_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('accomodation_exceeding_28_days_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_expression_ids': [ref('vat_on_accomodation_exceeding_28_days_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('accomodation_under_28_days_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_expression_ids': [ref('vat_on_accomodation_under_28_days_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('accomodation_under_28_days_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_expression_ids': [ref('vat_on_accomodation_under_28_days_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('second_hand_goods_export_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_expression_ids': [ref('vat_on_second_hand_goods_export_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('second_hand_goods_export_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_expression_ids': [ref('vat_on_second_hand_goods_export_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'plus_report_expression_ids': [ref('other_imported_services_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('200060'),
                'minus_report_expression_ids': [ref('other_imported_services_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_expression_ids': [ref('other_goods_services_supplied_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'minus_report_expression_ids': [ref('other_goods_services_supplied_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_expression_ids': [ref('capital_goods_services_supplied_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'minus_report_expression_ids': [ref('capital_goods_services_supplied_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_expression_ids': [ref('capital_goods_imported_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'minus_report_expression_ids': [ref('capital_goods_imported_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_expression_ids': [ref('other_goods_imported_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'minus_report_expression_ids': [ref('other_goods_imported_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_expression_ids': [ref('change_in_use_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'minus_report_expression_ids': [ref('change_in_use_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_expression_ids': [ref('bad_debts_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'minus_report_expression_ids': [ref('bad_debts_tag')],
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'plus_report_expression_ids': [ref('others_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('100060'),
                'minus_report_expression_ids': [ref('others_tag')],
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
    <mask id="b" x="6" y="5.9" width="49.2" height="35.53" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.48" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="900" height="600" transform="translate(6 5.9) scale(0.05 0.06)" xlink:href="data:image/png;base64,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"/>
    </g>
  </g>
</svg>

```

