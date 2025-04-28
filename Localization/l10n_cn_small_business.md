# Odoo Module: l10n_cn_small_business

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2007-2014 Jeff Wang(<http://jeff@osbzr.com>).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2008-2008 凯源吕鑫 lvxin@gmail.com   <basic chart data>
#                         维智众源 oldrev@gmail.com  <states data>
# Copyright (C) 2012-2012 南京盈通 ccdos@intoerp.com <small business chart>
# Copyright (C) 2008-now  开阖软件 jeff@osbzr.com    < PM and LTS >

{
    'name': 'China - Small Business CoA',
    'version': '1.8',
    'category': 'Localization',
    'author': 'www.openerp-china.org',
    'maintainer': 'jeff@osbzr.com',
    'website': 'http://openerp-china.org',
    'description': """

    科目类型\会计科目表模板\增值税\辅助核算类别\管理会计凭证簿\财务会计凭证簿

    添加中文省份数据

    增加小企业会计科目表

    """,
    'depends': ['l10n_cn'],
    'data': [
        'data/l10n_cn_small_business_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_cn_small_business_chart_post_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_data.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"l10n_cn_1012","Other Monetary Funds","1012","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1101","Transactional Financial Assets","1101","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1121","Bills Receivable","1121","account.data_account_type_receivable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_1122","Accounts Receivable","1122","account.data_account_type_receivable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_1123","Advance Payment","1123","account.data_account_type_prepayments","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1124","Accounts Receivable (PoS)","1124","account.data_account_type_receivable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_1131","Divident Receivable","1131","account.data_account_type_receivable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_1132","Interest Receivable","1132","account.data_account_type_receivable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_1221","Other Receivable","1221","account.data_account_type_receivable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_1231","Bad Debt Provisions","1231","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1401","Material Purchasing","1401","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1402","Materials in transit","1402","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1403","Raw Material","1403","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1404","Material Cost Variance","1404","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1405","Merchandise Inventory","1405","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1406","Goods shipped in transit","1406","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1407","Differences between purchasing and selling price","1407","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1408","Consigned processing materials","1408","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1471","Inventory falling price reserves","1471","account.data_account_type_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1501","Held to maturity Investment","1501","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1502","Provision for impairment of investments held to maturity","1502","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1503","Available for sale financial assets","1503","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1511","Long-term equity investment","1511","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1512","Impairment provision for long-term equity investments","1512","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1521","Investmental real estate","1521","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1531","Long-term receivables","1531","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_1601","Fixed assets","1601","account.data_account_type_fixed_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1602","Accumulated depreciation","1602","account.data_account_type_depreciation","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1603","Fixed assets depreciation reserves","1603","account.data_account_type_depreciation","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1604","Construction in progress","1604","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1605","Engineering materials","1605","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1606","Liquidation of fixed assets","1606","account.data_account_type_depreciation","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1701","Intangible Assets","1701","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1702","Accumulated amortization","1702","account.data_account_type_depreciation","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1703","Intangible Assets Depreciation Reserves","1703","account.data_account_type_depreciation","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1711","Goodwill","1711","account.data_account_type_non_current_assets","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_1801","Long-term amortized expenses","1801","account.data_account_type_depreciation","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_2001","Short-term borrowing","2001","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2201","Bills Payable","2201","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2202","Accounts Payable","2202","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2203","Deposit Received","2203","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2211","Payroll payable","2211","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2221","Tax payable","2221","account.data_account_type_current_liabilities","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2231","Interest payable","2231","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2241","Dividents payable","2241","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2501","Other payable","2501","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2502","Bonds Payable","2502","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2701","Long Term payables","2701","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2711","Account payable special funds","2711","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2801","Projected liabilities","2801","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_2901","Deferred Tax Liability","2901","account.data_account_type_payable","l10n_cn_small_business.l10n_chart_china_small_business","True"
"l10n_cn_4001","Paid in capital","4001","account.data_account_type_equity","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_4002","Capital Surplus","4002","account.data_account_type_equity","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_4003","Other Comprehensive Income","4003","account.data_account_type_equity","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_4101","Surplus Reserve","4101","account.data_account_type_equity","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_4103","Profit for the year","4103","account.data_account_type_equity","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_4104","Profit distribution","4104","account.data_account_type_equity","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_5001","Production Costs","5001","account.data_account_type_direct_costs","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_5101","Manufacturing Expenses","5101","account.data_account_type_direct_costs","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_5201","Service Cost","5201","account.data_account_type_direct_costs","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_5301","R & D expenditure","5301","account.data_account_type_direct_costs","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6001","Main Business Income","6001","account.data_account_type_revenue","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6051","Other Business Income","6051","account.data_account_type_other_income","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6101","Gains and Losses of fair value change","6101","account.data_account_type_other_income","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6111","Income from investment","6111","account.data_account_type_other_income","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6301","Non-operating Income","6301","account.data_account_type_other_income","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6401","Main Business Cost","6401","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6402","Other Operating Costs","6402","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6403","Operating Taxes and Surcharges","6403","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6601","Selling Expenses","6601","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6602","Management Expenses","6602","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6603","Financial Expenses","6603","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6701","Assets impairment Loss","6701","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6711","Non-operating expenses","6711","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6801","Income Tax Expense","6801","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"
"l10n_cn_6901","Prior year income adjustment","6901","account.data_account_type_expenses","l10n_cn_small_business.l10n_chart_china_small_business","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_cn_small_business.l10n_chart_china_small_business')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- sales tax included -->
    <record id="l10n_cn_small_business_sales_included_17" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收17％（含） - 中国小企业会计科目表</field>
        <field name="description">税收17％</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_17"/>
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
    <record id="l10n_cn_small_business_sales_included_11" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收11％（含） - 中国小企业会计科目表</field>
        <field name="description">税收11％</field>
        <field name="amount">11</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_11"/>
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
    <record id="l10n_cn_small_business_sales_included_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收6％（含） - 中国小企业会计科目表</field>
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
    <record id="l10n_cn_small_business_sales_included_3" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收3％（含） - 中国小企业会计科目表</field>
        <field name="description">税收3％</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_3"/>
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
    <record id="l10n_cn_small_business_sales_excluded_17" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收17％ - 中国小企业会计科目表</field>
        <field name="description">税收17％</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_17"/>
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
    <record id="l10n_cn_small_business_sales_excluded_11" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收11％ - 中国小企业会计科目表</field>
        <field name="description">税收11％</field>
        <field name="amount">11</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_11"/>
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
    <record id="l10n_cn_small_business_sales_excluded_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收6％ - 中国小企业会计科目表</field>
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
    <record id="l10n_cn_small_business_sales_excluded_3" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收3％ - 中国小企业会计科目表</field>
        <field name="description">税收3％</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_3"/>
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
    <record id="l10n_cn_small_business_purchase_excluded_17" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收17％ - 中国小企业会计科目表</field>
        <field name="description">税收17％</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_17"/>
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
    <record id="l10n_cn_small_business_purchase_excluded_11" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收11％ - 中国小企业会计科目表</field>
        <field name="description">税收11％</field>
        <field name="amount">11</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_11"/>
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
    <record id="l10n_cn_small_business_purchase_excluded_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收6％ - 中国小企业会计科目表</field>
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
    <record id="l10n_cn_small_business_purchase_excluded_3" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_small_business"/>
        <field name="name">税收3％ - 中国小企业会计科目表</field>
        <field name="description">税收3％</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="l10n_cn.l10n_cn_tax_group_vat_3"/>
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

## File: data\l10n_cn_small_business_chart_data.xml

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

## File: data\l10n_cn_small_business_chart_post_data.xml

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

## File: i18n_extra\l10n_cn_small_business.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
#	* l10n_cn_small_business
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 12.0alpha1+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2017-11-29 10:31+0000\n"
"PO-Revision-Date: 2017-11-29 10:31+0000\n"
"Last-Translator: <>\n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2711
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2711
msgid "專項應付款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6401
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6401
msgid "主營業務成本"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6001
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6001
msgid "主營業務收入"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1101
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1101
msgid "交易性金融資産"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6901
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6901
msgid "以前年度損益調整"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6101
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6101
msgid "公允價值變動損益"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6402
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6402
msgid "其他業務成本"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6051
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6051
msgid "其他業務收入"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2501
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2501
msgid "其他應付款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1221
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1221
msgid "其他應收款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_4003
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_4003
msgid "其他綜合收益"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1012
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1012
msgid "其他貨幣資金"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_4104
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_4104
msgid "利潤分配"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_5101
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_5101
msgid "制造費用"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_5201
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_5201
msgid "勞務成本"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1403
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1403
msgid "原材料"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1406
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1406
msgid "發出商品"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1503
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1503
msgid "可供出售金融資産"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1407
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1407
msgid "商品進銷差價"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1711
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1711
msgid "商譽"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1601
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1601
msgid "固定資産"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1603
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1603
msgid "固定資産减值準備"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1606
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1606
msgid "固定資産情况"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1604
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1604
msgid "在建工程"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1402
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1402
msgid "在途物資"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1231
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1231
msgid "壞賬準備"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_11
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_11
msgid "税收11％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_11
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_11
msgid "税收11％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_11
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_11
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_11
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_included_11
msgid "税收11％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_11
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_11
msgid "税收11％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_11
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_included_11
msgid "税收11％（含） - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_17
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_17
msgid "税收17％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_17
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_17
msgid "税收17％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_17
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_17
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_17
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_included_17
msgid "税收17％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_17
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_17
msgid "税收17％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_17
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_included_17
msgid "税收17％（含） - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_3
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_3
msgid "税收3％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_3
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_3
msgid "税收3％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_3
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_3
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_3
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_included_3
msgid "税收3％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_3
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_3
msgid "税收3％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_3
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_included_3
msgid "税收3％（含） - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_6
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_6
msgid "税收6％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_purchase_excluded_6
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_purchase_excluded_6
msgid "税收6％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_6
#: model:account.tax,description:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_6
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_6
#: model:account.tax.template,description:l10n_cn_small_business.l10n_cn_small_business_sales_included_6
msgid "税收6％"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_excluded_6
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_excluded_6
msgid "税收6％ - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.tax,name:l10n_cn_small_business.1_l10n_cn_small_business_sales_included_6
#: model:account.tax.template,name:l10n_cn_small_business.l10n_cn_small_business_sales_included_6
msgid "税收6％（含） - 中国小企业会计科目表"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1408
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1408
msgid "委托加工物資"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1471
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1471
msgid "存貨跌價準備"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_4001
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_4001
msgid "實收資本"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.chart.template,name:l10n_cn_small_business.l10n_chart_china_small_business
msgid "小企业会计科目表（财会[2011]17号《小企业会计准则》）"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1605
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1605
msgid "工程物資"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1405
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1405
msgid "庫存商品"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2221
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2221
msgid "應交税費"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2502
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2502
msgid "應付債券"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2231
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2231
msgid "應付利息"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2201
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2201
msgid "應付票據"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2211
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2211
msgid "應付職工薪酬"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2241
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2241
msgid "應付股利"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1122
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1122
msgid "應付賬款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2202
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2202
msgid "應付賬款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1132
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1132
msgid "應收利息"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1121
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1121
msgid "應收票據"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1131
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1131
msgid "應收股利"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6801
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6801
msgid "所得税費用"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1521
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1521
msgid "投資性房地産"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6111
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6111
msgid "投資收益"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1501
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1501
msgid "持有至到期投資"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1502
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1502
msgid "持有至到期投資减值準備"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1701
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1701
msgid "無形資産"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1703
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1703
msgid "無形資産减值準備"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_4103
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_4103
msgid "本年利潤"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1404
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1404
msgid "材料成本差异"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1401
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1401
msgid "材料采購"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_5001
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_5001
msgid "生産成本"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_4101
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_4101
msgid "盈餘公積"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2001
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2001
msgid "短期借款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_5301
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_5301
msgid "研發支出"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6602
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6602
msgid "管理費用"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1602
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1602
msgid "累計折舊"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1702
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1702
msgid "累計攤銷"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6711
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6711
msgid "營業外支出"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6301
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6301
msgid "營業外收入"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6403
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6403
msgid "營業税及附加"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6603
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6603
msgid "財務費用"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6701
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6701
msgid "資産减值損失"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_4002
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_4002
msgid "資本公積金"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2901
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2901
msgid "遞延所得税負債"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_6601
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_6601
msgid "銷售費用"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2701
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2701
msgid "長期應付款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1531
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1531
msgid "長期應收款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1801
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1801
msgid "長期待攤銷費用"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1511
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1511
msgid "長期股權投資"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1512
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1512
msgid "長期股權投資减值準備"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_1123
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_1123
msgid "預付賬款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2203
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2203
msgid "預收賬款"
msgstr ""

#. module: l10n_cn_small_business
#: model:account.account,name:l10n_cn_small_business.1_l10n_cn_2801
#: model:account.account.template,name:l10n_cn_small_business.l10n_cn_2801
msgid "預計負債"
msgstr ""


```

