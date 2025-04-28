# Odoo Module: l10n_my

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from odoo import api, SUPERUSER_ID


def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref("l10n_my.l10n_my_chart_template").process_coa_translations()

```

## File: __manifest__.py

```python
{
    "name": "Malaysia - Accounting",
    "author": "Odoo PS",
    "version": "1.0",
    "category": "Accounting/Localizations/Account Charts",
    "description": """
This is the base module to manage the accounting chart for Malaysia in Odoo.
==============================================================================
    """,
    "depends": [
        "account",
        "l10n_multilang",
    ],
    "data": [
        "data/l10n_my_chart_data.xml",
        "data/account.account.template.csv",
        "data/account_chart_template_data.xml",
        "data/account.tax.group.csv",
        "data/account_tax_template_data.xml",
        "data/account_chart_template_configure_data.xml",
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    "icon": "/base/static/img/country_flags/my.png",
    "post_init_hook": "load_translations",
    "license": "LGPL-3",
}

```

## File: data\account.account.template.csv

```csv
id,name,code,account_type,chart_template_id/id,tag_ids/id,reconcile
l10n_my_11,Fixed Assets,11,asset_fixed,l10n_my.l10n_my_chart_template,,False
l10n_my_1110,Furniture and Fixtures,1110,asset_non_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1130,Equipment,1130,asset_non_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1140,Decoration,1140,asset_non_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1160,Investments,1160,asset_non_current,l10n_my.l10n_my_chart_template,,False
l10n_my_12,Current Assets,12,asset_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1240,Account Receivable,1240,asset_receivable,l10n_my.l10n_my_chart_template,,True
l10n_my_1241,Utility & Rental Deposit,1241,asset_current,l10n_my.l10n_my_chart_template,,True
l10n_my_1242,Supplier Prepayments,1242,asset_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1243,Account Receivable (PoS),1243,asset_receivable,l10n_my.l10n_my_chart_template,,True
l10n_my_1245,Sundry Deposits,1245,asset_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1246,Other Receivable,1246,asset_receivable,l10n_my.l10n_my_chart_template,,True
l10n_my_1250,Stock Interim Account (Received),1250,asset_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1260,Stock Interim Account (Delivered),1260,asset_current,l10n_my.l10n_my_chart_template,,False
l10n_my_1270,Stock Valuation Account,1270,asset_current,l10n_my.l10n_my_chart_template,,False
l10n_my_21,Non-current Liabilities,21,liability_non_current,l10n_my.l10n_my_chart_template,,False
l10n_my_22,Current Liabilities,22,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_2210,Accruals,2210,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_22101,Accruals - KWSP,22101,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_22102,Accruals - PCB,22102,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_22103,Accruals - SOCSO,22103,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_22104,Accruals - EIS,22104,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_2211,Account Payable,2211,liability_payable,l10n_my.l10n_my_chart_template,,True
l10n_my_2212,Receipt in Advance (Customer Prepayments),2212,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_2213,SST Control Account,2213,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_2214,Provision for Taxation,2214,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_2215,Proposed Dividend,2215,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_2216,Other Payable,2216,liability_payable,l10n_my.l10n_my_chart_template,,True
l10n_my_31,Paid Capital,31,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_32,Accumulated Profit & Loss,32,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_33,Profit & Loss Account,33,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_34,Short-term Borrowing,34,liability_current,l10n_my.l10n_my_chart_template,,False
l10n_my_41,Trade Income,41,income,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_42,Other Income and Gains,42,income,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_4210,Sundry Income,4210,income,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_4220,Exchange Adjustment,4220,income,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_4230,Bank Interest Income,4230,income,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_4240,Foreign Exchange Gain,4240,income,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_51,Costs,51,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5101,Trade Costs,5101,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5105,Misc. Costs,5105,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5106,Costs of Transportation,5106,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5111,Declaration Fees,5111,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5112,Packing Fees,5112,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_52,Expenses,52,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5201,Bank Charges,5201,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5202,Entertainment,5202,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5203,Electricity & Water Fees,5203,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5205,Postage & Stamps,5205,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5206,Printing & Stationery,5206,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5207,Rent & Rates,5207,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5208,Sundry Expenses,5208,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5209,Telecommunication Expenses,5209,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5210,Traffic Fees,5210,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5211,IT Expenses,5211,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5214,Insurance,5214,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5215,Sales Commission,5215,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5216,Overseas Traveling,5216,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5218,Wages & Salaries,5218,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_52181,KWSP Contribution,52181,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_52182,PCB Contribution,52182,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_52183,EIS Contribution,52183,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5219,Bonus Payment,5219,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5221,SST,5221,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5222,Local Delivery,5222,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5223,Management Fees,5223,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5224,Depreciation,5224,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5225,Audit Fees,5225,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5226,Bad Debts,5226,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5228,Legal & Professional Fees,5228,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5229,Dividend,5229,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5231,Disposal,5231,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5234,Repair and Maintenance,5234,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5235,Advertising,5235,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False
l10n_my_5240,Foreign Exchange Loss,5240,expense,l10n_my.l10n_my_chart_template,account.account_tag_operating,False

```

## File: data\account.tax.group.csv

```csv
id,name,country_id/id
tax_group_sst,SST,base.my

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_my.l10n_my_chart_template')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="l10n_my_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="l10n_my_1240" />
        <field name="property_account_payable_id" ref="l10n_my_2211" />
        <field name="property_account_income_categ_id" ref="l10n_my_41" />
        <field name="property_account_expense_categ_id" ref="l10n_my_51" />
        <field name="expense_currency_exchange_account_id" ref="l10n_my_5240" />
        <field name="income_currency_exchange_account_id" ref="l10n_my_4240" />
        <field name="default_pos_receivable_account_id" ref="l10n_my_1243" />
        <field name="use_anglo_saxon" eval="True" />
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="l10n_my_tax_sale_5" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_my_chart_template" />
        <field name="name">SST 5%</field>
        <field name="sequence">1</field>
        <field name="description">5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="tax_scope">consu</field>
        <field name="amount">5.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_sst" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_my_2213'),
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
                    'account_id': ref('l10n_my_2213'),
                }),
            ]"/>
    </record>
    <record id="l10n_my_tax_sale_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_my_chart_template" />
        <field name="name">SST 6%</field>
        <field name="sequence">1</field>
        <field name="description">6%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="tax_scope">service</field>
        <field name="amount">6.0</field>
        <field name="active" eval="False"/>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_sst" />
        <field name="invoice_repartition_line_ids" eval="[
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_my_2213'),
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_my_2213'),
                }),
            ]"/>
    </record>
    <record id="l10n_my_tax_sale_8" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_my_chart_template" />
        <field name="name">SST 8%</field>
        <field name="sequence">1</field>
        <field name="description">8%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="tax_scope">service</field>
        <field name="amount">8.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_sst" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_my_2213'),
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
                    'account_id': ref('l10n_my_2213'),
                }),
            ]"/>
    </record>
    <record id="l10n_my_tax_sale_10" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_my_chart_template" />
        <field name="name">SST 10%</field>
        <field name="sequence">1</field>
        <field name="description">10%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="tax_scope">consu</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_sst" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_my_2213'),
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
                    'account_id': ref('l10n_my_2213'),
                }),
            ]"/>
    </record>
</odoo>

```

## File: data\l10n_my_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_my_chart_template" model="account.chart.template">
        <field name="name">Malaysia - Chart of Accounts</field>
        <field name="bank_account_code_prefix">1200</field>
        <field name="cash_account_code_prefix">1210</field>
        <field name="transfer_account_code_prefix">111220</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.MYR" />
        <field name="spoken_languages" eval="'ms_MY'"/>
        <field name="country_id" ref="base.my"/>
    </record>
</odoo>

```

