# Odoo Module: l10n_il

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Israel - Accounting',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest basic Israelian localisation necessary to run Odoo in Israel:
================================================================================

This module consists of:
 - Generic Israel Chart of Accounts
 - Taxes and tax report
 - Multiple Fiscal positions
 """,
    'website': 'http://www.odoo.com/accounting',
    'depends': ['l10n_multilang'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_account_tag.xml',
        'data/account.account.template.csv',
        'data/account.group.template.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/fiscal_templates_data.xml',
        'data/account_chart_template_post_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,account_type,reconcile,tag_ids/id,chart_template_id:id
il_account_100100,100100,Land and buildings,asset_fixed,FALSE,,il_chart_template
il_account_100200,100200,Machinery and equipment,asset_fixed,FALSE,,il_chart_template
il_account_100300,100300,Vehicles,asset_fixed,FALSE,,il_chart_template
il_account_100400,100400,Other property,asset_fixed,FALSE,,il_chart_template
il_account_101110,101110,Stock Valuation,asset_current,FALSE,,il_chart_template
il_account_101120,101120,Stock Interim Account (Received),asset_current,FALSE,,il_chart_template
il_account_101130,101130,Stock Interim Account (Delivered),asset_current,FALSE,,il_chart_template
il_account_101140,101140,Inventory - Raw materials,asset_current,FALSE,,il_chart_template
il_account_101150,101150,Inventory - Work in progress,asset_current,FALSE,,il_chart_template
il_account_101160,101160,Inventory - Finished goods,asset_current,FALSE,,il_chart_template
il_account_101200,101200,Account Receivable,asset_receivable,TRUE,,il_chart_template
il_account_101201,101201,Account Receivable(POS),asset_receivable,TRUE,,il_chart_template
il_account_101250,101250,Allowance for credit losses,asset_current,TRUE,,il_chart_template
il_account_101410,101410,Bank Deposit,asset_cash,FALSE,,il_chart_template
il_account_101420,101420,Cheques,asset_cash,FALSE,,il_chart_template
il_account_101430,101430,Financial Assets,asset_cash,FALSE,,il_chart_template
il_account_101440,101440,Petty Cash,asset_cash,FALSE,,il_chart_template
il_account_101310,101310,VAT - Inputs,asset_current,FALSE,,il_chart_template
il_account_101320,101320,VAT - Fixed Assets,asset_current,FALSE,,il_chart_template
il_account_101330,101330,VAT - Import Transactions,asset_current,FALSE,,il_chart_template
il_account_101340,101340,VAT - PA Import Transactions,asset_current,FALSE,,il_chart_template
il_account_101840,101840,VAT - Inputs import line,asset_current,FALSE,,il_chart_template
il_account_100110,100110,Land and buildings depreciation,asset_non_current,FALSE,,il_chart_template
il_account_100210,100210,Machinery and equipment depreciation,asset_non_current,FALSE,,il_chart_template
il_account_100310,100310,Vehicles depreciation,asset_non_current,FALSE,,il_chart_template
il_account_100410,100410,Other property depreciation,asset_non_current,FALSE,,il_chart_template
il_account_100420,100420,Intangible Assets,asset_non_current,FALSE,,il_chart_template
il_account_101350,101350,Prepayments,asset_current,FALSE,,il_chart_template
il_account_111000,111000,Current Liabilities,liability_current,FALSE,,il_chart_template
il_account_111100,111100,Account Payable,liability_payable,TRUE,,il_chart_template
il_account_111110,111110,VAT Sales,liability_current,FALSE,,il_chart_template
il_account_111120,111120,VAT PA Sales,liability_current,FALSE,,il_chart_template
il_account_111200,111200,Income tax withheld - vendors,liability_current,FALSE,account_tag_retention_tax_vendor_account,il_chart_template
il_account_111210,111210,Income tax withheld - employees,liability_current,FALSE,account_tag_retention_tax_employees_account,il_chart_template
il_account_111220,111220,Income tax withheld - dividends,liability_current,FALSE,account_tag_retention_tax_dividend_account,il_chart_template
il_account_111230,111230,Income tax withheld - customers,liability_current,FALSE,account_tag_retention_tax_customers_account,il_chart_template
il_account_300300,300300,Reserve and Profit/Loss,equity,FALSE,,il_chart_template
il_account_111400,111400,Credit cards,liability_current,FALSE,,il_chart_template
il_account_111450,111450,Short term loans,liability_current,FALSE,,il_chart_template
il_account_111460,111460,Interest Payable,liability_current,TRUE,,il_chart_template
il_account_111500,111500,Employees wages,liability_current,TRUE,,il_chart_template
il_account_111550,111550,Employees Benefits,liability_current,TRUE,,il_chart_template
il_account_112210,112210,Deferred Revenue,liability_non_current,FALSE,,il_chart_template
il_account_111650,111650,Advances,liability_current,TRUE,,il_chart_template
il_account_111660,111660,Other Payable,liability_current,TRUE,,il_chart_template
il_account_111700,111700,Income Tax,liability_current,TRUE,,il_chart_template
il_account_111710,111710,National Insurance,liability_current,TRUE,,il_chart_template
il_account_111720,111720,VAT due,liability_current,TRUE,,il_chart_template
il_account_111730,111730,Deferred Taxes,liability_current,FALSE,,il_chart_template
il_account_112000,112000,Non-current Liabilities,liability_non_current,FALSE,,il_chart_template
il_account_112100,112100,Long Term loans,liability_non_current,FALSE,,il_chart_template
il_account_112150,112150,Provisions,liability_non_current,FALSE,,il_chart_template
il_account_200000,200000,Product Sales,income,FALSE,account.account_tag_operating,il_chart_template
il_account_200100,200100,Services Sales,income,FALSE,account.account_tag_operating,il_chart_template
il_account_200200,200200,Other Income,income_other,FALSE,account.account_tag_operating,il_chart_template
il_account_200300,200300,Interest Income,income_other,FALSE,account.account_tag_operating,il_chart_template
il_account_202000,202000,Other Expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_202100,202100,Foreign Exchange Gain&Loss,expense,FALSE,account.account_tag_financing,il_chart_template
il_account_202200,202200,Interest Expenses,expense,FALSE,account.account_tag_financing,il_chart_template
il_account_202300,202300,Donations,expense,FALSE,,il_chart_template
il_account_202400,202400,Fines,expense,FALSE,,il_chart_template
il_account_201000,201000,Cost of Goods Sold,expense_direct_cost,FALSE,account.account_tag_operating,il_chart_template
il_account_211000,211000,Cost of Services Sold,expense_direct_cost,FALSE,account.account_tag_operating,il_chart_template
il_account_212000,212000,Customer discounts,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_212100,212100,Salary Expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_212200,212200,Purchase of Equipments,expense,FALSE,account.account_tag_investing,il_chart_template
il_account_212300,212300,Bank Fees,expense,FALSE,account.account_tag_financing,il_chart_template
il_account_213000,213000,Customer returns,expense_direct_cost,FALSE,account.account_tag_operating,il_chart_template
il_account_214000,214000,Vendor returns,expense_direct_cost,FALSE,account.account_tag_operating,il_chart_template
il_account_215000,215000,Vendor discounts,expense_direct_cost,FALSE,account.account_tag_operating,il_chart_template
il_account_216000,216000,Good's Shrinkage,expense_direct_cost,FALSE,account.account_tag_operating,il_chart_template
il_account_217000,217000,Lost goods,expense_direct_cost,FALSE,account.account_tag_operating,il_chart_template
il_account_220000,220000,G&A Expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220100,220100,Rent Expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220200,220200,Communication expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220300,220300,Transportation expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220400,220400,Depretiation expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220500,220500,Credit provision expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220600,220600,Sales and marketing expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220700,220700,Other non-operating expenses,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_220800,220800,Bad debts expense,expense,FALSE,account.account_tag_operating,il_chart_template
il_account_300100,300100,Capital,equity,FALSE,,il_chart_template
il_account_300110,300110,Ordinary Shares,equity,FALSE,,il_chart_template
il_account_300120,300120,Preferred Shares,equity,FALSE,,il_chart_template
il_account_300130,300130,Shares Premium,equity,FALSE,,il_chart_template
il_account_300270,300270,current year earnings,equity_unaffected,FALSE,,il_chart_template

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
il_group_100100,100100,100499,"Fixed Assets",l10n_il.il_chart_template
il_group_101110,101110,101400,"Current Assets",l10n_il.il_chart_template
il_group_101401,101401,101799,"Bank And Cash",l10n_il.il_chart_template
il_group_111000,111000,111999,"Current Liabilities",l10n_il.il_chart_template
il_group_112000,112000,112210,"Non-current Liabilities",l10n_il.il_chart_template
il_group_200000,200000,200199,"Sales Income",l10n_il.il_chart_template
il_group_200200,200200,200300,"Other Income",l10n_il.il_chart_template
il_group_201000,201000,201299,"Cost of Goods",l10n_il.il_chart_template
il_group_202000,202000,220900,"Expenses",l10n_il.il_chart_template
il_group_300000,300000,399999,"Capital And Shares",l10n_il.il_chart_template
```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_tag_retention_tax_vendor_account" model="account.account.tag">
        <field name="name">Withholding Vendor Tax Account</field>
    </record>

	<record id="account_tag_dividend_account" model="account.account.tag">
        <field name="name">Dividend Account</field>
    </record>

    <record id="account_tag_retention_tax_dividend_account" model="account.account.tag">
        <field name="name">Withholding Dividend Tax Account</field>
    </record>

    <record id="account_tag_retention_tax_employees_account" model="account.account.tag">
        <field name="name">Withholding Employees Tax Account</field>
    </record>

    <record id="account_tag_retention_tax_customers_account" model="account.account.tag">
        <field name="name">Withholding Customers Tax Account</field>
    </record>

</odoo>
```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_il.il_chart_template')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="il_chart_template" model="account.chart.template">
        <field name="name">Israel - Chart of Accounts</field>
        <field name="bank_account_code_prefix">1014</field>
        <field name="cash_account_code_prefix">1015</field>
        <field name="transfer_account_code_prefix">1017</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.ILS"/>
        <field name="spoken_languages" eval="'he_IL'"/>
        <field name="country_id" ref="base.il"/>
    </record>
</odoo>

```

## File: data\account_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="il_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="il_account_101200"/>
        <field name="property_account_payable_id" ref="il_account_111100"/>
        <field name="property_account_expense_categ_id" ref="il_account_212200"/>
        <field name="property_account_income_categ_id" ref="il_account_200000"/>
        <field name="property_stock_account_input_categ_id" ref="il_account_101120"/>
        <field name="property_stock_account_output_categ_id" ref="il_account_101130"/>
        <field name="property_stock_valuation_account_id" ref="il_account_101110"/>
        <field name="income_currency_exchange_account_id" ref="il_account_201000"/>
        <field name="expense_currency_exchange_account_id" ref="il_account_202100"/>
        <field name="default_pos_receivable_account_id" ref="il_account_101201"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="tax_group_vat_16" model="account.tax.group">
        <field name="name">VAT 16%</field>
        <field name="country_id" ref="base.il"/>
    </record>
    <record id="tax_group_vat_17" model="account.tax.group">
        <field name="name">VAT 17%</field>
        <field name="country_id" ref="base.il"/>
    </record>
    <record id="tax_group_vat_exempt" model="account.tax.group">
        <field name="name">VAT exempt sale</field>
        <field name="country_id" ref="base.il"/>
    </record>
    <record id="tax_group_vat_exempt_purchase" model="account.tax.group">
        <field name="name">VAT exempt purchase</field>
        <field name="country_id" ref="base.il"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="vat_report" model="account.report">
        <field name="name">VAT Report (PCN874)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.il"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="vat_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_out_base_title" model="account.report.line">
                <field name="name">VAT SALES (BASE)</field>
                <field name="aggregation_formula">ILTAX_OUT_BASE.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_out_base" model="account.report.line">
                        <field name="name">VAT SALES (BASE)</field>
                        <field name="code">ILTAX_OUT_BASE</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT SALES (BASE)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_vat_sales_tax" model="account.report.line">
                <field name="name">VAT SALES (TAX)</field>
                <field name="code">ILTAX_OUT_BALANCE</field>
                <field name="aggregation_formula">ILTAX_OUT_BALANCE_00.balance+ILTAX_OUT_BALANCE_PA.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_out_balance" model="account.report.line">
                        <field name="name">VAT Sales</field>
                        <field name="code">ILTAX_OUT_BALANCE_00</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_balance_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Sales</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_out_balance_pa" model="account.report.line">
                        <field name="name">VAT PA Sales</field>
                        <field name="code">ILTAX_OUT_BALANCE_PA</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_balance_pa_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT PA Sales</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_out_base_exempt_title" model="account.report.line">
                <field name="name">VAT Exempt Sales (BASE)</field>
                <field name="aggregation_formula">ILTAX_OUT_BASE_exempt.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_out_base_exempt" model="account.report.line">
                        <field name="name">VAT Exempt Sales (BASE)</field>
                        <field name="code">ILTAX_OUT_BASE_exempt</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_out_base_exempt_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Exempt Sales (BASE)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_in_balance" model="account.report.line">
                <field name="name">VAT INPUTS (TAX)</field>
                <field name="code">ILTAX_IN_BALANCE</field>
                <field name="aggregation_formula">ILTAX_IN_BALANCE_17.balance+ILTAX_IN_BALANCE_2_3.balance+ILTAX_IN_BALANCE_1_4.balance+ILTAX_IN_BALANCE_PA.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_in_balance_17" model="account.report.line">
                        <field name="name">VAT Inputs 17%</field>
                        <field name="code">ILTAX_IN_BALANCE_17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs 17%</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_in_balance_pa_16" model="account.report.line">
                        <field name="name">VAT Inputs PA 16%</field>
                        <field name="code">ILTAX_IN_BALANCE_PA</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_pa_16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs PA 16%</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_in_balance_2_3" model="account.report.line">
                        <field name="name">VAT Inputs 2/3</field>
                        <field name="code">ILTAX_IN_BALANCE_2_3</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_2_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs 2/3</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_in_balance_1_4" model="account.report.line">
                        <field name="name">VAT Inputs 1/4</field>
                        <field name="code">ILTAX_IN_BALANCE_1_4</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_in_balance_1_4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Inputs 1/4</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_vat_in_fa_title" model="account.report.line">
                <field name="name">VAT INPUTS (fixed assets)</field>
                <field name="aggregation_formula">ILTAX_VAT_IN_FA.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_vat_in_fa" model="account.report.line">
                        <field name="name">VAT INPUTS (fixed assets)</field>
                        <field name="code">ILTAX_VAT_IN_FA</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_vat_in_fa_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT INPUTS (fixed assets)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_vat_due" model="account.report.line">
                <field name="name">VAT DUE</field>
                <field name="code">ILTAX_VAT_DUE</field>
                <field name="aggregation_formula">(ILTAX_OUT_BALANCE_00.balance + ILTAX_OUT_BALANCE_PA.balance) - (ILTAX_IN_BALANCE_17.balance + ILTAX_IN_BALANCE_2_3.balance + ILTAX_IN_BALANCE_1_4.balance + ILTAX_IN_BALANCE_PA.balance) - ILTAX_VAT_IN_FA.balance</field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Sales Taxes -->
    <record id="il_vat_sales_17" model="account.tax.template">
        <field name="sequence">1</field>
        <field name="description">17%</field>
        <field name="name">VAT Sales</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_line_out_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
                'plus_report_expression_ids': [ref('account_tax_report_line_out_balance_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_line_out_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
                'minus_report_expression_ids': [ref('account_tax_report_line_out_balance_tag')],
            }),
        ]"/>
    </record>
    <record id="il_vat_pa_sales_17" model="account.tax.template">
        <field name="sequence">8</field>
        <field name="description">17%</field>
        <field name="name">VAT PA Sales</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_line_out_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111120'),
                'plus_report_expression_ids': [ref('account_tax_report_line_out_balance_pa_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_line_out_base_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111120'),
                'minus_report_expression_ids': [ref('account_tax_report_line_out_balance_pa_tag')],
            }),
        ]"/>
    </record>
    <record id="il_vat_sales_exempt" model="account.tax.template">
        <field name="sequence">9</field>
        <field name="description">0%</field>
        <field name="name">VAT exempt sales</field>
        <field name="price_include" eval="1"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_exempt"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_line_out_base_exempt_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_line_out_base_exempt_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
            }),
        ]"/>
    </record>
    <record id="il_vat_self_inv_purchase" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="description">17%</field>
        <field name="name">Self Invoice</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_line_out_base_tag')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'plus_report_expression_ids': [ref('account_tax_report_line_in_balance_17_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
                'minus_report_expression_ids': [ref('account_tax_report_line_out_balance_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_line_out_base_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'minus_report_expression_ids': [ref('account_tax_report_line_in_balance_17_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
                'plus_report_expression_ids': [ref('account_tax_report_line_out_balance_tag')],
            }),
        ]"/>
    </record>
    <!-- Purchase Taxes -->
    <record id="il_vat_inputs_17" model="account.tax.template">
        <field name="sequence">2</field>
        <field name="description">17%</field>
        <field name="name">VAT inputs</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'plus_report_expression_ids': [ref('account_tax_report_line_in_balance_17_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'minus_report_expression_ids': [ref('account_tax_report_line_in_balance_17_tag')],
            }),
        ]"/>
    </record>
    <record id="il_vat_pa_purchase_16" model="account.tax.template">
        <field name="sequence">3</field>
        <field name="description">16%</field>
        <field name="name">VAT 16% (PA)</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101340'),
                'plus_report_expression_ids': [ref('account_tax_report_line_in_balance_pa_16_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101340'),
                'minus_report_expression_ids': [ref('account_tax_report_line_in_balance_pa_16_tag')],
            }),
        ]"/>
    </record>
    <record id="il_vat_inputs_2_3_17" model="account.tax.template">
        <field name="sequence">4</field>
        <field name="description">17%</field>
        <field name="name">VAT Inputs 2/3</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 66.67,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'plus_report_expression_ids': [ref('account_tax_report_line_in_balance_2_3_tag')],
            }),
            (0,0, {
                'factor_percent': 33.33,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 66.67,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'minus_report_expression_ids': [ref('account_tax_report_line_in_balance_2_3_tag')],
            }),
            (0,0, {
                'factor_percent': 33.33,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="il_vat_inputs_1_4_17" model="account.tax.template">
        <field name="sequence">5</field>
        <field name="description">17%</field>
        <field name="name">VAT Inputs 1/4</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 75,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'plus_report_expression_ids': [ref('account_tax_report_line_in_balance_1_4_tag')],
            }),
            (0,0, {
                'factor_percent': 25,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 75,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
                'minus_report_expression_ids': [ref('account_tax_report_line_in_balance_1_4_tag')],
            }),
            (0,0, {
                'factor_percent': 25,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="il_vat_inputs_fa_17" model="account.tax.template">
        <field name="sequence">6</field>
        <field name="description">17%</field>
        <field name="name">VAT inputs for fixed assets</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101320'),
                'plus_report_expression_ids': [ref('account_tax_report_line_vat_in_fa_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101320'),
                'minus_report_expression_ids': [ref('account_tax_report_line_vat_in_fa_tag')],
            }),
        ]"/>
    </record>
    <record id="il_vat_purchase_exempt" model="account.tax.template">
        <field name="sequence">7</field>
        <field name="description">0%</field>
        <field name="name">VAT exempt purchase</field>
        <field name="amount">0</field>
        <field name="price_include" eval="1"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_exempt_purchase"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
            }),
        ]"/>
    </record>
    <record id="il_vat_only_purchase" model="account.tax.template">
        <field name="sequence">17</field>
        <field name="description">0%</field>
        <field name="name">VAT Import Line</field>
        <field name="description">VAT Import Line</field>
        <field name="amount">100</field>
        <field name="amount_type">division</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="price_include" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101840'),
                'plus_report_expression_ids': [ref('account_tax_report_line_in_balance_17_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101840'),
                'minus_report_expression_ids': [ref('account_tax_report_line_in_balance_17_tag')],
            }),
        ]"/>
    </record>
    <record id="il_vat_purchase_zero" model="account.tax.template">
        <field name="sequence">7</field>
        <field name="description">0%</field>
        <field name="name">VAT Zero</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="price_include" eval="1"/>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_exempt_purchase"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_101310'),
            }),
        ]"/>
    </record>
    <record id="il_vat_sales_zero" model="account.tax.template">
        <field name="sequence">9</field>
        <field name="description">0%</field>
        <field name="name">VAT Zero</field>
        <field name="price_include" eval="1"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_exempt"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_line_out_base_exempt_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_line_out_base_exempt_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\fiscal_templates_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Israel keep standard taxes -->
    <record id="account_fiscal_position_israel" model="account.fiscal.position.template">
        <field name="name">Israel</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="country_id" ref="base.il"/>
        <field name="auto_apply" eval="True"/>
    </record>

    <!-- Palestinian Authority (PA) -->
    <record id="account_fiscal_position_palestinian_authority" model="account.fiscal.position.template">
        <field name="name">Palestinian Authority (PA)</field>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ps"/>
        <field name="chart_template_id" ref="il_chart_template"/>
    </record>

    <record id="account_fiscal_position_palestinian_authority_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_sales_17"/>
        <field name="tax_dest_id" ref="il_vat_pa_sales_17"/>
        <field name="position_id" ref="account_fiscal_position_palestinian_authority"/>
    </record>

    <record id="account_fiscal_position_palestinian_authority_02" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_inputs_17"/>
        <field name="tax_dest_id" ref="il_vat_pa_purchase_16"/>
        <field name="position_id" ref="account_fiscal_position_palestinian_authority"/>
    </record>

    <!-- Import / Export -->
    <record id="account_fiscal_position_import_export" model="account.fiscal.position.template">
        <field name="name">Import / Export</field>
        <field name="auto_apply" eval="True"/>
        <field name="chart_template_id" ref="il_chart_template"/>
    </record>

    <record id="account_fiscal_position_import_export_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_sales_17"/>
        <field name="tax_dest_id" ref="il_vat_sales_exempt"/>
        <field name="position_id" ref="account_fiscal_position_import_export"/>
    </record>

    <record id="account_fiscal_position_import_export_02" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_inputs_17"/>
        <field name="position_id" ref="account_fiscal_position_import_export"/>
    </record>

    <!-- Eilat City -->
    <record id="account_fiscal_position_eilat" model="account.fiscal.position.template">
        <field name="name">Eilat</field>
        <field name="chart_template_id" ref="il_chart_template"/>
    </record>

    <record id="account_fiscal_position_eilat_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_sales_17"/>
        <field name="tax_dest_id" ref="il_vat_sales_exempt"/>
        <field name="position_id" ref="account_fiscal_position_eilat"/>
    </record>

    <record id="account_fiscal_position_eilat_02" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_inputs_17"/>
        <field name="tax_dest_id" ref="il_vat_purchase_exempt"/>
        <field name="position_id" ref="account_fiscal_position_eilat"/>
    </record>

    <!-- Vat Zero -->
    <record id="account_fiscal_position_vat_zero" model="account.fiscal.position.template">
        <field name="name">Vat Zero</field>
        <field name="chart_template_id" ref="il_chart_template"/>
    </record>

    <record id="account_fiscal_position_vat_zero_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_sales_17"/>
        <field name="tax_dest_id" ref="il_vat_sales_zero"/>
        <field name="position_id" ref="account_fiscal_position_vat_zero"/>
    </record>

    <record id="account_fiscal_position_vat_zero_02" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_inputs_17"/>
        <field name="tax_dest_id" ref="il_vat_purchase_zero"/>
        <field name="position_id" ref="account_fiscal_position_vat_zero"/>
    </record>

    <!-- Self Invoice -->
    <record id="account_fiscal_position_self_invoice" model="account.fiscal.position.template">
        <field name="name">Self Invoice</field>
        <field name="chart_template_id" ref="il_chart_template"/>
    </record>

    <record id="account_fiscal_position_self_invoice_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_inputs_17"/>
        <field name="tax_dest_id" ref="il_vat_self_inv_purchase"/>
        <field name="position_id" ref="account_fiscal_position_self_invoice"/>
    </record>
    
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106"><defs><mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse"><path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill:#fff;fill-rule:evenodd"/></mask><mask id="b" x="5.75" y="7.26" width="49.48" height="36" maskUnits="userSpaceOnUse"><rect x="6.15" y="10.78" width="48.45" height="28.75" rx="1" style="fill:#fff"/></mask><symbol id="c" viewBox="0 0 106 106"><g style="mask:url(#a)"><path d="M0,0H106V106H0Z" style="fill:#5a5a64;fill-rule:evenodd"/><path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill:#fff;fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill:#393939;fill-rule:evenodd;opacity:0.324000000953674;isolation:isolate"/><g style="opacity:0.30000000000000004"><path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/><path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/><path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/><path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/><path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/><path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/><path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/><path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/></g><path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill:#a8a9ab"/><path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill:#a8a9ab"/><path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill:#a8a9ab"/><path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill:#a8a9ab"/><path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill:#a8a9ab"/><path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill:#a8a9ab"/><path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill:#a8a9ab"/><path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill:#a8a9ab"/></g></symbol></defs><use width="106" height="106" xlink:href="#c"/><rect x="6.27" y="10.57" width="48.45" height="31.57" rx="1" style="fill:#393939;opacity:0.44;isolation:isolate"/><g style="mask:url(#b)"><path d="M5.75,34.26v-18H55.24v18H5.75ZM37.23,21.37H32.89a.24.24,0,0,1-.23-.12C32,20,31.27,18.83,30.57,17.62l-.08-.13-.07.13c-.7,1.21-1.4,2.41-2.09,3.62a.23.23,0,0,1-.23.13H23.77l.07.14c.7,1.22,1.4,2.43,2.11,3.64a.2.2,0,0,1,0,.22L23.84,29s0,.09-.07.14H28.1a.25.25,0,0,1,.23.13c.69,1.21,1.39,2.42,2.09,3.62l.08.13.08-.14,2.07-3.59a.26.26,0,0,1,.26-.15h4.32L37.14,29c-.7-1.21-1.39-2.42-2.1-3.63a.22.22,0,0,1,0-.22c.59-1,1.17-2,1.75-3Z" style="fill:#fff"/><path d="M55.24,16.27H5.75V10.65H55.24Z" style="fill:#0038b8"/><path d="M5.75,34.26H55.24v5.61H5.75Z" style="fill:#0038b8"/><path d="M55.24,10.65H5.75V7.26H55.24Z" style="fill:#fff"/><path d="M5.75,39.87H55.24v3.39H5.75Z" style="fill:#fff"/><path d="M37.23,21.37l-.44.77c-.58,1-1.16,2-1.75,3a.22.22,0,0,0,0,.22c.71,1.21,1.4,2.42,2.1,3.63l.09.15H32.91a.26.26,0,0,0-.26.15l-2.07,3.59L30.5,33l-.08-.13c-.7-1.2-1.4-2.41-2.09-3.62a.25.25,0,0,0-.23-.13H23.77s.05-.1.07-.14l2.1-3.64a.2.2,0,0,0,0-.22c-.71-1.21-1.41-2.42-2.11-3.64l-.07-.14H28.1a.23.23,0,0,0,.23-.13c.69-1.21,1.39-2.41,2.09-3.62l.07-.13.08.13c.7,1.21,1.4,2.41,2.09,3.63a.24.24,0,0,0,.23.12h4.34ZM30.5,27.93h1.43a.15.15,0,0,0,.16-.09l1.44-2.49a.17.17,0,0,0,0-.17l-1.44-2.5a.17.17,0,0,0-.16-.09c-.95,0-1.91,0-2.87,0a.16.16,0,0,0-.16.1c-.48.83-.95,1.66-1.43,2.48a.15.15,0,0,0,0,.18l1.43,2.48a.16.16,0,0,0,.16.1Zm3.78-1.45-.83,1.44h1.66Zm-3-5.11-.83-1.44-.84,1.44ZM30.5,30.6l.82-1.44H29.67Zm-3-8H25.88L26.71,24Zm5.91,0L34.28,24l.83-1.44Zm-7.57,5.32h1.66l-.83-1.44Z" style="fill:#0038b8"/><path d="M30.5,27.93H29.06a.16.16,0,0,1-.16-.1l-1.43-2.48a.15.15,0,0,1,0-.18c.48-.82,1-1.65,1.43-2.48a.16.16,0,0,1,.16-.1c1,0,1.92,0,2.87,0a.17.17,0,0,1,.16.09l1.44,2.5a.17.17,0,0,1,0,.17l-1.44,2.49a.15.15,0,0,1-.16.09Z" style="fill:#fff"/><path d="M34.28,26.48l.83,1.44H33.45Z" style="fill:#fff"/><path d="M31.33,21.37H29.66l.84-1.44Z" style="fill:#fff"/><path d="M30.5,30.6l-.83-1.44h1.65Z" style="fill:#fff"/><path d="M27.54,22.6,26.71,24l-.83-1.44Z" style="fill:#fff"/><path d="M33.45,22.6h1.66L34.28,24Z" style="fill:#fff"/><path d="M25.88,27.92l.83-1.44.83,1.44Z" style="fill:#fff"/></g></svg>
```

