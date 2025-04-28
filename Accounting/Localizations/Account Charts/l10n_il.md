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
 - Fiscal position for Palestina
 """,
    'website': 'http://www.odoo.com/accounting',
    'depends': ['account'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account_account_tag.xml',
        'data/account.account.template.csv',
        'data/account_data.xml',
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
id,code,name,user_type_id/id,reconcile,tag_ids/id,chart_template_id:id
il_account_100100,100100,Land and buildings,account.data_account_type_fixed_assets,FALSE,,il_chart_template
il_account_100200,100200,Machinery and equipment,account.data_account_type_fixed_assets,FALSE,,il_chart_template
il_account_100300,100300,Vehicles,account.data_account_type_fixed_assets,FALSE,,il_chart_template
il_account_100400,100400,Other property,account.data_account_type_fixed_assets,FALSE,,il_chart_template
il_account_101110,101110,Stock Valuation account,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101120,101120,Stock Interim Account (Received),account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101130,101130,Stock Interim Account (Delivered),account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101140,101140,Inventory - Raw materials,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101150,101150,Inventory - Work in progress,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101160,101160,Inventory - Finished goods,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101200,101200,Account Receivable,account.data_account_type_receivable,TRUE,,il_chart_template
il_account_101201,101201,Account Receivable(POS),account.data_account_type_receivable,TRUE,,il_chart_template
il_account_101250,101250,Allowance for credit losses,account.data_account_type_current_assets,TRUE,,il_chart_template
il_account_101300,101300,Income tax withheld - customers,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101410,101410,Bank Deposit,account.data_account_type_liquidity,FALSE,,il_chart_template
il_account_101420,101420,Cheques,account.data_account_type_liquidity,FALSE,,il_chart_template
il_account_101430,101430,Financial Assets,account.data_account_type_liquidity,FALSE,,il_chart_template
il_account_101440,101440,Petty Cash,account.data_account_type_liquidity,FALSE,,il_chart_template
il_account_101800,101800,VAT - Inputs,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101810,101810,VAT - Fixed Assets,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101820,101820,VAT - Import Transactions,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_101830,101830,VAT - PA Import Transactions,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_102110,102110,Land and buildings depreciation,account.data_account_type_non_current_assets,FALSE,,il_chart_template
il_account_102120,102120,Machinery and equipment depreciation,account.data_account_type_non_current_assets,FALSE,,il_chart_template
il_account_102130,102130,Vehicles depreciation,account.data_account_type_non_current_assets,FALSE,,il_chart_template
il_account_102140,102140,Other property depreciation,account.data_account_type_non_current_assets,FALSE,,il_chart_template
il_account_102150,102150,Accumulated depreciation,account.data_account_type_non_current_assets,FALSE,,il_chart_template
il_account_102200,102200,Intangible Assets,account.data_account_type_non_current_assets,FALSE,,il_chart_template
il_account_103000,103000,Prepayments,account.data_account_type_current_assets,FALSE,,il_chart_template
il_account_111000,111000,Current Liabilities,account.data_account_type_current_liabilities,FALSE,,il_chart_template
il_account_111100,111100,Account Payable,account.data_account_type_payable,TRUE,,il_chart_template
il_account_111110,111110,VAT Sales,account.data_account_type_current_liabilities,FALSE,,il_chart_template
il_account_111120,111120,VAT PA Sales,account.data_account_type_current_liabilities,FALSE,,il_chart_template
il_account_111200,111200,Income tax withheld - vendors,account.data_account_type_current_liabilities,FALSE,account_tag_retention_tax_vendor_account,il_chart_template
il_account_111210,111210,Income tax withheld - employees,account.data_account_type_current_liabilities,FALSE,,il_chart_template
il_account_111220,111220,Income tax withheld - dividends,account.data_account_type_current_liabilities,FALSE,account_tag_retention_tax_dividend_account,il_chart_template
il_account_111300,111300,Reserve and Profit/Loss Account,account.data_account_type_equity,FALSE,,il_chart_template
il_account_111400,111400,Credit cards,account.data_account_type_current_liabilities,FALSE,,il_chart_template
il_account_111450,111450,Short term loans,account.data_account_type_current_liabilities,FALSE,,il_chart_template
il_account_111460,111460,Interest Payable,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111500,111500,Employees wages,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111550,111550,Employees Benefits,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111600,111600,Deferred Revenue,account.data_account_type_non_current_liabilities,FALSE,,il_chart_template
il_account_111650,111650,Advances,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111660,111660,Other Payable,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111700,111700,Income Tax,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111710,111710,National Insurance,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111720,111720,VAT due,account.data_account_type_current_liabilities,TRUE,,il_chart_template
il_account_111730,111730,Deferred Taxes,account.data_account_type_current_liabilities,FALSE,,il_chart_template
il_account_112000,112000,Non-current Liabilities,account.data_account_type_non_current_liabilities,FALSE,,il_chart_template
il_account_112100,112100,Long Term loans,account.data_account_type_non_current_liabilities,FALSE,,il_chart_template
il_account_112150,112150,Provisions,account.data_account_type_non_current_liabilities,FALSE,,il_chart_template
il_account_200000,200000,Product Sales,account.data_account_type_revenue,FALSE,account.account_tag_operating,il_chart_template
il_account_200100,200100,Services Sales,account.data_account_type_revenue,FALSE,account.account_tag_operating,il_chart_template
il_account_200200,200200,Other Income,account.data_account_type_other_income,FALSE,account.account_tag_operating,il_chart_template
il_account_200300,200300,Interest Income,account.data_account_type_other_income,FALSE,account.account_tag_operating,il_chart_template
il_account_201000,201000,Foreign Exchange Gain,account.data_account_type_other_income,FALSE,account.account_tag_financing,il_chart_template
il_account_202000,202000,Other Expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_202100,202100,Foreign Exchange Loss,account.data_account_type_expenses,FALSE,account.account_tag_financing,il_chart_template
il_account_202200,202200,Interest Expenses,account.data_account_type_expenses,FALSE,account.account_tag_financing,il_chart_template
il_account_202300,202300,Donations,account.data_account_type_expenses,FALSE,,il_chart_template
il_account_202400,202400,Fines,account.data_account_type_expenses,FALSE,,il_chart_template
il_account_210000,210000,Cost of Goods Sold,account.data_account_type_direct_costs,FALSE,account.account_tag_operating,il_chart_template
il_account_211000,211000,Cost of Services Sold,account.data_account_type_direct_costs,FALSE,account.account_tag_operating,il_chart_template
il_account_212000,212000,Customer discounts,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_212100,212100,Salary Expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_212200,212200,Purchase of Equipments,account.data_account_type_expenses,FALSE,account.account_tag_investing,il_chart_template
il_account_212300,212300,Bank Fees,account.data_account_type_expenses,FALSE,account.account_tag_financing,il_chart_template
il_account_213000,213000,Customer returns,account.data_account_type_direct_costs,FALSE,account.account_tag_operating,il_chart_template
il_account_214000,214000,Vendor returns,account.data_account_type_direct_costs,FALSE,account.account_tag_operating,il_chart_template
il_account_215000,215000,Vendor discounts,account.data_account_type_direct_costs,FALSE,account.account_tag_operating,il_chart_template
il_account_216000,216000,Good's Shrinkage,account.data_account_type_direct_costs,FALSE,account.account_tag_operating,il_chart_template
il_account_217000,217000,Lost goods,account.data_account_type_direct_costs,FALSE,account.account_tag_operating,il_chart_template
il_account_220000,220000,G&A Expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220100,220100,Rent Expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220200,220200,Communication expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220300,220300,Transportation expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220400,220400,Depretiation expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220500,220500,Credit provision expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220600,220600,Sales and marketing expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220700,220700,Other non-operating expenses,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_220800,220800,Bad debts expense,account.data_account_type_expenses,FALSE,account.account_tag_operating,il_chart_template
il_account_300100,300100,Capital,account.data_account_type_equity,FALSE,,il_chart_template
il_account_300110,300110,Ordinary Shares,account.data_account_type_equity,FALSE,,il_chart_template
il_account_300120,300120,Preferred Shares,account.data_account_type_equity,FALSE,,il_chart_template
il_account_300130,300130,Shares Premium,account.data_account_type_equity,FALSE,,il_chart_template
il_account_300200,300200,Dividends,account.data_account_type_equity,FALSE,account_tag_dividend_account,il_chart_template
il_account_300270,300270,current year earnings,account.data_unaffected_earnings,FALSE,,il_chart_template

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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="tax_group_vat_16" model="account.tax.group">
        <field name="name">VAT 16%</field>
    </record>

    <record id="tax_group_vat_17" model="account.tax.group">
        <field name="name">VAT 17%</field>
    </record>

    <record id="tax_group_vat_exempt" model="account.tax.group">
        <field name="name">VAT exempt</field>
    </record>

    <record id="tax_group_retention_purchase" model="account.tax.group">
        <field name="name">Withholding purchase</field>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="vat_report" model="account.tax.report">
        <field name="name">VAT Report (PCN874)</field>
        <field name="country_id" ref="base.il"/>
    </record>

    <!-- SALES lines -->

    <record id="account_tax_report_line_out_base_title" model="account.tax.report.line">
        <field name="name">VAT SALES (BASE)</field>
        <field name="sequence">1</field>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_out_base" model="account.tax.report.line">
        <field name="name">VAT SALES (BASE)</field>
        <field name="tag_name">VAT SALES (BASE)</field>
        <field name="code">ILTAX_OUT_BASE</field>
        <field name="sequence">1</field>
        <field name="parent_id" ref="account_tax_report_line_out_base_title"/>
        <field name="report_id" ref="vat_report"/>
    </record>

	<record id="account_tax_report_line_vat_sales_tax" model="account.tax.report.line">
        <field name="name">VAT SALES (TAX)</field>
        <field name="formula">ILTAX_OUT_BALANCE_00+ILTAX_OUT_BALANCE_PA</field>
        <field name="code">ILTAX_OUT_BALANCE</field>
        <field name="sequence">2</field>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_out_balance" model="account.tax.report.line">
        <field name="name">VAT Sales</field>
        <field name="tag_name">VAT Sales</field>
        <field name="code">ILTAX_OUT_BALANCE_00</field>
        <field name="sequence">21</field>
        <field name="parent_id" ref="account_tax_report_line_vat_sales_tax"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_out_balance_pa" model="account.tax.report.line">
        <field name="name">VAT PA Sales</field>
        <field name="tag_name">VAT PA Sales</field>
        <field name="code">ILTAX_OUT_BALANCE_PA</field>
        <field name="sequence">22</field>
        <field name="parent_id" ref="account_tax_report_line_vat_sales_tax"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_out_base_exempt_title" model="account.tax.report.line">
        <field name="name">VAT Exempt Sales (BASE)</field>
        <field name="sequence">3</field>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_out_base_exempt" model="account.tax.report.line">
        <field name="name">VAT Exempt Sales (BASE)</field>
        <field name="tag_name">VAT Exempt Sales (BASE)</field>
        <field name="code">ILTAX_OUT_BASE_exempt</field>
        <field name="sequence">3</field>
        <field name="parent_id" ref="account_tax_report_line_out_base_exempt_title"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <!-- PURCHASE lines -->
    <record id="account_tax_report_line_in_balance" model="account.tax.report.line">
        <field name="name">VAT INPUTS (TAX)</field>
        <field name="formula">ILTAX_IN_BALANCE_17+ILTAX_IN_BALANCE_2_3+ILTAX_IN_BALANCE_1_4+ILTAX_IN_BALANCE_PA</field>
        <field name="code">ILTAX_IN_BALANCE</field>
        <field name="sequence">5</field>
        <field name="report_id" ref="vat_report"/>
    </record>

	<record id="account_tax_report_line_in_balance_17" model="account.tax.report.line">
        <field name="name">VAT Inputs 17%</field>
        <field name="tag_name">VAT Inputs 17%</field>
        <field name="code">ILTAX_IN_BALANCE_17</field>
        <field name="sequence">51</field>
        <field name="parent_id" ref="account_tax_report_line_in_balance"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_in_balance_pa_16" model="account.tax.report.line">
        <field name="name">VAT Inputs PA 16%</field>
        <field name="tag_name">VAT Inputs PA 16%</field>
        <field name="code">ILTAX_IN_BALANCE_PA</field>
        <field name="sequence">52</field>
        <field name="parent_id" ref="account_tax_report_line_in_balance"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_in_balance_2_3" model="account.tax.report.line">
        <field name="name">VAT Inputs 2/3</field>
        <field name="tag_name">VAT Inputs 2/3</field>
        <field name="code">ILTAX_IN_BALANCE_2_3</field>
        <field name="sequence">53</field>
        <field name="parent_id" ref="account_tax_report_line_in_balance"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_in_balance_1_4" model="account.tax.report.line">
        <field name="name">VAT Inputs 1/4</field>
        <field name="tag_name">VAT Inputs 1/4</field>
        <field name="code">ILTAX_IN_BALANCE_1_4</field>
        <field name="sequence">54</field>
        <field name="parent_id" ref="account_tax_report_line_in_balance"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_vat_in_fa_title" model="account.tax.report.line">
        <field name="name">VAT INPUTS (fixed assets)</field>
        <field name="sequence">6</field>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_vat_in_fa" model="account.tax.report.line">
        <field name="name">VAT INPUTS (fixed assets)</field>
        <field name="tag_name">VAT INPUTS (fixed assets)</field>
        <field name="code">ILTAX_VAT_IN_FA</field>
        <field name="sequence">6</field>
        <field name="parent_id" ref="account_tax_report_line_vat_in_fa_title"/>
        <field name="report_id" ref="vat_report"/>
    </record>

    <record id="account_tax_report_line_vat_due" model="account.tax.report.line">
        <field name="name">VAT DUE</field>
        <field name="code">ILTAX_VAT_DUE</field>
        <field name="formula">(ILTAX_OUT_BALANCE_00 + ILTAX_OUT_BALANCE_PA) - (ILTAX_IN_BALANCE_17 + ILTAX_IN_BALANCE_2_3 + ILTAX_IN_BALANCE_1_4 + ILTAX_IN_BALANCE_PA) - ILTAX_VAT_IN_FA</field>
        <field name="sequence">7</field>
        <field name="report_id" ref="vat_report"/>
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_out_base')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
                'plus_report_line_ids': [ref('account_tax_report_line_out_balance')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_out_base')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
                'minus_report_line_ids': [ref('account_tax_report_line_out_balance')]
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_out_base')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111120'),
                'plus_report_line_ids': [ref('account_tax_report_line_out_balance_pa')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_out_base')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111120'),
                'minus_report_line_ids': [ref('account_tax_report_line_out_balance_pa')]
            }),
        ]"/>
    </record>

    <record id="il_vat_sales_exempt" model="account.tax.template">
        <field name="sequence">9</field>
        <field name="description">0%</field>
        <field name="name">VAT exempt</field>
        <field name="price_include" eval="1"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_exempt"/>
        <field name="include_base_amount">1</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_out_base_exempt')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_out_base_exempt')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111110'),
            }),
        ]"/>
    </record>

    <record id="il_vat_self_inv_sale" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="description">17%</field>
        <field name="name">Self Invoice 17%</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_out_base')]
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('account_tax_report_line_out_balance')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_out_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('account_tax_report_line_out_balance')]
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
        <field name="include_base_amount" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101800'),
                'plus_report_line_ids': [ref('account_tax_report_line_in_balance_17')]
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
                'account_id': ref('il_account_101800'),
                'minus_report_line_ids': [ref('account_tax_report_line_in_balance_17')]
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101830'),
                'plus_report_line_ids': [ref('account_tax_report_line_in_balance_pa_16')]
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
                'account_id': ref('il_account_101830'),
                'minus_report_line_ids': [ref('account_tax_report_line_in_balance_pa_16')]
            }),
        ]"/>
    </record>

    <record id="il_vat_inputs_2_3_17" model="account.tax.template">
        <field name="sequence">4</field>
        <field name="description">17%</field>
        <field name="name">VAT Inputs 2/3</field>
        <field name="price_include" eval="1"/>
        <field name="amount">9.6866</field>
        <field name="amount_type">division</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101800'),
                'plus_report_line_ids':[ ref('account_tax_report_line_in_balance_2_3')]
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
                'account_id': ref('il_account_101800'),
                'minus_report_line_ids':[ ref('account_tax_report_line_in_balance_2_3')]
            }),
        ]"/>
    </record>

    <record id="il_vat_inputs_1_4_17" model="account.tax.template">
        <field name="sequence">5</field>
        <field name="description">17%</field>
        <field name="name">VAT Inputs 1/4</field>
        <field name="price_include" eval="1"/>
        <field name="amount">3.6325</field>
        <field name="amount_type">division</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101800'),
                'plus_report_line_ids':[ ref('account_tax_report_line_in_balance_1_4')]
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
                'account_id': ref('il_account_101800'),
                'minus_report_line_ids':[ ref('account_tax_report_line_in_balance_1_4')]
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_101810'),
                'plus_report_line_ids':[ref('account_tax_report_line_vat_in_fa')]
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
                'account_id': ref('il_account_101810'),
                'minus_report_line_ids':[ref('account_tax_report_line_vat_in_fa')]
            }),
        ]"/>
    </record>

    <record id="il_vat_purchase_exempt" model="account.tax.template">
        <field name="sequence">7</field>
        <field name="description">0%</field>
        <field name="name">VAT exempt</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_exempt"/>
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

    <record id="il_vat_retention_10" model="account.tax.template">
        <field name="sequence">13</field>
        <field name="description">Withholding 10%</field>
        <field name="name">Withholding 10%</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_retention_purchase"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111200'),
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
                'account_id': ref('il_account_111200'),
            }),
        ]"/>
    </record>

    <record id="il_vat_retention_35" model="account.tax.template">
        <field name="sequence">14</field>
        <field name="description">Withholding 35%</field>
        <field name="name">Withholding 35%</field>
        <field name="amount">-35</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_retention_purchase"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111200'),
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
                'account_id': ref('il_account_111200'),
            }),
        ]"/>
    </record>

    <record id="il_vat_retention_30" model="account.tax.template">
        <field name="sequence">15</field>
        <field name="description">Withholding 30%</field>
        <field name="name">Withholding 30%</field>
        <field name="amount">-30</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_retention_purchase"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111220'),
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
                'account_id': ref('il_account_111220'),
            }),
        ]"/>
    </record>

    <record id="il_vat_retention_25" model="account.tax.template">
        <field name="sequence">16</field>
        <field name="description">Withholding 25%</field>
        <field name="name">Withholding 25%</field>
        <field name="amount">-25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="tax_group_id" ref="tax_group_retention_purchase"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('il_account_111220'),
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
                'account_id': ref('il_account_111220'),
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

    <!-- Palestinian state -->
    <record id="account_fiscal_position_palestinian_authority" model="account.fiscal.position.template">
        <field name="name">Palestinian Authority (PA)</field>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ps"/>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="auto_apply" eval="True"/>
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

    <!-- Export -->
    <record id="account_fiscal_position_export" model="account.fiscal.position.template">
        <field name="name">Import / Export</field>
        <field name="auto_apply" eval="True"/>
        <field name="chart_template_id" ref="il_chart_template"/>
        <field name="auto_apply" eval="True"/>
    </record>

    <record id="account_fiscal_position_export_01" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="il_vat_sales_17"/>
        <field name="tax_dest_id" ref="il_vat_sales_exempt"/>
        <field name="position_id" ref="account_fiscal_position_export"/>
    </record>


</odoo>

```

