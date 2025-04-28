# Odoo Module: l10n_generic_coa

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


def uninstall_hook(cr, registry):
    cr.execute(
        "DELETE FROM ir_model_data WHERE module = 'l10n_generic_coa'"
    )

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Generic - Accounting',
    'version': '1.1',
    'category': 'Localization',
    'description': """
This is the base module to manage the generic accounting chart in Odoo.
==============================================================================

Install some generic chart of accounts.
    """,
    'depends': [
        'account',
    ],
    'data': [
        'data/l10n_generic_coa.xml',
        'data/account.account.template.csv',
        'data/l10n_generic_coa_post.xml',
    ],
    'demo': [
        'demo/account_bank_statement_demo.xml',
        'demo/account_invoice_demo.xml',
        'demo/account_reconcile_model.xml',
    ],
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","tag_ids/id","reconcile"
"current_assets","Current Assets","1010","account.data_account_type_current_assets","l10n_generic_coa.configurable_chart_template","","False"
"stock_valuation","Stock Valuation","1101","account.data_account_type_current_assets","l10n_generic_coa.configurable_chart_template","","False"
"stock_in","Stock Interim (Received)","1102","account.data_account_type_current_assets","l10n_generic_coa.configurable_chart_template","","True"
"stock_out","Stock Interim (Delivered)","1103","account.data_account_type_current_assets","l10n_generic_coa.configurable_chart_template","","True"
"receivable","Account Receivable","1210","account.data_account_type_receivable","l10n_generic_coa.configurable_chart_template","","True"
"tax_paid","Tax Paid","1310","account.data_account_type_current_assets","l10n_generic_coa.configurable_chart_template","","False"
"tax_receivable","Tax Receivable","1320","account.data_account_type_current_assets","l10n_generic_coa.configurable_chart_template","","False"
"prepayments","Prepayments","1410","account.data_account_type_prepayments","l10n_generic_coa.configurable_chart_template","","False"
"fixed_assets","Fixed Asset","1510","account.data_account_type_fixed_assets","l10n_generic_coa.configurable_chart_template","","False"
"non_current_assets","Non-current assets","1910","account.data_account_type_non_current_assets","l10n_generic_coa.configurable_chart_template","","False"
"current_liabilities","Current Liabilities","2010","account.data_account_type_current_liabilities","l10n_generic_coa.configurable_chart_template","","False"
"payable","Account Payable","2110","account.data_account_type_payable","l10n_generic_coa.configurable_chart_template","","True"
"tax_received","Tax Received","2510","account.data_account_type_current_liabilities","l10n_generic_coa.configurable_chart_template","","False"
"tax_payable","Tax Payable","2520","account.data_account_type_current_liabilities","l10n_generic_coa.configurable_chart_template","","False"
"non_current_liabilities","Non-current Liabilities","2910","account.data_account_type_non_current_liabilities","l10n_generic_coa.configurable_chart_template","","False"
"capital","Capital","3010","account.data_account_type_equity","l10n_generic_coa.configurable_chart_template","","False"
"dividends","Dividends","3020","account.data_account_type_equity","l10n_generic_coa.configurable_chart_template","","False"
"income","Product Sales","4000","account.data_account_type_revenue","l10n_generic_coa.configurable_chart_template","account.account_tag_operating","False"
"income_currency_exchange","Foreign Exchange Gain","4410","account.data_account_type_revenue","l10n_generic_coa.configurable_chart_template","account.account_tag_financing","False"
"cash_diff_income","Cash Difference Gain","4420","account.data_account_type_revenue","l10n_generic_coa.configurable_chart_template","account.account_tag_investing","False"
"other_income","Other Income","4500","account.data_account_type_other_income","l10n_generic_coa.configurable_chart_template","","False"
"cost_of_goods_sold","Cost of Goods Sold","5000","account.data_account_type_direct_costs","l10n_generic_coa.configurable_chart_template","account.account_tag_operating","False"
"expense","Expenses","6000","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_operating","False"
"expense_invest","Purchase of Equipments","6110","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_investing","False"
"expense_rent","Rent","6120","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_investing","False"
"expense_finance","Bank Fees","6200","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_financing","False"
"expense_salary","Salary Expenses","6300","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_operating","False"
"expense_currency_exchange","Foreign Exchange Loss","6410","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_financing","False"
"cash_diff_expense","Cash Difference Loss","6420","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_investing","False"
"expense_rd","RD Expenses","9610","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_investing","False"
"expense_sales","Sales Expenses","9620","account.data_account_type_expenses","l10n_generic_coa.configurable_chart_template","account.account_tag_investing","False"
"pos_receivable","Account Receivable (PoS)","1013","account.data_account_type_receivable","l10n_generic_coa.configurable_chart_template","","True"

```

## File: data\l10n_generic_coa.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Tax Group -->
    <record id="configurable_chart_template" model="account.chart.template">
        <field name="name">Configurable Account Chart Template</field>
        <field name="code_digits">6</field>
        <field name="bank_account_code_prefix">1014</field>
        <field name="cash_account_code_prefix">1015</field>
        <field name="transfer_account_code_prefix">1017</field>
        <field name="currency_id" ref="base.USD"/>
    </record>
</odoo>

```

## File: data\l10n_generic_coa_post.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <!-- Tax template for sale and purchase -->
    <record id="tax_group_15" model="account.tax.group">
        <field name="name">Tax 15%</field>
    </record>
</data>
<data>
    <!-- Chart template account links -->
    <record id="configurable_chart_template" model="account.chart.template">
        <field name="complete_tax_set" eval="False"/>
        <field name="use_anglo_saxon" eval="True"/>

        <field name="property_account_receivable_id" ref="receivable"/>
        <field name="property_account_payable_id" ref="payable"/>

        <field name="property_account_expense_categ_id" ref="expense"/>
        <field name="property_account_income_categ_id" ref="income"/>

        <field name="property_stock_account_input_categ_id" ref="stock_in"/>
        <field name="property_stock_account_output_categ_id" ref="stock_out"/>
        <field name="property_stock_valuation_account_id" ref="stock_valuation"/>

        <field name="income_currency_exchange_account_id" ref="income_currency_exchange"/>
        <field name="expense_currency_exchange_account_id" ref="expense_currency_exchange"/>

        <field name="default_cash_difference_income_account_id" ref="cash_diff_income"/>
        <field name="default_cash_difference_expense_account_id" ref="cash_diff_expense"/>
        <field name="default_pos_receivable_account_id" ref="pos_receivable"/>

        <field name="property_tax_payable_account_id" ref="tax_payable"/>
        <field name="property_tax_receivable_account_id" ref="tax_receivable"/>
    </record>

    <record id="sale_tax_template" model="account.tax.template">
        <field name="chart_template_id" ref="configurable_chart_template"/>
        <field name="name">Tax 15%</field>
        <field name="amount">15</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('tax_received'),
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
                'account_id': ref('tax_received'),
            }),
        ]"/>
    </record>

   <record id="purchase_tax_template" model="account.tax.template">
        <field name="chart_template_id" ref="configurable_chart_template"/>
        <field name="name">Purchase Tax 15%</field>
        <field name="amount">15</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('tax_paid'),
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
                'account_id': ref('tax_paid'),
            }),
        ]"/>
    </record>

    <!-- Try to instanciate for relevant companies -->
</data>
<data noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_generic_coa.configurable_chart_template')]"/>
    </function>
</data>
</odoo>

```

