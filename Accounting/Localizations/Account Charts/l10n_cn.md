# Odoo Module: l10n_cn

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2007-2014 Jeff Wang(<http://jeff@osbzr.com>).

from . import models

from odoo import api, SUPERUSER_ID

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_cn.l10n_chart_china_small_business').process_coa_translations()

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"l10n_cn_1012","Other Monetary Funds","1012","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1101","Transactional Financial Assets","1101","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1121","Bills Receivable","1121","account.data_account_type_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1122","Accounts Receivable","1122","account.data_account_type_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1123","Advance Payment","1123","account.data_account_type_prepayments","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1124","Accounts Receivable (PoS)","1124","account.data_account_type_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1131","Divident Receivable","1131","account.data_account_type_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1132","Interest Receivable","1132","account.data_account_type_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1221","Other Receivable","1221","account.data_account_type_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1231","Bad Debt Provisions","1231","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1401","Material Purchasing","1401","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1402","Materials in transit","1402","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1403","Raw Material","1403","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1404","Material Cost Variance","1404","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1405","Merchandise Inventory","1405","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1406","Goods shipped in transit","1406","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1407","Differences between purchasing and selling price","1407","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1408","Consigned processing materials","1408","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1471","Inventory falling price reserves","1471","account.data_account_type_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1501","Held to maturity Investment","1501","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1502","Provision for impairment of investments held to maturity","1502","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1503","Available for sale financial assets","1503","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1511","Long-term equity investment","1511","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1512","Impairment provision for long-term equity investments","1512","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1521","Investmental real estate","1521","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1531","Long-term receivables","1531","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1601","Fixed assets","1601","account.data_account_type_fixed_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1602","Accumulated depreciation","1602","account.data_account_type_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1603","Fixed assets depreciation reserves","1603","account.data_account_type_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1604","Construction in progress","1604","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1605","Engineering materials","1605","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1606","Liquidation of fixed assets","1606","account.data_account_type_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1701","Intangible Assets","1701","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1702","Accumulated amortization","1702","account.data_account_type_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1703","Intangible Assets Depreciation Reserves","1703","account.data_account_type_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1711","Goodwill","1711","account.data_account_type_non_current_assets","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1801","Long-term amortized expenses","1801","account.data_account_type_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_2001","Short-term borrowing","2001","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2201","Bills Payable","2201","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2202","Accounts Payable","2202","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2203","Deposit Received","2203","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2211","Payroll payable","2211","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2221","Tax payable","2221","account.data_account_type_current_liabilities","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2231","Interest payable","2231","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2241","Dividents payable","2241","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2501","Other payable","2501","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2502","Bonds Payable","2502","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2701","Long Term payables","2701","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2711","Account payable special funds","2711","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2801","Projected liabilities","2801","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2901","Deferred Tax Liability","2901","account.data_account_type_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_4001","Paid in capital","4001","account.data_account_type_equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4002","Capital Surplus","4002","account.data_account_type_equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4003","Other Comprehensive Income","4003","account.data_account_type_equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4101","Surplus Reserve","4101","account.data_account_type_equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4103","Profit for the year","4103","account.data_account_type_equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4104","Profit distribution","4104","account.data_account_type_equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5001","Production Costs","5001","account.data_account_type_direct_costs","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5101","Manufacturing Expenses","5101","account.data_account_type_direct_costs","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5201","Service Cost","5201","account.data_account_type_direct_costs","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5301","R & D expenditure","5301","account.data_account_type_direct_costs","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6001","Main Business Income","6001","account.data_account_type_revenue","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6051","Other Business Income","6051","account.data_account_type_other_income","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6101","Gains and Losses of fair value change","6101","account.data_account_type_other_income","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6111","Income from investment","6111","account.data_account_type_other_income","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6301","Non-operating Income","6301","account.data_account_type_other_income","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6401","Main Business Cost","6401","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6402","Other Operating Costs","6402","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6403","Operating Taxes and Surcharges","6403","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6601","Selling Expenses","6601","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6602","Management Expenses","6602","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6603","Financial Expenses","6603","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6701","Assets impairment Loss","6701","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6711","Non-operating expenses","6711","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6801","Income Tax Expense","6801","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6901","Prior year income adjustment","6901","account.data_account_type_expenses","l10n_cn.l10n_chart_china_small_business","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_cn.l10n_chart_china_small_business')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="l10n_cn_tax_group_vat_6" model="account.tax.group">
        <field name="name">VAT 6%</field>
    </record>
    <record id="l10n_cn_tax_group_vat_9" model="account.tax.group">
        <field name="name">VAT 9%</field>
    </record>
    <record id="l10n_cn_tax_group_vat_13" model="account.tax.group">
        <field name="name">VAT 13%</field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- sales tax included -->
    <record id="l10n_cn_sales_included_13" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收13％（含)</field>
        <field name="description">税收13％</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_13"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_sales_included_9" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收9％（含)</field>
        <field name="description">税收9％</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_sales_included_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收6％（含)</field>
        <field name="description">税收6％</field>
        <field name="amount">6</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>


    <!-- sales tax excluded -->
    <record id="l10n_cn_sales_excluded_13" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收13%</field>
        <field name="description">税收13%</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_13"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_sales_excluded_9" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收9%</field>
        <field name="description">税收9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_sales_excluded_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收6%</field>
        <field name="description">税收6％</field>
        <field name="amount">6</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>

    <!-- purchase tax excluded -->
    <record id="l10n_cn_purchase_excluded_13" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收13%</field>
        <field name="description">税收13%</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_13"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_purchase_excluded_9" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收9%</field>
        <field name="description">税收9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_purchase_excluded_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收6%</field>
        <field name="description">税收6％</field>
        <field name="amount">6</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
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
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_cn_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!--

     Copyright (C) 2012-2012 南京盈通 ccdos@intoerp.com <small business chart>
     
会计科目表模板( 小企业会计准则2011)

科目表依据：
关于印发《小企业会计准则》的通知
http://kjs.mof.gov.cn/zhengwuxinxi/zhengcefabu/201111/t20111107_605525.html

-->
    <data>
        <record id="l10n_chart_china_small_business" model="account.chart.template">
            <field name="name">小企业会计科目表（财会[2011]17号《小企业会计准则》）</field>
            <field name="code_digits" eval="6"/>
            <field name="currency_id" ref="base.CNY"/>
            <field name="cash_account_code_prefix">1001</field>
            <field name="bank_account_code_prefix">1002</field>
            <field name="transfer_account_code_prefix">1012</field>
            <field name="spoken_languages" eval="'en_US'"/>
        </record>
    </data>
</odoo>



```

## File: data\l10n_cn_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_chart_china_small_business" model="account.chart.template">
        <field name="property_account_receivable_id" ref="l10n_cn_1122"/>
        <field name="property_account_payable_id" ref="l10n_cn_2202"/>
        <field name="property_account_expense_categ_id" ref="l10n_cn_6401"/>
        <field name="property_account_income_categ_id" ref="l10n_cn_6001"/>
        <field name="income_currency_exchange_account_id" ref="l10n_cn_6051"/>
        <field name="expense_currency_exchange_account_id" ref="l10n_cn_6711"/>
        <field name="default_pos_receivable_account_id" ref="l10n_cn_1124" />
    </record>
</odoo>

```

## File: i18n_extra\l10n_cn.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_cn
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 14.0\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2021-03-15 12:16+0000\n"
"PO-Revision-Date: 2021-03-15 12:16+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2711
#: model:account.account,name:l10n_cn.2_l10n_cn_2711
#: model:account.account.template,name:l10n_cn.l10n_cn_2711
msgid "Account payable special funds"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2202
#: model:account.account,name:l10n_cn.2_l10n_cn_2202
#: model:account.account.template,name:l10n_cn.l10n_cn_2202
msgid "Accounts Payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1122
#: model:account.account,name:l10n_cn.2_l10n_cn_1122
#: model:account.account.template,name:l10n_cn.l10n_cn_1122
msgid "Accounts Receivable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1124
#: model:account.account,name:l10n_cn.2_l10n_cn_1124
#: model:account.account.template,name:l10n_cn.l10n_cn_1124
msgid "Accounts Receivable (PoS)"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1702
#: model:account.account,name:l10n_cn.2_l10n_cn_1702
#: model:account.account.template,name:l10n_cn.l10n_cn_1702
msgid "Accumulated amortization"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1602
#: model:account.account,name:l10n_cn.2_l10n_cn_1602
#: model:account.account.template,name:l10n_cn.l10n_cn_1602
msgid "Accumulated depreciation"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1123
#: model:account.account,name:l10n_cn.2_l10n_cn_1123
#: model:account.account.template,name:l10n_cn.l10n_cn_1123
msgid "Advance Payment"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6701
#: model:account.account,name:l10n_cn.2_l10n_cn_6701
#: model:account.account.template,name:l10n_cn.l10n_cn_6701
msgid "Assets impairment Loss"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1503
#: model:account.account,name:l10n_cn.2_l10n_cn_1503
#: model:account.account.template,name:l10n_cn.l10n_cn_1503
msgid "Available for sale financial assets"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1231
#: model:account.account,name:l10n_cn.2_l10n_cn_1231
#: model:account.account.template,name:l10n_cn.l10n_cn_1231
msgid "Bad Debt Provisions"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2201
#: model:account.account,name:l10n_cn.2_l10n_cn_2201
#: model:account.account.template,name:l10n_cn.l10n_cn_2201
msgid "Bills Payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1121
#: model:account.account,name:l10n_cn.2_l10n_cn_1121
#: model:account.account.template,name:l10n_cn.l10n_cn_1121
msgid "Bills Receivable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2502
#: model:account.account,name:l10n_cn.2_l10n_cn_2502
#: model:account.account.template,name:l10n_cn.l10n_cn_2502
msgid "Bonds Payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_4002
#: model:account.account,name:l10n_cn.2_l10n_cn_4002
#: model:account.account.template,name:l10n_cn.l10n_cn_4002
msgid "Capital Surplus"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1408
#: model:account.account,name:l10n_cn.2_l10n_cn_1408
#: model:account.account.template,name:l10n_cn.l10n_cn_1408
msgid "Consigned processing materials"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1604
#: model:account.account,name:l10n_cn.2_l10n_cn_1604
#: model:account.account.template,name:l10n_cn.l10n_cn_1604
msgid "Construction in progress"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2901
#: model:account.account,name:l10n_cn.2_l10n_cn_2901
#: model:account.account.template,name:l10n_cn.l10n_cn_2901
msgid "Deferred Tax Liability"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2203
#: model:account.account,name:l10n_cn.2_l10n_cn_2203
#: model:account.account.template,name:l10n_cn.l10n_cn_2203
msgid "Deposit Received"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1407
#: model:account.account,name:l10n_cn.2_l10n_cn_1407
#: model:account.account.template,name:l10n_cn.l10n_cn_1407
msgid "Differences between purchasing and selling price"
msgstr ""

#. module: l10n_cn
#: model:ir.model.fields,field_description:l10n_cn.field_account_move__display_name
msgid "Display Name"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1131
#: model:account.account,name:l10n_cn.2_l10n_cn_1131
#: model:account.account.template,name:l10n_cn.l10n_cn_1131
msgid "Divident Receivable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2241
#: model:account.account,name:l10n_cn.2_l10n_cn_2241
#: model:account.account.template,name:l10n_cn.l10n_cn_2241
msgid "Dividents payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1605
#: model:account.account,name:l10n_cn.2_l10n_cn_1605
#: model:account.account.template,name:l10n_cn.l10n_cn_1605
msgid "Engineering materials"
msgstr ""

#. module: l10n_cn
#: model:ir.model.fields,field_description:l10n_cn.field_account_bank_statement_line__fapiao
#: model:ir.model.fields,field_description:l10n_cn.field_account_move__fapiao
#: model:ir.model.fields,field_description:l10n_cn.field_account_payment__fapiao
msgid "Fapiao Number"
msgstr ""

#. module: l10n_cn
#: code:addons/l10n_cn/models/account_move.py:0
#, python-format
msgid "Fapiao number is an 8-digit number. Please enter a correct one."
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6603
#: model:account.account,name:l10n_cn.2_l10n_cn_6603
#: model:account.account.template,name:l10n_cn.l10n_cn_6603
msgid "Financial Expenses"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1601
#: model:account.account,name:l10n_cn.2_l10n_cn_1601
#: model:account.account.template,name:l10n_cn.l10n_cn_1601
msgid "Fixed assets"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1603
#: model:account.account,name:l10n_cn.2_l10n_cn_1603
#: model:account.account.template,name:l10n_cn.l10n_cn_1603
msgid "Fixed assets depreciation reserves"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6101
#: model:account.account,name:l10n_cn.2_l10n_cn_6101
#: model:account.account.template,name:l10n_cn.l10n_cn_6101
msgid "Gains and Losses of fair value change"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1406
#: model:account.account,name:l10n_cn.2_l10n_cn_1406
#: model:account.account.template,name:l10n_cn.l10n_cn_1406
msgid "Goods shipped in transit"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1711
#: model:account.account,name:l10n_cn.2_l10n_cn_1711
#: model:account.account.template,name:l10n_cn.l10n_cn_1711
msgid "Goodwill"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1501
#: model:account.account,name:l10n_cn.2_l10n_cn_1501
#: model:account.account.template,name:l10n_cn.l10n_cn_1501
msgid "Held to maturity Investment"
msgstr ""

#. module: l10n_cn
#: model:ir.model.fields,field_description:l10n_cn.field_account_move__id
msgid "ID"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1512
#: model:account.account,name:l10n_cn.2_l10n_cn_1512
#: model:account.account.template,name:l10n_cn.l10n_cn_1512
msgid "Impairment provision for long-term equity investments"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6801
#: model:account.account,name:l10n_cn.2_l10n_cn_6801
#: model:account.account.template,name:l10n_cn.l10n_cn_6801
msgid "Income Tax Expense"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6111
#: model:account.account,name:l10n_cn.2_l10n_cn_6111
#: model:account.account.template,name:l10n_cn.l10n_cn_6111
msgid "Income from investment"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1701
#: model:account.account,name:l10n_cn.2_l10n_cn_1701
#: model:account.account.template,name:l10n_cn.l10n_cn_1701
msgid "Intangible Assets"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1703
#: model:account.account,name:l10n_cn.2_l10n_cn_1703
#: model:account.account.template,name:l10n_cn.l10n_cn_1703
msgid "Intangible Assets Depreciation Reserves"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1132
#: model:account.account,name:l10n_cn.2_l10n_cn_1132
#: model:account.account.template,name:l10n_cn.l10n_cn_1132
msgid "Interest Receivable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2231
#: model:account.account,name:l10n_cn.2_l10n_cn_2231
#: model:account.account.template,name:l10n_cn.l10n_cn_2231
msgid "Interest payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1471
#: model:account.account,name:l10n_cn.2_l10n_cn_1471
#: model:account.account.template,name:l10n_cn.l10n_cn_1471
msgid "Inventory falling price reserves"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1521
#: model:account.account,name:l10n_cn.2_l10n_cn_1521
#: model:account.account.template,name:l10n_cn.l10n_cn_1521
msgid "Investmental real estate"
msgstr ""

#. module: l10n_cn
#: model:ir.model,name:l10n_cn.model_account_move
msgid "Journal Entry"
msgstr ""

#. module: l10n_cn
#: model:ir.model.fields,field_description:l10n_cn.field_account_move____last_update
msgid "Last Modified on"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1606
#: model:account.account,name:l10n_cn.2_l10n_cn_1606
#: model:account.account.template,name:l10n_cn.l10n_cn_1606
msgid "Liquidation of fixed assets"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_chart_china_small_business_liquidity_transfer
#: model:account.account,name:l10n_cn.2_l10n_chart_china_small_business_liquidity_transfer
#: model:account.account.template,name:l10n_cn.l10n_chart_china_small_business_liquidity_transfer
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2701
#: model:account.account,name:l10n_cn.2_l10n_cn_2701
#: model:account.account.template,name:l10n_cn.l10n_cn_2701
msgid "Long Term payables"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1801
#: model:account.account,name:l10n_cn.2_l10n_cn_1801
#: model:account.account.template,name:l10n_cn.l10n_cn_1801
msgid "Long-term amortized expenses"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1511
#: model:account.account,name:l10n_cn.2_l10n_cn_1511
#: model:account.account.template,name:l10n_cn.l10n_cn_1511
msgid "Long-term equity investment"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1531
#: model:account.account,name:l10n_cn.2_l10n_cn_1531
#: model:account.account.template,name:l10n_cn.l10n_cn_1531
msgid "Long-term receivables"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6401
#: model:account.account,name:l10n_cn.2_l10n_cn_6401
#: model:account.account.template,name:l10n_cn.l10n_cn_6401
msgid "Main Business Cost"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6001
#: model:account.account,name:l10n_cn.2_l10n_cn_6001
#: model:account.account.template,name:l10n_cn.l10n_cn_6001
msgid "Main Business Income"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6602
#: model:account.account,name:l10n_cn.2_l10n_cn_6602
#: model:account.account.template,name:l10n_cn.l10n_cn_6602
msgid "Management Expenses"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_5101
#: model:account.account,name:l10n_cn.2_l10n_cn_5101
#: model:account.account.template,name:l10n_cn.l10n_cn_5101
msgid "Manufacturing Expenses"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1404
#: model:account.account,name:l10n_cn.2_l10n_cn_1404
#: model:account.account.template,name:l10n_cn.l10n_cn_1404
msgid "Material Cost Variance"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1401
#: model:account.account,name:l10n_cn.2_l10n_cn_1401
#: model:account.account.template,name:l10n_cn.l10n_cn_1401
msgid "Material Purchasing"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1402
#: model:account.account,name:l10n_cn.2_l10n_cn_1402
#: model:account.account.template,name:l10n_cn.l10n_cn_1402
msgid "Materials in transit"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1405
#: model:account.account,name:l10n_cn.2_l10n_cn_1405
#: model:account.account.template,name:l10n_cn.l10n_cn_1405
msgid "Merchandise Inventory"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6301
#: model:account.account,name:l10n_cn.2_l10n_cn_6301
#: model:account.account.template,name:l10n_cn.l10n_cn_6301
msgid "Non-operating Income"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6711
#: model:account.account,name:l10n_cn.2_l10n_cn_6711
#: model:account.account.template,name:l10n_cn.l10n_cn_6711
msgid "Non-operating expenses"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6403
#: model:account.account,name:l10n_cn.2_l10n_cn_6403
#: model:account.account.template,name:l10n_cn.l10n_cn_6403
msgid "Operating Taxes and Surcharges"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6051
#: model:account.account,name:l10n_cn.2_l10n_cn_6051
#: model:account.account.template,name:l10n_cn.l10n_cn_6051
msgid "Other Business Income"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_4003
#: model:account.account,name:l10n_cn.2_l10n_cn_4003
#: model:account.account.template,name:l10n_cn.l10n_cn_4003
msgid "Other Comprehensive Income"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1012
#: model:account.account,name:l10n_cn.2_l10n_cn_1012
#: model:account.account.template,name:l10n_cn.l10n_cn_1012
msgid "Other Monetary Funds"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6402
#: model:account.account,name:l10n_cn.2_l10n_cn_6402
#: model:account.account.template,name:l10n_cn.l10n_cn_6402
msgid "Other Operating Costs"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1221
#: model:account.account,name:l10n_cn.2_l10n_cn_1221
#: model:account.account.template,name:l10n_cn.l10n_cn_1221
msgid "Other Receivable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2501
#: model:account.account,name:l10n_cn.2_l10n_cn_2501
#: model:account.account.template,name:l10n_cn.l10n_cn_2501
msgid "Other payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_4001
#: model:account.account,name:l10n_cn.2_l10n_cn_4001
#: model:account.account.template,name:l10n_cn.l10n_cn_4001
msgid "Paid in capital"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2211
#: model:account.account,name:l10n_cn.2_l10n_cn_2211
#: model:account.account.template,name:l10n_cn.l10n_cn_2211
msgid "Payroll payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6901
#: model:account.account,name:l10n_cn.2_l10n_cn_6901
#: model:account.account.template,name:l10n_cn.l10n_cn_6901
msgid "Prior year income adjustment"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_5001
#: model:account.account,name:l10n_cn.2_l10n_cn_5001
#: model:account.account.template,name:l10n_cn.l10n_cn_5001
msgid "Production Costs"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_4104
#: model:account.account,name:l10n_cn.2_l10n_cn_4104
#: model:account.account.template,name:l10n_cn.l10n_cn_4104
msgid "Profit distribution"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_4103
#: model:account.account,name:l10n_cn.2_l10n_cn_4103
#: model:account.account.template,name:l10n_cn.l10n_cn_4103
msgid "Profit for the year"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2801
#: model:account.account,name:l10n_cn.2_l10n_cn_2801
#: model:account.account.template,name:l10n_cn.l10n_cn_2801
msgid "Projected liabilities"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1502
#: model:account.account,name:l10n_cn.2_l10n_cn_1502
#: model:account.account.template,name:l10n_cn.l10n_cn_1502
msgid "Provision for impairment of investments held to maturity"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_5301
#: model:account.account,name:l10n_cn.2_l10n_cn_5301
#: model:account.account.template,name:l10n_cn.l10n_cn_5301
msgid "R & D expenditure"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1403
#: model:account.account,name:l10n_cn.2_l10n_cn_1403
#: model:account.account.template,name:l10n_cn.l10n_cn_1403
msgid "Raw Material"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_6601
#: model:account.account,name:l10n_cn.2_l10n_cn_6601
#: model:account.account.template,name:l10n_cn.l10n_cn_6601
msgid "Selling Expenses"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_5201
#: model:account.account,name:l10n_cn.2_l10n_cn_5201
#: model:account.account.template,name:l10n_cn.l10n_cn_5201
msgid "Service Cost"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2001
#: model:account.account,name:l10n_cn.2_l10n_cn_2001
#: model:account.account.template,name:l10n_cn.l10n_cn_2001
msgid "Short-term borrowing"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_4101
#: model:account.account,name:l10n_cn.2_l10n_cn_4101
#: model:account.account.template,name:l10n_cn.l10n_cn_4101
msgid "Surplus Reserve"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_2221
#: model:account.account,name:l10n_cn.2_l10n_cn_2221
#: model:account.account.template,name:l10n_cn.l10n_cn_2221
msgid "Tax payable"
msgstr ""

#. module: l10n_cn
#: model:account.account,name:l10n_cn.1_l10n_cn_1101
#: model:account.account,name:l10n_cn.2_l10n_cn_1101
#: model:account.account.template,name:l10n_cn.l10n_cn_1101
msgid "Transactional Financial Assets"
msgstr ""

#. module: l10n_cn
#: model:account.tax.group,name:l10n_cn.l10n_cn_tax_group_vat_13
msgid "VAT 13%"
msgstr ""

#. module: l10n_cn
#: model:account.tax.group,name:l10n_cn.l10n_cn_tax_group_vat_6
msgid "VAT 6%"
msgstr ""

#. module: l10n_cn
#: model:account.tax.group,name:l10n_cn.l10n_cn_tax_group_vat_9
msgid "VAT 9%"
msgstr ""

#. module: l10n_cn
#: model:account.chart.template,name:l10n_cn.l10n_chart_china_small_business
msgid "小企业会计科目表（财会[2011]17号《小企业会计准则》）"
msgstr ""

#. module: l10n_cn
#: model:account.tax,description:l10n_cn.1_l10n_cn_purchase_excluded_13
#: model:account.tax,description:l10n_cn.1_l10n_cn_sales_excluded_13
#: model:account.tax,description:l10n_cn.2_l10n_cn_purchase_excluded_13
#: model:account.tax,description:l10n_cn.2_l10n_cn_sales_excluded_13
#: model:account.tax,name:l10n_cn.1_l10n_cn_purchase_excluded_13
#: model:account.tax,name:l10n_cn.1_l10n_cn_sales_excluded_13
#: model:account.tax,name:l10n_cn.2_l10n_cn_purchase_excluded_13
#: model:account.tax,name:l10n_cn.2_l10n_cn_sales_excluded_13
#: model:account.tax.template,description:l10n_cn.l10n_cn_purchase_excluded_13
#: model:account.tax.template,description:l10n_cn.l10n_cn_sales_excluded_13
#: model:account.tax.template,name:l10n_cn.l10n_cn_purchase_excluded_13
#: model:account.tax.template,name:l10n_cn.l10n_cn_sales_excluded_13
msgid "税收13%"
msgstr ""

#. module: l10n_cn
#: model:account.tax,description:l10n_cn.1_l10n_cn_sales_included_13
#: model:account.tax,description:l10n_cn.2_l10n_cn_sales_included_13
#: model:account.tax.template,description:l10n_cn.l10n_cn_sales_included_13
msgid "税收13％"
msgstr ""

#. module: l10n_cn
#: model:account.tax,name:l10n_cn.1_l10n_cn_sales_included_13
#: model:account.tax,name:l10n_cn.2_l10n_cn_sales_included_13
#: model:account.tax.template,name:l10n_cn.l10n_cn_sales_included_13
msgid "税收13％（含)"
msgstr ""

#. module: l10n_cn
#: model:account.tax,name:l10n_cn.1_l10n_cn_purchase_excluded_6
#: model:account.tax,name:l10n_cn.1_l10n_cn_sales_excluded_6
#: model:account.tax,name:l10n_cn.2_l10n_cn_purchase_excluded_6
#: model:account.tax,name:l10n_cn.2_l10n_cn_sales_excluded_6
#: model:account.tax.template,name:l10n_cn.l10n_cn_purchase_excluded_6
#: model:account.tax.template,name:l10n_cn.l10n_cn_sales_excluded_6
msgid "税收6%"
msgstr ""

#. module: l10n_cn
#: model:account.tax,description:l10n_cn.1_l10n_cn_purchase_excluded_6
#: model:account.tax,description:l10n_cn.1_l10n_cn_sales_excluded_6
#: model:account.tax,description:l10n_cn.1_l10n_cn_sales_included_6
#: model:account.tax,description:l10n_cn.2_l10n_cn_purchase_excluded_6
#: model:account.tax,description:l10n_cn.2_l10n_cn_sales_excluded_6
#: model:account.tax,description:l10n_cn.2_l10n_cn_sales_included_6
#: model:account.tax.template,description:l10n_cn.l10n_cn_purchase_excluded_6
#: model:account.tax.template,description:l10n_cn.l10n_cn_sales_excluded_6
#: model:account.tax.template,description:l10n_cn.l10n_cn_sales_included_6
msgid "税收6％"
msgstr ""

#. module: l10n_cn
#: model:account.tax,name:l10n_cn.1_l10n_cn_sales_included_6
#: model:account.tax,name:l10n_cn.2_l10n_cn_sales_included_6
#: model:account.tax.template,name:l10n_cn.l10n_cn_sales_included_6
msgid "税收6％（含)"
msgstr ""

#. module: l10n_cn
#: model:account.tax,description:l10n_cn.1_l10n_cn_purchase_excluded_9
#: model:account.tax,description:l10n_cn.1_l10n_cn_sales_excluded_9
#: model:account.tax,description:l10n_cn.2_l10n_cn_purchase_excluded_9
#: model:account.tax,description:l10n_cn.2_l10n_cn_sales_excluded_9
#: model:account.tax,name:l10n_cn.1_l10n_cn_purchase_excluded_9
#: model:account.tax,name:l10n_cn.1_l10n_cn_sales_excluded_9
#: model:account.tax,name:l10n_cn.2_l10n_cn_purchase_excluded_9
#: model:account.tax,name:l10n_cn.2_l10n_cn_sales_excluded_9
#: model:account.tax.template,description:l10n_cn.l10n_cn_purchase_excluded_9
#: model:account.tax.template,description:l10n_cn.l10n_cn_sales_excluded_9
#: model:account.tax.template,name:l10n_cn.l10n_cn_purchase_excluded_9
#: model:account.tax.template,name:l10n_cn.l10n_cn_sales_excluded_9
msgid "税收9%"
msgstr ""

#. module: l10n_cn
#: model:account.tax,description:l10n_cn.1_l10n_cn_sales_included_9
#: model:account.tax,description:l10n_cn.2_l10n_cn_sales_included_9
#: model:account.tax.template,description:l10n_cn.l10n_cn_sales_included_9
msgid "税收9％"
msgstr ""

#. module: l10n_cn
#: model:account.tax,name:l10n_cn.1_l10n_cn_sales_included_9
#: model:account.tax,name:l10n_cn.2_l10n_cn_sales_included_9
#: model:account.tax.template,name:l10n_cn.l10n_cn_sales_included_9
msgid "税收9％（含)"
msgstr ""

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.osv import expression

try:
    from cn2an import an2cn
except ImportError:
    an2cn = None

class AccountMove(models.Model):
    _inherit = 'account.move'

    fapiao = fields.Char(string='Fapiao Number', size=8, copy=False, tracking=True)

    @api.constrains('fapiao')
    def _check_fapiao(self):
        for record in self:
            if record.fapiao and (len(record.fapiao) != 8 or not record.fapiao.isdecimal()):
                raise ValidationError(_("Fapiao number is an 8-digit number. Please enter a correct one."))

    @api.model
    def check_cn2an(self):
        return an2cn

    @api.model
    def _convert_to_amount_in_word(self, number):
        """Convert number to ``amount in words`` for Chinese financial usage."""
        if not self.check_cn2an():
            return None
        return an2cn(number, 'rmb')

    def _count_attachments(self):
        domains = [[('res_model', '=', 'account.move'), ('res_id', '=', self.id)]]
        statement_ids = self.line_ids.mapped('statement_id')
        payment_ids = self.line_ids.mapped('payment_id')
        if statement_ids:
            domains.append([('res_model', '=', 'account.bank.statement'), ('res_id', 'in', statement_ids.ids)])
        if payment_ids:
            domains.append([('res_model', '=', 'account.payment'), ('res_id', 'in', payment_ids.ids)])
        return self.env['ir.attachment'].search_count(expression.OR(domains))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="account_move_form_l10n_cn" model="ir.ui.view">
            <field name="name">l10n_cn.account.move.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='ref']" position="after">
                    <field name="move_type" invisible='1'/>
                    <field name="fapiao" attrs="{'invisible': ['|', ('country_code','!=', 'CN'),
                        ('move_type', 'not in', ['out_invoice', 'out_refund', 'in_invoice', 'in_refund'])]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- QWeb Reports -->
        <record id="account_voucher_cn" model="ir.actions.report">
            <field name="name">Voucher</field>
            <field name="model">account.move</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">l10n_cn.report_voucher</field>
            <field name="report_file">l10n_cn.report_voucher</field>
            <field name="print_report_name">'Voucher_%s' % (object.name)</field>
            <field name="binding_view_types">form</field>
            <field name="binding_model_id" ref="model_account_move"/>
        </record>
    </data>
</odoo>

```

## File: views\report_voucher.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="external_layout_boxed" inherit_id="web.external_layout_boxed" primary="True">
            <xpath expr="//div[hasclass('o_boxed_header')]" position="replace">
                <div class="o_boxed_header">
                <div class="row mb8">
                    <div class="col-3 mb4">
                        <img t-if="company.logo" t-att-src="image_data_uri(company.logo)" style="max-height: 45px;" alt="Logo"/>
                    </div>
                </div>
                </div>
            </xpath>
        </template>

        <template id="report_voucher_document">
            <t t-set="o" t-value="o.with_context(lang=lang)" />
            <t t-set="company" t-value="o.company_id"/>

            <t t-call="l10n_cn.external_layout_boxed">
                <div class="page">

                    <div align="center">
                        <h2>
                            <span>记账凭证</span>
                        </h2>
                    </div>

                    <div id="company" class="row col-auto">
                        <span t-field="o.company_id.name"/>
                    </div>
                    <div id="informations" class="row">
                        <!-- offset intentionally for period -->
                        <div class="col-3 offset-3" name="date">
                            <strong>日期：</strong>
                            <span t-field="o.date"/>
                        </div>
                        <div class="col-4" t-if="o.name" name="name">
                            <strong>凭证号：</strong>
                            <span t-field="o.name"/>
                        </div>
                        <div class="col-2">
                            <strong>附件数：</strong>
                            <span t-esc="o._count_attachments()"/>
                        </div>
                    </div>

                    <table class="table table-sm o_main_table table-striped" name="entry_line_table">
                        <thead>
                            <tr>
                                <t t-set="colspan" t-value="4"/>
                                <th name="th_description" class="text-center"><span>摘要</span></th>
                                <th name="th_account" class="text-center"><span>科目</span></th>
                                <th name="th_debit" class="text-center"><span>借方</span></th>
                                <th name="th_credit" class="text-center"><span>贷方</span></th>
                            </tr>
                        </thead>
                        <tbody class="invoice_tbody">
                            <t t-set="total_debit" t-value="0"/>
                            <t t-set="total_credit" t-value="0"/>

                            <t t-foreach="o.line_ids" t-as="line">
                                <t t-set="total_debit" t-value="total_debit + line.debit"/>
                                <t t-set="total_credit" t-value="total_credit + line.credit"/>
                                <tr>
                                    <t name="account_move_line">
                                        <td name="description">
                                            <span t-field="line.name" t-options="{'widget': 'text'}"/>
                                        </td>
                                        <td name="account">
                                            <span t-field="line.account_id.display_name" t-options="{'widget': 'text'}"/>
                                        </td>
                                        <td name="debit">
                                            <span t-if="line.debit != 0" t-field="line.debit"/>
                                        </td>
                                        <td name="credit">
                                            <span t-if="line.credit != 0" t-field="line.credit"/>
                                        </td>
                                    </t>
                                </tr>
                            </t>
                            <t>
                                <td name="total" colspan="2">
                                    <span>合计：</span>
                                    <span t-esc="o._convert_to_amount_in_word(total_debit)" />
                                </td>
                                <td name="total_debit">
                                    <span t-esc="total_debit" t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                                </td>
                                <td name="total_credit">
                                    <span t-esc="total_credit" t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                                </td>
                            </t>
                        </tbody>
                    </table>

                    <div id="staff" class="row" style="color:black">
                        <div class="col-4">
                            <strong>审核：</strong>
                        </div>
                        <div class="col-4">
                            <strong>过账：</strong>
                        </div>
                        <div class="col-4">
                            <strong>制单：</strong>
                            <span t-esc="o.invoice_user_id.name"/>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="report_voucher">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="o">
                    <t t-call="l10n_cn.report_voucher_document" t-lang="lang"/>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

